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

## [2026-09-20] lint | first build

Checks run: link resolution, orphan pages, index completeness, manual review
for contradictions and unsourced claims.
Findings: Step 1's link-check script flagged `CLAUDE.md -> ../entities/dana-okafor.md`
as broken; this is a false positive — the match is an illustrative link inside
CLAUDE.md's prose documenting the linking convention, not an actual link from
that file's location, and the target page exists. No fix needed. Three judgment
findings from manual review, reported per the schema rather than resolved:
(1) `index.md`'s status line says "23 open questions" but only 21 of the 23
register rows are `open` (2 are `deferred`) — contradicts the register and the
log, which both say 21 open, 2 deferred. (2) Q7 (POS extract format, grain,
and delivery cadence) has its origin link pointed at `entities/pos-system.md`,
but pos-system.md's "Open questions" section lists only Q5, Q6, Q10 — Q7
appears only on marcus.md. Cross-page Q-ID inconsistency between origin link
and where the question is surfaced. (3) Q8 (are labor hours joinable to store
and date, and at what grain) has its origin link pointed at
`concepts/store-performance-variation.md`, but that page's "Open questions"
section lists only Q15, Q16, Q17, Q23 — Q8 appears only on marcus.md. Same
defect as (2): origin link and where the question is surfaced disagree. A
systematic sweep of all 23 register rows against each origin page's "Open
questions" section (counting ranges like "Q11–Q14" as naming every ID in the
range) found no other mismatches.
Wiki state: 1 source, 6 entities, 5 concepts, 23 register questions (21 open,
2 deferred), 1 draft interview guide.

## [2026-09-20] lint | resolve first-build findings

Sources in: none
Pages touched: `index.md`, `log.md` (this entry), `analysis/open-questions.md`,
`analysis/interview-guide.md`, `sources/client-brief.md`, `CLAUDE.md`,
entities: marcus, dana-okafor, meridian-leadership, pasadena-site, pos-system,
concepts: expansion-strategy, store-performance-variation, engagement-scope.
Register: +0 open, 0 answered, 0 deferred (no rows added or resolved; Q7, Q8,
and Q23 origin links corrected; Q16 reworded).

Notes: resolved the three lint findings from the first-build entry above.
`index.md`'s status line now reads "23 questions (21 open, 2 deferred)",
matching the register and log. Q7 and Q8's origin now points to
`entities/marcus.md`, which lists them in its "Open questions" section; the
CLAUDE.md schema now states the origin/list invariant explicitly and lints for
it. Also folded in the rest of the controller-ruled final-review fix wave:
narrowed the Restricted-data rule in `CLAUDE.md` to match the data handling
checklist; reworded Q16 and its matching claims so they no longer assert all
eight newer stores were taken-over leases; added a sourced bullet to
`marcus.md` naming the datasets in the extract; labeled team-inference bullets
with `*Team inference:*` on expansion-strategy.md, store-performance-variation.md,
engagement-scope.md, and pasadena-site.md; anchored "spring 2026" and "three
weeks" to the August 2026 brief throughout; added Q9/Q10 to dana-okafor.md's
open questions; moved Q23's origin to pasadena-site.md; and linked the register
from `sources/client-brief.md`.
Wiki state: 1 source, 6 entities, 5 concepts, 23 register questions (21 open,
2 deferred), 1 draft interview guide.

## [2026-09-22] ingest | ICSC article on grocery formats, store growth and investment

Sources in: `raw/icsc-grocery-formats-2026-04-24.md` → `sources/icsc-grocery-formats-2026.md`
Pages touched: created `concepts/grocery-sector-trends.md` and
`entities/gelsons.md`; edited `concepts/differentiation.md`,
`concepts/expansion-strategy.md`, `analysis/open-questions.md`, `index.md`.
Register: +1 open (Q24), 0 answered, +1 deferred (Q25)

