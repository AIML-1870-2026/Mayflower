# Turing Pattern Explorer - Technical Specification

## Project Overview
An interactive web-based Turing pattern (reaction-diffusion) simulator that allows users to create, explore, and manipulate emergent patterns through GPU-accelerated computation. The system implements the Gray-Scott model with real-time parameter control, interactive painting tools, stamp modes, and animated color schemes.

## Core Features

### 1. Simulation Engine
- **Model**: Gray-Scott reaction-diffusion system
- **Implementation**: WebGL 1.0 (Safari-compatible)
- **Resolution**: 512×512 pixel simulation grid
- **Frame Rate**: 10 simulation steps per render frame
- **Texture Format**: RGBA32F (floating-point precision)
- **Architecture**: Ping-pong framebuffer technique for efficient GPU computation

### 2. Pattern Presets
Six curated parameter combinations that generate distinct pattern types:
- **Spots**: feed=0.055, kill=0.062, diffA=1.0, diffB=0.5
- **Stripes**: feed=0.037, kill=0.060, diffA=1.0, diffB=0.5
- **Waves**: feed=0.022, kill=0.051, diffA=1.0, diffB=0.5
- **Coral**: feed=0.062, kill=0.061, diffA=1.0, diffB=0.5
- **Maze**: feed=0.029, kill=0.057, diffA=1.0, diffB=0.5
- **Fingerprint**: feed=0.055, kill=0.062, diffA=0.8, diffB=0.4

### 3. Interactive Brush System
**Modes**:
- Add mode: Paints chemical B onto the simulation
- Remove mode: Erases chemical B from the simulation

**Controls**:
- Adjustable brush size: 10-100 pixels
- Real-time painting with mouse drag
- Automatic Y-axis coordinate correction for WebGL compatibility

**Stamp Tools**:
- **Star**: 5-pointed star using geometric spike algorithm
- **Heart**: Geometric composition (two circles + tapered triangle)
- **Smiley**: Thin outline circle with two eyes and smile arc
- **Brush**: Standard circular brush (default)

Stamp characteristics:
- Click-to-place (no drag)
- Size: 3× brush size parameter
- Sampling density: 2-pixel grid
- Dot size: 8 pixels per sample point

### 4. Color Schemes
Seven color schemes with 4-point gradient mapping:
- **Ocean**: Deep blue → ocean blue → sky blue → bright cyan
- **Fire**: Black → red → orange → yellow
- **Purple**: Dark purple → violet → bright purple → lavender
- **Mint**: Black → dark green → mint → pale mint
- **Sunset**: Dark blue → brown → orange → yellow
- **Grayscale**: Black → dark gray → light gray → white
- **Rainbow**: Animated cycling through color spectrum (Red → Yellow → Green → Blue → repeating)

### 5. Parameter Controls
Real-time adjustable parameters with live UI feedback:
- **Feed Rate (f)**: 0.01 - 0.1 (controls pattern growth)
- **Kill Rate (k)**: 0.045 - 0.07 (controls pattern decay)
- **Diffusion A**: 0.5 - 1.5 (chemical A spread rate)
- **Diffusion B**: 0.1 - 0.8 (chemical B spread rate)
- **Timestep**: 0.5 - 2.0 (simulation speed)
- **Brush Size**: 10 - 100 pixels

### 6. Randomization System
Generates new patterns with:
- Random preset selection from all 6 presets
- Parameter variation (±0.01 feed, ±0.005 kill)
- Random color scheme selection
- 150 random seed positions
- Seed sizes: 10-40 pixel radius
- Intensity variation: 0.6-1.0

## Technical Implementation

### Shader Programs

**Vertex Shader** (shared):
```glsl
attribute vec2 a_position;
varying vec2 v_texCoord;
void main() {
    v_texCoord = a_position * 0.5 + 0.5;
    gl_Position = vec4(a_position, 0.0, 1.0);
}
```

**Simulation Fragment Shader**:
- Implements Gray-Scott reaction-diffusion equations
- Laplacian operator with 9-point stencil
- Kernel weights: center=-1.0, cardinal=0.2, diagonal=0.05
- Equations:
  - `da = diffA * laplacian_a - a*b² + feed*(1-a)`
  - `db = diffB * laplacian_b + a*b² - (kill+feed)*b`

**Render Fragment Shader**:
- Maps chemical B concentration to color
- 3-stage gradient interpolation
- HSV-to-RGB conversion for rainbow animation
- Animation speed: 0.02 radians per frame

**Brush Fragment Shader**:
- Distance-based circular brush
- Smooth falloff: `(1 - dist/size) * strength`
- Clamped output: 0.0 - 1.0

### Browser Compatibility

**Safari-Specific Optimizations**:
- WebGL 1.0 (not 2.0) for maximum compatibility
- OES_texture_float extension required
- GLSL version 100 (not 300 es)
- texture2D() instead of texture()
- gl_FragColor instead of out vec4
- varying instead of in/out keywords
- attribute instead of in for vertex shader
- preserveDrawingBuffer: true
- Y-axis coordinate flip: `512 - y`

