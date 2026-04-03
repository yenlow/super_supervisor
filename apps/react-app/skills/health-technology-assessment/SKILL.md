---
name: health-technology-assessment
description: Conduct a Health Technology Assessment (HTA) evaluating safety, efficacy, and cost-effectiveness of a clinical intervention using PubMed literature, Medicare Part D data, and claims analytics. Use when the user wants to evaluate an intervention comprehensively — safety profile, clinical effectiveness, economic value, or coverage justification. Triggers include "HTA for [intervention]", "health technology assessment", "is [drug] effective", "cost-effectiveness of [therapy]", "is [procedure] worth it", "compare efficacy of [drug A] vs [drug B]", "clinical and economic evidence for [intervention]", "ICER for [drug]", "NNT for [treatment]", "value assessment", or any request to evaluate whether an intervention is safe, effective, and/or cost-effective. Returns a structured HTA report with strength-of-evidence grades across all three pillars.
---

# Health Technology Assessment

Evaluate a clinical intervention across three pillars — **Safety**, **Efficacy**, and **Cost-Effectiveness** — using PubMed biomedical literature, Medicare Part D data, and internal claims analytics. Each finding is graded with a formal strength-of-evidence rating.

## Scope

This skill covers any clinical intervention:
- **Drugs/medications** — including off-label use, combination regimens, biosimilars
- **Procedures** — surgical, diagnostic, and non-surgical interventions
- **Therapies** — behavioral, physical, radiation, gene therapy
- **Medical devices** — implants, diagnostics, durable medical equipment
- **Care models** — disease management programs, clinical pathways

## Tool Routing

| Tool | When to Use |
|------|-------------|
| **MCP: PubMed** | Search for clinical trials, systematic reviews, meta-analyses, safety reports, health economic evaluations, cost-effectiveness analyses, and comparative effectiveness studies |
| **MCP: Medicare** | Look up Medicare Part D drug spending, formulary tier/restrictions, step therapy, CMS coverage determinations, and national average cost per claim |
| **Genie** | Query internal claims data for utilization patterns, billed vs. paid amounts, denial rates by intervention, PA approval rates, and real-world outcome signals |

## Workflow Overview

1. **Clarify the intervention** → Identify drug/procedure, indication, population, comparator
2. **Pillar 1: Safety assessment** → Adverse events, contraindications, black box warnings
3. **Pillar 2: Efficacy assessment** → Clinical endpoints, NNT, effect sizes, comparative effectiveness
4. **Pillar 3: Cost-effectiveness assessment** → ICER, cost per QALY, budget impact, claims economics
5. **Grade all evidence** → Assign strength-of-evidence per finding
6. **Synthesize HTA report** → Integrated output with ratings across all three pillars

## Step 1: Clarify the Intervention

Establish the assessment parameters. Infer reasonable defaults from context but note assumptions.

| Parameter | Why It Matters |
|-----------|---------------|
| **Intervention name** | Exact drug (generic preferred), procedure name, or therapy |
| **Indication** | Efficacy and cost-effectiveness vary by condition |
| **Population** | Age, comorbidities, disease severity affect all three pillars |
| **Comparator** | Standard of care, placebo, or alternative intervention for relative assessment |
| **Dose / duration** | Affects safety profile and total cost of treatment |
| **Time horizon** | Short-term acute benefit vs. long-term chronic management |

---

## Step 2: Pillar 1 — Safety Assessment

### 2a. Search PubMed for Safety Evidence

Execute a tiered search, prioritizing higher-quality evidence:

```
PubMed: search_articles(query="<intervention> safety systematic review meta-analysis", max_results=5, sort="relevance")
```

```
PubMed: search_articles(query="<intervention> adverse effects randomized controlled trial", max_results=5, sort="relevance")
```

```
PubMed: search_articles(query="<intervention> adverse events post-marketing surveillance cohort", max_results=3, sort="relevance")
```

```
PubMed: search_articles(query="<intervention> serious adverse event case report", max_results=3, sort="relevance")
```

If a specific population was identified in Step 1:

```
PubMed: search_articles(query="<intervention> safety <population>", max_results=3, sort="relevance")
```

### 2b. Categorize Safety Signals

From retrieved literature, identify and categorize each distinct safety signal:

| Safety Domain | Examples |
|---------------|----------|
| **Common adverse effects** | Nausea, headache, injection site reactions |
| **Serious adverse effects** | Pancreatitis, cardiovascular events, organ damage |
| **Drug interactions** | CYP450 interactions, QT prolongation |
| **Contraindications** | Pregnancy, organ impairment, specific comorbidities |
| **Dose-dependent toxicity** | Hepatotoxicity thresholds, nephrotoxicity |
| **Long-term / chronic risks** | Carcinogenicity, cumulative organ damage |
| **Black box warnings** | FDA-mandated serious/life-threatening risk warnings |
| **Withdrawal / rebound** | Discontinuation syndromes |

### 2c. Assign Safety Rating

| Rating | Criteria |
|--------|----------|
| **FAVORABLE** | No serious AEs with Grade A/B evidence; common AEs mild and manageable |
| **ACCEPTABLE** | Manageable with monitoring; moderate risks but benefits generally outweigh |
| **CAUTION** | Serious AEs with Grade B+ evidence; requires specialist oversight; black box warnings |
| **UNFAVORABLE** | Serious safety concerns with Grade A/B evidence; risks likely outweigh benefits |

---

## Step 3: Pillar 2 — Efficacy Assessment

### 3a. Search PubMed for Efficacy Evidence

```
PubMed: search_articles(query="<intervention> efficacy <indication> systematic review meta-analysis", max_results=5, sort="relevance")
```

```
PubMed: search_articles(query="<intervention> <indication> randomized controlled trial", max_results=5, sort="relevance")
```

```
PubMed: search_articles(query="<intervention> vs <comparator> <indication>", max_results=5, sort="relevance")
```

For real-world effectiveness (beyond trial settings):

```
PubMed: search_articles(query="<intervention> <indication> real-world evidence effectiveness cohort", max_results=3, sort="relevance")
```

### 3b. Retrieve Article Details

For the most relevant articles across safety and efficacy searches (up to 15-20 total):

```
PubMed: get_article_metadata(pmids=["<PMID1>", "<PMID2>", ...])
```

Extract: title, authors, journal, year, abstract, study type, sample size, primary endpoints, effect sizes.

### 3c. Extract and Categorize Efficacy Endpoints

From the retrieved literature, extract clinical endpoints organized by type:

| Endpoint Type | What to Extract | Examples |
|---------------|-----------------|----------|
| **Primary clinical outcome** | Effect size, CI, p-value, NNT | Mortality reduction, event-free survival, HbA1c change |
| **Secondary outcomes** | Supporting endpoints | Weight loss, blood pressure, hospitalization rate |
| **Surrogate endpoints** | Biomarker changes (note lower certainty) | LDL reduction, tumor shrinkage, viral load |
| **Patient-reported outcomes** | Quality of life, symptom burden | SF-36, pain scores, functional status |
| **Time to effect** | Onset of clinical benefit | Days/weeks to therapeutic response |
| **Durability** | Sustained benefit over time | Relapse rates, durability at 12/24/36 months |

**Key efficacy metrics to calculate when data is available:**

