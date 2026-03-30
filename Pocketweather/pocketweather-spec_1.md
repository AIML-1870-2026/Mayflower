# PocketWeather™ — Project Specification

**Version:** 1.0  
**File:** `weather-dashboard.html` (single self-contained HTML file)  
**Data Source:** OpenWeatherMap API (key: user-provided)

---

## 1. Overview

PocketWeather™ is a single-page, self-contained weather dashboard styled as a retro handheld gaming device (Game Boy / Tamagotchi aesthetic). It displays real-time weather data for any city in the world, fetched from the OpenWeatherMap API. The entire app lives in one HTML file with no external dependencies beyond Google Fonts and the OWM API.

---

## 2. Aesthetic & Design

### 2.1 Theme
- **Tamagotchi / Game Boy** — pixel art, chunky outlines, retro handheld charm
- The UI is rendered as a physical handheld device shell centered on the page
- Shell colors: cream body (`#f4f0e8`), purple-red top stripe, dark shadow borders
- Physical details: D-pad, A/B buttons, SELECT/START buttons, speaker dot grid, power LED

### 2.2 Typography
| Role | Font | Source |
|---|---|---|
| Display / headings / labels | Press Start 2P | Google Fonts |
| LCD readouts / values | VT323 | Google Fonts |

### 2.3 Screen Palette (Temperature-Driven)
The LCD screen background color shifts dynamically based on current temperature (°C). This is the primary visual feedback for temperature. The palette applies to CSS variables `--sc`, `--sc2`, `--sc3` which cascade to all screen UI elements.

| Temperature Range | Screen Color | Vibe |
|---|---|---|
| ≤ −15°C | Blue `#a8d8f8` | Arctic |
| −15 to −5°C | Light blue `#b8d8f0` | Freezing |
| −5 to 4°C | Steel blue `#bcd4e8` | Cold |
| 4 to 10°C | Teal `#bcd8d8` | Cool |
| 10 to 16°C | Sage green `#b8d8c0` | Mild |
| 16 to 22°C | Yellow-green `#c8dca8` | Nice |
| 22 to 27°C | Warm yellow `#d8dc98` | Warm |
| 27 to 32°C | Amber `#e8d090` | Hot |
| 32 to 38°C | Orange `#e8b880` | Scorching |
| > 38°C | Red-orange `#e8a070` | Extreme |

**Night mode** overrides the palette with deep blue-slate (`#6888a8`) regardless of temperature.

### 2.4 Day / Night Mode
- Determined by comparing current time against API-returned `sunrise` / `sunset` timestamps
- **Night:** Screen palette overrides to dark blue, animated star field appears behind the device, badge reads "🌙 NIGHT"
- **Day:** Temperature palette active, stars hidden, badge reads "☀ DAY"

---

## 3. Search System

### 3.1 Input Fields
| Field | ID | Notes |
|---|---|---|
| City | `#ic` | Required. Triggers autocomplete after 2+ chars. |
| State (US only) | `#is` | Optional. 2-letter code (NE, SD, CA…). Accepts full names and converts automatically. |
| Country | `#ico` | Optional. Full name input with dropdown autocomplete → resolves to ISO 2-letter code internally. |

### 3.2 Country Autocomplete
- User types a full country name (e.g. "United Kingdom", "France", "United States")
- Dropdown appears with up to 7 matching suggestions
- Suggestions ranked: starts-with matches first, then contains matches
- Navigation: mouse click, or ↑↓ arrow keys + Enter
- Supports aliases: "UK" → GB, "England" → GB, "USA" → US, "UAE" → AE, etc.
- 150+ countries in the lookup table
- Selecting a country sets an internal ISO code variable used by the API query

### 3.3 City Autocomplete
- Fires after 2+ characters with a **300ms debounce** to avoid hammering the API
- Calls `api.openweathermap.org/geo/1.0/direct` with up to 6 results
- If a country (and/or state) is already filled in, results are scoped to that country
- Each suggestion shows: **City Name** + subtle subtitle with state/country
- Clicking a suggestion:
  - Fills the city field
  - Auto-fills country field (if blank) with the matched country full name
  - Auto-fills state field (if blank and US city)
  - Immediately fetches and loads weather — no button press required
- Arrow key + Enter also selects and fetches

### 3.4 Query Resolution Logic
When GO is pressed (or Enter in city/state fields):

```
if selectedCity (picked from dropdown):
    → use stored lat/lon directly (skip geocoding step)
else:
    if state + country → query = "city,state,country"
    elif country only  → query = "city,country"
    else               → query = "city"  (worldwide search)
    → call geo/1.0/direct, get lat/lon from first result
→ call data/2.5/weather?lat=&lon=
→ call data/2.5/forecast?lat=&lon=
```

Leaving country blank searches worldwide — "London" alone will find London, GB.

### 3.5 State Code Normalization
Full US state names are automatically converted to 2-letter codes before the API call (e.g. "Nebraska" → "NE", "South Dakota" → "SD").

