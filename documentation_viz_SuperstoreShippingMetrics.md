# Superstore Shipping Metrics — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `Superstore Shipping Metrics` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/SuperstoreShippingMetrics/Superstore |
| Recognition | Tableau **Viz of the Day** (`#VOTD`) |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `SuperstoreShippingMetrics.twbx` → `Superstore Shipping Metrics.twb`, decompiled and parsed field-by-field |
| Explicit exclusion | 66 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic**. They are listed by name only in Appendix B. |
| Companion file | `documentation_viz_SuperstoreShippingMetrics_README.md` — same facts, condensed overview format |

---

## 1. Executive summary

**Superstore Shipping Metrics** is a single-dashboard analytics application built on the
Sample - Superstore dataset, focused on **shipping and fulfilment performance with
current-year versus prior-year comparison** across every headline metric.

The workbook is unusual for a Superstore build in that almost nothing on the canvas is
hard-wired. Three parameters — **Metric**, **Category** and **Days** — rewrite what the
charts measure and how they break down, and a fourth (**View**) swaps the entire main stage
between a charts view, an order-details table and a distribution pie. What appears to be a
fixed dashboard is in fact one layout rendering many different analyses, driven by 21
Dynamic Zone Visibility bindings and 11 parameter actions.

The analytical backbone is a **dynamic current-year anchor**. Rather than hard-coding a
reporting year, the workbook derives `Max Order Date` from the data itself and defines every
CY/PY pair relative to it, so the dashboard re-bases automatically whenever the data is
refreshed.

Visually it is a dark teal application shell: a fixed icon rail down the left edge, a KPI
column of seven sparkline cards, and a wide content area whose title, charts and breakdowns
all rewrite themselves according to the active parameters.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | 53 |
| Dashboards | 1 (`Superstore`) |
| Data sources | 1 (Excel, 4 relations, 3 joins) |
| Parameters | 8 |
| Total fields in data pane | 181 (23 base + 158 calculated) |
| Calculated fields **used** | 92 named + 120 in-view ad-hoc |
| Calculated fields unused (not documented) | 66 |
| Dashboard canvas | 1600 × 950 px, fixed |
| Global font | Arial |
| Dashboard actions | 3 URL, 2 filter, 11 parameter |
| Dynamic Zone Visibility bindings | 21 |
| Custom colour palette | `Design_Catalog` (7 colours) |

### 1.2 Headline figures rendered by the dashboard

Values as published, with the default parameter state (Metric = Orders, Category = Shipping,
Days = Delivery):

| KPI card | Value | Change vs PY |
|---|---|---|
| Avg Delivery Days | **9.3** ▼ | −1 % (−0.1) vs PY 9.4 |
| Avg Fullfilment Days | **3.9** ▼ | −1 % (0.0) vs PY 4.0 |
| Orders | **1,723** ▲ | +29 % (+383) vs PY |
| Customers | **704** ▲ | +8 % (+51) vs PY |
| Returns | **105** ▲ | +36 % (+28) vs PY |

**Breakdown by Shipping** (current year):

| Ship Mode | Customers | Orders | Returns |
|---|---:|---:|---:|
| Same Day | 86 | 90 | 8 |
| First Class | 237 | 287 | 21 |
| Second Class | 262 | 330 | 22 |
| Standard Class | 565 | 1,016 | 58 |

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `18296408` |
| LUID | `41d8f18c-d39e-4474-9990-a92b3c90d8e2` |
| Repository URL | `SuperstoreShippingMetrics` |
| Default view | `Superstore` |
| Revision | 2.3 |
| First published | 2026-02-25 04:24 UTC |
| Last published | 2026-02-27 17:00 UTC |
| Last updated | 2026-08-21 15:30 UTC |
| Published size | 1,768,973 bytes (1.7 MB) |
| View count | 26,470 |
| Favourites | 151 |
| Data download allowed | Yes |
| Attribution | Inspired by Priya Padham — *Superstore Dashboard* |
| Author links | GitHub `jsjohansson` · X/Twitter `JohnSJohansson` |

**Published description**

> This modern #VizOfTheDay tracks shipping metrics using the Superstore dataset, comparing
> current and previous year performance across sales, orders, and delivery days.
> #VOTD #SuperStore

---

## 3. Data architecture

### 3.1 Connection and model

| Property | Value |
|---|---|
| Data source | `Sample - Superstore` |
| Connection class | `excel-direct` via a `federated` wrapper |
| Relations | 4 (one collection + three tables) |
| Joins | **3** |
| Custom SQL | None |
| Materialisation | Extract — `Data/Extracts/Sample _ Superstore.hyper`, last updated 2026-02-04 04:20 |
| Columns materialised | 25 |

The standard three-table Superstore model:

| Table | Role | Key |
|---|---|---|
| `Orders$` | Fact table — one row per order line | `Order ID`, `Product ID`, `Customer ID` |
| `People$` | Regional manager lookup | `Region` |
| `Returns$` | Returned-order flag | `Order ID` |

The `Returns$` join is what makes a returns metric possible at all; it contributes the second
`Order ID` column (documented below as `Order ID (Returns)`) and the `Returned` flag.

### 3.2 Grain

The extract is at **order-line grain** — one row per product per order. Two consequences run
through the whole workbook:

- Order and customer counts must use `COUNTD([Order ID])` / `COUNTD([Customer ID])`, never
  `COUNT`, or every multi-line order would be counted repeatedly.
- Shipping durations must be collapsed to order level before averaging. That is exactly what
  `Fixed Order F Days` and `Fixed Order D Days` do with `{FIXED [Order ID]}` — averaging the
  raw line-level day counts would weight big orders more heavily than small ones.

### 3.3 Data source filters

**None.** Every constraint in this workbook is applied at worksheet level, chiefly through
the `Rolling 24 Months` and `Rolling 12 Months` windows described in §6.2.

### 3.4 Materialised schema (25 columns)

| Column | Type | Approx. distinct | Notes |
|---|---|---:|---|
| `Row ID` | integer | 10,194 | Line identifier |
| `Order ID` | string | 3,384 | Order key |
| `Order Date` | date | 1,478 | Anchor for all time logic |
| `Ship Date` | date | 1,399 | Fulfilment date |
| `Ship Mode` | string | 4 | Same Day, First Class, Second Class, Standard Class |
| `Customer ID` | string | 1,175 | |
| `Customer Name` | string | 1,175 | |
| `Segment` | string | 3 | Consumer, Corporate, Home Office |
| `Country/Region` | string | 2 | United States, Canada |
| `City` | string | 592 | |
| `State/Province` | string | 62 | |
| `Postal Code` | string | 711 | |
| `Region` | string | 4 | Central, East, South, West |
| `Product ID` | string | 2,138 | |
| `Category` | string | 3 | Furniture, Office Supplies, Technology |
| `Sub-Category` | string | 17 | |
| `Product Name` | string | 2,094 | |
| `Sales` | real | 3,373 | |
| `Quantity` | integer | 16 | |
| `Discount` | real | 12 | |
| `Profit` | real | 3,856 | |
| `Regional Manager` | string | 4 | From `People$` |
| `Region` (People) | string | 4 | Join key |
| `Returned` | string | 1 | From `Returns$` |
| `Order ID` (Returns) | string | 296 | Returned orders only |

### 3.5 Data-pane organisation

The 158 calculations are filed into 20 folders, which is the clearest map of the workbook's
architecture:

| Folder | Purpose |
|---|---|
| `Sales: CY vs PY` | The full current-vs-prior comparison family for sales (18 fields) |
| `Profit: CY vs PY` | Same pattern for profit |
| `Orders: CY vs PY` | Same pattern for orders and returns |
| `Customers: CY vs PY` | Same pattern for customers |
| `Return: CY vs PY` | Return change metrics |
| `Days: CY vs PY` | Same pattern for both day measures |
| `Fulfillment Days` | The duration engine — `Fullfilment Days`, `Delivery Days`, `Est Delivery Date` |
| `Rolling 12 Months` | The time-window anchors |
| `Dynamic Category` | The three-level cascading category system |
| `Dynamic Metric` | Metric switching |
| `Display View` | View-state booleans driving Dynamic Zone Visibility |
| `Days` | Days toggle booleans |
| `Table Details` | Concatenated display strings for the order table |
| `Format Support` | Colour and abbreviation helpers |
| `Plot`, `Comparison Bar`, `Stagger Bar`, `Sunburst Explosion`, `Calculation Examples` | Chart-construction helpers, largely from earlier iterations |

