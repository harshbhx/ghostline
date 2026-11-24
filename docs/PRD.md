# Ghostline — Product Requirements Document (PRD)

| | |
|---|---|
| Product | Ghostline: an open-source skeleton for voice agents you can interrupt |
| Tagline | "Voice agents that know when to shut up." |
| Owner | Harsh Bhanushali, DualMind Labs (dualmindlab.tech) |
| Status | v3 · 24 Nov 2026 (schedule rebased; design system v3) |
| Built for | Kiroween 2026 (Devpost), category **Skeleton Crew** |
| Hard deadline | Fri 4 Dec 2026, 2:00 PM PT = Sat 5 Dec 2026, 03:30 IST. No extensions. |
| Internal deadline | **Fri 4 Dec 2026, 22:00 IST** |
| Repo path | `docs/PRD.md` |
| Authorship | Written by Harsh with AI research help (disclosed in the README). It is an input to Kiro's spec workflow, not Kiro output. |

---

## 0. How to use this document

This PRD says **what** we build, **for whom**, **why**, and **in what order**. It does not say how to build it.

### Source-of-truth map

| Question | Document that decides |
|---|---|
| What is in scope? What is the priority? | This PRD |
| How do the parts connect? What are the interfaces and schemas? | `docs/ARCHITECTURE.md` |
| What rules can never be broken in audio code? | `.kiro/steering/voice-invariants.md` |
| What are the exact provider API facts? | `.kiro/steering/providers.md` |
| What are the exact tasks? | `.kiro/specs/<spec>/tasks.md` |
| How is each metric calculated? | `docs/TESTING.md` |
| What does the UI look like? | `docs/DESIGN.md` |

If two documents disagree, the table above decides. Then fix the other document.

### Rules for any AI agent reading this

1. Requirement IDs (for example `FR-C6`) are stable. Use them in specs, tasks, tests and commit messages.
2. Build **P0** first. Do not start a P1 or P2 item unless its GitHub issue is open.
3. If a fact is not in this PRD or in the linked documents, **ask**. Do not guess.
4. A number marked "target" is a goal. It is never a result. Results come only from the bench, with a `run_id`.
5. Anything marked **UNVERIFIED** must be checked against the linked doc or a live test before code depends on it.

---

## 1. Summary

**Ghostline is a small, open TypeScript skeleton for real-time voice agents that stop talking when the user interrupts.** It runs on Cloudflare Workers and Durable Objects.

It ships three things:

1. **The skeleton.** A turn engine (who is talking now), a barge-in controller (stop the bot fast and cleanly), and a latency tracer (measure every turn). Speech-to-text, the LLM, text-to-speech and telephony plug in through adapters.
2. **Two apps on the same skeleton.**
   - **Dead Air FM**: a late-night AI radio host in the browser. You call in by talking and can cut in at any moment.
   - **Missed-Call Oracle**: ask a question by voice on any phone, in Hindi, Gujarati or English, and hear the answer on the same call. A browser phone simulator is always available.
3. **A Kiro power.** Steering files plus an MCP server (`ghostline-mcp`) that let Kiro build a new voice app on the skeleton and test its interruption logic.

**Headline:** voice AI you can interrupt like a real person, on any phone, and Kiro can build a new one in minutes.

**Proof:** a public benchmark that compares a normal pipeline ("naive") with Ghostline on the same recorded interruptions. All numbers are measured and published.

---

## 2. Context and constraints

### 2.1 Hackathon rules that shape the product

Check each row against https://kiroween.devpost.com/rules before submitting.

| Rule | Effect on the product |
|---|---|
| Skeleton Crew: a lean code template, proven with **two distinct applications** in two separate app folders | `apps/dead-air-fm/` and `apps/missed-call-oracle/` must both work and must look clearly different |
| The app must be built with Kiro. Too little Kiro use can disqualify | All code is produced through Kiro specs and tasks. `/.kiro` is committed and never gitignored |
| New projects only (built 30 Oct – 4 Dec 2026). Pre-existing code must be disclosed | No code from Tarang FM or any earlier project |
| Required: public GitHub repo with an open-source license visible in About, working app URL, demo video under 3 minutes, write-up on how Kiro was used | These are deliverables with their own requirements (section 8.11) |
| Judging: five criteria, 20 % each: Potential Impact, Technological Implementation (how well Kiro was used), Design, Innovation, Presentation | Every tier decision in section 13 is checked against these five |
| The write-up asks about specs, hooks, steering, MCP, property-based tests, powers, custom agents, Crew | Each of these has a real use in this product |
| Judging may use automated AI analysis | README, `/.kiro` and the write-up must be clean and easy to parse |

### 2.2 Builder constraints

- One builder, 24 Nov – 4 Dec after setup slipped, working beside college and other projects. Kiroween is the priority.
- **4,000 Kiro credits.** Documents must let Kiro get tasks right the first time.
- Available provider credits: Sarvam API credits and about 50,000 ElevenLabs credits.
- An Indian phone number exists only inside Sarvam's managed Voice Agents platform. That platform does not allow your own models or code, so it is **not used**.

### 2.3 Truth rules

1. No performance number is written anywhere unless the bench measured it. Until then, write `<measure>`.
2. The README lists every tool used besides Kiro.
3. `docs/kiro-usage.md` gets one true row per real event. Nothing is pre-filled.
4. Drafts written outside Kiro live in `docs/drafts/` and are named as such.

---

## 3. Problem

### 3.1 What goes wrong today

When a person interrupts a voice bot, five things usually go wrong:

1. **The bot keeps talking.** Audio that was already generated keeps playing.
2. **The bot stops for nothing.** A cough, a "hmm" or a door slam makes it stop mid-sentence.
3. **The phone network keeps playing.** Audio already sent to the telephony provider plays from its buffer after the bot "stopped".
4. **The bot remembers words nobody heard.** The full reply is saved in history even though the user heard only half.
5. **Nobody measures it.** Demos show the happy path. The time from "user starts talking" to "bot is silent" is rarely published.

### 3.2 Why it is hard

- Speed and calm pull against each other. Stop faster and you stop on coughs. Wait longer and you talk over people.
- Every provider cancels audio differently. Sarvam's streaming TTS has no cancel message, so you must close the socket. Exotel and Twilio have a `clear` message, and Exotel's removes only audio that has not started playing.
- Cloudflare's own voice pipeline (`agents/voice`) is marked Beta, and its interruption behavior is not the one this product needs.

### 3.3 Evidence

