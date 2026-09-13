# Ode to Du Bois — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `Ode to Du Bois \| B2VB` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/OdetoDuBois/OdetoDuBois |
| Challenge | **#B2VB 2024 Week 04** — Build a Text Table · `#DuBoisChallenge` |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `OdetoDuBois.twbx`, decompiled and parsed field-by-field |
| Explicit exclusion | 1 calculated field (`Year (copy)`) exists but is not referenced by any worksheet or dashboard logic. |

---

## 1. Executive summary

**Ode to Du Bois** recreates **Plate #19** from W.E.B. Du Bois's 1900 Paris Exposition data
portraits — acres of land owned by Black Americans in Georgia, 1874 to 1899 — and then does
something the original could not: it renders the same chart in **three different visual eras**,
switchable from a single control.

The three styles are **(1) Classic Du Bois**, **(2) Mid-Century Modern** and **(3) Sci-Fi
Interface**. Each is a complete parallel build — its own background, bar, chart title and
dashboard title sheets — and a `Portrait Style` parameter swaps between them through 11 Dynamic
Zone Visibility bindings.

That structure is the workbook's real subject. The data is trivial (two columns, 26 rows); the
engineering is in demonstrating that one dataset can carry three entirely different design
languages without duplicating the dashboard.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | 17 |
| Dashboards | **2** (`Ode to Du Bois`, `Dashboard 2`) |
| Data sources | 1 (single table, 2 columns) |
| Parameters | 1 |
| Calculated fields **used** | 8 named + 6 in-view ad-hoc |
| Calculated fields unused | 1 |
| Dashboard actions | 4 URL |
| Dynamic Zone Visibility bindings | 11 |
| Published size | 142,754 bytes (139 KB) |

### 1.2 The source data

| Column | Type | Rows |
|---|---|---:|
| `Year` | integer | 26 (1874 – 1899) |
| `Acres_Owned` | integer | 26 |

Range: **338,769 acres** (1874) rising to **1,023,741 acres** (1899).

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `14106069` |
| LUID | `32b9d2e5-e424-4290-9cb4-aa4bdd6ec428` |
| Repository URL | `OdetoDuBois` |
| Default view | `Ode to Du Bois` |
| Revision | 7.3 |
| First published | 2024-02-24 |
| Last published | 2025-07-11 |
| View count | 290 |
| Favourites | 1 |

**Published description**

> #B2VB 2024W04 | Build a Text Table | W.E.B. Du Bois
> Color and Style Changing Dashboard
> #DuBoisChallenge #DuBois #NAACP #History

---

## 3. Data architecture

| Property | Value |
|---|---|
| Data source | `DuBois Data` |
| Relations | 1 table |
| Joins | **None** |
| Custom SQL | None |
| Data source filters | None |
| Columns | **2** |

One row per year. No aggregation complexity whatsoever — which is the point: the workbook is
about presentation, not analysis.

---

## 4. Parameters

| Caption | Type | Domain | Default | Role |
|---|---|---|---|---|
| **Portrait Style** | integer list | `1` · `2` · `3` | `3` | Selects the visual era |

| Value | Style | Rendered as |
|---|---|---|
| 1 | Classic Du Bois | Cream ground, crimson bars, hand-lettered feel |
| 2 | Mid-Century Modern | 1970s treatment |
| 3 | Sci-Fi Interface | 2100 treatment |

The dashboard exposes this as a **slider labelled with years** — 1900, 1970, 2100 — so the
control itself reads as a time machine rather than a style picker.

---

## 5. Calculated fields in use

8 named calculations plus 6 in-view ad-hoc.

### 5.1 The style switcher

```
Style Filter        CASE [Portrait Style] WHEN 1 THEN 1 WHEN 2 THEN 2 WHEN 3 THEN 3 END   ← 17 sheets

Style Filter 1 T|F  CASE [Portrait Style] WHEN 1 THEN True  WHEN 2 THEN False WHEN 3 THEN False END
Style Filter 2 T|F  CASE [Portrait Style] WHEN 1 THEN False WHEN 2 THEN True  WHEN 3 THEN False END
Style Filter 3 T|F  CASE [Portrait Style] WHEN 1 THEN False WHEN 2 THEN False WHEN 3 THEN True  END
```