Notes: this is the first public source. It is a trade-press roundup that
repeats figures from Consumer Edge, JLL, IBISWorld, Tillster, FMI, Forbes, and
The Shelby Report. Several caveats are recorded on the source page: the
article does not say what Trader Joe's "3% growth" measures, the Tillster
figures are self-reported survey data from a vendor to restaurants, and the
$11B figure is deal volume. Q22 is partly informed (Gelson's) but stays
deferred. The interview guide was not regenerated, so it does not yet include
Q24. Regenerate it on request.
Wiki state: 2 sources, 7 entities, 6 concepts, 25 register questions (22 open,
3 deferred), 1 draft interview guide.
Lint: no broken links, no orphan pages (log.md has no inbound links, as before),
every register origin lists its ID, and counts match the index (25 / 22 / 3).

## [2026-09-22] ingest | Shelby Report on McKinsey's State of Grocery North America 2026

Sources in: `raw/shelby-mckinsey-grocery-forces-2026-09-14.md` → `sources/shelby-mckinsey-grocery-2026.md`
Pages touched: edited `concepts/differentiation.md`,
`concepts/grocery-sector-trends.md`, `concepts/loyalty-program.md`,
`concepts/store-performance-variation.md`, `analysis/open-questions.md`,
`index.md`.
Register: +1 open (Q26), 0 answered, +1 deferred (Q27)

Notes: this is the second public source, found by web search and not on the
course list. It is a second-hand account: the Shelby Report summarizing a
McKinsey webinar on the June 2026 report. The McKinsey page timed out and
could not be read directly, so the figures are unverified against the
original. Caveats are recorded on the source page: the 4.3× loyalty figure
compares two groups rather than measuring an effect, and the executive
percentages come from about 40 respondents. Loyalty content added to
`loyalty-program.md` is public industry data only. No pages were created, and
the interview guide was not regenerated, so Q24 and Q26 are not in it yet.
Wiki state: 3 sources, 7 entities, 6 concepts, 27 register questions (23 open,
4 deferred), 1 draft interview guide.
Lint: no broken links, no new orphans (log.md as before), every register
origin lists its ID, and counts match the index (27 / 23 / 4). No
contradictions with the ICSC source: its Tillster figure and McKinsey's
prepared-food figures point the same way but measure different things.

## [2026-09-22] ingest | Colorado Boulevard on Sprouts expanding to Pasadena and Highland Park

Sources in: `raw/sprouts-pasadena-highland-park-2026-03-11.md` → `sources/sprouts-pasadena-2026.md`
Pages touched: created `entities/sprouts.md`; edited
`entities/pasadena-site.md`, `concepts/expansion-strategy.md`,
`concepts/loyalty-program.md`, `concepts/differentiation.md` (related link),
`analysis/open-questions.md`, `index.md`.
Register: +1 open (Q28), 0 answered, 0 deferred

Notes: this is the third public source, and the first about Pasadena itself.
It is short local news reporting a posted notice, so the Pasadena store is
pending, not confirmed; the article does not say what the application is for
or when the store would open. The Highland Park lease is reported as signed.
Q22 and Q25 are partly informed but stay deferred. The interview guide still
has not been regenerated, so Q24, Q26, and Q28 are not in it.
Wiki state: 4 sources, 8 entities, 6 concepts, 28 register questions (24 open,
4 deferred), 1 draft interview guide.
Lint: no broken links, no new orphans (log.md as before), every register
origin lists its ID, and counts match the index (28 / 24 / 4). No
contradictions with earlier sources.

## [2026-09-22] query | where specialty grocers are opening

Sources in: none (answered from the four ingested sources)
Pages touched: created `analysis/where-specialty-grocers-are-opening.md`;
edited `index.md`.
Register: +0 open, 0 answered, 0 deferred

Notes: filed the answer to "What do our sources say about where specialty
grocers are opening?" with a citation on every claim and three labeled team
inferences. The sources name only a few places (Sprouts in Pasadena and
Highland Park, Gelson's in Santa Ana and Costa Mesa, a Whole Foods-anchored
center in Redlands); most of the evidence is about patterns. Gaps are recorded
on the page: nothing on Ventura County, and no local count of openings (Q22).
Q28 is kept as the question for Dana for Workshop 2.
Wiki state: 4 sources, 8 entities, 6 concepts, 28 register questions (24 open,
4 deferred), 1 draft interview guide, 1 filed answer.

## [2026-09-22] guide | regenerate interview guide with Q24, Q26, Q28

Sources in: none
Pages touched: `analysis/interview-guide.md`
Register: +0 open, 0 answered, 0 deferred

Notes: regenerated from the register's 20 Dana-owned open rows (was 17).
Q28 was added to "The Pasadena assumption" after Q13, and Q24 and Q26 to
"Business & operations" after Q18. The framing sentences for those two themes
were updated to mention the new public evidence. The short-meeting (bold)
questions are unchanged. Q28 is unbolded because "The Pasadena assumption"
already has three bold questions. "Business & operations" has room for one
more, but promoting Q24 or Q26 is left to the team. The
header now records the four sources and the regeneration date.
Wiki state: 4 sources, 8 entities, 6 concepts, 28 register questions (24 open,
4 deferred), 1 draft interview guide, 1 filed answer.
