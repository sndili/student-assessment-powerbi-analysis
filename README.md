# Power BI End-to-End Project: Education Sector Data Transformation & Analysis

## Overview

This project demonstrates analysis and visual solution using a messy dataset from the education sector. The objective was to transform this dataset into an optimized schema, develop analytical DAX measures and design a dashboard to deliver insights across various questions.


## Part 1: Data Cleaning (Power Query)

The raw dataset was loaded as a flat table and heavily scrubbed in **Power Query Editor** to guarantee reliability and consistency. Major transformation decisions included:

- **Handling Nulls and Blanks:** Applied cleaning methods by replacing missing rows with `"Unknown"`. This prevents primary key validation errors and tracks unassigned records cleanly.
- **Case Consistency & Text Fixes:** Cleaned up inconsistent casings (e.g., mixed upper/lowercase categories), eliminated hidden spaces, fixed typos and expanded abbreviations across `School Name`, `Subject` and `Gender`.
- **Data Type Corrections:** Converted numerical values stored as text into proper types. Ensured scores, attendance rates and currency values were designated awith their correct data types. (Whole number, Text, Currency)
- **Geographic Cleaning:** Corrected mismatched sub-county, county and regional hierarchies to create consistency when looking up values.



## Part 2: Data Modeling 

To optimize report performance and eliminate data repetition, the flat table was broken down and normalized into an efficient **Star Schema**.

### Entity Relationship Diagram Structure

**Fact Table:**
- `FactAssessment`: Houses foreign keys and volatile transactional data (`Score`, `Attendance Rate Pct`, `Fee Balance Kes`, `Payment Method`, `Term/Academic Period`).

**Dimension Tables:**
- `DimStudent`: Contains student demographics (`Student Id`, `Student Name`, `Gender`).
- `DimTeacher`: Holds faculty records (`TeacherID`, `Teacher Name`).
- `DimSchool`: Holds educational facility metrics (`School Id`, `School Name`).
- `DimSubject`: Unique listing of academic courses (`SubjectID`, `Subject Name`).
- `DimLocation`: Centralized geographic dimension tracking the hierarchy: `Region` ──► `County` ──► `Sub County`.

### Relationships

- **Many-to-One Relationships (\*:1):** Created standard active relationships linking foreign keys in `FactAssessment` to their respective unique keys in the dimension tables.
- **Handling Inactive Relationships:** To map dual geographic relationships (e.g., where a student resides vs. where a school is located) without creating confusion, secondary paths between `DimStudent`/`DimSchool` and `DimLocation` were configured as **Inactive Relationships** (dashed lines) to be called upon safely via DAX `USERELATIONSHIP`.
- **Duplicate Elimination:** Applied `Remove Duplicates` constraints on all primary keys (`TeacherID`, `SubjectID`, `School Id`) across the dimension tables to protect model integrity.



## Part 3: All Measures & Calculations (DAX)

The analytical metrics were solved using DAX measures inside a dedicated `_Measures`.

### 1. Count Measures

```dax
Total Students = DISTINCTCOUNT(FactAssessment[Student Id])

Total Schools = DISTINCTCOUNT(DimStudent[School Id])

Total Teachers = DISTINCTCOUNT(FactAssessment[TeacherID])
```

### 2. Performance Measures

```dax
Avg Score = AVERAGE(FactAssessment[Score])

Pass Rate = 
DIVIDE(
    CALCULATE(COUNTROWS(FactAssessment), FactAssessment[Score] >= 50),
    COUNTROWS(FactAssessment)
)

Fail Rate = 1 - [Pass Rate]

Avg Attendance = AVERAGE(FactAssessment[Attendance Rate Pct])
```

### 3. Revenue & Financial Performance Measures

```dax
Total Fee Balance (Outstanding) = SUM(FactAssessment[Fee Balance Kes])

Total Expected Revenue = [Total Students] * 5000

Total Revenue Collected = [Total Expected Revenue] - [Total Fee Balance (Outstanding)]

Avg Fee Paid = DIVIDE([Total Revenue Collected], [Total Students])
```

### 4. Operational Ratio Measures

```dax
Teacher Student Ratio = DIVIDE([Total Students], [Total Teachers])
```



## Part 4: Dashboard

To present the analysis requirements clearly without cluttering the canvas, the project features a 5-page interactive dashboard mapped out across the 35 business questions.

### Global Filters (Top Slicers on Every Page)

- `DimSchool[School Name]` (Dropdown)
- `DimLocation[County]` (Dropdown)
- `FactAssessment[Academic Year / Term]` (Dropdown)

### Page 1: School & Enrollment Overview 

**KPI Cards:** `[Total Students]`, `[Total Schools]`, `[Total Teachers]`

| Question | Visual | Configuration |
|---|---|---|
| Q1 & Q4 (School Enrollment) | Clustered Bar Chart | Y-Axis: `DimSchool[School Name]` · X-Axis: `[Total Students]` |
| Q2 (Top Counties) | Clustered Bar Chart | Y-Axis: `DimLocation[County]` · X-Axis: `[Total Students]` |
| Q3 (Regional Population) | Donut Chart | Legend: `DimLocation[Region]` · Values: `[Total Students]` |
| Q5 (Gender Distribution) | Stacked Bar Chart | Y-Axis: `DimSchool[School Name]` · X-Axis: `[Total Students]` · Legend: `DimStudent[Gender]` |
| Q6 (Lowest Sub-County) | Table Visual (sorted ascending) | Columns: `DimLocation[Sub County]`, `[Total Students]` |
| Q7 (Population Diversity) | Matrix Table | Rows: `DimSchool[School Name]` · Columns: `DimLocation[County]` · Values: `[Total Students]` |

