# OULAD Business Questions

### 1. Do demographics influence dropout?
- **Tables:** dim_student, fact_assessments  
- **Joins:** 
  - fact_assessments.id_student → dim_student.studentkey  
- **Additional columns:** Gender, age_band, region, education_level, imd_band

### 2. How does engagement in the VLE impact performance?
- **Tables:** fact_vle_interactions, fact_assessments, dim_vle, dim_student  
- **Joins:** 
  - fact_vle_interactions.id_student → dim_student.studentkey  
  - fact_vle_interactions.id_site → dim_vle.vlekey  
  - fact_assessments.id_student → dim_student.studentkey  
  - dim_vle.code_module → fact_vle_interactions.code_module  
  - dim_vle.code_presentation → fact_vle_interactions.code_presentation

### 3. How do age bands affect dropout rates?
- **Tables:** dim_student, fact_assessments  
- **Joins:** fact_assessments.id_student → dim_student.studentkey  
- **Additional columns:** Age_band, Final_result

### 4. Do students from certain regions or IMD bands perform better/worse?
- **Tables:** dim_student, fact_assessments  
- **Joins:** fact_assessments.id_student → dim_student.studentkey  
- **Additional columns:** score, final_result, region, imd_bad

### 5. Which assessments (Exam, TMA, CMA) have the highest failure rates?
- **Tables:** fact_assessments, dim_assessment  
- **Joins:** fact_assessments.id_assessment → dim_assessment.assessmentkey  
- **Additional columns:** assessment_type, fail_percent

### 6. Which VLE activity types (forum, quiz, wiki, resource) lead to better performance?
- **Tables:** fact_vle_interactions, dim_vle, fact_assessments, dim_student  
- **Joins:** 
  - fact_vle_interactions.id_student → dim_student.studentkey  
  - fact_assessments.id_student → dim_student.studentkey  
  - fact_vle_interactions.id_site → dim_vle.vlekey  
- **Additional columns** activity_type, sum_click, score
