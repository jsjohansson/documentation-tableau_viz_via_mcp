# U.S. Gas Infrastructure — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `U.S. Gas Infrastructure` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/U_S_GasDistributionInfrastructure/Material |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | **Local file** — `My Tableau Repository/Workbooks/DOT Distribution General/U.S. Gas Distribution Infrastructure.twb`, modified **2024-02-19**, decompiled and parsed field-by-field |
| Version alignment | The published workbook's last update is **2024-02-19** — the same date as the local file, so the two are aligned |
| Explicit exclusion | 55 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic**. Summarised in Appendix A. |

> **This supersedes an earlier partial version.** Tableau Public has data downloads disabled for
> this workbook, so it was previously documented from a single rendered image. The local `.twb`
> now supplies the full definition, and **all four published views have been rendered** — the
> Leaks and Damages dashboards were not covered before.

---

## 1. Executive summary

**U.S. Gas Infrastructure** reports the condition of the American gas distribution network from
**2013 to 2022**, built on the U.S. Department of Transportation's **PHMSA operator annual
reports** — the mandatory filings every gas distribution operator submits each year.

It is a **four-dashboard application**, not a single view:

| Dashboard | Subject |
|---|---|
| **Material** | What the network is made of, and how much predates 1970 |
| **Leaks** | Reportable leaks repaired, by cause, normalised to leak rate |
| **Damages** | Excavation damage and one-call ticket volumes, by root cause |
| **Data Source** | Provenance and methodology |

The analytical spine is **rate normalisation**. Raw counts are meaningless when operators differ
by orders of magnitude in size, so nearly every headline is expressed per unit of exposure —
leaks per main mile, hazardous leaks per service count, damages per 1,000 excavation tickets.
That is what lets DC (1.03 leaks/mile) and Hawaii (14.03 damages per 1,000 tickets) sit
meaningfully beside California and Texas.

The second structural idea is a **ten-year endpoint comparison** built from nested `{FIXED}` LODs
that resolve the first and last report years from the data itself (§6.3), so every "10 Year
Difference" re-bases automatically as new filings arrive.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | **75** |
| Dashboards | **4** (`Material`, `Leaks`, `Damages`, `Data Source`) |
| Data sources | 1 (`DOT Distribution Report 2010+`) |
| Parameters | 3 |
| Calculated fields **used** | 129 named + 133 in-view ad-hoc |
| Calculated fields unused (not documented) | 55 |
| Dashboard actions | 3 filter, 3 highlight |
| Dynamic Zone Visibility bindings | 0 |
| Canvas | 1400 × 826 px per dashboard |
| Published size | 4,107,048 bytes (3.9 MB) |

### 1.2 Headline figures — Material view

Reporting year **CY2022**, window 2013–2022.

| Metric | Value | Share | 10-Year Difference |
|---|---:|---:|---|
| Pre-1970 Mainline Pipe | **408,828** mi | 30 % | ▼ −67,490 (−14 %) |
| Industry Targeted Material | **66,057** mi | 5 % | ▼ −37,563 (−36 %) |
| Steel Service Lines | **14,353,586** | 20 % | ▼ −4,074,503 (−22 %) |
| Miles of Mainline (total) | **1,372,352** | | ▲ 116,901 (9 %) |
| Service Count (total) | **71,547,414** | | ▲ 4,387,780 (7 %) |

### 1.3 Headline figures — Leaks view

Leak group **Mainline & Service Leaks**:

| Cause | Leaks | Share | 10-Year Avg |
|---|---:|---:|---:|
| Equipment Failure | **186,768** | 37 % | 167,530 |
| Corrosion Failure | 91,296 | 18 % | 109,703 |
| Excavation Damage | 89,006 | 18 % | 80,881 |
| Pipe, Weld & Joint | 47,531 | 9 % | 51,144 |
| Other Cause | 33,979 | 7 % | 49,213 |
| Natural Force Damage | 21,261 | 4 % | 28,564 |
| Incorrect Operation | 17,105 | 3 % | 16,282 |
| Other Outside Force | 16,965 | 3 % | 15,753 |

| Total | Value |
|---|---|
| Total Leaks | **503,971** (10-year avg 519,070) |
| Mainline Leaks | 101,243 |
| Service Leaks | 402,728 |
| Hazardous Leaks | **40 %** |
| Open Leaks at EOY | 124,911 *(excluded from all other reports)* |
| **Leak Rate** | **0.37** — 503,971 leaks per 1,372,352 miles |

**Leak rate by state** — DC 1.03 · MD 0.66 · CA 0.66 · IL 0.65 · TX 0.59 · HI 0.57 · MA 0.48 ·
WV 0.47 · VA 0.46 · AR 0.44 · CO 0.41

### 1.4 Headline figures — Damages view

| Root cause | Damages | Share |
|---|---:|---:|
| Excavation Practice | **37,940** | 42 % |
| One-Call Notification Practice | 29,476 | 32 % |
| Locating Practice | 21,727 | 24 % |
| Other Practice | 1,555 | 2 % |

| Total | Value | 10-Year Difference |
|---|---:|---|
| Damages | **90,698** | ▲ 16,702 (23 %) |
| Excavation Tickets | **36,128,996** | ▲ 12,533,514 (53 %) |
| **Damages per 1,000 Tickets** | **2.51** | ▼ −0.63 (−20 %) |

Those three read together are the view's argument: damages rose 23 %, but excavation activity
rose 53 %, so the **rate fell 20 %**. Only the normalised figure supports a claim about safety.

**Damage rate by state** — HI 14.03 · AR 6.94 · ID 5.76 · AL 5.64 · AK 5.19 · MT 4.63 · SD 4.44 ·
MI 4.33 · MS 4.03 · TX 3.74 · NM 3.63 · MO 3.37 · TN 3.25 · WY 3.22 · GA 3.15

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `13187266` |
| LUID | `f0cfe367-c9c6-41f4-9fe6-f9e486935243` |
| Repository URL | `U_S_GasDistributionInfrastructure` |
| Published views | `Material` (default) · `Leaks` · `Damages` · `Data Source` |
| Revision | 1.6 |
| First published | 2023-10-18 |
| Last published / updated | 2024-02-19 |
| View count | 307 |
| Favourites | 1 |
| Byline shown | Yes |
| `allowDataAccess` | **false** — package not downloadable; documented from the local file |

**Published description**

> Public data source provided by the U.S. Department of Transportation (DOT), illustrating metrics
> of the U.S. gas distribution infrastructure for pipe material, leaks, and excavation damage
> rates reported for all operators.

---

## 3. Data architecture

### 3.1 Connection and model

| Property | Value |
|---|---|
| Data source | `DOT Distribution Report 2010+` |
| Connection class | `federated` |
| Relations | 1 |
| Joins | **None** |
| Custom SQL | None |
| Origin | PHMSA / U.S. DOT gas distribution operator annual reports |

### 3.2 Data source filter — applies to every worksheet

```
REPORT_YEAR >= #2013-01-01#
```

A single quantitative filter at the data source. The source carries filings from 2010 onward (the
name says `2010+`); this restricts every sheet to **2013–2022**, which is what makes the "10 Year"
framing in all three dashboard subtitles accurate. Change it and every ten-year comparison
silently re-bases.

### 3.3 Grain

**One row per operator × state × report year.** Each operator files annually for every state it
operates in, and the filing carries all measures as columns — mileage by material, service counts
by material, mileage by installation decade, leak counts by cause, hazardous leak counts,
excavation damages and tickets.

Two consequences run through the workbook:

- Every national or state figure is a `SUM` across operators; there is no pre-aggregated total to
  read, which is why the calculation library is dominated by additive roll-ups.
- The ten-year endpoint comparison cannot use a simple year filter, because each operator-state
  appears once per year. It uses nested `{FIXED}` LODs that resolve the max and min report year
  across the whole dataset (§6.3).

### 3.4 Schema

A wide filing table. Columns fall into seven families:

| Family | Columns | Purpose |
|---|---|---|
| **Filing identity** | `Operator Key`, `DATAFILE_AS_OF`, `REPORT_YEAR`, `REPORT_NUMBER`, `SUPPLEMENTAL_NUMBER`, `OPERATOR_ID`, `OPERATOR_NAME` | Who filed, when |
| **Location** | `OFFICE_ADDRESS_*`, `HQ_ADDRESS_*`, **`STOP`** | `STOP` is the *state of operation* — the geographic key behind every map and state ranking |
| **Classification** | `COMMODITY`, `OPERATOR_TYPE` | Filing scope |
| **Mainline mileage by material** | `MMILES_STEEL_UNP_BARE`, `MMILES_STEEL_UNP_COATED`, `MMILES_STEEL_CP_BARE`, `MMILES_STEEL_CP_COATED`, `MMILES_PLASTIC`, `MMILES_CI`, `MMILES_DI`, `MMILES_CU`, `MMILES_RCI`, `MMILES_OTHER`, `MMILES_TOTAL` | Steel splits four ways: protected/unprotected × bare/coated |
| **Service counts by material** | `NUM_SRVS_STEEL_UNP_BARE`, `NUM_SRVS_STEEL_UNP_COATED`, `NUM_SRVS_STEEL_CP_BARE`, `NUM_SRVS_STEEL_CP_COATED` and the plastic / iron / copper equivalents | Mirrors the mileage family at service level |
| **Mileage by installation decade** | `MMILES_BY_DCD_PRE1940`, `_1940_TO_1949`, `_1950_TO_1959`, `_1960_TO_1969`, `_UNK`, `_TOTAL` | Basis of the pre-1970 analysis |
| **Leaks and damages** | Main and service leak counts across eight causes, hazardous-leak counts across the same eight, `EXCAVATION_DAMAGES`, `EXCAVATION_TICKETS`, excavation root-cause counts | The Leaks and Damages dashboards |

---

## 4. Parameters

| Caption | Type | Domain | Default | Role |
|---|---|---|---|---|
| **Select an Material Over Time View Chart:** | string list | `1` Pre-1970 Mainline Mileage · `2` Industry Targeted Material · `3` Steel Service Line Counts | `1` | Switches the Material view's trend chart |
| **Select a Leak Type:** | string list | `Main & Service Leaks` · `Main Leaks` · `Service Leaks` | `Main & Service Leaks` | Re-points the **entire** Leaks dashboard |
| **Select an Leaks Over Time View Chart:** | string list | `A` Hazardous Leaks per Main Mile · `B` Corrosion Leaks per Main Mile · `C` Excavation Damage Leaks per Main Mile | `A` | Switches the leak-rate trend metric |

---

## 5. Field dictionary — derived base measures

Several apparent "source" measures are themselves compositions, because PHMSA reports material
categories more finely than the dashboard displays them.

```
Bare Steel (mileage)    [Miles of Steel Protected Bare] + [Miles of Steel Unprotected Bare]
Bare Steel (services)   [Num Srvs Steel Cp Bare] + [Num Srvs Steel Unp Bare]
Coated Steel (mileage)  [Miles of Steel Protected Coated] + [Miles of Steel Unprotected Coated]

Steel Service Lines     [Num Srvs Steel Cp Bare] + [Num Srvs Steel Cp Coated]
                      + [Num Srvs Steel Unp Bare] + [Num Srvs Steel Unp Coated]

Vintage Plastic         [Mmiles Abs Total] + [Mmiles Plastic Total] + [Mmiles Oth Plstc Unk]
```

---

## 6. Calculated fields in use

129 named calculations plus 133 in-view ad-hoc. Grouped by role.

### 6.1 The pre-1970 analysis

```
Pre-1970 Main Mileage (Including Unknown)                       ← 13 sheets
    [Mmiles By Dcd Pre1940] + [Mmiles By Dcd 1940 To 1949]
  + [Mmiles By Dcd 1950 To 1959] + [Mmiles By Dcd 1960 To 1969]
  + [Mmiles By Dcd Unk]

% of System Pre-1970 Main Mileage (Including Unknown)           ← 10 sheets
    SUM([Pre-1970 Main Mileage (Including Unknown)]) / SUM([Mmiles By Dcd Total])
```

The **"(Including Unknown)"** in the field name is a deliberate disclosure: pipe whose
installation decade was not reported is counted as pre-1970. That is a conservative assumption —
undated pipe is more likely old than new — and the name states it rather than burying it. This
produces the 408,828-mile / 30 % headline.

### 6.2 Industry Targeted Material

```
Total Industry Targeted Material                                 ← 6 sheets
    If ISNULL([Miles of Reconditioned Cast Iron])
    then ([Bare Steel] + [Miles of Cast Iron] + [Vintage Plastic])
    else ([Bare Steel] + [Miles of Cast Iron] + [Vintage Plastic]
          + [Miles of Reconditioned Cast Iron]) END

% of System Industry Targeted Mileage
    SUM([Total Industry Targeted Material]) / SUM([Total Miles of Mains])
```

The materials PHMSA prioritises for replacement — bare steel, cast iron, reconditioned cast iron
and vintage plastic. The `ISNULL` branch handles filings that omit the reconditioned cast iron
column: without it a single null would null that operator's entire total.

### 6.3 The ten-year endpoint comparison

The workbook's most systematic pattern — **eight metrics × five fields = 40 calculations** on one
template.

**The template**, shown with pre-1970 mileage:

```
Current Pre-1970    { FIXED : SUM( IF YEAR([Report Year]) = {fixed : YEAR(MAX([Report Year]))}
                                  THEN [Pre-1970 Main Mileage (Including Unknown)] END ) }

Previous Pre-1970   { FIXED : SUM( IF YEAR([Report Year]) = {fixed : YEAR(MIN([Report Year]))}
                                  THEN [Pre-1970 Main Mileage (Including Unknown)] END ) }

Diff Pre-1970       SUM([Current Pre-1970]) - SUM([Previous Pre-1970])
Diff % Pre-1970     (SUM([Current Pre-1970]) - SUM([Previous Pre-1970])) / SUM([Previous Pre-1970])
Direction Pre-1970  IF [Diff Pre-1970] < 0 THEN "▼" ELSEIF > 0 THEN "▲"
                    ELSEIF = 0 THEN "▶" ELSE "?" END
```

**The nested LOD is the mechanic worth understanding.** `{fixed : YEAR(MAX([Report Year]))}`
resolves the latest report year across the entire dataset; the outer `{FIXED : SUM(IF …)}` then
sums only rows from that year — independent of whatever dimensions are in the view. That is what
lets a state-level bar chart display a national ten-year difference without the filter context
interfering.

Because both endpoints are derived, **the comparison re-bases automatically** when a filing year
is added. The only thing pinning the window is the 2013 data source filter.

**The eight metrics:** Pre-1970 mileage (`Pre-1970`) · Industry Targeted Material (`ITM`) · Steel
Service Lines (`SSL`) · Mainline inventory (`M_Inventory`) · Service inventory (`S_Inventory`) ·
Excavation damages (`ExcDmg`) · Excavation tickets (`Tickets`) · Damage rate (`DmgRate`).

`Current DmgRate` is a ratio of two other Current fields rather than its own LOD:

```
Current DmgRate   sum([Current ExcDmg]) / (sum([Current Tickets]) / 1000)
```

### 6.4 Leak aggregation

PHMSA reports leaks split **eight causes × two asset types** (mains, services), and separately for
**hazardous** leaks. The workbook rolls these up in a strict hierarchy.

**Level 1 — per cause, combining mains and services:**
```
Corrosion Leak                     [Main Corrosion Failure Leak] + [Service Corrosion Failure Leak]
Equipment Failure Leak             [Main …] + [Service …]
Excavation Damage Leak             [Main …] + [Service …]
Incorrect Operation Leak           [Main …] + [Service …]
Natural Force Damage Leak          [Main …] + [Service …]
Other Cause Leak                   [Main …] + [Service …]
Other Outside Force Damage Leak    [Main …] + [Service …]
Pipe, Weld, or Joint Failure Leak  [Main …] + [Service …]
```

**Level 2 — all causes, per asset type** (each used in **22 sheets**):
```
All Main Leaks             sum of the eight main-leak columns
All Service Leaks          sum of the eight service-leak columns
All Main/Service Leaks     sum of both
All Main Hazleaks          sum of the eight hazardous main columns
All Service Hazleaks       sum of the eight hazardous service columns
All Main/Service Hazleaks  sum of both
```