`Style Filter` is applied as a filter on **all 17 sheets** — each style's sheets keep only their
own value, so the two inactive styles render no marks. The three booleans then drive Dynamic
Zone Visibility for the surrounding text and title zones.

Using both mechanisms together is deliberate: the sheet filter handles the charts (which would
otherwise render empty but still occupy space), and DZV handles the text and spacer zones
(which have no marks to filter).

### 5.2 The bar geometry

```
-X   -8000
-X2  -8000
+X   [Acres Owned] + ([-X] * -1)
```

`-X` pushes the axis origin left of zero by 8,000 units, creating a fixed gutter for the year
labels. `+X` then adds that offset back to the measure so the bars still start at the true zero
line. This is how the chart achieves Du Bois's characteristic flush-left label column with bars
beginning at a consistent inset.

### 5.3 Colour constant

```
BG  'A'     ← single-member dimension forcing background sheets to a fixed colour
```

---

## 6. Worksheet specifications

17 worksheets, prefixed by style.

| Prefix | Style | Sheets |
|---|---|---|
| **1SF-** | Sci-Fi Interface | `1SF- Background`, `1SF- Bar`, `1SF- CTitle`, `1SF- DTitle`, `1SF- Text2.2` |
| **2MC-** | Mid-Century Modern | `2MC- Background`, `2MC- Bar`, `2.MC- CTitle`, `2MC- DTitle` |
| **3DB-** | Classic Du Bois | `3DB- Background`, `3DB- Bar`, `3DB- CTitle`, `3DB- DTitle` |
| **4Link-** | Chrome | `4Link- GitHub`, `4Link- LoC`, `4Link- NAACP`, `4Link- Wiki` |

Each style family carries four parallel sheets:

| Suffix | Role |
|---|---|
| `Background` | The era's ground treatment |
| `Bar` | The chart itself — `Year` on rows, `+X` on columns |
| `CTitle` | Chart title ("ACRES OF LAND OWNED BY BLACK AMERICANS IN GEORGIA") |
| `DTitle` | Dashboard title ("Ode to W.E.B. Du Bois") |

`1SF- Text2.2` is an extra body-text sheet used only by the Sci-Fi style.

### 6.1 The second dashboard

| Dashboard | Size | Sheets | Purpose |
|---|---|---:|---|
| `Ode to Du Bois` | 1400 × 800 | 16 | The published view |
| `Dashboard 2` | 420 × 560 | 0 | Empty — a phone-layout shell, unused |

---

## 7. Dashboard specification

| Property | Value |
|---|---|
| Name | `Ode to Du Bois` |
| Canvas | 1400 × 800 px, fixed |
| Ground | Style-dependent — cream for Classic Du Bois |

**Regions** (as rendered in Classic Du Bois style)

```
Top-left     "Portrait Style" slider, labelled 1900 · 1970 · 2100
Title        "Ode to W.E.B. Du Bois" in large crimson serif
Chart        "ACRES OF LAND OWNED BY BLACK AMERICANS IN GEORGIA"
             26 horizontal crimson bars, 1874 → 1899, first and last values labelled
Right column Biographical essay on Du Bois, the Paris Exposition, the NAACP,
             and the author's note on the three-style construction
Resources    Four crimson link cards: Library of Congress · NAACP · GitHub · Wikipedia
Footer       "Project Back to Viz Basics | Challenge 2024 Week 04 - Build a Bar Chart |
              Recreated by John Johansson | Hashtag #DuBoisChallenge #B2VB"
```

The right-hand essay explains the build in the author's own words: Du Bois displayed 500
photographs, 200 books and over 60 handmade statistical charts at the 1900 Paris Exposition, and
this workbook recreates Plate #19 in three separate eras.

---

## 8. Interactivity

### 8.1 URL actions (4)

| Caption | Source sheet | Target |
|---|---|---|
| `Library of Congress` | `4Link- LoC` | African American Photographs from the 1900 Paris Exposition |
| `NAACP History` | `4Link- NAACP` | `https://naacp.org/about/our-history` |
| `Style Guide` | `4Link- GitHub` | `https://github.com/ajstarks/dubois-data-portraits` |
| `Wiki` | `4Link- Wiki` | `https://en.wikipedia.org/wiki/W._E._B._Du_Bois` |

