# HR Attrition Dashboard — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `HR Attrition Dashboard` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/HRAttritionDashboard_17728329067570/HRAttrition |
| Challenge | `#RWFD` — Real World Fake Data |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `HRAttritionDashboard_17728329067570.twbx`, decompiled and parsed field-by-field |
| Explicit exclusion | 1 calculated field (`Employee Count (copy)`) exists but is not referenced by any worksheet or dashboard logic. |

---

## 1. Executive summary

**HR Attrition Dashboard** profiles employee turnover across a 1,470-person workforce: who
leaves, why, from which roles and levels, and how satisfaction scores differ between leavers
and stayers.

The analytical centrepiece is the **Factor Correlation** matrix in the right-hand panel. Rather
than fixing two dimensions, it lets the reader cross-tabulate **any two of sixteen** employee
attributes against each other — or jump to one of three curated presets. That flexibility is
delivered by six paired `CASE` calculations (§5.4) that resolve the X and Y axes, and their
display names, from parameters.

The published description is a single line — *"Round Corners!"* — which is an accurate summary
of the design intent: every panel, KPI tile, chart element and numeric badge is a rounded
container, and the workbook is a study in that treatment. It is a rebuild of Pradeep Kumar G's
award-winning RWFD dashboard.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | 34 |
| Dashboards | 1 (`HR Attrition`) |
| Data sources | 1 (single CSV, no joins) |
| Parameters | 4 |
| Calculated fields **used** | 37 named + 72 in-view ad-hoc |
| Calculated fields unused | 1 |
| Dashboard actions | 3 URL, 7 parameter |
| Dynamic Zone Visibility bindings | 3 |
| Global font | **Poppins** |
| Published size | 147 KB — the smallest workbook in the portfolio |

### 1.2 Headline figures rendered by the dashboard

| KPI | Value | Satisfaction Score |
|---|---:|---:|
| Total Employees | **1,470** | 2.7 |
| Active Employees | **1,233** | 2.8 |
| Attrition | **237** | 2.5 |
| Attrition Rate | **16.1 %** | |
| Retention Rate | **83.9 %** | |

**Leave Reason** (2024 = 74, 2025 = 163)

| Reason | Count |
|---|---:|
| New Job Opportunity | 53 |
| Not Disclosed | 48 |
| Not Satisfied | 46 |
| Personal | 43 |
| Dislike Work Environment | 37 |
| Retired | 11 |

**Job Level** — L1 143/400 · L2 52/482 · L3 32/186 · L4 5/101 · L5 5/64 (attrition / active)

**Department** — R&D 133/828 · Sales 92/354 · HR 12/51

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `18347297` |
| LUID | `9a7c4a94-4849-4170-a421-edf0f7fa2c96` |
| Repository URL | `HRAttritionDashboard_17728329067570` |
| Default view | `HR Attrition` |
| Revision | 1.4 |
| First published | 2026-03-04 |
| Last published | 2026-03-08 |
| Published size | 150,606 bytes (147 KB) |
| View count | 13,936 |
| Favourites | 70 |
| Attribution | Rebuild of **Pradeep Kumar G** — *HR Attrition Dashboard \| VOTD \| #IIBAwards'22* |

**Published description**

> Round Corners!
> #RWFD

---

## 3. Data architecture

| Property | Value |
|---|---|
| Data source | `HR_Attrition.csv` |
| Relations | 1 table |
| Joins | **None** |
| Custom SQL | None |
| Data source filters | None |
| Columns | 37 |

### 3.1 Grain

One row per employee. `Employee Number` is the key; `Employee Count` is a constant 1 per row,
so headcount is `SUM([Employee Count])` and distinct-employee measures use
`COUNTD([Employee Number])`.

### 3.2 Schema (37 columns)