**Hazardous equivalents** repeat level 1 with a `Haz` prefix:
```
Haz Main & Service Corrosion Failure Leak   [Total Hazleaks Cor Mains] + [Total Hazleaks Cor Srvs]
… and seven more, one per cause
```

### 6.5 Rate normalisation

The analytical core — raw leak counts are not comparable across operators of different size.

```
Leaks per Mile (Mains)                   sum([All Main Leaks]) / sum([Total Miles of Mains])
Leaks per Mile (Mains & Services)        sum([All Main/Service Leaks]) / sum([Total Miles of Mains])
Leaks per Service Count                  SUM([All Service Leaks]) / SUM([Number of Services Total])

Hazardous Leaks per Mile (Mains)               sum([All Main Hazleaks]) / sum([Total Miles of Mains])
Hazardous Leaks per Mile (Mains & Services)    sum([All Main Hazleaks] + [All Service Hazleaks])
                                               / sum([Total Miles of Mains])
Hazardous Leaks per Service Count              sum([All Service Hazleaks]) / SUM([Number of Services Total])

Damages per 1000 Excavation Tickets      sum([Excavation Damages]) / (sum([Excavation Tickets]) / 1000)
```

The per-1,000-tickets denominator is what makes the Damages view's central finding possible.

### 6.6 The leak-type switcher

The largest family — **over 30 fields** reading `Select a Leak Type:` so the entire Leaks
dashboard re-points between mainline, service and combined.

```
Leak Type - All      CASE [Select a Leak Type:]
                       WHEN "Main & Service Leaks" THEN [All Main/Service Leaks]
                       WHEN "Main Leaks"           THEN [All Main Leaks]
                       WHEN "Service Leaks"        THEN [All Service Leaks] … END
```

Each of the eight causes gets **three** switcher fields:

```
Leak Type - <Cause>        the count, switched
Leak Type - <Cause> %      SUM([Leak Type - <Cause>]) / SUM([Leak Type - All])
Leak Type - <Cause> Avg    SUM([Leak Type - <Cause>]) / MAX([Max-Min Year Diff])
```

**The labels switch too**, which is what keeps the prose accurate in every state:

```
Leak Type        CASE … THEN "leaks" … WHEN "Haz…" THEN "hazardous leaks" END      ← 33 sheets
Leak Type Unit   CASE … THEN "mains and services" / "mains" / "services" END       ← 33 sheets
Leaks per Unit - Unit         'miles of mains' / 'counts'
Leaks per Unit - Unit Title   'Main & Service Leaks per Main Mile' / 'Main Leaks per Main Mile' / …
Leaks per Unit - Unit txt 2   'miles' / 'counts'
```

`Leak Type` and `Leak Type Unit` appear in **33 sheets each** — every title, caption and tooltip
rewrites itself rather than carrying generic wording.

```
Leak Type - % Haz   CASE … WHEN "Main & Service Leaks"
                      THEN SUM([All Main/Service Hazleaks]) / SUM([All Main/Service Leaks]) … END
```
Produces the **40 % Hazardous Leaks** figure.

### 6.7 The material switcher

The same pattern on the Material view:

```
*Material                    CASE … WHEN "1" THEN [Pre-1970 Main Mileage (Including Unknown)]
                                    WHEN "2" THEN [Total Industry Targeted Material]
                                    WHEN "3" THEN [Steel Service Lines] END
*Material_TotalInventory     the matching denominator — Total Miles of Mains or Number of Services
*Material %                  SUM([*Material]) / SUM([*Material_TotalInventory])
*Material Description        'Pre-1970 Mainline Pipe' / 'Industry Targeted Material' / …
*Material Description Type   'Mainline' / 'Mainline' / 'Service'
*Material Units              'Miles' / 'Miles' / 'Counts'
```

Note `*Material_TotalInventory` switching the **denominator** alongside the numerator — steel
service lines must be expressed against service counts, not mileage, or the percentage would be
nonsense.

### 6.8 Excavation root causes

```
EXC1_%  SUM([Excavation One-Call Notification Practice]) / SUM([Excavation Damages])
EXC2_%  SUM([Excavation Locating Practice])              / SUM([Excavation Damages])
EXC3_%  SUM([Excavation Excavation Practice])            / SUM([Excavation Damages])
EXC4_%  SUM([Excavation Other Practice])                 / SUM([Excavation Damages])
```

The 32 % / 24 % / 42 % / 2 % split on the Damages view.

### 6.9 Time-window helpers

```
Max-Min Year Diff   YEAR({FIXED: MAX([Report Year])}) - YEAR({FIXED: MIN([Report Year])}) + 1
                    ← 17 sheets
@UnitedStates       'United States'        ← constant label for the national roll-up
```

`Max-Min Year Diff` returns **10** for the current window and is the divisor behind every
"10 Year Avg". Because it is derived, those averages stay correct if the window changes.

---

## 7. Worksheet specifications

75 worksheets, prefixed by dashboard region.

| Prefix | Count | Dashboard | Role |
|---|---:|---|---|
| **A_** | 9 | Leaks | Leak trend sparklines, one per cause plus `A_All` |
| **B_** | 9 | Leaks | `*_txt` value labels for the A_ series |
| **C_** | 9 | Leaks | A second `*_txt` set — the 10-year averages |
| **D_** | 3 | Leaks | `D_LPM` leak-rate map · `D_Operator Map_Leaks` · `D_Title` |
| **E_** | 6 | Leaks | Totals panel — `E_HazLeak`, `E_LPU`, `E_LPU_Total`, `E_MainLCount`, `E_ServiceLCount`, `E_OpenLeaks` |
| **F_** | 8 | Material | Inventory — `F_InvChart`, `F_Map_Material`, `F_Material`, `F_MMat`, `F_SMat`, `F_ServiceCo`, `F_All_MInv_Txt`, `F_Title` |
| **G_** | 11 | Material | Metric band — `G_Pre70`, `G_ITM_txt`, `G_SSL`, `G_IIM` and their `_txt` / `_txt2` label variants |
| **H_** | 14 | Damages | `H_Map`, `H_Bar`, `H_DmgRate`, `H_ExcDmg`, `H_ExcTicket`, `H_Exc1`–`H_Exc4` with `_Txt` pairs, `H_Title` |
| **I_** | 6 | Damages | Totals panel — `I_DmgRate`, `I_ExcDmg`, `I_Tickets` and their `_txt` labels |

**The `_txt` convention.** Nearly half the sheets exist only to print a number beside a chart.
Tableau cannot freely place a label outside a mark, so the value is built as its own text sheet
and positioned on the dashboard. That is why 75 worksheets produce what reads as roughly 20
charts.

---

## 8. Dashboard specification

Four dashboards, each **1400 × 826 px**, sharing one template.

```
Masthead     "U.S. Gas Distribution Infrastructure Dashboard"
             subtitle naming the metric family and window, e.g.
             "10 Year Reportable Leaks Repaired from 2013 to 2022"
             right: three view tabs — Material · Leaks · Damages (active one outlined)
             "KPIs shown for the CY2022 reporting year" · "Data Source Information"
Metric band  left panel — a row of metric cards, each with value, share,
             10-year average or difference, and a sparkline
Totals       right panel — headline totals with 10-year differences and trends
Main         lower left: proportional-symbol state map ("Select Metric by State")
             lower right: ranked horizontal bar chart of states
```

| Dashboard | Metric band | Totals panel | Ranked bars |
|---|---|---|---|
| **Material** | Pre-1970 · Industry Targeted · Steel Service Lines | Miles of Mainline · Service Count, each with an 11-material breakdown table | Pre-1970 % of total infrastructure, by state |
| **Leaks** | Eight leak causes with counts, shares and 10-year averages | Total Leaks · Mainline · Service · Hazardous % · Open at EOY · Leak Rate | Leak rate per main mile, by state |
| **Damages** | Four excavation root causes | Damages · Tickets · Damages per 1,000 Tickets | Damages per 1,000 tickets, by state |
| **Data Source** | — | — | Provenance and methodology; carries no worksheets |

---

## 9. Interactivity

### 9.1 Filter actions (3)

| Caption | Target dashboard | Field |
|---|---|---|
| `Filter:State:Material` | Material | `STOP` (state of operation) |
| `Filter:State:Leaks` | Leaks | `STOP` |
| `Filter:State:Damages` | Damages | `STOP` |

### 9.2 Highlight actions (3)

| Caption | Target dashboard | Field |
|---|---|---|
| `Highlight:State:Material` | Material | `Operating State` |
| `Highlight:State:Leaks` | Leaks | `Operating State` |
| `Highlight:State:Damages` | Damages | `Operating State` |

Each dashboard carries its **own** filter/highlight pair on state, so selecting a state filters
the bars and highlights the map without leaking selection across dashboards.

### 9.3 Native parameter controls

| Control | Dashboard | Parameter |
|---|---|---|
| "Select a Leak Group" | Leaks | `Select a Leak Type:` |
| "Select an Material Over Time View Chart" | Material | material switcher |
| "Select Metric by State" | all three | rate metric for the map and bars |

### 9.4 Dynamic Zone Visibility

**None.** View switching uses **four separately published dashboards** with tab-styled navigation
in the masthead, not zone swapping.

---

## 10. Design system

| Token | Use |
|---|---|
| Mid blue | Headings, bars, trend lines, map symbols, KPI values |
| Deep blue | Active tab outline, top-ranked bar |
| Pale blue | Bar tracks, map fills, low-value states |
| White | Ground and card interiors |
| Light grey | Panel borders, table rules, secondary text |

A restrained single-hue scheme — blue on white, with **saturation** rather than hue carrying
magnitude. Appropriate to a regulatory subject, and it keeps eight small sparklines legible in one
row.

**Layout craft** — every panel is a white card with a thin grey rule and a small-caps heading. The
metric band uses equal-width cards; sparklines are unlabelled except at their endpoints, so the
row reads as shape rather than detail. The active view tab is outlined, the others grey.

---

## 11. Rebuild / maintenance runbook

**Adding a filing year**

1. Append the new PHMSA annual report rows, preserving column names.
2. **Update the data source filter.** It is pinned at `REPORT_YEAR >= 2013`; leaving it gives an
   11-year window while every dashboard subtitle still says "10 Year".
3. Everything else re-bases automatically — `Current …` / `Previous …` resolve the endpoints from
   the data, and `Max-Min Year Diff` recomputes the divisor behind the 10-year averages.
4. Update the masthead subtitles and the "KPIs shown for the CY2022 reporting year" caption; these
   are static text.

**The pre-1970 assumption**

`Pre-1970 Main Mileage (Including Unknown)` counts undated pipe as pre-1970. If PHMSA improves
decade reporting, reconsider whether `[Mmiles By Dcd Unk]` should still be included — and if it is
removed, change the field name with it, since the name *is* the disclosure.

**Adding a leak cause**

The eight-cause taxonomy is hard-coded across roughly 40 fields. A ninth requires:
1. A level-1 combiner (`<Cause> Leak = [Main …] + [Service …]`).
2. A hazardous equivalent (`Haz Main & Service <Cause> Leak`).
3. Adding the column to `All Main Leaks`, `All Service Leaks` and both hazardous roll-ups.
4. Three switcher fields (`Leak Type - <Cause>`, `… %`, `… Avg`).
5. An `A_` sparkline sheet plus `B_` and `C_` text sheets.

**The `ISNULL` guard**

`Total Industry Targeted Material` guards against a missing reconditioned-cast-iron column. If
other material columns become optional in future filings they need the same treatment — an
unguarded null will silently null the operator's whole total.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | DOT Distribution Report 2010+ |
| Connection class | `federated` |
| Tables / relations | 0 · joins: 0 |
| Extract rows | 28,126 |
| Physical columns materialised | 279 |
| Source fields in data pane | 279 |
| Calculated fields (used / total) | 129 / 184 |
| Parameters | 3 |
| Data source filters | 1 — see below |

**Data source filters run before every worksheet-level filter, on every sheet:**

- none:REPORT_YEAR:qk: #2013-01-01# to *

### Source fields

279 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Geography

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Operating State**<br>`STOP` | string · Dimension | 64 | yes | 75 | **State of operation.** The geographic key behind every map and state ranking. |

#### Filing identity

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Report Year**<br>`REPORT_YEAR` | date · Dimension | 19 | yes | 72 | Calendar year the filing covers. The data source filter restricts this to 2013 and later. |
| **Additional Information**<br>`ADDITIONAL_INFORMATION` | string · Dimension | 4,519 | yes | 0 | Free-text remarks the operator attached to the filing. *(not used in any sheet)* |
| **Datafile As Of**<br>`DATAFILE_AS_OF` | datetime · Dimension | 14 | yes | 0 | Date the source extract was produced. *(not used in any sheet)* |
| **Filing Date**<br>`FILING_DATE` | datetime · Dimension | 18,751 | yes | 0 | Date the report was filed. *(not used in any sheet)* |
| **Form Rev**<br>`FORM_REV` | string · Dimension | 6 | yes | 0 | Revision of the PHMSA form used. *(not used in any sheet)* |
| **Report Date**<br>`REPORT_DATE` | datetime · Dimension | 7,041 | yes | 0 | Date on the report. *(not used in any sheet)* |
| **Report Number**<br>`REPORT_NUMBER` | real · Dimension | 28,126 | yes | 0 | PHMSA report number. *(not used in any sheet)* |
| **Report Submission Type**<br>`REPORT_SUBMISSION_TYPE` | string · Dimension | 2 | yes | 0 | Whether the filing is an original or a resubmission. *(not used in any sheet)* |
| **Supplemental Number**<br>`SUPPLEMENTAL_NUMBER` | real · Dimension | 26,799 | yes | 0 | Supplemental filing sequence. *(not used in any sheet)* |

#### Hazardous leaks by cause

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Total Hazleaks Cor Mains**<br>`TOTAL_HAZLEAKS_COR_MAINS` | real · Measure (Sum) | 177 | yes | 25 | Hazardous leaks repaired on mainlines attributed to corrosion failure. Hazardous leaks are those requiring immediate repair. |
| **Total Hazleaks Cor Srvs**<br>`TOTAL_HAZLEAKS_COR_SRVS` | real · Measure (Sum) | 386 | yes | 25 | Hazardous leaks repaired on service lines attributed to corrosion failure. Hazardous leaks are those requiring immediate repair. |
| **Total Hazleaks Eq Mains**<br>`TOTAL_HAZLEAKS_EQ_MAINS` | real · Measure (Sum) | 161 | yes | 24 | Hazardous leaks repaired on mainlines attributed to equipment failure. Hazardous leaks are those requiring immediate repair. |
| **Total Hazleaks Eq Srvs**<br>`TOTAL_HAZLEAKS_EQ_SRVS` | real · Measure (Sum) | 277 | yes | 24 | Hazardous leaks repaired on service lines attributed to equipment failure. Hazardous leaks are those requiring immediate repair. |
| **Total Hazleaks Ex Mains**<br>`TOTAL_HAZLEAKS_EX_MAINS` | real · Measure (Sum) | 230 | yes | 24 | Hazardous leaks repaired on mainlines attributed to excavation damage. Hazardous leaks are those requiring immediate repair. |
| **Total Hazleaks Ex Srvs**<br>`TOTAL_HAZLEAKS_EX_SRVS` | real · Measure (Sum) | 519 | yes | 24 | Hazardous leaks repaired on service lines attributed to excavation damage. Hazardous leaks are those requiring immediate repair. |
| **Total Hazleaks Mat Weld Mains**<br>`TOTAL_HAZLEAKS_MAT_WELD_MAINS` | real · Measure (Sum) | 109 | yes | 24 | Hazardous leaks repaired on mainlines attributed to pipe, weld or joint failure. Hazardous leaks are those requiring immediate repair. |
| **Total Hazleaks Mat Weld Srvs**<br>`TOTAL_HAZLEAKS_MAT_WELD_SRVS` | real · Measure (Sum) | 173 | yes | 24 | Hazardous leaks repaired on service lines attributed to pipe, weld or joint failure. Hazardous leaks are those requiring immediate repair. |
| **Total Hazleaks Nf Mains**<br>`TOTAL_HAZLEAKS_NF_MAINS` | real · Measure (Sum) | 109 | yes | 24 | Hazardous leaks repaired on mainlines attributed to natural force damage. Hazardous leaks are those requiring immediate repair. |
| **Total Hazleaks Nf Srvs**<br>`TOTAL_HAZLEAKS_NF_SRVS` | real · Measure (Sum) | 177 | yes | 24 | Hazardous leaks repaired on service lines attributed to natural force damage. Hazardous leaks are those requiring immediate repair. |
| **Total Hazleaks Of Dam Mains**<br>`TOTAL_HAZLEAKS_OF_DAM_MAINS` | real · Measure (Sum) | 44 | yes | 24 | Hazardous leaks repaired on mainlines attributed to other outside force damage. Hazardous leaks are those requiring immediate repair. |
| **Total Hazleaks Of Dam Srvs**<br>`TOTAL_HAZLEAKS_OF_DAM_SRVS` | real · Measure (Sum) | 140 | yes | 24 | Hazardous leaks repaired on service lines attributed to other outside force damage. Hazardous leaks are those requiring immediate repair. |
| **Total Hazleaks Op Mains**<br>`TOTAL_HAZLEAKS_OP_MAINS` | real · Measure (Sum) | 29 | yes | 24 | Hazardous leaks repaired on mainlines attributed to incorrect operation. Hazardous leaks are those requiring immediate repair. |
| **Total Hazleaks Op Srvs**<br>`TOTAL_HAZLEAKS_OP_SRVS` | real · Measure (Sum) | 96 | yes | 24 | Hazardous leaks repaired on service lines attributed to incorrect operation. Hazardous leaks are those requiring immediate repair. |
| **Total Hazleaks Ot Mains**<br>`TOTAL_HAZLEAKS_OT_MAINS` | real · Measure (Sum) | 127 | yes | 24 | Hazardous leaks repaired on mainlines attributed to other causes. Hazardous leaks are those requiring immediate repair. |
| **Total Hazleaks Ot Srvs**<br>`TOTAL_HAZLEAKS_OT_SRVS` | real · Measure (Sum) | 194 | yes | 24 | Hazardous leaks repaired on service lines attributed to other causes. Hazardous leaks are those requiring immediate repair. |

