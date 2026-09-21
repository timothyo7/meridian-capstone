# Meridian Research Wiki Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Stand up an agent-maintained research wiki in `wiki/`, seeded from the client brief, that produces a themed interview guide for the Meridian stakeholder interview.

**Architecture:** Three layers per the LLM Wiki pattern — immutable sources in `raw/`, an agent-owned wiki in `wiki/`, and `wiki/CLAUDE.md` as the schema that governs page formats and the ingest/query/lint workflows. Pages link with relative Markdown links; only `sources/` pages carry YAML frontmatter. A themed question register is the system of record, and the interview guide is generated from it.

**Tech Stack:** Markdown, git, and shell one-liners for verification. No build step, no dependencies, no test framework — this is a documentation artifact, so each task is verified by reading the file and running a stated check command.

**Spec:** `docs/superpowers/specs/2026-09-20-research-wiki-design.md`

## Global Constraints

- **No restricted data, ever.** Nothing under `MERIDIAN_DATA_ROOT`, and no loyalty, labor, or row-level POS content, may enter `wiki/` or `raw/`. The wiki is written by an AI tool and is Shareable-class by construction (spec D7, and `docs/superpowers/specs/2026-09-20-data-handling-checklist-design.md`).
- **`raw/` is immutable.** The agent reads from it and never modifies it.
- **Relative Markdown links only.** `[text](../entities/dana-okafor.md)`. No `[[wikilinks]]` — the user does not run Obsidian (spec D3).
- **Frontmatter on `sources/` pages only.** Entity, concept, and analysis pages start at their `#` heading (spec D3).
- **Every assertion links to its source page.** An entity or concept claim with no link back to `sources/` is a lint failure (spec, Page formats).
- **Register themes, in this order:** Scope & success criteria; Data & access; The Pasadena assumption; Business & operations; Stakeholders & process (spec D5).
- **Register owners:** `Dana`, `Marcus`, `research`, `team`. Questions owned by `research` or `team` default to status `deferred` so they never crowd the interview guide (decided during planning; extends spec D4).
- **Register statuses:** `open`, `deferred`, `answered`, `superseded`.
- **Log entry prefix:** `## [YYYY-MM-DD] <operation> | <summary>` so `grep "^## \[" wiki/log.md` stays parseable.
- **Today's date for all dated content:** `2026-09-20`.

---

### Task 1: Scaffold the wiki and point Claude at it

**Files:**
- Create: `wiki/sources/.gitkeep`, `wiki/entities/.gitkeep`, `wiki/concepts/.gitkeep`, `wiki/analysis/.gitkeep`
- Create: `CLAUDE.md` (repo root)

**Interfaces:**
- Consumes: nothing.
- Produces: the directory layout every later task writes into; the root pointer that makes Claude open `wiki/CLAUDE.md` in future sessions.

- [ ] **Step 1: Create the directory skeleton**

```bash
mkdir -p wiki/sources wiki/entities wiki/concepts wiki/analysis
touch wiki/sources/.gitkeep wiki/entities/.gitkeep wiki/concepts/.gitkeep wiki/analysis/.gitkeep
```

- [ ] **Step 2: Write the root pointer**

Create `CLAUDE.md` at the repo root with exactly this content:

```markdown
# Meridian capstone — repository conventions

## Research wiki

A research wiki lives in `wiki/`. It is maintained by Claude, not by hand.

**Before creating, editing, or reading wiki pages, read `wiki/CLAUDE.md`.**
It defines the page formats, the ingest / query / lint workflows, and the
data-handling rule that governs what may enter the wiki.

Source documents live in `raw/` and are immutable — read them, never edit them.

## Data handling

No client data classified Restricted may enter this repository or any AI tool.
See `docs/superpowers/specs/2026-09-20-data-handling-checklist-design.md`.
```

- [ ] **Step 3: Verify the structure**

Run:

```bash
find wiki -type d | sort && test -f CLAUDE.md && echo "root pointer present"
```

Expected output:

```
wiki
wiki/analysis
wiki/concepts
wiki/entities
wiki/sources
root pointer present
```

- [ ] **Step 4: Commit**

```bash
git add wiki CLAUDE.md
git commit -m "Scaffold research wiki directories and root pointer"
```

**Done looks like:** Four empty wiki subdirectories exist and are tracked by git, and a root `CLAUDE.md` tells any future Claude session that the wiki exists and to read its schema first.

**How you check it:** Run the Step 3 command — you should see the five paths and the confirmation line. Open `CLAUDE.md` and confirm it points at `wiki/CLAUDE.md` rather than restating the schema itself.

---

### Task 2: Write the schema

**Files:**
- Create: `wiki/CLAUDE.md`

**Interfaces:**
- Consumes: the directory layout from Task 1.
- Produces: the page formats and workflow definitions every later task follows. Tasks 3–9 are each an application of a rule written here.

- [ ] **Step 1: Write `wiki/CLAUDE.md`**

Create the file with exactly this content:

````markdown
# Wiki schema

This directory is a research wiki maintained by Claude for the Meridian
Markets engagement. Claude writes every page here; the team reads them.

Its purpose: the team walks into the Dana Okafor stakeholder interview
informed, and `analysis/interview-guide.md` is what they carry into the room.

## Hard rule: what may enter this wiki

**Nothing classified Restricted may ever become a wiki source or appear on a
wiki page.** That means no loyalty program data, no labor schedules, no
row-level POS transactions, and nothing derived from them — regardless of
aggregation. Nothing under `MERIDIAN_DATA_ROOT` is ever read into this wiki.

This wiki is written by an AI tool, so every page in it is Shareable-class by
construction. The client data extract is analyzed in code, never ingested here.

