# Recorded in Literature — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `Recorded in Literature \| B2VB` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/B2VBRecordedinLiterature/CherryBlossoms |
| Challenge | **#B2VB 2024 Week 02** — Build a Scatterplot |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `B2VBRecordedinLiterature.twbx`, decompiled and parsed field-by-field |
| Explicit exclusion | 32 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic** — the workbook was built from a salary-analysis file and inherited its calculation library. See Appendix A. |

---

## 1. Executive summary

**Recorded in Literature** plots the **Kyoto cherry-blossom full-flowering date** for every year
it was recorded in Japanese literature — one of the longest continuous phenological records in
existence, running from roughly **800 AD to the present**.

Each point is a year's peak-bloom date; the vertical axis is month and day, the horizontal axis
is the year. A trend line through the cloud shows the bloom date drifting steadily earlier
across the last two centuries, and a histogram down the left edge gives the distribution of
bloom dates.

The lower panel lists every **literature reference** that supplied a record, and hovering it
filters and highlights the corresponding points — connecting each data point back to the
document that recorded it. That linkage is the workbook's subject: this is a dataset assembled
from diaries, chronicles and court records over twelve centuries.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | 5 |
| Dashboards | 1 (`Cherry Blossoms`) |
| Data sources | 1 (single table, **3 columns**) |
| Parameters | **0** |
| Calculated fields **used** | **3** named + 3 in-view ad-hoc |
| Calculated fields unused (not documented) | 32 |
| Dashboard actions | 3 highlight, 2 filter |
| Dynamic Zone Visibility bindings | 0 |
| Extract rows | **827** |
| Published size | 40,628 bytes (40 KB) |

### 1.2 The source data

| Column | Type | Role |
|---|---|---|
| `Year` | | The year of record — c. 800 to present |
| `DOY` | | Day of year of full flowering |
| `Reference Name` | string | The literary source that recorded it |

827 rows — one per recorded year. The axis in the published view runs from **0800 to 2100**, and
bloom dates span **Mar 27 to May 4**.

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Repository URL | `B2VBRecordedinLiterature` |
| Default view | `Cherry Blossoms` |
| View count | 369 |
| Favourites | 1 |
| Source | **Osaka Prefecture University** |

---

## 3. Data architecture

| Property | Value |
|---|---|
| Data source | `Kyoto Full Flower 7` |
| Relations | 1 table |
| Joins | **None** |
| Data source filters | None |
| Columns | **3** |
| Rows | 827 |

The simplest data model in the portfolio — three columns, one table, no joins and no parameters.

---

## 4. Calculated fields in use

Only **three** named calculations are in use.

### 4.1 Parsing the year

```
Year*  DATE( DATEPARSE( "yyyy", RIGHT([Year], 4) ) )      ← 4 sheets
```

The source `Year` column is text. `RIGHT(…, 4)` takes the four-digit year, `DATEPARSE` converts
it to a date and `DATE` strips the time component — giving a true continuous date axis rather
than a string category. This is what allows the scatterplot's X axis to run smoothly from 800 to
2100 with proportional spacing across twelve centuries.

### 4.2 The bloom-date colour scale

```
Color Scale  IF     [DOY] = #3/26/2000# OR [DOY] = #5/3/2000# THEN 19
             ELSEIF [DOY] = #3/27/2000# OR [DOY] = #5/2/2000# THEN 18
             ELSEIF [DOY] = #3/28/2000# OR [DOY] = #5/1/2000# THEN 17
             ELSEIF …
```

Used in 2 sheets. A **symmetric** scale: each branch pairs an early date with a late date and
assigns them the same value. March 26 and May 3 both score 19; March 27 and May 2 both score 18;
and so on inward.

The effect is that colour encodes **distance from the median bloom date** rather than the date
itself — extremes at both ends read as intense, and typical mid-April blooms read as pale. A
plain sequential ramp on `DOY` would have made early blooms look like "low" values rather than
unusual ones.

The year 2000 is an arbitrary anchor, used only so the literal date comparisons have a year
component.

### 4.3 Constant

```
1  1     ← unit constant, used in 2 sheets
```

---

## 5. Worksheet specifications

5 worksheets.

| Sheet | Role |
|---|---|
| `Scatterplot` | The main chart — `Year*` on columns, month/day on rows, one star-shaped mark per record, with a trend line |
| `Y-Bar` | The histogram down the left edge showing bloom-date distribution |
| `Literature` | The reference list in the lower panel |
| `Book` | The book icon beside the literature list |
| `list1` | Supporting list sheet |

The scatterplot uses a **star/blossom-shaped mark**, which is what gives the cloud its floral
texture rather than the uniformity of plain circles.

---

## 6. Dashboard specification

| Property | Value |
|---|---|
| Name | `Cherry Blossoms` |
| Canvas | 1400 × 800 px, fixed |
| Ground | Pale blue-grey |
| Masthead | Deep pink band |

