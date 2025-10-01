# Data Cleaning & Sanity Checks
  
> Tables: studentassessment & studentvle

## Overview
This document summarizes the cleaning and sanity checks performed on the `studentassessment` and `studentvle` tables as part of our dbt workflow. Goal for now is to ensure data is consistent, free of duplicate and decide how to handle null values if present.

---

## 5. studentassessment

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
    | 2014B            |
    | 2014J            |
    | 2015B            |
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