If the client volunteers customer or employee specifics during an interview,
those notes live in restricted storage outside this repository. Only
non-restricted content reaches the register.

See `docs/superpowers/specs/2026-09-20-data-handling-checklist-design.md`.

## Layers

- `raw/` — immutable source documents. Read, never modify. External research is
  saved here as local Markdown, never referenced only by URL.
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

Columns: `ID | Question | Theme | Owner | Origin | Why it matters | Status`

- **Themes**, always in this order: Scope & success criteria; Data & access;
  The Pasadena assumption; Business & operations; Stakeholders & process.
- **Owners**: `Dana`, `Marcus`, `research`, `team`.
- **Statuses**: `open`, `deferred`, `answered`, `superseded`.
- Questions owned by `research` or `team` default to `deferred` — they stay
  tracked but never crowd the interview guide.
- IDs are `Q<n>`, assigned in order and never reused.
- When a question is answered, set status to `answered` and record the answer
  inline in the row. After the interview the register is the interview record.

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
   answers as `answered`.
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

Report findings. Do not silently resolve anything requiring judgment.

## index.md and log.md

`index.md` is content-oriented: every page listed by category with a link and a
one-line summary. Updated on every ingest.

`log.md` is chronological and append-only. Every entry starts
`## [YYYY-MM-DD] <operation> | <summary>` so that
`grep "^## \[" wiki/log.md | tail -5` shows recent history.
````

- [ ] **Step 2: Verify the schema is complete and self-consistent**

Run:

```bash
grep -c "^## " wiki/CLAUDE.md && grep -n "Restricted\|wikilink\|deferred" wiki/CLAUDE.md | head -20
```

Expected: at least 8 `##` sections, and hits showing the Restricted rule, the wikilink prohibition, and the `deferred` default are all present.

- [ ] **Step 3: Commit**

```bash
git add wiki/CLAUDE.md
git commit -m "Add wiki schema with page formats, workflows, and data rule"
```

**Done looks like:** A schema a fresh Claude session can follow without this conversation — it states what may enter the wiki, the four page types and their formats, the linking rule, the register's columns and vocabularies, and the ingest/query/lint workflows.

**How you check it:** Read it start to finish and ask whether someone with no context could file a new source correctly from it alone. Specifically confirm three things are unambiguous: the Restricted-data rule, that only `sources/` pages get frontmatter, and that `research`/`team` questions default to `deferred`.

---

### Task 3: Ingest the client brief as source #1

**Files:**
- Create: `wiki/sources/client-brief.md`
- Read only: `raw/client-brief.md`

**Interfaces:**
- Consumes: the source page format from Task 2.
- Produces: `sources/client-brief.md`, the page every entity and concept claim in Tasks 4 and 5 links back to.

- [ ] **Step 1: Re-read the source**

```bash
cat raw/client-brief.md
```

- [ ] **Step 2: Write the source page**

Create `wiki/sources/client-brief.md`:

```markdown
---
source: raw/client-brief.md
ingested: 2026-09-20
type: client-document
---
# Client brief — Dana Okafor, VP Operations, August 2026

## What it says

Dana Okafor, VP of Operations at Meridian Markets, asks the LMU MSBA workshop
team for a dashboard showing sales performance by store and category, to support
a decision about where to open the chain's next location. She names Pasadena as
the presumed site and asks for data to back the choice before they commit. The
brief was written between flights and is explicitly rough.

## Key claims

Business:
- Specialty grocery chain, fourteen stores across Los Angeles, Orange, and
  Ventura counties.
- Roughly $78M annual revenue, about 620 employees.
- Competes on prepared foods, local sourcing, and a smaller footprint than
  national chains.
- Grew from six stores to fourteen in five years, mostly by taking over leases
  from chains that pulled out of neighborhoods Meridian judged underserved.

Problem framing:
- "Growth has been good, but it hasn't been even." Some stores took off
  immediately; others have been slower.
- Expansion decisions have been made "on instinct and a spreadsheet."
- "We know the Pasadena site is the obvious next step, but we want the data to
  back it up before we commit."
- A loyalty program of ~40,000 members exists and, per Dana, has probably never
  been used analytically.

Success criteria, as stated:
- Increase revenue, reduce operating costs, improve customer experience.

Data available:
- POS transactions, about three years.
- Loyalty membership and purchase history, ~40,000 members.
- Labor scheduling and hours.
- Store attributes: square footage, opening date, lease terms.
- POS system was migrated last spring.
- IT contact for the extract is Marcus; extract follows NDA signature.

Terms and logistics:
- NDA covers everything in the brief.
- Customer records and employee data — including loyalty data, labor schedules,
  and any excerpt of them — must not enter any AI tool. Counsel is firm and this
  is non-negotiable.
- Sales totals by store and week, and store attributes, are cleared for AI tools.
- Timeline about eight weeks, with something preliminary for a board meeting in
  three weeks.
- Dana prefers email, travels Tuesdays and Wednesdays, is slow to reply. Her
  assistant schedules but cannot answer analytics questions.

## What it changes

First source in the wiki. Created:

- Entities: [Dana Okafor](../entities/dana-okafor.md),
  [Marcus](../entities/marcus.md),
  [Meridian leadership and board](../entities/meridian-leadership.md),
  [Store network](../entities/store-network.md),
  [Pasadena site](../entities/pasadena-site.md),
  [POS system](../entities/pos-system.md)
- Concepts: [Expansion strategy](../concepts/expansion-strategy.md),
  [Loyalty program](../concepts/loyalty-program.md),
  [Differentiation](../concepts/differentiation.md),
  [Store performance variation](../concepts/store-performance-variation.md),
  [Engagement scope](../concepts/engagement-scope.md)

## Questions it raises

The brief is most informative in what it does not say: it names a preferred
outcome (Pasadena) before the analysis, states three success criteria without
ranking them, and describes the requested artifact (a dashboard) rather than the
decision it serves. Seeded into the register as Q1–Q23.
```