#### Leaks by cause

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Main Corrosion Failure Leak**<br>`TOTAL_LEAKS_COR_MAINS` | real · Measure (Sum) | 534 | yes | 25 | Leaks repaired on mainlines attributed to corrosion failure. |
| **Service Corrosion Failure Leak**<br>`TOTAL_LEAKS_COR_SRVS` | real · Measure (Sum) | 586 | yes | 25 | Leaks repaired on service lines attributed to corrosion failure. |
| **Main Equipment Failure Leak**<br>`TOTAL_LEAKS_EQ_MAINS` | real · Measure (Sum) | 269 | yes | 24 | Leaks repaired on mainlines attributed to equipment failure. |
| **Main Excavation Damage Leak**<br>`TOTAL_LEAKS_EX_MAINS` | real · Measure (Sum) | 310 | yes | 24 | Leaks repaired on mainlines attributed to excavation damage. |
| **Main Incorrect Operation Leak**<br>`TOTAL_LEAKS_OP_MAINS` | real · Measure (Sum) | 132 | yes | 24 | Leaks repaired on mainlines attributed to incorrect operation. |
| **Main Natural Force Damage Leak**<br>`TOTAL_LEAKS_NF_MAINS` | real · Measure (Sum) | 244 | yes | 24 | Leaks repaired on mainlines attributed to natural force damage. |
| **Main Other Cause Leak**<br>`TOTAL_LEAKS_OT_MAINS` | real · Measure (Sum) | 395 | yes | 24 | Leaks repaired on mainlines attributed to other causes. |
| **Main Other Outside Force Damage Leak**<br>`TOTAL_LEAKS_OF_DAM_MAINS` | real · Measure (Sum) | 87 | yes | 24 | Leaks repaired on mainlines attributed to other outside force damage. |
| **Main Pipe, Weld, or Joint Failure Leak**<br>`TOTAL_LEAKS_MAT_WELD_MAINS` | real · Measure (Sum) | 317 | yes | 24 | Leaks repaired on mainlines attributed to pipe, weld or joint failure. |
| **Service Equipment Failure Leak**<br>`TOTAL_LEAKS_EQ_SRVS` | real · Measure (Sum) | 536 | yes | 24 | Leaks repaired on service lines attributed to equipment failure. |
| **Service Excavation Damage Leak**<br>`TOTAL_LEAKS_EX_SRVS` | real · Measure (Sum) | 628 | yes | 24 | Leaks repaired on service lines attributed to excavation damage. |
| **Service Incorrect Operation Leak**<br>`TOTAL_LEAKS_OP_SRVS` | real · Measure (Sum) | 192 | yes | 24 | Leaks repaired on service lines attributed to incorrect operation. |
| **Service Natural Force Damage Leak**<br>`TOTAL_LEAKS_NF_SRVS` | real · Measure (Sum) | 308 | yes | 24 | Leaks repaired on service lines attributed to natural force damage. |
| **Service Other Cause Leak**<br>`TOTAL_LEAKS_OT_SRVS` | real · Measure (Sum) | 375 | yes | 24 | Leaks repaired on service lines attributed to other causes. |
| **Service Other Outside Force Damage Leak**<br>`TOTAL_LEAKS_OF_DAM_SRVS` | real · Measure (Sum) | 218 | yes | 24 | Leaks repaired on service lines attributed to other outside force damage. |
| **Service Pipe, Weld, or Joint Failure Leak**<br>`TOTAL_LEAKS_MAT_WELD_SRVS` | real · Measure (Sum) | 443 | yes | 24 | Leaks repaired on service lines attributed to pipe, weld or joint failure. |

#### Mainline mileage by install decade

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Mmiles By Dcd 1940 To 1949**<br>`MMILES_BY_DCD_1940_TO_1949` | real · Measure (Sum) | 1,027 | yes | 13 | Miles of mainline installed 1940–1949. The vintage analysis reads these columns; the unrecorded bucket is deliberately counted with pre-1970 pipe. |
| **Mmiles By Dcd 1950 To 1959**<br>`MMILES_BY_DCD_1950_TO_1959` | real · Measure (Sum) | 1,307 | yes | 13 | Miles of mainline installed 1950–1959. The vintage analysis reads these columns; the unrecorded bucket is deliberately counted with pre-1970 pipe. |
| **Mmiles By Dcd 1960 To 1969**<br>`MMILES_BY_DCD_1960_TO_1969` | real · Measure (Sum) | 1,604 | yes | 13 | Miles of mainline installed 1960–1969. The vintage analysis reads these columns; the unrecorded bucket is deliberately counted with pre-1970 pipe. |
| **Mmiles By Dcd Pre1940**<br>`MMILES_BY_DCD_PRE1940` | real · Measure (Sum) | 2,122 | yes | 13 | Miles of mainline installed before 1940. The vintage analysis reads these columns; the unrecorded bucket is deliberately counted with pre-1970 pipe. |
| **Mmiles By Dcd Unk**<br>`MMILES_BY_DCD_UNK` | real · Measure (Sum) | 1,146 | yes | 13 | Miles of mainline installed an unrecorded decade. The vintage analysis reads these columns; the unrecorded bucket is deliberately counted with pre-1970 pipe. |
| **Mmiles By Dcd Total**<br>`MMILES_BY_DCD_TOTAL` | real · Measure (Sum) | 4,110 | yes | 10 | Miles of mainline installed any period. The vintage analysis reads these columns; the unrecorded bucket is deliberately counted with pre-1970 pipe. |
| **Mmiles By Dcd 1970 To 1979**<br>`MMILES_BY_DCD_1970_TO_1979` | real · Measure (Sum) | 1,607 | yes | 0 | Miles of mainline installed 1970–1979. The vintage analysis reads these columns; the unrecorded bucket is deliberately counted with pre-1970 pipe. *(not used in any sheet)* |
| **Mmiles By Dcd 1980 To 1989**<br>`MMILES_BY_DCD_1980_TO_1989` | real · Measure (Sum) | 1,527 | yes | 0 | Miles of mainline installed 1980–1989. The vintage analysis reads these columns; the unrecorded bucket is deliberately counted with pre-1970 pipe. *(not used in any sheet)* |
| **Mmiles By Dcd 1990 To 1999**<br>`MMILES_BY_DCD_1990_TO_1999` | real · Measure (Sum) | 1,775 | yes | 0 | Miles of mainline installed 1990–1999. The vintage analysis reads these columns; the unrecorded bucket is deliberately counted with pre-1970 pipe. *(not used in any sheet)* |
| **Mmiles By Dcd 2000 To 2009**<br>`MMILES_BY_DCD_2000_TO_2009` | real · Measure (Sum) | 2,138 | yes | 0 | Miles of mainline installed 2000–2009. The vintage analysis reads these columns; the unrecorded bucket is deliberately counted with pre-1970 pipe. *(not used in any sheet)* |
| **Mmiles By Dcd 2010 To 2019**<br>`MMILES_BY_DCD_2010_TO_2019` | real · Measure (Sum) | 2,849 | yes | 0 | Miles of mainline installed 2010–2019. The vintage analysis reads these columns; the unrecorded bucket is deliberately counted with pre-1970 pipe. *(not used in any sheet)* |
| **Mmiles By Dcd 2020 To 2029**<br>`MMILES_BY_DCD_2020_TO_2029` | real · Measure (Sum) | 1,352 | yes | 0 | Miles of mainline installed 2020–2029. The vintage analysis reads these columns; the unrecorded bucket is deliberately counted with pre-1970 pipe. *(not used in any sheet)* |

#### Excavation damage

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Excavation Damages**<br>`EXCAV_DAMAGES` | real · Measure (Sum) | 585 | yes | 13 | Excavation damages reported for the year. The numerator of the damage rate. |
| **Excavation Tickets**<br>`EXCAV_TICKETS` | real · Measure (Sum) | 3,958 | yes | 9 | One-call locate tickets received. The denominator of the damage rate. |
| **Excavation Excavation Practice**<br>`EXCAV_EXCAV` | real · Measure (Sum) | 308 | yes | 2 | Damages whose root cause was excavation practice. |
| **Excavation Locating Practice**<br>`EXCAV_LOCATING` | real · Measure (Sum) | 278 | yes | 2 | Damages whose root cause was locating practice. |
| **Excavation One-Call Notification Practice**<br>`EXCAV_ONECALL` | real · Measure (Sum) | 242 | yes | 2 | Damages whose root cause was one-call notification practice. |
| **Excavation Other Practice**<br>`EXCAV_OTHER` | real · Measure (Sum) | 62 | yes | 2 | Damages attributed to some other root cause. |

#### System totals

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Number of Services Total**<br>`NUM_SRVCS_TOTAL` | real · Measure (Sum) | 5,920 | yes | 12 | Total number of service lines reported for the state and year. |
| **Total Miles of Mains**<br>`MMILES_TOTAL` | real · Measure (Sum) | 4,069 | yes | 11 | Total miles of distribution mainline the operator reports for the state and year. The denominator behind every mileage share in this workbook. |
| **Mmiles Part B2 Total**<br>`MMILES_PART_B2_TOTAL` | real · Measure (Sum) | 3,277 | yes | 0 | Mileage total carried on Part B2 of the report form. *(not used in any sheet)* |
| **Num Srvs Part B3 Total**<br>`NUM_SRVS_PART_B3_TOTAL` | real · Measure (Sum) | 5,006 | yes | 0 | Service-count total carried on Part B3 of the report form. *(not used in any sheet)* |

#### Service counts by material

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Num Srvs Steel Cp Bare**<br>`NUM_SRVS_STEEL_CP_BARE` | real · Measure (Sum) | 916 | yes | 8 | Count of service lines made of cathodically protected bare steel, as carried on the report's summary line. |
| **Num Srvs Steel Cp Coated**<br>`NUM_SRVS_STEEL_CP_COATED` | real · Measure (Sum) | 3,626 | yes | 8 | Count of service lines made of cathodically protected coated steel, as carried on the report's summary line. |
| **Num Srvs Steel Unp Bare**<br>`NUM_SRVS_STEEL_UNP_BARE` | real · Measure (Sum) | 2,205 | yes | 8 | Count of service lines made of unprotected bare steel, as carried on the report's summary line. |
| **Num Srvs Steel Unp Coated**<br>`NUM_SRVS_STEEL_UNP_COATED` | real · Measure (Sum) | 1,006 | yes | 7 | Count of service lines made of unprotected coated steel, as carried on the report's summary line. |
| **Num Srvs Abs Total**<br>`NUM_SRVS_ABS_TOTAL` | real · Measure (Sum) | 151 | yes | 1 | Count of service lines made of ABS plastic, all diameters. |
| **Num Srvs Ci**<br>`NUM_SRVS_CI` | real · Measure (Sum) | 190 | yes | 1 | Count of service lines made of cast iron, as carried on the report's summary line. |
| **Num Srvs Cu**<br>`NUM_SRVS_CU` | real · Measure (Sum) | 794 | yes | 1 | Count of service lines made of copper, as carried on the report's summary line. |
| **Num Srvs Di**<br>`NUM_SRVS_DI` | real · Measure (Sum) | 11 | yes | 1 | Count of service lines made of ductile iron, as carried on the report's summary line. |
| **Num Srvs Oth Plstc Total**<br>`NUM_SRVS_OTH_PLSTC_TOTAL` | real · Measure (Sum) | 78 | yes | 1 | Count of service lines made of other or unspecified plastic, all diameters. |
| **Num Srvs Other**<br>`NUM_SRVS_OTHER` | real · Measure (Sum) | 958 | yes | 1 | Count of service lines made of other materials, as carried on the report's summary line. |
| **Num Srvs Pe Total**<br>`NUM_SRVS_PE_TOTAL` | real · Measure (Sum) | 4,822 | yes | 1 | Count of service lines made of polyethylene plastic, all diameters. |
| **Num Srvs Plastic Total**<br>`NUM_SRVS_PLASTIC_TOTAL` | real · Measure (Sum) | 592 | yes | 1 | Count of service lines made of plastic of any type, all diameters. |
| **Num Srvs Rci**<br>`NUM_SRVS_RCI` | real · Measure (Sum) | 2 | yes | 1 | Count of service lines made of reconditioned cast iron, as carried on the report's summary line. |
| **Num Srvs Ci Wr Total**<br>`NUM_SRVS_CI_WR_TOTAL` | real · Measure (Sum) | 190 | yes | 0 | Count of service lines made of cast or wrought iron, all diameters. *(not used in any sheet)* |
| **Num Srvs Cu Total**<br>`NUM_SRVS_CU_TOTAL` | real · Measure (Sum) | 812 | yes | 0 | Count of service lines made of copper, all diameters. *(not used in any sheet)* |
| **Num Srvs Di Total**<br>`NUM_SRVS_DI_TOTAL` | real · Measure (Sum) | 11 | yes | 0 | Count of service lines made of ductile iron, all diameters. *(not used in any sheet)* |
| **Num Srvs Other Material Detail**<br>`NUM_SRVS_OTHER_MATERIAL_DETAIL` | string · Dimension | 65 | yes | 0 | Free-text description of what the operator counted as `other` service material. *(not used in any sheet)* |
| **Num Srvs Other Total**<br>`NUM_SRVS_OTHER_TOTAL` | real · Measure (Sum) | 912 | yes | 0 | Count of service lines made of other materials, all diameters. *(not used in any sheet)* |
| **Num Srvs Plastic**<br>`NUM_SRVS_PLASTIC` | real · Measure (Sum) | 4,807 | yes | 0 | Count of service lines made of plastic of any type, as carried on the report's summary line. *(not used in any sheet)* |
| **Num Srvs Rci Total**<br>`NUM_SRVS_RCI_TOTAL` | real · Measure (Sum) | 2 | yes | 0 | Count of service lines made of reconditioned cast iron, all diameters. *(not used in any sheet)* |
| **Num Srvs Steel Total**<br>`NUM_SRVS_STEEL_TOTAL` | real · Measure (Sum) | 3,696 | yes | 0 | Count of service lines made of steel, all diameters. *(not used in any sheet)* |

