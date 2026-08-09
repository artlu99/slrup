# Privacy and data posture

Maintainer-written, uncertified. Last review: 2026-08-09.

`slrup` converts files to Markdown in the browser. Bytes never leave the device. After first load, a service worker caches everything and the page works offline.

## Claims

| Badge | Claim | Evidence |
| --- | --- | --- |
| `scope: local` | Conversion happens in-browser via WASM. | `toMarkdownBytes` in `public/index.html`. No `fetch`, no `XMLHttpRequest`, no relay. |
| `tracking: none` | No analytics, no cookies, no logs. | No analytics in `public/index.html`. No `Set-Cookie`. `observability.enabled` is `false` in `wrangler.jsonc`. |
| `offline: yes` | Works without network after first load. | `public/sw.js` caches `index.html`, assets, and WASM on install. Fetch handler serves cache first. |

## Data flow

1. Browser fetches `index.html`, SVG assets, `sw.js`, and `public/pkg/anydoc_wasm*.wasm` from Cloudflare edge. CSS is inline.
2. User selects file → `Uint8Array` in memory.
3. Bytes → WASM → Markdown text. No network send.
4. Copy/Download use in-memory text.

## Third parties

Cloudflare only. It hosts the static site and sees the inbound HTTPS request (IP, time, User-Agent, path). Its privacy policy governs what it keeps. No Google Fonts, no analytics (Plausible, Fathom, GA, etc.), no CDN assets.

## Logs

- Application: none. `observability.enabled` is `false`. No Worker entrypoint.
- Edge: Cloudflare-controlled; maintainer cannot change.

## Client-side footprint

- Fonts: system stack only. No web fonts.
- Cookies: none.
- Storage: none (`localStorage`, `sessionStorage`, `IndexedDB`).
- Scripts: two inline in `index.html` (SW registration, WASM init + drag-and-drop). No external scripts.
- Service worker: `public/sw.js`. Caches on install, serves cache first, same-origin only.

## Supply chain

- WASM: `@firecrawl/anydoc-wasm@0.1.7` (npm), placed in `public/pkg/` via `npm pack`. Source: [firecrawl/anydoc](https://github.com/firecrawl/anydoc). See `README.md`.
- Build tool: `wrangler@4.118.0`, dev only.
- Runtime dependencies: none. Output is static files.

## Not a legal document

Badges describe technical facts, not legal facts (GDPR, CCPA, ePrivacy). Not a certification or pentest. Makes no claims about Cloudflare internal logs.

## Verify

```bash
grep observability wrangler.jsonc
grep -niE 'analytics|gtag|google|plausible|fathom|track' public/index.html
grep -nE 'fetch\(|XMLHttpRequest|navigator\.sendBeacon' public/index.html
ls public/pkg/
ls public/sw.js
grep -n 'serviceWorker' public/index.html
```
