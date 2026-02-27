# 🍪 Can't Catch Me! — Game Specification

**Type:** Endless side-scrolling runner  
**Engine:** Vanilla HTML5 Canvas + Web Audio API (single `.html` file)  
**Target:** Desktop & mobile browser, fullscreen 16:9 letterboxed

---

## Visual Design

### Theme
Candy-land aesthetic with a gingerbread man protagonist. Four distinct worlds cycle as the player progresses, each with its own color palette, parallax layers, ground texture, and obstacle set.

### Color Palettes by Zone

| Zone | Sky Top | Sky Bottom | Ground | Accent |
|------|---------|------------|--------|--------|
| 🌿 Gumdrop Hills | `#5BC8F5` | `#FFD6E8` | `#7BC74D` | `#FFB6C1` |
| 🍫 Chocolate River | `#7B4A1E` | `#C8822B` | `#3D1F00` | `#D2691E` |
| 🌈 Sprinkle Sky | `#B8E4FF` | `#FFE8F5` | `#E87DD0` | `#FF69B4` |
| 🏰 Candy Castle | `#1A003A` | `#6B0099` | `#8B0045` | `#FF2280` |

### Parallax Layers (per zone, independent scroll)

Each zone maintains two independent scroll offsets (`scrollFar`, `scrollMid`) that persist across crossfades so layers don't snap.

**Gumdrop Hills**
- Far layer (speed ×0.12): Rolling pink hill arcs at 35% alpha, repeating every 260px
- Mid layer (speed ×0.28): Darker pink hills at 55% alpha, repeating every 210px
- Floating: Gumdrop orbs (5 colors) drifting slowly with gentle rotation
- Ground pattern: 5-petal flowers every 60px in rotating colors

**Chocolate River**
- Far layer (×0.10): Semi-transparent brown waterfall columns every 140px
- Mid layer (×0.22): Chocolate mountain silhouettes every 300px
- Floating: Brown drip-drop shapes falling from top
- Ground pattern: Caramel ripple arcs every 50px

**Sprinkle Sky**
- Far layer (×0.08): Fluffy white clouds every 380px with vertical bob
- Mid layer (×0.18): 6-color rainbow stripe bands across sky (7% alpha each)
- Floating: Colorful sprinkle sticks rotating as they drift
- Ground pattern: Scattered sprinkle sticks (20 total, cycling across width)

**Candy Castle**
- Far layer (×0.08): Gothic castle silhouettes with battlements every 500px
- Mid layer (×0.16): Glowing arched windows (gold, 40% alpha)
- Floating: 5-pointed stars (gold/pink) with twinkling sine animation
- Ground pattern: Pink neon tile grid, 40px spacing
- Accent: Dashed neon pink line at y=50

### Zone Transitions (Procedural Theming)
- **Trigger:** Score milestones, not a timer — zones feel *earned*
- **Method:** Both outgoing and incoming zones render simultaneously with complementary `globalAlpha` values (`1-t` and `t`)
- **Sky/ground colors:** Always smoothly `lerpColor()`-interpolated between zones
- **Duration:** Each crossfade spans 400–1400 score points depending on zone
- **Looping:** After Candy Castle, cycles back to Gumdrop Hills with lap counter (✕2, ✕3...)

**Zone score thresholds (Lap 1):**

| Zone | scoreStart | fadeLen |
|------|-----------|---------|
| Gumdrop Hills | 0 | 400 |
| Chocolate River | 1,200 | 600 |
| Sprinkle Sky | 3,500 | 900 |
| Candy Castle | 7,500 | 1,400 |

Each subsequent lap adds 7,500 to all thresholds.

### Zone Toast Notification
- Pill-shaped overlay slides in from top on zone entry
- Shows zone emoji + name + lap number if >1 (e.g. "🍫 Chocolate River ✕2")
- Styled with zone accent color border
- Auto-dismisses after 3 seconds

### Zone Progress Bar
- 3px strip along bottom edge of canvas
- Fills from 0→100% showing progress toward next zone
- Gradient between current and next zone accent colors

---

## Player Mechanics

### Character
- Gingerbread man, fully drawn procedurally in canvas
- Faces right (flipped via `ctx.scale(-1,1)`)
- Origin point at feet (ground contact)
- Full limb animation: leg swing, arm swing, foot position — all driven by `Math.sin(gameTime)`

