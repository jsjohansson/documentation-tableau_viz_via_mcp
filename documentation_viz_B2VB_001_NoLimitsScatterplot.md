# No Limits Scatter Plot — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `No Limits Scatterplot \| B2VB` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/B2VB_001/NoLimitsScatterplot |
| Challenge | **#B2VB 2023 Week 21** |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `B2VB_001.twbx`, decompiled and parsed field-by-field |
| Explicit exclusion | 12 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic**. Listed in Appendix A. |

---

## 1. Executive summary

**No Limits Scatter Plot** is a **template**, not a finished analysis. Its stated purpose,
printed on the dashboard itself, is to demonstrate a technique: *"Have you ever needed to build
out the same Viz using various fields or charts? This template Viz can help you understand how
to dynamically change measures in marks, rows/columns shelves, and even charts by using simple
parameter functions."*

Every encoding on the chart is parameter-driven. **X**, **Y**, **Size**, **Colour**, **Shape**,
a dimension filter and a measure filter are each resolved by a `CASE` statement reading its own
parameter — twelve parameters in total — so the reader can rebuild the scatterplot into a
different analysis without touching the workbook.

The dataset is the author's IMDb movies-and-people extract (5,176,667 rows), used here purely as
a substrate for the demonstration.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | **3** |
| Dashboards | 1 (`No Limits Scatterplot`) |
| Data sources | 1 (single extract, no joins) |
| Parameters | **12** |
| Calculated fields **used** | 19 named + 28 in-view ad-hoc |
| Calculated fields unused | 12 |
| Dashboard actions | **0** |
| Dynamic Zone Visibility bindings | 0 |
| Extract rows | **5,176,667** |
| Global font | Tw Cen MT |
| Published size | 1,129,135,961 bytes (**1.1 GB**) |

### 1.2 As rendered

| Shelf | Current selection |
|---|---|
| X | **Years Since Release** (0 – 105) |
| Y | **IMDb Rating** (1 – 9) |
| Size | –None– |
| Colour | **Oscar** — Best Picture (amber) · Nomination (red) · None (blue) |
| Shape | ✦ |
| Filter One | Genre = Action |
| Filter Two | Votes, 10,000 to 2,791,424 and null values |
| Bonus chart | Scatterplot |

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Repository URL | `B2VB_001` |
| Default view | `No Limits Scatterplot` |
| View count | 391 |
| Favourites | 2 |
| Credits | Powered by Tableau + IMDb · Created for Back 2 Viz Basics 2023 Week21 |

---

## 3. Data architecture

| Property | Value |
|---|---|
| Data source | IMDb movies and people extract |
| Relations | 1 table (`Extract`) |
| Joins | **None** |
| Data source filters | None |
| Extract rows | **5,176,667** |
| Columns | 30+ |

Title × person grain — the same IMDb model behind the author's movie workbooks. The full 5.18M
row extract accounts for essentially all of the 1.1 GB package; the dashboard itself is three
sheets.

---

## 4. Parameters

Twelve parameters — eight driving the chart, four inherited.

| Caption | Domain | Default | Drives |
|---|---|---|---|
| **+X** | 5 members | `4` | X axis measure |
| **+Y** | 5 members | `1` | Y axis measure |
| **+Size** | 6 members | `13` | Size encoding |
| **+Color** | 7 members | `10` | Colour encoding |
| **+Shape** | 7 members | `✦` | Mark shape |
| **+Text Filter** | 7 members | `7` | Which dimension filters |
| **+Numeric Filter** | 5 members | `2` | Which measure filters |
| **+Unit** | 1 member | `1` | Level of detail |
| **+ Bonus Chart** | 2 members | `S` | Scatterplot or Bar |
| `Select Movie Genre` | 21 members | `Sci-Fi` | Inherited |
| `Time Chart Type` | 2 members | `1.` | Inherited |
| `Production Company` | any | `Marvel Studios` | Inherited |

---

## 5. Calculated fields in use

19 named calculations plus 28 in-view ad-hoc. The workbook is essentially **one pattern repeated
seven times**.

