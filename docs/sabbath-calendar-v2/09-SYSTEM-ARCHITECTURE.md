# 09 — System Architecture

## 9.1 Shape

```text
ChristianCalendar (legacy, frozen)          ChristianCalendarGrok (this repo)
  index.html, css/style.css, js/calendar.js
  oracle source only  -------------------->  tests copy oracle numbers
                                              src/chronology     BigInt core
                                              src/calendars      adapters
                                              src/ui             timeline
                                              index.html         static shell
                                              docs/sabbath-calendar-v2
```

Runtime, after a later implementation:

```text
browser
  loads static HTML, CSS, JS
  parses ?n=
  calls chronology (local)
  calls adapters (local)
  paints rows from one cursor bigint
```

No process sits between the user and the arithmetic in MVP (REQ-PERF-003, REQ-AI-001).

## 9.2 Components

| Component | Responsibility | Must not |
| --- | --- | --- |
| `chronology` | `n` ↔ JDN, cycle, invariants, canonical integers | Import adapters, touch DOM, read the clock |
| `calendars/gregorian` | Proleptic Gregorian ↔ JDN | Export a Sabbath function |
| `calendars/julian` | Julian ↔ JDN | Assume a constant day offset from Gregorian |
| `calendars/hebrew` | Calculated Hebrew ↔ JDN inside a verified range | Claim observation or sunset accuracy |
| `calendars/chinese` | Lunisolar ↔ JDN inside a verified range, or always unsupported | Invent dates to fill the row |
| `calendars/anniversary` | Legacy 4 April anniversary using Gregorian + core | Live in the core |
| `ui/cursor` | Holds `j` and derived `n`, zoom id, URL sync | Convert JDN through `Number` |
| `ui/timeline` | Pixels, labels, cross, row chrome | Compute `n` inline |
| `ui/proof` | Renders the invariant report and labels | Compose its own formula result |
| `ui/search` | Parses queries into core or adapter calls | Catch `unsupported` and substitute today |
| `notes` | Static cited annotations | Modify `j` |
| Future `sunset` | Local rise/set | Feed the core |
| Future `assistant` | Explain tool output | Run in MVP |

## 9.3 Data flow for a jump

1. User enters `-1` or opens `?n=-1`.
2. `validateSabbathNumber` accepts it.
3. `sabbathNumberToAbsoluteDay` returns `1733191n`.
4. Cursor stores `{ n: -1n, j: 1733191n }`.
5. Each adapter `fromAbsoluteDay(1733191n)` returns a label or `unsupported`.
6. Proof card calls `verifyChronologyInvariants([-1n])`.
7. Timeline recenters. Pixel positions use a safe delta from `j` only after zoom chooses a window that fits in a safe integer delta. If the user is zoomed to “day”, the window is a few dozen days and the delta is safe. If they are zoomed out, tick labels are computed by stepping `bigint` weeks or years using adapter helpers, not by `Math.pow`.

Search by Gregorian or Julian date reverses steps 3–4: adapter produces `j`, then `absoluteDayToSabbathCycle` produces the Saturday of that week (the cursor snaps to the Sabbath, and the inspector can still show the searched weekday). **PROPOSED snap:** date search selects that absolute day, and the Sabbath row highlights the cycle Sabbath. The cursor identity is the searched day if the user asked for a date, and the Sabbath if they asked for `n`. The proof card shows the cycle when the day is not itself a Sabbath. This avoids pretending Wednesday has a fractional Sabbath number (REQ-CHR-004).

## 9.4 URL

```text
?n=-1
?n=103992&scale=week
?d=1733204
```

`scale` is an id in `{day, week, year, century, millennium, age}`. Unknown scale is ignored and the default `week` is used. Unknown or illegal `n` or `d` replaces the timeline with a validation message and does not call adapters. `d` is a canonical absolute-day integer for a day that might not be a Sabbath. If both `n` and `d` are present and disagree, the page shows an error and does not choose one. No other query keys are read in MVP. Extra keys are ignored so the page cannot be turned into a general expression evaluator.

## 9.5 Build and hosting

- Language: TypeScript, strict mode, `bigint` isolated as described in doc 10.
- Tests: `node:test`, run without a browser. UI view-model tests run in Node by keeping paint functions separate from DOM wiring.
- Output: static files. GitHub Pages, Netlify, or any file host is enough, matching the legacy README’s hosting story.
- Dependencies: none required for arithmetic. A dev-only TypeScript compiler is allowed. Runtime dependencies need an ADR.
- Secrets: none in MVP.

## 9.6 Error boundaries

| Layer | Failure | User sees |
| --- | --- | --- |
| Parse | Bad `n` | “That Sabbath number is not valid.” The previous cursor stays if one existed; on first load, no fake cursor |
| Core invariant | Report failure | The proof card shows failure. The footer pattern from the legacy page applies: do not trust displayed numbers. Prefer hiding labels over showing them beside a failed invariant |
| Adapter | `unsupported` | That row shows “Unsupported for this day” and the reason |
| Adapter | `invalid-date` on search | The search box explains the date does not exist in that calendar |
| Cap | `input-too-large` | Ask for a shorter number. Do not compute |

A failed invariant is a red stop. An unsupported adapter is a normal gray row. They are different.

## 9.7 Repository boundary

Implementation PRs target `ChristianCalendarGrok` only. A PR that changes `ksaday/ChristianCalendar` is out of scope unless the owner opens that repository on purpose. The legacy commit to cite is `6229702`.

Suggested later tree (not created in the design phase):

```text
src/chronology/
src/calendars/
src/ui/
test/chronology/
test/calendars/
test/ui/
index.html
```

## 9.8 Optional later processes

None of these run in the static MVP:

- Cursor Automation for the Auditor (docs 17–18), on the owner’s Cursor account, outside the website process.
- A small server only if a future chatbot is approved (doc 19), holding the API credential.
- A sunset worker is still in-browser math plus a timezone dataset, and only in Phase 6. It is not required to justify a backend.
