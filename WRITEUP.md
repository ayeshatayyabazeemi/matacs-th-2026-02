# Write-up

I refer to proposals by letter. Where the brief was silent I made assumptions, and I list them in the README.

## 1. AI vs. rules

I use AI in one step only: reading each proposal and pulling out facts. Examples are who holds the licence, the price, the weeks, the warranty years, and whether a safety plan is mentioned. For each fact the AI gives a short exact quote and a page number. Proposals are free text in different layouts, so a fixed parser would break easily.

Everything else is plain code: file checks, checking the AI output, the pass/fail rules, scoring, ranking, and writing the explanation.

I drew the line there because AI is good at reading but weak at repeatable decisions. It can also be pushed around by the text it reads. Code gives the same answer every time, and I can show a reviewer exactly why. People make the judgement calls.

## 2. Consistency and explainability

If the same proposals were ranked twice, the result would be the same. Scoring is code, so the same facts always give the same scores. The AI step is the only part that could vary. I run it at temperature 0, check it, and save the result. Ranking always uses the saved result. If someone re-runs extraction, the system shows what changed instead of silently replacing it.

Each criterion gets 0 to 100 points. Then the weights are applied:

- **Experience:** qualifying installs ÷ 10, maximum 100.
- **Price:** lowest scored price ÷ this price × 100.
- **Timeline:** shortest weeks ÷ this proposal's weeks × 100.
- **Warranty:** average of (panel years ÷ 25) and (workmanship years ÷ 12), each capped at 1.
- **Team:** 25 points each for a named lead, a described crew, a safety plan and a monitoring plan.

The caps are settings stored with the criteria. Ties go to the lower price, then to the reviewer.

The reviewer sees one table per proposal: each criterion, the fact found, the quote, the page, the points and the weight. The written reason for each rank is built from that table by a template, so it always matches the numbers.

On the sample set, A scores 98.75, E 92.38 and C 85.49. B is ineligible and D is flagged.

## 3. Human in the loop

A person is involved at five points:

1. **Uncertain evidence.** If a fact is unclear or conflicting, or a quote cannot be found in the text, a reviewer checks it and can correct the value.
2. **Flagged proposals.** A reviewer decides what to do, for example ask the bidder to clarify (D).
3. **Results check.** The reviewer reads the table, can send a proposal back for correction, or can override a result. An override needs a written reason.
4. **Warnings.** For example, the hidden instruction in C is shown to the reviewer.
5. **Final award.** A person decides. The tool only ranks.

The audit log records:

- the criteria version, the file fingerprint, the prompt version and the model name
- the raw AI output and the validation results
- the pass/fail outcomes, the scores and the ranking
- every human edit or override, with who, when and why
- the final decision

## 4. Changing criteria

Criteria are versioned. An edit creates a new version and never changes the old one. Every result is stored with the version it was scored under, so old rankings stay correct and can still be explained.

After an edit, the reviewer sees that the ranking is out of date and can re-run it on the new version:

- If only weights or formula settings changed, the system recalculates from the saved facts. No AI call is needed.
- If a new criterion needs facts that were never extracted, the AI step runs again for those fields.
- If a mandatory rule changed, the pass/fail checks run again.

The system shows old and new rankings side by side. It never changes an old result silently.

## 5. Risks and safeguards

- **Wrong extraction.** Every fact needs a quote and a page. Code checks that the quote really appears on that page. Unclear facts go to a person.
- **Made-up facts.** The AI may answer "not stated". Anything not clearly stated is never treated as true. Missing means flagged, not guessed. This is how D's missing timeline is handled.
- **Manipulation.** C contains an instruction asking automated reviewers to rank it first. The prompt treats proposal text as data. The AI never scores or ranks, so nothing it reads can change a score. The instruction is logged and shown as a warning. I still score C on its real facts, and a reviewer can decide to disqualify it.
- **Weak mandatory evidence.** B says its electrical work will go to a licensed subcontractor to be confirmed after award. The brief says the bidder must hold the licence, so B is ineligible. A reviewer can overrule this with a reason.
- **Bias.** Scoring uses only the admin's criteria and one formula for everyone. Reviewers can see every number.
- **Wrong formulas.** The formulas are visible. Admins can change the settings, which creates a new version.
- **Cost.** Each proposal gets one AI call and the result is saved. Re-ranking and weight changes cost nothing. File size and page limits stop very large uploads.
