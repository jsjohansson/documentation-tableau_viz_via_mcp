# Movies Across Time & Space (Re-Viz) — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `Movies Across Time & Space (Re-Viz)` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/MoviesAcrossTimeSpaceRe-Viz/MoviesAcrossTimeSpace |
| Recognition | Tableau **Viz of the Day** (`#VOTD`) |
| Documentation scope | Complete workbook internals — data source, extract, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity, and design system |
| Source of record | `Movies Across Time & Space (Re-Viz).twbx` → `Movies Across Time & Space (Re-Viz).twb` (workbook XML), decompiled and read field-by-field |
| Explicit exclusion | 22 calculated fields that exist in the data pane but are **not referenced by any worksheet or by dashboard logic** are deliberately not documented. They are listed by name only in Appendix B for housekeeping. |
| Companion file | `documentation_viz_MoviesAcrossTimeSpaceRe-Viz_README.md` — same facts, condensed overview format |

---

## 1. Executive summary

**Movies Across Time & Space (Re-Viz)** is a single-dashboard, dark-theme data visualisation that profiles the **500 highest-rated science-fiction films on IMDb**, drawn from a multi-million-row IMDb titles-and-people dataset.

The signature view is a **trigonometric radial "galaxy"**: every qualifying film is a star plotted by polar coordinates derived from its release year (angle) and its rank within that year (radius), coloured by IMDb rating and sized by vote count. Concentric elliptical "orbit" rings and a glowing core anchor the composition, and a full-canvas starfield sheet sits behind everything as the background plate.

Around that centrepiece the dashboard delivers a KPI band, three ranked bar panels (top-rated, most-voted, Oscar Best Picture nominations), and a decade-by-decade population breakdown with a donut and summary table. A three-state **view switcher** (Space Radial / Timeline / Movie List) swaps the main stage using parameter actions plus Tableau's **Dynamic Zone Visibility**.

The workbook is a re-build ("Re-Viz") of the author's earlier *Movies Across Time & Space*, and the original is linked from the header.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Authoring build | 2026.2.5 (20262.26.0804.1806) |
| Author locale | `en_GB` |
| Worksheets | 32 |
| Dashboards | 1 (`Movies Across Time & Space`) |
| Stories | 0 |
| Data sources | 1 (single extract, no joins, no custom SQL) |
| Parameters | 11 |
| Total fields in data pane | 105 (33 base + 72 calculated) |
| Calculated fields **used in sheets / dashboard logic** | 50 named + 14 in-view ad-hoc constants |
| Calculated fields unused (not documented) | 22 |
| Dashboard canvas | 1600 × 2000 px, fixed size |
| Global font | Trebuchet MS |
| Dashboard actions | 3 URL, 1 highlight, 4 parameter actions |
| Dynamic Zone Visibility bindings | 12 zones |

### 1.2 Headline figures rendered by the dashboard

| KPI | Displayed value | Source worksheet |
|---|---|---|
| Total Sci-Fi movies (Top 500 of …) | `500 / 11,220` | `KPI-Movie Count` |
| Directors | `427` | `KPI-Directors` |
| Oscar Nominations | `15 ▲` | `KPI-Nom` |
| Oscar Best Picture | `1 ★` | `KPI-BP` |
| Population split | Top 500 = 500 (4.6 %) · Sci-Fi with 10,000+ votes = 724 (6.6 %) · Remaining Sci-Fi = 9,762 (88.9 %) | `E-Table`, `E-Pie` |

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `18944871` |
| LUID | `02183d0c-d6c3-4142-a272-ea30ca7f583d` |
| Repository URL | `MoviesAcrossTimeSpaceRe-Viz` |
| Default view | `Movies Across Time & Space` (`sheets/MoviesAcrossTimeSpace`) |
| Default view LUID | `c45f56d8-135b-4182-9a29-22065605f644` |
| Revision | 1.3 |
| First published | 2026-08-09 01:47 UTC |
| Last published | 2026-08-12 15:44 UTC |
| Last updated | 2026-08-21 15:29 UTC |
| Published size | 460,102,428 bytes (438.8 MB) |
| View count (at time of documentation) | 8,703 |
| Favourites | 29 |
| Data download allowed | Yes (`allowDataAccess = true`) |
| Tabs shown | No · Toolbar shown: Yes · Watermark: Yes |
| Attribution | Re-Viz of `Movies Across Time & Space` → `MoviesAcrossTimeSpace/MoviesAcrosstheDecades` |
| Author links | GitHub `jsjohansson` · X/Twitter `JohnSJohansson` |

**Published description**

> Explore the Top 500 Sci-Fi Movies in the #IMDb dataset
> Re-Viz reflects three years of progress across my design capabilities, #Tableau upgrades, and analytics knowledge.
> #DataPlusMovies #Movies #SciFi #Radial #VOTD #VizOfTheDay

---

## 3. Data architecture

### 3.1 Connection

| Property | Value |
|---|---|
| Data source caption | **IMDB movies and people Extract** |
| Internal name | `federated.0uydhq71ekj7nt13f7jp60qx5vg3` |
| Connection class | `federated` → single `hyper` named connection (`IMDB movies and people`) |
| Relation | `[Extract].[Extract]`, type `table` |
| Joins / unions | **None** — one physical table |
| Custom SQL | None |
| Live vs extract | **Extract** (embedded, enabled) |

### 3.2 Extract

| Property | Value |
|---|---|
| Extract file | `Data/Extracts/federated_0uydhq71ekj7nt13f7jp60.hyper` |
| Extract size | 934,608,896 bytes (891.3 MB) |
| Rows | **5,176,667** |
| Columns materialised | **18** |
| Refresh type | `create` (full) |
| Refresh timestamp | 2026-08-09 02:05:31 → 02:05:51 |
| Aggregation on extract | None (row-level) |
| Packaged `.twbx` contents | 1 `.twb` (28.2 MB) + 1 `.hyper` (891.3 MB); no embedded image assets |

> **Why the `.twb` is 28 MB:** the `Display Movie Info` parameter stores a materialised list of **453,768** film titles as allowable values. That single parameter accounts for the overwhelming majority of the workbook XML.

### 3.3 Grain

The extract is at **title × person** grain: one row per film credit. Practical consequences that shape every calculation in the workbook:

- `titleId` is **not** unique per row — all title-level metrics use `COUNTD([titleId])`, `MIN(...)`, `MAX(...)` or `{FIXED [titleId] : ...}` rather than `SUM`.
- People metrics use `COUNTD([Person Name ID])`.
- Approximate cardinalities in the extract: 866,005 distinct `titleId`; 971,095 distinct `Person Name ID`; 758,939 distinct `Title`; 124 distinct `Year of Release`.

### 3.4 Data source filters — applied to **every** worksheet

Two filters are defined at the data source level and therefore constrain all 32 worksheets before any sheet-level filter runs:

| # | Field | Type | Condition |
|---|---|---|---|
| 1 | `Genres (full list)` | Condition (formula) | `CONTAINS(LOWER([Genres (full list)]), LOWER('Sci-Fi'))` |
| 2 | `What did they do ?` | Categorical, member | `= "director"` |

Filter 2 is the reason `KPI-Directors` can count people simply with `COUNTD([Person Name ID])` — every row visible to the workbook is a *director* credit.

### 3.5 Source schema (33 columns available on the connection)

The upstream source exposes 33 columns. Only the 18 marked **✔ Extract** were materialised into the `.hyper` file; the remainder survive in the data pane as hidden legacy fields with no data behind them.

| # | Column | Type | Approx. distinct | In extract |
|---:|---|---|---:|:---:|
| 0 | `Production Companies (List)` | string | 403,059 | — |
| 1 | `Best Picture` | string | 3 | ✔ |
| 2 | `Certificate (GB)` | string | 23 | ✔ |
| 3 | `Certificate (US)` | string | 25 | ✔ |
| 4 | `imdbUrl (Person)` | string | 978,607 | — |
| 5 | `Person Name` | string | 966,016 | — |
| 6 | `Person Name ID` | string | 978,607 | ✔ |
| 7 | `titleId` | string | 898,114 | ✔ |
| 8 | `What did they do ?` | string | 3 | ✔ |
| 9 | `Who did they play ?` | string | 627,540 | — |
| 10 | `Billing (position in cast list)` | integer | 51 | — |
| 11 | `Plot` | string | 747,944 | ✔ |
| 12 | `Plot (medium)` | string | 550,841 | — |
| 13 | `Genres (full list)` | string | 12,232 | ✔ |
| 14 | `genres (1st)` | string | 26 | — |
| 15 | `genres (2nd)` | string | 24 | — |
| 16 | `genres (3rd)` | string | 28 | — |
| 17 | `runtime (minutes)` | integer | 497 | ✔ |
| 18 | `imdbUrl (title)` | string | 898,114 | ✔ |
| 19 | `Title` | string | 788,137 | ✔ |
| 20 | `Year of Release` | integer | 124 | ✔ |
| 21 | `color` | string | 5 | — |
| 22 | `Production Companies (1st)` | string | 206,929 | — |
| 23 | `Production Companies (2nd)` | string | 121,152 | — |
| 24 | `Production Companies (3rd)` | string | 80,131 | — |
| 25 | `image_url (title)` | string | 803,433 | — |
| 26 | `imdb rating` | real | 110 | ✔ |
| 27 | `number of votes` | integer | 50,707 | ✔ |
| 28 | `tagline` | string | 388,718 | ✔ |
| 29 | `Continent` | string | 6 | — |
| 30 | `Region` | string | 24 | ✔ |
| 31 | `Country` | string | 249 | ✔ |
| 32 | `Language` | string | 166 | ✔ |

### 3.6 Data-pane organisation

**Folders (7)**

| Folder | Contents |
|---|---|
| **Conditions** | `*Best Picture T\|F`, `Top 500 %`, `Groups`, `Top 500 Sci-Fi T\|F`, `Target Pop T\|F`, `Sci-Fi T\|F`, `Groups Name`, `Year of Release >= 1950 T\|F`, `Number Of Votes >10000 T\|F` |
| **Movie** | `Best Picture`, `Number of Titles`, `Certificate (GB)`, `Rating`, `*Genres (full list text)`, `*IMDB Rating (Color)`, `Language`, `Plot (medium)`, `Plot`, `Runtime`, `Title`, `Year`, `Color`, `Image Url (Title)`, `IMDB Rating`, `IMDB Url (title)`, `Number Of Votes`, `Runtime (Minutes)`, `Tagline`, `Title Id`, `Genre`, `Production Companies`, `Production Country` |
| **People (actors, directors)** | `Billing (position in cast list)`, `Director`, `Number of people`, `Person Name ID`, `Person Name`, `What did they do ?`, `Who did they play ?`, `IMDB Url (Person)` |
| **Radial Chart** | `*Angle`, `*Angle Adjusted`, `*Angle Adjusted Rings`, `*Index`, `*Index x-1`, `*YoR`, `*YoR /5`, `*X cos`, `*X cos Small 2…6, 6.1`, `*Y sin`, `*Y sin Small 2…6, 6.1`, constants `2`–`6` |
| **Show Movie Info** | `Show Movie Info: DZN`, `Show Movie Info: Sheet` *(both currently unused)* |
| **Toggles** | `Chart Toggle: Radial T\|F`, `Chart Toggle: Timeline T\|F`, `Chart Toggle: Radial/Timeline T\|F`, `Chart Toggle: Table T\|F` |
| **Top Bar Toggle** | `Bar Toggle: A/B/C/D T\|F` |

