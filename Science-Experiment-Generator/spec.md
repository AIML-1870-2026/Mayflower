# Science Experiment Generator — Spec

## Project Summary
A static single-page app that generates grade-appropriate science experiments from a grade level dropdown and a list of available supplies, using the OpenAI API called directly from the browser.

## Constraints

- **Static only** — one HTML file, no server, no Node.js, no npm
- **OpenAI exclusively** — calls `gpt-4o` via `https://api.openai.com/v1/chat/completions`
- **Unstructured output** — free-form markdown prose, no JSON schema or structured output
- **Markdown rendering** — `marked.js` (CDN) converts the model's markdown response to formatted HTML
- **In-memory key handling** — user uploads a `.env` file; `OPENAI_API_KEY` is parsed via `FileReader` and held in a JS variable for the session; never written to localStorage, cookies, or any server

## Reference Material

The `temp/` folder contains the complete LLM Switchboard project (`llm-switchboard_5.html`), which served as the reference implementation for:

- **`.env` file parsing** — the `loadEnvFile()` function mirrors the Switchboard's regex pattern for reading `OPENAI_API_KEY` from uploaded files
- **OpenAI `fetch()` call structure** — headers (`Authorization: Bearer ...`), body shape (`model`, `messages`, `max_tokens`), and response unwrapping (`choices[0].message.content`, `usage.total_tokens`) follow the Switchboard's `callOpenAI()` function exactly
- **Error handling** — the `try/catch` around the fetch, `.json()` error parsing, and visual error state follow the same pattern
- **UI / design system** — CSS custom properties, card layout, status dot, upload area, and metric row are adapted from the Switchboard's design

## File Structure

```
Science-Experiment-Generator/
├── index.html    # main app
├── spec.md       # this file
└── temp/
    └── llm-switchboard_5.html   # reference implementation (no nested .git or .gitignore)
```
