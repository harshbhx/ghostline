# Ghostline Architecture

| | |
|---|---|
| Repo path | `docs/ARCHITECTURE.md` |
| Decides | How the parts connect: packages, interfaces, wire protocol, trace schema, fallback chains |
| Does not decide | Scope and priority (`docs/PRD.md`), metric formulas (`docs/TESTING.md`), visuals (`docs/DESIGN.md`) |
| Status | Draft v1 · 24 Nov 2026. Written outside Kiro as input to spec mode. When a Kiro spec's `design.md` refines an interface, update this file to match |

Provider facts (endpoints, message shapes, audio formats) are in `.kiro/steering/providers.md`, checked against official docs on 24 Nov 2026.

---

## 1. Overview

```
 Browser (web-kit)              Cloudflare Worker                 Providers
 ───────────────────            ─────────────────                 ─────────
 mic → AudioWorklet ──PCM16──▶  router ─▶ Session DO (one per id)
 16 kHz, 20 ms frames           │          ├─ Executor ───────────▶ STT  (Workers AI | Sarvam)
 playback queue ◀──PCM16──────  │          │   runs the pure core   LLM  (Workers AI | Sarvam | Bedrock)
 instant duck ──"duck"───────▶  │          │                        TTS  (Workers AI | Sarvam | ElevenLabs)
                                │          └─ Trace sink ─▶ DO storage (once per turn)
 Phone (Exotel | Twilio) ◀────▶ │ phone doorways: same Session DO, different framing
 Simulator (browser phone) ◀──▶ │ simulator doorway: 8 kHz band-limited audio + keypad
```

The **core** (`skeleton/`) is pure. It decides; it never does I/O. The **executor** in the Durable Object turns the core's actions into real effects (send audio, cancel a stream, clear a phone buffer) and feeds results back to the core as events.

## 2. Packages and import rules

| Package | Path | May import | Must not import |
|---|---|---|---|
| Core | `skeleton/` | nothing outside itself | any runtime, network or clock API |
| Adapters | `adapters/` | `skeleton` (types only) | `runtime-cf`, apps |
| Runtime | `runtime-cf/` | `skeleton`, `adapters` | apps |
| Web kit | `web-kit/` | `skeleton` (types only) | `runtime-cf`, adapters |
| Apps | `apps/<name>/` | `skeleton`, `adapters`, `runtime-cf`, `web-kit` | other apps |
| Bench | `bench/` | everything | — |
| Power | `power/` | `skeleton` | network at test time |

Public API of each package is its `src/index.ts` only (FR-C15). A small import-guard script enforces this table in CI.

Tooling: TypeScript strict, pnpm workspaces, Vitest, fast-check, Wrangler. No `any` in public types.

## 3. Core engine

### 3.1 Shape

```ts
type Phase = "listening" | "thinking" | "speaking" | "interrupted";      // FR-C1

function step(state: EngineState, event: EngineEvent, now: number, config: EngineConfig)
  : { state: EngineState; actions: Action[] };                            // FR-C2, pure
```

`now` comes from an injected monotonic clock in milliseconds. The core never reads a clock itself.

### 3.2 Events (into the core)

| Event | From | Meaning |
|---|---|---|
| `vad_start` / `vad_end` | STT adapter or server VAD | Server thinks speech started / stopped |
| `client_duck` | browser | Client energy crossed the duck threshold (FR-W3) |
| `partial_transcript` / `final_transcript` | STT | Text, with language |
| `llm_token` / `llm_done` / `llm_error` | LLM | Streaming reply |
| `tts_audio` / `tts_done` / `tts_error` | TTS | Audio chunk with duration in ms |
| `playback_progress` | client or phone marks | Milliseconds actually played for a turn |
| `stopped` | client or phone | Playback ended after a stop; carries `playedMs` (FR-C8) |
| `timer` | executor | A timer the core asked for fired |
| `dtmf` | phone or simulator | Keypad digit |
| `hangup` | doorway | Session ends |

### 3.3 Actions (out of the core)

| Action | Executor does |
|---|---|
| `duck` / `unduck` | Tell the client to lower / restore gain |
| `stop_playback {turnId}` | Tell the client to stop and flush; or stop sending media on a phone |
| `cancel_llm {turnId}` / `cancel_tts {turnId}` | Abort the stream (per adapter rules) |
| `clear_transport {turnId}` | Phone `clear` message; no-op in the browser |
| `commit_turn {turnId, spokenText, outcome}` | Append to history (spoken part only, FR-C10) |
| `start_llm {turnId, messages}` / `speak {turnId, text}` | Start a stream |
| `set_timer {name, at}` / `clear_timer {name}` | Executor schedules `timer` events |
| `emit_trace {event}` / `finish_trace {trace}` | Stream to client; store once per turn |
| `send_audio {turnId, chunk}` | Forward TTS audio to the doorway |

