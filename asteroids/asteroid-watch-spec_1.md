# Asteroid Watch — Project Specification

## Overview

**Asteroid Watch** is a single-page interactive web dashboard that pulls live data from NASA's public APIs and presents near-Earth object (NEO) information across four distinct tabs. The project fulfills the NEO Dashboard assignment requirements, including a 3D interactive Earth visualization with asteroids positioned at their real miss distances and the Moon as a 1 lunar distance (LD) reference point.

---

## Assignment Requirements Checklist

| Requirement | Status | Implementation |
|---|---|---|
| Single-page web dashboard | ✅ | One self-contained `asteroid-watch.html` file |
| Three or more tabs | ✅ | Four tabs |
| Meaningful data/metrics on each tab | ✅ | See tab breakdown below |
| Data from real APIs (no hardcoded data) | ✅ | All data fetched live from NASA NeoWs API |
| Interactive 3D Earth with asteroids at miss distances | ✅ | globe.gl + Three.js visualization |
| Moon at 1 LD as reference point | ✅ | White sphere with pulse ring at exact 1 LD scaled position |

---

## Data Sources

### 1. NASA NeoWs Feed API
**Endpoint:** `https://api.nasa.gov/neo/rest/v1/feed`  
**Auth:** API key (`LjGIeQTqn8mTAuMbYqQJLqC4aj2zb5sSnfQ8e8zp`)  
**Used by:** 3D Close Approach Map, Time Machine, Rock of the Day  
**What it provides:** Every near-Earth asteroid making a close approach within a specified 7-day window. Returns estimated diameter, miss distance, relative velocity, hazard flag, and approach date.

### 2. NASA NeoWs Browse API
**Endpoint:** `https://api.nasa.gov/neo/rest/v1/neo/browse`  
**Auth:** Same API key  
**Used by:** Should We Panic? tab (hazardous NEO table)  
**What it provides:** Paginated list of all known NEOs with full orbital and physical data. Filtered client-side to `is_potentially_hazardous_asteroid = true`.

### 3. NASA NeoWs Lookup API
**Endpoint:** `https://api.nasa.gov/neo/rest/v1/neo/{spk_id}`  
**Auth:** Same API key  
**Used by:** Should We Panic? tab (Celebrity Asteroids section)  
**What it provides:** Full data record for a specific asteroid by its NASA SPK identifier. Used to fetch live data for Apophis, Bennu, 1950 DA, Didymos, and Icarus.

> **Note on JPL SBDB / Sentry APIs:** The assignment listed `ssd-api.jpl.nasa.gov` as a data source. JPL's own policy explicitly states *"You may not embed these APIs in your website (per NASA CORS policy)"* and enforces this with server-side CORS blocking. All JPL data references in the dashboard (Sentry tracking counts, Torino scale descriptions) use the NASA NeoWs API as a compliant alternative, with attribution to JPL Sentry noted in the UI.

---

## Tab Structure

### Tab 1 — 3D Close Approach Map 🌐
**The landing page and primary visual.**

- Interactive 3D Earth rendered via **globe.gl** library
- Every asteroid passing Earth this week is represented as a **colored sphere** positioned at its real proportional miss distance from Earth
- The **Moon appears as a white sphere** at exactly 1 LD with a pulse ring — the primary reference point
- A glowing **1 LD orbit ring** is injected into the Three.js scene to visually mark the Moon's orbit
- Color coding: 🔴 Red = hazardous, 🟠 Orange = inside Moon's orbit (<1 LD), 🟡 Amber = 1–3 LD, 🔵 Cyan = 3–10 LD, 🟢 Green = 10+ LD
- Animated distance lines arc from Earth's surface to each object
- Hazardous asteroids get pulsing red rings
- Click any sphere to see full details (name, date, distance, speed, size, hazard status)
- **Drag** to rotate · **Scroll** to zoom · **Click** for details
- Side panel shows a scrollable list of all this week's asteroids with distance, size, and speed
- Distance scale bar shows each asteroid's position relative to the Moon
- Stat cards at top: total asteroids, hazardous count, closest miss, fastest speed
- Countdown timer to next close approach

**Data source:** NASA NeoWs Feed API

---

### Tab 2 — Time Machine 🔭
**Search any week in history or the future.**

- Date range picker (max 7-day window per NASA API limits)
- Filters: danger level (all / scary only / harmless only), sort order (date / closest / biggest / fastest), minimum size in meters
- Results table with columns: Asteroid Name, Date, How Close?, How Fast?, How Big?, Should I Worry?
- Human-readable descriptions: speeds in mph, sizes with emoji labels (🚗 Car-sized, 🏟️ Stadium-sized, etc.), distances as multiples of the Moon's distance
- Color-coded proximity labels: 🚨 Inside Moon's orbit, 😬 Pretty close, 👀 In the neighborhood, ✅ Safely distant

