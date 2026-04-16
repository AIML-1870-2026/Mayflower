# Science Experiment Generator — Spec

## Project Summary
A static single-page app that generates grade-appropriate science experiments from a grade level dropdown and a list of available supplies, using the OpenAI API called directly from the browser.

---

## Constraints

### Static page — HTML, CSS, and JavaScript only
No server, no Node.js, no npm. The page is a single `.html` file that opens directly in the browser with zero build steps or dependencies beyond CDN-loaded libraries (`marked.js` for markdown rendering and Google Fonts).

### OpenAI models only — no provider switching
This project calls OpenAI's `gpt-4o` model exclusively. There is no Anthropic dropdown and no ability to switch providers. This is a deliberate architectural choice rooted in the **CORS lesson from the Switchboard project**: Anthropic's API blocks direct browser-to-API requests, so an app with no backend server *cannot* use Anthropic. OpenAI's API allows browser-to-API calls, which is why it's the only viable provider for a purely static page.

### Unstructured responses — free-form text, no JSON schema
The model returns a prose markdown document. There is no `response_format: json_object`, no schema template, and no structured output parsing. The model is prompted to follow a markdown section structure (Overview, Hypothesis, Procedure, etc.) but the response is raw text, not JSON.

### Markdown rendering
The model's response includes markdown formatting — bold text, numbered lists, headings, horizontal rules. The app uses `marked.js` (loaded from jsDelivr CDN) to convert this markdown to formatted HTML before injecting it into the page. The user never sees raw `**asterisks**` or `##` symbols.

### API key — loaded from .env, in-memory only
The user uploads a `.env` file via a file picker. The key is parsed from the file text using a regex (`OPENAI_API_KEY\s*=\s*...`) by the browser's `FileReader` API and stored in a JavaScript variable for the session. It is never written to `localStorage`, `sessionStorage`, cookies, or any server. When the page is closed, the key is gone.

### Deployment
Deployed as a static page to the class GitHub organization via GitHub Pages:
**https://aiml-1870-2026.github.io/Mayflower/Science-Experiment-Generator/**

---

## Reference Material

The `temp/` folder contains the complete LLM Switchboard project (`llm-switchboard_5.html`) as a reference implementation. The following patterns were directly adapted from it:

| Pattern | Switchboard source | Used in this project |
|---|---|---|
| `.env` file parsing | `loadEnvFile()` regex | Same regex extracts `OPENAI_API_KEY` |
| OpenAI `fetch()` structure | `callOpenAI()` — headers, body, response unwrap | `generateExperiment()` fetch block |
| Error handling | `try/catch`, `.json()` fallback, error state CSS | Same structure in `generateExperiment()` |
| UI design system | CSS custom properties, card layout, status dot, upload area, metric row | Adapted throughout |

---

## Stretch Challenges Implemented

### 1. Quick Supply Selector
A grid of clickable chips organized into three groups (Kitchen, Craft & Office, Science & Outdoor) lets users select common household supplies with one click. Selected chips are highlighted and their values are synced into the supplies textarea. Users can still type freely in the textarea.

### 2. Difficulty Rating
The system prompt instructs the model to include a `## Difficulty` section with exactly one word: `Easy`, `Medium`, or `Hard`. After generation, a regex extracts this value and displays it color-coded in the metrics row (green / amber / red).

### 3. Experiment History
A "Save to history" button on each result stores the experiment in an in-memory array. The History card below renders all saved entries as collapsible accordion items, each showing grade level, difficulty, supplies used, timestamp, token count, and the full formatted experiment. Entries can be individually removed.

---

## File Structure

```
Science-Experiment-Generator/
├── index.html    # main app (all HTML, CSS, JS in one file)
├── spec.md       # this file
└── temp/
    └── llm-switchboard_5.html   # reference implementation (no .git or .gitignore)
```
