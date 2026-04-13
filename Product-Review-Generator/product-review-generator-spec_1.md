# Product Review Generator — Project Spec

**Version:** 1.0  
**Deliverable:** `product-review-generator_4.html` — a single self-contained HTML file  
**Assignment context:** A dynamic single-page tool that uses OpenAI's API to generate structured, sentiment-aware product reviews based on user-configured inputs.

---

## Reference Implementation

The temp/ folder contains my complete LLM Switchboard
project (HTML, CSS, and JS files). This is NOT part of
the current project — do not include it in the final
build or deployment.

Use it as a reference for:
- How to parse a .env file for API keys (in-memory only)
- The fetch() call structure for OpenAI's chat completions API
- Error handling patterns for failed API requests
- How the code is organized across separate files
- The general approach to building a single-page LLM tool

Ignore these Switchboard features (not needed here):
- Anthropic integration (this project is OpenAI-only)
- The model selection dropdown / provider switching
- Structured output mode and JSON schema handling

This project uses unstructured (free-form) responses only.
Render the model's markdown output as formatted HTML.

---

## Overview

A browser-based product review generator powered by OpenAI's chat completions API. The user configures a product, adjusts sentiment per aspect, selects tone and length via sliders, and receives a fully formatted markdown review rendered as HTML. No backend, no build step — just open the HTML file or serve it from GitHub Pages.

---

## Core Requirements

### 1. OpenAI Only
- OpenAI models only — no Anthropic dropdown, no provider switching
- API key entered manually or loaded from a `.env` file (in-memory only, never persisted)
- Key is validated on submit: detects and rejects Anthropic keys (`sk-ant-...`) with a clear error message

### 2. Model Selection
Four models available as clickable cards:

| Model ID | Display | Notes |
|---|---|---|
| `gpt-4o` | GPT-4o | Flagship · best quality (default) |
| `gpt-4o-mini` | GPT-4o mini | Fast & affordable |
| `gpt-4-turbo` | GPT-4 Turbo | Extended context |
| `gpt-3.5-turbo` | GPT-3.5 Turbo | Lightweight & quick |

### 3. Unstructured Responses
- The model returns free-form text, not JSON
- No schema templates, no `response_format` parameter
- Response is passed through `marked.js` (pinned to v4) and rendered as formatted HTML

### 4. API Key Handling
- Manual entry via password field (`id="api-key"`)
- `.env` file upload: parses `OPENAI_API_KEY` via regex, loads value into the field in memory only
- Keys are never written to disk, localStorage, or any persistent storage

### 5. Deployment
- Deployed to the GitHub Organization: `https://aiml-1870-2026.github.io/Mayflower/Product-Review-Generator/product-review-generator_4.html`
- Single HTML file — no dependencies to install

---

## UI Components

### Product Details Form (Left Panel)
- **API Key field** — password input with `.env` upload button
- **Model cards** — 4 clickable cards, one selected at a time
- **Product Name** — text input
- **Category** — dropdown (Electronics, Audio, Home Appliances, Kitchen, Clothing, Books, Sports, Beauty, Toys, Software, Food, Other)
- **Key Features / Context** — optional textarea

### Sentiment by Aspect (Sliders)
Four independent range sliders, each ranging 1–5:

| Aspect | Default | Scale |
|---|---|---|
| Overall | 4 — Good | Negative → Excellent |
| Price | 3 — Mixed | Negative → Excellent |
| Features | 4 — Good | Negative → Excellent |
| Usability | 4 — Good | Negative → Excellent |

Labels update live as the slider moves. Each aspect's sentiment is passed individually into the prompt so the model reflects different sentiments per dimension (e.g. excellent features but poor value).

### Tone Slider
- Range: 1 (Casual) → 5 (Technical)
- Steps: Casual · Friendly · Balanced · Professional · Technical
- Default: Balanced (3)

