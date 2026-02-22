# Readable — Ocean Contrast Explorer
## Project Specification

**Version:** 1.0  
**Status:** Complete  
**Format:** Single-file HTML (no dependencies, no build step)  
**Theme:** Ocean Glassmorphism  

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Design Direction](#2-design-direction)
3. [Functional Requirements](#3-functional-requirements)
4. [WCAG Math & Formulas](#4-wcag-math--formulas)
5. [Synchronization Behavior](#5-synchronization-behavior)
6. [Stretch Features](#6-stretch-features)
7. [Preset Color Schemes](#7-preset-color-schemes)
8. [Visual & UX Behavior](#8-visual--ux-behavior)
9. [Accessibility Requirements](#9-accessibility-requirements)
10. [Responsive Layout](#10-responsive-layout)
11. [Deployment](#11-deployment)
12. [Conversation & Decision Log](#12-conversation--decision-log)

---

## 1. Project Overview

**Readable** is a browser-based accessibility tool that lets users explore the relationship between color contrast and readability. Users adjust background color, text color, and font size using synchronized sliders and number inputs, while the tool calculates luminance and WCAG contrast ratios in real time.

The tool serves designers, developers, and accessibility practitioners who want to verify their color combinations against WCAG 2.1 standards before publishing.

**Key goals:**
- Make contrast ratios intuitive and visually understandable
- Provide live feedback without any page reloads or button clicks
- Demonstrate both passing and failing WCAG combinations
- Simulate how different vision types perceive the same colors

---

## 2. Design Direction

### Aesthetic
**Ocean Glassmorphism** — frosted glass panels floating over an animated deep-sea background. The ocean itself reacts to the contrast score, shifting from dark abyssal murk (poor contrast) to bright sunlit shallows (excellent contrast).

### Typography
- **Display / headings:** Cormorant Garamond (elegant, italic, serif)
- **UI / body:** Josefin Sans (clean, geometric, sans-serif)
- Both loaded from Google Fonts

### Color Palette
| Role | Value |
|------|-------|
| Accent (aquamarine) | `#7fffd4` |
| Fail (pink-red) | `#ff6b9d` |
| Pass (aquamarine) | `#7fffd4` |
| Warning (yellow) | `#ffe066` |
| Red channel label | `#ff8fa3` |
| Green channel label | `#7fffd4` |
| Blue channel label | `#74b9ff` |
| Glass panel bg | `rgba(255,255,255,0.10)` |
| Glass panel border | `rgba(255,255,255,0.22)` |

### Animated Elements
- **Ocean background** — layered radial gradients shift color based on contrast score. Low ratio = dark navy abyss. High ratio = bright teal surface. Implemented via CSS custom properties updated by JavaScript on every render.
- **Animated wave blobs** — two `::before`/`::after` pseudo-elements animate in opposite directions using `@keyframes waveShift`.
- **Floating bubbles** — 12 `<div>` elements created dynamically by JavaScript with randomized size, position, duration, and delay. Opacity increases with contrast score.

### Mood Meter
A header pill showing an emoji + text verdict that updates with every contrast change:

| Ratio Range | Emoji | Verdict |
|-------------|-------|---------|
| 1.0 – 1.5 | 🌑 | Invisible |
| 1.5 – 2.5 | 🫧 | Very Poor |
| 2.5 – 3.0 | 🐟 | Poor |
| 3.0 – 4.5 | 🐠 | Okay (AA Large) |
| 4.5 – 7.0 | 🐬 | Good (AA) |
| 7.0 – 15.0 | 🌊 | Excellent (AAA) |
| 15.0 – 21+ | ☀️ | Perfect! |

### Depth Gauge
A progress bar below the contrast ratio number that fills left-to-right from 0% (ratio = 1) to 100% (ratio = 21), with ocean zone labels:

| Ratio | Zone Label |
|-------|-----------|
| < 3 | Abyssal Zone 🌑 |
| 3 – 4.5 | Midnight Zone 🌒 |
| 4.5 – 7 | Twilight Zone 🌊 |
| 7 – 15 | Sunlight Zone ✨ |
| 15+ | Surface Bright ☀️ |

---

## 3. Functional Requirements

### 3.1 Background Color Controls

- Three RGB channels: **R**, **G**, **B**, each 0–255
- Each channel has:
  - A range `<input type="range">` slider (min=0, max=255)
  - An integer `<input type="number">` field (min=0, max=255)
- Both controls are **synchronized in real time** (see Section 5)
- A color swatch (small square) updates to show the current background color
- A **hex code button** displays the current hex value (e.g. `#ffffff`), clickable to copy to clipboard
- A **luminance readout** shows the calculated relative luminance to 3 decimal places

### 3.2 Text Color Controls

Identical structure to background color controls (Section 3.1), independently controlling the text color used in the preview area. Includes its own swatch, hex button, and luminance readout.

### 3.3 Text Size Control

- A single range slider (min=8, max=72, in pixels)
- A synchronized integer number field
- A hint line below indicating whether the current size is "Normal text" or "Large text" and the applicable WCAG threshold:
  - `< 24px` → "Normal text — 4.5 : 1 required for AA"
  - `≥ 24px` → "Large text (≥ 24px) — 3.0 : 1 required for AA"

### 3.4 Text Display Area (Live Preview)

A rectangular region with:
- **Background color** set to the current background RGB values
- Three text blocks displayed using the current **text color** and **font size**:
  - **Heading** — Cormorant Garamond italic, scaled at `fontSize × 1.8` (capped at 66px)
  - **Body paragraph** — Josefin Sans, at the exact slider font size
  - **Small text** — Josefin Sans, at `fontSize − 4` (minimum 8px)
- All three blocks update instantly on any slider or input change

### 3.5 Contrast Ratio Display

- Shows the calculated contrast ratio as **`XXX : 1`** (space before colon, space before 1)
- Displayed prominently using Cormorant Garamond at large size
- Color-coded by quality:
  - Ratio ≥ 7.0 → `#fff176` (yellow — AAA)
  - Ratio ≥ 4.5 → `#7fffd4` (aquamarine — AA)
  - Ratio ≥ 3.0 → `#ffe066` (amber — AA Large only)
  - Ratio < 3.0 → `#ff6b9d` (pink-red — fail)
- Recalculates automatically whenever background or text color changes

### 3.6 Luminance Displays

Two dedicated cards show:
- **Background Luminance** — relative luminance value to 3 decimal places
- **Text Luminance** — relative luminance value to 3 decimal places
- **Formula Breakdown card** — shows the literal formula with current values substituted in:
  - e.g. `(0.93 + 0.05) / (0.02 + 0.05)` → `= 12.86 : 1`
  - Updates live so users can understand how the ratio is derived

---

## 4. WCAG Math & Formulas

### Relative Luminance

Per WCAG 2.1, each 8-bit RGB channel is first linearized:

```
if sRGB ≤ 0.04045:
    linear = sRGB / 12.92
else:
    linear = ((sRGB + 0.055) / 1.055) ^ 2.4
```

Where `sRGB = channel / 255`.

Relative luminance is then:

```
L = 0.2126 × R_linear + 0.7152 × G_linear + 0.0722 × B_linear
```

Result is in range [0, 1] where 0 = absolute black, 1 = absolute white.

### Contrast Ratio

```
Contrast Ratio = (L1 + 0.05) / (L2 + 0.05)
```

Where **L1** is the **lighter** (higher) luminance and **L2** is the **darker** (lower) luminance. Result is always ≥ 1.

### WCAG Pass/Fail Thresholds

| Level | Text Type | Minimum Ratio |
|-------|-----------|---------------|
| AA | Normal text (< 24px or < 18.67px bold) | 4.5 : 1 |
| AA | Large text (≥ 24px or ≥ 18.67px bold) | 3.0 : 1 |
| AAA | Normal text | 7.0 : 1 |
| AAA | Large text | 4.5 : 1 |

> **Implementation note:** This tool uses 24px as the large text threshold for simplicity, which is consistent with standard accessibility tooling practice.

---

## 5. Synchronization Behavior

All slider–number input pairs must be **bidirectionally synchronized in real time**:

### Slider → Number Field
- On every `input` event (fires while dragging)
- Number field updates immediately to the slider's integer value
- Preview and all calculations update instantly

### Number Field → Slider
- On every `input` event (fires on every keystroke)
- Slider updates immediately to match
- Value is clamped to valid range (0–255 for color, 8–72 for size)
- Preview and all calculations update instantly

### On Blur (Number Field)
- Final clamp applied if user typed an out-of-range value
- Both controls set to the clamped value
- Full re-render triggered

### Preset Load
- All six color number fields and both sliders update immediately
- Size slider and field also update
- Full re-render triggered synchronously

---

## 6. Stretch Features

### Option A — Vision Type Simulation

**Implementation:** Native HTML radio buttons styled as custom pill buttons. Each option uses a hidden `<input type="radio">` with a visible custom dot indicator.

**Vision types:**
| Radio Option | SVG Filter Applied | Description |
|---|---|---|
| Normal | None | Standard color vision |
| Protanopia | `feColorMatrix` | Red-blind (missing L-cones) |
| Deuteranopia | `feColorMatrix` | Green-blind (missing M-cones) |
| Tritanopia | `feColorMatrix` | Blue-blind (missing S-cones) |
| Monochromacy | `feColorMatrix saturate=0` | Complete color blindness (greyscale) |

The SVG filters are defined inline in the HTML using `<defs>` and applied to the preview text wrap `<div>` via `element.style.filter`.

**Color matrix values used:**

*Protanopia:*
```
0.567 0.433 0     0 0
0.558 0.442 0     0 0
0     0.242 0.758 0 0
0     0     0     1 0
```

*Deuteranopia:*
```
0.625 0.375 0   0 0
0.7   0.3   0   0 0
0     0.3   0.7 0 0
0     0     0   1 0
```

*Tritanopia:*
```
0.95  0.05  0     0 0
0     0.433 0.567 0 0
0     0.475 0.525 0 0
0     0     0     1 0
```

**Spec requirement — disable color editing under filter:**  
When any non-Normal vision type is selected, all RGB sliders and number inputs are `disabled` (HTML attribute). This prevents users from making color decisions while their perception of the preview is being altered. An amber warning banner appears explaining the lock. Controls re-enable immediately when Normal is reselected.

### Option B — WCAG Compliance Indicator

Four compliance cards displayed in a 2×2 grid:

| Card | Level | Text Type | Threshold |
|------|-------|-----------|-----------|
| 1 | AA | Normal Text | ≥ 4.5 : 1 |
| 2 | AA | Large Text | ≥ 3.0 : 1 |
| 3 | AAA | Normal Text | ≥ 7.0 : 1 |
| 4 | AAA | Large Text | ≥ 4.5 : 1 |

Each card shows:
- Level badge (AA / AAA)
- Text type label
- Threshold requirement
- **Result pill**: green background + "PASS" text, or red background + "FAIL" text

**Accessibility note:** Pass/fail status is communicated through both color AND text label — never color alone — ensuring the indicator is usable by people with color vision deficiencies.

The panel has `aria-live="polite"` so screen reader users hear updates when values change.

### Option C — Preset Color Schemes

Presets organized into three labeled groups so users immediately understand the purpose of each:

**✅ High Contrast**
| Name | Background | Text | Ratio |
|------|-----------|------|-------|
| Open Ocean | `rgb(255,255,255)` | `rgb(0,0,0)` | 21.00 : 1 |
| Deep Sea | `rgb(12,28,64)` | `rgb(127,255,212)` | ~11 : 1 |
| Bioluminescence | `rgb(2,62,80)` | `rgb(250,240,190)` | ~9 : 1 |

**🌐 Common Web**
| Name | Background | Text | Notes |
|------|-----------|------|-------|
| Coral Reef | `rgb(0,105,148)` | `rgb(255,255,255)` | Ocean blue + white |
| Sea Foam | `rgb(224,247,250)` | `rgb(0,60,80)` | Light blue + dark teal |
| Tide Pool | `rgb(230,245,255)` | `rgb(20,120,180)` | Light blue on blue, 24px |

**⚠️ Fails WCAG**
| Name | Background | Text | Why It Fails |
|------|-----------|------|-------------|
| Sunset Tide | `rgb(255,140,50)` | `rgb(255,250,240)` | Orange + near-white, low ratio |
| Abyss | `rgb(0,20,40)` | `rgb(20,80,120)` | Near-black + dark blue, very low contrast |
| Shallow Murk | `rgb(180,210,230)` | `rgb(150,185,210)` | Similar grey-blue tones, near-zero contrast |

Each preset button displays:
- Two small color swatches (background + text)
- Preset name
- Pre-calculated contrast ratio (color-coded by quality)

Clicking a preset instantly updates all six RGB sliders, all six number fields, the size slider and field, and triggers a full re-render.

---

## 7. Preset Color Schemes

See Section 6 Option C for full preset table. All ratios are calculated using the same WCAG formula as the live calculator.

---

## 8. Visual & UX Behavior

### Hex Copy
- Both the background and text hex code buttons are clickable
- On click: copies hex string to clipboard using `navigator.clipboard.writeText()`
- A toast notification appears at the bottom of the screen confirming the copy (e.g. `#0c1c40 copied!`)
- Toast disappears after 2 seconds
- When color editing is disabled (vision filter active), hex buttons are non-interactive

### Real-time Updates
Every single user interaction — slider drag, number field keystroke, preset click, vision radio change — triggers a full synchronous re-render of all outputs with no debounce or delay.

### Slider Thumb Styling
- White circular thumb, 15px diameter
- Glowing aquamarine box-shadow on hover
- Scales up 1.25× on hover via CSS transition
- Opacity reduces to 30% when disabled

### Panel Layout
Three-column grid:
- **Left (300px):** Color controls + vision simulation + text size
- **Center (fluid):** Contrast ratio hero + luminance cards + live preview + WCAG panel
- **Right (255px):** Preset buttons

---

## 9. Accessibility Requirements

- All sliders have descriptive `aria-label` attributes
- All number inputs have descriptive `aria-label` attributes
- Color swatch elements are `aria-hidden="true"` (decorative)
- WCAG compliance panel has `aria-live="polite"` for screen reader announcements
- Contrast ratio display has `aria-live="polite"`
- Mood meter has `aria-live="polite"`
- Vision radio group has `role="radiogroup"` and `aria-label`
- WCAG cards update their `aria-label` to include pass/fail status
- Pass/fail uses both color AND text label (never color alone)
- Vision simulation warning uses `role="alert"` for immediate screen reader announcement
- All interactive elements have `:focus-visible` outline in aquamarine
- Preset buttons have descriptive `aria-label` attributes

---

## 10. Responsive Layout

| Breakpoint | Behavior |
|------------|----------|
| > 1040px | Full three-column grid |
| 660px – 1040px | Two-column grid; presets expand to full-width row, displayed as wrapping grid |
| < 660px | Single column; luminance cards stack vertically; logo shrinks; ratio number shrinks |

---

## 11. Deployment

The entire application is a **single HTML file** with no external JavaScript dependencies, no build step, and no server required.

### Local Development
Open `readable-final.html` directly in any modern browser.

```bash
# Optional: use a local server for clipboard API support
npx serve .
# or
python -m http.server 8080
```

### GitHub Pages
1. Create a new GitHub repository
2. Rename the file to `index.html`
3. Push to the repository:
   ```bash
   git init
   git add index.html
   git commit -m "Initial commit: Readable Ocean Contrast Explorer"
   git branch -M main
   git remote add origin https://github.com/YOUR_USERNAME/readable.git
   git push -u origin main
   ```
4. Go to **Settings → Pages → Source: Deploy from branch → main / (root)**
5. Site is live at `https://YOUR_USERNAME.github.io/readable/`

### Requirements
- Any modern browser (Chrome, Firefox, Safari, Edge)
- No Internet connection required after initial font load (Google Fonts)
- Clipboard API (`navigator.clipboard`) requires HTTPS or localhost

---

## 12. Conversation & Decision Log

This section documents how the project evolved through conversation, capturing every design decision and requirement clarification.

---

### Turn 1 — Initial Brief
**User asked for:** A complete deployable webpage called Readable with:
- Background RGB sliders (0–255) synced with number inputs
- Text RGB sliders (0–255) synced with number inputs
- Text size slider synced with number input
- Live-updating sample text preview
- Luminance values for both colors
- WCAG contrast ratio: `(L1 + 0.05) / (L2 + 0.05)` displayed as `X.XX:1`
- Real-time WCAG compliance indicators (4.5:1 normal, 3:1 large)
- At least two stretch features from: A (vision simulation), B (WCAG pass/fail panel), C (preset color schemes)
- Complete HTML/CSS/JS files, responsive design, GitHub Pages instructions

**Delivered:** Three-file version (`index.html`, `style.css`, `script.js`) with dark editorial instrument-panel aesthetic (DM Mono + Playfair Display). All three stretch features (A, B, C) implemented. Vision used toggle buttons. Presets had ocean-themed names.

---

### Turn 2 — Example Reference Provided
**User shared:** A screenshot of an existing contrast explorer tool with a different layout — background/text color controls in separate panels, text size in center, vision type as radio buttons, luminance and contrast ratio displayed as cards below.

**User asked:** "We need to model off of this but also make it unique so give some fun ideas."

**Decision:** Proposed four aesthetic directions and two feature ideas to the user via an interactive question widget.

---

### Turn 3 — Aesthetic Vote
**User selected:**
- 🌊 Glassmorphism — frosted glass panels, blurred depth
- Live animated background that reacts to contrast score

**Decision:** Committed to ocean glassmorphism theme. Animated background shifts from dark abyss (bad contrast) to bright sunlit surface (good contrast).

---

### Turn 4 — Ocean Theme Confirmed
**User requested:** "Can we do something ocean themed?"

**Decision:** Rebuilt entirely as ocean-themed glassmorphism. Introduced:
- Cormorant Garamond + Josefin Sans typography
- Aquamarine (`#7fffd4`) as signature accent color
- Floating bubble particles
- Mood meter emoji (🌑 → 🫧 → 🐟 → 🐠 → 🐬 → 🌊 → ☀️)
- Depth gauge bar with ocean zone labels
- Ocean-named presets (Open Ocean, Deep Sea, Coral Reef, Sea Foam, Sunset Tide, Abyss, Tide Pool, Bioluminescence)
- Single-file HTML delivery

---

### Turn 5 — Download Error (First)
**User reported:** "It is saying failed to download and open file."

**Diagnosis:** Multi-file folder delivery can fail in some browser environments. Rebuilt as single self-contained `.html` file with all CSS and JS embedded.

---

### Turn 6 — Download Error (Second)
**User reported:** "It is saying failed to download and open file again."

**Diagnosis:** Persistent download failure unrelated to file content. File verified intact (51,919 bytes). Provided three workarounds:
1. Right-click → Save link as
2. Copy/paste code manually into local file
3. Paste into GitHub Gist as `index.html`

Also delivered the file again as a cleaner single-file build.

---

### Turn 7 — WCAG Compliance Question
**User asked:** "Why are some of them not passing some of the WCAG compliance?"

**Clarification provided:** This is correct and expected behavior. Explained the four different WCAG thresholds:
- AA Normal Text: 4.5:1 (most strict common standard)
- AA Large Text: 3.0:1 (lower bar because large text is more readable)
- AAA Normal Text: 7.0:1 (highest/hardest level)
- AAA Large Text: 4.5:1

Noted that failing presets (Abyss ⚠️, Sunset Tide ⚠️) are intentionally low-contrast to demonstrate real-world bad examples.

---

### Turn 8 — Full Spec Compliance Audit
**User provided:** Complete detailed specification with precise requirements including:
- Explicit format `"XXX: 1"` for contrast ratio display
- Bidirectional sync explicitly required both directions
- Vision simulation must use radio buttons (not toggle buttons)
- **Option A design consideration:** color adjustments should only be allowed when "Normal" vision is selected
- WCAG compliance must use green/red color coding PLUS text labels
- Preset groups should include: high contrast, low contrast, common website schemes, problematic WCAG-failing combinations

**Audit findings:**
1. Vision simulation used toggle `<button>` elements — spec requires `<input type="radio">`
2. Contrast ratio displayed as `"21.00:1"` — spec requires `"21.00 : 1"` (spaces)
3. Color sliders were never disabled under vision filters — spec requires disabling
4. No warning UI when color editing is locked
5. Presets not organized into labeled groups by type

**Changes made in final version:**
- Vision simulation rebuilt with native radio inputs, custom-styled
- Contrast format changed to `"XXX : 1"` throughout
- `setColorControlsDisabled(true/false)` function added — disables all 6 RGB sliders + 6 number inputs when non-Normal vision active
- Amber warning banner added with `role="alert"`
- Formula breakdown card added to luminance strip
- Presets reorganized into three labeled groups: High Contrast, Common Web, Fails WCAG
- Shallow Murk preset added (near-zero contrast grey-blue combination)
- Full ARIA audit pass on all interactive elements

---

### Turn 9 — Spec Document Requested
**User asked:** "Create a spec md of our conversation."

**Delivered:** This document.

---

*End of specification.*
