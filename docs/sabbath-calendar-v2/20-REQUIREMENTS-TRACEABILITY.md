# 20 — Requirements Traceability

Every requirement in [03-SOFTWARE-REQUIREMENTS-SRS.md](03-SOFTWARE-REQUIREMENTS-SRS.md) maps to an architecture home and a verification home. Test names in the future suite should use the requirement id (`REQ-CHR-001`) so this table does not need a second numbering system. Cases are detailed in [12-VERIFICATION-AND-TEST-PLAN.md](12-VERIFICATION-AND-TEST-PLAN.md).

| Requirement | Priority | Design home | Verification |
| --- | --- | --- | --- |
| REQ-CHR-001 | Must | ADR-001, ADR-002, doc 04 §4.3, doc 06 `sabbathNumberToAbsoluteDay` | Test anchor constant and JDN of `n = 1`. Inspection: constant has one definition |
| REQ-CHR-002 | Must | Doc 04 §4.3, doc 06 | Property test, step of 7. Inspection: no week loop |
| REQ-CHR-003 | Must | Doc 05 | Tests for #0, #−1, #−2 and the increasing sequence |
| REQ-CHR-004 | Must | Doc 04 §4.3, doc 06 inverse | Non-Sabbath returns `not-a-sabbath` |
| REQ-CHR-005 | Must | Doc 04 §4.8, ADR-004 | Sunday offset 0, Saturday offset 6, cycle number |
| REQ-CHR-006 | Must | Doc 06 interval functions | Signed interval #−2 to #3 is 5 Sabbaths and 35 days |
| REQ-CHR-007 | Must | ADR-005, doc 09 component table | Import-graph test or lint |
| REQ-CHR-008 | Must | Doc 06 `verifyChronologyInvariants` | Report passes on `{1,0,-1,103992}` |
| REQ-CHR-009 | Must | Doc 02 §2.5, doc 04 §4.5 | Legacy oracles: 2461142, 103992, anniversary year 33 |
| REQ-MATH-001 | Must | ADR-002 | Public APIs reject `number`. Round trip above `2^53` |
| REQ-MATH-002 | Must | Doc 04 §4.3 | `floorDiv(-8,7) = -2`, `euclidMod(-1,7) = 6` |
| REQ-MATH-003 | Must | Doc 04 §4.9, doc 10 §10.2 | Accept/reject table in doc 05 §5.5 |
| REQ-MATH-004 | Must | Doc 04 §4.3 | Anchor weekday 5. Next day 6. Previous day 4 |
| REQ-MATH-005 | Must | Doc 04 §4.7 | Year 0 ↔ 1 BCE, year −1 ↔ 2 BCE |
| REQ-CAL-001 | Must | Doc 07 §7.3, ADR-007 | 11 Apr 2026, 2 Apr 33, 15 Oct 1582, BCE round trip |
| REQ-CAL-002 | Must | Doc 07 §7.4 | Julian 4 Apr 33 = 1733205, distinct from Gregorian label |
| REQ-CAL-003 | Must | Doc 07 §7.5 | Three cited vectors before done, else not done. Unsupported outside range |
| REQ-CAL-004 | Must | Doc 07 §7.6 | Empty verified range allowed. Leap-month vector required if the range is filled |
| REQ-CAL-005 | Must | Doc 07 §7.2, ADR-005 | Wrong label does not change `n` |
| REQ-CAL-006 | Should | Doc 07 §7.3 anniversary | Year 33 anniversary is the anchor. Negative thousands are not milestones |
| REQ-CAL-007 | Later | Doc 07 §7.9 | Inspection when a fifth calendar is proposed |
| REQ-UI-001 | Must | ADR-012, doc 11 | View-model: one cursor JDN across five rows |
| REQ-UI-002 | Must | Doc 09 §9.3, doc 11 | Search oracles #−1, #0, #1, #103992 |
| REQ-UI-003 | Must | Doc 06 §6.5, doc 10 §10.9 | Layout scale does not change `j` |
| REQ-UI-004 | Must | Doc 10 §10.7, doc 11 | Proof fields come from core types |
| REQ-UI-005 | Must | Doc 04 §4.8, doc 11 | Sabbath #1 week shows both civil labels on Saturday |
| REQ-UI-006 | Must | Doc 07 §7.10 | Confidence class per row |
| REQ-UI-007 | Must | ADR-013, doc 10 §10.6 | `n=-1`, `n=103992`, reject `n=01`, conflicting `n` and `d` |
| REQ-UI-008 | Must | Doc 11 mobile wireframe | Keyboard path and 360px inspection |
| REQ-UI-009 | Should | Doc 10 §10.8 | Marker without source is rejected. Anchor marker is project-defined |
| REQ-UI-010 | Later | ADR-012 | MVP has no month-grid dependency |
| REQ-AI-001 | Must | ADR-011, doc 19 | No chat UI, no key, tests offline |
| REQ-AI-002 | Must | Doc 16 rules, doc 19 | Inspection of any later tool boundary |
| REQ-AI-003 | Later | Doc 17, doc 18 | No automation config in the design-phase tree |
| REQ-AI-004 | Must for MVP scope | ADR-011 | No chat panel |
| REQ-QA-001 | Must | Doc 12 §12.3 | Property sample, fixed seed |
| REQ-QA-002 | Must | Doc 12 §12.4 | Second expression for the formula. External vectors for Hebrew/Chinese |
| REQ-QA-003 | Must | Doc 12 §12.8, doc 13 | Failed test blocks release. Auditor text cannot waive it |
| REQ-PERF-001 | Must | Doc 06 §6.3 | Inspection plus smoke timing note |
| REQ-PERF-002 | Must | ADR-015 | 10,000 digits accepted, 10,001 rejected |
| REQ-PERF-003 | Must | Doc 09 §9.5 | Static open, network blocked after load |

## Coverage check

- Every Must requirement has a test or an inspection.
- Later requirements are inspected when that phase starts. They are not MVP exit tests, except REQ-AI-003 and REQ-AI-004, which are MVP inspections that the unwanted feature is absent.
- REQ-CAL-003 and REQ-CAL-004 are Must, with an honest empty-range path for Chinese. “Empty and labeled unsupported” satisfies REQ-CAL-004. “Numbers with no citation” fails it.
- No requirement assigns a Creation JDN. Doc 08 is traced by the absence of such an ID, on purpose.

## Owner-facing decisions that are not requirements

Tracked in doc 21, not as REQs, because they are approvals rather than software behavior:

- Commit this design set or leave it uncommitted.
- Install `.cursor/rules` from doc 16.
- Enable any Auditor routine.
- Fill the Hebrew and Chinese verified ranges.
- Add a Mode B Creation marker.
