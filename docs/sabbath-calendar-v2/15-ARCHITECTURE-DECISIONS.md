# 15 — Architecture Decisions

Each record is **PROPOSED** until the owner checks it in [21-OWNER-APPROVAL-CHECKLIST.md](21-OWNER-APPROVAL-CHECKLIST.md). After approval, the status becomes Accepted. None of these records authorize implementation by themselves.

## ADR-001 — Build V2 in a separate repository

- **Status:** PROPOSED
- **Context:** ChristianCalendar is a four-file static site with a correct anchor and a year clamp. This workspace (`ChristianCalendarGrok`) is an empty repository created for the V2 effort. The brief asked whether to extend in place, refactor in place, or preserve the original and develop V2 separately.
- **Decision:** Option C. Do not modify `ksaday/ChristianCalendar`. Implement V2 in `ChristianCalendarGrok`.
- **Why the alternatives lost:**
  - In-place extension keeps one URL but mixes `Number` math, DOM, and new adapters in one IIFE. Regression risk to the working page is high.
  - In-place modular refactor is the same repository risk with more churn, and it still forces the parchment month page to host a timeline it was not built for.
- **Comparison:**

| Criterion | A. Extend in place | B. Refactor in place | C. Separate V2 |
| --- | --- | --- | --- |
| Mathematical reliability | Weak. Old and new arithmetic share one file | Better if a core is extracted, still one deployable | Strong. Core has no DOM |
| Compatibility | Highest short-term | Medium. Behavior can drift during the split | Legacy stays bit-stable |
| Effort | Lowest at first | High | Medium. Greenfield modules, copied oracles |
| Maintainability | Poor | Fair | Good |
| Testability | Browser self-check only | Can add tests | `node:test` on pure functions |
| UI complexity | Timeline forced into the month page | Same | Timeline is the shell. Month grid can return later |
| Hosting cost | Static, unchanged | Static | Static |
| Operational cost | None | None | None if AI stays out |
| Extensibility | Adapters will tangle with render | Possible | Adapter interface is the extension point |

- **Consequences:** Two sites until the owner chooses to retire or redirect the original. Shared oracles must be copied into V2 tests, not imported across repositories. Documentation cites ChristianCalendar by URL and commit `6229702`.

## ADR-002 — BigInt chronology with floor division

- **Status:** PROPOSED
- **Context:** `sabbathNumber` uses IEEE-754 division. Exact inside the safe integer range, not an unbounded integer type. `BigInt` division truncates toward zero, which breaks negative Gregorian/Julian quotients.
- **Decision:** Authoritative Sabbath numbers, JDNs, and calendar quotients use `bigint`. Every quotient uses floor division (toward −∞). Modulo is Euclidean: remainder in `0 .. m-1` for `m > 0`. Serialize as canonical decimal strings with an optional leading `-`. No thousands separators in stored values.
- **Consequences:** UI formatting is a view concern. `Number(bigint)` is forbidden on authoritative values. Very large inputs are limited by available memory and by a declared input-length cap, not by `2^53`. See [04-MATHEMATICAL-FOUNDATION.md](04-MATHEMATICAL-FOUNDATION.md).

## ADR-003 — Absolute day is the civil JDN integer

- **Status:** PROPOSED
- **Context:** Astronomical Julian Date is a continuous count from noon. Calendars speak in civil dates. Mixing them causes off-by-one-day errors.
- **Decision:** The core’s absolute day is the integer JDN already assigned by `jdnFromGregorian` / `gregFromJdn` to a civil date. Sabbath arithmetic uses that integer only. No fractional JD. No time of day. Timezone and sunset are outside the core.
- **Consequences:** “Noon versus midnight” is documented so adapters do not add or subtract a day to “fix” it. Hebrew sunset boundaries are an uncertainty label on the Hebrew adapter, not a core offset.

## ADR-004 — Sunday-first cycle, Saturday is the Sabbath of that week

- **Status:** PROPOSED
- **Context:** The legacy table is Sunday-first. `dow` is Monday=0 … Saturday=5, Sunday=6. `sunCol` maps Saturday to column 6.
- **Decision:** For any absolute day, the cycle Sabbath is the Saturday of its Sunday–Saturday week. `absoluteDayToSabbathNumber` returns `n` only when the day is that Saturday. Otherwise it is a domain error, not a fraction.
- **Consequences:** Wednesday does not display a fractional Sabbath number. The seven-day inspector lists Sunday through Saturday and marks Saturday.

## ADR-005 — Adapters cannot assign Sabbath numbers

- **Status:** PROPOSED
- **Decision:** Only `sabbathNumberToAbsoluteDay` and the inverse on an actual Sabbath assign `n`. Adapters implement `toAbsoluteDay` / `fromAbsoluteDay` plus quality metadata. Unsupported input returns a typed unsupported result.
- **Consequences:** A Hebrew or Chinese bug can mis-label a day. It cannot renumber Sabbaths. The proof card’s Sabbath number and JDN remain correct even if a row says unsupported.

## ADR-006 — Four adapters, two of them qualified

