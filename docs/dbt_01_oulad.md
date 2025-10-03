# Data Cleaning & Sanity Checks
  
> Tables: studentassessment & studentvle

## Overview
This document summarizes the cleaning and sanity checks performed on the `studentassessment` and `studentvle` tables as part of our dbt workflow. Goal for now is to ensure data is consistent, free of duplicate and decide how to handle null values if present.

---

## 1. assessments

### a. Total Rows – `assessments`

- **Total rows before cleaning:** 206

  <img width="366" height="249" alt="image" src="https://github.com/user-attachments/assets/f3663912-75ad-49ec-8b3b-ea0f9c04e3e2" />  

- **Distinct rows (per key columns):**  206
  - `code_module`: 7
    | code_module |
    |-------------|
    | AAA         |
    | BBB         |
    | CCC         |
    | DDD         |
    | EEE         |
    | FFF         |
    | GGG         |
  - `code_presentation`: 4
    | code_presentation |
    |-------------------|
    | 2013J             | 
    | 2014J             |
    | 2013B             |
    | 2014B             |
    > Note: `code_presentation` indicates module start period (B = February, J = October)
  - `id_assessment`: 206
  - `assessment_type`: 3
    | assessment_type |
    |-----------------|
    | TMA             | 
    | Exam            |
    | CMA             |
 
- **Exact duplicates found:** 0
  - *Exact duplicates* defined as rows where **all fields are identical**  

  <img width="475" height="280" alt="image" src="https://github.com/user-attachments/assets/332d98f5-a34c-4020-9907-17fed3f4dfdf" />


### b. Null Checks – `assessments`

Checked all columns for missing values, and initial null check did not return any results

  <img width="1224" height="487" alt="image" src="https://github.com/user-attachments/assets/c33aff11-9968-47b0-953b-ca5d428ad2a3" />


| Column             | Null Count |
|--------------------|------------|
| id_assessment      | 0          |
| code_module        | 0          |
| code_presentation  | 0          |
| assessment_type    | 0          |
| date               | 0          |
| weight             | 0          |

> All key identifiers and fields are present

A further sanity check was performed by reviewing the values in each column. The `date` and `weight` column stood out because there are multiple rows with a value of **0** and **?**. According to the dataset documentation, date and weight are defined as such:
> DATE Definition from OULAD Dataset: information about the final submission date of the assessment, calculated as the number of days since the start of the module-presentation. The starting date of the presentation has number 0
> WEIGHT Definition from OULAD Dataset: weight of the assessment in %. Typically, exams are treated separately and have the weight 100%; sum of all other assessments is 100%

<img width="617" height="325" alt="image" src="https://github.com/user-attachments/assets/48410ba7-ba10-45a3-ad84-38e49797e17b" />


As such, **0** value would be treated as valid under date column but **?** values are to be considered as NULL as it is technically unrecorded/missing

Same goes for **0** valeus under weight column - will be kept as-is

### c. Cleaning Actions – `assessments`
- [x] Verified there are no exact duplicates (all fields matching)
- [x] Standardized **'?'** values under `date` to NULL
- [x] Retained '0' values under `date` (valid day-0 assessments) and `weight` (non-graded assessments)
- [x] Verified `date` and `weight` are within expected range (ex. for weight, expected 0-100)

### d. Sanity Checks Post-Cleaning – `assessments`
- Total rows: 206
- Nulls remaining: 11 (under `date` column) --retained as could be used for later analysis
- Values under `date` and `weight` are reasonable
> MIN `date`: -12
> MAX `date`: 261
> MIN `weight`: 0
> MAX `weight`: 100

  <img width="1231" height="421" alt="image" src="https://github.com/user-attachments/assets/33894f4a-891c-4ff1-b757-4f870f638817" />


## Notes / Observations

- No exact duplicates found as total count before and after cleaning remained the same
- **'?'** values were replaced with NULL. These were retained for now since they may provide useful insights in later analysis (eg. incomplete assessments, recording gaps, etc)
- **0** values treated as valid:
  - `date` = 0 (assessment taken on module start)
  - `weight` = 0 (measure of how an assessment counts toward the final grade: *possible non-graded*)
- Ranges appear to align with dataset documentation as well. For `date`, there are slightly negative values--worth checking in further analysis;
- As for `weight`, all exams count as 100, while other assessments (TMA/CMA) have different weights since they add up across modules
    
