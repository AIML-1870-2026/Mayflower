# spec.md — Blackjack AI Agent

## Assignment Brief

The temp/ folder contains a working example of a static webpage
that interacts with an LLM via an uploaded .env file. Use it as
a reference for:
  - How to parse a .env file for the API key (in-memory only)
  - The fetch() call structure for the LLM API
  - Error handling patterns for failed API requests

Do NOT include the temp/ folder in the final build or deployment.

Build a static Blackjack AI agent page that:
  - Accepts an API key via .env file upload
  - Implements a full Blackjack game (deck, dealing, scoring)
  - Calls the LLM to recommend hit/stand given the current hand
  - Returns a structured JSON response so the action can be
    extracted reliably (avoids keyword-search ambiguity)
  - Logs key interactions to the console for debugging
  - Tracks the player's balance across hands

---

## Visual Design

- **Theme:** Deep-ocean bioluminescent aesthetic — dark abyss fading to surface blue
- **Background:** Radial gradient (#020c16 → #0d6e9e) with animated caustic light ripples
- **Animated scene:** Rising bubbles, swaying kelp forest, shark gliding across, jellyfish at edges, fish parade on wins
- **Currency:** Pearls (🦪), rendered as iridescent chip tokens with 6 denominations (1/5/10/25/100/500)
- **Cards:** Sea-glass blue-green faces; ocean creature suits — 🐬 Dolphin, 🪸 Coral, 🐠 Clownfish, 🦑 Squid
- **Typography:** Cormorant Garamond (display), Nunito (UI), Cinzel (labels)
- **Layout:** Fixed game area (left) + fixed AI Agent panel (right, 300px wide)

---

## Blackjack Rules

| Rule | Implementation |
|------|----------------|
| Decks | 6-deck shoe (312 cards), reshuffled when < 52 remain |
| Dealer | Stands on soft 17+ |
| Blackjack | Pays 3:2 — `Math.floor(bet × 1.5)` |
| Push | Simultaneous blackjack → bet returned |
| Insurance | Dealer Ace → offer half-bet; pays 2:1 if dealer has BJ |
| Double Down | First 2 cards only; one additional card |
| Split | Equal ranks → two independent hands |
| Split Aces | Exactly 1 card each, then auto-stand |
| Card counting | Hi-Lo: 2-6=+1, 7-9=0, 10-A=-1 (running count displayed in HUD) |

---

## AI Agent Panel

The fixed right-side panel provides all AI agent functionality:

- **API Key:** Upload a `.env` file containing `OPENAI_API_KEY` or paste the key directly. Stored in-memory only — never persisted.
- **Model Selection:** Clickable cards for GPT-4o mini (default), GPT-4o, GPT-4 Turbo
- **Ask AI Agent button:** Sends current game state to the LLM; enabled only during active play with a key loaded
- **AI Analysis:** Displays the model's reasoning for its recommendation
- **Recommendation badge:** HIT or STAND extracted from structured JSON response
- **Execute Recommendation:** Manual button to apply the AI's suggested action

### Structured JSON Approach

The system prompt instructs the model to return **only** a JSON object:
```
{"action": "hit", "analysis": "one sentence explanation"}
```
The action is extracted from `parsed.action` — not from keyword search on prose — which eliminates ambiguity (e.g. "you should NOT hit, you should stand" contains both words).

### Console Logging

All key interactions are logged with `[AI Agent]` prefix:
- Game state string sent to the model
- Selected model and explanation mode
- System prompt and user message
- Raw API response object
- Fence-stripped text and parsed JSON
- Extracted action
- Execution calls

---

## Stretch Challenges Implemented

### 1. Strategy Visualization
Displays the full basic strategy decision matrix alongside the AI's recommendation. The current dealer upcard column is highlighted in gold. Supports hard totals, soft totals, and pairs.

### 2. Performance Analytics
Tracks AI-assisted hands separately from overall play:
- AI win rate with progress bar
- Comparison vs overall win rate
- Net P/L on AI-executed hands
- 20-hand dot history (glowing dots = AI-executed hands)

### 3. Explainability Controls
Three explanation modes change the system prompt:
- **Brief** — one sentence
- **Detailed** — 2–3 sentences with strategy context
- **Statistical** — probability-based reasoning

---

## Features Summary

| Category | Feature |
|----------|---------|
| **Core Rules** | 6-deck shoe, dealer stands soft 17, BJ 3:2, push, insurance, double down, split, split aces |
| **AI Agent** | .env key upload, OpenAI API, structured JSON, Execute Recommendation button, model selection |
| **UI** | Card deal + flip animations, win particles, fish parade, iridescent pearl chips |
| **Stretch 1** | Basic strategy matrix visualization with current position highlighted |
| **Stretch 2** | Performance analytics — AI win rate, P/L, dot history |
| **Stretch 3** | Explainability controls — Brief / Detailed / Statistical |
| **Accessibility** | Keyboard shortcuts: H (hit), S (stand), D (deal/new), X (double), C (clear) |
| **Responsive** | Mobile-optimized: agent panel becomes bottom drawer on small screens |
