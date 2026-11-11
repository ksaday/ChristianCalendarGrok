# 03 — Software Requirements Specification

Requirements in this set are **PROPOSED** until the owner accepts [21-OWNER-APPROVAL-CHECKLIST.md](21-OWNER-APPROVAL-CHECKLIST.md). Priorities: Must (MVP), Should (MVP if cheap), Later (Phase 5 or 6).

Verification methods: Test (automated), Inspection (document or code review), Demonstration (UI).

## 3.1 Chronology

### REQ-CHR-001 — Immutable anchor

- **Description:** `SabbathJDN(1)` equals `1733205`. The constant is the project Sabbath #1 (Julian 4 April AD 33).
- **Rationale:** Owner constraint. The existing app already uses this constant.
- **Priority:** Must
- **Dependencies:** none
- **Acceptance criteria:** A test named for this ID fails if the constant changes. No adapter, UI control, or AI tool can write the constant.
- **Verification:** Test, Inspection
- **Label:** PROJECT-DEFINED

### REQ-CHR-002 — Direct seven-day formula

- **Description:** For every accepted integer `n`, `SabbathJDN(n) = 1733205 + 7 * (n - 1)`, computed with arbitrary-precision integers. The implementation evaluates the formula. It does not loop from Sabbath #1.
- **Rationale:** Distant negative and positive indexes must be cheap and exact.
- **Priority:** Must
- **Dependencies:** REQ-CHR-001, REQ-MATH-001
- **Acceptance criteria:** Property tests for random and extreme `n` show a step of exactly 7 and a round trip to `n`. A review of the core shows no loop whose trip count depends on `|n|`.
- **Verification:** Test, Inspection
- **Label:** MATHEMATICALLY DERIVED from the project formula

### REQ-CHR-003 — Zero and negative indexes

- **Description:** `n = 0` and `n < 0` are valid Sabbath numbers. Zero is not skipped. Negative values are not renumbered.
- **Rationale:** The numbering contract in the brief, and the legacy seal already renders `sn < 1`.
- **Priority:** Must
- **Dependencies:** REQ-CHR-002
- **Acceptance criteria:** Sabbath #0 is JDN 1733198. Sabbath #−1 is JDN 1733191. Sabbath #−2 is JDN 1733184. `SabbathNumber` of each of those JDNs returns the same `n`.
- **Verification:** Test
- **Label:** MATHEMATICALLY DERIVED

### REQ-CHR-004 — Inverse only on Sabbaths

- **Description:** `absoluteDayToSabbathNumber` returns `n` when the absolute day is `SabbathJDN(n)`. For any other absolute day it returns a domain error and does not return a fractional number.
- **Rationale:** The legacy function uses floating division and would yield a fraction on a Wednesday. V2 must not present that fraction as a Sabbath number.
- **Priority:** Must
- **Dependencies:** REQ-CHR-002
- **Acceptance criteria:** Inverse of Sabbath #1 is 1. Inverse of the next day is an error whose code is `not-a-sabbath`. The error payload may include the cycle descriptor from REQ-CHR-005.
- **Verification:** Test

### REQ-CHR-005 — Seven-day cycle descriptor

- **Description:** `absoluteDayToSabbathCycle` maps every absolute day to the Saturday of its Sunday–Saturday week and to a day offset. Sunday is offset 0. Saturday is offset 6. The Sabbath number on that Saturday matches REQ-CHR-002.
- **Rationale:** Matches `sunCol` in `js/calendar.js` (Sunday column 0, Saturday column 6).
- **Priority:** Must
- **Dependencies:** REQ-CHR-002, REQ-CHR-004
- **Acceptance criteria:** For every tested Sabbath, offset is 6 and the number matches. For the previous Sunday, offset is 0 and the Sabbath number is that Saturday’s number. Offsets are only `0..6`.
- **Verification:** Test
- **Label:** PROPOSED cycle convention, aligned with the legacy UI

### REQ-CHR-006 — Neighbor and distance operations

