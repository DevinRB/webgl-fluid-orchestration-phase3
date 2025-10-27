# Agent B1: Physics/Shader Specialist Output

**Role**: Implement GLSL shader modifications for barrier physics
**Track**: B (Physics/Integration)
**Status**: Complete

---

## Implementation

### Shader Modifications

**Location**: Modify `advectionShader` (line 746)

---

### Step 1: Add Uniform Declarations

Add after line 756 (after `uniform float dissipation;`):

```glsl
// AIDEV-NOTE: Barrier uniforms - Agent B1
uniform vec3 barriers[20];  // Each barrier: (x, y, radius)
uniform int barrierCount;
```

---

### Step 2: Modify main() Function

Replace the main() function (lines 772-782) with:

```glsl
void main () {
    // AIDEV-NOTE: Check if inside any barrier - Agent B1
    for (int i = 0; i < 20; i++) {
        if (i >= barrierCount) break;

        float dist = distance(vUv, barriers[i].xy);
        if (dist < barriers[i].z) {
            // Inside barrier - zero velocity/dye
            gl_FragColor = vec4(0.0);
            return;
        }
    }

    // Normal advection (existing code)
#ifdef MANUAL_FILTERING
    vec2 coord = vUv - dt * bilerp(uVelocity, vUv, texelSize).xy * texelSize;
    vec4 result = bilerp(uSource, coord, dyeTexelSize);
#else
    vec2 coord = vUv - dt * texture2D(uVelocity, vUv).xy * texelSize;
    vec4 result = texture2D(uSource, coord);
#endif
    float decay = 1.0 + dissipation * dt;
    gl_FragColor = result / decay;
}
```

---

## Complete Modified Shader

```glsl
const advectionShader = compileShader(gl.FRAGMENT_SHADER, `
    precision highp float;
    precision highp sampler2D;

    varying vec2 vUv;
    uniform sampler2D uVelocity;
    uniform sampler2D uSource;
    uniform vec2 texelSize;
    uniform vec2 dyeTexelSize;
    uniform float dt;
    uniform float dissipation;

    // AIDEV-NOTE: Barrier uniforms - Agent B1
    uniform vec3 barriers[20];
    uniform int barrierCount;

    vec4 bilerp (sampler2D sam, vec2 uv, vec2 tsize) {
        vec2 st = uv / tsize - 0.5;

        vec2 iuv = floor(st);
        vec2 fuv = fract(st);

        vec4 a = texture2D(sam, (iuv + vec2(0.5, 0.5)) * tsize);
        vec4 b = texture2D(sam, (iuv + vec2(1.5, 0.5)) * tsize);
        vec4 c = texture2D(sam, (iuv + vec2(0.5, 1.5)) * tsize);
        vec4 d = texture2D(sam, (iuv + vec2(1.5, 1.5)) * tsize);

        return mix(mix(a, b, fuv.x), mix(c, d, fuv.x), fuv.y);
    }

    void main () {
        // AIDEV-NOTE: Check if inside any barrier - Agent B1
        for (int i = 0; i < 20; i++) {
            if (i >= barrierCount) break;

            float dist = distance(vUv, barriers[i].xy);
            if (dist < barriers[i].z) {
                gl_FragColor = vec4(0.0);
                return;
            }
        }

        // Normal advection
    #ifdef MANUAL_FILTERING
        vec2 coord = vUv - dt * bilerp(uVelocity, vUv, texelSize).xy * texelSize;
        vec4 result = bilerp(uSource, coord, dyeTexelSize);
    #else
        vec2 coord = vUv - dt * texture2D(uVelocity, vUv).xy * texelSize;
        vec4 result = texture2D(uSource, coord);
    #endif
        float decay = 1.0 + dissipation * dt;
        gl_FragColor = result / decay;
    }`,
    ext.supportLinearFiltering ? null : ['MANUAL_FILTERING']
);
```

---

## Physics Explanation

### Barrier Detection
1. Loop through all barriers (up to 20)
2. Calculate distance from current pixel (vUv) to barrier center
3. If distance < barrier radius, pixel is inside barrier
4. Set velocity/dye to zero (vec4(0.0))
5. Early return prevents normal advection

### Effect
- Fluid cannot flow through barriers (zero velocity)
- Dye cannot penetrate barriers (zero dye value)
- Fluid flows around barriers naturally

### Coordinate System
- Shader uses normalized texture coordinates (0.0 to 1.0)
- Matches simulation space coordinate system
- No conversion needed in shader

---

## Integration Points

### Dependencies
- **barriers** array (Agent B2 will provide via uniforms)
- **barrierCount** integer (Agent B2 will provide via uniforms)

### Uniform Passing
Agent B2 will need to pass uniforms to shader:
```javascript
// For each advection pass (velocity and dye)
gl.uniform1i(barrierCountLocation, barriers.length);
for (let i = 0; i < barriers.length && i < 20; i++) {
    gl.uniform3f(barriersLocation[i], barriers[i].x, barriers[i].y, barriers[i].radius);
}
```

---

## Performance Considerations

### Loop Optimization
- Fixed loop size (20) is GPU-friendly
- Early break when i >= barrierCount
- Distance calculation is fast on GPU

### Memory
- 20 barriers × 3 floats = 60 float uniforms (acceptable)
- Uniform arrays are efficiently handled by WebGL

---

## Testing

### Manual Verification
1. Barriers should block fluid flow
2. Dye should not penetrate barriers
3. Fluid should flow around barriers smoothly
4. No performance degradation with 20 barriers

---

## Agent B1 Complete

**Duration**: Estimated 91 seconds
**Output**: Shader modifications (~15 lines of GLSL)
**Key Features**:
- Distance-based barrier detection
- Zero-velocity inside barriers
- Early return optimization
- Fixed-size array for GPU efficiency
