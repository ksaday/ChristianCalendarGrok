# 01 — Project Vision

## Purpose

Sabbath Calendar V2 extends the existing Christian Calendar from a Gregorian month page with Saturday seals into a single absolute chronology:

```text
one Sabbath #1 anchor
one unbroken seven-day cycle
one integer index for every Sabbath, negative, zero, and positive
```

Gregorian, Julian, Hebrew, and Chinese dates are labels on that axis. They are not the axis.

## What the product is

A static website a person can open with no account, no AI, and no server. They can jump to any integer Sabbath number, including negative numbers, and see that day aligned across the calendars that actually support it. A proof card shows the formula and the pass/fail result from the chronology engine.

## What the product is not

- Not a conventional Gregorian calendar that merely highlights Saturday.
- Not a claim that mathematical extrapolation proves continuous Sabbath observance back to Creation.
- Not an observational reconstruction of ancient Hebrew or Chinese calendars.
- Not an AI product. Models may help the owner build and review it. They do not calculate it.

## Audiences

1. **Owner / maintainer.** Needs the anchor preserved, regressions caught, and historical claims labeled.
2. **Reader.** Needs a readable timeline, negative-number navigation, and a clear distinction between exact arithmetic and uncertain history.
3. **Future contributor or agent.** Needs contracts, requirement IDs, and rules that forbid changing the anchor or inventing dates.

## Design principles

1. **Preserve the anchor.** Sabbath #1, JDN 1,733,205, and the formula `SabbathJDN(n) = ANCHOR_JDN + 7 * (n - 1)` are **PROJECT-DEFINED** and immutable without explicit owner authorization.
2. **Calculate, do not enumerate.** Any Sabbath is computed from its number in constant time. The system never walks week by week from #1 and never stores every Sabbath.
3. **Exact integers.** Authoritative Sabbath arithmetic uses arbitrary-precision integers. Floating point is forbidden for those results.
4. **Separate layers.** Chronology core, calendar adapters, UI, and optional AI are different modules. Only the core assigns Sabbath numbers.
5. **Say what is unsupported.** A missing conversion is an unsupported result with a reason. It is not a guessed date.
6. **Label evidence.** Project convention, mathematical derivation, historical document, astronomical calculation, reconstruction, and theological interpretation stay distinct.
7. **Keep the legacy app.** The original ChristianCalendar remains a working reference. V2 does not rewrite it in place.
8. **No mandatory cost.** Basic use has no account, no database, and no paid API.

## Success looks like

- Sabbath #1, #0, #−1, a large positive number, and a large negative number all satisfy the invariants in [04-MATHEMATICAL-FOUNDATION.md](04-MATHEMATICAL-FOUNDATION.md).
- 11 April 2026 still resolves to JDN 2,461,142 and Sabbath #103,992.
- Julian 4 April AD 33 and proleptic Gregorian 2 April AD 33 both show Sabbath #1 and the same JDN.
- A request outside a calendar’s verified range shows unsupported, not a fabricated label.
- The timeline cursor, the proof card, and the URL `?n=` agree, including for negative `n`.
- The page still works with every AI feature removed.

## Non-goals for MVP

- A Creation start date.
- Location-based sunset times.
- A public conversational assistant.
- Automatic merges, deploys, or rule changes by an agent.
- Replacing the owner’s sketch with a different primary layout.

## Relationship to the sketch

The owner’s drawing is the primary UI inspiration: a black field, a gold cross at Sabbath #1, one shared vertical cursor, and cyan rows labeled Sabbath Calendar, Jewish, Gregorian, Chinese, and Julian. Row order in the drawing wins over the alternate order listed in the prompt. The drawing’s red callouts (“Sabbath #−1”, “Sabbath #104,017”, “10/3/2026”) illustrate the idea of aligned labels. They are not verified conversion results. The verified present-day oracle remains 11 April 2026 → Sabbath #103,992.

## Vision statement

The Sabbath Calendar is a complete seven-day index. Sabbath #1 is permanent. The index runs toward Sabbath #−N and Sabbath #+N. Every other calendar is a synchronized reading of the same days, shown only where that reading is actually supported.