- **Description:** `getNextSabbath(n) = n + 1`, `getPreviousSabbath(n) = n - 1`, `getSabbathInterval(n1, n2) = n2 - n1` in Sabbath counts, and the day distance is `7 * (n2 - n1)`. `getChronologicalDistance(day1, day2) = day2 - day1` in absolute days. `getCycleDayOffset` is the offset from REQ-CHR-005.
- **Rationale:** Navigation and the proof card need these without recomputing calendars.
- **Priority:** Must
- **Dependencies:** REQ-CHR-002, REQ-CHR-005
- **Acceptance criteria:** Interval from #−2 to #3 is 5 Sabbaths and 35 days. Distance is signed. Swapping endpoints negates the distance.
- **Verification:** Test

### REQ-CHR-007 — Core isolation

- **Description:** The chronology module imports no DOM API, no `Date`, no timezone database, no AI client, and no calendar adapter.
- **Rationale:** Sabbath numbers must not depend on where the page is opened or which adapter is loaded.
- **Priority:** Must
- **Dependencies:** none
- **Acceptance criteria:** A dependency check (import graph test or lint) fails if the core imports those modules. Core unit tests run in Node with no browser.
- **Verification:** Test, Inspection

### REQ-CHR-008 — Validation and invariants

- **Description:** `validateSabbathNumber` accepts a canonical integer (see REQ-MATH-003) inside the input cap (REQ-PERF-002) and rejects everything else. `verifyChronologyInvariants` checks the anchor, the step of 7, and round trip for a supplied set of numbers and returns a structured report.
- **Rationale:** The legacy footer self-check is the pattern. V2 makes it a callable report the UI and the Auditor can both run.
- **Priority:** Must
- **Dependencies:** REQ-CHR-001, REQ-MATH-003, REQ-PERF-002
- **Acceptance criteria:** The report for `{1, 0, -1, 103992}` passes. A mutated anchor in a test double fails the report. Production code has no test double.
- **Verification:** Test

### REQ-CHR-009 — Legacy oracles

- **Description:** The V2 core reproduces the legacy self-check: proleptic Gregorian 11 April 2026 is JDN 2461142 and Sabbath #103992; the anchor JDN is Sabbath #1; the year-33 anniversary JDN equals the anchor.
- **Rationale:** Preserve validated behavior.
- **Priority:** Must
- **Dependencies:** REQ-CAL-001, REQ-CHR-001
- **Acceptance criteria:** Tests assert those three facts. Failure text is suitable for a release gate (“do not trust displayed numbers”).
- **Verification:** Test
- **Label:** VERIFIED against `selfCheck` and an independent integer re-check

## 3.2 Mathematics

### REQ-MATH-001 — No floating authority

- **Description:** Sabbath indexes, absolute days, and calendar algorithm quotients that feed those values are `bigint`. IEEE-754 numbers are not an input to those calculations.
- **Rationale:** `Number` is exact only through `2^53 - 1`. The brief requires unbounded integer indexing.
- **Priority:** Must
- **Dependencies:** none
- **Acceptance criteria:** Core public functions do not accept `number` for `n` or JDN. A test feeds a value above `2^53` and gets an exact round trip.
- **Verification:** Test, Inspection

### REQ-MATH-002 — Floor division and Euclidean modulo

- **Description:** Integer division used by chronology and by calendar adapters is floor division. Modulo `a mod m` for `m = 7` (and any positive modulus in these algorithms) has remainder in `0 .. m-1`, including when `a` is negative.
- **Rationale:** `BigInt` truncating division breaks negative years. The legacy `dow` already normalizes modulo with `((j % 7) + 7) % 7`.
- **Priority:** Must
- **Dependencies:** REQ-MATH-001
- **Acceptance criteria:** `floorDiv(-8, 7) === -2` and `euclidMod(-1, 7) === 6`. A negative JDN that is a Saturday still has weekday index 5.
- **Verification:** Test
- **Label:** VERIFIED contrast between floor and truncating division

### REQ-MATH-003 — Canonical serialization

- **Description:** The canonical form of an integer is a decimal string with no leading zeros, no `+` sign, a leading `-` when negative, and no spaces or thousands separators. `0` is `"0"`. Parsers accept that form and reject `"01"`, `"1.0"`, `"1e3"`, `"++1"`, Unicode minus, and empty input. The UI may format separators for display and must parse them back to the canonical form before calling the core.
- **Rationale:** URLs, tests, and the proof card must exchange one spelling of each integer.
- **Priority:** Must
- **Dependencies:** REQ-MATH-001
- **Acceptance criteria:** Round trip of examples in REQ-CHR-003 and of a 100-digit value. Rejection cases above fail validation.
- **Verification:** Test

