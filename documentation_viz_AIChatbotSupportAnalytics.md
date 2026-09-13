# AI Chatbot Support Analytics — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `AI Chatbot Support Analytics` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/AIChatbotSupportAnalytics/ChatbotSupport |
| Dataset | Published free at https://github.com/jsjohansson/Data_AI-Chatbot-Support-Tickets |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `AIChatbotSupportAnalytics.twbx`, decompiled and parsed field-by-field |
| Explicit exclusion | 8 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic**. Listed by name only in Appendix B. |

---

## 1. Executive summary

**AI Chatbot Support Analytics** measures how an AI chatbot tier performs against a human IT
support tier on the same ticket queue: how many tickets each closes, how long each takes, and
how often the bot escalates to a person.

The workbook is notable for being an **end-to-end build** — the author designed the dataset,
built the visualisation, and published the data openly. The published description states the
pipeline explicitly: *Dataset Design ➡️ Viz Development ➡️ MCP Analysis*.

Architecturally the defining choice is the **union**. Rather than joining projections to
actuals, the data source unions an `Actual` table and a `Projection` table into one stream,
tagging each row with `Record Type`. A single bar chart then renders both series by splitting
on that tag — which is how the "Ticket Processing Counts | Actuals vs. Projections" panel shows
projection bars extending past the last month of real data.

Interactivity is parameter-driven throughout: a View toggle (Metrics / Details), a Metric
toggle (Processor / Status / Severity) that re-cuts the summary donut, and a five-year
selector — all wired through 14 actions and 16 Dynamic Zone Visibility bindings.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | 82 |
| Dashboards | 1 (`Chatbot Support`) |
| Data sources | 1 (union of two tables) |
| Parameters | 4 |
| Calculated fields **used** | 56 named + 75 in-view ad-hoc |
| Calculated fields unused (not documented) | 8 |
| Extract | 313,044 rows × 23 columns |
| Dashboard actions | 3 URL, 11 parameter |
| Dynamic Zone Visibility bindings | 16 |
| Global font | Arial |

### 1.2 Headline figures rendered by the dashboard

Default state — Year 2026, Metric = Processor, View = Metrics:

| Element | Value |
|---|---|
| Open Tickets | **2,272** |
| Earliest Open Ticket | 7/6/2026 |
| Last Updated | 10/1/2026 |
| AI Ticket Days | **6.7** |
| IT Escalation Days | **11.3** |
| Total Ticket Days | **12.9** |

**Ticket Processor Summary**

| Tier | Tickets | Share | Closed | Open |
|---|---:|---:|---:|---:|
| IT Support Escalation | 8,342 | 28.2 % | 7,724 | 618 |
| AI Bot Support | 21,278 | 71.8 % | 19,624 | 1,654 |

**Bot Closure Rate** — overall **66.3 %**

| Bot | Rate | Closed | Bot | Rate | Closed |
|---|---:|---:|---|---:|---:|
| Kaki Bot | 77.3 % | 3,736 | Nico Bot | 64.6 % | 3,186 |
| Alex Bot | 73.9 % | 3,603 | Ralph Bot | 59.1 % | 2,914 |
| Chad Bot | 70.3 % | 3,551 | Phil Bot | 52.7 % | 2,634 |

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `19015105` |
| LUID | `a2c5a5c1-74aa-46bc-a973-a068f75f8335` |
| Repository URL | `AIChatbotSupportAnalytics` |
| Default view | `Chatbot Support` |
| Revision | 2.0 |
| First published | 2026-09-04 |
| Last published | 2026-09-09 |
| Published size | 4,424,117 bytes (4.2 MB) |
| View count | 1,907 |
| Favourites | 19 |
| External link | `http://github.com/jsjohansson/Data_AI-Chatbot-Support-Tickets` |

**Published description**

> An IT support analytics dashboard measuring AI chatbot performance: bots closure rates, IT
> support escalation, and how the two stack up.
> End to End Stages: Dataset Design ➡️ Viz Development ➡️ MCP Analysis
> Dataset Free on GitHub

---

## 3. Data architecture

### 3.1 Connection and model

| Property | Value |
|---|---|
| Data source | `AI Chatbot Support Tickets` |
| Relations | 3 — `Actual+` (union) containing `Actual` and `Projection` |
| **Union** | **2 tables** |
| Joins | None |
| Custom SQL | None |
| Extract rows | **313,044** |
| Columns | 23 |
| Data source filters | None |

