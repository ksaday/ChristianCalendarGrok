# 18 — Grok Bot Role and Routines

**PROPOSED and opt-in.** Do not create these routines during the design phase. Each routine needs a separate yes, a Cursor plan that includes Automations, and the permission ceiling in [17-GROK-BOT-INTEGRATION-PLAN.md](17-GROK-BOT-INTEGRATION-PLAN.md).

The role name is Sabbath Calendar Auditor.

## 18.1 Standing instructions

Paste this into the automation prompt if the routine is created. It is the whole job description.

```text
You are the Sabbath Calendar Auditor for ChristianCalendarGrok.
You review. You do not own the arithmetic.

Immutable facts:
- ANCHOR_JDN = 1733205
- SabbathJDN(n) = 1733205 + 7 * (n - 1)
- n = 0 and negative n are valid
- 11 April 2026 proleptic Gregorian = JDN 2461142 = Sabbath #103992

You must not:
- Change the anchor or the formula
- Commit, merge, deploy, or tag
- Invent calendar dates, citations, or quotations
- Call your own prose a proof
- Treat a third-party library release as authority to edit the anchor
- Mark a failing test as acceptable

When you check a Sabbath number, compute 1733205 + 7*(n-1) yourself and compare.
When you check a calendar label, require a cited vector or call it unsupported.
Classify each finding as recomputed, externally checked, or not checked.

Write a report with:
1. Samples checked and the class of each check
2. Test results if you ran them
3. Negative, zero, and large-value coverage
4. Claims that lack citations
5. Risks
6. Recommended changes, as suggestions, not as commits

If tests fail, say the release is blocked. Do not propose waiving them.
```

## 18.2 Routine A — Repository change review

- **Trigger (opt-in):** pull request opened or pushed, on `ChristianCalendarGrok` only.
- **Tools:** read repo, run tests, comment on the PR if the owner enables that single tool.
- **Work:** Summarize diffs under `src/chronology`, `src/calendars`, `test`, and the design docs. Flag any edit to `1733205`, any use of `number` in the core, any truncating division, any new calendar label without a citation, any UI path that parses pixels into a JDN.
- **Output:** PR comment or a Markdown report the owner reads. No commit.
- **Stop condition:** If the diff does not touch those paths, reply in one line that chronology files were not changed.
- **Cost control:** Ignore lockfiles and images unless the PR is only those files, in which case say so and stop.

## 18.3 Routine B — Weekly project quality review

- **Trigger (opt-in):** a weekly cron the owner chooses. Suggestion: Monday 15:00 in the owner’s timezone. The design does not pick the timezone for them. **UNRESOLVED** until they name it.
- **Repository:** this repo, default branch, no writes.
- **Work:** Run the test command if present. List open design UNRESOLVED items from doc 14 §14.7. Note docs and code that disagree on the anchor, on zero, or on unsupported ranges. Do not tour unrelated GitHub issues in other projects.
- **Output:** One report. If nothing changed and tests pass, the report is a few lines. Do not pad it.
- **Skip:** Do not run this until Phase 4 exists. Before that, it would only restate the design set.

## 18.4 Routine C — Calendar reference watch

- **Trigger (opt-in):** monthly, or manual.
- **Work:** Check the pinned edition of *Calendrical Calculations* only if the Hebrew adapter cites it, and check release notes of any library the repo actually depends on. Today the intended dependency list is empty, so this routine should exit with “no calendar libraries pinned” until one is added.
- **Forbidden conclusion:** “The library changed, so the anchor should change.”
- **Allowed conclusion:** “A new version exists. Here is the diff risk to adapters. Anchor untouched.”
- **Sources:** Prefer the project’s own citations and the vendor’s changelog. Do not scrape random forum posts into the report as facts.

## 18.5 Routine D — Pre-release verification

- **Trigger:** manual, when the owner names a commit they want to ship.
- **Work:** Run the full test command. Confirm doc 12 §12.8 steps. List UNRESOLVED Hebrew/Chinese ranges. Confirm the built site has no API key and no chat endpoint (REQ-AI-001).
- **Output:** A go / no-go note. No-go if any Must test fails. The Auditor does not create the git tag.
- **Owner:** The owner ships. The routine does not.

## 18.6 Report template

```text
# Auditor report
Date:
Commit:
Model: (name the model actually used; do not hide helper models)

## Recomputed
- n = ... expected JDN ... got ...

## Externally checked
- source ... result ...

## Not checked
- ...

## Tests
- command ... exit ...

## Citation gaps
- ...

## Risks
- ...

## Suggestions
- ...

Release: blocked | not blocked
Anchor file changed by this run: no
```

“Anchor file changed by this run: no” is mandatory. If it is “yes”, the run violated its permissions and the owner should discard those commits.

## 18.7 Planted-defect drill

Before trusting a new routine, the owner (or a coding session they asked for) may put a throwaway branch that:

- replaces floor division with truncating division, or
- deletes the `#0` fixture, or
- adds a Chinese date with no citation.

The Auditor should flag it and leave the branch unmerged. Delete the branch afterward. This drill is **PROPOSED**, not scheduled.

## 18.8 Risk list the Auditor maintains

Keep a short section in the weekly report, not a second source of truth beside doc 14. Point at doc 14 and add only new risks. Suggested standing risks:

1. Truncating division on negative years.
2. Hebrew or Chinese dates without citations.
3. Float conversion of large JDNs in the timeline.
4. Any PR against `ksaday/ChristianCalendar` from a V2 agent.
5. Usage-pool surprise from a verbose weekly prompt.
6. A helper model being invoked for browser testing (disable environment Testing if the owner wants the run to stay on the selected Cursor model).

## 18.9 Creation of the routines

When the owner asks, create one routine at a time, starting with Routine A or with a manual cloud-agent run that has no schedule. Do not enable all four at once. Record the automation id in doc 14 or in the checklist notes after it exists. This file should then change “PROPOSED” to “ENABLED” for that routine only.