**Hierarchies / drill paths (3)**

| Drill path | Levels |
|---|---|
| `Genre` | `Genres (full list)` |
| `Production Companies` | *(empty)* |
| `Production Country` | `Region` → `Country` |

**Groups (13)** — all hidden, all auto-generated by Tableau to back dashboard highlight and tooltip behaviour (`Action (Continent,Country)`, `Action (Genres (1st))`, `Action (Language)`, `Action (Year of Release)`, `Action (Year of Release) 1`, `Tooltip (Best Picture)`, `Tooltip (Title)`, `Tooltip (Title,Year of Release)`, and five further tooltip cross-joins). None were hand-authored.

**Other data source settings**

- Start of week: Monday
- Default manual sort on `Forecast Indicator`: `Actual` → `Estimate`
- Geographic role assigned: `Country` → `[Country].[Name]` with locale hint `"United Kingdom"`
- Data pane layout: alphabetic ordering of dimensions and measures, 100 rows shown in preview

---

## 4. Field dictionary — base fields

### 4.1 Base fields referenced by worksheets

| Caption | Physical column | Data type | Role | Used in *n* sheets | Notes |
|---|---|---|---|---:|---|
| **Title Id** | `titleId` | string | Dimension | 26 | Primary film key; carries the Top-500 filter and all `COUNTD` film counts |
| **Title** | `Title` | string | Dimension | 9 | Row label on bar panels and movie list; source field of the hover parameter action |
| **Year** | `Year of Release` | integer | Dimension (discrete/continuous by view) | 31 | Drives radial angle, timeline position, decade grouping, and the 1950–2022 range filter |
| **IMDB Rating** | `imdb rating` | real | Measure | 21 | Colour encoding across radial/timeline; bar length on `D-TopIMDb Bar`; ranking basis |
| **Number Of Votes** | `number of votes` | integer | Measure | 31 | Size encoding everywhere; bar length on `D-TopVote Bar`; ≥ 10,000 threshold filter |
| **Runtime (Minutes)** | `runtime (minutes)` | integer | Measure | 8 | Tooltip only (rendered as `… mins`) |
| **Genres (full list)** | `Genres (full list)` | string | Dimension | 25 | Basis of the `Sci-Fi T\|F` test and the data source filter |
| **Best Picture** | `Best Picture` | string | Dimension | 9 | Domain: `Winner`, `Nominated`, null |
| **Rating** | `Certificate (US)` | string | Dimension | 8 | US certificate; tooltip field |
| **Certificate (GB)** | `Certificate (GB)` | string | Dimension | 4 | Carried on line-layer tooltips |
| **Country** | `Country` | string | Dimension | 8 | Tooltip; geo-role assigned |
| **Region** | `Region` | string | Dimension | 4 | Tooltip; upper level of the Production Country drill path |
| **Language** | `Language` | string | Dimension | 8 | Tooltip |
| **Plot** | `Plot` | string | Dimension | 7 | Long synopsis in the rich tooltip |
| **Tagline** | `tagline` | string | Dimension | 7 | Wrapped by `*Txt_Tagline` |
| **IMDB Url (title)** | `imdbUrl (title)` | string | Dimension | 4 | Held on tooltip detail |
| **Person Name ID** | `Person Name ID` | string | Dimension | 2 | `COUNTD` basis for the Directors KPI and `Number of people` |
| **What did they do ?** | `What did they do ?` | string | Dimension | 1 | Sheet-level `= "director"` filter on `KPI-Directors` (redundant with the data source filter, retained for clarity) |
| **Measure Names** | `:Measure Names` | string | Dimension | 3 | Colour/columns driver on the bar panels and `E-Table` |

### 4.2 Base fields present but not used in any worksheet

Hidden or dormant: `Billing (position in cast list)`, `Continent`, `Person Name`, `Plot (medium)`, `Production Companies (1st/2nd/3rd)`, `Production Companies (List)`, `Who did they play ?`, `color`, `genres (1st/2nd/3rd)`, `image_url (title)`, `imdbUrl (Person)`. Most were never materialised into the extract (§3.5) and are inherited from the source model.

---

## 5. Parameters

Eleven parameters exist. Four are **live controls** (driven by parameter actions), four are **geometry constants** consumed by the radial trigonometry, and three are latent/legacy.

| # | Caption | Internal name | Type | Domain | Current value | Status |
|---:|---|---|---|---|---|---|
| 1 | **View** | `[Parameter 7]` | string | List — `"1"` = Timeline, `"2"` = Space Radial, `"3"` = Movie List | `"2"` | **Live** — the master view switcher |
| 2 | **Select Movie Genre** | `[Production Company (copy)_2043508340865777664]` | string | List — 21 members: `All` (aliased *In Total*), Action, Adventure, Animation, Biography, Comedy, Crime, Drama, Family, Fantasy, History, Horror, Music, Musical, Mystery, Romance, **Sci-Fi**, Sport, Thriller, War, Western | `"Sci-Fi"` | **Live** — injected into the dashboard title and the genre badge |
| 3 | **Display Movie Info** | `[Parameter 1]` | string | List — 453,768 film titles | `"Bad Taste"` | **Live target** of the hover action; no visible consumer (see §11.4) |
| 4 | **Top Bar** | `[Parameter 2]` | string | List — `"A"` (*Top IMDb Rated Movie*), `"B"` (*Top Voted For Movie*), `"C"`, `"D"` | `"C"` | Semi-live — bound to Dynamic Zone Visibility of the Best Picture panel, but no on-canvas control |
| 5 | **\*Hole** | `[Parameter 4]` | real | Any | `7.` | Geometry — inner radius of the main radial |
| 6 | **\*Hole Small** | `[*Hole (copy)_2236600186040856580]` | real | Any | `6.` | Geometry — base radius of the five orbit rings |
| 7 | **\*Angle, Start** | `[Parameter 5]` | integer | Range 70 – 360 | `70` | Geometry — start angle (°) of the main radial sweep |
| 8 | **\*Angle, Start Rings** | `[*Angle, Start (copy)_3516544364994593]` | integer | Range 40 – 360 | `40` | Geometry — start angle (°) of the ring sweep |
| 9 | **\*Angle, End** | `[Start Angle (copy)_1009861854720749594]` | integer | Range from 0 | `405` | Geometry — end angle (°) shared by both sweeps |
| 10 | **Time Chart Type** | `[Parameter 6]` | real | List — `1.` = Timeline Chart, `2.` = Radial | `2.` | Legacy — superseded by `View` |
| 11 | **Production Company** | `[Parameter 3]` | string | Any | `"Marvel Studios"` | Legacy — its consumer calculation is unused |

> **Naming note.** Several parameters and calculations carry `(copy)` suffixes in their internal names because they were duplicated during authoring. The *caption* is always the meaningful identity; internal names are recorded here so the XML can be traced.

---

## 6. Calculated fields in use

50 named calculated fields are referenced by worksheets or by dashboard logic, plus 14 in-view ad-hoc calculations. They are grouped below by role. Formulas are reproduced exactly; field references are shown by caption for readability.

### 6.1 The radial geometry engine

This is the mathematical core of the signature chart. Read the group top-to-bottom — each step feeds the next.

#### `*Angle` — base angle in radians
```
2 * PI() *([Year of Release]-1950)
/
({MAX([Year of Release])}-1950)
```
Maps the release-year span onto a full 2π revolution, with 1950 pinned to 0. `{MAX([Year of Release])}` is a table-scoped LOD so the denominator is the dataset maximum rather than a per-partition value.
*Used in:* `A-Radial`, `A-Radial Cloud`, `A-Radial YR`, `A-Radial Circle2`–`Circle6`.

#### `*Angle Adjusted` — main sweep, compressed into the parameterised arc
```
(
[*Angle]
*
(([*Angle, End] - [*Angle, Start]) / 360)
+
([*Angle, Start] * 2 * PI() / 360)
)
```
Rescales the full revolution into the arc between **70°** and **405°** (a 335° sweep that deliberately leaves a wedge open on the right-hand side, producing the comet-tail silhouette).
*Used in:* `A-Radial`, `A-Radial Cloud`, `A-Radial YR`.

#### `*Angle Adjusted Rings` — identical transform, ring start angle
```
(
[*Angle]
*
(([*Angle, End] - [*Angle, Start Rings]) / 360)
+
([*Angle, Start Rings] * 2 * PI() / 360)
)
```
Same construction with **40° → 405°** so the orbit rings sweep wider than the star field.
*Used in:* `A-Radial Circle2`–`Circle6`.

#### `*Index` — radial distance driver
```
INDEX()-1
```
Table calculation. Computed along `Title Id` and partitioned by `Year`, so within a release year the *n*-th film sits one unit further from the core. This is what turns each year into a radiating spoke.
*Used in:* all eight radial sheets.

#### `*X cos` / `*Y sin` — main Cartesian projection
```
*X cos:  ([*Index] + [*Hole]) * (COS(MIN([*Angle Adjusted])))
*Y sin:  ([*Index] + [*Hole]) * (SIN(MIN([*Angle Adjusted])))
```
Standard polar-to-Cartesian conversion with `*Hole` (7) as the empty core radius.
*Used in:* `A-Radial`, `A-Radial Cloud`.

#### Ring coordinate pairs — five concentric ellipses

All ten follow one template, differing only by the offset subtracted from `*Hole Small` (6):

```
*X cos Small n:  ([*Index] + ([*Hole Small] - k)) * (COS(MIN([*Angle Adjusted Rings])))
*Y sin Small n:  ([*Index] + ([*Hole Small] - k)) * (SIN(MIN([*Angle Adjusted Rings])))
```

| Calculation pair | Offset *k* | Effective base radius | Worksheet |
|---|---:|---:|---|
| `*X cos Small 2` / `*Y sin Small 2` | 4 | 2 | `A-Radial Circle2` |
| `*X cos Small 3` / `*Y sin Small 3` | 3 | 3 | `A-Radial Circle3` |
| `*X cos Small 4` / `*Y sin Small 4` | 2 | 4 | `A-Radial Circle4` |
| `*X cos Small 5` / `*Y sin Small 5` | 1 | 5 | `A-Radial Circle5` |
| `*X cos Small 6` / `*Y sin Small 6` | 0 | 6 | `A-Radial Circle6` |

#### `*X cos Small 6.1` / `*Y sin Small 6.1` — year-label ring
```
*X cos Small 6.1:  ([*Index] + [*Hole Small]) * (COS(MIN([*Angle Adjusted])))
*Y sin Small 6.1:  ([*Index] + [*Hole Small]) * (SIN(MIN([*Angle Adjusted])))
```
Radius 6 like ring 6, but driven by the **main** sweep angle so the year captions align with the star field rather than the rings.
*Used in:* `A-Radial YR`.

