# Workforce Productivity Report — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `Workforce Productivity Report \| #MakeoverMonday` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/WorkforceProductivityReportMakeoverMonday/Viz |
| Challenge | **#MakeoverMonday Week 13** × **#B2VB Week 6** collaboration |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `WorkforceProductivityReportMakeoverMonday.twbx`, decompiled and parsed field-by-field |
| Explicit exclusion | 5 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic**. Listed in Appendix A. |

---

## 1. Executive summary

**Workforce Productivity Report** compares how five industries spend the working day — productive
time, focus time, collaboration time and break time — alongside a utilisation split of healthy,
over-utilised and under-utilised employees.

It is a consulting-report layout: five industry rows, each carrying a utilisation donut, a
sample-population block, and a four-bar aggregated work-time chart, with a survey-details and
key-takeaways column down the right. The published description names the intent — *"simple,
icons, consulting"*.

The one piece of real engineering is `Total Workspan` (§5.1), which parses a string like
`"8h 30m"` back into a decimal number so the bars can be scaled against a per-industry workday
length that varies by row.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | 13 |
| Dashboards | 1 (`Viz`) |
| Data sources | 1 (single CSV, 9 columns) |
| Parameters | 1 |
| Calculated fields **used** | 7 named + 10 in-view ad-hoc |
| Calculated fields unused | 5 |
| Dashboard actions | **0** |
| Dynamic Zone Visibility bindings | 0 |
| Global font | Calibri |
| Published size | 32,601 bytes (32 KB) |

### 1.2 Headline figures rendered by the dashboard

Survey scope: **Q1–Q2 2024 · 958 companies · 135,098 employees**

| Industry | Employees | Companies | Healthy | Over | Under | Productive | Focus | Collab | Break | Workday |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Cross-Industry | 135,098 | 872 | 66 % | 22 % | 12 % | 6.90 | 6.90 | 4.20 | 0.50 | 8.90 |
| Financial Services | 13,812 | 107 | 72 % | 14 % | 14 % | 7.30 | 6.90 | 4.00 | 0.50 | 9.00 |
| Healthcare | 5,821 | 63 | 62 % | 22 % | 16 % | 7.40 | 7.00 | 4.90 | 0.30 | 8.70 |
| Insurance | 8,698 | 75 | 73 % | 13 % | 14 % | 7.20 | 6.80 | 4.60 | 0.30 | 8.68 |
| Professional Services | 5,668 | 98 | 74 % | 13 % | 13 % | 7.10 | 6.70 | 4.20 | 0.30 | 8.87 |

Note the denominators differ per row — Healthcare's bars scale against an 8.70-hour day,
Financial Services against 9.00 — which is what `Total Workspan` exists to support.

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `16662655` |
| LUID | `85aa8d81-8812-4d36-9833-17e7f000fb6d` |
| Repository URL | `WorkforceProductivityReportMakeoverMonday` |
| Default view | `Viz` |
| Revision | 3.8 |
| First published | 2025-04-01 |
| Last published | 2025-04-01 |
| View count | 928 |
| Favourites | 5 |
| Source | Activtrak |

**Published description**

> Industry Workforce Productivity Report | #MakeoverMonday #B2VB
> #simple #icons #consulting

---

## 3. Data architecture

| Property | Value |
|---|---|
| Data source | `Work productivy Activtrak Report - Sheet1.csv` |
| Relations | 1 table |
| Joins | **None** |
| Data source filters | None |
| Columns | **9** |
| Rows | 5 (one per industry) |

| Column | Type | Note |
|---|---|---|
| `Industry` | string | The five rows |
| `Total Time (hrs)` | **string** | Stored as text, e.g. `"8h 54m"` — parsed by `Total Workspan` |
| `Productive Time (hrs)` | | |
| `Focus Time (hrs)` | | |
| `Collaboration Time (hrs)` | | |
| `Break Time (hrs)` | | |
| `Healthy (%)` · `Underutilized (%)` · `Overutilized (%)` | | The donut segments |

A five-row dataset. Company and employee counts are **not** in the source — they are hard-coded
in calculations (§5.2).

---

## 4. Parameters

| Caption | Type | Domain | Default | Role |
|---|---|---|---|---|
| **Time vs Rate** | string list | `Time` · `Rate` | `Time` | Switches the aggregated chart between absolute hours and percentage rates |

---

## 5. Calculated fields in use

7 named calculations plus 10 in-view ad-hoc.

### 5.1 Parsing the workday string