| Group | Columns |
|---|---|
| **Identity** | `Employee Number`, `Employee Count`, `Over18`, `Standard Hours` |
| **Demographics** | `Age`, `Gender`, `Marital Status`, `Distance From Home` |
| **Education** | `Education`, `Education Field` |
| **Role** | `Department`, `Job Role`, `Job Level`, `Business Travel`, `Over Time` |
| **Tenure** | `Total Working Years`, `Years At Company`, `Years In Current Role`, `Years Since Last Promotion`, `Years With Curr Manager`, `Num Companies Worked` |
| **Compensation** | `Monthly Income`, `Monthly Rate`, `Daily Rate`, `Hourly Rate`, `Percent Salary Hike`, `Stock Option Level` |
| **Satisfaction** | `Environment Satisfaction`, `Job Satisfaction`, `Relationship Satisfaction`, `Job Involvement`, `Work Life Balance`, `Performance Rating` |
| **Outcome** | `Attrition`, `Attrition Date`, `Training Times Last Year` |
| **Synthetic** | `Random Number` — used to derive leave reasons (§5.3) |

---

## 4. Parameters

| Caption | Type | Domain | Default | Role |
|---|---|---|---|---|
| **Select X** | string list | 16 members, `A`–`P`, aliased to attribute names | `F` (Annual Income) | X axis of the correlation matrix |
| **Select Y** | string list | 16 members, same domain | `D` (Education Field) | Y axis of the correlation matrix |
| **Correlation Radio** | string list | `Manual` · `Preset 1` · `Preset 2` · `Preset 3` | `Manual` | Preset selector |
| **Time/Reason** | string list | `T` (Monthly Attrition) · `R` (Leave Reason) | `R` | Attrition-summary chart mode |

The 16 attributes available on both axes: Gender, Age Group, Education, Education Field,
Marital Status, Annual Income, Department, Job Level, plus eight further grouped attributes
(Job Role, Distance From Home Group, Total Working Years Group, Year At Company Group, Years In
Current Role Group, Years Since Last Promotion Group, Years With Curr Manager Group, Leave
Reason).

---

## 5. Calculated fields in use

37 named calculations plus 72 in-view ad-hoc.

### 5.1 Core attrition measures

```
Attrition Count    SUM(IF [Attrition] = 'Yes' THEN 1 ELSE 0 END)        ← 16 sheets
Current Count      SUM(IF [Attrition] = 'No'  THEN 1 ELSE 0 END)
Current Employees  SUM([Employee Count]) - ([Attrition Count])
Attrition Rate     ([Attrition Count]) / SUM([Employee Count])
Active Rate        (SUM([Employee Count]) - ([Attrition Count])) / SUM([Employee Count])
Status             IF [Attrition] = 'Yes' THEN 'Attrition' ELSE 'Current' END   ← 11 sheets
```

`Status` is the workbook's primary colour dimension — the navy/grey split that runs through
every demographic bar on the dashboard.

### 5.2 The satisfaction composite

```
Survey Score 0-4   ( SUM([Environment Satisfaction] + [Relationship Satisfaction]
                        + [Job Satisfaction] + [Job Involvement] + [Work Life Balance]) / 5 )
                   / COUNTD([Employee Number])
```

Averages the five satisfaction sub-scores into one index, then divides by distinct employees to
return a per-person mean. This produces the 2.7 / 2.8 / 2.5 figures beside each KPI — and the
gap between leavers (2.5) and stayers (2.8) is the dashboard's central finding.

### 5.3 Derived leave reason

```
Leave Reason   IF [Attrition] != 'Yes' THEN 'Active Employee'
               ELSEIF [Age] >= 55 THEN 'Retired'
               ELSEIF RIGHT(STR([Random Number]),1) = '1' THEN 'Not Satisfied'
               ELSEIF RIGHT(STR([Random Number]),1) = '2' THEN …
               …
```

The source data records *that* an employee left but not *why*. This field synthesises a reason:
anyone 55 or over is classed as Retired, and everyone else is assigned a reason from the last
digit of the `Random Number` column. This is consistent with the `#RWFD` (Real World Fake Data)
framing — the reasons are plausible fabrications, not observations, and should be read as
illustrative.