---

## 4. Weather Data Displayed

### 4.1 Current Conditions Panel
- City name, country, state (if available), lat/lon coordinates
- Weather description (e.g. "CLEAR SKY", "MODERATE RAIN")
- Current temperature (large display)
- Feels like temperature
- Pixel art weather icon (canvas-drawn, 72×72px, 8px pixel grid)
- Last sync timestamp

### 4.2 Stat Grid (2×2)
- Wind speed + direction (cardinal)
- Humidity %
- Pressure (hPa)
- Visibility (mi or km)

### 4.3 Progress Bars (pixel-step animated)
- Humidity %
- Cloud cover %
- Wind speed (scaled 0–30 m/s = 0–100%)

### 4.4 Solar Tracker Bar
- Pixel bar showing sun position from sunrise to sunset
- Filled proportionally based on current time between sunrise/sunset
- Sunrise and sunset times shown at each end

### 4.5 5-Day Forecast Strip
- Groups 3-hour forecast slots by calendar day
- Shows: day abbreviation, weather emoji, high temp, low temp
- Excludes today (shows next 5 days)
- Hover highlight effect

### 4.6 Scrolling Ticker
- Auto-scrolling pixel-font ticker at screen bottom
- Displays: city, temp, description, feels like, humidity, wind, pressure, cloud cover, sunrise/sunset

---

## 5. Pixel Art Weather Icons

Hand-drawn on `<canvas>` (72×72px) using an 8px pixel grid. Icons use flat pixel fills only.

| OWM Code | Icon Key | Appearance |
|---|---|---|
| 01d | `sun` | Yellow sun with rays |
| 01n | `moon` | Crescent moon |
| 02d/03d/04d | `partcloud` | Sun peeking behind cloud |
| 02n/03n/04n | `cloud` | Grey cloud |
| 09d/09n/10d/10n | `rain` | Cloud with rain drops |
| 11d/11n | `storm` | Dark cloud with lightning bolt |
| 13d/13n | `snow` | Cloud with snow dots |
| 50d/50n | `mist` | Horizontal fog lines |

---

## 6. Tamagotchi Pet System

### 6.1 Overview
An animated emoji-based pet character lives in a panel on the screen. The pet species is chosen automatically based on weather conditions and temperature. Tapping/clicking the pet triggers a "pet" interaction.

### 6.2 Pet Selection Logic (priority order)
| Pet | Emoji | Trigger Condition |
|---|---|---|
| Rainy Frog | 🐸 | OWM code starts with `09` or `10` (rain/drizzle) |
| Night Owl | 🦉 | Current time is after sunset or before sunrise |
| Polar Bear | 🐻‍❄️ | Temperature ≤ −5°C |
| Penguin | 🐧 | −5°C to 5°C |
| Fox | 🦊 | 5°C to 15°C |
| Duck | 🦆 | 15°C to 22°C |
| Sunny Cat | 🐱 | 22°C to 28°C |
| Lizard | 🦎 | 28°C to 36°C |
| Desert Crab | 🦀 | > 36°C |

### 6.3 Animations (CSS keyframes)
Each pet has an assigned animation style:

| Animation | Effect | Used By |
|---|---|---|
| `bounce` | Bounces up and down | Fox, Lizard, Frog |
| `float` | Gentle float + tilt | Cat, Owl |
| `shake` | Side-to-side shake | Crab |
| `shiver` | Fast horizontal shiver | Polar Bear |
| `waddle` | Rocking rotation | Penguin |
| `bob` | Bounce + horizontal squish | Duck |

### 6.4 Pet Behavior
- **Mood text** cycles every 5 seconds through 4 species-specific phrases
- **Action text** changes randomly from a pool based on current state:
  - `idle` — default actions
  - `shiver` — triggered when temp ≤ 0°C
  - `sweat` — triggered when temp ≥ 32°C
  - `pet` — triggered on click/tap
- **Pet interaction:** clicking the pet box plays a sound (if SFX on), briefly scales the emoji up (`tama-pet` animation), flashes the panel, and shows a pet-specific response phrase

---

## 7. Particle Effects

Pixel particles rendered on a `<canvas>` overlay on the screen (`z-index: 9`). Canvas is resized to match the screen element on load and resize.

| Weather Condition | Effect |
|---|---|
| Rain / Drizzle (09, 10) | Blue falling rain pixels (1×4px), slight diagonal drift |
| Thunderstorm (11) | Same rain + random full-screen lightning flash (opacity overlay, ~0.4% chance per frame) |
| Snow (13) | White square snowflakes (3×3px), sinusoidal horizontal drift, slow fall |
| All others | No particles |

Particle count capped at 100. Old particles removed when they exit the bottom of the screen.

---

## 8. Sound Effects

Toggle via the **🔇 SFX** button in the brand bar. Uses the Web Audio API (`AudioContext`).