- [ ] **Step 3: Verify**

Run:

```bash
head -5 wiki/sources/client-brief.md && grep -c "^## " wiki/sources/client-brief.md
```

Expected: frontmatter with `source:`, `ingested: 2026-09-20`, `type: client-document`, and `4` top-level sections.

- [ ] **Step 4: Commit**

```bash
git add wiki/sources/client-brief.md
git commit -m "Ingest client brief as wiki source #1"
```

**Done looks like:** A source page that captures what the brief asserts, separated from what the team infers, with the four required sections and correct frontmatter.

**How you check it:** Read it against `raw/client-brief.md` and confirm every claim under "Key claims" is actually in the brief rather than inferred. The links under "What it changes" will be broken until Tasks 4 and 5 land — that is expected at this point and is fixed by Task 9's check.

---

### Task 4: Create the entity pages

**Files:**
- Create: `wiki/entities/dana-okafor.md`, `marcus.md`, `meridian-leadership.md`, `store-network.md`, `pasadena-site.md`, `pos-system.md`
- Delete: `wiki/entities/.gitkeep`

**Interfaces:**
- Consumes: `sources/client-brief.md` from Task 3; the entity page format from Task 2.
- Produces: the six entity pages that register rows in Task 6 cite as origins.

- [ ] **Step 1: Write `wiki/entities/dana-okafor.md` in full**

This is the worked example — every other entity page follows this exact shape.

```markdown
# Dana Okafor

VP of Operations at Meridian Markets and the engagement's primary client
contact. Author of the [client brief](../sources/client-brief.md).

## What we know

- Holds the VP of Operations role and is the person commissioning the analysis
  ([brief](../sources/client-brief.md)).
- Frames the ask as a dashboard for leadership to decide where to open next
  ([brief](../sources/client-brief.md)).
- States that expansion calls have been made "on instinct and a spreadsheet"
  ([brief](../sources/client-brief.md)).
- Believes the Pasadena site is "the obvious next step" and wants data to
  support it ([brief](../sources/client-brief.md)) — see
  [Expansion strategy](../concepts/expansion-strategy.md).
- Prefers email; travels Tuesdays and Wednesdays; slow to reply and says not to
  read silence as a problem ([brief](../sources/client-brief.md)).
- Her assistant can schedule time but cannot answer analytics questions
  ([brief](../sources/client-brief.md)).
- Explicitly invites questions: "If anything here is unclear, ask"
  ([brief](../sources/client-brief.md)).

## Open questions

Tracked in the [register](../analysis/open-questions.md) as Q1–Q4, Q11–Q21.

## Related pages

- [Marcus](marcus.md) — IT contact for the data extract
- [Meridian leadership and board](meridian-leadership.md)
- [Engagement scope](../concepts/engagement-scope.md)
```

- [ ] **Step 2: Write `wiki/entities/marcus.md`**

Same structure. Required content — every bullet cites
`../sources/client-brief.md`:

- IT contact at Meridian; pulls the data extract for the team.
- The extract is available only once the NDA is signed.
- No stated role, seniority, or availability — all unknown.
- Related: [POS system](pos-system.md), [Engagement scope](../concepts/engagement-scope.md).
- Open questions: Q5–Q8 in the register.

Note on the page, under `Open questions`, that Marcus is the only named route
to the data and the team knows nothing else about him — that gap is itself the
reason Q5–Q8 exist.

- [ ] **Step 3: Write `wiki/entities/meridian-leadership.md`**

Required content, each bullet citing the brief:

- "Leadership" is referenced as the audience for the dashboard and the decider
  on expansion; no individuals are named beyond Dana.
- A board exists and meets in three weeks; wants something preliminary then.
- Leadership "wants a clearer picture before the next round of expansion."
- Related: [Dana Okafor](dana-okafor.md), [Expansion strategy](../concepts/expansion-strategy.md).
- Open questions: Q2, Q19, Q20.

- [ ] **Step 4: Write `wiki/entities/store-network.md`**

Required content, each bullet citing the brief:

- Fourteen stores across Los Angeles, Orange, and Ventura counties.
- Grew from six to fourteen in five years.
- The eight newer stores came mostly from leases taken over from national
  chains that exited neighborhoods Meridian judged underserved.
- ~$78M annual revenue and ~620 employees across the chain.
- Store attributes available in the extract: square footage, opening date,
  lease terms.
- Performance is uneven — some stores "took off immediately," others were
  "slower to find their footing."
- Related: [Store performance variation](../concepts/store-performance-variation.md),
  [Pasadena site](pasadena-site.md).
- Open questions: Q15, Q16, Q17.

- [ ] **Step 5: Write `wiki/entities/pasadena-site.md`**

Required content:

- Named in the brief as "the obvious next step" for the next location
  ([brief](../sources/client-brief.md)).
- No supporting evidence for that judgment appears anywhere in the brief.
- Commitment status unknown — the brief says "before we commit," which implies
  no commitment yet but does not say so.
- Related: [Expansion strategy](../concepts/expansion-strategy.md),
  [Store network](store-network.md).
- Open questions: Q11–Q14.

Add a short `## Why this page matters` section stating plainly that this entity
is the engagement's central assumption and the wiki's job is to keep the
evidence for and against it in one place.