### Length Slider
- Range: 1 → 4
- Steps: Brief (100–150w) · Standard (200–280w) · Detailed (330–420w) · Long (480–580w)
- Default: Standard (2)
- Word count target is passed directly into the prompt

### Output Panel (Right Panel)
- **Top bar** — shows product name, model, tone, word target, and timestamp after generation
- **Empty state** — shown before first generation
- **Spinner** — shown during API call
- **Star display** — derived from the Overall sentiment slider (1–5 stars rendered as ★/☆)
- **Review content** — markdown rendered as HTML via `marked.parse()`
- **Copy button** — copies raw markdown to clipboard

---

## Prompt Structure

The prompt sent to the model incorporates all slider values:

```
Write a product review for the following:

Product: [name]
Category: [category]
Key details: [features]

Sentiment by aspect:
- Overall: Good (4/5)
- Price: Mixed (3/5)
- Features: Good (4/5)
- Usability: Good (4/5)

Writing tone: Balanced
Target length: 200–280 words

Use markdown formatting. Structure the review with:
- An H2 headline that reflects the overall sentiment
- An opening paragraph
- A "Pros" section (bulleted list)
- A "Cons" section (bulleted list)
- A "Verdict" paragraph

The review's sentiment for each aspect should closely match the specified ratings above.
Do not mention that this is AI-generated.
```

System prompt: `"You are an expert product reviewer who writes compelling, well-structured product reviews in markdown. Be specific, authentic-sounding, and match the requested tone precisely."`

---

## API Implementation

- **Endpoint:** `https://api.openai.com/v1/chat/completions`
- **Auth:** `Authorization: Bearer <key>` header
- **Parameters:** `model`, `max_tokens: 1000`, `messages` (system + user)
- **Error handling:** Single try/catch — checks `res.ok`, extracts error message from `res.json().catch(()=>({}))`, throws typed errors for 401, 429, 404
- **Key validation:** Detects `sk-ant-` prefix and surfaces a clear message before the request is made

---

## Diagnostics

A **"Test API Connection"** button is included below the Generate button. It makes a lightweight GET request to `https://api.openai.com/v1/models` with the provided key and displays:
- HTTP status + model count on success
- The API error message on 4xx/5xx
- Network error details if the request is blocked before reaching OpenAI

---

## Error States Handled

| Condition | Message shown |
|---|---|
| Empty API key | "Please enter your OpenAI API key." |
| Anthropic key detected | "That looks like an Anthropic key (sk-ant-...)..." |
| Empty product name | "Please enter a product name." |
| HTTP 401 | "Invalid API key. Double-check..." |
| HTTP 429 | "Rate limit or quota exceeded..." |
| HTTP 404 | "Model not available on your account..." |
| Network block | "Could not reach api.openai.com. Possible causes: billing, network firewall, browser extension." |

---

## Dependencies

| Library | Version | Purpose |
|---|---|---|
| `marked` | `@4` (pinned) | Markdown → HTML rendering |
| Google Fonts | — | Syne, Instrument Serif, JetBrains Mono |

> **Why marked is pinned to v4:** In marked v5+, `marked.parse()` returns a Promise instead of a string, which breaks synchronous `innerHTML` assignment. Pinning to v4 ensures synchronous rendering.

---

## Running the App

Open `product-review-generator_4.html` in any modern browser, or visit the live GitHub Pages URL.

If testing locally and you encounter CORS issues, serve via a local server:

```bash
# Python
python3 -m http.server 8000

# Node
npx serve .
```

Then open `http://localhost:8000/product-review-generator_4.html`.

### .env file format
```
OPENAI_API_KEY=sk-proj-...
```
Quoted values (`KEY="value"`) are also supported.

---

## Security Notes

- API key is held in JavaScript memory for the duration of the page session only
- Key is never sent anywhere except `https://api.openai.com`
- Do not commit `.env` files to version control
- Rotate any key that has been shared or exposed publicly
