# Franky's Adventure Supply Sales — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `Franky's Adventure Supply Sales` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/FrankysAdventureSupplySales/FrankysSales |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | **Local file** — `My Tableau Repository/Workbooks/Camping Sales/Franky's Adventure Supply Sales.twb`, modified **2023-11-02** |
| Version caveat | ⚠️ The published workbook was last updated **2023-12-07**, about five weeks *after* the newest local save. The local file may therefore be a slightly earlier state than what is live. See §0.1. |
| Explicit exclusion | 4 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic**. Listed in Appendix A. |

> **This supersedes an earlier partial version.** Tableau Public has data downloads disabled for
> this workbook, so it was previously documented from a rendered image alone. The local `.twb` now
> supplies the full definition.

### 0.1 Which local file, and how it relates to the published version

Two candidate files exist in the repository:

| File | Modified | Extraction result |
|---|---|---|
| `Camping Sales/Franky's Adventure Supply Sales.twb` | 2023-11-02 14:22 | 16 sheets · 36 calcs (32 used) · 2 parameters |
| `Camping Sales/test1.twb` | 2023-11-23 21:42 | **Identical** — 16 sheets · 36 calcs (32 used) · 2 parameters |

`test1.twb` is the newer save by date but extracts identically, so it is a re-save rather than a
revision. This document uses the named file.

Neither matches the published update date of 2023-12-07, so **figures quoted from the rendered
view may differ slightly from what the local definition produces**. Where a rendered figure is
cited below it is labelled as read from the published view; formulas and structure come from the
local file.

---

## 1. Executive summary

**Franky's Adventure Supply Sales** is a fiscal-year sales dashboard for a fictional outdoor
equipment retailer — sales against target, five-year trend, quarterly attainment, category and
product rankings, and an out-of-stock watchlist.

The definition reveals two things the rendered image could not.

First, the **data model is a six-way join**: a current sales table, a targets table, and four
historical year tables (2019–2022), joined twice over for 11 joins in total. The five-year
comparison is not a filter over one table — it is a union-by-join of separate yearly extracts.

Second, almost every visual emphasis is driven by **`WINDOW_MAX` / `WINDOW_MIN` table
calculations** (§5.4). The min/max dots on each sparkline, and the highlighted top product, are
not annotations — they are calculated fields that return a value only at the extreme and `NULL`
everywhere else.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | **16** |
| Dashboards | 1 (`Franky's Sales`) |
| Data sources | 1 (`CampingSales+`, **11 joins**) |
| Parameters | 2 |
| Calculated fields **used** | 32 named + 34 in-view ad-hoc |
| Calculated fields unused | 4 |
| Dashboard actions | **0** |
| Dynamic Zone Visibility bindings | 0 |
| Canvas | 1400 × 800 px |
| Published size | 1,157,959 bytes (1.1 MB) |

### 1.2 Headline figures

Read from the **published view** (FY2023):

| KPI | Value |
|---|---|
| Sales vs Target | **88.92 %** of a **$1,500K** target |
| Total Sales | **$1,334K** |
| Total Profit | **$800K** |
| Quantity | **17,698** |
| 5 Year Avg | $1,262K |
| 5 Year Sales | $6,310K |
| 5 Year Difference | ▲ $73K (6 %) |

**Quarterly Sales & Targets**

| Quarter | Sales | Target | Attainment | Variance |
|---|---:|---:|---:|---|
| Q1 | $307K | $300K | 102.46 % | **▲ $57K** |
| Q2 | $326K | $350K | 93.07 % | ▼ $24K |
| Q3 | $410K | $500K | 82.00 % | ▼ $90K |
| Q4 | $291K | $350K | 83.05 % | ▼ $59K |

**Sales by Category** — Tents $489K · Gear $380K · Kayaks $278K · Furniture $187K

**Top Selling Products (by Quantity)** — Carabiner 3,280 · Multi-tool 1,668 · Camp Chair 1,219 ·
Single Cozy 1,003 · Camp Table 838

**Out of Stock** — Knife (Gear, Sep 2023) · Multi-tool (Gear, Oct 2023) · Single Cozy (Tents, Nov 2023)