### REQ-MATH-004 — Weekday numbering

- **Description:** Weekday index is `euclidMod(jdn, 7)` with Monday = 0 … Saturday = 5, Sunday = 6, matching `dow` in `js/calendar.js`.
- **Rationale:** Keeps `isSabbath <=> index === 5` true for the anchor (1733205 mod 7 = 5). **VERIFIED**.
- **Priority:** Must
- **Dependencies:** REQ-MATH-002
- **Acceptance criteria:** Anchor index is 5. The next six days are 6,0,1,2,3,4. The previous day is 4.
- **Verification:** Test

### REQ-MATH-005 — Astronomical years inside, BCE/CE at the edge

- **Description:** Adapter internals use astronomical year numbering (year 0 is 1 BCE, year −1 is 2 BCE). Display uses BCE/CE (or BC/AD if the owner later insists on the legacy “AD” copy) and never shows year 0 to the reader. There is no year 0 in BCE/CE output.
- **Rationale:** Off-by-one-year errors come from forgetting that 1 BCE is year 0.
- **Priority:** Must
- **Dependencies:** REQ-CAL-001, REQ-CAL-002
- **Acceptance criteria:** Astronomical year 0 formats as 1 BCE. Astronomical year −1 formats as 2 BCE. Year 1 formats as 1 CE. A test documents the mapping in both directions.
- **Verification:** Test
- **Label:** PROPOSED display convention. Legacy UI says “AD” only for years `<= 100` and otherwise prints the bare year. V2 standardizes on BCE/CE for all non-positive and all positive years so the label is unambiguous.

## 3.3 Calendars

### REQ-CAL-001 — Proleptic Gregorian adapter

- **Description:** Port `jdnFromGregorian` and `gregFromJdn` to `bigint` floor division. Leap rule: divisible by 4, and not by 100 unless by 400. The adapter is proleptic for every astronomical year it accepts. It does not skip 5–14 October 1582.
- **Rationale:** Preserve validated behavior. The README already says pre-1582 display is proleptic Gregorian.
- **Priority:** Must
- **Dependencies:** REQ-MATH-002, REQ-MATH-005
- **Acceptance criteria:** 11 April 2026 → 2461142. 2 April 33 → 1733205. Round trip those dates. 15 October 1582 → 2299161. A label on screen includes the word Gregorian.
- **Verification:** Test, Demonstration
- **Label:** VERIFIED for the listed JDNs

### REQ-CAL-002 — Julian adapter

- **Description:** Julian civil dates convert to and from the same absolute day. Leap rule: every astronomical year divisible by 4 is a leap year, including year 0. Sabbath #1’s Julian label is 4 April 33 CE.
- **Rationale:** The anchor is defined on the Julian calendar. The legacy UI never shows that label in the grid.
- **Priority:** Must
- **Dependencies:** REQ-MATH-002
- **Acceptance criteria:** Julian 4 April 33 → 1733205. Proleptic Gregorian 2 April 33 → the same JDN. Julian and Gregorian labels differ for that day and are both shown. Round trip a BCE sample chosen in the test plan.
- **Verification:** Test
- **Label:** VERIFIED for 4 April 33 during inspection (Julian-style formula returned 1733205)

### REQ-CAL-003 — Calculated Hebrew adapter

- **Description:** A calculated civil Hebrew adapter converts within a documented range. It exposes quality metadata. Outside the verified range, and for any request that would require an observational or disputed calendar, it returns unsupported. It must not be described in the UI as the calendar used in the first century.
- **Rationale:** The brief requires a Jewish row and forbids invented ancient dates. Sunset day-start is a known mismatch with midnight/noon civil JDN.
- **Priority:** Must (qualified)
- **Dependencies:** REQ-CAL-005, REQ-CHR-007
- **Acceptance criteria:** At least three published reference pairs inside the verified range round-trip. One date outside the range returns `unsupported` with a reason code. Sabbath number for those days still comes from the core.
- **Verification:** Test, Inspection
- **Label:** PROPOSED algorithm family (Dershowitz and Reingold civil Hebrew). Specific reference triples are **UNRESOLVED** until implementation cites the exact table rows used. The design forbids shipping without those rows.

