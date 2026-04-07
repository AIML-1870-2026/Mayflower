# LLM Switchboard — Project Spec

**Version:** 3.0  
**Deliverable:** `llm-switchboard.html` — a single self-contained HTML file  
**Assignment context:** Reference infrastructure for interacting with large language models via their APIs, to be used as a base for future specialised tooling assignments.

---

## Overview

A single-page web app that lets a user send prompts to large language models through their APIs directly from the browser. No backend, no build step, no dependencies to install — just open the HTML file.

---

## Providers & Models

### Anthropic
| Model ID | Display Name |
|---|---|
| `claude-opus-4-5` | Claude Opus 4.5 |
| `claude-opus-4-0` | Claude Opus 4 |
| `claude-sonnet-4-5` | Claude Sonnet 4.5 |
| `claude-sonnet-4-0` | Claude Sonnet 4 |
| `claude-3-5-sonnet-20241022` | Claude 3.5 Sonnet |
| `claude-3-5-haiku-20241022` | Claude 3.5 Haiku |

### OpenAI
| Model ID | Display Name | Notes |
|---|---|---|
| `gpt-5` | GPT-5 | Uses `max_completion_tokens` |
| `gpt-5-mini` | GPT-5 mini | Uses `max_completion_tokens` |
| `gpt-4o` | GPT-4o | Uses `max_tokens` |
| `gpt-4o-mini` | GPT-4o mini | Uses `max_tokens` |
| `o3` | o3 | Uses `max_completion_tokens`, no JSON response format |
| `o4-mini` | o4 mini | Uses `max_completion_tokens`, no JSON response format |

> **Parameter note:** GPT-5, GPT-5 mini, o3, and o4-mini require `max_completion_tokens` instead of `max_tokens`. The o-series and GPT-5 models also do not support `response_format: json_object`.

---

## Core Requirements

### 1. API Key Handling
- Manual entry via password fields (one per provider)
- File upload: accepts `.env` or `.csv` files, auto-parses `ANTHROPIC_API_KEY` and `OPENAI_API_KEY` (handles quoted values like `KEY="sk-..."`)
- Keys are stored **in memory only** — never written to disk, localStorage, or any persistent storage
- Status indicator in the top bar shows which providers have keys set

### 2. Provider & Model Selection
- Provider dropdown (Anthropic / OpenAI) in the Config tab
- Model dropdown updates dynamically based on selected provider
- Configurable max tokens field (default: 1024)

### 3. Dual Output Modes
Toggle between two modes directly in the Chat tab:

**Free text** — model responds in natural language. No schema enforcement.

**JSON output** — model is instructed via prompt injection to return valid JSON matching a user-defined schema. For providers that support it, `response_format: { type: "json_object" }` is also set on the request.

Schema is entered as raw JSON in a textarea. Five example schemas are available as one-click pills:
- Sentiment analysis
- Entity extraction
- Structured summary
- Q&A pair
- Classification

### 4. Example Prompts
Ten pre-loaded example prompts displayed as cards in the Chat tab:

| Name | Mode |
|---|---|
| General question | Free text |
| Summarize text | Free text |
| Code review | Free text |
| Explain simply | Free text |
| Brainstorm ideas | Free text |
| Sentiment analysis | JSON |
| Entity extraction | JSON |
| Structured summary | JSON |
| Q&A extraction | JSON |
| Classify content | JSON |

Clicking a card: loads the prompt, switches to the correct mode, and pre-fills the matching schema if JSON. Cards for the non-active mode are dimmed to guide the user. All examples are also listed in the Library tab.

---

## Stretch Features

### Side-by-Side Comparison
- Separate tab ("Compare")
- Independent provider + model selectors for Model A and Model B
- Same prompt sent to both simultaneously via `Promise.all`
- Responses and metrics displayed in a two-column grid
- Each side shows its own response time, token count, and character count

### Response Metrics
Displayed below every response:
- **Response time** — wall-clock time from request to response, in seconds
- **Tokens used** — from the API's usage field (`input_tokens + output_tokens` for Anthropic, `total_tokens` for OpenAI)
- **Characters** — length of the response string

### Prompt Library
- Save any prompt from the Chat tab with one click ("Save prompt to library")
- Manually add prompts with a name + text form in the Library tab
- Browse, load, and delete saved prompts
- All pre-loaded example prompts are also accessible from the Library tab
- Storage is in-memory only (cleared on page refresh)

### Structured Output Validator
Appears below the response when in JSON output mode. Validates the model's response against the provided schema:
- Parses the response as JSON (shows error if not valid JSON)
- Checks each field in `required` and `properties` for presence and correct type
- Labels each field: `ok`, `missing`, or `wrong type (got X)`
- Shows an overall compliance score (percentage of fields passing) with a colour-coded bar (green / amber / red)

---

## API Implementation Details

### Anthropic
- Endpoint: `https://api.anthropic.com/v1/messages`
- Auth: `x-api-key` header
- Required headers: `anthropic-version: 2023-06-01`, `anthropic-dangerous-direct-browser-access: true` (enables direct browser calls)
- System prompt passed as top-level `system` field
- Response text extracted from `content[].text` where `type === "text"`
- Token count: `usage.input_tokens + usage.output_tokens`

### OpenAI
- Endpoint: `https://api.openai.com/v1/chat/completions`
- Auth: `Authorization: Bearer <key>` header
- System prompt passed as `{ role: "system" }` message
- Models using `max_completion_tokens`: `gpt-5`, `gpt-5-mini`, `o3`, `o4-mini`
- Models using `max_tokens`: `gpt-4o`, `gpt-4o-mini`
- `response_format: { type: "json_object" }` applied for JSON mode, except on o-series and GPT-5 models
- Response text: `choices[0].message.content`
- Token count: `usage.total_tokens`

### Schema injection (both providers)
When JSON mode is on, the schema is appended to the user prompt:
```
[user prompt]

Respond ONLY with valid JSON matching this schema:
[schema as JSON]
No markdown fences, no extra text.
```

---

## UI Structure

```
Topbar
├── Logo + version tag
├── Nav tabs: Config | Chat | Compare | Library
└── Status indicator (dot + label: no key / anthropic / openai / both keys set)

Config tab
├── API keys card (two password fields + .env upload)
└── Model selection card (provider dropdown, model dropdown, max tokens)

Chat tab
├── Example prompt cards (10 cards, dimmed by mode)
├── Output mode card
│   ├── Mode switcher: Free text | JSON output
│   ├── Mode description
│   ├── Schema editor + example pills (JSON mode only)
│   └── System prompt field
├── Prompt card (textarea + Send button)
└── Response card (response box + metrics + validator)

Compare tab
├── Config card (provider A + model A, provider B + model B, prompt)
└── Two-column results grid (response + metrics per side)

Library tab
├── Save new prompt form
├── Saved prompts list
└── All example prompts list
```

---

## Running the App

Open `llm-switchboard.html` in any modern browser. If you get CORS errors, serve it via a local server:

```bash
# Node
npx serve .

# Python
python3 -m http.server 8000
```

Then open `http://localhost:8000/llm-switchboard.html`.

### .env file format
```
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-proj-...
```
Quoted values (`KEY="value"`) are also supported.

---

## Security Notes

- API keys are held in JavaScript memory for the duration of the page session only
- Keys are never sent anywhere except the respective provider's API endpoint
- Do not commit `.env` files to version control
- Rotate any keys that have been shared or exposed publicly