```
Attrition Date+3   DATE(DATEADD('year', 3, [Attrition Date]))
Year               YEAR([Attrition Date+3])
```

Shifts the attrition dates forward three years so the chart reads 2024–2025 rather than the
source file's original window.

### 5.4 The correlation matrix engine

Six paired `CASE` calculations resolve the matrix axes. Each axis has a **value** field and a
**name** field, and each of those has a **manual** and a **preset-aware** layer.

**Layer 1 — resolve the manual selection**
```
X   CASE [Select X] WHEN 'A' THEN [Gender] WHEN 'B' THEN [Age Group]
                    WHEN 'C' THEN [Education] WHEN 'D' THEN [Education Field]
                    WHEN 'E' THEN [Marital Status] WHEN 'F' THEN [Annual Income] … END
Y   CASE [Select Y] … END   (same 16 branches)

X1 Name  CASE [Select X] WHEN 'A' THEN 'Gender' WHEN 'B' THEN 'Age Group' … END
Y1 Name  CASE [Select Y] … END
```

**Layer 2 — override with a preset**
```
X2  CASE [Correlation Radio]
      WHEN 'Manual'   THEN [X]
      WHEN 'Preset 1' THEN [Yearly Salary Group]
      WHEN 'Preset 2' THEN [Total Working Years Group]
      WHEN 'Preset 3' THEN [Age Group] END

Y2  CASE [Correlation Radio]
      WHEN 'Manual'   THEN [Y]
      WHEN 'Preset 1' THEN [Leave Reason]
      WHEN 'Preset 2' THEN [Department]
      WHEN 'Preset 3' THEN [Job Role] END

X2 Name / Y2 Name   same structure, returning the label text
```

The two-layer design is what allows presets and manual selection to coexist: the preset layer
falls through to the manual layer when `Manual` is selected, so the reader can pick a preset,
then switch back to Manual and find their own choices still in place.

**The three presets**

| Preset | X | Y |
|---|---|---|
| Preset 1 | Annual Income | Leave Reason |
| Preset 2 | Years of Experience | Department |
| Preset 3 | Age Group | Job Role |

### 5.5 Banding fields

Six fields bucket continuous attributes into ordinal bands, all following one template:

```
Total Working Years Group        IF [Total Working Years] <= 2 THEN '0-2 Years'
                                 ELSEIF <= 5 THEN '3-5 Years'
                                 ELSEIF <= 10 THEN '6-10 Years' ELSEIF …
Year At Company Group            same thresholds on [Years At Company]
Years In Current Role Group      same on [Years In Current Role]
Years Since Last Promotion Group same on [Years Since Last Promotion]
Years With Curr Manager Group    same on [Years With Curr Manager]
Distance From Home Group         IF <= 2 THEN '0-2 Miles' ELSEIF <= 5 THEN '3-5 Miles'
                                 ELSEIF <= 10 THEN '6-10 Miles' ELSEIF …
```

```
Annual Income        [Monthly Income] * 12
Yearly Salary Group  IF [Annual Income] < 50000 THEN '<$50K'
                     ELSEIF < 100000 THEN '$50-100K'
                     ELSEIF < 150000 THEN '$100-150K'
                     ELSEIF < 200000 THEN '$150-200K' ELSE '$200K+' END
```

Using one consistent band scheme (0-2 / 3-5 / 6-10 / …) across all five tenure attributes is
what makes them interchangeable on the correlation axes.

### 5.6 View-state booleans

| Field | Formula | Controls |
|---|---|---|
| **Time T\|F** | `[Time/Reason] = 'T'` | The `Time` zone (monthly attrition chart) |
| **Reason T\|F** | `[Time/Reason] = 'R'` | The `Reason` zone (leave-reason bars) |
| **Radio Hide T\|F** | `CASE [Correlation Radio] WHEN 'Manual' THEN True WHEN 'Preset 1' THEN False … END` | Hides the X/Y selector row whenever a preset is active |
| **Radio M/P1/P2/P3 T\|F** | `[Correlation Radio] = '<value>'` | Button colour on the four preset controls |

