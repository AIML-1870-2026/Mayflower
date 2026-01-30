# Dragon's Quest - Game Specification Document

## 📋 Overview

**Dragon's Quest** is a medieval-themed Snake game where players control a dragon that must collect treasure while avoiding cannonballs from attacking knights. The dragon evolves through six color stages as it grows more powerful, culminating in the legendary Rainbow Dragon King form.

**Genre**: Arcade / Action  
**Platform**: Web Browser (HTML5/Canvas)  
**Target Audience**: All ages  
**Play Style**: Single-player, endless survival  

---

## 🎮 Core Gameplay

### Objective
- Collect treasure to grow your dragon
- Survive as long as possible
- Achieve the highest score
- Evolve to Rainbow Dragon King (750+ points)

### Win Condition
There is no traditional "win" - this is an endless survival game. Success is measured by:
- High score achievement
- Dragon evolution progress
- Time survived
- Personal records

### Lose Condition
Game ends when the dragon collides with:
- Board walls/borders
- Its own body/tail
- Knight cannonballs

---

## 🐉 Dragon Mechanics

### Movement
- **Grid-based**: 30 columns × 20 rows
- **Controls**: Arrow Keys or WASD
- **Speed**: Starts at 120ms per move, increases with score
- **Direction**: Cannot reverse 180° into own body
- **Input buffering**: Queues next direction to prevent missed turns

### Visual Design

#### Head
- Circular shape with directional features
- **Eyes**: Glowing yellow with black pupils and white shine
- **Snout**: Extended forward with nostrils
- **Horns**: Two curved brown horns with golden tips
- **Fire Breath**: Periodic flame animation (every 3 seconds)
  - Orange/yellow gradient flames
  - Shoots forward 3-4 tiles
  - Flickering animation with inner bright core

#### Wings (Flapping Animation)
- **Size**: Extend 40 pixels from body
- **Structure**: 
  - 3 visible wing bones per wing
  - Semi-transparent membrane between bones
  - Edge highlights for depth
- **Animation**: 
  - Flap up and down in sine wave pattern
  - 8 flaps per second
  - Larger amplitude than body movement
- **Appearance**: Realistic bat-like dragon wings

#### Body Segments
- Circular segments (not squares)
- Scale texture details
- Lighter belly/underside
- Smooth appearance with shadows

#### Tail (Last Segment)
- **Arrow/spade shaped tip** extending past last segment
- **Spine ridge**: Dark line through center
- **Highlight**: White edge for 3D effect
- **Directional**: Points away from body
- **Tapered**: Gets narrower at the tip

### Evolution System

Dragon evolves through 6 stages based on score:

| Stage | Score Range | Scale Color | Description |
|-------|-------------|-------------|-------------|
| **Hatchling** | 0-50 | Green (#4CAF50) | Small, innocent baby dragon |
| **Young Dragon** | 51-150 | Blue (#2196F3) | Gaining strength and size |
| **Adult Dragon** | 151-300 | Red (#F44336) | Fierce warrior |
| **Ancient Dragon** | 301-500 | Purple (#9C27B0) | Legendary power |
| **Elder Dragon** | 501-750 | Gold (#FFD700) | Near-royalty status |
| **Dragon King** | 750+ | Rainbow (animated) | Ultimate form, worthy of crown |

#### Evolution Effects
When transitioning between stages:
- Screen flash animation (white overlay, 200ms)
- Particle burst (20 particles)
- "EVOLUTION!" banner appears center screen
- Brief celebration animation
- Updated HUD showing new stage name

---

## 💰 Treasure System

Treasure spawns continuously - there is ALWAYS at least one piece of food on screen at all times.

### Regular Treasure Types (8 varieties)

| Item | Emoji | Points | Spawn Rate | Description |
|------|-------|--------|------------|-------------|
| Gold Coin | 💰 | 5 | 35% | Common currency |
| Coin Pouch | 👛 | 10 | 20% | Bag of coins |
| Meat | 🍖 | 12 | 15% | Dragon food |
| Fish | 🐟 | 8 | 12% | Fresh catch |
| Ring | 💍 | 15 | 8% | Jeweled ring |
| Ruby Gem | 💎 | 20 | 5% | Precious gemstone |
| Chalice | 🏆 | 25 | 3% | Golden cup |
| Treasure Chest | 💼 | 30 | 2% | Rare treasure hoard |

### Visual Effects
- **Wobbling**: Sine wave animation (±2px vertical)
- **Glow**: Colored aura around each item
- **Sparkle**: White highlight particle
- **Emoji Display**: Shows appropriate emoji icon

### Special Item: Golden Crown 👑

- **Spawn Timer**: Every 20-30 seconds (random)
- **Duration**: Visible for 8 seconds only
- **Points**: 100
- **Visual**: 
  - Large ornate crown with jewels
  - Pulsing scale animation
  - Golden glow/halo
  - Countdown indicator
- **Special Effect**: "Royalty Mode"
  - None currently active (legacy feature)

### Treasure Spawning Logic
- **Immediate spawn** on game start
- **Instant respawn** when collected
- **Forced spawn** even if no empty position found
- **Safety check** every game update to ensure treasure exists
- **Console logging** for debugging

---

## ⚔️ Knight Catapult System

Medieval knights with catapults attack the dragon from the sides of the screen.

### Spawn Mechanics
- **Frequency**: Every 8 seconds
- **Spawn Chance**: 85% when timer expires
- **First Spawn**: 3 seconds after game starts
- **Spawn Location**: Randomly from left or right side
- **Vertical Position**: Random mid-height range

### Knight Visual Design

#### Knight Character
- **Armor**: Silver/grey (#C0C0C0)
- **Helmet**: Gold (#FFD700)
- **Plume**: Red feathers on helmet top
- **Shield**: Blue (#0000CD) with gold cross
- **Position**: Standing beside catapult

#### Catapult
- **Base**: Brown wood (#8B4513)
- **Arm**: Wooden beam that rotates
- **Loading Animation**: 
  - Arm pulls back over 3 seconds (60 frames)
  - Shows cannonball in basket
  - Tension builds visually
- **Fire Sequence**: Arm releases, cannonball launches

### Warning System
- **⚠️ Warning Symbol**: Flashes above knight
- **Duration**: 3 seconds before firing
- **Visual**: Red semi-transparent emoji
- **Gives player time** to prepare dodge

### Knight Lifecycle
1. Appears on left or right edge
2. Catapult arm loads (3 seconds)
3. Warning symbol flashes
4. Fires cannonball at dragon head
5. Smoke particles from cannon
6. Knight disappears after ~5 more seconds

### Cannonball Physics

#### Visual Design
- **Size**: 12 pixel radius
- **Color**: Dark grey to black gradient
- **Highlight**: White shine on top-left
- **Shadow**: Oval shadow beneath
- **Motion Trail**: Smoke trail behind

#### Trajectory
- **Aim**: Targets dragon's head position when fired
- **Speed**: 8 pixels per frame
- **Gravity**: 0.15 pixels/frame² downward
- **Arc Path**: Follows realistic ballistic arc
- **Prediction**: Aims where dragon WAS, not where it's going

#### Collision
- **Instant Death**: Any dragon segment hit = game over
- **Grid-based Detection**: Converts pixel position to grid coordinates
- **No mercy**: Cannot be dodged once hit

---

## 🌀 Portal System

Magical fire portals allow the dragon to teleport across the board.

### Spawn Mechanics
- **Score Threshold**: Appears at 200 points
- **Quantity**: Always 2 portals (linked pair)
- **Respawn**: After use, new pair spawns after 3 second cooldown
- **Placement**: Random positions avoiding obstacles/treasure

### Visual Design (Rings of Fire)

#### Outer Ring
- **12 rotating flames** around portal
- **Colors**: Alternating orange and yellow
- **Animation**: Flames flicker and rotate
- **Glow**: Orange radial gradient expanding outward

#### Stone Arch Border
- **Material**: Medieval stone (#8B4513)
- **Thickness**: 4 pixels
- **Highlight**: Lighter edge on one side for 3D effect

#### Portal Center
- **Swirling darkness**: Dark red to black gradient
- **Spiral Effect**: 3 rotating spiral lines
- **Depth**: Darker in center, lighter at edges

#### Spark Particles
- **Quantity**: 5 particles
- **Color**: Yellow-white (#FFFF64)
- **Motion**: Orbit around portal
- **Size**: 2 pixel radius

### Portal Mechanics
1. Dragon touches portal entrance
2. Instantly teleports to exit portal
3. 1 second invincibility (golden aura)
4. Both portals despawn
5. New portal pair spawns after cooldown
6. Particle effects at both locations

---

## 🏰 Environment & Background

### Sky
- **Gradient**: Light blue (#87CEEB) to green (#6B8E23)
- **Effect**: Simulates sky fading to ground

### Clouds
- **Quantity**: 4 white fluffy clouds
- **Movement**: Drift left slowly (parallax effect)
- **Design**: Multiple overlapping circles
- **Transparency**: Semi-transparent white

### Storm Clouds (Background)
Replace castles with ominous storm clouds:

- **Quantity**: 5 large dark clouds
- **Colors**: Dark grey (#3C3C46) with darker centers
- **Shape**: Overlapping circles creating cloud mass
- **Lightning**: Random lightning bolts (2% chance per frame)
  - Yellow-white color
  - Zigzag pattern
  - Flashes briefly
- **Placement**: Scattered across background sky
- **Effect**: Creates stormy, dramatic atmosphere

### Kingdom Ground (Bottom Third)

#### Layers
1. **Grass**: Green gradient (#6B8E23 to #556B2F)
2. **Stone Path**: Grey cobblestones (#A9A9A9)
3. **Castle Wall**: Grey wall at very bottom with crenellations

#### Village Buildings (3 houses)
- **Material**: Brown wood with red roofs
- **Windows**: Glowing yellow (#FFD700)
- **Doors**: Dark brown wood
- **Style**: Medieval half-timbered houses
- **Spacing**: Distributed across bottom

#### Castle Wall Details
- **Height**: 40 pixels at bottom
- **Color**: Grey (#696969)
- **Crenellations**: Notched top every 60 pixels
- **Brick Pattern**: Horizontal lines showing construction

### Grid
- **Color**: Semi-transparent brown (#8B4513 at 15% opacity)
- **Purpose**: Helps with navigation
- **Subtlety**: Barely visible, doesn't interfere with gameplay

---

## 📊 HUD (Heads-Up Display)

Located ABOVE the game canvas (not overlapping play area).

### Display Elements

| Element | Icon | Data Shown |
|---------|------|------------|
| Dragon Length | 🐉 | Number of body segments |
| Current Score | 💰 | Points earned this game |
| High Score | 👑 | Best score ever achieved |
| Fire Breath | 🔥 | Cooldown bar (not functional) |
| Evolution Stage | ⭐ | Current dragon form name |

### HUD Styling
- **Background**: Dark semi-transparent (#141414 at 90%)
- **Border**: Gold (#FFD700) 2px
- **Font**: MedievalSharp for medieval feel
- **Layout**: Horizontal row, evenly spaced
- **Colors**: Gold text on dark background

---

## 🎯 Game States & Screens

### Main Menu
- **Title**: "🐉 DRAGON'S QUEST 👑"
- **Subtitle**: "Claim the Crown, Rule the Kingdom"
- **Buttons**:
  - ⚔️ START QUEST (opens tutorial)
  - 👑 HALL OF KINGS (shows leaderboard)
- **Styling**: Medieval theme with gold borders

### Tutorial Screen
- **Title**: "Welcome, Young Dragon!"
- **Content**:
  - Controls explanation (Arrow/WASD, Spacebar)
  - Gameplay objectives
  - Evolution system teaser
  - Specific hazards listed:
    - 💰 Collect treasure
    - 👑 Grab crowns
    - 🌀 Use portals
    - ⚔️ Dodge cannonballs
    - 🧱 Avoid walls and tail
- **Buttons**: BEGIN THE QUEST, Back

### Game Over Screen

#### Statistics Display
- **Final Score**: Total points earned
- **Dragon Length**: Number of segments
- **Evolution Reached**: Highest form achieved
- **Time Survived**: Minutes:seconds format

#### Leaderboard Entry
- **Top 5 Scores**: Saved in localStorage
- **Name Entry**: Prompt for name if score makes top 5
- **Ranking Icons**:
  - 1st: 👑👑👑
  - 2nd: 👑👑
  - 3rd: 👑
  - 4th-5th: ⭐

#### Buttons
- PLAY AGAIN (restart game)
- MAIN MENU (return to title)

### Hall of Kings (Leaderboard)
- **Display**: Top 5 all-time scores
- **Format**: Rank, Name, Score, Crowns
- **Storage**: Browser localStorage
- **Sorting**: Highest score first
- **Fallback**: "No scores yet!" if empty

---

## ⚙️ Technical Specifications

### Technology Stack
- **HTML5**: Structure and canvas element
- **CSS3**: Styling and animations
- **JavaScript (ES6+)**: Game logic and rendering
- **Canvas API**: 2D rendering
- **localStorage**: Data persistence

### Performance Optimizations

#### Canvas Size
- **Resolution**: 720x480 pixels
- **Reason**: Reduced by 36% for better performance
- **Aspect Ratio**: 3:2
- **Responsive**: Scales with CSS

#### Particle System
- **Limit**: Maximum 50 particles rendered
- **Culling**: Older particles removed first
- **Reduced Counts**:
  - Treasure: 8 particles (was 10)
  - Crown: 20 particles (was 30)
  - Portal: 15 particles (was 20)
  - Cannon: 10 particles (was 15)

#### Cloud Simplification
- **Quantity**: 4 clouds (was 5)
- **Circles**: 3 per cloud (was 4)
- **Purpose**: Reduce draw calls

#### Village Buildings
- **Quantity**: 3 buildings (was 5)
- **Purpose**: Reduce draw calls

### Game Loop
- **Frame Rate**: 60 FPS target (requestAnimationFrame)
- **Update Rate**: Based on game speed (120ms initially)
- **Rendering**: Every frame
- **Time-based**: Delta time calculations

### Grid System
- **Size**: 30 columns × 20 rows
- **Cell Size**: 24 pixels
- **Total**: 600 playable cells
- **Coordinate System**: 0-indexed

### Speed Progression
- **Starting Speed**: 120ms between moves
- **Speed Increase**: -2ms per treasure collected
- **Minimum Speed**: 50ms (fastest possible)
- **Effect**: Game gets progressively harder

### Data Persistence
- **High Score**: Saved to localStorage
- **Leaderboard**: Top 5 scores saved
- **Key**: 'dragonQuestHighScore' and 'dragonQuestLeaderboard'
- **Format**: JSON for leaderboard

---

## 🎨 Visual Effects & Polish

### Particle Effects
- **Treasure Collection**: Gold sparkles burst outward
- **Crown Collection**: Major golden particle explosion
- **Evolution**: Magical circle with particles
- **Portal Use**: Fire and smoke particles at both portals
- **Cannon Fire**: Grey smoke puff
- **Gravity**: Particles fall downward over time

### Screen Effects
- **Evolution Flash**: White screen flash on evolution
- **Evolution Banner**: "EVOLUTION!" text appears

### Animations
- **Dragon Wings**: Continuous flapping (8 flaps/second)
- **Fire Breath**: Periodic flame bursts (every 3 seconds)
- **Treasure Wobble**: Vertical sine wave motion
- **Crown Pulse**: Scale pulsing animation
- **Portal Flames**: Rotating fire ring
- **Lightning**: Random flashes in storm clouds
- **Clouds**: Slow horizontal drift

### Audio (Not Implemented)
Placeholder for future audio system:
- Background music (progressive intensity)
- Treasure collection sounds
- Evolution fanfare
- Cannonball fire sound
- Portal whoosh
- Dragon roar on death

---

## 🎯 Scoring System

### Points Breakdown
- Gold Coin: 5 points
- Coin Pouch: 10 points
- Meat: 12 points
- Fish: 8 points
- Ring: 15 points
- Ruby: 20 points
- Chalice: 25 points
- Chest: 30 points
- **Golden Crown: 100 points**

### High Score
- Saved to localStorage
- Displayed in HUD
- Shown on game over screen
- Required for leaderboard entry (top 5)

---

## 🐛 Known Issues & Limitations

### Current Limitations
1. **Fire Breath**: Visual only, no gameplay effect
2. **Audio**: No sound effects or music
3. **Mobile**: Not fully optimized for touch controls
4. **Obstacles**: Removed for gameplay balance

### Debug Features
- **Console Logging**: Shows treasure spawns, knight spawns
- **Emergency Spawns**: Logged when backup systems activate
- **Browser Console**: Press F12 to view debug info

---

## 🎓 Educational Value

This project demonstrates:
- **Game Loop Architecture**: Update/render cycle
- **State Management**: Game states and transitions
- **Collision Detection**: Grid-based checking
- **Canvas Rendering**: 2D graphics programming
- **Animation Techniques**: Interpolation and timing
- **Input Handling**: Keyboard controls and buffering
- **Data Persistence**: localStorage usage
- **Performance Optimization**: Particle limits, rendering efficiency

---

## 🚀 Deployment

### GitHub Pages Deployment

1. Create repository on GitHub
2. Upload `dragon-quest-single-file.html`
3. Rename to `index.html` for cleaner URL
4. Enable GitHub Pages in Settings → Pages
5. Select main branch, root folder
6. Access at: `https://[username].github.io/[repo-name]/`

### File Structure
```
dragon-quest/
├── index.html (main game file)
├── spec.md (this document)
└── README.md (project description)
```

---

## 📝 Credits

- **Game Design**: Enhanced snake mechanics with medieval theme
- **Art Style**: Pixel art / medieval fantasy aesthetic
- **Fonts**: MedievalSharp & Cinzel (Google Fonts)
- **Inspiration**: Classic Snake game with major enhancements

---

## 🔄 Version History

### v1.0 - Final Release
- Removed castle obstacles
- Fixed treasure spawning (always visible)
- Fixed knight spawning (immediate start)
- Optimized performance (reduced lag)
- Added storm clouds background
- Enhanced dragon visuals (wings, horns, tail)
- Implemented knight catapult system
- Added 8 treasure varieties
- Complete evolution system (6 stages)
- Portal teleportation system
- Full HUD implementation
- Leaderboard system

---

## 🎮 Play Instructions

### Starting the Game
1. Open `index.html` in web browser
2. Click "⚔️ START QUEST"
3. Read tutorial (optional)
4. Click "BEGIN THE QUEST!"
5. Use Arrow Keys or WASD to move

### Gameplay Tips
- **Early Game**: Focus on collecting treasure, learn movement
- **Mid Game**: Watch for knight warnings, use portals strategically
- **Late Game**: Speed increases, precision required
- **Crowns**: Worth 100 points, grab them quickly!
- **Portals**: Can save you from dangerous situations

### Winning Strategy
1. Collect treasure efficiently
2. Dodge cannonballs by changing direction
3. Use portals when cornered
4. Avoid trapping yourself
5. Stay near center of board when possible
6. Plan your path ahead

---

**End of Specification Document**

*Dragon's Quest - Claim the Crown, Rule the Kingdom! 🐉👑*
