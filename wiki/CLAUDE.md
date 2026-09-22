# Wiki schema

This directory is a research wiki maintained by Claude for the Meridian
Markets engagement. Claude writes every page here; the team reads them.

Its purpose: the team walks into the Dana Okafor stakeholder interview
informed, and `analysis/interview-guide.md` is what they carry into the room.

## Hard rule: what may enter this wiki

**Nothing classified Restricted may ever become a wiki source or appear on a
wiki page.** No loyalty or labor data, and nothing derived from either,
regardless of aggregation. No row-level POS transactions (treated as
Restricted pending the client's answer — register Q10). Sales totals by
store × week and store attributes are Shareable. Nothing under
`MERIDIAN_DATA_ROOT` is ever read into this wiki.

This wiki is written by an AI tool, so every page in it is Shareable-class by
construction. The client data extract is analyzed in code, never ingested here.

If the client volunteers customer or employee specifics during an interview,
those notes live in restricted storage outside this repository. Only
non-restricted content reaches the register.

See `docs/superpowers/specs/2026-09-20-data-handling-checklist-design.md`.

## Layers

- `raw/` — immutable source documents. Read, never modify. The user adds
  documents to `raw/`; Claude saves external research there only when the
  user asks, as local Markdown, never referenced only by URL.
- `wiki/` — this directory. Claude owns it entirely.
- `wiki/CLAUDE.md` — this file. The schema.

## Page types

| Directory | Holds | Frontmatter |
|---|---|---|
| `sources/` | One summary page per ingested raw document | Yes |
| `entities/` | People, stores, competitors, systems | No |
| `concepts/` | Topics: expansion strategy, loyalty program, unit economics | No |
| `analysis/` | Synthesis pages, the register, the interview guide | No |

Filenames are kebab-case and match the subject: `entities/dana-okafor.md`,
`concepts/expansion-strategy.md`.

## Linking

Relative Markdown links only: `[Dana Okafor](../entities/dana-okafor.md)`.
Never `[[wikilink]]` syntax — this wiki is read in VS Code and on GitHub,
neither of which resolves it.

**Every assertion on an entity or concept page links to the source page it came
from.** A claim with no link back to `sources/` is a lint failure. This is what
lets an interview question be traced back through a page to a document.

**The team's inferences are labeled `*Team inference:*`, cite the premise they
rest on, and are never presented as the source's claim.**

## Page formats

### Source page

```markdown
---
source: raw/<filename>.md
ingested: YYYY-MM-DD
type: client-document | research | course-material
---
# <Title> — <author or origin>, <date>

## What it says
## Key claims
## What it changes
## Questions it raises
```

`Key claims` are candidates for the register. `What it changes` lists the wiki
pages updated because of this source.

### Entity and concept pages

No frontmatter. Start at the `#` heading. Sections:

```markdown
# <Subject>

## What we know
## Open questions
## Related pages
```

### The register — `analysis/open-questions.md`

The system of record for every open question. Maintained on every ingest, not
harvested at the end.

Columns: `ID | Question | Owner | Origin | Why it matters | Status`. Rows are grouped under one `##` section per theme, plus a final section for deferred rows.

- **Themes**, always in this order: Scope & success criteria; Data & access;
  The Pasadena assumption; Business & operations; Stakeholders & process.
- **Owners**: `Dana`, `Marcus`, `research`, `team`.
- **Statuses**: `open`, `deferred`, `answered`, `superseded`.
- Questions owned by `research` or `team` default to `deferred` — they stay
  tracked but never crowd the interview guide.
- IDs are `Q<n>`, assigned in order and never reused.
- When a question is answered, set status to `answered` and record the answer
  inline in the row. After the interview the register is the interview record.
- A register row's Origin is the page whose "What we know" motivates the
  question, and that page's "Open questions" section lists the row's ID.

### The interview guide — `analysis/interview-guide.md`

Generated from the register on request; never edited as the system of record.
Includes only rows with owner `Dana` and status `open`. Grouped by theme in the
order above. Each theme opens with a framing sentence and marks up to three
questions as the ones that survive if Dana's time collapses. Every question
carries its register ID and a link to its origin page.

## Operations

### Ingest (batch)

The user drops one or more sources into `raw/` and says go. For each source:

1. Read it.
2. Write `sources/<name>.md` in the source page format.
3. Update every affected entity and concept page; create pages that do not yet
   exist and are justified by the source.
4. Add register rows for questions the source raises; mark rows the source
   answers as `answered`. A register row's Origin is the page whose "What we
   know" motivates the question, and that page's "Open questions" section
   lists the row's ID.
5. Update `index.md`.

Then append **one** log entry for the whole batch, and run a lint pass.

### Query

Read `index.md` first to find relevant pages, then drill into them. Answer with
citations to wiki pages. Good answers are filed back into `analysis/` as new
pages rather than left in chat.

### Lint (automatic after every batch, or on request)

Check for:

- Contradictions between pages.
- Claims superseded by a newer source.
- Orphan pages with no inbound links.
- Concepts referenced repeatedly but lacking their own page.
- Assertions with no source link.
- Register hygiene: duplicate questions, questions the batch just answered,
  rows with no origin link.
- Register rows whose origin page does not list their ID.

Report findings. Do not silently resolve anything requiring judgment.

## index.md and log.md

`index.md` is content-oriented: every page listed by category with a link and a
one-line summary. Updated on every ingest.

`log.md` is chronological and append-only. Every entry starts
`## [YYYY-MM-DD] <operation> | <summary>` so that
`grep "^## \[" wiki/log.md | tail -5` shows recent history. Date entries with
the actual date of the work.

Log entry body template:

```markdown
Sources in: <sources touched, or "none">
Pages touched: <entities/concepts/analysis pages created or edited>
Register: +N open, M answered, K deferred
Notes: <anything worth flagging>
```

Index status-line format:

```
**Status:** N sources ingested · N entities · N concepts · N questions (N open, N deferred)
```
