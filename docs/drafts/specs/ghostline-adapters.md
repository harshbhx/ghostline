# Draft: ghostline-adapters

> Draft written outside Kiro. Input for Kiro spec mode. Covers FR-A1 – FR-A8. Package: `adapters/`. Interfaces: `docs/ARCHITECTURE.md` section 7. Provider facts: `.kiro/steering/providers.md`.

## Introduction

Adapters connect the runtime to speech and language providers behind small interfaces, so apps can switch providers by config.

## Requirements

### Requirement 1: Workers AI (FR-A1, P0)
1. THE package SHALL provide Workers AI adapters for STT, TTS and LLM.
2. WITH only a Cloudflare account and no other keys, both apps SHALL run end to end in English.
3. Models: STT `@cf/deepgram/nova-3`, TTS `@cf/deepgram/aura-1` (supports `Clear`), LLM `@cf/zai-org/glm-4.7-flash` (see providers.md).

### Requirement 2: Sarvam STT (FR-A2, P0)
1. THE adapter SHALL stream audio for `hi-IN`, `gu-IN`, `en-IN` and emit partial and final transcripts.
2. IT SHALL use the realtime endpoint (`/speech-to-text-realtime/ws`, `saaras:v4`, 16 kHz `linear16`) and fall back to the legacy `/speech-to-text/ws`.
3. Message shapes are in providers.md; one live test per language behind `LIVE=1`.

### Requirement 3: Sarvam TTS (FR-A3, P0)
1. Text SHALL be sent in sentence-sized chunks.
2. WHEN cancelled THEN the adapter SHALL stop forwarding audio, close the socket and open a new socket for the next reply (Sarvam has no in-band cancel).
3. AFTER cancel, no audio of the cancelled reply SHALL be emitted (fake-socket test).

### Requirement 4: ElevenLabs TTS (FR-A4, P1)
1. THE adapter SHALL stream English speech over the multi-context WebSocket; cancel = `close_context` and a new context (in-band).

### Requirement 5: Sarvam LLM (FR-A5, P1) and Bedrock (FR-A6, P2)
1. Sarvam LLM: `POST /v1/chat/completions`, model `sarvam-105b`. Bedrock only if P2 is reached.

### Requirement 6: Honest cancel (FR-A7)
1. EVERY TTS adapter SHALL declare `supportsInBandCancel`, true only if the provider really supports it.

### Requirement 7: Testing (FR-A8)
1. EVERY adapter SHALL have tests against recorded fixtures that run in CI without keys.
2. Live tests SHALL run only when `LIVE=1`.

## Suggested tasks (about 7)
Shared interfaces + fake sockets → Workers AI STT/TTS/LLM → Sarvam STT → Sarvam TTS + cancel test → ElevenLabs TTS (P1) → Sarvam LLM (P1) → provider switcher config.