```
Total Workspan  FLOAT( LEFT([Total Time (hrs)],1) + "."
                     + IF CONTAINS([Total Time (hrs)],'m')
                       THEN STR( ROUND( INT( LEFT(RIGHT([Total Time (hrs)],3),2) )/60*100, 0) )
                       ELSE … END )
```

Used in 7 sheets. The source stores the working day as text (`"8h 54m"`), so this reconstructs a
decimal:

1. `LEFT(…,1)` takes the hour digit → `"8"`
2. `RIGHT(…,3)` then `LEFT(…,2)` extracts the minutes → `"54"`
3. `/60*100` converts minutes to a decimal fraction → `90`
4. Concatenation and `FLOAT()` assemble `8.90`

This is what allows each industry's bars to scale against **its own** workday length rather than
a shared constant — the per-row denominators visible in §1.2.

### 5.2 Hard-coded population figures

```
Employees  IF [Industry] = 'Cross-Industry'      THEN 135098
           ELSEIF [Industry] = 'Financial Services' THEN 13812
           ELSEIF [Industry] = 'Healthcare'         THEN 5821
           ELSEIF [Industry] = 'Insurance'          THEN 8698
           ELSEIF [Industry] = 'Professional Services' THEN 5668 END

Companies  IF [Industry] = 'Cross-Industry'      THEN 872
           ELSEIF [Industry] = 'Financial Services' THEN 107
           ELSEIF [Industry] = 'Healthcare'         THEN 63
           ELSEIF [Industry] = 'Insurance'          THEN 75
           ELSEIF [Industry] = 'Professional Services' THEN 98 END
```

Both used in 5 sheets. The sample sizes appear in the published report but not in the CSV, so
they are encoded directly. These are the figures behind the "Sample Population" column.

### 5.3 Duration display strings

Four fields convert a decimal hour value into `"7h 24m"` display text:

```
Productive Time (hr_m)     Str(INT([Productive Time])) + 'h '
                         + Str( ROUND( ( FLOAT([Productive Time]) - INT([Productive Time]) ) * 60 ) ) + 'm'

Focus Time (hr_m)          same construction
Collaboration Time (hr_m)  same construction
Break Time (hr_m)          same construction
```

The inverse of `Total Workspan`: decimal in, formatted string out.

---

## 6. Worksheet specifications

13 worksheets, in three families of five plus two controls.

| Family | Sheets | Role |
|---|---|---|
| **Utilisation donuts** | `Bar-CI`, `Bar-FS`, `Bar-H`, `Bar-I`, `Bar-PS` | One per industry — the Healthy / Overutilized / Underutilized ring with the healthy percentage at centre |
| **Sample population** | `Population CI`, `Population FS`, `Population H`, `Population I`, `Population PS` | The employee and company counts with icons |
| **Work time** | `Pie` | The four-bar aggregated work-time chart |
| **Controls** | `Rate to Time`, `Time to Rate` | The `Time vs Rate` switch sheets |

The industry suffixes are `CI` Cross-Industry, `FS` Financial Services, `H` Healthcare,
`I` Insurance, `PS` Professional Services.

Building five separate sheets per family rather than one sheet with Industry on rows gives each
row independent formatting — which is what allows the donut centres, icon sizes and bar scales
to be tuned per industry.

---

## 7. Dashboard specification

| Property | Value |
|---|---|
| Name | `Viz` |
| Canvas | 1600 × 900 px, fixed |
| Ground | White |
| Masthead | Deep indigo band |

**Regions**

```
Masthead   "Industry Workforce Productivity Report"
           "Q1-Q2 2024 | Surveyed 958 Companies & 135,098 Employees" · logo mark
Columns    Industry | Utilization % | Sample Population | Aggregated Work Time
Rows       five industry rows, each with a donut, a population block and four bars
Right      Survey Details paragraph · Key Takeaways (four findings, keyword-highlighted)
Footer     "Project Makeover Monday x B2VB Collab | Challenge MM Week 13 x B2VB Week 6 |
            Designed by John Johansson | Source Activtrak"
```

The **Key Takeaways** column is written prose with industry names bolded in the accent colour —
the consulting-report convention the description refers to.

---

## 8. Interactivity

**No actions and no Dynamic Zone Visibility.**

The only control is the `Time vs Rate` parameter, with `Rate to Time` and `Time to Rate` as its
switch sheets. The dashboard is otherwise a static report — appropriate to its five-row dataset
and its stated "simple / consulting" intent.

---

## 9. Design system

