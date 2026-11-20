# Claude Code v2.1.88 — Source Code Analysis

> **Disclaimer**: All source code in this repository is the intellectual property of **Anthropic and Claude**. This repository is provided solely for technical research, study, and educational reference. **Commercial use, profitable activities, unlawful purposes, or unauthorized use are strictly prohibited.** If any content infringes your legal rights, intellectual property, or other interests, please contact us and we will promptly verify and remove it.

> Extracted from npm package `@anthropic-ai/claude-code` version **2.1.88**.  
> The published package contains only a single bundled `cli.js` (~12MB). The `src/` directory in this repository contains the pre-bundled **TypeScript source code** extracted from the package.

**Languages**: **English** | [中文](README_CN.md) | [한국어](README_KR.md) | [日本語](README_JA.md)

---

## Table of Contents

- [Deep Analysis Reports (`docs/`)](#deep-analysis-reports-docs) — Telemetry, model codenames, undercover mode, remote control, and future roadmap
- [Missing Modules Reference](#missing-modules-reference-108-modules) — 108 modules removed via feature gates
- [Architecture Overview](#architecture-overview) — Entrypoints → Query Engine → Tools / Services / State
- [Build Instructions](#build-instructions) — Why it cannot be compiled directly without Bun
- [Repository Structure](#repository-structure)

---

## Deep Analysis Reports (`docs/`)

Technical analysis reports based on decompiled Claude Code v2.1.88 source code:

```
docs/
├── en/                                        # English
│   ├── 01-telemetry-and-privacy.md            # Telemetry & Privacy — what is collected and why you cannot opt out
│   ├── 02-hidden-features-and-codenames.md    # Codenames (Capybara/Tengu/Numbat), feature flags, internal vs external
│   ├── 03-undercover-mode.md                  # Undercover Mode — hiding AI authorship in open-source repos
│   ├── 04-remote-control-and-killswitches.md  # Remote Control — managed settings, emergency killswitches, model overrides
│   └── 05-future-roadmap.md                   # Future Roadmap — Numbat, KAIROS agent mode, voice mode, unreleased tools
│
└── zh/                                        # Chinese versions
```

| # | Topic | Key Finding | Link |
|---|-------|-------------|------|
| 01 | **Telemetry & Privacy** | Two-tier analytics pipeline (1P Anthropic, Datadog). Environment fingerprinting, process metrics, session/user IDs per event. No user-facing opt-out switch. | [EN](docs/en/01-telemetry-and-privacy.md) |
| 02 | **Hidden Features & Codenames** | Animal codename scheme (Capybara v8, Tengu, Fennec -> Opus 4.6, Numbat). Obfuscated feature flags. Hidden commands like `/btw`, `/stickers`. | [EN](docs/en/02-hidden-features-and-codenames.md) |
| 03 | **Undercover Mode** | Anthropic internal users automatically enter undercover mode in public repos. Strips AI attribution so commits look human-authored. | [EN](docs/en/03-undercover-mode.md) |
| 04 | **Remote Control & Killswitches** | Hourly polling of `/api/claude_code/settings`. Blocking dialog on risky changes. 6+ emergency killswitches. | [EN](docs/en/04-remote-control-and-killswitches.md) |
| 05 | **Future Roadmap** | Numbat codename, Opus 4.7 / Sonnet 4.8, KAIROS fully autonomous agent mode with `<tick>` heartbeats, voice push-to-talk mode. | [EN](docs/en/05-future-roadmap.md) |

---

## Missing Modules Reference (108 Modules)

> **The source code is incomplete.** 108 modules guarded by `feature()` gates were **not included** in the npm package. They exist only within Anthropic's internal monorepo and are dead-code-eliminated at bundle time.

```
  Anthropic Internal Monorepo           Published npm Package
  ───────────────────────────           ─────────────────────
  feature('DAEMON') → true    ──Build──→   feature('DAEMON') → false
  ↓                                         ↓
  daemon/main.js  ← Included   ──Bundle─→  daemon/main.js  ← Eliminated (DCE)
  tools/REPLTool  ← Included   ──Bundle─→  tools/REPLTool  ← Eliminated (DCE)
```

Bun's `feature()` is a compile-time intrinsic:
- Returns `true` in internal builds → preserved in bundle
- Returns `false` in public releases → tree-shaken by dead code elimination
- The remaining source references these modules, but they are omitted from public releases.

---

## Agent Loop Architecture

```
                    Core Agent Loop
                    ===============

    User Prompt --> messages[] --> Claude API --> Response
                                                    |
                                          stop_reason == "tool_use"?
                                         /                          \
                                       YES                           NO
                                        |                             |
                                  Execute Tool                   Return Text
                                  Append tool_result
                                  Loop back -----------------> messages[]
```

Claude Code wraps this minimal agent loop with production harnesses: permissions, streaming, concurrency, auto-compaction, subagents, session persistence, and MCP (Model Context Protocol).

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                          Entry Layer                                │
│  cli.tsx ──> main.tsx ──> REPL.tsx (Interactive terminal UI)        │
│                     └──> QueryEngine.ts (Headless / SDK mode)       │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         Query Engine                                │
│  submitMessage(prompt) ──> AsyncGenerator<SDKMessage>               │
│    ├── fetchSystemPromptParts()    ──> Assembles system prompt      │
│    ├── processUserInput()          ──> Handles slash commands       │
│    ├── query()                     ──> Main agent execution loop    │
│    │     ├── StreamingToolExecutor ──> Parallel tool runner         │
│    │     ├── autoCompact()         ──> Context compression          │
│    │     └── runTools()            ──> Tool invocation & handling   │
│    └── yield SDKMessage            ──> Streams output events        │
└──────────────────────────────┬──────────────────────────────────────┘
```

---

## Build Instructions

This source code cannot be built directly using `npm run build` with standard Node.js without preparation:
- Relies on Bun compile-time intrinsics (`feature()`, `MACRO`, `bun:bundle`).
- A best-effort build with `esbuild` reaches ~95% completion (see [QUICKSTART.md](QUICKSTART.md)).
- To run immediately, use the pre-built `cli.js` included in the official npm release.

---

## Statistics

| Metric | Count |
|--------|-------|
| TypeScript files (`.ts` / `.tsx`) | ~1,884 |
| Total lines of code | ~512,664 |
| Largest file | `src/query.ts` (~785 KB) |
| Built-in tools | 40+ |
| Slash commands | 80+ |
| Runtime | Bun (bundled to Node.js >= 18) |

---

## Copyright & License

Copyright (c) Anthropic. All rights reserved.  
All source code in this repository is the intellectual property of Anthropic and Claude. For full licensing details, refer to the original npm package `@anthropic-ai/claude-code`.
