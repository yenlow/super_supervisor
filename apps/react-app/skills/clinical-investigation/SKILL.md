---
name: clinical-investigation
description: Investigate a patient's clinical case end-to-end. Use this skill when users ask to review a patient, investigate abnormal labs, find clinical notes by topic, assess admission patterns, or build a clinical timeline. Triggers include "look up patient", "investigate patient", "abnormal labs for", "clinical notes about", "patient timeline", "what happened to patient X", "review admission", "search notes for sepsis/pneumonia/etc". Chains UC functions (get_latest_admission → get_abnormal_labs → get_lab_type → get_clinical_notes) with Genie SQL for cohort context and PubMed for clinical evidence.
---

# Clinical Investigation — Patient Case Review

Investigate a patient's clinical case by chaining structured lookups, lab analysis, clinical note retrieval, and literature evidence.

## Tool Routing

This skill orchestrates three capabilities:

| Tool | When to Use |
|------|-------------|
| **UC Function: `get_latest_admission`** | Get a patient's most recent admission by SUBJECT_ID — returns HADM_ID, dates, type, insurance, diagnosis |
| **UC Function: `get_abnormal_labs`** | Get all abnormal lab results for an admission by HADM_ID |
| **UC Function: `get_lab_type`** | Resolve a lab ITEMID to its name, fluid, category, and LOINC code |
| **UC Function: `get_clinical_notes`** | Retrieve clinical notes for a specific admission and date |
| **UC Function: `clinical_notes_vector_search`** | Semantic search across all notes for a clinical topic |
| **Genie: `clinical_data`** | SQL queries over admissions, patients, labs, diagnoses for aggregate analysis |
| **MCP: `pubmed`** | Biomedical literature for clinical evidence and treatment guidelines |

## Workflow: Patient-Specific Investigation

### Step 1: Anchor the Timeline

Start with the patient's most recent admission:

```
clinical_tools: get_latest_admission(patient_id=<SUBJECT_ID>)
```

This returns: SUBJECT_ID, HADM_ID, ADMITTIME, DISCHTIME, ADMISSION_TYPE, INSURANCE, DIAGNOSIS

Present findings:
- **Patient:** SUBJECT_ID
- **Admission:** HADM_ID (ADMISSION_TYPE)
- **Period:** ADMITTIME → DISCHTIME
- **Diagnosis:** DIAGNOSIS
- **Insurance:** INSURANCE

### Step 2: Review Abnormal Labs

Using the HADM_ID from Step 1:

```
clinical_tools: get_abnormal_labs(admission_id=<HADM_ID>)
```

This returns all flagged lab results with ITEMIDs, values, units, and timestamps.

### Step 3: Interpret Lab Results

For each unique ITEMID from Step 2, resolve to human-readable names:

```
clinical_tools: get_lab_type(abnormal_lab_item_id=<ITEMID>)
```

Present as a table:
| Lab Name | Category | Value | Unit | LOINC | Time |
|----------|----------|-------|------|-------|------|

Group by category (Hematology, Chemistry, Blood Gas, etc.) for clinical readability.

### Step 4: Review Clinical Notes (Optional)

If the user asks for notes or you need clinical context:

**For a specific date:**
```
clinical_tools: get_clinical_notes(admission_id=<HADM_ID>, chart_date=<DATE>)
```

**For a clinical topic (e.g. "sepsis", "ventilator weaning"):**
```
clinical_tools: clinical_notes_vector_search(query="<topic>")
```

### Step 5: Contextualize with Cohort Data (Optional)

Use Genie for population-level context:

```
Genie: What percentage of emergency admissions have this diagnosis?
Genie: What is the average length of stay for patients with this diagnosis?
Genie: How many patients had the same abnormal lab pattern?
```

### Step 6: Literature Evidence (Optional)

For clinical decision support, search PubMed:

```
MCP pubmed: Search for treatment guidelines for <diagnosis>
MCP pubmed: Evidence for <intervention> in <condition>
```

## Workflow: Topic-Based Note Search

When the user asks about a clinical topic rather than a specific patient:

### Step 1: Semantic Search

```
clinical_tools: clinical_notes_vector_search(query="<clinical topic>")
```

Returns top 5 matching note excerpts with HADM_ID metadata.

### Step 2: Drill Down (Optional)

For any interesting HADM_ID from the results, investigate further:

```
clinical_tools: get_abnormal_labs(admission_id=<HADM_ID>)
```

## Workflow: Cohort Analysis

For aggregate/statistical questions, route directly to Genie:

```
Genie: How many patients were admitted with pneumonia in the last quarter?
Genie: What are the top 10 diagnoses by admission count?
Genie: Average abnormal lab count per admission by diagnosis category
Genie: Mortality rate by admission type
```

## Output Format

### Patient Case Summary
Present investigations as a structured clinical summary:

**Patient Overview**
- Demographics and admission details

**Abnormal Lab Findings**
- Table grouped by category with clinical significance

**Clinical Notes** (if retrieved)
- Key excerpts with dates

**Clinical Context** (if researched)
- Population comparison from Genie
- Literature evidence from PubMed

### Cohort Analysis
Present as markdown tables with clear column headers and totals where appropriate.
