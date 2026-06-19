# DAX Measures Reference — HR Recruitment Dashboard

All 8 measures used in this Power BI project. Store all measures in a dedicated `_Measures` table.

---

## Group 1 — Core KPIs

### Total Applications
Counts every candidate row in the dataset.
```dax
Total Applications = COUNT(Candidates[CandidateID])
```
**Format:** Whole Number
**Used on:** Page 1 (Overview), Page 2 (Funnel)

---

### Total Hires
Counts only candidates with Status = "Hired".
```dax
Total Hires =
CALCULATE(
    COUNT(Candidates[CandidateID]),
    Candidates[Status] = "Hired"
)
```
**Format:** Whole Number
**Used on:** Page 1, Page 3, Page 5

---

### Hiring Rate %
Percentage of all applicants who were hired. DIVIDE handles zero-division safely.
```dax
Hiring Rate % =
DIVIDE(
    [Total Hires],
    [Total Applications],
    0
) * 100
```
**Format:** Decimal Number (1 decimal place)
**Used on:** Page 1 (Overview)

---

### Avg Days to Hire
Average number of days from application date to hire date, for hired candidates only.
```dax
Avg Days to Hire =
AVERAGEX(
    FILTER(
        Candidates,
        Candidates[Status] = "Hired"
    ),
    Candidates[DaysToHire]
)
```
**Format:** Decimal Number (1 decimal place)
**Used on:** Page 1, Page 3 (Sources)

---

## Group 2 — Source & Cost

### Total Recruitment Cost
Sums the cost-per-hire for every hired candidate using the Recruiters lookup table.
```dax
Total Recruitment Cost =
SUMX(
    FILTER(
        Candidates,
        Candidates[Status] = "Hired"
    ),
    RELATED(Recruiters[CostPerHire_EUR])
)
```
**Format:** Currency (€, 0 decimal places)
**Used on:** Page 1, Page 5 (Recruiters)

> **Note:** RELATED() works here because Candidates and Recruiters are connected via RecruiterID.

---

### Interview Pass Rate %
Percentage of interview rounds that resulted in a Pass.
```dax
Interview Pass Rate % =
DIVIDE(
    CALCULATE(
        COUNT(Interviews[Result]),
        Interviews[Result] = "Pass"
    ),
    COUNT(Interviews[InterviewID]),
    0
) * 100
```
**Format:** Decimal Number (1 decimal place)
**Used on:** Page 2 (Funnel), Page 5 (Recruiters)

---

## Group 3 — Recruiter Performance

### Cost per Hire
Average recruitment cost for one successful hire.
```dax
Cost per Hire =
DIVIDE(
    [Total Recruitment Cost],
    [Total Hires],
    0
)
```
**Format:** Currency (€, 0 decimal places)
**Used on:** Page 5 (Recruiters)

---

### Hires This Month
Month-to-date hire count using time intelligence. Requires the DateTable to be marked as a Date Table.
```dax
Hires This Month =
CALCULATE(
    [Total Hires],
    DATESMTD(DateTable[Date])
)
```
**Format:** Whole Number
**Used on:** Page 1 (Overview)

---

## Date Table (DAX)

Created in Table view using New Table:
```dax
DateTable = CALENDARAUTO()
```

Mark as Date Table: Table Tools → Mark as Date Table → select Date column.

Connect to Candidates: drag `DateTable[Date]` → `Candidates[ApplyDate]`.

---

## Notes

- All measures stored in `_Measures` table (underscore makes it sort to top of field list)
- CALCULATE is the most powerful DAX function — it changes filter context
- DIVIDE always preferred over `/` for safe division
- AVERAGEX and SUMX are iterator functions — they loop row by row through a table
- DATESMTD requires a properly configured Date Table to work