**Hierarchies**

| Drill path | Levels |
|---|---|
| `Location` | `Country/Region` → `State/Province` |
| `Product` | `Product Name` → `Category` → `Sub-Category` → `Manufacturer` |

**Groups** — 5, all auto-generated by dashboard actions and tooltips; none hand-authored.

---

## 4. Field dictionary — base fields

| Caption | Type | Role | Used in | Purpose |
|---|---|---|---:|---|
| `Order Date` | date | Dimension | most | Drives `Year`, `Max Order Date`, both rolling windows and every sparkline axis |
| `Ship Date` | date | Dimension | 7 | Second term in the fulfilment duration |
| `Ship Mode` | string | Dimension | many | Shipping breakdown; drives the delivery-day estimate |
| `Order ID` | string | Dimension | many | Order counting and the `{FIXED}` collapses |
| `Order ID (Returns)` | string | Dimension | 3 | Returns counting — from the `Returns$` join |
| `Customer ID` | string | Dimension | 6 | Distinct customer counts |
| `Customer Name` | string | Dimension | 2 | Order-detail table |
| `Sales` | real | Measure | many | Primary value measure |
| `Profit` | real | Measure | 4 | Profit KPI card |
| `Quantity` | integer | Measure | 2 | Item counts in the order table |
| `Segment` | string | Dimension | — | Product-mode category level 1 |
| `Category` / `Sub-Category` | string | Dimension | many | Product breakdown levels |
| `Region` | string | Dimension | many | Location breakdown level 1 |
| `State/Province` | string | Dimension | many | Location breakdown level 3; map geography |
| `City` | string | Dimension | 1 | Order-detail location string |
| `Country/Region` | string | Dimension | 3 | Map scope (US + Canada) |

---

## 5. Parameters

| Caption | Internal name | Type | Domain | Default | Role |
|---|---|---|---|---|---|
| **View** | `[Parameter 4]` | integer list | `1` Sales Metrics · `2` Order Details · `3` Distribution Pie | `1` | Master view switcher |
| **Dynamic Metric** | `[Dynamic Category (copy)_1561347270422533]` | string list | `Sales` · `Orders` | `Orders` | What the charts measure |
| **Dynamic Category** | `[View (copy)_2196584468738089]` | string list | `Shipping` · `Product` · `Location` | `Shipping` | Which dimension family breaks the data down |
| **Days** | `[View (copy)_2408522740838401]` | string list | `Fullfilment` · `Delivery` | `Delivery` | Which duration measure the days chart shows |
| **Show Breakout P** | `[Parameter 5]` | boolean list | `true` · `false` | `false` | Reveals the pie breakout table |
| **Top Customers** | `[Parameter 1]` | integer range | 5 – 20, step 5 | `5` | Top-N control |
| **Profit Bin Size** | `[Parameter 2]` | integer range | 50 – 200, step 50 | `200` | Histogram bin width |
| **100** | `[Parameter 3]` | real | any | `100.` | Scaling constant |

---

## 6. Calculated fields in use

92 named calculations plus 120 in-view ad-hoc calculations are referenced by sheets or
dashboard logic. They are grouped by role below.

### 6.1 The time anchor — how "current year" is defined

#### `Max Order Date` — used in **53 of 53 worksheets**
```
DATE({ FIXED :max([Order Date])})
```
A table-scoped LOD returning the latest order date anywhere in the data. Every current-year
and prior-year field is defined relative to this rather than to a literal year, which is what
lets the dashboard re-base itself on a data refresh without a single edit. Its presence in
every sheet is the signature of that design decision.

#### `Year`
```
YEAR([Order Date])
```
Used in 22 sheets as the comparison key against `YEAR([Max Order Date])`.

### 6.2 Rolling time windows

| Field | Formula | Purpose |
|---|---|---|
| **Rolling 24 Months** | `if DATETRUNC('month',[Order Date]) > DATETRUNC('month', DATEADD('month',-24,[Max Order Date])) then "Last 24 Months" else "Older" END` | The workbook-wide window — applied in **all 53 sheets**, so every figure covers two years and CY/PY both have data |
| **Rolling 12 Months** | Same construction with `-12` | Narrows the charts that show only the current year (7 sheets) |
| **CY or PY** | `if DATETRUNC('month',[Order Date]) > DATETRUNC('month', DATEADD('month',-12,[Max Order Date])) then "CY" elseif … then "PY" else "Older"` | Three-way period label used to colour the delivery-days comparison bars |
| **Placement** | `IIF([CY or PY]='CY',2,1)` | Ordering key that places the CY bar to the right of the PY bar |

### 6.3 The duration engine

#### `Fullfilment Days` — actual warehouse time
```
DATEDIFF('day',[Order Date], [Ship Date],'sunday')
```
Days between order and despatch. Used in 7 sheets.

#### `Delivery Days` — modelled doorstep time
```
If      [Ship Mode] = 'Same Day'      THEN [Fullfilment Days] + 0
ELSEIF  [Ship Mode] = 'First Class'   THEN [Fullfilment Days] + 2
ELSEIF  [Ship Mode] = 'Second Class'  THEN [Fullfilment Days] + …
ELSE                                       [Fullfilment Days] + …
END
```
The Superstore dataset records despatch but not delivery, so transit time is **modelled** by
adding a fixed allowance per shipping class on top of the measured fulfilment time. This is
the workbook's core analytical assumption: *Delivery Days* is a derived estimate, not an
observed value.

#### `Est Delivery Date`
```
DATE(If [Ship Mode] = 'Same Day' THEN [Ship Date]
     ELSEIF [Ship Mode] = 'First Class' THEN DATEADD('day',2,[Ship Date])
     … END)
```
The same allowances expressed as a date, shown in the order-details table.

#### Order-level collapses
```
Fixed Order F Days:  { FIXED [Order ID]: MIN([Fullfilment Days]) }
Fixed Order D Days:  { FIXED [Order ID]: MIN([Delivery Days])   }
```
These take a single duration per order regardless of how many lines it contains, so the
averages behind the KPI cards are per-order rather than per-line. Without them a ten-item
order would pull the average ten times as hard as a one-item order.

#### `Item Count` / `Total Sales`
```
Item Count:   { FIXED [Order ID]:SUM([Quantity]) }
Total Sales:  { FIXED [Order ID]:SUM([Sales])    }
```
Order-level rollups for the order-details table.

### 6.4 The CY vs PY comparison family

This is the largest group and the most systematic. **Seven metric families** — Sales, Profit,
Orders, Customers, Returns, Fulfilment Days, Delivery Days — each implement the same six-part
pattern. Learn it once and all 46 fields read the same way.

**The template**, shown with Sales:

```
CY Sales            IF [Year] == YEAR([Max Order Date])     THEN [Sales] END
PY Sales            IF [Year] == (YEAR([Max Order Date])-1) THEN [Sales] END
$ Change            (SUM([CY Sales]) - SUM([PY Sales]))
% Change            (SUM([CY Sales]) - SUM([PY Sales])) / SUM([PY Sales])
% Change +|-        IF (SUM([CY Sales]) - SUM([PY Sales]))/SUM([PY Sales]) > 0 THEN '+' ELSE '-' END
CY vs PY Symbol     IF SUM([CY Sales]) > SUM([PY Sales]) THEN "▲"
                    ELSEIF SUM([CY Sales]) < SUM([PY Sales]) THEN "▼"
                    ELSEIF SUM([CY Sales]) = SUM([PY Sales]) THEN "▶" END
```

**The variants**, with the suffix used to distinguish each family:

| Family | Suffix | CY definition | Aggregation |
|---|---|---|---|
| Sales | *(none)* | `IF [Year] == YEAR([Max Order Date]) THEN [Sales] END` | `SUM` |
| Profit | `(P)` | `IF [Year] == YEAR([Max Order Date]) THEN [Profit] END` | `SUM` |
| Orders | `(O)` | `COUNTD(IF [Year] = YEAR([Max Order Date]) THEN [Order ID] END)` | built-in |
| Customers | `(C)` | `COUNTD(IF … THEN [Customer ID] END)` | built-in |
| Returns | `(R)` | `COUNTD(IF … THEN [Order ID (Returns)] END)` | built-in |
| Fulfilment Days | `(F)` | `AVG(IF … THEN [Fixed Order F Days] END)` | built-in |
| Delivery Days | `(D)` | `AVG(IF … THEN [Fixed Order D Days] END)` | built-in |

