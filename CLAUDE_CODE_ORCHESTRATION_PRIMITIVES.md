# Claude Code Orchestration Primitives Design

**Date**: 2025-10-27
**Purpose**: Enable validated orchestration patterns from experiment
**Based On**: 5-phase empirical study (θ measurements, parallel vs linear topologies)

---

## Executive Summary

This experiment validated that **parallel orchestration reduces coordination overhead by 90-93%** (θ=0.097 vs 0.867-1.46 for linear patterns). To enable these workflows in Claude Code, we need new primitives for:

1. **Parallel agent execution** (critical - enables 540% speedup)
2. **Topology definitions** (declarative orchestration patterns)
3. **Metrics tracking** (automatic θ calculation and monitoring)
4. **Pattern library** (reusable N=3, N=4, N=5 configurations)
5. **Smart routing** (automatic pattern selection)

---

## 1. Orchestration Topology Definitions

### New Component: `.claude/topologies/`

**Purpose**: Declarative orchestration pattern definitions

**File Structure**: YAML-based topology files

```yaml
# .claude/topologies/parallel-ui-logic.yaml
name: "parallel-ui-logic"
description: "Parallel tracks for UI/Visual and Logic/Integration work"
version: "1.0"

topology:
  type: "parallel"  # or "linear", "hybrid", "dag"

  tracks:
    - name: "ui-visual"
      agents:
        - name: "ui-designer"
          role: "Design GUI components and interactions"
          outputs: ["ui-spec.md", "gui-mockups/"]

        - name: "rendering-engineer"
          role: "Implement visual rendering code"
          inputs: ["ui-spec.md"]
          outputs: ["rendering-code/"]
          tools: ["Read", "Write", "Edit", "Bash"]

    - name: "logic-integration"
      agents:
        - name: "logic-developer"
          role: "Implement business logic and algorithms"
          outputs: ["logic-code/"]

        - name: "integration-engineer"
          role: "Wire all components together"
          inputs: ["rendering-code/", "logic-code/"]
          outputs: ["integrated-code/"]
          tools: ["Read", "Write", "Edit", "Grep", "Bash"]

  merge:
    strategy: "manual"  # or "auto", "review"
    validator: "code-reviewer"  # optional agent for post-merge review

metrics:
  track_theta: true
  classify_agents:
    work: ["rendering-engineer", "logic-developer", "integration-engineer"]
    coordination: ["ui-designer"]
  alert_threshold: 1.5  # warn if θ > 1.5
```

---

## 2. Parallel Agent Execution System

### New Tool: `TaskParallel`

**Purpose**: Launch multiple agents simultaneously (critical for θ=0.097 performance)

**Usage in Claude Code**:
```typescript
// Invoke via extended Task tool
{
  "tool": "TaskParallel",
  "topology": "parallel-ui-logic",  // reference to .claude/topologies/
  "context": {
    "feature": "Add user authentication",
    "requirements": "OAuth2, JWT tokens, session management"
  }
}
```

**Behavior**:
1. Parse topology definition
2. Launch all parallel track agents simultaneously
3. Wait for all to complete (or timeout)
4. Collect outputs from each agent
5. Execute merge strategy
6. Return integrated result

**Output**:
```json
{
  "status": "complete",
  "duration": 127,
  "metrics": {
    "theta": 0.097,
    "work_time": 331,
    "coordination_time": 32,
    "parallelism_efficiency": 2.6
  },
  "results": {
    "ui-visual": { "duration": 78, "outputs": [...] },
    "logic-integration": { "duration": 95, "outputs": [...] }
  },
  "integrated_output": "path/to/merged/code"
}
```

---

## 3. Agent Interface Contracts

### New Component: `.claude/contracts/`

**Purpose**: Formal agent communication protocols (enables clean parallel execution)

**File Structure**: JSON Schema or TypeScript interfaces

