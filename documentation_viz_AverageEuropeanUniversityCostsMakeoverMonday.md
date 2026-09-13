# Average European University Costs — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `Average European University Costs \| #MakeoverMonday` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/AverageEuropeanUniversityCostsMakeoverMonday/UniversityCosts |
| Challenge | **#MakeoverMonday 2024 Week 05** |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `AverageEuropeanUniversityCostsMakeoverMonday.twbx`, decompiled and parsed field-by-field |
| Explicit exclusion | **None** — all 8 calculated fields are in use. |

---

## 1. Executive summary

**Average European University Costs** ranks 28 European countries by the annual cost of
attending university, from cheapest (Bulgaria) to most expensive (Denmark), and arranges them
as a **pyramid grid**: one country at the apex, two on the next row, three on the next, and so
on.

Two techniques carry the workbook. The first is the pyramid layout itself, built from two
hand-coded `CASE` statements that map each rank to a row and column position (§5.2). The second
is a **text-character bar chart** — the horizontal bars beside each country are not marks at
all, but strings of the block character `▇` sized by a `LEFT(…, CEILING(…))` expression (§5.3).

A ranking parameter re-sorts the whole pyramid by Total Expenses, Living Costs or Tuition & Fees,
and the entire dashboard exists in **three parallel colour builds** — green, blue and red — one
per expense type, which is the "colour changing viz" the description advertises.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | 15 |
| Dashboards | 1 (`University Costs`) |
| Data sources | 1 (single table, 7 columns) |
| Parameters | 1 |
| Calculated fields **used** | **8 of 8** |
| Dashboard actions | 1 highlight |
| Dynamic Zone Visibility bindings | 0 |
| Global font | Arial |
| Published size | 41,949 bytes (41 KB) |

### 1.2 Headline figures rendered by the dashboard

Ranked by **Total Expenses** (default), low to high:

| Rank | Country | Total | Living | Tuition |
|---:|---|---:|---:|---:|
| 1 | Bulgaria | £9.0K | £7.2K | £1.8K |
| 2 | Romania | £10.6K | £8.9K | £3.8K |
| 3 | Portugal | £10.6K | £9.7K | £0.7K |
| 4 | Lithuania | £10.5K | £8.9K | £1.1K |
| 5 | Slovakia | £11.0K | £9.3K | £1.7K |
| … | | | | |
| 25 | United Kingdom | £23.1K | £13.9K | £9.3K |
| 26 | Netherlands | £24.2K | £14.5K | £9.7K |
| 27 | Ireland | £24.3K | £16.6K | £7.8K |
| 28 | **Denmark** | **£25.1K** | £15.8K | £9.3K |

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `13962080` |
| LUID | `8927ca85-08d5-4d60-b9bc-2042a6083517` |
| Repository URL | `AverageEuropeanUniversityCostsMakeoverMonday` |
| Default view | `University Costs` |
| Revision | 2.8 |
| First published | 2024-02-05 |
| Last published | 2024-08-23 |
| View count | 5,271 |
| Favourites | 47 |
| Source | Finder |
| Attribution | Makeover of **Tamás Varga** — *The cheapest countries to study in Europe 2023* |

**Published description**

> #MakeoverMonday 2024W05 | #Pyramid #Grid ranking chart illustrating the average EU costs for
> attending a university by country ranked from low to high.
> #Color Changing Viz | #Parameter | #CaseStatement | #ColorShade

---

## 3. Data architecture

| Property | Value |
|---|---|
| Data source | `Cost to study in select Europea…` |
| Relations | 1 table |
| Joins | **None** |
| Data source filters | None |
| Columns | **7** |
| Rows | 28 (one per country) |

| Column | Role |
|---|---|
| `Country` | The dimension |
| `Total yearly living costs and fees (£)` | Total expense measure |
| `Yearly student living costs (£)` | Living component |
| `Average yearly tuition fees (£)` | Tuition component |
| `Rank (Total)` · `Rank (Living)` · `Rank (Tuition)` | **Pre-computed ranks**, one per expense type |

The three rank columns are supplied in the data rather than calculated. That is what makes the
pyramid layout possible without table calculations: the `CASE` statements in §5.2 can key
directly off a stable rank value regardless of what is filtered.

---

## 4. Parameters

| Caption | Type | Domain | Default | Role |
|---|---|---|---|---|
| **Expense Type** | string list | `1` Total Expenses · `2` Living Costs · `3` Tuition & Fees | `1` | Which ranking drives the pyramid, and which colour build shows |

---

## 5. Calculated fields in use

All 8 calculated fields are in use — an unusually clean workbook.

### 5.1 The expense-type switch

```
*Case       CASE [Expense Type] WHEN '1' THEN 1 WHEN '2' THEN 2 WHEN '3' THEN 3 END   ← 15 sheets

*Case Rank  CASE [Expense Type] WHEN '1' THEN [Rank (Total)]
                                WHEN '2' THEN [Rank (Living)]
                                WHEN '3' THEN [Rank (Tuition)] END                     ← 9 sheets
```

