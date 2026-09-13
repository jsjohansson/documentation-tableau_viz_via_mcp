# Movies Across Time & Space (original) — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `Movies Across Time & Space` — the original ("Classic") build |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/MoviesAcrossTimeSpace/MoviesAcrosstheDecades |
| Successor | Rebuilt three years later as **Movies Across Time & Space (Re-Viz)** — documented separately in `documentation_viz_MoviesAcrossTimeSpaceRe-Viz.md` |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `MoviesAcrossTimeSpace.twbx`, decompiled and parsed field-by-field |
| Explicit exclusion | 11 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic**. Listed in Appendix A. |

---

## 1. Executive summary

**Movies Across Time & Space** visualises the **300 top-rated films in a chosen genre** as a
radial galaxy: each film is a point placed by release year (angle) and rank within that year
(radius), sized by vote count and coloured by IMDb rating, orbiting a glowing green core ringed
with decade markers.

This is the **original** build, published November 2023. The author rebuilt it from scratch in
August 2026 as *Movies Across Time & Space (Re-Viz)*, describing that version as reflecting
"three years of progress across my design capabilities, Tableau upgrades, and analytics
knowledge". Read side by side the two workbooks are a useful record of how the author's
technique changed — §11 sets out the differences concretely.

The core trigonometry is already here and is essentially unchanged in the rebuild. What is
absent is everything built *around* it: this version has **no actions and no Dynamic Zone
Visibility at all**. View switching is done with a native parameter control (a radio button in
the masthead) rather than clickable sheets, and there is no movie-list view, no KPI band, no
hover interactivity.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | 20 |
| Dashboards | **2** (`Movies Across the Decades`, `Viz - Stripped`) |
| Data sources | 1 (single extract, no joins) |
| Parameters | 8 |
| Calculated fields **used** | 34 named + 41 in-view ad-hoc |
| Calculated fields unused (not documented) | 11 |
| Dashboard actions | **0** |
| Dynamic Zone Visibility bindings | **0** |
| Global font | Tw Cen MT |
| Published size | 1,151,429,858 bytes (**1.1 GB**) |

### 1.2 Headline figures rendered by the dashboard

| Element | Value |
|---|---|
| Population | **300 Top Rated Sci-Fi Movies** |
| Criteria | Highest IMDb Rating · Released After 1950 · 10,000+ Votes |
| Oscar Best Picture | **1 ★** |
| Oscar Nominations | **15 ▲** |

**Top 10 IMDb Rated Movies** — Inception 8.8 · Interstellar 8.7 · Star Wars: Episode V 8.7 ·
The Matrix 8.7 · Star Wars: Episode IV 8.6 · Terminator 2 8.6 · Alien 8.5 · Back to the Future
8.5 · The Prestige 8.5 · Aliens 8.4

**Top 10 Voted for Movies** — Inception 2,460,667 · The Matrix 1,985,278 · Interstellar
1,978,103 · Star Wars: Episode IV 1,410,494 · The Prestige 1,391,979 · Avatar 1,359,036 ·
Star Wars: Episode V 1,338,673 · Back to the Future 1,259,680 · Guardians of the Galaxy
1,240,886 · Avengers: Endgame 1,209,637

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `13390540` |
| LUID | `533dd875-d5f6-458b-9de6-c79da8d873f8` |
| Repository URL | `MoviesAcrossTimeSpace` |
| Default view | `Movies Across the Decades` |
| Revision | 2.9 |
| First published | 2023-11-13 |
| Last published | 2026-08-09 |
| View count | 7,318 |
| Favourites | 61 |
| Credits | Powered by Tableau + IMDb · Created for Data + Movie |

**Published description**

> Check out this Viz that I am calling 'Movies Across Time & Space', where you can visualize the
> 300 top rated movies in a genre across time and discover your own movie watchlist.
> #DataPlusMovies #Movies #SciFi #Radial #IMDb