- [ ] **Step 6: Write `wiki/entities/pos-system.md`**

Required content:

- Meridian migrated to a new POS system in spring 2026 and calls it an
  improvement ([brief](../sources/client-brief.md)).
- About three years of POS transaction history exists, spanning the migration.
- The brief does not say whether history is continuous across the migration.
- **Data handling note:** row-level POS transactions are treated as Restricted
  pending the client's answer; only sales totals by store and week are
  Shareable. See the data handling checklist spec.
- Related: [Marcus](marcus.md), [Engagement scope](../concepts/engagement-scope.md).
- Open questions: Q5, Q6, Q10.

- [ ] **Step 7: Remove the placeholder and verify**

```bash
rm wiki/entities/.gitkeep
ls wiki/entities/ && grep -L "sources/client-brief.md" wiki/entities/*.md
```

Expected: six `.md` files listed, and the `grep -L` prints nothing — meaning every entity page cites the source at least once.

- [ ] **Step 8: Commit**

```bash
git add wiki/entities
git commit -m "Add entity pages seeded from the client brief"
```

**Done looks like:** Six entity pages, each in the standard three-section shape, with every factual claim linking to `sources/client-brief.md`, and each page's `Open questions` section naming the register IDs it generated.

**How you check it:** Run the Step 7 `grep -L` — silence means no unsourced page. Then open `pasadena-site.md` and confirm it reads as a page about an *assumption*, not a page asserting Pasadena is a good site. That distinction is the point of the whole wiki.

---

### Task 5: Create the concept pages

**Files:**
- Create: `wiki/concepts/expansion-strategy.md`, `loyalty-program.md`, `differentiation.md`, `store-performance-variation.md`, `engagement-scope.md`
- Delete: `wiki/concepts/.gitkeep`

**Interfaces:**
- Consumes: `sources/client-brief.md` (Task 3), entity pages (Task 4), the concept page format (Task 2).
- Produces: the five concept pages cited as origins by register rows in Task 6.

- [ ] **Step 1: Write `wiki/concepts/expansion-strategy.md` in full**

The worked example for this task.

```markdown
# Expansion strategy

How Meridian has chosen new locations so far, and how it proposes to choose the
next one.

## What we know

- The chain grew from six stores to fourteen in five years
  ([brief](../sources/client-brief.md)).
- Growth came "mostly by taking over leases from chains that pulled out of
  neighborhoods we thought were underserved" ([brief](../sources/client-brief.md)).
  The site-selection criterion was therefore availability plus a judgment about
  under-service, not a modeled trade area.
- Decisions to date have been made "on instinct and a spreadsheet"
  ([brief](../sources/client-brief.md)).
- [Pasadena](../entities/pasadena-site.md) is named as "the obvious next step,"
  with the analysis framed as backing up that choice rather than testing it
  ([brief](../sources/client-brief.md)).
- Results have been uneven across the resulting stores — see
  [Store performance variation](store-performance-variation.md).

## Tension worth naming

The brief asks for data "to back it up before we commit." Read literally, that
is a request for confirmation of a decision already made. The engagement is
more useful if it tests the decision instead. Whether Dana means confirmation
or evaluation is Q12 in the [register](../analysis/open-questions.md), and it
changes what the team should build.

## Open questions

Q11–Q14 in the [register](../analysis/open-questions.md).

## Related pages

- [Pasadena site](../entities/pasadena-site.md)
- [Store network](../entities/store-network.md)
- [Store performance variation](store-performance-variation.md)
```

- [ ] **Step 2: Write `wiki/concepts/loyalty-program.md`**

Required content, each claim citing the brief:

- ~40,000 members, with membership and purchase history available.
- Dana: "I don't think we've ever really used that data."
- Understanding the customer better is named as a want, separate from the
  dashboard ask.
- **Data handling note, stated prominently:** loyalty data is Restricted. It
  never enters this wiki or any AI tool, and anything derived from it is
  Restricted regardless of aggregation. Analysis of it happens in code against
  data outside this repository.
- Open questions: Q18, and Q22 as deferred research.
- Related: [Engagement scope](engagement-scope.md), [Differentiation](differentiation.md).

- [ ] **Step 3: Write `wiki/concepts/differentiation.md`**

Required content:

- Meridian competes on prepared foods, local sourcing, and a smaller footprint
  than national chains ([brief](../sources/client-brief.md)).
- The brief offers no evidence about whether these actually drive performance.
- The category dimension of the requested dashboard is where this gets tested.
- Open questions: Q22 (deferred, research into the competitive set).
- Related: [Expansion strategy](expansion-strategy.md), [Loyalty program](loyalty-program.md).

- [ ] **Step 4: Write `wiki/concepts/store-performance-variation.md`**

Required content:

- "Growth has been good, but it hasn't been even" — some stores took off
  immediately, others were slower ([brief](../sources/client-brief.md)).
- No definition of "performance" is given; revenue, margin, and growth rate are
  all plausible and lead to different rankings.
- The original six stores and the eight taken-over leases are plausibly
  different populations and worth separating in any analysis.
- Confounders available in the data: square footage, opening date, lease terms.
- Open questions: Q15, Q16, Q17, and Q23 as deferred research.
- Related: [Store network](../entities/store-network.md), [Expansion strategy](expansion-strategy.md).

- [ ] **Step 5: Write `wiki/concepts/engagement-scope.md`**

Required content:

- Requested artifact: a dashboard of sales performance by store and by category
  ([brief](../sources/client-brief.md)).
- Stated success: increase revenue, reduce operating costs, improve customer
  experience — three goals, unranked.