`*Case` is applied as a filter on **all 15 sheets** so only the matching colour build renders.
`*Case Rank` selects which pre-computed rank column drives the layout.

### 5.2 The pyramid grid

Two hand-coded `CASE` statements convert a rank into a grid position:

```
*ROW  FLOAT(if [*Case Rank] IN (1)              Then 7
            ELSEIF [*Case Rank] IN (2, 3)        Then 6
            ELSEIF [*Case Rank] IN (4, 5, 6)     Then 5
            ELSEIF [*Case Rank] IN (7, 8, 9, 10) Then 4
            ELSEIF … END)

*COL  FLOAT(if [*Case Rank] IN (22)      Then 1
            ELSEIF [*Case Rank] IN (16)   Then 2
            ELSEIF [*Case Rank] IN (11, 23) Then 3
            ELSEIF [*Case Rank] IN (7, 17)  Then 4
            ELSEIF … END)
```

`*ROW` uses **triangular-number boundaries** — 1, then 2–3, then 4–6, then 7–10, then 11–15,
16–21, 22–28 — so each row holds one more country than the row above, producing the pyramid.
`*COL` then assigns a horizontal slot within the row, offsetting alternate rows so the shapes
interlock.

Rank 1 sits at row 7 (the apex, since higher row values plot upward) and the bottom row holds
ranks 22–28.

### 5.3 Text-character bar charts

The horizontal bars beside each country are **not marks** — they are strings:

```
*Fixed Total                { FIXED : MAX([Total yearly living costs and fees (£)]) }

*Bar Builder - Living Cost  LEFT("▇▇▇▇▇▇▇▇▇▇▇",
                                 CEILING(SUM([Yearly student living costs (£)])
                                         / SUM([*Fixed Total]) * 10))

*Bar Builder - Tuition      LEFT("▇▇▇▇▇▇▇▇▇▇▇",
                                 CEILING(SUM([Average yearly tuition fees (£)])
                                         / SUM([*Fixed Total]) * 10))

*Bar Build - Total          [*Bar Builder - Living Cost] + [*Bar Builder - Tuition]
```

Each value is expressed as a fraction of the dataset maximum (`*Fixed Total`), scaled to 0–10,
rounded up, and used to take that many block characters from an 11-character string. Concatenating
the living-cost and tuition strings produces a stacked bar in a **single text mark** — which is
what lets the bars sit inside the country tiles without a second sheet or a dual axis.

Using `{FIXED : MAX(...)}` as the denominator means all 28 bars share one scale, so they remain
comparable regardless of which country is filtered or highlighted.

---

## 6. Worksheet specifications

15 worksheets — **three parallel builds of five sheets**, one per colour.

| Sheet | Green (`G-`) | Blue (`B-`) | Red (`R-`) |
|---|---|---|---|
| Grid Background | `G-Grid Background` | `B-Grid Background` | `R-Grid Background` |
| Grid Chart | `G-Grid Chart` | `B-Grid Chart` | `R-Grid Chart` |
| Map | `G-Map` | `B-Map` | `R-Map` |
| Details | `G-Details` | `B-Details` | `R-Details` |
| Title | `G-Title` | `B-Title` | `R-Title` |

| Suffix | Role |
|---|---|
| `Grid Background` | The ring/donut behind each country tile |
| `Grid Chart` | The country shape and rank label, positioned by `*COL` / `*ROW` |
| `Map` | The small Europe reference map, top-right |
| `Details` | The cost figures and text bars inside each tile |
| `Title` | The masthead and key |

Each set is filtered to its own `*Case` value, so exactly one colour build renders at a time.

> **Note on the approach:** three parallel builds is a heavier construction than a single build
> with a calculated colour, but it gives complete independence — each expense type can use a
> different shade ramp, ring treatment and emphasis without compromise.

---

## 7. Dashboard specification

| Property | Value |
|---|---|
| Name | `University Costs` |
| Canvas | 1600 × 900 px, fixed |
| Ground | White |

**Regions**

```
Masthead   "AVERAGE EUROPEAN UNIVERSITY COSTS" · "2023 | Low-to-High Costs"
           explanatory paragraph and method note
           KEY | Ranking Expense: Total Expenses · Living Costs · Tuition & Fees
Main       the pyramid grid — 28 country tiles, each a ring containing the country's
           outline, its rank (#1 … #28), and three cost figures with text bars
Top-right  small Europe reference map
Footer     "Project Makeover Monday | Challenge 2024 Week 05 | Designed by John Johansson |
            Source Finder"
```

Each tile is a composite: a coloured ring (the background sheet), the country's silhouette and
rank, and the three cost values with their block-character bars.

---

## 8. Interactivity

### 8.1 Highlight action (1)

| Caption | Effect |
|---|---|
| `Highlight1` | Hovering a country highlights it across the pyramid and the reference map |

### 8.2 Native control

The `Expense Type` parameter is exposed as the **KEY | Ranking Expense** radio control in the
masthead. Selecting a different expense re-ranks the entire pyramid *and* switches the colour
build — the instruction line states it: *"Select a ranking expense to change the chart. This
will change the color and ranking order for the selected expense."*

### 8.3 Dynamic Zone Visibility