Two design details worth noting. The count and average families embed their aggregation
inside the field, so the change calculations reference them bare — `([CY Orders]) - ([PY
Orders])` — while the sales and profit families keep the row-level `IF` and wrap it at use
time with `SUM`. And `% Change +|-` returns `'+'` or `''` for most families but `'+'`/`'-'`
for Sales and Profit, which is what produces the differing sign treatment on the cards.

### 6.5 The dynamic category system

Three parameters rewrite what every chart is broken down by. This is the mechanism that lets
one layout serve three analyses.

```
Dynamic Category     Case [Dynamic Category]
                       WHEN 'Shipping' THEN [Ship Mode]
                       WHEN 'Product'  THEN [Segment]
                       WHEN 'Location' THEN [Region] END

Dynamic Category 2   WHEN 'Shipping' THEN [Region]
                     WHEN 'Product'  THEN [Category]
                     WHEN 'Location' THEN [Sub-Region] END

Dynamic Category 3   WHEN 'Shipping' THEN [Category]
                     WHEN 'Product'  THEN [Sub-Category]
                     WHEN 'Location' THEN [State/Province] END
```

A three-level cascading hierarchy that swaps wholesale with the parameter. `Dynamic Category`
is used in **36 sheets**, level 2 in 27, level 3 in 4 — the further down the hierarchy, the
fewer charts consume it.

| Supporting field | Formula | Purpose |
|---|---|---|
| **Dynamic Category 2 Name** | `Case [Dynamic Category] WHEN 'Shipping' THEN 'Region' WHEN 'Product' THEN 'Category' WHEN 'Location' THEN 'Sub-Region' END` | Renders the level's *name* as a row header so the table is self-labelling |
| **Dynamic Category 3 Name** | Same pattern for level 3 | As above |
| **Dynamic Color** | `Case [Dynamic Category] WHEN 'Shipping' THEN 'A' WHEN 'Product' THEN 'A' WHEN 'Location' THEN [Region] END` | Keeps the map a single colour except in Location mode, where it colours by region |
| **Sub-Region** | `IF [State/Province] IN('Illinois','Indiana','Wisconsin','Michigan') THEN 'Central-Alpha' ELSEIF …` | Hand-built sub-regional grouping of states, used in 28 sheets as the Location mode's middle level |
| **State Abbv** | `Case [State/Province] When 'British Columbia' Then 'BC' …` | Two-letter codes for compact display |

### 6.6 Metric switching

| Field | Formula | Purpose |
|---|---|---|
| **Dynamic Metric** | `Case [Dynamic Metric] WHEN 'Sales' THEN SUM([Sales]) WHEN 'Orders' THEN COUNT([Order ID]) END` | Returns the active measure |
| **Dynamic Metric (distinct)** | Same, but `COUNTD([Order ID])` for Orders | The correct version for order-line grain; used where duplicate lines would otherwise inflate the count |
| **Dynamic Metric $** | `Case [Dynamic Metric] WHEN 'Sales' THEN '$' WHEN 'Orders' THEN '' …` | Supplies or suppresses the currency prefix so one label serves both metrics |
| **Dynamic Title** | `IF [View]=1 AND [Dynamic Category]='Shipping' THEN [Dynamic Metric]+' by '+[Dynamic Category]+' Choice' ELSEIF …` | Composes the dashboard subtitle from the active parameter state — the rendered *"Orders by Shipping Choice"* |

### 6.7 View-state booleans (Dynamic Zone Visibility controls)

| Field | Formula | Controls |
|---|---|---|
| **1-Viz T\|F** | `[View] = 1` | The charts view container |
| **2-Table T\|F** | `[View] = 2` | The order-details table |
| **3-Pie T\|F** | `[View] = 3` | The distribution pie and its caption |
| **1or3 T\|F** | `[View] = 1 OR [View] = 3` | Chrome shared by the charts and pie views |
| **1-Viz Sales T\|F** | `[View] = 1 AND [Dynamic Metric] = 'Sales'` | The sales-specific main container |
| **1-Viz Orders T\|F** | `[View] = 1 AND [Dynamic Metric] = 'Orders'` | The orders-specific main container |
| **M1 T\|F** / **M2 T\|F** | `[Dynamic Metric] = 'Orders'` / `= 'Sales'` | Metric button colour; the KPI column pairs |
| **D1 T\|F** / **D2 T\|F** | `[Days] = 'Fullfilment'` / `= 'Delivery'` | Days button colour |
| **1 T\|F** / **2 T\|F** / **3 T\|F** | `[Dynamic Category] = 'Shipping'` / `'Product'` / `'Location'` | Category button colour |
| **Show Breakout** | `True` | Constant source for the breakout parameter action |

Note the compound conditions on `1-Viz Sales T|F` and `1-Viz Orders T|F`: view switching and
metric switching are combined into a single boolean so that two differently-built main
containers can be swapped without nesting logic in the dashboard.

### 6.8 Display strings for the order-details table

| Field | Formula |
|---|---|
| **Order** | `'#'+[Order ID] +" "+ Str([Order Date])` |
| **Shipping** | `[Ship Mode] +" "+ Str([Ship Date]) +" "` |
| **Delivery** | `Str([Delivery Days])+' Days' +" "+ Str([Est Delivery Date]) +" "` |
| **Customer** | `[Customer Name] +" "+ [Customer ID] +" "` |
| **Location** | `[City]+', '+[State Abbv] +" "+ [Region]` |

Each packs several attributes into one string so the table presents as five rich columns
rather than a dozen narrow ones.

### 6.9 Helpers and constants

| Field | Formula | Purpose |
|---|---|---|
| **A_Color_Icons** | `'A'` | Single-member dimension forcing the icon sheets to white (`#ffffff`); used in 11 sheets |
| **B_Color_Box** | `'B'` | Forces a box fill to `#3a6466` |
| **C_Color** | `'C'` | Forces `#0f646a` |
| **Center** | `MAKEPOINT(0,0)` | Origin point anchoring the pie/sunburst construction |
| **Item #** | `INDEX()-1` | Table calculation supplying the vertical offset that turns the dot plot into a jitter plot |
| **Sales (No Format)** | `[Sales]` | An unformatted alias used where the currency format would interfere |
| **1** | `1` | Unit constant for Gantt-bar sizing |

---

## 7. Worksheet specifications

53 worksheets. Grouped by function.

### 7.1 Shared filter pattern

Most analytic sheets carry some combination of:

| Filter | Setting | Purpose |
|---|---|---|
| `Rolling 24 Months` / `Rolling 12 Months` | keep `"Last 24/12 Months"` | The time window |
| `Manufacturer` | keep `%null%` | Suppresses an unused hierarchy level |
| `Action (Dynamic Category)` / `Action (Dynamic Category 2)` | level-members | Auto-generated by the two dashboard filter actions |
| `Category`, `Region`, `State/Province` | level-members | Targets for cross-filtering |

### 7.2 KPI column — 7 card pairs (14 sheets)

Each metric has a **T** (text) sheet and a **C** (chart) sheet, stacked into one card:

| Pair | T sheet shows | C sheet shows |
|---|---|---|
| Sales | `SUM(CY Sales)`, `CY vs PY Symbol`, `% Change +\|-` | Line/Line/Area sparkline of PY vs CY sales by month |
| Profit | `SUM(CY Profit)`, symbol, `% Change (P)` | PY vs CY profit sparkline |
| Orders | `CY Orders`, symbol, `% Change (O)` | PY vs CY orders sparkline |
| Customers | `CY Customers`, symbol, `% Change (C)` | PY vs CY customers sparkline |
| Returns | `CY Returns`, symbol, `% Change (R)` | PY vs CY returns sparkline |
| Fulfilment Days | `CY Fullfilment Days`, symbol, `% Change (F)` | PY vs CY fulfilment sparkline |
| Delivery Days | `CY Delivery Days`, symbol, `% Change (D)` | PY vs CY delivery sparkline |

**Construction of every C sheet:** Columns `MONTH(Order Date)`; Rows the PY measure **+** the
CY measure as a dual axis; three mark layers — **Line** (PY), **Line** (CY), **Area** (the
filled band beneath). Transparent background. The J F M A M J J A S O N D axis labels visible
on the published dashboard come from these.

**Construction of every T sheet:** Text mark, no shelves, a custom multi-line label
concatenating value, symbol and percentage change.

Only four pairs are visible at a time — `M1 T|F` and `M2 T|F` swap the Sales/Profit pair for
the Delivery/Fulfilment pair according to the active metric.

### 7.3 Breakdown bars (4 sheets)