- Timeline: about eight weeks, with a preliminary readout for the board in
  three weeks.
- Data access gated on NDA signature, via [Marcus](../entities/marcus.md).
- Note the gap plainly: the brief describes an artifact and three broad
  outcomes, but not the decision criterion that would make the artifact useful.
- Open questions: Q1, Q2, Q3, Q4, Q9, Q20.
- Related: [Dana Okafor](../entities/dana-okafor.md),
  [Meridian leadership and board](../entities/meridian-leadership.md).

- [ ] **Step 6: Remove the placeholder and verify**

```bash
rm wiki/concepts/.gitkeep
ls wiki/concepts/ && grep -L "sources/client-brief.md" wiki/concepts/*.md
```

Expected: five `.md` files, and `grep -L` prints nothing.

- [ ] **Step 7: Commit**

```bash
git add wiki/concepts
git commit -m "Add concept pages seeded from the client brief"
```

**Done looks like:** Five concept pages that synthesize across the brief rather than restating it — each naming what is known, what is assumed, and which register questions it generated.

**How you check it:** Read `expansion-strategy.md`. It should surface the confirmation-versus-evaluation tension explicitly. If a concept page reads like a paraphrase of one brief paragraph, it has not earned its place as a page and should be folded into an entity page instead.

---

### Task 6: Seed the register

**Files:**
- Create: `wiki/analysis/open-questions.md`

**Interfaces:**
- Consumes: entity pages (Task 4) and concept pages (Task 5) as origins; the register format from Task 2.
- Produces: the register — the system of record that Task 8's interview guide is generated from. Question IDs Q1–Q23 are referenced by every page written in Tasks 4 and 5.

- [ ] **Step 1: Write the register**

Create `wiki/analysis/open-questions.md`:

```markdown
# Open questions register

The system of record for everything the team does not yet know. Maintained on
every ingest. The [interview guide](interview-guide.md) is generated from the
rows here with owner `Dana` and status `open`.

**Owners:** `Dana`, `Marcus`, `research`, `team`.
**Statuses:** `open`, `deferred`, `answered`, `superseded`.
Rows owned by `research` or `team` default to `deferred` — tracked, but never
crowding the interview guide.

When a question is answered, set the status and record the answer inline. After
the interview this table is the interview record.

## Scope & success criteria

| ID | Question | Owner | Origin | Why it matters | Status |
|---|---|---|---|---|---|
| Q1 | Revenue, operating cost, and customer experience are named as goals with no ranking. If they conflict, which wins? | Dana | [engagement-scope](../concepts/engagement-scope.md) | Three unranked goals cannot all drive one recommendation; the ranking determines what the analysis optimizes | open |
| Q2 | What decision does the board actually need to make in three weeks, and what would change it? | Dana | [engagement-scope](../concepts/engagement-scope.md) | Determines whether the preliminary readout is a status update or a decision input | open |
| Q3 | At week eight, is the deliverable a working dashboard, a written recommendation, or both? | Dana | [engagement-scope](../concepts/engagement-scope.md) | The brief asks for a dashboard but the underlying need is a siting decision; these are different products | open |
| Q4 | Who besides leadership will use the dashboard, and what decisions do they make with it? | Dana | [engagement-scope](../concepts/engagement-scope.md) | Store managers and leadership need different grain and different metrics | open |

## Data & access

| ID | Question | Owner | Origin | Why it matters | Status |
|---|---|---|---|---|---|
| Q5 | Do POS transaction rows carry a loyalty identifier? | Marcus | [pos-system](../entities/pos-system.md) | If yes, row-level POS is loyalty data in substance and is Restricted — this governs the entire data handling posture | open |
| Q6 | Does the spring POS migration break comparability of the three-year history? | Marcus | [pos-system](../entities/pos-system.md) | A discontinuity mid-series would invalidate naive year-over-year comparisons | open |
| Q7 | What format, grain, and delivery cadence will the extract arrive in? | Marcus | [pos-system](../entities/pos-system.md) | Determines the ingestion work and whether weekly refresh is possible | open |
| Q8 | Are labor hours joinable to store and date, and at what grain? | Marcus | [store-performance-variation](../concepts/store-performance-variation.md) | Labor is the main controllable operating cost; without a clean join the cost half of the analysis is not possible | open |
| Q9 | Has the NDA been signed, and when can the extract be requested? | Dana | [engagement-scope](../concepts/engagement-scope.md) | Everything downstream is gated on this, and three weeks to the board is short | open |
| Q10 | Does Meridian consider row-level POS transactions restricted for AI tool use, or only loyalty and labor data? | Dana | [pos-system](../entities/pos-system.md) | The brief is silent; the team has assumed Restricted, which constrains the workflow | open |

## The Pasadena assumption

| ID | Question | Owner | Origin | Why it matters | Status |
|---|---|---|---|---|---|
| Q11 | Is there a signed LOI or lease on the Pasadena site, or is it still exploratory? | Dana | [pasadena-site](../entities/pasadena-site.md) | Determines whether the team is validating a decision or informing one | open |
| Q12 | What makes Pasadena "obvious" to leadership — what evidence exists today? | Dana | [expansion-strategy](../concepts/expansion-strategy.md) | Surfaces the implicit model leadership is already using, which the analysis must either support or contradict | open |
| Q13 | What would have to be true for leadership to decide against Pasadena? | Dana | [expansion-strategy](../concepts/expansion-strategy.md) | If nothing would, the engagement is confirmation and should be renegotiated | open |
| Q14 | Are there other candidate sites Pasadena should be evaluated against? | Dana | [pasadena-site](../entities/pasadena-site.md) | A single-site question is a go/no-go; a multi-site question is a ranking, and they need different methods | open |

## Business & operations

| ID | Question | Owner | Origin | Why it matters | Status |
|---|---|---|---|---|---|
| Q15 | Which stores does leadership consider successes, and which laggards? | Dana | [store-performance-variation](../concepts/store-performance-variation.md) | Gives a labeled set to test the data against, and exposes where leadership's read and the data disagree | open |
| Q16 | Do the eight taken-over leases differ systematically from the original six? | Dana | [store-network](../entities/store-network.md) | Two populations in one dataset; pooling them would hide the real pattern | open |
| Q17 | What drives operating cost differences between stores — labor, rent, shrink, something else? | Dana | [store-performance-variation](../concepts/store-performance-variation.md) | "Reduce operating costs" is a stated goal with no cost model behind it yet | open |
| Q18 | Has the loyalty program ever been used for anything — promotions, segmentation, site selection? | Dana | [loyalty-program](../concepts/loyalty-program.md) | Determines whether this is greenfield or a repeat of work already tried and abandoned | open |

## Stakeholders & process

| ID | Question | Owner | Origin | Why it matters | Status |
|---|---|---|---|---|---|
| Q19 | Who on leadership decides on expansion, and who are the skeptics? | Dana | [meridian-leadership](../entities/meridian-leadership.md) | A recommendation that ignores the skeptic's objection does not survive the room | open |
| Q20 | Who signs off on the final deliverable, and in what format does the board expect it? | Dana | [meridian-leadership](../entities/meridian-leadership.md) | Avoids building the right analysis in the wrong container | open |
| Q21 | When you are traveling, who can answer analytics questions, and what is the escalation path? | Dana | [dana-okafor](../entities/dana-okafor.md) | Dana is slow to reply by her own account and the timeline is tight; a blocked week is expensive | open |

## Deferred — research and team

| ID | Question | Owner | Origin | Why it matters | Status |
|---|---|---|---|---|---|
| Q22 | Who are the specialty grocery competitors in LA, Orange, and Ventura counties, and where do they operate? | research | [differentiation](../concepts/differentiation.md) | Answerable without Dana; needed before any trade-area claim | deferred |
| Q23 | How does the Pasadena trade area compare demographically to existing store trade areas? | research | [store-performance-variation](../concepts/store-performance-variation.md) | Public data can partly answer the Pasadena question independently of the client's view | deferred |
```

