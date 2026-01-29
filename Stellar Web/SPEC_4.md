# Stellar Web Explorer - Project Specification

## Overview
Stellar Web Explorer is an interactive particle network visualization that creates dynamic "stellar webs" in a galaxy-themed environment. Users can explore proximity-based node connections, discover constellations, and encounter playful easter eggs while navigating through a beautiful purple-pink cosmic backdrop.

## Core Features

### 1. Particle System & Network Visualization
- **Nodes**: 150 animated particles (stars) moving through 3D space
- **Proximity-Based Connections**: Edges form between nodes within a configurable connectivity radius
- **Galaxy Color Scheme**: Connections use vibrant purple, pink, and magenta gradients
- **3D Depth Effects**: Node size and opacity vary based on z-position
- **Trail Effects**: Semi-transparent overlay creates smooth motion trails

### 2. Interactive Controls Panel
**Location**: Right side of screen, upper position
**Controls Include**:
- Node Count (20-300, default: 150)
- Connectivity Radius (50-500px, default: 300)
- Node Speed (0.1-2.0, default: 0.5)
- Node Size (2-10, default: 4)
- Edge Opacity (0.1-1.0, default: 0.6)
- Edge Thickness (0.5-4.0, default: 1.5)
- Constellation Mode (ON/OFF toggle)

**Features**:
- Collapsible with toggle button (◀/▶)
- Cyan border and accents
- Smooth slide animations
- Real-time updates on slider changes

### 3. Network Statistics Panel
**Location**: Right side of screen, bottom position
**Metrics Displayed**:
- Active Nodes
- Total Edges
- Average Connections per Node
- Network Density (percentage)
- Max Connections
- Min Connections
- Isolated Nodes (nodes with zero connections)

**Features**:
- Independent collapse/expand from controls
- Pink/magenta border and accents
- Real-time statistical updates
- Auto-calculates network metrics

