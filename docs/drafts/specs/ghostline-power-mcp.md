# Draft: ghostline-power-mcp

> Draft written outside Kiro. Input for Kiro spec mode. Covers FR-P1 – FR-P5 (P1). Folder: `power/`.

## Introduction

A Kiro power that teaches Kiro to build voice apps on Ghostline: steering, plus an MCP server (`ghostline-mcp`) whose tools let Kiro test interruption logic and read traces without any audio. Minimum if time is short: steering + `simulate_interruption`.

## Requirements

### Requirement 1: Power manifest (FR-P1)
1. `power/` SHALL use Kiro's Agent Plugins format: `plugin.json` (`name`, `version`, `description`, `author`, `keywords`), `mcp.json` for `ghostline-mcp`, and `skills/<skill>/SKILL.md` that say when to use each tool.

### Requirement 2: simulate_interruption (FR-P2)
1. THE tool SHALL run the pure core with a synthetic scenario: backchannel, cough, real interruption, stop word, interruption during thinking.
2. IT SHALL return the action log and pass or fail, with no network calls, and the same output for the same input.

### Requirement 3: get_turn_traces (FR-P3)
1. THE tool SHALL return recorded traces and a summary: n, barge-in p50 and p95, false stops, missed stops, time-to-first-audio p50, using the TESTING formulas.

### Requirement 4: scaffold_voice_app (FR-P4)
1. THE tool SHALL create `apps/<name>` from a template that compiles and passes its tests.
2. IT SHALL never overwrite an existing folder.

### Requirement 5: install-hooks (FR-P5)
1. THE command SHALL copy hook files (`.kiro/hooks/<id>.json`, `version: "v1"`) into the target project and SHALL NOT overwrite without `--force`. Hooks are not part of the plugin format, so this is how the power ships them.

## Suggested tasks (about 6)
plugin.json + skills → MCP server skeleton → simulate_interruption + determinism test → get_turn_traces + fixture → scaffold_voice_app + temp-folder build test → install-hooks.
