# Viz Design Catalog (Vol. I–IV) — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `Viz Design Catalog` — a **single workbook** containing all four volumes |
| Author | John Johansson (`john.johansson`) |
| Recognition | Tableau **Viz of the Day** (`#VOTD`, Vol. I) |
| Published as | Four separate Tableau Public entries, one per dashboard (see §2) |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, the full chart-technique index, dashboard layout, interactivity and design system |
| Source of record | `VizDesignCatalog.twbx` → `Viz Design Catalog Vol. I  #VOTD.twb`, decompiled and parsed field-by-field |
| Explicit exclusion | 25 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic**. They are listed by name only in Appendix B. |
| Companion file | `documentation_viz_VizDesignCatalog_README.md` — same facts, condensed overview format |

> **One workbook, four publications.** Vol. I–IV are four dashboards inside one `.twb`. Each is
> published separately on Tableau Public (four repo URLs, four view counts), but all four
> entries are the same ~4.25 MB file with a different default view. Editing any technique in
> any volume means editing one workbook. This document therefore covers all four.

---

## 1. Executive summary

**Viz Design Catalog** is a reference library of chart-construction techniques, not an
analytical dashboard. Across four volumes it builds roughly **85 distinct chart types** —
every one on the same Sample - Superstore data — so that the data is held constant and the
*technique* is the subject.

Each volume is a 1600 × 2000 poster divided into labelled sections (KPIs, Bars, Pies, Lines,
Radials, Maps, Funnels and so on), with three or four worked examples per section. The result
is a visual index: a reader looking for "how do I build a coxcomb chart" can see the output
and, with this document, go straight to the sheet and the calculation behind it.

The technical substance lives in the **calculation library**. Roughly a dozen reusable
patterns — radial trigonometry, polar/coxcomb geometry, funnel stage counting, gauge slices,
hand-coded grid coordinates, running-total waterfalls — are what generate the more exotic
charts. Those patterns are the transferable part, and §6 documents each one.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | **132** |
| Dashboards | **4** (`Vol_I`, `Vol_II`, `Vol_III`, `Vol_IV`) |
| Data sources | 1 (Sample - Superstore, 3 joins) |
| Parameters | 9 |
| Total fields in data pane | 138 calculated + base Superstore fields |
| Calculated fields **used** | 113 named + 141 in-view ad-hoc |
| Calculated fields unused (not documented) | 25 |
| Chart techniques catalogued | ~85 |
| Dashboard canvas | 1600 × 2000 px each, fixed |
| Global font | Arial |
| Dashboard actions | 28 URL actions (volume navigation, social, palette download) |
| Dynamic Zone Visibility bindings | 0 |
| Custom colour palettes | 2 (`CB_BuPu`, `DC_Green2Grey`) |

### 1.2 The four volumes

| Volume | Sheets | Sections |
|---|---:|---|
| **Vol. I** | 37 | KPIs · Bars · Pies · Lines · Percentage Charts · Shape Grids · Misc. |
| **Vol. II** | 31 | Radials · Bars · Dots · Plots · Groups · Tables |
| **Vol. III** | 30 | Bars · Line Bars · More Bars · Maps · Pies · Misc. |
| **Vol. IV** | 45 | KPIs · Bars · Funnels · Lines · Combo Charts · Misc. |

Sheet counts sum to more than 132 because the four navigation sheets (`BV-I`…`BV-IV`) and the
four chrome sheets (`LinkedIn`, `Public`, `Twitter`, `DL Color`) appear on every volume.

---

## 2. Publication metadata

All four entries are the same workbook file.

| | Vol. I | Vol. II | Vol. III | Vol. IV |
|---|---|---|---|---|
| Repo URL | `VizDesignCatalog` | `VizDesignCatalogVol_II` | `VizDesignCatalogVol_III` | `VizDesignCatalogVol_IV` |
| Workbook ID | 17207455 | 17222679 | 17291084 | 17421087 |
| Default view | `Vol_I` | `Vol_II` | `Vol_III` | `Vol_IV` |
| Revision | 3.3 | 1.7 | 1.5 | 1.3 |
| First published | 2025-07-03 | 2025-07-07 | 2025-07-22 | 2025-08-22 |
| Last published | 2025-08-24 | 2025-08-24 | 2025-12-10 | 2025-08-24 |
| Size (bytes) | 4,249,421 | 4,250,112 | 4,250,482 | 4,250,356 |
| Views | 31,779 | 10,318 | 19,329 | 8,110 |
| Favourites | 242 | 84 | 145 | 84 |

**Direct URLs**

- Vol. I — https://public.tableau.com/app/profile/john.johansson/viz/VizDesignCatalog/Vol_I
- Vol. II — https://public.tableau.com/app/profile/john.johansson/viz/VizDesignCatalogVol_II/Vol_II
- Vol. III — https://public.tableau.com/app/profile/john.johansson/viz/VizDesignCatalogVol_III/Vol_III
- Vol. IV — https://public.tableau.com/app/profile/john.johansson/viz/VizDesignCatalogVol_IV/Vol_IV

**Published description** (Vol. I; the others differ only in volume number and hashtags)

> Design Catalog Volume I of a IV part series illustrating my developer capabilities and
> knowledge. Use the navigation at the top right to toggle between volumes.
> #Catalog #Design #Color #KPI #Bars #Pies #Lines #VOTD #VizOfTheDay

---

## 3. Data architecture

### 3.1 Connection and model

| Property | Value |
|---|---|
| Data source | `Sample - Superstore` |
| Connection class | `excel-direct` behind a `federated` wrapper |
| Relations | 4 (one collection + `Orders`, `People`, `Returns`) |
| Joins | **3** |
| Custom SQL | None |
| Materialisation | Extract, last updated 2025-07-18 18:21 |
| Data source filters | **None** |

### 3.2 Materialised schema (15 columns)

The extract is deliberately narrow — only what the catalogue actually plots:

| Column | Role in the catalogue |
|---|---|
| `Order ID` | Order counting; funnel stages; radial path |
| `Order Date` | Every time axis; the radial angle; calendar and grid coordinates |
| `Ship Date` | Gantt duration |
| `Ship Mode` | Trellis columns; Gantt rows; coxcomb path binning |
| `Country/Region`, `State/Province` | Maps, hex map, triangle grid |
| `Region` | The primary categorical dimension — colour on most charts |
| `Category`, `Sub-Category` | Secondary and tertiary breakdowns; treemap, sunburst, word cloud |
| `Product Name` | Word cloud, packed bubble |
| `Sales` | The primary measure almost everywhere |
| `Quantity` | Histogram bins, jitterplot |
| `Profit` | Profit ratio, diverging bar, candlestick percentiles |
| `Region` (People), `Order ID` (Returns) | Join keys |

### 3.3 Grain

Order-line grain, as with any Superstore build. Because this is a catalogue rather than an
analysis, the grain matters less than usual — most sheets aggregate with `SUM([Sales])` at a
coarse level. Where exact counts matter (funnels, KPI order counts) the workbook uses
`{FIXED}` rollups documented in §6.6.

---

## 4. Parameters

Nine parameters. Seven exist purely to drive the geometry of the radial and half-radial
charts; only two are general-purpose controls.

| Caption | Type | Domain | Default | Role |
|---|---|---|---|---|
| **\*Angle, Start** | integer range | 2 – 360 | `2` | Start angle of the full radial sweep |
| **\*Angle, End** | integer range | from 0 | `360` | End angle of the full radial sweep |
| **\*Hole** | real | any | `2.` | Inner radius of the full radial |
| **\*Angle, Start Half** | integer range | 2 – 180 | `2` | Start angle of the half-radial sweep |
| **\*Angle, End Half** | integer range | from 0 | `178` | End angle of the half-radial sweep |
| **\*Hole 2** | real | any | `10.` | Inner radius of the half radial |
| **Top Customers** | integer range | 5 – 20, step 5 | `5` | Top-N control |
| **Profit Bin Size** | integer range | 50 – 200, step 50 | `200` | Histogram bin width |
| **100** | real | any | `100.` | Scaling constant |

The paired full/half geometry parameters are what let the same trigonometric pattern produce
both the closed ring (Vol. II *Radial*) and the半 arc (Vol. II *Half Sun Radial*) without
duplicating the maths — the formulas are identical, only the parameter set differs.

---

## 5. Field dictionary — the workhorse fields

