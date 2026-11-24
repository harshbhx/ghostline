---
inclusion: fileMatch
fileMatchPattern: ["adapters/**", "runtime-cf/src/doorways/**", "power/**", ".kiro/hooks/**"]
---

# Provider facts

Checked against each provider's official docs on **24 Nov 2026**. Source links are under each section. If a provider behaves differently from what is written here, check the linked page again and update this file in the same commit as the code.

## Cloudflare Workers AI

| Role | Model ID | Notes |
|---|---|---|
| STT, streaming | `@cf/deepgram/nova-3` | Real-time over WebSocket. Languages include `en`, `en-IN`, `hi`. **No Gujarati.** |
| STT, streaming (alt) | `@cf/deepgram/flux` | Real-time over WebSocket |
| STT, batch | `@cf/openai/whisper-large-v3-turbo` | Not streaming; fixtures and fallback only |
| TTS, streaming | `@cf/deepgram/aura-1`, `@cf/deepgram/aura-2-en` | English. WebSocket messages: send `{"type":"Speak","text":…}`, `{"type":"Flush"}`, `{"type":"Clear"}`, `{"type":"Close"}`; receive binary PCM audio and JSON `Metadata`, `Flushed`, `Cleared`, `Warning`. **`Clear` = in-band cancel.** |
| TTS, batch | `@cf/myshell-ai/melotts` | Returns MP3; not for streaming |
| LLM | `@cf/zai-org/glm-4.7-flash` (default) | Fast, multilingual (100+ languages), function calling, 131k context |
| LLM (alt) | `@cf/meta/llama-3.3-70b-instruct-fp8-fast`, `@cf/openai/gpt-oss-20b` | Both support function calling |
| End of turn (P2) | `@cf/pipecat-ai/smart-turn-v2` | Turn detection; WebSocket supported |

- Realtime WebSocket URL: `wss://api.cloudflare.com/client/v4/accounts/{ACCOUNT_ID}/ai/run/@cf/deepgram/aura-1` (same pattern for `nova-3`), API token in the `Authorization` header. Nova-3 query example: `encoding=linear16&sample_rate=16000&interim_results=true`.
- Do not use `@cf/zai-org/glm-5.2` (planned deprecation 20 Nov 2026) or paid-only models.
- Hindi and Gujarati **speech output** are not available on Workers AI: those need Sarvam.
- Sources: developers.cloudflare.com/workers-ai/models/, /workers-ai/models/nova-3/, /ai-gateway/usage/websockets-api/realtime-api/, /realtime/realtimekit/ai/transcription/, /changelog/post/2025-08-27-partner-models/

## Sarvam

**Auth:** header `api-subscription-key: <key>` (TTS also accepts `Authorization: Bearer <key>`, or the WebSocket subprotocol `api-subscription-key.<key>`; never a query parameter).

### STT, realtime (preferred, FR-A2)
- `wss://api.sarvam.ai/speech-to-text-realtime/ws`
- Query: `language_code` (required: `hi-IN`, `gu-IN`, `en-IN`, … or `auto`), `model` (default `saaras:v4`), `stream_type` (`fast` | `balanced` | `simulated`), `endpointing` (`vad` | `manual`), `encoding` (`linear16` | `mulaw` | …), `sample_rate` (**8000 or 16000 only**), `threshold`, `silence_duration_ms` (default 1000), `prefix_padding_ms`, `min_speech_duration_ms` (default 250).
- Send `{"event":"audio_input","audio":"<base64 PCM>"}` in ~100 ms chunks (max 500 ms on `fast`); `{"event":"end"}` to finish. Manual mode: `speech_start`, `speech_end`, `flush`.
- Receive: `session.begin`, `vad.speech_start`, `vad.speech_end`, `transcript.partial {text}`, `transcript.final {text}`, `config.updated`, `pong`, `session.end {audio_duration_s}`, `error {code, message, is_fatal, status_code}`.
- Close codes: 1003 key/quota, 1008 no audio for 60 s, 1011 internal (retry with backoff), 4000 bad parameter or account not enabled.
- Our VAD timing stays in the core: use Sarvam's VAD events as input, not as the decision.

### STT, legacy (fallback)
- `wss://api.sarvam.ai/speech-to-text/ws` (Saaras). Final transcript per utterance only, no partials.

### TTS, streaming (FR-A3)
- `wss://api.sarvam.ai/text-to-speech/ws?model=bulbul:v3` (also `bulbul:v4-flash`, the low-latency tier). `send_completion_event=true` to get an `event` after each flush.
- First message `{"type":"config","data":{"language_code":"hi-IN","speaker":"shubh","speech_sample_rate":16000,"output_audio_codec":"linear16","min_buffer_size":50,"max_chunk_length":150}}`. `speech_sample_rate`: 8000, 16000, 22050, 24000. `output_audio_codec`: `linear16`, `mulaw`, … (default `mp3`). Speaker IDs are lowercase and must match the model (v4-flash uses persona IDs such as `shubh_hi_devotional`).
- Then `{"type":"text","data":{"text":"…"}}` (1–2500 chars), `{"type":"flush"}`, `{"type":"ping"}` (idle close after 60 s).
- Receive `{"type":"audio","data":{"content_type","audio":"<base64>","request_id"}}`, `{"type":"event","data":{"event_type":"final"}}`, `{"type":"error",…}`.
- **No in-band cancel.** Cancel = stop forwarding audio, close the socket, open a new one for the next reply. `supportsInBandCancel = false`.
- Languages include `hi-IN`, `gu-IN`, `en-IN`.