`Radio Hide T|F` is a nice touch: the manual X and Y pickers disappear when a preset is chosen,
so the interface only ever offers the controls that currently apply.

### 5.7 Helpers

`A_Color_Icons` = `'A'`, `B_Color_Icons` = `'B'`, `C_Color_Icons` = `'A'` — single-member
dimensions forcing fixed icon colours. `Gender Abb` aliases `Gender` for a second use.

---

## 6. Worksheet specifications

34 worksheets.

| Group | Sheets | Construction |
|---|---|---|
| **KPI tiles** | `KPI: Total Employees`, `KPI: Active Employees`, `KPI: Total Attrition`, `KPI: Attrition Rate`, `KPI: Retention Rate` | Text marks in rounded containers |
| **Satisfaction scores** | `Score: All Employees`, `Score: Active Employees`, `Score: Attrition Employees` | `Survey Score 0-4` per population |
| **Demographics row** | `Gender`, `Age Group`, `Marriage Status`, `Education`, `Education Field`, `Annual Income` | Horizontal bars split by `Status`; Gender uses circular avatar marks |
| **Attrition summary** | `Reason`, `Time`, `Time Sum`, `Text` | Two mutually exclusive charts controlled by `Time/Reason` |
| **Job details** | `Job Level`, `Department`, `Job Role` | Job Level as five numeric tiles; Department as three donuts; Job Role as a ranked list |
| **Correlation** | `Correlation` | The cross-tab matrix on `X2` × `Y2`, coloured by attrition count |
| **Controls** | `Manual`, `P1`, `P2`, `P3`, `T1`, `T2` | Button sheets, coloured by their matching boolean |
| **Chrome** | `LI`, `TP`, `RWFD` | Icon links |
| **Unused** | `Sheet 18`, `Sheet 20`, `Sheet 23` | Scratch sheets not placed on the dashboard |

---

## 7. Dashboard specification

| Property | Value |
|---|---|
| Name | `HR Attrition` |
| Canvas | 1600 × 926 px, fixed |
| Ground | Light grey |
| Masthead | Navy `#1f3864`-family band with a curved right edge |
| Panels | White rounded cards |

**Regions**

```
Masthead        "HR ATTRITION DASHBOARD" · LinkedIn · Tableau · RWFD icons
Demographics    six attribute breakdowns across one row, with an Attrition/Retention legend
Attrition Summary   leave-reason bars (or monthly trend) + 3 KPI rows with satisfaction scores
Job Details     Job Level tiles · Department donuts · Job Role ranked list
Factor Correlation  preset buttons · X/Y pickers · the cross-tab matrix
```

The rounded-corner treatment named in the description is applied at every level: the masthead's
curved edge, the white panel cards, the KPI pills, the numeric badges on every bar, and the
donut rings in Job Details.

---

## 8. Interactivity

### 8.1 Parameter actions (7)

| Caption | Source sheet | Payload | Target |
|---|---|---|---|
| `Manual Default` | `Manual` | `'Manual'` | **Correlation Radio** |
| `X- Manual Default` | `Manual` | `'F'` | **Select X** |
| `Y-P1` | `P1` | `'Preset 1'` | **Correlation Radio** |
| `Y-P2` | `P2` | `'Preset 2'` | **Correlation Radio** |
| `Y-P3` | `P3` | `'Preset 3'` | **Correlation Radio** |
| `Monthly` | `T1` | `'T'` | **Time/Reason** |
| `Reason` | `T2` | `'R'` | **Time/Reason** |

Note that the `Manual` button fires **two** actions — it resets the radio to Manual *and* resets
`Select X` to `F` (Annual Income), so returning from a preset lands on a known good state rather
than whatever the preset left behind.

