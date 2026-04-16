# Science Experiment Generator — Spec

## Project Summary
A static webpage (HTML, CSS, and JavaScript) that enables a user to interact with an LLM for the purposes of generating grade-appropriate science experiments from a list of available materials. The user provides their OpenAI API key by uploading a `.env` file — the key is read in-memory only and never stored or transmitted beyond the API call.

The page allows the user to provide the following input:
- **Grade Level** — a dropdown menu allowing the user to select grade level (K-2, 3-5, 6-8, 9-12)
- **Available Supplies** — a text input field allowing the user to enter a list of supplies that are on hand or easily acquired

The completed Science Experiment Generator feels similar to the LLM Switchboard — a clean, self-contained static page that opens directly in the browser with no server required. The user uploads a `.env` file containing their OpenAI API key, selects a grade level, enters the supplies they have on hand, and submits the form. The model's response is rendered as formatted HTML on the page.

---

## Key Design Constraints

### Static page — HTML, CSS, and JavaScript only
No server, no Node.js, no npm. The page runs entirely in the browser. All dependencies (marked.js for markdown rendering, Google Fonts) are loaded from CDN.

### OpenAI models only — no Anthropic dropdown, no provider switching
This project uses OpenAI's `gpt-4o` exclusively. There is no Anthropic integration and no ability to switch providers. This is a deliberate architectural choice based on the CORS lesson from the Switchboard: Anthropic's API blocks direct browser requests, so a static page with no backend cannot use it. OpenAI's API allows browser-to-API calls without a backend server — that's why this project is OpenAI-only.

### Unstructured responses — free-form text, no JSON schema
The model returns free-form text. There is no `response_format: json_object`, no schema templates, and no structured output parsing. This project uses unstructured (free-form) responses only.

### Markdown rendering
The model's response includes markdown formatting (bold, lists, headings, etc.). The app renders this as properly formatted HTML using `marked.js`, not raw text.

### API keys loaded from .env — in-memory only
The user uploads a `.env` file; the key is read once via the browser's `FileReader` API and stored in a JavaScript variable for the session. It is never written to `localStorage`, `sessionStorage`, cookies, or any server. When the page is closed, the key is gone. A direct paste field is also provided as a convenience — same in-memory-only behavior.

### Deployment
Deployed to the GitHub organization for this class:
**https://aiml-1870-2026.github.io/Mayflower/Science-Experiment-Generator/**

---

## Reference Implementation

The `temp/` folder contains my complete LLM Switchboard project (HTML, CSS, and JS files). This is NOT part of the current project — do not include it in the final build or deployment.

Use it as a reference for:
- How to parse a `.env` file for API keys (in-memory only)
- The `fetch()` call structure for OpenAI's chat completions API
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

## Stretch Challenges Implemented

### 1. Quick Supply Selector
A grid of clickable chips organized into three groups (Kitchen, Craft & Office, Science & Outdoor) lets users select common household supplies with one click. Selected chips sync into the supplies textarea. Users can still type freely.

### 2. Difficulty Rating
The system prompt instructs the model to include a `## Difficulty` section with exactly one word: Easy, Medium, or Hard. After generation, a regex parses this value and displays it color-coded in the metrics row (green / amber / red).

### 3. Experiment History
A "Save to history" button stores each result in an in-memory array. The History card renders saved entries as collapsible accordion items showing grade level, difficulty, supplies used, timestamp, and the full formatted experiment. Entries can be individually removed.

---

## File Structure

```
Science-Experiment-Generator/
├── index.html    # main app — all HTML, CSS, JS in one file
├── spec.md       # this file
└── temp/
    └── llm-switchboard_5.html   # reference only — not part of this project
```
