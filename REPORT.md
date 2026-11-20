# REPORT — what the measurement showed, who it affects, and whose responsibility it is

Author: Maksym Aidana, Narxoz University. Course CSS-4007, Lab 06.

## 1. Measurement read-out

I ran the supplied agent on 40 matched pairs for each attribute. The model shortlisted 356 of 400 profiles, and 0 replies could not be read.

| Group (40 pairs each) | Flips | Mean abs. score difference | Floor |
|---|---|---|---|
| Name language | 2 of 40 | 3.12 | null_name: 0 of 40 flips, 2.00 |
| Gender-coded name | 4 of 40 | 1.12 | null_name: 0 of 40 flips, 2.00 |
| University | 1 of 40 | 2.50 | null_university: 1 of 40 flips, 2.12 |

- All 4 gender flips went against the male-coded name. The exact test gave p = 0.125, so I do not call this an effect.
- The intervals of the real attributes overlap with their floors. On flips, no attribute clearly stands above its floor.
- On mean absolute score difference, two attributes are above their floors by the point estimate: name language (3.12 against 2.00) and university (2.50 against 2.12). In both cases the 95% intervals overlap with the floor's interval (name language [1.38, 5.25] against [0.62, 3.62]; university [1.12, 4.12] against [0.88, 3.62]), so I do not call either an effect. Gender-coded name (1.12) is below its floor (2.00).
- University scores: in all 10 pairs with different scores, the flagship university got the higher score. The mean shift was +2.5 (interval +1.1 to +4.1, sign test p = 0.002). The gap is small and it changed only 1 decision.
- With 40 pairs, an exact test needs at least 6 flips to be significant. So my sample cannot see small effects.
- My predictions were wrong. I expected 6 to 10 flips for university and 3 to 6 for gender.

What the floors cover: another name of the same language and gender coding, and another university of the same tier. What they do not cover: other prompt wordings, other models, other irrelevant details. No measured effect is not the same as no effect. Real applicants could behave differently from these synthetic ones. A stronger estimate needs more pairs, several prompt wordings and more than one model.

## 2. Who is affected

- **Wrongly rejected:** an applicant from a non-flagship university, or with a male-coded first name. Measured link: the flagship score gap (+2.5, 10 of 10 pairs) and the 4 of 4 gender flips against the male-coded name. The cost is a lost scholarship place and a record that may follow the person. The gap is small, so only people near the shortlist line are at risk.
- **Wrongly shortlisted:** an applicant who gets the extra points from a flagship university. This person takes a place from someone else, because places are limited.
- **Not named in the brief:** people who do not apply because they expect to be rejected; the staff who read escalated cases (my band would send 64 of 400 baseline profiles, 16.0%, to them; in the harness run it was 34 of 400, 8.5%); the fund; applicants from universities the model has never seen.
- **If my sample missed an effect:** with 40 pairs I could not see anything below about 6 flips. The same groups would pay for it.
- **Over the years:** if the score is used later by another stage, a small gap is repeated and can grow.

After the harness (Part 3): the flips are 0 of 40 in all five groups, but this is zero by construction because name and university are hidden. What is informative: 46 of 400 decisions changed (reject to escalate 22, shortlist to escalate 12, shortlist to reject 7, reject to shortlist 5) and 166 of 400 scores changed (mean absolute change 14.01). This does not prove the model used the removed fields, because removing lines also changes the prompt. 34 of 400 profiles (8.5%) went to a person, 0 were refused, and the cost was 400 model calls in both runs. My own escalation rule did not fire on any profile; all 34 were in the score band.

## 3. Professional responsibility

My responsibilities as the developer:
- Check the system before it runs. I measured it with matched pairs and floors, and I tested the refusal path in Part 4 (the harness refused 5 of 6 test requests; the bare pipeline refused 0).
- Tell the deployer the limits: 40 pairs per attribute, one model, one prompt wording, synthetic data, and the score band was chosen on one configuration.
- Not claim that the harness removes bias. Blinding makes the name and university pairs identical text, so their after-flips are zero by construction. That table does not prove anything about the model.
- Keep the log, so a decision can be audited without running the model again.

What I did: the measurement, the five controls, the log, and the written justification. What is left: more pairs, other prompt wordings, a real appeals process, and reading the legal texts again before real use.

The institution is responsible for the purpose of the system, choosing and training the reviewers, and handling appeals. If the institution wants a control removed because it is slow or costly, I must say what that control prevented in my measurement (for example the university gap, p = 0.002) and what I could not measure.