On a confirmed interruption the actions are emitted in exactly this order (FR-C6): `duck` → `stop_playback` → `cancel_llm` → `cancel_tts` → `clear_transport` → `commit_turn`.

### 3.4 Turn policy

- `ghostline` (default): interruption needs `minSpeechMs` of speech (default 120, range 60–400, FR-C3), is not a backchannel (FR-C4), and in the browser while the bot speaks also needs the client's duck (echo guard, FR-W4).
- `naive` (baseline only, FR-C13): never interrupts; plays every reply to the end.

Backchannel lists live in config per language (`en`, `hi`, `gu`), for example `hmm`, `haan`, `haa`, `ok`, `accha`.

### 3.5 Config

```ts
interface EngineConfig {
  policy: "ghostline" | "naive";
  minSpeechMs: number;          // 120
  eotSilenceMs: number;         // 500 (FR-C5)
  eotDetector?: { threshold: number };   // P2, 0.5
  stopAckTimeoutMs: number;     // 500 (FR-C8)
  backchannels: Record<"en" | "hi" | "gu", string[]>;
}
```

## 4. Runtime (`runtime-cf/`)

| Route | What |
|---|---|
| `POST /session` | Create a session; returns `{id, token, expiresAt}` (FR-R4) |
| `WS /session/:id/browser` | Browser doorway (token in the `Sec-WebSocket-Protocol` header or a query param) |
| `WS /session/:id/simulator` | Phone simulator doorway |
| `WS /session/:id/exotel` | Exotel AgentStream (HMAC token in the URL) |
| `WS /session/:id/twilio` | Twilio Media Streams (signature checked on the TwiML webhook) |
| `POST /twilio/voice`, `POST /exotel/*` | Telephony webhooks that start a stream |
| `GET /traces?session=` | JSONL of finished traces; needs a token (FR-R8) |
| `GET /health` | `{version, gitSha}` (FR-R11) |

**Session Durable Object.** One per session ID. Holds the inbound socket, outbound provider sockets, the engine state, timers and the trace buffer.

**Hot path rule (FR-R5).** Inside a per-audio-frame handler the executor never awaits storage or a network round trip. Storage is written once per turn (`finish_trace`). Outbound sends are fire-and-forget with their own error handling.

**Limits (FR-R9).** Session max 20 minutes, idle 3 minutes, phone call 5 minutes (FR-O11). The bot says goodbye before closing. On disconnect every provider stream is cancelled and the trace is flushed (FR-R10).

## 5. Browser protocol

Binary frames are audio. Text frames are JSON control messages with a `type`.

| Direction | Audio |
|---|---|
| client → server | PCM16 LE, mono, 16 kHz, 20 ms per frame (640 bytes) (FR-R2) |
| server → client | PCM16 LE, mono, at the rate announced in `audio_format` |

Every server audio frame belongs to the current `turnId`; the client drops frames for a turn it has been told to stop.

| Type | Direction | Body |
|---|---|---|
| `audio_format` | S→C | `{sampleRate, channels: 1, encoding: "pcm16"}` |
| `state` | S→C | `{phase, turnId}` |
| `duck` / `unduck` | S→C | `{turnId}` |
| `stop` | S→C | `{turnId}` |
| `trace` | S→C | one trace event `{turnId, name, t}` |
| `transcript` | S→C | `{turnId, role, text, spokenUntil?, final}` |
| `duck` | C→S | `{turnId, t}` (client energy duck) |
| `playback_progress` | C→S | `{turnId, playedMs}` every 100 ms while playing |
| `stopped` | C→S | `{turnId, playedMs}` (FR-W2) |
| `dtmf` | C→S | `{digit}` (simulator only) |

## 6. TurnTrace schema (`ghostline.turn.v1`)

One per user turn (FR-C12). All times are milliseconds from the session's monotonic clock; `null` when the event did not happen.

```ts
interface TurnTrace {
  schema: "ghostline.turn.v1";
  sessionId: string; turnId: string;
  app: "dead-air-fm" | "missed-call-oracle" | "bench";
  doorway: "browser" | "simulator" | "exotel" | "twilio";
  policy: "ghostline" | "naive";
  providers: { stt: string; llm: string; tts: string };
  language: "en-IN" | "hi-IN" | "gu-IN";
  t: {
    speechStart: number | null;        // user speech onset (server)
    endOfTurn: number | null;
    llmFirstToken: number | null;
    firstAudio: number | null;         // first bot audio sent for this reply
    interruptionStart: number | null;  // user speech onset that interrupted
    clientDuck: number | null;
    stopCommand: number | null;
    lastAudioFrame: number | null;     // from the stopped ack, or estimated
  };
  outcome: "full" | "interrupted" | "cancelled";   // FR-C11
  playedMs: number;
  isEstimatedStop: boolean;            // FR-C8
  runId?: string; gitSha?: string;     // bench only
  replyText?: string;                  // bench runs or with consent only
}
```