- [ ] **Step 2: Verify the register's shape**

Run:

```bash
grep -c "^| Q" wiki/analysis/open-questions.md
grep -o "| deferred |" wiki/analysis/open-questions.md | wc -l
grep -c "^## " wiki/analysis/open-questions.md
```

Expected: `23` question rows; `2` deferred rows; `6` sections (five themes plus Deferred).

- [ ] **Step 3: Verify no question is owned by research or team while still `open`**

```bash
grep "^| Q" wiki/analysis/open-questions.md | grep "| research \|| team " | grep -v "deferred" || echo "OK: no research/team rows left open"
```

Expected: `OK: no research/team rows left open`

- [ ] **Step 4: Commit**

```bash
git add wiki/analysis/open-questions.md
git commit -m "Seed question register with 23 questions across five themes"
```

**Done looks like:** 23 questions, each with an owner, a link to the page that produced it, and a one-line statement of why it matters — the "why" being what makes a question defensible rather than merely curious.

**How you check it:** Run the Step 2 and Step 3 commands. Then read the "Why it matters" column alone, top to bottom: if any entry could be swapped onto a different question without seeming wrong, that question is too generic and needs sharpening.

---

### Task 7: Write the index and the log

**Files:**
- Create: `wiki/index.md`, `wiki/log.md`

**Interfaces:**
- Consumes: every page created in Tasks 3–6.
- Produces: the catalog a future session reads first when answering a query, and the chronological record every later ingest appends to.

- [ ] **Step 1: Write `wiki/index.md`**

```markdown
# Wiki index

Catalog of every page. Read this first when answering a question, then drill in.
Updated on every ingest.

**Status:** 1 source ingested · 6 entities · 5 concepts · 23 open questions
**Last updated:** 2026-09-20

## Analysis

- [Open questions register](analysis/open-questions.md) — the system of record; 23 questions across five themes
- [Interview guide](analysis/interview-guide.md) — draft, generated from the register for the Dana interview

## Sources

- [Client brief](sources/client-brief.md) — Dana Okafor, August 2026; the engagement's founding document

## Entities

- [Dana Okafor](entities/dana-okafor.md) — VP Operations, client contact, author of the brief
- [Marcus](entities/marcus.md) — IT contact; the only named route to the data extract
- [Meridian leadership and board](entities/meridian-leadership.md) — the decision-makers; board meets in three weeks
- [Store network](entities/store-network.md) — fourteen stores, ~$78M, ~620 employees, three counties
- [Pasadena site](entities/pasadena-site.md) — the presumed next location; the engagement's central assumption
- [POS system](entities/pos-system.md) — migrated spring 2026; three years of history spanning the change

## Concepts

- [Expansion strategy](concepts/expansion-strategy.md) — how sites have been chosen, and the confirmation-vs-evaluation tension
- [Loyalty program](concepts/loyalty-program.md) — 40,000 members, never analyzed; Restricted data
- [Differentiation](concepts/differentiation.md) — prepared foods, local sourcing, smaller footprint
- [Store performance variation](concepts/store-performance-variation.md) — uneven growth and its candidate explanations
- [Engagement scope](concepts/engagement-scope.md) — the ask, the timeline, and what the brief leaves undefined
```

