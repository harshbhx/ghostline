---
inclusion: always
---

# Structure

```
skeleton/src/        engine.ts (step), policy.ts, bargein.ts, history.ts, tracer.ts, audio.ts, types.ts, index.ts
runtime-cf/src/      worker.ts (router), session.ts (Durable Object), executor.ts, doorways/*.ts, index.ts
web-kit/src/         capture.worklet.ts, playback.ts, duck.ts, widgets/*, index.ts
adapters/src/        workers-ai/*, sarvam/*, elevenlabs/*, exotel/*, twilio/*, index.ts
apps/<name>/         app logic + UI; one wrangler config per app
bench/               clips/manifest.json, runner.ts, metrics.ts, report.ts, hf/
power/               POWER.md, steering/, mcp/ (ghostline-mcp), install-hooks.ts
```

- Interfaces and message shapes: `docs/ARCHITECTURE.md`. Do not invent new message types; add them there first.
- Metric formulas: `docs/TESTING.md`. The bench imports them; the UI never recomputes them differently.
- UI: `docs/DESIGN.md`, tokens from `design/tokens.css` only.