#### Mainline mileage by material and diameter

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Mmiles Oth Plstc Unk**<br>`MMILES_OTH_PLSTC_UNK` | real · Measure (Sum) | 2 | yes | 7 | Miles of mainline made of other or unspecified plastic, diameter not recorded. |
| **Mmiles Abs 2In To 4In**<br>`MMILES_ABS_2IN_TO_4IN` | real · Measure (Sum) | 111 | yes | 0 | Miles of mainline made of ABS plastic, 2 to 4 inches. *(not used in any sheet)* |
| **Mmiles Abs 4In To 8In**<br>`MMILES_ABS_4IN_TO_8IN` | real · Measure (Sum) | 1 | yes | 0 | Miles of mainline made of ABS plastic, 4 to 8 inches. *(not used in any sheet)* |
| **Mmiles Abs 8In To 12In**<br>`MMILES_ABS_8IN_TO_12IN` | real · Measure (Sum) | 1 | yes | 0 | Miles of mainline made of ABS plastic, 8 to 12 inches. *(not used in any sheet)* |
| **Mmiles Abs Gt12In**<br>`MMILES_ABS_GT12IN` | real · Measure (Sum) | 1 | yes | 0 | Miles of mainline made of ABS plastic, over 12 inches. *(not used in any sheet)* |
| **Mmiles Abs Lt2In**<br>`MMILES_ABS_LT2IN` | real · Measure (Sum) | 93 | yes | 0 | Miles of mainline made of ABS plastic, under 2 inches. *(not used in any sheet)* |
| **Mmiles Abs Unk**<br>`MMILES_ABS_UNK` | real · Measure (Sum) | 1 | yes | 0 | Miles of mainline made of ABS plastic, diameter not recorded. *(not used in any sheet)* |
| **Mmiles Ci Wr 2In To 4In**<br>`MMILES_CI_WR_2IN_TO_4IN` | real · Measure (Sum) | 892 | yes | 0 | Miles of mainline made of cast or wrought iron, 2 to 4 inches. *(not used in any sheet)* |
| **Mmiles Ci Wr 4In To 8In**<br>`MMILES_CI_WR_4IN_TO_8IN` | real · Measure (Sum) | 1,252 | yes | 0 | Miles of mainline made of cast or wrought iron, 4 to 8 inches. *(not used in any sheet)* |
| **Mmiles Ci Wr 8In To 12In**<br>`MMILES_CI_WR_8IN_TO_12IN` | real · Measure (Sum) | 595 | yes | 0 | Miles of mainline made of cast or wrought iron, 8 to 12 inches. *(not used in any sheet)* |
| **Mmiles Ci Wr Gt12In**<br>`MMILES_CI_WR_GT12IN` | real · Measure (Sum) | 301 | yes | 0 | Miles of mainline made of cast or wrought iron, over 12 inches. *(not used in any sheet)* |
| **Mmiles Ci Wr Lt2In**<br>`MMILES_CI_WR_LT2IN` | real · Measure (Sum) | 219 | yes | 0 | Miles of mainline made of cast or wrought iron, under 2 inches. *(not used in any sheet)* |
| **Mmiles Ci Wr Unk**<br>`MMILES_CI_WR_UNK` | real · Measure (Sum) | 5 | yes | 0 | Miles of mainline made of cast or wrought iron, diameter not recorded. *(not used in any sheet)* |
| **Mmiles Cu 2In To 4In**<br>`MMILES_CU_2IN_TO_4IN` | real · Measure (Sum) | 1 | yes | 0 | Miles of mainline made of copper, 2 to 4 inches. *(not used in any sheet)* |
| **Mmiles Cu 4In To 8In**<br>`MMILES_CU_4IN_TO_8IN` | real · Measure (Sum) | 1 | yes | 0 | Miles of mainline made of copper, 4 to 8 inches. *(not used in any sheet)* |
| **Mmiles Cu 8In To 12In**<br>`MMILES_CU_8IN_TO_12IN` | real · Measure (Sum) | 1 | yes | 0 | Miles of mainline made of copper, 8 to 12 inches. *(not used in any sheet)* |
| **Mmiles Cu Gt12In**<br>`MMILES_CU_GT12IN` | real · Measure (Sum) | 1 | yes | 0 | Miles of mainline made of copper, over 12 inches. *(not used in any sheet)* |
| **Mmiles Cu Lt2In**<br>`MMILES_CU_LT2IN` | real · Measure (Sum) | 32 | yes | 0 | Miles of mainline made of copper, under 2 inches. *(not used in any sheet)* |
| **Mmiles Cu Unk**<br>`MMILES_CU_UNK` | real · Measure (Sum) | 1 | yes | 0 | Miles of mainline made of copper, diameter not recorded. *(not used in any sheet)* |
| **Mmiles Di 2In To 4In**<br>`MMILES_DI_2IN_TO_4IN` | real · Measure (Sum) | 116 | yes | 0 | Miles of mainline made of ductile iron, 2 to 4 inches. *(not used in any sheet)* |
| **Mmiles Di 4In To 8In**<br>`MMILES_DI_4IN_TO_8IN` | real · Measure (Sum) | 297 | yes | 0 | Miles of mainline made of ductile iron, 4 to 8 inches. *(not used in any sheet)* |
| **Mmiles Di 8In To 12In**<br>`MMILES_DI_8IN_TO_12IN` | real · Measure (Sum) | 33 | yes | 0 | Miles of mainline made of ductile iron, 8 to 12 inches. *(not used in any sheet)* |
| **Mmiles Di Gt12In**<br>`MMILES_DI_GT12IN` | real · Measure (Sum) | 18 | yes | 0 | Miles of mainline made of ductile iron, over 12 inches. *(not used in any sheet)* |
| **Mmiles Di Lt2In**<br>`MMILES_DI_LT2IN` | real · Measure (Sum) | 11 | yes | 0 | Miles of mainline made of ductile iron, under 2 inches. *(not used in any sheet)* |
| **Mmiles Di Unk**<br>`MMILES_DI_UNK` | real · Measure (Sum) | 1 | yes | 0 | Miles of mainline made of ductile iron, diameter not recorded. *(not used in any sheet)* |
| **Mmiles Oth Plstc 2In To 4In**<br>`MMILES_OTH_PLSTC_2IN_TO_4IN` | real · Measure (Sum) | 32 | yes | 0 | Miles of mainline made of other or unspecified plastic, 2 to 4 inches. *(not used in any sheet)* |
| **Mmiles Oth Plstc 4In To 8In**<br>`MMILES_OTH_PLSTC_4IN_TO_8IN` | real · Measure (Sum) | 11 | yes | 0 | Miles of mainline made of other or unspecified plastic, 4 to 8 inches. *(not used in any sheet)* |
| **Mmiles Oth Plstc 8In To 12In**<br>`MMILES_OTH_PLSTC_8IN_TO_12IN` | real · Measure (Sum) | 5 | yes | 0 | Miles of mainline made of other or unspecified plastic, 8 to 12 inches. *(not used in any sheet)* |
| **Mmiles Oth Plstc Gt12In**<br>`MMILES_OTH_PLSTC_GT12IN` | real · Measure (Sum) | 2 | yes | 0 | Miles of mainline made of other or unspecified plastic, over 12 inches. *(not used in any sheet)* |
| **Mmiles Oth Plstc Lt2In**<br>`MMILES_OTH_PLSTC_LT2IN` | real · Measure (Sum) | 32 | yes | 0 | Miles of mainline made of other or unspecified plastic, under 2 inches. *(not used in any sheet)* |
| **Mmiles Other 2In To 4In**<br>`MMILES_OTHER_2IN_TO_4IN` | real · Measure (Sum) | 367 | yes | 0 | Miles of mainline made of other materials, 2 to 4 inches. *(not used in any sheet)* |
| **Mmiles Other 4In To 8In**<br>`MMILES_OTHER_4IN_TO_8IN` | real · Measure (Sum) | 61 | yes | 0 | Miles of mainline made of other materials, 4 to 8 inches. *(not used in any sheet)* |
| **Mmiles Other 8In To 12In**<br>`MMILES_OTHER_8IN_TO_12IN` | real · Measure (Sum) | 151 | yes | 0 | Miles of mainline made of other materials, 8 to 12 inches. *(not used in any sheet)* |
| **Mmiles Other Gt12In**<br>`MMILES_OTHER_GT12IN` | real · Measure (Sum) | 18 | yes | 0 | Miles of mainline made of other materials, over 12 inches. *(not used in any sheet)* |
| **Mmiles Other Lt2In**<br>`MMILES_OTHER_LT2IN` | real · Measure (Sum) | 561 | yes | 0 | Miles of mainline made of other materials, under 2 inches. *(not used in any sheet)* |
| **Mmiles Other Unk**<br>`MMILES_OTHER_UNK` | real · Measure (Sum) | 144 | yes | 0 | Miles of mainline made of other materials, diameter not recorded. *(not used in any sheet)* |
| **Mmiles Pe 2In To 4In**<br>`MMILES_PE_2IN_TO_4IN` | real · Measure (Sum) | 2,897 | yes | 0 | Miles of mainline made of polyethylene plastic, 2 to 4 inches. *(not used in any sheet)* |
| **Mmiles Pe 4In To 8In**<br>`MMILES_PE_4IN_TO_8IN` | real · Measure (Sum) | 1,878 | yes | 0 | Miles of mainline made of polyethylene plastic, 4 to 8 inches. *(not used in any sheet)* |
| **Mmiles Pe 8In To 12In**<br>`MMILES_PE_8IN_TO_12IN` | real · Measure (Sum) | 902 | yes | 0 | Miles of mainline made of polyethylene plastic, 8 to 12 inches. *(not used in any sheet)* |
| **Mmiles Pe Gt12In**<br>`MMILES_PE_GT12IN` | real · Measure (Sum) | 35 | yes | 0 | Miles of mainline made of polyethylene plastic, over 12 inches. *(not used in any sheet)* |
| **Mmiles Pe Lt2In**<br>`MMILES_PE_LT2IN` | real · Measure (Sum) | 3,748 | yes | 0 | Miles of mainline made of polyethylene plastic, under 2 inches. *(not used in any sheet)* |
| **Mmiles Pe Unk**<br>`MMILES_PE_UNK` | real · Measure (Sum) | 160 | yes | 0 | Miles of mainline made of polyethylene plastic, diameter not recorded. *(not used in any sheet)* |
| **Mmiles Plastic 2In To 4In**<br>`MMILES_PLASTIC_2IN_TO_4IN` | real · Measure (Sum) | 165 | yes | 0 | Miles of mainline made of plastic of any type, 2 to 4 inches. *(not used in any sheet)* |
| **Mmiles Plastic 4In To 8In**<br>`MMILES_PLASTIC_4IN_TO_8IN` | real · Measure (Sum) | 93 | yes | 0 | Miles of mainline made of plastic of any type, 4 to 8 inches. *(not used in any sheet)* |
| **Mmiles Plastic 8In To 12In**<br>`MMILES_PLASTIC_8IN_TO_12IN` | real · Measure (Sum) | 1 | yes | 0 | Miles of mainline made of plastic of any type, 8 to 12 inches. *(not used in any sheet)* |
| **Mmiles Plastic Gt12In**<br>`MMILES_PLASTIC_GT12IN` | real · Measure (Sum) | 1 | yes | 0 | Miles of mainline made of plastic of any type, over 12 inches. *(not used in any sheet)* |
| **Mmiles Plastic Lt2In**<br>`MMILES_PLASTIC_LT2IN` | real · Measure (Sum) | 309 | yes | 0 | Miles of mainline made of plastic of any type, under 2 inches. *(not used in any sheet)* |
| **Mmiles Plastic Unk**<br>`MMILES_PLASTIC_UNK` | real · Measure (Sum) | 44 | yes | 0 | Miles of mainline made of plastic of any type, diameter not recorded. *(not used in any sheet)* |
| **Mmiles Rci 2In To 4In**<br>`MMILES_RCI_2IN_TO_4IN` | real · Measure (Sum) | 5 | yes | 0 | Miles of mainline made of reconditioned cast iron, 2 to 4 inches. *(not used in any sheet)* |
| **Mmiles Rci 4In To 8In**<br>`MMILES_RCI_4IN_TO_8IN` | real · Measure (Sum) | 5 | yes | 0 | Miles of mainline made of reconditioned cast iron, 4 to 8 inches. *(not used in any sheet)* |
| **Mmiles Rci 8In To 12In**<br>`MMILES_RCI_8IN_TO_12IN` | real · Measure (Sum) | 11 | yes | 0 | Miles of mainline made of reconditioned cast iron, 8 to 12 inches. *(not used in any sheet)* |
| **Mmiles Rci Gt12In**<br>`MMILES_RCI_GT12IN` | real · Measure (Sum) | 17 | yes | 0 | Miles of mainline made of reconditioned cast iron, over 12 inches. *(not used in any sheet)* |
| **Mmiles Rci Lt2In**<br>`MMILES_RCI_LT2IN` | real · Measure (Sum) | 2 | yes | 0 | Miles of mainline made of reconditioned cast iron, under 2 inches. *(not used in any sheet)* |
| **Mmiles Rci Unk**<br>`MMILES_RCI_UNK` | real · Measure (Sum) | 2 | yes | 0 | Miles of mainline made of reconditioned cast iron, diameter not recorded. *(not used in any sheet)* |
| **Mmiles Steel 2In To 4In**<br>`MMILES_STEEL_2IN_TO_4IN` | real · Measure (Sum) | 2,011 | yes | 0 | Miles of mainline made of steel, 2 to 4 inches. *(not used in any sheet)* |
| **Mmiles Steel 4In To 8In**<br>`MMILES_STEEL_4IN_TO_8IN` | real · Measure (Sum) | 1,809 | yes | 0 | Miles of mainline made of steel, 4 to 8 inches. *(not used in any sheet)* |
| **Mmiles Steel 8In To 12In**<br>`MMILES_STEEL_8IN_TO_12IN` | real · Measure (Sum) | 1,145 | yes | 0 | Miles of mainline made of steel, 8 to 12 inches. *(not used in any sheet)* |
| **Mmiles Steel Gt12In**<br>`MMILES_STEEL_GT12IN` | real · Measure (Sum) | 932 | yes | 0 | Miles of mainline made of steel, over 12 inches. *(not used in any sheet)* |
| **Mmiles Steel Lt2In**<br>`MMILES_STEEL_LT2IN` | real · Measure (Sum) | 2,383 | yes | 0 | Miles of mainline made of steel, under 2 inches. *(not used in any sheet)* |
| **Mmiles Steel Unk**<br>`MMILES_STEEL_UNK` | real · Measure (Sum) | 389 | yes | 0 | Miles of mainline made of steel, diameter not recorded. *(not used in any sheet)* |

