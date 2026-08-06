# slrup

Static deployment of Firecrawl’s [anydoc](https://github.com/firecrawl/anydoc) browser demo: a WASM document→Markdown converter that runs entirely client-side. Hosted on Cloudflare Workers via **Static Assets** — Wrangler serves `./public` with no Worker entry and no app build step.

Live conversion happens in the browser. Files never leave the machine.

## Prerequisites

- [Bun](https://bun.sh)
- A Cloudflare account (for deploy)

## Setup

```bash
bun install
```

`public/pkg/` is **gitignored**. After a fresh clone you must populate it (see below) or the page stays on “Loading the converter…” because `index.html` imports `./pkg/anydoc_wasm.js`.

## Populate `public/pkg/` (WASM glue)

You need at least:

- `public/pkg/anydoc_wasm.js`
- `public/pkg/anydoc_wasm_bg.wasm`

### Option A — copy from the published npm package (recommended)

This matches what the demo imports and does not require Rust tooling:

```bash
mkdir -p public/pkg
cd /tmp
npm pack @firecrawl/anydoc-wasm
tar xzf firecrawl-anydoc-wasm-*.tgz
cp package/anydoc_wasm.js package/anydoc_wasm_bg.wasm /path/to/slrup/public/pkg/
```

Replace `/path/to/slrup` with this repo’s absolute path. To pin a version:

```bash
npm pack @firecrawl/anydoc-wasm@0.1.6
```

### Option B — build from the anydoc source

Requires [Rust](https://rustup.rs) and [`wasm-pack`](https://rustwasm.github.io/wasm-pack/):

```bash
git clone https://github.com/firecrawl/anydoc.git
cd anydoc
wasm-pack build wasm --release --target web --no-pack --out-dir /path/to/slrup/public/pkg
```

That is the same layout the upstream demo uses (`wasm/www/pkg`). Rebuild whenever you want a newer anydoc than the last npm release.

### Refreshing later

Repeat Option A or B into `public/pkg/` (overwrite the files). Then redeploy if the site is already live — Wrangler only uploads what is on disk under `public/`.

## Local development

```bash
bun run dev
```

Opens the Wrangler dev server (default [http://127.0.0.1:8787](http://127.0.0.1:8787)).

## Deploy

```bash
bun run deploy
```

Runs `wrangler deploy` and publishes the contents of `public/` (including `pkg/` when present).

## Layout

| Path | Role |
| --- | --- |
| `public/index.html` | Demo UI + client-side conversion script |
| `public/assets/` | Fonts and logos |
| `public/pkg/` | WASM module + JS glue (not in git) |
| `wrangler.jsonc` | Static assets config for Cloudflare |

## Upstream

- Library and WASM bindings: [firecrawl/anydoc](https://github.com/firecrawl/anydoc)
- npm package: [@firecrawl/anydoc-wasm](https://www.npmjs.com/package/@firecrawl/anydoc-wasm)
- Official demo: [firecrawl.github.io/anydoc](https://firecrawl.github.io/anydoc/)