#### `*YoR /5` — year-label filter
```
IF ([Year of Release] IN (1900, 1910, 1920, 1930, 1940, 1950, 1960, 1970, 1980, 1990, 2000, 2010, 2020, 2030, 2040))
OR
([Year of Release] IN (1905, 1915, 1925, 1935, 1945, 1955, 1965, 1975, 1985, 1995, 2005, 2015, 2025, 2035, 2045))
then [Year of Release]
END
```
Emits a year only every five years, so the ring is labelled 1955 · 1960 · 1965 … without collision. The decade and half-decade sets are written separately for editability.
*Used in:* `A-Radial YR`.

#### Ring stroke-width constants — `2`, `3`, `4`, `5`, `6`
Five one-line calculations returning their own integer. Placed on **Size** of the line layer in the matching ring sheet so each orbit gets a distinct, deterministic stroke weight.
*Used in:* `A-Radial Circle2`–`Circle6` respectively.

### 6.2 Timeline geometry

#### `*YoR` — horizontal position
```
INDEX()+Min([Year of Release])
```
Table calculation; reconstructs a continuous year axis from the index so the timeline can be densely packed.
*Used in:* `B-Timeline`, `B-Timeline Cloud`.

#### `*Index x-1` — vertical stacking
```
(INDEX()-1)*-1
```
Negated index so films stack **downward** from the axis within each year.
*Used in:* `B-Timeline`, `B-Timeline Cloud`.

### 6.3 Population classification — how "Top 500" is defined

Three related fields define the analytical population. Note that the workbook derives the Top 500 **twice**, by two different mechanisms (see §12).

#### `Sci-Fi T|F`
```
IF CONTAINS([*Genres (full list text)],'Sci-Fi') THEN TRUE
ELSE FALSE
END
```
Sheet-level guard applied to **25 of 32 worksheets** as a context filter (`= TRUE`). Duplicates the data source filter, which makes each sheet self-documenting and lets the filter run in the context group.
*Used in:* every analytic worksheet.

#### `Target Pop T|F` — the eligible population, computed with LODs
```
{ FIXED [titleId] : MAX([number of votes]) } > 10000
AND CONTAINS({ FIXED [titleId] : MAX([Genres (full list)]) }, 'Sci-Fi')
AND { FIXED [titleId] : MAX([Year of Release]) } >= 1950
```
Three `{FIXED [titleId]}` expressions collapse the title × person grain to one verdict per film, so the three qualifying conditions (10,000+ votes, Sci-Fi, released 1950 or later) are evaluated at film level regardless of how many credit rows a film has. **This is the only LOD-based field in the workbook.**
*Used in:* `E -Boxes`, `E -Decades`, `E-Pie`, `E-Table`, `F-Movie Table`.

#### `Top 500 Sci-Fi T|F` — the frozen Top-500 list
```
//TOP 500 IMDb ratings from
//Votes > 10000, Sci-FI, Year >= 1950

IF [titleId] IN ( "tt0043456", "tt0044121", … 500 IMDb title IDs … )
THEN TRUE
ELSE FALSE
END
```
A hard-coded membership test over 500 explicit `titleId` values, carrying the author's own comment describing the selection rule. Freezing the list guarantees the breakdown panels and the movie list stay stable regardless of rating drift.
*Used in:* `E -Boxes`, `E -Decades`, `E-Pie`, `E-Table`, `F-Movie Table`.

#### `Groups` — ordinal cohort
```
IF   [Top 500 Sci-Fi T|F] = TRUE              THEN 1
ELSEIF [Target Pop T|F]   = TRUE              THEN 2
ELSEIF [Sci-Fi T|F]       = TRUE              THEN 3
ELSE 4
END
```
Priority-ordered cohort assignment; the breakdown sheets filter to members 1–3 and discard 4.
*Used in:* `E -Boxes`, `E -Decades`, `E-Pie`, `E-Table`.

#### `Groups Name` — cohort label
```
IF   [Top 500 Sci-Fi T|F] = TRUE THEN 'Top 500 Sci-Fi Movies'
ELSEIF [Target Pop T|F]   = TRUE THEN 'Sci-Fi with 10,000+ Votes Movies'
ELSEIF [Sci-Fi T|F]       = TRUE THEN 'Remaining Sci-Fi Movies'
ELSE 'Non-Sci-Fi Movies'
END
```
The string twin of `Groups`, used for colour and row headers.
*Used in:* `E -Boxes`, `E -Decades`, `E-Pie`, `E-Table`, `F-Movie Table`.

#### `Decade`
```
Left(Str([Year of Release]),3)+"0's"
```
String truncation decade label — `1994` → `1990's`.
*Used in:* `E -Boxes`, `E -Decades`.

### 6.4 Ranking

| Field | Formula | Purpose | Used in |
|---|---|---|---|
| **Rank** | `RANK_UNIQUE(min([imdb rating]),'desc')` | Rank by rating, ties broken deterministically | `D-TopIMDb Bar`, `F-Movie Table` |
| **\*Rank_Voting** | `RANK_UNIQUE(min([number of votes]),'desc')` | Rank by vote volume | `D-TopVote Bar` |
| **\*Rank_BP** | 16-branch `IF [titleId] = "tt…" THEN n … ELSE 0 END` | Hand-curated display order for the Oscar panel, ranking *Everything Everywhere All at Once* (1) through *Blade Runner* (16) | `D-Best Picture` |

### 6.5 Awards encoding

| Field | Formula | Purpose |
|---|---|---|
| **\*Best Picture Score** | `IF [Best Picture]='Winner' THEN 'BP' ELSEIF [Best Picture]='Nominated' THEN 'ON' ELSE '' END` | Compact internal code |
| **\*Best Picture Symbol** | `IF [*Best Picture Score]='BP' THEN '★' ELSEIF [*Best Picture Score]='ON' THEN '▲' ELSE '' END` | Glyph shown as a mark label on stars, timeline dots and bar rows |
| **\*Best Picture Text** | `IF [*Best Picture Score]='BP' THEN 'Oscar Best Picture' ELSEIF [*Best Picture Score]='ON' THEN 'Oscar Nomination' END` | Human-readable award line in tooltips and on the Oscar panel |

All three are used in `A-Radial`, `A-Radial Cloud`, `B-Timeline`, `B-Timeline Cloud`, `D-Best Picture`, `D-TopIMDb Bar`, `D-TopVote Bar`.

### 6.6 Text presentation

| Field | Formula | Purpose | Used in |
|---|---|---|---|
| **\*Genres (full list text)** | `REGEXP_REPLACE([Genres (full list)], "," , ", ")` | Inserts a space after each comma so genre lists wrap legibly in tooltips; also the input to `Sci-Fi T\|F` | 25 sheets |
| **\*Txt_Tagline** | `IF ISNULL([tagline]) then '' ELSE '◄'+REGEXP_REPLACE([tagline],'"','')+'►' END` | Strips stray quote marks and wraps the tagline in guillemet-style glyphs; null-safe | 7 sheets |
| **Runtime** | `STR([runtime (minutes)])+ ' min'` | Pre-formatted runtime string for the movie list column | `F-Movie Table` |
| **\*Genre Abbv** | 21-branch `CASE [Select Movie Genre] … When "Sci-Fi" then "SF" … END` | Three-letter (or two-letter) genre abbreviation rendered inside the header badge | `C-Pie Abbv` |
| **A_Color** | `'A'` | Single-member dimension whose only purpose is to force the link icons to a specific palette colour (`#ffc63c`) | `Link: Classic`, `Link: LI`, `Link: Tableau` |

### 6.7 Aggregate measures

| Field | Formula | Purpose | Used in |
|---|---|---|---|
| **Number of Titles** | `// a count distincy of the number of Titles in the dataset`<br>`COUNTD([titleId])` | Film count at any level of detail; also the basis of the `% of Total` table calculation | `E -Boxes`, `E -Decades`, `E-Pie`, `E-Table` |
| **Number of people** | `COUNTD([Person Name ID])` | Distinct people; with the data source director filter in force this is a director count | `A-StarLine` |

### 6.8 Toggle booleans and Dynamic Zone Visibility controls

| Field | Formula | Role |
|---|---|---|
| **Chart Toggle: Radial T\|F** | `[View] = '2'` | Colours the *Space Radial* button; shows zones `A-Radial`, `A-Radial Cloud`, `A-Radial YR` |
| **Chart Toggle: Timeline T\|F** | `[View] = '1'` | Colours the *Timeline* button; shows zones `B-Timeline`, `B-Timeline Cloud` |
| **Chart Toggle: Table T\|F** | `[View] = '3'` | Colours the *Movie List* button; shows the `Table` container |
| **Chart Toggle: Radial/Timeline T\|F** | `[View] = '1' OR [View] = '2'` | Shows the shared chrome for both chart views — the `Bars` row, the `Breakdown Bar/Pie` row, the `Planets` and `Bubbles` containers and the main chart spacer |
| **Bar Toggle: C T\|F** | `[Top Bar] = 'C'` | Shows the `D-Best Picture` zone |

### 6.9 In-view ad-hoc calculations

Fourteen unnamed calculations live inside individual worksheets rather than the data pane. They are axis anchors, constant labels and parameter-action payloads.

| Expression | Purpose | Worksheet(s) |
|---|---|---|
| `SUM(0)` | Zero-height anchor row for the core/badge marks | `A-Radial Center1`, `A-Radial Center2`, `C-Pie Abbv` |
| `MIN(0)` | Zero anchor for the donut | `E-Pie` |
| `MIN(1)` | Single-unit bar length on the Oscar panel | `D-Best Picture` |
| `MIN(10.0)` | Full-scale (rating = 10) background bar | `D-TopIMDb Bar` |
| `MIN(2460667)` | Full-scale background bar — the maximum vote count in the set | `D-TopVote Bar` |
| `'IMDb Votes'` | Static column header | `F-Movie Table` |
| `''` (×2) | Invisible anchor dimensions for the link and toggle buttons | `Link: *`, `Toggle:*` |
| `'Space Radial'`, `'Timeline'`, `'Movie List'` | Button captions | `Toggle:Radial`, `Toggle:Timeline`, `Toggle:Movie List Table` |
| `'2'`, `'1'`, `'3'` | Payload values passed by the parameter actions into `View` | `Toggle:Radial`, `Toggle:Timeline`, `Toggle:Movie List Table` |

---

## 7. Worksheet specifications

32 worksheets. Grouped by function; every sheet reads from the single IMDB extract.

### 7.0 Shared filter stack

Most analytic sheets carry the same four-filter stack (three of them **context filters**, which is what makes the Top-500 filter behave correctly):

| Filter | Class | Setting | Context |
|---|---|---|---|
| `Sci-Fi T\|F` | Categorical | `= true` | ✔ context |
| `Year` | Quantitative | 1950 – 2022, in-range | ✔ context |
| `Number Of Votes` | Quantitative | ≥ 10,000 (no upper bound) | ✔ context |
| `Title Id` | Top-N | **Top 500 by `MAX([imdb rating])` DESC** | — |