---

## 3. Data architecture

| Property | Value |
|---|---|
| Data source | IMDb movies and people extract |
| Relations | 1 table (`Extract`) |
| Joins | **None** |
| Custom SQL | None |
| Data source filters | **None** |

### 3.1 Grain

Title × person — one row per film credit, the same IMDb model used across the author's movie
workbooks. Film counts therefore use `COUNTD([Title Id])` and people counts
`COUNTD([Person Name ID])`.

**No data source filters are applied.** Genre restriction is handled entirely by the
`Contains Genre?` boolean (§5.3), which is applied on 18 of 20 sheets. This is a meaningful
difference from the Re-Viz, which pushes the Sci-Fi and director filters down to the data source.

---

## 4. Parameters

| Caption | Type | Domain | Default | Role |
|---|---|---|---|---|
| **Select Movie Genre** | string list | 21 genres | `Sci-Fi` | The genre being profiled |
| **Chart** | string list | 2 members | `2` | Timeline / Space Radial switch |
| **Time Chart Type** | real list | 2 members | `1.` | Legacy chart-type switch |
| **\*Angle, Start** | integer range | 90 – 360 | `90` | Radial sweep start |
| **\*Angle, End** | integer range | from 0 | `405` | Radial sweep end |
| **\*Hole** | real | any | `7.` | Main radial inner radius |
| **\*Hole Small** | real | any | `6.` | Ring set base radius |
| **Production Company** | string | any | `Marvel Studios` | Legacy, unused |

Both `Select Movie Genre` and `Chart` are exposed as **native Tableau parameter controls** in
the masthead ("Select a Movie Genre", "Select a View" with Timeline / Space Radial radio
buttons). There are no custom button sheets — which is why the workbook has zero actions.

---

## 5. Calculated fields in use

34 named calculations plus 41 in-view ad-hoc.

### 5.1 The radial geometry engine

```
*Angle           2 * PI() * ([Year of Release] - 1950) / ({MAX([Year of Release])} - 1950)
*Angle Adjusted  ( [*Angle] * (([*Angle, End] - [*Angle, Start]) / 360)
                 + ([*Angle, Start] * 2 * PI() / 360) )
*Index           INDEX()-1
*X cos           ([*Index] + [*Hole]) * COS(MIN([*Angle Adjusted]))
*Y sin           ([*Index] + [*Hole]) * SIN(MIN([*Angle Adjusted]))
```

Used in 8 sheets. `1950` is the pinned origin year, matching the "Released After 1950"
criterion in the subtitle. The sweep runs **90° to 405°** — a 315° arc leaving a wedge open,
which produces the comet-tail silhouette.

**The five orbit rings** follow one template, offsetting from `*Hole Small` (6):

```
*X cos Small n  ([*Index] + ([*Hole Small] - k)) * COS(MIN([*Angle Adjusted]))
*Y sin Small n  ([*Index] + ([*Hole Small] - k)) * SIN(MIN([*Angle Adjusted]))
```

| Pair | Offset *k* | Base radius | Sheet |
|---|---:|---:|---|
| `*X/*Y cos/sin Small 2` | 4 | 2 | `A-Radial Circle2` |
| `… Small 3` | 3 | 3 | `A-Radial Circle3` |
| `… Small 4` | 2 | 4 | `A-Radial Circle4` |
| `… Small 5` | 1 | 5 | `A-Radial Circle5` |
| `… Small 6` | 0 | 6 | `A-Radial Circle6`, `A-Radial YR` |

Ring stroke widths come from five constant calculations returning their own integer: `2`, `3`,
`4`, `5`, `6`.

> **Difference from the Re-Viz:** here **all** rings share the single `*Angle Adjusted` field.
> The rebuild introduces a second geometry chain (`*Angle Adjusted Rings`, driven by a separate
> `*Angle, Start Rings` parameter at 40°) so the rings can sweep wider than the star field. This
> version has one sweep for everything.