### REQ-CAL-004 — Chinese lunisolar adapter

- **Description:** A Chinese lunisolar adapter converts only inside a verified window backed by published tables (leap months and solar terms included). Outside that window it returns unsupported. Ancient regional variants are not implemented.
- **Rationale:** Same evidence rule as Hebrew, with a weaker public algorithmic consensus for deep history.
- **Priority:** Must (narrow)
- **Dependencies:** REQ-CAL-005
- **Acceptance criteria:** Same shape as REQ-CAL-003. The verified window is stated as concrete years in the adapter metadata before the feature is marked done. If no table is cited, the adapter’s supported range stays empty and every call returns unsupported. An empty range is an acceptable MVP outcome. A fabricated range is not.
- **Verification:** Test, Inspection
- **Label:** UNRESOLVED numeric window until a cited table is chosen. The behavior when unresolved is specified, not left blank.

### REQ-CAL-005 — Adapter metadata and non-interference

- **Description:** Every adapter exposes the metadata block in [07-CALENDAR-ADAPTER-DESIGN.md](07-CALENDAR-ADAPTER-DESIGN.md). No adapter function returns a Sabbath number. No adapter writes core constants.
- **Rationale:** Owner constraint 11.
- **Priority:** Must
- **Dependencies:** REQ-CHR-007
- **Acceptance criteria:** Type definitions contain no Sabbath field on adapter results. A test double that returns a wrong civil label does not change `n` or JDN in the proof card.
- **Verification:** Inspection, Test

### REQ-CAL-006 — Anniversary and milestone are presentation

- **Description:** The anniversary Sabbath (first Saturday on or after Gregorian 4 April, with year 33 fixed at the anchor) and the every-1,000th milestone for `n >= 1 && n % 1000 === 0` may be shown as markers. They are not core chronology.
- **Rationale:** Preserve legacy meaning without letting a solar-year rule redefine the index.
- **Priority:** Should
- **Dependencies:** REQ-CAL-001, REQ-CHR-002
- **Acceptance criteria:** `annivJdnOfYear(33)` equivalent equals 1733205. A non-anniversary Sabbath keeps its index. Negative multiples of 1000 are not styled as milestones unless the owner later changes this rule.
- **Verification:** Test
- **Label:** VERIFIED legacy behavior, PROPOSED to retain

### REQ-CAL-007 — Future adapters

- **Description:** A new calendar is a new adapter module plus metadata plus tests. Adding one does not change core signatures.
- **Priority:** Later
- **Dependencies:** REQ-CAL-005
- **Acceptance criteria:** Design review shows no core diff required to register an adapter.
- **Verification:** Inspection

## 3.4 User interface

### REQ-UI-001 — Synchronized rows

- **Description:** One horizontal time axis, one vertical cursor, five rows in sketch order: Sabbath, Jewish, Gregorian, Chinese, Julian. Scrolling or jumping moves all rows together.
- **Priority:** Must
- **Dependencies:** REQ-CHR-002, REQ-CAL-005
- **Acceptance criteria:** Jumping to Sabbath #−1 moves the cursor to JDN 1733191 on every row. Rows that do not support that day show unsupported at that x-position, still aligned.
- **Verification:** Demonstration, Test of the view-model (row models share one cursor day)

### REQ-UI-002 — Search

- **Description:** The user can search by Sabbath number (any accepted integer), Gregorian date, Julian date, and Hebrew or Chinese date when the adapter supports that date.
- **Priority:** Must
- **Dependencies:** REQ-MATH-003, REQ-CAL-001, REQ-CAL-002, REQ-CAL-003, REQ-CAL-004
- **Acceptance criteria:** Search `#-1`, `#0`, `#1`, `#103992` land on the verified JDNs. An unsupported Hebrew or Chinese query explains that the adapter refused it.
- **Verification:** Demonstration, Test

### REQ-UI-003 — Scale without floating authority