### 8.2 URL actions (3)

| Caption | Target |
|---|---|
| `LI` | `https://www.linkedin.com/in/johnsjohansson/` |
| `TP` | `https://public.tableau.com/app/profile/john.johansson/vizzes` |
| `RWFD` | `https://sonsofhierarchies.com/real-world-fake-data/` |

### 8.3 Dynamic Zone Visibility (3 bindings)

| Field | Zone |
|---|---|
| `Time T\|F` | `Time` — the monthly attrition trend |
| `Reason T\|F` | `Reason` — the leave-reason bars |
| `Radio Hide T\|F` | The X/Y picker row |

---

## 9. Design system

| Token | Use |
|---|---|
| Navy | Masthead, attrition bars, KPI accents, matrix high values |
| Mid grey | Retention/current bars, the "Retention" legend swatch |
| Light grey | Dashboard ground |
| White | Panel cards |
| Pale blue → navy | The correlation matrix's sequential ramp |

The two-colour convention — **navy = attrition, grey = retention** — is applied to every
demographic bar, so the reader learns it once in the legend and reads it everywhere.

**Typography** — **Poppins**, set at workbook level. This is the only workbook in the portfolio
not using Arial, and the geometric sans is what gives the dashboard its distinct, softer feel
alongside the rounded corners.

---

## 10. Rebuild / maintenance runbook

**Refreshing the data**

1. Replace `HR_Attrition.csv`, preserving the `Random Number` column — `Leave Reason` derives
   from its last digit and will fail without it.
2. Check `Attrition Date+3`. The three-year shift is a presentation choice that moves the source
   dates into a 2024–2025 window; adjust or remove it if the new data already sits in the
   desired period.

**Adding an attribute to the correlation matrix**

Six fields must be edited as a set, or the axes and their labels will disagree:

1. `X` and `Y` — add the new `WHEN` branch (same letter code in both).
2. `X1 Name` and `Y1 Name` — add the matching label branch.
3. Add the letter code to both the `Select X` and `Select Y` parameter domains, with an alias.
4. If the attribute is continuous, add a `… Group` banding field first, following the existing
   0-2 / 3-5 / 6-10 template so it stays interchangeable with the other tenure bands.

`X2` / `Y2` and their name fields need no change — they fall through to the manual layer.

**Adding a preset**

1. Add a member to `Correlation Radio`.
2. Add the `WHEN` branch to all four of `X2`, `Y2`, `X2 Name`, `Y2 Name`.
3. Add a `Radio P4 T|F` boolean and a button sheet coloured by it.
4. Add a parameter action from the button.
5. Extend `Radio Hide T|F` with the new value returning `False`.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | HR_Attrition |
| Connection class | `federated` |
| Tables / relations | 1 · joins: 0 |
| Physical columns materialised | 37 |
| Source fields in data pane | 35 |
| Calculated fields (used / total) | 37 / 38 |
| Parameters | 4 |
| Data source filters | 0 (none) |

### Source fields

35 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Identity

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Employee Count**<br>`EmployeeCount` | integer · Measure (Sum) | 1 | yes | 17 | Constant 1 per row; `SUM` gives headcount. |
| **Employee Number**<br>`EmployeeNumber` | integer · Dimension | 1,470 | yes | 5 | Unique employee key. Basis of distinct headcount. |
| **Standard Hours**<br>`StandardHours` | integer · Measure (Sum) | 1 | yes | 0 | Constant 80. No analytical use. *(not used in any sheet)* |

