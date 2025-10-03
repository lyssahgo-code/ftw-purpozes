# Data Cleaning & Sanity Checks
  
> **Tables:** assessments, courses, studentAssessment, studentInfo, studentRegistration & studentVle

## Overview
This document summarizes the cleaning and sanity checks performed on the tables listed above (sourced from *Open University Learning Analytics* dataset) as part of our dbt workflow. Goal for now is to ensure data is consistent, free of duplicates and decide how to handle null values if present

---

# Table: assessments

## 1. Overview
- **Total rows before cleaning:** 206
  
  <img width="366" height="249" alt="image" src="https://github.com/user-attachments/assets/f3663912-75ad-49ec-8b3b-ea0f9c04e3e2" />
  
- **Distinct rows (key columns):**

| Column             | Distinct Values | Notes / Examples |
|-------------------|----------------|-----------------|
| id_assessment      | 206            | All IDs unique; too many to list |
| code_module        | 7              | See table below |
| code_presentation  | 4              | See table below |
| assessment_type    | 3              | See table below |

### code_module (7 distinct values)
| code_module |
|------------|
| AAA        |
| BBB        |
| CCC        |
| DDD        |
| EEE        |
| FFF        |
| GGG        |

### code_presentation (4 distinct values)
| code_presentation | Description           |
|------------------|---------------------|
| 2013J            | October 2013 start   |
| 2014J            | October 2014 start   |
| 2013B            | February 2013 start  |
| 2014B            | February 2014 start  |

> `code_presentation` indicates module start period (B = February, J = October)

### assessment_type (3 distinct values)
| assessment_type | Description |
|----------------|-------------|
| TMA            | Tutor Marked Assessment – graded by a tutor |
| CMA            | Computer Marked Assessment – automatically graded |
| Exam           | Final Exam – graded final exam for the module |

  <img width="1035" height="348" alt="image" src="https://github.com/user-attachments/assets/1e87df0e-c39f-47a1-869c-a3a9f268d400" />
 
- **Exact duplicates found:** 0
  - *Exact duplicates* defined as rows where **all fields are identical**  

  <img width="475" height="280" alt="image" src="https://github.com/user-attachments/assets/332d98f5-a34c-4020-9907-17fed3f4dfdf" />

## 2. Null & Anomaly Checks

Checked all columns for missing values, and initial null check did not return any results

  <img width="1224" height="487" alt="image" src="https://github.com/user-attachments/assets/c33aff11-9968-47b0-953b-ca5d428ad2a3" />
  
- **Null counts per column:**

| Column             | Null Count |
|--------------------|------------|
| id_assessment      | 0          |
| code_module        | 0          |
| code_presentation  | 0          |
| assessment_type    | 0          |
| date               | 0          |
| weight             | 0          |

> All key identifiers and fields are present

A further sanity check was performed by reviewing the values in each column. The `date` and `weight` column stood out because there are multiple rows with a value of **0** and **?**. According to the dataset dictionary, date and weight are defined as such:
> `date`: information about the final submission date of the assessment, calculated as the number of days since the start of the module-presentation. The starting date of the presentation has number 0

> `weight`: weight of the assessment in %. Typically, exams are treated separately and have the weight 100%; sum of all other assessments is 100%

<img width="617" height="325" alt="image" src="https://github.com/user-attachments/assets/48410ba7-ba10-45a3-ad84-38e49797e17b" />

As such, **0** value would be treated as valid under date column but **?** values are to be considered as NULL as it is technically unrecorded/missing

Same goes for **0** values under weight column - will be kept as-is