**Regions**

```
Credit line  "Designed by John Johansson | Source Osaka Prefecture University |
              Project Back to Viz Basics | Challenge 2024 Week 02 - Build a Scatterplot"
Masthead     "Recorded in Literature" in white on deep pink
Instruction  "*Hover over the chart to filter the literature and highlight the related points."
Left         MONTH & DAY axis with the distribution histogram
Main         the scatterplot, 800 → 2100, with its trend line
Lower        "LITERATURE" — every reference name with its recorded dates, book icon at left
```

---

## 7. Interactivity

Five actions, no parameters — the interactivity is entirely selection-driven.

| Caption | Type | Effect |
|---|---|---|
| `Scatter to Bar 1` | highlight | Hovering a point highlights its position in the histogram |
| `Bar to Scatter` | highlight | Hovering a histogram bar highlights the matching points |
| `Scatter Same Literature` | highlight | Hovering a point highlights every other point from the same literary source |
| `Filter Literature` | filter | Filters the literature list to the hovered point's reference |
| `Filter Literature 2` | filter | Second filter action in the same chain |

`Scatter Same Literature` is the one that carries the idea: hovering a single bloom record lights
up every other year recorded in that same document, so the reader can see which chronicle covered
which centuries.

---

## 8. Design system

| Token | Use |
|---|---|
| Deep pink / magenta | Masthead band, trend line, dense marks |
| Pale pink | Histogram bars, sparse marks |
| Pale blue-grey | Dashboard ground |
| White | Chart panel, masthead text |
| Tan | Book icon |

The palette is cherry-blossom pink throughout, with the `Color Scale` field (§4.2) providing
intensity by distance from the median bloom date.

---

## 9. Rebuild / maintenance runbook

**Refreshing the data**

`Year*` assumes the source `Year` column ends with a four-digit year (`RIGHT(…, 4)`). If the
source format changes — a year range, or an appended qualifier — the parse will fail silently
and points will drop off the axis.

**Extending the colour scale**

`Color Scale` enumerates date pairs explicitly, working inward from March 26 / May 3. If future
records fall outside that window, new branches must be added at both ends to preserve the
symmetry — otherwise an unusually early bloom will fall through to the default and lose its
colour.

**The inherited calculation library**

32 of the 35 calculated fields are unused and belong to a **salary analysis** — `Current Avg
Salary`, `Diff C-F Avg Salary`, `Salary Avg-Min Difference` and similar. They came in with the
file this workbook was built from and have no bearing on the cherry-blossom data. A maintainer
should not assume a field is relevant just because it exists; this document covers only the
three that are actually wired in.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | Kyoto Full Flower 7 |
| Connection class | `federated` |
| Tables / relations | 1 · joins: 0 |
| Extract rows | 827 |
| Physical columns materialised | 3 |
| Source fields in data pane | 5 |
| Calculated fields (used / total) | 3 / 35 |
| Parameters | 0 |
| Data source filters | 0 (none) |

### Source fields

5 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Observation

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **DOY** | date · Dimension | 37 | yes | 5 | Day of year on which full flowering was recorded in Kyoto. |

#### Provenance

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Reference Name** | string · Dimension | 223 | yes | 5 | The literary source — diary, chronicle or court record — that recorded the observation. |

#### Election

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Year** | string · Dimension | 827 | yes | 4 | Election year. |

#### Salaries

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Average Player Salary ($)** | integer · Measure | — | — | 0 | Mean MLB player salary for the year, in dollars. *(not used in any sheet)* |
| **Minimum Player Salary ($)** | integer · Measure | — | — | 0 | League-minimum MLB salary for the year. *(not used in any sheet)* |

### Calculated fields in use

3 of the workbook's 35 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| **1**<br>`Calculation_1540231119620120576` | integer · Measure | Basic | — | — | 2 |
| **Color Scale**<br>`Calculation_924082395622207488` | integer · Measure | Basic | DOY | — | 2 |
| **Year***<br>`Calculation_1972576683690348545` | date · Dimension | Basic | Year | — | 4 |

## Appendix A — Calculated fields excluded from this documentation

32 calculated fields exist in the data pane but are not referenced by any worksheet or dashboard
logic. They are inherited from an unrelated salary-analysis workbook:

`0` · `Salary Avg-Min Difference` · `Current Avg Salary` · `Diff C-F Avg Salary` ·
`Diff C-F Avg Direction` · `Diff C-F Year` · `Current Avg-Min Salary` · `First Avg Salary` ·
`Current Min Salary` · `Min Year` · `Salary Avg-Min Direction` · `Diff C-F Min Direction` ·
`Previous Year Diff Avg` · `Previous Year Direction Avg Salary` · `Diff S-F` · and 17 further
salary-comparison helpers.

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas and settings are quoted verbatim from the workbook
definition; rendered figures are read from the published view.*
