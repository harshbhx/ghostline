---
inclusion: always
---

# Tech

- TypeScript, `strict: true`, ES modules. No `any` in public types.
- pnpm workspaces. Packages: `skeleton`, `runtime-cf`, `web-kit`, `adapters`, `apps/*`, `bench`, `power`.
- Tests: Vitest; property tests with fast-check (at least 200 runs each). Fake clock: pass `now` explicitly, never sleep.
- Cloudflare: Wrangler, Workers, Durable Objects, Workers AI. Secrets via `wrangler secret`, locally `.dev.vars` (git-ignored).
- CI runs `pnpm -r test` and the import guard with no API keys. Live provider tests only when `LIVE=1`.
- Each package exposes its API only from `src/index.ts`. Import rules: `docs/ARCHITECTURE.md` section 2.
- Audio: PCM16 little-endian mono. Browser in 16 kHz, 20 ms frames. Phone 8 kHz (Twilio mu-law).
- Commit messages: Conventional Commits with the issue, e.g. `feat(core): add barge-in controller (refs #9)`.
