---
name: "antiscrap"
description: "Harden a web project you own against scraping and AI cloning: AI-bot blocking, defensive honeypot/tarpit, production-only obfuscation with no source maps, repo/endpoint lockdown, and a secret review across every file (code, text, docs, slides). Use before deploy."
---

# antiscrap

Lock down a web project **you own** against automated scraping, bundle theft and cloning by AI agents. Keep source clean in development; every aggressive measure ships to the production build only. This is defense on your own origin, never an attack on third parties.

## Non-negotiable rules

- Apply **all four layers** plus the per-file secret review. A partial pass is a fail. Do not stop early.
- Never weaken TLS, never commit secrets, never touch systems you do not own.
- Every layer ends with the matching verification command in the final section returning the expected result. No claim of "done" without command output proving it.
- Source maps in production: **must be off**. Treat a shipped `.map` as a release blocker.
- **Surgical edits only.** When fixing anything, change the exact offending line or element. NEVER rebuild a whole file or document from scratch to fix a small issue.

## Step 0 — Detect the stack (required first)

Inspect the project root and lock the plan to what is actually there:
- `next.config.js` / `app` or `pages` dir -> Next.js
- `vite.config.*` -> Vite
- `webpack.config.*` -> Webpack
- `vercel.json` / `netlify.toml` -> deploy platform
- `.htaccess` (Apache) vs Nginx conf vs serverless/edge

Headers and build hooks go in the file that matches the detected stack. If the stack is ambiguous, confirm before writing.

## Layer 1 — Block AI crawlers (hard)

**`public/robots.txt`** — explicit `Disallow: /` per bot, do not rely on a wildcard alone:
GPTBot, ChatGPT-User, OAI-SearchBot, ClaudeBot, Claude-Web, anthropic-ai, CCBot, Google-Extended, Bytespider, Amazonbot, PerplexityBot, PerplexityBot-User, Omgilibot, Applebot-Extended, Diffbot, ImagesiftBot, cohere-ai, Meta-ExternalAgent, FacebookBot, Timpibot, Scrapy.

**HTTP headers on every route:**
- `X-Robots-Tag: noai, noimageai, noarchive`
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `Referrer-Policy: strict-origin-when-cross-origin`
- `Permissions-Policy: geolocation=(), camera=(), microphone=()`
- `Content-Security-Policy` scoped to your own origins (no `unsafe-inline` for scripts).

**Edge/middleware User-Agent filter** (stricter than robots, since robots is advisory):
- Empty UA -> `403`.
- UA matching the blocklist above, or a known headless signature (`HeadlessChrome`, `python-requests`, `Go-http-client`, `curl`, `wget`, `node-fetch`, `axios`) on HTML routes -> `403`.
- Enforce in `middleware.ts` (Next), edge function (Vercel/Netlify), or Nginx `map $http_user_agent`. robots.txt asks; the filter enforces.

## Layer 2 — Honeypot / tarpit (defensive)

Known technique, same category as Cloudflare AI Labyrinth, Nepenthes, Iocaine. Entertains bots that ignore robots.txt without ever reaching humans.

- Trap route not linked for humans (e.g. `/api/internal-data-map`), reachable only via a DOM link with `aria-hidden="true"`, keyboard-unfocusable (`tabindex="-1"`), pushed off-screen by CSS (absolute negative offset / `clip`), never `display:none`.
- Also list the trap path as `Disallow` in robots.txt so any client that hits it has already ignored the rules — a clean signal.
- On hit: log IP + UA + timestamp + path, return generated decoy JSON (deeply nested, self-referential link structure) with a small fixed delay, and add the IP to a blocklist store (in-memory map, KV, or Redis).
- Blocked IPs get aggressive rate-limit / `403` on subsequent requests across the whole site, with TTL expiry.
- Scope strictly to requests arriving at your own origin. No outbound action against the client.

## Layer 3 — Production-only obfuscation

