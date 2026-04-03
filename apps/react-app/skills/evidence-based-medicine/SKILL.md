---
name: evidence-based-medicine
description: Search PubMed for evidence-based medicine to evaluate whether an intervention is effective vs a comparator for a specific indication in a defined patient population, reporting relevant clinical outcomes. Use when the user asks about clinical evidence for a treatment, drug efficacy, comparative effectiveness, "is [drug] effective for [condition]", "evidence for [intervention] vs [comparator]", "what does the literature say about [treatment]", "PICO search", "clinical outcomes for [therapy]", or any request to find published evidence supporting or refuting a clinical intervention.
---

# Evidence-Based Medicine — PICO Literature Search

Search PubMed for clinical evidence evaluating an **Intervention** against a **Comparator** for a specific **Indication** in a defined **Population**, reporting relevant **Outcomes**. Follows the PICO framework (Population, Intervention, Comparison, Outcome).

## Tool Routing

| Tool | When to Use |
|------|-------------|
| **MCP: PubMed** | All literature searches — systematic reviews, meta-analyses, RCTs, observational studies, clinical guidelines |

## Workflow

### Step 1: Define the PICO Question

Extract or infer the PICO components from the user's request. If any are ambiguous, make reasonable clinical assumptions and state them.

| Component | Description | Example |
|-----------|-------------|---------|
| **P — Population** | Patient demographics, disease stage, comorbidities | Adults with type 2 diabetes and BMI >30 |
| **I — Intervention** | Drug, procedure, therapy being evaluated | Semaglutide 2.4mg SC weekly |
| **C — Comparator** | Standard of care, placebo, or alternative treatment | Liraglutide 3.0mg SC daily |
| **O — Outcomes** | Primary and secondary clinical endpoints of interest | Weight loss %, HbA1c reduction, cardiovascular events, adverse events |

Formulate the structured PICO question:

> **In [Population], is [Intervention] more effective than [Comparator] for [Outcomes]?**

### Step 2: Tiered Literature Search

Search PubMed in order of evidence quality, starting with the highest level:

**Tier 1 — Systematic Reviews & Meta-Analyses:**
```
PubMed: search_articles(query="<intervention> vs <comparator> <indication> systematic review meta-analysis", max_results=5, sort="relevance")
```

**Tier 2 — Randomized Controlled Trials:**
```
PubMed: search_articles(query="<intervention> vs <comparator> <indication> randomized controlled trial", max_results=5, sort="relevance")
```

**Tier 3 — Real-World Evidence & Observational Studies:**
```
PubMed: search_articles(query="<intervention> vs <comparator> <indication> cohort real-world evidence", max_results=3, sort="relevance")
```

**Tier 4 — Clinical Guidelines & Consensus:**
```
PubMed: search_articles(query="<intervention> <indication> clinical practice guideline recommendation", max_results=3, sort="relevance")
```

If the population is specific (e.g. elderly, pediatric, renal impairment):
```
PubMed: search_articles(query="<intervention> <indication> <population qualifier>", max_results=3, sort="relevance")
```

### Step 3: Retrieve Article Details

For the most relevant articles (up to 10-15):
```
PubMed: get_article_metadata(pmids=["<PMID1>", "<PMID2>", ...])
```

Extract: title, authors, journal, year, abstract, study design, sample size, primary endpoints, key results.

### Step 4: Extract Outcomes

From the retrieved literature, extract and organize findings by outcome type:

| Outcome Category | What to Extract |
|-----------------|-----------------|
| **Primary efficacy endpoint** | Effect size, 95% CI, p-value, NNT if calculable |
| **Secondary efficacy endpoints** | Supporting clinical measures |
| **Safety / adverse events** | Incidence of serious AEs, discontinuation rates, common side effects |
| **Patient-reported outcomes** | Quality of life, symptom scores, functional status |
| **Time to effect** | Onset of clinical benefit |
| **Durability** | Sustained response at 6/12/24 months |

**Key metrics to report when available:**
- **NNT** (Number Needed to Treat): 1 / Absolute Risk Reduction
- **Relative Risk (RR)** or **Hazard Ratio (HR)** with 95% CI
- **Absolute Risk Reduction (ARR)**: control rate - treatment rate
- **Mean difference** with 95% CI for continuous outcomes

### Step 5: Grade the Evidence

