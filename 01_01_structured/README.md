# 01_01_structured

Minimal example demonstrating OpenAI API structured outputs with JSON schema validation.

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
npm run lesson1:structured
```

Or with node directly:

```bash
node app.js
```

## What it does

1. Defines a JSON schema for a "person" object (name, age, occupation, skills)
2. Sends unstructured text to the API with `text.format` set to the schema
3. The model returns guaranteed valid JSON matching the schema
4. Parses and displays the extracted structured data

## Key concept

The `text.format` parameter with `type: "json_schema"` and `strict: true` ensures the model output always conforms to your schema — no parsing errors, no missing fields.
