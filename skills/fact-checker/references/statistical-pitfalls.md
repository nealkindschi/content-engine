# Statistical Pitfalls in Verification

## Table of contents
1. Correlation vs. causation
2. Absolute vs. relative risk
3. Single study syndrome
4. P-values and P-hacking
5. Animal studies and generalization
6. Cherry-picked timescales
7. Misleading denominators and base rates
8. Data integrity checks for large datasets

---

## 1. Correlation vs. causation

**The problem:** A relationship between two variables (they move together) does not mean one causes the other. This is the single most common error in science reporting.

**Red flag phrases:** "Linked to," "associated with," "connected to," "raises the risk of."

**How to check:**
- Is the study observational or experimental? Only randomized controlled trials can demonstrate causation.
- Are confounders acknowledged? Ice cream sales and drowning deaths both rise in summer — the confounder is temperature, not a causal link.
- Was there a control group? Without one, you cannot isolate the effect.

**Example:**
- Claim: "People who drink red wine live longer."
- Reality: Red wine consumption correlates with higher income and better healthcare access. The study may have controlled for some factors, but residual confounding is likely.

---

## 2. Absolute vs. relative risk

**The problem:** Relative risk inflates the perceived importance of small effects. Always convert to absolute terms.

**How to check:**
- Ask: "What is the base rate?" If a risk goes from 1 in 100,000 to 2 in 100,000, it has doubled (100% relative increase) but the absolute increase is 0.001%.
- Report both numbers. The audience needs both to evaluate the claim.

**Example:**
- Claim: "Processed meat increases colon cancer risk by 18%."
- Reality: Lifetime absolute risk of colon cancer is ~5%. An 18% relative increase means absolute risk rises to ~5.9%. The 18% figure is not wrong, but without the base rate it is misleading.

---

## 3. Single study syndrome

**The problem:** One study is not settled science. Science is cumulative, and individual studies can be wrong for many reasons (small sample, flawed design, honest error).

**How to check:**
- Search for systematic reviews or meta-analyses on the same topic. Use PubMed, Google Scholar, or Cochrane.
- Has the study been replicated? If a finding has not been reproduced independently, treat it as provisional.
- What is the sample size? Small studies (n < 100) are more likely to produce false positives.
- Who funded it? Industry-funded studies are more likely to find results favorable to the funder.

**Red flag phrases:** "A new study finds," "Scientists discover," "Groundbreaking research shows." These framings often overstate single-study findings.

---

## 4. P-values and P-hacking

**The problem:** A P-value below 0.05 means there is a less than 5% probability the result occurred by chance — assuming the study design was sound and only one hypothesis was tested. If researchers test 20 hypotheses, one will likely show p < 0.05 purely by chance.

**How to check:**
- Was the hypothesis pre-registered? ClinicalTrials.gov and OSF registries show what researchers planned to test before they saw the data.
- How many outcomes were measured? If dozens, p-hacking is likely.
- Is the effect size reported? A tiny but "significant" effect may have no practical importance.
- What is the confidence interval? Wide intervals suggest high uncertainty.

**P-hacking techniques to watch for:**
- Testing many subgroups and reporting only the significant ones
- Collecting data until significance is reached, then stopping
- Excluding outliers without pre-specified criteria
- Switching primary outcomes after seeing results

---

## 5. Animal studies and generalization

**The problem:** Results in animal models (mice, rats, fruit flies, cell lines) frequently do not translate to humans. The biological gap is enormous.

**Red flag phrases:** "Could lead to a cure," "promising results in mice," "scientists have reversed [condition] in animal models."

**How to check:**
- Is the claim about human effects? If the study was in animals or cells, the claim must include that caveat.
- How many phases of human trials remain? Animal studies precede Phase I human trials by years. Most drug candidates that pass animal testing fail in human trials.

---

## 6. Cherry-picked timescales

**The problem:** Choosing a convenient start date can make any trend look good or bad.

**How to check:**
- Why was this starting point chosen? Is it the most recent peak or trough? Does the claim pick a recession bottom to show "strong growth"?
- Check alternative time windows: 1-year, 5-year, 10-year, and max available.
- For "since [event]" claims: did the trend exist before the event? If so, attributing it to the event is misleading.

**Example:**
- Claim: "The stock market has risen 50% since [President] took office."
- Check: What was the market doing in the months before their term? Was the starting point an unusual low?

---

## 7. Misleading denominators and base rates

**The problem:** Percentages without context, small sample sizes presented as percentages, and shifting denominators create false impressions.

**How to check:**
- Ask for the raw numbers. "200% increase" from 1 to 3 is mathematically true but not meaningful.
- Check the denominator. "50% of respondents" from a survey of 12 people is noise.
- Is the denominator the right one? "Crime rate per capita" vs. "total crimes" can tell different stories depending on population growth.

---

## 8. Data integrity checks for large datasets

When a claim relies on a dataset, treat the data itself as a source to verify:

1. **Metadata review** — Examine how data points are defined, what counts as a "case," what is treated as missing, and how estimates differ from counts.
2. **Boundary testing** — Look at the highest and lowest entries. Do they make sense? Suspicious outliers may indicate data entry errors or measurement flaws.
3. **Random sampling** — Pick 3-5 specific records and trace them back to primary sources. If the sampled records check out, the dataset gains credibility.
4. **Consistency audits** — Do totals add up? Do time series have unexplained gaps or jumps? Are there duplicates?
5. **Temporal coherence** — Do timestamps follow a logical sequence? Are dates in the future or impossibly far in the past?