### Page 2: Academic Performance Analysis

**KPI Cards:** `[Avg Score]`, `[Pass Rate]`, `[Fail Rate]`

| Question | Visual | Configuration |
|---|---|---|
| Q13 (Overall Performance) | Card Visual | `[Avg Score]` |
| Q14 (Highest School Performance) | Table (sorted descending) | Columns: `DimSchool[School Name]`, `[Avg Score]` |
| Q15 & Q16 (Subject Ranking Metrics) | Clustered Column Chart | X-Axis: `DimSubject[Subject Name]` · Y-Axis: `[Avg Score]` |
| Q18 (County Performance) | Filled Map / Data Table | `DimLocation[County]` and `[Avg Score]` |
| Q19 (At-Risk Demographics) | Table Visual | Columns: `DimStudent[Student Name]`, `[Avg Score]` · Filter: `[Avg Score] < 50` |
| Q20 (Passing Baseline) | Card KPI | `[Pass Rate]` |
| Q21 (Failure Metrics) | Clustered Bar Chart | Y-Axis: `DimSubject[Subject Name]` · X-Axis: `[Fail Rate]` |
| Q22 (Academic Support Priorities) | Sorting Matrix | Columns: `DimSchool[School Name]`, `[Pass Rate]`, `[Avg Score]` (sorted ascending) |

### Page 3: Faculty & Teacher Analysis

**KPI Cards:** `[Total Teachers]`, `[Teacher Student Ratio]`

| Question | Visual | Configuration |
|---|---|---|
| Q8 (Staff Availability) | Card Visual | `[Total Teachers]` |
| Q9 (Teacher Allocations) | Clustered Bar Chart | Y-Axis: `DimTeacher[Teacher Name]` · X-Axis: `[Total Students]` |
| Q10 (Workload Density) | Table (sorted descending) | Columns: `DimSchool[School Name]`, `[Teacher Student Ratio]` |
| Q11 (Staff Geographics) | Data Table | Columns: `DimLocation[County]`, `[Total Teachers]` |
| Q12 (Resource Distribution) | Line and Stacked Column Chart | X-Axis: `DimLocation[County]` · Column: `[Total Students]` · Line: `[Total Teachers]` |
| Q17 (Classroom Performance Champions) | Table (sorted descending) | Columns: `DimTeacher[Teacher Name]`, `[Avg Score]` |

### Page 4 : Revenue & Financial Health

**KPI Cards:** `[Total Expected Revenue]`, `[Total Revenue Collected]`, `[Total Fee Balance (Outstanding)]`

| Question | Visual | Configuration |
|---|---|---|
| Q23 & Q26 (Financial Position) | Dual KPI Summary Cards | Collection totals vs. outstanding arrears |
| Q24 & Q30 (School Billings & Arrears) | Combined Matrix / Column Chart | X-Axis: `DimSchool[School Name]` · Y-Axis: `[Total Revenue Collected]` and `[Total Fee Balance (Outstanding)]` |
| Q25 (Geographic Revenue Generations) | Clustered Bar Chart | Y-Axis: `DimLocation[County]` · X-Axis: `[Total Revenue Collected]` |
| Q27 (Pending Balances List) | Table Visual | Columns: `DimStudent[Student Name]`, `[Total Fee Balance (Outstanding)]` · Filter: `> 0` |
| Q28 (Average Collection Fee) | Summary Card | `[Avg Fee Paid]` |
| Q29 (Payment Channels) | Pie Chart | Legend: `FactAssessment[Payment Method]` · Values: `[Total Students]` (transaction counts) |


### Page 5 : Location Based Analysis

**KPI Cards:** `[Best Performing County]`, `[Highest Revenue Region]`, `[Best Performing Sub County]`

| Question | Visual | Configuration |
|---|---|---|
| Q31 & Q33 (Advanced Locations) | Cross-tabulated Table | `DimLocation[Sub County]` vs. `[Avg Score]` |
| Q32 (Regional Financial Yield) | Donut Chart | Legend: `DimLocation[Region]` · Values: `[Total Revenue Collected]` |
| Q34 & Q35 (Risk Quadrant Outliers) | Scatter Chart | X-Axis: `[Total Revenue Collected]` (or `[Total Students]`) · Y-Axis: `[Avg Score]` (or `[Avg Attendance]`) · Details: `DimLocation[County]` |
---

## Tools Used

- Power BI Desktop
- Power Query 
- DAX (Data Analysis Expressions)

## Repository Structure

```
├── data/              # Raw and cleaned datasets
├── pbix/              # Power BI project file
├── visuals/           # Dashboard preview images
└── README.md
```

## Feedback & Contributions
If you spot any errors, want to suggest updates to the DAX measures or have ideas to improve the data model, feel free to reach out.