**The union is the workbook's central design decision.** `Actual` holds real ticket records;
`Projection` holds forecast volumes. Unioning rather than joining puts both on the same rows
and columns, distinguished only by `Record Type`. Every measure can then be split by that
field, and the comparison chart renders both series from one mark type — projection bars
simply continue past the last actual month.

### 3.2 Grain

One row per ticket record. `Ticket ID` is the key; counts use `COUNTD([Ticket ID])` throughout.
Because the union doubles the conceptual row space (actuals + projections), **every aggregate
must be scoped by `Record Type`** or it will silently sum forecast into fact — which is why
the `{FIXED}` fields in §6.3 all carry `[Record Type]` in their dimensionality.

### 3.3 Materialised schema (23 columns)

| Column | Type | Role |
|---|---|---|
| `Ticket ID` | string | Primary key; all counts |
| `Requestor Name`, `Requestor ID` | string | Who raised the ticket |
| `Office Location`, `Team Name` | string | Organisational breakdown |
| `Issue Category`, `Issue Type` | string | The escalation-issues bar chart |
| `Ticket Type` | string | Classification |
| `Severity` | string | High / Medium / Low — one of the three Metric cuts |
| `Ticket Status` | string | Open / Closed / Closed AI Task |
| `Ticket Sub-Status` | string | Resolved, Cancelled, No Action, User Feedback… |
| `Ticket Processor Status` | string | Open/Closed × AI Ticket/Escalation |
| `Created Date`, `Escalated Date`, `Closed Date` | date | The duration measures |
| `Days Open` | integer | Pre-computed age |
| `Assigned AI Bot` | string | Which of the six bots took it |
| `Assigned IT Processor` | string | Which human |
| `Current Assigned Support`, `Current Assigned Name` | string | Where it sits now |
| `Record Type` | string | **`Actual` or `Projection`** |
| `Sheet`, `Table Name` | string | Union provenance columns |

---

## 4. Parameters

| Caption | Type | Domain | Default | Role |
|---|---|---|---|---|
| **P-View** | string list | `Metrics` · `Details` | `Metrics` | Master view switch |
| **P-Metric** | string list | `Processor` · `Status` · `Severity` | `Processor` | Re-cuts the summary donut and legend |
| **P-Year** | real list | 2021 – 2027 | `2026` | Reporting year |
| **P-Status** | string list | `All` · `Open` · `Closed` | `All` | Status filter |

---

## 5. Calculated fields in use

56 named calculations plus 75 in-view ad-hoc. Grouped by role.

### 5.1 The core measure — did the bot resolve it?

#### `AI Completed` — used in **25 sheets**
```
IF [Assigned AI Bot] = [Current Assignment] THEN 'AI' ELSE 'HUMAN' END
```
The analytical heart of the workbook. A ticket was resolved by the bot if the bot it was
*assigned to* is still the support it is *currently assigned to*; if the current assignment has
moved on, a human took it. This single comparison is what separates the two tiers, and
everything downstream — closure rates, tier splits, day counts — derives from it.

#### `AI Count`
```
IF [AI Completed] = 'AI' THEN 1 ELSE 0 END
```
The 1/0 flag form, used as a summable measure.

#### `AI % Service Rate` — used in 13 sheets
```
[AI Count] / [AI Total Assigned]
```
Per-bot closure rate — the 77.3 % / 73.9 % / 70.3 % / 64.6 % / 59.1 % / 52.7 % figures.

```
AI Total Assigned        { FIXED [Assigned AI Bot], [Year], [Record Type]: COUNTD([Ticket ID]) }
AI Total Assigned (ALL)  { FIXED [Year], [Record Type]: COUNTD([Ticket ID]) }
AI % Service Rate (ALL)  [AI Count] / [AI Total Assigned (ALL)]
```
The `(ALL)` pair drops `Assigned AI Bot` from the dimensionality to produce the **overall
66.3 %** rate, so the per-bot bars and the overall badge can sit in the same view without
either recomputing the other's denominator. Note `[Record Type]` in both — without it the
projection rows would inflate every rate.

### 5.2 The date spine

#### `Date` — used in **72 sheets**
```
IF ISNULL([Closed Date]) THEN [Created Date] ELSE [Closed Date] END
```
A single coalesced date so open and closed tickets can share one time axis: closed tickets are
placed at their closure, open ones at their creation.

