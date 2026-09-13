# Generative AI Search Trends — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `Generative AI Search Trends \| #MakeoverMonday` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/GenerativeAISearchTrendsMakeoverMonday/AISearchTrends |
| Challenge | **#MakeoverMonday 2024 Week 08** |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `GenerativeAISearchTrendsMakeoverMonday.twbx`, decompiled and parsed field-by-field |
| Explicit exclusion | 1 calculated field (`2 Color`) exists but is not referenced by any worksheet or dashboard logic. |

---

## 1. Executive summary

**Generative AI Search Trends** maps Google search interest in three image-generation tools —
**Midjourney**, **Stable Diffusion** and **DALL·E** — across the 50 US states plus DC, for
January 2022 to February 2024.

Each state is a hexagon containing a **three-segment donut**, one arc per tool, so the relative
search share reads directly from the ring. A ranked bar panel on the right lists the top five
states per tool, colour-matched to the donut segments.

The workbook's structural choice is a **three-way union**: rather than pivoting a wide table,
the three tools arrive as three separate tables (`MJ`, `DE`, `SD`) unioned into one stream with
an `AI` column identifying the source. Three one-line calculations then split that stream back
into per-tool measures.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | 10 |
| Dashboards | 1 (`AI Search Trends`) |
| Data sources | 1 (**union of 3 tables**) |
| Parameters | 1 |
| Calculated fields **used** | 7 named + 9 in-view ad-hoc |
| Calculated fields unused | 1 |
| Dashboard actions | **0** |
| Dynamic Zone Visibility bindings | 0 |
| Published size | 27,093 bytes (26 KB) — the smallest workbook in the portfolio |

### 1.2 Headline figures rendered by the dashboard

**Top 5 States by AI Search Trends**

| Midjourney | | Stable Diffusion | | DALL·E | |
|---|---:|---|---:|---|---:|
| Hawaii | 51 % | Washington | 32 % | North Dakota | 38 % |
| California | 49 % | New Hampshire | 31 % | South Dakota | 38 % |
| Colorado | 49 % | California | 29 % | West Virginia | 38 % |
| Georgia | 49 % | Idaho | 29 % | Alaska | 37 % |
| New Jersey | 49 % | Vermont | 29 % | Kentucky | 36 % |

Midjourney leads in most states; the three states where DALL·E leads — **North Dakota, South
Dakota and West Virginia** — are picked out on the map with a gold hexagon border (§5.3).

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `14538283` |
| LUID | `36e6bdce-71b3-4b93-b908-bb7ebe3fa17a` |
| Repository URL | `GenerativeAISearchTrendsMakeoverMonday` |
| Default view | `AI Search Trends` |
| Revision | 1.2 |
| First published | 2024-04-19 |
| Last published | 2024-08-13 |
| View count | 457 |
| Favourites | 2 |
| Source | Google Trends |

**Published description**

> #MakeoverMonday 2024W08 | Generative AI Search Trends | Hex-tile

---

## 3. Data architecture

| Property | Value |
|---|---|
| Relations | 4 — `MJ+` (union) containing `MJ`, `DE`, `SD` |
| **Union** | **3 tables** |
| Joins | None |
| Data source filters | None |
| Columns | 8 |

| Column | Role |
|---|---|
| `Region` | State name |
| `Abbv` | Two-letter state code |
| `AI` | **Which tool** — `Midjourney`, `DALL E` or `Stable Diffusion` |
| `Percent` | Search share |
| `X`, `Y` | **Hex coordinates, supplied in the data** |
| `Sheet`, `Table Name` | Union provenance columns |

Two things are notable. First, the union: three identically-shaped tables, one per tool,
distinguished by the `AI` column — the same pattern the author uses in *AI Chatbot Support
Analytics*. Second, the **hex coordinates are in the source**, not calculated. This is the
opposite choice from *US Presidential Elections*, which hard-codes the same layout in two
`CASE` statements. Supplying them as data makes the workbook smaller and the layout editable in
a spreadsheet; calculating them makes the workbook self-contained.

