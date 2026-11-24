# Draft: dead-air-fm

> Draft written outside Kiro. Input for Kiro spec mode. Covers FR-F1 – FR-F10. Folder: `apps/dead-air-fm/`. Visuals: `docs/DESIGN.md` section 10 (child theme `deadair`) and the Dead Air FM boards.

## Introduction

A late-night AI radio host in the browser. The host talks; anyone listening can cut in by speaking, get answered on air, and the show goes on. It shows Ghostline's interruption handling in the most obvious way.

## Requirements

### Requirement 1: The show (FR-F1, FR-F2)
1. THE host SHALL speak in segments of one or two sentences, in a fixed persona defined in `persona.ts` (name and voice: PRD Q7).
2. WHEN there is no bot audio and no user speech for 4 s THEN a new segment SHALL start.

### Requirement 2: Calling in (FR-F3 – FR-F5)
1. A listener SHALL interrupt by speaking. There SHALL be no push-to-talk.
2. AFTER an interruption, the next reply SHALL treat the listener as a caller and answer them (the prompt carries a caller flag).
3. WHEN 2 s pass after the answer with no new speech THEN the monologue SHALL resume.

### Requirement 3: Console (FR-F6 – FR-F9)
1. THE console SHALL show state lamps, the live waveform, the barge-in gauge (last value and session median, `—` until measured), the dead-air meter and the transcript.
2. Text cut off by an interruption SHALL be shown struck through, with a "never played" note.
3. WHEN a duck happens THEN an orange (`state-heard`) edge flash SHALL last at most 150 ms.
4. A NAIVE switch on the main screen SHALL change the policy so anyone can compare.

### Requirement 4: Provider switcher (FR-F10, P1)
1. A switcher SHALL change the TTS provider (Workers AI, Sarvam, ElevenLabs).

### Requirement 5: Look and access
1. THE app root SHALL use `data-theme="deadair"`; all colours from tokens.
2. Screens: Tune in, On air, You cut in, Mobile (DESIGN section 10.1).
3. EVERYTHING SHALL work by keyboard; the transcript works as captions; reduced motion stops all atmosphere animation.
4. Sound SHALL NOT play before the visitor presses Tune in.

## Suggested tasks (about 6)
Persona + segment loop → dead-air timer and resume → caller flag in prompts → tune-in screen → on-air console widgets → cut-in states + NAIVE switch.
