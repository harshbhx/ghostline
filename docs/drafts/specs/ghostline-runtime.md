# Draft: ghostline-runtime

> Draft written outside Kiro. Input for Kiro spec mode. Covers FR-R1 – FR-R11 and the web kit FR-W1 – FR-W6. Packages: `runtime-cf/`, `web-kit/`.

## Introduction

The runtime hosts one Durable Object per session, runs the core, and talks to the browser and providers. The web kit captures the mic, plays bot audio, ducks instantly and shows state. Protocol: `docs/ARCHITECTURE.md` sections 4–5.

## Requirements

### Requirement 1: Routing and sessions (FR-R1, FR-R4)
1. THE Worker SHALL route `WS /session/:id/<doorway>` to one Durable Object per session ID, for doorways `browser`, `simulator` (P0), `exotel`, `twilio` (P1).
2. `POST /session` SHALL return a signed token that expires.
3. IF the token is missing, invalid or expired THEN the connection SHALL be refused.

### Requirement 2: Browser audio and control (FR-R2, FR-R3)
1. THE browser doorway SHALL accept PCM16 mono 16 kHz 20 ms binary frames.
2. THE server SHALL announce its output sample rate in `audio_format` before sending audio.
3. THE server SHALL send `state`, `duck`, `unduck`, `stop`, `audio_format`, `trace`, `transcript`; the client SHALL send `duck`, `playback_progress`, `stopped`, and `dtmf` (simulator).

### Requirement 3: Hot path (FR-R5)
1. THE executor SHALL NOT await storage or network inside a per-audio-frame handler.
2. THE runtime SHALL write storage at most once per turn.

### Requirement 4: Configuration and fallback (FR-R6, FR-R7)
1. STT, LLM, TTS and policy SHALL be chosen per session by config.
2. (P1) WHEN a provider errors or stalls for 3 s THEN the session SHALL switch to the next adapter in the chain (ARCHITECTURE section 8) and record it in the trace.

### Requirement 5: Traces (FR-R8)
1. EACH trace event SHALL be streamed to the client live.
2. EACH finished `TurnTrace` SHALL be stored; `GET /traces?session=` SHALL return JSONL and require a token.

### Requirement 6: Limits, disconnect, health (FR-R9 – FR-R11)
1. Sessions SHALL end after 20 minutes, or 3 minutes idle (config values); the bot SHALL say goodbye first.
2. WHEN a client disconnects THEN all provider streams SHALL be cancelled and the trace flushed.
3. `GET /health` SHALL return the version and git SHA.

### Requirement 7: Web kit capture (FR-W1, FR-W6)
1. Capture SHALL run in an AudioWorklet and send 20 ms PCM16 frames at 16 kHz with echo cancellation on.
2. THE mic permission prompt SHALL appear only after the user clicks a clear button.

### Requirement 8: Playback, duck, echo guard (FR-W2 – FR-W4)
1. THE playback queue SHALL know how many milliseconds were played; on `stop` it SHALL empty the queue and send `stopped {turnId, playedMs}`.
2. WHEN local mic energy stays above the threshold for 40 ms THEN playback gain SHALL drop to 0.2 and `duck` SHALL be sent.
3. Gain SHALL return on `unduck`, or after 300 ms without confirmation.
4. Frames for a stopped `turnId` SHALL be dropped.

### Requirement 9: Widgets (FR-W5)
1. Widgets (state lamps, waveform, latency gauge, dead-air meter, transcript) SHALL read only `state`, `trace` and `transcript` messages.
2. Widgets SHALL use tokens from `design/tokens.css` and work in the parent and `deadair` themes.

## Gate
Gate A (spike) and Gate B (first real interruption with a trace) are checked against this spec. See PRD section 14.

## Suggested tasks (about 8)
Worker router + tokens → Session DO + executor → browser doorway + protocol contract test → trace streaming + storage + `/traces` → limits/disconnect/health → web-kit capture worklet → playback queue + duck + echo guard → widgets.
