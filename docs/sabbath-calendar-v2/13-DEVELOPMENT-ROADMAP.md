# 13 — Development Roadmap

No phase in this document is authorized to start coding until the owner accepts the checklist in [21-OWNER-APPROVAL-CHECKLIST.md](21-OWNER-APPROVAL-CHECKLIST.md). This design phase is the planning work that precedes Phase 0’s code. Writing these Markdown files is not Phase 0.

MVP is Phases 0 through 4. Phase 5 and Phase 6 are optional.

Complexity words: Low, Medium, High. They are relative to one implementer who already has this design.

## Phase 0 — Existing repository assessment

- **Scope:** Freeze the oracle pack from doc 12 into comments or fixtures in this repo. Do not edit ChristianCalendar. Confirm commit `6229702` still matches the cited line references if that repo moves; if it moves, update doc 02 rather than changing oracles silently.
- **Dependencies:** Design acceptance.
- **Deliverables:** `test/oracles.json` (canonical strings) and a one-page note in the V2 README pointing at the legacy URL. README text is a later implementation edit, not part of this design commit unless the owner asks.
- **Complexity:** Low.
- **Risks:** Copying a sketch number (`104017`) into the oracle file. Guard: review against doc 12 only.
- **Acceptance:** Every O-* row from doc 12 is present. No Sabbath formula is “corrected”.
- **Done:** Fixtures match doc 12 and a test loads them.
- **Owner boundary:** Stop after fixtures if the owner wants another look before the core.

## Phase 1 — Arbitrary-precision chronology core

- **Scope:** `src/chronology/**` and its tests. Functions in doc 06. No UI.
- **Dependencies:** Phase 0 fixtures.
- **Deliverables:** Core module, property tests, invariant report.
- **Complexity:** Medium. The arithmetic is small. The negative-division discipline is the whole risk.
- **Risks:** Truncating `BigInt` division. Unicode minus. Skipping 0.
- **Acceptance:** REQ-CHR-001 through REQ-CHR-009, REQ-MATH-001 through REQ-MATH-004, REQ-QA-001, REQ-PERF-001, REQ-PERF-002.
- **Done:** `node --test` green on the core. A 20-digit round trip passes.
- **Owner boundary:** Do not start adapters until the core gate is green.

## Phase 2 — Calendar adapters

- **Scope:** Gregorian, Julian, Hebrew, Chinese, anniversary helper, metadata.
- **Dependencies:** Phase 1.
- **Deliverables:** Adapters. Gregorian and Julian complete. Hebrew and Chinese either cited or honestly unsupported.
- **Complexity:** Medium for Gregorian and Julian. High for Hebrew and Chinese if a verified range is filled in the same phase.
- **Risks:** Invented ancient dates. Constant “minus two days” Julian shortcut. Importing a library that also exports a “sabbath” helper and using that helper.
- **Acceptance:** REQ-CAL-001 through REQ-CAL-006, REQ-MATH-005, REQ-QA-002.
- **Done:** Anchor shows both civil labels. Unsupported paths tested. No core diff except bug fixes that keep the formula.
- **Owner boundary:** Shipping Chinese as unsupported is allowed. Shipping Chinese with dates requires the owner to see the citations first.

## Phase 3 — Multi-calendar timeline UI

- **Scope:** Static shell, search, zoom, synchronized rows, proof card, cycle inspector, URL, keyboard, mobile layout from doc 11.
- **Dependencies:** Phase 2.
- **Deliverables:** Usable static site in this repository.
- **Complexity:** High, because of scale and `bigint` pixels, not because of a framework.
- **Risks:** Float coercion of JDNs. Rows that drift. Hover-only proof card.
- **Acceptance:** REQ-UI-001 through REQ-UI-009, REQ-PERF-003, REQ-AI-001, REQ-AI-004.
- **Done:** Demonstrations in doc 12 §12.8 step 4.
- **Owner boundary:** Visual tweaks that do not change row order or the anchor can follow. Changing row order needs an ADR update.

## Phase 4 — Verification infrastructure

- **Scope:** Wire the full suite into a single command. Document the release gate in the V2 README. Add the import-boundary check.
- **Dependencies:** Phases 1–3, so the gate covers UI view-models too. Unit tests from earlier phases already exist; this phase makes the gate official.
- **Deliverables:** One command, documented failure behavior, CI only if the owner wants a host. CI is optional and must not be a paid service the project requires.
- **Complexity:** Low.
- **Risks:** A green badge that skips property tests.
- **Acceptance:** REQ-QA-003. A deliberately broken anchor on a branch fails the command.
- **Done:** The failure message tells a human not to trust the numbers, echoing the legacy footer.
- **Owner boundary:** No release tag without this gate.

## Phase 5 — Grok Bot auditor integration

- **Scope:** Optional. Create the Auditor automation only after a separate yes. Docs 17 and 18 are the spec.
- **Dependencies:** Phase 4, and a Cursor plan that includes Automations (Premium and above as of the 11 November 2026 pricing page).
- **Deliverables:** One opt-in automation, read/report first, no merge rights required.
- **Complexity:** Low to medium, mostly prompt and permissions.
- **Risks:** The bot “fixes” the anchor. Usage-pool cost. A helper model being used for browser tools (Cursor forum note, November 2026: computer-use helpers may run on other models unless Testing is disabled).
- **Acceptance:** REQ-AI-002, REQ-AI-003. A dry run that finds a planted missing citation and does not commit.
- **Done:** The owner has a written report and the anchor file is unchanged by that run.
- **Owner boundary:** This phase does not start from MVP completion automatically.

## Phase 6 — Optional extensions

Separate approvals for each item.

| Item | Scope | Dependency | Complexity | Risk | Done when |
| --- | --- | --- | --- | --- | --- |
| Sunset module | Coordinates in, instants out, core untouched | Phase 3 | High | High latitudes, historical TZ | A place with no sunset returns a clear error and the Sabbath number stays |
| More calendars | New adapter | Phase 2 interface | Varies | Invented dates | Doc 07 §7.9 checklist |
| Creation study | Overlay module | Owner picks a cited model | Medium | Fake precision | Marker visible, core constant unchanged, default overlay off |
| Public assistant | Server + tool calls | Phase 4, cost approval | High | Keys in the browser, invented dates | Doc 19 threat list addressed, or the item stays unbuilt |

## MVP definition

A static site in `ChristianCalendarGrok` that:

- Implements the formula for negative, zero, and positive `n` with `bigint`.
- Shows Gregorian and Julian labels for the anchor correctly.
- Shows Hebrew and Chinese without fabricating unsupported dates.
- Proves each selected Sabbath from the engine.
- Passes the Phase 4 gate.
- Does not call an AI API.

## Suggested implementation order inside MVP

Phase 0 fixtures, Phase 1 core, Gregorian and Julian, UI against those two rows with Hebrew and Chinese visibly unsupported, then fill Hebrew or Chinese only with citations, then Phase 4 as the gate. That order keeps a true timeline on screen before the hardest calendars.
