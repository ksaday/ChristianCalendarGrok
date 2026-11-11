# 10 — Data Models and Interfaces

Types below are the contract an implementation must satisfy. They are design notation, not code dropped into `src/` during this phase.

## 10.1 Branded integers

```text
type SabbathNumber = bigint & { readonly brand: "SabbathNumber" }
type AbsoluteDay  = bigint & { readonly brand: "AbsoluteDay" }
```

Constructors:

```text
function sabbathNumber(n: bigint): SabbathNumber
function absoluteDay(j: bigint): AbsoluteDay
```

They run the digit cap. They do not check `IsSabbath` (`absoluteDay` allows every civil day). `absoluteDayToSabbathNumber` is the Sabbath check.

Display formatters accept the brand and return strings. Parsers return the brand or an error. No function accepts `number`.

## 10.2 Canonical integer

```text
type CanonicalInteger = string   // REQ-MATH-003

function parseCanonical(input: string): Result<bigint>
function formatCanonical(n: bigint): CanonicalInteger
function formatGrouped(n: bigint): string   // UI only, may use U+2212 and grouping
```

`formatGrouped` output is not valid input to `parseCanonical` until hyphens and separators are normalized by `normalizeUserInteger(input: string): string`.

Normalization, UI only:

- Trim spaces.
- Map U+2212 to `-`.
- Remove `,` and thin spaces used as thousands separators.
- Leave the result for `parseCanonical` to accept or reject.

## 10.3 Results

```text
type Ok<T> = { ok: true, value: T }
type Err = { ok: false, error: { code: ErrorCode, reason?: string } }
type Result<T> = Ok<T> | Err

type ErrorCode =
  | "invalid-integer"
  | "input-too-large"
  | "not-a-sabbath"
  | "invalid-date"
  | "ambiguous-date"
  | "unsupported"
  | "invariant-failed"
```

## 10.4 Core records

```text
type Cycle = {
  sabbathNumber: SabbathNumber
  sabbathJdn: AbsoluteDay
  dayOffset: 0 | 1 | 2 | 3 | 4 | 5 | 6
}

type Interval = {
  sabbathCount: bigint      // signed
  dayDistance: bigint       // signed, exactly 7 * sabbathCount
}

type InvariantFailure = { n: CanonicalInteger, code: "invariant-failed" | "not-a-sabbath" }
type InvariantReport = { anchorOk: boolean, failures: InvariantFailure[] }
```

Function signatures match [06-CHRONOLOGY-CORE-ARCHITECTURE.md](06-CHRONOLOGY-CORE-ARCHITECTURE.md). Names to implement:

- `sabbathNumberToAbsoluteDay`
- `absoluteDayToSabbathNumber`
- `absoluteDayToSabbathCycle`
- `getPreviousSabbath`
- `getNextSabbath`
- `getSabbathInterval`
- `getCycleDayOffset`
- `getChronologicalDistance`
- `validateSabbathNumber`
- `verifyChronologyInvariants`

## 10.5 Calendar records

```text
type Confidence =
  | "mathematically-exact"
  | "algorithmically-calculated"
  | "historically-documented"
  | "historically-reconstructed"
  | "uncertain"
  | "unsupported"

type CalendarDate = {
  calendarId: "gregorian-proleptic" | "julian" | "hebrew-calculated" | "chinese-lunisolar"
  fields: Record<string, string>
  display: string
}

type AdapterMeta = {
  calendarId: CalendarDate["calendarId"]
  name: string
  variant: string
  rules: string
  supportedRange: string
  verifiedRange: string
  method: string
  sources: string[]
  assumptions: string[]
  uncertainty: Confidence
  unsupportedBehavior: string
}
```

Field keys, so search forms stay aligned:

| calendarId | Required fields |
| --- | --- |
| `gregorian-proleptic` | `year` (astronomical canonical integer), `month` (`1`..`12`), `day` |
| `julian` | `year`, `month`, `day` (same ranges) |
| `hebrew-calculated` | `year`, `month` (adapter-defined canonical month id, including a leap Adar flag in the id), `day` |
| `chinese-lunisolar` | `year`, `month`, `leapMonth` (`true` or `false`), `day` |

Era labels are not input fields. The form collects BCE/CE and converts to the astronomical year with REQ-MATH-005 before `toAbsoluteDay`.

## 10.6 Cursor and URL

```text
type ScaleId = "day" | "week" | "year" | "century" | "millennium" | "age"

type Cursor = {
  j: AbsoluteDay
  n: SabbathNumber | null     // null when j is not a Sabbath
  cycle: Cycle
  scale: ScaleId
}

type ShareQuery = {
  n?: CanonicalInteger
  scale?: ScaleId
}
```

When the user searches for a Sabbath number, `Cursor.n` is set and `j` is that Sabbath. When the user searches for a civil date that is not a Saturday, `Cursor.n` is null, `j` is that date, and `cycle.sabbathNumber` is the Saturday of the week. The address bar updates to `n` only when `Cursor.n` is non-null. For a midweek selection the address uses `j` as an additional key `d=<canonical jdn>`. **PROPOSED:** `d` is allowed only as a canonical JDN, and if both `n` and `d` are present and disagree, the page shows `ambiguous-date` and does not pick one silently.

This is the one addition to the `?n=` decision in ADR-013, required so a non-Sabbath inspection is shareable without pretending it is a Sabbath number.

## 10.7 Proof card model

```text
type ProofCard = {
  sabbathNumber: SabbathNumber | null
  anchorJdn: "1733205"
  jdn: CanonicalInteger
  sevenDayOffset: CanonicalInteger     // "0" at n=1, otherwise 7*(n-1) when n is known
  formula: "SabbathJDN(n) = 1733205 + 7*(n-1)"
  labels: Array<{ calendarId: string, display: string, confidence: Confidence }>
  invariant: InvariantReport
}
```

`anchorJdn` is the literal constant. `sevenDayOffset` is omitted or null when `sabbathNumber` is null, and the card explains that the day is not a Sabbath and shows `cycle` instead.

## 10.8 Markers

```text
type Marker = {
  id: string
  j: AbsoluteDay
  title: string
  source: string          // required, non-empty
  uncertainty: Exclude<Confidence, "mathematically-exact" | "unsupported">
}
```

The built-in anchor marker uses `uncertainty: "historically-documented"` only if the owner attaches the Humphreys & Waddington citation as background for the **date choice**. The Sabbath index itself stays project-defined in the proof card. To avoid blurring those, the anchor marker’s uncertainty field is a dedicated value in prose (“project-defined anchor”) even though the union above is for optional historical markers. Implementation may use `historically-documented` for external events only, and a separate `project-defined` string for the anchor. **PROPOSED:** extend `Confidence` with `project-defined` for that marker alone.

```text
type Confidence =
  ... existing ...
  | "project-defined"
```

Only the anchor marker may use `project-defined`.

## 10.9 View layout box (non-authoritative)

```text
type LayoutBox = {
  originJdn: AbsoluteDay
  pxPerDay: number          // display scale only
  widthPx: number
}
```

Invariant checked in UI tests: changing `pxPerDay` does not change `Cursor.j` or `ProofCard.jdn`.

## 10.10 Serialization examples

| Value | Canonical | Grouped display (example) |
| --- | --- | --- |
| Sabbath #0 | `0` | `0` |
| Sabbath #−1 | `-1` | `−1` |
| JDN of #1 | `1733205` | `1,733,205` |
| Sabbath #103992 | `103992` | `103,992` |

JSON fixtures store canonical strings, not numbers, so a large integer cannot be silently rounded by a JSON parser that uses IEEE-754.
