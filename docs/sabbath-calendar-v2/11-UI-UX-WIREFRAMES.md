# 11 — UI / UX Wireframes

Visual source: the owner’s sketch (black field, gold cross, one cursor, cyan rows). Row order follows the sketch (ADR-012). Numbers shown below are **VERIFIED** or **MATHEMATICALLY DERIVED** oracles. The sketch’s “#104,017” and “10/3/2026” are not used.

Palette for the timeline shell:

- Field `#000000`.
- Row track `#3ec6ff` (cyan), close to the drawing.
- Cross and Sabbath #1 mark: legacy gold `#B0842C` or brighter `#f5c542` so it reads on black.
- Cursor: 1px white or cyan vertical line.
- Unsupported text: `#8a8a8a`.
- Failure: `#e08a8a` (legacy `.selfcheck.fail`).

The parchment month page is not this shell (REQ-UI-010).

## 11.1 Desktop, week scale, cursor on Sabbath #1

```text
+------------------------------------------------------------------+
| Sabbath Calendar                                              [?] |
| Search [ Sabbath number v ] [ -1____________ ] [ Go ]            |
| Scale  ( day ) ( week ) ( year ) ( century ) ( millennium ) (age)|
+------------------------------------------------------------------+
|                                                                  |
|                         +  Sabbath #1                            |
|  Sabbath #0             |                          Sabbath #2   |
| ========================|======================================  |
|                         |                          Sabbath       |
| ========================|======================================  |
|  26 March 33 CE         |  2 April 33 CE           9 April 33 CE |
| ========================|======================================  |
|                         |                          Gregorian     |
| ========================|======================================  |
|  unsupported            |  unsupported             unsupported   |
| ========================|======================================  |
|                         |                          Chinese       |
| ========================|======================================  |
|  28 March 33 CE         |  4 April 33 CE           11 April 33   |
| ========================|======================================  |
|                         |                          Julian        |
+------------------------------------------------------------------+
| Proof                                                            |
| Sabbath #1                                                       |
| Anchor JDN 1733205                                               |
| Calculated JDN 1733205                                           |
| Offset 7*(1-1) = 0                                               |
| Formula SabbathJDN(n) = 1733205 + 7*(n-1)                        |
| Invariants: pass                                                 |
| Gregorian  2 April 33 CE     algorithmically calculated          |
| Julian     4 April 33 CE     algorithmically calculated          |
| Hebrew     unsupported       outside verified range              |
| Chinese    unsupported       outside verified range              |
| [ Seven-day cycle ]                                              |
+------------------------------------------------------------------+
```

The Gregorian labels in the picture match the baseline. The Julian labels were produced with the same Julian JDN formula as the baseline inspection, and they round-trip: Sabbath #0 is 28 March 33 CE Julian, Sabbath #1 is 4 April 33 CE Julian, Sabbath #2 is 11 April 33 CE Julian. **MATHEMATICALLY DERIVED.** Tests should assert those triples from the formula, not treat the ASCII art as a looser sketch. Hebrew and Chinese stay unsupported in this frame until a verified range includes these days.

Hebrew and Chinese say unsupported until their verified ranges include year 33. That is the correct MVP picture if those ranges are still empty. It is not a blank row.

## 11.2 Desktop, cursor on Sabbath #−1

Same chrome. Cursor moves left. Sabbath row reads `#−1`. Gregorian row reads 19 March 33 CE. Proof offset is `7*(-1-1) = -14`. JDN `1733191`. Invariants pass. URL is `?n=-1&scale=week`.

Search field accepts `-1`, `#-1`, and a pasted Unicode minus after normalization.

## 11.3 Desktop, present-day oracle

Search `103992`. Cursor shows Sabbath #103992, Gregorian 11 April 2026 CE, JDN 2461142. The cross of Sabbath #1 is off-screen to the left, available by a “Go to #1” control in the header, not only by dragging. Dragging week-by-week from 2026 back to year 33 is allowed and is not how search works.