| Field | Formula | Used in | Purpose |
|---|---|---:|---|
| **Max Order Date** | `DATE({ FIXED :max([Order Date])})` | **87 sheets** | The dynamic time anchor. Every current-year / prior-year split is relative to this, so the whole catalogue re-bases on a data refresh |
| **Year** | `YEAR([Order Date])` | 34 | Comparison key |
| **Rolling 12 Months** | `if DATETRUNC('month',[Order Date]) > DATETRUNC('month', DATEADD('month',-12,[Max Order Date])) then "Last 12 Months" else "Older" END` | **56 sheets** | The default window for most charts |
| **Rolling 24 Months** | Same with `-24` | 2 | Two-year window where PY comparison is shown |
| **CY Sales** | `IF [Year] == YEAR([Max Order Date]) THEN [Sales] END` | **26 sheets** | Current-year measure |
| **PY Sales** | `IF [Year] == (YEAR([Max Order Date])-1) THEN [Sales] END` | 16 | Prior-year measure |
| **CY or PY** | Three-way period label (`CY` / `PY` / `Older`) | 8 | Colour split on comparison charts |
| **Index** | `INDEX()-1` | 9 | The radial and unit-bar positioning driver |
| **Order Date Number** | `INT([Order Date])` | 8 | Date as an integer — the radial angle input |
| **Total Orders** | `COUNT([Order ID])` | 8 | Funnel denominator |

---

## 6. The calculation library

113 named calculations are in use. The reusable patterns are grouped below — these are the
transferable part of the catalogue.

### 6.1 Current year vs prior year

The standard comparison family, used across the KPI cards and comparison bars:

```
CY Sales           IF [Year] == YEAR([Max Order Date])     THEN [Sales] END
PY Sales           IF [Year] == (YEAR([Max Order Date])-1) THEN [Sales] END
$ Change           SUM([CY Sales]) - SUM([PY Sales])
% Change           (SUM([CY Sales]) - SUM([PY Sales])) / SUM([PY Sales])
% Change +         IF … > 0 THEN … ELSE NULL END     ← positive values only
% Change -         IF … < 0 THEN … ELSE NULL END     ← negative values only
% Change +|-       IF … > 0 THEN '+' ELSE '-' END
% Change T+|F-     IF … > 0 THEN TRUE ELSE FALSE END
CY vs PY Symbol    ▲ / ▼ / ▶
CY vs PY Symbol +  ▲ only (null otherwise)
CY vs PY Symbol -  ▼ only (null otherwise)
```

Splitting positive and negative into separate fields (`% Change +` / `% Change -`, and the
matching `Symbol +` / `Symbol -`) is what allows a KPI card to colour the up-case green and
the down-case red without a dual-axis trick — each field is placed on its own text mark and
formatted independently.

```
CY Sales %   SUM([CY Sales]) / TOTAL(SUM([CY Sales]))     ← table calc, percentage bars
PY Sales %   SUM([PY Sales]) / TOTAL(SUM([PY Sales]))
-PY Sales    IF [Year] == (YEAR([Max Order Date])-1) THEN ([Sales]*-1) END   ← population chart
CY-PY -Diff  SUM([PY Sales]) - SUM([CY Sales])
```

### 6.2 Radial geometry — full and half

The pattern behind *Spiral Chart*, *Radial*, *Radial Stacked* (Vol. II) and *Half Sun Radial*.

**Step 1 — date to angle**
```
Order Date Number = INT([Order Date])

Angle = 2 * PI() * ([Order Date Number] - 45290) / ({MAX([Order Date Number])} - 45290)
```
`45290` is the serial number for the start of the plotted year; the commented-out line above
it in the workbook preserves the previous year's constant (`44197`), which is how the author
re-bases the radial between years.

**Step 2 — compress into the parameterised arc**
```
Angle Adjusted = ( [Angle] * (([*Angle, End] - [*Angle, Start]) / 360)
                 + ([*Angle, Start] * 2 * PI() / 360) )
```

**Step 3 — polar to Cartesian**
```
X cos = ([Index] + [*Hole]) * COS(MIN([Angle Adjusted]))
Y sin = ([Index] + [*Hole]) * SIN(MIN([Angle Adjusted]))
```

**The half-radial variant** is the identical chain against the half-geometry parameters:
```
Angle Half          same formula as Angle
Angle Adjusted Half ( [Angle Half] * (([*Angle, End Half] - [*Angle, Start Half]) / 360)
                    + ([*Angle, Start Half] * 2 * PI() / 360) )
X cos Half   = ([Index] + [*Hole 2])     * COS(MIN([Angle Adjusted Half]))
Y sin Half   = ([Index] + [*Hole 2])     * SIN(MIN([Angle Adjusted Half]))
X cos Half-S = ([Index] + [*Hole 2] - 1) * COS(MIN([Angle Adjusted Half]))   ← inner ring
Y sin Half-S = ([Index] + [*Hole 2] - 1) * SIN(MIN([Angle Adjusted Half]))
```
The `-S` pair sits one unit further in, producing the filled sun body beneath the rays.

### 6.3 Polar area and coxcomb geometry

A different construction from the radial — this one builds **polygons**, not points, so each
wedge is a filled shape. Behind *Polar Area Chart* and *Coxcomb Chart* (Vol. III).

```
P_Date Part        WINDOW_MAX(MAX(DATEPART("month",[Order Date])))
P_Sales            WINDOW_SUM(SUM([Sales]))
P_Total Sales      WINDOW_SUM(MAX({SUM([Sales])}))
P_Percentage       [P_Sales] / [P_Total Sales]
P_Percentage_Adj   [P_Percentage] / WINDOW_MAX([P_Percentage])   ← normalise to 0–1
P_Starting Point   ([P_Date Part] - 1) * 360 / 12                ← month → degrees
P_Size             3.6 / 12                                       ← degrees per path step

Polar X  IF [Index-2] = -1 THEN 0
         ELSE SIN(RADIANS([Index-2]*[P_Size] + [P_Starting Point])) * [P_Percentage_Adj] END
Polar Y  IF [Index-2] = -1 THEN 0
         ELSE COS(RADIANS([Index-2]*[P_Size] + [P_Starting Point])) * [P_Percentage_Adj] END
```

The `Index-2 = -1 THEN 0` branch is the key detail: it pins the first point of every polygon
to the origin, which is what closes each wedge into a proper pie slice rather than leaving an
open arc.

Supporting fields:
```
Path      IIF([Ship Mode]="First Class", 1, 102)   ← densification endpoints
Path bin  [Path]                                    ← binned to generate intermediate points
Index-2   INDEX()-2
```
`Path` / `Path bin` is the standard polygon-densification trick: binning a field whose domain
runs 1–102 forces Tableau to generate 102 rows per wedge, giving the polygon enough vertices
to render as a smooth arc.

### 6.4 Gauge slices

Behind *Percentage Gauge* (Vol. I). A gauge is a pie with five carefully sized wedges:

```
Percentage %  0.67                                          ← the value being shown
% Slice 1     IF [Percentage %] > 0.5 THEN [Percentage %] - 0.5 ELSE 0 END
% Slice 2     IF [Percentage %] > 0.5 THEN 1 - [Percentage %] ELSE 0 END
% Slice 3     1                                             ← the hidden lower half
% Slice 4     IF [Percentage %] < 0.5 THEN [Percentage %] ELSE 0.5 END
% Slice 5     IF [Percentage %] < 0.5 THEN 0.5 - [% Slice 4] ELSE 0 END
```

Slice 3 is a full-width wedge rendered in the background colour, which is what turns a
complete pie into a half-circle gauge. Slices 1/2 and 4/5 handle the two cases either side of
the halfway mark, so the filled arc grows correctly whether the value is above or below 50 %.

### 6.5 Hand-coded grid coordinates

Three grid charts place marks at hand-specified positions rather than deriving them.

**Hex Map** (Vol. I) — every US state gets an X and Y:
```
X  Case [State/Province] When 'Alabama' Then 7.5 When 'Alaska' Then 0.5
                         When 'Arizona' Then 3   When 'California' Then 2 … END
Y  Case [State/Province] When 'Alabama' Then 6   When 'Alaska' Then 0
                         When 'Arizona' Then 5   When 'California' Then 5 … END
State Abbv  Case [State/Province] When 'Alabama' Then 'AL' … END
```
Plotted as a Shape mark on a hexagon shape file. There is no geographic projection involved —
the layout is a designed arrangement that preserves rough geography while giving every state
equal visual weight.

