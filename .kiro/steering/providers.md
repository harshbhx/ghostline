---
inclusion: fileMatch
fileMatchPattern: ["adapters/**", "runtime-cf/src/doorways/**"]
---

# Provider facts

Status: **VERIFIED** (checked in docs or a live call, with date) or **UNVERIFIED (Ux)** (PRD section 12). Before writing or changing an adapter, check every UNVERIFIED fact it uses against the provider's current docs, then update this file with the date and source.

## Cloudflare Workers AI
- STT, TTS and LLM model names: **UNVERIFIED (U8)**. Check developers.cloudflare.com/workers-ai/models before use.
- Must run with only a Cloudflare account and no other keys (FR-A1).

## Sarvam
- Languages we use: `hi-IN`, `gu-IN`, `en-IN`.
- Streaming STT over WebSocket; prefer the realtime endpoint with partial transcripts, fall back to the older endpoint (FR-A2). Event names and query parameters: **UNVERIFIED (U1)**.
- Bulbul TTS streaming over WebSocket. No in-band cancel: cancel = stop forwarding, close, reopen (FR-A3). Sample-rate config field: **UNVERIFIED (U2)**.
- LLM (OpenAI-compatible) URL and model IDs: **UNVERIFIED (U3)**.
- Credits usable on raw STT/TTS APIs: **UNVERIFIED (U4)**, one live call.
- Do not use Sarvam's managed Voice Agents platform (PRD D6).

## ElevenLabs
- Streaming TTS for English voices (FR-A4). Endpoint, output format names and cancel behaviour: check the API reference before writing the adapter.

## Exotel AgentStream (P1)
- Bidirectional WebSocket; JSON events with base64 PCM16 media; `clear` and `mark` supported.
- Outbound media chunk size within Exotel's limits and a multiple of 320 bytes; we send 100 ms chunks (FR-O9).
- Sample rate on our account (8 or 16 kHz), exact event shapes, enablement status, India data-residency rules: **UNVERIFIED (U5)**.

## Twilio Media Streams (P1 backup)
- Media: base64 mu-law, 8 kHz. Outbound `media`, `clear`, `mark` messages with `streamSid`.
- Webhook requests checked by signature. Trial limits when calling from India: **UNVERIFIED (U6)**.

## Kiro
- Hook file format and triggers, whether a power can ship hooks, remote MCP key: **UNVERIFIED (U7)**. Create one hook in the Kiro UI and compare before writing hook files by hand.
