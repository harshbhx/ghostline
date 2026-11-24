# Ghostline Testing and Bench Method

| | |
|---|---|
| Repo path | `docs/TESTING.md` |
| Decides | Property tests, the bench procedure and every metric formula |
| Status | Draft v1 · 24 Nov 2026. Written outside Kiro as input to spec mode |

**The one rule:** a number is a result only if the bench produced it with a `run_id` and `git_sha`. Everything else is a target or `—`.

---

## 1. Test layers

| Layer | Tool | Runs in CI | Needs keys |
|---|---|---|---|
| Unit | Vitest | yes | no |
| Property | Vitest + fast-check | yes | no |
| Contract (protocol) | Vitest with a fake client | yes | no |
| Adapter fixtures | Vitest with recorded fixtures | yes | no |
| Live adapter | Vitest, `LIVE=1` | no | yes |
| Bench | `bench/` runner against a deployed or local URL | no | Workers AI at least |

Fake clock everywhere in the core: tests pass `now` explicitly. No test may sleep.

## 2. Property tests (core)

Each property runs at least 200 generated cases. A failing seed is copied into a regression unit test.

| ID | Property | Generator |
|---|---|---|
| P1 | After `stop_playback` for turn T, no `send_audio` for T is ever emitted (FR-C7) | Random event sequences with interruptions at random times |
| P2 | Every user turn ends in exactly one outcome: `full`, `interrupted` or `cancelled` (FR-C11) | Random sessions of 1–20 turns |
| P3 | Committed history contains only text whose audio was played, cut at a word boundary; unplayed text never appears in any later `start_llm` messages (FR-C10) | Random replies, random `playedMs` |
| P4 | Speech shorter than `minSpeechMs` never stops the bot; speech of `minSpeechMs` or longer that is not a backchannel always does (ghostline policy) (FR-C3) | Random durations around the threshold, random config in 60–400 |
| P5 | Every turn produces exactly one `TurnTrace`, and its times are non-decreasing in the order speechStart ≤ endOfTurn ≤ llmFirstToken ≤ firstAudio where present (FR-C12) | Random sessions |
| P6 | `step` is deterministic: same state, event, now and config give the same result (FR-C2) | Random states and events, run twice |
| P7 | Resampling 8↔16↔24 kHz preserves length within one sample per conversion; mu-law encode then decode stays within the mu-law quantisation error (FR-C14) | Random PCM16 buffers |
| O1 | Exotel outbound chunks are always 3200 bytes to 100 KB and a multiple of 320 bytes, and no audio is lost or duplicated by chunking (FR-O9) | Random audio lengths |
| O2 | Callback: at most one call per number per 10 minutes; a bad token never places a call (FR-O13) | Random webhook sequences |

Exact-order test (FR-C6): on a confirmed interruption the action list equals `[duck, stop_playback, cancel_llm, cancel_tts, clear_transport, commit_turn]` for that turn.

## 3. Clip set (FR-B4)

16 kHz mono WAV, each with a JSON sidecar: `{id, type, language, speechOnsetMs, speaker, consent: true}`.

| Type | Languages | Example | Expected (ghostline) |
|---|---|---|---|
| `interrupt` | en, hi, gu | "Wait, which one?" | stop |
| `stopword` | en, hi, gu | "Stop." / "रुको" / "ઊભા રહો" | stop |
| `backchannel` | en, hi, gu | "hmm", "haan", "accha" | no stop |
| `noise` | — | cough, door, keyboard, TV | no stop |

That is 13 clip types (4 types × 3 languages, plus noise). Aim for at least 5 clips per type. Every recorded voice has written consent for CC BY 4.0 release; the consent form lives outside the repo, and the manifest records `consent: true` only after it is signed.

## 4. Bench procedure (FR-B1 – FR-B3)

1. Start a session as a fake browser client (same protocol as `web-kit`).
2. Wait until the bot is speaking (`state` = `speaking`).
3. Wait a seeded random delay of 0.5–3.0 s, then stream one clip in real time (20 ms frames).
4. Keep reading until the turn's trace is finished, or 5 s pass.
5. Record the trace plus `{clipId, clipType, language, seed, policy, runId, gitSha}`.
6. Run every clip under **both** policies with the **same seeds**.

Run IDs: `run-YYYYMMDD-HHMM-<shortsha>`. Output: `bench/out/<runId>.jsonl` (P0), Parquet (P1).

## 5. Metric formulas

Let `onset` = the moment the clip's speech starts on the server clock: `clipStart + speechOnsetMs`.

| Metric | Formula | Over which clips |
|---|---|---|
| Barge-in latency | `lastAudioFrame − onset` | `interrupt`, `stopword` that stopped |
| Client duck latency | `clientDuck − onset` | same, browser only |
| Stop latency | `lastAudioFrame − stopCommand` | same |
| False stop rate | stops ÷ clips | `backchannel`, `noise` |
| Missed stop | no `lastAudioFrame` within 1000 ms of `onset` | `interrupt`, `stopword` |
| Missed stop rate | missed ÷ clips | `interrupt`, `stopword` |
| Time to first audio | `firstAudio − endOfTurn` | every turn with a reply |

For the naive policy, "barge-in latency" is the time until the reply ends on its own; it is reported in the same column so the comparison is honest.

**Percentiles:** nearest-rank method. p50 = value at rank ⌈0.5·n⌉ of the sorted list; p95 at ⌈0.95·n⌉. Always print `n` beside every percentile. With n < 20, do not print p95; print `—`.

**Estimated stops:** traces with `isEstimatedStop = true` are counted, but reported separately as "estimated", never mixed into the measured browser numbers. Phone barge-in is always labelled estimated unless the provider sent a mark.

## 6. Reporting

- The bench prints one table per doorway: metric, naive, ghostline, n.
- The README results table is copied from the latest run, with the `runId` beside it.
- A hand-computed fixture (5 traces) checks every formula (FR-B2).
- Synthetic caller (P1, FR-B6): 100 calls per policy, prints p50, p95 and n.

## 7. Manual checks before submitting

- Laptop speakers, no headphones: the bot does not interrupt itself (FR-W4).
- Keyboard only: every control reachable, focus visible.
- Reduced motion on: no movement in either app.
- Second device: both public URLs work (FR-S2).