### 7.1 Background plate

#### `A-StarLine` — the starfield

| Property | Value |
|---|---|
| Mark | Text, shape `asterisk`, transparency 103 |
| Columns / Rows | `Title` / `Year` |
| Size | `SUM([Number Of Votes])` |
| Text | `Number of people` |
| Sheet background | `#151311` (near-black warm) |
| Filters | Year 1950–2022, Votes ≥ 10,000, Title Id (Top-N) — **no** Sci-Fi guard |
| Dashboard role | Zone 302, friendly-named **"Tile: Background Stars"**, stretched to the full canvas beneath every other element |

### 7.2 Space Radial view (10 sheets)

#### `A-Radial` — the primary star field

| Property | Value |
|---|---|
| Columns | `*X cos` (fixed axis **−12 → 32**) |
| Rows | `*Y sin` + `*Y sin` — dual axis, synchronised (fixed axis **−20 → 20**) |
| Marks | Three layers: **Circle** (halo, `#005500`, size 2) · **Circle** (film mark, `:filled/circle`, base `#5d9451`) · **Line** (the spoke, `#eae0d5`, transparency 155, width 0.58) |
| Level of detail | `Year`, `Title Id` |
| Colour | `MIN(IMDB Rating)` → custom sequential palette **Space Cyan2**, 7 steps, upper bound fixed at 8.0 |
| Size | `MIN(Number Of Votes)`, domain 10,000 → 1,000,000 |
| Label | `MAX(*Best Picture Symbol)` at 6–8 pt (`#cc5803` on the film layer, `#0a0908` on the line layer) |
| Tooltip | The rich movie card (§7.8) |
| Axis ratio | The asymmetric fixed ranges (44 wide × 40 tall, rendered into a wide zone) are what flatten the circle into the **galaxy ellipse** |

#### `A-Radial Cloud` — soft glow duplicate
Same shelves and geometry as `A-Radial`; layer 2 is a **Shape** mark and the line layer is fully transparent. Rendered in the `Bubbles` container directly beneath `Planets`, producing the diffuse bloom around each star. Colour uses an 11-stop grey-blue custom ramp (`#f1f1f1` → `#778da9`).

#### `A-Radial Circle2` … `A-Radial Circle6` — five orbit rings
Identical construction, one per ring:

| Sheet | Columns | Rows (dual axis) | Line size constant |
|---|---|---|---|
| `A-Radial Circle2` | `*X cos Small 2` | `*Y sin Small 2` ×2 | `2` |
| `A-Radial Circle3` | `*X cos Small 3` | `*Y sin Small 3` ×2 | `3` |
| `A-Radial Circle4` | `*X cos Small 4` | `*Y sin Small 4` ×2 | `4` |
| `A-Radial Circle5` | `*X cos Small 5` | `*Y sin Small 5` ×2 | `5` |
| `A-Radial Circle6` | `*X cos Small 6` | `*Y sin Small 6` ×2 | `6` |

Each has three layers (Circle / Circle / **Line** with `Year` on **Path**), the same fixed axes as `A-Radial`, colour on `MIN(IMDB Rating)` with the grey-blue ramp, and size on `MIN(Number Of Votes)`.

#### `A-Radial YR` — year captions

| Property | Value |
|---|---|
| Columns / Rows | `*X cos Small 6.1` / `*Y sin Small 6.1` (dual axis) |
| Marks | Three **Text** layers |
| Text | Layer 2: `MIN(*YoR /5)` (every five years) · Layer 3: `MIN(Year)` |
| Size | `MIN(Number Of Votes)` |
| Mark colour | `#4b7f52`, labels `#0000009b` |
| Result | The gold year ring — 1951, 1955, 1960 … 2020 — sitting just outside the core |

#### `A-Radial Center1` / `A-Radial Center2` — the glowing core

| Property | `Center1` | `Center2` |
|---|---|---|
| Rows | `SUM(0)` | `SUM(0)` |
| Mark | **Heatmap** (density), intensity 0.146, kernel 12.43 | **Circle**, size 2.91, kernel 14.55, stroke on |
| Mark colour | `#eae0d5` | `#03393d` |
| Label colour | `#f4f5f7`, 12 pt | `#f4f5f7`, 12 pt |
| Dashboard placement | Floating zone 384 at (15188, 26400), 27000 × 19000 | Floating zone 381 at (15188, 277500), 27000 × 19000 |

Together they form the star at the heart of the galaxy: a density blur over a dark filled circle.

### 7.3 Timeline view (2 sheets)

#### `B-Timeline`

| Property | Value |
|---|---|
| Columns | `*YoR` ×2 (dual axis), axis from 1950, major spacing 10, minor 1, ticks hidden |
| Rows | `*Index x-1` ×2 (dual axis), max 1, major spacing 5 |
| Marks | **Circle** (halo `#eae0d5`) · **Shape** (film mark) · **Line** (`#eae0d5`, transparency 155) |
| Colour | `MIN(IMDB Rating)` → **Space Cyan2**, max 8.0 |
| Size | `MIN(Number Of Votes)` |
| Sort | Ascending by `MIN(IMDB Rating)` |
| Label | `MAX(*Best Picture Symbol)` |

#### `B-Timeline Cloud`
The glow twin of `B-Timeline` — same shelves, Circle layer at transparency 52, line layer at transparency 0, grey-blue colour ramp.

### 7.4 Header sheets

#### `C-Pie Abbv` — the genre badge

| Property | Value |
|---|---|
| Rows | `SUM(0)` ×2 (dual axis) |
| Marks | Three **Pie** layers: `#d7a64b` (outer gold ring) · `#eae0d5` (cream) · `#cc5803` (orange) |
| Text | `Select Movie Genre` + `*Genre Abbv` → renders **"SF"** |
| Label | `#f4f5f7`, 12 pt |

#### `Link: Classic`, `Link: Tableau`, `Link: LI` — header icon buttons

| Property | Value |
|---|---|
| Rows | `''` (invisible anchor) |
| Mark | **Shape**, size 2, colour `A_Color` → `#ffc63c` |
| Custom shapes | `Scifi/icons8-record-player-100.png` · `Professional/icons8-tableau-software-100 black.png` · `Scifi/icons8-antenna-100.png` |
| Tooltips | "See the Classic Viz" · "Find me on Tableau Public" · "Connect with me on LinkedIn" |
| Actions | One URL action each (§11.2) |

### 7.5 KPI band (4 sheets)

All four use an **Automatic** mark with a custom multi-line label, transparent background, mark colour `#d7a64b`, and no shelves.

| Sheet | Label template | Filters beyond the shared stack | Rendered |
|---|---|---|---|
| `KPI-Movie Count` | `500 / <CNTD(Title Id)>` (20 pt bold `#cc5803`) + `Total Sci-Fi Movies` (18 pt `#eae0d5`) | **Only** `Sci-Fi T\|F = true` — no year, vote or Top-N filter | `500 / 11,220` |
| `KPI-Directors` | `<CNTD(Person Name ID)>` + `Directors` | `What did they do ? = "director"` | `427` |
| `KPI-Nom` | `<CNTD(Title Id)>▲` + `Oscar Nominations` | `Best Picture = "Nominated"` | `15 ▲` |
| `KPI-BP` | `<CNTD(Title Id)>★` + `Oscar Best Picture` | `Best Picture = "Winner"` | `1 ★` |

> `KPI-Movie Count` intentionally omits the year and vote filters so the denominator (11,220) represents *all* Sci-Fi films in the source, which is why it exceeds the 10,986 total of the three cohorts in the breakdown panel.

### 7.6 Ranked bar panels (3 sheets)

#### `D-TopIMDb Bar` — "Top 5 IMDb Rated Movies"

| Property | Value |
|---|---|
| Rows | `Title` / `MAX(*Best Picture Symbol)` / `Rank` |
| Columns | `MIN(10.0)` + `MIN(IMDB Rating)` — dual axis, both fixed 0 → 10, axes hidden |
| Mark | **Bar**, three layers |
| Colour | `:Measure Names` → `MIN(10.0)` = `#cc5803` (orange track), `MIN(IMDB Rating)` = `#eae0d5` (cream fill) |
| Text | `MIN(IMDB Rating)` + `Title` (title label `#0a0908`, others `#f4f5f7`) |
| Extra filter | `Title` — **Top 15 by `MIN([imdb rating])` DESC** |
| Column widths | Title 280 px · Rank 36 px · Symbol 28 px |

#### `D-TopVote Bar` — "Top 5 Voted for Movies"
Same construction; columns are `MIN(2460667)` + `MIN(Number Of Votes)` (fixed 0 → 2,583,701), extra filter is **Top 15 by `MIN([number of votes])` DESC**, and rows use `*Rank_Voting`.

#### `D-Best Picture` — "Oscar Best Picture Nominations"

| Property | Value |
|---|---|
| Rows | `Title` / `MAX(*Best Picture Symbol)` / `MIN(*Rank_BP)` |
| Columns | `MIN(1)` (fixed 0 → 1) |
| Mark | **Bar**, `#eae0d5` with matching stroke |
| Text | `MIN(Number Of Votes)`, `Title`, `*Best Picture Text` |
| Extra filter | `Best Picture` ∈ {`Nominated`, `Winner`} |
| Order | Hand-curated via `*Rank_BP` (16 films) |

### 7.7 Breakdown panel (4 sheets)

All four filter to `Groups ∈ {1, 2, 3}` and `Year` 1950–2022.

| Sheet | Construction |
|---|---|
| **`E -Decades`** ("IMDb Sci-Fi Movies Breakdown") | Bar. Columns `Decade × Groups` (Groups axis fixed 0–6), Rows `Number of Titles`, Colour `Groups Name`, Text `Number of Titles`. Row axis hidden; column gridlines `#eae0d5` |
| **`E -Boxes`** | Shape mark using custom image `DL Shapes 400/icons8-rectangle-400.png`. Rows `Groups Name`, Columns `Decade`, Colour `Groups Name`, Text `Number of Titles` — the numeric chips beneath the decade bars |
| **`E-Pie`** | Three-layer Pie (donut). Rows `MIN(0)` ×2, Angle `Number of Titles`, Colour `Groups Name` on the outer layer, hole layer painted `#112a2c`. Text shows `% of Total Number of Titles` and the count |
| **`E-Table`** | Shape-mark table. Rows `Groups Name`, Columns `:Measure Names` filtered to `Number of Titles` and `% of Total Number of Titles` (formatted `p0.0%`). Header width 268 px, row height 32 px, measure labels `#f28e2b`, values `#eae0d5` |

### 7.8 Movie List view

#### `F-Movie Table` — "Top 500 Movie List"