---

## 2. courses

### a. Total Rows – `courses`

- **Total rows before cleaning:** 22

  <img width="475" height="258" alt="image" src="https://github.com/user-attachments/assets/a6915f79-0664-4a07-aa3a-fd41ed2db6db" />


- **Distinct rows (per key columns):**  22
  - `code_module`: 7
    | code_module |
    |-------------|
    | AAA         |
    | BBB         |
    | CCC         |
    | DDD         |
    | EEE         |
    | FFF         |
    | GGG         |
  - `code_presentation`: 4
    | code_presentation |
    |-------------------|
    | 2013J             | 
    | 2014J             |
    | 2013B             |
    | 2014B             |
    > Note: `code_presentation` indicates module start period (B = February, J = October)
 
- **Exact duplicates found:** 0
  - *Exact duplicates* defined as rows where **all fields are identical**  

    <img width="585" height="280" alt="image" src="https://github.com/user-attachments/assets/c2d7535b-5237-491b-9c46-210cbb552b92" />


### b. Null Checks – `courses`

Checked all columns for missing values, and everything appears to be intact as initial null checks did not return any results. No **0 or '?'** were found either

  <img width="952" height="361" alt="image" src="https://github.com/user-attachments/assets/e4cbc052-6183-4fae-856f-6b9ac852cc7d" />


| Column                       | Null Count |
|------------------------------|------------|
| code_module                  | 0          |
| code_presentation            | 0          |
| module_presentation_length   | 0          |

> All key identifiers and fields are present

### c. Cleaning Actions – `courses`
- [x] Verified there are no exact duplicates (all fields are unique)
- [x] Retained repeating `code_module` values since each one is tied to a different `code_presentation` (the same module offered in different terms)
- [x] Retained repeating `code_presentation` values since they occur across different modules (different courses offered in the same term)
- [x] Verified values under `module_presentation_length` are consistent across rows and in integer format

### d. Sanity Checks Post-Cleaning – `courses`
- Total rows: 22
- Nulls remaining: 0
- Values range:
  > MIN `module_presentation_length`: 234
  > MAX `module_presentation_length`: 269

  <img width="387" height="267" alt="image" src="https://github.com/user-attachments/assets/d6d00d7c-51bd-432b-84cc-e5c5765df1e9" />
  <img width="580" height="286" alt="image" src="https://github.com/user-attachments/assets/3b723f42-413a-4675-bf37-d2e63f1b7d8f" />

## Notes / Observations

- Dataset is small so minimal cleaning required. No nulls/exact duplicates detected
- `module_presentation_length` measured in days and fall between 234-269 days, meaning modules usually run for a course of 7-9 months

---

## 3. studentassessment

### a. Total Rows – `studentassessment`