Two of the three out-of-stock items are also top-five sellers — the juxtaposition of those two
panels is the dashboard's sharpest observation.

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `13308717` |
| LUID | `8e8630db-2875-403c-b961-3fdb5126ab2a` |
| Repository URL | `FrankysAdventureSupplySales` |
| Default view | `Franky's Sales` |
| Revision | 2.1 |
| First published | 2023-11-02 |
| Last published | 2023-12-07 |
| View count | 382 |
| Favourites | 6 |
| `allowDataAccess` | **false** — package not downloadable; documented from the local file |

**Published description**

> Franky's Adventure Supply Sales

---

## 3. Data architecture

### 3.1 Connection and model

| Property | Value |
|---|---|
| Data source | `CampingSales+` (Camping Sales Data) |
| Connection class | `federated` |
| Relations | 12 |
| **Joins** | **11** |
| Custom SQL | None |
| Data source filters | **None** |

**The tables**

| Table | Role |
|---|---|
| `CampingSales` | Current-year transactions |
| `Targets` | Sales targets by period |
| `CampingSales_2022` | Prior-year transactions |
| `CampingSales_2021` | |
| `CampingSales_2020` | |
| `CampingSales_2019` | |

Each appears **twice** in the relation list, giving 11 joins across 12 relations. The five-year
history is assembled by joining separate yearly tables rather than filtering one long table —
which is why `FY` is a column on every row and the `FY (T/F)` boolean (§5.2) is applied on 11 of
16 sheets.

### 3.2 Grain

**One row per transaction line.** Columns: `ID`, `Type`, `Unique`, `Date`, `Quarter`, `FY`,
`Product Type`, `Product`, `Quantity`, `Unit Price`, `Cost`, `Sales`, `Profit`, `Inventory`,
`Target`, plus `Sheet` and `Table Name` provenance columns from the multi-table assembly.

Because `Target` arrives on the same rows as `Sales`, attainment is a simple
`SUM([Sales])/SUM([Target])` rather than a cross-source comparison.

### 3.3 Schema

| Column | Type | Role |
|---|---|---|
| `ID`, `Unique` | | Row identity |
| `Type` | | Record classification |
| `Date` | date | The time spine; drives Quarter and the sparklines |
| `Quarter` | | Q1–Q4 |
| `FY` | | Fiscal year — the join/selection key |
| `Product Type` | | Category: Tents · Gear · Kayaks · Furniture |
| `Product` | | Carabiner, Multi-tool, Camp Chair, … |
| `Quantity` | integer | Units sold |
| `Unit Price`, `Cost` | currency | Unit economics |
| `Sales`, `Profit` | currency | The two headline measures |
| `Inventory` | integer | Stock on hand — basis of the out-of-stock test |
| `Target` | currency | Sales target |
| `Sheet`, `Table Name` | | Multi-table provenance |

---

## 4. Parameters

| Caption | Type | Domain | Default | Role |
|---|---|---|---|---|
| **Select a Fiscal Year:** | string list | `2023` FY2023 · `2022` · `2021` · `2020` · `2019` | `2023` | The reporting year |
| **Top Product:** | string list | `Quantity` · `Sales` · `Profit` | `Quantity` | Which measure ranks the Top Selling Products panel |

---

## 5. Calculated fields in use

32 named calculations plus 34 in-view ad-hoc.

### 5.1 Target attainment

```
%                    SUM([Sales]) / SUM([Target])                              ← 3 sheets
Sale to Target       IF SUM([Target]) - SUM([Sales]) > 0
                     then SUM([Target]) - SUM([Sales]) else 0 end              ← 3 sheets
Difference           IF SUM([Target])-SUM([Sales]) > 0 then SUM([Sales])-SUM([Target])
                     elseif SUM([Target])-SUM([Sales]) < 0 then SUM([Sales])-SUM([Target])
                     else 0 end
Difference (Symbol)  IF SUM([Target])-SUM([Sales]) > 0 then "▼"
                     elseif < 0 then "▲" else "►" end
```

`Sale to Target` floors at zero — it is the *shortfall*, used to draw the unfilled remainder of
the target gauge. A quarter that beats target contributes nothing to it, which is why Q1 shows no
grey remainder.

### 5.2 Fiscal-year selection

```
FY (T/F)                   IF [Select a Fiscal Year:] = [FY] then TRUE else FALSE end   ← 11 sheets
FY (T/F) (Yr-to-Yr Chart)  IF INT([Select a Fiscal Year:]) >= int([FY]) then TRUE else FALSE end  ← 4 sheets
```

