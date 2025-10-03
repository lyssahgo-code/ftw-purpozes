# Modelling Documentation

test schema only. all tables were derived from **Mart**:
- `group1_fact_assessment`
- `group1_fact_vle_interactions`
- `group1_dim_assessment`
- `group1_dim_course`
- `group1_dim_date`
- `group1_dim_student`
- `group1_dim_vle`

goal is to design a star schema:
- fact tables: actual measures (scores, clicks, etc)
- dimension tables: only connecting to fact tables. *with own surrogate key if possible*

## Fact Tables

### `fact_assessments`
> stores student assessment scores for each module + presentation

| Column | Type | Notes / Links |
|--------|------|---------------|
| id_student | INT32 | FK → `dim_student.studentkey` |
| id_assessment | INT32 | FK → `dim_assessment.assessmentkey` |
| code_module | Nullable STRING | FK → `dim_course.code_module` |
| code_presentation | Nullable STRING | FK → `dim_course.code_presentation` |
| date_key | INT32 | FK → `dim_date.datekey` |
| score | INT32 | Score student got |

---

### `fact_vle_interactions`
> tracks student activity in the VLE (clicks, interactions)

| Column | Type | Notes / Links |
|--------|------|---------------|
| id_student | INT32 | FK → `dim_student.studentkey` |
| id_site | INT32 | FK → `dim_vle.vlekey` |
| code_module | Nullable STRING | FK → `dim_course.code_module` |
| code_presentation | Nullable STRING | FK → `dim_course.code_presentation` |
| date_key | INT32 | FK → `dim_date.datekey` |
| sum_click | INT32 | Total clicks/interactions |

---

## Dimension Tables

### `dim_assessment`
> additional assessment details

| Column | Type | Notes |
|--------|------|-------|
| assessmentkey | INT32 | PK |
| code_module | Nullable STRING | Links to course |
| code_presentation | Nullable STRING | Links to course |
| assessment_type | Nullable STRING | TMA, exam, etc. |
| days_since_code_presentation | INT32 | Days offset from start |
| weight | INT32 | Weight in grade calc |

---

### `dim_course`
> additional module + presentation details

| Column | Type | Notes |
|--------|------|-------|
| code_module | Nullable STRING | PK (with code_presentation) |
| code_presentation | Nullable STRING | PK (with code_module) |
| module_presentation_length | INT32 | Length of presentation |

---

### `dim_date`
> dates breakdown

| Column | Type | Notes |
|--------|------|-------|
| code_presentation | Nullable STRING | Link to course presentation |
| datekey | INT32 | PK |
| presentation_year | INT32 | Year |
| presentation_semester | STRING | Semester (Jan, Oct) |
| year_completed | Nullable UINT16 | Year completed |
| months_since_start | Nullable INT64 | Months since start |

---

### `dim_student`
> student demographics and results

| Column | Type | Notes |
|--------|------|-------|
| studentkey | INT32 | PK |
| gender | STRING |  |
| region | STRING |  |
| education_level | STRING |  |
| imd_band | Nullable STRING | Deprivation index band |
| age_band | Nullable STRING | Age group |
| num_of_prev_attempts | INT32 | How many attempts before |
| studied_credits | INT32 | Total credits |
| has_disability | Nullable BOOL | Disability status |
| final_result | Nullable STRING | Pass/Fail/Withdrawn |

---

### `dim_vle`
> additional details regarding VLE activities

| Column | Type | Notes |
|--------|------|-------|
| vlekey | Nullable INT64 | PK |
| code_module | Nullable STRING | Link to course |
| code_presentation | Nullable STRING | Link to course |
| activity_type | Nullable STRING | e.g. forum, quiz |

---

## Relationships

### fact_assessments
- **id_student** → joins to `dim_student.studentkey`
- **id_assessment** → joins to `dim_assessment.assessmentkey`
- **code_module** → shared key, joins to `dim_assessment`, `dim_course`, `dim_vle`
- **code_presentation** → shared key, joins to `dim_assessment`, `dim_course`, `dim_date`, `dim_vle`
- **date_key** → joins to `dim_date.datekey`
- **score** → main measure for assessment performance

---

### fact_vle_interactions
- **id_student** → joins to `dim_student.studentkey`
- **id_site** → joins to `dim_vle.vlekey`
- **code_module** → shared key, joins to `dim_assessment`, `dim_course`, `dim_vle`
- **code_presentation** → shared key, joins to `dim_assessment`, `dim_course`, `dim_date`, `dim_vle`
- **date_key** → joins to `dim_date.datekey`
- **sum_click** → god measure (main metric of engagement)

### Quick Takeaways
- Both fact tables carry `code_module` + `code_presentation`, which act as **shared foreign keys** across multiple dims

---

## Schema 

```mermaid
erDiagram
  FACT_ASSESSMENTS {
    int id_student
    int id_assessment
    string code_module
    string code_presentation
    int date_key
    int score
  }

  FACT_VLE_INTERACTIONS {
    int id_student
    int id_site
    string code_module
    string code_presentation
    int date_key
    int sum_click
  }

  DIM_STUDENT {
    int studentkey
    string gender
    string region
    string education_level
    string imd_band
    string age_band
    int num_of_prev_attempts
    int studied_credits
    bool has_disability
    string final_result
  }

  DIM_ASSESSMENT {
    int assessmentkey
    string code_module
    string code_presentation
    string assessment_type
    int days_since_code_presentation
    int weight
  }

  DIM_COURSE {
    string code_module
    string code_presentation
    int module_presentation_length
  }

  DIM_DATE {
    string code_presentation
    int datekey
    int presentation_year
    string presentation_semester
    int year_completed
    int months_since_start
  }

  DIM_VLE {
    int vlekey
    string code_module
    string code_presentation
    string activity_type
  }

  FACT_ASSESSMENTS }o--|| DIM_STUDENT : "id_student = studentkey"
  FACT_ASSESSMENTS }o--|| DIM_ASSESSMENT : "id_assessment = assessmentkey"
  FACT_ASSESSMENTS }o--|| DIM_COURSE : "code_module, code_presentation"
  FACT_ASSESSMENTS }o--|| DIM_DATE : "date_key = datekey"
  FACT_ASSESSMENTS }o--|| DIM_VLE : "code_module, code_presentation"

  FACT_VLE_INTERACTIONS }o--|| DIM_STUDENT : "id_student = studentkey"
  FACT_VLE_INTERACTIONS }o--|| DIM_VLE : "id_site = vlekey"
  FACT_VLE_INTERACTIONS }o--|| DIM_COURSE : "code_module, code_presentation"
  FACT_VLE_INTERACTIONS }o--|| DIM_DATE : "date_key = datekey"
  FACT_VLE_INTERACTIONS }o--|| DIM_ASSESSMENT : "code_module, code_presentation"