## 3. Cleaning Actions
- [x] Verified there are no exact duplicates (all fields unique)
- [x] Standardized **'?'** values under `date` to NULL
- [x] Retained '0' values under `date` (valid day-0 assessments) and `weight` (non-graded assessments)
- [x] Verified `date` and `weight` are within expected range (ex. for weight, expected 0-100)
- [x] Ensured `weight` remains in **float format** to accommodate modules split into smaller weights that sum to 100
  > Note: Exams are standalone assessments and have weight 100 on their own

  <img width="948" height="627" alt="image" src="https://github.com/user-attachments/assets/f9904600-8a20-4e92-9fd9-bbcbc5846ab0" />


## 4. Sanity Checks Post-Cleaning
- Total rows: 206
- Nulls remaining: 11 (under `date` column) --retained as could be used for later analysis
- Value ranges:
  - `date`: -12 -> 261
  - `weight`: 0 -> 100 

  <img width="1231" height="421" alt="image" src="https://github.com/user-attachments/assets/33894f4a-891c-4ff1-b757-4f870f638817" />

## 5. Notes / Observations
- No exact duplicates found; total row count stayed the same before and after cleaning
- **'?'** values were replaced with `NULL`. Retained for now since they might provide insights later (e.g., incomplete assessments, missing recordings).  
- **0 values treated as valid:**  
  - `date` = 0 → assessment taken on module start  
  - `weight` = 0 → indicates how the assessment contributes to the final grade (*possible non-graded*)  
- Value ranges align with dataset documentation. Slightly negative `date` values may require further analysis
- **Weight handling:**  
  - All Exams count as 100  
  - Other assessments (TMA/CMA) are split into smaller weights but should sum up to 100 per module  
  - We kept `weight` as **float** format since summing as integers previously returned totals like 99 instead of 100 across modules  
- **Module GGG**: both CMA and TMA returned 0; only the Exam has a recorded weight
    
---

# Table: courses

## 1. Overview
- **Total rows before cleaning:** 22

  <img width="475" height="258" alt="image" src="https://github.com/user-attachments/assets/a6915f79-0664-4a07-aa3a-fd41ed2db6db" />

- **Distinct rows (key columns):**

| Column                     | Distinct Values | Notes / Examples |
|----------------------------|----------------|-----------------|
| code_module                | 7              | See table below |
| code_presentation          | 4              | See table below |
| module_presentation_length | 22             | All values unique |

### code_module (7 distinct values)
| code_module |
|------------|
| AAA        |
| BBB        |
| CCC        |
| DDD        |
| EEE        |
| FFF        |
| GGG        |

### code_presentation (4 distinct values)
| code_presentation | Description           |
|------------------|---------------------|
| 2013J            | October 2013 start   |
| 2014J            | October 2014 start   |
| 2013B            | February 2013 start  |
| 2014B            | February 2014 start  |

> Note: `code_presentation` indicates module start period (B = February, J = October)
 
- **Exact duplicates found:** 0
  - *Exact duplicates* defined as rows where **all fields are identical**  

    <img width="585" height="280" alt="image" src="https://github.com/user-attachments/assets/c2d7535b-5237-491b-9c46-210cbb552b92" />

## 2. Null & Anomaly Checks

Checked all columns for missing values, and everything appears to be intact as initial null checks did not return any results. No **0 or '?'** values were found either

  <img width="952" height="361" alt="image" src="https://github.com/user-attachments/assets/e4cbc052-6183-4fae-856f-6b9ac852cc7d" />

| Column                       | Null Count |
|------------------------------|------------|
| code_module                  | 0          |
| code_presentation            | 0          |
| module_presentation_length   | 0          |

> All key identifiers and fields are present

## 3. Cleaning Actions
- [x] Verified there are no exact duplicates (all fields are unique)
- [x] Retained repeating `code_module` values (same module in different terms)  
- [x] Retained repeating `code_presentation` values (different modules offered in same term)
- [x] Verified values under `module_presentation_length` are consistent across rows and in integer format

