# Bar Design Guide — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `Bar Design Guide` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/BarDesignGuide/BarDesignGuide |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, the bar-design index, dashboard layout, interactivity and design system |
| Source of record | `BarDesignGuide.twbx`, decompiled and parsed field-by-field |
| Explicit exclusion | 105 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic** — the workbook was built from a larger design-catalogue file and inherited its calculation library. See Appendix A. |

---

## 1. Executive summary

**Bar Design Guide** is a single-screen teaching reference: 21 bar-chart designs, one displayed
at a time, chosen from a navigation list on the left that is graded **Beginner → Intermediate →
Advanced**.

Structurally it is the inverse of a poster catalogue. Instead of showing every technique at
once, it shows one large, and swaps it via **Dynamic Zone Visibility** — 21 boolean fields, each
controlling three zones, for 63 bindings in total. A companion "Chart Alternatives" panel on
the right shows the same data in two other forms, so the reader can compare the featured design
against substitutes.

The workbook belongs to the same design-guide family as *Viz Design Catalog*: it shares the
`DC_Green2Grey` palette, the same four-region colour convention, and links to the same
downloadable `.tps` palette file.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | 76 |
| Dashboards | 1 (`Bar Design Guide`) |
| Data sources | 1 (Sample - Superstore, 3 joins) |
| Parameters | 4 |
| Bar designs catalogued | **21** |
| Calculated fields **used** | 54 named + 47 in-view ad-hoc |
| Calculated fields unused (not documented) | 105 |
| Dashboard actions | 4 URL, 1 parameter |
| Dynamic Zone Visibility bindings | **63** |
| Global font | Arial |
| Custom palette | `DC_Green2Grey` |

### 1.2 The 21 bar designs

| # | Design | Tier |
|---:|---|---|
| 1 | Stacked Bar | Beginner |
| 2 | Bar Dual | Beginner |
| 3 | Side-by-Side Bar | Beginner |
| 4 | Curved Bar | Beginner |
| 5 | Histogram | Beginner |
| 6 | Diverging Bar | Beginner |
| 7 | Lollipop Bar | Beginner |
| 8 | Bar in Bar | Intermediate |
| 9 | Labels Above Bar | Intermediate |
| 10 | Bar & Total Line | Intermediate |
| 11 | Percentage Bar | Intermediate |
| 12 | Unit Bar | Intermediate |
| 13 | Butterfly Chart | Intermediate |
| 14 | Inverse Bar | Intermediate |
| 15 | Bar & Candlestick | Advanced |
| 16 | L Bar | Advanced |
| 17 | Comparison Bar | Advanced |
| 18 | Stagger Bar | Advanced |
| 19 | Timeline Bar | Advanced |
| 20 | Population Chart | Advanced |
| 21 | Waterfall | Advanced |

The published default is **18 — Stagger Bar**.

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `17660828` |
| LUID | `96806504-aa74-4564-9917-13bb3dbe1516` |
| Repository URL | `BarDesignGuide` |
| Default view | `Bar Design Guide` |
| Revision | 2.0 |
| First published | 2025-10-12 |
| Last published | 2025-10-14 |
| Published size | 2,734,229 bytes (2.6 MB) |
| View count | 10,109 |
| Favourites | 111 |

**Published description**

> Bar Design Guide illustrating 21 bar designs and chart display alternatives. Use the click
> navigation on the left to view the bar designs.
> #Design #Bars #Waterfall

---

## 3. Data architecture

| Property | Value |
|---|---|
| Data source | `Sample - Superstore` |
| Relations | 4 (collection + `Orders`, `People`, `Returns`) |
| Joins | **3** |
| Custom SQL | None |
| Data source filters | None |
| Materialised columns | 12 |

**Schema** — `Order ID`, `Order Date`, `State/Province`, `Region`, `Category`, `Sub-Category`,
`Product Name`, `Sales`, `Quantity`, `Profit`, plus the two join keys (`Region` from People,
`Order ID` from Returns).

A deliberately narrow extract: the guide only ever plots Sales, Profit and Quantity by Region,
Category, Sub-Category and date.

### 3.1 Grain

Order-line grain. As a teaching workbook the grain rarely bites, but `Fixed Total Orders` and
`Total Year Orders` (§5.5) use `{FIXED}` rollups where exact counts matter.

---