### 3.1 Grain

One row per state × tool — 51 states × 3 tools = 153 rows.

---

## 4. Parameters

| Caption | Type | Domain | Default | Role |
|---|---|---|---|---|
| **Select a Bar Chart:** | integer list | `1` · `2` | `1` | Switches the right-hand panel between the Top-5 view and the all-states view |

---

## 5. Calculated fields in use

7 named calculations plus 9 in-view ad-hoc. This is a compact workbook and every field earns its
place.

### 5.1 Splitting the union back into tools

```
MJ %  IF [AI] = 'Midjourney'        Then [Percent] ELSE Null END
DE %  IF [AI] = 'DALL E'            Then [Percent] ELSE Null END
SD %  IF [AI] = 'Stable Diffusion'  Then [Percent] ELSE Null END
```

The counterpart to the union. Returning `Null` rather than `0` for non-matching rows is what
lets all three measures sit in one view without the zeros distorting the donut arcs or the bar
axis.

### 5.2 Chart switching

```
Case Filter  CASE [Select a Bar Chart:] WHEN 1 THEN 1 WHEN 2 THEN 2 END       ← 4 sheets

Case Title   CASE [Select a Bar Chart:]
               WHEN 1 THEN 'Top 5 States by AI Search Trends'
               WHEN 2 THEN 'State AI Search Trends' END
```

`Case Filter` is applied as a sheet filter so the inactive panel renders no marks; `Case Title`
supplies the matching heading, so the title changes with the content from a single text mark.

### 5.3 The map highlight

```
Hex Color  IF [Abbv] IN ('ND','SD','WV') Then 'Midjourney' Else 'DALL E' END
```

A hard-coded list of the three states where DALL·E out-searches Midjourney. These get the gold
hexagon border visible on the map — a deliberate editorial highlight rather than a computed
comparison.

> Note the field name and its return values read inversely: the three listed states return
> `'Midjourney'` and everything else returns `'DALL E'`. The values are being used as colour
> keys rather than as labels, so what matters is that the three states resolve to a different
> colour from the rest.

### 5.4 Colour constant

```
1 Color  1        ← single-member measure forcing a fixed mark colour
```

---

## 6. Worksheet specifications

10 worksheets.

| Sheet | Role |
|---|---|
| `Map3: Hex` | The hexagon tiles, positioned by the source `X` / `Y` columns, coloured by `Hex Color` |
| `Map2: Pie` | The three-segment donut inside each hexagon — one arc per tool |
| `Map1: Top` | The state-code label layer |
| `B: Title` | The right-panel heading, driven by `Case Title` |
| `B1: MJ` | Midjourney top-5 bars |
| `B1: SD` | Stable Diffusion top-5 bars |
| `B3: DE` | DALL·E top-5 bars |
| `B2: All` | The all-states variant (Chart = `2`) |
| `TT: Bar`, `TT: State` | Viz-in-tooltip sheets shown on hover |

The map is a three-layer composite: hexagon, donut, label — floated at matching coordinates.

---

## 7. Dashboard specification

| Property | Value |
|---|---|
| Name | `AI Search Trends` |
| Canvas | 1600 × 900 px, fixed |
| Ground | White |

**Regions**

```
Masthead   "Generative AI Search Trends" on a red band
           "Google | United States | Jan 2022 - Feb 2024" on a gold band
           right side: three stacked tool labels — Midjourney (blue), Stable Diffusion (red),
           DALL E (gold) — doubling as the colour legend
Map        51 hexagons, each a dark tile with a three-arc donut and state code
Right      "Top 5 States by AI Search Trends" — three colour-matched bar groups
Footer     "Project Makeover Monday | Challenge 2024 Week 08 | Designed by John Johansson |
            Source Google Trends"
```

The masthead's three tool labels are also the legend — each sits on its own coloured band,
matching the donut arcs and the bar fills, so no separate legend is needed.

---

## 8. Interactivity