### 4. Galaxy Background
**Base Colors**: Deep purple gradient (#2d0a45 → #1a0530 → #0d0220)

**Nebula Layers** (multiple radial gradients):
- Magenta glow (bottom-left): rgba(180, 0, 150, 0.7)
- Deep purple nebula fill: rgba(80, 20, 120, 0.6)
- Darker pink regions (right side): rgba(160, 30, 140, 0.65)
- Deep violet clouds (top): rgba(100, 30, 140, 0.6)
- Subtle magenta accents: rgba(140, 10, 120, 0.55)
- Dark purple fill layer: rgba(90, 25, 100, 0.5)

**Star Field**: Multi-layered repeating background with:
- White stars (various sizes)
- Pink-tinted stars
- Purple/violet stars
- Lavender stars
- Twinkling animation (4s cycle)
- Drifting animation (120s cycle)

**Animation**: Slow rotation (120s) with scale variation

### 5. Node Rendering

#### Regular Nodes (Star Mode)
- **5 Color Variations**: Purple, Pink, Blue, White, Medium Purple
- **Twinkling Effect**: Sine wave animation for realistic star shimmer
- **Colorful Halos**: Radial gradients around each star
- **White Core**: Bright center point
- **Sparkle Effect**: Cross pattern when brightness exceeds threshold
- **Size Variation**: Based on 3D depth (z-position)

#### Constellation Mode Nodes
- **Golden Color**: Bright yellow/gold tint
- **Enhanced Glow**: Larger, more prominent halos
- **Twinkling Animation**: Independent timing per star
- **Fixed Positions**: Stationary constellation patterns

### 6. Constellation System

**Total Constellations**: 18 major constellations spread across 3x canvas area

**Constellations Include**:
- Big Dipper (top-left quadrant)
- Orion (far right)
- Cassiopeia (top center)
- Leo (bottom left far)
- Cygnus (center right)
- Ursa Minor (top right far)
- Scorpius (bottom center)
- Gemini (left center)
- Aquila (dead center)
- Lyra (upper center)
- Draco (far top right)
- Andromeda (upper left)
- Perseus (middle left)
- Pegasus (right side)
- Taurus (lower left)
- Sagittarius (bottom right)
- Virgo (center bottom)
- Aries (middle top)

**Features**:
- Golden connecting lines between constellation stars
- Hover detection shows constellation name
- Name displayed in golden glowing box at top center
- Stars remain in fixed positions
- Patterns recognizable and spread for exploration

### 7. Camera & Navigation

**Roaming System**:
- Click and drag to pan/explore the galaxy
- Smooth camera interpolation (10% smoothing factor)
- Universe size: 3x viewport dimensions
- Infinite wrapping for seamless exploration
- Camera offset applied to all rendering

**Mouse Interactions**:
- Drag detection with start/end coordinates
- Hover detection for constellation identification
- Node interaction (attraction/repulsion within radius)
- Cursor changes to crosshair

### 8. Easter Eggs

#### Nyan Cat Node
**Timing**:
- Appears every 8 seconds
- Stays visible for 5 seconds
- Random node selection

**Visual**:
- Pixel art Nyan Cat with pink Pop-Tart body
- Colorful sprinkles on body
- Grey cat with black eyes and pink mouth
- 7-color rainbow trail (red, orange, yellow, green, blue, indigo, violet)
- 8x larger than normal nodes

**Behavior**:
- Maintains normal node movement
- Keeps all connections
- Seamless transformation and reversion
- Console logging for debugging

#### Alien UFO
**Timing**:
- Spawns every 12 seconds
- Multiple UFOs can exist simultaneously
- Despawns when off-screen

**Visual Design**:
- Classic flying saucer shape
- Silver metallic body (grey ellipse)
- Green transparent dome on top
- Cute green alien pilot inside with:
  - Big black eyes with white shine
  - Red antenna on head
- 6 colored lights around saucer edge (rotating)
- Cyan tractor beam underneath

**Animation**:
- Flies across from left or top edge
- Speed: 2-4 pixels per frame
- Bobbing motion (sine wave, 3px amplitude)
- Wobbling effect on lights
- Random trajectory variations

**Effects**:
- Colorful lights pulse and glow
- Tractor beam with gradient transparency
- Independent movement from nodes
- Drawn before nodes (background layer)

## Technical Architecture

### Canvas Setup
- Full viewport dimensions
- 2D rendering context
- Transparent background
- Z-index: 2 (above galaxy background)
- Dark overlay: rgba(0, 0, 0, 0.12)

### Performance Optimizations
- Trail effect reduces full redraws
- Efficient distance calculations
- Node connection culling
- Constellation nodes cached separately
- UFO cleanup when off-screen

### Responsive Design
- Mobile support with adjusted panel positions
- Touch-friendly controls
- Flexible canvas sizing
- Window resize handlers

## Code Structure

### Main Components
1. **Canvas & Rendering Context**
2. **Camera System** (position, smoothing, targets)
3. **Configuration Object** (all adjustable parameters)
4. **Mouse Tracking** (position, drag state, hover detection)
5. **Node Class** (position, velocity, depth, rendering)
6. **Constellation System** (patterns, names, generation)
7. **AlienUFO Class** (position, velocity, animation state)
8. **Animation Loop** (update, draw, statistics)
9. **Control Panel Handlers**
10. **Statistics Calculator**

### Key Functions
- `initNodes()` - Initialize node array
- `generateConstellationNodes()` - Create constellation patterns
- `drawConnections()` - Render edges between nodes
- `updateStats()` - Calculate network metrics
- `animate()` - Main animation loop
- `resizeCanvas()` - Handle viewport changes

### Data Structures
- `nodes[]` - Array of Node instances
- `constellationNodes[]` - Array of constellation Node instances
- `ufos[]` - Array of active AlienUFO instances
- `config{}` - Configuration parameters object
- `stats{}` - Real-time statistics object
- `camera{}` - Camera position and interpolation
- `mouse{}` - Mouse state and interactions

## Design Specifications

### Typography
- **Headers**: Orbitron (weights: 400, 700, 900)
- **Body/UI**: Space Mono (weights: 400, 700)
- **Panel Headers**: 0.9rem Orbitron
- **Control Labels**: 0.7rem Space Mono, uppercase
- **Statistics**: 0.65-0.75rem
- **Constellation Names**: 24px Orbitron

### Color Palette
- **Primary Purple**: #2d0a45, #1a0530, #0d0220
- **Accent Cyan**: #00d4ff, #00ffcc
- **Accent Magenta**: #b388ff, #ff69b4
- **Panel Background**: rgba(10, 14, 39, 0.92)
- **Panel Borders**: rgba(0, 212, 255, 0.3) / rgba(255, 105, 180, 0.4)
- **Star Colors**: Various purples, pinks, whites

### Spacing & Layout
- Panel Width: 240px
- Panel Padding: 1rem
- Control Spacing: 1rem between groups
- Panel Position: Right side, 10rem from top (controls), 2rem from bottom (stats)
- Border Radius: 8px (panels), 4px (buttons)

## Browser Compatibility
- Modern browsers with Canvas API support
- ES6+ JavaScript features
- CSS Grid and Flexbox
- Backdrop filter support recommended

## Performance Targets
- 60 FPS animation
- Smooth camera interpolation
- Responsive controls (<16ms input lag)
- Efficient collision detection for connections

## Future Enhancement Ideas
- Sound reactivity to music/microphone
- Gravity wells with orbital mechanics
- Color theme presets
- Export/screenshot functionality
- VR mode support
- Node types (attractors, repellers, anchors)
- Social network data visualization
- More constellation patterns
- Additional easter eggs

## Known Limitations
- Network calculations scale O(n²) with node count
- Large connectivity radius can impact performance
- Mobile performance may vary with high node counts
- Browser storage APIs not available

## Credits & Attribution
- Font: Orbitron & Space Mono (Google Fonts)
- Design: Space/galaxy theme with cosmic aesthetics
- Easter Eggs: Nyan Cat homage, classic alien UFO design

## Version
1.0.0 - Initial Release

## License
Created as an educational/demo project

---

**Last Updated**: January 2026
