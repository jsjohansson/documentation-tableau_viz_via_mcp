# HBCU Catalog — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `HBCU Catalog \| B2VB` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/B2VBHBCU/HBCU |
| Challenge | **#B2VB 2024 Week 03** — Build a Text Table |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `B2VBHBCU.twbx`, decompiled and parsed field-by-field |
| Explicit exclusion | 4 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic**. Listed in Appendix A. |

---

## 1. Executive summary

**HBCU Catalog** is a reference table of Historically Black Colleges and Universities in the
United States — one row per institution, carrying its crest, location, degrees offered, tuition,
gender ratio, population, campus setting and graduation rate.

The challenge was "Build a Text Table", and the workbook's answer is to build the *entire*
dashboard from **two worksheets**. Everything visible — crest, state silhouette, three tuition
bars, gender pie, population figures, campus icon and graduation badge — is composed inside a
single text-table sheet plus an icon sheet.

The tuition bars are the notable technique: like the author's *Average European University
Costs*, they are **strings of block characters** sized by a `LEFT(…, CEILING(…))` expression
rather than marks (§5.2).

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | **2** |
| Dashboards | 1 (`HBCU`) |
| Data sources | 1 (4 tables, **4 joins**) |
| Parameters | 1 |
| Calculated fields **used** | 15 named + 27 in-view ad-hoc |
| Calculated fields unused | 4 |
| Dashboard actions | 2 URL |
| Dynamic Zone Visibility bindings | 0 |
| Extract rows | 449 |
| Published size | 4,811,755 bytes (4.6 MB) |

### 1.2 As rendered

Ordered by **Established Year**, oldest first:

| Est. | Institution | Location | Degrees | In / Out / Room | Female | Population | Setting | Grad Rate |
|---|---|---|---|---|---:|---:|---|---:|
| 1837 | Cheyney University of Pennsylvania | Cheyney, PA | Master | $8,404 / $17,942 / $8,910 | 54 % | 1,808 | Online Options | 15 % |
| 1851 | University of the District of Columbia | Washington, DC | Master, Bachelor, Associate | $4,318 / $8,590 / $0 | 61 % | 5,518 | Campus Only | 35 % |
| 1854 | Lincoln University: Pennsylvania | Lincoln University, PA | Master, Bachelor | $8,984 / $13,222 / $8,004 | 58 % | 2,361 | Online Options | 45 % |
| 1856 | Central State University | Wilberforce, OH | Master, Bachelor | $5,672 / $12,648 / $8,484 | 46 % | 2,288 | Campus Only | 25 % |
| 1856 | Wilberforce University | Wilberforce, OH | Bachelor | $12,470 / $12,470 / $5,700 | 57 % | 1,180 | Campus Only | 41 % |
| 1857 | Harris-Stowe State University | St. Louis, MO | Bachelor | $4,776 / $9,500 / $9,360 | 65 % | 1,500 | Online Options | 20 % |

The dashboard scrolls through the full catalogue.

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `14027621` |
| LUID | `4f290411-bc53-4017-ab82-893b30d28b06` |
| Repository URL | `B2VBHBCU` |
| Default view | `HBCU` |
| Revision | 2.2 |
| First published | 2024-02-13 |
| Last published | 2024-02-15 |
| View count | 486 |
| Favourites | 3 |
| Sources | HBCU Connect & NCES |

**Published description**

> #B2VB 2024W03 | Build a Text Table | Historically Black Colleges and Universities
> #HBCU #Table

---

## 3. Data architecture

| Property | Value |
|---|---|
| Relations | 5 — a collection joining `Sheet1`, `CollegeNavigator_20240210_Use`, `Research 01`, `hbcu_spreadsheet` |
| Joins | **4** |
| Data source filters | None |
| Columns | 30 |
| Rows | 449 |

| Table | Supplies |
|---|---|
| `Sheet1` | College name, link, populations, mascot, tuition, gender split, online classes, room & board |
| `CollegeNavigator_20240210_Use` | NCES data — type, campus setting, campus housing, graduation rate, transfer-out rate, net price |
| `Research 01` | Display ordering and the two-part college-name split |
| `hbcu_spreadsheet` | City, state, website |

Four sources joined to assemble one catalogue row — the data assembly is most of the work in
this workbook.

---

## 4. Parameters

| Caption | Type | Domain | Default | Role |
|---|---|---|---|---|
| **Order By:** | string list | **11 members** | `1` (Established Year) | Sort order for the catalogue |

Eleven sort options including Established Year, In-State Tuition, Out-of-State Tuition, Room &
Board and others.

---

## 5. Calculated fields in use

15 named calculations plus 27 in-view ad-hoc.

