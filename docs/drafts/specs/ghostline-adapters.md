# Draft: ghostline-adapters

> Draft written outside Kiro. Input for Kiro spec mode. Covers FR-A1 – FR-A8. Package: `adapters/`. Interfaces: `docs/ARCHITECTURE.md` section 7. Provider facts: `.kiro/steering/providers.md`.

## Introduction

Adapters connect the runtime to speech and language providers behind small interfaces, so apps can switch providers by config.

## Requirements

### Requirement 1: Workers AI (FR-A1, P0)
1. THE package SHALL provide Workers AI adapters for STT, TTS and LLM.
2. WITH only a Cloudflare account and no other keys, both apps SHALL run end to end in English.
3. Model names SHALL be confirmed against the current Workers AI catalogue first (U8).

### Requirement 2: Sarvam STT (FR-A2, P0)
1. THE adapter SHALL stream audio for `hi-IN`, `gu-IN`, `en-IN` and emit partial and final transcripts.
2. IT SHALL prefer the realtime endpoint and fall back to the older endpoint.
3. Event names and parameters SHALL be confirmed first (U1); one live test per language behind `LIVE=1`.

### Requirement 3: Sarvam TTS (FR-A3, P0)
1. Text SHALL be sent in sentence-sized chunks.
2. WHEN cancelled THEN the adapter SHALL stop forwarding audio, close the socket and open a new socket for the next reply.
3. AFTER cancel, no audio of the cancelled reply SHALL be emitted (fake-socket test).

### Requirement 4: ElevenLabs TTS (FR-A4, P1)
1. THE adapter SHALL stream English speech; endpoint and formats confirmed from the API reference first.

### Requirement 5: Sarvam LLM (FR-A5, P1) and Bedrock (FR-A6, P2)
1. Sarvam LLM through its OpenAI-compatible API (U3). Bedrock only if P2 is reached.

### Requirement 6: Honest cancel (FR-A7)
1. EVERY TTS adapter SHALL declare `supportsInBandCancel`, true only if the provider really supports it.

### Requirement 7: Testing (FR-A8)
1. EVERY adapter SHALL have tests against recorded fixtures that run in CI without keys.
2. Live tests SHALL run only when `LIVE=1`.

## Suggested tasks (about 7)
Shared interfaces + fake sockets → Workers AI STT/TTS/LLM → Sarvam STT → Sarvam TTS + cancel test → ElevenLabs TTS (P1) → Sarvam LLM (P1) → provider switcher config.
