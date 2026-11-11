# 06 — Chronology Core Architecture

## 6.1 Role

The chronology core is the only module that may turn a Sabbath number into an absolute day or the reverse. It is pure: same inputs, same outputs, no clock, no locale, no network, no adapter calls.

Dependency direction:

```text
UI  -->  chronology core
UI  -->  calendar adapters
calendar adapters  -->  absolute-day type only (a bigint branded as AbsoluteDay)
calendar adapters  -X->  SabbathNumber functions
chronology core    -X->  adapters, DOM, Date, AI
```

The absolute-day brand stops a raw `bigint` year or a Sabbath number from being passed where a JDN is required. The types may be erased at runtime; the constructors are the boundary.

## 6.2 Module sketch (not implementation)

```text
src/chronology/constants.ts      ANCHOR_JDN, WEEKDAY_SATURDAY = 5, INPUT_DIGIT_CAP
src/chronology/int.ts            parseCanonical, formatCanonical, floorDiv, euclidMod
src/chronology/sabbath.ts        the functions in section 6.3
src/chronology/invariants.ts     verifyChronologyInvariants
src/chronology/errors.ts         error codes
```

No function in this tree returns a month name.

## 6.3 Contracts

All integer inputs are `bigint` or canonical strings that `validate*` parses first. Outputs that are integers are `bigint`. Error results are values, not thrown strings, so the UI and tests can branch without locale text. A short English `message` may accompany the code for logs. The UI chooses display copy.

### `validateSabbathNumber(input: string): Result<bigint>`

- **Success:** canonical integer, digit length `<= 10000`.
- **Errors:** `invalid-integer`, `input-too-large`.
- **Precision:** exact. No rounding.

### `sabbathNumberToAbsoluteDay(n: bigint): Result<bigint>`

- **Precondition:** `n` already validated, or this function validates digit length again.
- **Success:** `1733205n + 7n * (n - 1n)`.
- **Errors:** `input-too-large` if `n` or the resulting JDN exceeds the digit cap. (The JDN has similar magnitude to `7*n`, so the cap is checked on both.)
- **Precision:** exact `bigint`.
- **Complexity:** O(digits), no loop over `n`.

### `absoluteDayToSabbathNumber(j: bigint): Result<bigint>`

- **Success:** `(j - 1733205n) / 7n + 1n` only when `(j - 1733205n)` is divisible by 7. Use exact division, not floor, so a non-multiple cannot become a nearby Sabbath.
- **Errors:** `not-a-sabbath`, `invalid-integer`, `input-too-large`.
- **Precision:** exact. Non-Sabbaths do not return a rounded `n`.

### `absoluteDayToSabbathCycle(j: bigint): Result<Cycle>`

```text
Cycle = { sabbathNumber: bigint, sabbathJdn: bigint, dayOffset: 0|1|2|3|4|5|6 }
```

- **Behavior:** compute `cycleSabbathJdn` as in [04-MATHEMATICAL-FOUNDATION.md](04-MATHEMATICAL-FOUNDATION.md) section 4.8, then the number. Always succeeds for a validated `j` inside the cap.
- **Errors:** `invalid-integer`, `input-too-large`.
- **Precision:** `dayOffset` is a small integer 0..6. It may be a TypeScript numeric literal union because it is not an absolute day.

### `getPreviousSabbath(n: bigint): Result<bigint>`

- **Success:** `n - 1n`, including crossing 0 (`1 → 0`, `0 → -1`).
- **Errors:** digit-cap errors if the input is illegal. `n - 1` does not grow the digit length except at powers of ten, which remain inside the cap when `n` is.

### `getNextSabbath(n: bigint): Result<bigint>`

- **Success:** `n + 1n`.

### `getSabbathInterval(n1: bigint, n2: bigint): Result<bigint>`

- **Success:** `n2 - n1` (signed Sabbath count).
- **Note:** day distance is `7n * (n2 - n1)`, exposed as a separate field `dayDistance` on the same result so callers do not reintroduce `number`.

### `getCycleDayOffset(j: bigint): Result<0|1|2|3|4|5|6>`

- **Success:** the `dayOffset` from the cycle function.

### `getChronologicalDistance(j1: bigint, j2: bigint): Result<bigint>`

- **Success:** `j2 - j1` in absolute days, signed, any integers inside the cap, Sabbath or not.

### `verifyChronologyInvariants(samples: readonly bigint[]): InvariantReport`

```text
InvariantReport = {
  anchorOk: boolean
  failures: Array<{ n: string, code: string }>
}
```

Checks, for the built-in set `{1, 0, -1, 103992}` union `samples`:

- `sabbathNumberToAbsoluteDay(1) = 1733205`
- step 7 between `n` and `n+1` for each sample
- inverse returns `n`
- weekday of each Sabbath JDN is 5

This function does not catch exceptions and turn them into `anchorOk: true`. Any arithmetic error is a failure.

## 6.4 Error catalog

| Code | When |
| --- | --- |
| `invalid-integer` | Not canonical decimal |
| `input-too-large` | More than 10,000 digits |
| `not-a-sabbath` | Inverse called on a non-multiple |
| `invariant-failed` | Used inside the report, not thrown to users as a generic fault |

No error code says “clamped to year 33”. The core has no year clamp.

## 6.5 Precision guarantee

Inside the digit cap, the functions in 6.3 are exact on `Z` or they return an error. They do not return an approximation. Callers that need a pixel may convert a **delta from the viewport origin**, after subtracting two `bigint`s that are close, into a `number` at the UI boundary, and only when the absolute value of that delta is below `2^53`. If the delta is larger, the UI changes zoom or recenters instead of coercing the JDN.

```text
allowed:     number(jdn - viewportOrigin) when the difference is a safe integer
forbidden:   number(jdn) when jdn may exceed 2^53
forbidden:   parsing a pixel x back into a JDN by inverting a float scale
```

The viewport origin itself remains a `bigint` equal to a real absolute day.

## 6.6 Worked call sequence for Sabbath #−1

```text
validateSabbathNumber("-1") -> -1n
sabbathNumberToAbsoluteDay(-1n) -> 1733191n
absoluteDayToSabbathNumber(1733191n) -> -1n
absoluteDayToSabbathCycle(1733190n) -> { sabbathNumber: -1n, sabbathJdn: 1733191n, dayOffset: 5 }
getPreviousSabbath(-1n) -> -2n
getNextSabbath(-1n) -> 0n
getSabbathInterval(-1n, 1n) -> 2n Sabbaths, dayDistance 14n
```

`1733190` is the Friday before Sabbath #−1. **MATHEMATICALLY DERIVED** from weekday 5 on `1733191`.

## 6.7 What the core refuses to decide

- Month, weekday name, and era label.
- Which civil calendar a reader “should” use.
- Whether a community starts the Sabbath at Friday sunset.
- Whether a negative index is “before Creation”.
- Which historical event happened on that day.

Those belong to adapters, the UI, or documents. If they leak into this module, REQ-CHR-007 fails.
