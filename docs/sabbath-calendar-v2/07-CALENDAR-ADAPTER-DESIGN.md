# 07 — Calendar Adapter Design

## 7.1 Role

Adapters translate between a civil calendar label and the absolute day (`bigint` JDN from the chronology core). They do not compute Sabbath numbers. The UI asks the core for `n` and asks each adapter for a label of the same `j`.

```text
label  --toAbsoluteDay-->  j  --core-->  n
n      --core-->           j  --fromAbsoluteDay-->  label
```

If `toAbsoluteDay` fails, the UI does not guess `j`. If `fromAbsoluteDay` fails, the row says unsupported and the cursor still sits on `j`.

## 7.2 Interface

```text
type Confidence =
  | "mathematically-exact"
  | "algorithmically-calculated"
  | "historically-documented"
  | "historically-reconstructed"
  | "uncertain"
  | "unsupported"
  | "project-defined"

type CalendarDate = {
  calendarId: string
  fields: Record<string, string>   // canonical pieces, all strings
  display: string                  // human line, produced by the adapter
}

type AdapterMeta = {
  calendarId: string
  name: string
  variant: string
  rules: string                    // short description, full text in this doc
  supportedRange: string           // what the algorithm will attempt
  verifiedRange: string            // what tests have checked against outside sources
  method: string
  sources: string[]                // bibliographic ids
  assumptions: string[]
  uncertainty: Confidence
  unsupportedBehavior: string
}

type Adapter = {
  meta: AdapterMeta
  toAbsoluteDay(date: CalendarDate): Result<bigint>
  fromAbsoluteDay(j: bigint): Result<CalendarDate>
}
```

`Result` errors use `unsupported`, `invalid-date`, `ambiguous-date`, or `input-too-large`. `unsupported` must include `meta.calendarId` and a stable `reason` string.

**Registration:** the UI holds an ordered list. Order for MVP is fixed by ADR-012: Hebrew, Gregorian, Chinese, Julian, under the Sabbath row (the Sabbath row is the core, not an adapter). Adding a calendar appends or inserts in that list. It does not change `sabbath.ts`.

Sabbath-row confidence is always `mathematically-exact` for `n` and `j`. That value is assigned by the UI from the core, not by an adapter.

## 7.3 Gregorian adapter

| Field | Content |
| --- | --- |
| calendarId | `gregorian-proleptic` |
| name | Gregorian calendar |
| variant | Proleptic (extended backward and forward with the same leap rule) |
| rules | Leap day 29 February when the astronomical year is divisible by 4, except years divisible by 100 unless divisible by 400. Months of the usual lengths. |
| supportedRange | Any date whose JDN’s canonical form is inside the digit cap, and whose Y-M-D fields are canonical integers |
| verifiedRange | The legacy oracles: 2 April 33, 11 April 2026, 15 October 1582, plus round trips in the test plan including a BCE year and a negative JDN |
| method | `bigint` port of `jdnFromGregorian` and `gregFromJdn` in `js/calendar.js`, with `floorDiv` |
| sources | Existing repository commit `6229702`; Fliegel and Van Flandern, Communications of the ACM 11 (1968), as the usual citation for this algorithm family |
| assumptions | The civil date is the date the legacy function names. No timezone. Proleptic before the historical introduction of the calendar. |
| uncertainty | `algorithmically-calculated` |
| unsupportedBehavior | Invalid Y-M-D (31 April, 29 February in a common year) returns `invalid-date`. It does not roll into the next month. Digit-cap overflow returns `input-too-large`. |

**1582.** This adapter does **not** omit 5–14 October 1582. Those Y-M-D values are ordinary proleptic dates with their own JDNs. The historical papal skip, and later national reforms, are notes the UI may show from the marker/note table in section 7.7. They are not holes in this function. **VERIFIED** legacy behavior: the README says dates before 15 October 1582 are shown in the proleptic Gregorian calendar.

**Display.** Include the era via REQ-MATH-005 (`12 March 33 CE`, `1 January 1 BCE`). Always show the calendar name somewhere on the row so “4 April 33” cannot be read as Julian by accident.

**Anniversary helper** (not part of `toAbsoluteDay`): first Saturday on or after Gregorian 4 April, except year 33, which returns the anchor JDN. Lives in `src/calendars/anniversary.ts` and may call the core and this adapter. It must not live in `src/chronology/`.

## 7.4 Julian adapter

| Field | Content |
| --- | --- |
| calendarId | `julian` |
| name | Julian calendar |
| variant | Astronomical Julian leap rule, proleptic |
| rules | Leap day every astronomical year divisible by 4, including year 0. No century exception. |
| supportedRange | Same digit cap as the core |
| verifiedRange | Julian 4 April 33 CE = JDN 1733205 (**VERIFIED** in the baseline inspection). Further BCE and leap-day cases are required before the adapter is marked done; they are not yet an external published table in this repo |
| method | Standard Julian conversion to JDN using `floorDiv`. The inspection used `day + floor((153*m+2)/5) + 365*y + floor(y/4) - 32083` with the usual March-based month shift. Implementation must match that identity on 4 April 33 and must document the formula in the module header |
| sources | The project anchor statement; the formula above as checked in the baseline |
| assumptions | “Julian” here means the algorithmic calendar, not a claim about which cities had adopted it on that day |
| uncertainty | `algorithmically-calculated` |
| unsupportedBehavior | Same invalid-date policy as Gregorian |

**Anchor display.** `fromAbsoluteDay(1733205)` returns 4 April 33 CE Julian. The Gregorian adapter returns 2 April 33 CE for the same integer. Tests fail if either label is attached to the wrong adapter.

