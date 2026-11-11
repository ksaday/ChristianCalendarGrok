# 16 — Cursor Rules Proposal

**PROPOSED. Not installed.** This phase does not create `.cursor/rules/`, `AGENTS.md`, or `CLAUDE.md`.

## 16.1 What already exists

| Location | Finding |
| --- | --- |
| `ksaday/ChristianCalendar` | No `AGENTS.md`, no `CLAUDE.md`, no `.cursor/rules/` (**VERIFIED** by tree listing) |
| `ChristianCalendarGrok` | No agent instruction files at inspection, before this design set |

There is nothing to reuse and nothing to conflict with. Do not invent an `AGENTS.md` in the legacy repo. If the owner later wants one file for agents that do not load `.mdc` rules, add `AGENTS.md` in **this** repository only, and make it a short pointer:

```text
Read docs/sabbath-calendar-v2/00-EXECUTIVE-SUMMARY.md and
docs/sabbath-calendar-v2/16-CURSOR-RULES-PROPOSAL.md.
The anchor is JDN 1733205. Do not change it without an explicit owner instruction
in the current conversation.
```

That pointer is also **PROPOSED**, not created.

## 16.2 Model independence

Rules must not name Grok 4.7, Composer, or any other model id. The owner can switch coding models per chat. The arithmetic does not change when the model changes. A workflow note (not a rule) may say that a stronger reasoning model is a good fit for reviewing chronology diffs and a faster model is a good fit for mechanical refactors. That note lives in doc 17 so the rules stay stable.

## 16.3 Proposed files

Install only after checklist item F1. Suggested directory:

```text
.cursor/rules/sabbath-anchor.mdc
.cursor/rules/chronology-mathematics.mdc
.cursor/rules/calendar-adapters.mdc
.cursor/rules/historical-evidence.mdc
.cursor/rules/verification-standards.mdc
.cursor/rules/implementation-approval.mdc
```

Each file’s proposed body follows. Frontmatter `alwaysApply: true` is appropriate because a chronology mistake can land in any file. If the owner prefers scoped rules, set `globs` to `src/chronology/**`, `src/calendars/**`, and `test/**` for the math rules, and keep the anchor rule always on.

### `sabbath-anchor.mdc`

```markdown
---
description: Immutable Sabbath #1 anchor for Sabbath Calendar V2
alwaysApply: true
---

# Immutable anchor

Sabbath #1 is a project-defined constant.

- ANCHOR_JDN = 1733205
- Civil identity: Saturday, 4 April AD 33, Julian calendar
- SabbathJDN(n) = 1733205 + 7 * (n - 1) for every integer n, including 0 and negatives
- Sabbath #0 is valid. Do not skip zero. Do not renumber negatives so that the Saturday before #1 becomes #-1.

Do not change the constant, the formula, or the meaning of n unless the owner explicitly authorizes that change in the current instruction. A review comment, a paper, a library update, or a model preference is not authorization.

The legacy repository ksaday/ChristianCalendar is read-only reference. Do not open pull requests that edit it as part of V2 work.

Preserve the regression oracle: proleptic Gregorian 11 April 2026 is JDN 2461142 and Sabbath #103992.
```

### `chronology-mathematics.mdc`

```markdown
---
description: Exact integer rules for Sabbath chronology
alwaysApply: true
---

# Mathematical precision

Authoritative Sabbath numbers, absolute days, and the quotients that produce them use bigint.

- Floor division (toward negative infinity) is required. BigInt truncating division is not acceptable for these algorithms.
- Modulo is Euclidean: remainder in 0 .. m-1 for positive m.
- Do not use JavaScript number, Math.floor on a number, or Date for authoritative chronology.
- Do not loop week by week from the anchor to reach a distant n.
- Do not coerce a bigint JDN or Sabbath number through Number.
- Pixel layout may use floating point only for screen positions derived from a safe delta. Never parse pixels back into a JDN.
- Canonical serialization is ASCII decimal, optional leading -, no leading zeros, no exponent notation.

If a change touches src/chronology or calendar conversion, run the property tests. A failing test stays failing. Do not explain it away.
```

### `calendar-adapters.mdc`

```markdown
---
description: Calendar adapters must not define the Sabbath index
globs: src/calendars/**,src/ui/**
alwaysApply: false
---

# Calendar separation

The Sabbath index comes only from the chronology core.

- Adapters convert labels to and from an absolute day.
- Adapters do not export Sabbath numbers and do not write ANCHOR_JDN.
- Proleptic Gregorian and Julian are different adapters. Do not hard-code a fixed day gap.
- Hebrew output is the calculated civil calendar, and only inside its verified range. Do not present it as observed ancient practice.
- Chinese output is allowed only inside a cited verified range. Otherwise return unsupported.
- Unsupported is a normal result. Do not invent a date to fill a row.
- Historical reform notes do not punch holes in the JDN sequence.
```

### `historical-evidence.mdc`

```markdown
---
description: Evidence labels for historical and theological claims
alwaysApply: true
---

# Evidence quality

Do not fabricate dates, citations, quotations, or conversion results.

Label claims in docs and UI copy:

- PROJECT-DEFINED for the anchor and other owner conventions
- MATHEMATICALLY DERIVED for results of the formula
- VERIFIED when checked against the legacy software or an independent calculation recorded in docs/sabbath-calendar-v2
- PROPOSED for design choices not yet accepted
- UNRESOLVED when evidence was deliberately not invented

Do not assign a Creation JDN. Do not claim that extrapolating Sabbath numbers proves continuous observance or locates the seventh day of Genesis.

Markers need a source string. No source, no marker.
```

### `verification-standards.mdc`

```markdown
---
description: Tests that must pass before a chronology change is done
globs: src/**,test/**
alwaysApply: false
---

# Test discipline

Any change to chronology or adapters must pass:

- Sabbath #1, #0, #-1 round trips
- The 11 April 2026 oracle
- Property tests: step of 7 and inverse, including negatives and values above 2^53
- floorDiv(-8, 7) === -2 and euclidMod(-1, 7) === 6
- Adapter vectors that are cited, and unsupported behavior where the verified range is empty

Add a test for a bug before fixing it when the bug is arithmetic.

An AI review is optional and cannot override a red test.
```

### `implementation-approval.mdc`

```markdown
---
description: Stay inside the phase the owner has approved
alwaysApply: true
---

# Approval boundary

Read docs/sabbath-calendar-v2/21-OWNER-APPROVAL-CHECKLIST.md before starting a new phase.

- Design documents under docs/sabbath-calendar-v2 do not authorize application code by themselves.
- Do not install these rules, add dependencies, enable automations, deploy, or publish unless the current instruction says so.
- Do not add a public chatbot, a sunset engine, or a Creation date as a drive-by.
- Do not commit or push unless the owner asked.

When a task is ambiguous, stop at the smallest change that matches the approved phase.
```

## 16.4 Conflict rule

If a future `AGENTS.md` disagrees with these files, the anchor rule wins until the owner edits the constant on purpose. Update both in the same change so agents do not see two anchors.

## 16.5 What not to add

- A rule that names a vendor model.
- A rule that tells agents to “be helpful and fill unknown dates”.
- A rule that allows waiving tests when a deadline is close.