`A: Bar Sales`, `A: Bar Orders`, `A: Bar Customers`, `A: Bar Returns`

| Property | Value |
|---|---|
| Columns | The CY measure **+** the PY measure (dual axis) |
| Rows | `Dynamic Category` |
| Marks | Bar / Bar / **GanttBar** — the Gantt layer draws the thin PY reference tick over the CY bar |
| Colour | `Dynamic Category` |
| Text | `Dynamic Category` and the CY value |
| Titles | "Sales", "Orders", "Customers", "Returns" |

These produce the *Breakdown by Shipping* row: a bar per Ship Mode with its PY marker.

### 7.4 Delivery-days comparison (2 sheets)

| Sheet | Construction |
|---|---|
| **A3: Bar** | Columns `Days Toggle * AVG(Placement)`; Rows `CNT(Order ID)`; colour `CY or PY`; LOD `Year`. The `Placement` field orders CY to the right of PY within each day bucket, producing the paired bars at 0–15 days |
| **A3: Table** | Columns `Days Toggle`; Rows `CY or PY`; text `CNTD(Order ID)` — the CY/PY figure rows printed beneath the chart |

`Days Toggle` = `Case [Days] WHEN 'Fullfilment' THEN [Fullfilment Days] WHEN 'Delivery' THEN
[Delivery Days] END`, so both sheets re-plot when the Days button is pressed.

### 7.5 Distribution views (5 sheets)

| Sheet | Mark | Construction |
|---|---|---|
| **A2: Map** | Multipolygon / Multipolygon / Shape | Filled US + Canada map; colour `Dynamic Color`, size `CNTD(Customer ID)`; rendered as *Customer Distribution by State* |
| **A2: Jitterplot** | Shape | Columns `SUM(Sales)`; Rows `Dynamic Category * Item #` — the `Item #` table calculation spreads overlapping orders vertically. Rendered as *Order Distribution by Sales* |
| **A2: Dotplot** | Shape | The un-jittered variant: Rows `Dynamic Category` only |
| **A2: Bar** | Bar ×3 | Columns `MONTH(Order Date)`; Rows `SUM(CY Sales)` dual axis; colour `Dynamic Category` |
| **A2: Table** | Automatic | The tabular twin of A2: Bar |
| **A2+: Jitter Tooltip** | Automatic | Viz-in-tooltip sheet: Columns `Order Date`, Rows `Product Name / Quantity`, showing order contents on hover |

### 7.6 Pie view (2 sheets)

| Sheet | Construction |
|---|---|
| **C: Pie A** | **Eight** Pie mark layers built on `Center` = `MAKEPOINT(0,0)`; wedge size `Dynamic Metric`; colour alternating `Dynamic Category` and `Dynamic Category 2` to produce concentric rings. Title composed from `Dynamic Title` |
| **C: Table** | Columns `Dynamic Category 2 Name / Dynamic Category 2`; Rows `Dynamic Category 3 Name / Dynamic Category 3`; text `Dynamic Metric $` + `Dynamic Metric (distinct)`. Revealed by `Show Breakout P` |

### 7.7 Sunburst / step build (4 sheets)

`D: Step 1` → `D: Step 4` decompose a hierarchical bar build:

| Sheet | Mark | Rows | Purpose |
|---|---|---|---|
| `D: Step 1` | GanttBar | `(Dynamic Category / Dynamic Category 2) * MIN(0)` | Level-1 labels |
| `D: Step 2` | Bar | `Dynamic Category / Dynamic Category 2` | Level-2 values |
| `D: Step 3` | GanttBar | `(Dynamic Category 2 / Dynamic Category 3) * MIN(0)` | Level-3 labels |
| `D: Step 4` | Bar | `Dynamic Category / Dynamic Category 2 / Dynamic Category 3` | Full three-level bar |

### 7.8 Order details (1 sheet)

**B: Order Details Table** — Columns `Category`; Rows the five packed display strings
(`Order / Shipping / Delivery / Customer / Location`) plus `Segment`, `Order ID`,
`SUM(Item Count)` and `Total Sales`. The View = 2 stage.

### 7.9 Navigation and chrome (17 sheets)

| Group | Sheets | Construction |
|---|---|---|
| View nav | `Nav: Charts`, `Nav: Pie`, `Nav: Table` | Shape marks, colour `A_Color_Icons` (white), detail carrying the payload `1` / `3` / `2` |
| Metric nav | `Nav2: Sales`, `Nav2: Orders` | Square marks, colour `M2 T\|F` / `M1 T\|F`, text `'Sales'` / `'Orders'` |
| Days nav | `Nav3: F`, `Nav3: D` | Square marks, colour `D1 T\|F` / `D2 T\|F`, text `'Fullfilment'` / `'Delivery'` |
| Category nav | `Nav4: Shipping`, `Nav4: Product`, `Nav4: Location` | Square marks, colour `1/2/3 T\|F`, payload `1` / `2` / `3` |
| Social / export | `LinkedIn`, `Public`, `Twitter`, `DL`, `DL Color`, `PDF`, `Image`, `Sym` | Shape marks with custom icons, colour `A_Color_Icons` |
| Title | `Title` | Text mark composing the header from `Dynamic Category`, `Dynamic Metric` and `SUM(CY Sales)` |
| Reference | `Map` | Standalone Sub-Region reference map |

---

## 8. Dashboard specification

### 8.1 Canvas

| Property | Value |
|---|---|
| Name | `Superstore` |
| Sizing | **Fixed**, 1600 × 950 px |
| Root background | `#052527` (near-black teal) |
| Dashboard style | Table background transparent |

### 8.2 Layout tree

Container names are the author's own.

```
[6]   layout-basic   "Container System"            bg #052527
└ [401] flow horz    "HC Highest"                  bg #ffffff
  ├ [403] flow horz  "HC Icons"                    bg #052527   ← the left rail
  │ ├ [373] flow vert                              fixed 100 px
  │ │ ├ [440] worksheet  Sym                       fixed 92 px  (Estella Express logo)
  │ │ ├ [460] text
  │ │ ├ [382] flow vert                            fixed 180 px
  │ │ │ ├ [383] Nav: Charts
  │ │ │ ├ [387] Nav: Pie
  │ │ │ └ [388] Nav: Table
  │ │ ├ [391] flow vert                            fixed 120 px
  │ │ │ ├ [392] DL
  │ │ │ └ [397] Image
  │ │ └ [378] flow vert                            fixed 150 px
  │ │   ├ [285] LinkedIn
  │ │   ├ [287] Public
  │ │   └ [289] Twitter
  │ └ [462][464][399][463][461][400]  six 5 px strips
  │        #062b2d → #073134 → #08373a → #093e41 → #215154 → #3a6466
  └ [346] flow horz
    ├ [349] flow vert  KPI column                  fixed 252 px
    │ ├ [331] Sales pair    (KPI Sales T + C)      hidden-by-user
    │ ├ [352] Profit pair   (KPI Profit T + C)     hidden-by-user
    │ ├ [502] Delivery pair (KPI DDays T + C)
    │ ├ [501] Fulfilment pair (KPI FDays T + C)
    │ ├ [355] Orders pair
    │ ├ [359] Customers pair
    │ └ [470] Returns pair
    ├ [437] empty  4 px divider                    bg #849ea0
    └ [374] flow vert  "VC Superstore"
      ├ [500] worksheet  Title                     fixed 100 px
      └ [439] flow vert  "VC BASE"
        ├ [438] flow vert  "1"                     fixed 900 px  ← charts view
        │ ├ [433] flow horz   A: Bar Customers | A: Bar Orders | A: Bar Returns
        │ ├ [447] empty  2 px rule                 bg #849ea0
        │ ├ [10]  flow vert  "VC: Main"            ← Sales-metric stage
        │ └ [517] flow vert  "VC: Main 2"          ← Orders-metric stage
        ├ [405] "3"                                ← pie view
        └ [406] "2"                                ← table view
```

**The six-strip gradient** between the icon rail and the content area (zones 462, 464, 399,
463, 461, 400) is a hand-built gradient: six 5 px empty containers stepping from `#062b2d` to
`#3a6466`. Tableau has no gradient fill, so the effect is constructed from stacked spacers.

### 8.3 Text zones

| Content | Role |
|---|---|
| `Breakdown by Shipping` | Section header above the three bar panels |
| `Delivery Days` | Section header above the comparison chart |
| `Customers` / `Orders` / `Returns` | Column labels |
| `Hover to select` | Interaction hints beneath the nav button groups |
| `Metric` / `Category` | Labels above the top-right button groups |
| `CY or PY`, `CY`, `PY` | Legend text for the paired bars |