| Token | Use |
|---|---|
| Deep indigo | Masthead, industry names, productive-time bars |
| Mid purple | Focus and collaboration bars |
| Pale lavender | Break-time bars, bar tracks |
| Green | Healthy segment of the utilisation donut |
| Red | Overutilized segment |
| Amber | Underutilized segment |
| White | Ground |

The donut uses a **traffic-light convention** (green healthy, amber under, red over) while the
bars use a monochrome purple ramp — so utilisation reads as a judgement and work time reads as
a neutral quantity.

**Typography** — Calibri, set at workbook level.

---

## 10. Rebuild / maintenance runbook

**Refreshing the data**

1. `Employees` and `Companies` are **hard-coded per industry**. New survey figures require
   editing both fields — the CSV does not carry them.
2. `Total Workspan` parses `Total Time (hrs)` on the assumption of a **single-digit hour** —
   `LEFT(…,1)` takes one character. A workday of 10 hours or more would parse as `1.xx`. If the
   data could exceed 9 hours, this field needs widening.
3. The masthead figures (958 companies, 135,098 employees) are static text on the dashboard and
   must be updated alongside the calculations.

**Adding an industry**

1. Add the row to the CSV.
2. Add the `ELSEIF` branch to both `Employees` and `Companies`.
3. Duplicate one `Bar-*` sheet and one `Population *` sheet, filter to the new industry.
4. Add the row to the dashboard's vertical flow.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | Work productivy Activtrak Report - Sheet1 |
| Connection class | `federated` |
| Tables / relations | 1 · joins: 0 |
| Physical columns materialised | 9 |
| Source fields in data pane | 7 |
| Calculated fields (used / total) | 7 / 12 |
| Parameters | 1 |
| Data source filters | 0 (none) |

### Source fields

7 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Utilisation

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Overutilized %**<br>`Overutilized (%)` | integer · Measure (Sum) | 4 | yes | 7 | Share over-utilised — a burnout signal. |
| **Healthy %**<br>`Healthy (%)` | integer · Measure (Sum) | 5 | yes | 6 | Share of employees at healthy utilisation. |
| **Underutilized %**<br>`Underutilized (%)` | integer · Measure (Sum) | 4 | yes | 1 | Share under-utilised. |

#### Time

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Break Time**<br>`Break Time (hrs)` | real · Measure (Sum) | 2 | yes | 6 | Hours on break. |
| **Collaboration Time**<br>`Collaboration Time (hrs)` | real · Measure (Sum) | 4 | yes | 6 | Hours in collaborative work. |
| **Focus Time**<br>`Focus Time (hrs)` | real · Measure (Sum) | 4 | yes | 6 | Hours of uninterrupted focus. |
| **Productive Time**<br>`Productive Time (hrs)` | real · Measure (Sum) | 5 | yes | 6 | Hours classed as productive. |

### Calculated fields in use

7 of the workbook's 12 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| **Break Time (hr_m)**<br>`Productive Time (hr_M) (copy 2)_1100004270913339406` | string · Dimension | Basic | Break Time | Break Time Txt | 1 |
| **Collaboration Time (hr_m)**<br>`Break Time (hr_m) (copy)_1100004270914072591` | string · Dimension | Basic | Collaboration Time | Collaboration Time Txt | 1 |
| **Companies**<br>`Employees (copy)_1100004270934028313` | integer · Measure | Basic | — | — | 5 |
| **Employees**<br>`Calculation_1100004270924922904` | integer · Measure | Basic | — | — | 5 |
| **Focus Time (hr_m)**<br>`Productive Time (hr_m) (copy 2)_1100004270917943314` | string · Dimension | Basic | Focus Time | Focus Time Txt | 1 |
| **Productive Time (hr_m)**<br>`Productive Time (hr_M) (copy)_1100004270912753677` | string · Dimension | Basic | Productive Time | Productive Time Txt | 1 |
| **Total Workspan**<br>`Calculation_1100004270060597253` | real · Dimension | Basic | — | Total Time Txt | 7 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| **Time vs Rate**<br>`Parameter 1` | string · list | `"Time"` | list of 2 — "Time", "Rate" |

## Appendix A — Calculated fields excluded from this documentation

5 calculated fields exist in the data pane but are not referenced by any worksheet or dashboard
logic:

`Collaboration Time Txt` · `Focus Time Txt` · `Total Time Txt` · `Break Time Txt` ·
`Productive Time Txt`

These are earlier text-formatting attempts superseded by the `(hr_m)` family in §5.3.

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas and settings are quoted verbatim from the workbook
definition; rendered figures are read from the published view.*