| Property | Value |
|---|---|
| Rows | `Rank` / `Title` / `Title Id` / `Year` / `IMDB Rating` / `Groups Name` / `Rating` / `Country` / `Language` / `Runtime` / `Number Of Votes` |
| Columns | `'IMDb Votes'` (static header) |
| Mark | **Text** |
| Number formats | `IMDB Rating` → `#,##0.0` · `Number Of Votes` → `#,##0` |
| Extra filter | `Top 500 Sci-Fi T\|F = true` (the frozen list) |
| Visibility | Container zone 451 ("Table"), shown only when `View = "3"` |

### 7.9 View switcher buttons (3 sheets)

`Toggle:Radial`, `Toggle:Timeline`, `Toggle:Movie List Table` share one pattern:

| Property | Value |
|---|---|
| Rows | `''` (invisible anchor) |
| Mark | **Square**, size 14.55, base colour `#767f8b` |
| Colour | The matching `Chart Toggle …` boolean → **`#cc5803` when true**, `#767f8b` when false |
| Text | Constant caption — `'Space Radial'` / `'Timeline'` / `'Movie List'`, 10 pt bold `#eae0d5` |
| Detail | Constant payload — `'2'` / `'1'` / `'3'` — read by the parameter action |
| Tooltip | "Click to view the …" |

### 7.10 The rich movie tooltip

Used verbatim on `A-Radial`, `A-Radial Cloud`, `B-Timeline`, `B-Timeline Cloud`, `D-TopIMDb Bar`, `D-TopVote Bar` and `D-Best Picture`:

```
<Title>  (<Year>)                         16 pt bold  #066067
<*Txt_Tagline>                            bold        #0a0908
IMDb Rating <IMDb Rating> (<Votes> Votes) bold        #d7a64b
<*Best Picture Symbol> <*Best Picture Text>  bold     #cc5803
<Plot>                                    8 pt        #0a0908
Genres:   <*Genres (full list text)>      8 pt        label #0a0908 / value #066067
Length:   <Runtime> mins
Rating:   <Rating>
Language: <Language>
Country:  <Country>
```

Simpler default tooltips are retained on the structural sheets (`A-Radial YR`, `E-*`, `F-Movie Table`).

---

## 8. Dashboard specification

### 8.1 Canvas

| Property | Value |
|---|---|
| Name | `Movies Across Time & Space` |
| Sizing | **Fixed** |
| Dimensions | 1600 × 2000 px (min = max) |
| Base background | `#066067` on the outer flow container, over the `A-StarLine` tile (`#151311`) |
| Parameter control title style | Bold, `#555555` |

### 8.2 Layout tree

The author named the containers, and those names are preserved in the file. Tiled structure:

```
zone 162  layout-basic   "Tile: Background Stars"
└─ 414    flow (horz)    background #066067
   └─ 195 flow (vert)    "V Stack", margin 15
      └─ 302 worksheet   A-StarLine            (full-bleed background plate)

zone 367  flow (vert)    "Baseline Vertical (Title/KPIs/Chart/Bars)"
├─ 25     flow (horz)    "Title"       fixed 130 px, border #066067 3 px, fill #0660674e
│  ├─ 73  worksheet      C-Pie Abbv    (genre badge)
│  ├─ 18  title zone     dashboard title
│  ├─ 292 empty          spacer
│  └─ 410 flow (vert)    author block
│     ├─ 411 flow (horz) → Link: Classic · Link: Tableau · Link: LI
│     └─ 397 text        "DESIGNED BY / JOHN JOHANSSON"
├─ 196    empty          "Spacer" 10 px, fill #066067
├─ 403    flow (horz)    "KPIs/Toggles" fixed 140 px
│  ├─ 406 flow (vert)    "Toggle"  → text "Select a View" + 3 toggle sheets + spacer
│  ├─ 335 flow (horz)    "KPIs" (distribute evenly)
│  │  ├─ 364 "Total SF Movies"  → KPI-Movie Count
│  │  ├─ 394 "Directors"        → KPI-Directors
│  │  ├─ 362 "Oscar Nomination" → KPI-Nom
│  │  └─ 359 "Best Picture"     → KPI-BP
│  └─ 399 flow (vert)    "Filters" → "Top 500 Conditions" text panel
├─ 368    empty          "Main Chart Spacer" (reserves the radial/timeline stage)
├─ 451    flow (vert)    "Table" → 446 "Table View" → F-Movie Table
├─ 354    flow (horz)    "Bars" fixed 450 px (distribute evenly)
│  ├─ 353 "Bars: Top IMDb Rated" → D-TopIMDb Bar
│  ├─ 352 "Bars: Top Voted"      → D-TopVote Bar
│  └─ 328 "Bars: Nominations"    → D-Best Picture
└─ 426    flow (vert)    "Breakdown Bar/Pie"
   └─ 420 flow (horz)
      ├─ 432 flow (vert) → text "IMDb Sci-Fi Movies Breakdown" + E -Decades + E -Boxes
      └─ 427 flow (vert) → E-Table + E-Pie
```

**Floating zones** (layered over the tiled stack, all at the main-stage coordinates):

| Zone | Sheet | Position (x, y) | Size (w × h) |
|---:|---|---|---|
| 385 | container "Bubbles" → `B-Timeline Cloud`, `A-Radial Cloud` | 0, 16500 | 100000 × 37500 |
| 205 | container "Planets" → `B-Timeline`, `A-Radial` | 0, 16500 | 100000 × 37500 |
| 380 / 378 / 376 / 374 / 369 | `A-Radial Circle6 / 5 / 4 / 3 / 2` | 0, 16500–16900 | 100000 × 37500 |
| 334 | `A-Radial YR` | 0, 16500 | 100000 × 37500 |
| 384 | `A-Radial Center1` | 15188, 26400 | 27000 × 19000 |
| 381 | `A-Radial Center2` | 15188, 277500 | 27000 × 19000 |
| 454 | text `1970` (gold, 12 pt bold) | 14750, 32900 | 3438 × 1500 |

The z-order of these floating zones is the composition: rings furthest back, year ring, glow clouds, then the star field, with the core sheets and one manual year caption on top.

### 8.3 Dashboard text content

| Zone | Content | Formatting |
|---|---|---|
| 18 (title) | `Movies Across Time & Space ` + `(Re-Viz)` / `500 Top Rated <Select Movie Genre> Movies` | 40 pt bold `#eae0d5`; "(Re-Viz)" in `#cc5803`; subtitle 28 pt bold `#ffc63c`, with the genre parameter interpolated |
| 397 | `DESIGNED BY` / `JOHN JOHANSSON` | 10 pt bold white / 13 pt bold `#ffc63c`, right-aligned |
| 300 | `Select a View` | 12 pt bold `#eae0d5` |
| 402 | `Top 500 Conditions` | 12 pt bold `#eae0d5` |
| 407 | `Sci-Fi Genre` / `Released 1950 - 2022` / `10,000+ IMDb Votes` / `500 Highest IMDb Rating` | 10 pt bold `#cc5803` — the on-canvas statement of the population rule |
| 442 | `IMDb Sci-Fi Movies Breakdown` | 14 pt bold `#eae0d5` |
| 454 | `1970` | 12 pt bold `#ffc63c` |

---

## 9. Interactivity

### 9.1 Parameter actions (4)

| Caption | Source sheet | Trigger | Source field | Target parameter | On clear |
|---|---|---|---|---|---|
| `View: Radial` | `Toggle:Radial` | On select | `MIN('2')` | **View** | Keep value |
| `View: Timeline` | `Toggle:Timeline` | On select | `MIN('1')` | **View** | Keep value |
| `View: List` | `Toggle:Movie List Table` | On select | `MIN('3')` | **View** | Keep value |
| `Movie Info 1` | `A-Radial` | **On hover** | `ATTR([Title])` | **Display Movie Info** | Keep value (`ZZZ` sentinel) |

### 9.2 URL actions (3)

| Caption | Source sheet | Target URL |
|---|---|---|
| `Link:Classic` | `Link: Classic` | `https://public.tableau.com/app/profile/john.johansson/viz/MoviesAcrossTimeSpace/MoviesAcrosstheDecades` |
| `Link: Tableau Public` | `Link: Tableau` | `https://public.tableau.com/app/profile/john.johansson` |
| `Link: LI` | `Link: LI` | `https://www.linkedin.com/in/johnsjohansson/` |

### 9.3 Highlight action (1)

| Property | Value |
|---|---|
| Caption | `Highlight Movie` |
| Trigger | **On hover**, auto-clear on exit |
| Field | `Title` |
| Scope | Dashboard-wide, with 29 sheets explicitly excluded — leaving the highlight effective between the three bar panels and the movie table |

### 9.4 Dynamic Zone Visibility (12 bindings)

View switching is implemented natively: each container's visibility is bound to a boolean field, so no sheet-swap hacks are needed.

| Controlling field | Zone | Zone contents |
|---|---:|---|
| `Chart Toggle: Radial T\|F` | 210 | `A-Radial` |
| `Chart Toggle: Radial T\|F` | 392 | `A-Radial Cloud` |
| `Chart Toggle: Radial T\|F` | 334 | `A-Radial YR` |
| `Chart Toggle: Timeline T\|F` | 206 | `B-Timeline` |
| `Chart Toggle: Timeline T\|F` | 386 | `B-Timeline Cloud` |
| `Chart Toggle: Table T\|F` | 451 | "Table" container → `F-Movie Table` |
| `Chart Toggle: Radial/Timeline T\|F` | 205 | "Planets" container |
| `Chart Toggle: Radial/Timeline T\|F` | 385 | "Bubbles" container |
| `Chart Toggle: Radial/Timeline T\|F` | 368 | "Main Chart Spacer" |
| `Chart Toggle: Radial/Timeline T\|F` | 354 | "Bars" row |
| `Chart Toggle: Radial/Timeline T\|F` | 426 | "Breakdown Bar/Pie" row |
| `Bar Toggle: C T\|F` | 331 | `D-Best Picture` |

**Resulting view states**

| `View` | Visible | Hidden |
|---|---|---|
| `"2"` Space Radial *(default)* | Radial star field, glow, year ring, orbit rings, core, Bars row, Breakdown row | Timeline sheets, Movie List |
| `"1"` Timeline | Timeline + glow, Bars row, Breakdown row | Radial sheets, Movie List |
| `"3"` Movie List | `F-Movie Table` only (below the KPI band) | Both chart stages, Bars row, Breakdown row |

> The five orbit-ring zones and the two core zones are not DZV-bound; they are floating decoration that reads as part of the radial composition.

---

## 10. Design system

### 10.1 Palette

| Token | Hex | Applied to |
|---|---|---|
| Deep teal (structure) | `#066067` | Panel borders (3 px), spacer bars, dashboard ground |
| Teal wash (fills) | `#0660674e` | Every panel interior (teal at ~30 % alpha) |
| Near-black warm | `#151311` | `A-StarLine` background — the visual base of the canvas |
| Core dark | `#03393d`, `#112a2c` | Radial core circle, donut hole |
| Burnt orange (accent / active) | `#cc5803` | "(Re-Viz)", KPI numerals, active toggle, Top-500 cohort, bar tracks, condition list |
| Gold (identity) | `#ffc63c` | Subtitle, author name, link icons, year ring caption |
| Muted gold | `#d7a64b` | KPI mark colour, sheet titles on the radial core, tooltip rating line |
| Cream (primary text / marks) | `#eae0d5` | Titles, labels, bar fills, line marks, gridlines |
| Off-white (labels) | `#f4f5f7` | Mark labels inside panels |
| Inactive grey-blue | `#767f8b` | Inactive toggle buttons, link icon base |
| Deep ink | `#0a0908` | Dark-on-light label text |
| Table accent | `#f28e2b` | `E-Table` measure headers |