- **Status:** PROPOSED
- **Decision:**
  - MVP exact: proleptic Gregorian (port of the existing functions) and Julian (so Sabbath #1 reads 4 April AD 33).
  - MVP qualified: calculated civil Hebrew, cited, with a verified range. Not the ancient observational calendar.
  - MVP narrow: Chinese lunisolar only inside a range checked against published tables. Elsewhere unsupported.
- **Consequences:** The timeline always has five rows. Unsupported rows render an explicit empty state. They do not drop out and they do not invent labels. See [07-CALENDAR-ADAPTER-DESIGN.md](07-CALENDAR-ADAPTER-DESIGN.md).

## ADR-007 — Proleptic Gregorian does not model the 1582 skip

- **Status:** PROPOSED
- **Context:** The legacy app states that pre-1582 dates are proleptic Gregorian. Historical civil calendars skipped 5–14 October 1582 in the papal reform, and other countries reformed later.
- **Decision:** The Gregorian adapter stays proleptic for every year, matching `jdnFromGregorian`. The skipped dates and country-by-country reform dates are historical notes, not holes in the JDN sequence. Julian is a separate adapter.
- **Consequences:** 4 October 1582 Gregorian and 4 October 1582 Julian are different absolute days. The UI must show the calendar name beside every date. No silent conversion.

## ADR-008 — No UI framework for MVP

- **Status:** PROPOSED
- **Context:** The brief says not to pick a framework for popularity. The legacy app is dependency-free.
- **Decision:** TypeScript compiled to static ES modules. Tests use `node:test` and a small in-repo deterministic sampler. No React, no calendar library as the Sabbath authority, no database, no backend. A bundler may be added only if module delivery cannot be done with native ES modules.
- **Consequences:** Timeline virtualization is hand-written and must keep `BigInt` coordinates. That is extra UI work and less dependency risk. See ADR-002.

## ADR-009 — Creation boundary stays outside the core

- **Status:** PROPOSED
- **Decision:** Mode A (unbounded integers) is the only engine mode in MVP. Mode B (a finite Creation boundary) may be researched and, only after a later approval, attached as an optional labeled overlay. No Creation date is selected in this design.
- **Consequences:** [08-GENESIS-CREATION-BOUNDARY.md](08-GENESIS-CREATION-BOUNDARY.md) records traditions and uncertainty. It does not add a constant to the core.

## ADR-010 — Sunset is Phase 6

- **Status:** PROPOSED
- **Decision:** Sabbath index does not depend on coordinates, timezone, DST, or sunrise/sunset. A later sunset module may show Friday sunset and Saturday sunset for a chosen place. High latitudes and missing rise/set are that module’s problem. MVP may keep the legacy sentence only if it is clearly copy, not a computed instant. Prefer omitting computed sunset until Phase 6.
- **Consequences:** “Sabbath day” in the product means the indexed Saturday civil date, not a sunset-to-sunset interval.

## ADR-011 — Deterministic MVP, optional owner-side Auditor, no public chatbot

- **Status:** PROPOSED
- **Decision:** Ship the calendar with no model in the request path (Option A for the product). Document an optional Cursor Automation, Sabbath Calendar Auditor, for the owner (Option B). Do not build an in-app chatbot (Option C) in MVP.
- **Why:** Arithmetic must stay testable with the network off. Current Cursor pricing (checked 11 November 2026) puts Cloud Agents, Automations, and Grok Bot on Premium and above, billed from the weekly usage pool. The Grok API is billed separately and is not included in that pool. A personal assistant subscription does not power a public website.
- **Consequences:** Project rules must not name a model. Auditor routines are opt-in and are not created by this design phase. See [16-CURSOR-RULES-PROPOSAL.md](16-CURSOR-RULES-PROPOSAL.md) through [19-OPTIONAL-AI-ASSISTANT-FEASIBILITY.md](19-OPTIONAL-AI-ASSISTANT-FEASIBILITY.md).

## ADR-012 — Sketch order and non-oracle callouts

- **Status:** PROPOSED
- **Decision:** Row order is Sabbath, Jewish, Gregorian, Chinese, Julian, top to bottom, matching the drawing. Gold cross marks Sabbath #1 only. Red sketch callouts are not test data.
- **Consequences:** Wireframes in [11-UI-UX-WIREFRAMES.md](11-UI-UX-WIREFRAMES.md) use the verified examples (#1, #0, #−1, #103992) when they show real numbers.

## ADR-013 — Shareable state is `?n=`

- **Status:** PROPOSED
- **Decision:** The canonical link for a Sabbath is query parameter `n`, the canonical decimal Sabbath number, including a leading `-`. Optional `scale` sets zoom. A non-Sabbath day is shared with `d`, a canonical JDN. Opening the link recomputes labels through the engine. If `n` and `d` are both present and disagree, the page errors. No hash of AI text. No stored server session.
- **Consequences:** Negative numbers must survive URL parsing (`n=-1`, and the encoded form). Invalid `n` or `d` shows a validation error and does not fall through to “today”.

## ADR-014 — Documentation and rules are not activated in the design phase

- **Status:** PROPOSED (already the rule for this phase)
- **Decision:** This phase may add only `docs/sabbath-calendar-v2/**`. Proposed `.mdc` contents live inside [16-CURSOR-RULES-PROPOSAL.md](16-CURSOR-RULES-PROPOSAL.md). They are not written to `.cursor/rules/`, and `AGENTS.md` is not created, until the owner approves that installation.
- **Consequences:** Agents that have not been given those rules can still see them by reading doc 16. Installation is a separate change.

## ADR-015 — Input cap instead of a fake infinity

- **Status:** PROPOSED
- **Decision:** The mathematical model is unbounded. A single request accepts Sabbath numbers and JDNs whose canonical decimal form has at most 10,000 digits, and refuses larger input with a validation error. This is a denial-of-service and UI limit, not a change to the formula. Ten thousand digits is far beyond any historical or planned display range and still trivial for `BigInt`.
- **Consequences:** Tests include one value near the cap and one value over the cap. The cap is a named constant, not an implicit `Number` limit. The owner may lower it before implementation if a smaller cap is preferred. **UNRESOLVED** only if the owner rejects 10,000 digits. The default stands if unchecked.
