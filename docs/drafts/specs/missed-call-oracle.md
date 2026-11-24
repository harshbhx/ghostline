# Draft: missed-call-oracle

> Draft written outside Kiro. Input for Kiro spec mode. Covers FR-O1 – FR-O13. Folder: `apps/missed-call-oracle/`. Visuals: parent theme, DESIGN section 11.

## Introduction

Ask a question by voice on any phone, in Hindi, Gujarati or English, and hear a short answer on the same call. A browser phone simulator always works, so judges can try it without a phone number.

## Requirements

### Requirement 1: Greeting and language (FR-O1, FR-O2)
1. THE greeting SHALL say the caller is talking to an AI assistant, in Hindi by default, and offer 1 Hindi, 2 Gujarati, 3 English.
2. WHEN the caller presses 1, 2 or 3 THEN STT and TTS SHALL switch to that language for the session.

### Requirement 2: Answers (FR-O3, FR-O6, FR-O7)
1. Answers SHALL be at most two short sentences, in the session language.
2. WHEN asked for live facts (prices, weather, news) THEN the Oracle SHALL say it cannot check live information yet.
3. WHEN asked about health, legal or emergency matters THEN it SHALL give general information only and tell the caller to contact a doctor, a lawyer or the local emergency number; it SHALL never give a diagnosis or a medicine dose.

### Requirement 3: Interruption (FR-O4)
1. Interruption SHALL work on every doorway. On phone doorways the provider's `clear` message SHALL be sent and media sending SHALL stop.

### Requirement 4: Simulator (FR-O5, FR-O8, P0)
1. A browser phone simulator (keypad, Call, Hang up) SHALL send keypad digits and 8 kHz band-limited audio through the same Oracle logic.
2. THE console SHALL show call status, language, phase, estimated barge-in time and the transcript; Hindi and Gujarati lines carry `lang`.

### Requirement 5: Real phone (FR-O9 – FR-O12, P1)
1. Exotel: media in 100 ms chunks that follow Exotel's size rules (property O1). Facts confirmed first (U5).
2. Twilio Media Streams as backup only if Exotel is not enabled.
3. A phone call SHALL end politely after 5 minutes.
4. Phone numbers SHALL be stored only as salted SHA-256 hashes.

### Requirement 6: Missed-call callback (FR-O13, P2)
1. A valid webhook for caller N SHALL place one call back to N, at most once per 10 minutes per number (property O2).
2. IF the token is bad THEN it SHALL return 403 and place no call.

## Suggested tasks (about 7)
Prompts + language switch → safety/live-data prompt tests → simulator doorway (8 kHz band-limit) → simulator console → Exotel adapter + one real call (P1) → Twilio backup (P1) → callback (P2).
