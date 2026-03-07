# Grounding Pipeline

Transforms markdown notes into interactive HTML with fact-checked, source-annotated concepts. Uses OpenAI's structured outputs and web search to extract, deduplicate, verify, and ground claims with citations.

## How It Works

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  INPUT: Markdown file                                                       │
│         ↓                                                                   │
│  1. EXTRACT ──► 1 paragraph = 1 API call (parallel batches of 5)            │
│         ↓                                                                   │
│  2. DEDUPE ───► group synonyms under canonical labels                       │
│         ↓                                                                   │
│  3. SEARCH ───► 1 concept = 1 API call + web search (parallel batches of 5) │
│         ↓                                                                   │
│  4. GROUND ───► 1 paragraph = 1 API call (parallel batches of 5)            │
│                                                                             │
│  OUTPUT: grounded.html                                                      │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Laser-focused processing**: Each paragraph and concept gets its own dedicated API call, ensuring maximum quality. Parallel execution (5 concurrent requests) keeps throughput high.

## Requirements

- Node.js 24+
- OpenAI API key or OpenRouter API key

## Setup

**1. Install dependencies**

```bash
cd 01_01_grounding
npm install
```

**2. Configure API key**

Use one shared config for all `01_01*` examples.

Create `.env` in the workspace root (`./.env`):

```
OPENAI_API_KEY=your_api_key_here
# or
OPENROUTER_API_KEY=your_api_key_here
```