```typescript
// .claude/contracts/ui-spec.contract.ts
export interface UISpecification {
  version: string;
  components: {
    name: string;
    type: "button" | "input" | "dropdown" | "custom";
    properties: Record<string, any>;
    events: string[];
  }[];
  layouts: {
    name: string;
    grid: string;  // CSS grid template
    responsive: boolean;
  }[];
  interactions: {
    trigger: string;
    action: string;
    target: string;
  }[];
}
```

**Validation Hook**:
```json
{
  "PreToolUse": {
    "matcher": "Write",
    "command": "validate-contract --file ${FILE} --contract ui-spec.contract.ts"
  }
}
```

---

## 4. Orchestration Pattern Library

### New Skill: `orchestration-patterns`

**Purpose**: Reusable, validated patterns from experiment

**Structure**:
```
.claude/skills/orchestration-patterns/
  ├── SKILL.md
  ├── patterns/
  │   ├── linear-n3.yaml         # θ=0.867, balanced
  │   ├── parallel-n4.yaml       # θ=0.097, optimal for decomposable
  │   ├── linear-n5.yaml         # θ=~1.4, comprehensive QA
  │   └── custom-template.yaml
  ├── metrics/
  │   └── theta-calculator.py    # Automatic θ calculation
  └── docs/
      └── pattern-selection.md   # Decision framework
```

**SKILL.md**:
```markdown
---
name: orchestration-patterns
description: |
  Use this skill when implementing features that could benefit from
  orchestrated agent collaboration. Triggers on: "implement feature",
  "add functionality", "build component", or when task requires >3 steps.

  MUST BE USED proactively to recommend optimal orchestration pattern
  based on feature characteristics.
allowed-tools: [Read, Grep, Bash]
---

# Orchestration Patterns Skill

## Pattern Selection Decision Tree

1. **Is feature decomposable?** (clear UI/logic/integration boundaries)
   - YES → Recommend `parallel-n4.yaml` (θ=0.097, fastest)
   - NO → Continue...

2. **Is feature complex?** (>500 LOC, multi-subsystem)
   - YES → Recommend `linear-n5.yaml` (θ=~1.4, comprehensive)
   - NO → Continue...

3. **Is quality critical?** (production, user-facing)
   - YES → Recommend `linear-n3.yaml` (θ=0.867, balanced)
   - NO → Recommend solo development

## Usage

```bash
# Check available patterns
ls .claude/skills/orchestration-patterns/patterns/

# Calculate θ for current session
python .claude/skills/orchestration-patterns/metrics/theta-calculator.py \
  --session-log .claude/session.log
```
```

---

## 5. Metrics Tracking & Monitoring

### New Hook: `PostToolUse` → θ Tracker

**Purpose**: Automatic coordination overhead measurement

```json
{
  "PostToolUse": {
    "matcher": "*",
    "command": ".claude/hooks/theta-tracker.sh"
  }
}
```

**theta-tracker.sh**:
```bash
#!/bin/bash
# Track agent timing and classify as work vs coordination

AGENT_NAME="${AGENT_NAME:-unknown}"
DURATION="${DURATION:-0}"
TOOL="${TOOL:-unknown}"

# Classify agent type
if [[ "$AGENT_NAME" =~ (developer|engineer|implementor) ]]; then
  CATEGORY="work"
elif [[ "$AGENT_NAME" =~ (architect|designer|reviewer|tester|pm) ]]; then
  CATEGORY="coordination"
else
  CATEGORY="unknown"
fi

# Log to metrics file
echo "$TIMESTAMP,$AGENT_NAME,$DURATION,$CATEGORY,$TOOL" >> .claude/metrics/theta.csv

# Calculate current θ
WORK_TIME=$(awk -F',' '$4=="work" {sum+=$3} END {print sum}' .claude/metrics/theta.csv)
COORD_TIME=$(awk -F',' '$4=="coordination" {sum+=$3} END {print sum}' .claude/metrics/theta.csv)

if [ "$WORK_TIME" -gt 0 ]; then
  THETA=$(echo "scale=3; $COORD_TIME / $WORK_TIME" | bc)

  # Alert if θ > threshold
  if (( $(echo "$THETA > 1.5" | bc -l) )); then
    echo "⚠️  ALERT: θ=$THETA exceeds threshold (1.5)" >&2
    echo "Consider switching to parallel orchestration" >&2
  fi
fi
```