### 5.2 Timeline geometry

```
*YoR        INDEX() + Min([Year of Release])
*Index x-1  (INDEX()-1) * -1
```
Horizontal position and downward stacking for the Timeline view.

### 5.3 Population scoping

```
Contains Genre?   If [Select Movie Genre] = 'All' then True
                  ELSE CONTAINS(LOWER([Genres (full list)]), LOWER([Select Movie Genre])) END
```

Used in **18 of 20 sheets**. A single elegant field: it takes the genre parameter and tests it
against the film's genre list, with an `All` escape hatch. This is what makes the workbook
genuinely generic across 21 genres — the Re-Viz, by contrast, hard-codes Sci-Fi at the data
source and uses the genre parameter only for labelling.

### 5.4 Awards encoding

```
*Best Picture Score   IF [Best Picture] = 'Winner' then 2
                      ELSEIF [Best Picture] = 'Nominated' then 1 ELSE 0 END
*Best Picture Symbol  IF [*Best Picture Score] = 2 then '★'
                      ELSEIF [*Best Picture Score] = 1 then '▲' ELSE '' END
*Best Picture Text    IF [*Best Picture Score] = 2 then 'Oscar Best Picture'
                      ELSEIF [*Best Picture Score] = 1 then 'Oscar Nomination' END
```

A **numeric** intermediate (2 / 1 / 0). The Re-Viz replaces this with a string code (`'BP'` /
`'ON'` / `''`) — functionally equivalent, but the numeric form here also allows the score to be
summed or sorted, which the string version cannot.

### 5.5 Ranking

```
*Rank_IMDB    RANK_UNIQUE(min([IMDB Rating]),'desc')
*Rank_Voting  RANK_UNIQUE(min([Number Of Votes]),'desc')
```
Drive the two Top 10 bar panels.

### 5.6 Text presentation

```
*Genres (full list text)  REGEXP_REPLACE([Genres (full list)], "," , ", ")
*Txt_Tagline              IF ISNULL([Tagline]) then ''
                          ELSE '◄'+REGEXP_REPLACE([Tagline],'"','')+'►' END
*Genre Abbv               CASE [Select Movie Genre] When "All" then "All"
                          When "Action" then "Act" … When "Sci-Fi" then "SF" … END
*YoR /5                   IF [Year of Release] IN (1900,1910,…,2040)
                          OR [Year of Release] IN (1905,1915,…,2045)
                          then [Year of Release] END
```
`*YoR /5` emits a year only every five years, labelling the decade ring 1951 · 1955 · 1960 …
2020 without collision.

### 5.7 Chart switching

```
*Chart Type  [Chart]      ← used in 12 sheets
```

A pass-through of the `Chart` parameter, used as a filter on each sheet. When the parameter
does not match, the sheet returns no marks and collapses. **This is the pre-DZV sheet-swapping
technique** — the Re-Viz replaces it with 12 Dynamic Zone Visibility bindings, which is cleaner
because the container collapses rather than the sheet rendering empty.

### 5.8 Counts

```
Number of people  COUNTD([Person Name ID])
```

---

## 6. Worksheet specifications

20 worksheets across two dashboards.

| Sheet | Role |
|---|---|
| `A-StarLine` | Full-canvas starfield background plate |
| `A-Radial` | The primary star field — films as points |
| `A-Radial Cloud` | Soft glow duplicate layered beneath |
| `A-Radial YR` | The decade year labels on the outer ring |
| `A-Radial Circle2` … `Circle6` | Five concentric orbit rings |
| `A-Radial Center1`, `Center2` | The glowing green core — a density layer over a filled disc |
| `A-Timeline` | The alternate timeline view |
| `A-Timeline Cloud` | Its glow duplicate |
| `A-Hidden Legend` | Supplies the Votes size legend and IMDb Rating colour ramp shown top-left |
| `C-Pie Abbv` | The "SF" genre badge in the masthead |
| `D-TopIMDb Bar` | Top 10 IMDb Rated Movies |
| `D-TopVote Bar` | Top 10 Voted for Movies |
| `E-BP Card` | Oscar Best Picture card (1★) |
| `E-Nom Card` | Oscar Nomination card (15▲) |

