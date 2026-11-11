# 19 — Optional AI Assistant Feasibility

## 19.1 Question

Should the public Sabbath Calendar include a conversational assistant that answers questions such as “Show Sabbath #−1000” or “Why does #0 come before #1”?

**Recommendation: not in the MVP.** Ship Option A. Keep Option B as the owner-side Auditor in docs 17 and 18. Leave Option C unbuilt until the costs and controls below are accepted in a later phase.

## 19.2 Options

### Option A — Deterministic UI only

Search, proof card, cycle inspector, and the short help text answer the example questions without a model.

| Example question | Deterministic answer |
| --- | --- |
| Show Sabbath #−1000 | Search box, URL `?n=-1000`, proof card |
| Gregorian date of Sabbath #50000 | Gregorian row and proof card |
| Hebrew label, if supported | Hebrew row, or “unsupported” |
| Why does #0 occur before #1? | Help text: the formula is linear, zero is the Saturday one week before #1 |
| Julian versus Gregorian | Row labels plus the anchor pair 4 April / 2 April 33 |
| Mathematical verification | Proof card from `verifyChronologyInvariants` |

This option has no per-request fee, no key, and no way for a model to invent a date. It is the MVP (REQ-AI-001, REQ-AI-004).

### Option B — Owner-side Auditor

Covered by docs 17 and 18. It never faces the public. It can be declined without changing the site. A personal Grok Bot or Cursor subscription does not become a public API. **VERIFIED** from the pricing page on 11 November 2026: the Grok API is billed separately and does not draw from the Cursor weekly pool.

### Option C — Public chatbot

A later website feature. A visitor types a question. A server calls a model. The model may call chronology tools. The browser never holds the API credential.

This is **PROPOSED** as a design sketch only.

## 19.3 Mandatory shape if Option C is ever built

```text
browser chat box
    -->  same-origin server route (rate limited)
          -->  model, with tools only
                tools: resolveSabbath, resolveDate, explainInvariant, readAdapterMeta
                tools call the same chronology build the site uses
          <--  answer that quotes tool JSON
    <--  rendered answer, with uncertainty labels
```

Rules:

- The model cannot answer a “what Sabbath is this” question without a tool result. If the tool returns `unsupported` or `not-a-sabbath`, the answer says that.
- The model does not receive a general code-execution tool. It does not `eval` visitor input.
- Historical sentences require a citation already stored in the repo (markers or docs). Otherwise the answer says the project does not have a cited date.
- The system prompt states the anchor and states that the model must not propose a different one. The tool layer still ignores any model-supplied JDN that disagrees with the formula.
- Removing the server and the chat box leaves the static calendar working. No chronology import points at the chat module.
- Provider: a thin interface with one implementation. Grok’s API is a candidate, not a requirement. Another provider can sit behind the same tools. The interface is **PROPOSED** as `complete(prompt, toolResults) -> text` plus a tool schema. No SDK choice is made here.
- The server loads the credential from the host’s secret store. The static files do not contain it. Logs store hashes or nothing, not full transcripts, unless the owner later writes a retention policy. Default: do not store transcripts.
- Rate limit per IP and a maximum input length. Reject inputs that are not questions about the calendar (the tool list is the real boundary; the length cap is the cheap one).
- Show a visible notice: explanations are generated, numbers are from the chronology engine, and the visitor can open the proof card for the same `n`.

## 19.4 Cost

| Path | Who pays | MVP |
| --- | --- | --- |
| Option A | Nobody per visit. Static hosting only | Yes |
| Option B | Owner’s Cursor weekly pool, and only if they enable it | No |
| Option C | Owner’s API bill, unbounded with traffic unless capped | No |

A quiet cap for a future Option C, **PROPOSED** if it is ever built: a monthly budget the server enforces, after which the chat returns “assistant unavailable” and the calendar keeps working. The number is **UNRESOLVED** because it is a spending decision.

There is no hidden request from the static page to an API “just for explanations”.

## 19.5 Security

| Threat | MVP stance | If Option C exists |
| --- | --- | --- |
| Key in the repository or the JS bundle | Do not add a key | Server-only secret |
| Visitor expression executed | Search parses integers and dates only | Tools parse integers and dates only |
| Prompt injection via a marker or doc | Docs are owner-written | Tools return data. Instructions in a marker are data, and the model is told not to follow them. Still not a proof against a bad model. Another reason to stay on Option A |
| Invented Sabbath | Impossible if the UI only prints core output | Tool mismatch is shown as an error, not as the model’s number |
| Personal data | Do not ask for accounts | Do not add accounts in the same phase |
| Abuse and cost | None | Rate limit and monthly cap |

## 19.6 Comparison

|  | A. No chatbot | B. Owner Auditor | C. Public chat |
| --- | --- | --- | --- |
| Arithmetic reliability | Highest. Tests only | Tests still gate releases | Depends on the tool boundary staying intact |
| Cost | Lowest | Usage pool, opt-in | API bill grows with visitors |
| Complexity | Already in the UI plan | One automation | A server the rest of the design avoids |
| Public value | Proof card answers the numeric questions | None direct | Freer wording |
| Fits “no mandatory commercial service” | Yes | Yes, because it is absent unless enabled | No |

## 19.7 MVP configuration

Option A on the website. Option B documented and disabled. Option C not started.

Example questions in the brief are acceptance tests for the **UI copy and the proof card**, not for a model. Phase 3 is done only if those questions can be answered by search and the proof card.