### Physics
- **Gravity:** Variable — `0.45` while rising, `1.1` while falling (snappy arc, floaty peak)
- **Terminal velocity:** 22 px/frame
- **Jump force:** `-14` (normal), `-18` with Super Jump powerup
- **Double jump:** Always available; force `-13` (normal), `-17` with powerup

### Jump Feel (Juice)
- **Anticipation crouch:** On keydown while grounded, squash to `squashY=0.72` before launch
- **Launch squash:** `squashY=0.55, squashX=1.5` on jump
- **Peak stretch:** When `|vy| < 2.5` near apex, nudge toward `squashY=1.22` (elongated)
- **Landing squash:** Scales with impact velocity — harder fall = more squash
- **Coyote time:** 6 frames after walking off a ledge, jump still fires
- **Jump buffer:** Press jump up to 8 frames early; fires automatically on landing

### Squash & Stretch System
Spring physics:
```
sqDiff = 1 - squashY
squashVY += sqDiff * 0.3
squashVY *= 0.8  (damping)
squashY += squashVY
squashX = 1 + (1 - squashY) * -0.5  (inverse coupling)
```
Clamped: `squashY` in [0.5, 1.5], `squashX` in [0.5, 1.5]

### Input
- `SPACE` or `↑` arrow: jump
- Canvas click / tap: jump
- All input flows through a single `jump()` function

### Animation States
| State | Legs | Arms |
|-------|------|------|
| Running | `sin(gameTime×0.35)×0.45` swing | opposite phase |
| Jumping (up) | Static | raised (`-0.5`) |
| Falling | Static | lowered (`+0.3`) |
| Dead | Fade out over 60 frames | N/A |

---

## Hazard Types & Behaviors

### Zone 0 — Gumdrop Hills

| Obstacle | Behavior | Hitbox |
|----------|----------|--------|
| **Spike** | Static on ground, triangular | Reduced margin (6px) |
| **Donut** | Static, ring-shaped — safe in the hole | Ring hitbox: outer `r×0.54`, inner `r×0.22` safe |
| **Pit** | Gap in ground, falls are lethal | Player x between edges AND feet at ground level |
| **Gummy Bear** | Snap trap — animates open/closed via `sin()` | Only deadly when `snapFrac > 0.4` (mouth open) |

### Zone 1 — Chocolate River

| Obstacle | Behavior | Hitbox |
|----------|----------|--------|
| **Chocolate Wall** | Segmented bar with drip detail | Standard solid (6px margin) |
| **Chocolate Pit** | Bubbling lava pit with hot glow | Pit logic (edge margin 8px) |
| **Rolling Ball** | Truffle falls from 220px above, bounces once | Dangerous below y=180; `vy += 0.4`, bounces at `vy × -0.35` |
| **Sticky Puddle** | Wide caramel spread with warning symbol | Pit logic |

### Zone 2 — Sprinkle Sky

| Obstacle | Behavior | Hitbox |
|----------|----------|--------|
| **Ice Cream** | Cone + 2 scoops + cherry, tall static | Standard solid |
| **Lollipop** | Swirl disc on stick | Standard solid |
| **Cloud Puff** | Hangs at 145px above ground | Kills if head inside cloud (`x+10, y+5, w-20, h-5`) |
| **Sugar Crystal** | 3 crystal spires | Standard solid |

### Zone 3 — Candy Castle

| Obstacle | Behavior | Hitbox |
|----------|----------|--------|
| **Candy Cane** | Red/white striped hook | Standard solid |
| **Licorice** | Swings like a pendulum via `sin(gameTime×0.07)` | Tip hitbox only (28×28px) |
| **Jawbreaker** | Falls from 240px, bounces | Same as Rolling Ball |
| **Guard Tower** | Mini castle with battlements | Standard solid |

### Hitbox Notes
- Player hitbox is **forgiving**: 22px wide × `h×0.65` tall (narrower/shorter than visual)
- **Invincibility frames:** 120 frames (2s) after shield breaks
- Ghost powerup skips all collision checks entirely

---

## Pattern Library (28 Chunks)

Patterns are arrays of obstacle descriptors with relative x offsets. Each chunk spawns as a unit with guaranteed 600–1100px clear runway before the next.

### Zone 0 Patterns
1. Single spike
2. Single donut
3. Wide pit (78px)
4. Double spike — gap of 130px between
5. Single gummy bear
6. Spike + donut (offset 160px)
7. Donut + pit (offset 180px)
8. Gummy + spike (offset 200px)

