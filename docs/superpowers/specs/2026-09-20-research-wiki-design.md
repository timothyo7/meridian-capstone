# Design: Meridian research wiki

**Date:** 2026-09-20
**Status:** Approved design, not yet implemented
**Sources:** `raw/client-brief.md`; the LLM Wiki pattern
(https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
**Related:** [data handling checklist design](2026-09-20-data-handling-checklist-design.md)

## Purpose

The team has one dense client brief and a stakeholder interview with Dana
Okafor ahead of it. Dana is the scarce resource: she travels Tuesdays and
Wednesdays, replies slowly, and her assistant cannot answer analytics
questions. Interview time has to be spent on what only she can settle.

Reading the brief once and arriving with a list of questions wastes that. What
the team needs is accumulated understanding — the brief's claims made explicit,
external research folded in against them, and the gaps that remain surfaced as
questions with a traceable reason for existing.

This spec instantiates the LLM Wiki pattern for that purpose. The wiki exists
so the team walks into the interview informed; `analysis/interview-guide.md` is
the artifact they carry into the room.

It must:

- Accumulate rather than re-derive — each new source updates the existing
  synthesis instead of being read fresh at query time.
- Make every interview question traceable back through the page, and through it
  the source, that motivated it.
- Remain useful after the interview, when the register becomes the record of
  Dana's answers.

## Scope

**In scope**

- The three layers below, instantiated in this repository.
- Ingest, query, and lint workflows, documented in the schema so they survive
  across sessions.
- A themed question register and the interview guide generated from it.
- Sources: the client brief, plus external public research (industry,
  demographics, competitors, the Pasadena trade area).

**Out of scope**

- Search tooling such as qmd. `index.md` is sufficient at this scale
  (one source now, tens expected).
- Obsidian-specific features — graph view, backlinks, Dataview. The user does
  not run Obsidian (see D3).
- Any ingest of the client data extract. Structurally prohibited (see D7).
- Slide or Marp output. Markdown only.
- A fifth `assumptions/` page type. The register carries that load (see D2).

## Layers

The pattern defines three layers. Mapped onto this engagement:

**Raw sources** — `raw/`. Immutable. The user adds documents; the agent reads
but never modifies them. Currently `client-brief.md`; external research is
saved here as local Markdown rather than referenced by URL, so the corpus stays
reviewable and stable (see D7).

**The wiki** — `wiki/`. Agent-owned in its entirety. The agent creates pages,
updates them as sources arrive, maintains cross-references, and keeps the
catalog and log current. The team reads this layer; the agent writes it.

**The schema** — `wiki/CLAUDE.md`. Page formats, naming conventions, and the
ingest/query/lint workflows, plus the data-handling rule of D7. This is what
makes the agent a disciplined maintainer across sessions rather than a fresh
chatbot each time. A short root `CLAUDE.md` points to it without loading the
whole schema into unrelated sessions.

## Directory layout

```
raw/                        immutable; the user adds, the agent never modifies
  client-brief.md
  <research sources>.md

wiki/
  CLAUDE.md                 schema: page formats, workflows, conventions, D7 rule
  index.md                  catalog by category: page, link, one-line summary
  log.md                    append-only, "## [YYYY-MM-DD] ingest | N sources"
  sources/                  one page per raw document, with frontmatter
  entities/                 Dana, Marcus, the board, stores, competitors, POS system
  concepts/                 expansion strategy, loyalty program, prepared foods,
                            unit economics
  analysis/
    open-questions.md       the register
    interview-guide.md      the deliverable, generated from the register
    <synthesis pages>

CLAUDE.md                   root pointer to wiki/CLAUDE.md
```

## Decisions

### D1 — The guide is derived, the wiki is the point

The wiki's purpose is shared understanding; `analysis/interview-guide.md` is a
generated view, not the thing being maintained. The team reads the wiki and
walks in informed; the guide is what they carry into the room. This means the
guide can be regenerated at any time and is never edited in place as the
system of record — the register is.

### D2 — Karpathy's four page types, adapted

`sources/` (one summary per ingested document), `entities/` (people, stores,
competitors, systems), `concepts/` (topics such as expansion strategy and the
loyalty program), `analysis/` (synthesis pages, the register, the guide), plus
`index.md` and `log.md`.

A domain-specific `assumptions/` directory was considered and rejected: the
register (D4) carries that load without a fifth page type, and the generic
taxonomy keeps the wiki useful if it outlives the interview.

### D3 — Relative Markdown links; frontmatter on source pages only

Navigation uses standard relative links (`[Dana](../entities/dana-okafor.md)`),
clickable in VS Code and on GitHub. Wikilink syntax was rejected because the
user does not run Obsidian, which is the only reader that resolves it — without
it, wikilinks are inert text and the graph view that justifies them does not
exist.

Frontmatter is applied to `sources/` pages only, where origin and ingest date
genuinely need to be machine-readable for tracking what has been processed.
Entity, concept, and analysis pages start at their heading.

### D4 — A running register, maintained on every ingest

`analysis/open-questions.md` holds one row per open question, updated during
ingest rather than harvested at the end. Inline "open questions" sections on
individual pages were rejected as the system of record: questions raised in a
page nobody reopens do not reach the guide.

Columns: ID, question, theme, origin page, why it matters, status. Status moves
`open` → `answered` → `superseded`. Answered rows retain Dana's answer inline,
so after the interview the register is the interview record.

### D5 — Grouped by theme, not globally ranked

Questions cluster into five themes rather than being ranked one through thirty:

1. Scope & success criteria
2. Data & access
3. The Pasadena assumption
4. Business & operations
5. Stakeholders & process

Interviews flow by topic; a globally ranked list forces topic-switching that
costs rapport and context. Within each theme, up to three questions are marked
as the ones that survive if Dana's time collapses.

### D6 — Batch ingest with automatic lint

Sources are processed in batches rather than one at a time with discussion.
Each batch: read every source, write its source page, update every affected
entity and concept page, add register rows, then append one log entry naming
the sources in and the pages touched.

A lint pass runs automatically at the end of every batch, on the reasoning that
a batch is exactly what introduces contradictions between sources read in the
same pass. Lint checks: contradictions between pages, claims superseded by a
newer source, orphan pages with no inbound links, concepts referenced
repeatedly but lacking a page, assertions with no source link, and register
hygiene (duplicates, questions the batch just answered). Findings are reported
rather than silently resolved where they require judgment.

### D7 — The wiki is Shareable-class by construction

`wiki/CLAUDE.md` carries a hard rule: nothing under `MERIDIAN_DATA_ROOT` ever
becomes a wiki source. The wiki is written by an AI tool, so under the
[data handling checklist](2026-09-20-data-handling-checklist-design.md) every
page in it is Shareable-class by definition. Marcus's extract is analyzed in
code and never ingested here.

Two consequences the schema must state explicitly:

- Research sources are saved into `raw/` as local Markdown rather than
  referenced as URLs — immutable local copies per the pattern, and it keeps the
  corpus reviewable.
- If Dana volunteers customer or employee specifics during the interview, those
  notes live in restricted storage outside the repo. Only non-restricted
  content reaches the register.

### D8 — Wiki in this repo; schema in `wiki/CLAUDE.md`

`wiki/` is a directory of this repository, not a separate vault, so it shares
the repo's git history and the team's existing access. `wiki/CLAUDE.md` holds
page formats, naming conventions, and the ingest/query/lint workflows. A short
root `CLAUDE.md` tells Claude the wiki exists and when to open it, without
loading the full schema into every unrelated session.

## Page formats

### Source page

```markdown
---
source: raw/client-brief.md
ingested: 2026-09-20
type: client-document
---
# Client brief — Dana Okafor, Aug 2026

## What it says
## Key claims          ← each a candidate for the register
## What it changes     ← pages updated because of this source
## Questions it raises
```

### Entity and concept pages

No frontmatter. Sections: **What we know** (every assertion linking to the
source page behind it) → **Open questions** → **Related pages**. The linking
discipline is what lets a guide question trace back through a page to a
document.

### Register row

| ID | Question | Theme | Origin | Why it matters | Status |
|---|---|---|---|---|---|
| Q7 | Does the Pasadena site have a signed LOI, or is it still exploratory? | Pasadena | `concepts/expansion-strategy.md` | Determines whether the engagement validates a decision or informs one | open |

### Interview guide

Grouped by the five themes of D5, in order. Each question carries its register
ID and a link to its origin page. Each theme opens with a framing sentence and
marks its top three.

### Log entry

```markdown
## [2026-09-20] ingest | 4 sources
Sources: industry-report-2026, la-county-demographics, ...
Pages touched: entities/pasadena-site, concepts/expansion-strategy, ...
Register: +6 open, 1 answered
```

The consistent `## [` prefix keeps the log parseable with
`grep "^## \[" wiki/log.md | tail -5`.

## First build

1. Scaffold `wiki/` and its subdirectories.
2. Write `wiki/CLAUDE.md` — the schema, including the D7 data rule.
3. Write the root `CLAUDE.md` pointer.
4. Ingest `raw/client-brief.md` as source #1.
5. Create the entity and concept pages the brief justifies.
6. Seed the register from the brief's claims and, as importantly, its silences.
7. Write `index.md` and the first `log.md` entry.

The result is a working wiki off a single source, ready to accept research.

## Open questions

- Which external sources to gather first, and whether any require access the
  team does not have.
- Whether the register should carry a sixth theme for questions aimed at
  Marcus rather than Dana, or whether an owner column is the better fit.