- **Description:** Zoom levels cover day, week, year, century, millennium, and a coarser “age” level. Tick placement may use floating point for pixels. The cursor’s identity remains a `bigint` absolute day or Sabbath number. Pixel math must not be parsed back into a JDN.
- **Priority:** Must
- **Dependencies:** REQ-MATH-001
- **Acceptance criteria:** At the coarsest zoom, activating the cursor still shows the exact Sabbath number and JDN in the proof card. A test builds a view-model at a large `n` and checks the identity fields are strings/bigints, not `number`.
- **Verification:** Test, Inspection

### REQ-UI-004 — Proof card

- **Description:** For the selected Sabbath, show number, anchor JDN, calculated JDN, offset `7*(n-1)`, the formula, calendar labels that resolved, unsupported calendars, and the invariant check result from `verifyChronologyInvariants`.
- **Priority:** Must
- **Dependencies:** REQ-CHR-008, REQ-CAL-005
- **Acceptance criteria:** Every numeric field is produced by the core or an adapter. The card has no free-text generator in MVP.
- **Verification:** Test, Demonstration

### REQ-UI-005 — Cycle inspector

- **Description:** Choosing a Sabbath opens the seven civil days of its Sunday–Saturday week with labels from each adapter.
- **Priority:** Must
- **Dependencies:** REQ-CHR-005
- **Acceptance criteria:** Sabbath #1’s Saturday row is Julian 4 April 33 and Gregorian 2 April 33. The other six days show their own dates and do not show a different Sabbath number.
- **Verification:** Test

### REQ-UI-006 — Confidence display

- **Description:** Each label shows one of: mathematically exact, algorithmically calculated, historically documented, historically reconstructed, uncertain, unsupported.
- **Priority:** Must
- **Dependencies:** REQ-CAL-005
- **Acceptance criteria:** Sabbath index and JDN are “mathematically exact”. Proleptic Gregorian and Julian labels are “algorithmically calculated” and, where a historical civil calendar differs, the UI also says so. Hebrew and Chinese never show “historically documented” unless a source is attached to that marker. Unsupported is visually distinct.
- **Verification:** Inspection, Demonstration

### REQ-UI-007 — Shareable URL

- **Description:** `?n=<canonical>` reproduces the same cursor. Optional `scale` selects a zoom id. Invalid values show an error.
- **Priority:** Must
- **Dependencies:** REQ-MATH-003, ADR-013
- **Acceptance criteria:** `?n=-1` and `?n=103992` match the oracles. `?n=01` is invalid. No network is required to interpret `n`.
- **Verification:** Test

### REQ-UI-008 — Accessible, responsive shell

- **Description:** Keyboard access to search, next/previous Sabbath, zoom, and the proof card. Focus is visible. The layout works at about 360px width and at desktop width. Motion respects `prefers-reduced-motion`. Text alternatives exist for the cross and the cursor.
- **Priority:** Must
- **Dependencies:** REQ-UI-001
- **Acceptance criteria:** A keyboard-only pass can move from #1 to #−1 and read the proof card. At 360px the rows stack or become a single scrollable aligned list without losing the shared cursor. Wireframes in doc 11 are the visual target.
- **Verification:** Demonstration, Inspection

### REQ-UI-009 — Historical markers

- **Description:** Optional markers may be placed on an absolute day. Each marker has a title, a source citation, and an uncertainty class. Markers do not alter arithmetic. MVP may ship with zero markers plus the anchor marker.
- **Priority:** Should
- **Dependencies:** REQ-UI-006
- **Acceptance criteria:** The anchor marker is present and classified PROJECT-DEFINED. A test marker with no source is rejected by the marker schema.
- **Verification:** Test

### REQ-UI-010 — Legacy month grid

- **Description:** The parchment month page is not the V2 shell. It may be reimplemented later as a secondary inspector.
- **Priority:** Later
- **Dependencies:** REQ-UI-001
- **Acceptance criteria:** MVP navigation does not depend on a month table.
- **Verification:** Inspection

## 3.5 AI

### REQ-AI-001 — Calendar works with no AI

- **Description:** Search, navigation, conversion, and the proof card do not call a model or a paid API.
- **Priority:** Must
- **Dependencies:** none
- **Acceptance criteria:** The static build contains no API key and no chat endpoint. Tests run offline.
- **Verification:** Inspection

### REQ-AI-002 — Models are not the chronology authority

