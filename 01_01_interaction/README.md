# 01_01_interaction

Minimal example demonstrating OpenAI API interaction with follow-up responses.

## Requirements

- Node.js 24+
- OpenAI API key or OpenRouter API key

## Setup

Use one shared config for all `01_01*` examples.

Create `.env` in the workspace root (`./.env`):

```
OPENAI_API_KEY=your_api_key_here
# or
OPENROUTER_API_KEY=your_api_key_here
```

See global setup guide: [`../README.md`](../README.md#lesson-01-setup)

## Run

```bash
npm run start
```

From workspace root:

```bash
npm run lesson1:interaction
```

Or with node directly:

```bash
node app.js
```

## What it does

1. Asks a question: "What is 25 * 48?"
2. Makes a follow-up request: "Divide that by 4." by passing full input history
3. Displays both responses with reasoning token counts
