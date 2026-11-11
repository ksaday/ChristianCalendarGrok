# 02 — Repository Baseline Analysis

Inspection date: 11 November 2026. Method: GitHub API reads of `ksaday/ChristianCalendar` and a local integer re-check of the published formulas. No file in that repository was modified.

Evidence in this document is **VERIFIED** unless marked otherwise.

## 2.1 Repositories and git state

### Reference application (read-only)

- Repository: https://github.com/ksaday/ChristianCalendar
- Default branch: `main`
- Single commit: `6229702`, message “Add files via upload”, author date 2026-07-22. `pushed_at` on the API snapshot was 2026-11-11.
- Language reported by GitHub: CSS. Size about 114 KB.
- Working tree was not checked out in this workspace. Remote listing shows no uncommitted files because the inspection used the published tree, not a dirty local clone.
- No `AGENTS.md`, `CLAUDE.md`, `.cursor/rules/`, test directory, or package manifest.

Tree:

```text
README.md
index.html
css/style.css
js/calendar.js
```

### This workspace (V2 design home)

- Repository: `ChristianCalendarGrok` (origin `github.com/ksaday/ChristianCalendarGrok`)
- Branch: `main`, up to date with `origin/main` at inspection
- Commit: `61282e2` “Initial commit”
- Git status at inspection: clean
- Only tracked file before this design set: `README.md`, contents `# ChristianCalendarGrok`
- **PROPOSED** home for V2. Design files for this phase live in `docs/sabbath-calendar-v2/` only.

These are different repositories. Editing one does not edit the other.

## 2.2 What the README claims

Source: `README.md` in ChristianCalendar.

- Sabbath #1 is Saturday, 4 April AD 33, Julian Day Number 1,733,205, called the Passover Sabbath of the Resurrection week. **PROJECT-DEFINED** in the product; the README cites Humphreys & Waddington, *Nature* 306 (1983), and the NASA Five Millennium Canon as dating background. This design does not re-litigate that citation.
- Formula stated in prose: `SabbathNumber = (JDN − 1,733,205) / 7 + 1`.
- A day is a Sabbath iff `JDN mod 7 == 5` (Saturday).
- Self-check: 11 April 2026 → JDN 2,461,142 → Sabbath #103,992. Anchor day → Sabbath #1.
- Dates before 15 October 1582 are **displayed in the proleptic Gregorian calendar**. The anchor therefore appears as **2 April AD 33** in the year-33 view. Same JDN, different label.
- Anniversary Sabbath: first Saturday on or after 4 April each year, not every 52nd Sabbath number.
- Milestone: every 1,000th Sabbath. Gold seal on Saturdays. Green circle on today.
- No build step and no server. Open `index.html` or host the folder statically.
- Video cards are placeholders.

## 2.3 Page structure (`index.html`)

- Header states Sabbath #1 and JDN 1,733,205.
- Controls: previous month, today, next month, month `<select>`, year `<input type="number" min="33" max="3000">`, Go.
- Calendar table is Sunday-first. The seventh column header is the text “Sabbath”, not “Sat”.
- Legend: gold Sabbath number, anniversary star, wine milestone.
- Aside: three video placeholders.
- Footer: Hebrews 4:9, method note, `#selfCheck`.
- Script: `js/calendar.js` only. No modules, no network calls.

## 2.4 Styles (`css/style.css`)

Reusable visual language, not reusable chronology:

- Palette: `--midnight` `#1B2240`, `--parchment` `#F6F2E8`, `--gold` `#B0842C`, `--wine` `#6C2433`, `--olive` `#4A6741`.
- `.sab-seal.pre` is a gray seal (`#B6AD99`) for Sabbath numbers less than 1.
- `.sab-seal.mile` uses wine.
- Breakpoint at 820px stacks the video column under the calendar.
- The V2 timeline sketch is a different visual system (black field, cyan rows, gold cross). Do not force the parchment month page to be the V2 shell. Tokens for gold, wine, and midnight may be reused for seals, milestones, and the cross.

## 2.5 Algorithm (`js/calendar.js`)

The file is one IIFE. There is no export surface. Functions below are the real names.

### Constants and core

```text
ANCHOR_JDN = 1733205
jdnFromGregorian(year, month, day)
gregFromJdn(j)
dow(j) = ((j % 7) + 7) % 7     // 0=Monday … 5=Saturday, 6=Sunday
isSabbath(j) = dow(j) === 5
sabbathNumber(j) = (j - ANCHOR_JDN) / 7 + 1
```

`jdnFromGregorian` and `gregFromJdn` are the integer proleptic Gregorian algorithms (Fliegel–Van Flandern family):

```text
a = floor((14 - month) / 12)
y = year + 4800 - a
m = month + 12 * a - 3
JDN = day + floor((153 * m + 2) / 5) + 365 * y
    + floor(y / 4) - floor(y / 100) + floor(y / 400) - 32045
```

The inverse recovers `day`, `month`, `year` with the matching `floor` steps. `Math.floor` rounds toward −∞, which is the behavior V2 must keep when these quotients move to `BigInt`.

`dow` already uses a positive modulo so negative JDN values would land in `0..6` if they were ever passed. The UI never passes them today.

`sabbathNumber` uses IEEE-754 division, not an integer-divide-and-check. On a true Sabbath inside the safe-integer range the result is an integer. On any other day it is a fraction. Call sites in the UI invoke it only for Sabbaths, plus the self-check.

### Independent integer check (this inspection)

Reimplemented with Python floor division, not by executing the page:

| Check | Result |
| --- | --- |
| `dow(1733205)` | 5 (Saturday) |
| `gregFromJdn(1733205)` | year 33, month 4, day 2 |
| Julian-style JDN of astronomical year 33, month 4, day 4 | 1733205 |
| `jdnFromGregorian(2026, 4, 11)` | 2461142 |
| Sabbath number of that JDN | 103992 |
| `(2461142 - 1733205) / 7 + 1` | 727937 / 7 + 1 = 103992 |
| Max Sabbath number still inside `2^53 - 1` | about 1.29e15 (not unbounded) |

Proleptic Gregorian dates for nearby indexes:

| n | JDN | Date |
| ---: | ---: | --- |
| −12 | 1733114 | 1 January 33 (Saturday) |
| −13 | 1733107 | 25 December 32 |
| 0 | 1733198 | 26 March 33 |
| 1 | 1733205 | 2 April 33 |

`15 October 1582` under the proleptic function is JDN 2,299,161, the familiar civil JDN of that Gregorian date. `4 October 1582` under the **same proleptic function** is JDN 2,299,150. That is not the historical Julian 4 October 1582 (the day before the reform skip). The app does not model the skipped civil dates 5–14 October 1582. **VERIFIED** limitation.

### Other functions

| Function | Behavior | V2 note |
| --- | --- | --- |
| `daysInMonth(y, m)` | Gregorian leap rule: divisible by 4, not by 100, unless by 400 | Gregorian adapter only. Not Julian. |
| `fmt(n)` | Thousands separators; Unicode minus for negatives | Reuse the idea. Do not use it to compute. |
| `ord(n)` | English ordinals | Anniversary copy only. |
| `annivJdnOfYear(y)` | Year 33 returns `ANCHOR_JDN`. Otherwise JDN of Gregorian 4 April, then increments until Saturday | **PROPOSED:** keep as a presentation rule, outside the core. It is calendar-anchored, so it is not “Sabbath number mod 52”. |
| `annivInfo` | Anniversary only when `j` is that Saturday and the Gregorian date falls in April 2–10, or when `j` is the anchor | The April 2–10 window matches the documented Gregorian display of the anchor. |
| `clampY(y)` | `min(3000, max(33, y))` | This is the hard navigation limit. |
| `sunCol(j)` | `(dow(j) + 1) % 7` so Sunday is column 0 and Saturday is column 6 | Cycle definition for V2 should match this Sunday-first week. |
| `selfCheck` | Asserts the 2026 oracle and `sabbathNumber(ANCHOR_JDN) === 1` | Regression reference. Footer text on failure: “SELF-CHECK FAILED — do not trust displayed numbers.” |

### UI behavior that is easy to misread

- “Today” comes from `new Date()` local year/month/day, then `jdnFromGregorian`. The chronology of a selected civil date does not depend on timezone. Which civil date is “today” does.
- Countdown copy says the next Sabbath “begins at sunset on Friday” and names the previous Gregorian date. There is no solar calculation.
- Popover shows Sabbath number, JDN, days since anchor, and weeks since anchor (`ds / 7`). For a Sabbath, weeks since anchor equals `n - 1`.
- Milestone styling requires `sn >= 1 && sn % 1000 === 0`. Negative multiples of 1000 are not milestones.
- `sn < 1` uses the gray `.pre` seal. So #0 and #−1 through #−12 render if the user opens January–March 33.
- Previous-month from January 33 calls `clampY(32)`, which returns 33. The year field cannot be typed below 33. Sabbath #−13 and below are unreachable. **VERIFIED**.
- Future navigation stops at year 3000.

## 2.6 Date-range limitations (current product)

| Limit | Cause |
| --- | --- |
| Display years 33 through 3000 | `clampY` and the year input |
| Negative Sabbaths only #−12 through #0 | Those Saturdays fall in proleptic Gregorian year 33. Earlier years are clamped |
| No BCE, no year 0, no negative JDN in the UI | Year control and Gregorian-only renderer |
| No historical Julian labels except the prose statement of the anchor | No Julian function exists |
| No Hebrew, no Chinese | Not implemented |
| Safe integer ceiling near JDN `2^53 - 1` | `Number`, not `BigInt` |
| Proleptic Gregorian through 1582 | No reform-gap calendar |
| “Sunset” is a sentence | No coordinates, no time-of-day model |

## 2.7 Reuse and refactor boundaries

**Reuse as specification, not as a library import.**

The legacy file has no module boundary. V2 should re-specify these behaviors in TypeScript and keep the legacy file frozen:

- Anchor constant and Sabbath formula.
- Proleptic Gregorian pair (`jdnFromGregorian`, `gregFromJdn`) after floor-division is made explicit.
- Saturday test `dow === 5` with the same weekday numbering, including the positive-modulo pattern.
- Self-check oracles: JDN 2461142, Sabbath 103992, anchor Sabbath #1, `annivJdnOfYear(33) === ANCHOR_JDN`.
- Anniversary as a UI rule, including the year-33 special case.
- Gold / wine / gray seal meanings.
- Static hosting, no login.

**Do not carry forward:**

- Year clamp 33..3000 as a chronology limit.
- IEEE-754 as the authoritative type.
- Calling `sabbathNumber` on non-Sabbaths and displaying a fraction.
- Any future edit that “fixes” the anchor inside `calendar.js`.

**Refactor shape (PROPOSED, not done):** new modules in this repository. Zero edits under the ChristianCalendar tree. Suggested later layout, not created now:

```text
src/chronology/        pure BigInt core
src/calendars/         gregorian, julian, hebrew, chinese
src/ui/                timeline, proof card, search
test/                  node:test property and regression tests
```

## 2.8 Baseline conclusion

The original application correctly implements the project formula inside a proleptic Gregorian month UI for years 33–3000, and its self-check matches an independent integer evaluation. It is the wrong place to grow unbounded negative indexes, Julian/Hebrew/Chinese adapters, or exact `BigInt` arithmetic. Preserve it. Build V2 beside it.
