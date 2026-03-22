# Project Memory

> **AGENT NOTE:** If you are reading this file first, stop. Read **CLAUDE.md** before proceeding — it is the session bootloader and defines which files to read, in what order, and why. MEMORY.md is a facts store, not an orientation guide.

> *Persistent learned facts about this project. Append new entries with ISO dates. When this file exceeds 200 lines, consolidate: promote permanent knowledge to the appropriate context file, remove stale entries.*

## Architectural Decisions
- [2026-02-23] Split CLAUDE.md (589 lines) into multi-file context system inspired by OpenClaw architecture
- [2026-02-23] File hierarchy for conflicts: SOUL.md > AGENT.md > PRINCIPLES.md > USER.md > MEMORY.md > SKILL.md
- [2026-02-23] CLAUDE.md serves as bootloader/router (~80 lines), not knowledge store
- [2026-02-23] Context files kept in project root (not subdirectory) for easy access
- [2026-02-23] Daily session logs stored in memory/ directory

## User Preferences
- [2026-02-23] User values the "garbage in, garbage out" / Luke 6:45 principle — quality of foundation determines quality of output
- [2026-02-23] User sees Galatians 5:23 ("against such things there is no law") as key framing — principles should be liberating, not restrictive
- [2026-02-23] User wants to explore OpenClaw-style persistence for Claude Code sessions
- [2026-02-23] User interested in how Biblical context can unlock AI creativity and problem-solving

## Lessons Learned
- [2026-02-23] Claude Code files under 200 lines achieve 92% rule application rate; drops to 71% beyond 400 lines
- [2026-02-23] OpenClaw's memory system acknowledges "forgetting is the expected outcome" for model heuristics — manual curation is most reliable
- [2026-02-23] Many popular quotes attributed to historical figures are misattributed (Francis, Mother Teresa) — always verify sources

## Negative Knowledge (What Did NOT Work)
- [2026-02-23] A single 589-line CLAUDE.md tried to be everything — philosophical treatise AND operational manual. Splitting by concern is essential.