- **NNT (Number Needed to Treat)**: 1 / Absolute Risk Reduction — lower is better
- **Relative Risk Reduction (RRR)**: (control rate - treatment rate) / control rate
- **Absolute Risk Reduction (ARR)**: control event rate - treatment event rate
- **Odds Ratio / Hazard Ratio**: from study results, with 95% CI
- **Effect size** (Cohen's d or standardized mean difference): for continuous outcomes

### 3d. Comparative Effectiveness

If a comparator was specified in Step 1, or if the literature contains head-to-head trials:

| Dimension | Intervention | Comparator | Difference | Evidence Grade |
|-----------|-------------|------------|------------|----------------|
| Primary outcome | [value] | [value] | [delta, CI] | [A/B/C/D/I] |
| Secondary outcome | [value] | [value] | [delta, CI] | [A/B/C/D/I] |
| Time to effect | [value] | [value] | [delta] | [A/B/C/D/I] |
| Durability | [value] | [value] | [delta] | [A/B/C/D/I] |

### 3e. Assign Efficacy Rating

| Rating | Criteria |
|--------|----------|
| **STRONG** | Consistent, clinically meaningful benefit on primary endpoints with Grade A evidence; NNT favorable; clear superiority or non-inferiority to standard of care |
| **MODERATE** | Benefit demonstrated with Grade A/B evidence, but effect sizes are modest, or evidence is strong for surrogate endpoints only |
| **WEAK** | Some evidence of benefit (Grade B/C) but inconsistent, small effect sizes, short follow-up, or efficacy demonstrated only in subpopulations |
| **INSUFFICIENT** | No reliable efficacy evidence, conflicting results, or evidence limited to Grade D/I |

---

## Step 4: Pillar 3 — Cost-Effectiveness Assessment

### 4a. Search PubMed for Health Economic Evidence

```
PubMed: search_articles(query="<intervention> cost-effectiveness <indication>", max_results=5, sort="relevance")
```

```
PubMed: search_articles(query="<intervention> economic evaluation cost-utility QALY", max_results=3, sort="relevance")
```

```
PubMed: search_articles(query="<intervention> budget impact analysis <indication>", max_results=3, sort="relevance")
```

If a comparator was specified:

```
PubMed: search_articles(query="<intervention> vs <comparator> cost-effectiveness", max_results=3, sort="relevance")
```

### 4b. Query Medicare Part D for Drug Economics

```
MCP Medicare: What is the total Medicare Part D spending for <drug_name> and what is the average cost per claim nationally?
```

```
MCP Medicare: What formulary tier is <drug_name> on, and are there step therapy, prior authorization, or quantity limit requirements under Medicare Part D?
```

```
MCP Medicare: What are the top drugs by Medicare Part D spending in the <therapeutic class> category?
```

### 4c. Query Internal Claims Data via Genie

```
Genie: What is the average billed and paid amount for claims involving <drug_name or procedure_code> over the last 12 months?
```

```
Genie: What is the prior authorization approval rate for <drug_name>, and how does it compare to the therapeutic class average?
```

```
Genie: What is the total spend on <drug_name> across all members, and what is the trend over the last 4 quarters?
```

### 4d. Extract and Categorize Economic Findings

| Economic Metric | What to Extract | Interpretation |
|-----------------|-----------------|----------------|
| **ICER** (Incremental Cost-Effectiveness Ratio) | Cost per QALY gained vs. comparator | <$50K/QALY = high value; $50-150K = intermediate; >$150K = low value (US thresholds) |
| **Cost per QALY** | Total cost divided by quality-adjusted life years | Absolute measure of value |
| **NNT-adjusted cost** | Cost of intervention x NNT | Total spend to achieve one additional good outcome |
| **Budget impact** | Estimated annual cost to the plan for the covered population | Sustainability at scale |
| **Drug acquisition cost** | Per-unit or per-course cost | From Medicare Part D or internal claims |
| **Total cost of care** | Drug cost + administration + monitoring + AE management | Full economic picture |
| **Cost offsets** | Avoided hospitalizations, ER visits, procedures | Downstream savings from effective treatment |

### 4e. Assign Cost-Effectiveness Rating

| Rating | Criteria |
|--------|----------|
| **HIGH VALUE** | ICER <$50K/QALY with Grade A/B evidence; or demonstrated cost savings vs. comparator; strong budget impact case |
| **REASONABLE VALUE** | ICER $50-150K/QALY with Grade A/B evidence; or cost-neutral with superior efficacy; manageable budget impact |
| **LOW VALUE** | ICER >$150K/QALY; or similar efficacy to cheaper alternatives; significant budget impact without commensurate benefit |
| **UNCERTAIN** | Insufficient economic evidence (Grade C/D/I); no published CEA; cost data available but no QALY/outcome linkage |

---

## Step 5: Grade All Evidence

Apply the same strength-of-evidence grading framework across all three pillars.

### Evidence Grading Scale

| Grade | Label | Criteria | Typical Sources |
|-------|-------|----------|-----------------|
| **A** | **Strong** | Consistent findings from multiple well-designed RCTs or a high-quality SR/meta-analysis with low heterogeneity | Cochrane reviews, large multi-center RCTs, FDA pivotal trials, NICE/ICER technology assessments |
| **B** | **Moderate** | At least one well-designed RCT, or consistent results from multiple well-designed observational studies | Single RCTs, large prospective cohorts, well-designed economic evaluations alongside RCTs |
| **C** | **Limited** | Observational studies with methodological limitations, extrapolated from different populations, or modeled economic analyses with uncertain inputs | Retrospective cohorts, registries, decision-analytic models, pharmacovigilance databases |
| **D** | **Very Limited** | Case reports, case series, expert opinion, mechanistic reasoning, or animal/in-vitro data; unpublished economic models | Case reports, expert consensus, manufacturer-sponsored models without peer review |
| **I** | **Insufficient** | Evidence absent, conflicting with no clear direction, or too limited to draw conclusions | No studies found, conflicting RCTs, underpowered analyses |

**Grading modifiers:**
- **Downgrade** one level if: high risk of bias, small sample (<100), short follow-up for chronic interventions, significant heterogeneity, or industry-only funding with no independent replication
- **Upgrade** one level if: large effect size (RR >2 or <0.5), dose-response relationship, consistent across diverse populations, or multiple independent confirmatory studies

---

## Step 6: Synthesize HTA Report

### Output Format

```markdown
## Health Technology Assessment: <Intervention Name>

**Intervention:** <Name> (<route, dose if applicable>)
**Indication:** <Condition>
**Comparator:** <Standard of care or named alternative>
**Population:** <Target population>
**Assessment date:** <Today's date>
**Articles reviewed:** <N>

---

### Executive Summary

| Pillar | Rating | Evidence Grade | Key Finding |
|--------|--------|----------------|-------------|
| Safety | [FAVORABLE / ACCEPTABLE / CAUTION / UNFAVORABLE] | [Strongest grade across signals] | [1-sentence summary] |
| Efficacy | [STRONG / MODERATE / WEAK / INSUFFICIENT] | [Grade for primary endpoint] | [1-sentence summary] |
| Cost-Effectiveness | [HIGH VALUE / REASONABLE VALUE / LOW VALUE / UNCERTAIN] | [Grade for economic evidence] | [1-sentence summary] |

**Overall HTA Recommendation:** [1-2 sentences integrating all three pillars]

---

### Pillar 1: Safety

#### Safety Signals

| # | Signal | Severity | Incidence | Evidence Grade | Finding |
|---|--------|----------|-----------|----------------|---------|
| 1 | [Signal] | [Mild / Moderate / Severe / Life-threatening] | [Common >10% / Uncommon 1-10% / Rare <1% / Unknown] | [A/B/C/D/I] | [Summary] |

#### Medicare Part D Safety Context

| Factor | Status | Relevance |
|--------|--------|-----------|
| Black box warning | [Yes/No — details] | [Interpretation] |
| Prior auth required | [Yes/No] | [Interpretation] |
| Step therapy | [Yes/No — details] | [Interpretation] |
| Quantity limits | [Yes/No — details] | [Interpretation] |

**Safety rating: [FAVORABLE / ACCEPTABLE / CAUTION / UNFAVORABLE]**

---

### Pillar 2: Efficacy

#### Clinical Endpoints

| Endpoint | Result | vs. Comparator | NNT | Evidence Grade |
|----------|--------|----------------|-----|----------------|
| [Primary outcome] | [Effect size, 95% CI] | [Relative difference] | [NNT if calculable] | [A/B/C/D/I] |
| [Secondary outcome] | [Effect size, 95% CI] | [Relative difference] | — | [A/B/C/D/I] |

#### Comparative Effectiveness (if comparator assessed)

| Dimension | <Intervention> | <Comparator> | Difference | Grade |
|-----------|---------------|-------------|------------|-------|
| [Primary] | [value] | [value] | [delta, CI] | [A/B/C/D/I] |

#### Real-World Effectiveness

[Summary of observational/real-world evidence if available, noting how it aligns or diverges from trial results]

**Efficacy rating: [STRONG / MODERATE / WEAK / INSUFFICIENT]**

---

### Pillar 3: Cost-Effectiveness

#### Economic Evidence from Literature

| Metric | Value | Comparator | Evidence Grade |
|--------|-------|------------|----------------|
| ICER | [$X / QALY] | [vs. comparator] | [A/B/C/D/I] |
| Cost per QALY | [$X] | — | [A/B/C/D/I] |
| Budget impact | [$X / year for plan] | — | [A/B/C/D/I] |

#### Medicare Part D Economics

| Metric | Value |
|--------|-------|
| National avg cost per claim | [$X] |
| Total Part D spending | [$X] |
| Formulary tier | [Tier X] |
| Therapeutic class rank | [#X of Y in class] |

#### Internal Claims Economics (Molina)

| Metric | Value | Trend |
|--------|-------|-------|
| Avg billed per claim | [$X] | [Up/Down/Stable] |
| Avg paid per claim | [$X] | [Up/Down/Stable] |
| PA approval rate | [X%] | [Up/Down/Stable] |
| Total plan spend (12 mo) | [$X] | [Up/Down/Stable] |

**Cost-effectiveness rating: [HIGH VALUE / REASONABLE VALUE / LOW VALUE / UNCERTAIN]**

---

### Strength of Evidence Summary

| Grade | Safety Findings | Efficacy Findings | Cost-Effectiveness Findings |
|-------|-----------------|-------------------|-----------------------------|
| A — Strong | [list or "—"] | [list or "—"] | [list or "—"] |
| B — Moderate | [list or "—"] | [list or "—"] | [list or "—"] |
| C — Limited | [list or "—"] | [list or "—"] | [list or "—"] |
| D — Very Limited | [list or "—"] | [list or "—"] | [list or "—"] |
| I — Insufficient | [list or "—"] | [list or "—"] | [list or "—"] |

**Key evidence gaps:** [List domains where evidence is absent or insufficient]

---

### Clinical & Policy Implications

- **For appeals:** [How the combined safety/efficacy/cost evidence supports or challenges medical necessity and coverage determinations]
- **For formulary decisions:** [Whether evidence supports current tier placement, PA requirements, or step therapy]
- **For prescribers:** [Key considerations for patient selection, monitoring, and alternative therapies]
- **For members:** [Risk-benefit-cost context in accessible language]

### Recommended Actions

- [ ] [Action item derived from the HTA findings]
- [ ] [Action item derived from the HTA findings]

### References

| # | PMID | Authors | Title | Journal | Year | Pillar |
|---|------|---------|-------|---------|------|--------|
| 1 | [PMID](https://pubmed.ncbi.nlm.nih.gov/<PMID>/) | [First author et al.] | [Title] | [Journal] | [Year] | [Safety/Efficacy/Cost] |
```

---

## Error Handling

- **Intervention not recognized**: Ask the user for the generic drug name, NDC, CPT/HCPCS code, or a more specific description
- **No PubMed results for a pillar**: Report evidence grade I (Insufficient) for that pillar; widen search terms (drop population/indication qualifiers) before concluding
- **No economic studies found**: Use Medicare Part D spend data and internal claims as the primary economic evidence; assign grade C or D for the cost-effectiveness pillar and note the gap
- **Medicare MCP unavailable**: Proceed with PubMed + Genie; note that formulary/regulatory context could not be retrieved
- **Conflicting evidence**: Present both sides, assign grade based on preponderance, flag discrepancy explicitly
- **Only animal/in-vitro data**: Assign grade D, note absence of human data as a major gap
- **Comparator not specified**: Default to standard of care for the indication; if unclear, assess the intervention in absolute terms and note that comparative assessment requires a named comparator

## Example Usage

**User**: "HTA for semaglutide for weight management in elderly patients"

**Agent workflow**:
1. Clarify: semaglutide (Wegovy 2.4mg SC weekly), weight management, 65+ population, comparator = lifestyle modification
2. **Safety**: PubMed searches for AE profiles → GI events (Grade A), pancreatitis (Grade B), thyroid C-cell (Grade C), sarcopenia in elderly (Grade C) → Rating: CAUTION
3. **Efficacy**: STEP trials for weight loss endpoints → 15% body weight reduction (Grade A), cardiovascular benefit SELECT trial (Grade A), limited elderly subgroup data (Grade C) → Rating: STRONG
4. **Cost-effectiveness**: Published CEAs → $100-130K/QALY (Grade B); Medicare Part D: Tier 3, PA required, $1,300/month; Genie: Molina avg paid $980/claim, PA approval rate 62% → Rating: REASONABLE VALUE
5. Synthesize: Strong efficacy with caution-level safety profile and reasonable but borderline value. Evidence supports coverage with PA for appropriate patients; elderly-specific data is a gap warranting monitoring.

**User**: "Compare cost-effectiveness of Humira vs Rinvoq for rheumatoid arthritis"

**Agent workflow**:
1. Clarify: Humira (adalimumab) vs Rinvoq (upadacitinib), RA indication, general adult population
2. Safety for both interventions (parallel searches)
3. Efficacy: head-to-head trials (SELECT-COMPARE), SR/MAs comparing biologics vs JAK inhibitors
4. Cost-effectiveness: Published CEAs for both, Medicare Part D spend comparison, internal claims billed/paid for both
5. Side-by-side HTA comparison table with differential evidence grades per pillar
