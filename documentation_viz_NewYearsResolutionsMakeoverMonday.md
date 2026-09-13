# New Year's Resolutions — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `New Year's Resolutions \| #MakeoverMonday` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/NewYearsResolutionsMakeoverMonday_17054389050480/NewYearsResolutions |
| Challenge | **#MakeoverMonday** |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `NewYearsResolutionsMakeoverMonday_17054389050480.twbx`, decompiled and parsed field-by-field |
| Explicit exclusion | 2 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic**: `Total (M/F)` and `Total (Age)`. |

---

## 1. Executive summary

**New Year's Resolutions** compares what UK men and women resolved to do in 2024, across
fourteen resolutions, using a **dumbbell chart**: for each resolution, three lettered markers —
**M** (male), **A** (all) and **F** (female) — sit on a shared percentage axis, joined by a
connecting band whose width is the gender gap.

The design decision that carries the chart is placing **A between M and F**: because "all" always
falls between the two gendered figures, the band reads as a range and the marker order itself
shows which gender leads. Where M and F nearly coincide — *Spending less time on social media*
at 21 % / 21 % — the three markers collapse into one cluster, and the absence of a band is the
finding.

A four-column table on the left prints the underlying percentages: All, Male, Diff, Female.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | **2** |
| Dashboards | 1 (`New Year's Resolutions`) |
| Data sources | 1 (single table, 8 columns) |
| Parameters | **0** |
| Calculated fields **used** | 5 named + 5 in-view ad-hoc |
| Calculated fields unused | 2 |
| Dashboard actions | **0** |
| Dynamic Zone Visibility bindings | 0 |
| Rows | **14** |
| Global font | Lucida Sans |
| Published size | 13,626 bytes (13 KB) — the smallest workbook in the portfolio |

### 1.2 As rendered

UK 2024, ordered by overall popularity:

| Resolution | All | Male | Diff | Female |
|---|---:|---:|---:|---:|
| Doing more exercise or improving fitness | 56 % | 49 % | 12 % | 61 % |
| Saving more money | 49 % | 43 % | 9 % | 52 % |
| Losing weight | 45 % | 43 % | 4 % | 47 % |
| Improving my diet | 42 % | 41 % | 2 % | 43 % |
| Spending less time on social media | 21 % | 21 % | 0 % | 21 % |
| Spending more time with my family | 20 % | 20 % | 1 % | 21 % |
| Pursuing a career ambition | 20 % | 26 % | 10 % | 16 % |
| Taking up a new hobby | 19 % | 21 % | 3 % | 18 % |
| Something else | 18 % | 16 % | 3 % | 19 % |
| Decorating or renovating part of my home | 15 % | 9 % | 9 % | 18 % |
| Volunteering or doing more charity work | 14 % | 18 % | 7 % | 11 % |
| Cutting down on drinking | 12 % | 13 % | 2 % | 11 % |
| Raising money for a charity | 6 % | 10 % | 6 % | 4 % |
| Giving up smoking | 6 % | 4 % | 4 % | 8 % |

Women lead on fitness, saving, weight and diet; men lead on career ambition, charity work and
new hobbies.

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Repository URL | `NewYearsResolutionsMakeoverMonday_17054389050480` |
| Default view | `New Year's Resolutions` |
| View count | 453 |
| Favourites | 3 |
| Source | **YouGov** |

---

## 3. Data architecture

| Property | Value |
|---|---|
| Data source | `Sheet1` |
| Relations | 1 table |
| Joins | **None** |
| Data source filters | None |
| Columns | **8** |
| Rows | **14** |

| Column | Role |
|---|---|
| `Resolution` | The fourteen rows |
| `Male` · `Female` · `All Gender` | The three plotted percentages |
| `18-24` · `25-49` · `50-64` · `65+` | Age breakdowns — present in the data, not used in this view |

Fourteen rows and eight columns. The four age columns are carried but unplotted, which is what
the two unused `Total (…)` calculations were for.

---

## 4. Calculated fields in use

Five named calculations — this is the whole engine.

### 4.1 The three markers

```
M  [Male]
F  [Female]
A  [All]
```

Three one-line aliases. They exist so the three values can be placed on a shared axis as
**Measure Values**, with `Measure Names` supplying the M / A / F letter that becomes each
marker's label. Without the aliases the markers would carry the source column names.

### 4.2 The gap

```
Difference  ABS([Male] - [Female])     ← 2 sheets
```

The absolute gender gap, printed in the **Diff** column and used to size the connecting band. The
absolute value is deliberate — the band's *width* shows how far apart the genders are, and its
direction is already visible from whether M sits left or right of F.

### 4.3 The direction

```
M-to-F Win  IF [Male] > [Female] then 1
            ELSEIF [Female] > [Male] then -1
            ELSE 0 END
```

A three-state direction flag: +1 where men lead, −1 where women lead, 0 where they tie. This
drives the band's colour, so the reader can see at a glance which resolutions skew male and which
skew female — and *Spending less time on social media*, the single tie, gets the neutral state.