**No actions and no Dynamic Zone Visibility.**

| Mechanism | Effect |
|---|---|
| `Select a Bar Chart:` parameter | Swaps the right panel via the `Case Filter` sheet filter |
| `TT: Bar` / `TT: State` | Viz-in-tooltip sheets appear on hover over a hexagon |

---

## 9. Design system

| Token | Hex family | Use |
|---|---|---|
| Blue | Google blue | Midjourney — arcs, bars, masthead label |
| Red | Google red | Stable Diffusion — and the masthead band |
| Gold / yellow | Google yellow | DALL·E — and the subtitle band |
| Near-black | | Hexagon tile fill |
| White | | Ground, state codes |

The palette is **Google's own brand colours**, which is apt for a dashboard built on Google
Trends data and gives the three tools an immediately legible three-way split.

---

## 10. Rebuild / maintenance runbook

**Refreshing the data**

1. Update all three union members (`MJ`, `DE`, `SD`), keeping identical column structure — the
   union collapses on column name and a mismatch silently produces nulls.
2. **Re-check `Hex Color`.** The list `('ND','SD','WV')` is a hard-coded editorial highlight of
   the three states where DALL·E leads. New data may change which states qualify; the field will
   not update itself.

**Adding a fourth tool**

1. Add the table to the union with the same columns and a new `AI` value.
2. Add a `XX %` splitter field following the `IF [AI] = '…' Then [Percent] ELSE Null END`
   pattern.
3. Add the arc to `Map2: Pie` and a bar group to the right panel.
4. Add a masthead label band in the new colour.

**The hex layout**

Coordinates live in the `X` and `Y` source columns, so the map is edited in the data rather than
in Tableau. This keeps the workbook at 26 KB — but it also means the layout travels with the
data file, and a source refresh that drops those columns will break the map.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | AI Search Trends, Google, Modified |
| Connection class | `federated` |
| Tables / relations | 4 · joins: 3 |
| Extract rows | 153 |
| Physical columns materialised | 8 |
| Source fields in data pane | 4 |
| Calculated fields (used / total) | 7 / 8 |
| Parameters | 1 |
| Data source filters | 0 (none) |

### Source fields

4 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Location

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Region** | string · Dimension | 51 | yes | 9 | Sales region: Central, East, South, West. The primary categorical split. |

#### Measures

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Percent** | real · Measure (Sum) | 29 | yes | 7 | Google search share for that tool in that state. |

#### Tool

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **AI** | string · Dimension | 3 | yes | 6 | Which generative AI tool the row measures — Midjourney, DALL E, Stable Diffusion. The union discriminator. |

#### Geography

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Abbv** | string · Dimension | 51 | yes | 5 | Two-letter state code. |

### Calculated fields in use

7 of the workbook's 8 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| **1 Color**<br>`Calculation_1622421773839007746` | integer · Measure | Basic | — | — | 1 |
| **Case Filter**<br>`Calculation_738590347448778772` | integer · Dimension | Basic | — | — | 4 |
| **Case Title**<br>`Case Filter (copy)_738590347458584597` | string · Dimension | Basic | — | — | 1 |
| **DE %**<br>`Calculation_738590347434856449` | real · Measure | Basic | AI, Percent | — | 1 |
| **Hex Color**<br>`Calculation_1650006321891946496` | string · Dimension | Basic | Abbv | — | 1 |
| **MJ %**<br>`DE % (copy) (copy)_738590347435728900` | real · Measure | Basic | AI, Percent | — | 1 |
| **SD %**<br>`DE % (copy)_738590347435716611` | real · Measure | Basic | AI, Percent | — | 1 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| **Select a Bar Chart:**<br>`Parameter 1` | integer · list | `1` | list of 2 — 1 → Top 5 States, 2 → All States |

## Appendix A — Calculated field excluded from this documentation

`2 Color` — present in the data pane, not referenced by any worksheet or dashboard logic.

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas and settings are quoted verbatim from the workbook
definition; rendered figures are read from the published view.*