### 10.2 Colour encodings

| Encoding | Field | Specification |
|---|---|---|
| Custom sequential — **Space Cyan2** | `MIN(IMDB Rating)` | `#cddfe0` → `#044348`, 7 steps, upper bound fixed at 8.0. Used on `A-Radial` and `B-Timeline` |
| Custom sequential — grey-blue | `MIN(IMDB Rating)` | 11 stops `#f1f1f1 · #e2e5e9 · #d5dbe2 · #c7d0db · #bbc6d4 · #aebccd · #a2b2c5 · #97a8be · #8b9fb7 · #8195b0 · #778da9`. Used on the glow and ring sheets |
| Custom sequential — orange | `MIN(IMDB Rating)` | 11 stops `#f1f1f1` → `#e06236`, lower bound 4.0. Defined on `D-TopIMDb Bar` |
| Categorical — cohorts | `Groups Name` | Top 500 Sci-Fi = `#ffc63c` · Sci-Fi with 10,000+ Votes = `#cc5803` · Remaining Sci-Fi = `#eae0d5` |
| Categorical — cohorts (ordinal twin) | `Groups` | 1 = `#4e79a7` · 2 = `#f28e2b` · 3 = `#e15759` · 4 = `#76b7b2` |
| Categorical — toggle state | `Chart Toggle …` booleans | true = `#cc5803` · false = `#767f8b` |
| Categorical — bar layers | `:Measure Names` | Rating/vote fill = `#eae0d5` · full-scale track = `#cc5803` · counts = `#76b7b2` · votes = `#f28e2b` |
| Categorical — awards | `Best Picture` | Winner = `#ee7422` · Nominated = `#f4d166` · null = `#4e79a7` |
| Single-value | `A_Color` | `'A'` = `#ffc63c` |

### 10.3 Size encoding

`Number Of Votes` is the universal size channel: domain **10,000 → 1,000,000**, mark size 0 → 1. Applying the same domain on every sheet keeps a film's visual weight identical between the radial and timeline views.

### 10.4 Typography

| Level | Spec |
|---|---|
| Workbook default | **Trebuchet MS** (set on element `all`) |
| Dashboard title | 40 pt bold (the ampersand is explicitly set in Arial for glyph quality) |
| Subtitle | 28 pt bold |
| KPI numeral | 20 pt bold `#cc5803` |
| KPI caption | 18 pt `#eae0d5` |
| Panel titles | 14 pt bold `#eae0d5` |
| Section/label headers | 12 pt bold |
| Condition list, buttons | 10 pt bold |
| Mark labels on the radial | 6–8 pt |
| Tooltip title / body | 16 pt bold / 8 pt |

### 10.5 Iconography

| Asset | Use |
|---|---|
| `Scifi/icons8-record-player-100.png` | Link to the original "Classic" viz |
| `Professional/icons8-tableau-software-100 black.png` | Link to the author's Tableau Public profile |
| `Scifi/icons8-antenna-100.png` | Link to LinkedIn |
| `DL Shapes 400/icons8-rectangle-400.png` | Chip backgrounds in `E -Boxes` and `E-Table` |
| `asterisk` (built-in) | Starfield marks |

Icons are referenced from the author's local Tableau Repository shapes folder; they are **not** embedded in the `.twbx`.

### 10.6 Layout craft

- Fixed 1600 × 2000 canvas, so every position is deterministic.
- Consistent panel recipe: 3 px `#066067` border + `#0660674e` fill + a 10 px `#066067` spacer strip beneath, giving each block an underline.
- `distribute-evenly` on the KPI row and the Bars row keeps the four KPIs and three bar panels perfectly equal.
- All analytic sheets are transparent (`#00000000`), so the single starfield sheet shows through the entire composition.
- Sheet titles hidden on every zone except the three bar panels and the movie table.
- Axis lines, gridlines, zero lines, droplines and reference lines are switched off across the board; only deliberate rules remain (for example the `#eae0d5` row axis in `E -Decades`).

---

## 11. Calculation dependency map

```
Year of Release ──► *Angle ──┬─► *Angle Adjusted ───────┬─► *X cos / *Y sin           (A-Radial, A-Radial Cloud)
                             │   (*Angle,Start,*Angle,End)│
                             │                           └─► *X cos/*Y sin Small 6.1  (A-Radial YR)
                             └─► *Angle Adjusted Rings ───► *X/*Y cos/sin Small 2…6    (A-Radial Circle2…6)
                                 (*Angle,Start Rings)
INDEX() ──► *Index ─────────────────────────────────────► all radial coordinate calcs
INDEX() ──► *YoR, *Index x-1 ───────────────────────────► B-Timeline, B-Timeline Cloud

Genres (full list) ──► *Genres (full list text) ──► Sci-Fi T|F ──┐
number of votes ─┐                                               │
Year of Release ─┼─► Target Pop T|F (3 × FIXED LODs) ────────────┼─► Groups ──► (E- sheets)
titleId ─────────┴─► Top 500 Sci-Fi T|F (frozen ID list) ────────┴─► Groups Name ──► (E- sheets, F-Movie Table)

Best Picture ──► *Best Picture Score ──┬─► *Best Picture Symbol  (labels)
                                       └─► *Best Picture Text    (tooltips, Oscar panel)

View parameter ──┬─► Chart Toggle: Radial T|F ─────────► DZV: A-Radial, Cloud, YR + button colour
                 ├─► Chart Toggle: Timeline T|F ───────► DZV: B-Timeline, Cloud + button colour
                 ├─► Chart Toggle: Table T|F ──────────► DZV: Table container + button colour
                 └─► Chart Toggle: Radial/Timeline T|F ► DZV: Planets, Bubbles, Spacer, Bars, Breakdown
Top Bar parameter ──► Bar Toggle: C T|F ────────────────► DZV: D-Best Picture
Select Movie Genre ─┬─► *Genre Abbv ────────────────────► C-Pie Abbv badge
                    └─► dashboard title text
```

Dependency depth is shallow and deliberate: the deepest chain is four levels (`Year of Release → *Angle → *Angle Adjusted → *X cos`). There are no circular references.

---

## 12. How "Top 500" is actually derived

The workbook defines its population **three** different ways, each serving a different purpose. Understanding this is essential before editing anything.

| # | Mechanism | Where | Behaviour |
|---|---|---|---|
| 1 | **Top-N filter** on `Title Id` — top 500 by `MAX([imdb rating])` DESC, over context filters (Sci-Fi, 1950–2022, votes ≥ 10,000) | Every radial/timeline/bar/KPI sheet | **Dynamic.** Recomputes if the data changes |
| 2 | **`Top 500 Sci-Fi T\|F`** — hard-coded list of 500 `titleId` values | `E -Boxes`, `E -Decades`, `E-Pie`, `E-Table`, `F-Movie Table` | **Frozen.** Guarantees the breakdown and the movie list never drift |
| 3 | **`Target Pop T\|F`** — LOD test of the three qualifying conditions | Same five sheets | Defines the *eligible* population (cohort 2), i.e. everything that met the bar but missed the top 500 |

Mechanisms 1 and 2 should agree at the moment of publication. If the extract is refreshed with newer IMDb ratings, mechanism 1 moves and mechanism 2 does not — the frozen list would need regenerating to keep the two panels consistent.

---

## 13. Notes and maintenance considerations

Observations recorded for the maintainer. None of these are defects in the published output; they are things that will matter on the next edit.

1. **Bar panel titles say "Top 5" but each shows 15 rows.** `D-TopIMDb Bar` and `D-TopVote Bar` carry a `Top 15` filter with titles reading *Top 5 IMDb Rated Movies* / *Top 5 Voted for Movies*. The Oscar panel lists 16 films. Titles are inherited from an earlier iteration.
2. **`Display Movie Info` has no visible consumer.** The on-hover parameter action from `A-Radial` writes the film title into the parameter, but the two calculations that would read it (`Show Movie Info: Sheet`, `Show Movie Info: DZN`) are not used in any sheet or zone. The action is harmless but currently inert — and it is the reason the `.twb` carries 453,768 parameter members and weighs 28 MB. Trimming that parameter's allowable-values list to *All* would shrink the workbook dramatically.
3. **`Top Bar` has no on-canvas control.** It is fixed at `"C"`, which keeps `D-Best Picture` permanently visible via DZV. The A/B/D toggle calculations exist but are unused, suggesting a planned rotating-panel feature.
4. **Sheet-level Sci-Fi and director filters duplicate the data source filters.** Deliberate defensive redundancy — each sheet states its own population — but a change to the data source filters would need mirroring in 25 sheets.
5. **Internal names do not match captions.** Extensive duplication during authoring means e.g. the field captioned `*Best Picture Symbol` is internally `[*Best Picture Text (copy)_1523061119528886284]`, and `*X cos Small 2` is internally `[*X cos Small 3 (copy)_…]`. Always edit by caption; §6 records both.
6. **Custom shapes are not packaged.** The five custom images live in the author's local Tableau Repository. Opening the `.twbx` on another machine will render default shapes until those files are copied into `My Tableau Repository/Shapes`.
7. **Extract size.** At 891 MB and 5.18 M rows the extract is far larger than the visualisation needs — the dashboard only ever surfaces ~11,000 Sci-Fi titles. Adding the data source filters at extract-creation time (rather than query time) would cut both the `.hyper` and the publish payload by orders of magnitude.
8. **KPI denominator differs from the breakdown total.** 11,220 (KPI) vs 10,986 (500 + 724 + 9,762) because `KPI-Movie Count` deliberately omits the year and vote filters. Intentional, but worth knowing before "fixing" it.

---

## 14. Rebuild / maintenance runbook

**To re-point the workbook at a refreshed IMDb extract**

1. Refresh the extract, keeping the 18 materialised columns and the two data source filters.
2. Verify `MAX([Year of Release])` — `*Angle` divides by it, so a new maximum year rescales the entire radial automatically (intended behaviour).
3. Regenerate the frozen list in `Top 500 Sci-Fi T|F`: build a temporary view of `Title Id` filtered to Sci-Fi, year ≥ 1950, votes ≥ 10,000, sorted by `MAX([imdb rating])` DESC, take the top 500 IDs, and paste them into the `IN (…)` list.
4. Update the hand-curated `*Rank_BP` order if the Oscar set changes.
5. Refresh the `MIN(2460667)` constant on `D-TopVote Bar` to the new maximum vote count so the background track still represents full scale.
6. Re-check the `Year` filter upper bound (currently 2022) and the on-canvas text panel in zone 407, which states the rule in words.

**To re-tune the radial composition**