### New Command: `/metrics`

**Purpose**: Real-time orchestration performance dashboard

```markdown
# .claude/commands/metrics.md
---
name: metrics
description: Show orchestration metrics (θ, duration, efficiency)
---

Display current session orchestration metrics:

```bash
#!/bin/bash
python3 << 'EOF'
import pandas as pd
import sys

# Read metrics
df = pd.read_csv('.claude/metrics/theta.csv',
                 names=['timestamp', 'agent', 'duration', 'category', 'tool'])

# Calculate metrics
work_time = df[df['category'] == 'work']['duration'].sum()
coord_time = df[df['category'] == 'coordination']['duration'].sum()
theta = coord_time / work_time if work_time > 0 else 0

# Display
print(f"📊 Orchestration Metrics")
print(f"=" * 40)
print(f"Work Time:        {work_time}s")
print(f"Coordination Time: {coord_time}s")
print(f"θ (overhead):     {theta:.3f} ({theta*100:.1f}%)")
print(f"")
print(f"Agent Breakdown:")
for agent, group in df.groupby('agent'):
    dur = group['duration'].sum()
    cat = group['category'].iloc[0]
    print(f"  {agent:20s} {dur:5.0f}s  [{cat}]")
EOF
```
```

---

## 6. Smart Pattern Routing

### New Agent: `orchestration-advisor`

**Purpose**: Automatic pattern selection based on task analysis

```markdown
# .claude/agents/orchestration-advisor.md
---
name: orchestration-advisor
description: |
  MUST BE USED proactively when user requests feature implementation.
  Analyzes task characteristics and recommends optimal orchestration
  pattern. Triggers on: "implement", "add feature", "build", "create".
tools: Read,Grep,Bash
model: sonnet
---

# Orchestration Advisor

You are an expert at analyzing software tasks and recommending optimal
orchestration patterns based on empirical θ measurements.

## Decision Criteria

Analyze the requested feature and extract:

1. **Decomposability Score** (0-10)
   - Can UI, logic, integration be separated?
   - Are interfaces clear and stable?
   - Are dependencies unidirectional?

2. **Complexity Score** (0-10)
   - Lines of code estimate
   - Number of subsystems involved
   - External dependencies count

3. **Criticality Score** (0-10)
   - Production vs prototype
   - User-facing vs internal
   - Regulatory requirements

4. **Time Constraints** (0-10)
   - Deadline urgency
   - Opportunity cost of delay
   - Current sprint capacity

## Pattern Recommendations

Based on scores, recommend:

- **Parallel N=4** if decomposability > 7 and time_constraints > 6
  - θ=0.097, duration ≈ 127s
  - 62% faster than solo
  - Best for decomposable, time-critical features

- **Linear N=3** if criticality > 7 or decomposability < 5
  - θ=0.867, duration ≈ 351s
  - Balanced speed/quality
  - Safe default for most work

- **Linear N=5** if criticality > 9 or complexity > 8
  - θ=~1.4, duration ≈ 656s
  - Comprehensive QA
  - Complex/critical features only

- **Solo N=1** if complexity < 3 and developer_experience > 8
  - θ=0, duration ≈ 335s
  - Fast for simple features

## Output Format

```yaml
recommendation:
  pattern: "parallel-n4"
  confidence: 0.95
  rationale: |
    Feature is highly decomposable (UI/logic/integration separate),
    time-critical (tight deadline), and has clear interfaces.
    Parallel N=4 reduces coordination overhead to 9.7% while
    maintaining zero-bug quality.

  metrics_prediction:
    theta: 0.097
    duration: "~130s"
    speedup_vs_solo: "62% faster"

  topology: ".claude/topologies/parallel-ui-logic.yaml"