Assign a strength-of-evidence grade to each outcome:

| Grade | Label | Criteria |
|-------|-------|----------|
| **A** | Strong | Consistent findings from multiple RCTs or high-quality SR/meta-analysis |
| **B** | Moderate | At least one RCT, or consistent results from well-designed observational studies |
| **C** | Limited | Observational studies with limitations, or extrapolated from different populations |
| **D** | Very Limited | Case reports, expert opinion, or mechanistic reasoning only |
| **I** | Insufficient | No evidence found, or conflicting results with no clear direction |

**Modifiers:**
- Downgrade if: high risk of bias, small sample, short follow-up, significant heterogeneity, industry-only funding
- Upgrade if: large effect size (RR >2 or <0.5), dose-response relationship, consistent across populations

### Step 6: Synthesize Report

Present findings in this format:

```markdown
## Evidence-Based Medicine Report

**PICO Question:** In [Population], is [Intervention] more effective than [Comparator] for [Outcomes]?

**Search date:** <today>
**Articles reviewed:** <N>

---

### Summary of Evidence

| Outcome | Intervention | Comparator | Effect Size (95% CI) | p-value | NNT | Evidence Grade |
|---------|-------------|------------|---------------------|---------|-----|----------------|
| [Primary outcome] | [result] | [result] | [HR/RR/MD, CI] | [p] | [NNT] | [A/B/C/D/I] |
| [Secondary outcome] | [result] | [result] | [HR/RR/MD, CI] | [p] | — | [A/B/C/D/I] |
| [Safety endpoint] | [rate] | [rate] | [HR/RR, CI] | [p] | [NNH] | [A/B/C/D/I] |

### Key Findings

1. **[Primary outcome]:** [1-2 sentence summary with numbers]
2. **[Secondary outcome]:** [1-2 sentence summary]
3. **[Safety]:** [1-2 sentence summary]

### Clinical Bottom Line

[2-3 sentences integrating efficacy and safety evidence to answer the PICO question. State whether the evidence supports, is equivocal about, or argues against using the intervention over the comparator.]

### Evidence Gaps

- [Domains where evidence is absent or insufficient]

### Guideline Recommendations

- [Relevant guideline positions, if found]

### References

| # | PMID | Authors | Title | Journal | Year | Study Design |
|---|------|---------|-------|---------|------|-------------|
| 1 | [PMID](https://pubmed.ncbi.nlm.nih.gov/<PMID>/) | [First author et al.] | [Title] | [Journal] | [Year] | [SR/RCT/Cohort/etc.] |
```

## Error Handling

- **Intervention not recognized**: Ask for the generic drug name or a more specific description
- **No comparator specified**: Default to placebo or standard of care for the indication; note the assumption
- **No PubMed results**: Widen search by dropping population qualifiers, then comparator; if still nothing, report Grade I
- **Conflicting evidence**: Present both sides, grade based on preponderance, flag the discrepancy
- **Only surrogate endpoints available**: Report them but note they are surrogates (lower certainty than hard clinical outcomes)
- **PubMed MCP unavailable**: Report that literature search could not be completed

## Example Usage

**User**: "Is semaglutide better than liraglutide for weight loss in obese adults?"

**Agent workflow**:
1. PICO: P=adults with obesity (BMI>=30), I=semaglutide 2.4mg, C=liraglutide 3.0mg, O=% body weight loss, adverse events
2. Tier 1: Search for SRs/MAs comparing semaglutide vs liraglutide for weight
3. Tier 2: Search for head-to-head RCTs (STEP trials)
4. Tier 3: Real-world comparative effectiveness
5. Extract: weight loss %, proportion achieving >=5%/10% loss, GI AE rates, discontinuation
6. Grade each outcome, synthesize report with clinical bottom line

**User**: "Evidence for pembrolizumab vs chemotherapy in first-line NSCLC with PD-L1 >=50%"

**Agent workflow**:
1. PICO: P=treatment-naive NSCLC with PD-L1 TPS>=50%, I=pembrolizumab, C=platinum-based chemo, O=OS, PFS, ORR, immune-related AEs
2. Search for KEYNOTE-024/042 trials, SRs, guidelines
3. Extract survival data (HR, median OS/PFS), response rates, Grade 3+ AE rates
4. Grade and synthesize with guideline context (NCCN, ESMO)