Two selection booleans doing different jobs. The first is an **equality** test — used by the KPI
cards and quarterly panels, which show one year. The second is a **cumulative** test (`>=`) — used
by the five-year comparison chart, which must show the selected year *and every year before it*.
Selecting FY2021 therefore shortens the trend line rather than emptying it.

### 5.3 The five-year comparison

```
FY Min Year              {FIXED: YEAR(MIN([Date]))}
FY Select Year           Case [Select a Fiscal Year:] When '2023' then '2023' When '2022' then '2022' … END
FY Select-Min Year Diff  INT([FY Select Year]) - YEAR({FIXED: MIN([Date])}) + 1        ← 3 sheets

FY Min Sales     {FIXED : SUM( IF [FY] = (STR([FY Min Year])) THEN [Sales] END )}
FY Select Sales  {FIXED : SUM( IF [FY] = ([FY Select Year]) THEN [Sales] END )}

FY Sales Difference    SUM([FY Select Sales]) - SUM([FY Min Sales])
FY Sales Difference %  (SUM([FY Select Sales]) - SUM([FY Min Sales])) / SUM([FY Min Sales])
FY Sale Direction      IF [FY Sales Difference] < 0 THEN "▼" ELSEIF > 0 THEN "▲"
                       ELSEIF = 0 THEN "▶" ELSE "?" END

Sales Avg              SUM([Sales]) / MAX([FY Select-Min Year Diff])            ← 2 sheets
```

An endpoint comparison between the **earliest year in the data** and the **selected year**,
implemented with `{FIXED}` so both survive the view's filter context. `FY Select-Min Year Diff`
counts the years spanned and is the divisor behind the 5 Year Avg — so the average stays correct
when a different year is selected, rather than always dividing by five.

These produce the ▲ $73K (6 %) figure.

### 5.4 Min/max emphasis — the visual signature

Seven table calculations drive every highlighted point on the dashboard.

**The value pattern** — return the measure only at the extreme, `NULL` otherwise:
```
Min/Max Sales     IF SUM([Sales]) = WINDOW_MAX(SUM([Sales]))   THEN SUM([Sales])
                  ELSEIF SUM([Sales]) = WINDOW_MIN(SUM([Sales])) THEN SUM([Sales])
                  ELSE NULL END
Min/Max Profit    same construction on [Profit]
Min/Max Quantity  same construction on [Quantity]
```

**The colour pattern** — label the extreme so it can be coloured:
```
Min/Max Color Sales     IF SUM([Sales]) = WINDOW_MAX(SUM([Sales]))   THEN "MAX"
                        ELSEIF SUM([Sales]) = WINDOW_MIN(SUM([Sales])) THEN "MIN"
                        ELSE "" END
Min/Max Color Profit    same on [Profit]
Min/Max Color Quantity  same on [Quantity]
```

Returning `NULL` for every non-extreme point is what makes the amber dots appear *only* at the
peak and trough of each sparkline — there is no separate annotation layer. The paired
value/colour fields exist because one supplies the mark position and the other the colour
encoding.

```
Max Color   IF [Metric: Top Product] = WINDOW_MAX([Metric: Top Product]) THEN "MAX" ELSE "REG" END
```
The same idea on the Top Selling Products bars, highlighting the leader.

### 5.5 The top-product metric switcher

```
Metric: Top Product           CASE [Top Product:] WHEN "Quantity" THEN SUM([Quantity])
                                                  WHEN "Sales"    THEN SUM([Sales])
                                                  WHEN "Profit"   THEN SUM([Profit]) END
Metric: Top Product (Symbol)  CASE [Top Product:] WHEN "Quantity" THEN ""
                                                  WHEN "Sales"    THEN "$"
                                                  WHEN "Profit"   THEN "$" END
```

The symbol field supplies or suppresses the currency prefix, so one label serves all three
measures — Quantity renders bare, Sales and Profit render with `$`.

### 5.6 Out-of-stock detection

```
Out of Stock        IF SUM([Quantity]) = SUM([Inventory]) then 'Out of Stock' else 'Instock' END
Out of Stock Since  IF SUM([Quantity]) = SUM([Inventory]) then 'Date' else 'N/A' END
Out of Stock Date   [Date]
```

A product is out of stock when quantity sold equals inventory held — i.e. everything on hand has
been sold. `Out of Stock Since` returns the literal string `'Date'` as a placeholder that the
view replaces with the actual date field, which is how the table shows a month and year only for
genuinely depleted products.

