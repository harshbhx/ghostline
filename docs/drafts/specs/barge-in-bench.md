# Draft: barge-in-bench

> Draft written outside Kiro. Input for Kiro spec mode. Covers FR-B1 – FR-B6 and FR-H1 – FR-H3. Folder: `bench/`. Method and formulas: `docs/TESTING.md` sections 3–6.

## Introduction

The bench is the proof. It plays the same recorded interruptions into a naive pipeline and into Ghostline, and publishes only measured numbers.

## Requirements

### Requirement 1: Fake caller (FR-B1)
1. THE fake caller SHALL connect like a browser, wait until the bot speaks, play a clip at a seeded random moment 0.5–3 s in, and record the trace.

### Requirement 2: Metrics (FR-B2)
1. Metrics SHALL be computed exactly as `docs/TESTING.md` section 5 defines them, checked by a hand-computed fixture.
2. Every percentile SHALL be printed with its `n`; estimated stops SHALL be reported separately.

### Requirement 3: Both policies (FR-B3)
1. EVERY scenario SHALL run under `naive` and `ghostline` with the same seeds.

### Requirement 4: Clips (FR-B4)
1. THE clip manifest SHALL cover the 13 clip types (TESTING section 3).
2. A clip SHALL be used only if its manifest entry has `consent: true`.

### Requirement 5: Output (FR-B5, FR-B6)
1. Results SHALL be written as JSONL (P0) and Parquet (P1), each row with `runId` and `gitSha`.
2. (P1) A synthetic caller SHALL run 100 calls per policy and print p50, p95 and n.

### Requirement 6: Hugging Face (FR-H1 – FR-H3, P1)
1. A public dataset SHALL hold bench traces and consented clips with a full dataset card (CC BY 4.0).
2. A public Space SHALL show naive vs Ghostline distributions and a metric table with n, using the same formulas.
3. No real caller audio, phone numbers or tokens SHALL be published (pre-push checklist).

## Suggested tasks (about 6)
Clip manifest + validator → fake caller → metrics + fixture → dual-policy runner + JSONL → report table → (P1) Parquet, synthetic caller, HF dataset + Space.
