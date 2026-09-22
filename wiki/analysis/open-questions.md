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
| Q2 | What decision does the board actually need to make when it meets three weeks from the August brief, and what would change it? | Dana | [engagement-scope](../concepts/engagement-scope.md) | Determines whether the preliminary readout is a status update or a decision input | open |
| Q3 | At week eight, is the deliverable a working dashboard, a written recommendation, or both? | Dana | [engagement-scope](../concepts/engagement-scope.md) | The brief asks for a dashboard but the underlying need is a siting decision; these are different products | open |
| Q4 | Who besides leadership will use the dashboard, and what decisions do they make with it? | Dana | [engagement-scope](../concepts/engagement-scope.md) | Store managers and leadership need different grain and different metrics | open |

## Data & access

| ID | Question | Owner | Origin | Why it matters | Status |
|---|---|---|---|---|---|
| Q5 | Do POS transaction rows carry a loyalty identifier? | Marcus | [pos-system](../entities/pos-system.md) | If yes, row-level POS is loyalty data in substance and is Restricted — this governs the entire data handling posture | open |
| Q6 | Does the spring POS migration break comparability of the three-year history? | Marcus | [pos-system](../entities/pos-system.md) | A discontinuity mid-series would invalidate naive year-over-year comparisons | open |
| Q7 | What format, grain, and delivery cadence will the extract arrive in? | Marcus | [marcus](../entities/marcus.md) | Determines the ingestion work and whether weekly refresh is possible | open |
| Q8 | Are labor hours joinable to store and date, and at what grain? | Marcus | [marcus](../entities/marcus.md) | Labor is the main controllable operating cost; without a clean join the cost half of the analysis is not possible | open |
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
| Q16 | Which of the eight newer stores were taken-over leases, and do they behave differently from the original six? | Dana | [store-network](../entities/store-network.md) | Two populations in one dataset; pooling them would hide the real pattern | open |
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
| Q23 | How does the Pasadena trade area compare demographically to existing store trade areas? | research | [pasadena-site](../entities/pasadena-site.md) | Public data can partly answer the Pasadena question independently of the client's view | deferred |
