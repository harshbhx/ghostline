# AGENTS.md

Rules for Kiro and any other coding agent working in this repo.

## Read first, in this order

1. `docs/PRD.md`: what we build and in what order. Requirement IDs (`FR-C6`) are stable.
2. `docs/ARCHITECTURE.md`: packages, interfaces, protocol, `TurnTrace` schema.
3. `docs/TESTING.md`: property tests and metric formulas.
4. `.kiro/steering/`: always-on rules. `voice-invariants.md` is never broken.
5. `docs/DESIGN.md` and `design/tokens.css`: only when touching UI.

If two documents disagree, the PRD's source-of-truth map decides. Then fix the other document.

## Rules

1. **P0 first.** Do not start P1 or P2 work unless its GitHub issue is open.
2. **One task at a time.** Finish, test, then stop. Do not regenerate whole specs.
3. **Never invent a number.** Results come only from the bench with a `run_id`. Otherwise write `—` (UI) or `<measure>` (docs).
4. **Provider facts** come from `.kiro/steering/providers.md` (checked against official docs, with sources). Never guess an endpoint or message shape; if something is missing or a provider behaves differently, check its docs and update that file in the same commit.
5. **No secrets in code or git.** Use `wrangler secret` or `.dev.vars`. Never write to `.env*`, `.dev.vars*` or anything under `secrets/`.
6. **Tests run without keys.** Live tests only behind `LIVE=1`.
7. **The core is pure.** `skeleton/` has no network, storage, timers or clock calls.
8. **Commits** name the issue: `feat(core): add turn policy guard (refs #8)`. The last commit of an issue says `closes #8`.
9. **Log Kiro usage truthfully** in `docs/kiro-usage.md`: one row per real event, never pre-filled.

## Layout

```
skeleton/            pure core: engine, policy, barge-in, history, tracer, audio helpers
runtime-cf/          Worker + Session Durable Object + doorways
web-kit/             browser capture, playback, duck, widgets
adapters/            Workers AI, Sarvam, ElevenLabs, Exotel, Twilio, (Bedrock)
apps/dead-air-fm/    app 1, browser radio host (theme: deadair)
apps/missed-call-oracle/  app 2, phone Q&A + simulator (parent theme)
bench/               fake caller, clips manifest, metrics, runner, hf/
power/               Kiro power: plugin.json, skills, mcp.json (ghostline-mcp), hook installer
docs/                PRD, architecture, testing, design, drafts, kiro-usage
design/              tokens.css, tokens.json
assets/brand/        logo kit
.kiro/               steering, specs, hooks (committed, never ignored)
```