### 5.1 The sort switch

```
**Order By (Case)  CASE [Order By:]
                     WHEN '1' THEN -[Founded]
                     WHEN '2' THEN 0
                     WHEN '3' THEN [In-State Tuition]
                     WHEN '4' THEN [Out-of-State Tuition]
                     WHEN '5' THEN [Room & Board]
                     … END
```

Returns whichever measure the reader chose, as a single sort key. Note `-[Founded]` — negating
the year sorts oldest-first, which is the published default.

### 5.2 Text-character tuition bars

```
*Fixed Max Total  { FIXED : MAX([In-State Tuition]) }

*Bar Builder - In-State Tuition
    LEFT("▇▇▇▇▇▇▇▇▇▇▇", CEILING(SUM([In-State Tuition]) / SUM([*Fixed Max Total]) * 10))

*Bar Builder - Out-of-State Tuition   same construction on [Out-of-State Tuition]
*Bar Builder - Room & Board           same construction on [Room & Board]
```

Each cost is expressed as a fraction of the dataset maximum, scaled to 0–10, rounded up, and
used to take that many block characters from an 11-character string. The result is three
coloured bars (green / amber / gold) inside a text table, with no mark-based chart involved.

All three share the **same denominator** (`*Fixed Max Total`, the maximum in-state tuition), so
the three bars in a row are directly comparable — out-of-state fees routinely overflow further
than in-state, which is exactly the comparison the design wants to show.

### 5.3 Name and title handling

```
CollegeName-All Rows  IF [CollegeName-Rows] = 2 THEN [CollegeName-P1] + " " + [CollegeName-P2]
                      ELSE [CollegeName-P1] END

College Title  If     CONTAINS([College],'University') THEN 'University'
               ELSEIF CONTAINS([College],'College')    THEN 'College'
               ELSE 'School' END
```

`CollegeName-All Rows` reassembles a college name that was pre-split into two parts in the
`Research 01` table — a manual word-wrap so long institution names break at a chosen point
rather than wherever the column edge falls.

### 5.4 Degree flags

```
Degree A  IF CONTAINS([Degrees], 'Associate') then 'Associate' END
Degree B  IF CONTAINS([Degrees], 'Bach')     then 'Bachelor'  END
Degree M  IF CONTAINS([Degrees], 'Master')   then 'Master'
          ELSEIF [Degrees] = 'Graduate Level' then 'Master'   END
Degree D  IF CONTAINS([Degrees], 'Doctor')   then 'Doctoral'  END
```

The source stores degrees as a single free-text string. Four `CONTAINS` tests split it into
independent flags so the Degrees column can list each level on its own line. `Degree M` carries
an extra branch because the source sometimes writes "Graduate Level" instead of "Master".

### 5.5 Composite description

```
School Type-Campus  [Type (Year)] + ' ' + [Type (Private/Public)]
                    + ' School in a ' + [Campus setting (Size)]
                    + ' ' + [Campus setting (Location)]
```

Assembles a readable sentence — e.g. "4-year Public School in a Small Suburban" — from four
separate NCES columns.

```
Total Population           SUM([Grad Population]) + SUM([Undergrad Population])
Online Classes Description [Online Classes]
1                          1
```

---

## 6. Worksheet specifications

**Two worksheets.**

| Sheet | Role |
|---|---|
| `Text Table` | The entire catalogue — every column, bar, pie and badge |
| `Icon` | The institution crest images |

Everything in the published view except the crests is composed inside one text table: the state
silhouette, the three character bars, the gender pie, the population block, the campus-setting
icon and the graduation-rate badge. This is the challenge brief taken at its word.

---

## 7. Dashboard specification

| Property | Value |
|---|---|
| Name | `HBCU` |
| Canvas | 1600 × 900 px, fixed |
| Ground | Off-white |
| Masthead | Dark navy band with a gold `HBCU` wordmark |

**Regions**

```
Masthead   gold "HBCU" wordmark on navy · "Historically Black Colleges and Universities Catalog"
           "United States | In Order by Established Year" · building icon
Header row Order by: control · Degrees · Websites · Tuition (In/Out/Room key) ·
           Gender Ratio (Female/Male key) · Population · Class Settings · Graduation Rate
Catalogue  scrolling rows, one per institution
Footer     "Project Back to Viz Basics | Challenge 2024 Week 03 - Build a Text Table |
            Designed by John Johansson | Sources HBCU Connect & NCES"
```

---

## 8. Interactivity

### 8.1 URL actions (2)

| Caption | Target |
|---|---|
| `1) <College>'s Website` | `<[Website]>` — the institution's own site |
| `2) Webpage on HBCU Connect` | `<[Link]>` — its HBCU Connect profile |

