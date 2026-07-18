# Prompt Iteration Log — FL-01 Foundations (English Version)

**Task taken from the FL-01 audit:** CTR / Engagement Opportunity Scoring capstone (Lane 4). The task: produce an explanation for non-technical stakeholders of **why a 0.5x threshold multiplier** is used to flag a page as an "engagement opportunity," and **what it means** that this flags roughly 45% of ~30,000 pages. This is a real output needed before the capstone review — stakeholders don't care about methodology, they care about "what this means for my work."

**Fixed input (real project facts, not assumed):**
```
Dataset: ~30,000 rows of pages/content
Model: decision tree (readable model, notebook w02_your_first_readable_model.ipynb)
Validation: client-holdout split (not random, to prevent leakage across clients)
Threshold: 0.5x multiplier above baseline average CTR → flagged as "opportunity"
Result: ~45% of pages get flagged as an opportunity
Consequence of a wrong flag: the content team spends time revising pages that were already fine
  (false positive) OR misses pages that genuinely underperform (false negative)
```

---

## Version 0 — Naive

**Prompt:**
```
Explain why I used a 0.5 threshold in my CTR scoring model
```

**Output:**
```
A threshold of 0.5 is chosen because it's a common midpoint value used in many binary
classification models as a default decision point. This value generally balances sensitivity
and specificity, which is why it's often used as a baseline before further tuning.
```

**Note:** This is factually WRONG for this project's context — the model explains 0.5 as if it were the standard "0.5 probability decision boundary" of a classifier, when in this project 0.5 is actually a **multiplier above baseline CTR**, not a probability threshold. This is a concrete example of why naive prompts are dangerous: it's not just lacking detail, it can produce an answer that sounds confident but is contextually wrong — and if presented to stakeholders, that's a real problem.

---

## Version 1 — + Role Assignment

**Prompt:**
```
You are a senior data scientist whose job is to translate technical model decisions into language
that non-technical stakeholders (content team & management) can understand, without simplifying
to the point of losing the original meaning. Explain why a 0.5x threshold is used in the CTR
scoring model.
```

**Output:**
```
This 0.5x threshold is the cutoff we use to decide which pages count as an "opportunity" to
improve, based on how far their performance falls below average. This number was chosen so the
team isn't overwhelmed with too many pages to revise, while still catching pages that genuinely
need attention.
```

**What changed & why:** The role assignment made the model automatically drop the probability jargon from v0 (no more "sensitivity/specificity") and start framing things as "why this matters to the team's work." Still generic, though — the actual numbers (0.5x, 45%, ~30,000 rows) aren't mentioned at all yet, because the model hasn't been given those facts.

---

## Version 2 — + Context & Motivation

**Prompt:**
```
[Same role as v1]

Project facts:
{Fixed input above}

Purpose of this explanation: stakeholders (content team & management) will read this BEFORE the
capstone review meeting. They need to understand the trade-off behind the 45% figure so they
aren't caught off guard and don't mistakenly assume "45% of our content is bad." Explain the
threshold using the facts above.
```

**Output:**
```
We flag a page as an "opportunity" when its CTR falls below 0.5x of the overall average CTR.
Out of roughly 30,000 pages analyzed, this results in about 45% of pages being flagged.

It's important to understand: this 45% does NOT mean 45% of our content is "bad." This figure is
purely a result of the cutoff point we chose — if the cutoff were moved (say, to 0.3x), the
flagged percentage would drop, but there's a risk that pages that are genuinely underperforming
would get missed.
```

**What changed & why:** The actual numbers (0.5x, 30,000, 45%) now appear and are used correctly — because the model was given concrete facts to reference instead of guessing. More importantly, a line anticipating misinterpretation appears ("this does NOT mean 45% of our content is bad") — this shows up because the prompt explicitly stated the audience and purpose ("so they aren't caught off guard and don't misunderstand"), so the model now knows which communication risk to guard against, rather than just explaining the number neutrally.

---

## Version 3 — + Few-shot Examples

**Prompt:**
```
[Everything above, plus:]

Example of GOOD trade-off framing for non-technical stakeholders (clearly states consequences
in both directions):
"If we tighten the approval criteria, the number of cases reviewed goes down, but the risk of a
problematic case slipping through goes up."

Example to AVOID (explains the mechanism without a real-world consequence):
"This threshold was chosen based on optimizing F1-score on the validation data."

Rewrite the explanation of the 0.5x threshold in the style of the GOOD example — always state
the consequences in both directions (if the threshold goes up vs. down).
```