#### Mainline mileage by material

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Miles of Cast Iron**<br>`MMILES_CI` | real · Measure (Sum) | 841 | yes | 7 | Miles of mainline made of cast iron, as carried on the report's summary line. |
| **Miles of Reconditioned Cast Iron**<br>`MMILES_RCI` | real · Measure (Sum) | 32 | yes | 7 | Miles of mainline made of reconditioned cast iron, as carried on the report's summary line. |
| **Miles of Steel Protected Bare**<br>`MMILES_STEEL_CP_BARE` | real · Measure (Sum) | 873 | yes | 7 | Miles of mainline made of cathodically protected bare steel, as carried on the report's summary line. |
| **Miles of Steel Unprotected Bare**<br>`MMILES_STEEL_UNP_BARE` | real · Measure (Sum) | 1,509 | yes | 7 | Miles of mainline made of unprotected bare steel, as carried on the report's summary line. |
| **Mmiles Abs Total**<br>`MMILES_ABS_TOTAL` | real · Measure (Sum) | 113 | yes | 7 | Miles of mainline made of ABS plastic, all diameters. |
| **Mmiles Plastic Total**<br>`MMILES_PLASTIC_TOTAL` | real · Measure (Sum) | 318 | yes | 7 | Miles of mainline made of plastic of any type, all diameters. |
| **Miles of Steel Unprotected Coated**<br>`MMILES_STEEL_UNP_COATED` | real · Measure (Sum) | 1,109 | yes | 2 | Miles of mainline made of unprotected coated steel, as carried on the report's summary line. |
| **Miles of Copper**<br>`MMILES_CU` | real · Measure (Sum) | 32 | yes | 1 | Miles of mainline made of copper, as carried on the report's summary line. |
| **Miles of Ductile Iron**<br>`MMILES_DI` | real · Measure (Sum) | 357 | yes | 1 | Miles of mainline made of ductile iron, as carried on the report's summary line. |
| **Miles of Other**<br>`MMILES_OTHER` | real · Measure (Sum) | 219 | yes | 1 | Miles of mainline made of other materials, as carried on the report's summary line. |
| **Miles of Steel Protected Coated**<br>`MMILES_STEEL_CP_COATED` | real · Measure (Sum) | 2,724 | yes | 1 | Miles of mainline made of cathodically protected coated steel, as carried on the report's summary line. |
| **Mmiles Pe Total**<br>`MMILES_PE_TOTAL` | real · Measure (Sum) | 4,050 | yes | 1 | Miles of mainline made of polyethylene plastic, all diameters. |
| **Miles of Plastic**<br>`MMILES_PLASTIC` | real · Measure (Sum) | 3,918 | yes | 0 | Miles of mainline made of plastic of any type, as carried on the report's summary line. *(not used in any sheet)* |
| **Mmiles Ci Wr Total**<br>`MMILES_CI_WR_TOTAL` | real · Measure (Sum) | 927 | yes | 0 | Miles of mainline made of cast or wrought iron, all diameters. *(not used in any sheet)* |
| **Mmiles Cu Total**<br>`MMILES_CU_TOTAL` | real · Measure (Sum) | 32 | yes | 0 | Miles of mainline made of copper, all diameters. *(not used in any sheet)* |
| **Mmiles Di Total**<br>`MMILES_DI_TOTAL` | real · Measure (Sum) | 386 | yes | 0 | Miles of mainline made of ductile iron, all diameters. *(not used in any sheet)* |
| **Mmiles Oth Plstc Total**<br>`MMILES_OTH_PLSTC_TOTAL` | real · Measure (Sum) | 72 | yes | 0 | Miles of mainline made of other or unspecified plastic, all diameters. *(not used in any sheet)* |
| **Mmiles Other Material Detail**<br>`MMILES_OTHER_MATERIAL_DETAIL` | string · Dimension | 36 | yes | 0 | Free-text description of what the operator counted as `other` mainline material. *(not used in any sheet)* |
| **Mmiles Other Total**<br>`MMILES_OTHER_TOTAL` | real · Measure (Sum) | 275 | yes | 0 | Miles of mainline made of other materials, all diameters. *(not used in any sheet)* |
| **Mmiles Rci Total**<br>`MMILES_RCI_TOTAL` | real · Measure (Sum) | 32 | yes | 0 | Miles of mainline made of reconditioned cast iron, all diameters. *(not used in any sheet)* |
| **Mmiles Steel Total**<br>`MMILES_STEEL_TOTAL` | real · Measure (Sum) | 2,860 | yes | 0 | Miles of mainline made of steel, all diameters. *(not used in any sheet)* |

#### Leaks

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Known Leaks**<br>`KNOWN_LEAKS` | real · Measure (Sum) | 639 | yes | 1 | Known system leaks scheduled for repair but still open at year end. |
| **Fed Land Leaks Repaired**<br>`FED_LAND_LEAKS_REPAIRED` | real · Measure (Sum) | 54 | yes | 0 | Leaks repaired on federal land. *(not used in any sheet)* |
| **Mechanical Joint Leaks**<br>`MECHANICAL_JOINT_LEAKS` | real · Measure (Sum) | 104 | yes | 0 | Leaks occurring at mechanical joints. *(not used in any sheet)* |

#### Service counts by material and diameter

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Num Srvs Abs 1In To 2In**<br>`NUM_SRVS_ABS_1IN_TO_2IN` | real · Measure (Sum) | 18 | yes | 0 | Count of service lines made of ABS plastic, 1 to 2 inches. *(not used in any sheet)* |
| **Num Srvs Abs 2In To 4In**<br>`NUM_SRVS_ABS_2IN_TO_4IN` | real · Measure (Sum) | 1 | yes | 0 | Count of service lines made of ABS plastic, 2 to 4 inches. *(not used in any sheet)* |
| **Num Srvs Abs 4In To 8In**<br>`NUM_SRVS_ABS_4IN_TO_8IN` | real · Measure (Sum) | 5 | yes | 0 | Count of service lines made of ABS plastic, 4 to 8 inches. *(not used in any sheet)* |
| **Num Srvs Abs Gt8In**<br>`NUM_SRVS_ABS_GT8IN` | real · Measure (Sum) | 1 | yes | 0 | Count of service lines made of ABS plastic, over 8 inches. *(not used in any sheet)* |
| **Num Srvs Abs Lt1In**<br>`NUM_SRVS_ABS_LT1IN` | real · Measure (Sum) | 151 | yes | 0 | Count of service lines made of ABS plastic, under 1 inch. *(not used in any sheet)* |
| **Num Srvs Abs Unk**<br>`NUM_SRVS_ABS_UNK` | real · Measure (Sum) | 5 | yes | 0 | Count of service lines made of ABS plastic, diameter not recorded. *(not used in any sheet)* |
| **Num Srvs Ci Wr 1In To 2In**<br>`NUM_SRVS_CI_WR_1IN_TO_2IN` | real · Measure (Sum) | 450 | yes | 0 | Count of service lines made of cast or wrought iron, 1 to 2 inches. *(not used in any sheet)* |
| **Num Srvs Ci Wr 2In To 4In**<br>`NUM_SRVS_CI_WR_2IN_TO_4IN` | real · Measure (Sum) | 200 | yes | 0 | Count of service lines made of cast or wrought iron, 2 to 4 inches. *(not used in any sheet)* |
| **Num Srvs Ci Wr 4In To 8In**<br>`NUM_SRVS_CI_WR_4IN_TO_8IN` | real · Measure (Sum) | 46 | yes | 0 | Count of service lines made of cast or wrought iron, 4 to 8 inches. *(not used in any sheet)* |
| **Num Srvs Ci Wr Gt8In**<br>`NUM_SRVS_CI_WR_GT8IN` | real · Measure (Sum) | 44 | yes | 0 | Count of service lines made of cast or wrought iron, over 8 inches. *(not used in any sheet)* |
| **Num Srvs Ci Wr Lt1In**<br>`NUM_SRVS_CI_WR_LT1IN` | real · Measure (Sum) | 172 | yes | 0 | Count of service lines made of cast or wrought iron, under 1 inch. *(not used in any sheet)* |
| **Num Srvs Ci Wr Unk**<br>`NUM_SRVS_CI_WR_UNK` | real · Measure (Sum) | 5 | yes | 0 | Count of service lines made of cast or wrought iron, diameter not recorded. *(not used in any sheet)* |
| **Num Srvs Cu 1In To 2In**<br>`NUM_SRVS_CU_1IN_TO_2IN` | real · Measure (Sum) | 301 | yes | 0 | Count of service lines made of copper, 1 to 2 inches. *(not used in any sheet)* |
| **Num Srvs Cu 2In To 4In**<br>`NUM_SRVS_CU_2IN_TO_4IN` | real · Measure (Sum) | 11 | yes | 0 | Count of service lines made of copper, 2 to 4 inches. *(not used in any sheet)* |
| **Num Srvs Cu 4In To 8In**<br>`NUM_SRVS_CU_4IN_TO_8IN` | real · Measure (Sum) | 2 | yes | 0 | Count of service lines made of copper, 4 to 8 inches. *(not used in any sheet)* |
| **Num Srvs Cu Gt8In**<br>`NUM_SRVS_CU_GT8IN` | real · Measure (Sum) | 1 | yes | 0 | Count of service lines made of copper, over 8 inches. *(not used in any sheet)* |
| **Num Srvs Cu Lt1In**<br>`NUM_SRVS_CU_LT1IN` | real · Measure (Sum) | 924 | yes | 0 | Count of service lines made of copper, under 1 inch. *(not used in any sheet)* |
| **Num Srvs Cu Unk**<br>`NUM_SRVS_CU_UNK` | real · Measure (Sum) | 11 | yes | 0 | Count of service lines made of copper, diameter not recorded. *(not used in any sheet)* |
| **Num Srvs Di 1In To 2In**<br>`NUM_SRVS_DI_1IN_TO_2IN` | real · Measure (Sum) | 2 | yes | 0 | Count of service lines made of ductile iron, 1 to 2 inches. *(not used in any sheet)* |
| **Num Srvs Di 2In To 4In**<br>`NUM_SRVS_DI_2IN_TO_4IN` | real · Measure (Sum) | 2 | yes | 0 | Count of service lines made of ductile iron, 2 to 4 inches. *(not used in any sheet)* |
| **Num Srvs Di 4In To 8In**<br>`NUM_SRVS_DI_4IN_TO_8IN` | real · Measure (Sum) | 11 | yes | 0 | Count of service lines made of ductile iron, 4 to 8 inches. *(not used in any sheet)* |
| **Num Srvs Di Gt8In**<br>`NUM_SRVS_DI_GT8IN` | real · Measure (Sum) | 18 | yes | 0 | Count of service lines made of ductile iron, over 8 inches. *(not used in any sheet)* |
| **Num Srvs Di Lt1In**<br>`NUM_SRVS_DI_LT1IN` | real · Measure (Sum) | 1 | yes | 0 | Count of service lines made of ductile iron, under 1 inch. *(not used in any sheet)* |
| **Num Srvs Di Unk**<br>`NUM_SRVS_DI_UNK` | real · Measure (Sum) | 1 | yes | 0 | Count of service lines made of ductile iron, diameter not recorded. *(not used in any sheet)* |
| **Num Srvs Oth Plstc 1In To 2In**<br>`NUM_SRVS_OTH_PLSTC_1IN_TO_2IN` | real · Measure (Sum) | 72 | yes | 0 | Count of service lines made of other or unspecified plastic, 1 to 2 inches. *(not used in any sheet)* |
| **Num Srvs Oth Plstc 2In To 4In**<br>`NUM_SRVS_OTH_PLSTC_2IN_TO_4IN` | real · Measure (Sum) | 32 | yes | 0 | Count of service lines made of other or unspecified plastic, 2 to 4 inches. *(not used in any sheet)* |
| **Num Srvs Oth Plstc 4In To 8In**<br>`NUM_SRVS_OTH_PLSTC_4IN_TO_8IN` | real · Measure (Sum) | 17 | yes | 0 | Count of service lines made of other or unspecified plastic, 4 to 8 inches. *(not used in any sheet)* |
| **Num Srvs Oth Plstc Gt8In**<br>`NUM_SRVS_OTH_PLSTC_GT8IN` | real · Measure (Sum) | 2 | yes | 0 | Count of service lines made of other or unspecified plastic, over 8 inches. *(not used in any sheet)* |
| **Num Srvs Oth Plstc Lt1In**<br>`NUM_SRVS_OTH_PLSTC_LT1IN` | real · Measure (Sum) | 113 | yes | 0 | Count of service lines made of other or unspecified plastic, under 1 inch. *(not used in any sheet)* |
| **Num Srvs Oth Plstc Unk**<br>`NUM_SRVS_OTH_PLSTC_UNK` | real · Measure (Sum) | 32 | yes | 0 | Count of service lines made of other or unspecified plastic, diameter not recorded. *(not used in any sheet)* |
| **Num Srvs Other 1In To 2In**<br>`NUM_SRVS_OTHER_1IN_TO_2IN` | real · Measure (Sum) | 400 | yes | 0 | Count of service lines made of other materials, 1 to 2 inches. *(not used in any sheet)* |
| **Num Srvs Other 2In To 4In**<br>`NUM_SRVS_OTHER_2IN_TO_4IN` | real · Measure (Sum) | 53 | yes | 0 | Count of service lines made of other materials, 2 to 4 inches. *(not used in any sheet)* |
| **Num Srvs Other 4In To 8In**<br>`NUM_SRVS_OTHER_4IN_TO_8IN` | real · Measure (Sum) | 12 | yes | 0 | Count of service lines made of other materials, 4 to 8 inches. *(not used in any sheet)* |
| **Num Srvs Other Gt8In**<br>`NUM_SRVS_OTHER_GT8IN` | real · Measure (Sum) | 2 | yes | 0 | Count of service lines made of other materials, over 8 inches. *(not used in any sheet)* |
| **Num Srvs Other Lt1In**<br>`NUM_SRVS_OTHER_LT1IN` | real · Measure (Sum) | 557 | yes | 0 | Count of service lines made of other materials, under 1 inch. *(not used in any sheet)* |
| **Num Srvs Other Unk**<br>`NUM_SRVS_OTHER_UNK` | real · Measure (Sum) | 1,384 | yes | 0 | Count of service lines made of other materials, diameter not recorded. *(not used in any sheet)* |
| **Num Srvs Pe 1In To 2In**<br>`NUM_SRVS_PE_1IN_TO_2IN` | real · Measure (Sum) | 1,471 | yes | 0 | Count of service lines made of polyethylene plastic, 1 to 2 inches. *(not used in any sheet)* |
| **Num Srvs Pe 2In To 4In**<br>`NUM_SRVS_PE_2IN_TO_4IN` | real · Measure (Sum) | 597 | yes | 0 | Count of service lines made of polyethylene plastic, 2 to 4 inches. *(not used in any sheet)* |
| **Num Srvs Pe 4In To 8In**<br>`NUM_SRVS_PE_4IN_TO_8IN` | real · Measure (Sum) | 261 | yes | 0 | Count of service lines made of polyethylene plastic, 4 to 8 inches. *(not used in any sheet)* |
| **Num Srvs Pe Gt8In**<br>`NUM_SRVS_PE_GT8IN` | real · Measure (Sum) | 53 | yes | 0 | Count of service lines made of polyethylene plastic, over 8 inches. *(not used in any sheet)* |
| **Num Srvs Pe Lt1In**<br>`NUM_SRVS_PE_LT1IN` | real · Measure (Sum) | 4,677 | yes | 0 | Count of service lines made of polyethylene plastic, under 1 inch. *(not used in any sheet)* |
| **Num Srvs Pe Unk**<br>`NUM_SRVS_PE_UNK` | real · Measure (Sum) | 578 | yes | 0 | Count of service lines made of polyethylene plastic, diameter not recorded. *(not used in any sheet)* |
| **Num Srvs Plastic 1In To 2In**<br>`NUM_SRVS_PLASTIC_1IN_TO_2IN` | real · Measure (Sum) | 193 | yes | 0 | Count of service lines made of plastic of any type, 1 to 2 inches. *(not used in any sheet)* |
| **Num Srvs Plastic 2In To 4In**<br>`NUM_SRVS_PLASTIC_2IN_TO_4IN` | real · Measure (Sum) | 18 | yes | 0 | Count of service lines made of plastic of any type, 2 to 4 inches. *(not used in any sheet)* |
| **Num Srvs Plastic 4In To 8In**<br>`NUM_SRVS_PLASTIC_4IN_TO_8IN` | real · Measure (Sum) | 3 | yes | 0 | Count of service lines made of plastic of any type, 4 to 8 inches. *(not used in any sheet)* |
| **Num Srvs Plastic Gt8In**<br>`NUM_SRVS_PLASTIC_GT8IN` | real · Measure (Sum) | 1 | yes | 0 | Count of service lines made of plastic of any type, over 8 inches. *(not used in any sheet)* |
| **Num Srvs Plastic Lt1In**<br>`NUM_SRVS_PLASTIC_LT1IN` | real · Measure (Sum) | 672 | yes | 0 | Count of service lines made of plastic of any type, under 1 inch. *(not used in any sheet)* |
| **Num Srvs Plastic Unk**<br>`NUM_SRVS_PLASTIC_UNK` | real · Measure (Sum) | 72 | yes | 0 | Count of service lines made of plastic of any type, diameter not recorded. *(not used in any sheet)* |
| **Num Srvs Rci 1In To 2In**<br>`NUM_SRVS_RCI_1IN_TO_2IN` | real · Measure (Sum) | 2 | yes | 0 | Count of service lines made of reconditioned cast iron, 1 to 2 inches. *(not used in any sheet)* |
| **Num Srvs Rci 2In To 4In**<br>`NUM_SRVS_RCI_2IN_TO_4IN` | real · Measure (Sum) | 2 | yes | 0 | Count of service lines made of reconditioned cast iron, 2 to 4 inches. *(not used in any sheet)* |
| **Num Srvs Rci 4In To 8In**<br>`NUM_SRVS_RCI_4IN_TO_8IN` | real · Measure (Sum) | 2 | yes | 0 | Count of service lines made of reconditioned cast iron, 4 to 8 inches. *(not used in any sheet)* |
| **Num Srvs Rci Gt8In**<br>`NUM_SRVS_RCI_GT8IN` | real · Measure (Sum) | 2 | yes | 0 | Count of service lines made of reconditioned cast iron, over 8 inches. *(not used in any sheet)* |
| **Num Srvs Rci Lt1In**<br>`NUM_SRVS_RCI_LT1IN` | real · Measure (Sum) | 2 | yes | 0 | Count of service lines made of reconditioned cast iron, under 1 inch. *(not used in any sheet)* |
| **Num Srvs Rci Unk**<br>`NUM_SRVS_RCI_UNK` | real · Measure (Sum) | 2 | yes | 0 | Count of service lines made of reconditioned cast iron, diameter not recorded. *(not used in any sheet)* |
| **Num Srvs Steel 1In To 2In**<br>`NUM_SRVS_STEEL_1IN_TO_2IN` | real · Measure (Sum) | 1,554 | yes | 0 | Count of service lines made of steel, 1 to 2 inches. *(not used in any sheet)* |
| **Num Srvs Steel 2In To 4In**<br>`NUM_SRVS_STEEL_2IN_TO_4IN` | real · Measure (Sum) | 682 | yes | 0 | Count of service lines made of steel, 2 to 4 inches. *(not used in any sheet)* |
| **Num Srvs Steel 4In To 8In**<br>`NUM_SRVS_STEEL_4IN_TO_8IN` | real · Measure (Sum) | 221 | yes | 0 | Count of service lines made of steel, 4 to 8 inches. *(not used in any sheet)* |
| **Num Srvs Steel Gt8In**<br>`NUM_SRVS_STEEL_GT8IN` | real · Measure (Sum) | 49 | yes | 0 | Count of service lines made of steel, over 8 inches. *(not used in any sheet)* |
| **Num Srvs Steel Lt1In**<br>`NUM_SRVS_STEEL_LT1IN` | real · Measure (Sum) | 3,870 | yes | 0 | Count of service lines made of steel, under 1 inch. *(not used in any sheet)* |
| **Num Srvs Steel Unk**<br>`NUM_SRVS_STEEL_UNK` | real · Measure (Sum) | 568 | yes | 0 | Count of service lines made of steel, diameter not recorded. *(not used in any sheet)* |

