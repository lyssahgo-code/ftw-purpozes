# Modelling Documentation

goal is to design a star schema:
- fact tables: actual measures (scores, clicks, etc)
- dimension tables: only connecting to fact tables. *with own surrogate key if possible*

## Fact Tables

### `fact_assessments`
> stores student assessment scores for each module + presentation

| Column | Type | Notes / Links |
|--------|------|---------------|
| studentkey | INT32 | FK → `dim_student.studentkey` |
| assessmentkey | INT32 | FK → `dim_assessment.assessmentkey` |
| code_module | STRING | FK → `dim_course.code_module`, `dim_vle.code_module` |
| code_presentation | STRING | FK → `dim_course.code_presentation`, `dim_date.code_presentation`, `dim_vle.code_presentation` |
| datekey | INT32 | FK → `dim_date.datekey` |
| score | INT32 | Measure: student score |

---

### `fact_vle_interactions`
> tracks student activity in the VLE (clicks, interactions)

| Column | Type | Notes / Links |
|--------|------|---------------|
| studentkey | INT32 | FK → `dim_student.studentkey` |
| vlekey | INT32 | FK → `dim_vle.vlekey` |
| code_module | STRING | FK → `dim_course.code_module`, `dim_vle.code_module` |
| code_presentation | STRING | FK → `dim_course.code_presentation`, `dim_date.code_presentation`, `dim_vle.code_presentation` |
| datekey | INT32 | FK → `dim_date.datekey` |
| sum_click | INT32 | Measure: VLE interactions |

---

## Dimension Tables

### `dim_assessment`
> additional assessment details

| Column | Type | Notes |
|--------|------|-------|
| assessmentkey | INT32 | PK |
| code_module | STRING | Links to course |
| code_presentation | STRING | Links to course |
| assessment_type | STRING | TMA, exam, etc. |
| days_since_code_presentation | INT32 | Offset from module start |
| weight | FLOAT | Weight in grade calculation |

---

### `dim_course`
> additional module + presentation details

| Column | Type | Notes |
|--------|------|-------|
| code_module | STRING | PK (together with code_presentation) |
| code_presentation | STRING | PK (together with code_module) |
| module_presentation_length | INT32 | Length of presentation |

---

### `dim_date`
> dates breakdown

| Column | Type | Notes |
|--------|------|-------|
| code_presentation | STRING | Link to course |
| datekey | INT32 | PK |
| presentation_year | INT32 | Year |
| presentation_semester | STRING | Semester (Spring/Autumn) |
| year_completed | UINT16 | Nullable |
| months_since_start | INT64 | Nullable |

---

### `dim_student`
> student demographics and results

| Column | Type | Notes |
|--------|------|-------|
| studentkey | INT32 | PK |
| gender | STRING |  |
| region | STRING |  |
| education_level | STRING |  |
| imd_band | STRING | Nullable |
| age_band | STRING | Nullable |
| num_of_prev_attempts | INT32 |  |
| studied_credits | INT32 |  |
| has_disability | BOOL | Nullable |
| final_result | STRING | Nullable (Pass/Fail/Withdrawn/Distinction) |

---

### `dim_vle`
> additional details regarding VLE activities

| Column | Type | Notes |
|--------|------|-------|
| vlekey | INT64 | PK |
| code_module | STRING | Link to course |
| code_presentation | STRING | Link to course |
| activity_type | STRING | e.g. forum, quiz |

---

## Relationships

### fact_assessments
- **studentkey** → `dim_student.studentkey`  
- **assessmentkey** → `dim_assessment.assessmentkey`  
- **code_module** → shared key to: `dim_assessment.code_module`, `dim_course.code_module`, `dim_vle.code_module`  
- **code_presentation** → shared key to: `dim_assessment.code_presentation`, `dim_course.code_presentation`, `dim_date.code_presentation`, `dim_vle.code_presentation`  
- **datekey** → `dim_date.datekey`  
- **score** → main measure for assessment performance  

---

### fact_vle_interactions
- **studentkey** → `dim_student.studentkey`  
- **vlekey** → `dim_vle.vlekey`  
- **code_module** → shared key to: `dim_assessment.code_module`, `dim_course.code_module`, `dim_vle.code_module`  
- **code_presentation** → shared key to: `dim_assessment.code_presentation`, `dim_course.code_presentation`, `dim_date.code_presentation`, `dim_vle.code_presentation`  
- **datekey** → `dim_date.datekey`  
- **sum_click** → main metric of student engagement  

---

### Quick Takeaways
- Both fact tables carry `code_module` + `code_presentation`, which act as **shared foreign keys** across multiple dims

---

## Schema 

```mermaid
erDiagram
    %% Fact Tables
    FACT_ASSESSMENTS {
        Int64 StudentKey FK
        Int64 AssessmentKey FK
        String code_module
        String code_presentation
        Int64 DateKey FK
        Int64 is_banked
        Int64 score
    }

    FACT_VLE_INTERACTIONS {
        Int64 StudentKey FK
        Int64 VLEKey FK
        String code_module
        String code_presentation
        Int64 DateKey FK
        Int64 sum_click
    }

    %% Dimension Tables
    DIM_STUDENT_DEMOGRAPHICS {
        Int64 StudentKey PK
        String gender
        String region
        String education_level
        String imd_band
        String age_band
        Int64 has_disability
    }

    DIM_STUDENT_MODULE {
        Int64 StudentKey PK
        String code_module
        String code_presentation
        Int64 num_of_prev_attempts
        Int64 studied_credits
        String final_result
    }

    DIM_COURSE {
        String code_module PK
        String code_presentation PK
        Int64 module_presentation_length
    }

    DIM_ASSESSMENT {
        Int64 AssessmentKey PK
        String code_module
        String code_presentation
        String assessment_type
        Int32 days_since_code_presentation
        Float64 weight
    }

    DIM_DATE {
        Int64 DateKey PK
        String code_presentation
        Int32 presentation_year
        String presentation_semester
        Int32 year_completed
        Int32 months_since_start
    }

    %% Relationships
    FACT_ASSESSMENTS ||--o{ DIM_STUDENT_DEMOGRAPHICS : "StudentKey"
    FACT_ASSESSMENTS ||--o{ DIM_STUDENT_MODULE : "StudentKey"
    FACT_ASSESSMENTS ||--o{ DIM_ASSESSMENT : "AssessmentKey"
    FACT_ASSESSMENTS ||--o{ DIM_DATE : "DateKey"

    FACT_VLE_INTERACTIONS ||--o{ DIM_STUDENT_DEMOGRAPHICS : "StudentKey"
    FACT_VLE_INTERACTIONS ||--o{ DIM_DATE : "DateKey"
    FACT_VLE_INTERACTIONS ||--o{ DIM_COURSE : "code_module + code_presentation"

    DIM_STUDENT_MODULE ||--o{ DIM_COURSE : "code_module + code_presentation"