| Condition | Sound |
|---|---|
| Rain / Drizzle | Band-pass filtered white noise (centre ~1200Hz), low volume loop |
| Thunderstorm | Sawtooth oscillator boom (~55Hz, 1.4s decay), repeats every 3.5–7s randomly |
| Mist / temp < 0°C | Low-pass filtered white noise (cutoff ~280Hz), wind-like loop |
| Successful search | 4-note ascending chime (C5→E5→G5→C6, sine wave) |
| Pet tap | 3-note ascending pip (A5→C6→A5, sine wave) |
| Other conditions | No ambient sound |

Sound is off by default. Enabling sound resumes the AudioContext (required for browser autoplay policy compliance) and starts the ambient loop for current conditions.

---

## 9. Unit System

- Toggle between **°F** and **°C** via buttons in the search bar
- All temperature, wind speed, and visibility values re-render on toggle without a new API call
- Conversion functions:
  - `toF(kelvin)` → Fahrenheit
  - `toC(kelvin)` → Celsius
  - Wind: m/s → mph (×2.237) or km/h (×3.6)
  - Visibility: metres → miles (÷1609.34) or km (÷1000)
  - Wind direction: degrees → 16-point cardinal (N, NNE, NE… NNW)

---

## 10. API Integration

### 10.1 Endpoints Used
| Endpoint | Purpose |
|---|---|
| `geo/1.0/direct?q=...&limit=6` | City autocomplete suggestions |
| `geo/1.0/direct?q=...&limit=1` | Resolve city name → lat/lon for search |
| `data/2.5/weather?lat=&lon=` | Current conditions |
| `data/2.5/forecast?lat=&lon=` | 5-day / 3-hour forecast |

### 10.2 All weather data is fetched in Kelvin (no `units` param) and converted client-side.

### 10.3 Error Handling
- Geocoding failure → displays pixel error bar with message
- "Not found" → prompts user to try adding a country
- Network/HTTP errors → caught and displayed in error bar
- City autocomplete failures → silently returns empty (no visible error)

---

## 11. Device Shell UI

Purely cosmetic — no functional interaction.

| Element | Description |
|---|---|
| Top stripe | Purple-red gradient across top of shell |
| Power LED | Green pulsing dot, top-right of brand bar |
| D-pad | 4-directional pixel pad, bottom-left of controls |
| A / B buttons | Red rounded buttons, bottom-right |
| SELECT / START | Flat oval buttons, bottom-centre |
| Speaker | 3×3 grid of dot holes, bottom-far-right |

---

## 12. File Structure

Single HTML file — no build step, no dependencies to install.

```
weather-dashboard.html
├── <style>          — All CSS (inline)
│   ├── CSS variables (palette)
│   ├── Device shell styles
│   ├── Screen & panel styles
│   ├── Autocomplete dropdown styles
│   ├── Particle canvas
│   └── Animation keyframes (tama pets)
├── <body>
│   ├── #stars       — Fixed star overlay
│   ├── #device      — Outer shell
│   │   ├── #brand-bar
│   │   ├── #screen-bezel
│   │   │   └── #screen
│   │   │       ├── #pcv (particle canvas)
│   │   │       ├── #loading overlay
│   │   │       ├── #clock-strip
│   │   │       ├── #srow (search inputs)
│   │   │       ├── #wmain (icon + weather text)
│   │   │       ├── #tbox (tamagotchi pet)
│   │   │       ├── #sgrid (stat boxes)
│   │   │       ├── bar rows
│   │   │       ├── #sunrow
│   │   │       ├── #fstrip (forecast)
│   │   │       └── #ticker
│   │   └── #ctrl (D-pad, buttons, speaker)
└── <script>         — All JS (inline)
    ├── Stars generation
    ├── Temperature palette system
    ├── Day/night detection
    ├── Pixel art icon drawing (canvas)
    ├── Tamagotchi pet system
    ├── Particle effects system
    ├── Web Audio sound system
    ├── Clock ticker
    ├── Unit conversion helpers
    ├── US state code map + normalizer
    ├── Country name → ISO code map (150+ entries)
    ├── Country autocomplete (dropdown)
    ├── City autocomplete (live API, debounced)
    └── fetchWeather() + render()
```

---

## 13. Known Constraints & Notes

- **Browser only** — the OWM API calls are made from the browser (CORS is allowed by OWM for client-side use)
- **No persistence** — no localStorage, no cookies; state resets on page reload
- **Single file** — all CSS, JS, and HTML are inline; no external JS files or frameworks
- **API key** — hardcoded in the script; replace `API` constant to change it
- **Sound requires user gesture** — Web Audio API requires the user to interact with the page before audio can play; the SFX toggle handles this
- **City autocomplete debounce** — 300ms delay after typing stops before API call fires
- **Forecast grouping** — the OWM free-tier forecast returns 3-hour slots; these are grouped by calendar day client-side, with the middle slot used for the representative icon/description
