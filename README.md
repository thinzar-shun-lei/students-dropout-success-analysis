# Student Dropout & Academic Success: Intervention Management

## Project Overview

This project analyzes **student dropout and academic success** to identify key risk factors and translate the findings into actionable **intervention management strategies**.

The dashboard was developed using **Power BI** for the **Matrix College Dashboard Challenge 2026** and was **nominated in the Top 5**.

The analysis focuses on four key areas:

- What factors influence student dropout and academic success?
- Which students and courses are most at risk?
- Where should intervention resources be prioritized?
- What actions can support student retention and academic success?

---

## Business Problem & Key Question

Universities need to identify the factors contributing to student dropout and determine where intervention resources should be prioritized.

### Key Business Question

> **What factors are influencing student dropout and academic success, and what intervention strategies can the university take?**

---

## Dataset

The analysis uses the **Predict Students' Dropout and Academic Success** dataset from the **UCI Machine Learning Repository**.

**Dataset Source:**  
[UCI Machine Learning Repository – Predict Students' Dropout and Academic Success](https://archive.ics.uci.edu/dataset/697/predict+students+dropout+and+academic+success)

---

## Data Connection, Cleaning & Transformation

The dataset required **minimal cleaning**, with no significant missing values, duplications, or invalid records identified.

### Data Preparation

- Connected the UCI dataset to **Power BI**
- Reviewed data types and data quality
- Created supporting dimension tables:
  - `dim_course`
  - `dim_applicationmode`
- Built a **Star Schema** data model
- Created analytical categories to support business-focused analysis

### Student Categorization

Students were categorized based on:

- **Age Groups**
- **Application Order**
- **Financial Status**
- **Marital Status**
- **Academic Progression**
- **Student Profile**
- **Semester Academic Profile**

### Course Prioritization

The **Pareto Principle** was applied to course-level dropout and graduation analysis.

Courses were ranked according to their contribution to student outcomes and grouped into **A, B, and C priority categories** to support targeted intervention.

---

## Data Model

The Power BI model follows a **Star Schema** approach.

<img width="938" height="631" alt="image" src="https://github.com/user-attachments/assets/642c7d75-f330-4380-b37b-b687e476c249" />

## DAX & Analytical Logic
`SWITCH(TRUE())` was mainly used to implement business rules and classify students based on multiple conditions.

### Semester Academic Profile
Students were classified using:
* **Semester Grade**
* **Student Approval Rate**

The thresholds were:
* **Grade:** `12`
* **Approval Rate:** `70%`

```dax
S1 Profile =
VAR gradeThreshold = 12
VAR approvalThreshold = 0.7

RETURN
    SWITCH(
        TRUE(),

        fact_student[1S Grade] >= gradeThreshold &&
        fact_student[1S Student Approval Rate] >= approvalThreshold,
        "Strong Performers",

        fact_student[1S Grade] < gradeThreshold &&
        fact_student[1S Student Approval Rate] >= approvalThreshold,
        "Struggling but Engaged",

        fact_student[1S Grade] >= gradeThreshold &&
        fact_student[1S Student Approval Rate] < approvalThreshold,
        "Inconsistent / Disengaging",

        fact_student[1S Grade] < gradeThreshold &&
        fact_student[1S Student Approval Rate] < approvalThreshold,
        "Highest Academic Risk",

        BLANK()
    )
```
---

### Overall Academic Risk
Academic risk combines semester profiles with progression between Semester 1 and Semester 2.
```dax
Overall Academic Risk =
    SWITCH(
        TRUE(),

        fact_student[Status between semesters] IN {"Declined", "No Grade"}
            &&
        (
            fact_student[S1 Profile] IN {
                "Highest Academic Risk",
                "Inconsistent / Disengaging"
            }
            ||
            fact_student[S2 Profile] IN {
                "Highest Academic Risk",
                "Inconsistent / Disengaging"
            }
        ),
        "Higher Academic Risk",

        "Lower Academic Risk"
    )
```
---

### Cumulative Dropout
Course dropout ranking and cumulative dropout were calculated to support Pareto-based prioritization.
```dax
Course Dropout Rank =
VAR CurrentDropouts = [Dropout Students]

RETURN
    RANKX(
        ALL(
            dim_course[CourseCode],
            dim_course[Course Name]
        ),
        CALCULATE([Dropout Students]),
        CurrentDropouts,
        DESC,
        DENSE
    )
```
```dax
Cumulative Dropout =
VAR CurrentRank =
    [Course Dropout Rank]

RETURN
    CALCULATE(
        [Dropout Students],
        FILTER(
            ALL(
                dim_course[CourseCode],
                dim_course[Course Name]
            ),
            [Course Dropout Rank] <= CurrentRank
        )
    )
```
---

## Key Insights & Recommendations

### Age
* **Insight:** Dropout is higher among middle- and older-age groups, while younger students show stronger graduation outcomes. The *"Over 23 Years Old"* application group also shows elevated dropout.
* **Recommendation:** Prioritize older and working-age students for proactive monitoring and targeted support. Investigate work, course-load, and scheduling constraints.

### Courses
* **Insight:** Dropout is concentrated in a smaller group of courses, with **Management courses** (particularly evening courses) showing higher dropout risk. **Nursing** provides a strong benchmark for student retention.
* **Recommendation:** Prioritize high-risk courses and investigate differences between daytime and evening delivery. Adapt effective teaching and student-support practices from stronger-performing courses.

### Academic Progression
* **Insight:** Grade decline between semesters is associated with higher dropout. Students with no recorded grades also show substantial dropout, requiring data validation.
* **Recommendation:** Establish an academic risk matrix for semester-to-semester monitoring and early intervention. Review course-load management, provide targeted academic support, and validate zero-grade records.

### Financial & Economic Factors
* **Insight:** Scholarship support decreases with age, while financial burden and dropout increase among working-age students. Financial risk is associated with dropout even among students with lower academic risk.
* **Recommendation:** Expand targeted financial assistance and scholarships, particularly for middle-age students. During economic pressure, strengthen financial support alongside career guidance, internships, and employment support.

### Demographic Factors
* **Insight:** Male, married, and divorced students show higher dropout risk, with middle-age students also requiring greater retention attention.
* **Recommendation:** Develop targeted retention programs combining academic monitoring, counselling, and flexible support for students managing work and family commitments.