### 5.1 The swap pattern

Every shelf is resolved by a `CASE` over its parameter:

```
+X Variable  CASE [+X] WHEN '1' THEN MAX([IMDB Rating])
                       WHEN '2' THEN MAX([Number Of Votes])
                       WHEN '3' THEN MAX([Year of Release])
                       WHEN '4' THEN MAX([+Years Since Released])
                       WHEN '5' THEN … END

+Y Variable      same five branches over [+Y]
+Size Changer    six branches over [+Size]
+Color Changer   CASE [+Color] WHEN '7'  THEN [+Primary Genre]
                               WHEN '8'  THEN [Certificate (US)]
                               WHEN '9'  THEN [Certificate (GB)]
                               WHEN '10' THEN [+Best Picture Text] … END
+Shape Changer   CASE [+Shape] WHEN '✦' THEN '✦' WHEN '●' THEN '●' WHEN '○' THEN '○'
                               WHEN '■' THEN '■' WHEN '□' THEN '□'
                               WHEN '▲' THEN '▲' WHEN '△' THEN '△' END
+Filter One (T)  seven dimension branches over [+Text Filter]
+Filter Two (N)  five measure branches over [+Numeric Filter]
+Unit            CASE [+Unit] WHEN '1' THEN STR([Title Id])
                              WHEN '2' THEN [+Primary Genre]
                              WHEN '3' THEN STR([Year of Release]) END
+Unit Description  the label counterpart of +Unit
+Bonus Chart     CASE [+ Bonus Chart] When 'S' then 'S' When 'B' then 'B' END
```

Each is dropped on its shelf once; the parameter then decides what it means. `+Shape Changer` is
the neatest of the set — it returns the glyph character itself, so the mark's shape *is* the
calculation's output.

### 5.2 The null guards

The one piece of genuine defensive engineering:

```
+Excluder (Number)  IF     ISNULL([+X Variable])    then 'Exclude'
                    ELSEIF ISNULL([+Y Variable])    then 'Exclude'
                    ELSEIF ISNULL([+Size Changer])  then 'Exclude'
                    ELSEIF ISNULL(…)                then 'Exclude'
                    ELSE 'Include' END

+Excluder (Text)    IF     ISNULL([+Color Changer]) then 'Exclude'
                    ELSEIF ISNULL([+Filter One (T)]) then 'Exclude'
                    ELSE 'Include' END
```

Because any shelf can be pointed at any field, a swap can easily produce nulls — a film with no
certificate, a person row with no rating. These two fields catch that across all the swappable
encodings at once and exclude the row, so the chart never silently misplots. Without them the
template would break the first time a reader chose an incomplete field combination.

### 5.3 Supporting fields

```
+Years Since Released  INT(YEAR(TODAY()) - [Year of Release])
+Primary Genre         [Genres (1st)]
+Best Picture Score    IF [Best Picture] = 'Winner' then 2 ELSEIF 'Nominated' then 1 ELSE 0 END
+Best Picture Text     IF [+Best Picture Score] = 2 then 'Oscar Best Picture'
                       ELSEIF = 1 then 'Oscar Nomination' ELSE 'None' END
*Txt_Tagline           IF ISNULL([Tagline]) then '' ELSE '◄'+REGEXP_REPLACE([Tagline],'"','')+'►' END
Number of titles       // a count distincy of the number of Titles in the dataset
                       COUNTD([Title Id])
Number of people       COUNTD([Person Name ID])
```

`+Years Since Released` uses `TODAY()`, so the X axis re-bases every time the dashboard is
opened — the 0–105 range shown will widen as time passes.

---

## 6. Worksheet specifications

Three worksheets.

| Sheet | Role |
|---|---|
| `A-Scatterplot` | The main chart — `+X Variable` / `+Y Variable` with all four encodings |
| `B-Bonus Bar Chart` | The alternate chart form, selected by `+ Bonus Chart` |
| `C-Background` | The dark canvas plate |

---

## 7. Dashboard specification

| Property | Value |
|---|---|
| Name | `No Limits Scatterplot` |
| Canvas | 1600 × 800 px, fixed |
| Ground | Dark slate blue |

