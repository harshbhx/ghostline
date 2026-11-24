<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/brand/ghostline-lockup-animated-dark.svg">
    <img src="assets/brand/ghostline-lockup-animated-on-light.svg" alt="Ghostline" width="420">
  </picture>
</p>

<p align="center"><strong>Voice agents that know when to shut up.</strong></p>

Ghostline is a small, open TypeScript skeleton for real-time voice agents that stop talking the moment you interrupt. It runs on Cloudflare Workers and Durable Objects, and it measures how fast it stopped, every turn.

> **Status: planning.** The product, architecture, design system and spec drafts are in place. All code will be built in Kiro, starting 25 Nov 2026. Entry for [Kiroween 2026](https://kiroween.devpost.com/), category Skeleton Crew.

## What it is

- **The skeleton.** A turn engine (who is talking now), a barge-in controller (stop the bot fast and cleanly), and a latency tracer (measure every turn). Speech-to-text, the LLM, text-to-speech and telephony plug in through adapters.
- **Two apps on the same skeleton.**
  - **Dead Air FM** (`apps/dead-air-fm/`): a late-night AI radio host in the browser. Cut in at any moment by talking.
  - **Missed-Call Oracle** (`apps/missed-call-oracle/`): ask a question by voice, in Hindi, Gujarati or English, and hear the answer on the same call. A browser phone simulator always works.
- **A Kiro power.** Steering plus an MCP server (`ghostline-mcp`) so Kiro can build a new voice app on the skeleton and test its interruption logic.

## Why

Most voice bots keep talking when you interrupt, then answer as if you heard everything they said. Ghostline treats interruption as a tested contract:

1. Duck the bot's audio on the client at once.
2. Confirm on the server that it is real speech, not "hmm" or a cough.
3. Stop playback, cancel the LLM and TTS, clear the phone buffer.
4. Keep only the words that were actually played in the conversation history.
5. Record a trace with the real time of the last audio frame.

## Results

Not measured yet. Every number in this repo will come from the bench (`bench/`) with a `run_id`. Until then it is written as `—`.

| Metric | Naive | Ghostline |
|---|---|---|
| Barge-in, p50 (browser) | — ms | — ms |
| False stops | — | — |
| Missed stops | — | — |
| Time to first audio, p50 | — ms | — ms |

## Repository map

| Path | What |
|---|---|
| `docs/PRD.md` | What we build, for whom, in what order |
| `docs/ARCHITECTURE.md` | How the parts connect: interfaces, protocol, trace schema |
| `docs/TESTING.md` | Property tests, bench method and every metric formula |
| `docs/DESIGN.md`, `design/` | Design system (Graphite + Blue, with Dead Air FM as a child theme) and tokens |
| `assets/brand/` | Logo kit |
| `AGENTS.md`, `.kiro/steering/` | Rules for Kiro and any other agent working here |
| `docs/drafts/specs/` | Spec drafts, written outside Kiro, that feed Kiro's spec mode |
| `.kiro/specs/` | The real specs, generated in Kiro |
| `docs/kiro-usage.md` | A true log of how Kiro was used |

## How this is built

- **Code:** written in Kiro, from specs and tasks in `.kiro/specs/`. Every Kiro event that matters is logged in `docs/kiro-usage.md`.
- **Tools used besides Kiro** (disclosed as the rules ask):
  - Claude (Anthropic): research, the PRD, architecture and testing docs, the design system, the logo kit and the spec drafts in `docs/drafts/`. None of these are code.
  - Provider APIs at runtime: Cloudflare Workers AI, Sarvam, ElevenLabs, Exotel / Twilio.

## License

Code: [MIT](LICENSE). Bench dataset (when published): CC BY 4.0.