### LLM (FR-A5, P1)
- `POST https://api.sarvam.ai/v1/chat/completions`, models `sarvam-105b`, `sarvam-105b-conversations`. Same auth header (or Bearer). `sarvam-m` is deprecated: do not use.
- Sources: docs.sarvam.ai realtime streaming guide, TTS WebSocket reference, chat completions overview.

## ElevenLabs (P1, FR-A4)
- Multi-context TTS WebSocket: `wss://api.elevenlabs.io/v1/text-to-speech/{voice_id}/multi-stream-input`, header `xi-api-key`.
- `output_format` as `codec_samplerate[_bitrate]`, for example `pcm_16000` for the browser, `ulaw_8000` for Twilio.
- Each reply gets its own `context_id`. On interruption send `{"context_id": old, "close_context": true}` and start a new context: this is in-band cancel, `supportsInBandCancel = true`. Max 5 contexts per connection; contexts time out after 20 s idle (`keep_alive` resets it). `close_socket` ends everything.
- Source: elevenlabs.io/docs, multi-context WebSocket reference and guide.

## Exotel AgentStream, Voicebot applet (P1)
- Exotel connects **to us**: the applet's URL is our `wss://` endpoint (or an HTTPS endpoint that returns `{"url":"wss://…"}`). Up to 3 custom query parameters, 256 chars total.
- Auth options: Basic Auth in the URL (`wss://KEY:TOKEN@host/path`, arrives as an `Authorization: Basic` header) or IP allow-listing via hello@exotel.com. We use Basic Auth plus our own HMAC token as a custom parameter.
- Inbound events: `connected`, `start` (`stream_sid`, `call_sid`, `from`, `to`, `custom_parameters`, `media_format {encoding:"raw", sample_rate}`), `media {chunk, timestamp, payload}`, `dtmf {digit, duration}`, `mark {name}`, `stop {reason: "stopped" | "callended"}`.
- Outbound: `{"event":"media","stream_sid":…,"media":{"payload":"<base64>"}}`, `{"event":"mark","stream_sid":…,"mark":{"name":…}}`, `{"event":"clear","stream_sid":…}`.
- Audio: raw 16-bit mono little-endian PCM, base64. **8000 Hz by default; 16000 or 24000 by adding `?sample-rate=16000` to the URL.** We use 16000.
- Outbound chunks: **at least 3200 bytes, at most 100 KB, a multiple of 320 bytes.** At 16 kHz, 100 ms = 3200 bytes.
- `clear` drops only audio that has not started playing, so keep chunks small (100 ms) for fast stops.
- Source: developer.exotel.com/docs/agentstream/stream-voicebot-applet

## Twilio Media Streams (P1 backup)
- Bidirectional only with `<Connect><Stream>`. Audio both ways: `audio/x-mulaw`, 8000 Hz, mono, base64, no file headers.
- Outbound: `media {payload}`, `mark {name}`, `clear` (each with `streamSid`). After `clear`, Twilio returns the pending marks.
- Webhooks checked with Twilio's request signature.
- Trial accounts: can call and receive calls only from **verified caller IDs** (verified by SMS); a verified caller ID is needed before buying a number.
- Sources: twilio.com/docs/voice/media-streams/websocket-messages, support.twilio.com trial limitations.

## Kiro
- **Steering:** front matter `inclusion: always | fileMatch | manual`; `fileMatchPattern` is a glob string or a list. Manual files are pulled in with `#name` in chat.
- **Hooks:** one JSON file per hook set at `.kiro/hooks/<id>.json`: `{"version":"v1","hooks":[{"name","description?","trigger","matcher?","action":{"type":"command","command"} | {"type":"agent","prompt"},"timeout?","enabled?"}]}`. Command actions run in the project root and get context as JSON on STDIN. Documented trigger names: `PostFileSave`, `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `SessionStart`, `Stop`. IDE also has file create/delete and pre/post spec-task triggers: create those in Kiro's Hooks panel (describe the hook in plain words) and commit the file Kiro writes. **File triggers fire only on the agent's edits, not on manual edits.**
- **MCP:** workspace config `.kiro/settings/mcp.json`, `{"mcpServers": {"name": {"url": "https://…", "headers": {…}}}}` for remote servers, `command`/`args`/`env` for local ones. Cloudflare docs server: `https://docs.mcp.cloudflare.com/mcp` (the old `/sse` URL is gone). Cloudflare API server: `https://mcp.cloudflare.com/mcp`.
- **Powers:** the recommended format is **Agent Plugins**: `plugin.json` (`$schema`, `name` in kebab-case, `version`, `description`, `author`, `keywords`), optional `mcp.json`, optional `skills/<skill>/SKILL.md`. The older `POWER.md` format still works. Hooks are not part of the plugin format, so our hooks ship through `install-hooks` (FR-P5). Install locally: Powers panel → Add Custom Power → Import power from a folder.
- Sources: kiro.dev/docs/steering/, /docs/hooks/, /docs/hooks/types/, /docs/mcp/configuration/, /docs/powers/create/

## Checks only your account can answer
These cannot be read from public docs. Do them once, then note the answer here.
- [ ] Sarvam: one live STT and one TTS call with your key, to confirm the credits work on the raw APIs.
- [ ] Exotel: Voicebot applet enabled on the account; whether the app server may be hosted on Cloudflare (outside India); whether a webhook fires on an unanswered call.
- [ ] Twilio: whether your trial can reach an Indian mobile from the number it offers.