| # | Claim | Source | Status |
|---|---|---|---|
| E1 | Barge-in is described as the hardest part of voice-agent engineering | KDnuggets article | Secondary. Find and check the link before citing |
| E2 | Callers interrupt in about 1 of 5 calls | PolyAI | Secondary. Check before citing |
| E3 | In human conversation most turn changes happen within about 0–200 ms | Stivers et al., PNAS 2009 (10 languages) | Published paper. Check the exact wording |
| E4 | Production voice agents answer in roughly 680 ms (median) | Vixi Agency, citing 2026 benchmarks | Secondary and weak. Use only as context |
| E5 | A minimum-duration guard cuts false stops a lot but adds delay | DEV Community post citing Future AGI | Secondary. Our own bench must confirm |
| E6 | Sarvam streaming TTS has no in-band cancel | docs.sarvam.ai (TTS streaming guide) | Provider docs, read 19 Nov 2026 |
| E7 | Sarvam Voice Agents does not allow your own STT, LLM or TTS | docs.sarvam.ai (models page) | Provider docs, read 19 Nov 2026 |
| E8 | Exotel `clear` removes only audio that has not started playing | Exotel AgentStream docs | Provider docs, from research |
| E9 | Free AI voice helplines for basic phones already exist (India 155261, Pakistan 7272, Viamo 3-2-1) | Government and operator pages | Shows demand for voice on basic phones. We do not compete with them; we give builders the kit |
| E10 | Our own numbers: naive pipeline vs Ghostline | `bench/` | `<measure>` |

Only E10 may be used as a performance claim in the submission.

### 3.4 What exists and where the gap is

| Existing option | What it gives | What is missing |
|---|---|---|
| Managed platforms (Sarvam Voice Agents, ElevenLabs Agents) | A working phone agent quickly | Closed. You cannot change turn-taking or see per-turn timing |
| Provider tutorials (Twilio, Exotel, Sarvam) | A sample bot with basic interruption | One provider each, no tests, no measurement, no reusable core |
| Cloudflare `agents/voice` | STT, TTS and interruption in a Durable Object | Beta. No client-side duck, no telephony buffer clearing, no spoken-only history |
| **Ghostline** | A tested, measured, provider-neutral core that Kiro can build on | — |

---

## 4. Vision and positioning

**For** developers building voice agents, **who** need the bot to stop when people interrupt, **Ghostline is** an open skeleton **that** gets interruption right in the browser and on phone calls and proves it with numbers. **Unlike** closed platforms and single-provider tutorials, **it** is small, tested, measured, and ready for an AI coding agent to extend.

### What makes it different

1. **Fast and calm together.** It stops on real speech and ignores coughs and "hmm".
2. **Measured in the open.** Every turn has a trace. The bench data is public.
3. **Two doorways, one core.** The same engine serves a browser mic and a phone call.
4. **Kiro-ready.** The power teaches Kiro the rules, so a new app works on the first try.

### Product principles

1. Interruption is a contract with tests, not a feature flag.
2. The core has no network code. Everything outside it is replaceable.
3. If it was not played, it was not said.
4. A number without a measurement is not allowed.
5. The simulator must always work. No demo depends on an outside approval.

---

## 5. Goals, non-goals and success metrics

### Goals

- **G1** An open TypeScript skeleton where turn policy, barge-in and tracing are tested and pluggable.
- **G2** Two clearly different apps on the same core.
- **G3** A true latency number for every turn, shown live and exported as open data.
- **G4** A Kiro power and MCP server so Kiro can scaffold and test a new voice app.
- **G5** A complete, valid Kiroween submission by the internal deadline.

### Non-goals

- Speech-to-speech models, voice cloning, training any model.
- Multi-party calls, call transfer, call recording.
- User accounts, billing, an admin panel, production uptime promises.
- Sarvam's managed Voice Agents platform or ElevenLabs' managed Agents platform.
- Telephony providers other than Exotel and Twilio.
- Live data in Oracle answers (weather, prices) in version 1.
- Competing with government helplines.

### Success metrics

| ID | Metric | Target | How it is measured |
|---|---|---|---|
| M1 | Barge-in latency, browser (speech start → last bot audio) | 200–300 ms (target) | Bench, p50 and p95 with n |
| M2 | Barge-in latency, phone (estimated) | 250–400 ms (target) | Bench or real calls, reported separately |
| M3 | False-stop rate (stopped on backchannel or noise) | below 5 % (target) | Bench clip set |
| M4 | Missed-stop rate (real interruption, no stop within 1 s) | 0 (target) | Bench clip set |
| M5 | Time to first audio after the user's turn ends | below 1 s browser, below 1.2 s phone (target) | Traces |
| M6 | Naive vs Ghostline on the same seeds | Ghostline clearly better on M1, with M3 and M4 shown | Dual-policy bench run |
| M7 | A new app scaffolded by the power compiles and passes its tests on the first run | Yes | Recorded in `docs/kiro-usage.md` |
| M8 | Submission complete | By Fri 4 Dec, 22:00 IST | Checklist in section 16 |

---

## 6. Users and scenarios

### Personas

| ID | Persona | Needs |
|---|---|---|
| P-DEV | Voice builder. A TypeScript developer who wants an interruptible voice agent | Clear interfaces, tests, a quick start, real numbers |
| P-CALLER | Basic-phone user. No smartphone or data plan. Speaks Hindi, Gujarati or English | Call, ask, hear a short answer, cut in when the answer is wrong |
| P-LISTENER | Someone trying Dead Air FM in a browser | No setup, one button, it reacts like a person |
| P-AGENT | Kiro (or another coding agent) working in the repo | Exact rules, exact API facts, small tasks, a way to test without spending credits |
| P-JUDGE | Hackathon judge, possibly helped by AI analysis | A working URL, a clear README, a readable `/.kiro`, claims backed by data |

### Scenarios

**S1 — Interrupt the radio host (P-LISTENER).**
1. Opens Dead Air FM and clicks "Go on air". The browser asks for the mic.
2. The host starts a short monologue.
3. The listener says "wait, play something calm".
4. The host's volume drops at once, then the host stops. The gauge shows the measured time.
5. The host greets the caller and answers. The unheard part of the old sentence is struck through in the transcript.

**S2 — A "hmm" does not stop it (P-LISTENER).** While the host talks, the listener says "hmm". The volume dips for a moment and returns. The host keeps talking.

**S3 — Ask the Oracle (P-CALLER).**
1. Calls the number (or opens the simulator and presses CALL).
2. Hears: "Namaste, I am an AI assistant. Press 1 for Hindi, 2 for Gujarati, 3 for English."
3. Presses 2 and asks a question in Gujarati.
4. Hears an answer of at most two short sentences in Gujarati.

**S4 — Interrupt the Oracle on a phone (P-CALLER).** The caller speaks over the answer. Ghostline tells the phone provider to clear its buffer, stops sending audio, and listens.

