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
that file's location, and the target page exists. No fix needed. Two judgment
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
