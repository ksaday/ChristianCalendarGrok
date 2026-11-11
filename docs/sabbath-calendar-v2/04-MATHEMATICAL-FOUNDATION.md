# 04 — Mathematical Foundation

## 4.1 Status of each statement

| Statement | Label |
| --- | --- |
| Sabbath #1 is JDN 1,733,205, Julian 4 April AD 33 | PROJECT-DEFINED |
| `SabbathJDN(n) = 1733205 + 7*(n-1)` | PROJECT-DEFINED formula; results MATHEMATICALLY DERIVED |
| Anchor weekday index is Saturday under the legacy `dow` | VERIFIED (`1733205 % 7 = 5`) |
| 11 April 2026 is JDN 2,461,142 and Sabbath #103,992 | VERIFIED (legacy self-check and independent integer re-check) |
| Floor division differs from truncating division on negatives | VERIFIED (`-8 // 7 = -2`) |
| This formula proves continuous observance to Creation | Not claimed. False if asserted. |

## 4.2 Domain

Let `Z` be the integers. The practical input domain `Z_cap` is every integer whose canonical decimal form has at most 10,000 digits (REQ-PERF-002). The mathematical identities below are stated for all of `Z`. The software implements them on `Z_cap` and rejects the complement.

Absolute days are elements of `Z`, interpreted as civil-date Julian Day Numbers (section 4.6). Sabbath numbers are elements of `Z`.

## 4.3 Definitions

```text
ANCHOR_JDN = 1733205

SabbathJDN(n) = ANCHOR_JDN + 7 * (n - 1)

IsSabbath(j) ⟺ there exists an integer n such that j = SabbathJDN(n)
             ⟺ (j - ANCHOR_JDN) is divisible by 7

SabbathNumber(j) = (j - ANCHOR_JDN) / 7 + 1
                   defined only when IsSabbath(j)
```

Division in `SabbathNumber` is exact integer division, not rounding.

**Weekday index** (legacy numbering, VERIFIED at the anchor):

```text
Weekday(j) = euclidMod(j, 7)
0 Monday, 1 Tuesday, 2 Wednesday, 3 Thursday, 4 Friday, 5 Saturday, 6 Sunday
IsSabbath(j) ⟺ Weekday(j) = 5
```

This equivalence holds because `Weekday(ANCHOR_JDN) = 5` and the step is 7. It is a theorem of the project constants, not an extra convention.

**Euclidean modulo and floor division** for `m > 0`:

```text
floorDiv(a, m) = the unique q such that q * m <= a < (q + 1) * m
euclidMod(a, m) = a - m * floorDiv(a, m)     // result in 0 .. m-1
```

`BigInt` `/` and `%` truncate toward zero and must not be used raw. When `/` and `%` are that truncating pair (`a = (a / m) * m + (a % m)`), floor division is:

```text
floorDiv(a, m) = (a % m < 0) ? (a / m - 1) : (a / m)
euclidMod(a, m) = (a % m < 0) ? (a % m + m) : (a % m)
```

This is exact for multiples and non-multiples. Tests must lock `floorDiv(-8, 7) = -2`, `floorDiv(-14, 7) = -2`, and `euclidMod(-1, 7) = 6`.

## 4.4 Invariants

For every integer `n` in the accepted domain:

```text
SabbathJDN(1) = 1733205
SabbathJDN(n + 1) - SabbathJDN(n) = 7
SabbathJDN(n) - SabbathJDN(n - 1) = 7
SabbathNumber(SabbathJDN(n)) = n
Weekday(SabbathJDN(n)) = 5
```

These are the release invariants (REQ-QA-001, REQ-CHR-001).

## 4.5 Worked values (MATHEMATICALLY DERIVED)

| n | Computation | JDN | Weekday |
| ---: | --- | ---: | --- |
| 3 | 1733205 + 14 | 1733219 | Saturday |
| 2 | 1733205 + 7 | 1733212 | Saturday |
| 1 | 1733205 + 0 | 1733205 | Saturday |
| 0 | 1733205 − 7 | 1733198 | Saturday |
| −1 | 1733205 − 14 | 1733191 | Saturday |
| −2 | 1733205 − 21 | 1733184 | Saturday |
| 103992 | 1733205 + 7*103991 | 2461142 | Saturday |

`7 * 103991 = 727937` and `1733205 + 727937 = 2461142`. **VERIFIED**.

Proleptic Gregorian labels of the small indexes are in [00-EXECUTIVE-SUMMARY.md](00-EXECUTIVE-SUMMARY.md). They come from the Gregorian adapter, not from the core. The core does not know month names.

## 4.6 What “JDN” means here

Three ideas that must not be collapsed:

1. **Julian Day (JD).** A continuous astronomical time scale, traditionally measured from noon Universal Time. A fractional JD names an instant.
2. **Julian Day Number (JDN).** The integer of the civil date associated with the noon of that civil date in the usual calendar algorithms. This is what `jdnFromGregorian` returns.
3. **Civil midnight interval.** The local calendar date as people live it, which depends on longitude, timezone, and (for some traditions) sunset.

**Decision (ADR-003):** the chronology core uses (2) only. Sabbath #1 is the civil date that the legacy software calls JDN 1,733,205. The core does not store hours. Adding 0.5 to “move noon to midnight” is forbidden, because the legacy integer already means that civil date.

**Negative JDN.** The same formulas apply. `Weekday` uses Euclidean modulo so a negative JDN still has an index in `0..6`. There is no special case that stops the sequence at JDN 0. JDN 0 is a normal integer in the absolute-day model (the conventional astronomical epoch label, noon-based). It is not Sabbath #1 and it is not a Creation date.

## 4.7 Year numbering

| Astronomical year used in algorithms | Historical label |
| ---: | --- |
| 2 | 2 CE |
| 1 | 1 CE |
| 0 | 1 BCE |
| −1 | 2 BCE |
| −2 | 3 BCE |

There is no year 0 in the historical label. Display conversion:

```text
if astronomical >= 1:  "{year} CE"
if astronomical <= 0:  "{1 - astronomical} BCE"
```

Inverse:

```text
CE y (y >= 1) → y
BCE y (y >= 1) → 1 - y
```

**Off-by-one-year guard:** tests must show that 1 BCE is astronomical 0, and that a Julian leap day exists in astronomical year 0 (divisible by 4). Treating 1 BCE as year −1 would move that leap day and many Sabbath labels by one year.

**Off-by-one-day guard:**

- Do not convert a Julian date with the Gregorian function or the reverse.
- Do not increment a JDN to “skip” the 1582 reform. The proleptic Gregorian adapter has no hole. The historical hole is a note on the Gregorian row when the civil date falls in a locally unreformed era. See ADR-007.
- Do not define Sabbath number on non-Saturdays by rounding.

## 4.8 Cycle map

Legacy columns are Sunday-first. For absolute day `j`:

```text
saturdayDelta(j) = euclidMod(5 - Weekday(j), 7)   // 0 if already Saturday
cycleSabbathJdn(j) = j + saturdayDelta(j)
cycleSabbathNumber(j) = SabbathNumber(cycleSabbathJdn(j))
dayOffset(j) = euclidMod(Weekday(j) + 1, 7)       // Sunday 0 … Saturday 6
```

`dayOffset` matches `sunCol` in the legacy file. The Sabbath of the cycle is the Saturday at the end of that Sunday–Saturday week, which is the same direction the legacy month grid uses.

Example (MATHEMATICALLY DERIVED): JDN 1733205 has `Weekday` 5, delta 0, offset 6, number 1. The previous day has `Weekday` 4 (Friday), delta 1, so its cycle Sabbath is still #1, and its offset is 5.

## 4.9 Serialization

| Concern | Rule |
| --- | --- |
| In memory | `bigint` |
| Across functions | `bigint`, never `number` |
| JSON, URL, logs, test fixtures | canonical decimal string (REQ-MATH-003) |
| Display | grouping separators and a minus glyph are allowed in the view only |
| Temporary UI scale | pixel floats may exist in the view-model’s layout box and must sit in a different field from `jdn` and `n` |

Parser rejects scientific notation, hex, leading zeros, `+`, spaces, and the Unicode minus (`U+2212`). The legacy `fmt` function emits `U+2212`. The V2 parser must not accept that glyph. The view converts it to ASCII `-` before validation. This avoids a class of “looks negative, fails to parse” bugs and a class of “Unicode minus smuggled into the core” bugs.

## 4.10 Complexity and limits

- `SabbathJDN` and `SabbathNumber` are a constant number of arithmetic operations. Digit length grows with `|n|`, so wall-clock time grows with digit length, not with the distance from the anchor in weeks.
- Memory per value is proportional to digit length. A 10,000-digit number is small. The cap exists so a hostile query string cannot request a multi-megabyte integer.
- The model is infinite. The process is not. Doc 14 states that honestly.
- JavaScript `Date` is not used for any conversion. Its reliable range is irrelevant to the core and too small for the product even if it were exact, which it is not (it is millisecond-based and timezone-aware).

## 4.11 What is exact, and what is not

Exact, inside `Z_cap`, once the tests pass:

- The Sabbath index.
- The absolute day of that index.
- The weekday index.
- The proleptic Gregorian and Julian labels, as outputs of their stated algorithms.

Not exact, and not to be spoken of as exact:

- That the seven-day rhythm was observed on every one of those days in history.
- Ancient Hebrew dates that depend on observation.
- Chinese dates outside a cited table.
- Any Creation year.
- The civil date on which a local community began the Sabbath at sunset.