| Effect | Parameter |
|---|---|
| Size of the empty core | `*Hole` (7) |
| Ring spacing / ring set radius | `*Hole Small` (6) and the per-ring offsets in the Small calcs |
| Where the sweep starts | `*Angle, Start` (70°) for stars, `*Angle, Start Rings` (40°) for rings |
| How far the sweep travels | `*Angle, End` (405°) |
| Ellipse flattening | The fixed axis ranges on each radial sheet: X −12 → 32, Y −20 → 20 |

**To add a fourth view**

1. Build the sheet(s).
2. Add a member to the `View` parameter list.
3. Add a `Chart Toggle: … T|F` boolean testing the new value.
4. Duplicate a toggle button sheet, swap the caption constant, the payload constant and the colour field.
5. Add a parameter action from the new button to `View`.
6. Bind the new container's Dynamic Zone Visibility to the new boolean, and extend `Chart Toggle: Radial/Timeline T|F` if the shared chrome should appear for the new view.

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
| Physical columns materialised | 18 |
| Source fields in data pane | 31 |
| Calculated fields (used / total) | 50 / 72 |
| Parameters | 11 |
| Data source filters | 2 — see below |

**Data source filters run before every worksheet-level filter, on every sheet:**

- Genres (full list): condition CONTAINS(LOWER([Genres (full list)]), LOWER('Sci-Fi'))
- What did they do ?: keep "director"

### Source fields

31 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Ratings

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Number Of Votes**<br>`number of votes` | integer · Measure (Sum) | 50,863 | yes | 31 | Count of IMDb user votes. The size encoding everywhere, and the quality gate for inclusion. A single IMDb user can cast a maximum of one vote. This field can be missing when we do not yet have an IMDb rating for the title in question.   This can occur either because it does not yet have enough votes, or it has not yet been released.   A TV series rating is not the weighted average of the ratings of individual episodes. Instead, customers vote separately for the rating of the series as a whole via each title’s series page. |
| **IMDB Rating**<br>`imdb rating` | real · Measure (Sum) | 118 | yes | 21 | IMDb weighted user rating, 1–10. The colour encoding across the radial and timeline views. The IMDb Rating for the title. The rating is between 1 and 10 and given to one decimal place. |

#### Election

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Year**<br>`Year of Release` | integer · Dimension | 124 | yes | 31 | Election year. The year of the earliest release of this title globally. |

#### Title identity

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Title Id**<br>`titleId` | string · Dimension | 866,005 | yes | 26 | IMDb title key in `tt0000000` form. The film-level key — all film counts use `COUNTD`. The unique identifier used by IMDB |
| **Title** | string · Dimension | 758,939 | yes | 9 | Film title as displayed on IMDb. The original title text of the title, normally what the title is known as in its original country of release. |

#### Title detail

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Genres (full list)** | string · Dimension | 12,208 | yes | 25 | Comma-separated genre list, e.g. `Action,Adventure,Sci-Fi`. Tested with `CONTAINS` rather than split. The |
| **Language** | string · Dimension | 165 | yes | 8 | Primary language of the film. This is the primary language spoken in the title |
| **Rating**<br>`Certificate (US)` | string · Dimension | 29 | yes | 8 | US content rating — G, PG, PG-13, R and so on. What rating certificate was this film given in USA? |
| **Runtime (Minutes)**<br>`runtime (minutes)` | integer · Measure (Sum) | 507 | yes | 8 | Running time in minutes. The running time of this title in minutes. |
| **Certificate (GB)** | string · Dimension | 21 | yes | 4 | UK BBFC certificate — U, PG, 12A, 15, 18. What rating certificate was this film given in Great Britain? |
| **Color**<br>`color` | string · Dimension | 5 | yes | 0 | Colour information — Color, Black and White, or a combination. Was the film black & white, colour, or a mix? *(hidden, not materialised, not used in any sheet)* |
| **Genres (1st)**<br>`genres (1st)` | string · Dimension | 26 | yes | 0 | First-listed genre — IMDb's primary classification. The *(hidden, not materialised, not used in any sheet)* |
| **Genres (2nd)**<br>`genres (2nd)` | string · Dimension | 24 | yes | 0 | Second-listed genre. The *(hidden, not materialised, not used in any sheet)* |
| **Genres (3rd)**<br>`genres (3rd)` | string · Dimension | 28 | yes | 0 | Third-listed genre. The *(hidden, not materialised, not used in any sheet)* |

#### Awards

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Best Picture** | string · Dimension | 3 | yes | 9 | Academy Award Best Picture status: `Winner`, `Nominated`, or null. Was the picture |