### 5.7 Product display helpers

```
Product              IF [Product Type Set] THEN [Product] ELSE "" END
Product (Symbol)     IF [Product Type Set] THEN "▼" else "►" END
Exclude Null Product [Product]                                         ← 3 sheets
Date (Txt)           [Date]                                            ← 3 sheets
1                    1
```

`Product` and `Product (Symbol)` read a **set** (`Product Type Set`) to drive the category
drill-down: expanded categories show their products and a ▼ glyph; collapsed ones show a ► and
blank product names. This is the mechanism behind the *"Click the arrows to drill down into
products"* instruction on the canvas.

---

## 6. Worksheet specifications

16 worksheets, prefixed by dashboard region.

| Sheet | Role |
|---|---|
| `A- Qtr Targets` | Quarterly sales bars against target |
| `A- Qtr Table` | The Sales / Target / % attainment table beneath them |
| `B- TargetPie` | The Sales-vs-Target radial gauge (88.92 %) |
| `C- Products` | Sales by Category with drill-down to products |
| `C- Top Product` | Top Selling Products, ranked by the chosen metric |
| `D- AllFYSales` | The five-year comparison area chart |
| `D- CallOut1` | 5 Year Avg |
| `D- CallOut2` | 5 Year Sales |
| `D- CallOut3` | 5 Year Difference with direction glyph |
| `D- Out of Stock` | The out-of-stock watchlist |
| `E- Sales` | Total Sales sparkline |
| `E- Sales_Txt` | Its value label |
| `E- Profit` | Total Profit sparkline |
| `E- Profit_Txt` | Its value label |
| `E- Qty` | Quantity sparkline |
| `E- Qty_Txt` | Its value label |

The `E-` pairs follow the same chart/label split seen elsewhere in the portfolio — Tableau cannot
freely place a value beside a sparkline, so each KPI is two sheets.

---

## 7. Dashboard specification

| Property | Value |
|---|---|
| Name | `Franky's Sales` |
| Canvas | **1400 × 800 px**, fixed |
| Ground | Pale sage |
| Masthead | Forest green band |

**Regions**

```
Masthead      dog mascot avatar · "Franky's Adventure Supply" ·
              "Sales Dashboard: FY2023"
              right: "Select a Fiscal Year:" control · Min/Max legend
KPI band      Sales-vs-Target gauge · Total Sales · Total Profit · Quantity,
              each with a 12-month sparkline carrying min/max dots
Left column   FY Comparison — five-year area chart with three call-out figures
              Out of Stock Products — three-row table
Centre        Quarterly Sales & Targets — bars with variance labels,
              plus the Sales / Target / % table
Right column  Sales by Category — four bars with drill-down arrows
              Top Selling Products — five bars with a "View By:" control
```

---

## 8. Interactivity

**No actions and no Dynamic Zone Visibility.** Interactivity is entirely parameter- and
set-driven:

| Control | Mechanism |
|---|---|
| **Select a Fiscal Year:** | Native parameter control → `FY (T/F)` on 11 sheets and `FY (T/F) (Yr-to-Yr Chart)` on 4 |
| **View By:** | Native parameter control → `Metric: Top Product` |
| **Category drill-down** | A Tableau **set** (`Product Type Set`) read by `Product` and `Product (Symbol)` |

The absence of actions is notable given the dashboard's apparent interactivity — the drill-down
that looks like a hierarchy action is in fact set membership driving two calculated fields.

---

## 9. Design system

| Token | Use |
|---|---|
| Forest green | Masthead, bars, gauge ring, KPI values |
| Pale sage | Dashboard ground, panel backgrounds, bar tracks |
| White | Card interiors |
| Amber / gold | Sparkline min/max dots, target markers, the "Target" legend |
| Dark grey | Body text, axis labels |
| Tan | Masthead subtitle |

A deliberate outdoor-retail scheme — forest green and tan on white cards, with **amber reserved
exclusively** for target and extreme-value markers so those readings stand out against the green.
That reservation is what makes the `Min/Max` calculations legible: the only amber on the dashboard
is a peak, a trough or a target.

**Layout craft** — every panel is a white rounded card on pale sage, titled in dark green. The KPI
band uses four equal cards; the body is three columns with the centre widest.

---

## 10. Rebuild / maintenance runbook

**Adding a fiscal year**

1. Add the new yearly table to the data source and join it into `CampingSales+` — the model joins
   separate yearly tables rather than appending to one.
