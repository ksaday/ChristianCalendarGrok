# 12 — Verification and Test Plan

## 12.1 Gate

MVP cannot ship if any Must test in this document fails. A review comment, an Auditor report, or a model explanation does not flip a failure to a pass (REQ-QA-003).

Runner: Node’s built-in test runner. Property samples use a tiny deterministic xorshift (or similar) with a seed printed in the test name or log, so a failure can be replayed. No network during unit tests.

Fixtures store integers as canonical strings.

## 12.2 Oracle pack (copy into tests, do not import the legacy repo)

| Id | Input | Expected | Label |
| --- | --- | --- | --- |
| O-1 | Sabbath #1 | JDN `1733205`, weekday 5 | PROJECT-DEFINED / VERIFIED |
| O-0 | Sabbath #0 | JDN `1733198` | MATHEMATICALLY DERIVED |
| O-N1 | Sabbath #−1 | JDN `1733191` | MATHEMATICALLY DERIVED |
| O-N2 | Sabbath #−2 | JDN `1733184` | MATHEMATICALLY DERIVED |
| O-3 | Sabbath #3 | JDN `1733219` | MATHEMATICALLY DERIVED |
| O-2026 | proleptic Gregorian 11 April 2026 | JDN `2461142`, Sabbath #103992 | VERIFIED |
| O-GREG-ANCHOR | proleptic Gregorian 2 April 33 | JDN `1733205` | VERIFIED |
| O-JUL-ANCHOR | Julian 4 April 33 | JDN `1733205` | VERIFIED |
| O-JUL-0 | Sabbath #0 | Julian 28 March 33 | MATHEMATICALLY DERIVED (same formula, round-trips) |
| O-JUL-2 | Sabbath #2 | Julian 11 April 33 | MATHEMATICALLY DERIVED (same formula, round-trips) |
| O-1582 | proleptic Gregorian 15 October 1582 | JDN `2299161` | VERIFIED |
| O-N12 | Sabbath #−12 | JDN `1733114`, Gregorian 1 January 33 | VERIFIED in baseline |
| O-N13 | Sabbath #−13 | JDN `1733107`, Gregorian 25 December 32 | VERIFIED in baseline. This is the first Sabbath the legacy UI cannot navigate to |

## 12.3 Property tests (REQ-QA-001)

Seed recorded. At least:

- 200 random values in `[-10^6, 10^6]`, always including the oracle pack.
- 20 values with more than 15 digits, both signs (above `2^53`).
- The digit-cap boundary (REQ-PERF-002).

For each valid `n`:

```text
SabbathJDN(n + 1) - SabbathJDN(n) = 7
SabbathNumber(SabbathJDN(n)) = n
Weekday(SabbathJDN(n)) = 5
getNextSabbath(n) = n + 1
getPreviousSabbath(n) = n - 1
```

Including pairs that cross 0: `n = -1, 0, 1`.

## 12.4 Independent expression (REQ-QA-002)

One test file computes expected JDNs as `1733205n + 7n * (n - 1n)` locally and compares to the core. The core module under test should be imported, and the expected expression should be written in the test, not imported from a hidden helper that the core also uses. That is the “second spelling” of the same definition. It catches wiring mistakes. It does **not** by itself prove the anchor is historically true. The test name must say that.

A stronger check, required for adapters:

- Gregorian and Julian oracles in §12.2 were reproduced by an independent Python floor-division script during design (doc 02). Re-run that logic in the test, or commit the vectors and cite doc 02.
- Hebrew: no vector is approved in this design. The test file ships with `verifiedCases = []` and a test that `fromAbsoluteDay` returns `unsupported` for JDN `1733205` while the list is empty. When a case is added, the same PR adds a citation comment (book, edition, page or algorithm id).
- Chinese: same empty-list pattern, plus, when non-empty, one leap-month case and one non-leap case.

## 12.5 Case list by theme