```
Year        YEAR([Date])
Year T|F    YEAR([Date]) = [P-Year]      ← used in 72 sheets as the year filter
Backlog Date  { FIXED : MIN([Created Date]) }     ← the "Earliest Open Ticket" badge
Update Date   DATE('10/01/2026')                  ← the "Last Updated" badge
```

`Year T|F` appearing in 72 of 82 sheets is the pattern: rather than a filter card, the year
parameter is applied as a boolean on every sheet.

### 5.3 Duration

```
Human Ticket Days  DATEDIFF('day',[Escalated Date],[Closed Date])
```
Human handling time is measured from **escalation**, not creation — so the 11.3-day IT
Escalation figure excludes the time the bot already spent on it. `AI Ticket Days` (6.7) and
`Total Ticket Days` (12.9) come from the source `Days Open` column measured over the
corresponding populations.

### 5.4 Share-of-total

```
Fixed Total Tickets  { FIXED [Year T|F], [Record Type] : COUNTD([Ticket ID]) }
Fixed % of Total     COUNTD([Ticket ID]) / SUM([Fixed Total Tickets])
```
Used in 9 sheets. Fixing on `[Year T|F]` rather than `[Year]` is deliberate — the denominator
follows the *selected* year, so percentages re-base when the year changes.

### 5.5 Status classification

A family of fields that collapse the two-column status model (`Ticket Status` ×
`Ticket Sub-Status`) into single labelled categories for the donut segments:

| Field | Formula (abbreviated) |
|---|---|
| **Closed 03** | `IF [Ticket Status]='Closed' AND [Ticket Sub-Status]='Resolved' THEN 'Closed-Closed' ELSEIF [Ticket Status]='Open' or [Ticket Sub-Status]='User Feedback' THEN 'False1' ELSE …` |
| **Open 03** | `IF [Ticket Processor Status Group]='Open AI Ticket' AND [Ticket Sub-Status]='Open AI Ticket' THEN 'Open AI-Open' ELSEIF …` |
| **Pending 03** | Complementary branch of the above |
| **No Action 03** | `IF [Ticket Sub-Status]='Cancelled' THEN 'Closed-Cancelled' ELSEIF [Ticket Sub-Status]='No Action' THEN 'Closed-No Action' ELSEIF …` |
| **Closed HML 03** | Severity × closed-state cross-classification |
| **Open AI HML 03** | Severity × open-AI cross-classification |
| **Open Escalation HML 03** | Severity × open-escalation cross-classification |

The `False1` / `False 2` return values are placeholder categories that render transparent —
the standard technique for building a multi-ring donut where each ring shows only its own
slice of the classification and leaves the rest blank.

```
Closed Escalation   IF [Ticket Processor Status]='Closed Escalation' THEN 'True'
                    ELSEIF [Ticket Processor Status]='Open Escalation' THEN 'False 2' ELSE 'False 1' END
Open Escalation     mirror of the above
Closed AI Ticket    [Ticket Processor Status] = 'Closed AI Ticket'
Open AI Ticket      [Ticket Processor Status] = 'Open AI Ticket'
```

```
Closed AI Task-x1    COUNTD(IF [Ticket Status]='Closed AI Task' then [Ticket ID] end)
Closed AI Task-x2 %  ([Closed AI Task-x1] / COUNT([Ticket ID])) * 100
```

### 5.6 The metric switcher

```
Color Legend  CASE [P-Metric]
                WHEN 'Processor' THEN [AI Completed]
                WHEN 'Status'    THEN [Ticket Processor Status Group]
                WHEN 'Severity'  THEN [Severity] END
```
One field returning whichever dimension the Metric toggle selects, so the donut and its legend
re-cut without swapping sheets. Supporting fields `Group Serverity-H/M/L` alias the processor
status group per severity band.

```
Processor A|H  IF [Current Support]='AI Bot' THEN 'A' ELSE 'H' END
```

### 5.7 View and metric booleans (Dynamic Zone Visibility controls)

