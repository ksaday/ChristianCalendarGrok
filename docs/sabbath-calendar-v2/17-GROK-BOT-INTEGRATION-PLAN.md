# 17 — Grok Bot Integration Plan

Three different tools get mixed together in conversation. They stay separate here.

| Tool | What it is | In this project |
| --- | --- | --- |
| A Cursor coding model (Grok 4.7, Grok 4.6, Grok 4.5, Composer 2.5, or another model the account can select) | The model that edits and reviews in a chat or cloud agent | Use freely for design and later code. It is not an authority for `n` |
| Grok Bot / Cloud Agents / Automations | Cursor products that run agents on a schedule or on GitHub events, on Premium and above | Optional owner-side Auditor. Not the website |
| Grok API | A separately billed HTTP API | Only relevant to a future public chatbot. Not included in the Cursor usage pool |

Sources checked 11 November 2026:

- https://cursor.com/docs/models-and-pricing
- https://cursor.com/docs/cloud-agent
- https://cursor.com/docs/cloud-agent/automations
- https://cursor.com/help/models-and-usage/available-models

Re-check those pages before Phase 5. Plan names and prices move. **Label: VERIFIED as of that date, not a permanent tariff.**

## 17.1 Coding models

Public list on that date, for Cursor’s own models:

| Model | Fit for this project | Not a rule |
| --- | --- | --- |
| Grok 4.7 | Architecture review, math review, long docs, hard bugs. Long context above 256k is priced higher on the public table | Good default for chronology diffs |
| Grok 4.6 and 4.5 | Same family, when 4.7 is unnecessary or unavailable | Acceptable |
| Composer 2.5 | Mechanical edits, fixtures, wireframe CSS | Weaker choice for a subtle negative-division review |

Plus includes those models at fixed settings and excludes Cloud Agents, Automations, Bugbot, and Grok Bot. Premium adds those products and bills third-party models (Claude, GPT, Gemini) from prepaid credit at list price. The weekly usage pool is shared by editor agents, Cloud Agents, and Grok Bot. It is finite.

**Workflow, not a project rule:** pick any available model. Keep docs 04–06 and the tests as the spec so a model switch cannot rewrite the foundation. If the account does not list a model named in an old chat, use the closest Cursor model and do not block the work.

Third-party models are optional reviewers. They are not required. If used, their output is still subordinate to the test gate.

## 17.2 What “Grok Bot” should mean here

**PROPOSED** role name: **Sabbath Calendar Auditor**.

It is an optional Cursor Automation (or a manually launched cloud agent) that:

1. Reads the chronology code and the design docs.
2. Recomputes selected samples, including negatives and zero.
3. Looks for citations on Hebrew, Chinese, and historical markers.
4. Reads test output when the owner provides it or when the agent is allowed to run `node --test`.
5. Writes a structured report for the owner.
6. Does not treat a reimplementation that copies `SabbathJDN` and prints “verified” as an independent proof. Independence means the checks in doc 12 §12.4: a second spelling of the formula in the test, external calendar vectors, round trips, and boundary samples.

It is not a chatbot inside the calendar website.

## 17.3 Minimum permissions

| Capability | Auditor default |
| --- | --- |
| Read this repository | Yes |
| Run tests | Yes, when a human or a scheduled job asks for a review |
| Comment on a PR | Only if the owner turns that tool on |
| Commit | Off |
| Open or merge a PR | Off |
| Deploy, tags, releases | Off |
| Edit `ANCHOR_JDN` | Off, even if commit is later turned on for some other bot |
| Post to public Slack or social | Off until a report channel is chosen |
| Secrets | None required for the arithmetic review |

Create the automation with the smallest tool set. Add “comment on pull request” only for the PR-review routine in doc 18. Do not add a tool because the template offers it.

Repository choice: `ChristianCalendarGrok` when code exists. The Auditor may read `ChristianCalendar` as a second repo for oracle comparison, read-only. It must not push to the legacy repo.

## 17.4 Cost

- Automations create cloud agents and are billed as cloud-agent usage (Cursor automations doc).
- There is no context-window toggle on automations. They use the model’s maximum window. A weekly “read the whole history of calendars” prompt can be expensive. Keep prompts pointed at `src/chronology`, `src/calendars`, `test`, and `docs/sabbath-calendar-v2`.
- Plus cannot run this product tier, as of the page cited above. If the owner is on Plus, Phase 5 waits on an upgrade or stays manual: the owner pastes a diff into a normal chat and asks for the same report. Manual chat still consumes the weekly pool, but it does not need Automations.
- Do not buy API credit for the Auditor. It does not need the Grok API.

## 17.5 Independence method

When the Auditor checks a sample `n`:

1. Compute `1733205 + 7*(n-1)` in its own scratch arithmetic and compare to the function output or to the fixture.
2. Check `n`, `0`, `n+1` so the step is 7, including across zero.
3. For a civil date, prefer a published example or the oracle pack over “the adapter returned what the adapter returned”.
4. Say **recomputed** when (1) matches, and **externally checked** only when a citation outside the repository agrees.
5. Say **not checked** when it only read the code.

A report that says “verified” without one of those words is incomplete. The owner can send it back.

## 17.6 Disagreement path

```text
test failure  -->  blocks release, regardless of the Auditor
Auditor finds a citation gap  -->  issue, row stays unsupported
Auditor disagrees with a passing test  -->  owner decides; nobody edits the anchor inside the review
Auditor and tests agree on a bug  -->  a human or a coding session fixes it in a PR, then tests run again
```

No model overrides a failed test by assertion (REQ-QA-003).

## 17.7 What this phase did not do

No automation was created. No GitHub connection was added. No routine was saved. Enabling them is checklist E4 and F-items, plus Phase 5.