### 6.1 The two dashboards

| Dashboard | Size | Sheets | Purpose |
|---|---|---:|---|
| **Movies Across the Decades** | 1600 × 950 | 19 | The published view — full composition including all five orbit rings and both core sheets |
| **Viz - Stripped** | 1600 × 950 | 12 | A reduced variant omitting `A-Radial Circle2`–`Circle6` and the two `Center` sheets — the galaxy without its rings or core |

`Viz - Stripped` is not published as its own Tableau Public entry; it exists in the workbook as
an alternative composition.

---

## 7. Dashboard specification

| Property | Value |
|---|---|
| Name | `Movies Across the Decades` |
| Canvas | 1600 × 950 px, fixed |
| Ground | Dark green-black |
| Masthead | Deep green band |

**Regions**

```
Masthead   "SF" genre badge · "Movies Across Time & Space" · orange subtitle
           "300 Top Rated Sci-Fi Movies" · criteria line
           right side: Select a Movie Genre control · Select a View radio
           Designed By / Powered By / Created for credits
Left rail  Votes size legend · IMDb Rating colour ramp · Oscar legend
           Oscar Best Picture card (1★) · Oscar Nomination card (15▲)
Main stage The radial galaxy with decade ring labels 1951–2020
Lower      Top 10 IMDb Rated Movies | Top 10 Voted for Movies
```

---

## 8. Interactivity

**None.** The workbook has **zero actions** and **zero Dynamic Zone Visibility bindings**.

Both controls are native Tableau parameter controls placed on the dashboard:

| Control | Parameter | Effect |
|---|---|---|
| "Select a Movie Genre" dropdown | `Select Movie Genre` | Re-filters all 18 sheets via `Contains Genre?` |
| "Select a View" radio (Timeline / Space Radial) | `Chart` | Sheets filter on `*Chart Type`; non-matching sheets render empty and collapse |

This is the single largest architectural difference from the Re-Viz, which carries 4 parameter
actions, 3 URL actions, 1 highlight action and 12 DZV bindings.

---

## 9. Design system

| Token | Use |
|---|---|
| Dark green-black | Dashboard ground, starfield |
| Deep green | Masthead band, panel borders, bar fills |
| Orange | Subtitle, Oscar Best Picture star, accent |
| Gold / amber | Decade ring labels |
| Lime-yellow | The glowing core |
| Pale blue-grey | Film points, sized by votes |
| Cream | Body text |

**Colour encodings**

| Encoding | Field | Spec |
|---|---|---|
| Colour | IMDb Rating | Sequential ramp, legend shown 7.0 → 8.0 |
| Size | Number of Votes | 10,000 → 1,000,000+, legend shown in six steps |
| Label | `*Best Picture Symbol` | ★ Best Picture · ▲ Nomination |

**Typography** — **Tw Cen MT**, set at workbook level. This is the only workbook in the
portfolio using it; the Re-Viz switches to Trebuchet MS.

---

## 10. Rebuild / maintenance runbook

**Refreshing the data**

1. `*Angle` divides by a pinned origin year of `1950`, matching the "Released After 1950"
   criterion. Update both together if the population window changes.
2. `{MAX([Year of Release])}` is a table-scoped LOD, so the outer edge of the galaxy re-bases
   automatically as newer films arrive.
3. The 300-film population and the 10,000-vote threshold are applied as sheet-level filters, not
   calculations — check them on each sheet after a refresh.

**Re-tuning the composition**

| Effect | Control |
|---|---|
| Core size | `*Hole` (7) |
| Ring spacing | `*Hole Small` (6) and the per-ring offsets |
| Sweep start / end | `*Angle, Start` (90°) / `*Angle, End` (405°) |

