# Design: Meridian Markets data handling checklist

**Date:** 2026-09-20
**Status:** Approved design, not yet implemented
**Source:** `raw/client-brief.md` (Dana Okafor, VP Operations, August 2026)

## Problem

Meridian's NDA draws a hard line: customer records and employee data must
never enter Claude, ChatGPT, Copilot, or any other AI tool. That covers the
loyalty program data, the labor schedules, and any excerpt of them. Sales
totals by store and week, and the store attributes, are explicitly cleared for
use with those tools.

The team needs that line turned into something it can actually follow while
doing eight weeks of analysis — including at 11pm before a deliverable, which
is when a rule that requires interpretation fails. The same document has to
stand up as a client-facing assurance artifact and as a Workshop 1
deliverable.

## Goals

- Make the restricted/shareable boundary impossible to get wrong in the moment.
- Cover the full data lifecycle, from Marcus's extract to eventual disposal.
- Serve three readers at once: the team, the client and its counsel, and the
  course.
- Make the safe path the easy path through repo structure, not willpower alone.

## Non-goals

- Incident response procedure. Deliberately out of scope for v1.
- Named governance roles or a sign-off log.
- Automated enforcement (pre-commit hooks, PII scanners). Structure and
  `.gitignore` only.
- Tool-by-tool configuration guidance. See "AI principles" below.

## Decisions

Each of these was chosen deliberately during design; the rationale matters more
than the choice if this is ever revisited.

### D1 — One document, three audiences

A single `docs/data-handling-checklist.md` rather than separate internal and
client versions. Two documents drift, and the drift always favors the one
nobody reads. The appendix structure carries the client and course weight
without diluting the operational front half.

### D2 — Strict lineage over an aggregation threshold

Anything derived from loyalty or labor data inherits its restriction,
regardless of how aggregated it is. An aggregation threshold would be more
analytically convenient, but it requires judgment at exactly the moment
judgment is worst. The strict rule can be applied correctly by a tired person
in five seconds.

Consequence: `data/shareable/` can only ever contain outputs computed from
shareable sources. Each export's derivation note is therefore also the proof
that it belongs there.

### D3 — Restricted data lives outside the repo

Restricted data never sits inside the working tree. Scripts locate it through
a `MERIDIAN_DATA_ROOT` environment variable. A tracked `data/shareable/` holds
cleared aggregates, each accompanied by a note on how it was derived.
`.gitignore` rules act as a backstop, not as the primary control — you cannot
commit what was never in the tree.

### D4 — Tool-agnostic AI principles

The rules are stated about the data, not about products: restricted data does
not leave the local machine and does not enter a service that transmits or
trains on it. This survives tool changes over the engagement and covers tools
not yet adopted. Note that D3 does most of the enforcement work here — an
agentic tool running in the repo cannot reach a data root that is outside it.

### D5 — Disposal on request

The team maintains a record of where restricted copies live and deletes on
Dana's request rather than automatically at engagement close.

This is the document's weakest clause and was flagged as such during design:
an inventory with no deletion trigger is what counsel is most likely to push
on. It is recorded in Appendix B as an open question for Dana rather than
being resolved unilaterally. Shareable aggregates, code, and documentation are
retained regardless.

### D6 — Row-level POS treated as Restricted

The brief clears "sales totals by store and week" and forbids loyalty data,
but is silent on row-level POS transactions. If those rows carry a loyalty
identifier, they are loyalty data in substance. They are classified Restricted
pending Dana's answer (Appendix B). Dana's brief explicitly invites questions.

## Document structure

```
Red lines            4 lines, top of page. What never goes in an AI tool.
§1 Classification    The table. Single source of truth for every dataset.
§2 The lineage rule  One test, stated once, with two worked examples.
§3 Lifecycle         Receive → Store → Access → Analyze → AI → Share → Dispose
§4 AI principles     Tool-agnostic rules about the data, not the products.
Appendix A           NDA mapping — which line of the brief each rule comes from.
Appendix B           Open questions for Dana.
```

The table and the lineage rule serve counsel and the course. The lifecycle
checkboxes serve the team. Appendix A is what makes the document credible as a
client-facing artifact: every rule traces to a line in Dana's brief rather than
to the team's invention.

### §1 Classification table

| Dataset | Class | AI tools | Tracked in git |
|---|---|---|---|
| POS transactions, row level | Restricted | No | No |
| Sales totals by store × week | Shareable | Yes | Yes |
| Loyalty membership & purchase history | Restricted | Never | No |
| Labor scheduling & hours | Restricted | Never | No |
| Store attributes (sq ft, opening date, lease terms) | Shareable | Yes | Yes |
| Anything derived from a Restricted row | Restricted | No | No |

### §2 The lineage rule

Stated as a single question:

> Could this number have been produced without ever touching loyalty or labor
> data? If no, it is Restricted — no matter how aggregated it is.

Worked examples:

- Revenue per square foot by store — **Shareable.** POS totals plus store
  attributes; no restricted input.
- Average basket size for loyalty members at Pasadena — **Restricted.** Derived
  from loyalty data; aggregation does not rescue it.

### §3 Lifecycle phases

Checkbox lists, each phase referencing §1 for classification.

- **Receive** — obtain the extract from Marcus only after the NDA is signed;
  classify every file on arrival per §1; place it under the correct root.
- **Store** — restricted data under `MERIDIAN_DATA_ROOT`, outside the repo;
  shareable exports in `data/shareable/` with derivation notes.
- **Access** — restricted data stays on team machines; no copies to personal
  cloud storage or email.
- **Analyze** — every output is classified before it is saved; the lineage rule
  decides.
- **Use an AI tool** — §4 principles; confirm the input's class per §1 first.
- **Share** — three audiences, below.
- **Dispose** — maintain the location record; delete on request (D5).

### Sharing

Three distinct audiences, each with its own rule:

1. **Client** — deliverables to Dana and the board, including the preliminary
   readout at week three.
2. **Academic** — instructors, classmates, workshop presentations.
3. **Post-engagement portfolio** — methodology and anonymized or synthetic
   figures only; no real store-level numbers without written permission.

## Repo changes

1. `docs/data-handling-checklist.md` — the checklist itself.
2. `data/shareable/` — tracked, with a `README.md` stating what may live there
   and requiring a derivation note per export.
3. `.gitignore` — backstop rules excluding common restricted-data patterns and
   any `data/restricted/` path created by mistake.
4. `MERIDIAN_DATA_ROOT` — documented in the checklist and in the top-level
   `README.md`.

## Open questions for Dana (Appendix B content)

- Are row-level POS transactions restricted, particularly where they carry a
  loyalty identifier? (D6)
- What are Meridian's expectations for data destruction at engagement close?
  (D5)
- May store-level figures appear in academic presentations to instructors and
  classmates, and in a portfolio afterward?