| Field | Formula | Controls |
|---|---|---|
| **Metrics T\|F** | `[P-View] = 'Metrics'` | The `Main` metrics container |
| **Details T\|F** | `[P-View] = 'Details'` | The details container |
| **Metric-P T\|F** | `[P-Metric] = 'Processor'` | `Pie Processor` + 3 supporting zones |
| **Metric-St T\|F** | `[P-Metric] = 'Status'` | `Pie Status` + 3 supporting zones |
| **Metric-Se T\|F** | `[P-Metric] = 'Severity'` | `Pie HML` + 3 supporting zones |
| **Status-Year 2026 T\|F** | `[P-Year] = 2026` | Two spacer zones |
| **Status-Year 2022/23/24/25 T\|F** | `[P-Year] = <year>` | Year-button colour |

### 5.8 Year navigation constants

```
CY    2026
CY-1  [CY]-1        CY-2  [CY]-2        CY-3  [CY]-3        CY-4  [CY]-4
```
Five constants feeding the five year buttons. Each button sheet carries its constant on Detail
and a parameter action passes it to `P-Year`.

### 5.9 Helpers

| Field | Formula | Purpose |
|---|---|---|
| **A_Color_Icons / B_Color_Icons / C_Color_Icons** | `'A'` / `'B'` / `'B'` | Single-member dimensions forcing fixed icon colours |
| **Center** | `MAKEPOINT(0,0)` | Donut anchor point |
| **Placement** | `IF [Record Type]='Actual' THEN 1 ELSE 2 END` | Orders actual bars left of projection bars |
| **Ticket Age < 31** / **Ticket Age >100** | `[Days Open] < 31` / `> 100` | Age-band flags |
| **Random Number 02** | `RIGHT(LEFT([Ticket ID],7),1)` | Pseudo-random digit extracted from the ticket ID, used for sampling in the details view |
| **Ticket H Sub-Status** | `[Ticket Sub-Status]` | Alias for a second use of the field |

---

## 6. Worksheet specifications

82 worksheets. Grouped by dashboard region.

### 6.1 Header badges (3 sheets)

`Open Tickets`, `Earliest Open Ticket`, `Last Updated` — Text marks on rounded containers,
fed by `COUNTD([Ticket ID])` filtered to open, `Backlog Date`, and `Update Date` respectively.

### 6.2 Control rail (10 sheets)

| Group | Sheets | Construction |
|---|---|---|
| View | `Ticket Metrics`, `Ticket Details` | Square/Text marks; colour by `Metrics T\|F` / `Details T\|F`; parameter action → `P-View` |
| Metric | `M-P`, `M-St`, `M-Se` | Colour by the matching `Metric-* T\|F`; payload `'Processor'` / `'Status'` / `'Severity'` → `P-Metric` |
| Year | `CY`, `CY-1`, `CY-2`, `CY-3`, `CY-4` | Colour by `Status-Year <year> T\|F`; detail carries the year constant → `P-Year` |

### 6.3 Ticket Processor Summary (the donut cluster)

A multi-ring donut built from three parallel sheet families, only one of which is visible at a
time:

| Family | Sheets | Shown when |
|---|---|---|
| `Pie Processor` + supporting rings | AI vs Human split | Metric = Processor |
| `Pie Status` + supporting rings | Open/Closed classification | Metric = Status |
| `Pie HML` + supporting rings | High/Medium/Low severity | Metric = Severity |

Each ring uses one of the `… 03` classification fields from §5.5, with the `False1` / `False 2`
categories rendered transparent so each ring contributes only its own arc.

Beneath the donut, two tier cards (`8,342 / 28.2 %` IT Support Escalation and
`21,278 / 71.8 %` AI Bot Support) break down into Closed and Open counts.

### 6.4 Average Ticket Closure Days

A line chart of monthly average days with three summary figures above it (AI Ticket Days 6.7,
IT Escalation Days 11.3, Total Ticket Days 12.9). Circles mark each month; values printed
beneath (12.1, 12.1, 12.9, 12.5, 13.0, 12.9, 13.0, 14.5, 12.8).

### 6.5 Ticket Processing Counts — Actuals vs. Projections

The union payoff. A bar chart where:

- **Columns**: month
- **Rows**: `COUNTD([Ticket ID])`
- **Colour**: `Record Type` → Projection (dark navy), IT Support (teal), AI Bot (pink)
- **Order**: `Placement` puts actual bars left of projection bars

Below it a four-row table prints Projection, Actual Total, IT Support and AI Bot counts per
month. Actual data stops after Sep 26; Oct–Dec 26 show projection bars only (3,378 / 3,403 /
3,682), which is exactly what the union enables.

### 6.6 Top Ticket Escalation Issues