## 4. Parameters

| Caption | Type | Domain | Default | Role |
|---|---|---|---|---|
| **BAR** | real list | 21 members, `1` – `21`, aliased to the design names | `18` (Stagger Bar) | **The master selector** — drives all 63 DZV bindings |
| **Top Customers** | integer range | 5 – 20, step 5 | `5` | Top-N control |
| **Profit Bin Size** | integer range | 50 – 200, step 50 | `200` | Histogram bin width |
| **100** | real | any | `100.` | Scaling constant |

---

## 5. Calculated fields in use

54 named calculations plus 47 in-view ad-hoc.

### 5.1 The selector booleans — 21 fields

```
BAR 1 T|F   [BAR] = 1
BAR 2 T|F   [BAR] = 2
…
BAR 21 T|F  [BAR] = 21
```

One boolean per design, each testing the `BAR` parameter against its number. Each controls
**three zones** — the chart itself, its title zone, and its alternatives panel — giving 63
bindings. This is the entire navigation mechanism.

```
Title Chart T|F   [BAR] = ATTR([Chart Number])
```
Compares the parameter against a `Chart Number` field in the navigation data, which is how the
left-hand list highlights the active entry.

### 5.2 The time spine

| Field | Formula | Used in |
|---|---|---:|
| **Max Order Date** | `DATE({ FIXED :max([Order Date])})` | **58 sheets** |
| **Year** | `YEAR([Order Date])` | 40 |
| **Rolling 12 Months** | `if DATETRUNC('month',[Order Date]) > DATETRUNC('month', DATEADD('month',-12,[Max Order Date])) then "Last 12 Months" else "Older" END` | 24 |
| **CY or PY** | Three-way period label (`CY` / `PY` / `Older`) | 15 |
| **Order Date Number** | `INT([Order Date])` | 6 |

As with the author's other Superstore builds, the current year is derived rather than hard-coded,
so the guide re-bases on a data refresh.

### 5.3 Current year vs prior year

```
CY Sales     IF [Year] == YEAR([Max Order Date])     THEN [Sales] END     ← 29 sheets
PY Sales     IF [Year] == (YEAR([Max Order Date])-1) THEN [Sales] END     ← 17 sheets
CY           IF [Year] == YEAR([Max Order Date])     THEN [Year] END
PY           IF [Year] == (YEAR([Max Order Date])-1) THEN [Year] END
-PY Sales    IF [Year] == (YEAR([Max Order Date])-1) THEN ([Sales]*-1) END
CY-PY -Diff  SUM([PY Sales]) - SUM([CY Sales])
CY Sales %   SUM([CY Sales]) / TOTAL(SUM([CY Sales]))     ← table calc
PY Sales %   SUM([PY Sales]) / TOTAL(SUM([PY Sales]))     ← table calc
Sales %      SUM([Sales])    / TOTAL(SUM([Sales]))        ← table calc
```

`-PY Sales` is what powers design **20 — Population Chart**: negating prior year mirrors it to
the left of a shared central axis.

### 5.4 Bar-specific technique fields

| Field | Formula | Powers |
|---|---|---|
| **Placement** | `IIF([CY or PY]='CY', 2, 1)` | **17 — Comparison Bar**: a numeric placement field beside the dimension offsets CY next to PY rather than stacking them |
| **Placement Year** | `IIF([Year]=2024,4, IIF([Year]=2023,3, IIF([Year]=2022,2,1)))` | **18 — Stagger Bar**: four years offset within each region, producing the four-bar clusters in the published view |
| **Region CY/PY** | `[Region] + ' ' + [CY or PY]` | Composite colour key so each region gets a light/dark pair |
| **Region Year** | `STR([Year]) + ' ' + [Region]` | Composite colour key for the stagger bars |
| **- Sales** | `-[Sales]` | **21 — Waterfall**: the negated measure sizing a Gantt bar, which extends downward from its position |
| **Profit (if >0)** | `IF SUM([Profit])<0 then SUM([Profit]) ELSE 0 END` | **6 — Diverging Bar**: isolates the negative side |
| **Candle** | `IF YEAR([Order Date])=2024 THEN [Sales] END` | **15 — Bar & Candlestick** |
| **Quantity (bin)** | `[Quantity]` | **5 — Histogram** |
| **Index** | `INDEX()-1` | **12 — Unit Bar**, **19 — Timeline Bar**: stacks one mark per record |
| **RE CAT** | `UPPER(LEFT([Region],2)+' '+Left([Category],3))` | Compact composite axis label |