| Theme | Cases |
| --- | --- |
| Anchor | O-1. Constant equals `1733205n`. Inverse returns `1n` |
| Zero | O-0. Interval from 0 to 1 is 1 Sabbath and 7 days |
| Negative | O-N1, O-N2, O-N12, O-N13. Sequence −2..2 strict |
| Large positive | `n` with 20 digits round-trips. `#103992` oracle |
| Large negative | `n = -1000000` equals `1733205 + 7*(-1000001) = -5266802` (doc 06) |
| Consecutive cycles | 400 consecutive `n` around 0, step 7 |
| Inversion | Non-Sabbath `1733206` is `not-a-sabbath`. Cycle of that day is Sabbath #1, offset 0 if weekday math says Sunday. **Check during implementation:** weekday of `1733205` is 5 Saturday, so `1733206` is 6 Sunday, `dayOffset` 0, cycle Sabbath still #1. Lock that in the test rather than assuming from this sentence if the weekday function disagrees; if it disagrees, the weekday function is wrong |
| BCE / CE | Astronomical 0 displays `1 BCE`. Astronomical −1 displays `2 BCE`. Round trip |
| Negative JDN | `euclidMod` cases. A Sabbath with negative JDN (`-1000000` above) has weekday 5 |
| Julian / Gregorian | Same JDN, two labels, at the anchor. A second date away from year 33 where the day numbers differ, computed by both functions and compared to each other only after each matches a hand-checked vector |
| Reform boundary | O-1582. A test documents that proleptic 4 October 1582 is a different JDN from the historical Julian date and does not “skip” ten days inside the Gregorian function. Expected proleptic JDN for 4 October 1582 is `2299150` (**VERIFIED** in the baseline inspection) |
| Hebrew leap month | Only when a cited vector exists. Otherwise the empty-range test |
| Chinese leap month | Same |
| Unsupported | Hebrew and Chinese on the anchor while unverified. Digit overflow. 31 April |
| Serialization | Reject `01`, `1.0`, `1e3`, `+1`, U+2212, `-0`. Accept `-1`. Grouped display is not parsed raw |
| UI identity | View-model at `scale=age` and `n=-1` still has JDN `1733191`. Changing `pxPerDay` does not change `j` |
| URL | `n=-1`, `n=103992`, `n=01` invalid, `n` and `d` in conflict |
| Milestones | `1000` is a milestone. `0` and `-1000` are not |
| Anniversary | Year 33 returns anchor JDN |
| Isolation | Chronology sources do not import `calendars`, `Date`, or DOM |

The Sunday-offset sentence in the inversion row is **to be locked by the test**. The arithmetic is specified in doc 04: `Weekday(1733205) = 5`, so the next JDN is Sunday only if Sunday = 6 in `Weekday` and `dayOffset` = `(Weekday + 1) mod 7` = 0. That is consistent. The test should assert it directly.

## 12.6 Modulo cases

| Call | Expected |
| --- | --- |
| `floorDiv(-8, 7)` | `-2` |
| `floorDiv(8, 7)` | `1` |
| `floorDiv(-14, 7)` | `-2` |
| `euclidMod(-1, 7)` | `6` |
| `euclidMod(1733205, 7)` | `5` |
| `euclidMod(-5266802, 7)` | `5` |

The last row is the large negative Sabbath from doc 05 and must match `IsSabbath`.

## 12.7 Historical verification policy

| Class | How a test may use it |
| --- | --- |
| Project-defined anchor | Assert the constant. Do not “discover” a different one |
| Mathematically derived | Assert equality with the formula |
| Historical document | Quote only with a citation stored next to the vector. Not required for MVP arithmetic |
| Astronomical calculation | Not used for Sabbath `n` |
| Reconstructed ancient calendar | Not an MVP oracle |
| Theological interpretation | Not an oracle |

Humphreys & Waddington, *Nature* 306 (1983), remains the legacy README’s background citation for why the owner chose the anchor. Tests do not parse that paper and they do not treat it as a reason to change `1733205`.

## 12.8 Release procedure

1. `node --test` exits 0.
2. The invariant report for the oracle pack is all pass.
3. A reviewer confirms Hebrew/Chinese either have citations or return unsupported.
4. UI demonstration: `#1`, `#0`, `#-1`, `#103992`, bad input, mobile width, keyboard Home to #1.
5. If an Auditor report exists, disagreements become issues. They do not edit the constant in the same breath as the review.

## 12.9 Collaboration roles

| Step | Who |
| --- | --- |
| Implement | A later coding session in this repo |
| Deterministic tests | The suite in this document |
| Independent read | Optional Auditor (doc 18), different session, instructed to recompute samples and to look for missing citations |
| Decision | Owner, if a test and a narrative disagree |
| Forbidden | Any model marking a red test green by argument |