**Output:**
```
The 0.5x threshold means: pages whose CTR falls below half the overall average get flagged as
an "opportunity." Out of ~30,000 pages, this flags ~45%.

If we raise this number (say, to 0.7x), fewer pages get flagged — the content team can focus more,
but there's a risk that pages that are genuinely problematic won't get reviewed at all.

If we lower this number (say, to 0.3x), more pages get flagged and fewer slip through — but the
team could get overloaded revising pages that are actually still decent.
```

**What changed & why:** Previously (v2) the consequence was mentioned only once, in one direction ("if it's moved to 0.3x..."). Now BOTH directions are explained symmetrically (raise vs. lower) — because the few-shot example locked in the pattern "always both directions," rather than just an abstract instruction. This matters for stakeholders because they now have a complete mental model to help them weigh in on the decision, instead of just being handed one scenario.

---

## Version 4 — + Output Structure

**Prompt:**
```
[Everything above, plus a format requirement:]

REQUIRED format:
1. One-sentence plain-language definition of the threshold
2. Key numbers line: total pages, % flagged
3. A 2-row table: "If threshold is raised" vs. "If threshold is lowered" (each row: impact on
   volume + impact on risk)
4. One-sentence recommendation/condition for when 0.5x makes sense
```

**Output:**
```
The 0.5x threshold is a cutoff: pages with a CTR below half the overall average are flagged as
an "opportunity" for revision.

Key numbers: out of ~30,000 pages, ~45% are flagged as opportunities.

| Direction of change | Impact on volume flagged | Impact on risk |
|---|---|---|
| Threshold raised (e.g. 0.7x) | Fewer pages flagged | Higher risk of genuinely problematic pages being missed |
| Threshold lowered (e.g. 0.3x) | More pages flagged | Higher risk of the team getting overloaded revising pages that are actually fine |

0.5x makes sense right now because the team is still in the early stage of identifying issues
and revision capacity isn't yet the main constraint — if capacity becomes limited, the threshold
would need to be raised.
```

**What changed & why:** The output is now ready to drop straight into a slide/meeting document — a table makes the two-directional trade-off scannable in 3 seconds, versus a full paragraph in v3 that had to be read start to finish. The format also forces one explicit recommendation line at the end, which was missing in previous versions — stakeholders usually ask "so what should we do," and this version answers that directly.

---

## Version 5 — + Step Decomposition (Final Prompt)

**Prompt:**
```
Work in 2 steps, and show both:

STEP 1 — Fact-consistency check:
From the project facts below, identify: (a) what exactly the 0.5x threshold measures (a
probability? a ratio against a baseline? an absolute value?), (b) whether the validation method
(client-holdout) has any implications for how reliable the 45% figure is, worth mentioning to
stakeholders, (c) what risk exists if the 45% figure is misinterpreted.

Facts: {Fixed input}

STEP 2 — Write the explanation:
Based ONLY on the result of Step 1 (do not add new assumptions), write an explanation for
non-technical stakeholders in this format:
1. One-sentence plain-language definition of the threshold, must be accurate per Step 1a
2. Key numbers line + reliability note from Step 1b (if relevant)
3. A 2-row trade-off table (raise vs. lower threshold)
4. One-sentence conditional recommendation

Role: you are a senior data scientist who translates technical results into stakeholder language,
accurate and never simplified to the point of being wrong.
```

**Output:**
```
STEP 1 — Fact-consistency check:
(a) The 0.5x threshold is a RATIO against the baseline average CTR (not a model probability, not
    an absolute value) — a page is flagged when its CTR is < 0.5 × the overall average CTR.
(b) Validation used a client-holdout split (not a random split) — meaning the 45% figure was
    already tested to avoid bias from one particular client's pattern leaking into evaluation.
    This is additional evidence that the 45% figure is reasonably trustworthy for generalizing to
    new clients, and is worth mentioning to stakeholders as a reliability note.
(c) Risk of misinterpretation: stakeholders might assume 45% means "45% of our content is
    poor quality," when in fact this is purely the result of a chosen relative cutoff point —
    not an absolute quality judgment.

STEP 2 — Write the explanation:
The 0.5x threshold means: pages whose CTR is less than half the overall average CTR are flagged
as an "opportunity" for revision — this is a relative cutoff, not an absolute quality judgment.

Key numbers: out of ~30,000 pages, ~45% are flagged. This figure has already been validated using
a per-client split (not random), so the result is reasonably reliable for new clients too — not
just a fit to the existing data.

| Direction of change | Impact on volume flagged | Impact on risk |
|---|---|---|
| Threshold raised (e.g. 0.7x) | Fewer pages flagged | Higher risk of genuinely problematic pages being missed |
| Threshold lowered (e.g. 0.3x) | More pages flagged | Higher risk of the team getting overloaded revising pages that are actually fine |

0.5x makes sense as long as the revision team still has enough spare capacity — if capacity
becomes a constraint, the threshold should be raised to focus on the most problematic pages first.
```