**Triangle Rank Grid** (Vol. I) — a pyramid of ranked states:
```
Rank Sales  RANK(SUM([Sales]))
Row Rank    FLOAT(IF [Rank Sales] = 1 Then 1 ELSEIF [Rank Sales] <= 3 Then 2
                  ELSEIF [Rank Sales] <= 6 Then 3 ELSEIF [Rank Sales] <= 10 Then 4
                  ELSEIF [Rank Sales] <= 15 Then 5 ELSEIF [Rank Sales] <= 21 Then 6
                  ELSEIF [Rank Sales] <= 28 Then 7 … END)
Col Rank    FLOAT(IF [Rank Sales] = 1 Then 6 ELSEIF [Rank Sales] = 2 Then 5.5
                  ELSEIF [Rank Sales] = 3 Then 6.5 … END)
```
`Row Rank` uses triangular numbers (1, 3, 6, 10, 15, 21, 28…) as its thresholds — each row of
the pyramid holds one more item than the row above. `Col Rank` then places each rank at a
half-step offset so the rows interlock.

### 6.6 Funnel stages

Behind *Funnel*, *Half Funnel* and *Donut Funnel* (Vol. IV). Each stage is a progressively
narrower filter, counted as a 1/0 flag:

```
Total Orders  COUNT([Order ID])

Funnel 2   IF [Region] = 'West' OR [Region] = 'East' THEN 1 ELSE 0 END
Funnel 3   IF ([Region] = 'West' OR [Region] = 'East')
              AND [Category] = 'Office Supplies' THEN 1 ELSE 0 END
Funnel 4   IF ([Region] = 'West' OR [Region] = 'East')
              AND [Category] = 'Office Supplies'
              AND ([Sub-Category] = 'Binders' OR [Sub-Category] = 'Paper') THEN 1 ELSE 0 END

Funnel 1 %   ([Total Orders]) / ([Total Orders])      ← always 1, the 100 % stage
Funnel 2 %   SUM([Funnel 2]) / ([Total Orders])
Funnel 3 %   SUM([Funnel 3]) / ([Total Orders])
Funnel 4 %   SUM([Funnel 4]) / ([Total Orders])
Funnel n -%  1 - ([Funnel n %])                       ← the complement, for the empty portion
```

The rendered funnel reads 3,379 → 2,077 → 1,218 → 588 orders, i.e. 100 % → 61 % → 36 % → 17 %.
Each `-%` complement field is what draws the pale "lost" portion beside each filled stage.

### 6.7 Winged donut

Behind *Winged Donut* (Vol. IV) — a donut showing attainment against a target, flanked by bars:

```
Sales Target         SUM([Sales]) * 1.21
Sales Target Diff    ([Sales Target]) - SUM([Sales])
Sales Target Diff %  (SUM([Sales]) / ([Sales Target]))
Fixed Total Region   { FIXED [Year], [Region] : SUM([Sales]) }
```
Rendered as 83 % attainment ($746K of $902K). The `Fixed Total Region` LOD gives each wing bar
its regional total independent of the donut's level of detail.

### 6.8 Waterfall

```
- Sales  -[Sales]
```
Combined with a `RUNNING_SUM(SUM([Sales]))` on rows and a Gantt bar mark sized by the negated
measure. The Gantt bar is the essential choice: its size extends *downward* from its position,
which is exactly what a waterfall step needs.

### 6.9 Candlestick and soundwave

```
*candle top     MAX([Profit]) - PERCENTILE([Profit],.99)
*candle bottom  MIN([Profit]) - PERCENTILE([Profit],.01)
Candle          IF YEAR([Order Date]) = 2024 THEN [Sales] END
```
The percentile offsets define the wick extents, clipping the top and bottom 1 % so outliers
do not stretch the chart.

```
Sales *-1  -[Sales]     (soundwave mirroring — see Appendix B, currently unused)
```

### 6.10 Shape-bar and grid scaling

```
Fixed Total Orders  { FIXED :COUNT([Order ID]) }
Total Year Orders   { FIXED [Year] : COUNT([Order ID]) }
Year Orders %       ROUND(([Total Year Orders] / [Fixed Total Orders]), 2) * 100
Fixed Total Sales   { FIXED [Year] : SUM([Sales]) }
Fixed Category Sales{ FIXED [Year], [Category] : SUM([Sales]) }
Index-1             INDEX()-1
Index+1 Shapex10    IF CONTAINS(Str([Index-1]),'5') or CONTAINS(Str([Index-1]),'10')
                    THEN TRUE ELSE FALSE END
```
`Index+1 Shapex10` marks every fifth and tenth unit so a unit chart can highlight decade
gridlines without a separate axis.

### 6.11 Comparison and stagger bars

```
CY or PY        three-way period label
Placement       IIF([CY or PY]='CY', 2, 1)          ← CY sits right of PY
Placement 2     IIF([CY or PY]='CY', 4, 1)          ← wider gap variant (slope chart)
Placement Year  IIF([Year]=2024,4, IIF([Year]=2023,3, IIF([Year]=2022,2,1)))
Region CY/PY    [Region] + ' ' + [CY or PY]         ← composite colour key
Region Year     STR([Year]) + ' ' + [Region]
```
Putting a numeric placement field on columns *beside* the dimension is what staggers the bars
into offset pairs and quartets rather than stacking or side-by-siding them.

### 6.12 Miscellaneous technique helpers

| Field | Formula | Technique |
|---|---|---|
| **Box Color** | `SUM([Percentage %]) >= SUM([Percentage])` | Waffle chart fill test |
| **\*Win Loss** | `IF COUNT(Orders) > 100 THEN 1 ELSEIF < 100 THEN -1 ELSE 0 END` | Win-loss bar |
| **Days** | `DATEDIFF('day',[Order Date],[Ship Date])` | Gantt duration |
| **Day Name** | `[Order Date]` | Square grid / calendar alias |
| **Day Today T\|F** | `DAY([Order Date]) = DAY(TODAY())` | Calendar "today" highlight |
| **Profit Ratio** | `SUM([Profit])/SUM([Sales])` | Donut ratio, KPI containers |
| **Profit to Sales Rate** | `IF SUM([Sales])-SUM([Profit]) > 0 then SUM([Sales])-SUM([Profit]) else 0 end` | Donut ratio remainder |
| **Center** | `MAKEPOINT(0,0)` | Sunburst explosion anchor |
| **Rank Sales** | `RANK(SUM([Sales]))` | Triangle grid, bump chart |
| **Category Abbv / Region Abbv / Sub-Category Abbv / Ship Mode Abbv** | `UPPER(LEFT(field, n))` | Compact axis labels |
| **A_Color / B_Color / C_Color** | `'A'` / `'B'` / `'C'` | Single-member dimensions forcing a fixed mark colour |

---

## 7. Chart technique index

The catalogue itself. For each technique: the section it appears in, the worksheet that builds
it, the mark type, and the construction that makes it work.

### 7.1 Volume I — Foundations

| Section | Technique | Sheet | Mark | Construction |
|---|---|---|---|---|
| KPIs | KPI Card & Bar | `KPI Card 3.1`, `KPI 3.2`, `KPI Card 3.3` | Bar/Text | Value text beside category bars with grey track |
| KPIs | KPI Card | `KPI Card` | Text | `SUM(CY Sales)` + `CY vs PY Symbol` + `% Change` |
| KPIs | KPI Card & Line | `KPI Card 2.1`, `KPI Card 2.2` | Text + Line/Area | Card with CY/PY sparkline |
| Bars | Bar | `Bar` | Bar | Stacked bar by month, coloured by Sub-Category |
| Bars | Bar Dual | `Bar Dual` | Bar ×2 | CY bar over PY reference bar |
| Bars | Bar Curved | `Bar Curved` | **Line** | Bars drawn as lines with `:Measure Names` on Path — the curve comes from path interpolation, not a bar mark |
| Bars | Lollipop | `Lollipop` | Bar/Bar/Circle | Dual axis: thin bar + circle head |
| Pies | Pie | `Pie` | Pie | `SUM(Sales)` by Region |
| Pies | Sunburst | `Sunburst` | Pie ×3 | Three concentric pie layers on `MIN(0)` dual axis, each a deeper level |
| Pies | Donut | `Donut` | Pie ×2 | Pie with a background-coloured inner pie |
| Pies | Donut Ratio | `Donut Ratio` | Pie ×3 | Donut showing `Profit Ratio` (13 %) with `Profit to Sales Rate` as remainder |
| Lines | Line Chart | `Line Chart` | Line | `SUM(Sales)` by month |
| Lines | Line Dual | `Line Dual` | Line ×2 | CY line over PY grey line |
| Lines | Area | `Area` | Area | Stacked area by Category |
| Lines | Bump Chart | `Bump Chart` | Line/Line/Circle | `RANK` on rows by quarter; circles carry the rank label |
| Percentage | Percentage Bar Filled | `Percentage Bar Filled` | Bar | Filled portion over a grey track, by year |
| Percentage | Percentage Gauge | `Percentage Gauge` | Pie/Pie/Circle | The five-slice gauge construction (§6.4) |
| Percentage | Waffle | `Waffle` | Bar | 10 × 10 grid; `Box Color` tests each cell against the target |
| Percentage | Percentage Bars | `Percentage Bar CY`, `Percentage Bar PY` | Bar | `CY Sales %` / `PY Sales %` table calcs as 100 % stacked bars |
| Shape Grids | Square Grid | `Square Grid` | Shape | Weekday × month grid, coloured by `SUM(Sales)` |
| Shape Grids | Hex Map Grid | `Hex Map` | Shape | Hand-coded `X` / `Y` state coordinates (§6.5) |
| Shape Grids | Triangle Rank Grid | `Triangle Rank Grid` | Shape ×3 | `Row Rank` / `Col Rank` triangular-number layout (§6.5) |
| Misc. | Gantt | `Gantt` | Automatic | `TDY(Order Date)` on columns, `Ship Mode / Region` on rows, sized by `Days` |
| Misc. | Trellis | `Trellis` | Automatic | Small multiples: `Ship Mode / MONTH` × `Region Abbv` |
| Misc. | Pareto | `Pareto` | Bar/Bar/Line | Bars plus a `PCT_OF_TOTAL(RUNNING_SUM(Sales))` cumulative line |