### 5.5 Aggregate rollups

```
Fixed Total Orders  { FIXED :COUNT([Order ID]) }
Total Year Orders   { FIXED [Year] : COUNT([Order ID]) }
Year Orders %       ROUND(([Total Year Orders] / [Fixed Total Orders]), 2) * 100
Fixed Total Sales   { FIXED [Year] : SUM([Sales]) }
```

### 5.6 Colour constants

`A_Color` = `'A'`, `B_Color` = `'B'`, `C_Color` = `'C'`, `D_Color` = `'A'` — single-member
dimensions used to force specific marks to fixed palette colours.

---

## 6. Worksheet specifications

76 worksheets, following a strict and highly legible naming convention.

### 6.1 The naming pattern

Every design occupies **three sheets**:

| Sheet | Role | Example |
|---|---|---|
| `<n>: <Name>` | The featured chart, large | `18: Stagger Bar` |
| `<n>.1` | Chart alternative 1 | `18.1` |
| `<n>.2` | Chart alternative 2 | `18.2` |

So design 18 is built from `18: Stagger Bar`, `18.1` and `18.2` — the large chart plus the two
alternatives shown in the right-hand panel. Multiply by 21 designs and that accounts for 63 of
the 76 sheets, matching the 63 DZV bindings exactly (three zones per design).

Two designs need extra sheets because they are built from multiple layers:

| Design | Sheets |
|---|---|
| 11 — Percentage Bar | `11: Percentage Bar CY`, `11: Percentage Bar PY` |
| 14 — Inverse Bar | `14: Inverse 1`, `14: Inverse 2` |
| 20 — Population Chart | `20: Population Chart 1`, `2`, `3` |

### 6.2 Chrome sheets (7)

| Sheet | Role |
|---|---|
| `Vertical` | The left navigation list — the parameter-action source |
| `Title ` | Dashboard title |
| `X` | Spacer/utility |
| `BV-I`, `WV-I` | Volume/brand marks |
| `DL Color` | Palette download icon |
| `LinkedIn`, `Public`, `Twitter` | Social icons |

---

## 7. Dashboard specification

| Property | Value |
|---|---|
| Name | `Bar Design Guide` |
| Canvas | 1600 × 926 px, fixed |
| Ground | White |
| Masthead | Black rule beneath `BAR DESIGN GUIDE` wordmark |

**Regions**

```
Masthead     "BAR DESIGN GUIDE" wordmark · DESIGNED BY JOHN JOHANSSON · 4 icons
Left rail    "Click to Select a Bar Type" + the Vertical nav sheet
             grouped Beginner (1–7) / Intermediate (8–14) / Advanced (15–21)
Main stage   the featured chart, with an icon + title above it
Right panel  "Chart Alternatives" — the <n>.1 and <n>.2 sheets stacked
```

The navigation tiers are colour-coded down the left edge: **Beginner** blue,
**Intermediate** dark, **Advanced** pink — matching the increasing construction difficulty.

---

## 8. Interactivity

### 8.1 Parameter action (1)

| Caption | Source sheet | Payload | Target |
|---|---|---|---|
| `PA Chart Change` | `Vertical` | `Chart Number` | **BAR** |

A single action does all the navigation. Clicking any entry in the left list passes that row's
`Chart Number` into the `BAR` parameter, and the 21 booleans re-evaluate.

### 8.2 URL actions (4)

| Caption | Source | Target |
|---|---|---|
| `DL Color` | `DL Color` | `DesignCatalogColors-Preferences.tps` on Dropbox |
| `LI` | `LinkedIn` | `https://www.linkedin.com/in/johnsjohansson/` |
| `Public` | `Public` | `https://public.tableau.com/app/profile/john.johansson/vizzes` |
| `Twitter` | `Twitter` | `https://twitter.com/JohnSJohansson` |

The `DL Color` link is the **same `.tps` palette file** the *Viz Design Catalog* ships,
confirming the two workbooks are part of one design-guide family.

### 8.3 Dynamic Zone Visibility (63 bindings)

Twenty-one controlling fields, three zones each:

| Field | Zones controlled |
|---|---|
| `BAR 1 T\|F` … `BAR 21 T\|F` | The featured chart zone · the title zone · the alternatives zone |

Exactly one design is visible at any time; all 63 zones collapse except the three whose boolean
is true. Because the zones sit in flow containers, the hidden ones take no space and the layout
does not shift between designs.

---

## 9. Design system

| Token | Hex | Use |
|---|---|---|
| Primary teal | `#0f646a` | Featured measure colour |
| Track grey | `#b6bbc1` | Background tracks |
| Central | `#568ea3` | Region colour (steel blue) |
| East | `#44633f` | Region colour (forest green) |
| South | `#faa916` | Region colour (amber) |
| West | `#b95f89` | Region colour (plum) |
| Beginner | blue | Navigation tier |
| Intermediate | dark | Navigation tier |
| Advanced | pink | Navigation tier |

**Custom palette** — `DC_Green2Grey` (ordered-diverging): `#0f646a → #b6bbc1`

**Global formatting**, set once at workbook level and inherited by all 76 sheets:

```
axis      line-visibility: off
gridline  line-visibility: off
zeroline  line-visibility: off
title     color #000000, bold
all       font-family Arial
```

Every chart is stripped of default chrome, so the bar designs read as pure form.

---

## 10. Rebuild / maintenance runbook

**Adding a 22nd design**

1. Build three sheets following the convention: `22: <Name>`, `22.1`, `22.2`.
2. Add a member to the `BAR` parameter (`22`, aliased to the design name).
3. Add a `BAR 22 T|F` boolean (`[BAR] = 22`).
4. Add the three zones to the dashboard and bind each to the new boolean.
5. Add the entry to the navigation data behind the `Vertical` sheet with `Chart Number` = 22 —
   the existing parameter action needs no change, which is the benefit of driving navigation
   from data rather than from per-button actions.

**Refreshing the data**

`Max Order Date` is used in 58 of 76 sheets, so all CY/PY logic re-bases automatically. Two
literals need attention:

- `Candle` hard-codes `YEAR([Order Date])=2024` for design 15.
- `Placement Year` hard-codes 2024/2023/2022 for design 18's four-year stagger.

**The inherited calculation library**