**Changing genre**

Nothing to edit — `Contains Genre?` handles all 21 genres from the parameter, and `*Genre Abbv`
supplies the badge text. This workbook is more genre-portable than its successor.

---

## 11. Differences from the Re-Viz

Recorded because the two workbooks share a lineage and the comparison is the author's own stated
purpose for the rebuild.

| Aspect | Original (2023) | Re-Viz (2026) |
|---|---|---|
| Worksheets | 20 | 32 |
| Dashboards | 2 (one unpublished variant) | 1 |
| Canvas | 1600 × 950 | 1600 × 2000 |
| Population | 300 top-rated, any genre via parameter | 500 top-rated Sci-Fi, fixed at source |
| Data source filters | None | Two (Sci-Fi genre, director role) |
| Genre switching | Fully generic — `Contains Genre?` | Parameter used for labelling only |
| View switching | Native parameter control + `*Chart Type` filter | 4 parameter actions + 12 DZV bindings |
| Actions | **0** | 4 parameter, 3 URL, 1 highlight |
| Ring geometry | One sweep for stars and rings | Separate `*Angle Adjusted Rings` chain |
| Awards encoding | Numeric score (2/1/0) | String code (`BP`/`ON`) |
| KPI band | None | Four KPI cards |
| Movie list view | None | Full Top 500 table |
| Decade breakdown | None | Decade bars, donut and cohort table |
| Font | Tw Cen MT | Trebuchet MS |
| Palette | Green / orange / gold | Teal / burnt orange / gold |

The geometry engine is substantially the same in both. What the three years added was
interaction architecture — actions and Dynamic Zone Visibility in place of parameter controls
and empty-sheet collapse — and a much larger analytical surface around the same centrepiece.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | IMDB movies and people Extract |
| Connection class | `federated` |
| Tables / relations | 1 · joins: 0 |
| Physical columns materialised | 0 |
| Source fields in data pane | 30 |
| Calculated fields (used / total) | 34 / 45 |
| Parameters | 8 |
| Data source filters | 0 (none) |

### Source fields

30 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Title identity

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Title Id**<br>`titleId` | string · Dimension | 898,114 | yes | 20 | IMDb title key in `tt0000000` form. The film-level key — all film counts use `COUNTD`. The unique identifier used by IMDB |
| **Year of Release** | integer · Dimension | 124 | yes | 20 | Release year. Drives the radial angle, timeline position and decade grouping. The year of the earliest release of this title globally. |
| **Title** | string · Dimension | 788,137 | yes | 8 | Film title as displayed on IMDb. The original title text of the title, normally what the title is known as in its original country of release. |

#### Ratings

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **IMDB Rating**<br>`imdb rating` | real · Measure (Sum) | 110 | yes | 20 | IMDb weighted user rating, 1–10. The colour encoding across the radial and timeline views. The IMDb Rating for the title. The rating is between 1 and 10 and given to one decimal place. |
| **Number Of Votes**<br>`number of votes` | integer · Measure (Sum) | 50,707 | yes | 20 | Count of IMDb user votes. The size encoding everywhere, and the quality gate for inclusion. A single IMDb user can cast a maximum of one vote. This field can be missing when we do not yet have an IMDb rating for the title in question.   This can occur either because it does not yet have enough votes, or it has not yet been released.   A TV series rating is not the weighted average of the ratings of individual episodes. Instead, customers vote separately for the rating of the series as a whole via each title’s series page. |