#### Satisfaction

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Environment Satisfaction**<br>`EnvironmentSatisfaction` | integer · Measure (Sum) | 4 | yes | 5 | Satisfaction with the work environment, 1–4. |
| **Job Involvement**<br>`JobInvolvement` | integer · Measure (Sum) | 4 | yes | 4 | Degree of job involvement, 1–4. |
| **Job Satisfaction**<br>`JobSatisfaction` | integer · Measure (Sum) | 4 | yes | 4 | Satisfaction with the job itself, 1–4. |
| **Relationship Satisfaction**<br>`RelationshipSatisfaction` | integer · Measure (Sum) | 4 | yes | 4 | Satisfaction with workplace relationships, 1–4. |
| **Work Life Balance**<br>`WorkLifeBalance` | integer · Measure (Sum) | 4 | yes | 4 | Work-life balance rating, 1–4. |
| **Performance Rating**<br>`PerformanceRating` | integer · Measure (Sum) | 2 | yes | 0 | Performance rating, 1–4. *(not used in any sheet)* |
| **Training Times Last Year**<br>`TrainingTimesLastYear` | integer · Dimension | 7 | yes | 0 | Training sessions attended in the last year. *(not used in any sheet)* |

#### Demographics

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Age** | integer · Measure (Sum) | 43 | yes | 4 | Employee age in years. |
| **Age Group** | string · Dimension | — | — | 2 | Age banded for display, replacing the continuous Age value on categorical axes. |
| **Marital Status**<br>`MaritalStatus` | string · Dimension | 3 | yes | 2 | Single, Married or Divorced. |
| **Distance From Home**<br>`DistanceFromHome` | integer · Measure (Sum) | 29 | yes | 1 | Commute distance in miles. Banded into 0-2 / 3-5 / 6-10 / … |

#### Synthetic

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Random Number** | real · Dimension | 199 | yes | 3 | Random value carried in the source. Its last digit derives the synthetic leave reason — the data records *that* someone left, not why. |

#### Compensation

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Monthly Income**<br>`MonthlyIncome` | integer · Measure (Sum) | 1,181 | yes | 2 | Gross monthly salary. `Annual Income` multiplies this by 12. |
| **Daily Rate**<br>`DailyRate` | integer · Measure (Sum) | 798 | yes | 0 | Daily rate. *(not used in any sheet)* |
| **Hourly Rate**<br>`HourlyRate` | integer · Measure (Sum) | 71 | yes | 0 | Hourly rate. *(not used in any sheet)* |
| **Monthly Rate**<br>`MonthlyRate` | integer · Measure (Sum) | 1,267 | yes | 0 | Monthly billing or cost rate. *(not used in any sheet)* |
| **Percent Salary Hike**<br>`PercentSalaryHike` | integer · Measure (Sum) | 15 | yes | 0 | Percentage of the most recent salary increase. *(not used in any sheet)* |
| **Stock Option Level**<br>`StockOptionLevel` | integer · Measure (Sum) | 4 | yes | 0 | Stock option tier, 0–3. *(not used in any sheet)* |

#### Role

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Department** | string · Dimension | 3 | yes | 2 | Department: Research & Development, Sales, Human Resources. |
| **Job Level**<br>`JobLevel` | integer · Measure (Sum) | 5 | yes | 2 | Seniority band 1–5. |
| **Job Role**<br>`JobRole` | string · Dimension | 9 | yes | 2 | Specific job title — Laboratory Technician, Sales Executive, Research Scientist and so on. |
| **Department Abbv** | string · Dimension | — | — | 1 | Abbreviated department name, for labels too narrow to carry the full text. |
| **Business Travel**<br>`BusinessTravel` | string · Dimension | 3 | yes | 0 | Travel frequency — Non-Travel, Travel_Rarely, Travel_Frequently. *(not used in any sheet)* |
| **Over Time**<br>`OverTime` | string · Dimension | 2 | yes | 0 | Whether the employee works overtime: Yes or No. *(not used in any sheet)* |

#### Education

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Education Field**<br>`EducationField` | string · Dimension | 6 | yes | 2 | Field of study — Life Sciences, Medical, Marketing, Technical Degree, Human Resources, Other. |