### Performance Characteristics
- GPU-accelerated computation (all operations on GPU)
- No CPU pixel readback during painting
- Efficient ping-pong buffer swapping
- 60 FPS target frame rate
- Minimal JavaScript computation

## User Interface

### Layout
- Left panel: 512×512 canvas with simulation
- Right panel: 320px control sidebar with sections:
  - Presets (2×3 grid)
  - Simulation parameters (5 sliders)
  - Color schemes (4×2 grid with rainbow)
  - Brush tools (2 modes)
  - Stamp shapes (4 options)
  - Actions (randomize, clear, pause toggle)

### Visual Design
- Gradient background: Purple to blue
- White control panel with rounded corners
- Slider color: Purple (#667eea)
- Active button highlight: Purple background
- Status messages: Bottom-center toast notifications
- Cursor: Crosshair for brush, pointer for stamps

## User Interactions

### Mouse Controls
- **Mouse down**: Start painting/place stamp
- **Mouse move**: Continue brush stroke (stamps don't drag)
- **Mouse up**: End painting
- **Mouse leave**: Cancel painting

### Stamp Behavior
- Cursor changes to pointer in stamp mode
- Single click places complete shape
- Shape geometry calculated via helper functions:
  - `isInsideStar()`: Geometric spike algorithm
  - `isInsideHeart()`: Two circles + tapered triangle
  - `isInsideSmiley()`: Circle outline + eyes + smile arc

### Status Messages
- Preset loading: "✓ [preset name]"
- Randomization: "🎲 [PRESET] + [COLOR]"
- Clear canvas: "🗑️ Cleared"
- Pause toggle: "⏸️ Paused" / "▶️ Running"
- Stamp selection: "Stamp: [shape]"
- Color change: "🎨 [color]"

## Mathematical Model

### Gray-Scott Equations
```
∂A/∂t = Da∇²A - AB² + f(1-A)
∂B/∂t = Db∇²B + AB² - (k+f)B
```

Where:
- A, B: Chemical concentrations (0.0 - 1.0)
- Da, Db: Diffusion rates
- f: Feed rate
- k: Kill rate
- ∇²: Laplacian operator

### Laplacian Computation
9-point discrete Laplacian stencil:
```
[ 0.05  0.20  0.05 ]
[ 0.20 -1.00  0.20 ]
[ 0.05  0.20  0.05 ]
```

### Stamp Shape Algorithms

**Star (5-pointed)**:
```javascript
section = floor(angle / (2π/5)) % 5
t = (angle % (2π/5)) / (2π/5)
maxRadius = 1.0 - |t - 0.5| * 1.2
inside = dist < maxRadius
```

**Heart**:
```javascript
leftCircle = (x+0.35)² + (y-0.35)² < 0.25
rightCircle = (x-0.35)² + (y-0.35)² < 0.25
triangleWidth = 0.7 * (y+1.0) / 1.1
inside = leftCircle OR rightCircle OR (y<0.1 AND |x|<triangleWidth)
```

**Smiley**:
```javascript
outline = 0.85 < dist < 0.92
leftEye = distance(x+0.3, y-0.3) < 0.1
rightEye = distance(x-0.3, y-0.3) < 0.1
smile = |y - smileArc(x)| < 0.06 AND y < 0
```

## File Structure
```
turing-patterns.html (726 lines)
├── HTML structure (200 lines)
│   ├── Canvas container
│   └── Control panel with sections
├── CSS styling (250 lines)
│   ├── Layout and positioning
│   ├── Button and slider styles
│   └── Responsive design
└── JavaScript (276 lines)
    ├── WebGL initialization
    ├── Shader compilation
    ├── Simulation loop
    ├── User interaction handlers
    ├── Stamp geometry functions
    └── Utility functions
```

## Stretch Goals Achieved
✅ **Interactive brush tools**: Add/Remove modes with adjustable brush sizes
✅ **Pattern presets with descriptions**: 6 curated parameter combinations
🟡 **Multiple chemical systems**: Gray-Scott implemented (partial - 1 of 3 models)

## Future Enhancement Opportunities
- Parameter space animation (smooth transitions through f/k space)
- High-resolution export (PNG download at 2048×2048)
- Side-by-side comparison mode (dual simulation view)
- Additional models: Brusselator, Schnakenberg
- Touch device optimization
- Keyboard shortcuts
- Undo/redo system
- Pattern library saving

## System Requirements
- Modern web browser with WebGL 1.0 support
- OES_texture_float extension
- Minimum 512MB GPU memory
- macOS Safari 14+, Chrome 90+, Firefox 88+, Edge 90+

## Known Limitations
- Rainbow color scheme is animated globally (not per-brush-stroke)
- Stamps take ~100-500ms to render (many GPU draw calls)
- No persistent storage (patterns reset on page reload)
- Fixed 512×512 resolution (no zoom or pan)
- Single simulation instance (no side-by-side comparison)