**Regions**

```
Masthead   "No Limits Scatter Plot" · "Swapping Measures in the Marks & Shelves using Parameters"
           Designed by John Johansson · Powered by Tableau + IMDb ·
           Created for Back 2 Viz Basics 2023 Week21 · #DataPlusMovies #B2VB
Left rail  the explanatory paragraph, then the parameter controls in three groups:
             X · Y · Size · Color · Shape
             Filter Type One (Dimension Filter) · Filter Type Two (Measure Filter)
             a summary block echoing the current selections + Bonus Chart Swap
Main       the scatterplot
Right      Color legend — Oscar Best Picture · Oscar Nomination · None
Footer     axis label "Years Since Release"
```

The left rail doubles as documentation: the summary block at its foot prints the active
selections (`X: Years Since Release`, `Y: IMDb Rating`, `Size: –None–`, `Color: Oscar`,
`Shape: ✦`, `Filter One: Genre`, `Filter Two: Votes`), so the reader can always see what the
chart currently means.

---

## 8. Interactivity

**No actions and no Dynamic Zone Visibility.** All twelve parameters are exposed as native
controls in the left rail. Interactivity is entirely parameter-driven, which is the point of the
template.

---

## 9. Design system

| Token | Use |
|---|---|
| Dark slate blue | Dashboard ground and chart plate |
| Amber / gold | Oscar Best Picture marks, headings, the instruction text |
| Red | Oscar Nomination marks |
| Steel blue | Marks with no Oscar |
| White | Titles, axis labels |
| Mid grey | Body text, gridlines |

**Typography** — Tw Cen MT, set at workbook level — the same font as the author's original
*Movies Across Time & Space*, from which the IMDb extract and several calculations are inherited.

---

## 10. Rebuild / maintenance runbook

**Adding a field to a shelf's options**

Three edits per shelf, and they must stay in step:

1. Add a member to the parameter (`+X`, `+Y`, `+Size`, `+Color`, `+Shape`, `+Text Filter`,
   `+Numeric Filter`).
2. Add the matching `WHEN` branch to the resolver (`+X Variable`, `+Color Changer`, …).
3. **Add the field to the relevant `+Excluder`** — otherwise a null in the new field will pass
   the guard and misplot.

**Measures and dimensions cannot be mixed**

`+X Variable`, `+Y Variable` and `+Size Changer` all return aggregated measures;
`+Color Changer`, `+Filter One (T)` and `+Unit` return dimensions. A `CASE` cannot return both
types, so a new option must match its resolver's type.

**The TODAY() dependency**

`+Years Since Released` recalculates from `TODAY()`. Any published screenshot of the X axis ages
— which is expected behaviour for this template, but worth knowing before treating an axis range
as fixed.

**Package size**

At 1.1 GB the workbook is almost entirely its 5.18M-row IMDb extract. As a *template* the data is
incidental; swapping in a small sample dataset would reduce the package by several orders of
magnitude without changing what it demonstrates.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | IMDB movies and people Extract |
| Connection class | `federated` |
| Tables / relations | 1 · joins: 0 |
| Extract rows | 5,176,667 |
| Physical columns materialised | 33 |
| Source fields in data pane | 30 |
| Calculated fields (used / total) | 19 / 31 |
| Parameters | 12 |
| Data source filters | 0 (none) |

### Source fields

30 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Title identity

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Title Id**<br>`titleId` | string · Dimension | 886,118 | yes | 3 | IMDb title key in `tt0000000` form. The film-level key — all film counts use `COUNTD`. The unique identifier used by IMDB |
| **Year of Release** | integer · Dimension | 124 | yes | 3 | Release year. Drives the radial angle, timeline position and decade grouping. The year of the earliest release of this title globally. |
| **Title** | string · Dimension | 769,268 | yes | 2 | Film title as displayed on IMDb. The original title text of the title, normally what the title is known as in its original country of release. |