- [ ] **Step 2: Write `wiki/log.md`**

```markdown
# Wiki log

Append-only. Every entry starts `## [YYYY-MM-DD] <operation> | <summary>` so
that `grep "^## \[" wiki/log.md | tail -5` shows recent history.

## [2026-09-20] ingest | 1 source — client brief

Sources in: `raw/client-brief.md`

Pages created:
- `sources/client-brief.md`
- Entities: dana-okafor, marcus, meridian-leadership, store-network,
  pasadena-site, pos-system
- Concepts: expansion-strategy, loyalty-program, differentiation,
  store-performance-variation, engagement-scope
- `analysis/open-questions.md` seeded with Q1–Q23 (21 open, 2 deferred)
- `analysis/interview-guide.md` drafted from a single source

Notes: first build. The wiki is ready to accept external research. The guide is
marked draft and should be regenerated once research sources land.
```

- [ ] **Step 3: Verify**

```bash
grep "^## \[" wiki/log.md
grep -o "](\([a-z]*\)/[a-z-]*\.md)" wiki/index.md | wc -l
```

Expected: one log entry line matching the prefix format, and `14` — one source, six entities, five concepts, the register, and the guide.

- [ ] **Step 4: Commit**

```bash
git add wiki/index.md wiki/log.md
git commit -m "Add wiki index and log with first ingest entry"
```

**Done looks like:** An index where every page has a one-line summary specific enough to decide whether to open it, and a log entry that a future session can read to understand what has been done.

**How you check it:** Click three links in `wiki/index.md` from VS Code — they should resolve. Then run `grep "^## \[" wiki/log.md` and confirm the prefix format matches exactly, since that pattern is what keeps the log parseable as it grows.

---

### Task 8: Generate the draft interview guide

**Files:**
- Create: `wiki/analysis/interview-guide.md`
- Delete: `wiki/analysis/.gitkeep`

**Interfaces:**
- Consumes: the register from Task 6 — rows with owner `Dana` and status `open` only.
- Produces: the engagement deliverable. Regenerated from the register whenever the register changes.

- [ ] **Step 1: Extract the qualifying rows**

```bash
grep "^| Q" wiki/analysis/open-questions.md | grep "| Dana |" | grep "| open |" | wc -l
```

Expected: `17` — Q1–Q4, Q9–Q21 minus the Marcus-owned rows (Q5–Q8). These 17 are the guide's content; Q5–Q8 (Marcus) and Q22–Q23 (research) are excluded.

- [ ] **Step 2: Write the guide**

```markdown
# Interview guide — Dana Okafor

> **Draft.** Generated from the [register](open-questions.md) on 2026-09-20,
> when the wiki held a single source: the client brief. It will be regenerated
> once external research is ingested. Do not treat this as the finished plan.

Includes register rows owned by Dana with status `open` — 17 questions.
Questions for Marcus (Q5–Q8) are held for a separate IT conversation; research
questions (Q22–Q23) are answerable without Dana.

Grouped by theme, not ranked globally — the conversation should flow by topic.
**Bolded questions are the ones that survive if the meeting is cut short.**

## 1. Scope & success criteria

*Opening: the brief describes an artifact — a dashboard — but the team's read
is that the underlying need is a siting decision. This theme establishes which
one the engagement is for.*

- **Q1 — Revenue, operating cost, and customer experience are all named as
  goals. If they pull in different directions, which one wins?**
  ([engagement-scope](../concepts/engagement-scope.md))
- **Q2 — What decision does the board need to make in three weeks, and what
  would change it?** ([engagement-scope](../concepts/engagement-scope.md))
- Q3 — At week eight, do you need a working dashboard, a written
  recommendation, or both? ([engagement-scope](../concepts/engagement-scope.md))
- Q4 — Who besides leadership will use the dashboard, and what do they decide
  with it? ([engagement-scope](../concepts/engagement-scope.md))

## 2. Data & access

*Opening: everything downstream is gated on the extract, and three weeks to the
board is short. These are the questions Marcus cannot answer.*

- **Q9 — Has the NDA been signed, and when can we request the extract from
  Marcus?** ([engagement-scope](../concepts/engagement-scope.md))
- **Q10 — Does Meridian consider row-level POS transactions restricted for AI
  tool use, or only loyalty and labor data? We have assumed restricted.**
  ([pos-system](../entities/pos-system.md))

## 3. The Pasadena assumption

*Opening: handle with care. The brief states Pasadena is the obvious next step
and asks for data to back it up. The goal here is to understand the existing
reasoning, not to challenge it in the room.*

- **Q12 — What makes Pasadena obvious to leadership? What do you already know
  that points there?** ([expansion-strategy](../concepts/expansion-strategy.md))
- **Q11 — Is there a signed LOI or lease on the site, or is it still
  exploratory?** ([pasadena-site](../entities/pasadena-site.md))
- **Q13 — What would have to be true for you to decide against it?**
  ([expansion-strategy](../concepts/expansion-strategy.md))
- Q14 — Are there other candidate sites we should be evaluating Pasadena
  against? ([pasadena-site](../entities/pasadena-site.md))

## 4. Business & operations

*Opening: the brief says growth has been uneven but does not say which stores
or why. Leadership's read is a useful check on whatever the data shows.*

- **Q15 — Which stores do you consider the successes, and which have been
  slower?** ([store-performance-variation](../concepts/store-performance-variation.md))
- **Q16 — Do the eight stores you took over from departing chains behave
  differently from the original six?** ([store-network](../entities/store-network.md))
- Q17 — What drives operating cost differences between stores — labor, rent,
  shrink? ([store-performance-variation](../concepts/store-performance-variation.md))
