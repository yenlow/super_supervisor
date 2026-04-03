# API Reference

## PubMed MCP Tools

### Article Search

| Tool | Purpose | Parameters |
|------|---------|------------|
| `PubMed:search_articles` | Search PubMed | `query`, `max_results`, `sort`, `date_from`, `date_to` |
| `PubMed:get_article_metadata` | Get article details by PMID | `pmids` (array) |

### PICO Search Query Templates

#### Tier 1 — Systematic Reviews & Meta-Analyses
```
"<intervention> vs <comparator> <indication> systematic review meta-analysis"
"<intervention> <indication> Cochrane review"
```

#### Tier 2 — Randomized Controlled Trials
```
"<intervention> vs <comparator> <indication> randomized controlled trial"
"<intervention> <indication> phase III trial"
```

#### Tier 3 — Real-World Evidence
```
"<intervention> vs <comparator> <indication> cohort real-world evidence"
"<intervention> <indication> effectiveness observational"
```

#### Tier 4 — Clinical Guidelines
```
"<intervention> <indication> clinical practice guideline recommendation"
"<indication> treatment guideline <year>"
```

#### Safety-Focused
```
"<intervention> safety systematic review"
"<intervention> adverse effects randomized controlled trial"
"<intervention> vs <comparator> adverse events"
```

#### Population-Specific
```
"<intervention> <indication> <population qualifier>"
"<intervention> elderly <indication>"
"<intervention> pediatric <indication>"
"<intervention> renal impairment <indication>"
```

---

## Evidence Grading Reference

### Strength-of-Evidence Grades

| Grade | Label | Criteria |
|-------|-------|----------|
| **A** | Strong | Multiple well-designed RCTs or high-quality SR/MA with low heterogeneity |
| **B** | Moderate | >=1 well-designed RCT or consistent multiple observational studies |
| **C** | Limited | Observational with limitations, or extrapolated across populations |
| **D** | Very Limited | Case reports, expert opinion, mechanistic/animal data |
| **I** | Insufficient | Absent, conflicting, or too limited to conclude |

### Grading Modifiers

| Modifier | Direction | Criteria |
|----------|-----------|----------|
| Downgrade | -1 level | High risk of bias, N<100, short follow-up, high heterogeneity, industry-only funding |
| Upgrade | +1 level | Large effect (RR >2 or <0.5), dose-response, consistent across diverse populations |

---

## Clinical Outcome Metrics

| Metric | Formula | Interpretation |
|--------|---------|----------------|
| NNT | 1 / ARR | Patients treated per one additional good outcome (lower = better) |
| NNH | 1 / ARI | Patients treated per one additional harm (higher = better) |
| ARR | CER - EER | Absolute risk reduction |
| RRR | (CER - EER) / CER | Relative risk reduction (0-1) |
| OR | (a/b) / (c/d) | Odds ratio (1 = no effect) |
| HR | hazard in treated / hazard in control | Hazard ratio (<1 = benefit) |

Where CER = control event rate, EER = experimental event rate.

---

## URL Formats

- **PubMed Article**: `https://pubmed.ncbi.nlm.nih.gov/{PMID}/`
- **ClinicalTrials.gov**: `https://clinicaltrials.gov/study/{NCT_ID}`
- **FDA Drug Label**: `https://dailymed.nlm.nih.gov/dailymed/`