**Data source:** NASA NeoWs Feed API

---

### Tab 3 — Should We Panic? ☠️
**Earth's official asteroid watch list.**

**Short answer: No.**

Two sections:

**Live Hazardous NEO Table** — pulls all potentially hazardous asteroids from NASA's browse endpoint. Sortable by size, closest approach, velocity, or name. Filterable by large (>500m) or close approachers (<5 LD). Shows next upcoming visit, size with plain-English description, record close approach distance, and top speed.

**Celebrity Asteroids** — live API lookup of 5 famous tracked asteroids by NASA SPK ID:
- **99942 Apophis** (SPK: 2099942) — passes closer than our satellites in 2029
- **101955 Bennu** (SPK: 2101955) — NASA landed a spacecraft on it
- **29075 (1950 DA)** (SPK: 2029075) — highest long-term impact probability
- **65803 Didymos** (SPK: 2065803) — NASA crashed DART into its moon in 2022
- **1566 Icarus** (SPK: 2001566) — one of the largest known PHAs

Stat cards: total hazardous asteroid count (live from API), Sentry-tracked object count, highest Torino score, monitoring horizon.

**Data source:** NASA NeoWs Browse API + NASA NeoWs Lookup API

---

### Tab 4 — Rock of the Day 🪨
**This week's most interesting asteroid, explained for everyone.**

- Auto-selects the most interesting asteroid from this week's feed (largest hazardous, or largest overall if none flagged)
- **Plain-English description** of speed (in mph with comparison to speed of sound), distance (as multiples of Moon's distance with km), and size
- **Size comparison chips** showing which real-world objects the asteroid is bigger than (city bus, football field, Eiffel Tower, cruise ship, etc.)
- **Stat cards:** smallest possible size, largest possible size, Moon distances away (with km), speed in km/s with mph equivalent and Mach number
- **"Should You Lose Sleep Over This?" meter** — animated worry gauge from 😴 to 😱 with a full paragraph explanation in plain English
- **Impact context cards:** "If This Hit a City" with consequence description, "Fun Fact" with Chelyabinsk context
- Link to full NASA/JPL database entry for the featured asteroid

**Data source:** NASA NeoWs Feed API

---

## Technical Stack

| Component | Technology |
|---|---|
| Framework | Vanilla HTML/CSS/JavaScript — no build step |
| 3D Globe | [globe.gl](https://globe.gl/) v2.27.2 |
| 3D Engine | [Three.js](https://threejs.org/) r128 (CDN) |
| Fonts | Google Fonts: Orbitron, Share Tech Mono, Exo 2 |
| Earth texture | NASA Blue Marble via Three.js examples CDN |
| Star background | vasturiano/three-globe night sky asset |

---

## Visual Design

- **Theme:** Dark space — deep navy/black backgrounds, glowing cyan accents
- **Typography:** Orbitron (headers, data values), Share Tech Mono (technical readouts), Exo 2 (body text)
- **Animated starfield** canvas background rendered on every page load
- **Color system:**
  - Accent: `#00c8ff` (cyan glow)
  - Danger: `#ff4444` (red)
  - Warning: `#ffaa00` (amber)
  - Safe: `#00ff88` (green)
- Cards have hover glow effects; hazardous content uses red glow
- Responsive design with mobile breakpoints

---

## Key Design Decisions

**Why scale asteroid distances instead of showing true scale?**  
At true 1:1 scale, even the Moon (384,400 km away) would be completely off-screen given Earth's visual size. The dashboard scales all distances proportionally within the visible frame — relative ordering is preserved (asteroids closer than the Moon appear inside its ring; farther ones appear outside), but absolute distances are compressed for usability.

**Why use NASA NeoWs instead of JPL SBDB?**  
JPL's SBDB and Sentry APIs explicitly block browser-side requests via CORS policy. NASA's NeoWs API serves the same underlying data (sourced from the same JPL orbital mechanics) with full CORS support and a free API key.

**Why plain English throughout?**  
The assignment called for the dashboard to be fun and accessible to people who don't understand asteroid science. All technical units (km/s → mph, LD → Moon distances, Torino scale) are explained in context.

---

## File Structure

```
asteroid-watch.html     — Complete self-contained dashboard (single file)
asteroid-watch-spec.md  — This specification document
```

---

## API Key

```
LjGIeQTqn8mTAuMbYqQJLqC4aj2zb5sSnfQ8e8zp
```

Obtained from [api.nasa.gov](https://api.nasa.gov). Free tier with rate limits of 1,000 requests/hour. The NASA-provided `DEMO_KEY` is not used due to its much lower limits (30/hour) which would cause errors during active development.