#### Service counts by install decade

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Num Srvs By Dcd 1940 To 1949**<br>`NUM_SRVS_BY_DCD_1940_TO_1949` | real · Measure (Sum) | 1,484 | yes | 0 | Service lines installed 1940–1949. *(not used in any sheet)* |
| **Num Srvs By Dcd 1950 To 1959**<br>`NUM_SRVS_BY_DCD_1950_TO_1959` | real · Measure (Sum) | 2,274 | yes | 0 | Service lines installed 1950–1959. *(not used in any sheet)* |
| **Num Srvs By Dcd 1960 To 1969**<br>`NUM_SRVS_BY_DCD_1960_TO_1969` | real · Measure (Sum) | 2,235 | yes | 0 | Service lines installed 1960–1969. *(not used in any sheet)* |
| **Num Srvs By Dcd 1970 To 1979**<br>`NUM_SRVS_BY_DCD_1970_TO_1979` | real · Measure (Sum) | 1,852 | yes | 0 | Service lines installed 1970–1979. *(not used in any sheet)* |
| **Num Srvs By Dcd 1980 To 1989**<br>`NUM_SRVS_BY_DCD_1980_TO_1989` | real · Measure (Sum) | 2,209 | yes | 0 | Service lines installed 1980–1989. *(not used in any sheet)* |
| **Num Srvs By Dcd 1990 To 1999**<br>`NUM_SRVS_BY_DCD_1990_TO_1999` | real · Measure (Sum) | 2,309 | yes | 0 | Service lines installed 1990–1999. *(not used in any sheet)* |
| **Num Srvs By Dcd 2000 To 2009**<br>`NUM_SRVS_BY_DCD_2000_TO_2009` | real · Measure (Sum) | 2,406 | yes | 0 | Service lines installed 2000–2009. *(not used in any sheet)* |
| **Num Srvs By Dcd 2010 To 2019**<br>`NUM_SRVS_BY_DCD_2010_TO_2019` | real · Measure (Sum) | 1,847 | yes | 0 | Service lines installed 2010–2019. *(not used in any sheet)* |
| **Num Srvs By Dcd 2020 To 2029**<br>`NUM_SRVS_BY_DCD_2020_TO_2029` | real · Measure (Sum) | 1,013 | yes | 0 | Service lines installed 2020–2029. *(not used in any sheet)* |
| **Num Srvs By Dcd Pre1940**<br>`NUM_SRVS_BY_DCD_PRE1940` | real · Measure (Sum) | 1,895 | yes | 0 | Service lines installed before 1940. *(not used in any sheet)* |
| **Num Srvs By Dcd Total**<br>`NUM_SRVS_BY_DCD_TOTAL` | real · Measure (Sum) | 5,935 | yes | 0 | Service lines installed any period. *(not used in any sheet)* |
| **Num Srvs By Dcd Unk**<br>`NUM_SRVS_BY_DCD_UNK` | real · Measure (Sum) | 2,780 | yes | 0 | Service lines installed an unrecorded decade. *(not used in any sheet)* |

#### Operator address

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Hq Address City**<br>`HQ_ADDRESS_CITY` | string · Dimension | 2,339 | yes | 0 | Headquarters city. Administrative; not used analytically. *(not used in any sheet)* |
| **Hq Address County**<br>`HQ_ADDRESS_COUNTY` | string · Dimension | 1,239 | yes | 0 | Headquarters county. Administrative; not used analytically. *(not used in any sheet)* |
| **Hq Address State**<br>`HQ_ADDRESS_STATE` | string · Dimension | 83 | yes | 0 | Headquarters state. Administrative; not used analytically. *(not used in any sheet)* |
| **Hq Address Street**<br>`HQ_ADDRESS_STREET` | string · Dimension | 3,582 | yes | 0 | Headquarters street. Administrative; not used analytically. *(not used in any sheet)* |
| **Hq Address Zip**<br>`HQ_ADDRESS_ZIP` | string · Dimension | 2,727 | yes | 0 | Headquarters postal code. Administrative; not used analytically. *(not used in any sheet)* |
| **Office Address City**<br>`OFFICE_ADDRESS_CITY` | string · Dimension | 3,597 | yes | 0 | Operating office city. Administrative; not used analytically. *(not used in any sheet)* |
| **Office Address County**<br>`OFFICE_ADDRESS_COUNTY` | string · Dimension | 2,158 | yes | 0 | Operating office county. Administrative; not used analytically. *(not used in any sheet)* |
| **Office Address State**<br>`OFFICE_ADDRESS_STATE` | string · Dimension | 62 | yes | 0 | Operating office state. Administrative; not used analytically. *(not used in any sheet)* |
| **Office Address Street**<br>`OFFICE_ADDRESS_STREET` | string · Dimension | 5,753 | yes | 0 | Operating office street. Administrative; not used analytically. *(not used in any sheet)* |
| **Office Address Zip**<br>`OFFICE_ADDRESS_ZIP` | string · Dimension | 3,123 | yes | 0 | Operating office postal code. Administrative; not used analytically. *(not used in any sheet)* |

#### Mainline mileage by diameter

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Mmiles 2In To 4In Total**<br>`MMILES_2IN_TO_4IN_TOTAL` | real · Measure (Sum) | 2,663 | yes | 0 | Miles of mainline of every material, 2 to 4 inches. *(not used in any sheet)* |
| **Mmiles 4In To 8In Total**<br>`MMILES_4IN_TO_8IN_TOTAL` | real · Measure (Sum) | 1,897 | yes | 0 | Miles of mainline of every material, 4 to 8 inches. *(not used in any sheet)* |
| **Mmiles 8In To 12In Total**<br>`MMILES_8IN_TO_12IN_TOTAL` | real · Measure (Sum) | 1,085 | yes | 0 | Miles of mainline of every material, 8 to 12 inches. *(not used in any sheet)* |
| **Mmiles Gt12In Total**<br>`MMILES_GT12IN_TOTAL` | real · Measure (Sum) | 643 | yes | 0 | Miles of mainline of every material, over 12 inches. *(not used in any sheet)* |
| **Mmiles Lt2In Total**<br>`MMILES_LT2IN_TOTAL` | real · Measure (Sum) | 3,282 | yes | 0 | Miles of mainline of every material, under 2 inches. *(not used in any sheet)* |
| **Mmiles Unk Total**<br>`MMILES_UNK_TOTAL` | real · Measure (Sum) | 364 | yes | 0 | Miles of mainline of every material, diameter not recorded. *(not used in any sheet)* |

#### Service counts by diameter

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Num Srvs 1In To 2In Total**<br>`NUM_SRVS_1IN_TO_2IN_TOTAL` | real · Measure (Sum) | 1,276 | yes | 0 | Count of service lines of every material, 1 to 2 inches. *(not used in any sheet)* |
| **Num Srvs 2In To 4In Total**<br>`NUM_SRVS_2IN_TO_4IN_TOTAL` | real · Measure (Sum) | 669 | yes | 0 | Count of service lines of every material, 2 to 4 inches. *(not used in any sheet)* |
| **Num Srvs 4In To 8In Total**<br>`NUM_SRVS_4IN_TO_8IN_TOTAL` | real · Measure (Sum) | 274 | yes | 0 | Count of service lines of every material, 4 to 8 inches. *(not used in any sheet)* |
| **Num Srvs Gt8In Total**<br>`NUM_SRVS_GT8IN_TOTAL` | real · Measure (Sum) | 81 | yes | 0 | Count of service lines of every material, over 8 inches. *(not used in any sheet)* |
| **Num Srvs Lt1In Total**<br>`NUM_SRVS_LT1IN_TOTAL` | real · Measure (Sum) | 5,415 | yes | 0 | Count of service lines of every material, under 1 inch. *(not used in any sheet)* |
| **Num Srvs Unk Total**<br>`NUM_SRVS_UNK_TOTAL` | real · Measure (Sum) | 924 | yes | 0 | Count of service lines of every material, diameter not recorded. *(not used in any sheet)* |

#### Filing contact

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Preparers Email**<br>`PREPARERS_EMAIL` | string · Dimension | 4,762 | yes | 0 | The email address of the person who prepared the filing. Administrative; carries no analytical value. *(not used in any sheet)* |
| **Preparers Fax**<br>`PREPARERS_FAX` | string · Dimension | 3,608 | yes | 0 | The fax number of the person who prepared the filing. Administrative; carries no analytical value. *(not used in any sheet)* |
| **Preparers Name**<br>`PREPARERS_NAME` | string · Dimension | 4,813 | yes | 0 | The name of the person who prepared the filing. Administrative; carries no analytical value. *(not used in any sheet)* |
| **Preparers Phone**<br>`PREPARERS_PHONE` | string · Dimension | 4,450 | yes | 0 | The telephone number of the person who prepared the filing. Administrative; carries no analytical value. *(not used in any sheet)* |
| **Preparers Title**<br>`PREPARERS_TITLE` | string · Dimension | 1,098 | yes | 0 | The job title of the person who prepared the filing. Administrative; carries no analytical value. *(not used in any sheet)* |

#### Operator

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Commodity**<br>`COMMODITY` | string · Dimension | 5 | yes | 0 | Commodity distributed — natural gas throughout. *(not used in any sheet)* |
| **Operator Id**<br>`OPERATOR_ID` | real · Dimension | 3,093 | yes | 0 | PHMSA operator identifier. *(not used in any sheet)* |
| **Operator Name**<br>`Operator Name (Group)` | string · Dimension | — | — | 0 | Author-defined grouping built on **Operator Name**. Operator's registered name. *(not used in any sheet)* |
| **Operator Name (dont use)**<br>`OPERATOR_NAME` | string · Dimension | 3,917 | yes | 0 | Operator's registered name. *(not used in any sheet)* |
| **Operator Type**<br>`OPERATOR_TYPE` | string · Dimension | 5 | yes | 0 | Ownership type — investor-owned, municipal, cooperative and so on. *(not used in any sheet)* |

#### Safety devices

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **EFV In System**<br>`EFV_IN_SYSTEM` | real · Measure (Sum) | 2,046 | yes | 0 | Excess flow valves present in the system. *(not used in any sheet)* |
| **EFV Installed CY**<br>`EFV_INSTALLED_CY` | real · Measure (Sum) | 1,056 | yes | 0 | Excess flow valves installed during the calendar year. *(not used in any sheet)* |
| **Shutoff Valve In System**<br>`SHUTOFF_VALVE_IN_SYSTEM` | integer · Measure (Sum) | 759 | yes | 0 | Manual service shutoff valves present in the system. *(not used in any sheet)* |
| **Shutoff Valve Installed CY**<br>`SHUTOFF_VALVE_INSTALLED_CY` | integer · Measure (Sum) | 237 | yes | 0 | Manual service shutoff valves installed during the calendar year. *(not used in any sheet)* |

#### System characteristics

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Average Service Length ft**<br>`AVERAGE_LENGTH` | real · Measure (Sum) | 609 | yes | 0 | Average service-line length in feet. *(not used in any sheet)* |
| **Percent Unacc Gas**<br>`PERCENT_UNACC_GAS` | real · Measure (Sum) | 1,285 | yes | 0 | Percentage of gas unaccounted for: the gap between gas bought and gas delivered. *(not used in any sheet)* |

### Calculated fields in use