Both are **parameterised by row**, so the link chip in each row navigates to that institution.
The first action's caption is itself a calculation, so the menu reads "Cheyney University of
Pennsylvania's Website" rather than a generic label.

### 8.2 Native control

The `Order By:` parameter is exposed at the top-left of the header row.

---

## 9. Design system

| Token | Use |
|---|---|
| Dark navy | Masthead band, institution names |
| Gold | `HBCU` wordmark, graduation-rate badges, room & board bars |
| Green | In-state tuition bars |
| Amber | Out-of-state tuition bars |
| Pink | Female share of the gender pie |
| Blue | Male share |
| Tan | State silhouettes |
| Off-white | Ground, row bands |

**Iconography** — institution crests (from the `Icon` sheet), state silhouettes, link chips,
globe/building campus-setting icons, and mortarboard graduation badges.

---

## 10. Rebuild / maintenance runbook

**Refreshing the data**

1. All four source tables must be refreshed together and keep their join keys — the catalogue
   row assembles from all four.
2. `CollegeName-P1` / `CollegeName-P2` / `CollegeName-Rows` are **manually split** name parts in
   the `Research 01` table. A new institution needs its name split by hand, and `CollegeName-Rows`
   set to 1 or 2, or the name will not render correctly.

**The character bars**

All three `*Bar Builder` fields divide by `*Fixed Max Total` (maximum in-state tuition) and take
from an **11-character** string scaled to 10. If out-of-state fees ever exceed 11× the maximum
in-state fee, the bar will clip silently. Widening means changing the source string, the `* 10`
multiplier, and all three fields together.

**Adding a sort option**

Add a member to `Order By:` and a matching `WHEN` branch to `**Order By (Case)`. Remember that
descending sorts are expressed by negating the measure, as `-[Founded]` does.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | Sheet1 (HBCU) |
| Connection class | `federated` |
| Tables / relations | 5 · joins: 4 |
| Extract rows | 449 |
| Physical columns materialised | 38 |
| Source fields in data pane | 29 |
| Calculated fields (used / total) | 15 / 19 |
| Parameters | 1 |
| Data source filters | 0 (none) |

### Source fields

29 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Institution

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **College** | string · Dimension | 109 | yes | 2 | Institution name. |
| **Campus setting** | string · Dimension | 12 | yes | 1 | NCES campus setting, such as `City: Midsize`. Split into size and locale parts. |
| **Campus setting (Location)** | string · Dimension | — | — | 1 | Locale half of the campus setting: City, Suburb, Town or Rural. |
| **Campus setting (Size)** | string · Dimension | — | — | 1 | Size half of the campus setting: Large, Midsize or Small. |
| **Founded** | integer · Measure (Sum) | 66 | yes | 1 | Year the institution was founded. |
| **Type (Private/Public)** | string · Dimension | — | — | 1 | Control: public or private. |
| **Type (Year)** | string · Dimension | — | — | 1 | Programme length: two-year or four-year. |
| **Type (sub-text)**<br>`Type (Private/Public) (copy)_554787233606295553` | string · Dimension | — | — | 0 | Remaining descriptive text from the type string. *(not used in any sheet)* |

#### Population

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Grad Population** | integer · Measure (Sum) | 57 | yes | 1 | Graduate enrolment. |
| **Percent Female**<br>`Percent Women` | real · Measure (Sum) | 47 | yes | 1 | Share of the student body that is female. |
| **Percent Male**<br>`Percent Men` | real · Measure (Sum) | 47 | yes | 1 | Share of the student body that is male. |
| **Undergrad Population** | integer · Measure (Sum) | 106 | yes | 1 | Undergraduate enrolment. |
| **Student population** | integer · Measure (Sum) | 106 | yes | 0 | Enrolment as reported by NCES. *(not used in any sheet)* |
| **Undergraduate students** | integer · Measure (Sum) | 103 | yes | 0 | Undergraduate enrolment as reported by NCES. *(not used in any sheet)* |

#### Costs

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **In-State Tuition** | integer · Measure (Sum) | 109 | yes | 1 | Annual in-state tuition. The shared denominator for all three cost bars. |
| **Out-of-State Tuition** | integer · Measure (Sum) | 107 | yes | 1 | Annual out-of-state tuition. |
| **Room & Board** | integer · Measure (Sum) | 98 | yes | 1 | Annual room and board cost. |

#### Geography

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **State (Research 01)** | string · Dimension | — | — | 1 | State name. Arrives from the **Research 01** table in the join, duplicating the same column on the left-hand side. |
| **State** | string · Dimension | 26 | yes | 0 | State name. *(not used in any sheet)* |
| **Zip** | string · Dimension | 104 | yes | 0 | Postal code. *(not used in any sheet)* |

