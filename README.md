# heart-failure-phenotypes-los
Code for the analysis done in the published paper 'Mining Themes in Clinical Notes to Identify Phenotypes and to Predict Length of Stay in Patients admitted with Heart Failure' https://ieeexplore.ieee.org/abstract/document/10224690

# Mining Themes in Clinical Notes — Heart-Failure Phenotypes & Length-of-Stay Prediction

A reproducible pipeline that uses **topic modeling (LDA)** on the diagnostic codes and procedure reports of heart-failure patients to (1) uncover clinical **phenotypes** as latent themes and (2) **predict length of stay (LOS)** for each hospitalization.

---

## Overview

The pipeline runs entirely on **your own dataset** (a REDCap-style EHR export). At a high level:

```
ICD-10-CM codes ──► strip decimals ──► CCSR category text ──► drop "Heart failure"
                                                                      │
procedure report ─► extract IMPRESSION ─► clean (commas→sentences)    │
                    ─► negation (negspaCy) ─► retain entity phrases    │
                    ─► stop-word removal ─► merge per encounter        │
                                                                      ▼
                                  LDA on TF-IDF  ×2  (diagnostic + procedure)
                                  k chosen by peak c_v coherence (not fixed)
                                                                      │
                          topic-distribution features per encounter   │
                                                                      ▼
Admission/Discharge dates ─► length of stay (5 classes) ─► SMOTE ─► classifiers
                                                          (KNN, MLR, SVM, AdaBoost, RF)
```

---

### Files you provide (not included in this repo)

| File | Description |
|------|-------------|
| Your EHR export (`DATA_PATH`) | CSV/Excel in the schema described below. |
| CCSR mapping (`CCSR_PATH`) | AHRQ CCSR file mapping decimal-free ICD-10-CM `Code` → `Category` name (one code may map to several). Download from AHRQ — see below. |

> **AHRQ CCSR mapping:** obtain the "CCSR for ICD-10-CM Diagnoses" tool from
> <https://www.hcup-us.ahrq.gov/toolssoftware/ccsr/dxccsr.jsp> and prepare a two-column
> CSV with headers `Code` (decimal-free ICD-10-CM code) and `Category` (CCSR category name).

---

## Input data format

You provide **two files at runtime** (neither is included in the repo): your **EHR export** (CSV or Excel, one row per procedure record) and the **CCSR mapping** file. Expected EHR columns:

| Column | Meaning |
|--------|---------|
| `record_id` | Patient identifier. |
| `redcap_repeat_instrument` | REDCap repeating instrument name (e.g. `procedures`). |
| `redcap_repeat_instance` | Per-patient running counter of procedure rows (**not** used for grouping). |
| `encounter_id` | **Hospitalization key** — several procedure rows roll up to one `encounter_id`. |
| `Admission Date` | Admission date (any parseable date format). |
| `Discharge Date` | Discharge date. |
| `primary diagnosis` | ICD-10-CM code(s). May contain **multiple codes** separated by `;`. |
| `secondary diagnosis` | ICD-10-CM code(s), `;`-separated. |
| `procedure report` | Free-text report containing an `IMPRESSION:` (or `IMPRESSION :`) section. |

Column names are configurable at the top of the notebook (the `COL_*` variables) if your headers differ.

If your diagnosis columns already hold **text** instead of ICD-10-CM codes, set `USE_CCSR = False` and the CCSR conversion is skipped.

---
