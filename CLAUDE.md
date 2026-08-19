# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

"Glomi" is a single-page prototype for a live AI makeup coach. A user picks an occasion and lists (or photographs) their products, Claude generates a step-by-step routine, and then a camera-based "live" screen lets them ask Claude to look at a captured frame and give spoken feedback per step.

The whole repo is two files plus a setup README:
- `index.html` — the entire frontend: markup, CSS, and vanilla JS all inline in one file. No framework, no bundler, no build step.
- `api/claude.js` — a single Vercel serverless function that proxies requests to the Anthropic Messages API, keeping `ANTHROPIC_API_KEY` server-side.
- `README.md` — end-user instructions for uploading this repo to GitHub, deploying it on Vercel, and wiring up the API key/domain. It is not developer documentation.

There is no `package.json`, no dependency manifest, and no test/lint/build tooling anywhere in the repo — there are no such commands to run. Development is just editing `index.html` and `api/claude.js` directly and deploying to Vercel to test.

## Architecture

**Deployment model**: static file (`index.html`) + one serverless function (`api/claude.js`), deployed on Vercel with zero config. Vercel auto-detects any file under `api/` exporting a default handler as a serverless function; `index.html` is served as a static asset. The API key lives only in the Vercel project's environment variables (`ANTHROPIC_API_KEY`), never in client code.

**Client → server → Anthropic flow**: the frontend never calls Anthropic directly. All Claude calls go through the `callClaude(content)` helper in `index.html`, which POSTs `{ content }` to `/api/claude`. `api/claude.js` wraps that `content` in `messages: [{ role: 'user', content }]`, forwards it to `https://api.anthropic.com/v1/messages` with the server-side key, model `claude-sonnet-5`, and `max_tokens: 1000`, and relays the raw Anthropic response (or error) back to the client. `content` can be plain text or a mixed array of `{type: "text"}` / `{type: "image", source: {type: "base64", ...}}` blocks — the API function is a thin, opinion-free passthrough, so all prompt construction and response parsing happens client-side.

**App state machine** (`index.html`, `<script>` at the bottom): a single global `state` object (`occasion`, `products`, `steps`, `currentStep`, `stream`) drives four screens toggled via the `.hidden` class — `#setup-screen` → `#loading-screen` → `#live-screen` → `#done-screen`. There is no router or framework; screens are shown/hidden directly and DOM elements are looked up by ID on demand rather than cached.

**Setup screen**: user picks an occasion chip and types/pastes products, or uploads a product photo which is sent to Claude vision (`callClaude` with an image block) to extract product names as comma-separated text appended into the products textarea.

**Step generation**: on "Start my live guide", a prompt asking for 4–6 JSON-described steps (`title`, `instructions`, `product`, `zone`, `markerCount`, `diagramNote`) is sent via `callClaude`. The response is stripped of markdown code fences (`stripFences`) and `JSON.parse`d into `state.steps`. Claude is expected to return *only* raw JSON — if that contract breaks (extra prose, invalid JSON), generation fails and the user is bounced back to the setup screen with an error message. There is no retry or schema validation beyond the `try/catch`.

**Live screen**: shows a circular camera preview (`getUserMedia`), the current step's text, and an SVG face diagram (`#face-diagram`) with dot markers placed according to `zoneCoords` (a hand-authored map of named face zones — `forehead`, `brows`, `eyelids`, `under-eyes`, `cheeks`, `nose`, `lips`, `jawline`, `chin`, `all-over` — to `[x, y]` coordinates on a 200×240 viewBox). `getMarkerPoints` spreads multiple markers per zone outward in rings when `markerCount` exceeds the zone's base point count. Steps can optionally be read aloud via the Web Speech API (`speechSynthesis`).

**Live feedback ("Check my progress")**: captures the current video frame to a hidden `<canvas>`, un-mirrors it (camera preview is CSS-mirrored via `scaleX(-1)` for a natural selfie view, but the captured frame is flipped back before sending so Claude sees it right-reading), and sends it plus a step-aware coaching prompt to `callClaude`. The spoken/text feedback is rendered in `#feedback-card` and read aloud if enabled.

## Conventions

- Keep everything in `index.html` unless a change specifically needs new server-side logic — this project intentionally has no build step, and splitting JS/CSS into separate files would require introducing tooling that doesn't exist yet.
- CSS uses custom properties defined once on `:root` (colors, radius) — reuse those variables (`--bg`, `--surface`, `--pink`, `--black`, `--text`, `--muted`, `--good`) rather than hardcoding new colors.
- All Claude prompts request a specific, strict output contract (plain text for coaching feedback, fenced-free JSON for step generation) and the client code parses that contract directly — if you change a prompt's expected output shape, update the corresponding parsing code (`stripFences`/`JSON.parse` for steps, plain `.trim()` for feedback text) in the same change.
- `api/claude.js` is intentionally a dumb passthrough (no prompt logic, no validation beyond method/error checks) — put all prompt engineering in `index.html`, not in the serverless function.
