---
inclusion: fileMatch
fileMatchPattern: ["skeleton/**", "runtime-cf/**", "web-kit/**", "adapters/**", "apps/**"]
---

# Voice invariants (never break these)

1. **Four phases only:** `listening`, `thinking`, `speaking`, `interrupted`.
2. **The core is pure.** `step(state, event, now, config) → {state, actions}`. No I/O, no clock reads, no randomness.
3. **Stop order is fixed:** duck → stop playback → cancel LLM → cancel TTS → clear transport → commit spoken part.
4. **After stop, silence.** No audio of a stopped turn is sent again, from any path.
5. **History holds only what was heard,** cut at a word boundary. Unplayed text is display-only and never reaches the LLM.
6. **One outcome per user turn:** `full`, `interrupted` or `cancelled`. One `TurnTrace` per turn.
7. **Backchannels never stop the bot** ("hmm", "haan", "accha"…). Speech must last `minSpeechMs` (default 120) to count.
8. **Echo guard:** in the browser, while the bot speaks, an interruption needs both server speech detection and the client duck.
9. **No waiting in the frame path.** Never await storage or network inside a per-audio-frame handler. Store traces once per turn.
10. **Every outbound socket has a timeout.** A failing provider never crashes the Worker.
11. **Stop time is real or labelled.** Use the client/phone `stopped` ack; if none within 500 ms, estimate and set `isEstimatedStop = true`.
12. **TTS cancel is honest.** `supportsInBandCancel` is true only if the provider really supports it. Otherwise: stop forwarding, close the socket, open a new one for the next reply.
13. **Privacy:** no raw call audio stored; phone numbers only as salted SHA-256; no full numbers in logs.