2. Add the year to the `Select a Fiscal Year:` parameter domain.
3. **Extend `FY Select Year`** — it is a `CASE` enumerating each year explicitly and will return
   null for an unlisted one, which would break `FY Select Sales` and every five-year figure.
4. `FY Min Year`, `FY Select-Min Year Diff` and `Sales Avg` all derive from the data and need no
   change.

**The min/max emphasis**

The seven `WINDOW_` fields are **table calculations**, so their result depends on what is in the
view. If a sparkline's level of detail changes — say, from monthly to weekly — the highlighted
points move to the new extremes automatically, but the addressing must still be along the date
axis or the highlight will land on the wrong mark.

**Out-of-stock logic**

`Out of Stock` tests `SUM([Quantity]) = SUM([Inventory])`. That is an equality, not a threshold —
a product with inventory remaining is never flagged, and a data error making quantity exceed
inventory would also fail to flag. If partial-stock warnings are wanted, this needs a `>=`.

**Version note**

The published workbook is dated five weeks after the newest local save (§0.1). Before relying on
this document for a change, confirm the local file is current — or republish from it so the two
align.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | CampingSales+ (Camping Sales Data) |
| Connection class | `federated` |
| Tables / relations | 12 · joins: 11 |
| Extract rows | 1,214 |
| Physical columns materialised | 17 |
| Source fields in data pane | 8 |
| Calculated fields (used / total) | 32 / 36 |
| Parameters | 2 |
| Data source filters | 0 (none) |

### Source fields

8 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Measures

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Sales** | integer · Measure (Sum) | 630 | yes | 11 | Line revenue. |
| **Quantity** | integer · Measure (Sum) | 211 | yes | 4 | Units sold. |
| **Target** | integer · Measure (Sum) | 8 | yes | 4 | Sales target for the period, joined from the Targets table. |
| **Profit** | real · Measure (Sum) | 632 | yes | 3 | Line profit. |
| **Inventory** | integer · Measure (Sum) | 112 | yes | 1 | Stock on hand. A product is flagged out of stock when quantity sold equals inventory. |

#### Dates

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Date** | date · Dimension | 60 | yes | 7 | Transaction date. Drives the monthly sparklines and quarter grouping. |

#### Product

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Category**<br>`Product Type` | string · Dimension | 5 | yes | 2 | Top product tier: Furniture, Office Supplies, Technology. |

#### Identity

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **ID** | integer · Dimension | 302 | yes | 0 | Transaction line identifier. *(not used in any sheet)* |

### Calculated fields in use