**Difference from Gregorian.** The calendars agree in the late third century around the reform’s usual coincidence and diverge elsewhere. The implementation must not hard-code “Julian = Gregorian − 2 days”. The two-day gap at the anchor is a result, not the algorithm.

## 7.5 Hebrew adapter

| Field | Content |
| --- | --- |
| calendarId | `hebrew-calculated` |
| name | Hebrew calendar |
| variant | Modern calculated (rabbinic arithmetic) civil dates |
| rules | Lunisolar year. Months Nisan through Adar with the calculated lengths. Leap years follow the 19-year cycle and insert Adar I. Postponement rules (`dehiyyot`) determine Rosh Hashanah. Day boundaries in the traditional calendar are sunset; this adapter does **not** apply a location-specific sunset shift |
| supportedRange | The years the chosen reference implementation defines, once that book or paper is pinned |
| verifiedRange | Empty until at least three external published date pairs are copied into tests with citations. **UNRESOLVED** which three, because this design will not invent them |
| method | Dershowitz and Reingold, *Calendrical Calculations*, calculated Hebrew calendar. Pin the edition in the module header during implementation |
| sources | That edition. Do not cite “the Hebrew calendar” with no page |
| assumptions | Arithmetic molad, not observation of the new moon. Midnight/noon JDN alignment is the book’s civil-date convention, which can differ by one evening from a sunset-based community date |
| uncertainty | `algorithmically-calculated` inside the verified range. The UI subtitle says the date is calculated, not observed. Do not use `historically-documented` for the algorithm’s output alone |
| unsupportedBehavior | Outside the pinned verified range, return `unsupported` with reason `outside-verified-range`, even if the arithmetic would emit a month. This is stricter than the Gregorian adapter on purpose |

**Observational ancient calendar.** Not this adapter. A future adapter id `hebrew-observational` would be a different module, default unsupported, and could not ship a date without a cited reconstruction. Until then the product has no such dates.

**Sunset.** Documented mismatch only. Phase 6 may compute a local sunset. It must not silently add one to the Hebrew JDN in this adapter.

**Sabbath number.** The Hebrew row’s Saturday is still core `n`. The calculated Hebrew weekday must be checked against `Weekday(j)` for the verified samples. A disagreement is a failed adapter, not a change to `n`.

## 7.6 Chinese adapter

| Field | Content |
| --- | --- |
| calendarId | `chinese-lunisolar` |
| name | Chinese calendar |
| variant | Lunisolar, to be pinned to one published algorithmic variant |
| rules | Months follow the moon. A leap month is inserted by the chosen solar-term rule. Sexagenary day/year labels may be displayed only if the same source defines them. Timezone and historical capital-versus-local conventions are part of the variant statement |
| supportedRange | Empty at the start of implementation |
| verifiedRange | Empty until a published table is cited and at least three pairs, including one leap month, pass. **UNRESOLVED** which table |
| method | Not selected beyond “a cited algorithm”. Prefer an algorithm with a published test vector over a from-scratch astronomical integration |
| sources | To be filled when the table is chosen. No source, no dates |
| assumptions | Modern algorithmic rules are not the rules of every historical court. Regional and timezone changes are real |
| uncertainty | `unsupported` while the verified range is empty. `algorithmically-calculated` only inside a filled verified range. Never `historically-documented` without a source on that date |
| unsupportedBehavior | `unsupported` / `outside-verified-range`. The row stays visible |

**MVP acceptance.** An empty verified range is a passing design outcome (REQ-CAL-004). Filling it with unchecked dates is a defect.

**Leap months.** Tests, when the range is filled, must include one date inside a leap month and one date the source says is not a leap month, so the leap flag cannot be a constant.

## 7.7 Historical notes (not adapters)

A small static list may annotate a JDN range. Suggested first entries, all **historically documented** only at the level of well-known reform facts, and none of them change conversion:

| Note id | Topic | Effect on arithmetic |
| --- | --- | --- |
| `reform-1582-papal` | Papal Gregorian reform: civil dates 5–14 October 1582 were skipped in Rome. Other countries switched in other years | None. Proleptic adapter still defines those Y-M-D values. The note warns the reader |
| `anchor-project` | Sabbath #1 is a project convention tied to JDN 1733205 | None |

Country-by-country reform dates are **UNRESOLVED** as a dataset. Do not ship a long table from memory. Add a row only with a citation.

## 7.8 Quality metadata in the UI

Every row header exposes `name`, `variant`, and `uncertainty`. A details control shows the rest of `AdapterMeta` in plain language. The Sabbath row’s details show the formula and the anchor, with label PROJECT-DEFINED for the anchor and MATHEMATICALLY DERIVED for the computed JDN.

## 7.9 Adding a calendar later

1. New file under `src/calendars/<id>.ts` implementing `Adapter`.
2. Metadata filled. `verifiedRange` empty if evidence is not ready.
3. Tests: invalid dates, one round trip per cited vector, one unsupported sample, one test that a wrong label does not change `n`.
4. Insert the id into the row-order list.
5. No edit to `src/chronology/*` unless an ADR says the absolute-day definition changed. That ADR would be an owner-level event, not an adapter task.

## 7.10 Mapping to confidence words

| Situation | Confidence |
| --- | --- |
| Sabbath `n` and JDN from the core | mathematically-exact |
| The anchor marker (the choice of Sabbath #1) | project-defined |
| Gregorian or Julian label from the adapters | algorithmically-calculated |
| Hebrew or Chinese label inside a cited verified range | algorithmically-calculated |
| A marker that quotes a dated chronicle with a citation | historically-documented |
| A reconstruction that an author proposes | historically-reconstructed |
| Calculated label known to ignore sunset or local reform | uncertain (add this beside algorithmically-calculated; do not replace the algorithm class) |
| No verified range, or out of range | unsupported |