#### Layout

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **CollegeName-Rows** | integer · Measure (Sum) | 2 | yes | 1 | How many lines the institution name needs, 1 or 2. |
| **Order** | integer · Measure (Sum) | 109 | yes | 1 | Display ordering key. |

#### Location

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **City (Research 01)** | string · Dimension | — | — | 1 | City of the shipping address. Arrives from the **Research 01** table in the join, duplicating the same column on the left-hand side. |
| **City** | string · Dimension | 92 | yes | 0 | City of the shipping address. *(not used in any sheet)* |

#### Outcomes

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Graduation Rate** | integer · Measure (Sum) | 50 | yes | 1 | Share of the cohort graduating. |
| **Transfer-Out Rate** | integer · Measure (Sum) | 34 | yes | 0 | Share transferring out. *(not used in any sheet)* |

#### Provision

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Online Classes** | string · Dimension | 2 | yes | 1 | Whether online classes are offered. |
| **Campus housing** | string · Dimension | 3 | yes | 0 | Whether the institution provides campus housing. *(not used in any sheet)* |

#### Identity

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Type** | string · Dimension | 6 | yes | 1 | Record classification. |

### Calculated fields in use

15 of the workbook's 19 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| ****Order By (Case)**<br>`Calculation_1775544207547928578` | real · Dimension | Basic | Founded, In-State Tuition, Out-of-State Tuition, Room & Board, Percent Female, Percent Male … | — | 1 |
| ***Bar Builder - In-State Tuition**<br>`Calculation_1939925593318551557` | string · Measure | Basic | In-State Tuition, *Fixed Max Total | — | 1 |
| ***Bar Builder - Out-of-State Tuition**<br>`*Bar Builder - In-State Tuition (copy)_1939925593322729481` | string · Measure | Basic | Out-of-State Tuition, *Fixed Max Total | — | 1 |
| ***Bar Builder - Room & Board**<br>`*Bar Builder - In-State Tuition (copy)_1939925593323184139` | string · Measure | Basic | Room & Board, *Fixed Max Total | — | 1 |
| ***Fixed Max Total**<br>`Calculation_1939925593318359044` | integer · Measure | LOD | In-State Tuition | *Bar Builder - In-State Tuition, *Bar Builder - Out-of-State Tuition, *Bar Builder - Room & Board | 1 |
| **1**<br>`Calculation_2385219007378313216` | integer · Measure | Basic | — | — | 2 |
| **College Title**<br>`Calculation_2281917690527993858` | string · Dimension | Basic | College | — | 1 |
| **CollegeName-All Rows**<br>`CollegeName-P1 (copy)_747034642557272064` | string · Dimension | Basic | CollegeName-Rows | — | 1 |
| **Degree A**<br>`Calculation_554787233609523202` | string · Dimension | Basic | — | — | 1 |
| **Degree B**<br>`Degree A (copy)_554787233610407939` | string · Dimension | Basic | — | — | 1 |
| **Degree D**<br>`Degree M (copy)_554787233610678278` | string · Dimension | Basic | — | — | 1 |
| **Degree M**<br>`Degree B (copy)_554787233610539012` | string · Dimension | Basic | — | — | 1 |
| **Online Classes Description**<br>`Online Classes (copy)_2500623747447681027` | string · Dimension | Basic | Online Classes | — | 1 |
| **School Type-Campus**<br>`Calculation_1901926472155508736` | string · Dimension | Basic | Type (Year), Type (Private/Public), Campus setting (Size), Campus setting (Location) | — | 1 |
| **Total Population**<br>`Calculation_1939925593302745089` | integer · Measure | Basic | Grad Population, Undergrad Population | — | 1 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| **Order By:**<br>`Parameter 1` | string · list | `"1"` | list of 11 — "1" → Established Year, "2" → Name, "3" → In-State Tuition, "4" → Out-State Tuition, "5" → Room & Board, "6" → Percent Female |

### How the numbers are computed

**Level-of-detail expressions — 1.** These fix their own level of detail, so they return the same value regardless of what dimensions are in the view:

> `*Fixed Max Total`

## Appendix A — Calculated fields excluded from this documentation

4 calculated fields exist in the data pane but are not referenced by any worksheet or dashboard
logic:

`**Case Ranking (copy)` · `*Fixed Total - Room & Board` · `*Fixed Total - Out-of-State` · `4`

The two unused `*Fixed Total` fields are earlier per-measure denominators, superseded by the
single shared `*Fixed Max Total`.

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas and settings are quoted verbatim from the workbook
definition; rendered figures are read from the published view.*
