# African Savanna Boids Ecosystem - Project Specification

## Project Overview
An interactive web-based simulation demonstrating flocking behavior (boids algorithm) themed as African savanna wildlife. The simulation features realistic animal representations, environmental interactions, and murmuration-like synchronized movement patterns.

---

## Core Requirements

### 1. Boids Algorithm Implementation
**Required Parameters (User-Adjustable):**
- Separation Weight (0-3)
- Alignment Weight (0-5)
- Cohesion Weight (0-5)
- Neighbor Radius (20-200 pixels)
- Max Speed (1-5)

**Flocking Behavior:**
- Animals must demonstrate classic boids behavior
- Movement should resemble starling murmurations (high synchronization)
- Smooth, flowing, liquid-like group dynamics
- Consistent speed maintenance (85% of max speed)

### 2. Species Implementation

**Gazelles:**
- Color: Tan (#d4a574)
- Count: 15 animals
- Speed: Fast (2.5 max speed multiplier)
- Behavior: Moderate alignment (2.2x), moderate cohesion (1.8x)
- Visual: Detailed gazelle with body, head, horns, legs, white belly, eyes

**Zebras:**
- Color: Black and white striped
- Count: 10 animals
- Speed: Slower (1.8 max speed multiplier)
- Behavior: Very high alignment (2.5x), strong cohesion (2.0x)
- Visual: Detailed zebra with stripes, mane, tail, legs, body

### 3. Interactive Features

**Mouse Following:**
- Toggle button: "🖱️ Follow Mouse" / "🖱️ Stop Following"
- When active, animals gently follow cursor
- Golden crosshair indicator shown at cursor
- Animals maintain flocking behavior while following

**Predator System:**
- Add/Remove lion predators via buttons
- Click to place predators on canvas
- Lions hunt nearest prey within 200px radius
- Prey flee from predators (150px flee radius, 5x force)
- Visual: Detailed lion with mane, body, legs, paws, tail, golden eyes

**Leader System:**
- "⭐ Tag Leader" button
- Click a gazelle to make it a leader
- Leader displays golden crown with star emoji
- Other gazelles follow their leader

**Obstacle System:**
- "🌳 Add Tree" and "🗑️ Remove Tree" buttons
- Trees as circular obstacles (25px radius)
- Animals strongly avoid obstacles (40px avoidance radius, 6x force)
- Visual: Green foliage with brown trunk

**Watering Hole:**
- Central blue water feature (60px radius)
- Animals attracted from 250px distance
- Strong avoidance of entering water (25px buffer, 6x force)
- Toggle on/off in Display Options
- Visual: Blue water with shimmer, brown border

### 4. Preset Behaviors

**☀️ Peaceful Grazing:**
- Separation: 1.8
- Alignment: 2.5
- Cohesion: 2.5
- Max Speed: 1.8

**🏃 Migration Mode:**
- Separation: 2.0
- Alignment: 4.0 (extreme synchronization)
- Cohesion: 3.0
- Max Speed: 3.5

**⚠️ High Alert:**
- Separation: 2.5
- Alignment: 3.5
- Cohesion: 2.0
- Max Speed: 4.5

### 5. Instrumentation (Live Statistics)

**Required Readouts:**
- FPS counter
- Gazelle count
- Zebra count
- Predator count
- Average neighbor count (per animal)

### 6. Simulation Controls

**Buttons:**
- ⏸️ Pause / ▶️ Resume
- 🔄 Reset (reinitializes simulation)
- 👣 Toggle Trails (show/hide with text change)

**Boundary Options:**
- Toggle: Wrap boundaries vs Solid boundaries
- Wrap: Animals teleport to opposite edge
- Solid: Turning force at margins

### 7. Visual Features

**Motion Trails (Footprints):**
- Prey: Brown circular footprints (2.5px)
- Frequency: Every 2 frames
- Opacity: 0-60% fade over 30 frames
- Predator: Brown continuous trails (4px width)
- Opacity: 0-70% fade over 40 frames

**Display Options:**
- Show/Hide footprint trails (checkbox + button)
- Show/Hide perception radius circles
- Show/Hide watering hole
- Toggle boundary mode

### 8. Theme & Aesthetics

**Environment:**
- Background: Vertical gradient (sky blue to sandy brown to darker earth tones)
- Grass texture overlay (subtle green dots)
- Savanna color palette throughout UI

**Canvas:**
- Responsive sizing (max 1000x700, fits container)
- Golden/brown border
- Crosshair cursor when in placement modes

**Control Panel:**
- Width: 350px, scrollable
- Dark brown semi-transparent background
- Golden/tan text colors
- Grouped sections with headers
- Color-coded buttons (green for controls, orange for actions, red for danger)

### 9. Technical Implementation

**File Structure:**
- Single HTML file with embedded CSS and JavaScript
- No external dependencies except browser APIs

**Classes:**
- `Boid` - Base animal class with species-specific properties
- `Predator` - Lion class with hunting behavior
- `Obstacle` - Tree class with collision detection
- `WateringHole` - Object with attraction/avoidance zones

**Force Calculations:**
- Separation: Push away from neighbors (1x base)
- Alignment: Match velocity of neighbors (0.15x multiplier for murmuration effect)
- Cohesion: Pull toward center of neighbors (0.025x multiplier)
- Species multipliers applied to each force type

**Movement:**
- Velocity-based physics
- Speed maintenance system (target 85% max)
- Smooth acceleration/deceleration
- Boundary handling (wrap or bounce)

### 10. Legend

**Species Indicators:**
- 🦌 Gazelle (Tan circle)
- 🦓 Zebra (Black circle)
- 🦁 Predator (Brown circle)
- ⭐ Leader (Gold circle)

---

## Key Design Decisions

### Murmuration-Like Movement
The simulation prioritizes synchronized, flowing movement similar to starling murmurations:
- Very high alignment weight (3.0 default, up to 4.0+)
- Strong cohesion to keep groups together
- Consistent speed maintenance
- Wide perception radius (120px)
- High alignment force multipliers (0.15)

### Obstacle Avoidance
Animals properly navigate around obstacles and water:
- Strong avoidance forces (6x multiplier)
- Larger avoidance radii than visual size
- No walking through trees or water
- Smooth steering around obstacles

### Species Differentiation
Each species has distinct flocking characteristics:
- Gazelles: Faster, moderate cohesion, independent
- Zebras: Slower, very cohesive, highly synchronized
- Visual differences in size and appearance

### Trail Visibility
Trails are prominent to show movement patterns:
- High opacity (60-70%)
- Frequent updates (every 2 frames)
- Larger footprint sizes
- Gradual fade for depth

---

## User Interaction Flow

1. **Initial State:**
   - 15 gazelles spawn randomly
   - 10 zebras spawn randomly
   - 3 trees placed randomly
   - Watering hole at center
   - Trails enabled by default
   - Murmuration-style movement active

2. **Adjusting Parameters:**
   - User moves sliders to tune flocking behavior
   - Changes are applied immediately
   - Visual feedback in real-time

3. **Preset Behaviors:**
   - Click preset buttons for pre-configured behaviors
   - Sliders update to reflect preset values
   - Observe different movement patterns

4. **Adding Features:**
   - Click "Add Predator" → click canvas to place
   - Click "Tag Leader" → click gazelle to mark
   - Click "Add Tree" → click canvas to place
   - Click "Follow Mouse" → animals chase cursor

5. **Viewing Information:**
   - Live stats update continuously
   - Toggle trails to see/hide movement history
   - Toggle perception radius to see neighbor detection

---

## Performance Considerations

- Target: 60 FPS with 25 animals
- Canvas size: ~1000x700 max
- Trail length: 30-40 points per animal
- Neighbor search: O(n²) acceptable for <100 animals
- Force calculations: Applied every frame
- Rendering: Draw trails → obstacles → animals → UI overlays

---

## Future Enhancement Ideas (Not Implemented)

These features were discussed but not included in final version:
- Reproduction mechanics based on food abundance
- Drought effects on cohesion
- Food abundance affecting speed
- Multiple herding clusters with cluster loyalty
- Species-specific cluster initialization

---

## Testing & Validation

**Behavioral Tests:**
- ✓ Animals move in synchronized groups
- ✓ Smooth murmuration-like flowing motion
- ✓ Prey flee from predators effectively
- ✓ Animals avoid obstacles and water
- ✓ Leaders influence their species
- ✓ Mouse following works smoothly

**Control Tests:**
- ✓ All sliders affect behavior correctly
- ✓ Preset buttons apply correct values
- ✓ Pause/Resume works
- ✓ Reset reinitializes properly
- ✓ Toggle trails shows/hides + clears
- ✓ Boundary toggle switches modes

**Visual Tests:**
- ✓ Animals render as realistic savanna creatures
- ✓ Trails are visible and fade appropriately
- ✓ UI is readable and well-organized
- ✓ Canvas maintains proper aspect ratio

---

## Project Completion Status

**✅ Completed Features:**
- Core boids algorithm with adjustable parameters
- African savanna theme with realistic animal designs
- Mouse following system
- Predator/prey dynamics
- Leader tagging
- Obstacle avoidance with trees
- Watering hole with attraction/avoidance
- Three preset behaviors
- Live statistics dashboard
- Motion trails (footprints)
- Pause/Resume/Reset controls
- Toggle trails button
- Boundary mode switching
- Murmuration-like synchronized movement
- Perception radius visualization

**❌ Removed Features:**
- Environmental factors (drought, food abundance)
- Cluster-based initialization
- Cluster loyalty system

---

## File Deliverable

**Filename:** `savanna-ecosystem.html`

**Size:** ~1400+ lines of HTML/CSS/JavaScript

**Browser Compatibility:** Modern browsers with Canvas API support (Chrome, Firefox, Safari, Edge)

---

## Summary

This project successfully implements an educational and visually engaging boids simulation with African savanna theming. The simulation demonstrates advanced flocking behavior resembling starling murmurations, with intuitive controls for exploring different parameters and behaviors. The realistic animal designs, environmental features (watering hole, obstacles), and interactive elements (predators, leaders, mouse following) create a rich ecosystem simulation suitable for educational purposes or as a demonstration of emergent behavior in artificial life systems.