**S5 — Kiro builds a third app (P-DEV with P-AGENT).** The developer types "make a voice app that quizzes me on capitals". Kiro uses the power, calls `scaffold_voice_app`, runs the tests, and checks with `simulate_interruption` that a backchannel does not stop the bot and a real interruption does.

**S6 — A judge checks the claim (P-JUDGE).** Opens the dashboard and sees the naive and Ghostline distributions from real runs, with n and run IDs.

**S7 — Missed call, then callback (P-CALLER, stretch).** The caller rings once and hangs up. Within about a minute the Oracle calls back.

---

## 7. What we ship

| Component | Folder | What it is | Tier |
|---|---|---|---|
| Core engine | `skeleton/` | Pure logic: turn state machine, turn policy, barge-in, history, tracer, audio helpers | P0 |
| Runtime | `runtime-cf/` | Cloudflare Worker + one Durable Object per session; doorways | P0 (browser, simulator) / P1 (phone) |
| Web kit | `web-kit/` | Browser mic capture, playback queue, instant duck, console widgets | P0 |
| Adapters | `adapters/` | Workers AI (STT, TTS, LLM), Sarvam (STT, TTS) | P0 |
| | | ElevenLabs TTS, Sarvam LLM, Exotel, Twilio | P1 |
| | | Amazon Bedrock LLM | P2 |
| App 1 | `apps/dead-air-fm/` | Browser AI radio host | P0 |
| App 2 | `apps/missed-call-oracle/` | Phone Q&A with simulator | P0 (simulator) / P1 (real phone) / P2 (callback) |
| Bench | `bench/` | Fake caller, clip set, metrics, naive vs Ghostline | P0 (clips) / P1 (100 synthetic calls) |
| Kiro power | `power/` | POWER.md, steering, `ghostline-mcp`, hook installer | P1 (first in line) |
| Open data | `bench/hf/` | Hugging Face dataset and dashboard Space | P1 |
| Crew Exorcist | separate repo | Kiro Crew bot that runs a nightly check and files a report | P2 |

---

## 8. Functional requirements

Columns: **Pri** = priority. **Verified by** = the test or check that proves it.

### 8.1 Core engine (`skeleton/`)

| ID | Requirement | Pri | Verified by |
|---|---|---|---|
| FR-C1 | The engine has exactly four phases: `listening`, `thinking`, `speaking`, `interrupted`. | P0 | Unit test per transition |
| FR-C2 | The engine is a pure function: `(state, event, now, config) → (new state, actions)`. It makes no network, storage or clock calls. | P0 | Property P6 (same input, same output) |
| FR-C3 | While the bot speaks, user speech counts as an interruption only after it lasts `minSpeechMs` (default 120 ms, allowed 60–400 ms). | P0 | Unit: 119 ms no stop, 120 ms stop. Property P4 |
| FR-C4 | A backchannel (a short listener word such as "hmm", "haan", "haa") never stops the bot. Word lists exist for English, Hindi and Gujarati and can be changed in config. | P0 | Unit per language. Bench false-stop rate |
| FR-C5 | The user's turn ends after `eotSilenceMs` of silence (default 500 ms). An optional end-of-turn detector may end it earlier when its score is 0.5 or more. | P0 (silence) / P2 (detector) | Fake-clock unit tests |
| FR-C6 | On a confirmed interruption the engine emits these actions in this order: duck → stop playback → cancel LLM → cancel TTS → clear transport buffer → commit the spoken part. | P0 | Exact-order unit test |
| FR-C7 | After stop is issued for a turn, no more audio of that turn is sent. | P0 | Property P1 |
| FR-C8 | The engine waits for a "stopped" acknowledgement to record the time of the last audio frame. If none arrives within 500 ms it estimates the time and sets `is_estimated_stop = true`. | P0 | Unit: ack path and timeout path |
| FR-C9 | If confirmed user speech arrives while the bot is still thinking, the engine cancels the LLM and records outcome `cancelled`. | P0 | Unit |
| FR-C10 | History stores only reply text whose audio was played, cut at a word boundary. The unplayed rest is kept for display only and is never sent to the LLM. | P0 | Property P3 |
| FR-C11 | Every user turn ends in exactly one outcome: full reply, interrupted reply, or cancelled. | P0 | Property P2 |
| FR-C12 | Every turn produces exactly one `TurnTrace` (schema `ghostline.turn.v1`) with times from an injected monotonic clock. | P0 | Property P5 |
| FR-C13 | A `naive` policy ignores interruptions and plays every reply to the end. It exists only as the comparison baseline. | P0 | Unit |
| FR-C14 | Audio helpers: resample PCM16 between 8, 16 and 24 kHz; mu-law encode and decode. | P0 | Property P7 |
| FR-C15 | The package exposes its public API only through `skeleton/src/index.ts`. | P0 | Import-guard script |

### 8.2 Runtime and doorways (`runtime-cf/`)

| ID | Requirement | Pri | Verified by |
|---|---|---|---|
| FR-R1 | The Worker routes `WS /session/:id/<doorway>` to one Durable Object per session ID. Doorways: `browser`, `simulator`, `exotel`, `twilio`. | P0 (first two) / P1 | Integration test |
| FR-R2 | Browser audio in: PCM16, mono, 16 kHz, 20 ms frames, binary WebSocket. Audio out: PCM16 with the sample rate announced in an `audio_format` message. | P0 | Integration test |
| FR-R3 | Control messages. Server to client: `state`, `duck`, `unduck`, `stop`, `audio_format`, `trace`. Client to server: `duck`, `playback_progress`, `stopped {turnId, playedMs}`. | P0 | Contract test with a fake client |
| FR-R4 | A browser session needs a signed token that expires. `POST /session` creates it. | P0 | Unit: bad or expired token is refused |
| FR-R5 | The executor never waits on storage or network inside a per-audio-frame handler. Storage is written once per turn. | P0 | Code review checklist; latency traces |
| FR-R6 | STT, LLM, TTS and policy are chosen per session by config. | P0 | Unit |
| FR-R7 | When a provider fails, the session falls back to the next adapter in a fixed chain (see ARCHITECTURE §8). | P1 | Fixture test with a failing fake |
| FR-R8 | Each trace event is streamed to the client live. Each finished `TurnTrace` is stored. `GET /traces?session=` returns JSONL and needs a token. | P0 | Integration test |
| FR-R9 | Session limits: maximum length 20 minutes, idle timeout 3 minutes. Both are config values. The bot says goodbye before closing. | P0 | Fake-clock unit test |
| FR-R10 | On disconnect, all provider streams are cancelled and the trace is flushed. | P0 | Unit |
| FR-R11 | `GET /health` returns the version and git SHA. | P0 | Smoke test |

### 8.3 Web kit (`web-kit/`)

