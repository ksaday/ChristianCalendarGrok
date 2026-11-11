# 00 — Executive Summary

Status: **PROPOSED** design, ready for owner review. No application code is authorized by this document.

Evidence labels used throughout this set: **VERIFIED** (checked against the existing repository or an independent integer check of its formulas), **MATHEMATICALLY DERIVED** (follows from the project formula), **PROJECT-DEFINED** (an owner convention, not an independent historical proof), **PROPOSED** (a design choice awaiting approval), **UNRESOLVED** (explicitly not decided).

## What this phase delivers

Sabbath Calendar V2 is a design for a mathematically complete seven-day chronology. Every Sabbath has one integer index. Sabbath #1 stays the permanent anchor. Other calendars are synchronized labels on that same day axis.

This phase writes design documents only, under `docs/sabbath-calendar-v2/` in the `ChristianCalendarGrok` repository. It does not change the original ChristianCalendar application.

## Anchor (PROJECT-DEFINED, preserved)

```text
Sabbath #1 = Saturday, 4 April AD 33, Julian calendar
ANCHOR_JDN = 1733205
SabbathJDN(n) = 1733205 + 7 * (n - 1)
```

`n` is any integer, including 0 and negative integers. The anchor is an immutable project convention. These documents do not treat it as independent theological or historical proof, and they do not reopen it.

**VERIFIED** contract values:

| n | JDN | Proleptic Gregorian date |
| --- | ---: | --- |
| 3 | 1733219 | 16 April 33 |
| 2 | 1733212 | 9 April 33 |
| 1 | 1733205 | 2 April 33 |
| 0 | 1733198 | 26 March 33 |
| −1 | 1733191 | 19 March 33 |
| −2 | 1733184 | 12 March 33 |

Julian 4 April AD 33 and proleptic Gregorian 2 April AD 33 are the same physical day (JDN 1,733,205). **VERIFIED** regression oracle from the existing self-check: 11 April 2026 → JDN 2,461,142 → Sabbath #103,992.

The sketch labels “Sabbath #104,017” and “10/3/2026” are layout cues, not oracles.

## Recommendation

**PROPOSED / ADR-001:** Option C. Leave [ksaday/ChristianCalendar](https://github.com/ksaday/ChristianCalendar) untouched as the legacy reference. Build V2 in this repository (`ChristianCalendarGrok`) as a static TypeScript application:

1. A pure chronology core using `BigInt`, with no DOM, timezone, AI, or civil-calendar imports.
2. Calendar adapters (proleptic Gregorian, Julian, calculated Hebrew, Chinese lunisolar) that convert to and from the absolute day and never write the Sabbath index.
3. A synchronized multi-row timeline UI matching the owner’s sketch.
4. Property and regression tests as a release gate.
5. No database, no required backend, no required AI.

MVP is Phases 0–4 in [13-DEVELOPMENT-ROADMAP.md](13-DEVELOPMENT-ROADMAP.md). The Sabbath Calendar Auditor (Phase 5) and sunset, Creation-boundary, and public-chatbot work (Phase 6) stay opt-in and out of the engine.

## Findings the design must carry

- The current page already shows Sabbath #−12 through #0 inside year 33, then `clampY` and `<input min="33" max="3000">` block year 32 and earlier. Sabbath #−13 (25 December 32, proleptic Gregorian) is the first Sabbath the UI cannot open. **VERIFIED**.
- Authoritative math is IEEE-754 `number`. It is exact for the published oracles inside `2^53` and is not an unbounded integer type.
- Porting the existing quotients to `BigInt` with truncating division is incorrect for negative values. Floor division is required. **VERIFIED**: `−8 // 7 = −2`, while truncating division yields `−1`.
- There is no Julian, Hebrew, or Chinese converter, no BigInt, and no sunset calculation. “Begins at sunset Friday” is display copy only.
- Hebrew and Chinese dates are not historically certain just because a modern algorithm returns a label. Each adapter must publish its range and return unsupported outside it.

## AI boundary

The MVP calendar is deterministic. Cursor coding models are swappable and are not named in project rules. An optional owner-side Auditor may review later. It is not an authority for arithmetic. A public chatbot is not recommended for MVP. A personal Grok Bot subscription does not provide website API access. See [17-GROK-BOT-INTEGRATION-PLAN.md](17-GROK-BOT-INTEGRATION-PLAN.md) and [19-OPTIONAL-AI-ASSISTANT-FEASIBILITY.md](19-OPTIONAL-AI-ASSISTANT-FEASIBILITY.md).

## Owner decisions still required

Listed in full in [21-OWNER-APPROVAL-CHECKLIST.md](21-OWNER-APPROVAL-CHECKLIST.md). The decisions that change the build are: accept Option C, accept BigInt plus floor division, accept qualified Hebrew and Chinese ranges, keep Creation out of the core, ship MVP without a public chatbot, and decide whether this design set should be committed.