- **Total rows before cleaning:** 347,824 😳 

  ![Total Rows](https://github.com/user-attachments/assets/05a359dd-63c1-40c6-84ad-a9d9d0fa5213)

- **Distinct rows (per key columns):**  
  - `id_assessment`: 188  
  - `id_student`: 23,369  

  ![Distinct Assessments](https://github.com/user-attachments/assets/96d5a7ed-0873-4766-9516-6998cf0443d0)
  ![Distinct Students](https://github.com/user-attachments/assets/32542c7c-aeee-44d9-b3dd-34589a42825c)

- **Exact duplicates found:** 173,912 😬
  - *Exact duplicates* defined as rows where **all fields are identical**  
  - Rows with the same student and assessment but differing submission dates, scores, or other fields are to be considered **valid and retained**

### b. Null Checks – `studentassessment`

Checked all columns for missing values, and everything appears to be intact as null checks did not return any results

<img width="1197" height="393" alt="image" src="https://github.com/user-attachments/assets/158bc8c6-5912-4016-9fdc-723bac743e6e" />


| Column         | Null Count |
|----------------|------------|
| id_student     | 0          |
| id_assessment  | 0          |
| date_submitted | 0          |
| is_banked      | 0          |
| score          | 0          |

> All key identifiers and fields are present

A further sanity check was performed by reviewing the values in each column. The `score` column stood out because there are multiple rows with a value of **0**. According to the dataset documentation, the valid score range is **0 to 100**.
> Definition from OULAD dataset: "Score in this assessment. The range is from 0 to 100. Scores lower than 40 are interpreted as Fail. The marks are in the range from 0 to 100"

**Note:** In the original CSV dataset, missing or unrecorded scores were represented as `?`, which differs from what queries in DBeaver returned. In the current table, all `score` values are numeric, and there are no nulls. 

<img width="846" height="520" alt="image" src="https://github.com/user-attachments/assets/0e9d6bf8-68bf-4c83-91ee-7ed4d18f3724" />

Possible explanations for a score of 0 or ? include:  
- The student failed the assessment  
- The student *did not attempt* the assessment  
- The student was *not enrolled* in that assessment

As a result, 0 or ? values were retained, as they may provide useful information for further analysis, particularly when combined with other tables to gain more context about students who scored 0 or ?

### c. Cleaning Actions – `studentassessment`
- [x] Removed exact duplicates (all fields matching)  
- [x] Retained multiple attempts for same `id_student` + `id_assessment` if dates/scores differ  
- [x] Verified `date_submitted` is numeric and within expected range (negative/positive values are considered)
> MIN `date_submitted`: -11

> MAX `date_submitted`: 608 
- [x] Verified no nulls remain
> Rows with 0 or ? values: 502

### d. Sanity Checks Post-Cleaning – `studentassessment`
- Total rows: 173,912
- Nulls remaining: 0
> Rows with 0 or ? values: 502

<img width="475" height="283" alt="image" src="https://github.com/user-attachments/assets/20827e1a-0398-4b85-b268-5d7a44edc08b" />


## Notes / Observations

- `date_submitted` is **integer days**, not an actual date
  > Definition from OULAD dataset: "The date of student submission, measured as the number of days since the start of the module presentation."  
- Negative values are **valid**, indicating days *before* module start
- Score range: 0–100, with <40 interpreted as **Fail**. Missing scores could mean the student did not take the assessment
- Cleaning was limited to **removing exact duplicates**, **verifying nulls**, and **checking formats** Semantic derivation (e.g., `pass/fail`, handling missing scores) will be done later in **dim tables** once business questions are finalized  
- No additional normalization or derived columns were added at this stage

---

## 4. studentInfo

### a. Total Rows – `studentInfo`

- **Total rows before cleaning:** 65,186

  <img width="520" height="246" alt="image" src="https://github.com/user-attachments/assets/825437e9-e1e8-4fb7-99ea-d6d427ba9510" />

- **Distinct rows (per key columns):**  _________
  - `code_module`: 7
  - `code_presentation`: 4
  - `id_student`: 28,785
  - `gender`: 2
    | gender |
    |--------|
    | M      |
    | F      |
  - `region`: 13
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
  - `highest_education`: 5
    | highest_education          |
    |----------------------------|
    | A Level or Equivalent      |
    | HE Qualification           |
    | Lower Than A Level         |
    | No Formal quals            |
    | Post Graduate Qualification|
  - `imd_band`: 11
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
    | ? *(to do further check — likely missing/unrecorded data)* |

  - `age_band`: 3
    | age_band |
    |----------|
    | 0-35     |
    | 35-55    |
    | 55<=     |

  - `disability`: 2
    | disability |
    |------------|
    | N          |
    | Y          |

  - `final_result`: 4
    | final_result |
    |--------------|
    | Distinction  |
    | Fail         |
    | Pass         |
    | Withdrawn    |
    
 
- **Exact duplicates found:** 32,593
  - *Exact duplicates* defined as rows where **all fields are identical**  

    <img width="1264" height="861" alt="image" src="https://github.com/user-attachments/assets/cc259270-8342-4c00-bd54-5dba789373df" />


### b. Null Checks – `studentInfo`

Checked all columns for missing values, and everything appears to be intact as null checks did not return any results

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
> IMD_BAND definition from OULAD dataset: "Specifies the Index of Multiple Depravation band of the place where the student lived during the module-presentation"
> NUM_OF_PREV_ATTEMPTS definition from OULAD dataset: "Number of times the student has attempted this module

<img width="1205" height="552" alt="image" src="https://github.com/user-attachments/assets/ad49499d-ecd2-4caa-bcc5-4b10a218b63e" />

Going off this, **0** values are to be treated as valid under `num_of_prev_attempts`. Although **'?'** values under `imd_band` should be treated as NULL to flag as missing/unrecorded data


### c. Cleaning Actions – `studentInfo`
- [x] Removed exact duplicates (all fields matching)  
- [x] Retained **0** values under `num_of_prev_attempts`
- [x] Standardized **'?'** values under `imd_band` to NULL
- [x] Verified `date_submitted` is numeric and within expected range (negative/positive values are considered)

### d. Sanity Checks Post-Cleaning – `studentInfo`
- Total rows: 32,593
- Nulls remaining: 0

  <img width="501" height="276" alt="image" src="https://github.com/user-attachments/assets/86fa5aec-11c0-44b2-a641-9510f1694bae" />


## Notes / Observations - `studentInfo`

---

## 5. studentRegistration

### a. Total Rows – `____________`

- **Total rows before cleaning:**

  [screenshot raw count here]

- **Distinct rows (per key columns):**  _________
  - `_________`: ____
  - `_________`: ____
 
- **Exact duplicates found:** _________
  - *Exact duplicates* defined as rows where **all fields are identical**  
  - Rows with the same student and assessment but differing submission dates, scores, or other fields are to be considered **valid and retained**

### b. Null Checks – `____________`

Checked all columns for missing values, and everything appears to be intact as null checks did not return any results

  [screenshot null query here]

| Column         | Null Count |
|----------------|------------|
| ___________    | 0          |
| ___________    | 0          |
| ___________    | 0          |
| ___________    | 0          |
| ___________    | 0          |

> All key identifiers and fields are present

### c. Cleaning Actions – `____________`
- [x] Removed exact duplicates (all fields matching)  
- [x] Retained multiple attempts for 
- [x] Verified `date_submitted` is numeric and within expected range (negative/positive values are considered)

### d. Sanity Checks Post-Cleaning – `____________`
- Total rows: 173,912
- Nulls remaining: 0

## Notes / Observations

---

## 6. studentvle

### a. Total Rows – `studentVle`
- **Total rows before cleaning:** 10,655,280

  <img width="372" height="271" alt="image" src="https://github.com/user-attachments/assets/a6ff5bfd-ec40-4a19-b172-609f254a5ef3" />


- **Distinct rows (per key columns):**  
  - `code_module`: 7
    | code_module |
    |-------------|
    | AAA         |
    | BBB         |
    | CCC         |
    | DDD         |
    | EEE         |
    | FFF         |
    | GGG         |
    
  - `code_presentation`: 4
    | code_presentation |
    |------------------|
    | 2013J            | 
    | 2014J            |
    | 2013B            |
    | 2014B            |
    > Note: `code_presentation` indicates module start period (B = February, J = October)
    
  - `id_student`: 26,074
  - `id_site`: 6,268
  - `date`: 295
 
 
    <img width="1028" height="381" alt="image" src="https://github.com/user-attachments/assets/4cedf82f-1c72-4611-9291-42ed15f1e486" />


- **Exact duplicates found:** 787,170
  - *Exact duplicates* defined as rows where **all fields are identical**
  - Rows with the same student and modules/presentation but differing submission dates, sum clicks, or other fields are to be considered **valid and retained**. Depending on finalized business questions, would consider aggregation to denormalize

    <img width="560" height="369" alt="image" src="https://github.com/user-attachments/assets/68272f93-d06c-486d-8595-2aa2c01a75e4" />


### b. Null Checks – `studentVle`

Checked all columns for missing values, and everything appears to be intact as null checks did not return any results

<img width="1199" height="410" alt="image" src="https://github.com/user-attachments/assets/02472e36-cd71-4774-b90b-8173ba49d28c" />


### c. Cleaning Actions – `studentVle`
- [x] Removed exact duplicates (all columns matching)
- [x] Retained rows with same student and site but different dates or sum_click values
- [x] Verified `date` is numeric and within expected range (negative values are valid)
- [x] Verified no nulls remain in key columns (`code_module`, `code_presentation`, `id_student`, `id_site`, `date`, `sum_click`)

### d. Sanity Checks Post-Cleaning – `studentVle`
- Total rows: 9,868,110
- Nulls remaining: 0

## Notes / Observations – `studentVle`

- `date` is also measured in **integer days** since the start of the module presentation; negative values indicate interactions **before the official start**
- `sum_click` represents the number of times a student interacted with a VLE resource on a certain day; any zero values were considered valid and retained
- Rows with the same student, module, and site but differing `date` or `sum_click` were retained, as they represent **valid multiple interactions**
- No aggregation or derived columns were applied at this stage yet; denormalization or summarization to be handled later in **mart/dim tables** depending on business questions