#### Title detail

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Genres (full list)** | string · Dimension | 12,232 | yes | 18 | Comma-separated genre list, e.g. `Action,Adventure,Sci-Fi`. Tested with `CONTAINS` rather than split. The |
| **Certificate (US)** | string · Dimension | 25 | yes | 6 | US content rating — G, PG, PG-13, R and so on. What rating certificate was this film given in USA? |
| **Language** | string · Dimension | 166 | yes | 6 | Primary language of the film. This is the primary language spoken in the title |
| **Runtime (Minutes)**<br>`runtime (minutes)` | integer · Measure (Sum) | 497 | yes | 6 | Running time in minutes. The running time of this title in minutes. |
| **Certificate (GB)** | string · Dimension | 23 | yes | 4 | UK BBFC certificate — U, PG, 12A, 15, 18. What rating certificate was this film given in Great Britain? |
| **Color**<br>`color` | string · Dimension | 5 | yes | 0 | Colour information — Color, Black and White, or a combination. Was the film black & white, colour, or a mix? *(not used in any sheet)* |
| **Genres (1st)**<br>`genres (1st)` | string · Dimension | 26 | yes | 0 | First-listed genre — IMDb's primary classification. The *(not used in any sheet)* |
| **Genres (2nd)**<br>`genres (2nd)` | string · Dimension | 24 | yes | 0 | Second-listed genre. The *(not used in any sheet)* |
| **Genres (3rd)**<br>`genres (3rd)` | string · Dimension | 28 | yes | 0 | Third-listed genre. The *(not used in any sheet)* |

#### Awards

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Best Picture** | string · Dimension | 3 | yes | 8 | Academy Award Best Picture status: `Winner`, `Nominated`, or null. Was the picture |