Horizontal bar chart of `Issue Type` by ticket count — Account Access 1,594, App Errors 1,067,
Email/Collaboration 899, Hardware 777, Software Installs 772, Network 745, MFA 705,
VPN/Remote Access 689, Mobile/BYOD 381, Printer/Peripheral 265, Data Recovery 262,
AI Assistant 166.

### 6.7 Bot Closure Rate

Six paired sheets, one per bot: a label with the bot name and `AI % Service Rate`, and a
stacked bar showing closed (pink) against remaining (grey). An **Overall Rate: 66.3 %** badge
uses the `(ALL)` variant of the rate.

### 6.8 Details view

Revealed when `P-View = 'Details'` — a ticket-level table using `Random Number 02` for
sampling and the age-band flags for filtering.

---

## 7. Dashboard specification

| Property | Value |
|---|---|
| Name | `Chatbot Support` |
| Canvas | 1600 × 926 px |
| Ground | Dark navy `#2d3142`-family |
| Panels | White cards on the dark ground |

**Regions**

```
Header       robot avatar · title · three badges · designer credit · social icons
Left rail    View toggle · Metric toggle · Year buttons · "Click to select criteria"
             Ticket Processor Summary (donut + two tier cards)
Main         Average Ticket Closure Days (line + 3 figures)
             Ticket Processing Counts (bars + 4-row table)
             Top Ticket Escalation Issues (bar) | Bot Closure Rate (6 bars)
```

---

## 8. Interactivity

### 8.1 Parameter actions (11)

| Caption | Source sheet | Payload | Target |
|---|---|---|---|
| `Ticket-Metrics` | `Ticket Metrics` | `'Metrics'` | **P-View** |
| `View-Details` | `Ticket Details` | `'Details'` | **P-View** |
| `Metric-Processor` | `M-P` | `'Processor'` | **P-Metric** |
| `Metric-Status` | `M-St` | `'Status'` | **P-Metric** |
| `Metric-Severity` | `M-Se` | `'Severity'` | **P-Metric** |
| `Year-26` | `CY` | `'2026'` | **P-Status** |
| `Year-25` | `CY-1` | `CY-1` | **P-Year** |
| `Year-24` | `CY-2` | `CY-2` | **P-Year** |
| `Status-Open` | `CY-2` | `CY-2` | **P-Year** |
| `Year-23` | `CY-3` | `CY-3` | **P-Year** |
| `Year-22` | `CY-4` | `CY-4` | **P-Year** |

All trigger on select.

### 8.2 URL actions (3)

| Caption | Target |
|---|---|
| `Data Link` | `https://github.com/jsjohansson/Data_AI-Chatbot-Support-Tickets` |
| `TP` | `https://public.tableau.com/app/profile/john.johansson` |
| `LI` | `https://www.linkedin.com/in/johnsjohansson/` |

The `Data Link` action is unusual and worth noting: the dashboard links directly to its own
source dataset, which is what makes the "end to end" claim in the description verifiable.

### 8.3 Dynamic Zone Visibility (16 bindings)

| Controlling field | Zones |
|---|---|
| `Metrics T\|F` | `Main` |
| `Details T\|F` | The details flow container |
| `Metric-P T\|F` | `Pie Processor` + 2 flow containers + 1 text zone |
| `Metric-St T\|F` | `Pie Status` + 2 flow containers + 1 text zone |
| `Metric-Se T\|F` | `Pie HML` + 2 flow containers + 1 text zone |
| `Status-Year 2026 T\|F` | 2 spacer zones |

**Resulting states**

| P-View | P-Metric | Main stage | Donut |
|---|---|---|---|
| Metrics | Processor *(default)* | Metrics panels | AI vs Human split |
| Metrics | Status | Metrics panels | Open/Closed classification |
| Metrics | Severity | Metrics panels | High/Medium/Low |
| Details | any | Ticket detail table | hidden |

---

## 9. Design system

| Token | Hex family | Use |
|---|---|---|
| Dark navy | `#2d3142` | Dashboard ground, header, projection bars |
| Teal | `#3f7f8c` | IT Support tier, secondary bars |
| Pink/magenta | `#c4557f` | AI Bot tier, closure-rate fill |
| White | `#ffffff` | Panel cards |
| Grey | `#b6bbc1` | Remainder portions on rate bars |