```
```

---

## 7. Merge Strategies

### New Tool: `MergeParallel`

**Purpose**: Automatic integration of parallel agent outputs

**Strategies**:

```yaml
# .claude/merge-strategies/auto-integrate.yaml
name: "auto-integrate"
description: "Automatic code integration from parallel tracks"

steps:
  - name: "validate-outputs"
    check: "all agents completed successfully"

  - name: "resolve-conflicts"
    strategy: "three-way-merge"
    base: "original-baseline"
    branches: ["track-a-output", "track-b-output"]

  - name: "run-tests"
    command: "npm test"
    required: true

  - name: "syntax-check"
    tools: ["node -c", "eslint", "prettier"]

  - name: "integration-review"
    agent: "code-reviewer"
    optional: true

output:
  path: "integrated/"
  preserve_originals: true
  create_commit: true
  commit_message: "Merge parallel tracks: ${TRACK_A} + ${TRACK_B}"
```

---

## 8. Topology Visualizer

### New Command: `/visualize-topology`

**Purpose**: Render orchestration topology diagrams

```bash
# .claude/commands/visualize-topology.sh
#!/bin/bash

TOPOLOGY_FILE="$1"

python3 << EOF
import yaml
import graphviz

with open('$TOPOLOGY_FILE') as f:
    topology = yaml.safe_load(f)

dot = graphviz.Digraph(comment=topology['name'])
dot.attr(rankdir='TB')

# Add nodes for each agent
for track in topology['topology']['tracks']:
    with dot.subgraph(name=f"cluster_{track['name']}") as c:
        c.attr(label=track['name'])
        for agent in track['agents']:
            c.node(agent['name'], label=f"{agent['name']}\n{agent['role']}")

# Add edges for data flow
# ... (render based on inputs/outputs)

dot.render('/tmp/topology-viz', format='png', view=True)
EOF
```

---

## 9. Real-time θ Alerts

### New Hook: `PostToolUse` → θ Alert

```json
{
  "PostToolUse": {
    "matcher": "*",
    "command": ".claude/hooks/theta-alert.sh"
  }
}
```

**theta-alert.sh**:
```bash
#!/bin/bash
# Alert if θ exceeds threshold during execution

THRESHOLD=1.5
CURRENT_THETA=$(calculate-current-theta)  # from metrics

if (( $(echo "$CURRENT_THETA > $THRESHOLD" | bc -l) )); then
  cat << EOF
⚠️  ORCHESTRATION ALERT

Current θ: $CURRENT_THETA (exceeds threshold: $THRESHOLD)

This indicates over-coordination. Consider:
1. Switching to parallel topology (reduces θ by 90%)
2. Merging redundant agents
3. Reducing handoff documentation overhead

Recommended action:
  Use /orchestration-patterns to find better pattern
EOF
fi
```

---

## 10. Pattern Performance Benchmarks

### New Skill Component: `benchmarks/`

**Purpose**: Track pattern performance over time

```
.claude/skills/orchestration-patterns/benchmarks/
  ├── linear-n3-history.csv
  ├── parallel-n4-history.csv
  └── report-generator.py
```

**report-generator.py**:
```python
#!/usr/bin/env python3
"""
Generate orchestration pattern performance report
"""
import pandas as pd
import matplotlib.pyplot as plt

# Load historical data
patterns = {
    'Linear N=3': pd.read_csv('linear-n3-history.csv'),
    'Parallel N=4': pd.read_csv('parallel-n4-history.csv'),
    'Linear N=5': pd.read_csv('linear-n5-history.csv'),
}

# Calculate statistics
for name, df in patterns.items():
    print(f"{name}:")
    print(f"  Mean θ: {df['theta'].mean():.3f}")
    print(f"  Median duration: {df['duration'].median():.0f}s")
    print(f"  Bug rate: {(df['bugs'] > 0).mean()*100:.1f}%")
    print()

# Visualization
plt.figure(figsize=(10, 6))
for name, df in patterns.items():
    plt.scatter(df['complexity'], df['theta'], label=name, alpha=0.6)
plt.xlabel('Feature Complexity (LOC)')
plt.ylabel('θ (Coordination Overhead)')
plt.legend()
plt.title('Orchestration Pattern Performance')
plt.savefig('pattern-performance.png')
```

