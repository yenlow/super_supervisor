# API Reference

## PubMed MCP Tools

### Article Search

| Tool | Purpose | Parameters |
|------|---------|------------|
| `PubMed:search_articles` | Search PubMed | `query`, `max_results`, `sort`, `date_from`, `date_to` |
| `PubMed:get_article_metadata` | Get article details by PMID | `pmids` (array) |

### Search Query Templates

#### Safety
```
"<intervention> safety systematic review meta-analysis"
"<intervention> adverse effects randomized controlled trial"
"<intervention> adverse events post-marketing surveillance cohort"
"<intervention> serious adverse event case report"
"<intervention> safety <population>"
"<intervention> drug interaction"
"<intervention> black box warning FDA"
```

#### Efficacy
```
"<intervention> efficacy <indication> systematic review meta-analysis"
"<intervention> <indication> randomized controlled trial"
"<intervention> vs <comparator> <indication>"
"<intervention> <indication> real-world evidence effectiveness cohort"
"<intervention> <indication> NNT number needed to treat"
"<intervention> <indication> clinical outcome"
```

#### Cost-Effectiveness
```
"<intervention> cost-effectiveness <indication>"
"<intervention> economic evaluation cost-utility QALY"
"<intervention> budget impact analysis <indication>"
"<intervention> vs <comparator> cost-effectiveness"
"<intervention> ICER incremental cost-effectiveness ratio"
"<intervention> value assessment <indication>"
```

---

## Medicare MCP Tools

| Query Type | Example Query |
|------------|--------------|
| Drug spending | "What is the total Medicare Part D spending for <drug>?" |
| Formulary lookup | "What formulary tier is <drug> on under Medicare Part D?" |
| Step therapy | "Are there step therapy requirements for <drug>?" |
| PA requirements | "Is prior authorization required for <drug> under Part D?" |
| Class comparison | "Top drugs by Part D spending in <therapeutic class>" |
| Coverage policy | "CMS national coverage determinations for <intervention>" |

---

## Genie Queries (Internal Claims Data)

| Query Type | Example Query |
|------------|--------------|
| Utilization | "Average billed and paid for claims involving <drug> over last 12 months" |
| PA rates | "Prior authorization approval rate for <drug> vs therapeutic class average" |
| Spend trends | "Total spend on <drug> across all members, trend over last 4 quarters" |
| Denial patterns | "Top denial reason codes for claims involving <drug>" |

---

## Evidence Grading Reference

### Strength-of-Evidence Grades

| Grade | Label | Criteria |
|-------|-------|----------|
| **A** | Strong | Multiple well-designed RCTs or high-quality SR/MA with low heterogeneity |
| **B** | Moderate | ≥1 well-designed RCT or consistent multiple observational studies |
| **C** | Limited | Observational with limitations, or extrapolated across populations |
| **D** | Very Limited | Case reports, expert opinion, mechanistic/animal data |
| **I** | Insufficient | Absent, conflicting, or too limited to conclude |

### Grading Modifiers

| Modifier | Direction | Criteria |
|----------|-----------|----------|
| Downgrade | −1 level | High risk of bias, N<100, short follow-up, high heterogeneity, industry-only funding |
| Upgrade | +1 level | Large effect (RR >2 or <0.5), dose-response, consistent across diverse populations |

---

## ICER Thresholds (US)

| ICER Range | Value Category | Interpretation |
|------------|----------------|----------------|
| <$50,000 / QALY | High value | Clearly cost-effective by US standards |
| $50,000–$100,000 / QALY | Intermediate-high | Generally considered acceptable |
| $100,000–$150,000 / QALY | Intermediate-low | May be acceptable depending on context |
| >$150,000 / QALY | Low value | Above commonly accepted willingness-to-pay |
| Cost-saving | Dominant | Cheaper and more effective — preferred |

---

## Clinical Outcome Metrics

| Metric | Formula | Interpretation |
|--------|---------|----------------|
| NNT | 1 / ARR | Patients treated per one additional good outcome (lower = better) |
| NNH | 1 / ARI | Patients treated per one additional harm (higher = better) |
| ARR | CER − EER | Absolute risk reduction |
| RRR | (CER − EER) / CER | Relative risk reduction (0-1) |
| OR | (a/b) / (c/d) | Odds ratio (1 = no effect) |
| HR | hazard in treated / hazard in control | Hazard ratio (<1 = benefit) |

Where CER = control event rate, EER = experimental event rate.

---

## URL Formats

- **PubMed Article**: `https://pubmed.ncbi.nlm.nih.gov/{PMID}/`
- **ClinicalTrials.gov**: `https://clinicaltrials.gov/study/{NCT_ID}`
- **FDA Drug Label**: `https://dailymed.nlm.nih.gov/dailymed/`