129 of the workbook's 184 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| **% of System Industry Targeted Mileage**<br>`Calculation_1714464112009760779` | real · Measure | Basic | Total Industry Targeted Material, Total Miles of Mains | — | 2 |
| **% of System Pre-1970 Main Mileage (Including Unknown)**<br>`Calculation_1714464112006627332` | real · Measure | Basic | Pre-1970 Main Mileage (Including Unknown), Mmiles By Dcd Total | — | 10 |
| **% of System Steel Service Lines**<br>`Calculation_1714464112007864328` | real · Measure | Basic | Num Srvs Steel Cp Bare, Num Srvs Steel Cp Coated, Num Srvs Steel Unp Bare, Num Srvs Steel Unp Coated, Number of Services Total | — | 3 |
| ***Material**<br>`Leaks per Unit (copy)_1133781216286425103` | real · Measure | Basic | Pre-1970 Main Mileage (Including Unknown), Total Industry Targeted Material, Steel Service Lines | *Material % | 3 |
| ***Material %**<br>`*Material (copy)_1979332047057956868` | real · Measure | Basic | *Material, *Material_TotalInventory | — | 3 |
| ***Material Description**<br>`*Material (copy)_1133781216292053011` | string · Dimension | Basic | — | — | 3 |
| ***Material Description Type**<br>`*Material Description (copy)_1979332046988898305` | string · Dimension | Basic | — | — | 1 |
| ***Material Units**<br>`*Material Description (copy)_1979332046986534912` | string · Dimension | Basic | — | — | 5 |
| ***Material_TotalInventory**<br>`*Material (copy)_1979332047054708738` | real · Measure | Basic | Total Miles of Mains, Number of Services Total | *Material % | 3 |
| **@UnitedStates**<br>`Calculation_1267200346051051520` | string · Dimension | Basic | — | — | 3 |
| **All Main Hazleaks**<br>`Calculation_653021999113138180` | real · Measure | Basic | Total Hazleaks Cor Mains, Total Hazleaks Eq Mains, Total Hazleaks Ex Mains, Total Hazleaks Mat Weld Mains, Total Hazleaks Nf Mains, Total Hazleaks Of Dam Mains … | Hazardous Leaks per Mile (Mains & Services), Hazardous Leaks per Mile (Mains & Services) Avg, Hazardous Leaks per Mile (Mains), Hazardous Leaks per Mile (Mains) Avg … | 22 |
| **All Main Leaks**<br>`Calculation_653021999108980737` | real · Measure | Basic | Main Corrosion Failure Leak, Main Equipment Failure Leak, Main Excavation Damage Leak, Main Pipe, Weld, or Joint Failure Leak, Main Natural Force Damage Leak, Main Other Outside Force Damage Leak … | Leak Type - % Haz, Leak Type - All, Leak Type - Main Count, Leaks per Mile (Mains) … | 22 |
| **All Main/Service Hazleaks**<br>`Calculation_653021999113551878` | real · Measure | Basic | Total Hazleaks Cor Srvs, Total Hazleaks Eq Srvs, Total Hazleaks Ex Srvs, Total Hazleaks Mat Weld Srvs, Total Hazleaks Nf Srvs, Total Hazleaks Of Dam Srvs … | Leak Type - % Haz, Leak Type - All, Leaks per Unit - Leaks | 22 |
| **All Main/Service Leaks**<br>`Calculation_653021999112105987` | real · Measure | Basic | Service Corrosion Failure Leak, Service Equipment Failure Leak, Service Excavation Damage Leak, Service Pipe, Weld, or Joint Failure Leak, Service Natural Force Damage Leak, Service Other Outside Force Damage Leak … | Leak Type - % Haz, Leak Type - All, Leaks per Mile (Mains & Services), Leaks per Mile (Mains & Services) Avg … | 22 |
| **All Service Hazleaks**<br>`Calculation_653021999113412613` | real · Measure | Basic | Total Hazleaks Cor Srvs, Total Hazleaks Eq Srvs, Total Hazleaks Ex Srvs, Total Hazleaks Mat Weld Srvs, Total Hazleaks Nf Srvs, Total Hazleaks Of Dam Srvs … | Hazardous Leaks per Mile (Mains & Services), Hazardous Leaks per Mile (Mains & Services) Avg, Hazardous Leaks per Service Count, Hazardous Leaks per Service Count Avg … | 22 |
| **All Service Leaks**<br>`Calculation_653021999109562370` | real · Measure | Basic | Service Corrosion Failure Leak, Service Equipment Failure Leak, Service Excavation Damage Leak, Service Pipe, Weld, or Joint Failure Leak, Service Natural Force Damage Leak, Service Other Outside Force Damage Leak … | Leak Type - % Haz, Leak Type - All, Leak Type - Service Count, Leaks per Service Count … | 22 |
| **Bare Steel**<br>`Calculation_1714464112008347657` | real · Measure | Basic | Miles of Steel Protected Bare, Miles of Steel Unprotected Bare | Total Industry Targeted Material | 7 |
| **Bare Steel**<br>`Calculation_2060396840461590529` | real · Measure | Basic | Num Srvs Steel Cp Bare, Num Srvs Steel Unp Bare | — | 1 |
| **Coated Steel**<br>`Calculation_1756403885944459264` | real · Measure | Basic | Miles of Steel Protected Coated, Miles of Steel Unprotected Coated | — | 1 |
| **Coated Steel**<br>`Calculation_2060396840461705218` | real · Measure | Basic | Num Srvs Steel Cp Coated, Miles of Steel Unprotected Coated | — | 1 |
| **Corrosion Leak**<br>`Main Corrosion Failure Leak (copy)_539024620999712770` | real · Measure | Basic | Main Corrosion Failure Leak, Service Corrosion Failure Leak | % of Corrosion Leaks, Corrosion Leaks Repaired per mile of main, Leak Type - Corrosion Failure | 5 |
| **Current DmgRate**<br>`Current Tickets (copy)_2060396840693760051` | real · Measure | Basic | Current ExcDmg, Current Tickets | Diff % DmgRate, Diff DmgRate | 1 |
| **Current ExcDmg**<br>`Current M_Inventory (copy)_2060396840691736617` | real · Dimension | LOD | Report Year, Excavation Damages | Current DmgRate, Diff % ExcDmg, Diff ExcDmg | 2 |
| **Current ITM**<br>`Current Pre-1970 (copy) (copy)_2060396840646299662` | real · Dimension | LOD | Report Year, Total Industry Targeted Material | Diff % ITM, Diff ITM | 1 |
| **Current M_Inventory**<br>`Current Pre-1970 (copy)_2060396840650248215` | real · Dimension | LOD | Report Year, Total Miles of Mains | Diff % M_Inventory, Diff M_Inventory | 1 |
| **Current Pre-1970**<br>`Max-Min Year Diff (copy)_2060396840636432388` | real · Dimension | LOD | Report Year, Pre-1970 Main Mileage (Including Unknown) | Diff % Pre-1970, Diff Pre-1970 | 1 |
| **Current S_Inventory**<br>`Current M_Inventory (copy)_2060396840651620376` | real · Dimension | LOD | Report Year, Number of Services Total | Diff % S_Inventory, Diff S_Inventory | 1 |
| **Current SSL**<br>`Current Pre-1970 (copy)_2060396840646275085` | real · Dimension | LOD | Report Year, Steel Service Lines | Diff % SSL, Diff SSL | 1 |
| **Current Tickets**<br>`Current ExcDmg (copy)_2060396840692125742` | real · Dimension | LOD | Report Year, Excavation Tickets | Current DmgRate, Diff % Tickets, Diff Tickets | 2 |
| **Damages per 1000 Excavation Tickets**<br>`Calculation_1243837974908375043` | real · Measure | Basic | Excavation Damages, Excavation Tickets | — | 5 |
| **Diff % DmgRate**<br>`Diff % Tickets (copy)_2060396840694267957` | real · Measure | Basic | Current DmgRate, Previous DmgRate | — | 1 |
| **Diff % ExcDmg**<br>`Diff % M_Inventory (copy)_2060396840691736618` | real · Measure | Basic | Current ExcDmg, Previous ExcDmg | — | 1 |
| **Diff % ITM**<br>`Diff % Pre-1970 (copy) (copy)_2060396840646299663` | real · Measure | Basic | Current ITM, Previous ITM | — | 1 |
| **Diff % M_Inventory**<br>`Diff % Pre-1970 (copy)_2060396840650248213` | real · Measure | Basic | Current M_Inventory, Previous M_Inventory | — | 1 |
| **Diff % Pre-1970**<br>`Diff Pre-1970 (copy)_2060396840641654791` | real · Measure | Basic | Current Pre-1970, Previous Pre-1970 | — | 1 |
| **Diff % S_Inventory**<br>`Diff % M_Inventory (copy)_2060396840651620377` | real · Measure | Basic | Current S_Inventory, Previous S_Inventory | — | 1 |
| **Diff % SSL**<br>`Diff % Pre-1970 (copy)_2060396840646275083` | real · Measure | Basic | Current SSL, Previous SSL | — | 1 |
| **Diff % Tickets**<br>`Diff % ExcDmg (copy)_2060396840692125743` | real · Measure | Basic | Current Tickets, Previous Tickets | — | 1 |
| **Diff DmgRate**<br>`Diff Tickets (copy)_2060396840694267958` | real · Measure | Basic | Current DmgRate, Previous DmgRate | Direction DmgRate | 1 |
| **Diff ExcDmg**<br>`Diff M_Inventory (copy)_2060396840691736619` | real · Measure | Basic | Current ExcDmg, Previous ExcDmg | Direction ExcDmg | 1 |
| **Diff ITM**<br>`Diff Pre-1970 (copy) (copy)_2060396840646299664` | real · Measure | Basic | Current ITM, Previous ITM | Direction ITM | 1 |
| **Diff M_Inventory**<br>`Diff Pre-1970 (copy)_2060396840650248212` | real · Measure | Basic | Current M_Inventory, Previous M_Inventory | Direction M_Inventory | 1 |
| **Diff Pre-1970**<br>`Current Pre-1970 (copy)_2060396840641429510` | real · Measure | Basic | Current Pre-1970, Previous Pre-1970 | Direction Pre-1970 | 1 |
| **Diff S_Inventory**<br>`Diff M_Inventory (copy)_2060396840651620378` | real · Measure | Basic | Current S_Inventory, Previous S_Inventory | Direction S_Inventory | 1 |
| **Diff SSL**<br>`Diff Pre-1970 (copy)_2060396840646275082` | real · Measure | Basic | Current SSL, Previous SSL | Direction SSL | 1 |
| **Diff Tickets**<br>`Diff ExcDmg (copy)_2060396840692125744` | real · Measure | Basic | Current Tickets, Previous Tickets | Direction Tickets | 1 |
| **Direction DmgRate**<br>`Direction Tickets (copy)_2060396840694267959` | string · Measure | Basic | Diff DmgRate | — | 1 |
| **Direction ExcDmg**<br>`Direction M_Inventory (copy)_2060396840691736620` | string · Measure | Basic | Diff ExcDmg | — | 1 |
| **Direction ITM**<br>`Direction Pre-1970 (copy) (copy)_2060396840646299665` | string · Measure | Basic | Diff ITM | — | 1 |
| **Direction M_Inventory**<br>`Direction Pre-1970 (copy)_2060396840650248214` | string · Measure | Basic | Diff M_Inventory | — | 1 |
| **Direction Pre-1970**<br>`Diff Pre-1970 (copy)_2060396840643641352` | string · Measure | Basic | Diff Pre-1970 | — | 1 |
| **Direction S_Inventory**<br>`Direction M_Inventory (copy)_2060396840651620379` | string · Measure | Basic | Diff S_Inventory | — | 1 |
| **Direction SSL**<br>`Direction Pre-1970 (copy)_2060396840646275084` | string · Measure | Basic | Diff SSL | — | 1 |
| **Direction Tickets**<br>`Direction ExcDmg (copy)_2060396840692125745` | string · Measure | Basic | Diff Tickets | — | 1 |
| **Equipment Failure Leak**<br>`Main Equipment Failure Leak (copy)_539024620999925763` | real · Measure | Basic | Main Equipment Failure Leak, Service Equipment Failure Leak | Leak Type - Equipment Failure | 4 |
| **EXC1_%**<br>`Calculation_2060396840668250144` | real · Measure | Basic | Excavation One-Call Notification Practice, Excavation Damages | — | 1 |
| **EXC2_%**<br>`EXC1_% (copy)_2060396840668557345` | real · Measure | Basic | Excavation Locating Practice, Excavation Damages | — | 1 |
| **EXC3_%**<br>`EXC1_% (copy) (copy)_2060396840668565538` | real · Measure | Basic | Excavation Excavation Practice, Excavation Damages | — | 1 |
| **EXC4_%**<br>`EXC1_% (copy) (copy) (copy)_2060396840668577827` | real · Measure | Basic | Excavation Other Practice, Excavation Damages | — | 1 |
| **Excavation Damage Leak**<br>`Main Excavation Damage Leak (copy)_539024620999954436` | real · Measure | Basic | Main Excavation Damage Leak, Service Excavation Damage Leak | % of Excavation Damage Leaks, Excavation Damage Leaks Repaired per mile of main, Leak Type - Excavation Damage | 4 |
| **Haz Main & Service Corrosion Failure Leak**<br>`Main & Service Corrosion Failure Leak (copy)_539024621001924619` | real · Measure | Basic | Total Hazleaks Cor Mains, Total Hazleaks Cor Srvs | Leak Type - Corrosion Failure | 5 |
| **Haz Main & Service Equipment Failure Leak**<br>`Main & Service Equipment Failure Leak (copy)_539024621002227725` | real · Measure | Basic | Total Hazleaks Eq Mains, Total Hazleaks Eq Srvs | Leak Type - Equipment Failure | 4 |
| **Haz Main & Service Excavation Damage Leak**<br>`Main & Service Excavation Damage Leak (copy)_539024621002248206` | real · Measure | Basic | Total Hazleaks Ex Mains, Total Hazleaks Ex Srvs | Leak Type - Excavation Damage | 4 |
| **Haz Main & Service Incorrect Operation Leak**<br>`Main & Service Incorrect Operation Leak (copy)_539024621002260495` | real · Measure | Basic | Total Hazleaks Op Mains, Total Hazleaks Op Srvs | Leak Type - Incorrect Operation | 4 |
| **Haz Main & Service Natural Forces Damage Leak**<br>`Main & Service Natural Forces Damage Leak (copy)_539024621002276880` | real · Measure | Basic | Total Hazleaks Nf Mains, Total Hazleaks Nf Srvs | Leak Type - Natural Force Damage | 4 |
| **Haz Main & Service Other Cause Leak**<br>`Main & Service Other Cause Leak (copy)_539024621002289169` | real · Measure | Basic | Total Hazleaks Ot Mains, Total Hazleaks Ot Srvs | Leak Type - Other Cause | 5 |
| **Haz Main & Service Other Outside Force Damage Leak**<br>`Main & Service Other Outside Force Damage Leak (copy)_539024621002305554` | real · Measure | Basic | Total Hazleaks Of Dam Mains, Total Hazleaks Of Dam Srvs | Leak Type - Other Outside Force Damage | 4 |
| **Haz Main & Service Pipe, Weld, or Joint Failure Leak**<br>`Main & Service Pipe, Weld, or Joint Failure Leak (copy)_539024621002321939` | real · Measure | Basic | Total Hazleaks Mat Weld Mains, Total Hazleaks Mat Weld Srvs | Leak Type - Pipe, Weld, or Joint Failure | 4 |
| **Hazardous Leaks per Mile (Mains & Services)**<br>`Hazardous Leaks per Mile (main leaks per main miles) (copy 2)` | real · Measure | Basic | All Main Hazleaks, All Service Hazleaks, Total Miles of Mains | Leaks per Unit | 4 |
| **Hazardous Leaks per Mile (Mains)**<br>`Calculation_891712746027474950` | real · Measure | Basic | All Main Hazleaks, Total Miles of Mains | Leaks per Unit | 4 |
| **Hazardous Leaks per Service Count**<br>`Hazardous Leaks per Mile (main leaks per main miles) (copy)` | real · Measure | Basic | All Service Hazleaks, Number of Services Total | Leaks per Unit | 4 |
| **Incorrect Operation Leak**<br>`Main Incorrect Operation Leak (copy)_539024620999970821` | real · Measure | Basic | Main Incorrect Operation Leak, Service Incorrect Operation Leak | Leak Type - Incorrect Operation | 4 |
| **Leak Type**<br>`Leak Type Unit (copy)_539024621016002591` | string · Dimension | Basic | — | — | 33 |
| **Leak Type - % Haz**<br>`Leak Type - All (copy)_1133781216225292289` | real · Measure | Basic | All Main/Service Hazleaks, All Main/Service Leaks, All Main Hazleaks, All Main Leaks, All Service Hazleaks, All Service Leaks | — | 1 |
| **Leak Type - All**<br>`Leaks per Unit - Leaks (copy)_539024620995436544` | real · Measure | Basic | All Main/Service Leaks, All Main Leaks, All Service Leaks, All Main/Service Hazleaks, All Main Hazleaks, All Service Hazleaks | Leak Type - All %, Leak Type - All Avg, Leak Type - Corrosion Failure %, Leak Type - Equipment Failure % … | 20 |
| **Leak Type - All Avg**<br>`Leak Type - All (copy)_1133781216229990404` | real · Measure | Basic | Leak Type - All, Max-Min Year Diff | — | 1 |
| **Leak Type - Corrosion Failure**<br>`Leaks Type (copy)_539024621001482250` | real · Measure | Basic | Corrosion Leak, Main Corrosion Failure Leak, Service Corrosion Failure Leak, Haz Main & Service Corrosion Failure Leak, Total Hazleaks Cor Mains, Total Hazleaks Cor Srvs | Leak Type - Corrosion Failure %, Leak Type - Corrosion Failure Avg | 5 |
| **Leak Type - Corrosion Failure %**<br>`Leak Type - Corrosion Failure (copy)_1027102219201011712` | real · Measure | Basic | Leak Type - Corrosion Failure, Leak Type - All | — | 1 |
| **Leak Type - Corrosion Failure Avg**<br>`Leak Type - Corrosion Failure % (copy)_935622827673165833` | real · Measure | Basic | Leak Type - Corrosion Failure, Max-Min Year Diff | — | 2 |
| **Leak Type - Equipment Failure**<br>`Leaks Type - Corrosion Failure (copy)_539024621004791830` | real · Measure | Basic | Equipment Failure Leak, Main Equipment Failure Leak, Service Equipment Failure Leak, Haz Main & Service Equipment Failure Leak, Total Hazleaks Eq Mains, Total Hazleaks Eq Srvs | Leak Type - Equipment Failure %, Leak Type - Equipment Failure Avg | 4 |
| **Leak Type - Equipment Failure %**<br>`Leak Type - Corrosion Failure % (copy)_1027102219210977281` | real · Measure | Basic | Leak Type - Equipment Failure, Leak Type - All | — | 1 |
| **Leak Type - Equipment Failure Avg**<br>`Leak Type - Corrosion Failure Avg (copy)_935622827675607052` | real · Measure | Basic | Leak Type - Equipment Failure, Max-Min Year Diff | — | 1 |
| **Leak Type - Excavation Damage**<br>`Leaks Type - Corrosion Failure (copy) (copy)_539024621004812311` | real · Measure | Basic | Excavation Damage Leak, Main Excavation Damage Leak, Service Excavation Damage Leak, Haz Main & Service Excavation Damage Leak, Total Hazleaks Ex Mains, Total Hazleaks Ex Srvs | Leak Type - Excavation Damage %, Leak Type - Excavation Damage Avg | 4 |
| **Leak Type - Excavation Damage %**<br>`Leak Type - Corrosion Failure % (copy) (copy)_1027102219210989570` | real · Measure | Basic | Leak Type - Excavation Damage, Leak Type - All | — | 1 |
| **Leak Type - Excavation Damage Avg**<br>`Leak Type - Corrosion Failure Avg (copy) (copy)_935622827675619341` | real · Measure | Basic | Leak Type - Excavation Damage, Max-Min Year Diff | — | 1 |
| **Leak Type - Incorrect Operation**<br>`Leaks Type - Corrosion Failure (copy) (copy) (copy)_539024621004840984` | real · Measure | Basic | Incorrect Operation Leak, Main Incorrect Operation Leak, Service Incorrect Operation Leak, Haz Main & Service Incorrect Operation Leak, Total Hazleaks Op Mains | Leak Type - Incorrect Operation %, Leak Type - Incorrect Operation Avg | 4 |
| **Leak Type - Incorrect Operation %**<br>`Leak Type - Corrosion Failure % (copy) (copy) (copy)_1027102219210997763` | real · Measure | Basic | Leak Type - Incorrect Operation, Leak Type - All | — | 1 |
| **Leak Type - Incorrect Operation Avg**<br>`Leak Type - Corrosion Failure Avg (copy) (copy) (copy)_935622827675627534` | real · Measure | Basic | Leak Type - Incorrect Operation, Max-Min Year Diff | — | 1 |
| **Leak Type - Main Count**<br>`Leak Type - All (copy)_1133781216227926018` | real · Measure | Basic | All Main Leaks, All Main Hazleaks | — | 2 |
| **Leak Type - Natural Force Damage**<br>`Leaks Type - Corrosion Failure (copy)_539024621005246489` | real · Measure | Basic | Natural Force Damage Leak, Main Natural Force Damage Leak, Service Natural Force Damage Leak, Haz Main & Service Natural Forces Damage Leak, Total Hazleaks Nf Mains, Total Hazleaks Nf Srvs | Leak Type - Natural Force Damage %, Leak Type - Natural Force Damage Avg | 4 |
| **Leak Type - Natural Force Damage %**<br>`Leak Type - Corrosion Failure % (copy) (copy) (copy) (copy)_1027102219211005956` | real · Measure | Basic | Leak Type - Natural Force Damage, Leak Type - All | — | 1 |
| **Leak Type - Natural Force Damage Avg**<br>`Leak Type - Corrosion Failure Avg (copy) (copy)_935622827679326230` | real · Measure | Basic | Leak Type - Natural Force Damage, Max-Min Year Diff | — | 1 |
| **Leak Type - Other Cause**<br>`Leaks Type - Corrosion Failure (copy)_539024621005942810` | real · Measure | Basic | Other Cause Leak, Main Other Cause Leak, Service Other Cause Leak, Haz Main & Service Other Cause Leak, Total Hazleaks Ot Mains, Total Hazleaks Ot Srvs | Leak Type - Other Cause %, Leak Type - Other Cause Avg | 5 |
| **Leak Type - Other Cause %**<br>`Leak Type - Natural Force Damage % (copy)_1027102219214815237` | real · Measure | Basic | Leak Type - Other Cause, Leak Type - All | — | 1 |
| **Leak Type - Other Cause Avg**<br>`Leak Type - Corrosion Failure Avg (copy)_935622827679313941` | real · Measure | Basic | Leak Type - Other Cause, Max-Min Year Diff | — | 2 |
| **Leak Type - Other Outside Force Damage**<br>`Leaks Type - Corrosion Failure (copy 3)_539024621005963292` | real · Measure | Basic | Other Outside Force Damage Leak, Main Other Outside Force Damage Leak, Service Other Outside Force Damage Leak, Haz Main & Service Other Outside Force Damage Leak, Total Hazleaks Of Dam Mains, Total Hazleaks Of Dam Srvs | Leak Type - Other Outside Force Damage %, Leak Type - Other Outside Force Damage Avg | 4 |
| **Leak Type - Other Outside Force Damage %**<br>`Leak Type - Natural Force Damage % (copy) (copy)_1027102219214827526` | real · Measure | Basic | Leak Type - Other Outside Force Damage, Leak Type - All | — | 1 |
| **Leak Type - Other Outside Force Damage Avg**<br>`Leak Type - Corrosion Failure Avg (copy) (copy) (copy)_935622827679334423` | real · Measure | Basic | Leak Type - Other Outside Force Damage, Max-Min Year Diff | — | 1 |
| **Leak Type - Pipe, Weld, or Joint Failure**<br>`Leaks Type - Corrosion Failure (copy 2)_539024621005955099` | real · Measure | Basic | Pipe, Weld, or Joint Failure Leak, Main Pipe, Weld, or Joint Failure Leak, Service Pipe, Weld, or Joint Failure Leak, Haz Main & Service Pipe, Weld, or Joint Failure Leak, Total Hazleaks Mat Weld Mains, Total Hazleaks Mat Weld Srvs | Leak Type - Pipe, Weld, or Joint Failure %, Leak Type - Pipe, Weld, or Joint Failure Avg | 4 |
| **Leak Type - Pipe, Weld, or Joint Failure %**<br>`Leak Type - Natural Force Damage % (copy) (copy) (copy)_1027102219214843911` | real · Measure | Basic | Leak Type - Pipe, Weld, or Joint Failure, Leak Type - All | — | 1 |
| **Leak Type - Pipe, Weld, or Joint Failure Avg**<br>`Leak Type - Corrosion Failure Avg (copy) (copy) (copy) (copy)_935622827679346712` | real · Measure | Basic | Leak Type - Pipe, Weld, or Joint Failure, Max-Min Year Diff | — | 1 |
| **Leak Type - Service Count**<br>`Leak Type - Main Count (copy)_1133781216228085763` | real · Measure | Basic | All Service Leaks, All Service Hazleaks | — | 1 |
| **Leak Type Unit**<br>`Leaks Type (copy)_539024621013073950` | string · Dimension | Basic | — | — | 33 |
| **Leaks per Mile (Mains & Services)**<br>`Leaks per Mile (main leaks per main miles) (copy)` | real · Measure | Basic | All Main/Service Leaks, Total Miles of Mains | Leaks per Unit | 4 |
| **Leaks per Mile (Mains)**<br>`Calculation_653021999114092551` | real · Measure | Basic | All Main Leaks, Total Miles of Mains | Leaks per Unit | 4 |
| **Leaks per Service Count**<br>`Calculation_653021999140106250` | real · Measure | Basic | All Service Leaks, Number of Services Total | Leaks per Unit | 4 |
| **Leaks per Unit**<br>`Calculation_969681333266161675` | real · Measure | Basic | Leaks per Mile (Mains & Services), Leaks per Mile (Mains), Leaks per Service Count, Hazardous Leaks per Mile (Mains & Services), Hazardous Leaks per Mile (Mains), Hazardous Leaks per Service Count | — | 4 |
| **Leaks per Unit - Leaks**<br>`Leak per ((Miles (copy)_1266355960159866881` | real · Measure | Basic | All Main/Service Leaks, All Main Leaks, All Service Leaks, All Main/Service Hazleaks, All Main Hazleaks, All Service Hazleaks | — | 4 |
| **Leaks per Unit - Mileage**<br>`Leak per (copy)_1266355960158748672` | real · Measure | Basic | Total Miles of Mains, Number of Services Total | — | 4 |
| **Leaks per Unit - Unit**<br>`Leaks per Unit - Mileage (copy)_1266355960173420552` | string · Dimension | Basic | — | — | 4 |
| **Leaks per Unit - Unit Title**<br>`Leaks per Unit - Unit (copy)_1133781216256692236` | string · Dimension | Basic | — | — | 4 |
| **Leaks per Unit - Unit txt 2**<br>`Leaks per Unit - Unit (copy)_1133781216237322247` | string · Dimension | Basic | — | — | 2 |
| **Max-Min Year Diff**<br>`Calculation_935622827673276426` | integer · Dimension | LOD | Report Year | Leak Type - All Avg, Leak Type - Corrosion Failure Avg, Leak Type - Equipment Failure Avg, Leak Type - Excavation Damage Avg … | 17 |
| **Natural Force Damage Leak**<br>`Main Natural Forces Damage Leak (copy)_539024620999995398` | real · Measure | Basic | Main Natural Force Damage Leak, Service Natural Force Damage Leak | Leak Type - Natural Force Damage | 4 |
| **Other Cause Leak**<br>`Main Other Cause Leak (copy)_539024621000011783` | real · Measure | Basic | Main Other Cause Leak, Service Other Cause Leak | Leak Type - Other Cause | 5 |
| **Other Outside Force Damage Leak**<br>`Main Other Outside Force Damage Leak (copy)_539024621000036360` | real · Measure | Basic | Main Other Outside Force Damage Leak, Service Other Outside Force Damage Leak | Leak Type - Other Outside Force Damage | 4 |
| **Pipe, Weld, or Joint Failure Leak**<br>`Main Pipe, Weld, or Joint Failure Leak (copy)_539024621000056841` | real · Measure | Basic | Main Pipe, Weld, or Joint Failure Leak, Service Pipe, Weld, or Joint Failure Leak | Leak Type - Pipe, Weld, or Joint Failure | 4 |
| **Pre-1970 Main Mileage (Including Unknown)**<br>`Calculation_1714464112005423104` | real · Measure | Basic | Mmiles By Dcd Pre1940, Mmiles By Dcd 1940 To 1949, Mmiles By Dcd 1950 To 1959, Mmiles By Dcd 1960 To 1969, Mmiles By Dcd Unk | % of System Pre-1970 Main Mileage (Including Unknown), *Material, Current Pre-1970, Previous Pre-1970 | 13 |
| **Previous DmgRate**<br>`Previous Tickets (copy)_2060396840694108212` | real · Measure | Basic | Previous ExcDmg, Previous Tickets | Diff % DmgRate, Diff DmgRate | 1 |
| **Previous ExcDmg**<br>`Previous M_Inventory (copy)_2060396840691736621` | real · Dimension | LOD | Report Year, Excavation Damages | Diff % ExcDmg, Diff ExcDmg, Previous DmgRate | 2 |
| **Previous ITM**<br>`Previous Pre-1970 (copy) (copy)_2060396840646299666` | real · Dimension | LOD | Report Year, Total Industry Targeted Material | Diff % ITM, Diff ITM | 1 |
| **Previous M_Inventory**<br>`Previous Pre-1970 (copy)_2060396840650248211` | real · Dimension | LOD | Report Year, Total Miles of Mains | Diff % M_Inventory, Diff M_Inventory | 1 |
| **Previous Pre-1970**<br>`Current Pre-1970 (copy)_2060396840640622597` | real · Dimension | LOD | Report Year, Pre-1970 Main Mileage (Including Unknown) | Diff % Pre-1970, Diff Pre-1970 | 1 |
| **Previous S_Inventory**<br>`Previous M_Inventory (copy)_2060396840651620380` | real · Dimension | LOD | Report Year, Number of Services Total | Diff % S_Inventory, Diff S_Inventory | 1 |
| **Previous SSL**<br>`Previous Pre-1970 (copy)_2060396840646275081` | real · Dimension | LOD | Report Year, Steel Service Lines | Diff % SSL, Diff SSL | 1 |
| **Previous Tickets**<br>`Previous ExcDmg (copy)_2060396840692125746` | real · Dimension | LOD | Report Year, Excavation Tickets | Diff % Tickets, Diff Tickets, Previous DmgRate | 2 |
| **Steel Service Lines**<br>`Calculation_1714464112007512071` | real · Measure | Basic | Num Srvs Steel Cp Bare, Num Srvs Steel Cp Coated, Num Srvs Steel Unp Bare, Num Srvs Steel Unp Coated | *Material, Current SSL, Previous SSL | 6 |
| **Total Industry Targeted Material**<br>`Calculation_1714464112021135380` | real · Measure | Basic | Miles of Reconditioned Cast Iron, Bare Steel, Miles of Cast Iron, Vintage Plastic | % of System Industry Targeted Mileage, *Material, Current ITM, Previous ITM | 6 |
| **Vintage Plastic**<br>`Calculation_1714464112009375754` | real · Measure | Basic | Mmiles Abs Total, Mmiles Plastic Total, Mmiles Oth Plstc Unk | Total Industry Targeted Material | 6 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| **Select an Material Over Time View Chart:**<br>`Parameter 3` | string · list | `"1"` | list of 3 — "1" → Pre-1970 Mainline Mileage, "2" → Industry Targeted Material, "3" → Steel Service Line Counts |
| **Select a Leak Type:**<br>`Select a Measure Type: (copy)_539024620995678209` | string · list | `"Main & Service Leaks"` | list of 3 — "Main & Service Leaks" → Mainline & Service Leaks, "Main Leaks" → Mainline Leaks, "Service Leaks" → Service Leaks |
| **Select an Leaks Over Time View Chart:**<br>`Select an Over Time View Chart: (copy)_1027102219249680398` | string · list | `"A"` | list of 3 — "A" → Hazardous Leaks per Main Mile, "B" → Corrosion Leaks per Main Mile, "C" → Excavation Damage Leaks per Main Mile |

### How the numbers are computed

**Level-of-detail expressions — 15.** These fix their own level of detail, so they return the same value regardless of what dimensions are in the view:

> `Max-Min Year Diff`, `Current Tickets`, `Current S_Inventory`, `Current ExcDmg`, `Current ITM`, `Previous Pre-1970`, `Current SSL`, `Current M_Inventory`, `Current Pre-1970`, `Previous Tickets`, `Previous S_Inventory`, `Previous ExcDmg`, `Previous ITM`, `Previous SSL`, `Previous M_Inventory`

## Appendix A — Calculated fields excluded from this documentation

55 calculated fields exist in the data pane but are not referenced by any worksheet or dashboard
logic. They comprise unused halves of the leak-switcher family (per-cause variants for leak types
not currently exposed), alternative rate denominators, and earlier per-material breakdown
experiments superseded by the `*Material` switcher.

---

*Compiled from the local workbook file (`U.S. Gas Distribution Infrastructure.twb`, modified
2024-02-19) parsed field-by-field, with rendered figures read from all four published views via
the Tableau Public MCP server. All formulas and settings are quoted verbatim from the workbook
definition.*