---

## 9. Interactivity

### 9.1 Parameter actions (11)

| Caption | Source sheet | Trigger | Payload | Target parameter |
|---|---|---|---|---|
| `Nav: Chart` | `Nav: Charts` | on-select | `1` | **View** |
| `Nav: Table` | `Nav: Table` | on-select | `2` | **View** |
| `Nav: Pie` | `Nav: Pie` | on-select | `3` | **View** |
| `Nav2: Sales` | `Nav2: Sales` | on-hover | `'Sales'` | **Dynamic Metric** |
| `Nav2: Order` | `Nav2: Orders` | on-hover | `'Orders'` | **Dynamic Metric** |
| `Nav: F Days` | `Nav3: F` | on-hover | `'Fullfilment'` | **Days** |
| `Nav: D Days` | `Nav3: D` | on-hover | `'Delivery'` | **Days** |
| `Change: Shipping` | `Nav4: Shipping` | on-hover | `'Shipping'` | **Dynamic Category** |
| `Change: Product` | `Nav4: Product` | on-hover | `'Product'` | **Dynamic Category** |
| `Change: Location` | `Nav4: Location` | on-hover | `'Location'` | **Dynamic Category** |
| `Display` | `C: Pie A` | on-hover | `Show Breakout` | **Show Breakout P** |

The **View** switches are click-driven while every other control is hover-driven — which is
why the dashboard carries the *"Hover to select"* hints beneath the Metric and Category
groups.

### 9.2 URL actions (3)

| Caption | Source | Target |
|---|---|---|
| `Twitter` | `Twitter` | `https://twitter.com/JohnSJohansson` |
| `LI` | `LinkedIn` | `https://www.linkedin.com/in/johnsjohansson/` |
| `Public` | `Public` | `https://public.tableau.com/app/profile/john.johansson/vizzes` |

### 9.3 Filter actions (2)

`Dynamic F1` and `Dynamic F2`, both hover-triggered, cross-filtering on `Dynamic Category`
and `Dynamic Category 2`. These generate the `Action (Dynamic Category)` filters present on
most sheets.

### 9.4 Dynamic Zone Visibility (21 bindings)

| Controlling field | Zones |
|---|---|
| `1-Viz T\|F` | 438 (charts view container), 473 (its caption) |
| `2-Table T\|F` | 406 (table view) |
| `3-Pie T\|F` | 405 (pie view), 496 (its caption) |
| `1or3 T\|F` | 465, 489 (shared chrome), 494, 497, 498, 499 (shared captions/spacers) |
| `1-Viz Sales T\|F` | 10 (`VC: Main` — the Sales stage) |
| `1-Viz Orders T\|F` | 517 (`VC: Main 2` — the Orders stage), 519, 525 |
| `M1 T\|F` | 501, 502 (Delivery + Fulfilment KPI pairs) |
| `M2 T\|F` | 331, 352 (Sales + Profit KPI pairs) |
| `Show Breakout P` | 495 (`C: Table`), 488 (its spacer) |

**Resulting states**

| View | Metric | Main stage | KPI column shows |
|---|---|---|---|
| 1 Sales Metrics | Sales | `VC: Main` | Sales, Profit, Orders, Customers, Returns |
| 1 Sales Metrics | Orders | `VC: Main 2` | Delivery, Fulfilment, Orders, Customers, Returns |
| 2 Order Details | either | `B: Order Details Table` | as above |
| 3 Distribution Pie | either | `C: Pie A` (+ `C: Table` if Show Breakout) | as above |

The KPI column swap is the subtle part: the same seven cards exist, but which four accompany
Orders/Customers/Returns depends on the metric, so the column always shows metrics relevant
to the active analysis.

---

## 10. Design system

### 10.1 Palette

| Token | Hex | Applied to |
|---|---|---|
| Shell background | `#052527` | Root container and icon rail |
| Gradient steps | `#062b2d` `#073134` `#08373a` `#093e41` `#215154` `#3a6466` | The six-strip rail-to-content transition |
| Primary teal | `#0f646a` | `Design_Catalog` colour 1; measure colour; positive states |
| Active teal | `#3a6466` | Active nav buttons, box fills |
| CY teal | `#3a6466` | Current-year bars |
| PY grey | `#767f8b` | Prior-year bars, inactive nav buttons |
| Steel blue | `#568ea3` | `Design_Catalog` 2; Furniture; secondary category |
| Forest green | `#44633f` | `Design_Catalog` 3; Office Supplies |
| Amber | `#faa916` | `Design_Catalog` 4; Technology; "Older" period |
| Plum | `#b95f89` | `Design_Catalog` 5 |
| Light grey | `#b6bbc1` | `Design_Catalog` 6; Canada |
| Divider | `#849ea0` | 2–4 px rules between regions |
| Icon white | `#ffffff` | All rail icons via `A_Color_Icons` |
| Navy | `#083d77` | West region / Standard Class |

**Custom palette `Design_Catalog`** (regular/categorical):
`#0f646a · #568ea3 · #44633f · #faa916 · #b95f89 · #b6bbc1 · #767f8b`

### 10.2 Categorical colour assignments

| Field | Mapping |
|---|---|
| `Category` | Office Supplies `#44633f` · Furniture `#568ea3` · Technology `#faa916` |
| `CY or PY` | CY `#3a6466` · PY `#767f8b` · Older `#f28e2b` |
| `Dynamic Category` | Standard Class / West `#083d77` · Home Office / Second Class / South / Technology `#568ea3` · Central / Consumer / Office Supplies / Same Day `#b95f89` · Corporate / East / First Class `#faa916` |
| `Country/Region` | United States `#767f8b` · Canada `#b6bbc1` |
| All nav booleans | true `#3a6466` · false `#767f8b` |
| `YR(Order Date)` | 2021 `#b95f89` · 2022 `#faa916` · 2023 `#44633f` · 2024 `#568ea3` |
| `Region` (shape) | Four petal images — `Petals/TopLeft.png`, `TopRight`, `BottomLeft`, `BottomRight` |

### 10.3 Typography

| Level | Spec |
|---|---|
| Workbook default | **Arial** |
| Titles | Bold, `#000000` by default, overridden per zone |
| Dashboard title | Large teal — *"Superstore Shipping Metrics"* |
| Subtitle | Composed by `Dynamic Title` — *"Orders by Shipping Choice"* |
| KPI value | Large bold |
| KPI delta | Small, with ▲ / ▼ / ▶ glyph |
| Section headers | Bold, teal |

### 10.4 Global chart formatting

Set once at workbook level and inherited everywhere:

```
axis      line-visibility: off
gridline  line-visibility: off
zeroline  line-visibility: off
```

Every chart is therefore clean by default, and any visible rule is a deliberate exception.
All analytic sheets have transparent backgrounds (`#00000000`) over the dark shell.

### 10.5 Iconography

Custom shapes from the author's local repository: the Estella Express brand mark, navigation
glyphs, download/PDF/image export icons, LinkedIn / Tableau / X marks, and the four
`Petals/*.png` directional shapes used for regional encoding. Not embedded in the `.twbx`.

---

## 11. Calculation dependency map

```
Order Date ──► Max Order Date {FIXED :MAX} ──┬─► Year comparison (CY/PY across 7 families)
               (used in all 53 sheets)       ├─► Rolling 12 Months ──► chart windows
                                             ├─► Rolling 24 Months ──► every sheet
                                             └─► CY or PY ──► Placement ──► A3: Bar

Order Date ─┬─► Fullfilment Days ──┬─► Delivery Days ──┬─► Fixed Order D Days ──► CY/PY Delivery Days
Ship Date ──┘                      │   (Ship Mode      └─► Est Delivery Date ──► Delivery (string)
                                   │    allowance)
                                   └─► Fixed Order F Days ──► CY/PY Fullfilment Days
                                   └─► Days Toggle ──► A3: Bar / A3: Table

Dynamic Category param ─┬─► Dynamic Category   (36 sheets)
                        ├─► Dynamic Category 2 (27 sheets)
                        ├─► Dynamic Category 3 (4 sheets)
                        ├─► Dynamic Category 2/3 Name ──► C: Table headers
                        ├─► Dynamic Color ──► A2: Map
                        └─► 1/2/3 T|F ──► nav button colour

Dynamic Metric param ─┬─► Dynamic Metric, Dynamic Metric (distinct), Dynamic Metric $
                      ├─► M1/M2 T|F ──► DZV: KPI column pairs
                      └─► with View ──► 1-Viz Sales/Orders T|F ──► DZV: main stage

View param ──┬─► 1-Viz T|F, 2-Table T|F, 3-Pie T|F, 1or3 T|F ──► DZV: 11 zones
             └─► Dynamic Title ──► dashboard subtitle

Days param ──┬─► D1/D2 T|F ──► nav button colour
             └─► Days Toggle ──► delivery-days comparison
```