The two-colour tier convention (teal = IT Support, pink = AI Bot) holds across the donut, the
processing-counts bars and the tier cards, so the reader learns it once.

**Typography** — Arial throughout, set at workbook level. Large bold KPI numerals, small
uppercase captions, bold panel titles.

---

## 10. Rebuild / maintenance runbook

**Refreshing the data**

1. Update both union members. The `Actual` and `Projection` tables must keep identical column
   structure — the union collapses on column name, and a mismatch silently produces nulls.
2. Update `Update Date` (`DATE('10/01/2026')`), which is a hard-coded literal powering the
   "Last Updated" badge.
3. Update the `CY` constant (`2026`) if the reporting year moves; `CY-1` through `CY-4` follow
   automatically.
4. Extend the `P-Year` parameter's allowable values if a new year is added.

**Watch the Record Type scoping**

Every `{FIXED}` in this workbook carries `[Record Type]` in its dimensionality. If a new
aggregate is added without it, projections will be counted as actuals. This is the single
easiest mistake to make in this model.

**Adding a metric cut**

1. Add a member to `P-Metric`.
2. Add a `Metric-<X> T|F` boolean.
3. Extend `Color Legend` with the new `WHEN` branch.
4. Build the donut ring sheets using the `… 03` classification pattern, returning placeholder
   categories for the slices that ring should not draw.
5. Add a nav sheet, a parameter action, and four DZV bindings (pie + two containers + caption).

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | AI Chatbot Support Tickets |
| Connection class | `federated` |
| Tables / relations | 3 · joins: 2 |
| Extract rows | 313,044 |
| Physical columns materialised | 23 |
| Source fields in data pane | 8 |
| Calculated fields (used / total) | 56 / 64 |
| Parameters | 4 |
| Data source filters | 0 (none) |

### Source fields

8 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Assignment

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Current Assignment**<br>`Current Assigned Name` | string · Dimension | 32 | yes | 26 | Named party currently holding the ticket. |
| **Current Support**<br>`Current Assigned Support` | string · Dimension | 2 | yes | 8 | Tier currently holding the ticket. Compared against Assigned AI Bot to determine who resolved it. |

#### Status

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Ticket Status** | string · Dimension | 2 | yes | 25 | Open, Closed, or Closed AI Task. |
| **Ticket Processor Status** | string · Dimension | 4 | yes | 22 | Combined tier and state — Open/Closed × AI Ticket/Escalation. |
| **Ticket Processor Status Group** | string · Dimension | — | — | 17 | Grouping of Ticket Processor Status that collapses the tier-and-state combinations into the handful of states the dashboard distinguishes. |
| **Ticket Sub-Status** | string · Dimension | 6 | yes | 8 | Detail beneath status — Resolved, Cancelled, No Action, User Feedback. |

#### Issue

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Severity** | string · Dimension | 3 | yes | 12 | Severity band — High, Medium, Low. One of the three Metric cuts. |
| **Severity (group)** | string · Dimension | — | — | 8 | Author-defined grouping built on **Severity**. Severity band — High, Medium, Low. One of the three Metric cuts. |

### Calculated fields in use

