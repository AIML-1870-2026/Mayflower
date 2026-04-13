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

## Key Design Constraints

- **OpenAI models only** — no Anthropic dropdown, no provider switching. You already know why from the Switchboard's CORS lesson.
- **Unstructured responses** — the model returns free-form text, not JSON. No schema templates needed.
- **Markdown rendering** — the model's response will include markdown formatting (bold, lists, headings, etc.). The app renders this as properly formatted HTML, not raw text.
- **API keys loaded from `.env`** — same in-memory-only pattern as the Switchboard. Nothing stored, nothing persisted.
- **Deployment** — deployed to the GitHub Organization for this class.

> **Why OpenAI only?** In the Switchboard project, Anthropic's API blocked direct browser requests due to CORS. OpenAI's API allows browser-to-API calls without a backend server. This is a deliberate architectural choice based on that lesson — not a limitation of the code.

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
The user selects a specific model within the OpenAI family using clickable cards:

| Model ID | Display | Notes |
|---|---|---|
| `gpt-4o` | GPT-4o | Flagship · best quality (default) |
| `gpt-4o-mini` | GPT-4o mini | Fast & affordable |
| `gpt-4-turbo` | GPT-4 Turbo | Extended context |
| `gpt-3.5-turbo` | GPT-3.5 Turbo | Lightweight & quick |

> Since this project uses a single API key (OpenAI only), a model family dropdown is not required — the model cards cover selection within the OpenAI family.

### 3. Unstructured Responses
- The model returns free-form text, not JSON
- No schema templates, no `response_format` parameter
- Response is passed through `marked.js` (pinned to v4) and rendered as formatted HTML

### 4. API Key Handling
- Manual entry via password field
- `.env` file upload: parses `OPENAI_API_KEY` via regex, loads value into the field in memory only
- Keys are never written to disk, localStorage, or any persistent storage

### 5. Deployment
- Deployed to the GitHub Organization: `https://aiml-1870-2026.github.io/Mayflower/Product-Review-Generator/product-review-generator_4.html`
- Single HTML file — no dependencies to install

---

## Stretch Features (All Implemented)

### Multiple Sentiment Layers
Four independent sentiment sliders — one per product aspect — allow users to express nuanced opinions rather than a single star rating. Each slider feeds its own sentiment value into the prompt individually.

| Aspect | Default | Scale |
|---|---|---|
| Overall | 4 — Good | Negative → Excellent |
| Price | 3 — Mixed | Negative → Excellent |
| Features | 4 — Good | Negative → Excellent |
| Usability | 4 — Good | Negative → Excellent |

Labels update live as the slider moves. The model is instructed to reflect each aspect's sentiment independently — e.g. a review can praise features while criticising price.

### Rich UI Components

**Tone Slider**
- Range: 1 (Casual) → 5 (Technical)
- Steps: Casual · Friendly · Balanced · Professional · Technical
- Default: Balanced (3)
- Replaces the tone dropdown from the original design

**Length Slider**
- Range: 1 → 4
- Steps and word targets:

| Position | Label | Word Count |
|---|---|---|
| 1 | Brief | 50–80 words (single paragraph, no bullet lists) |
| 2 | Standard | 180–220 words |
| 3 | Detailed | 400–460 words |
| 4 | Long | 700–800 words |

- Word count is enforced strictly in the prompt
- At Brief length, the model is instructed to skip headings and bullet lists entirely and write one tight paragraph

---

## UI Structure

```
Left Panel (form)
├── OpenAI API Key (password field + Load .env button)
├── Model selection (4 clickable cards)
├── Product Details
│   ├── Product Name (text input)
│   ├── Category (dropdown)
│   └── Key Features / Context (textarea, optional)
├── Sentiment by Aspect
│   ├── Overall slider (1–5)
│   ├── Price slider (1–5)
│   ├── Features slider (1–5)
│   └── Usability slider (1–5)
├── Tone slider (1–5)
├── Length slider (1–4)
└── Generate Review button

Right Panel (output)
├── Top bar (product · model · tone · word target · timestamp)
├── Error bar (shown on API or validation errors)
├── Empty state (before first generation)
├── Spinner (during API call)
└── Review output
    ├── Star display (derived from Overall slider)
    ├── Rendered markdown content
    └── Copy button (copies raw markdown)
```

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
Target length: Standard — STRICTLY 180–220 words total.

Use markdown formatting. Structure the review with:
- An H2 headline that reflects the overall sentiment
- An opening paragraph
- A "Pros" section (bulleted list)
- A "Cons" section (bulleted list)
- A "Verdict" paragraph

The review's sentiment for each aspect should closely match the specified ratings above.
Do not mention that this is AI-generated.
```

> At Brief length (50–80 words), the structure instruction changes to: *"A single short paragraph with an inline verdict. No headings, no bullet lists."*

**System prompt:** `"You are an expert product reviewer who writes compelling, well-structured product reviews in markdown. Be specific, authentic-sounding, and match the requested tone precisely."`

---

## API Implementation

- **Endpoint:** `https://api.openai.com/v1/chat/completions`
- **Auth:** `Authorization: Bearer <key>` header
- **Parameters:** `model`, `max_tokens: 1200`, `messages` (system + user)
- **Error handling:** Single try/catch — checks `res.ok`, extracts error message from `res.json().catch(()=>({}))`, throws typed errors for 401, 429, 404
- **Key validation:** Detects `sk-ant-` prefix and surfaces a clear message before the request is made

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
| Network block | Descriptive message covering billing, firewall, and extension causes |

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
