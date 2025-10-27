# Barrier Feature Documentation

## Overview

This WebGL Fluid Simulation now includes interactive barrier placement that allows users to create obstacles that block fluid flow.

## Usage

### Desktop
- **Place Barrier**: Hold `Shift` key and click anywhere on the canvas (cursor shows crosshair)
- **Preview Placement**: Hold `Shift` key and move mouse to see preview circle before clicking
- **Remove Barrier**: Right-click on an existing barrier (hover highlights it in red, cursor shows pointer)
- **Clear All Barriers**: Press `C` key or click "Clear barriers (C)" button in GUI
- **Hide/Show Instructions**: Press `H` key to toggle instruction overlay
- **Adjust Barrier Size**: Use the "barrier radius" slider in the GUI (right side)
- **Save Layout**: Click "Save barriers" button in GUI (persists to localStorage)
- **Load Layout**: Click "Load barriers" button in GUI (auto-loads on page load)

### Mobile/Touch
- **Place Barrier**: Use two-finger touch simultaneously
- **Clear All Barriers**: Use the "Clear barriers (C)" button in GUI

## How It Works

### Data Structure
Barriers are stored in a simple array:
```javascript
let barriers = [];  // Array of {x, y, radius} objects
```

### Visual Rendering
- Barriers appear as bright white circles
- Rendered at 10x intensity for high visibility
- Preview circles shown at 5x intensity when Shift is held (before placement)
- Persist across frames (redrawn each frame)
- Barrier count display changes color based on performance:
  - Blue (normal): < 30 barriers
  - Orange (caution): 30-50 barriers
  - Red (warning): > 50 barriers

### Physics
- Velocity is zeroed out at barrier positions
- Applied 3 times per frame with 1.2x radius for stronger blocking
- Fluid cannot penetrate barriers
- Fluid flows around barriers

## Implementation Details

### Files Modified
- `script.js`: Core barrier logic, UI handling, physics integration
- `index.html`: User instructions overlay and barrier count display

### Key Functions
- `addBarrier(x, y, radius)`: Add a new barrier at texture coordinates
- `clearBarriers()`: Remove all barriers
- `removeBarrierAt(x, y)`: Remove a single barrier at specified location
- `saveBarriers()`: Save current barrier layout to localStorage
- `loadBarriers()`: Load barrier layout from localStorage
- `applyBarriersToVelocity()`: Zero velocity at barrier positions
- `applyBarriersToDye()`: Clear dye at barrier positions
- `drawBarriers(target)`: Render barriers visually (with hover highlighting)
- `updateBarrierCount()`: Update UI counter with color-coded performance warnings

### Configuration
- `config.BARRIER_RADIUS`: Default barrier radius (0.005 - 0.1)
- Configurable via GUI slider

## Performance Notes

- Barrier count is displayed in real-time
- Each barrier applies velocity zeroing 3 times per frame
- Recommended: Keep barrier count < 50 for optimal performance
- No significant FPS impact with reasonable barrier counts

## Future Enhancements

Potential improvements:
- Barrier deletion (click to remove individual barriers)
- Different barrier shapes (rectangle, polygon)
- Barrier persistence across window resize
- Save/load barrier configurations
- Barrier strength/permeability settings
- Visual preview before placement

## Technical Notes

### Coordinate System
- Barriers use texture coordinates (0-1 range)
- X: 0 (left) to 1 (right)
- Y: 0 (bottom) to 1 (top)

### Shader Integration
- Uses existing `splatProgram` shader for both rendering and physics
- Barriers are "splatted" with zero velocity each frame
- Visual rendering uses high-intensity white color
- Physics uses zero velocity vector

### Browser Compatibility
- Works on all browsers supporting WebGL
- Touch support for mobile devices
- Keyboard support for desktop

## Acceptance Criteria Met

✅ User can place barriers with Shift+Click
✅ Barriers appear visually distinct (bright white)
✅ Fluid flow stops at barriers
✅ Multiple barriers supported
✅ Clear function implemented (C key)
✅ Touch support for mobile
✅ GUI controls for configuration
✅ Responsive design

## License

Same as parent project (MIT License - Pavel Dobryakov)
