---
name: provider-peer-benchmark
description: Benchmark a provider against specialty peers to identify overprescribers and anomalous billing patterns. Use this skill when users ask to compare a provider's prescribing, denials, or billing against peers, find overprescribers, flag outlier providers, investigate provider anomalies, or assess provider risk. Triggers include "benchmark provider", "compare provider to peers", "find overprescribers", "provider outlier analysis", "who is overprescribing", "provider risk score", "peer comparison for PRV-XXXX", or any request to evaluate whether a provider's patterns deviate from specialty norms. Combines the provider_peer_benchmark UC function, Medicare Part D MCP for drug-level spend context, and Genie claims-to-SQL for detailed pattern analysis.
---

# Provider Peer Benchmark — Overprescriber Detection

Benchmark a provider against their specialty peers using UC functions, Medicare Part D data, and claims/appeals SQL analysis to surface overprescribers and anomalous patterns.

## Tool Routing

This skill orchestrates three capabilities:

| Tool | When to Use |
|------|-------------|
| **UC Function: `provider_peer_benchmark`** | Get the provider's stats vs. specialty peer averages (denial rate, claim volume, avg billed, GLP-1 PAs, billed-to-paid ratio) |
| **Genie: `agent-claim-denials-appeals-review`** | Query claims data for detailed breakdowns — top NDCs by volume, time-series trends, denial reason codes, member overlap, billed amounts by CPT |
| **MCP: `mcp-medicare`** | Look up Medicare Part D drug spending, formulary tier, step therapy requirements, and CMS policy to contextualize prescribing patterns |

## Workflow Overview

1. **Identify the provider** → Resolve provider ID and specialty
2. **Run peer benchmark** → Quantify deviations from specialty norms
3. **Drill into prescribing patterns** → Top drugs, volumes, costs via Genie
4. **Cross-reference Medicare Part D** → National spend context and formulary status
5. **Synthesize risk profile** → Flag levels, evidence summary, recommended actions

## Step 1: Identify the Provider

If the user provides a provider ID (e.g., PRV-0042), proceed directly.

If the user provides a name or partial info, use Genie to resolve:

```
Genie: Find provider ID, name, specialty, and NPI for providers matching "<name or keyword>"
```

Confirm the provider identity with the user before proceeding if multiple matches are returned.

## Step 2: Run Provider Peer Benchmark

Call the UC function to get the structured peer comparison:

```
provider_peer_benchmark(provider_id="<PRV-XXXX>")
```

This returns a table comparing the provider against specialty peer averages:

| Metric | Provider Value | Peer Average | Flag |
|--------|---------------|--------------|------|
| Total Claims | N | N | NORMAL / ELEVATED / HIGH |
| Denial Rate | X% | X% | NORMAL / ELEVATED / HIGH |
| Avg Billed Amount | $X,XXX | $X,XXX | NORMAL / ELEVATED / HIGH |
| Billed-to-Paid Ratio | X.XX | X.XX | NORMAL / ELEVATED / HIGH |
| GLP-1 Prior Auths | N | N | NORMAL / ELEVATED / HIGH |

**Interpretation thresholds:**
- **NORMAL** — within 1 standard deviation of specialty mean
- **ELEVATED** — 1–2 standard deviations above mean
- **HIGH** — >2 standard deviations above mean (requires investigation)

If any metric is flagged HIGH or multiple are ELEVATED, proceed to Steps 3–4 for deeper analysis. If all metrics are NORMAL, report the provider as within expected ranges and skip to Step 5.

## Step 3: Drill into Prescribing Patterns via Genie

Run targeted queries against the claims/appeals data to build an evidence profile. Choose queries based on which benchmark metrics were flagged.

### 3a. If Denial Rate is ELEVATED or HIGH

```
Genie: What are the top 10 denial reason codes for provider <PRV-XXXX>, with count and percentage of total denials?
```

```
Genie: Show monthly denial rate trend for provider <PRV-XXXX> over the last 12 months
```

### 3b. If Claim Volume or Avg Billed is ELEVATED or HIGH

```
Genie: What are the top 15 NDC codes billed by provider <PRV-XXXX> ranked by total billed amount, with claim count and avg billed per claim?
```

```
Genie: Compare the top 5 CPT/HCPCS codes billed by provider <PRV-XXXX> against the specialty average billed amount
```

### 3c. If GLP-1 Prior Auths is ELEVATED or HIGH

```
Genie: Show all GLP-1 drug claims for provider <PRV-XXXX> with NDC, drug name, claim count, total billed, total paid, and PA approval rate
```

```
Genie: How many unique members has provider <PRV-XXXX> prescribed GLP-1 medications to, compared to the specialty peer average?
```

### 3d. If Billed-to-Paid Ratio is ELEVATED or HIGH

