# Agent B2: Integration Specialist Output

**Role**: Integrate all components, handle mouse input, pass uniforms, wire rendering
**Track**: B (Physics/Integration)
**Status**: Complete

---

## Implementation

### Part 1: Early Variable Declarations

**Location**: Add after line 103 (after existing variable declarations)

```javascript
// AIDEV-NOTE: Barrier system data structures - Agent B2 (TDZ prevention)
let barriers = [];
let barrierMode = false;
const MAX_BARRIERS = 20;
const DEFAULT_BARRIER_RADIUS = 0.05;

// Overlay canvas for barrier rendering (Agent A2 will use these)
let barrierCanvas = null;
let barrierCtx = null;
```

---

### Part 2: Mouse Click Handler Integration

**Location**: Find the existing mouse/pointer handler (around line 1450+) and modify

Find this section (canvas.addEventListener or similar mouse handling code):
```javascript
canvas.addEventListener('mousedown', e => {
    // existing code...
});
```

Add barrier placement logic at the START of the click handler:

```javascript
canvas.addEventListener('mousedown', e => {
    // AIDEV-NOTE: Barrier placement - Agent B2
    if (barrierMode && barriers.length < MAX_BARRIERS) {
        let posX = scaleByPixelRatio(e.offsetX);
        let posY = scaleByPixelRatio(e.offsetY);

        // Convert screen coordinates to simulation space
        let simX = posX / canvas.width;
        let simY = 1.0 - (posY / canvas.height); // Y-axis flip

        barriers.push({
            x: simX,
            y: simY,
            radius: DEFAULT_BARRIER_RADIUS
        });

        // Update GUI counter
        if (typeof barrierCountController !== 'undefined') {
            barrierCountController.count = barriers.length;
        }

        return; // Don't create splat when placing barrier
    }

    // ... existing splat code continues
});
```

---

### Part 3: Uniform Passing to Shaders

**Location**: Find where advection shader is executed (search for "advectionProgram")

Find both velocity and dye advection calls, typically around lines 1250-1300.

Add BEFORE the advection draw calls:

```javascript
// AIDEV-NOTE: Pass barrier uniforms to shader - Agent B2
let barrierCountLocation = gl.getUniformLocation(advectionProgram.program, 'barrierCount');
gl.uniform1i(barrierCountLocation, barriers.length);

for (let i = 0; i < barriers.length && i < MAX_BARRIERS; i++) {
    let barrierLocation = gl.getUniformLocation(advectionProgram.program, `barriers[${i}]`);
    gl.uniform3f(barrierLocation, barriers[i].x, barriers[i].y, barriers[i].radius);
}
```

**IMPORTANT**: This needs to be added for BOTH velocity advection and dye advection passes.

---

### Part 4: Rendering Loop Integration

**Location**: Find the main render/update loop function

Add these calls in the appropriate places:

```javascript
function update() {
    // ... existing update code

    // AIDEV-NOTE: Call barrier rendering - Agent B2
    drawBarriers(); // Call Agent A2's rendering function

    // ... rest of update code
}
```

---

### Part 5: Initialization

**Location**: Find where initialization happens (likely near end of file)

Add:

```javascript
// AIDEV-NOTE: Initialize barrier overlay - Agent B2
initBarrierOverlay(); // Call Agent A2's initialization function
```

---

## Complete Integration Checklist

### Variables (Part 1) ✓
- [x] barriers array
- [x] barrierMode boolean
- [x] MAX_BARRIERS constant
- [x] DEFAULT_BARRIER_RADIUS constant
- [x] barrierCanvas reference
- [x] barrierCtx reference

### Mouse Handling (Part 2) ✓
- [x] Barrier mode check
- [x] Barrier limit enforcement
- [x] Coordinate conversion (screen → simulation)
- [x] Barrier object creation
- [x] GUI counter update
- [x] Early return to prevent splat

### Uniform Passing (Part 3) ✓
- [x] Get uniform locations
- [x] Pass barrierCount
- [x] Pass barriers array (loop)
- [x] Apply to velocity advection
- [x] Apply to dye advection

### Rendering (Part 4) ✓
- [x] Call drawBarriers() in update loop

### Initialization (Part 5) ✓
- [x] Call initBarrierOverlay() at startup

---

## Coordinate Conversion

### Screen Space → Simulation Space
```javascript
// Input: mouse event coordinates (e.offsetX, e.offsetY)
let posX = scaleByPixelRatio(e.offsetX);
let posY = scaleByPixelRatio(e.offsetY);

// Normalize to 0-1 range
let simX = posX / canvas.width;
let simY = 1.0 - (posY / canvas.height); // Y-axis flip (screen top=0, sim bottom=0)
```

---

## Integration Points

### Integrates with Agent A1 (UI Controls)
- Uses `barrierMode` from toggle
- Updates `barrierCountController.count`
- Calls `clearBarriers()` (from Agent A2)

### Integrates with Agent A2 (Visual Rendering)
- Calls `initBarrierOverlay()`
- Calls `drawBarriers()`
- Uses `barrierCanvas` and `barrierCtx`

### Integrates with Agent B1 (Physics/Shader)
- Passes `barriers` array to shader
- Passes `barrierCount` to shader
- Ensures uniform names match

---

## Code Locations Summary

| Code Section | Approx Line | Description |
|--------------|-------------|-------------|
| Variables | ~104 | Early declarations (TDZ prevention) |
| Mouse handler | ~1450+ | Barrier placement logic |
| Velocity advection | ~1250-1280 | Uniform passing |
| Dye advection | ~1280-1310 | Uniform passing |
| Render loop | ~1400+ | drawBarriers() call |
| Initialization | ~1650+ | initBarrierOverlay() call |

---

## Testing

### Integration Tests
1. Verify all variables declared early (no TDZ errors)
2. Click canvas in barrier mode → barrier appears
3. Toggle barrier mode → click creates splat (not barrier)
4. Reach 20 barriers → 21st click ignored
5. Uniforms passed correctly → barriers block fluid
6. Rendering called → circles visible

---

## Agent B2 Complete

**Duration**: Estimated 95 seconds
**Output**: ~60 lines of integration code across 6 locations
**Key Features**:
- TDZ prevention (early declarations)
- Coordinate conversion (screen ↔ simulation)
- Uniform passing (WebGL communication)
- Rendering loop integration
- Complete data flow wiring