### Zone 1 Patterns
9. Single chocolate pit (90px wide)
10. Single chocolate wall (short)
11. Double chocolate wall — short then tall (140px gap)
12. Rolling ball (falls from height)
13. Sticky puddle (100px wide)
14. Chocolate pit + wall combo
15. Rolling ball + pit combo

### Zone 2 Patterns
16. Single ice cream
17. Single lollipop
18. Cloud puff (hangs overhead)
19. Sugar crystal cluster
20. Double ice cream (160px gap)
21. Lollipop + crystal combo
22. Cloud + ice cream combo

### Zone 3 Patterns
23. Single candy cane
24. Swinging licorice (hanging)
25. Falling jawbreaker
26. Double candy cane — different heights
27. Double licorice (220px gap)
28. Jawbreaker + candy cane combo

### Tier Unlock System
Each zone's harder combo patterns unlock progressively by score:

| Zone | Tier 1 (always) | Tier 2 | Tier 3 |
|------|----------------|--------|--------|
| Gumdrop | 0 | 400 | 700–1000 |
| Chocolate | 0 (zone entry) | 1500 | 2200–3200 |
| Sprinkle | 0 (zone entry) | 4500 | 5500–6800 |
| Castle | 0 (zone entry) | 9000 | 11000–13000 |

---

## Juice Effects

### Squash & Stretch
Spring-physics system (see Player Mechanics). Applied to the entire player via `ctx.scale(squashX, squashY)` with foot-anchor compensation.

### Screen Shake
- Hit/collision: `screenShake = 10`
- Death: `screenShake = 20`
- Shield break: `screenShake = 12`
- Applied as `ctx.translate(shakeX, shakeY)` on every render frame
- Decays: `screenShake -= dt × 0.8`

### Freeze Frames (Hit-Stop)
- **Death:** 3-frame pause — world freezes, makes impact feel massive
- **Powerup collect:** 1-frame pop
- Screen shake continues during freeze for extra drama
- Implemented by early-returning from `update()` while `freezeFrames > 0`

### Particles
| Type | Trigger | Appearance |
|------|---------|------------|
| **Crumbs** | Landing (5) + death (20) | Brown squares, rotate as they fall |
| **Frosting splat** | Death | White blobs spread in arc, linger |
| **Sparkles** | Double jump, powerup collect | 8-point stars, 12 burst outward |
| **Powerup burst** | Powerup activated | 12 colored circles in zone's palette |
| **Coin collect** | Coin pickup | 4 gold circles burst outward |
| **Speed trail** | Running at speed > 9 | Horizontal white streaks behind player |
| **Super Jump flame** | Active powerup | Green flame particles under feet each frame |
| **Shield break burst** | Shield absorbs hit | 16 cyan circles explode outward |

All particles have: position, velocity, gravity (`+0.2`/frame), life (0→1 fade), maxLife, size, color, type.

### Anticipation
- Pre-jump crouch squash on `keydown` (before `jump()` fires)
- Peak stretch at jump apex (`|vy| < 2.5`)
- Impact-scaled landing squash

### Zone Transition Flash
No longer used (replaced by smooth alpha crossfade). `zoneFlashAlpha` variable remains but is 0 throughout.

---

## Difficulty Progression

### Speed Formula
```
speed = 5 + Math.pow(score, 0.38) × 0.18
```
No hard cap — speed grows forever, decelerating asymptotically.

| Score | Speed | Obstacle gap |
|-------|-------|-------------|
| 0 | 5.0 | ~3.7s |
| 300 | 6.6 | ~2.8s |
| 1,200 | 7.7 | ~2.3s |
| 3,500 | 9.0 | ~1.9s |
| 7,500 | 10.3 | ~1.5s |
| 15,000 | 12.0 | ~1.0s |
| 30,000 | 14.0 | ~0.7s |

### Obstacle Gap Formula
```
clearPixels = Math.max(600, 1100 - score × 0.025)
patternCooldown = clearPixels / speed  (frames)
```

### Slow Motion Powerup
When active, `effectiveSpeed = speed × 0.45` for all obstacle/powerup/coin movement. Player physics unaffected.

---

## Powerups

Five powerup types spawn as spinning hex gems that bob vertically:

| Powerup | Color | Duration | Effect |
|---------|-------|----------|--------|
| 🛡️ Shield | Cyan `#00BFFF` | 8s | Absorbs one hit; blue ring visual around player; shatters with burst |
| 🧲 Magnet | Pink `#FF69B4` | 7s | Pulls coins and powerups within 200–250px |
| ⏱️ Slow Mo | Gold `#FFD700` | 5s | All obstacles at 45% speed; blue screen tint |
| 🦿 Super Jump | Lime `#ADFF2F` | 6s | Jump force ×1.28; green flame trail |
| 👻 Ghost | Purple `#DA70D6` | 4s | Complete collision immunity; purple aura |

Powerup spawn cooldown: 300–600 frames (randomized). Coins spawn in 5-coin arcs for +20 score each.

Active powerup HUD: pill badges with shrinking color bars shown at bottom center.

---

## UI

### Tutorial Screen (first launch only)
- Full-viewport overlay with blur backdrop
- 2×2 grid of rule cards: Jump, Dodge Hazards, Grab Powerups, Collect Coins
- Powerup quick-reference row (5 chips)
- Zone progression strip listing all 4 worlds
- "▶ Let's Run!" button — dismisses tutorial and starts game

### HUD (in-game)
- **Top left:** Current score
- **Top center:** Zone name (updates on transition)
- **Top right:** High score ("Best")
- **Bottom center:** Active powerup bars (dynamic, injected by JS)
- **Bottom edge:** Zone progress bar (3px, gradient)
- **Above player:** "2×" indicator when double jump is available mid-air

### Game Over Screen
- Title: "Oh Crumbs!" (shake animation)
- Final score display
- "🍪 Run Again!" button — shows compact restart screen (not full tutorial)

### Restart Screen (subsequent runs)
- Simple overlay: title + "▶ RUN!" button
- Tutorial not shown again

### Zone Toast
- Slides in from top on zone entry
- Zone emoji + name + lap number if looping

---

## Audio

All audio uses Web Audio API (`OscillatorNode` + `GainNode` with exponential ramp to 0). No external files required.

### Sound Effects

| Sound | Trigger | Character |
|-------|---------|-----------|
| **Jump** | First jump | 300→500→700 Hz sine arpeggio, 3 notes |
| **Double jump** | Second jump | 600→900→1200 Hz sine, brighter |
| **Land** | Touch ground | 150 Hz square + 80 Hz sine thud |
| **Hit/death** | Collision | 100 Hz sawtooth + 60 Hz square, harsh |
| **Death sequence** | Game over | 5-note descending sawtooth, 80ms apart |
| **Coin collect** | Coin pickup | 1200→900 Hz ping |
| **Powerup** | Gem collected | 523→659→784→1047 Hz rising jingle (4 notes) |
| **Shield hit** | Shield absorbs | 400→500→300 Hz soft thud, 3 notes |

### Background Music
Procedurally generated, tempo-synced to game speed:

- **BPM:** `120 + musicSpeed × 40` (scales with speed)
- **Bass:** Square wave on downbeat, cycles through `[130, 146, 164, 196]` Hz
- **Hi-hat:** High-frequency square burst every 16th note
- **Melody:** Triangle wave every other 16th note, cycles through 6-note scale

Music restarts when BPM changes by more than 20%. Stops on death, restarts on new game.

---

## High Score Persistence

```javascript
// Save
localStorage.setItem('gbHighScore', highScore);

// Load
let highScore = parseInt(localStorage.getItem('gbHighScore') || '0');
```

High score is checked and updated on every game over. Displayed in top-right HUD throughout gameplay and on the game over screen.

---

## Technical Notes

### Canvas Setup
- Fixed 16:9 aspect ratio, letterboxed to fit window
- `resizeCanvas()` called on load and `window.resize`
- Container sized to match canvas so UI overlays align exactly
- `GROUND_Y = H - 80` recalculated on resize

### Game Loop
```
requestAnimationFrame(loop)
  dt = clamp((now - lastTime) / 16.67, 0, 3)
  update(dt)   // physics, scoring, spawning, collision
  render()     // background, obstacles, player, particles, UI
```

### Coordinate System
- Origin top-left
- Player `y` = foot contact point (ground level)
- `GROUND_Y = H - 80`
- Obstacles: `y` = top of sprite for solids, `GROUND_Y` for pits

### Performance
- `bgParticles`: 50 persistent ambient particles, reused each frame
- `particles`: ephemeral, filtered when `life <= 0`
- `obstacles`: filtered when `x + w < -80` (off-screen left)
- No external dependencies — everything drawn procedurally