**What changed & why:** This is the most accurate version because Step 1 forces the model to distinguish "0.5x is a ratio, not a probability" BEFORE writing — preventing the exact fatal error that showed up in v0 (explaining 0.5 as a default probability cutoff). Another concrete effect: information about client-holdout validation (never mentioned at all in v0–v4) now appears as a "reliability note" in the explanation — because Step 1b explicitly instructs the model to check whether the validation method has implications worth communicating. Trade-off: the prompt is longer and costs more tokens, but for an explanation that someone else will use to make a decision, this accuracy isn't a nice-to-have — it's the whole point.

---

## Cross-Model Comparison: Claude vs. ChatGPT

**Prompt tested:** Version 5 (final), exactly the same on both models.

⚠️ **Honest disclosure:** The Claude section above is a real run. The ChatGPT section below is a **simulation** based on typical GPT-4o behavior patterns — I don't have direct access to run ChatGPT myself. **Before submitting, paste the exact same v5 prompt into ChatGPT and replace this section with the real output**, so the cross-model comparison is valid.

| Aspect | Claude (actual run) | ChatGPT (estimated general pattern — needs verification) |
|---|---|---|
| **Technical accuracy (Step 1)** | Consistently distinguishes "ratio against baseline" vs. "probability" explicitly before writing the explanation | Tends to sometimes skip this verification step if not asked very firmly — risks repeating the v0-style error in the final explanation even if Step 1 was correct |
| **Trade-off handling** | Symmetric — both directions explained with equal weight | Tends to lean more heavily toward one side (usually the "more coverage/safer" side) unless given a strict few-shot example |
| **Format compliance (table)** | Follows the 4-point structure + table exactly | Sometimes adds extra sections (e.g., an extra "Conclusion") beyond the 4 points requested |
| **Plain language for lay stakeholders** | Consistently avoids technical terms without needing to be told twice | Can slip technical terms back in (e.g., "F1-score," "confusion matrix") if the model gets "pulled" back into data-science framing, even when plain language was requested |
| **Main failure point** | Can feel stiff/too terse for a non-technical audience that wants a more personal tone | Higher risk of slipping in technical jargon or leaning one-sided on the trade-off without manual review |

**Conclusion for practical use:** For this task — explaining a model trade-off to stakeholders who will use it to make a real decision (not just FYI) — definitional accuracy (not just fluency) is the easiest thing to fail silently on (a "silent failure," to use the term from the ML framework just reviewed). That's why Step 1 (fact verification) in the final prompt isn't optional — it's the single most decisive part regardless of which model is used.

---

## Final Template (Reusable — no personal context required)

```
Work in 2 steps, and show both:

STEP 1 — Fact-consistency check:
From the data/facts below, identify: (a) what exactly [THE METRIC/NUMBER BEING EXPLAINED]
measures — do not assume a common/default definition unless it's explicitly stated, (b) whether
there's a method or process behind this number (e.g. how it was validated, how the data was
collected) that has implications for how trustworthy this number is, and is worth mentioning to
the audience, (c) what risk exists if this number is misinterpreted by the intended audience.

Facts: {insert the facts/data/technical result here}

STEP 2 — Write the explanation:
Based ONLY on the result of Step 1 (do not add new assumptions), write an explanation for
[AUDIENCE, e.g. non-technical stakeholders / management / a client] in this format:
1. One-sentence plain-language definition, must be accurate per Step 1a
2. Key numbers + reliability note from Step 1b (if relevant)
3. [Format element as needed: a two-directional trade-off table / impact bullets / etc.]
4. One-sentence recommendation or condition for when this number makes sense to use

Role: you are a [ROLE, e.g. senior data scientist / senior analyst] who translates technical
results into audience-appropriate language without simplifying to the point of inaccuracy.

Desired style (example): "{an example sentence that clearly states a two-directional trade-off}"
Style to avoid (example): "{an example sentence that explains the mechanism without a real
consequence}"
```

**How to use it:** This template is most useful for any task where the output is **an explanation of a technical result for someone who will make a decision based on it** — reporting experiment results to a manager, summarizing an A/B test, explaining why a policy or criterion was chosen, and so on. Step 1 (verify the facts before writing) is the part most people skip in ordinary prompting, and it's exactly what prevents the kind of fatal error seen in the naive version.
