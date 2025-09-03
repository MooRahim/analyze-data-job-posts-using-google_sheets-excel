# 📊 Formulas & Calculations Documentation

This document collects all key formulas, Power Query steps, and DAX measures used in the **Data Jobs Market 2023** project.  
It serves as a technical reference for anyone who wants to reproduce or learn from this work.

---

## 🔹 Google Sheets Formulas

### Dynamic ranges & quick stats

**- Count postings per role (with filters):**
```gs
=COUNTIFS(
  jobs[job_title_short], A2,
  jobs[job_schedule_type], type,
  jobs[job_country], country
)
```
**- Median salary per role + country + schedule (ignore blanks & zeros):**
```gs
=MEDIAN(
   IF(
     (jobs[job_title_short] = A2) *
     (jobs[job_country] = country) *
     (ISNUMBER(SEARCH(type, jobs[job_schedule_type]))) *
     (jobs[salary_year_avg] <> 0),
     jobs[salary_year_avg]
   )
)
```
**- Interactive controls:**
```gs
=SORT(FILTER(...), 3, FALSE)       // sort filtered table by salary desc
=UNIQUE(jobs[job_country])         // unique list of countries
```
**- Job schedule parsing:**
```gs
=UNIQUE(jobs[job_schedule_type])   // Returns distinct job schedule strings (e.g., “Full-Time and Contract”, “Part-Time”).
=IFERROR(FILTER(G2, NOT(ISNUMBER(SEARCH("and ", G2)))),"") 
// extract part before "and" (keeps full text if "and" not found).
```

---

## 🔹 Excel Power Query (M) — Key Steps

### 1. Source & Type

  - Connect to dataset
  - Set correct column types (Date, Text, Number, Currency)

### 2. Cleaning

  - Trim/Clean text
  - Remove duplicates

### 3. Transformations

  - Remove brackets/quotes, then split job_skills by comma.
  - Extracted `Month` and `Year` columns from `job_posted_date`.
  - Built `avg_adjusted_salary` by merging `avg_yearly_salary` with converted hourly salaries (hourly × 2080). Missing values were replaced with null.

### 4. Outputs

  - Load to Data Model → build Pivot Tables/Charts:
    - Median/Avg salary by role
    - Median/Avg salary by country
    - Skills frequency
    - Monthly posting trend

---

## 🔹 DAX Measures (Excel Power Pivot)

### Median Salary
```dax
Median Salary :=
MEDIAN(Data_jobs[salary_year_avg])

Median Salary US :=
CALCULATE([Median Salary], Data_jobs[job_country] = "United States")

Median Salary Non-US :=
CALCULATE([Median Salary], Data_jobs[job_country] <> "United States")

Median Salary per Skills :=
CALCULATE(
    [Median Salary],
    CROSSFILTER(Data_jobs[job_id], Data_jobs_skills[job_id], BOTH)
) // In this model, the relationship between Data_jobs and Data_jobs_skills was single-direction.
 // I applied CROSSFILTER with BOTH to make the filter context work in both directions for this calculation.
```
### Postings Count
```dax
Count Jobs :=
DISTINCTCOUNT(Data_jobs[job_id])
```
### Skills Count
```dax
Count Skills :=
COUNT(Data_jobs_skills[job_skills])
```
### Average Skills per Job Posting
```dax
Average Skills per Job :=
DIVIDE([Count Skills], [Count Jobs])
```

---

## 📑 Notes

  - Google Sheets formulas use structured references (`jobs[column]`) because the dataset was converted into a table.
  - Power Query (M) and DAX were created in Excel 365 with Power Pivot.
  - This document is for transparency and reproducibility.