| ID | Requirement | Pri | Verified by |
|---|---|---|---|
| FR-W1 | Mic capture runs in an AudioWorklet and sends 20 ms PCM16 frames at 16 kHz. Browser echo cancellation is on. | P0 | Manual check + frame-size unit test |
| FR-W2 | The playback queue knows exactly how many milliseconds were played. On `stop` it empties the queue and sends `stopped {playedMs}`. | P0 | Unit with fake audio clock |
| FR-W3 | Instant duck: when local mic energy stays above a threshold for 40 ms, playback gain drops to 0.2 and a `duck` message is sent. Gain returns on `unduck`, or after 300 ms with no confirmation. | P0 | Unit with fake energy input |
| FR-W4 | Echo guard: while the bot speaks, an interruption needs both the server's speech detection and the client's energy detection. | P0 | Manual test with laptop speakers, no headphones |
| FR-W5 | Widgets: state lamps, waveform, latency gauge, dead-air meter, transcript. They read only trace and state messages. | P0 | Component tests |
| FR-W6 | The mic permission prompt appears only after the user clicks a clear button. | P0 | Manual check |

### 8.4 Provider adapters (`adapters/`)

| ID | Requirement | Pri | Verified by |
|---|---|---|---|
| FR-A1 | Workers AI adapters for STT, TTS and LLM. With these, the project runs with only a Cloudflare account and no other keys. | P0 | Fixture tests; live smoke test |
| FR-A2 | Sarvam streaming STT adapter for `hi-IN`, `gu-IN`, `en-IN`. It prefers the realtime endpoint that sends partial transcripts and can fall back to the older endpoint. | P0 | Fixture tests; one live test per language |
| FR-A3 | Sarvam Bulbul streaming TTS adapter. Text is sent in sentence-sized chunks. Cancel means: stop playback, close the socket, open a new socket for the next reply. | P0 | Fake-socket test: no audio after cancel |
| FR-A4 | ElevenLabs streaming TTS adapter for English voices. | P1 | Fixture test; live smoke test |
| FR-A5 | Sarvam LLM adapter (OpenAI-compatible). | P1 | Fixture test |
| FR-A6 | Amazon Bedrock LLM adapter. | P2 | Fixture test |
| FR-A7 | Every TTS adapter states truthfully whether it supports in-band cancel. | P0 | Type check + adapter test |
| FR-A8 | Adapters are tested against recorded fixtures. Live tests run only when `LIVE=1` is set. | P0 | CI runs without keys |

### 8.5 App 1 — Dead Air FM (`apps/dead-air-fm/`)

| ID | Requirement | Pri | Verified by |
|---|---|---|---|
| FR-F1 | The host speaks in segments of one or two sentences, in a fixed persona defined in `persona.ts`. | P0 | Prompt unit test |
| FR-F2 | When there is no bot audio and no user speech for 4 seconds, a new segment starts. | P0 | Fake-clock test |
| FR-F3 | A listener interrupts by speaking. There is no push-to-talk. | P0 | Manual + bench |
| FR-F4 | After an interruption, the next reply greets the listener as a caller and answers them. | P0 | Unit: prompt contains the caller flag |
| FR-F5 | Two seconds after answering, with no new speech, the monologue resumes. | P0 | Fake-clock test |
| FR-F6 | The console shows: state lamps, live waveform, barge-in gauge (last value and session median), dead-air meter, and transcript. | P0 | Component tests |
| FR-F7 | Text cut off by an interruption is shown struck through. | P0 | Component test |
| FR-F8 | On a duck, an orange (`state-heard`) border flashes for at most 150 ms. | P0 | Component test |
| FR-F9 | A NAIVE switch changes the policy so anyone can compare. | P0 | Manual |
| FR-F10 | A provider switcher changes the TTS voice provider. | P1 | Manual |

### 8.6 App 2 — Missed-Call Oracle (`apps/missed-call-oracle/`)

