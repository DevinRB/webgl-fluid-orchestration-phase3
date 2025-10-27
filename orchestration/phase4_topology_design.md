# Phase 4: Parallel N=4 Topology Design

**Date**: 2025-10-27
**Pattern**: Parallel Dual-Track Orchestration
**Hypothesis**: Parallel execution reduces coordination overhead vs linear N=5

---

## Orchestration Topology

```
START
  ├─→ Track A (UI/Visual) ────┐
  │    ├─→ A1: UI Controls    │
  │    └─→ A2: Visual Render  │
  │                            │
  ├─→ Track B (Physics/Integ) │
  │    ├─→ B1: Physics/Shader │
  │    └─→ B2: Integration    │
  │                            ↓
  └─────────────────────→ MERGE → VALIDATE
```

**Key Characteristics**:
- **Two parallel tracks** working simultaneously
- **Minimal inter-track dependencies** (loose coupling)
- **Intra-track coordination** (tight coupling within track)
- **Single merge point** for integration

---

## Agent Roles and Responsibilities

### Track A: UI/Visual Layer

#### A1: UI Controls Specialist
**Focus**: dat.GUI integration for barrier controls
**Deliverables**:
- Barrier Mode toggle (checkbox)
- Clear Barriers button
- Barrier counter display
- GUI folder structure

**Dependencies**: None (can start immediately)
**Estimated Duration**: 60-80 seconds

#### A2: Visual Rendering Specialist
**Focus**: Canvas overlay for barrier visualization
**Deliverables**:
- Overlay canvas creation and positioning
- Circle rendering for barriers
- Coordinate conversion (screen space)
- Visual styling (colors, borders)

**Dependencies**: None (can start immediately)
**Estimated Duration**: 70-90 seconds

---

### Track B: Physics/Integration Layer

#### B1: Physics/Shader Specialist
**Focus**: GLSL shader modifications for barrier physics
**Deliverables**:
- Barrier uniform declarations (vec3 array, count)
- Advection shader modifications (velocity + dye)
- Distance checking logic
- Zero-velocity inside barriers

**Dependencies**: None (can start immediately)
**Estimated Duration**: 80-100 seconds

#### B2: Integration Specialist
**Focus**: Coordinate conversion, mouse handling, data flow
**Deliverables**:
- Barrier data structures (array, constants)
- Mouse click handler for placement
- Barrier limit enforcement (max 20)
- Uniform passing to shaders
- Clear barriers function
- TDZ prevention (early declarations)

**Dependencies**: None (can start immediately, interfaces are known)
**Estimated Duration**: 80-100 seconds

---

## Coordination Strategy

### Parallel Execution Phase
- **All 4 agents start simultaneously** with shared context:
  - Feature requirements (same as Phase 3)
  - Code structure (script.js baseline)
  - Interface contracts (uniform names, data structures)

### Merge Phase
- **No dedicated merge agent** (I will integrate)
- Combine outputs from all 4 agents
- Resolve any conflicts (expected: minimal)
- Ensure consistent naming conventions
- Validate integration

### Validation Phase
- **Reuse existing Playwright test suite** from Phase 3
- Run 8 automated browser tests
- Verify all acceptance criteria

---

## Expected Outcomes

### Timing Prediction
- **Parallel work**: ~90-100 seconds (longest single agent)
- **Merge/integration**: ~30-40 seconds
- **Test execution**: ~30-40 seconds
- **Total duration**: ~150-180 seconds (2.5-3 minutes)

### θ (Coordination Overhead) Prediction
- **Work time**: ~300-370 seconds (sum of all 4 agents)
- **Coordination time**: ~30-40 seconds (merge only)
- **θ = 30-40 / 300-370 = 0.08-0.13**

**Expected result**: θ much lower than N=3 (0.867) due to parallel execution!

### Comparison to Previous Phases
| Phase | Pattern | N | Total Duration | θ |
|-------|---------|---|----------------|---|
| Phase 1 | Solo | 1 | 335s | 0.0 |
| Phase 2 | Linear | 3 | 351s | 0.867 |
| Phase 3 | Linear | 5 | 656s | ~1.4-1.5 (adjusted) |
| **Phase 4** | **Parallel** | **4** | **~150-180s** | **~0.08-0.13** |

---

## Risk Assessment

### Potential Issues
1. **Interface misalignment**: Agents might use different variable names
   - **Mitigation**: Provide clear interface contracts upfront

2. **Duplicate code**: Two agents might implement similar logic
   - **Mitigation**: Clear role boundaries, post-merge deduplication

3. **Integration conflicts**: Code from different agents might conflict
   - **Mitigation**: I'll handle merge carefully, test thoroughly

### Success Criteria
- ✅ Feature works identically to Phase 3 implementation
- ✅ All 8 browser tests pass (reusing Phase 3 test suite)
- ✅ Total duration < Phase 2 (351s)
- ✅ θ significantly lower than linear patterns

---

## Interface Contracts (Shared Context)

### Data Structures
```javascript
let barriers = [];  // Array of {x, y, radius}
let barrierMode = false;
const MAX_BARRIERS = 20;
const DEFAULT_BARRIER_RADIUS = 0.05;
let barrierCanvas = null;
let barrierCtx = null;
```

### Shader Uniforms
```glsl
uniform vec3 barriers[20];  // (x, y, radius)
uniform int barrierCount;
```

### GUI Integration
```javascript
let barrierFolder = gui.addFolder('Barriers');
// Toggle, button, counter go here
```

### Functions to Implement
- `initBarrierOverlay()` - A2 responsibility
- `drawBarriers()` - A2 responsibility
- `clearBarriers()` - A2 or B2 responsibility
- Shader modifications - B1 responsibility
- Mouse click handler - B2 responsibility

---

## Execution Plan

1. **Start Timer** (mark Phase 4 start)
2. **Launch 4 agents in parallel** with this design document
3. **Collect outputs** from all 4 agents
4. **Merge phase** (integrate all code)
5. **Validation** (run Playwright tests)
6. **Calculate metrics** (θ, duration, comparison)
7. **Document results** (Phase 4 results markdown)

---

**Status**: Design complete, ready for parallel execution
**Next**: Launch all 4 agents simultaneously