```
Genie: Show the top 10 claims by billed-to-paid ratio for provider <PRV-XXXX>, including claim ID, CPT code, billed amount, and paid amount
```

### 3e. Cross-cutting anomaly checks (always run if any flag is HIGH)

```
Genie: Are there claims from provider <PRV-XXXX> for inactive members (was_member_active = false)?
```

```
Genie: Are there duplicate or near-duplicate claims from provider <PRV-XXXX> (same member, same date, same service code)?
```

## Step 4: Cross-Reference with Medicare Part D

For each high-volume or high-cost drug identified in Step 3, query Medicare Part D for national context:

```
MCP Medicare: What is the total Medicare Part D spending for <drug_name> and what is the average cost per claim nationally?
```

```
MCP Medicare: What formulary tier is <drug_name> on and are there step therapy or prior authorization requirements under Medicare Part D?
```

This establishes whether the provider's prescribing volume or cost for a given drug is anomalous relative to national Medicare benchmarks, not just specialty peers.

### PubMed evidence (optional, for GLP-1 cases)

If the investigation involves GLP-1 drugs and appeals, search for clinical evidence:

```
MCP PubMed: Search for "<drug_name> prescribing patterns overprescribing" with max_results=5
```

This can support or challenge the clinical justification for high-volume prescribing.

## Step 5: Synthesize Risk Profile

Compile all findings into a structured risk assessment.

### Provider Risk Summary

```markdown
## Provider Risk Assessment: <Provider Name> (<PRV-XXXX>)

**Specialty:** <Specialty>
**NPI:** <NPI if available>
**Assessment Date:** <Today's date>

### Peer Benchmark Summary

| Metric | Provider | Peer Avg | Deviation | Flag |
|--------|----------|----------|-----------|------|
| Total Claims | X | X | +X% | FLAG |
| Denial Rate | X% | X% | +X pp | FLAG |
| Avg Billed | $X,XXX | $X,XXX | +X% | FLAG |
| Billed-to-Paid Ratio | X.XX | X.XX | +X% | FLAG |
| GLP-1 PAs | X | X | +X% | FLAG |

### Key Findings

1. **[Finding category]**: [Specific evidence with data points]
2. **[Finding category]**: [Specific evidence with data points]
3. ...

### Overprescribing Indicators

| Drug / NDC | Provider Volume | Peer Avg Volume | Natl Medicare Avg | Deviation |
|------------|----------------|-----------------|-------------------|-----------|
| <drug> | X claims | X claims | X claims | +X% |

### Risk Level: [LOW / MODERATE / HIGH / CRITICAL]

**Risk justification:**
- [Bullet point summarizing why this risk level was assigned]

### Recommended Actions

- [ ] [Specific action item based on findings]
- [ ] [Specific action item based on findings]
```

### Risk Level Criteria

| Level | Criteria |
|-------|----------|
| **LOW** | All metrics NORMAL; no anomalous patterns detected |
| **MODERATE** | 1–2 metrics ELEVATED; patterns explainable by case mix |
| **HIGH** | Any metric flagged HIGH, or 3+ ELEVATED; overprescribing indicators present |
| **CRITICAL** | Multiple HIGH flags + inactive member claims, duplicates, or billing outside specialty |

## Error Handling

- **Provider not found**: Ask the user to verify the ID or provide the provider name for a Genie lookup
- **Peer benchmark returns no data**: The provider may be new or have too few claims; report insufficient data for comparison
- **Medicare MCP unavailable**: Proceed with internal peer data only; note that national benchmarks could not be retrieved
- **Genie query returns no results**: Adjust the query scope (e.g., expand time range) or note the data gap

## Example Usage

**User**: "Benchmark provider PRV-0042 against their peers"

**Agent workflow**:
1. `provider_peer_benchmark(provider_id="PRV-0042")` → Denial rate HIGH, GLP-1 PAs ELEVATED
2. Genie: Top denial reason codes for PRV-0042 → 60% are "medical necessity not established"
3. Genie: GLP-1 claims for PRV-0042 → 3x peer average volume for semaglutide
4. Genie: Inactive member claims check → 2 claims for inactive members found
5. MCP Medicare: semaglutide Part D spend → Provider's avg cost/claim is 40% above national average
6. Synthesize: Risk Level HIGH — elevated GLP-1 prescribing, high denial rate, inactive member claims

**User**: "Find overprescribers in cardiology"

**Agent workflow**:
1. Genie: List all cardiology providers ranked by total claim volume, showing denial rate and GLP-1 PA count
2. Identify top outliers (e.g., providers >2 std dev above specialty mean on any metric)
3. For each outlier: `provider_peer_benchmark(provider_id="<PRV-XXXX>")`
4. Cross-reference top prescribed drugs against Medicare Part D national averages
5. Synthesize: Ranked list of providers with risk levels and key deviation metrics
