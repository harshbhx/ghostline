# Draft: ghostline-core

> Draft written outside Kiro. Input for Kiro spec mode. Covers FR-C1 – FR-C15. Package: `skeleton/`.

## Introduction

The core is a pure turn engine that decides when the bot listens, thinks, speaks and stops. It performs no I/O. A runtime feeds it events and executes the actions it returns. Interfaces: `docs/ARCHITECTURE.md` section 3. Tests: `docs/TESTING.md` section 2.

## Requirements

### Requirement 1: Phases and pure step (FR-C1, FR-C2)
**User story:** As a voice app builder, I want a deterministic engine, so that interruption behaviour can be tested without audio.

1. THE engine SHALL have exactly four phases: `listening`, `thinking`, `speaking`, `interrupted`.
2. THE engine SHALL expose `step(state, event, now, config) → {state, actions}`.
3. WHEN `step` is called twice with equal inputs THEN it SHALL return equal outputs (P6).
4. THE `skeleton/` package SHALL NOT call any network, storage, timer, random or clock API (checked by the import guard).

### Requirement 2: Deciding an interruption (FR-C3, FR-C4)
**User story:** As a caller, I want the bot to stop when I really speak, and not when I say "hmm".

1. WHILE the phase is `speaking`, WHEN user speech lasts at least `minSpeechMs` (default 120, allowed 60–400) THEN the engine SHALL confirm an interruption.
2. WHEN user speech lasts less than `minSpeechMs` THEN the engine SHALL NOT stop the bot (P4).
3. WHEN the transcript of the speech is a configured backchannel for the session language THEN the engine SHALL NOT stop the bot.
4. THE backchannel lists SHALL exist for `en`, `hi` and `gu` and SHALL be configurable.
5. WHERE the doorway is `browser`, WHILE the bot speaks, THE engine SHALL require both server speech and a client duck before confirming (echo guard, FR-W4).

### Requirement 3: End of turn (FR-C5)
1. WHEN the user is silent for `eotSilenceMs` (default 500) after speech THEN the engine SHALL end the user turn.
2. WHERE an end-of-turn detector is configured (P2), WHEN its score is 0.5 or more THEN the engine MAY end the turn earlier.

### Requirement 4: Stop sequence (FR-C6, FR-C7)
1. WHEN an interruption is confirmed THEN the engine SHALL emit, for that turn and in this exact order: `duck`, `stop_playback`, `cancel_llm`, `cancel_tts`, `clear_transport`, `commit_turn`.
2. AFTER `stop_playback` for turn T THE engine SHALL NOT emit `send_audio` for T (P1).

### Requirement 5: Stop time (FR-C8)
1. WHEN a `stopped {playedMs}` acknowledgement arrives THEN the engine SHALL record `lastAudioFrame` from it.
2. IF no acknowledgement arrives within 500 ms THEN the engine SHALL estimate `lastAudioFrame` and set `isEstimatedStop = true`.

### Requirement 6: Interruption while thinking (FR-C9)
1. WHILE the phase is `thinking`, WHEN confirmed user speech arrives THEN the engine SHALL emit `cancel_llm` and record outcome `cancelled`.

### Requirement 7: Spoken-only history (FR-C10, FR-C11)
1. WHEN a reply is interrupted THEN the engine SHALL commit only the text whose audio was played, cut at a word boundary (P3).
2. THE unplayed remainder SHALL be kept for display only and SHALL NOT appear in any later LLM request.
3. EVERY user turn SHALL end in exactly one outcome: `full`, `interrupted` or `cancelled` (P2).

### Requirement 8: Traces (FR-C12)
1. EVERY turn SHALL produce exactly one `TurnTrace` (schema `ghostline.turn.v1`, ARCHITECTURE section 6) with times from the injected clock (P5).

### Requirement 9: Naive baseline (FR-C13)
1. WHERE `policy = "naive"`, THE engine SHALL ignore interruptions and play every reply to the end.

### Requirement 10: Audio helpers (FR-C14)
1. THE package SHALL resample PCM16 between 8, 16 and 24 kHz and encode/decode mu-law (P7).

### Requirement 11: Public API (FR-C15)
1. THE package SHALL export its public API only from `skeleton/src/index.ts`.

## Notes for design.md
- Model timing (`minSpeechMs`, `eotSilenceMs`, stop-ack timeout) as `set_timer` / `timer` round trips so the core stays pure.
- Word-boundary cut: map `playedMs` to text using TTS chunk timings; if timings are missing, cut proportionally and back off to the previous space.

## Suggested tasks (about 11)
Types and config → phase machine → policy (minSpeech, backchannel, echo guard) → end of turn → stop sequence → stop-ack and estimate → thinking-cancel → history commit → tracer → naive policy → audio helpers → property tests P1–P7 → import guard.