#### Ratings

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **IMDB Rating**<br>`imdb rating` | real · Measure (Sum) | 118 | yes | 3 | IMDb weighted user rating, 1–10. The colour encoding across the radial and timeline views. The IMDb Rating for the title. The rating is between 1 and 10 and given to one decimal place. |
| **Number Of Votes**<br>`number of votes` | integer · Measure (Sum) | 51,343 | yes | 3 | Count of IMDb user votes. The size encoding everywhere, and the quality gate for inclusion. A single IMDb user can cast a maximum of one vote. This field can be missing when we do not yet have an IMDb rating for the title in question.   This can occur either because it does not yet have enough votes, or it has not yet been released.   A TV series rating is not the weighted average of the ratings of individual episodes. Instead, customers vote separately for the rating of the series as a whole via each title’s series page. |

#### Title detail

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Certificate (GB)** | string · Dimension | 19 | yes | 2 | UK BBFC certificate — U, PG, 12A, 15, 18. What rating certificate was this film given in Great Britain? |
| **Certificate (US)** | string · Dimension | 30 | yes | 2 | US content rating — G, PG, PG-13, R and so on. What rating certificate was this film given in USA? |
| **Genres (1st)**<br>`genres (1st)` | string · Dimension | 24 | yes | 2 | First-listed genre — IMDb's primary classification. The |
| **Runtime (Minutes)**<br>`runtime (minutes)` | integer · Measure (Sum) | 476 | yes | 2 | Running time in minutes. The running time of this title in minutes. |
| **Color**<br>`color` | string · Dimension | 5 | yes | 0 | Colour information — Color, Black and White, or a combination. Was the film black & white, colour, or a mix? *(not used in any sheet)* |
| **Genres (2nd)**<br>`genres (2nd)` | string · Dimension | 30 | yes | 0 | Second-listed genre. The *(not used in any sheet)* |
| **Genres (3rd)**<br>`genres (3rd)` | string · Dimension | 28 | yes | 0 | Third-listed genre. The *(not used in any sheet)* |
| **Genres (full list)** | string · Dimension | 12,249 | yes | 0 | Comma-separated genre list, e.g. `Action,Adventure,Sci-Fi`. Tested with `CONTAINS` rather than split. The *(not used in any sheet)* |
| **Language** | string · Dimension | 167 | yes | 0 | Primary language of the film. This is the primary language spoken in the title *(not used in any sheet)* |