#### Origin

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Country** | string · Dimension | 263 | yes | 8 | Country of production. This is the country of the primary production company associated with this title. ( |
| **Continent** | string · Dimension | 6 | yes | 0 | Continent of production. *(hidden, not materialised, not used in any sheet)* |

#### Text

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Tagline**<br>`tagline` | string · Dimension | 338,446 | yes | 7 | Marketing tagline. Wrapped in guillemets by `*Txt_Tagline`. A tagline is a short description or comment on a title that is often displayed on posters. |
| **Plot (medium)** | string · Dimension | 550,841 | yes | 0 | Shorter plot synopsis variant. *(hidden, not materialised, not used in any sheet)* |

#### Links

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **IMDB Url (title)**<br>`imdbUrl (title)` | string · Dimension | 866,005 | yes | 4 | Deep link to the film's IMDb page. A full URL to see the name or title on www.imdb.com. |
| **Image Url (Title)**<br>`image_url (title)` | string · Dimension | 803,433 | yes | 0 | Poster image URL, served from Amazon media. A URL linking to the primary image associated with this title, such as a movie poster or still frame. *(hidden, not materialised, not used in any sheet)* |
| **IMDB Url (Person)**<br>`imdbUrl (Person)` | string · Dimension | 978,607 | yes | 0 | Deep link to the person's IMDb page. A full URL to see this person on www.imdb.com. *(hidden, not materialised, not used in any sheet)* |

#### People

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Person Name ID** | string · Dimension | 971,095 | yes | 2 | IMDb person key in `nm0000000` form. Basis of distinct people counts. The IMDB id for the person. |
| **What did they do ?** | string · Dimension | 3 | yes | 1 | Credit role — `director`, `actor`, `actress`. The row's reason for existing. What was their job in the movie. We include only |
| **Billing (position in cast list)** | integer · Measure (Sum) | 51 | yes | 0 | Cast-list order; 1 is top billing. The Billing represents the position in the cast list this person appeared. Number 1 represents top billing. For this dataset we only have the top 50 people in each title. *(hidden, not materialised, not used in any sheet)* |
| **Person Name** | string · Dimension | 966,016 | yes | 0 | Cast or crew member's name. *(hidden, not materialised, not used in any sheet)* |
| **Who did they play ?** | string · Dimension | 627,540 | yes | 0 | Character name for acting credits. The name of they character they played in this movie/show *(hidden, not materialised, not used in any sheet)* |

#### Production

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Production Companies (1st)** | string · Dimension | 206,929 | yes | 0 | First-listed production company. The *(hidden, not materialised, not used in any sheet)* |
| **Production Companies (2nd)** | string · Dimension | 121,152 | yes | 0 | Second-listed production company. The *(hidden, not materialised, not used in any sheet)* |
| **Production Companies (3rd)** | string · Dimension | 80,131 | yes | 0 | Third-listed production company. The *(hidden, not materialised, not used in any sheet)* |
| **Production Companies (List)** | string · Dimension | 403,059 | yes | 0 | Comma-separated list of production companies. The full list of Production Companies.   Where a movie has multiple Production Companies, there is no specific order in where they appear in the list. For example. *(hidden, not materialised, not used in any sheet)* |

### Calculated fields in use

50 of the workbook's 72 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| ***Angle**<br>`Calculation_222646726344683528` | real · Measure | LOD | Year | *Angle Adjusted, *Angle Adjusted Rings | 8 |
| ***Angle Adjusted**<br>`Year of Release, Angle (copy)_1009861854723629084` | real · Dimension | Basic | *Angle | *X cos, *X cos Small 6.1, *Y sin, *Y sin Small 6.1 | 3 |
| ***Angle Adjusted Rings**<br>`*Angle Adjusted (copy)_3516544365117474` | real · Dimension | Basic | *Angle | *X cos Small 2, *X cos Small 3, *X cos Small 4, *X cos Small 5 … | 5 |
| ***Best Picture Score**<br>`Calculation_1523061119493234693` | string · Dimension | Basic | Best Picture | *Best Picture Symbol, *Best Picture Symbol ▲, *Best Picture Symbol ★, *Best Picture Text … | 7 |
| ***Best Picture Symbol**<br>`*Best Picture Text (copy)_1523061119528886284` | string · Dimension | Basic | *Best Picture Score | — | 7 |
| ***Best Picture Text**<br>`*Best Picture Score (copy)_1523061119509893128` | string · Dimension | Basic | *Best Picture Score | — | 7 |
| ***Genre Abbv**<br>`Calculation_1523061119512969227` | string · Dimension | Basic | — | — | 1 |
| ***Genres (full list text)**<br>`Genres (full list) (copy)_1261570865657135110` | string · Dimension | Basic | Genres (full list) | Sci-Fi T\|F | 25 |
| ***Index**<br>`Calculation_1261570865630474240` | integer · Measure | Table calc | — | *X cos, *X cos Small 2, *X cos Small 3, *X cos Small 4 … | 8 |
| ***Index x-1**<br>`*Index (copy)_1666331882452328452` | integer · Measure | Table calc | — | — | 2 |
| ***Rank_BP**<br>`*Rank_Voting (copy)_4385433817407492` | integer · Measure | Basic | Title Id | — | 1 |
| ***Rank_Voting**<br>`*Rank_IMDB (copy)_1523061119491559428` | integer · Measure | Table calc | Number Of Votes | — | 1 |
| ***Txt_Tagline**<br>`Calculation_1261570865762828296` | string · Dimension | Basic | Tagline | — | 7 |
| ***X cos**<br>`*X cos (copy)_1261570865648197635` | real · Measure | Basic | *Index, *Angle Adjusted | — | 2 |
| ***X cos Small 2**<br>`*X cos Small 3 (copy)_1753589126861361167` | real · Measure | Basic | *Index, *Angle Adjusted Rings | — | 1 |
| ***X cos Small 3**<br>`*X cos Small 3 (copy)_1753589126859653128` | real · Measure | Basic | *Index, *Angle Adjusted Rings | — | 1 |
| ***X cos Small 4**<br>`*X cos Small 2 (copy)_1753589126858907654` | real · Measure | Basic | *Index, *Angle Adjusted Rings | — | 1 |
| ***X cos Small 5**<br>`*X cos Small (copy)_1753589126855798786` | real · Measure | Basic | *Index, *Angle Adjusted Rings | — | 1 |
| ***X cos Small 6**<br>`*X cos adj (copy)_2236600186041303046` | real · Measure | Basic | *Index, *Angle Adjusted Rings | — | 1 |
| ***X cos Small 6.1**<br>`*X cos Small 6 (copy)_3516544366211108` | real · Measure | Basic | *Index, *Angle Adjusted | — | 1 |
| ***Y sin**<br>`*Y sin (copy)_1261570865648214020` | real · Measure | Basic | *Index, *Angle Adjusted | — | 2 |
| ***Y sin Small 2**<br>`*Y sin Small 3 (copy)_1753589126861328398` | real · Measure | Basic | *Index, *Angle Adjusted Rings | — | 1 |
| ***Y sin Small 3**<br>`*Y sin Small 3 (copy)_1753589126859665417` | real · Measure | Basic | *Index, *Angle Adjusted Rings | — | 1 |
| ***Y sin Small 4**<br>`*Y sin Small 2 (copy)_1753589126858924039` | real · Measure | Basic | *Index, *Angle Adjusted Rings | — | 1 |
| ***Y sin Small 5**<br>`*Y sin Small (copy)_1753589126855872515` | real · Measure | Basic | *Index, *Angle Adjusted Rings | — | 1 |
| ***Y sin Small 6**<br>`*Y sin adj (copy)_2236600186041323527` | real · Measure | Basic | *Index, *Angle Adjusted Rings | — | 1 |
| ***Y sin Small 6.1**<br>`*Y sin Small 6 (copy)_3516544366170147` | real · Measure | Basic | *Index, *Angle Adjusted | — | 1 |
| ***YoR**<br>`*Index *-1 (copy)_1666331882467725322` | integer · Measure | Table calc | Year | — | 2 |
| ***YoR /5**<br>`*YoR (copy)_2236600186036539395` | integer · Measure | Basic | Year | — | 1 |
| **2**<br>`Calculation_1753589126863831060` | integer · Measure | Basic | — | — | 1 |
| **3**<br>`Calculation_1753589126863802387` | integer · Measure | Basic | — | — | 1 |
| **4**<br>`Calculation_1753589126863527954` | integer · Measure | Basic | — | — | 1 |
| **5**<br>`Calculation_1753589126863163409` | integer · Measure | Basic | — | — | 1 |
| **6**<br>`Calculation_1753589126862995472` | integer · Measure | Basic | — | — | 1 |
| **A_Color**<br>`Calculation_3516544505270311` | string · Dimension | Basic | — | — | 3 |
| **Bar Toggle: C T\|F**<br>`Bar Toggle: B T\|F (copy)_3516543516250134` | boolean · Dimension | Basic | — | — | 1 |
| **Chart Toggle: Radial T\|F**<br>`Calculation_3516543423504390` | boolean · Dimension | Basic | — | — | 2 |
| **Chart Toggle: Radial/Timeline T\|F**<br>`Chart Toggle: Radial T\|F (copy)_2446995060924416` | boolean · Dimension | Basic | — | — | 1 |
| **Chart Toggle: Table T\|F**<br>`Chart Toggle: Radial T\|F (copy)_2446995061051393` | boolean · Dimension | Basic | — | — | 2 |
| **Chart Toggle: Timeline T\|F**<br>`Chart Toggle T\|F (copy)_3516543423799303` | boolean · Dimension | Basic | — | — | 2 |
| **Decade**<br>`Calculation_3516543860805661` | string · Dimension | Basic | Year | — | 2 |
| **Groups**<br>`Calculation_4385433640398848` | integer · Dimension | Basic | Top 500 Sci-Fi T\|F, Target Pop T\|F, Sci-Fi T\|F | — | 4 |
| **Groups Name**<br>`Groups (copy)_4385433894932496` | string · Dimension | Basic | Top 500 Sci-Fi T\|F, Target Pop T\|F, Sci-Fi T\|F | — | 5 |
| **Number of people**<br>`Calculation_659777350465458179` | integer · Measure | Basic | Person Name ID | — | 1 |
| **Number of Titles**<br>`Calculation_659777350463983618` | integer · Measure | Basic | Title Id | — | 4 |
| **Rank**<br>`*Rank_Unique_TitleID (copy)_1523061119489306627` | integer · Measure | Table calc | IMDB Rating | — | 2 |
| **Runtime**<br>`Runtime (Minutes) (copy)_2446995078602758` | string · Dimension | Basic | Runtime (Minutes) | — | 1 |
| **Sci-Fi T\|F**<br>`Contains Production Company? (copy)_2043508340865888257` | boolean · Dimension | Basic | *Genres (full list text) | Groups, Groups Name | 25 |
| **Target Pop T\|F**<br>`Calculation_4385433826676744` | boolean · Dimension | LOD | Title Id, Number Of Votes, Genres (full list), Year | Groups, Groups Name, Top 500 % | 5 |
| **Top 500 Sci-Fi T\|F**<br>`Calculation_4385433771683842` | boolean · Dimension | Basic | Title Id | Groups, Groups Name, Top 500 % | 5 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| ***Angle, Start Rings**<br>`*Angle, Start (copy)_3516544364994593` | integer · range | `40` | range 40 to 360 |
| ***Hole Small**<br>`*Hole (copy)_2236600186040856580` | real · any | `6.` | any value |
| **Display Movie Info**<br>`Parameter 1` | string · list | `"Bad Taste"` | list of 453,768 — "0 kara no kaze", "0 no teikô", "0 Uhr 15, Zimmer 9", "0_1_0", "0-18 or A Message from the Sky", "0-41*" … |
| **Top Bar**<br>`Parameter 2` | string · list | `"C"` | list of 4 — "A" → Top IMDb Rated Movie, "B" → Top Voted For Movie, "C", "D" |
| **Production Company**<br>`Parameter 3` | string · any | `"Marvel Studios"` | any value |
| ***Hole**<br>`Parameter 4` | real · any | `7.` | any value |
| ***Angle, Start**<br>`Parameter 5` | integer · range | `70` | range 70 to 360 |
| **Time Chart Type**<br>`Parameter 6` | real · list | `2.` | list of 2 — 1. → Timeline Chart, 2. → Radial |
| **View**<br>`Parameter 7` | string · list | `"2"` | list of 3 — "1" → Timeline, "2" → Space Radial, "3" → Movie List |
| **Select Movie Genre**<br>`Production Company (copy)_2043508340865777664` | string · list | `"Sci-Fi"` | list of 21 — "All" → In Total, "Action", "Adventure", "Animation", "Biography", "Comedy" |
| ***Angle, End**<br>`Start Angle (copy)_1009861854720749594` | integer · range | `405` | range 0 to — |

### How the numbers are computed

**Level-of-detail expressions — 2.** These fix their own level of detail, so they return the same value regardless of what dimensions are in the view:

> `*Angle`, `Target Pop T\|F`

**Table calculations — 5.** These compute across the rendered table, so their result depends on what is in the view at the time:

> `*Index x-1`, `*YoR`, `*Rank_Voting`, `Rank`, `*Index`

The workbook file records only each table calculation's `ordering-type`. The addressing and partitioning described elsewhere in this document are **derived from each sheet's level-of-detail shelf**, not read from the file.

## Appendix A — Worksheet index

| # | Worksheet | Mark type | Role | Visible when |
|---:|---|---|---|---|
| 1 | `A-StarLine` | Text (asterisk) | Background starfield | Always |
| 2 | `A-Radial` | Circle / Circle / Line | Primary star field | View = 2 |
| 3 | `A-Radial Cloud` | Circle / Shape / Line | Glow layer | View = 2 |
| 4 | `A-Radial YR` | Text ×3 | Year ring captions | View = 2 |
| 5–9 | `A-Radial Circle2` … `Circle6` | Circle / Circle / Line | Orbit rings 2–6 | Always (decoration) |
| 10 | `A-Radial Center1` | Heatmap | Core glow | Always (decoration) |
| 11 | `A-Radial Center2` | Circle | Core disc | Always (decoration) |
| 12 | `B-Timeline` | Circle / Shape / Line | Timeline stage | View = 1 |
| 13 | `B-Timeline Cloud` | Circle ×2 / Line | Timeline glow | View = 1 |
| 14 | `C-Pie Abbv` | Pie ×3 | Header genre badge | Always |
| 15 | `KPI-Movie Count` | Automatic | KPI — total films | Always |
| 16 | `KPI-Directors` | Automatic | KPI — directors | Always |
| 17 | `KPI-Nom` | Automatic | KPI — nominations | Always |
| 18 | `KPI-BP` | Automatic | KPI — Best Picture | Always |
| 19 | `D-TopIMDb Bar` | Bar ×3 | Top-rated ranking | View = 1 or 2 |
| 20 | `D-TopVote Bar` | Bar ×3 | Most-voted ranking | View = 1 or 2 |
| 21 | `D-Best Picture` | Bar | Oscar panel | View = 1 or 2, Top Bar = C |
| 22 | `E -Decades` | Bar | Decade breakdown | View = 1 or 2 |
| 23 | `E -Boxes` | Shape | Decade count chips | View = 1 or 2 |
| 24 | `E-Pie` | Pie ×3 | Cohort donut | View = 1 or 2 |
| 25 | `E-Table` | Shape | Cohort summary table | View = 1 or 2 |
| 26 | `F-Movie Table` | Text | Top 500 movie list | View = 3 |
| 27 | `Toggle:Radial` | Square | View button | Always |
| 28 | `Toggle:Timeline` | Square | View button | Always |
| 29 | `Toggle:Movie List Table` | Square | View button | Always |
| 30 | `Link: Classic` | Shape | URL button | Always |
| 31 | `Link: Tableau` | Shape | URL button | Always |
| 32 | `Link: LI` | Shape | URL button | Always |

## Appendix B — Calculated fields excluded from this documentation

Per scope, the following 22 calculated fields exist in the data pane but are **not referenced by any worksheet or by dashboard logic**, and are therefore not documented above. Listed for housekeeping only:

`*Best Picture Symbol ▲` · `*Best Picture Symbol ★` · `*Best Picture T|F` · `▲` · `*Chart Type` · `*IMDB Rating (Color)` · `*Rank_Actor` · `*Rank_Movies` · `Bar Toggle: A T|F` · `Bar Toggle: B T|F` · `Bar Toggle: D T|F` · `Center` · `Contains Production Company?` · `Director` · `Min Year` · `Number Of Votes >10000 T|F` · `Sci-Fi Rank` · `Show Movie Info: DZN` · `Show Movie Info: Sheet` · `Title/Tag` · `Top 500 %` · `Year of Release >= 1950 T|F`

## Appendix C — Package inventory

| File | Size | Notes |
|---|---:|---|
| `Movies Across Time & Space (Re-Viz).twb` | 28.2 MB | Workbook XML; ~27 MB is the `Display Movie Info` parameter domain |
| `Data/Extracts/federated_0uydhq71ekj7nt13f7jp60.hyper` | 891.3 MB | 5,176,667 rows × 18 columns |
| Embedded images | — | None; custom shapes referenced from the local repository |
| **Total `.twbx`** | **438.8 MB** | Compressed |

---

*Documentation compiled from the published `.twbx` package via the Tableau Public MCP server and direct analysis of the workbook XML. All formulas, colour values, dimensions and settings quoted above are taken verbatim from the workbook definition.*