---

## 12. Rebuild / maintenance runbook

**Refreshing the data**

1. Replace the `Sample - Superstore` Excel source and refresh the extract. The three-table
   join (Orders / People / Returns) must be preserved — `Order ID (Returns)` from the
   `Returns$` table is what the entire returns family counts.
2. No year constants need changing. `Max Order Date` re-derives the current year from the
   data, and every CY/PY field follows it. This is the main benefit of the design.
3. Confirm the rolling windows still make sense: `Rolling 24 Months` is applied in all 53
   sheets and assumes at least two years of history so that PY is populated.

**Re-tuning the delivery model**

`Delivery Days` adds a fixed per-ship-mode allowance to the measured fulfilment time. If the
assumed transit times change, edit both `Delivery Days` and `Est Delivery Date` together —
they encode the same allowances in two forms, and letting them drift apart would make the
estimated date disagree with the estimated duration.

**Changing what the category buttons show**

The three-level cascade lives in four fields. To alter a mode, edit `Dynamic Category`,
`Dynamic Category 2`, `Dynamic Category 3` and the two `… Name` fields as a set, so the row
headers keep matching the data being shown.

**Adding a metric to the CY vs PY family**

Copy an existing family and rename the suffix. Each needs six fields: `CY x`, `PY x`,
`# / $ Change`, `% Change`, `% Change +|-`, `CY vs PY Symbol`. Use the count/average pattern
(aggregation inside the field) for discrete measures and the Sales pattern (row-level `IF`,
aggregated at use) for additive ones. Then build a T sheet and a C sheet and add them to the
KPI column as a new flow container.

**Adding a view**

1. Add a member to the `View` parameter.
2. Add a `…T|F` boolean testing it, and extend `1or3 T|F` if the new view shares chrome.
3. Duplicate a `Nav:` sheet, swap the payload constant and colour field.
4. Add a parameter action from the new button to `View`.
5. Bind the new container's Dynamic Zone Visibility to the new boolean.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | (unnamed) |
| Connection class | `federated` |
| Tables / relations | 4 · joins: 3 |
| Physical columns materialised | 23 |
| Source fields in data pane | 19 |
| Calculated fields (used / total) | 92 / 158 |
| Parameters | 8 |
| Data source filters | 0 (none) |

### Source fields

19 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Dates

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Order Date** | date · Dimension | 1,478 | yes | 53 | Date the customer placed the order. The anchor for every time axis and for `Max Order Date`. |

#### Location

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Region** | string · Dimension | 4 | yes | 39 | Sales region: Central, East, South, West. The primary categorical split. |
| **State/Province** | string · Dimension | 62 | yes | 39 | State or province. The geographic key for maps and the hex grid. |
| **Country/Region** | string · Dimension | 2 | yes | 2 | Country. United States and Canada only in this extract. |
| **City** | string · Dimension | 592 | yes | 1 | City of the shipping address. |
| **Postal Code** | string · Dimension | 711 | yes | 0 | Postal code of the shipping address. *(not used in any sheet)* |

#### Product

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Category** | string · Dimension | 3 | yes | 37 | Top product tier: Furniture, Office Supplies, Technology. |
| **Manufacturer**<br>`Product Name (group)` | string · Dimension | — | — | 30 | Manufacturer name. Largely null in this extract and filtered out on most sheets. |
| **Product Name** | string · Dimension | 2,094 | yes | 30 | Full product description. |
| **Sub-Category** | string · Dimension | 17 | yes | 7 | Second product tier — 17 values such as Binders, Chairs, Phones. |

#### Customer

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Segment** | string · Dimension | 3 | yes | 37 | Customer type: Consumer, Corporate, Home Office. |
| **Customer ID** | string · Dimension | 1,175 | yes | 8 | Stable customer key. Used for distinct customer counts. |

#### Shipping

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Ship Mode** | string · Dimension | 4 | yes | 37 | Service level chosen: Same Day, First Class, Second Class, Standard Class. Drives the delivery-time allowance. |

#### Measures

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Sales** | real · Measure (Sum) | 3,373 | yes | 16 | Line revenue in dollars, net of discount. The primary measure. |
| **Profit** | real · Measure (Sum) | 3,856 | yes | 2 | Line profit in dollars. Can be negative where discounting exceeds margin. |
| **Quantity** | integer · Measure (Sum) | 16 | yes | 2 | Units sold on the line. |
| **Discount** | real · Measure (Sum) | 12 | yes | 0 | Discount rate applied, 0–0.8. *(not used in any sheet)* |

#### Order

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Order ID** | string · Dimension | 296 | yes | 16 | Order key in `CA-2023-100006` form. Orders span multiple lines, so distinct counts must use `COUNTD`. |

#### Identity

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Row ID** | integer · Dimension | 10,194 | yes | 0 | Sequential line identifier. One per order line; not used analytically. *(not used in any sheet)* |

### Calculated fields in use