### 7.2 Volume II — Radials, dots and groups

| Section | Technique | Sheet | Mark | Construction |
|---|---|---|---|---|
| Radials | Spiral Chart | `Spiral Chart` | Circle | `X cos` / `Y sin` with `Index` growing continuously — the growing radius is what spirals it |
| Radials | Radial | `Radial` | Line/Line/Circle | Closed ring; `Order Date Number` on Path |
| Radials | Radial Stacked | `Radial Stacked` | Line/Line/Circle | As above but pathed by `Order ID`, stacking multiple orders per date into rays |
| Radials | Half Sun Radial | `Half Radial`, `Half Radial Sun` | Line/Circle + Area | Half-geometry parameters (§6.2); the `-S` inner pair fills the sun body |
| Bars | L Bar | `L Bar` | Bar/Line/Bar | Label column + value bar sharing one row |
| Bars | Unit Bar | `Unit Bar` | Shape | One mark per order, stacked by `Index` |
| Bars | Timeline Bar | `Timeline Bar` | Shape/Line/Shape | Unit bars along a date axis with a connecting line |
| Bars | Bar in Bar | `Bar in Bar` | Bar ×3 | CY bar rendered narrower inside the PY bar |
| Dots | Box & Whisker | `Box & Whisker` | Circle | Circles by Sub-Category with Tableau's box plot reference distribution |
| Dots | Dotplot | `Dotplot` | Circle | One circle per order along a sales axis |
| Dots | Jitterplot | `Jitterplot` | Shape | `Region * Index` on rows — `Index` spreads overlapping marks vertically |
| Plots | Scatterplot | `Scatterplot` | Circle | Sales × Profit |
| Plots | Enclosed Dotplot | `Enclosed Dotplot` | Circle/Line/Circle | Min and max circles joined by a capsule line |
| Plots | Bubbleplot | `Bubbleplot` | Circle | Scatter sized by a third measure |
| Groups | Treemap | `Treemap` | Automatic | Region → Category → Sub-Category nesting |
| Groups | Packed Bubble | `Packed Bubble` | Circle | Sized by Sales, coloured by Region/Category |
| Groups | Word Cloud | `Word Cloud` | Text | `Sub-Category` text sized by `SUM(Sales)` |
| Groups | Shapes | `Shapes` | Shape | Shape grid by Region |
| Tables | Fixed Total Table | `Fixed Total Table`, `Fixed Total Table 2` | Automatic | Grand-total row built with `{FIXED}` rather than Tableau's own totals |
| Tables | Calendar | `Calendar` | Shape | `WEEKDAY` × `WEEK` grid; `Day Today T\|F` highlights today |
| Tables | Heatmap Table | `Heatmap Table` | Automatic | Region × quarter, cell-coloured |

### 7.3 Volume III — Comparison, maps and polar pies

| Section | Technique | Sheet | Mark | Construction |
|---|---|---|---|---|
| Bars | Inverse Bar | `Inverse 1`, `Inverse 2` | Bar | Two sheets stacked, the lower one axis-reversed |
| Bars | Comparison Bar | `Comparison Bar` | Bar | `Region * AVG(Placement)` — placement offsets CY beside PY |
| Bars | Stagger Bar | `Stagger Bar` | Bar | `Region * AVG(Placement Year)` — four years offset within each region |
| Bars | Comparison Soundwave Bar | `Comparison Soundwave Bar` | Line | Mirrored bars drawn as paths |
| Line Bars | Candlestick | `Candlestick` | Line | `*candle top` / `*candle bottom` percentile wicks on LOD |
| Line Bars | Slope | `Slope` | Line/Line/Circle | `Placement 2` gives the two-point PY→CY axis |
| Line Bars | Barbells | `Barbells` | Line/Line/Circle | Two measures joined by a line, circles at each end |
| Line Bars | Soundwave Bar | `Soundwave Bar` | Line | `MONTH * MONTH` nested axis with `:Measure Names` on Path |
| More Bars | Bar & Candlestick | `Bar & Candlestick` | Bar + Line | Horizontal bars with candlestick overlay |
| More Bars | Diverging Bar | `Diverging Bar` | Bar | `SUM(Profit)` on columns, coloured by the same measure so negatives read red |
| More Bars | Butterfly Chart | `Butterfly Chart` | Bar | Two measures mirrored around a central axis |
| Maps | Pie Map | `Pie Map` | Pie ×3 | Pies placed at hand-coded `X` / `Y` hex coordinates |
| Maps | Map | `Map` | Filled map | Standard geographic fill, sized bubbles overlaid |
| Maps | Line Chart Map | `Chart Map` | Area/Area/Line | A sparkline drawn inside each state's hex cell — `RUNNING_MIN(-25)` supplies the baseline |
| Pies | Sunburst Explosion Pie | `Sunburst Explosion Pie` | Pie ×7 | Seven layers anchored on `Center` = `MAKEPOINT(0,0)` |
| Pies | Polar Area Chart | `Polar Area Chart` | **Polygon** | `Polar X` / `Polar Y` with `Path bin` densification (§6.3) |
| Pies | Coxcomb Chart | `Coxcomb Chart` | **Polygon** | Same geometry, coloured by year, nested wedges |
| Pies | Petals | `Petals` | Shape | Four directional petal images, one per region |
| Misc. | Histogram | `Histogram` | Bar | `Quantity (bin)` split CY vs PY |
| Misc. | Sankey | `Sankey` | **VizExtension** | A Tableau Viz Extension — note that extensions cannot be exported to an image, which is why the published view shows a placeholder |
| Misc. | Sparkline | `Sparkline` | Line | Minimal line per region with an end-point marker |

### 7.4 Volume IV — KPIs, funnels and combos

| Section | Technique | Sheet | Mark | Construction |
|---|---|---|---|---|
| KPIs | KPI Shapes | `KPI Shapes` | Shape | `:Measure Names` on columns, values inside rounded shapes |
| KPIs | KPI Card | `KPI Card 5`, `KPI Card 5.1` | Text + Area | Value, change and a filled sparkline |
| KPIs | KPI Containers | `KPI 4.1A`–`KPIA 4.4A` (8 sheets) | Shape/Text | Eight paired sheets: an icon tile (`$`, `#`, `+`, `%`) beside a value tile |
| Bars | Labels Above Bar | `Labels Above Bar` | Bar | Label placed on the bar's own row above it |
| Bars | Waterfall | `Waterfall` | **GanttBar** | `RUNNING_SUM(Sales)` on rows, sized by `- Sales` (§6.8) |
| Bars | Population Chart | `Population Chart 1/2/3` | Bar | `SUM(-PY Sales)` mirrors PY to the left of a central axis |
| Bars | Bar & Total Line | `Bar & Total Line` | Bar + Line | Bars with a total reference line |
| Funnels | Half Funnel | `Half Funnel 1.1`, `1.2` | Circle/Circle/Line | One-sided funnel with percentage bubbles down the left |
| Funnels | Funnel | `Funnel` | Bar | Symmetric centred bars, one per stage (§6.6) |
| Funnels | Donut Funnel | `Funnel 2.1D`–`2.5C` (5 sheets) | Pie | Each stage as its own donut showing the stage percentage |
| Lines | Win-Loss Bar | `Win-Loss Bar` | Line | `*Win Loss` (+1/−1/0) drawn as up/down ticks |
| Lines | Split Area Chart | `Split Area Chart` | Area | `Region * SUM(Sales)` — one band per region, split vertically |
| Lines | Running Total Line | `Running Total Line` | Line/Line/Area | `RUNNING_SUM` with a filled area beneath |
| Combo | Winged Donut | `Winged 1`, `Winged 2` | Pie ×3 + Bar ×3 | Target donut (§6.7) flanked by regional bars with grey tracks |
| Combo | Microchip Grid | `Combo_Left`, `Combo_Grid`, `Comb1_Right` | Area/Line/Shape | A weekday × month grid flanked by a sparkline and row totals |
| Misc. | Dual Chart | `Dual Chart` | Bar + Line | Bars with a line on a secondary axis |
| Misc. | Pie x 4 | `Pie x 4` | Pie ×4 | Four concentric rings, each a deeper hierarchy level |
| Misc. | Side-by-Side Bar | `Side-by-Side Bar` | Bar | Two measures per region, adjacent |