- Q18 — Has the loyalty data ever been used for anything — promotions,
  segmentation, site selection? ([loyalty-program](../concepts/loyalty-program.md))

## 5. Stakeholders & process

*Opening: short, and worth doing even if time is tight — it determines whether
the next eight weeks are blocked.*

- **Q19 — Who decides on expansion, and who is most skeptical of it?**
  ([meridian-leadership](../entities/meridian-leadership.md))
- **Q21 — When you are traveling, who can answer analytics questions?**
  ([dana-okafor](../entities/dana-okafor.md))
- Q20 — Who signs off on the final deliverable, and what format does the board
  expect? ([meridian-leadership](../entities/meridian-leadership.md))

---

## After the interview

Record answers inline in the [register](open-questions.md), set those rows to
`answered`, and append a log entry. If Dana volunteers customer or employee
specifics, those notes go to restricted storage outside this repository — not
into the wiki.
```

- [ ] **Step 3: Verify every guide question exists in the register**

```bash
rm -f wiki/analysis/.gitkeep
for q in $(grep -o "^- \*\{0,2\}Q[0-9]*" wiki/analysis/interview-guide.md | grep -o "Q[0-9]*"); do
  grep -q "^| $q |" wiki/analysis/open-questions.md || echo "MISSING from register: $q"
done; echo "cross-check complete"
```

Expected: `cross-check complete` with no MISSING lines.

- [ ] **Step 4: Verify no Marcus or research question leaked in**

```bash
grep -E "^- \*{0,2}Q(5|6|7|8|22|23) " wiki/analysis/interview-guide.md || echo "OK: no out-of-scope questions in guide"
```

Expected: `OK: no out-of-scope questions in guide`

- [ ] **Step 5: Commit**

```bash
git add wiki/analysis/interview-guide.md
git commit -m "Generate draft interview guide from the register"
```

**Done looks like:** A guide you could walk into a room with — 17 questions in five themed blocks, each with a framing sentence, each question traceable to its register ID and origin page, with the short-meeting subset marked and the draft status stated at the top.

**How you check it:** Read only the bolded questions in order and ask whether that alone would be a good twenty-minute conversation. Then pick any question and follow its link back to the page and from there to the source — that chain is the thing the whole wiki exists to provide.

---

### Task 9: Lint pass and handoff

**Files:**
- Modify: any page the lint pass finds wanting
- Modify: `wiki/log.md` (append the lint entry)

**Interfaces:**
- Consumes: the whole wiki.
- Produces: a clean first build and a log entry recording the lint.

- [ ] **Step 1: Check every relative link resolves**

```bash
cd wiki && for f in $(find . -name "*.md"); do
  d=$(dirname "$f")
  grep -o "](\([^)]*\.md\))" "$f" | sed 's/](//;s/)//' | while read -r l; do
    [ -f "$d/$l" ] || echo "BROKEN: $f -> $l"
  done
done; cd ..; echo "link check complete"
```

Expected: `link check complete` with no BROKEN lines. Fix any that appear — the most likely cause is a filename that drifted from what `sources/client-brief.md` promised in "What it changes".

- [ ] **Step 2: Check for orphan pages**

```bash
cd wiki && for f in $(find entities concepts -name "*.md"); do
  n=$(basename "$f")
  c=$(grep -rl "$n" --include="*.md" . | grep -v "^./$f$" | wc -l)
  [ "$c" -eq 0 ] && echo "ORPHAN: $f"
done; cd ..; echo "orphan check complete"
```

Expected: no ORPHAN lines. Every entity and concept page should be reachable from `index.md` at minimum.

- [ ] **Step 3: Check every page is in the index**

```bash
cd wiki && for f in $(find sources entities concepts analysis -name "*.md"); do
  grep -q "${f#./}" index.md || echo "NOT IN INDEX: $f"
done; cd ..; echo "index check complete"
```

Expected: no output before `index check complete`.

- [ ] **Step 4: Review by hand for the checks a script cannot do**

Read the wiki and look for:
- Contradictions between pages.
- Assertions on entity or concept pages with no link to `sources/`.
- Concepts mentioned repeatedly across pages but lacking their own page.
- Register rows that duplicate each other in substance.

Report findings rather than silently resolving anything that requires judgment.

- [ ] **Step 5: Append the lint entry to `wiki/log.md`**

```markdown
## [2026-09-20] lint | first build

Checks run: link resolution, orphan pages, index completeness, manual review
for contradictions and unsourced claims.
Findings: <list them, or "none">
Wiki state: 1 source, 6 entities, 5 concepts, 23 register questions (21 open,
2 deferred), 1 draft interview guide.
```

- [ ] **Step 6: Commit**

```bash
git add wiki
git commit -m "Lint first build and log the pass"
```

**Done looks like:** Every relative link resolves, no orphans, every page catalogued in the index, and a log entry recording what was checked and what was found.

**How you check it:** Run the three commands in Steps 1–3 yourself; all three should complete silently except for their "complete" lines. Then open `wiki/log.md` and confirm it now has two entries, both matching the `## [date] operation | summary` prefix.

---

## What this plan does not do

Stated so the boundary is explicit:

- **No external research is gathered.** Task 9 leaves a wiki ready to accept it. Gathering sources is your call (spec, Open questions) — the ingest workflow in `wiki/CLAUDE.md` handles them when they arrive.
- **No search tooling.** `index.md` is sufficient at this scale (spec, Scope).
- **No client data touches anything.** The extract is out of scope by construction (spec D7).
- **The guide is a draft.** It is generated from one source and says so at the top. Regenerate it after the first research batch lands.