56 of the workbook's 64 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| **A_Color_Icons**<br>`Calculation_1642440910688283` | string · Dimension | Basic | — | — | 4 |
| **AI % Service Rate**<br>`AI Count (copy)_0113016085852162` | real · Measure | Basic | AI Count, AI Total Assigned | — | 13 |
| **AI % Service Rate (ALL)**<br>`AI % Service Rate (copy)_0605597422452744` | real · Measure | Basic | AI Count, AI Total Assigned (ALL) | — | 1 |
| **AI Completed**<br>`Calculation_1694662654562325` | string · Dimension | Basic | Current Assignment | AI Count, Color Legend, Human Count | 25 |
| **AI Count**<br>`Calculation_0113016085516288` | integer · Measure | Basic | AI Completed | AI % Service Rate, AI % Service Rate (ALL) | 14 |
| **AI Total Assigned**<br>`AI Count (copy)_0113016087400451` | integer · Measure | LOD | Year | AI % Service Rate | 13 |
| **AI Total Assigned (ALL)**<br>`AI Total Assigned (copy)_0605597422379015` | integer · Measure | LOD | Year | AI % Service Rate (ALL) | 1 |
| **B_Color_Icons**<br>`A_Color_Icons (copy)_2198799253688328` | string · Dimension | Basic | — | — | 1 |
| **Backlog Date**<br>`Update Date (copy)_2198799258259468` | date · Dimension | LOD | — | — | 3 |
| **C_Color_Icons**<br>`B_Color_Icons (copy)_1554760334643203` | string · Dimension | Basic | — | — | 3 |
| **Center**<br>`Calculation_0596262132809729` | spatial · Measure | Basic | — | — | 3 |
| **Closed 03**<br>`Closed AI Ticket (copy)_2217497971789825` | string · Dimension | Basic | Ticket Status, Ticket Sub-Status | — | 1 |
| **Closed AI Task-x1**<br>`Count Template 01 (copy)_2039097017262093` | integer · Measure | Basic | Ticket Status | Closed AI Task-x2 % | 3 |
| **Closed AI Task-x2 %**<br>`Count Template 02 (copy)_2039097017585678` | real · Measure | Basic | Closed AI Task-x1 | — | 3 |
| **Closed AI Ticket**<br>`Closed AI Task-x1 (copy)_2039097018421266` | boolean · Dimension | Basic | Ticket Processor Status | — | 1 |
| **Closed Escalation**<br>`Closed AI Task (copy)_2039097021562901` | string · Dimension | Basic | Ticket Processor Status | — | 1 |
| **Closed HML 03**<br>`Closed 03 (copy)_1450872307433472` | string · Dimension | Basic | Severity (group), Ticket Processor Status Group | — | 1 |
| **Color Legend**<br>`Calculation_1504722419068929` | string · Dimension | Basic | AI Completed, Ticket Processor Status Group, Severity | x | 3 |
| **CY**<br>`Calculation_1694662641647627` | integer · Dimension | Basic | — | CY-1, CY-2, CY-3, CY-4 | 4 |
| **CY-1**<br>`CY (copy)_1694662641844236` | integer · Dimension | Basic | CY | — | 1 |
| **CY-2**<br>`CY-1 (copy)_1694662641954829` | integer · Dimension | Basic | CY | — | 1 |
| **CY-3**<br>`CY-2 (copy)_1694662641987598` | integer · Dimension | Basic | CY | — | 1 |
| **CY-4**<br>`CY-3 (copy)_1694662642774031` | integer · Dimension | Basic | CY | — | 1 |
| **Date**<br>`Calculation_1694662622236672` | date · Dimension | Basic | — | MY, Whitespace Show\|Hide, Year, Year T\|F | 72 |
| **Details T\|F**<br>`Metrics T\|F (copy)_1554760341999643` | boolean · Dimension | Basic | — | — | 2 |
| **Fixed % of Total**<br>`Calculation_1450872391180302` | real · Measure | Basic | Fixed Total Tickets | — | 9 |
| **Fixed Total Tickets**<br>`Calculation_1450872390045709` | integer · Measure | LOD | Year T\|F | Fixed % of Total | 9 |
| **Group Serverity-H**<br>`Group Status-All (copy)_0235571782746114` | string · Dimension | Basic | Ticket Processor Status Group | — | 1 |
| **Group Serverity-L**<br>`Group Serverity-All (copy)_1694662635536390` | string · Dimension | Basic | Ticket Processor Status Group | — | 1 |
| **Group Serverity-M**<br>`Group Serverity-All (copy) (copy)_1694662635544583` | string · Dimension | Basic | Ticket Processor Status Group | — | 1 |
| **Human Ticket Days**<br>`Calculation_1694662658682903` | integer · Measure | Basic | — | — | 1 |
| **Metric-P T\|F**<br>`Who-AI T\|F (copy)_0694393118048266` | boolean · Dimension | Basic | — | — | 2 |
| **Metric-Se T\|F**<br>`Metric-F T\|F (copy) (copy)_0694393118322700` | boolean · Dimension | Basic | — | — | 2 |
| **Metric-St T\|F**<br>`Metric-F T\|F (copy 2)_0694393118322701` | boolean · Dimension | Basic | — | — | 2 |
| **Metrics T\|F**<br>`Status-AI T\|F (copy)_1554760341860378` | boolean · Dimension | Basic | — | — | 2 |
| **No Action 03**<br>`Closed 03 (copy)_2217497972760578` | string · Dimension | Basic | Ticket Sub-Status, Ticket Status | — | 1 |
| **Open 03**<br>`No Action 03 (copy)_2217497972985860` | string · Dimension | Basic | Ticket Processor Status Group, Ticket Sub-Status | — | 1 |
| **Open AI HML 03**<br>`Open Escalation HML 03 (copy)_1450872316493826` | string · Dimension | Basic | Severity (group), Ticket Processor Status Group | — | 1 |
| **Open AI Ticket**<br>`Closed AI Task (copy)_2039097019596820` | boolean · Dimension | Basic | Ticket Processor Status | — | 1 |
| **Open Escalation**<br>`Closed Escalation (copy)_2039097022717974` | string · Dimension | Basic | Ticket Processor Status | — | 1 |
| **Open Escalation HML 03**<br>`Closed HML 03 (copy)_1450872315863041` | string · Dimension | Basic | Severity (group), Ticket Processor Status Group | — | 1 |
| **Pending 03**<br>`Open 03 (copy)_2217497973821445` | string · Dimension | Basic | Ticket Processor Status Group, Ticket Sub-Status | — | 1 |
| **Placement**<br>`Calculation_1694662649491475` | integer · Dimension | Basic | — | — | 3 |
| **Processor A\|H**<br>`Random Number Calc (copy)_1554760352845856` | string · Dimension | Basic | Current Support | — | 2 |
| **Random Number 02**<br>`Random Number 02 (copy)_1537445531119617` | string · Dimension | Basic | — | — | 1 |
| **Status-Year 2022 T\|F**<br>`Status-Year 2023 T\|F (copy)_1694662645461009` | boolean · Dimension | Basic | — | — | 1 |
| **Status-Year 2023 T\|F**<br>`Status-Year 2024 T\|F (copy)_1694662645297168` | boolean · Dimension | Basic | — | — | 1 |
| **Status-Year 2024 T\|F**<br>`Status-Open T\|F (copy)_1554760338513928` | boolean · Dimension | Basic | — | — | 1 |
| **Status-Year 2025 T\|F**<br>`Calculation_1554760338288647` | boolean · Dimension | Basic | — | — | 1 |
| **Status-Year 2026 T\|F**<br>`Status-All T\|F (copy)_1554760339640335` | boolean · Dimension | Basic | — | — | 2 |
| **Ticket Age < 31**<br>`Ticket Age < 60 (copy)_0801322700156929` | boolean · Dimension | Basic | — | — | 1 |
| **Ticket Age >100**<br>`Ticket Age (copy)_2039097010597895` | boolean · Dimension | Basic | — | — | 1 |
| **Ticket H Sub-Status**<br>`Ticket Sub-Status (copy)_1694662633951237` | string · Dimension | Basic | Ticket Sub-Status | — | 1 |
| **Update Date**<br>`Calculation_2198799254478857` | date · Dimension | Basic | — | — | 3 |
| **Year**<br>`Year (copy)_1694662647898130` | integer · Dimension | Basic | Date | AI Total Assigned, AI Total Assigned (ALL) | 14 |
| **Year T\|F**<br>`Date (copy)_1694662622457857` | boolean · Dimension | Basic | Date | Fixed Total Tickets | 72 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| **P-Metric**<br>`P-Who (copy)_0694393114480646` | string · list | `"Processor"` | list of 3 — "Processor", "Status", "Severity" |
| **P-View**<br>`P-Who (copy)_1554760341561366` | string · list | `"Metrics"` | list of 2 — "Metrics", "Details" |
| **P-Year**<br>`Parameter 1694662624628738` | real · list | `2026.0` | list of 7 — 2021.0, 2022.0, 2023.0, 2024.0, 2025.0, 2026.0 |
| **P-Status**<br>`Parameter 1` | string · list | `"All"` | list of 3 — "All", "Open", "Closed" |

### How the numbers are computed

**Level-of-detail expressions — 4.** These fix their own level of detail, so they return the same value regardless of what dimensions are in the view:

> `AI Total Assigned`, `AI Total Assigned (ALL)`, `Fixed Total Tickets`, `Backlog Date`

## Appendix A — Calculated fields excluded from this documentation

8 calculated fields exist in the data pane but are not referenced by any worksheet or dashboard
logic:

`Human Count` · `Whitespace Show|Hide` · `x` · `U_Circle` · `P-Status Toggle T|F` ·
`Group Status-All` · `Random Number 01` · `MY`

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas and settings are quoted verbatim from the workbook
definition; rendered figures are read from the published view.*