---

## 8. Dashboard specification

All four volumes share one template.

### 8.1 Canvas

| Property | Value |
|---|---|
| Sizing | **Fixed**, 1600 × 2000 px |
| Root background | `#767f8b` (slate grey) |
| Section panels | `#ffffff` on the grey ground |
| Masthead | `#000000` band |

### 8.2 Layout pattern

```
[6]   layout-basic                               bg #767f8b
└ [322] flow vert
  └ [5] flow vert
    ├ [8]  text      masthead  "VIZ DESIGN CATALOG Vol. N"   bg #000000
    ├ [9]  empty     white rule
    └ [10] flow vert
      └ [248] flow vert  section panel            bg #ffffff
        ├ [247] text     section label  ("KPIs", "Bars", …)
        ├ [266] empty    grey divider             bg #767f8b
        └ [251] flow horz
          └ per-technique columns: a text caption above each worksheet
```

Every volume repeats this: a black masthead, then seven (Vol. I) or six (Vols. II–IV) white
section panels, each holding a row of labelled technique columns. The grey ground showing
between panels is what separates the sections visually — there are no borders.

### 8.3 Masthead

| Element | Content |
|---|---|
| Title | `VIZ DESIGN CATALOG` + volume numeral, white on black |
| Byline | `DESIGNED BY JOHN JOHANSSON` |
| Icons | `DL Color`, `LinkedIn`, `Public`, `Twitter` |
| Volume navigation | `BV-I`, `BV-II`, `BV-III`, `BV-IV` — four boxed numerals, the active one highlighted |

---

## 9. Interactivity

28 URL actions and nothing else — no parameter actions, no filter actions, no Dynamic Zone
Visibility. The catalogue is a static reference; its only interactivity is navigation.

| Source sheet | Actions | Target |
|---|---:|---|
| `BV-I` | 3 | Vol. I URL (from each of the other three volumes) |
| `BV-II` | 3 | Vol. II URL |
| `BV-III` | 3 | Vol. III URL |
| `BV-IV` | 3 | Vol. IV URL |
| `LinkedIn` | 4 | `https://www.linkedin.com/in/johnsjohansson/` |
| `Public` | 4 | `https://public.tableau.com/app/profile/john.johansson/vizzes` |
| `Twitter` | 4 | `https://twitter.com/JohnSJohansson` |
| `DL Color` | 4 | The catalogue's own colour palette file (see below) |

Each of the four chrome sheets carries one action per volume — four copies of the same
destination — because an action is scoped to a dashboard, and the sheet appears on all four.

**Volume navigation works by URL, not by zone swapping.** Clicking a volume numeral opens that
volume's *separately published* Tableau Public page. This is why the workbook has four
dashboards and four publications rather than one dashboard with Dynamic Zone Visibility: each
volume is a standalone page with its own URL, view count and share link.

**Palette download.** The `DL Color` icon links to a `.tps` file —
`DesignCatalogColors-Preferences.tps` hosted on Dropbox — so a reader can install the
catalogue's exact palette into their own Tableau Repository.

---

## 10. Design system

### 10.1 Palette

| Token | Hex | Applied to |
|---|---|---|
| Slate ground | `#767f8b` | Dashboard background, section dividers |
| Panel white | `#ffffff` | Section panels |
| Masthead black | `#000000` | Title band |
| Primary teal | `#0f646a` | The dominant measure colour across all four volumes |
| Track grey | `#b6bbc1` | Background tracks on bars, gauges and donuts |
| Forest green | `#44633f` | East region |
| Steel blue | `#568ea3` | Central region |
| Amber | `#faa916` | South region |
| Plum | `#b95f89` | West region |

**Custom palettes defined in the workbook**

| Name | Type | Colours |
|---|---|---|
| `CB_BuPu` | ordered-sequential | `#f7fcfd · #e0ecf4 · #bfd3e6 · #9ebcda · #8c96c6 · #8c6bb1 · #88419d · #810f7c · #4d004b` |
| `DC_Green2Grey` | ordered-diverging | `#0f646a → #b6bbc1` |

`DC_Green2Grey` is the signature ramp — teal to grey — used wherever a measure is encoded
continuously, and it is what gives all four volumes a single visual identity despite covering
85 different chart types.

### 10.2 The four-region colour convention

Region is the catalogue's primary categorical dimension, and it keeps the same four colours
throughout: Central `#568ea3`, East `#44633f`, South `#faa916`, West `#b95f89`. Holding this
constant across every chart in every volume is what lets a reader compare techniques rather
than decode legends.

### 10.3 Global chart formatting

Set once at workbook level and inherited by all 132 sheets:

```
axis      line-visibility: off
gridline  line-visibility: off
zeroline  line-visibility: off
title     color #000000, bold
all       font-family Arial
```

Every chart in the catalogue is therefore stripped of default chrome; any visible axis or rule
is a deliberate per-sheet exception.

### 10.4 Typography

| Level | Spec |
|---|---|
| Workbook default | **Arial** |
| Masthead | Very large bold white, volume numeral in grey |
| Section labels | Large bold black, left-aligned in the panel |
| Technique captions | Small bold, centred above each chart |
| Chart labels | Small, teal or black |

---

## 11. Rebuild / maintenance runbook

**Re-basing the radial charts to a new year**

The radial angle divides by a hard-coded date serial:

```
Angle = 2 * PI() * ([Order Date Number] - 45290) / ({MAX([Order Date Number])} - 45290)
```

`45290` is 1 January of the plotted year. To move the radials forward a year, update this
constant in **both** `Angle` and `Angle Half`. The workbook preserves the previous constant
(`44197`) as a comment above the live line — keep that habit, it is what makes the change
reversible.

**Refreshing the data**

`Max Order Date` is a `{FIXED :max()}` LOD used in 87 sheets, so every CY/PY split re-bases
automatically. Nothing else needs editing except the radial constants above.

**Adding a technique to a volume**

1. Build the sheet.
2. Add a text caption zone and the worksheet to the relevant section's horizontal flow
   container.
3. The section panels are fixed-width flows — adding a fourth technique to a three-technique
   row will compress the others, so check the row still reads at 1600 px.

**Adding a fifth volume**

1. Duplicate a volume dashboard and rename it `Vol_V`.
2. Add a `BV-V` navigation sheet and place it on all five dashboards.
3. Add a URL action from `BV-V` on each of the other dashboards, and one from each existing
   `BV-*` sheet on `Vol_V` — the action count grows as *n*×(*n*−1) plus chrome.
4. Publish `Vol_V` as its own Tableau Public entry; the navigation is URL-based, so the new
   volume needs a live URL before the links work.

**Keeping the four publications in sync**

Because all four entries are the same file, republishing one volume uploads the whole
workbook. Publish all four after any edit, or the volumes will drift to different revisions —
the current revision numbers (3.3 / 1.7 / 1.5 / 1.3) show this has happened historically.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | (unnamed) |
| Connection class | `federated` |
| Tables / relations | 4 · joins: 3 |
| Physical columns materialised | 13 |
| Source fields in data pane | 25 |
| Calculated fields (used / total) | 113 / 138 |
| Parameters | 9 |
| Data source filters | 0 (none) |

### Source fields