**None.** Build switching is handled entirely by the `*Case` sheet filter.

---

## 9. Design system

Three palettes, one per expense type:

| Expense Type | Palette | Rendered |
|---|---|---|
| Total Expenses | **Green** | The published default |
| Living Costs | Blue | |
| Tuition & Fees | Red | |

Within the green build:

| Token | Use |
|---|---|
| Dark green | Masthead, country silhouettes, total-expense bars |
| Mid green | Ring fills |
| Pale green | Ring backgrounds, the reference map |
| Navy | Living-cost bar segments |
| White | Ground, tile interiors |

**Typography** — Arial, set at workbook level. Large bold masthead, small tile labels.

---

## 10. Rebuild / maintenance runbook

**Refreshing the data**

The three `Rank (…)` columns are **supplied in the source, not calculated**. If the cost figures
change, the ranks must be recomputed upstream or the pyramid will place countries incorrectly —
`*Case Rank` trusts them completely.

**Changing the number of countries**

`*ROW` and `*COL` hard-code every rank position. Moving from 28 countries to a different count
means rewriting both:

- `*ROW` boundaries follow triangular numbers (1, 3, 6, 10, 15, 21, 28). For 36 countries the
  next boundary is 36, adding an eighth row.
- `*COL` assigns a slot per rank and must be extended to match, keeping the alternate-row offset
  so the tiles continue to interlock.

**Adding a fourth expense type**

1. Add the rank column to the source.
2. Add a member to `Expense Type`.
3. Extend `*Case` and `*Case Rank` with the new branch.
4. Duplicate one of the five-sheet colour builds, filter it to the new `*Case` value, and
   restyle.

**The text bars**

`*Bar Builder - Living Cost` and `*Bar Builder - Tuition` both take from an **11-character**
source string and scale to 10. If the scale changes, both the multiplier (`* 10`) and the source
string length must change together, or the longest bars will clip.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | Cost to study in select Europea (cheapest_countries_to_study_Europe_2023) |
| Connection class | `federated` |
| Tables / relations | 1 · joins: 0 |
| Extract rows | 28 |
| Physical columns materialised | 7 |
| Source fields in data pane | 7 |
| Calculated fields (used / total) | 8 / 8 |
| Parameters | 1 |
| Data source filters | 0 (none) |

### Source fields

7 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Costs

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Average yearly tuition fees (£)** | integer · Measure (Sum) | 25 | yes | 9 | Annual tuition component. |
| **Total yearly living costs and fees (£)** | integer · Measure (Sum) | 28 | yes | 9 | Combined annual living costs and tuition, in pounds. |
| **Yearly student living costs (£)** | integer · Measure (Sum) | 28 | yes | 9 | Annual living-cost component. |

#### Ranking

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Rank (Living)** | integer · Measure (Sum) | 28 | yes | 9 | Pre-computed rank by living costs. |
| **Rank (Total)** | integer · Measure (Sum) | 28 | yes | 9 | Pre-computed rank by total cost, supplied in the source. Drives the pyramid layout. |
| **Rank (Tuition)** | integer · Measure (Sum) | 28 | yes | 9 | Pre-computed rank by tuition. |

#### Origin

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Country** | string · Dimension | 28 | yes | 9 | Country of production. |

### Calculated fields in use

8 of the workbook's 8 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| ***Bar Build - Total**<br>`Calculation_597008476902350850` | string · Measure | Basic | *Bar Builder - Living Cost, *Bar Builder - Tuition | — | 9 |
| ***Bar Builder - Living Cost**<br>`Calculation_597008476902862852` | string · Measure | Basic | Yearly student living costs (£), *Fixed Total | *Bar Build - Total | 9 |
| ***Bar Builder - Tuition**<br>`Calculation_597008476903067653` | string · Measure | Basic | Average yearly tuition fees (£), *Fixed Total | *Bar Build - Total | 9 |
| ***Case**<br>`*Case Rank (copy)_597008477305544726` | integer · Dimension | Basic | — | — | 15 |
| ***Case Rank**<br>`Calculation_597008477094965254` | integer · Measure | Basic | Rank (Total), Rank (Living), Rank (Tuition) | *COL, *ROW | 9 |
| ***COL**<br>`Col (copy)_1461136655090253835` | real · Measure | Basic | *Case Rank | — | 6 |
| ***Fixed Total**<br>`Calculation_597008476902576131` | integer · Measure | LOD | Total yearly living costs and fees (£) | *Bar Builder - Living Cost, *Bar Builder - Tuition | 9 |
| ***ROW**<br>`Col (copy)_1461136655087243270` | real · Measure | Basic | *Case Rank | — | 6 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| **Expense Type**<br>`Parameter 1` | string · list | `"1"` | list of 3 — "1" → Total Expenses, "2" → Living Costs, "3" → Tuition & Fees |

### How the numbers are computed

**Level-of-detail expressions — 1.** These fix their own level of detail, so they return the same value regardless of what dimensions are in the view:

> `*Fixed Total`

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas and settings are quoted verbatim from the workbook
definition; rendered figures are read from the published view.*
