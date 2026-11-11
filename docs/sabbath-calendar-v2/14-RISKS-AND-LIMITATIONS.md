# 14 — Risks and Limitations

## 14.1 Mathematical limits (honest)

- The formula is defined for every integer. A browser tab is not. The 10,000-digit cap (ADR-015) is a product limit. Beyond it the software refuses. That is not a flaw in the seven-day line.
- `bigint` time and memory grow with digits. Normal Sabbath numbers (even millions of weeks) are tiny. The cap is far above any timeline a person can scroll.
- Pixel positions are floating point. They are not the chronology. At coarse zoom the UI must recenter using `bigint` origins (doc 06 §6.5). If someone later “simplifies” hit-testing by inverting a float scale, distant days will be off by more than a day. That bug is release-blocking if it can change `n`.
- Floor division must be used everywhere. One truncating quotient in a negative year is enough to move a date.

## 14.2 Calendar limits

| Topic | Limitation |
| --- | --- |
| Gregorian adapter | Proleptic. It is not a map of which kingdom used which calendar in which year |
| 1582 and later reforms | Noted, not encoded as skipped JDNs |
| Julian adapter | Algorithmic leap rule, including year 0. Not a statement about local adoption |
| Hebrew | Calculated civil calendar only, and only inside a cited range. Not first-century observation. Sunset can disagree by an evening |
| Chinese | No dates until a table is cited. Ancient regional calendars are out of range even after a modern table is added |
| Weekday continuity | The project assumes the seven-day cycle of the JDN weekday, which is continuous through the Gregorian reform in the arithmetic sense. That is not a sociological claim about every community |
| “Today” | Depends on the device timezone. The selected civil date, once chosen, does not |

## 14.3 Historical and theological limits

- Sabbath #1 is **PROJECT-DEFINED**. The legacy README points at Humphreys & Waddington (1983) and NASA eclipse tables as background. This design does not re-derive their crucifixion date and does not upgrade it to “proved by the software”.
- Extrapolating `n` backward does not prove the Sabbath was kept on those dates, and does not locate Creation (doc 08).
- Markers without citations are forbidden. The absence of a marker is not evidence that nothing happened.
- Anniversary Sabbaths follow the Gregorian 4 April rule. They drift relative to a pure “every 52 Sabbaths” count because a solar year is not 52 weeks. The legacy README already says this. V2 must not “fix” that by renumbering.

## 14.4 Product and engineering risks

| Risk | Mitigation |
| --- | --- |
| Editing the legacy repo by mistake | ADR-001. PRs target ChristianCalendarGrok. Doc 16 forbids anchor edits without the owner |
| Sketch numbers treated as oracles | Doc 11 and doc 12 name the real oracles |
| A dependency that computes its own Sabbath | Adapters are written as thin conversions. Libraries, if ever added, are wrapped and tested against the core, not the other way around |
| UI framework creep | ADR-008. A framework needs a new ADR |
| Accessibility skipped on a canvas timeline | Rows are DOM elements with text, not only a bitmap. Doc 11 keyboard table is in the Phase 3 acceptance list |
| Negative URL parsing | Tests for `n=-1` and `n=01` |
| Agent “helpfulness” | Rules in doc 16. Auditor cannot merge |

## 14.5 AI and cost limits

Checked against Cursor’s public models and pricing page on 11 November 2026. Prices and plan names change. Re-read https://cursor.com/docs/models-and-pricing before enabling Phase 5.

- Plus includes Cursor models (Grok 4.7, 4.6, 4.5, Composer 2.5) and does **not** include Cloud Agents, Automations, or Grok Bot.
- Premium and above add Cloud Agents, Automations, and Grok Bot. They draw from the same weekly usage pool as editor agents. They are not free and not unlimited.
- Third-party models bill separately at list price from prepaid credit on Premium and above.
- The Grok API bills separately and does not draw from that pool. It is the wrong assumption for a public website.
- Automations that open a browser may invoke other helper models unless environment Testing is disabled (Cursor forum, November 2026). An Auditor that must stay on one model should not rely on those helpers, or the owner accepts the extra usage.
- A model can misstate a calendar conversion with confidence. The test gate exists because of that.

## 14.6 Security limits of the MVP

- Static files have no account and no secret. That is the security model.
- Search strings are parsed as dates or integers only. They are not evaluated as JavaScript.
- There is no user-generated content store, so there is no stored XSS vector from other users. DOM text should still be set as text, not as HTML from a query string.
- A future chatbot (Phase 6) changes this model. It needs a server, rate limits, and a key that is not in the repo. Until then, do not add a key “for later”.

## 14.7 Known gaps left UNRESOLVED

1. Which published Hebrew vectors will fill `verifiedRange`.
2. Which Chinese table, if any, will be adopted.
3. Country-by-country Gregorian reform dataset.
4. Whether the 10,000-digit cap should be smaller.
5. Whether milestones should include negative multiples of 1000.
6. The owner’s Cursor plan tier, which decides whether Phase 5 is even available.
7. Whether this design set should be committed.

None of these block understanding the core. Items 1 and 2 block claiming those rows are dated. Item 7 blocks publication, not local writing.