25 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Location

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Region** | string · Dimension | 4 | yes | 110 | Sales region: Central, East, South, West. The primary categorical split. |
| **State/Province** | string · Dimension | 62 | yes | 110 | State or province. The geographic key for maps and the hex grid. |
| **Country/Region** | string · Dimension | 2 | yes | 11 | Country. United States and Canada only in this extract. |
| **City** | string · Dimension | — | yes | 0 | City of the shipping address. *(hidden, not materialised, not used in any sheet)* |
| **Postal Code** | string · Dimension | — | yes | 0 | Postal code of the shipping address. *(hidden, not materialised, not used in any sheet)* |
| **Region (People)** | string · Dimension | — | — | 0 | Sales region: Central, East, South, West. The primary categorical split. Arrives from the **People** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |

#### Product

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Category** | string · Dimension | 3 | yes | 109 | Top product tier: Furniture, Office Supplies, Technology. |
| **Manufacturer**<br>`Product Name (group)` | string · Dimension | — | — | 109 | Manufacturer name. Largely null in this extract and filtered out on most sheets. |
| **Product Name** | string · Dimension | 2,063 | yes | 109 | Full product description. |
| **Product ID** | string · Dimension | — | yes | 0 | Product key. *(hidden, not materialised, not used in any sheet)* |

#### Dates

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Order Date** | date · Dimension | 1,371 | yes | 107 | Date the customer placed the order. The anchor for every time axis and for `Max Order Date`. |

#### Measures

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Sales** | real · Measure (Sum) | 3,565 | yes | 78 | Line revenue in dollars, net of discount. The primary measure. |
| **Profit** | real · Measure (Sum) | 4,366 | yes | 13 | Line profit in dollars. Can be negative where discounting exceeds margin. |
| **Quantity** | integer · Measure (Sum) | 13 | yes | 4 | Units sold on the line. |
| **Discount** | real · Measure (Sum) | — | yes | 0 | Discount rate applied, 0–0.8. *(hidden, not materialised, not used in any sheet)* |

#### Order

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Order ID** | string · Dimension | 296 | yes | 19 | Order key in `CA-2023-100006` form. Orders span multiple lines, so distinct counts must use `COUNTD`. |
| **Order ID (Returns)** | string · Dimension | — | — | 0 | Order key in `CA-2023-100006` form. Orders span multiple lines, so distinct counts must use `COUNTD`. Arrives from the **Returns** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |

#### Waffle scaffold

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Column** | integer · Dimension | — | — | 1 | Waffle-grid column index, 1 to 10. Supplied by the template table rather than calculated. |
| **Rows** | integer · Dimension | — | — | 1 | Waffle-grid row index, 1 to 10. With Column it addresses one of the 100 cells. |

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

### Calculated fields in use

