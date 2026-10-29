# MATACS-TH-2026-02: AI Proposal Triage Design

**Name:** [Ayesha Tayyab Azeemi]
**Reference:** MATACS-TH-2026-02
**Time actually spent:** [1 hr]

## What this is

A design for a tool that helps a facilities team rank contractor proposals. AI reads each proposal and copies out facts with quotes as proof. Plain code checks the rules, calculates scores and ranks. People review uncertain cases and make the final award decision.

## Files

- [WRITEUP.md](WRITEUP.md): answers to the five questions
- [PROMPTS.md](PROMPTS.md): the extraction prompt
- [ranking.json](ranking.json): my expected ranking of proposals A to E
- [diagram/proposal-flow.png](diagram/proposal-flow.png): process flow from upload to final decision

## Result on the sample set

| Proposal | Status | Score |
|---|---|---|
| A | Ranked 1 | 98.75 |
| E | Ranked 2 | 92.38 |
| C | Ranked 3 | 85.49 |
| D | Flagged: no completion time given | not scored |
| B | Ineligible: bidder does not hold the licence | not scored |

Points per criterion (0 to 100, before weights):

| | Experience | Price | Timeline | Warranty | Team | Total |
|---|---|---|---|---|---|---|
| A | 100 | 100 | 100 | 91.67 | 100 | 98.75 |
| E | 80 | 98.14 | 92.31 | 100 | 100 | 92.38 |
| C | 90 | 97.54 | 85.71 | 91.67 | 50 | 85.49 |

The formulas are in WRITEUP.md.

## Assumptions

The brief does not say these, so I chose them:

1. **Licence:** the bidder itself must hold the Ontario electrical licence. A subcontractor "to be confirmed" does not count.
2. **Budget and 16 weeks:** I treat both as pass/fail limits. Over budget or over 16 weeks fails. If the completion time is not stated, the proposal is flagged for a person, not guessed.
3. **Scoring formulas and caps** (10 installs, 25 and 12 warranty years, four team checks) are my own choices. In the real tool they would be admin settings.
4. **Experience:** I use install counts as written. Some proposals say "since 2020" or "since 2021", which may include installs older than five years. A reviewer should check this.
5. **Price and timeline** are scored against the best value among the scored proposals. If that set changes, scores are recalculated.
6. **Proposal C** has an instruction aimed at automated systems. I ignore it, score C on its real facts, and show a warning. A reviewer can still disqualify it.
7. Things outside the criteria (roof assessments, progress reports) are not scored.
8. **Ties** go to the lower price, then to the reviewer.

## How I used AI

I used ChatGPT to draft a first solution. I then used Claude to review it against the brief, recheck the arithmetic, and rebuild it. I edited the result so I can explain it in my own words.

**Where I disagreed with or corrected the earlier AI output:**

- The ChatGPT warranty formula capped workmanship credit at 10 years, so E's 12-year warranty earned no extra points. Its ranking text still listed that warranty as a reason E ranked well. I changed the formula so the 12-year warranty counts, and kept the text consistent.
- It used a $180,000 price floor and "full marks at 10 weeks". The brief gives neither. I replaced them with simple formulas that compare each bid to the best one.
- It used a second AI prompt for review. I removed it because AI adds variation and cost, and code can check the quotes.
- Neither draft checked the budget ceiling at first. I added it as a pass/fail check.
