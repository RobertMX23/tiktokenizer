# Tiktokenizer

![Tiktokenizer](https://user-images.githubusercontent.com/1443449/222597674-287aefdc-f0e1-491b-9bf9-16431b1b8054.svg)

**An online playground for token counting and inspection** — accurately count tokens for prompts using OpenAI’s [tiktoken](https://github.com/openai/tiktoken) and Hugging Face tokenizers, with visual segment-level breakdowns.

[Demo video](https://user-images.githubusercontent.com/1443449/222598119-0a5a536e-6785-44ad-ba28-e26e04f15163.mp4)

---

## Table of contents

- [What it does](#what-it-does)
- [Features](#features)
- [Supported models and encodings](#supported-models-and-encodings)
- [Tech stack](#tech-stack)
- [Project structure](#project-structure)
- [Getting started](#getting-started)
- [Environment variables](#environment-variables)
- [Scripts](#scripts)
- [How tokenization works](#how-tokenization-works)
- [API](#api)
- [Configuration](#configuration)
- [Development](#development)
- [Testing](#testing)
- [License and acknowledgments](#license-and-acknowledgments)

---

## What it does

Tiktokenizer lets you:

- **Count tokens** for any supported model or encoding before sending requests to OpenAI or other APIs, so you can stay within context limits and estimate cost.
- **Inspect token boundaries** — see exactly which substring corresponds to each token, with grapheme-aware segment highlighting (handles emoji and complex scripts).
- **Compare encodings** — switch between raw encodings (e.g. `cl100k_base`, `o200k_base`) and full model presets (e.g. GPT-4o, GPT-3.5-turbo) to see how tokenization differs.
- **Use chat-style formatting** — for chat models, build multi-turn conversations with system/user/assistant messages and see the exact token count of the serialized prompt (including special tokens like `<|im_start|>` and `<|im_end|>`).
- **Try open-source tokenizers** — run Hugging Face tokenizers (CodeLlama, Llama 3, Phi-2, Gemma, etc.) in the browser via [Transformers.js](https://huggingface.co/docs/transformers.js), with tokenizer files either pre-downloaded at build time or loaded from your deployment.

The app runs tokenization in the browser where possible (e.g. tiktoken for OpenAI, Transformers.js for Hugging Face), and falls back to server-side encoding when needed. The encode API can be used programmatically for automation or integration.

---

## Features

- **Real-time token count** as you type, with optional whitespace visualization (spaces, tabs, newlines).
- **Segment-level highlighting** — hover over a segment to see its token IDs and the exact text span; each segment is colored for clarity.
- **URL-driven model selection** — the chosen model/encoder is stored in the query string (`?model=...`), so you can share or bookmark a specific configuration.
- **Chat composer** for OpenAI chat models: add system/user/assistant messages and optional names; the serialized format (with chat special tokens) is tokenized so the count matches what the API will see.
- **Searchable model picker** — filter by name to quickly switch between dozens of models and encodings.
- **Pre-downloaded Hugging Face tokenizers** — `tokenizer.json` and `tokenizer_config.json` are fetched at build/dev time and served from `public/hf/`, so open-source tokenizers work without hitting Hugging Face on every load (gated models require `HF_API_KEY`).

---

## Supported models and encodings

### OpenAI encodings (raw)

Use these when you only care about the encoding, not a specific model:

| Encoding     | Typical use                    |
|-------------|---------------------------------|
| `gpt2`      | Legacy GPT-2                    |
| `r50k_base` | Base BPE (e.g. Davinci, Codex)  |
| `p50k_base` | Codex, text-davinci-*           |
| `p50k_edit` | Edit models                     |
| `cl100k_base` | GPT-3.5-turbo, GPT-4, embeddings |
| `o200k_base` | GPT-4o                          |

### OpenAI chat models

- `gpt-4o`, `gpt-3.5-turbo`, `gpt-4`, `gpt-4-32k`, `gpt-4-1106-preview`

These use the correct encoding and chat special tokens (`<|im_start|>`, `<|im_end|>`, `<|im_sep|>`).

### OpenAI legacy text and embedding models

Including `text-davinci-003`, `text-embedding-ada-002`, `text-embedding-3-small`, `text-embedding-3-large`, and other legacy text/embedding/code models defined in the codebase.  
*(Note: some newer embedding models may be disabled with a “Model may be too new” message until support is added.)*

### Open-source (Hugging Face)

Tokenizers are loaded via [@xenova/transformers](https://github.com/xenova/transformers.js) from pre-downloaded files under `public/hf/`:

- **Code Llama:** `codellama/CodeLlama-7b-hf`, `codellama/CodeLlama-70b-hf`
- **Llama 3:** `meta-llama/Meta-Llama-3-8B`, `meta-llama/Meta-Llama-3-70B`
- **Others:** `microsoft/phi-2`, `google/gemma-7b`, `deepseek-ai/DeepSeek-R1`, `Qwen/Qwen2.5-72B`, `tiiuae/falcon-7b`, `01-ai/Yi-6B`, `openai/whisper-tiny`

Gated models require a Hugging Face API key (see [Environment variables](#environment-variables)).

---

## Tech stack

| Layer        | Technology |
|-------------|------------|
| **Framework** | [Next.js 13](https://nextjs.org/) (Pages Router) |
| **UI**      | [React 18](https://react.dev/), [Tailwind CSS](https://tailwindcss.com/), [Radix UI](https://www.radix-ui.com/) (Select, Popover, Checkbox), [cmdk](https://cmdk.paco.me/) (command palette), [Lucide](https://lucide.dev/) icons |
| **Styling** | [class-variance-authority](https://cva.style/), [clsx](https://github.com/lukeed/clsx), [tailwind-merge](https://github.com/dcastil/tailwind-merge) |
| **Data / API** | [tRPC](https://trpc.io/) v10, [TanStack React Query](https://tanstack.com/query), [Zod](https://zod.dev/) for validation |
| **Tokenizers** | [tiktoken](https://github.com/openai/tiktoken) (OpenAI), [@xenova/transformers](https://github.com/xenova/transformers.js) (Hugging Face) |
| **Text / segments** | [graphemer](https://github.com/orling/graphemer) for grapheme-aware splitting (emoji, etc.) |
| **Misc**     | [superjson](https://github.com/blitz-js/superjson), [bignumber.js](https://mikemcl.github.io/bignumber.js/), [Vercel Analytics](https://vercel.com/analytics) |

The project follows the [T3 Stack](https://create.t3.gg/) (TypeScript, tRPC, Tailwind, Next.js) and uses shadcn/ui-style components with Radix primitives.

---

## Project structure

```
tiktokenizer/
├── public/
│   └── hf/                    # Pre-downloaded Hugging Face tokenizers (org/model/tokenizer.json, etc.)
├── src/
│   ├── components/            # Reusable UI (Button, Input, Select, Command, Popover, Checkbox)
│   ├── env.mjs                # Server/client env validation (Zod)
│   ├── models/
│   │   ├── index.ts           # Model/encoding enums and helpers (AllOptions, isChatModel, etc.)
│   │   └── tokenizer.ts       # TiktokenTokenizer, OpenSourceTokenizer, createTokenizer()
│   ├── pages/
│   │   ├── _app.tsx           # App shell, React Query + tRPC providers
│   │   ├── index.tsx          # Main page: encoder select, editor, token viewer
│   │   └── api/
│   │       ├── trpc/[trpc].ts # tRPC handler
│   │       └── v1/
│   │           ├── encode.ts  # REST: encode text by model or encoder
│   │           └── edge.ts   # Edge runtime demo (tiktoken WASM)
│   ├── sections/
│   │   ├── ChatGPTEditor.tsx  # Chat message composer for OpenAI chat models
│   │   ├── EncoderSelect.tsx  # Model/encoder dropdown with search
│   │   └── TokenViewer.tsx    # Token count + segment highlighting
│   ├── server/
│   │   └── api/               # tRPC router and root
│   ├── scripts/
│   │   └── download.ts        # Fetches HF tokenizer files into public/hf/
│   ├── styles/
│   │   └── globals.css
│   └── utils/
│       ├── api.ts
│       ├── cn.ts              # className helper
│       └── segments.ts        # Map tokens to text segments (tiktoken + Hugging Face)
├── .env.example
├── next.config.mjs            # Env validation, webpack async WASM
├── package.json
├── tailwind.config.cjs
└── tsconfig.json
```

- **`models/`** — Defines which models and encodings exist and how they map to tokenizers.
- **`tokenizer.ts`** — Implements `TiktokenTokenizer` (tiktoken) and `OpenSourceTokenizer` (Transformers.js), plus `createTokenizer(model)` which chooses and instantiates the right one.
- **`utils/segments.ts`** — Builds segment lists (text + token IDs) for the UI; uses graphemer so boundaries respect graphemes (e.g. emoji).
- **`scripts/download.ts`** — Run at `dev`/`build`; downloads `tokenizer.json` and `tokenizer_config.json` for each open-source model into `public/hf/`.

---

## Getting started

### Prerequisites

- **Node.js** 18+ (for Next.js 13 and current tooling)
- **Yarn** (recommended; the repo uses `yarn` in scripts)

### Installation

```bash
git clone https://github.com/dqbd/tiktokenizer.git
cd tiktokenizer
yarn install
```

Copy environment variables and set a Hugging Face API key if you need gated open-source models:

```bash
cp .env.example .env
# Edit .env and set HF_API_KEY (required for build/dev unless you skip validation — see below)
```

Run the app (this will run the download script first, then start Next.js):

```bash
yarn dev
```

Open [http://localhost:3000](http://localhost:3000). Select a model or encoding, type or paste text (or use the chat editor for chat models), and see the token count and segment view.

---

## Environment variables

Defined and validated in `src/env.mjs` (Zod). Server-side only unless prefixed with `NEXT_PUBLIC_`.

| Variable | Required | Description |
|----------|----------|-------------|
| `NODE_ENV` | Inferred | `development`, `test`, or `production`. Defaults to `development` if unset. |
| `HF_API_KEY` | Yes (by default) | [Hugging Face API token](https://huggingface.co/settings/tokens). Used by `download.ts` to fetch tokenizer files for gated models (e.g. Llama, Code Llama). |

To skip env validation (e.g. Docker builds or when not using open-source tokenizers), set:

```bash
SKIP_ENV_VALIDATION=1
```

Then you can run `yarn build` or `yarn dev` without setting `HF_API_KEY`. The app will still work for OpenAI models and encodings; open-source tokenizers that need the download step may fail if files are missing.

---

## Scripts

| Command | Description |
|---------|-------------|
| `yarn dev` | Runs `download.ts` (with env), then `next dev`. Use for local development. |
| `yarn build` | Runs `download.ts`, then `next build`. Use for production build. |
| `yarn start` | Runs `next start`. Serve a previously built app. |
| `yarn lint` | Runs `next lint` (ESLint). |

The download script writes into `public/hf/<org>/<model>/tokenizer.json` and `tokenizer_config.json`. If a file already exists, it is skipped. Remove files or the directory to re-download.

---

## How tokenization works

### Two backends

1. **OpenAI (tiktoken)**  
   For OpenAI encodings and models, the app uses the [tiktoken](https://github.com/openai/tiktoken) library. Encodings are resolved via `get_encoding()` or `encoding_for_model()`. Chat models (e.g. GPT-4o, GPT-3.5-turbo) use the correct encoding and optional extra special tokens for chat (e.g. `cl100k_base` / `o200k_base` with `<|im_start|>`, `<|im_end|>`, `<|im_sep|>`).  
   Tokenization runs in the main thread; for a minimal Edge example using tiktoken WASM, see `src/pages/api/v1/edge.ts`.

2. **Open-source (Hugging Face)**  
   For open-source models, the app uses [@xenova/transformers](https://github.com/xenova/transformers.js) (`PreTrainedTokenizer`). Tokenizer files are loaded from `public/hf/` (populated by `download.ts`). On the client, `env.remoteHost` is set to the current origin so requests go to your server.  
   Some models (e.g. Code Llama, Llama 2) use a leading `<s>` token that is stripped in the segment logic via `hackModelsRemoveFirstToken` so the highlighted segments align with the visible text.

### Segments

- **Purpose:** Show which substring corresponds to which token(s), including multi-token graphemes (e.g. emoji).
- **Implementation:**  
  - **Tiktoken:** `utils/segments.ts` uses `getTiktokenSegments()`: encode with `"all"` (all allowed tokenizations), decode token bytes, and align to input graphemes (via [graphemer](https://github.com/orling/graphemer)) to build segment boundaries.  
  - **Hugging Face:** `getHuggingfaceSegments()` uses the tokenizer’s `convert_ids_to_tokens` and aligns to the same grapheme-split input.  
- **UI:** `TokenViewer` shows each segment with a background color; hovering highlights the segment and can show whitespace (spaces, tabs, newlines) for debugging.

---

## API

### `POST /api/v1/encode`

Encodes text with a given model or encoder and returns token IDs and count. Useful for scripts or external tools.

**Request body (JSON):**

- **By encoder:** `{ "text": string, "encoder": "<encoding>" }`  
  `encoder` must be one of the OpenAI encoding names (e.g. `cl100k_base`, `o200k_base`).
- **By model:** `{ "text": string, "model": "<model>" }`  
  `model` must be one of the supported OpenAI or open-source model names.

**Response (JSON):**

```json
{
  "name": "cl100k_base",
  "tokens": [9906, 1917],
  "count": 2
}
```

- `name` — Encoding or tokenizer name.
- `tokens` — Array of token IDs.
- `count` — Length of `tokens`.

Validation is done with Zod; invalid `encoder` or `model` yields a 400-style parse error.

---

## Configuration

- **Next.js** (`next.config.mjs`):  
  - Imports `src/env.mjs` for env validation unless `SKIP_ENV_VALIDATION` is set.  
  - Enables `asyncWebAssembly` and `layers` in webpack for tiktoken WASM (e.g. Edge route).  
  - `i18n`: single locale `en`.

- **Tailwind** (`tailwind.config.cjs`), **PostCSS** (`postcss.config.cjs`), **ESLint** (`.eslintrc.cjs`), **Prettier** (`prettier.config.cjs`) are standard for a T3/Next + Tailwind setup.

---

## Development

- **Adding a new OpenAI model/encoding:** Extend the Zod enums in `src/models/index.ts` and, if needed, add branch logic in `src/models/tokenizer.ts` (e.g. for new chat special tokens or encodings).
- **Adding an open-source model:** Add it to `openSourceModels` in `src/models/index.ts`. Run `yarn dev` or `yarn build` so `download.ts` fetches its tokenizer files into `public/hf/`. If the tokenizer uses a leading sentinel token (like `<s>`), add the model to `hackModelsRemoveFirstToken` in `src/models/index.ts` so segments align correctly.
- **Llama 3 revisions:** Some Llama 3 variants use a non-`main` revision; `tempLlama3HackGetRevision()` in `src/models/index.ts` maps them to the correct ref (e.g. `refs/pr/35`).

---

## Testing

Tests use [Vitest](https://vitest.dev/). Segment logic is covered in `src/utils/segments.test.ts`. Run tests with:

```bash
yarn test
```

(If a test script is not in `package.json`, add one, e.g. `"test": "vitest"`.)

---

## License and acknowledgments

- **License:** [MIT](LICENSE). Copyright (c) 2023 Tat Dat Duong.
- **Author:** [dqbd](https://duong.dev)  
- **Sponsorship:** [Diagram](https://diagram.com/) for sponsorship and guidance.
- **Thanks:** [T3 Stack](https://create.t3.gg/), [shadcn/ui](https://github.com/shadcn/ui), [openai/tiktoken](https://github.com/openai/tiktoken), [Transformers.js](https://github.com/xenova/transformers.js).
