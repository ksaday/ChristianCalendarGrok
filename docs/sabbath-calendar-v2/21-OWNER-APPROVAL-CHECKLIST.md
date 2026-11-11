# 21 — Owner Approval Checklist

Design status after this writing pass: documents exist under `docs/sabbath-calendar-v2/`. No application source, no `.cursor/rules`, no automation, no commit, unless you ask for a commit after this checklist.

Mark each line when you decide it. “Accept” means the ADR or the default in the design stands. “Reject” means stop and revise that document before any code.

## A. Foundation

- [ ] **A1.** Sabbath #1 remains Julian 4 April AD 33, JDN `1733205`. (Required. A reject here stops the project as specified.)
- [ ] **A2.** The formula `SabbathJDN(n) = 1733205 + 7*(n-1)` is the only numbering rule, including for `0` and negative `n`.
- [ ] **A3.** Legacy oracles stay: 11 April 2026 → JDN `2461142` → Sabbath `#103992`.
- [ ] **A4.** Sketch labels `#104017` and `10/3/2026` are not oracles.

## B. Repository and stack

- [ ] **B1.** Accept ADR-001 Option C: leave `ksaday/ChristianCalendar` unchanged; build V2 in `ChristianCalendarGrok`.
- [ ] **B2.** Accept ADR-002: `bigint`, floor division, Euclidean modulo, canonical decimal strings.
- [ ] **B3.** Accept ADR-008: TypeScript, `node:test`, no UI framework, no database, no mandatory backend.
- [ ] **B4.** Accept ADR-015: 10,000-digit input cap. Reject if you want a different cap (write the number here: ________).

## C. Calendars

- [ ] **C1.** Gregorian adapter is proleptic and does not skip 5–14 October 1582 inside the JDN function (ADR-007).
- [ ] **C2.** Julian adapter exists so Sabbath #1 displays as 4 April 33 CE Julian.
- [ ] **C3.** Hebrew row is the calculated calendar only, sunset mismatch disclosed, no ancient observational dates (REQ-CAL-003).
- [ ] **C4.** Chinese row may ship with an empty verified range and a visible unsupported state (REQ-CAL-004).
- [ ] **C5.** No adapter is allowed to change `n`.

## D. UI

- [ ] **D1.** Timeline shell matches doc 11, row order Sabbath, Jewish, Gregorian, Chinese, Julian.
- [ ] **D2.** Proof card and cycle inspector are engine output, not prose from a model.
- [ ] **D3.** URLs use `n`, optional `d` for non-Sabbaths, optional `scale` (ADR-013 as updated).
- [ ] **D4.** Month grid is not the MVP shell.

## E. Boundaries that stay out of the core

- [ ] **E1.** No Creation date is chosen (doc 08). Mode B stays unbuilt.
- [ ] **E2.** Sunset calculation is Phase 6 and is not an input to `n` (ADR-010).
- [ ] **E3.** MVP has no public chatbot (ADR-011, REQ-AI-004).
- [ ] **E4.** Auditor automations are not created until a later yes (Phase 5).

## F. Rules and publication

- [ ] **F1.** Doc 16’s `.mdc` texts are proposals only. Do not install them until you check this line.
- [ ] **F2.** Do not commit or push this design set until you check this line and say so in chat.
- [ ] **F3.** Do not open a pull request until you check this line.

## G. Phase start

- [ ] **G1.** Phase 0 fixture work may begin after A, B, and F2 are decided. Coding the core still waits on A and B.
- [ ] **G2.** Hebrew or Chinese dates may be filled only with citations you have seen.
- [ ] **G3.** Phase 5 waits on your plan tier (Premium and above for Automations, as of 11 November 2026) and on E4.

## How to use this list

Reply with the item ids you accept or reject. Silence is not acceptance of F2 or F3. The design documents already treat A1–A3 as fixed project constraints from your brief. B1–E4 are the recommendations you are being asked to confirm before code.

## What was not changed

- `ksaday/ChristianCalendar` was not cloned into this workspace and was not modified.
- No file outside `docs/sabbath-calendar-v2/` was added for this design, other than this documentation directory itself.
- No dependency was installed. No rule file was activated. No automation was saved.