113 of the workbook's 138 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| **$ Change**<br>`% Change (copy)_2367486074249367583` | real · Measure | Basic | CY Sales, PY Sales | — | 1 |
| **% Change**<br>`CY vs PY Symbol (copy)_99360729838260239` | real · Measure | Basic | CY Sales, PY Sales | — | 4 |
| **% Change +**<br>`% Change (copy)_1197957530486571008` | real · Measure | Basic | CY Sales, PY Sales | — | 1 |
| **% Change +\|-**<br>`% Change T+\|F- (copy)_1197957530490093571` | string · Measure | Basic | CY Sales, PY Sales | — | 2 |
| **% Change -**<br>`% Change + (copy)_1197957530486960129` | real · Measure | Basic | CY Sales, PY Sales | — | 1 |
| **% Change T+\|F-**<br>`% Change + (copy)_1197957530487439362` | boolean · Measure | Basic | CY Sales, PY Sales | — | 2 |
| **% Slice 1**<br>`Calculation_3958664101417140232` | real · Measure | Basic | Percentage % | — | 1 |
| **% Slice 2**<br>`% Slice 1 (copy)_3958664101417426953` | real · Measure | Basic | Percentage % | — | 1 |
| **% Slice 3**<br>`% Slice 1 (copy) (copy) (copy)_3958664101417439243` | integer · Measure | Basic | — | — | 1 |
| **% Slice 4**<br>`% Slice 1 (copy) (copy) (copy) (copy)_3958664101417443340` | real · Measure | Basic | Percentage % | % Slice 5 | 1 |
| **% Slice 5**<br>`% Slice 1 (copy) (copy)_3958664101417431050` | real · Measure | Basic | Percentage %, % Slice 4 | — | 1 |
| ***candle bottom**<br>`*candle (copy)_3032048483384066052` | real · Measure | Basic | Profit | — | 1 |
| ***candle top**<br>`Calculation_3032048483380891651` | real · Measure | Basic | Profit | — | 1 |
| ***Win Loss**<br>`Calculation_2367486073832202246` | integer · Measure | Basic | — | — | 1 |
| **- Sales**<br>`Calculation_2367486073699655681` | real · Measure | Basic | Sales | — | 1 |
| **-PY Sales**<br>`PY Sales (copy)_3314086408495951887` | real · Measure | Basic | Year, Max Order Date, Sales | — | 2 |
| **100**<br>`Year Orders % (copy)_1559371376288940042` | integer · Measure | Basic | — | — | 1 |
| **2 Year Profit**<br>`Profit (copy)_2367486073685716992` | real · Measure | Basic | Profit | — | 1 |
| **A_Color**<br>`Calculation_3958664101442371614` | string · Dimension | Basic | — | — | 5 |
| **Angle**<br>`Order Date Number (copy)_1043146293701652486` | real · Dimension | LOD | Order Date Number | Angle Adjusted | 3 |
| **Angle Adjusted**<br>`Calculation_1043146293705011207` | real · Measure | Basic | Angle | X cos, Y sin | 3 |
| **Angle Adjusted Half**<br>`Angle Adjusted (copy)_1043146293741060115` | real · Measure | Basic | Angle Half | X cos Half, X cos Half-S, Y sin Half, Y sin Half-S | 2 |
| **Angle Half**<br>`Angle (copy)_1043146293741060116` | real · Dimension | LOD | Order Date Number | Angle Adjusted Half | 2 |
| **B_Color**<br>`A_Color (copy)_570549806785167360` | string · Dimension | Basic | — | — | 8 |
| **Box Color**<br>`Calculation_3958664101426249749` | boolean · Measure | Basic | Percentage % | — | 1 |
| **C_Color**<br>`B_Color (copy)_1043146293717553163` | string · Dimension | Basic | — | — | 8 |
| **Candle**<br>`Calculation_3542644089719500801` | real · Measure | Basic | Order Date, Sales | — | 1 |
| **Category Abbv**<br>`Sub-Category Abbv (copy)_2266155069273137152` | string · Dimension | Basic | Category | — | 1 |
| **Center**<br>`Calculation_3032048483372380162` | spatial · Measure | Basic | — | — | 2 |
| **Col Rank**<br>`Row Rank (copy)_3958664101404819463` | real · Measure | Basic | Rank Sales | — | 1 |
| **CY**<br>`CY/PY Year (copy)_99360729852071959` | integer · Dimension | Basic | Year, Max Order Date | — | 4 |
| **CY or PY**<br>`Calculation_3314086408456491009` | string · Dimension | Basic | Order Date, Max Order Date | Placement, Placement 2, Region CY/PY | 8 |
| **CY Sales**<br>`Calculation_99360729833828364` | real · Measure | Basic | Year, Max Order Date, Sales | $ Change, % Change, % Change +, % Change +\|- … | 26 |
| **CY Sales %**<br>`CY Sales (copy)_99360729865863193` | real · Measure | Table calc | CY Sales | — | 1 |
| **CY vs PY Symbol**<br>`PY Sales (copy)_99360729837473806` | string · Measure | Basic | CY Sales, PY Sales | — | 4 |
| **CY vs PY Symbol +**<br>`CY vs PY Symbol (copy)_99360729839702033` | string · Measure | Basic | CY Sales, PY Sales | — | 5 |
| **CY vs PY Symbol -**<br>`CY vs PY Symbol (copy)_99360729839808530` | string · Measure | Basic | CY Sales, PY Sales | — | 5 |
| **CY-PY -Diff**<br>`Calculation_3542644089717784576` | real · Measure | Basic | PY Sales, CY Sales | — | 1 |
| **Day Name**<br>`Calculation_3958664101392506880` | date · Dimension | Basic | Order Date | — | 5 |
| **Day Today T\|F**<br>`Month Today T\|F (copy)_1043146293688737794` | boolean · Dimension | Basic | Order Date | — | 1 |
| **Days**<br>`Days (copy)_1766536995691900933` | integer · Measure | Basic | Order Date | — | 1 |
| **Fixed Category Sales**<br>`Fixed Total Sales (copy)_3604005630813573124` | real · Measure | LOD | Year, Category, Sales | — | 1 |
| **Fixed Total Orders**<br>`Index+1 Shapex100 (copy)_1559371376286896135` | integer · Measure | LOD | Order ID | Year Orders % | 4 |
| **Fixed Total Region**<br>`Sales Target (copy)_2379026544813780997` | real · Measure | LOD | Year, Region, Sales | — | 1 |
| **Fixed Total Sales**<br>`Total Orders (copy)_3604005630813118467` | real · Measure | LOD | Year, Sales | — | 4 |
| **Funnel 1 %**<br>`Funnel 2 % (copy)_2367486074159591440` | real · Measure | Basic | Total Orders | Funnel 1 -% | 2 |
| **Funnel 1 -%**<br>`Funnel 1 % (copy)_2367486074174840860` | real · Measure | Basic | Funnel 1 % | — | 1 |
| **Funnel 2**<br>`Calculation_2367486074152169482` | integer · Measure | LOD | Order ID, Region | Funnel 2 % | 8 |
| **Funnel 2 %**<br>`Funnel 2 (copy)_2367486074158911503` | real · Measure | Basic | Funnel 2, Total Orders | Funnel 2 -% | 4 |
| **Funnel 2 -%**<br>`Funnel 2 % (copy)_2367486074171990042` | real · Measure | Basic | Funnel 2 % | — | 3 |
| **Funnel 3**<br>`Total West-East Orders (copy)_2367486074153848843` | integer · Measure | LOD | Order ID, Region, Category | Funnel 3 % | 6 |
| **Funnel 3 %**<br>`Funnel 2 % (copy)_2367486074160136211` | real · Measure | Basic | Funnel 3, Total Orders | Funnel 3 -% | 2 |
| **Funnel 3 -%**<br>`Funnel 2 -% (copy)_2367486074172354587` | real · Measure | Basic | Funnel 3 % | — | 1 |
| **Funnel 4**<br>`Funnel 3 (copy)_2367486074155589644` | integer · Measure | LOD | Order ID, Region, Category | Funnel 4 % | 5 |
| **Funnel 4 %**<br>`Funnel 2 % (copy) (copy)_2367486074160152596` | real · Measure | Basic | Funnel 4, Total Orders | — | 1 |
| **Index**<br>`Calculation_1043146293700063235` | integer · Measure | Table calc | — | X cos, X cos Half, X cos Half-S, Y sin … | 9 |
| **Index+1 Shapex10**<br>`Index+1 (copy)_1559371376285540358` | boolean · Measure | Basic | Index-1 | — | 1 |
| **Index-1**<br>`Calculation_1559371376281403393` | integer · Measure | Table calc | — | Index+1 Shapex10 | 1 |
| **Index-2**<br>`Cox X (copy)_3542644089768128546` | integer · Measure | Table calc | — | Polar X, Polar Y | 2 |
| **Last Month Sales**<br>`CY Sales (copy)_3542644089763242014` | real · Measure | Basic | Year, Max Order Date, Month, Sales | — | 1 |
| **Max Order Date**<br>`Calculation_99360729564491781` | date · Dimension | LOD | Order Date | -PY Sales, CY, CY Profit, CY Sales … | 87 |
| **Month**<br>`Year (copy)_3314086408467492875` | integer · Dimension | Basic | Order Date | Last Month Sales, Placement Month, Region Month | 1 |
| **Order Date Number**<br>`Order Date (copy)_1043146293700997125` | integer · Dimension | Basic | Order Date | Angle, Angle Half, ODN | 8 |
| **P_Date Part**<br>`Calculation_3542644089768325155` | integer · Measure | Table calc | Order Date | P_Starting Point | 2 |
| **P_Percentage**<br>`TC_Percentage (Adjusted) (copy)_3542644089769095210` | real · Measure | Basic | P_Sales, P_Total Sales | P_Percentage_Adj | 2 |
| **P_Percentage_Adj**<br>`TC_Sales (copy)_3542644089768960041` | real · Measure | Table calc | P_Percentage | Polar X, Polar Y | 2 |
| **P_Sales**<br>`TC_Date Part (copy)_3542644089768550436` | real · Measure | Table calc | Sales | P_Percentage | 2 |
| **P_Size**<br>`TC_Date Part (copy)_3542644089768714278` | real · Measure | Basic | — | Polar X, Polar Y | 2 |
| **P_Starting Point**<br>`TC_Date Part (copy)_3542644089768628261` | real · Measure | Basic | P_Date Part | Polar X, Polar Y | 2 |
| **P_Total Sales**<br>`TC_Sales (copy)_3542644089768788007` | real · Measure | LOD + Table calc | Sales | P_Percentage | 2 |
| **Path**<br>`Path (copy)_3542644089767665696` | integer · Measure | Basic | — | Path bin | 2 |
| **Path bin**<br>`Path (102) (bin)` | integer · Dimension | Basic | Path | — | 2 |
| **Percentage %**<br>`Calculation_3958664101418663951` | real · Measure | Basic | — | % Slice 1, % Slice 2, % Slice 4, % Slice 5 … | 2 |
| **Placement**<br>`Calculation_3314086408456806402` | integer · Measure | Basic | CY or PY | — | 2 |
| **Placement 2**<br>`Placement (copy)_3542644089753579549` | integer · Measure | Basic | CY or PY | — | 1 |
| **Placement Year**<br>`Placement (copy)_3314086408460079112` | integer · Measure | Basic | Year | — | 1 |
| **Polar X**<br>`Path (102) (copy)_3542644089767952417` | real · Measure | Basic | Index-2, P_Size, P_Starting Point, P_Percentage_Adj | — | 2 |
| **Polar Y**<br>`Cox X (copy)_3542644089768890408` | real · Measure | Basic | Index-2, P_Size, P_Starting Point, P_Percentage_Adj | — | 2 |
| **Profit Ratio**<br>`Calculation_1368249927221915648` | real · Measure | Basic | Profit, Sales | — | 4 |
| **Profit to Sales Rate**<br>`Calculation_99360729866936346` | real · Measure | Basic | Sales, Profit | — | 1 |
| **PY**<br>`CY (copy)_99360729852239896` | integer · Dimension | Basic | Year, Max Order Date | — | 4 |
| **PY Sales**<br>`CY (copy)_99360729834057741` | real · Measure | Basic | Year, Max Order Date, Sales | $ Change, % Change, % Change +, % Change +\|- … | 16 |
| **PY Sales %**<br>`CY Sales % (copy)_3958664101420974096` | real · Measure | Table calc | PY Sales | — | 1 |
| **Quantity (bin)** | integer · Dimension | Basic | Quantity | — | 1 |
| **Rank Sales**<br>`Calculation_3958664101399441410` | integer · Measure | Table calc | Sales | Col Rank, Row Rank | 1 |
| **Region Abbv**<br>`Region (copy)_3604005630809735170` | string · Dimension | Basic | Region | — | 2 |
| **Region CY/PY**<br>`Calculation_3314086408457351171` | string · Dimension | Basic | Region, CY or PY | — | 1 |
| **Region Year**<br>`Region CY/PY (copy)_3314086408459624455` | string · Dimension | Basic | Year, Region | — | 1 |
| **Rolling 12 Months**<br>`Calculation_99360729563422724` | string · Dimension | Basic | Order Date, Max Order Date | — | 56 |
| **Rolling 24 Months**<br>`Rolling 12 Months (copy)_3314086408457809924` | string · Dimension | Basic | Order Date, Max Order Date | — | 2 |
| **Row Rank**<br>`Calculation_3958664101402251270` | real · Measure | Basic | Rank Sales | — | 1 |
| **Sales Furniture**<br>`Calculation_1043146293779251229` | real · Measure | Basic | Category, Sales | Sales Furniture % | 2 |
| **Sales Furniture %**<br>`Sales Office % (copy)_2266155069312938004` | real · Measure | LOD | Sales Furniture, Region, Year, Sales | — | 1 |
| **Sales Office Supplies**<br>`Sales Furniture (copy 2)_1043146293779570719` | real · Measure | Basic | Category, Sales | Sales Office % | 1 |
| **Sales Target**<br>`Profit to Sales Rate (copy)_2379026544676048898` | real · Measure | Basic | Sales | Sales Target Diff, Sales Target Diff % | 1 |
| **Sales Target Diff**<br>`Sales Target (copy)_2379026544676536323` | real · Measure | Basic | Sales Target, Sales | — | 1 |
| **Sales Target Diff %**<br>`Sales Target Diff (copy)_2379026544677638148` | real · Measure | Basic | Sales, Sales Target | — | 1 |
| **Sales Technology**<br>`Sales Furniture (copy)_1043146293779558430` | real · Measure | Basic | Category, Sales | Sales Technology % | 1 |
| **Ship Mode Abbv**<br>`Sub-Category Abbv (copy)_2266155069306327055` | string · Dimension | Basic | — | — | 1 |
| **State Abbv**<br>`X (copy)_99360729846267924` | string · Dimension | Basic | State/Province | — | 4 |
| **Sub-Category Abbv**<br>`Calculation_3604005630807711744` | string · Dimension | Basic | — | — | 2 |
| **Total Orders**<br>`Funnel 2 (copy)_2367486074157002766` | integer · Measure | Basic | Order ID | Funnel 1 %, Funnel 2 %, Funnel 3 %, Funnel 4 % | 8 |
| **Total Year Orders**<br>`Total Orders (copy)_1559371376287309832` | integer · Measure | LOD | Year, Order ID | Year Orders % | 4 |
| **X**<br>`Calculation_99360729821892616` | real · Dimension | Basic | State/Province | — | 4 |
| **X cos**<br>`Calculation_1043146293705113608` | real · Measure | Basic | Index, Angle Adjusted | — | 3 |
| **X cos Half**<br>`X cos (copy)_1043146293740466192` | real · Measure | Basic | Index, Angle Adjusted Half | — | 1 |
| **X cos Half-S**<br>`X cos Half (copy)_1043146293760786453` | real · Measure | Basic | Index, Angle Adjusted Half | — | 1 |
| **Y**<br>`Calculation_99360729822072841` | integer · Dimension | Basic | State/Province | — | 4 |
| **Y sin**<br>`Calculation_1043146293705207817` | real · Measure | Basic | Index, Angle Adjusted | — | 3 |
| **Y sin Half**<br>`Y sin (copy)_1043146293740650513` | real · Measure | Basic | Index, Angle Adjusted Half | — | 1 |
| **Y sin Half-S**<br>`Y sin Half (copy)_1043146293760786454` | real · Measure | Basic | Index, Angle Adjusted Half | — | 1 |
| **Year**<br>`Calculation_99360729557819395` | integer · Dimension | Basic | Order Date | -PY Sales, CY, CY Profit, CY Sales … | 34 |
| **Year Orders %**<br>`Total Year Orders (copy)_1559371376287551497` | real · Measure | Basic | Total Year Orders, Fixed Total Orders | Year Orders % (bin) | 4 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| ***Angle, End Half**<br>`*Angle, End (copy)_1043146293740290063` | integer · range | `178` | range 0 to — |
| ***Angle, Start Half**<br>`*Angle, Start (copy)_1043146293740290062` | integer · range | `2` | range 2 to 180 |
| ***Hole 2**<br>`*Hole (copy)_1043146293740744722` | real · any | `10.` | any value |
| **Top Customers**<br>`Parameter 1` | integer · range | `5` | range 5 to 20, step 5 |
| **Profit Bin Size**<br>`Parameter 2` | integer · range | `200` | range 50 to 200, step 50 |
| **100**<br>`Parameter 3` | real · any | `100.` | any value |
| ***Hole**<br>`Parameter 4` | real · any | `2.` | any value |
| ***Angle, Start**<br>`Parameter 5` | integer · range | `2` | range 2 to 360 |
| ***Angle, End**<br>`Start Angle (copy)_1009861854720749594` | integer · range | `360` | range 0 to — |

