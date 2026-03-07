# AI_devs Examples

Global setup guide for running lesson examples.

## Lesson 01 Setup

This setup is shared by:

- `01_01_interaction`
- `01_01_structured`
- `01_01_grounding`

Configure your API key once in the workspace root and reuse it across all `01_01*` examples.

### API Keys

Create `.env` in the workspace root (you can copy `env.example`):

```bash
# macOS / Linux
cp env.example .env
```

```powershell
# Windows PowerShell
Copy-Item env.example .env
```

Edit `.env` and set one key:

```bash
OPENAI_API_KEY=your_api_key_here
# or
OPENROUTER_API_KEY=your_api_key_here
```

Optional: if both keys are set, choose provider explicitly:

```bash
AI_PROVIDER=openai
# or
AI_PROVIDER=openrouter
```

### Run examples

Optional (future-proof): install dependencies for all `01_01*` examples at once:

```bash
npm run lesson1:install
```

```bash
npm run lesson1:interaction
```

```bash
npm run lesson1:structured
```

```bash
npm run lesson1:grounding
```

For `01_01_grounding`, pass file names/flags like this:

```bash
npm run lesson1:grounding -- my-note.md
npm run lesson1:grounding -- --force
```

### Notes

- Local `.env` files inside `01_01*` folders are no longer required.
- Provider is auto-detected from `./.env` (`OPENAI_API_KEY` or `OPENROUTER_API_KEY`).
- If both keys exist, default provider is `openai` (override with `AI_PROVIDER`).
- Endpoint switches automatically to:
  - `https://api.openai.com/v1/responses` (OpenAI)
  - `https://openrouter.ai/api/v1/responses` (OpenRouter)
- `01_01_grounding` uses:
  - OpenAI: native `tools: [{ type: "web_search" }]`
  - OpenRouter: `:online` model suffix (web plugin path)