- Install `javascript-obfuscator` via the right integration: `vite-plugin-javascript-obfuscator`, `rollup-plugin-obfuscator`, or a Webpack/Next post-build step. Gate it on `NODE_ENV === 'production'`.
- Medium-high preset: `identifierNamesGenerator: 'hexadecimal'`, `controlFlowFlattening: true` (threshold ~0.75), `deadCodeInjection: true` (threshold ~0.3), `stringArray: true`, `stringArrayEncoding: ['base64','rc4']`, `stringArrayThreshold: 0.75`, `splitStrings: true`, `transformObjectKeys: true`, `selfDefending: true`, `disableConsoleOutput: true`.
- **Source maps off in prod**: `build.sourcemap: false` (Vite), `productionBrowserSourceMaps: false` (Next), `devtool: false` (Webpack prod). Confirm no `.map` is emitted or served.
- Measure bundle size and runtime before/after — `selfDefending` + control-flow flattening cost performance; tune the thresholds down if a critical path regresses.
- Development source stays untouched and readable.

## Layer 4 — Repo & endpoint lockdown

- `.gitignore` must cover: `.env*`, `*.pem`, `*.key`, credentials, internal configs, architecture maps, `/logs`. If any secret is already committed, flag it as compromised — it must be rotated, not just removed.
- Any external-API call with a key living in the frontend: move it server-side (API route / serverless function). The browser should never hold a private key.
- Rate-limit every public/server endpoint (sliding window per IP, e.g. token bucket in KV/Redis), returning `429` with `Retry-After` past the limit.
- Add bot-abuse limits on form and auth endpoints specifically.

## Per-file secret review (run after finishing EACH file — every file type)

Applies to **every file touched or produced**, not just code: source files, text/markdown, config (`.env`, JSON, YAML, TOML), and documents — Word (`.docx`), PowerPoint (`.pptx`), Excel (`.xlsx`), PDF, and images (check embedded metadata/EXIF and visible text). For binary docs, extract the text first, then scan.

Scan the finished file for anything that should not ship or be shared:
- API keys / tokens (`sk-`, `ghp_`, `AKIA`, `AIza`, bearer tokens, JWTs, long hex/base64 blobs).
- Hardcoded passwords or credentials.
- Real email addresses that should not be public, internal URLs, private IPs, author/company metadata, or other unwanted PII.

If something is found:
1. Report the exact file, the line or location (cell, slide, paragraph, metadata field), and what it looks like with the value masked (e.g. `sk-...4f2a`).
2. **Ask the user** whether to change/remove it.
3. If the user says yes: **remove only that single line or element** (the one cell, the one slide text box, the one paragraph, the one metadata field), or replace just that value with an env-var reference/placeholder. NEVER rebuild the whole file, document, or deck from scratch — touch only the offending item.
4. If the secret was real and live, remind the user to rotate it, since prior exposure means it is already compromised.
5. If the user says no, leave it and note it as an accepted risk.

Do not auto-delete without asking. Do not expand a one-item fix into a full-file or full-document rewrite.

## Verification (must pass before deploy)

1. `curl -s https://DOMAIN/robots.txt` -> every listed bot shows `Disallow: /`.
2. `curl -sI https://DOMAIN/` -> `X-Robots-Tag: noai, noimageai, noarchive` present.
3. `curl -s -o /dev/null -w '%{http_code}' -A '' https://DOMAIN/` -> `403` (empty UA).
4. `curl -s -o /dev/null -w '%{http_code}' -A 'python-requests/2.31' https://DOMAIN/` -> `403`.
5. Hit the honeypot route -> log entry written, decoy JSON returned; repeat normal request from that IP -> blocked/`403`.
6. Inspect prod build output -> JS identifiers are hex, no `.map` files emitted or reachable.
7. Hammer a protected endpoint past the window -> `429` with `Retry-After`.
8. Scan the final tree — code, text, config and documents — for secret patterns -> no live tokens/passwords/unwanted emails/PII remain (or each is user-accepted).

If any check fails, fix that layer before continuing (surgical edit only). Done means all eight pass with output shown.

## Execution checklist

- [ ] Step 0: stack + config files detected
- [ ] Layer 1: robots.txt, headers, enforcing UA filter
- [ ] Layer 2: honeypot + log + IP blocklist + rate-limit
- [ ] Layer 3: prod obfuscation on, source maps off, perf checked
- [ ] Layer 4: .gitignore clean, secrets safe/rotated, keys server-side, endpoints rate-limited
- [ ] Secret review run after each file (code, text, docs, slides); found items asked about and fixed item-only
- [ ] All eight verification checks pass with command output