---

## 5. Worksheet specifications

**Two worksheets.**

| Sheet | Role |
|---|---|
| `Gender Chart` | The dumbbell chart — Resolution on rows, Measure Values on columns, M/A/F markers and the connecting band |
| `Gender Text` | The four-column table at left: All, Male, Diff, Female, each value in a coloured pill |

---

## 6. Dashboard specification

| Property | Value |
|---|---|
| Name | `New Year's Resolutions` |
| Canvas | 1400 × 800 px, fixed |
| Ground | Cream |

**Regions**

```
Masthead   "New Year's Resolutions" in heavy dark type
           "UK 2024 | Male vs Female" with Male in teal and Female in coral
           right: "#MakeoverMonday | Source YouGov | Designed by John Johansson"
Left       four columns of coloured value pills — All · Male · Diff · Female
Centre     resolution labels
Right      the dumbbell chart, 0 % – 70 % axis with dotted gridlines
```

The subtitle is doing double duty as the legend: "Male" is printed in the male marker's colour
and "Female" in the female marker's, so no separate legend is needed.

---

## 7. Interactivity

**None.** No actions, no parameters, no Dynamic Zone Visibility. A static chart — appropriate to
a fourteen-row dataset.

---

## 8. Design system

| Token | Use |
|---|---|
| Cream | Dashboard ground |
| Deep brown `#503d2e` | All text and gridlines — set as a global colour |
| Magenta / plum | "All" pills and the A marker |
| Teal | "Male" pills and the M marker |
| Coral / salmon | "Female" pills and the F marker |
| Amber | "Diff" pills |
| Pale peach | The connecting band |

**Global formatting**

```
gridline  stroke-color #503d2e, line-pattern-only dotted
all       font-family Lucida Sans, color #503d2e
```

Both the font and the text colour are set at workbook level. The dotted brown gridlines against
cream give the chart a printed-almanac feel that suits the subject.

---

## 9. Rebuild / maintenance runbook

**Refreshing the data**

Replace the 14-row source, keeping the `Male`, `Female` and `All Gender` column names — the three
aliases (`M`, `F`, `A`) reference them directly.

**Adding the age breakdown**

The four age columns (`18-24`, `25-49`, `50-64`, `65+`) are already in the data but unplotted.
The two unused calculations `Total (M/F)` and `Total (Age)` appear to be the start of that work.
Extending the dumbbell to four age markers would mean four new aliases on the same pattern as
`M` / `F` / `A`, and a revised `Difference` — the current one compares exactly two series.

**The direction flag**

`M-to-F Win` returns 0 for an exact tie. With only fourteen rows a tie is visible, but on a larger
dataset the neutral state would need a tolerance band rather than an exact equality test.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | Sheet1 (Full_New_Years_Resolutions_Gender_Age_Breakdown) |
| Connection class | `federated` |
| Tables / relations | 1 · joins: 0 |
| Extract rows | 14 |
| Physical columns materialised | 8 |
| Source fields in data pane | 7 |
| Calculated fields (used / total) | 5 / 7 |
| Parameters | 0 |
| Data source filters | 0 (none) |

### Source fields

7 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Survey

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **All**<br>`All Gender` | integer · Measure (Sum) | 12 | yes | 2 | Percentage across all respondents. |
| **Female** | integer · Measure (Sum) | 11 | yes | 2 | Percentage of female respondents naming this resolution. |
| **Male** | integer · Measure (Sum) | 12 | yes | 2 | Percentage of male respondents naming this resolution. |
| **18-24** | integer · Measure (Sum) | 13 | yes | 0 | Percentage among 18–24 year olds. Present in the data but not plotted. *(not used in any sheet)* |
| **25-49** | integer · Measure (Sum) | 13 | yes | 0 | Percentage among 25–49 year olds. Not plotted. *(not used in any sheet)* |
| **50-64** | integer · Measure (Sum) | 13 | yes | 0 | Percentage among 50–64 year olds. Not plotted. *(not used in any sheet)* |
| **65+** | integer · Measure (Sum) | 12 | yes | 0 | Percentage among respondents 65 and over. Not plotted. *(not used in any sheet)* |

### Calculated fields in use

5 of the workbook's 7 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| **A**<br>`Total (M/F) (copy)_1181069047023337472` | integer · Measure | Basic | All | — | 1 |
| **Difference**<br>`Male (copy)_1789899421348548608` | integer · Measure | Basic | Male, Female | — | 2 |
| **F**<br>`Female (copy)_1789899421354749955` | integer · Measure | Basic | Female | — | 1 |
| **M**<br>`Male (copy)_1789899421354770436` | integer · Measure | Basic | Male | — | 1 |
| **M-to-F Win**<br>`Calculation_1789899421358817289` | integer · Measure | Basic | Male, Female | — | 1 |

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas and settings are quoted verbatim from the workbook
definition; rendered figures are read from the published view.*