#### Origin

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Continent** | string · Dimension | 6 | yes | 2 | Continent of production. |
| **Country** | string · Dimension | 245 | yes | 2 | Country of production. This is the country of the primary production company associated with this title. ( |

#### Awards

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Best Picture** | string · Dimension | 3 | yes | 2 | Academy Award Best Picture status: `Winner`, `Nominated`, or null. Was the picture |

#### People

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Person Name ID** | string · Dimension | 978,071 | yes | 1 | IMDb person key in `nm0000000` form. Basis of distinct people counts. The IMDB id for the person. |
| **Billing (position in cast list)** | integer · Measure (Sum) | 51 | yes | 0 | Cast-list order; 1 is top billing. The Billing represents the position in the cast list this person appeared. Number 1 represents top billing. For this dataset we only have the top 50 people in each title. *(not used in any sheet)* |
| **Person Name** | string · Dimension | 967,432 | yes | 0 | Cast or crew member's name. *(not used in any sheet)* |
| **What did they do ?** | string · Dimension | 3 | yes | 0 | Credit role — `director`, `actor`, `actress`. The row's reason for existing. What was their job in the movie. We include only *(not used in any sheet)* |
| **Who did they play ?** | string · Dimension | 640,782 | yes | 0 | Character name for acting credits. The name of they character they played in this movie/show *(not used in any sheet)* |

#### Text

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Tagline**<br>`tagline` | string · Dimension | 371,871 | yes | 1 | Marketing tagline. Wrapped in guillemets by `*Txt_Tagline`. A tagline is a short description or comment on a title that is often displayed on posters. |

#### Production

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Production Companies (1st)** | string · Dimension | 203,605 | yes | 0 | First-listed production company. The *(not used in any sheet)* |
| **Production Companies (2nd)** | string · Dimension | 123,298 | yes | 0 | Second-listed production company. The *(not used in any sheet)* |
| **Production Companies (3rd)** | string · Dimension | 78,023 | yes | 0 | Third-listed production company. The *(not used in any sheet)* |
| **Production Companies (List)** | string · Dimension | 397,824 | yes | 0 | Comma-separated list of production companies. The full list of Production Companies.   Where a movie has multiple Production Companies, there is no specific order in where they appear in the list. For example. *(not used in any sheet)* |

#### Links

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Image Url (Title)**<br>`image_url (title)` | string · Dimension | 791,708 | yes | 0 | Poster image URL, served from Amazon media. A URL linking to the primary image associated with this title, such as a movie poster or still frame. *(not used in any sheet)* |
| **IMDB Url (Person)**<br>`imdbUrl (Person)` | string · Dimension | 978,071 | yes | 0 | Deep link to the person's IMDb page. A full URL to see this person on www.imdb.com. *(not used in any sheet)* |
| **IMDB Url (title)**<br>`imdbUrl (title)` | string · Dimension | 886,118 | yes | 0 | Deep link to the film's IMDb page. A full URL to see the name or title on www.imdb.com. *(not used in any sheet)* |

### Calculated fields in use

19 of the workbook's 31 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| ***Txt_Tagline**<br>`Calculation_1261570865762828296` | string · Dimension | Basic | Tagline | — | 1 |
| **+Best Picture Score**<br>`Calculation_1523061119493234693` | integer · Measure | Basic | Best Picture | *Best Picture Symbol, *Best Picture Symbol ▲, *Best Picture Symbol ★, +Best Picture Text | 2 |
| **+Best Picture Text**<br>`*Best Picture Score (copy)_1523061119509893128` | string · Dimension | Basic | +Best Picture Score | +Color Changer, +Filter One (T) | 2 |
| **+Bonus Chart**<br>`Calculation_1248623019843264544` | string · Dimension | Basic | — | — | 2 |
| **+Color Changer**<br>`+Color Numeric Changer  (copy)_1248623019522334735` | string · Dimension | Basic | +Primary Genre, Certificate (US), Certificate (GB), +Best Picture Text, Continent, Country | +Excluder (Text) | 2 |
| **+Excluder (Number)**<br>`Calculation_1248623019533844499` | string · Measure | Basic | +X Variable, +Y Variable, +Size Changer, +Filter Two (N) | — | 2 |
| **+Excluder (Text)**<br>`+Excluder (Number) (copy)_1248623019534909460` | string · Dimension | Basic | +Color Changer, +Filter One (T) | — | 2 |
| **+Filter One (T)**<br>`+Color Changer (copy)_1248623019546374173` | string · Dimension | Basic | +Primary Genre, Certificate (US), Certificate (GB), +Best Picture Text, Continent, Country | +Excluder (Text) | 2 |
| **+Filter Two (N)**<br>`+X Variable (copy)_1248623019548979231` | real · Measure | Basic | IMDB Rating, Number Of Votes, Year of Release, +Years Since Released, Runtime (Minutes), Title Id | +Excluder (Number) | 2 |
| **+Primary Genre**<br>`Calculation_1248623019496439808` | string · Dimension | Basic | Genres (1st) | +Color Changer, +Filter One (T), +Unit, +Unit Description | 2 |
| **+Shape Changer**<br>`+Y Variable (copy)_1248623019525095441` | string · Dimension | Basic | — | — | 1 |
| **+Size Changer**<br>`+Color Changer (copy)_1248623019512754188` | real · Measure | Basic | IMDB Rating, Number Of Votes, Year of Release, +Years Since Released, Runtime (Minutes), Title Id | +Excluder (Number) | 2 |
| **+Unit**<br>`+Y Variable (copy)_1248623019539742744` | string · Dimension | Basic | Title Id, +Primary Genre, Year of Release | — | 1 |
| **+Unit Description**<br>`+Unit (copy)_1248623019540701209` | string · Dimension | Basic | Title, +Primary Genre, Year of Release | — | 1 |
| **+X Variable**<br>`Calculation_1248623019503468546` | real · Measure | Basic | IMDB Rating, Number Of Votes, Year of Release, +Years Since Released, Runtime (Minutes), Title Id | +Excluder (Number), +Rank_X, +X Variable Unit | 2 |
| **+Y Variable**<br>`+X Variable (copy)_1248623019510009863` | real · Measure | Basic | IMDB Rating, Number Of Votes, Year of Release, +Years Since Released, Runtime (Minutes), Title Id | +Excluder (Number), +Rank_Y | 2 |
| **+Years Since Released**<br>`Calculation_1248623019507765252` | integer · Dimension | Basic | Year of Release | +Filter Two (N), +Size Changer, +X Variable, +Y Variable | 2 |
| **Number of people**<br>`Calculation_659777350465458179` | integer · Measure | Basic | Person Name ID | — | 1 |
| **Number of titles**<br>`Calculation_659777350463983618` | integer · Measure | Basic | Title Id | — | 1 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| **+Text Filter**<br>`+Color (C) (copy)_1248623019545931803` | string · list | `"7"` | list of 7 — "7" → Genre, "8" → US Rating, "9" → UK Rating, "10" → Oscar, "11" → Continent, "12" → Country |
| **+Size**<br>`+Color (copy)_1248623019512573963` | string · list | `"13"` | list of 6 — "1" → IMDb Rating, "2" → Votes, "3" → Release Year, "4" → Years Since Release, "5" → Movie Length (Min), "13" → -None- |
| **+Y**<br>`+X (copy)_1248623019509510149` | string · list | `"1"` | list of 5 — "1" → IMDb Rating, "2" → Votes, "3" → Release Year, "4" → Years Since Release, "5" → Movie Length (Min) |
| **+Numeric Filter**<br>`+X (copy)_1248623019546169372` | string · list | `"2"` | list of 5 — "1" → IMDb Rating, "2" → Votes, "3" → Release Year, "4" → Years Since Release, "5" → Movie Length (Min) |
| **+Color**<br>`+Y (copy)_1248623019510394889` | string · list | `"10"` | list of 7 — "7" → Genre, "8" → US Rating, "9" → UK Rating, "10" → Oscar, "11" → Continent, "12" → Country |
| **+Shape**<br>`+Y (copy)_1248623019524272144` | string · list | `"✦"` | list of 7 — "✦" → ✦, "●", "○", "■", "□", "▲" |
| **+Unit**<br>`+Y (copy)_1248623019539329047` | string · list | `"1"` | list of 1 — "1" → Movie |
| **+X**<br>`Parameter 1` | string · list | `"4"` | list of 5 — "1" → IMDb Rating, "2" → Votes, "3" → Release Year, "4" → Years Since Release, "5" → Movie Length (Min) |
| **+ Bonus Chart**<br>`Parameter 2` | string · list | `"S"` | list of 2 — "S" → Scatterplot, "B" → Bar |
| **Production Company**<br>`Parameter 3` | string · any | `"Marvel Studios"` | any value |
| **Time Chart Type**<br>`Parameter 6` | real · list | `1.` | list of 2 — 1. → Timeline Chart, 2. → Radial |
| **Select Movie Genre**<br>`Production Company (copy)_2043508340865777664` | string · list | `"Sci-Fi"` | list of 21 — "All" → In Total, "Action", "Adventure", "Animation", "Biography", "Comedy" |

## Appendix A — Calculated fields excluded from this documentation

12 calculated fields exist in the data pane but are not referenced by any worksheet or dashboard
logic:

`*Best Picture Symbol ▲` · `*Best Picture Symbol ★` · `*Best Picture Symbol` · `+Rank_X` ·
`+Rank_Y` · `+X Variable Unit` · `1` · `Min Year` · `0` · `Contains Production Company?` ·
`Contains Genre?` · `*Genres (full list text)`

Most are inherited from the author's *Movies Across Time & Space* workbook along with the extract.

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas and settings are quoted verbatim from the workbook
definition; rendered figures are read from the published view.*
