# Prompt for New Session: Claude Code Orchestration Infrastructure Design

You are tasked with designing Claude Code primitives to enable the orchestration workflow validated in this experiment. **CRITICAL**: You must first thoroughly learn Claude Code's actual capabilities before designing anything.

## Phase 1: Deep Learning (REQUIRED FIRST)

### 1.1 Study Claude Code Documentation Comprehensively

Read and synthesize these docs (use WebFetch systematically):

**Core Agent System:**
- `/docs/claude-code/sub-agents.md` - How agents are defined, invoked, and executed
  - **CRITICAL QUESTIONS**:
    - Can multiple agents execute in parallel? How?
    - What's the invocation mechanism (Task tool parameters)?
    - How do agents communicate results back?
    - Can agents communicate with each other during execution?
    - What isolation exists between parallel agents?

**Extensibility Infrastructure:**
- `/docs/claude-code/skills.md` - Skill system architecture
- `/docs/claude-code/slash-commands.md` - Command creation and usage
- `/docs/claude-code/hooks-guide.md` + `/docs/claude-code/hooks.md` - Event-driven capabilities
- `/docs/claude-code/plugins.md` - Plugin architecture
- `/docs/claude-code/output-styles.md` - Response customization

**Integration Points:**
- `/docs/claude-code/mcp.md` - Model Context Protocol servers
- Look for any message bus, coordination, or communication primitives

### 1.2 Test Your Understanding

After reading, answer these questions WITHOUT GUESSING:
1. Does Task tool support parallel agent execution? If yes, how is it invoked?
2. What parameters does Task tool accept? Can you pass multiple agents in one call?
3. How do parallel agents share context or communicate?
4. What happens when parallel agents complete - is there merge coordination?
5. Can hooks invoke agents? Can agents invoke skills?
6. What existing coordination primitives exist?

**IF YOU DON'T KNOW**: Say so explicitly and re-read the relevant docs.

---

## Phase 2: Analyze the Experiment

Review these experiment artifacts in `/home/devin/AIWorkflow/webgl-fluid-phase3-experiment/`:

### 2.1 Core Findings
- `COMPREHENSIVE_ANALYSIS.md` - All 5 phases synthesized
- `PHASE4_PARALLEL_N4_RESULTS.md` - Critical: θ=0.097, 127s, parallel topology
- `PHASE5_FINE_GRAINED_N8_RESULTS.md` - Over-coordination: θ=1.46, 813s

### 2.2 Key Metrics to Understand
- **θ (theta)** = Coordination Time / Work Time
  - Phase 1 (Solo): θ=0.0, 335s
  - Phase 2 (Linear N=3): θ=0.867, 351s
  - Phase 3 (Linear N=5): θ=~1.4, 656s
  - **Phase 4 (Parallel N=4): θ=0.097, 127s** ✅
  - Phase 5 (Fine-grained N=8): θ=1.46, 813s

### 2.3 Critical Questions About What Was Actually Done
1. How were "parallel agents" executed in Phase 4?
   - Were they actual parallel executions or simulated?
   - What was the coordination mechanism?
   - How were outputs merged?

2. What enabled the 90-93% θ reduction in Phase 4?
   - Was it parallelism itself?
   - Was it coordination pattern (fewer handoffs)?
   - Was it communication mechanism?

3. The user mentioned: "tmux panes and file or orchestrator message bus improve independence and unlock direct communication-based coordination"
   - What does this mean for the actual gaps?
   - What coordination mechanisms are needed vs execution mechanisms?

---

## Phase 3: Identify ACTUAL Gaps

Based on **verified Claude Code capabilities** and **actual experiment needs**, identify what's missing:

### 3.1 Categorize Gaps
**NOT GAPS** (already exists):
- List capabilities that Claude Code already has
- Don't design solutions for non-problems

**REAL GAPS** (missing capabilities):
- What coordination primitives don't exist?
- What metrics/observability is missing?
- What reusable patterns aren't packageable?
- What integration points are manual?

### 3.2 User's Hint About Real Needs
The user said: "tmux panes and file or orchestrator message bus improve independence and unlock direct communication-based coordination"

This suggests:
- Coordination mechanisms (message bus) may be the gap
- Independence/isolation between agents
- Direct communication protocols
- Not about enabling parallelism itself (already exists)

---

## Phase 4: Design Primitives

Design Claude Code components that:

### 4.1 Core Principles (User Requirements)
- **Modularity**: Each component is independent and reusable
- **Compositionality**: Components combine to build complex workflows
- **Flexibility**: Support linear, parallel, hybrid, custom topologies
- **Accuracy**: Based on empirical θ measurements from experiment
- **Power**: Enable the 90-93% θ reduction demonstrated in Phase 4

### 4.2 Component Categories to Consider

**Coordination Infrastructure:**
- Message bus for agent-to-agent communication
- Shared state management
- Event-driven coordination
- Handoff protocols

**Pattern Definitions:**
- Declarative topology specifications
- Reusable pattern templates (N=3 linear, N=4 parallel, etc.)
- Interface contracts between agents
- Merge strategies

**Observability & Metrics:**
- θ calculation and tracking
- Real-time coordination overhead monitoring
- Pattern performance benchmarking
- Alert systems

**Developer Experience:**
- Pattern selection helpers (orchestration-advisor)
- Visualization tools
- Testing frameworks
- Migration guides

### 4.3 Design Format

For each primitive, specify:

```markdown
## Primitive: [Name]

**Category**: [Coordination/Pattern/Metrics/DX]

**What**: [One sentence description]

**Why Needed**: [What specific gap this fills, with evidence from experiment]

**Existing Claude Code Foundation**: [What already exists that this builds on]

**Implementation**:
- Type: [Agent/Skill/Hook/Command/Tool/MCP/Other]
- Location: [.claude/X/ or ~/.claude/X/]
- Specification: [YAML/Markdown/JSON/Script]
- Example: [Concrete usage example]

**Integration Points**: [How this connects to other primitives or Claude Code]

**Validation**: [How this enables experiment's validated patterns]

**Priority**: [P0: Critical / P1: Important / P2: Nice-to-have]
- Rationale: [Why this priority, based on impact]
```

---

## Phase 5: Self-Validation

Before presenting your design, validate:

### 5.1 Accuracy Check
- [ ] Did I actually read Claude Code docs or guess?
- [ ] Can I quote specific doc sections for each capability claim?
- [ ] Did I identify what already exists vs what's missing?

### 5.2 Experiment Alignment
- [ ] Does this enable Phase 4's parallel N=4 pattern (θ=0.097)?
- [ ] Does this help avoid Phase 5's over-coordination (θ=1.46)?
- [ ] Can this implement linear N=3 pattern (θ=0.867)?

### 5.3 Design Principles
- [ ] Is each primitive modular (independent)?
- [ ] Do primitives compose (work together)?
- [ ] Is the system flexible (supports all topologies)?
- [ ] Are metrics accurate (based on experiment data)?
- [ ] Does this provide power (enable 90-93% θ reduction)?

### 5.4 Red Flags (Stop and Rethink)
- If you're designing "TaskParallel" but don't know if Task already does parallel execution
- If you're assuming capabilities without doc quotes
- If your design doesn't reference the user's hint about "message bus" and "direct communication"
- If you can't explain how each primitive enables the experiment's validated patterns

---

## Expected Output Structure

```markdown
# Claude Code Orchestration Infrastructure Design

## Executive Summary
[Key findings from Claude Code docs]
[What already exists vs what's missing]
[Design philosophy based on actual gaps]

## Part 1: Claude Code Capabilities Analysis
### 1.1 Agent Execution System
[What docs say about parallel execution - QUOTE DOCS]

### 1.2 Coordination Primitives
[What coordination mechanisms exist - QUOTE DOCS]

### 1.3 Identified Gaps
[What's actually missing, with evidence]

## Part 2: Experiment Insights
### 2.1 θ Scaling Patterns
[Summary of what experiment proved]

### 2.2 Critical Success Factors
[What enabled Phase 4's success]

### 2.3 Infrastructure Requirements
[What coordination infrastructure the experiment needed]

## Part 3: Primitive Designs
[10-20 modular primitives, each with full specification]

## Part 4: Integration Architecture
[How primitives work together]
[Example: End-to-end workflow using all primitives]

## Part 5: Implementation Roadmap
[Priority tiers with rationale]
[Dependencies between primitives]
[Validation criteria]
```

---

## Key Constraints

1. **NO GUESSING**: If you don't know a Claude Code capability, say so and research it
2. **QUOTE DOCS**: Reference specific doc sections for every capability claim
3. **USER HINT**: The message bus / direct communication hint is important - explore this
4. **EXPERIMENT DATA**: Every design decision must tie back to empirical θ measurements
5. **MODULARITY**: Each primitive must work independently and compose with others

---

**Start by saying**: "I will begin by thoroughly studying Claude Code documentation to understand actual capabilities before designing anything. Starting with sub-agents.md..."