### How the numbers are computed

**Level-of-detail expressions — 13.** These fix their own level of detail, so they return the same value regardless of what dimensions are in the view:

> `Angle Half`, `Funnel 2`, `Max Order Date`, `Fixed Category Sales`, `Funnel 4`, `Fixed Total Orders`, `Angle`, `Sales Furniture %`, `Fixed Total Region`, `P_Total Sales`, `Total Year Orders`, `Fixed Total Sales`, `Funnel 3`

**Table calculations — 10.** These compute across the rendered table, so their result depends on what is in the view at the time:

> `PY Sales %`, `CY Sales %`, `Index`, `Index-1`, `P_Date Part`, `Rank Sales`, `Index-2`, `P_Sales`, `P_Total Sales`, `P_Percentage_Adj`

The workbook file records only each table calculation's `ordering-type`. The addressing and partitioning described elsewhere in this document are **derived from each sheet's level-of-detail shelf**, not read from the file.

## Appendix A — Worksheet index by volume

**Vol. I (37)** — `Area`, `Bar`, `Bar Curved`, `Bar Dual`, `Bump Chart`, `Donut`,
`Donut Ratio`, `Gantt`, `Hex Map`, `KPI 3.2`, `KPI Card`, `KPI Card 2.1`, `KPI Card 2.2`,
`KPI Card 3.1`, `KPI Card 3.3`, `Line Chart`, `Line Dual`, `Lollipop`, `Pareto`,
`Percentage Bar CY`, `Percentage Bar Filled`, `Percentage Bar PY`, `Percentage Gauge`, `Pie`,
`Square Grid`, `Sunburst`, `Trellis`, `Triangle Rank Grid`, `Waffle`, `WV-I` + chrome/nav

**Vol. II (31)** — `Bar in Bar`, `Box & Whisker`, `Bubbleplot`, `Calendar`, `Dotplot`,
`Enclosed Dotplot`, `Fixed Total Table`, `Fixed Total Table 2`, `Half Radial`,
`Half Radial Sun`, `Heatmap Table`, `Jitterplot`, `L Bar`, `Packed Bubble`, `Radial`,
`Radial Stacked`, `Scatterplot`, `Shapes`, `Spiral Chart`, `Timeline Bar`, `Treemap`,
`Unit Bar`, `Word Cloud`, `WV-II` + chrome/nav

**Vol. III (30)** — `Bar & Candlestick`, `Barbells`, `Butterfly Chart`, `Candlestick`,
`Chart Map`, `Comparison Bar`, `Comparison Soundwave Bar`, `Coxcomb Chart`, `Diverging Bar`,
`Histogram`, `Inverse 1`, `Inverse 2`, `Map`, `Petals`, `Pie Map`, `Polar Area Chart`,
`Sankey`, `Slope`, `Soundwave Bar`, `Sparkline`, `Stagger Bar`, `Sunburst Explosion Pie`,
`WV-III` + chrome/nav

**Vol. IV (45)** — `Bar & Total Line`, `Comb1_Right`, `Combo_Grid`, `Combo_Left`,
`Dual Chart`, `Funnel`, `Funnel 2.1D`–`2.5C`, `Half Funnel 1.1`, `Half Funnel 1.2`,
`KPI 4.1A`–`KPIA 4.4A`, `KPI Card 5`, `KPI Card 5.1`, `KPI Shapes`, `Labels Above Bar`,
`Microchip Grid`, `Pie x 4`, `Population Chart 1/2/3`, `Running Total Line`,
`Side-by-Side Bar`, `Split Area Chart`, `Waterfall`, `Win-Loss Bar`, `Winged 1`, `Winged 2`,
`WV-IV` + chrome/nav

**Shared across all volumes (8)** — `BV-I`, `BV-II`, `BV-III`, `BV-IV` (volume navigation);
`DL Color`, `LinkedIn`, `Public`, `Twitter` (chrome)

## Appendix B — Calculated fields excluded from this documentation

25 calculated fields exist in the data pane but are not referenced by any worksheet or
dashboard logic:

`Region_Category` · `CY Profit` · `Month Today T|F` · `ODN` · `CountD` · `Map Label` ·
`State Label` · `Sales (Stream Bin)` · `Profit +|-` · `Hide` · `Dummy Count` ·
`Avg Dummy Count (for Qtrs)` · `Month Order Date` · `Placement Month` · `Profit (bin)` ·
`Region Month` · `Sales (bin)` · `Sales *-1` · `Sales Region %` · `Sales Office %` ·
`Sales Technology %` · `Ship Mode (copy)` · `Year Orders % (bin)` · `MoM % Change` ·
`Number of Records`

## Appendix C — Package inventory

| File | Size |
|---|---:|
| `Viz Design Catalog Vol. I  #VOTD.twb` | ~4.0 MB |
| Embedded extract (`TEMP_0sdl9me1l012ew15qk3cp0dmyhow.hyper`) | included |
| **Total `.twbx`** | **4.1 MB** |

Custom shapes (hexagons, petals, rounded tiles, social icons) are referenced from the author's
local Tableau Repository and are **not** packaged.

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas, colour values and settings are quoted verbatim
from the workbook definition; rendered figures are read from the published views.*