## 7. Adapter interfaces

```ts
interface SttAdapter {
  id: string;
  open(opts: { language: Lang; sampleRate: 8000 | 16000 }): SttStream;
}
interface SttStream { push(frame: Int16Array): void; close(): void; on(cb: (e: SttEvent) => void): void }

interface TtsAdapter {
  id: string;
  supportsInBandCancel: boolean;       // FR-A7: must be true only if the provider really supports it
  open(opts: { language: Lang; voice: string; sampleRate: number }): TtsStream;
}
interface TtsStream { sendText(chunk: string): void; flush(): void; cancel(): void; on(cb: (e: TtsEvent) => void): void }

interface LlmAdapter {
  id: string;
  stream(req: { messages: Message[]; signal: AbortSignal; maxTokens: number }): AsyncIterable<string>;
}

interface PhoneTransport {   // Exotel, Twilio, simulator
  sendAudio(chunk: Uint8Array): void;
  clear(): void;             // drop audio already queued on the provider side
  mark(name: string): void;  // ask for a callback when playback reaches this point
}
```

In-band cancel: Workers AI Aura (`Clear`) and ElevenLabs (`close_context`) support it. Sarvam does not (FR-A3): stop forwarding audio, close the socket, open a fresh socket for the next reply.

Adapters are tested against recorded fixtures; live tests run only with `LIVE=1` (FR-A8).

## 8. Fallback chains (FR-R7, P1)

A provider error or a 3 s silence on a stream that should be producing moves the session to the next adapter for the rest of the session, and records it in the trace's `providers`.

| Role | Chain |
|---|---|
| STT | Sarvam realtime → Sarvam legacy → Workers AI Nova-3 (covers en and hi; for gu the bot apologises and switches to English) |
| LLM | Workers AI `glm-4.7-flash` → Sarvam `sarvam-105b` → Bedrock (P2) |
| TTS (en) | ElevenLabs → Workers AI Aura → Sarvam |
| TTS (hi, gu) | Sarvam only (Workers AI has no Hindi or Gujarati voice). If Sarvam fails, the bot says sorry in English with Aura and offers English |

## 9. Phone doorways

**Exotel AgentStream** (P1). Exotel opens a WebSocket to our `/session/:id/exotel` endpoint, with `?sample-rate=16000` on the URL (the default is 8 kHz). Media is base64 16-bit PCM inside JSON `media` events. Outbound chunks must be 3200 bytes to 100 KB and a multiple of 320 bytes; we send 100 ms chunks (3200 bytes at 16 kHz), because `clear` only drops audio that has not started playing. `mark` reports playback position, which gives the stop acknowledgement on the phone. Exact shapes: `.kiro/steering/providers.md`.

**Twilio Media Streams** (P1 backup). `<Connect><Stream>` from TwiML. Audio both ways is base64 `audio/x-mulaw` at 8 kHz. Outbound `{"event":"media","streamSid":…,"media":{"payload":…}}`, `{"event":"clear","streamSid":…}`, `{"event":"mark",…}`; after `clear`, Twilio returns pending marks. Trial accounts can only call verified numbers.

**Simulator.** Same Oracle logic and the same `PhoneTransport` interface. Audio is band-limited to 300–3400 Hz and resampled to 8 kHz in the browser so it sounds and behaves like a phone (FR-O5).

## 10. Storage and privacy

- Durable Object storage: finished traces (once per turn), session config. No raw audio, ever.
- Transcripts deleted 60 s after hang-up unless the session is a bench run.
- Phone numbers: salted SHA-256 only (FR-O12).
- Secrets: `wrangler secret` in production, `.dev.vars` locally (ignored by git).

## 11. Apps on the skeleton

| | Dead Air FM | Missed-Call Oracle |
|---|---|---|
| Doorways | browser | simulator, exotel, twilio |
| Languages | en (P0) | hi (default), gu, en; keypad 1/2/3 (FR-O2) |
| Turn pattern | Host monologue in 1–2 sentence segments; dead air 4 s starts a segment (FR-F2); caller flag after an interruption (FR-F4); resume after 2 s (FR-F5) | Greeting discloses AI (FR-O1); answers at most two short sentences (FR-O3) |
| Theme | `data-theme="deadair"` | parent theme |
| Extra rules | NAIVE switch on the main screen (FR-F9) | No live data (FR-O6); general info only for health, legal, emergency (FR-O7) |