## 11.4 Zoom

| Scale | Tick | Cursor identity |
| --- | --- | --- |
| day | one absolute day | unchanged `bigint` |
| week | Sabbath ticks, legacy default | unchanged |
| year | year labels from the selected row’s adapter, default Gregorian | unchanged |
| century | century labels | unchanged |
| millennium | millennium labels | unchanged |
| age | coarse ticks labeled by Sabbath-number ranges, not by float years | unchanged |

At year and coarser scales, Hebrew and Chinese ticks appear only where supported. Elsewhere the tick says “— unsupported —” without dropping the row’s baseline, so the columns still line up.

## 11.5 Seven-day inspector

Opened from the proof card for Sabbath #1:

```text
Week containing Sabbath #1
offset  weekday    Gregorian        Julian            Hebrew     Chinese
0       Sunday     (computed)       (computed)        unsupported ...
1       Monday     ...
2       Tuesday
3       Wednesday
4       Thursday
5       Friday
6       Saturday   2 April 33 CE    4 April 33 CE     unsupported ...
```

Only offset 6 carries the Sabbath number. Other rows show the civil date and the cycle’s Sabbath number as context (“week of Sabbath #1”), not as their own index.

## 11.6 Confidence and anchor

A lock-shaped text button “Project anchor” sits under the cross and opens:

```text
Sabbath #1 is a project-defined anchor.
Julian 4 April 33 CE.
JDN 1733205.
This screen does not treat that choice as a new historical proof.
```

## 11.7 Mobile (about 360px)

```text
+---------------------------+
| Sabbath Calendar          |
| Search                    |
| [ -1____________ ] [ Go ] |
| < week >                  |
+---------------------------+
|            + #1           |
| ============|============ |
| Sabbath     |             |
| ============|============ |
| Gregorian   | 2 Apr 33 CE |
| ============|============ |
| Hebrew      | unsupported |
| ============|============ |
| Chinese     | unsupported |
| ============|============ |
| Julian      | 4 Apr 33 CE |
+---------------------------+
| Proof                     |
| #1   JDN 1,733,205        |
| pass                      |
| [ Cycle ]                 |
+---------------------------+
```

Rows remain one shared cursor. The page scrolls vertically through rows. The timeline still scrolls horizontally. Zoom is a single control that cycles scale, plus the full list behind it. Proof card is below the rows, not a hover popover (hover does not exist for touch). The legacy popover pattern is not the mobile design.

Focus order: search, go, scale, then each row’s “details” button, then proof, then cycle.

## 11.8 Keyboard

| Key | Action |
| --- | --- |
| `/` | Focus search |
| Enter | Run search |
| Left / Right | Previous / next Sabbath when the cursor is a Sabbath. Previous / next day when the cursor is a midweek day |
| Home | Go to Sabbath #1 |
| `+` / `-` | Finer / coarser scale |
| Escape | Close inspector |
| `?` | Open the short help, which includes the anchor sentence from doc 08 |

Visible focus rings use gold on black, 2px.

## 11.9 Empty and error states

| State | Screen |
| --- | --- |
| First visit, no query | Cursor on “today’s” civil date from local `Date`, then converted with the Gregorian adapter. Core still does not read the clock. The UI layer may. Copy says the date is this device’s local civil date |
| Bad `n` | Message, no cursor, no adapter calls |
| `n` and `d` disagree | Message naming both values, no cursor |
| Invariant failure | Labels hidden, failure text from the report |
| Adapter unsupported | Row message, other rows normal |
| Reduced motion | Cursor jumps. No animated glide |

## 11.10 Share control

A “Copy link” button writes the current `?n=` or `?d=` URL. The button does not call a network API.

## 11.11 What the drawing’s red notes mean in the product

- “Sabbath #−1” is a real selectable index (doc 05).
- “Sabbath #1” under the cross is the only gold cross.
- A far-right Sabbath number and a Gregorian date illustrate alignment. The product uses computed labels. The red ink in the sketch is not a second data source.