See global setup guide: [`../README.md`](../README.md#lesson-01-setup)

> Note: on OpenAI this example uses native `web_search` tool calls; on OpenRouter it uses `:online` model suffix to enable web plugin grounding.

**3. Add a markdown file**

Place your `.md` file in the `notes/` folder (created automatically on first run).

## Run

```bash
# Process first markdown file found in notes/
npm run start

# Process a specific file
npm run start -- my-note.md

# Force regenerate all outputs (ignore cache)
npm run start -- --force
```

From workspace root:

```bash
npm run lesson1:grounding
npm run lesson1:grounding -- my-note.md
npm run lesson1:grounding -- --force
```

Or with node directly:

```bash
node app.js
```

## Output Files

All outputs are written to `output/`:

| File | Description |
|------|-------------|
| `concepts.json` | Extracted concepts per paragraph with scores |
| `dedupe.json` | Grouped canonical concepts with aliases |
| `search_results.json` | Web search summaries and sources |
| `grounded.html` | Final interactive HTML with tooltips |

## Pipeline Stages

### 1. Extract (`concepts.json`)

Each paragraph gets its own API call using `extractSchema`. Runs 5 paragraphs in parallel per batch.

```json
{
  "label": "Transformer introduced by Google researchers (2017)",
  "category": "claim",
  "needsSearch": true,
  "searchQuery": "Transformer introduced by Google researchers 2017 paper",
  "reason": "Specific historical attribution; best grounded by original publication.",
  "surfaceForms": ["introduced by Google researchers in 2017"],
  "scores": {
    "specificity": 2,
    "searchValue": 2,
    "relevance": 2,
    "canonicality": 2
  },
  "scoreTotal": 8
}
```

**Scoring rubric** (0-2 each, must total ≥6 to pass):

| Score | Meaning |
|-------|---------|
| `specificity` | 0=generic, 1=mildly specific, 2=named/technical |
| `searchValue` | 0=no validation needed, 1=optional, 2=needs verification |
| `relevance` | 0=peripheral, 1=mentioned, 2=central idea |
| `canonicality` | 0=redundant, 1=similar exists, 2=distinct and stable |

**Categories**: `claim`, `result`, `method`, `metric`, `resource`, `definition`, `term`, `entity`, `reference`

### 2. Dedupe (`dedupe.json`)

Groups equivalent concepts under canonical labels using `dedupeSchema`:

```json
{
  "canonical": "Transformer self-attention parallelism",
  "ids": [5],
  "aliases": [
    "Transformers use self-attention for parallel token processing",
    "process all tokens in parallel"
  ],
  "rationale": "Same architectural property: self-attention enables parallel processing."
}
```

Only concepts with `needsSearch: true` are deduplicated.

### 3. Search (`search_results.json`)

Each canonical concept gets its own dedicated API call with `web_search` tool. Runs 5 concepts in parallel per batch.

```json
{
  "canonical": "Scaling yields emergent capabilities",
  "summary": "Research shows LLMs exhibit emergent capabilities when scaled to hundreds of billions of parameters.",
  "keyPoints": [
    "Emergent capabilities appear at certain scale thresholds",
    "Examples include chain-of-thought reasoning",
    "Documented in 'Emergent Abilities of Large Language Models' (2022)"
  ],
  "sources": [
    { "title": "Emergent Abilities of Large Language Models", "url": "https://arxiv.org/abs/2206.07682" }
  ]
}
```

### 4. Ground (`grounded.html`)

Each paragraph gets its own API call to generate HTML with annotations. Runs 5 paragraphs in parallel per batch.

```html
<span class="grounded" data-grounding="{&quot;summary&quot;:&quot;...&quot;,&quot;sources&quot;:[...]}">
  introduced by Google researchers in 2017
</span>
```

The template (`template.html`) provides:
- Dark theme with Lexend/JetBrains Mono fonts
- Hoverable tooltips with summaries
- Clickable source links with favicons
- Mobile-friendly bottom sheet on touch devices
- Keyboard support (Esc to close)

## Project Structure

```
01_01_grounding/
├── app.js                    # Entry point - orchestrates pipeline
├── src/
│   ├── api.js                # OpenAI Responses API client
│   ├── config.js             # Paths, models, CLI parsing
│   ├── schemas/
│   │   ├── index.js          # Re-exports all schemas
│   │   ├── categories.js     # Category taxonomy enum
│   │   ├── extract.js        # extractSchema - single paragraph extraction
│   │   ├── dedupe.js         # dedupeSchema - concept grouping
│   │   ├── search.js         # searchSchema - single concept web search
│   │   └── ground.js         # groundSchema - single paragraph HTML output
│   ├── pipeline/
│   │   ├── extract.js        # Parallel paragraph extraction
│   │   ├── concept-filter.js # Post-extraction validation
│   │   ├── dedupe.js         # Grouping similar concepts
│   │   ├── search.js         # Parallel web search per concept
│   │   └── ground.js         # Parallel HTML generation per paragraph
│   ├── prompts/
│   │   ├── index.js          # Re-exports all prompt builders
│   │   ├── extract.js        # Single paragraph extraction prompt
│   │   ├── dedupe.js         # Dedupe prompt
│   │   ├── search.js         # Single concept search prompt
│   │   └── ground.js         # Single paragraph grounding prompt
│   └── utils/
│       ├── file.js           # JSON read/write, directory utils
│       ├── text.js           # Paragraph splitting, chunking
│       └── hash.js           # Content hashing for cache
├── template.html             # HTML shell with CSS and tooltip JS
├── notes/                    # Input markdown files
├── output/                   # Generated files
└── package.json
```

## Configuration

Edit `src/config.js` to change:

```javascript
export const models = {
  extract: resolveModelForProvider("gpt-5.4"), // Concept extraction
  search: resolveModelForProvider("gpt-5.4"),  // Web search synthesis
  ground: resolveModelForProvider("gpt-5.4")   // HTML generation
};

export const api = {
  endpoint: RESPONSES_API_ENDPOINT, // OpenAI or OpenRouter (auto-detected)
  timeoutMs: 180_000,
  retries: 3,
  retryDelayMs: 1000
};
```

Concurrency is set to 5 parallel requests per batch in each pipeline stage.

## Key Design Decisions

**Laser-focused processing** — Each paragraph/concept gets its own API call. No batching multiple items into a single prompt. This ensures maximum quality and attention from the model.

**Parallel execution** — 5 concurrent requests per batch balances throughput with API rate limits. Each stage processes items in parallel within batches, then persists results before the next batch.

**Content hashing** — Cache invalidation uses SHA-256 hashes of paragraph content, not file paths. Unchanged paragraphs skip re-extraction.

**Surface forms** — Each concept tracks exact text spans from the source (`surfaceForms`), enabling precise HTML annotation without hallucinated text.

**Strict JSON schemas** — All LLM outputs use `strict: true` JSON schemas with descriptions, ensuring valid structured responses.

**Progressive caching** — Each stage persists results immediately after processing each batch, enabling resume on interruption.
