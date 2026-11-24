---
inclusion: always
---

# Product

Ghostline: an open TypeScript skeleton for real-time voice agents that stop talking when the user interrupts. Runs on Cloudflare Workers + Durable Objects. Kiroween 2026, Skeleton Crew.

- Two apps on one skeleton: **Dead Air FM** (browser AI radio host, `apps/dead-air-fm/`) and **Missed-Call Oracle** (voice Q&A in hi/gu/en by phone or browser simulator, `apps/missed-call-oracle/`).
- A Kiro power (`power/`) with `ghostline-mcp`.
- Proof: a bench comparing `naive` vs `ghostline` policies on the same recorded interruptions.

Truth rules:
- No performance number unless the bench measured it (`run_id`). UI shows `—`, docs show `<measure>`.
- The Oracle discloses it is an AI, has no live data in v1, and gives only general information on health, legal or emergency questions.
- Scope and priority: `docs/PRD.md` (P0 → P1 → P2).