105 of the 159 calculated fields are unused. They came in with the file the guide was built
from (the author's design-catalogue workbook) and are harmless, but a maintainer should not
assume a field is wired in just because it exists — check usage before editing. This document
covers only the 54 that are actually referenced.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | (unnamed) |
| Connection class | `federated` |
| Tables / relations | 4 · joins: 3 |
| Extract rows | 10,998 |
| Physical columns materialised | 10 |
| Source fields in data pane | 29 |
| Calculated fields (used / total) | 54 / 159 |
| Parameters | 4 |
| Data source filters | 0 (none) |

### Source fields

29 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Location

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Region** | string · Dimension | 4 | yes | 67 | Sales region: Central, East, South, West. The primary categorical split. |
| **State/Province** | string · Dimension | 57 | yes | 67 | State or province. The geographic key for maps and the hex grid. |
| **City** | string · Dimension | — | yes | 0 | City of the shipping address. *(hidden, not materialised, not used in any sheet)* |
| **Country/Region** | string · Dimension | — | yes | 0 | Country. United States and Canada only in this extract. *(hidden, not materialised, not used in any sheet)* |
| **Postal Code** | string · Dimension | — | yes | 0 | Postal code of the shipping address. *(hidden, not materialised, not used in any sheet)* |
| **Region (People)** | string · Dimension | — | — | 0 | Sales region: Central, East, South, West. The primary categorical split. Arrives from the **People** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |

#### Product

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Category** | string · Dimension | 3 | yes | 67 | Top product tier: Furniture, Office Supplies, Technology. |
| **Manufacturer**<br>`Product Name (group)` | string · Dimension | — | — | 67 | Manufacturer name. Largely null in this extract and filtered out on most sheets. |
| **Product Name** | string · Dimension | 2,085 | yes | 67 | Full product description. |
| **Product ID** | string · Dimension | — | yes | 0 | Product key. *(hidden, not materialised, not used in any sheet)* |

#### Dates

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Order Date** | date · Dimension | 1,487 | yes | 67 | Date the customer placed the order. The anchor for every time axis and for `Max Order Date`. |
| **Ship Date** | date · Dimension | — | yes | 0 | Date the order left the warehouse. With Order Date it gives fulfilment duration. *(hidden, not materialised, not used in any sheet)* |

#### Measures

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Sales** | real · Measure (Sum) | 3,577 | yes | 55 | Line revenue in dollars, net of discount. The primary measure. |
| **Profit** | real · Measure (Sum) | 4,104 | yes | 3 | Line profit in dollars. Can be negative where discounting exceeds margin. |
| **Quantity** | integer · Measure (Sum) | 14 | yes | 3 | Units sold on the line. |
| **Discount** | real · Measure (Sum) | — | yes | 0 | Discount rate applied, 0–0.8. *(hidden, not materialised, not used in any sheet)* |

#### Order

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Order ID** | string · Dimension | 296 | yes | 10 | Order key in `CA-2023-100006` form. Orders span multiple lines, so distinct counts must use `COUNTD`. |
| **Order ID (Returns)** | string · Dimension | — | — | 0 | Order key in `CA-2023-100006` form. Orders span multiple lines, so distinct counts must use `COUNTD`. Arrives from the **Returns** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |

#### Catalog scaffold

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Chart Number** | integer · Dimension | — | — | 2 | Sequential chart number in the catalog. Drives navigation order and the parameter payload. |
| **Level** | string · Dimension | — | — | 2 | Tier the chart entry sits at in the catalog. |
| **Name** | string · Dimension | — | — | 2 | Chart name as listed in the catalog. |
| **C** | real · Dimension | — | — | 0 | Category the chart entry belongs to. *(not used in any sheet)* |

#### Customer

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Customer ID** | string · Dimension | — | yes | 0 | Stable customer key. Used for distinct customer counts. *(hidden, not materialised, not used in any sheet)* |
| **Customer Name** | string · Dimension | — | yes | 0 | Customer's display name. Presentation only. *(hidden, not materialised, not used in any sheet)* |
| **Segment** | string · Dimension | — | yes | 0 | Customer type: Consumer, Corporate, Home Office. *(hidden, not materialised, not used in any sheet)* |

#### Identity

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Row ID** | integer · Dimension | — | yes | 0 | Sequential line identifier. One per order line; not used analytically. *(hidden, not materialised, not used in any sheet)* |

#### Join: People

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Regional Manager** | string · Dimension | — | yes | 0 | Manager responsible for the region. Joined from the People table. *(hidden, not materialised, not used in any sheet)* |

#### Join: Returns

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Returned** | string · Dimension | — | yes | 0 | Flag marking a returned order. Present only for returned orders. *(hidden, not materialised, not used in any sheet)* |

#### Shipping

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Ship Mode** | string · Dimension | — | yes | 0 | Service level chosen: Same Day, First Class, Second Class, Standard Class. Drives the delivery-time allowance. *(hidden, not materialised, not used in any sheet)* |

### Calculated fields in use

54 of the workbook's 159 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| **- Sales**<br>`Calculation_2367486073699655681` | real · Measure | Basic | Sales | — | 1 |
| **-PY Sales**<br>`PY Sales (copy)_3314086408495951887` | real · Measure | Basic | Year, Max Order Date, Sales | — | 4 |
| **A_Color**<br>`Calculation_3958664101442371614` | string · Dimension | Basic | — | — | 1 |
| **B_Color**<br>`A_Color (copy)_570549806785167360` | string · Dimension | Basic | — | — | 2 |
| **BAR 1 T\|F**<br>`BAR T\|F (copy)_2031686442042626051` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 10 T\|F**<br>`BAR 7 T\|F (copy) (copy) (copy) (copy) (copy)_1550927183002423304` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 11 T\|F**<br>`BAR 7 T\|F (copy) (copy) (copy) (copy)_1550927183002390531` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 12 T\|F**<br>`BAR 7 T\|F (copy) (copy) (copy) (copy 2)_1550927183002423305` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 13 T\|F**<br>`BAR 7 T\|F (copy) (copy 2)_1550927183002390533` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 14 T\|F**<br>`BAR 13 T\|F (copy)_1550927183002710028` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 15 T\|F**<br>`BAR 10 T\|F (copy)_1550927183002771470` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 16 T\|F**<br>`BAR 11 T\|F (copy)_1550927183002771471` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 17 T\|F**<br>`BAR 12 T\|F (copy)_1550927183002771469` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 18 T\|F**<br>`BAR 14 T\|F (copy)_1550927183002898451` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 19 T\|F**<br>`BAR 15 T\|F (copy)_1550927183002898448` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 2 T\|F**<br>`BAR 1 T\|F (copy)_2031686442042818564` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 20 T\|F**<br>`BAR 16 T\|F (copy)_1550927183002898449` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 21 T\|F**<br>`BAR 17 T\|F (copy)_1550927183002898450` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 3 T\|F**<br>`BAR 2 T\|F (copy 2)_2031686442043740166` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 4 T\|F**<br>`BAR 2 T\|F (copy 2) (copy)_2031686442043756552` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 5 T\|F**<br>`BAR 2 T\|F (copy 2) (copy) (copy 2)_2031686442043809805` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 6 T\|F**<br>`BAR 2 T\|F (copy)_2031686442043699205` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 7 T\|F**<br>`BAR 2 T\|F (copy 2) (copy) (copy) (copy)_2031686442043809804` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 8 T\|F**<br>`BAR 7 T\|F (copy)_1550927183002341376` | boolean · Dimension | Basic | — | — | 1 |
| **BAR 9T\|F**<br>`BAR 7 T\|F (copy) (copy 2) (copy)_1550927183002423302` | boolean · Dimension | Basic | — | — | 1 |
| **C_Color**<br>`B_Color (copy)_1043146293717553163` | string · Dimension | Basic | — | — | 4 |
| **Candle**<br>`Calculation_3542644089719500801` | real · Measure | Basic | Order Date, Sales | — | 2 |
| **CY**<br>`CY/PY Year (copy)_99360729852071959` | integer · Dimension | Basic | Year, Max Order Date | — | 9 |
| **CY or PY**<br>`Calculation_3314086408456491009` | string · Dimension | Basic | Order Date, Max Order Date | Placement, Region CY/PY | 15 |
| **CY Sales**<br>`Calculation_99360729833828364` | real · Measure | Basic | Year, Max Order Date, Sales | $ Change, % Change, % Change +, % Change +\|- … | 29 |
| **CY Sales %**<br>`CY Sales (copy)_99360729865863193` | real · Measure | Table calc | CY Sales | — | 1 |
| **CY-PY -Diff**<br>`Calculation_3542644089717784576` | real · Measure | Basic | PY Sales, CY Sales | — | 2 |
| **D_Color**<br>`A_Color (copy)_239535268360003586` | string · Dimension | Basic | — | — | 1 |
| **Fixed Total Orders**<br>`Index+1 Shapex100 (copy)_1559371376286896135` | integer · Measure | LOD | Order ID | Year Orders % | 4 |
| **Fixed Total Sales**<br>`Total Orders (copy)_3604005630813118467` | real · Measure | LOD | Year, Sales | — | 6 |
| **Index**<br>`Calculation_1043146293700063235` | integer · Measure | Table calc | — | X cos, X cos Half, X cos Half-S, Y sin … | 6 |
| **Max Order Date**<br>`Calculation_99360729564491781` | date · Dimension | LOD | Order Date | -PY Sales, CY, CY Profit, CY Sales … | 58 |
| **Order Date Number**<br>`Order Date (copy)_1043146293700997125` | integer · Dimension | Basic | Order Date | Angle, Angle Half, ODN | 6 |
| **Placement**<br>`Calculation_3314086408456806402` | integer · Measure | Basic | CY or PY | — | 2 |
| **Placement Year**<br>`Placement (copy)_3314086408460079112` | integer · Measure | Basic | Year | — | 1 |
| **Profit (if >0)**<br>`Profit (copy)_239535268359524353` | real · Measure | Basic | Profit | — | 1 |
| **PY**<br>`CY (copy)_99360729852239896` | integer · Dimension | Basic | Year, Max Order Date | — | 9 |
| **PY Sales**<br>`CY (copy)_99360729834057741` | real · Measure | Basic | Year, Max Order Date, Sales | $ Change, % Change, % Change +, % Change +\|- … | 17 |
| **PY Sales %**<br>`CY Sales % (copy)_3958664101420974096` | real · Measure | Table calc | PY Sales | — | 1 |
| **Quantity (bin)** | integer · Dimension | Basic | Quantity | — | 3 |
| **RE CAT**<br>`Calculation_239535268350115840` | string · Dimension | Basic | Region, Category | — | 2 |
| **Region CY/PY**<br>`Calculation_3314086408457351171` | string · Dimension | Basic | Region, CY or PY | — | 4 |
| **Region Year**<br>`Region CY/PY (copy)_3314086408459624455` | string · Dimension | Basic | Year, Region | — | 3 |
| **Rolling 12 Months**<br>`Calculation_99360729563422724` | string · Dimension | Basic | Order Date, Max Order Date | — | 24 |
| **Sales %**<br>`Sales (copy)_239535268370923529` | real · Measure | Table calc | Sales | — | 2 |
| **Title Chart T\|F**<br>`BAR 1 T\|F (copy)_239535268381569034` | boolean · Measure | Basic | Chart Number | — | 1 |
| **Total Year Orders**<br>`Total Orders (copy)_1559371376287309832` | integer · Measure | LOD | Year, Order ID | Year Orders % | 4 |
| **Year**<br>`Calculation_99360729557819395` | integer · Dimension | Basic | Order Date | -PY Sales, CY, CY Profit, CY Sales … | 40 |
| **Year Orders %**<br>`Total Year Orders (copy)_1559371376287551497` | real · Measure | Basic | Total Year Orders, Fixed Total Orders | Year Orders % (bin) | 4 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| **Top Customers**<br>`Parameter 1` | integer · range | `5` | range 5 to 20, step 5 |
| **Profit Bin Size**<br>`Parameter 2` | integer · range | `200` | range 50 to 200, step 50 |
| **100**<br>`Parameter 3` | real · any | `100.` | any value |
| **BAR**<br>`Parameter 6` | real · list | `18.0` | list of 21 — 1.0 → Stacked Bar, 2.0 → Bar Dual, 3.0 → Side-by-Side Bar, 4.0 → Curved Bar, 5.0 → Histogram, 6.0 → Diverging Bar |

### How the numbers are computed

**Level-of-detail expressions — 4.** These fix their own level of detail, so they return the same value regardless of what dimensions are in the view:

> `Max Order Date`, `Fixed Total Orders`, `Total Year Orders`, `Fixed Total Sales`

**Table calculations — 4.** These compute across the rendered table, so their result depends on what is in the view at the time:

> `PY Sales %`, `CY Sales %`, `Index`, `Sales %`

The workbook file records only each table calculation's `ordering-type`. The addressing and partitioning described elsewhere in this document are **derived from each sheet's level-of-detail shelf**, not read from the file.

## Appendix A — Calculated fields excluded from this documentation

105 calculated fields exist in the data pane but are not referenced by any worksheet or
dashboard logic. They are inherited from the author's larger design-catalogue workbook and fall
into these families:

- **Radial / polar geometry** — `Angle`, `Angle Adjusted`, `X cos`, `Y sin`, and the half-radial
  equivalents
- **Funnel** — `Funnel 1`–`Funnel 4` and their `%` / `-%` variants
- **Gauge** — `% Slice 1`–`% Slice 5`
- **Grid coordinates** — `X`, `Y`, `State Abbv`, `Row Rank`, `Col Rank`, `Rank Sales`
- **Winged donut** — `Sales Target`, `Sales Target Diff`, `Fixed Total Region`
- **CY/PY extras** — `% Change`, `$ Change`, `% Change +`, `% Change -`, `CY vs PY Symbol` and
  variants, `MoM % Change`, `Last Month Sales`
- **Plot / stream / stagger helpers** — `Sales Furniture`, `Sales Technology`,
  `Sales Office Supplies` and their `%` variants, `Sales (Stream Bin)`, `Placement Month`,
  `Region Month`
- **Miscellaneous** — `CountD`, `Dummy Count`, `Avg Dummy Count (for Qtrs)`, `Month Order Date`,
  `Center`, `Number of Records`, assorted abbreviation and colour helpers

---

## Appendix B — Package inventory

| File | Size |
|---|---:|
| `Bar Design Guide.twb` | ~2.5 MB |
| Embedded extract | included |
| **Total `.twbx`** | **2.6 MB** |

Custom shapes (navigation icons, social marks) are referenced from the author's local Tableau
Repository and are **not** packaged.

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas and settings are quoted verbatim from the workbook
definition; the rendered figure is read from the published view.*