The GitHub link points at the community Du Bois style guide — the reference the Classic style is
built against.

### 8.2 Dynamic Zone Visibility (11 bindings)

| Field | Zones |
|---|---|
| `Style Filter 1 T\|F` | `1SF- DTitle` + 2 text zones + 1 spacer |
| `Style Filter 2 T\|F` | `2MC- DTitle` + 2 text zones + 1 spacer |
| `Style Filter 3 T\|F` | `3DB- DTitle` + 2 text zones |

### 8.3 Native control

The `Portrait Style` parameter is exposed as a slider in the top-left corner.

---

## 9. Design system

The workbook has **three** design systems, one per style. As published (style 3, Classic Du Bois
rendering):

| Token | Use |
|---|---|
| Cream / parchment | Dashboard ground |
| Crimson | Title, bars, link cards |
| Black | Body text, chart title, axis labels |

**Global formatting**

```
axis      line-visibility: off
gridline  line-visibility: off
zeroline  line-visibility: off
all       color #000000
```

Note this workbook sets a global **colour** rather than a global font — unusual among the
author's builds, and consistent with a workbook whose typography changes per style.

---

## 10. Rebuild / maintenance runbook

**Adding a fourth style**

1. Add a member to `Portrait Style` (`4`).
2. Extend `Style Filter` with `WHEN 4 THEN 4`.
3. Add a `Style Filter 4 T|F` boolean and extend the existing three with `WHEN 4 THEN False`.
4. Build four sheets following the naming convention: `4XX- Background`, `- Bar`, `- CTitle`,
   `- DTitle`.
5. Apply `Style Filter = 4` as a filter on each new sheet.
6. Add the zones to the dashboard and bind the text and title zones to the new boolean.

**The axis offset**

`-X` and `-X2` both hold `-8000` and must stay in step with each other and with `+X`, which adds
the offset back. Changing the gutter width means editing all three.

**Style filter placement**

`Style Filter` is applied on all 17 sheets including the four link sheets — so a new chrome
sheet also needs the filter, or it will appear in every style.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | DuBois Data (DuBois Data) |
| Connection class | `federated` |
| Tables / relations | 1 · joins: 0 |
| Extract rows | 25 |
| Physical columns materialised | 2 |
| Source fields in data pane | 2 |
| Calculated fields (used / total) | 8 / 9 |
| Parameters | 1 |
| Data source filters | 0 (none) |

### Source fields

2 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Election

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Year** | integer · Dimension | 25 | yes | 3 | Election year. |

#### Observation

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Acres Owned**<br>`Acres_Owned` | real · Measure (Sum) | 22 | yes | 3 | Acres of land owned by Black Americans in Georgia in that year. |

### Calculated fields in use

8 of the workbook's 9 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| **+X**<br>`0 (copy)_899312608492392451` | real · Measure | Basic | Acres Owned, -X | — | 1 |
| **-X**<br>`Calculation_899312608484876289` | integer · Measure | Basic | — | +X | 2 |
| **-X2**<br>`-X (copy)_899312608497950724` | integer · Measure | Basic | — | — | 1 |
| **BG**<br>`Calculation_899312608526852103` | string · Dimension | Basic | — | — | 3 |
| **Style Filter**<br>`Calculation_899312608502190086` | integer · Dimension | Basic | — | — | 17 |
| **Style Filter 1 T\|F**<br>`Style Filter (copy)_562949983823953920` | boolean · Dimension | Basic | — | — | 1 |
| **Style Filter 2 T\|F**<br>`Style Filter (copy) (copy)_562949983823962113` | boolean · Dimension | Basic | — | — | 1 |
| **Style Filter 3 T\|F**<br>`Style Filter (copy) (copy 2)_562949983823970306` | boolean · Dimension | Basic | — | — | 1 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| **Portrait Style**<br>`Parameter 1` | integer · list | `3` | list of 3 — 3 → Classic Du Bois, 2 → Mid-Century Modern, 1 → Sci-Fi Interface |

## Appendix A — Calculated field excluded from this documentation

`Year (copy)` — present in the data pane, not referenced by any worksheet or dashboard logic.

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas and settings are quoted verbatim from the workbook
definition; rendered figures are read from the published view.*
