# 05 — Negative Sabbath Numbering

## 5.1 Contract

Negative numbering is the same function as positive numbering. There is no second epoch, no “year zero Sabbath”, and no skipped index.

```text
SabbathJDN(n) = 1733205 + 7 * (n - 1)    for n in Z
```

**PROJECT-DEFINED** formula. The rows below are **MATHEMATICALLY DERIVED** and were re-checked with floor-division integers during the baseline inspection.

| n | JDN | Days from anchor | Proleptic Gregorian | Reachable in legacy UI? |
| ---: | ---: | ---: | --- | --- |
| 1 | 1733205 | 0 | 2 April 33 | Yes |
| 0 | 1733198 | −7 | 26 March 33 | Yes |
| −1 | 1733191 | −14 | 19 March 33 | Yes |
| −2 | 1733184 | −21 | 12 March 33 | Yes |
| −12 | 1733114 | −84 | 1 January 33 | Yes |
| −13 | 1733107 | −91 | 25 December 32 | No. `clampY` stops at year 33 |

Sabbath #0 is a real index. The legacy seal class `.sab-seal.pre` already styles `sn < 1`, so the original product visually admits numbers below 1. It does not admit navigation far enough to show most of them.

## 5.2 Why zero must stay

If an implementation “helpfully” mapped mathematical non-positive values onto 1, 2, 3, … or jumped from 1 to −1, then:

```text
SabbathJDN(0) would be missing
SabbathNumber(SabbathJDN(n)) would fail for every n <= 0
the step between #1 and the previous Saturday would be described as 1 index and 7 days, which is the #0 slot
```

The previous Saturday before Sabbath #1 is #0, not #−1. **#−1 is two weeks before #1.**

## 5.3 Negative division

The dangerous port is:

```text
legacy:  (j - ANCHOR) / 7 + 1          // IEEE-754, exact only in the safe range
bigint:  (j - ANCHOR) / 7n + 1n        // truncates toward 0
```

For a true Sabbath, `j - ANCHOR` is a multiple of 7, including negative multiples (`-14 / 7 = -2` in both conventions). So the Sabbath inverse of an actual Sabbath happens to agree under truncating division.

The surrounding calendar algorithms do **not** have that guarantee. Gregorian and Julian quotients are not always exact multiples. `floorDiv(-8, 7)` is −2. Truncating division yields −1. Using truncating division for negative years shifts dates. REQ-MATH-002 therefore applies to every quotient in the core and the adapters, not only to the Sabbath inverse.

Weekday of a negative JDN:

```text
euclidMod(-1, 7) = 6
truncating remainder of -1 divided by 7 is -1
```

The legacy `dow` already corrects truncating remainder with `((j % 7) + 7) % 7`. V2 must keep that correction, implemented once as `euclidMod`, including for negative Sabbaths’ neighbors.

## 5.4 Direct lookup, both directions

To render Sabbath #−1000000 the UI calls `SabbathJDN(-1000000)` once.

```text
offset = 7 * (-1000000 - 1) = 7 * -1000001 = -7000007
JDN = 1733205 - 7000007 = -5266802
```

That value is **MATHEMATICALLY DERIVED** from the formula. It is not a historical claim about the civil calendar label, which the Gregorian or Julian adapter may then compute. No loop from #1 is permitted (REQ-PERF-001).

Inverse, only if `IsSabbath(j)`:

```text
n = (j - 1733205) / 7 + 1
```

If someone passes `JDN(-5266802) + 1`, the inverse throws `not-a-sabbath`. The cycle helper still names the Saturday of that week.

## 5.5 Serialization of negatives

Canonical form of Sabbath #−1 is the three characters `-1` (ASCII hyphen-minus). URL query `n=-1` does not need percent-encoding for the hyphen. Implementations must still accept a correctly percent-decoded string.

Rejected:

| Input | Reason |
| --- | --- |
| `-0` | Not canonical. Zero is `0` |
| `-01` | Leading zero |
| `−1` (U+2212) | Not the canonical hyphen. Legacy `fmt` emits this glyph for display |
| `--1` | Not an integer spelling |

Display may use U+2212. The core never sees it.

## 5.6 UI rules that are not numbering rules

- Gray “pre-anchor” styling may continue for `n < 1`, including 0 and negatives. Styling is not a second number line.
- Milestones stay `n >= 1 && n % 1000 === 0`, matching the legacy condition. Sabbath #−1000 is not a wine milestone. Changing that is an owner decision, not a bug fix.
- The gold cross marks `n = 1` only. Negative regions of the timeline do not get a second cross.
- Search must accept a leading hyphen without treating it as a flag. A bare `-` is invalid.

## 5.7 BCE and negative indexes are different axes

A negative Sabbath number is an index on the seven-day line. A BCE year is a label on a civil calendar. They coincide only after an adapter converts the JDN.

Examples that must not be conflated:

- Sabbath #−13 falls in proleptic Gregorian December of year 32 CE, which is not BCE.
- Sabbath numbers deep enough to land in BCE years are still ordinary outputs of the same formula. Their BCE labels come from REQ-MATH-005 plus the adapter.
- JDN 0 is not Sabbath #0. Sabbath #0 is JDN 1,733,198.

## 5.8 Tests that exist to stop a “helpful” renumber

Specified in [12-VERIFICATION-AND-TEST-PLAN.md](12-VERIFICATION-AND-TEST-PLAN.md), required by REQ-CHR-003:

1. `#1 - #0 = 1` index and 7 days.
2. `#0 - #-1 = 1` index and 7 days.
3. Sequence `#-2, #-1, #0, #1, #2` is strictly increasing by 1, and the JDNs increase by 7.
4. No test fixture uses a map that omits 0.
5. Parsing `#-1`, `n=-1`, and bigint `-1` all yield the same JDN.

## 5.9 Relationship to the sketch

The drawing places “Sabbath #−1” to the left of the cursor and the cross of Sabbath #1 above the line. That composition is the right mental model: negative indexes sit on the same line, toward the past, at a real distance of 14 days from #1 to #−1. The drawing does not define a different formula.