- **Description:** No agent, bot, or future chatbot may change the anchor, the formula, or a displayed Sabbath number. A future chatbot may only explain values returned by the core and adapters.
- **Priority:** Must
- **Dependencies:** REQ-AI-001, REQ-CHR-001
- **Acceptance criteria:** Architecture review of any later chat design shows a tool boundary. This MVP has no chat tool.
- **Verification:** Inspection

### REQ-AI-003 — Auditor is opt-in and non-writing by default

- **Description:** The Sabbath Calendar Auditor is specified, not enabled, in this phase. If later enabled, it may read, test, and report. It may not merge, deploy, or edit the anchor.
- **Priority:** Later
- **Dependencies:** REQ-AI-002
- **Acceptance criteria:** This repository contains no automation configuration after the design phase. Doc 18 lists the permission ceiling.
- **Verification:** Inspection

### REQ-AI-004 — Public chatbot is out of MVP

- **Description:** No in-app conversational assistant in the MVP.
- **Priority:** Later
- **Dependencies:** REQ-AI-001
- **Acceptance criteria:** UI has no chat panel.
- **Verification:** Inspection

## 3.6 Verification and performance

### REQ-QA-001 — Property tests

- **Description:** For a deterministic sample of valid `n`, including negatives, zero, and values above `2^53`: `SabbathJDN(n+1) - SabbathJDN(n) = 7` and `SabbathNumber(SabbathJDN(n)) = n`.
- **Priority:** Must
- **Dependencies:** REQ-CHR-002, REQ-CHR-004
- **Acceptance criteria:** The suite fails on a broken step or a broken inverse. Sample size and seed are recorded in the test log.
- **Verification:** Test

### REQ-QA-002 — Independent check, not a renamed copy

- **Description:** At least one test computes `1733205 + 7*(n-1)` in a second, obviously local expression (or a second language in the Auditor’s manual kit) and compares. Adapter tests compare to published examples, not only to the adapter’s own inverse.
- **Priority:** Must
- **Dependencies:** REQ-QA-001, REQ-CAL-003, REQ-CAL-004
- **Acceptance criteria:** The Gregorian/Julian oracles include the values in REQ-CAL-001 and REQ-CAL-002. Hebrew and Chinese tests cite an external table row. A comment in the test names the source.
- **Verification:** Test, Inspection

### REQ-QA-003 — Release gate

- **Description:** A release candidate is blocked if REQ-CHR-009, REQ-QA-001, or any Must calendar round-trip fails. An AI report cannot waive a failed test.
- **Priority:** Must
- **Dependencies:** REQ-QA-001
- **Acceptance criteria:** The roadmap’s definition of done points at this ID. The Auditor’s instructions say a failed test stays failed.
- **Verification:** Inspection

### REQ-PERF-001 — Constant-time index

- **Description:** Computing a Sabbath from `n`, or `n` from a Sabbath JDN, is a fixed number of arithmetic operations, not a loop over weeks.
- **Priority:** Must
- **Dependencies:** REQ-CHR-002
- **Acceptance criteria:** Code inspection plus a timing smoke test that `|n| = 10^6` and `|n| = 1` are the same operation count (not the same wall clock down to the microsecond; BigInt cost may grow with digit length). No `for`/`while` over the interval.
- **Verification:** Inspection, Test

### REQ-PERF-002 — Input cap

- **Description:** Canonical decimal inputs longer than 10,000 digits are rejected. This includes the minus sign as a character but the digit count is 10,000.
- **Priority:** Must
- **Dependencies:** REQ-MATH-003
- **Acceptance criteria:** 10,000 digits is accepted if it is a canonical integer. 10,001 digits is rejected with `input-too-large`. The cap is a named constant.
- **Verification:** Test
- **Label:** PROPOSED cap (ADR-015)

### REQ-PERF-003 — Static hosting

- **Description:** The MVP build is static files. No database and no mandatory server.
- **Priority:** Must
- **Dependencies:** ADR-008
- **Acceptance criteria:** Opening the built `index.html` (or the static host equivalent) runs search and proof for the oracles with the network blocked after load.
- **Verification:** Demonstration

## 3.7 Explicitly out of scope for these IDs

- Choosing a Creation date (see doc 08). No requirement will assign one.
- Sunset instants (Phase 6).
- Editing `ksaday/ChristianCalendar`.
- Installing `.cursor/rules` during the design phase.