92 of the workbook's 158 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| **# Change (C)**<br>`$ Change (O) (copy)_2196584445612047` | integer · Measure | Basic | CY Customers, PY Customers | — | 1 |
| **# Change (D)**<br>`# Change (F) (copy)_1358763908968452` | real · Measure | Basic | CY Delivery Days, PY Delivery Days | — | 1 |
| **# Change (F)**<br>`# Change (C) (copy)_1358763907084288` | real · Measure | Basic | CY Fullfilment Days, PY Fullfilment Days | — | 1 |
| **# Change (O)**<br>`$ Change (P) (copy)_2196584415879178` | integer · Measure | Basic | CY Orders, PY Orders | — | 1 |
| **# Change (R)**<br>`# Change (O) (copy)_2117149703892996` | integer · Measure | Basic | CY Returns, PY Returns | — | 1 |
| **$ Change**<br>`% Change (copy)_2367486074249367583` | real · Measure | Basic | CY Sales, PY Sales | — | 1 |
| **$ Change (P)**<br>`$ Change (copy)_2196584415133703` | real · Measure | Basic | CY Profit, PY Profit | — | 1 |
| **% Change**<br>`CY vs PY Symbol (copy)_99360729838260239` | real · Measure | Basic | CY Sales, PY Sales | — | 1 |
| **% Change (C)**<br>`% Change (O) (copy)_2196584445612048` | real · Measure | Basic | CY Customers, PY Customers | — | 1 |
| **% Change (D)**<br>`% Change (F) (copy)_1358763908968453` | real · Measure | Basic | CY Delivery Days, PY Delivery Days | — | 1 |
| **% Change (F)**<br>`% Change (C) (copy)_2179824139530254` | real · Measure | Basic | CY Fullfilment Days, PY Fullfilment Days | — | 1 |
| **% Change (O)**<br>`% Change (P) (copy)_2196584415854600` | real · Measure | Basic | CY Orders, PY Orders | — | 1 |
| **% Change (P)**<br>`% Change (copy)_2196584415117318` | real · Measure | Basic | CY Profit, PY Profit | — | 1 |
| **% Change (R)**<br>`% Change (O) (copy)_2117149703892997` | real · Measure | Basic | CY Returns, PY Returns | — | 1 |
| **% Change +\|-**<br>`% Change T+\|F- (copy)_1197957530490093571` | string · Measure | Basic | CY Sales, PY Sales | — | 1 |
| **% Change +\|- (C)**<br>`% Change +\|- (O) (copy)_2196584445612049` | string · Measure | Basic | CY Customers, PY Customers | — | 1 |
| **% Change +\|- (D)**<br>`% Change +\|- (F) (copy)_1358763908968454` | string · Measure | Basic | CY Delivery Days, PY Delivery Days | — | 1 |
| **% Change +\|- (F)**<br>`CY Fullfilment Days (copy)_2179824139067405` | string · Measure | Basic | CY Fullfilment Days, PY Fullfilment Days | — | 1 |
| **% Change +\|- (O)**<br>`% Change +\|- (P) (copy)_2196584415866889` | string · Measure | Basic | CY Orders, PY Orders | — | 1 |
| **% Change +\|- (P)**<br>`% Change +\|- (copy)_2196584415105029` | string · Measure | Basic | CY Profit, PY Profit | — | 1 |
| **% Change +\|- (R)**<br>`% Change +\|- (O) (copy)_2117149703892998` | string · Measure | Basic | CY Returns, PY Returns | — | 1 |
| **1**<br>`Calculation_2117149735387150` | integer · Measure | Basic | — | — | 2 |
| **1 T\|F**<br>`Calculation_3967076110352402` | boolean · Dimension | Basic | — | — | 1 |
| **1-Viz Orders T\|F**<br>`1-Viz T\|F (copy)_2408522748567556` | boolean · Dimension | Basic | — | — | 1 |
| **1-Viz Sales T\|F**<br>`1-Viz Orders T\|F (copy)_2408522748760069` | boolean · Dimension | Basic | — | — | 1 |
| **1-Viz T\|F**<br>`Calculation_2196584472920107` | boolean · Dimension | Basic | — | — | 1 |
| **1or3 T\|F**<br>`1-Viz T\|F (copy)_0520386045841416` | boolean · Dimension | Basic | — | — | 1 |
| **2 T\|F**<br>`1 T\|F (copy)_3967076110458899` | boolean · Dimension | Basic | — | — | 1 |
| **2-Table T\|F**<br>`Viz T\|F (copy)_2196584473067564` | boolean · Dimension | Basic | — | — | 1 |
| **3 T\|F**<br>`2 T\|F (copy)_3967076110503956` | boolean · Dimension | Basic | — | — | 1 |
| **3-Pie T\|F**<br>`Viz T\|F (copy) (copy)_2196584473075757` | boolean · Dimension | Basic | — | — | 1 |
| **A_Color_Icons**<br>`Calculation_3958664101442371614` | string · Dimension | Basic | — | — | 11 |
| **Center**<br>`Calculation_3032048483372380162` | spatial · Measure | Basic | — | — | 1 |
| **Customer**<br>`Order Details (copy)_2196584460210212` | string · Dimension | Basic | Customer ID | — | 1 |
| **CY Customers**<br>`CY Orders (copy)_2196584417636365` | integer · Measure | Basic | Year, Max Order Date, Customer ID | # Change (C), % Change (C), % Change +\|- (C), CY vs PY Symbol (C) | 3 |
| **CY Delivery Days**<br>`CY Fullfilment Days (copy)_2179824137834508` | real · Measure | Basic | Year, Max Order Date, Fixed Order D Days | # Change (D), % Change (D), % Change +\|- (D), CY vs PY Symbol (D) | 2 |
| **CY Fullfilment Days**<br>`Calculation_2179824136896521` | real · Measure | Basic | Year, Max Order Date, Fixed Order F Days | # Change (F), % Change (F), % Change +\|- (F), CY vs PY Symbol (F) | 2 |
| **CY or PY**<br>`Calculation_3314086408456491009` | string · Dimension | Basic | Order Date, Max Order Date | Placement, Placement 2, Placement 3, Region CY/PY … | 2 |
| **CY Orders**<br>`CY Profit (copy)_2196584416010251` | integer · Measure | Basic | Year, Max Order Date, Order ID | # Change (O), % Change (O), % Change +\|- (O), CY vs PY Symbol (O) | 3 |
| **CY Profit**<br>`CY Sales (copy)_1622421828877811712` | real · Measure | Basic | Year, Max Order Date, Profit | $ Change (P), % Change (P), % Change +\|- (P), CY vs PY Symbol (P) | 2 |
| **CY Returns**<br>`CY Orders (copy)_2355463910199299` | integer · Measure | Basic | Year, Max Order Date | # Change (R), % Change (R), % Change +\|- (R), CY vs PY Symbol (R) | 3 |
| **CY Sales**<br>`Calculation_99360729833828364` | real · Measure | Basic | Year, Max Order Date, Sales | $ Change, % Change, % Change +, % Change +\|- … | 6 |
| **CY vs PY Symbol**<br>`PY Sales (copy)_99360729837473806` | string · Measure | Basic | CY Sales, PY Sales | — | 1 |
| **CY vs PY Symbol (C)**<br>`CY vs PY Symbol (O) (copy)_2196584445612051` | string · Measure | Basic | CY Customers, PY Customers | — | 1 |
| **CY vs PY Symbol (D)**<br>`CY vs PY Symbol (F) (copy)_1358763908976647` | string · Measure | Basic | CY Delivery Days, PY Delivery Days | — | 1 |
| **CY vs PY Symbol (F)**<br>`CY vs PY Symbol (O) (copy)_1358763907317761` | string · Measure | Basic | CY Fullfilment Days, PY Fullfilment Days | — | 1 |
| **CY vs PY Symbol (O)**<br>`CY vs PY Symbol (P) (copy)_2196584418390030` | string · Measure | Basic | CY Orders, PY Orders | — | 1 |
| **CY vs PY Symbol (P)**<br>`CY vs PY Symbol (copy)_2196584413700096` | string · Measure | Basic | CY Profit, PY Profit | — | 1 |
| **CY vs PY Symbol (R)**<br>`CY vs PY Symbol (C) (copy)_2117149702713347` | string · Measure | Basic | CY Returns, PY Returns | — | 1 |
| **D1 T\|F**<br>`M1 T\|F (copy)_2408522753552390` | boolean · Dimension | Basic | — | — | 1 |
| **D2 T\|F**<br>`D1 T\|F (copy)_2408522753765383` | boolean · Dimension | Basic | — | — | 1 |
| **Days Toggle**<br>`Dynamic Category (copy)_2408522741055491` | integer · Dimension | Basic | Fullfilment Days, Delivery Days | — | 2 |
| **Delivery**<br>`Shipping (copy)_2179824135229446` | string · Dimension | Basic | Delivery Days, Est Delivery Date | — | 1 |
| **Delivery Days**<br>`Days to Ship (copy)_2179824131891204` | integer · Measure | Basic | Ship Mode, Fullfilment Days | Days Toggle, Delivery, Fixed Order D Days | 5 |
| **Dynamic Category**<br>`Dynamic Category (copy)_3967076092485646` | string · Dimension | Basic | Ship Mode, Segment, Region | Fixed Max - Dynamic Metric | 36 |
| **Dynamic Category 2**<br>`Dynamic Category (copy)_2105055365771264` | string · Dimension | Basic | Region, Category, Sub-Region | Fixed Max - Dynamic Metric | 27 |
| **Dynamic Category 2 Name**<br>`Dynamic Category 2 (copy)_0520386027536385` | string · Dimension | Basic | — | — | 2 |
| **Dynamic Category 3**<br>`Dynamic Category 2 (copy)_2105055369719809` | string · Dimension | Basic | Category, Sub-Category, State/Province | — | 4 |
| **Dynamic Category 3 Name**<br>`Dynamic Category 3 (copy)_0520386027630594` | string · Dimension | Basic | — | — | 2 |
| **Dynamic Color**<br>`Dynamic Category (copy)_0520386035060739` | string · Dimension | Basic | Region | — | 1 |
| **Dynamic Metric**<br>`Dynamic Category (copy)_1561347269570564` | real · Measure | Basic | Sales, Order ID | — | 1 |
| **Dynamic Metric $**<br>`Dynamic Metric Name (copy)_1561347286855697` | string · Dimension | Basic | — | — | 4 |
| **Dynamic Metric (distinct)**<br>`Dynamic Metric (copy)_1420335861870603` | real · Measure | Basic | Sales, Order ID | — | 3 |
| **Dynamic Title**<br>`Dynamic Category (copy)_0520386038878214` | string · Dimension | Basic | — | — | 1 |
| **Est Delivery Date**<br>`Delivery Days (copy)_2179824134582277` | date · Dimension | Basic | Ship Mode | Delivery | 1 |
| **Fixed Order D Days**<br>`Calculation_2179824136265735` | integer · Measure | LOD | Order ID, Delivery Days | CY Delivery Days, PY Delivery Days | 2 |
| **Fixed Order F Days**<br>`Fixed Order D Days (copy)_2179824136626184` | integer · Measure | LOD | Order ID, Fullfilment Days | CY Fullfilment Days, PY Fullfilment Days | 2 |
| **Fullfilment Days**<br>`Calculation_2179823979888640` | integer · Measure | Basic | Order Date | Days Bin, Days Toggle, Delivery Days, Fixed Order F Days … | 7 |
| **Item #**<br>`Calculation_1043146293700063235` | integer · Measure | Table calc | — | — | 1 |
| **Item Count**<br>`Quantity (copy)_2196584462286885` | integer · Measure | LOD | Order ID, Quantity | — | 1 |
| **Location**<br>`Shipping Details (copy)_2196584457977890` | string · Dimension | Basic | City, State Abbv, Region | — | 1 |
| **M1 T\|F**<br>`M1 T\|F (copy)_1420335850835975` | boolean · Dimension | Basic | — | — | 2 |
| **M2 T\|F**<br>`1 T\|F (copy)_1420335850352646` | boolean · Dimension | Basic | — | — | 2 |
| **Max Order Date**<br>`Calculation_99360729564491781` | date · Dimension | LOD | Order Date | -PY Sales, CY, CY Customers, CY Delivery Days … | 53 |
| **Order**<br>`Calculation_2196584454078494` | string · Dimension | Basic | Order ID, Order Date | — | 1 |
| **Placement**<br>`Calculation_3314086408456806402` | integer · Measure | Basic | CY or PY | — | 1 |
| **PY Customers**<br>`PY Orders (copy)_2196584445612050` | integer · Measure | Basic | Year, Max Order Date, Customer ID | # Change (C), % Change (C), % Change +\|- (C), CY vs PY Symbol (C) | 3 |
| **PY Delivery Days**<br>`PY Fullfilment Days (copy)_2179824137834507` | real · Measure | Basic | Year, Max Order Date, Fixed Order D Days | # Change (D), % Change (D), % Change +\|- (D), CY vs PY Symbol (D) | 2 |
| **PY Fullfilment Days**<br>`CY Fullfilment Days (copy)_2179824137199626` | real · Measure | Basic | Year, Max Order Date, Fixed Order F Days | # Change (F), % Change (F), % Change +\|- (F), CY vs PY Symbol (F) | 2 |
| **PY Orders**<br>`CY Orders (copy)_2196584417505292` | integer · Measure | Basic | Year, Max Order Date, Order ID | # Change (O), % Change (O), % Change +\|- (O), CY vs PY Symbol (O) | 3 |
| **PY Profit**<br>`PY Sales (copy)_2196584414089218` | real · Measure | Basic | Year, Max Order Date, Profit | $ Change (P), % Change (P), % Change +\|- (P), CY vs PY Symbol (P) | 2 |
| **PY Returns**<br>`PY Orders (copy)_2355463910211588` | integer · Measure | Basic | Year, Max Order Date | # Change (R), % Change (R), % Change +\|- (R), CY vs PY Symbol (R) | 3 |
| **PY Sales**<br>`CY (copy)_99360729834057741` | real · Measure | Basic | Year, Max Order Date, Sales | $ Change, % Change, % Change +, % Change +\|- … | 3 |
| **Rolling 12 Months**<br>`Calculation_99360729563422724` | string · Dimension | Basic | Order Date, Max Order Date | — | 7 |
| **Rolling 24 Months**<br>`Rolling 12 Months (copy)_3314086408457809924` | string · Dimension | Basic | Order Date, Max Order Date | — | 53 |
| **Sales (No Format)**<br>`Sales (copy)_1420335848169474` | real · Measure | Basic | Sales | — | 1 |
| **Shipping**<br>`Order Detail* (copy)_2196584455655455` | string · Dimension | Basic | Ship Mode | — | 1 |
| **Show Breakout**<br>`Calculation_1561347302473748` | boolean · Dimension | Basic | — | Show Breakout T\|F | 1 |
| **State Abbv**<br>`X (copy)_99360729846267924` | string · Dimension | Basic | State/Province | Location | 1 |
| **Sub-Region**<br>`Calculation_2105055583731714` | string · Dimension | Basic | State/Province | Dynamic Category 2 | 28 |
| **Total Sales**<br>`Item Count (copy)_2196584462667814` | real · Measure | LOD | Order ID, Sales | — | 1 |
| **Year**<br>`Calculation_99360729557819395` | integer · Dimension | Basic | Order Date | -PY Sales, CY, CY Customers, CY Delivery Days … | 22 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| **Dynamic Metric**<br>`Dynamic Category (copy)_1561347270422533` | string · list | `"Orders"` | list of 2 — "Sales" → Sales, "Orders" → Orders |
| **Top Customers**<br>`Parameter 1` | integer · range | `5` | range 5 to 20, step 5 |
| **Profit Bin Size**<br>`Parameter 2` | integer · range | `200` | range 50 to 200, step 50 |
| **100**<br>`Parameter 3` | real · any | `100.` | any value |
| **View**<br>`Parameter 4` | integer · list | `1` | list of 3 — 1 → Sales Metrics, 2 → Order Details, 3 → Distribution Pie |
| **Show Breakout P**<br>`Parameter 5` | boolean · list | `false` | list of 2 — true, false |
| **Dynamic Category**<br>`View (copy)_2196584468738089` | string · list | `"Shipping"` | list of 3 — "Shipping" → Shipping, "Product" → Product, "Location" → Location |
| **Days**<br>`View (copy)_2408522740838401` | string · list | `"Delivery"` | list of 2 — "Fullfilment" → Fullfilment Days, "Delivery" → Delivery Days |