---

## Implementation Priority

### Phase 1: Critical (Enables Core Workflow)
1. ✅ **TaskParallel tool** - Parallel agent execution
2. ✅ **Topology definitions** - Declarative patterns
3. ✅ **θ metrics tracking** - PostToolUse hook

### Phase 2: Enhanced (Improves UX)
4. **orchestration-advisor agent** - Smart routing
5. **orchestration-patterns skill** - Pattern library
6. **MergeParallel tool** - Auto-integration

### Phase 3: Polish (Power User Features)
7. **Interface contracts** - Type-safe agent communication
8. **/metrics command** - Performance dashboard
9. **/visualize-topology** - Topology diagrams
10. **Benchmark tracking** - Historical analysis

---

## Usage Example: End-to-End

```typescript
// User request
"Implement user authentication with OAuth2"

// 1. orchestration-advisor agent analyzes task
{
  decomposability: 8,   // UI, logic, integration separable
  complexity: 7,        // Medium complexity
  criticality: 9,       // User-facing, security critical
  time_constraints: 7   // Moderate urgency
}
// → Recommends: Parallel N=4

// 2. Load topology
topology: ".claude/topologies/parallel-n4-auth.yaml"

// 3. Execute via TaskParallel
{
  track_a: ["ui-designer", "oauth-flow-ui"],
  track_b: ["auth-logic-dev", "integration-engineer"]
}
// All 4 agents run simultaneously

// 4. θ tracker monitors in real-time
// No alerts (θ=0.095, well below 1.5 threshold)

// 5. MergeParallel integrates outputs
strategy: "auto-integrate"
// Tests pass, code merged

// 6. Report metrics
{
  theta: 0.095,
  duration: 134s,
  speedup_vs_solo: "60% faster",
  bugs: 0
}
```

---

## Key Design Principles

### 1. Modularity
- Each component (topology, contract, merge strategy) is independent
- Can mix and match: parallel topology + manual merge + auto metrics

### 2. Compositionality
- Topologies reference agents, agents use contracts, hooks track metrics
- Build complex workflows from simple primitives

### 3. Flexibility
- Support linear, parallel, hybrid, and custom DAG topologies
- User can override any default (merge strategy, θ threshold, etc.)

### 4. Accuracy
- Metrics based on empirical measurements (5-phase experiment)
- θ thresholds derived from actual data (θ<1.0 good, θ>1.5 bad)

### 5. Power
- Enables 540% speedup (Phase 4 vs Phase 5)
- Reduces coordination overhead by 90-93%
- Maintains zero-bug quality

---

## Validation Against Experiment

This design enables all validated patterns:

| Pattern | Enabled By | Expected θ |
|---------|------------|------------|
| Solo N=1 | Default (no orchestration) | 0.0 |
| Linear N=3 | `linear-n3.yaml` topology | 0.867 |
| Linear N=5 | `linear-n5.yaml` topology | ~1.4 |
| **Parallel N=4** ✅ | **TaskParallel + parallel-n4.yaml** | **0.097** |
| Fine-grained N=8 | `fine-grained-n8.yaml` (anti-pattern) | 1.46 |

**Critical Achievement**: TaskParallel enables the 90-93% θ reduction that made Phase 4 optimal.

---

**Status**: Design complete, ready for implementation
**Next**: Prototype TaskParallel tool and parallel topology execution