#### Outcome

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Attrition Date** | date · Dimension | 112 | yes | 2 | Date of departure. Shifted forward three years by `Attrition Date+3` for presentation. |

#### Tenure

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Total Working Years**<br>`TotalWorkingYears` | integer · Measure (Sum) | 40 | yes | 1 | Total career experience in years, across all employers. |
| **Years At Company**<br>`YearsAtCompany` | integer · Measure (Sum) | 37 | yes | 1 | Years with this employer. |
| **Years In Current Role**<br>`YearsInCurrentRole` | integer · Measure (Sum) | 19 | yes | 1 | Years in the present role. |
| **Years Since Last Promotion**<br>`YearsSinceLastPromotion` | integer · Measure (Sum) | 16 | yes | 1 | Years since the last promotion — a common attrition signal. |
| **Years With Curr Manager**<br>`YearsWithCurrManager` | integer · Measure (Sum) | 18 | yes | 1 | Years reporting to the current manager. |
| **Num Companies Worked**<br>`NumCompaniesWorked` | integer · Measure (Sum) | 10 | yes | 0 | Number of previous employers. *(not used in any sheet)* |

### Calculated fields in use

37 of the workbook's 38 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| **A_Color_Icons**<br>`Calculation_1642440910688283` | string · Dimension | Basic | — | — | 6 |
| **Active Rate**<br>`Current Employees 2 (copy)_1617425002397702` | real · Measure | Basic | Employee Count, Attrition Count | — | 1 |
| **Annual Income**<br>`Calculation_1091858945314821` | integer · Measure | Basic | Monthly Income | Yearly Salary Group | 2 |
| **Attrition Count**<br>`Calculation_1617425000976386` | integer · Measure | Basic | — | Active Rate, Attrition Rate, Current Employees | 16 |
| **Attrition Date+3**<br>`Attrition Date (copy)_0948649584300032` | date · Dimension | Basic | Attrition Date | Year | 2 |
| **Attrition Rate**<br>`Current Employees (copy)_1617425002070021` | real · Measure | Basic | Attrition Count, Employee Count | — | 1 |
| **B_Color_Icons**<br>`A_Color_Icons (copy)_1642440912261148` | string · Dimension | Basic | — | — | 3 |
| **C_Color_Icons**<br>`A_Color_Icons (copy)_0088558196187136` | string · Dimension | Basic | — | — | 2 |
| **Current Count**<br>`Attrition Count (copy)_0000593406160896` | integer · Measure | Basic | — | — | 1 |
| **Current Employees**<br>`Calculation_1617425001517059` | integer · Measure | Basic | Employee Count, Attrition Count | — | 1 |
| **Distance From Home Group**<br>`Calculation_0549525946163204` | string · Dimension | Basic | Distance From Home | X, Y | 1 |
| **Gender Abb**<br>`Gender (copy)_1642440900784151` | string · Dimension | Basic | — | — | 1 |
| **Leave Reason**<br>`Calculation_1106703399645184` | string · Dimension | Basic | Age, Random Number | X, Y, Y2 | 3 |
| **Radio Hide T\|F**<br>`Calculation_1450301716557824` | boolean · Dimension | Basic | — | — | 1 |
| **Radio M T\|F**<br>`Calculation_1642440914006049` | boolean · Dimension | Basic | — | — | 1 |
| **Radio P1 T\|F**<br>`Radio O T\|F (copy)_1642440914137122` | boolean · Dimension | Basic | — | — | 1 |
| **Radio P2 T\|F**<br>`Radio P1 T\|F (copy)_1642440914194467` | boolean · Dimension | Basic | — | — | 1 |
| **Radio P3 T\|F**<br>`Radio P1 T\|F (copy) (copy)_1642440914198564` | boolean · Dimension | Basic | — | — | 1 |
| **Reason T\|F**<br>`Time T\|F (copy)_0948649631756318` | boolean · Dimension | Basic | — | — | 2 |
| **Status**<br>`Attrition Count (copy)_0000593407406083` | string · Dimension | Basic | — | — | 11 |
| **Survey Score 0-4**<br>`Calculation_0549525950697478` | real · Measure | Basic | Environment Satisfaction, Relationship Satisfaction, Job Satisfaction, Job Involvement, Work Life Balance, Employee Number | — | 4 |
| **Time T\|F**<br>`Calculation_0948649631645725` | boolean · Dimension | Basic | — | — | 2 |
| **Total Working Years Group**<br>`Total Working Years (copy)_0549525947383813` | string · Dimension | Basic | Total Working Years | X, X2, Y | 1 |
| **X**<br>`Calculation_1984388068917248` | string · Dimension | Basic | Age Group, Education Field, Marital Status, Yearly Salary Group, Department, Job Level … | X2 | 1 |
| **X1 Name**<br>`X (copy)_0948649628651545` | string · Dimension | Basic | — | X2 Name | 1 |
| **X2**<br>`Calculation_0948649592860692` | string · Dimension | Basic | X, Yearly Salary Group, Total Working Years Group, Age Group | — | 1 |
| **X2 Name**<br>`X2 (copy)_0948649595101206` | string · Dimension | Basic | X1 Name | — | 1 |
| **Y**<br>`Calculation_1984388069044225` | string · Dimension | Basic | Age Group, Education Field, Marital Status, Yearly Salary Group, Department, Job Level … | Y2 | 1 |
| **Y1 Name**<br>`X1 Name (copy)_0948649629425690` | string · Dimension | Basic | — | Y2 Name | 1 |
| **Y2**<br>`X2 (copy)_0948649593319445` | string · Dimension | Basic | Y, Leave Reason, Department, Job Role | — | 1 |
| **Y2 Name**<br>`Y2 (copy)_0948649595551767` | string · Dimension | Basic | Y1 Name | — | 1 |
| **Year**<br>`Calculation_0948649587118081` | integer · Dimension | Basic | Attrition Date+3 | — | 1 |
| **Year At Company Group**<br>`Calculation_0549525943414784` | string · Dimension | Basic | Years At Company | X, Y | 1 |
| **Yearly Salary Group**<br>`Calculation_1091858945773574` | string · Dimension | Basic | Annual Income | X, X2, Y | 2 |
| **Years In Current Role Group**<br>`Years In Current Role (copy)_0549525944385537` | string · Dimension | Basic | Years In Current Role | X, Y | 1 |
| **Years Since Last Promotion Group**<br>`Years Since Last Promotion (copy)_0549525945372675` | string · Dimension | Basic | Years Since Last Promotion | X, Y | 1 |
| **Years With Curr Manager Group**<br>`Years With Curr Manager (copy)_0549525944815618` | string · Dimension | Basic | Years With Curr Manager | X, Y | 1 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| **Select X**<br>`Parameter 1` | string · list | `"F"` | list of 16 — "A" → Gender, "B" → Age Group, "C" → Education, "D" → Education Field, "E" → Maritial Status, "F" → Annual Income |
| **Correlation Radio**<br>`Parameter 2` | string · list | `"Manual"` | list of 4 — "Manual" → Manual, "Preset 1" → Preset 1, "Preset 2" → Preset 2, "Preset 3" → Preset 3 |
| **Time/Reason**<br>`Parameter 3` | string · list | `"R"` | list of 2 — "T" → Monthly Attrition, "R" → Leave Reason |
| **Select Y**<br>`Select X (copy)_1984388071624706` | string · list | `"D"` | list of 16 — "A" → Gender, "B" → Age Group, "C" → Education, "D" → Education Field, "E" → Maritial Status, "F" → Annual Income |

## Appendix A — Calculated field excluded from this documentation

`Employee Count (copy)` — present in the data pane, not referenced by any worksheet or dashboard
logic.

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas and settings are quoted verbatim from the workbook
definition; rendered figures are read from the published view.*