| ID | Requirement | Pri | Verified by |
|---|---|---|---|
| FR-O1 | The greeting says the caller is talking to an AI assistant. It is in Hindi by default and offers: 1 Hindi, 2 Gujarati, 3 English. | P0 | Unit |
| FR-O2 | A keypad digit 1–3 switches the STT and TTS language for the session. | P0 | Unit |
| FR-O3 | Answers are at most two short sentences, in the session language. | P0 | Prompt unit test |
| FR-O4 | Interruption works. On phone doorways the provider's `clear` message is sent and media sending stops. | P0 (simulator) / P1 (phone) | Unit with exact JSON shape |
| FR-O5 | A browser phone simulator (keypad, CALL, HANG UP) sends keypad digits and 8 kHz band-limited audio through the same Oracle logic. | P0 | Build + manual |
| FR-O6 | Version 1 has no live data. Asked for live facts (today's prices, weather), the Oracle says it cannot check live information yet. | P0 | Prompt test with 3 sample questions |
| FR-O7 | For health, legal or emergency questions the Oracle gives general information only and tells the caller to contact a doctor, a lawyer or the local emergency number. It never gives a diagnosis or a medicine dose. | P0 | Prompt test with 3 sample questions |
| FR-O8 | The console shows call status, language, phase, estimated barge-in time and transcript. | P0 | Component test |
| FR-O9 | Real phone calls through Exotel. Media is sent in 100 ms chunks that follow Exotel's size rules. | P1 | Property O1 + one recorded real call |
| FR-O10 | Twilio Media Streams as backup when Exotel is not enabled. | P1 | Fixture test + one real call |
| FR-O11 | A phone call ends politely after 5 minutes. | P1 | Fake-clock test |
| FR-O12 | Phone numbers are stored only as salted SHA-256 hashes. | P1 | Unit |
| FR-O13 | Missed-call callback: a valid webhook for caller N places one call back to N, at most once per 10 minutes per number. A bad token returns 403 and places no call. | P2 | Property O2; 403 test |

### 8.7 Kiro power and MCP server (`power/`)

| ID | Requirement | Pri | Verified by |
|---|---|---|---|
| FR-P1 | `power/POWER.md` follows the current Kiro power format and tells Kiro when to use each tool. | P1 | Front-matter lint script |
| FR-P2 | `simulate_interruption` runs the pure core with a synthetic scenario (backchannel, cough, real interruption, stop word, during thinking) and returns the action log and pass or fail. It makes no network calls. | P1 | Unit: deterministic output |
| FR-P3 | `get_turn_traces` returns recorded traces and a summary: n, barge-in p50 and p95, false stops, missed stops, time-to-first-audio p50. | P1 | Fixture test |
| FR-P4 | `scaffold_voice_app` creates `apps/<name>` from a template. The new app compiles and passes its tests. It never overwrites an existing folder. | P1 | Temp-folder build test |
| FR-P5 | `install-hooks` copies the hook files into `.kiro/hooks/` and does not overwrite without `--force`. | P1 | Temp-folder test |

If time is short, FR-P2 alone plus the steering files is the minimum power.

### 8.8 Bench (`bench/`)

| ID | Requirement | Pri | Verified by |
|---|---|---|---|
| FR-B1 | A fake caller connects like a browser, waits until the bot speaks, plays a recorded clip at a seeded random moment (0.5–3 s in), and records the trace. | P0 | One-clip integration test |
| FR-B2 | Metrics are calculated exactly as written in `docs/TESTING.md`. | P0 | Hand-computed fixture |
| FR-B3 | Every scenario runs under both policies (naive and Ghostline) with the same seeds. | P0 | Report shows both |
| FR-B4 | The clip set covers stop words, real interruptions, backchannels and noise in English, Hindi and Gujarati (13 clip types). Every recorded voice has written consent. | P0 | Manifest check |
| FR-B5 | Results are written as JSONL and Parquet with `run_id` and `git_sha`. | P0 (JSONL) / P1 (Parquet) | Schema check |
| FR-B6 | A synthetic caller runs 100 calls per policy and prints p50, p95 and n. | P1 | Dry run with n = 3 |

### 8.9 Open data on Hugging Face (`bench/hf/`)

| ID | Requirement | Pri | Verified by |
|---|---|---|---|
| FR-H1 | A public dataset holds the bench traces and the consented clips, with a full dataset card. | P1 | Dataset viewer loads |
| FR-H2 | A public Space shows naive vs Ghostline distributions and a metric table with n, using the same formulas as the bench. | P1 | Space loads; numbers match the bench report |
| FR-H3 | No real caller audio, no phone numbers, and no access tokens are published. | P1 | Pre-push checklist |

### 8.10 Crew Exorcist (separate repo)

| ID | Requirement | Pri | Verified by |
|---|---|---|---|
| FR-X1 | A Kiro Crew bot runs a small bench every night against the deployed URL and writes a report when a metric gets worse or a spec and its tests disagree. | P2 | One real nightly report |

### 8.11 Submission deliverables

| ID | Requirement | Pri | Verified by |
|---|---|---|---|
| FR-S1 | Public GitHub repo, MIT license visible in About, `/.kiro` committed. | P0 | Checklist |
| FR-S2 | A working public URL for both apps. | P0 | Smoke test from a second device |
| FR-S3 | README with quick start, both apps, measured results table, how Kiro was used, tools disclosure. | P0 | Review |
| FR-S4 | Demo video under 3 minutes, public on YouTube, no copyrighted music, showing naive vs Ghostline. | P0 | Timer + checklist |
| FR-S5 | Devpost write-up mapped to the five judging criteria. | P0 | Review |
| FR-S6 | `docs/kiro-usage.md` is complete and true. | P0 | Review |
| FR-S7 | Blog post and social post. | P1 | Links |

---

## 9. Non-functional requirements

### 9.1 Latency budget (targets, not results)

| Stage | Browser | Phone |
|---|---|---|
| Client duck | 30 ms or less | not possible |
| Confirming real speech | 120–200 ms | 120–250 ms |
| Stop command → last bot audio | 80 ms or less | 150 ms or less, including `clear` |
| **Barge-in total** | **200–300 ms** | **250–400 ms** |
| Time to first audio | under 1000 ms | under 1200 ms |
| False stops | under 5 % | under 5 % |
| Missed stops | 0 | 0 |

### 9.2 Other requirements

| Area | Requirement |
|---|---|
| Reliability | A failing provider or a dropped socket must never crash the Worker. Every outbound socket has a timeout. |
| Cost | Cost per call minute is calculated from real provider prices on the day of calculation and written in the README. Formula: STT minutes + TTS characters + LLM tokens + telephony minutes + Cloudflare duration, each times its rate. No remembered prices. |
| Privacy | No raw call audio is stored. Transcripts are deleted 60 seconds after hang-up unless the session is a bench run. Phone numbers are stored only as salted hashes. |
| Consent | Bench clips are recorded only with written consent for CC BY 4.0 release. |
| Security | Secrets live only in `wrangler secret` or a local untracked file. Twilio requests are checked by signature. Exotel stream and webhook URLs carry an HMAC token. Browser tokens expire. An agent hook blocks writes to secret files. |
| Accessibility | WCAG 2.2 AA. Everything works by keyboard. The transcript works as captions. State is never shown by color alone. Reduced-motion is respected. |
| Languages | Speech: `en-IN`, `hi-IN`, `gu-IN`. UI text is English; language names are shown in their own script. |
| Browsers | Latest Chrome on desktop and Android are P0. Safari and Firefox are untested (P2). |
| Observability | Traces and logs contain no raw audio and no full phone numbers. |
| Code quality | Strict TypeScript, no `any` in public types, tests in every package, CI runs without any API key. |
| License | Code: MIT. Dataset: CC BY 4.0. |

---

## 10. Experience requirements

Full visual rules are in `docs/DESIGN.md`. These are the product-level rules:

1. **First run in about a minute.** `pnpm i`, `pnpm dev`, open the URL, click one button, talk.
2. **Every spooky element is an instrument.** The orange flash means interruption detected. The dead-air meter is the silence budget. If an element measures nothing, remove it.
3. **Numbers always have units.** When nothing was measured yet, show `—`, never 0.
4. **The comparison is one click away.** The NAIVE switch is on the main screen.
5. **Errors are plain.** Example: "Line dropped (speech service). Reconnecting…"
6. **The simulator looks like a phone**, so the second app is clearly a different product from the first.

---

## 11. Data

- **TurnTrace** (one per turn): session and turn IDs, app, doorway, policy, providers, language, and timestamps in milliseconds for speech start, end of turn, first LLM token, first audio, interruption start, client duck, stop command, last audio frame; plus outcome, played milliseconds, and whether the stop time is estimated. The full schema is in ARCHITECTURE §6.
- **Reply text** is stored in traces only for bench runs or with consent.
- **Bench clips**: 16 kHz mono WAV, labelled `interrupt`, `stopword`, `backchannel` or `noise`.
- **Published**: bench traces and consented clips only.

---

## 12. Dependencies, assumptions and unverified facts

### Dependencies

| Dependency | Needed for | Lead time | Tier |
|---|---|---|---|
| Kiro credits (4,000) with ID check | All building | Up to 48 business hours. Request deadline 2 Dec, 01:30 IST | P0 |
| Cloudflare account | Runtime, Workers AI | Minutes | P0 |
| Sarvam API key | Hindi and Gujarati speech | Minutes | P0 |
| Hugging Face account and token | Dataset, Space | Minutes | P1 |
| ElevenLabs API key | English voices | Minutes | P1 |
| Exotel account, KYC, Voicebot enablement | Real Indian phone calls | Unknown; could be days. Start on day 1 | P1 |
| Twilio trial | Backup phone number | Minutes to hours | P1 |

### Assumptions

- A1 The Sarvam credits can be spent on the raw STT and TTS APIs, not only on the Voice Agents platform.
- A2 Using other tools beside Kiro for research and documents is allowed when disclosed.
- A3 Hosting on Cloudflare is allowed. Nothing in the rules read so far forbids it.
- A4 A Durable Object can hold one inbound WebSocket and two or three outbound provider WebSockets for a whole call with acceptable delay. The day-1 spike tests this.

### Unverified facts (check before code depends on them)

| ID | Fact to check | Where |
|---|---|---|
| U1 | Sarvam realtime STT event names and query parameters | docs.sarvam.ai realtime STT reference |
| U2 | The sample-rate field in the Sarvam TTS WebSocket config | docs.sarvam.ai TTS streaming reference |
| U3 | Sarvam LLM endpoint URL and model IDs | docs.sarvam.ai chat completions |
| U4 | Assumption A1 | Sarvam dashboard; one live API call |
| U5 | Exotel: support email, enablement time, 16 kHz on our account, any India data-residency rule for the bot server, whether a webhook fires on an unanswered call | Exotel support reply; one test call |
| U6 | Twilio trial limits when calling from India | Twilio console |
| U7 | Kiro: exact hook file format and trigger names, whether a power can ship hooks, the key for remote MCP servers | kiro.dev/docs; create one hook in the Kiro UI and compare |
| U8 | Current Workers AI model names for STT, TTS, LLM; the Cloudflare docs MCP URL | developers.cloudflare.com |
| U9 | Hackathon: assumptions A2 and A3, whether more than one entry per person is allowed, what counts as a Kiro Crew app | Rules, FAQ, Kiro Discord `#kiroween-hackathon` |

---

## 13. Scope tiers, hours and cut order

Hours are estimates by the author, not measurements.

| Tier | Work | Hours |
|---|---|---|
| P0 | Setup, planning docs, steering, running specs through Kiro | 5 |
| P0 | Spike: audio round trip on Cloudflare | 2 |
| P0 | Core engine and property tests | 14 |
| P0 | Runtime: Worker, session object, browser doorway | 7 |
| P0 | Web kit: capture, playback, duck, widgets | 7 |
| P0 | Workers AI adapters | 4 |
| P0 | Sarvam STT and TTS adapters | 6 |
| P0 | Dead Air FM | 6 |
| P0 | Simulator doorway and Oracle | 5 |
| P0 | Bench (clips), naive baseline, real numbers | 5 |
| P0 | Deploy, README, usage log, video, Devpost | 8 |
| | **P0 total** | **69** |
| P1 | Kiro power and `ghostline-mcp` | 5 |
| P1 | Real phone call (Exotel, or Twilio backup) | 6 |
| P1 | Hugging Face dataset and Space | 4 |
| P1 | ElevenLabs adapter | 2 |
| P1 | Synthetic caller, 100 calls | 2 |
| P1 | Blog and social posts | 2 |
| | **P1 total** | **21** |
| P2 | Missed-call callback | 3 |
| P2 | Crew Exorcist | 4 |
| P2 | End-of-turn detector (Smart Turn) | 2 |
| P2 | Bedrock adapter, one live-data tool | 2 |
| | **P2 total** | **11** |

**Reading the table.** P0 alone is about 69 hours, close to 5 hours a day for 15 days. P0 plus the Kiro power is about 74 hours. All of P1 needs about 90 hours, or 6 hours a day. P2 happens only if P0 and P1 finish early.

**P1 order:** Kiro power → real phone call → Hugging Face → ElevenLabs → synthetic caller → blog and social.

**Cut order when late (cut from the top):**
1. Bedrock, live-data tool, end-of-turn detector
2. Crew Exorcist
3. Missed-call callback
4. ElevenLabs adapter
5. Synthetic 100-call run (keep the clip runs)
6. Hugging Face Space (keep the dataset)
7. Real phone call (use the simulator in the video)
8. Power reduced to steering files plus `simulate_interruption`

**Never cut:** the core, the browser doorway, both apps, the bench numbers, the video.

---

## 14. Schedule and gates

Rebased on Tue 24 Nov: setup slipped from 19–20 Nov, scope is unchanged, so each day is longer (about 9–10 hours). If a day slips, the next P2 item moves to Thu 3 Dec; the video and write-up never move.

| Day | Work | Issues (section 17) |
|---|---|---|
| Tue 24 Nov | Repo, planning docs, steering, spec drafts, Kiro usage log. Exotel status check; Twilio trial if needed | #1–#5 |
| Wed 25 Nov | **Gate A.** Spike. Core spec in Kiro; start the core | #6, #7, #8 |
| Thu 26 Nov | Barge-in, history, tracer, naive policy, audio helpers, property tests, hooks. Runtime and adapter specs | #9–#12 |
| Fri 27 Nov | Worker, session object, browser doorway, web kit, Workers AI adapters. **Gate B (end of day)** | #13–#16 |
| Sat 28 Nov | Sarvam and ElevenLabs adapters, provider switcher. Dead Air FM loop and persona | #17, #18, #24 |
| Sun 29 Nov | Dead Air FM console. Exotel adapter and one real call (or Twilio) | #19, #22, #23 |
| Mon 30 Nov | Simulator doorway, Oracle logic and console. Missed-call callback | #20, #21, #25 |
| Tue 1 Dec | Record clips. Bench, synthetic 100 calls, real numbers. Hugging Face dataset and Space | #26–#28, #30 |
| Wed 2 Dec | Kiro power, MCP server, hook installer. Deploy and smoke test. Docs site | #29, #32 |
| Thu 3 Dec | Crew Exorcist. UI polish. README, usage log. Record the video | #31, #33–#35 |
| Fri 4 Dec | Edit video, Devpost write-up, blog and social. **Submit by 22:00 IST** | #35–#37 |

### Gate A — Wed 25 Nov, end of day: "Can Cloudflare carry the audio?"

**Pass:** the spike takes speech from the browser mic, sends it through a Durable Object to STT, an LLM and TTS, and plays the reply in the browser.
**Fail:** use Cloudflare's `agents/voice` for the audio transport and keep the Ghostline engine on top. If that also fails by Thursday night, go to the fallback below.

### Gate B — Fri 27 Nov, end of day (may slip to noon 28 Nov): "Does interruption work?"

**Pass:** in the browser, talking over the bot stops it, and a `TurnTrace` with a real `last_audio_frame` time is produced.
**Phone decision at the same time:** go for a real phone call only if Exotel has confirmed enablement or a Twilio trial number already works. Otherwise the simulator is the only phone doorway, and the freed hours go to bench quality and the video.

### Fallback if Gate B fails

1. Cut to: browser only, English only, Workers AI only. Keep both apps.
2. If interruption still does not work by Sun 29 Nov, switch to the backup project **Spec Poltergeist** (a Kiro tool that detects stale specs; it needs no audio).

---

## 15. Risks

| # | Risk | Chance | Effect | What we do |
|---|---|---|---|---|
| R1 | Cloudflare voice features are Beta and may change or misbehave | Medium | High | The core does not depend on them. Spike on day 1. Pin package versions |
| R2 | Exotel KYC or enablement is slow | High | Medium | Email on day 1. Twilio backup. Simulator always works |
| R3 | 8 kHz phone audio hurts speech recognition | Medium | Medium | Ask Exotel for 16 kHz. Demo in the language that tests best |
| R4 | Kiro credits run out | Medium | High | One task per request. Short steering files. Never regenerate a whole spec. Log the balance daily |
| R5 | The bot hears itself through laptop speakers and interrupts itself | High | High | Echo cancellation on. Echo guard (FR-W4). Test without headphones early |
| R6 | Hindi or Gujarati recognition is slow or wrong | Medium | Medium | Test both on day 5. Demo with the better one |
| R7 | Time runs out | High | High | Tiers and cut order (section 13). Two gates. Friday is buffer only |
| R8 | A provider API fact in the docs is wrong | Medium | Medium | Unverified list (section 12). One live test per adapter before building on it |
| R9 | A claim in the submission is not backed by data | Low | High | Truth rules (section 2.3). All numbers carry a `run_id` |
| R10 | Too little visible Kiro use | Low | Very high | All code through specs and tasks. Hooks really run. Usage log kept daily |
| R11 | The Oracle gives harmful advice to a caller | Low | High | FR-O6 and FR-O7. The demo number is not advertised |

---

## 16. Acceptance

### Definition of done for any task

- Its test passes and the full test suite is green.
- The import guard is clean.
- No provider field is used that is missing from `providers.md`.
- The trace schema is unchanged, or a decision record was added.
- One true row was added to `docs/kiro-usage.md`.

### Definition of done for the product (P0)

- [ ] Talking over the Dead Air FM host stops it, and the gauge shows a measured time
- [ ] A "hmm" or a cough does not stop it in most trials (rate measured)
- [ ] The Oracle answers in Hindi and Gujarati on the simulator and can be interrupted
- [ ] The NAIVE switch shows the difference live
- [ ] The bench produced naive and Ghostline numbers with n and a `run_id`
- [ ] Both apps are live at a public URL and work from a second device

### Submission checklist

- [ ] Public repo, MIT in About, `/.kiro` committed, nothing secret in history
- [ ] README complete, results table filled from a real run, tools disclosure written
- [ ] Video under 3 minutes, public, no copyrighted music
- [ ] Devpost write-up mapped to the five criteria
- [ ] `docs/kiro-usage.md` reviewed
- [ ] Student and Startup fields filled
- [ ] Submitted before Fri 4 Dec, 22:00 IST

---

## 17. Work plan: specs and GitHub issues

### 17.1 Specs (built by Kiro's spec mode from drafts)

| # | Spec | Covers | Approx. tasks |
|---|---|---|---|
| 1 | `ghostline-core` | FR-C1 – FR-C15 | 11 |
| 2 | `ghostline-runtime` | FR-R1 – FR-R11, FR-W1 – FR-W6 | 8 |
| 3 | `ghostline-adapters` | FR-A1 – FR-A8 | 7 |
| 4 | `dead-air-fm` | FR-F1 – FR-F10 | 6 |
| 5 | `missed-call-oracle` | FR-O1 – FR-O13 | 7 |
| 6 | `barge-in-bench` | FR-B1 – FR-B6, FR-H1 – FR-H3 | 6 |
| 7 | `ghostline-power-mcp` | FR-P1 – FR-P5 | 6 |

About **50 Kiro tasks** in total. Drafts exist for specs 1, 4, 5, 6 and 7. **Drafts for specs 2 and 3 still need to be written.**

### 17.2 GitHub issues

**Rule:** one issue is one work package of 2 to 8 hours that ends in something you can test or show. Kiro tasks stay in `tasks.md`. Do not create one issue per Kiro task.

**Totals: 37 issues in 6 milestones.** 29 are P0, 6 are P1, 2 are P2. Kiro leads 15, Harsh leads 12, 10 are shared.

**Owner key:** **Kiro** = Kiro writes the code from spec tasks and Harsh reviews. **Me** = Harsh does it by hand. **Both** = Harsh decides or tests, Kiro builds.

#### Milestone M0 — Setup (due Wed 25 Nov)

| # | Issue | Pri | Owner |
|---|---|---|---|
| 1 | Accounts and keys: Kiro credits and ID check, Cloudflare, Sarvam key (one live test call), ElevenLabs key, Hugging Face token | P0 | Me |
| 2 | Exotel sign-up, KYC and enablement email; Twilio trial as backup | P0 | Me |
| 3 | Ask the organizers the open rule questions in Discord (U9) | P0 | Me |
| 4 | Repo bootstrap: license, workspace, TypeScript config, CI that runs tests without keys | P0 | Kiro |
| 5 | Add planning docs, `AGENTS.md` and steering files; spec drafts into `docs/drafts/` | P0 | Me |
| 6 | Spike: audio round trip on Cloudflare (Gate A) | P0 | Both |

#### Milestone M1 — Core (due Thu 26 Nov)

| # | Issue | Pri | Owner |
|---|---|---|---|
| 7 | `ghostline-core` spec through Kiro spec mode | P0 | Both |
| 8 | State machine and turn policy (FR-C1 – C5) | P0 | Kiro |
| 9 | Barge-in controller and spoken-only history (FR-C6 – C11) | P0 | Kiro |
| 10 | Tracer, naive policy, audio helpers (FR-C12 – C15) | P0 | Kiro |
| 11 | Property tests P1–P7 and hooks running on save | P0 | Kiro |

#### Milestone M2 — Browser end to end (due Fri 27 Nov)

| # | Issue | Pri | Owner |
|---|---|---|---|
| 12 | `ghostline-runtime` and `ghostline-adapters` specs through Kiro spec mode | P0 | Both |
| 13 | Worker router, session object, browser doorway (FR-R1 – R11) | P0 | Kiro |
| 14 | Web kit: capture, playback queue, instant duck, echo guard (FR-W1 – W4, W6) | P0 | Kiro |
| 15 | Workers AI adapters: STT, TTS, LLM (FR-A1) | P0 | Kiro |
| 16 | First end-to-end interruption with a trace; write the go / no-go note (Gate B) | P0 | Me |

#### Milestone M3 — Apps (due Mon 30 Nov)

| # | Issue | Pri | Owner |
|---|---|---|---|
| 17 | Sarvam STT and TTS adapters; live check of U1, U2, U4 | P0 | Both |
| 18 | Dead Air FM: spec, radio loop, persona (FR-F1 – F5) | P0 | Kiro |
| 19 | Dead Air FM: console (FR-F6 – F9, FR-W5) | P0 | Both |
| 20 | Simulator doorway and Oracle logic (FR-O1 – O7) | P0 | Kiro |
| 21 | Oracle simulator console (FR-O5, O8) | P0 | Both |
| 22 | Exotel adapter and one real phone call (FR-O9, O11, O12) | P1 | Both |
| 23 | Twilio backup adapter (FR-O10) — only if Exotel is not enabled | P1 | Kiro |
| 24 | ElevenLabs TTS adapter and provider switcher (FR-A4, FR-F10) | P1 | Kiro |
| 25 | Missed-call callback (FR-O13) | P2 | Kiro |

#### Milestone M4 — Proof (due Tue 1 Dec)

| # | Issue | Pri | Owner |
|---|---|---|---|
| 26 | Record the clip set and collect written consent (FR-B4) | P0 | Me |
| 27 | Bench: fake caller, metrics, dual-policy runner, export (FR-B1 – B3, B5; B6 as P1 checkbox) | P0 | Kiro |
| 28 | Run the bench and record the real numbers | P0 | Me |
| 29 | Kiro power, `ghostline-mcp`, hook installer (FR-P1 – P5) | P1 | Kiro |
| 30 | Hugging Face dataset and Space (FR-H1 – H3) | P1 | Both |
| 31 | Crew Exorcist in its own repo (FR-X1) | P2 | Both |

#### Milestone M5 — Ship (due Fri 4 Dec)

| # | Issue | Pri | Owner |
|---|---|---|---|
| 32 | Deploy and smoke-test on a laptop and a phone (FR-S2) | P0 | Me |
| 33 | UI polish and accessibility pass | P0 | Both |
| 34 | README with measured results, tools disclosure, usage-log review (FR-S3, S6) | P0 | Me |
| 35 | Demo video: script, record, edit, upload (FR-S4) | P0 | Me |
| 36 | Devpost submission: write-up, screenshots, Student and Startup fields (FR-S1, S5) | P0 | Me |
| 37 | Blog post and social post (FR-S7) | P1 | Me |

### 17.3 How issues, specs and commits connect

- **Labels:** `p0`, `p1`, `p2`; `kiro`, `me`, `both`; `gate`; one area label (`core`, `runtime`, `adapters`, `app-fm`, `app-oracle`, `bench`, `power`, `docs`).
- **Issue body:** goal in one line, requirement IDs, the spec tasks it covers, and a "done when" checklist.
- **Commits:** one file per commit with a clear message that names the issue, for example `feat(core): add turn policy guard (refs #8)`. The last commit of an issue says `closes #8`.
- **Order:** work on one issue at a time. Close it before opening the next, except while waiting on an outside party (Exotel).

---

## 18. Decisions and open questions

### Decisions already made

| # | Decision | Reason |
|---|---|---|
| D1 | Category: Skeleton Crew | The 2025 top prize went to a skeleton that made Kiro reliable in a hard domain |
| D2 | Main template: Ghostline (voice), not an offline-sync kit or an any-channel kit | Empty lane among public 2026 entries; memorable live demo; fits the builder's experience |
| D3 | App 2 is Missed-Call Oracle, replacing an earlier booking-line idea | Stronger impact story; proves the kit works on a phone |
| D4 | One submission; Crew Exorcist is an add-on repo | Avoids duplicate entries and split effort |
| D5 | The core pipeline is our own; Cloudflare's voice helper is optional | Beta status; we need behaviors it does not have |
| D6 | Sarvam and ElevenLabs are used as raw speech APIs only | Their managed agent platforms would replace the product |
| D7 | Only measured numbers are claimed | Trust; judges can check the data |
| D8 | Version 1 of the Oracle has no live data and gives no medical or legal advice | Safety and scope |

### Open questions

| # | Question | Owner | Needed by |
|---|---|---|---|
| Q1 | Do the hackathon rules allow planning documents written outside Kiro and other tools beside Kiro? | Harsh (Discord) | 20 Nov |
| Q2 | Is more than one entry per person allowed? What counts as a Kiro Crew app? | Harsh (Discord) | 30 Nov |
| Q3 | Does Exotel give 16 kHz audio and allow a bot server on Cloudflare? | Harsh (Exotel email) | 24 Nov |
| Q4 | Which language tests better on phone audio, Hindi or Gujarati? | Bench | 28 Nov |
| Q5 | Are the backchannel word lists enough, or is an end-of-turn model needed? | Bench | 29 Nov |
| Q6 | What is the real cost per call minute? | Harsh | 1 Dec |
| Q7 | What is the host's name and voice in Dead Air FM? | Harsh | 25 Nov |

---

## 19. Glossary

| Term | Meaning |
|---|---|
| Barge-in | The user starts talking while the bot is talking. The bot must stop |
| Backchannel | A short listener sound ("hmm", "haan") that is not an attempt to take the turn |
| VAD | Voice activity detection: deciding whether a slice of audio is speech |
| End of turn | The moment the system decides the user has finished speaking |
| Time to first audio | Time from end of the user's turn to the first bot audio |
| Duck | Lowering the bot's volume at once when the user might be speaking |
| Doorway | A way into the system: browser, simulator, Exotel, Twilio |
| Turn commit | Saving a turn into the conversation history |
| Spoken part | The part of a reply whose audio was actually played |
| False stop | The bot stopped although the user did not mean to interrupt |
| Missed stop | The user interrupted and the bot did not stop within one second |
| Naive policy | A pipeline with no interruption handling, used as the baseline |
| TurnTrace | The record of one turn, with all its timestamps |
| Steering | Kiro's always-available project rules in `.kiro/steering/` |
| Spec | Kiro's requirements, design and task files for one feature |
| Power | A Kiro package of steering and tools that teaches Kiro a domain |

---

## Appendix A — Where each judging criterion is earned

| Criterion (20 % each) | Earned by |
|---|---|
| Potential Impact | The Oracle for basic-phone users in Indian languages; an open benchmark and kit for every voice builder |
| Technological Implementation | Seven specs, steering with inclusion modes, hooks that really run, property tests, the MCP server and power, the Crew bot; all logged in `docs/kiro-usage.md` |
| Design | A complete console where every element measures something; two apps that look and behave differently; a simulator that always works |
| Innovation | Interruption treated as a tested contract; client duck with server confirmation; spoken-only history; open barge-in data |
| Presentation | A video that opens with naive vs Ghostline side by side; a README with measured results; a public dashboard |