## 4. Sanity Checks Post-Cleaning
- Total rows: 22
- Nulls remaining: 0
- Values range:
  - `module_presentation_length`: 234 -> 269 

  <img width="387" height="267" alt="image" src="https://github.com/user-attachments/assets/d6d00d7c-51bd-432b-84cc-e5c5765df1e9" />
  <img width="580" height="286" alt="image" src="https://github.com/user-attachments/assets/3b723f42-413a-4675-bf37-d2e63f1b7d8f" />

## 5. Notes / Observations

- Dataset is small so minimal cleaning required. No nulls/exact duplicates detected
- `module_presentation_length` measured in days and fall between 234-269 days, meaning modules usually run for a course of **7-9 months**

---

# Table: studentAssessment

## 1. Overview

- **Total rows before cleaning:** 347,824 😳 

  ![Total Rows](https://github.com/user-attachments/assets/05a359dd-63c1-40c6-84ad-a9d9d0fa5213)

- **Distinct rows (per key columns):**  

| Column         | Distinct Values | Notes / Examples |
|----------------|----------------|-----------------|
| id_assessment  | 188            | All unique IDs; multiple students per assessment |
| id_student     | 23,369         | All unique students |


  ![Distinct Assessments](https://github.com/user-attachments/assets/96d5a7ed-0873-4766-9516-6998cf0443d0)
  ![Distinct Students](https://github.com/user-attachments/assets/32542c7c-aeee-44d9-b3dd-34589a42825c)

- **Exact duplicates found:** 173,912 😬
  - *Exact duplicates* defined as rows where **all fields are identical**  
  - Rows with the same student and assessment but differing submission dates, scores, or other fields are to be considered **valid and retained**

## 2. Null Checks – `studentassessment

Checked all columns for missing values, and everything appears to be intact as initial null checks did not return any results

<img width="1197" height="393" alt="image" src="https://github.com/user-attachments/assets/158bc8c6-5912-4016-9fdc-723bac743e6e" />

| Column         | Null Count |
|----------------|------------|
| id_student     | 0          |
| id_assessment  | 0          |
| date_submitted | 0          |
| is_banked      | 0          |
| score          | 0          |

> All key identifiers and fields are present

A further sanity check was performed by reviewing the values in each column. The `score` column stood out because there are multiple rows with a value of **0**. According to the dataset dictionary, the valid score range is **0 to 100**.
> `score`: "Score in this assessment. The range is from 0 to 100. Scores lower than 40 are interpreted as Fail. The marks are in the range from 0 to 100"

**Note:** In the original CSV dataset, missing or unrecorded scores were represented as `?`, which differs from what queries in DBeaver returned. In the current table, all `score` values are numeric, and there are no nulls. 

<img width="846" height="520" alt="image" src="https://github.com/user-attachments/assets/0e9d6bf8-68bf-4c83-91ee-7ed4d18f3724" />

Possible explanations for a score of 0 or ? include:  
- The student failed the assessment  
- The student *did not attempt* the assessment  
- The student was *not enrolled* in that assessment

As a result, 0 values were retained while ? are to be converted to NULL. May provide useful information for further analysis, particularly when combined with other tables to gain more context about students who scored 0 or ?

## 3. Cleaning Actions
- [x] Removed exact duplicates (all fields identical)
- [x] Standardized **'?'** values to NULL 
- [x] Retained multiple attempts for same `id_student` + `id_assessment` if dates/scores differ  
- [x] Verified `date_submitted` is numeric and within expected range
> MIN: -11, MAX: 608
- [x] Verified no nulls remain
> Rows with 0 or ? values: 502

## 4. Sanity Checks Post-Cleaning – `studentassessment
- **Total rows:** 173,912
- **Nulls remaining:** 
> **Rows with 0 or ? values:** 502

<img width="475" height="283" alt="image" src="https://github.com/user-attachments/assets/20827e1a-0398-4b85-b268-5d7a44edc08b" />


## 5. Notes / Observations

- `date_submitted` is **integer days**, not an actual date  
  > Days measured from start of module presentation; negative values considered valid (submission before start)  
- Score range: 0–100, <40 = **Fail**. Missing/0 scores retained for context  
- Cleaning focused on:  
  - Removing exact duplicates  
  - Verifying nulls  
  - Checking data types and formats  
- No additional normalization or derived columns added at this stage  
  > Semantic derivation (e.g., pass/fail flags, handling missing scores) will be done later in **dim tables** once business questions are finalized

---

# Table: studentInfo

## 1. Overview

- **Total rows before cleaning:** 65,186

  <img width="520" height="246" alt="image" src="https://github.com/user-attachments/assets/825437e9-e1e8-4fb7-99ea-d6d427ba9510" />

- **Distinct rows (per key columns):**  

| Column                 | Distinct Values | Notes / Examples |
|------------------------|----------------|-----------------|
| code_module            | 7              | See Overview / courses table |
| code_presentation      | 4              | See Overview / courses table |
| id_student             | 28,785         | Unique students |
| gender                 | 2              | M / F |
| region                 | 13             | See table below |
| highest_education      | 5              | Levels from No Formal quals → Post Graduate |
| imd_band               | 11             | 0-100% + '?' to flag missing |
| age_band               | 3              | 0-35, 35-55, 55<= |
| disability             | 2              | N / Y |
| final_result           | 4              | Distinction, Pass, Fail, Withdrawn |

### region (13 distinct values)
| region |
|--------|
| East Anglian Region |
| East Midlands Region |
| Ireland |
| London Region |
| North Region |
| North Western Region |
| Scotland |
| South East Region |
| South Region |
| South West Region |
| Wales |
| West Midlands Region |
| Yorkshire Region |

### highest_education (5 distinct values)
| highest_education |
|------------------|
| A Level or Equivalent |
| HE Qualification |
| Lower Than A Level |
| No Formal quals |
| Post Graduate Qualification |

### imd_band (11 distinct values)
| imd_band |
|----------|
| 0-10%    |
| 10-20%   |
| 20-30%   |
| 30-40%   |
| 40-50%   |
| 50-60%   |
| 60-70%   |
| 70-80%   |
| 80-90%   |
| 90-100%  |
| ? *(missing/unrecorded)* |

- **Exact duplicates found:** 32,593  
  - *Exact duplicates* defined as rows where **all fields are identical**  

    <img width="1264" height="861" alt="image" src="https://github.com/user-attachments/assets/cc259270-8342-4c00-bd54-5dba789373df" />


## 2. Null & Anomaly Checks 

Checked all columns for missing values, and everything appears to be intact as initial null checks did not return any results

  <img width="1151" height="588" alt="image" src="https://github.com/user-attachments/assets/410b4a62-27c7-4b70-9e0b-670046919fca" />
  <img width="1255" height="568" alt="image" src="https://github.com/user-attachments/assets/24ce207f-f27b-45a8-875f-706cf987062e" />


| Column                | Null Count |
|-----------------------|------------|
| code_module           | 0          |
| code_presentation     | 0          |
| id_student            | 0          |
| gender                | 0          |
| region                | 0          |
| highest_education     | 0          |
| imd_band              | 0          |
| age_band              | 0          |
| num_of_prev_attempts  | 0          |
| studied_credits       | 0          |
| disability            | 0          |
| final_result          | 0          |


> All key identifiers and fields are present at initial check

A further sanity check was performed by reviewing the values in each column. The `imd_band` and `num_of_prev_attempts` column stood out because there are multiple rows with a value of **'?' and 0**, respectively. According to the dataset documentation, the columns are defined as such:
> `imd_band`: "Specifies the Index of Multiple Depravation band of the place where the student lived during the module-presentation"
> `num_of_prev_attempts`: "Number of times the student has attempted this module"

<img width="1205" height="552" alt="image" src="https://github.com/user-attachments/assets/ad49499d-ecd2-4caa-bcc5-4b10a218b63e" />

Going off this, **0** values are to be treated as valid under `num_of_prev_attempts`. Although **'?'** values under `imd_band` should be treated as NULL to flag as missing/unrecorded data

## 3. Cleaning Actions
- [x] Removed exact duplicates (all fields matching)  
- [x] Retained **0** values under `num_of_prev_attempts`
- [x] Standardized **'?'** values under `imd_band` to NULL
- [x] Verified numeric columns (`num_of_prev_attempts`, `studied_credits`) are in expected range and reasonable values

## 4. Sanity Checks Post-Cleaning
- **Total rows:** 32,593
- **Nulls remaining:** 2,222 (`imd_band`)

  <img width="501" height="276" alt="image" src="https://github.com/user-attachments/assets/86fa5aec-11c0-44b2-a641-9510f1694bae" />

## 5. Notes / Observations 
- Exact duplicates accounted for half the total rows; cleaning was ideal for optimization
- `num_of_prev_attempts` = 0 is valid; '?' in `imd_band` to be treated as NULL
- No further transformations applied at this stage; semantic derivation (e.g., pass/fail flags, grouping IMD bands) will be handled in later dim tables once business questions are finalized

---

# Table: studentRegistration

## 1. Overview

- **Total rows before cleaning:** 32,593

  <img width="452" height="252" alt="image" src="https://github.com/user-attachments/assets/2fd15ed4-4e54-42bc-8807-faad88d26f24" />

| Column               | Distinct Values | Notes / Examples |
|----------------------|----------------|-----------------|
| code_module          | 7              | AAA → GGG |
| code_presentation    | 4              | 2013B, 2013J, 2014B, 2014J |
| id_student           | 28,785         | Unique students |
| date_registration    | 333            | Integer days relative to module start (negative allowed) |
| date_unregistration  | 417            | Integer days relative to module start; null/missing if completed |
 
- **Exact duplicates found:** 0
  - *Exact duplicates* defined as rows where **all fields are identical**  
  - Rows with the same student and but differing dates, modules, or presentation are to be considered **valid and retained**
 
    s<img width="1238" height="332" alt="image" src="https://github.com/user-attachments/assets/9be0df0b-d2bf-4645-9052-2b19dd7bbc35" />

## 2. Null & Anomaly Checks

Checked all columns for missing values, and everything appears to be intact as initial null checks did not return any results

  <img width="1173" height="397" alt="image" src="https://github.com/user-attachments/assets/7a966eaa-84e4-4f00-9368-d59b06d6e318" />

| Column                | Null Count | Notes |
|-----------------------|------------|-------|
| code_module           | 0          | -     |
| code_presentation     | 0          | -     |
| id_student            | 0          | -     |
| date_registration     | 0          | 45 rows with `'?'` → to be converted to NULL. Normally an integer representing number of days relative to module start (negative = registered before module start) |
| date_unregistration   | 0          | 22,521 rows with `'?'` → to be converted to NULL. According to dataset dictionary, empty means student **completed the course**; unregistered students are flagged in `studentInfo.final_result` as Withdrawn |

> All key identifiers and fields are present

A further sanity check was performed by reviewing the values in each column. The `date_registration` and `date_unregistration` column stood out because there are multiple rows with a value of **'?'**. According to the dataset documentation, the columns are defined as such:
> **`date_registration`** – the date a student registered on the module presentation, measured in days relative to module start (e.g., -30 = 30 days before module start)

> **`date_unregistration`** – the date a student unregistered from the module presentation, measured in days relative to module start. Empty means student **completed** the course; unregistered students are flagged as **Withdrawn** in `studentInfo.final_result`

  <img width="903" height="314" alt="image" src="https://github.com/user-attachments/assets/e6508591-b732-42d7-8712-496c26927c35" />


Given this, '?' values will be standardized to NULL. For `date_registration` specifically, NULL likely could be unrecorded/missing data, while for `date_unregistration`, it's possible that value is missing due to student having finished the course; this will be clarified when joined with `studentInfo.final_result` to determine **Fail / Pass / Withdrawn**

## 3. Cleaning Actions
- [x] Verified there are no exact duplicates (all fields matching)  
- [x] Standardized `'?'` values to NULL  
  - `date_registration`: 45 rows → treat as missing/unrecorded  
  - `date_unregistration`: 22,521 rows → likely students who completed the module  
- [x] Retained 0 values as valid (day 0 = module start)  
- [x] Verified `date_registration` and `date_unregistration` were numeric and within expected range (negative/positive values allowed)  

## 4. Sanity Checks Post-Cleaning
- **Total rows:** 32,593
- **Nulls remaining:**
  - `date_registration`: 45 (after converting `'?'` → NULL)  
  - `date_unregistration`: 22,521 (after converting `'?'` → NULL)

## 5. Notes / Observations
- No exact duplicates were found
- Cleaning focused on converting `'?'` values to **NULL** for standardization  
- `date_registration` is generally negative/positive relative to module start; `date_unregistration` NULLs mostly correspond to students who **completed the course**, while Withdrawn students are flagged in `studentInfo.final_result`
- Will require joining with `studentInfo` to understand the context behind `'?'` values, e.g., distinguishing **completed vs. withdrawn**

---

# Table: studentVle

## 1. Overview

- **Total rows before cleaning:** 10,655,280

  <img width="372" height="271" alt="image" src="https://github.com/user-attachments/assets/a6ff5bfd-ec40-4a19-b172-609f254a5ef3" />
  
- **Distinct rows (per key columns):**  

| Column           | Distinct Values | Notes / Examples |
|------------------|----------------|-----------------|
| code_module      | 7              | AAA → GGG |
| code_presentation| 4              | 2013B, 2013J, 2014B, 2014J |
| id_student       | 26,074         | Unique students |
| id_site          | 6,268          | VLE resources / activities |
| date             | 295            | Integer days since module start |
 
    
  <img width="1028" height="381" alt="image" src="https://github.com/user-attachments/assets/4cedf82f-1c72-4611-9291-42ed15f1e486" />

- **Exact duplicates found:** 787,170
  - *Exact duplicates* defined as rows where **all fields are identical**
  - Rows with same student/module/site but differing `date` or `sum_click` are **valid and retained**.  
    - Aggregation/denormalization can be applied later in **mart/dim tables** depending on business questions

    <img width="560" height="369" alt="image" src="https://github.com/user-attachments/assets/68272f93-d06c-486d-8595-2aa2c01a75e4" />

## 2. Null & Anomaly Checks

Checked all columns for missing values, and everything appears to be intact as initial null checks did not return any results

<img width="1199" height="410" alt="image" src="https://github.com/user-attachments/assets/02472e36-cd71-4774-b90b-8173ba49d28c" />

## 3. Cleaning Actions
- [x] Removed exact duplicates (all columns matching)
- [x] Retained rows with same student and site but different dates or sum_click values
- [x] Verified `date` is numeric and within expected range (negative values considered valid)
- [x] Verified no nulls remain in key columns (`code_module`, `code_presentation`, `id_student`, `id_site`, `date`, `sum_click`)

## 4. Sanity Checks Post-Cleaning
- **Total rows after cleaning:** 9,868,110  
- **Nulls remaining:** 0  

## 5. Notes / Observations

- `date` is also measured in **integer days** since the start of the module presentation; negative values indicate interactions **before the official start**
- `sum_click` represents the number of times a student interacted with a VLE resource on a certain day; any zero values were considered valid and retained
- Rows with the same student, module, and site but differing `date` or `sum_click` were retained, as they represent **valid multiple interactions**
- No aggregation or derived columns were applied at this stage yet; denormalization or summarization to be handled later in **mart/dim tables** depending on business questions