### How the numbers are computed

**Level-of-detail expressions — 5.** These fix their own level of detail, so they return the same value regardless of what dimensions are in the view:

> `Fixed Order D Days`, `Max Order Date`, `Fixed Order F Days`, `Total Sales`, `Item Count`

**Table calculations — 1.** These compute across the rendered table, so their result depends on what is in the view at the time:

> `Item #`

The workbook file records only each table calculation's `ordering-type`. The addressing and partitioning described elsewhere in this document are **derived from each sheet's level-of-detail shelf**, not read from the file.

## Appendix A — Worksheet index

| Group | Sheets |
|---|---|
| KPI text cards | `KPI Sales T`, `KPI Profit T`, `KPI Orders T`, `KPI Customers T`, `KPI Returns T`, `KPI FDays T`, `KPI DDays T` |
| KPI sparklines | `KPI Sales C`, `KPI Profit C`, `KPI Orders C`, `KPI Customers C`, `KPI Returns C`, `KPI FDays C`, `KPI DDays C` |
| Breakdown bars | `A: Bar Sales`, `A: Bar Orders`, `A: Bar Customers`, `A: Bar Returns` |
| Days comparison | `A3: Bar`, `A3: Table` |
| Distribution | `A2: Map`, `A2: Jitterplot`, `A2: Dotplot`, `A2: Bar`, `A2: Table`, `A2+: Jitter Tooltip` |
| Pie view | `C: Pie A`, `C: Table` |
| Step build | `D: Step 1`–`D: Step 4` |
| Table view | `B: Order Details Table` |
| Navigation | `Nav: Charts`, `Nav: Pie`, `Nav: Table`, `Nav2: Sales`, `Nav2: Orders`, `Nav3: F`, `Nav3: D`, `Nav4: Shipping`, `Nav4: Product`, `Nav4: Location` |
| Chrome | `Title`, `Sym`, `DL`, `DL Color`, `PDF`, `Image`, `LinkedIn`, `Public`, `Twitter` |
| Reference | `Map` |

## Appendix B — Calculated fields excluded from this documentation

66 calculated fields exist in the data pane but are not referenced by any worksheet or
dashboard logic. They belong chiefly to the `Plot`, `Comparison Bar`, `Stagger Bar`,
`Sunburst Explosion` and `Calculation Examples` folders — chart experiments retained from
earlier iterations — together with the unused halves of several CY/PY families
(`% Change +`, `% Change -`, `MoM % Change`, `Last Month Sales`, `CY Sales %`, `PY Sales %`,
`-PY Sales`, `Region CY/PY`, `Placement 2`, `Placement 3`, `Placement Year`,
`Placement Month`, `Region Year`, `Region Month`, `Sales Furniture`, `Sales Technology`,
`Sales Office Supplies` and their `%` variants, `CountD`, `Dummy Count`,
`Avg Dummy Count (for Qtrs)`, `Month Order Date`, `Order_Days`, `Fixed Max - Dynamic Metric`,
`Sub-Category Abbv`, `Category Abbv`, `Ship Mode Abbv`, `Region Abbv`, and related helpers).

## Appendix C — Package inventory

| File | Size |
|---|---:|
| `Superstore Shipping Metrics.twb` | ~1.5 MB |
| `Data/Extracts/Sample _ Superstore.hyper` | embedded |
| **Total `.twbx`** | **1.7 MB** |

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas, colour values and settings are quoted verbatim
from the workbook definition; rendered figures are read from the published view.*