32 of the workbook's 36 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| **%**<br>`Calculation_1218505191722606593` | real · Measure | Basic | Sales, Target | — | 3 |
| **1**<br>`Calculation_1218505191724408835` | integer · Measure | Basic | — | — | 1 |
| **Date (Txt)**<br>`Date (copy)_1282400012092030993` | date · Dimension | Basic | Date | — | 3 |
| **Difference**<br>`Sale to Target (copy)_199565776392253440` | integer · Measure | Basic | Target, Sales | — | 2 |
| **Difference (Symbol)**<br>`Difference (copy)_199565776397205506` | string · Measure | Basic | Target, Sales | — | 1 |
| **Exclude Null Product**<br>`Calculation_649925739056513024` | string · Dimension | Basic | — | — | 3 |
| **FY (T/F)**<br>`Calculation_1282400012068335620` | boolean · Dimension | Basic | — | — | 11 |
| **FY (T/F) (Yr-to-Yr Chart)**<br>`FY (T/F) (copy)_1282400012074397706` | boolean · Dimension | Basic | — | — | 4 |
| **FY Min Sales**<br>`Select Sales (copy)_1282400012083560461` | integer · Measure | LOD | FY Min Year, Sales | FY Sales Difference, FY Sales Difference % | 1 |
| **FY Min Year**<br>`Max Year (copy)_1282400012063903746` | integer · Dimension | LOD | Date | FY Min Sales | 1 |
| **FY Sale Direction**<br>`Calculation_1282400012085956623` | string · Measure | Basic | FY Sales Difference | — | 1 |
| **FY Sales Difference**<br>`Calculation_1282400012084133902` | integer · Measure | Basic | FY Select Sales, FY Min Sales | FY Sale Direction | 1 |
| **FY Sales Difference %**<br>`FY Sales Difference (copy)_1282400012086505488` | real · Measure | Basic | FY Select Sales, FY Min Sales | — | 1 |
| **FY Select Sales**<br>`Calculation_1282400012082618380` | integer · Measure | LOD | FY Select Year, Sales | FY Sales Difference, FY Sales Difference % | 1 |
| **FY Select Year**<br>`Calculation_1282400012072423430` | string · Dimension | Basic | — | FY Select Sales, FY Select-Min Year Diff | 3 |
| **FY Select-Min Year Diff**<br>`Calculation_1282400012072030213` | integer · Measure | LOD | FY Select Year, Date | Sales Avg | 3 |
| **Max Color**<br>`Calculation_199565776398569475` | string · Measure | Table calc | Metric: Top Product | — | 1 |
| **Metric: Top Product**<br>`Calculation_649925739068755974` | real · Measure | Basic | Quantity, Sales, Profit | Max Color | 1 |
| **Metric: Top Product (Symbol)**<br>`Calculation_649925739069247495` | string · Dimension | Basic | — | — | 1 |
| **Min/Max Color Profit**<br>`Min/Max Profit (copy)_649925739080146961` | string · Measure | Table calc | Profit | — | 1 |
| **Min/Max Color Quantity**<br>`Min/Max Quantity (copy)_649925739079917582` | string · Measure | Table calc | Quantity | — | 1 |
| **Min/Max Color Sales**<br>`Min/Max Sales (copy)_649925739080146960` | string · Measure | Table calc | Sales | — | 1 |
| **Min/Max Profit**<br>`Calculation_649925739079135245` | real · Measure | Table calc | Profit | — | 1 |
| **Min/Max Quantity**<br>`Calculation_649925739078930443` | integer · Measure | Table calc | Quantity | — | 1 |
| **Min/Max Sales**<br>`Calculation_649925739079069708` | integer · Measure | Table calc | Sales | — | 1 |
| **Out of Stock**<br>`Calculation_1218505191709143040` | string · Measure | Basic | Quantity, Inventory | — | 1 |
| **Out of Stock Date**<br>`Date (copy)_649925739073347592` | date · Dimension | Basic | Date | — | 1 |
| **Out of Stock Since**<br>`Out of Stock (copy)_649925739076665353` | string · Measure | Basic | Quantity, Inventory | — | 1 |
| **Product**<br>`Calculation_649925739060686851` | string · Dimension | Basic | — | — | 1 |
| **Product (Symbol)**<br>`Calculation_649925739058122754` | string · Dimension | Basic | — | — | 1 |
| **Sale to Target**<br>`% Reached (copy)_1754996497336745984` | integer · Measure | Basic | Target, Sales | — | 3 |
| **Sales Avg**<br>`Sales (copy)_1282400012079591435` | real · Measure | Basic | Sales, FY Select-Min Year Diff | — | 2 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| **Top Product:**<br>`Parameter 1` | string · list | `"Quantity"` | list of 3 — "Quantity", "Sales", "Profit" |
| **Select a Fiscal Year:**<br>`Parameter 2` | string · list | `"2023"` | list of 5 — "2023" → FY2023, "2022" → FY2022, "2021" → FY2021, "2020" → FY2020, "2019" → FY2019 |

### How the numbers are computed

**Level-of-detail expressions — 4.** These fix their own level of detail, so they return the same value regardless of what dimensions are in the view:

> `FY Select-Min Year Diff`, `FY Select Sales`, `FY Min Year`, `FY Min Sales`

**Table calculations — 7.** These compute across the rendered table, so their result depends on what is in the view at the time:

> `Max Color`, `Min/Max Quantity`, `Min/Max Sales`, `Min/Max Profit`, `Min/Max Color Profit`, `Min/Max Color Quantity`, `Min/Max Color Sales`

The workbook file records only each table calculation's `ordering-type`. The addressing and partitioning described elsewhere in this document are **derived from each sheet's level-of-detail shelf**, not read from the file.

## Appendix A — Calculated fields excluded from this documentation

4 calculated fields exist in the data pane but are not referenced by any worksheet or dashboard
logic.

---

*Compiled from the local workbook file (`Franky's Adventure Supply Sales.twb`, modified
2023-11-02) parsed field-by-field, with rendered figures read from the published view via the
Tableau Public MCP server. All formulas and settings are quoted verbatim from the workbook
definition; §0.1 records the version caveat.*