#### Origin

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Country** | string · Dimension | 249 | yes | 6 | Country of production. This is the country of the primary production company associated with this title. ( |
| **Continent** | string · Dimension | 6 | yes | 0 | Continent of production. *(not used in any sheet)* |

#### Text

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Tagline**<br>`tagline` | string · Dimension | 388,718 | yes | 6 | Marketing tagline. Wrapped in guillemets by `*Txt_Tagline`. A tagline is a short description or comment on a title that is often displayed on posters. |

#### Links

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **IMDB Url (title)**<br>`imdbUrl (title)` | string · Dimension | 898,114 | yes | 4 | Deep link to the film's IMDb page. A full URL to see the name or title on www.imdb.com. |
| **Image Url (Title)**<br>`image_url (title)` | string · Dimension | 803,433 | yes | 0 | Poster image URL, served from Amazon media. A URL linking to the primary image associated with this title, such as a movie poster or still frame. *(not used in any sheet)* |
| **IMDB Url (Person)**<br>`imdbUrl (Person)` | string · Dimension | 978,607 | yes | 0 | Deep link to the person's IMDb page. A full URL to see this person on www.imdb.com. *(not used in any sheet)* |

#### People

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Person Name ID** | string · Dimension | 978,607 | yes | 2 | IMDb person key in `nm0000000` form. Basis of distinct people counts. The IMDB id for the person. |
| **Billing (position in cast list)** | integer · Measure (Sum) | 51 | yes | 0 | Cast-list order; 1 is top billing. The Billing represents the position in the cast list this person appeared. Number 1 represents top billing. For this dataset we only have the top 50 people in each title. *(not used in any sheet)* |
| **Person Name** | string · Dimension | 966,016 | yes | 0 | Cast or crew member's name. *(not used in any sheet)* |
| **What did they do ?** | string · Dimension | 3 | yes | 0 | Credit role — `director`, `actor`, `actress`. The row's reason for existing. What was their job in the movie. We include only *(not used in any sheet)* |
| **Who did they play ?** | string · Dimension | 627,540 | yes | 0 | Character name for acting credits. The name of they character they played in this movie/show *(not used in any sheet)* |

#### Production

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Production Companies (1st)** | string · Dimension | 206,929 | yes | 0 | First-listed production company. The *(not used in any sheet)* |
| **Production Companies (2nd)** | string · Dimension | 121,152 | yes | 0 | Second-listed production company. The *(not used in any sheet)* |
| **Production Companies (3rd)** | string · Dimension | 80,131 | yes | 0 | Third-listed production company. The *(not used in any sheet)* |
| **Production Companies (List)** | string · Dimension | 403,059 | yes | 0 | Comma-separated list of production companies. The full list of Production Companies.   Where a movie has multiple Production Companies, there is no specific order in where they appear in the list. For example. *(not used in any sheet)* |

### Calculated fields in use

34 of the workbook's 45 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| ***Angle**<br>`Calculation_222646726344683528` | real · Measure | LOD | Year of Release | *Angle Adjusted | 8 |
| ***Angle Adjusted**<br>`Year of Release, Angle (copy)_1009861854723629084` | real · Dimension | Basic | *Angle | *X cos, *X cos Small 2, *X cos Small 3, *X cos Small 4 … | 8 |
| ***Best Picture Score**<br>`Calculation_1523061119493234693` | integer · Measure | Basic | Best Picture | *Best Picture Symbol, *Best Picture Symbol ▲, *Best Picture Symbol ★, *Best Picture Text | 8 |
| ***Best Picture Symbol**<br>`*Best Picture Text (copy)_1523061119528886284` | string · Dimension | Basic | *Best Picture Score | — | 6 |
| ***Best Picture Text**<br>`*Best Picture Score (copy)_1523061119509893128` | string · Dimension | Basic | *Best Picture Score | — | 6 |
| ***Chart Type**<br>`Calculation_2236600186019762178` | string · Dimension | Basic | — | — | 12 |
| ***Genre Abbv**<br>`Calculation_1523061119512969227` | string · Dimension | Basic | — | — | 1 |
| ***Genres (full list text)**<br>`Genres (full list) (copy)_1261570865657135110` | string · Dimension | Basic | Genres (full list) | — | 6 |
| ***Index**<br>`Calculation_1261570865630474240` | integer · Measure | Table calc | — | *X cos, *X cos Small 2, *X cos Small 3, *X cos Small 4 … | 8 |
| ***Index x-1**<br>`*Index (copy)_1666331882452328452` | integer · Measure | Table calc | — | — | 3 |
| ***Rank_IMDB**<br>`*Rank_Unique_TitleID (copy)_1523061119489306627` | integer · Measure | Table calc | IMDB Rating | — | 1 |
| ***Rank_Voting**<br>`*Rank_IMDB (copy)_1523061119491559428` | integer · Measure | Table calc | Number Of Votes | — | 1 |
| ***Txt_Tagline**<br>`Calculation_1261570865762828296` | string · Dimension | Basic | Tagline | — | 6 |
| ***X cos**<br>`*X cos (copy)_1261570865648197635` | real · Measure | Basic | *Index, *Angle Adjusted | — | 2 |
| ***X cos Small 2**<br>`*X cos Small 3 (copy)_1753589126861361167` | real · Measure | Basic | *Index, *Angle Adjusted | — | 1 |
| ***X cos Small 3**<br>`*X cos Small 3 (copy)_1753589126859653128` | real · Measure | Basic | *Index, *Angle Adjusted | — | 1 |
| ***X cos Small 4**<br>`*X cos Small 2 (copy)_1753589126858907654` | real · Measure | Basic | *Index, *Angle Adjusted | — | 1 |
| ***X cos Small 5**<br>`*X cos Small (copy)_1753589126855798786` | real · Measure | Basic | *Index, *Angle Adjusted | — | 1 |
| ***X cos Small 6**<br>`*X cos adj (copy)_2236600186041303046` | real · Measure | Basic | *Index, *Angle Adjusted | — | 2 |
| ***Y sin**<br>`*Y sin (copy)_1261570865648214020` | real · Measure | Basic | *Index, *Angle Adjusted | — | 2 |
| ***Y sin Small 2**<br>`*Y sin Small 3 (copy)_1753589126861328398` | real · Measure | Basic | *Index, *Angle Adjusted | — | 1 |
| ***Y sin Small 3**<br>`*Y sin Small 3 (copy)_1753589126859665417` | real · Measure | Basic | *Index, *Angle Adjusted | — | 1 |
| ***Y sin Small 4**<br>`*Y sin Small 2 (copy)_1753589126858924039` | real · Measure | Basic | *Index, *Angle Adjusted | — | 1 |
| ***Y sin Small 5**<br>`*Y sin Small (copy)_1753589126855872515` | real · Measure | Basic | *Index, *Angle Adjusted | — | 1 |
| ***Y sin Small 6**<br>`*Y sin adj (copy)_2236600186041323527` | real · Measure | Basic | *Index, *Angle Adjusted | — | 2 |
| ***YoR**<br>`*Index *-1 (copy)_1666331882467725322` | integer · Measure | Table calc | Year of Release | — | 3 |
| ***YoR /5**<br>`*YoR (copy)_2236600186036539395` | integer · Measure | Basic | Year of Release | — | 1 |
| **2**<br>`Calculation_1753589126863831060` | integer · Measure | Basic | — | — | 1 |
| **3**<br>`Calculation_1753589126863802387` | integer · Measure | Basic | — | — | 1 |
| **4**<br>`Calculation_1753589126863527954` | integer · Measure | Basic | — | — | 1 |
| **5**<br>`Calculation_1753589126863163409` | integer · Measure | Basic | — | — | 1 |
| **6**<br>`Calculation_1753589126862995472` | integer · Measure | Basic | — | — | 1 |
| **Contains Genre?**<br>`Contains Production Company? (copy)_2043508340865888257` | boolean · Dimension | Basic | Genres (full list) | — | 18 |
| **Number of people**<br>`Calculation_659777350465458179` | integer · Measure | Basic | Person Name ID | — | 2 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| ***Hole Small**<br>`*Hole (copy)_2236600186040856580` | real · any | `6.` | any value |
| **Production Company**<br>`Parameter 3` | string · any | `"Marvel Studios"` | any value |
| ***Hole**<br>`Parameter 4` | real · any | `7.` | any value |
| ***Angle, Start**<br>`Parameter 5` | integer · range | `90` | range 90 to 360 |
| **Time Chart Type**<br>`Parameter 6` | real · list | `1.` | list of 2 — 1. → Timeline Chart, 2. → Radial |
| **Chart**<br>`Parameter 7` | string · list | `"2"` | list of 2 — "1" → Timeline, "2" → Space Radial |
| **Select Movie Genre**<br>`Production Company (copy)_2043508340865777664` | string · list | `"Sci-Fi"` | list of 21 — "All" → In Total, "Action", "Adventure", "Animation", "Biography", "Comedy" |
| ***Angle, End**<br>`Start Angle (copy)_1009861854720749594` | integer · range | `405` | range 0 to — |

### How the numbers are computed

**Level-of-detail expressions — 1.** These fix their own level of detail, so they return the same value regardless of what dimensions are in the view:

> `*Angle`

**Table calculations — 5.** These compute across the rendered table, so their result depends on what is in the view at the time:

> `*Index x-1`, `*YoR`, `*Rank_Voting`, `*Rank_IMDB`, `*Index`

The workbook file records only each table calculation's `ordering-type`. The addressing and partitioning described elsewhere in this document are **derived from each sheet's level-of-detail shelf**, not read from the file.

## Appendix A — Calculated fields excluded from this documentation

11 calculated fields exist in the data pane but are not referenced by any worksheet or dashboard
logic:

`*Angle (copy)` · `*Best Picture Symbol ▲` · `*Best Picture Symbol ★` · `*Contains Star Trek` ·
`*Rank_Actor` · `1` · `Min Year` · `0` · `Contains Production Company?` · `Number of titles` ·
`*IMDB Rating (Color)`

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas and settings are quoted verbatim from the workbook
definition; rendered figures are read from the published view.*
