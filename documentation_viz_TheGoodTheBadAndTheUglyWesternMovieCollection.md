# The Good, The Bad And The Ugly of Western Movies — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `The Good, The Bad And The Ugly of Western Movies` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/TheGoodTheBadAndTheUglyWesternMovieCollection/GBUWestern |
| Competition | **2025 Iron Viz Qualifier** |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `TheGoodTheBadAndTheUglyWesternMovieCollection.twbx`, decompiled and parsed field-by-field |
| Explicit exclusion | 39 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic**. Listed by name only in Appendix A. |

---

## 1. Executive summary

**The Good, The Bad And The Ugly** profiles the complete IMDb Western movie collection —
8,345 films released between 1900 and 2022 — and grades every one against the Sergio Leone
trichotomy: **The Good**, **The Bad**, **The Ugly**, or **Not Rated**.

The signature view is a **sunburst of rays**: each film is a line radiating from a setting sun
on the horizon, positioned by release year around a half-circle and coloured by its GBU grade.
The visual metaphor — a western sunset — carries the subject without a single decorative image.

Beneath it, a **GBU Trail** lists every film from the selected year in four graded columns, and
a **Selected Movie** panel resolves to a single title with its directors, cast, plot and specs.
A **Western Movie History** timeline runs along the bottom, marking events, iconic releases and
release years from 1894 to 2021.

Three chart modes — Sun Radial, GBU Snake, Timeline — swap on **hover**, driven by 30 Dynamic
Zone Visibility bindings. Clicking any film in the sunburst fires four parameter actions at
once, repointing the entire lower half of the dashboard, including a live IMDb poster image and
a working link to that film's IMDb page.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | 26 |
| Dashboards | 1 (`GBU Western`) |
| Data sources | 1 (single extract, no joins) |
| Parameters | 11 |
| Calculated fields **used** | 64 named + 70 in-view ad-hoc |
| Calculated fields unused (not documented) | 39 |
| Dashboard actions | 1 URL, 7 parameter |
| Dynamic Zone Visibility bindings | **30** |
| Global font | Calibri |
| Published size | 1,152,115,575 bytes (**1.1 GB**) |

### 1.2 Headline figures rendered by the dashboard

| GBU grade | Rating band | Films |
|---|---|---:|
| **THE GOOD** | 6.3 – 10.0 | **1,053** |
| **THE BAD** | 5.7 – 6.3 | **898** |
| **THE UGLY** | 0.0 – 5.6 | **938** |
| **NOT RATED** | — | **5,456** |

| Element | Value |
|---|---|
| Oscar Nominations | **20 ▲** |
| Oscar Best Picture | **3 ★** |
| Selected year | **1968** — 26 Good · 31 Bad · 34 Ugly · 45 Not Rated |
| Selected movie | **Once Upon a Time in the West** (1968), THE GOOD, IMDb 8.5 (340,929 votes), 165 min, M rating, dir. Sergio Leone |

**Western Movie Eras** — Silent 1894–1927 · Sound 1927–1930 · Depression 1930–1939 ·
Golden Age 1940–1959 · Revivals 1960–1989 · Neo Western 1989–2022

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `15579335` |
| LUID | `a96cc684-967e-4b7c-9062-d7a86620bd7e` |
| Repository URL | `TheGoodTheBadAndTheUglyWesternMovieCollection` |
| Default view | `GBU Western` |
| Revision | 1.7 |
| First published | 2024-10-31 |
| Last published | 2024-11-01 |
| Last updated | 2025-07-24 |
| View count | 2,482 |
| Favourites | 16 |
| Credits | Sources: Tableau + IMDb, Wikipedia · `#IronViz #DataPlusMovies` |

**Published description**

> Explore the entire IMDb Western Movie Collection, view the GBU of iconic westerns and B-rated gems.
> 2025 Iron Viz
> Parameter Actions | URL Actions | Radial | Timeline | Snake Chart
> #IronViz #DataPlusMovies #Western #IMDb

---

## 3. Data architecture

| Property | Value |
|---|---|
| Data source | IMDb movies and people extract |
| Relations | 1 table (`Extract`) |
| Joins | **None** |
| Custom SQL | None |
| Data source filters | **None** |
| Columns | 33 |

### 3.1 Grain

**Title × person** — one row per film credit. This is the same IMDb model the author's
*Movies Across Time & Space* uses, and it drives the same defensive aggregation: film counts
use `COUNTD([Title Id])`, and the GBU grade is resolved to film level with a `{FIXED}` before
being compared (§5.2).

Unlike the *Movies Across Time & Space* workbook, this one applies **no data source filters** —
the Western restriction is a sheet-level boolean (`Western Genre T|F`, applied on 26 of 26
sheets).

### 3.2 Schema (33 columns)

`Title Id` · `Title` · `Year of Release` · `imdb rating` · `number of votes` ·
`runtime (minutes)` · `Genres (full list)` · `genres (1st/2nd/3rd)` · `Best Picture` ·
`Certificate (US)` / `(GB)` · `Plot` · `Plot (medium)` · `tagline` · `imdbUrl (title)` ·
`image_url (title)` · `Person Name` · `Person Name ID` · `What did they do ?` ·
`Who did they play ?` · `Billing (position in cast list)` · `imdbUrl (Person)` ·
`Production Companies (List/1st/2nd/3rd)` · `color` · `Continent` · `Region` · `Country` ·
`Language`

Note that `image_url (title)` and `imdbUrl (title)` **are** materialised here — unlike the later
Re-Viz workbook — because this dashboard uses them live to display posters and link out.

---

## 4. Parameters

Eleven parameters. Four are geometry, four are click-target stores, three are controls.

| Caption | Type | Domain | Default | Role |
|---|---|---|---|---|
| **Chart** | string list | `R` Sun Radial · `S` GBU Snake · `T` Timeline | `R` | Chart-mode switch |
| **Year Parameter** | integer list | 121 members, 1902 – 2022 | `1968` | Selected year |
| **Movie Title** | string list | **453,767** members | `Once Upon a Time in the West` | Selected film |
| **Movie Image URL** | string list | **377,391** members | an Amazon media URL | Poster image for the selected film |
| **Movie Page URL** | string list | **513,122** members | `https://www.imdb.com/title/tt0064116/` | IMDb link target |
| **\*Angle, Start** | integer range | 2 – 180 | `2` | Sunburst sweep start |
| **\*Angle, End** | integer range | from 0 | `178` | Sunburst sweep end |
| **\*Hole** | real | any | `30.` | Sun radius |
| **\*Hole Small** | real | any | `5.` | Inner ring radius |
| **Select Movie Genre** | string list | 21 genres | `Western` | Genre label |
| **Production Company** | string | any | `Marvel Studios` | Legacy, unused |

The three URL/title parameters carry enormous domains — 453,767 titles, 377,391 image URLs and
513,122 page URLs. Between them they account for essentially the whole **1.1 GB** package size.
They exist because Tableau parameter actions can only write to a parameter whose domain contains
the incoming value, so every possible click target must be enumerated in advance.

---

## 5. Calculated fields in use

64 named calculations plus 70 in-view ad-hoc.

### 5.1 The GBU grading system

This is the workbook's central idea, and it is built in three steps.

**Step 1 — clean the rating**
```
IMDB Rating QA   IF [Number Of Votes] < 100 then 0
                 ELSEIF [IMDB Rating] <= 0 then 0
                 ELSEIF ISNULL([IMDB Rating]) then 0
                 ELSE [IMDB Rating] END
```
Films with fewer than 100 votes, a zero rating or a null rating collapse to `0` — the
"Not Rated" sentinel. Setting the bar at 100 votes is what separates a genuine low score from
an unrated obscurity, and it is why the Not Rated bucket (5,456) is by far the largest.

**Step 2 — resolve to film level**
```
IMDB Rating QA Fixed   { FIXED [Title Id] : MAX([IMDB Rating QA]) }
```
Because the grain is title × person, the rating must be collapsed to one value per film before
grading, or a film with 40 credits would be graded 40 times.

**Step 3 — grade**
```
GBU Calc   IF     [IMDB Rating QA Fixed] =  0    Then 'NOT RATED'
           ELSEIF [IMDB Rating QA Fixed] >= 6.3  Then 'THE GOOD'
           ELSEIF [IMDB Rating QA Fixed] >= 5.7  Then 'THE BAD'
           ELSEIF [IMDB Rating QA Fixed] <  5.7  Then 'THE UGLY' END
```
Used in 11 sheets. `*GBU Color` is the same logic wrapped in `MIN()` for use as a colour
encoding; `GBU Score` is a third copy used where a dimension is needed.

**The four counters** each return 1 for their own band and 0 for the rest:
```
GBU Good Count  SUM(IF … >= 6.3 Then 1 ELSE 0 …)
GBU Bad Count   SUM(IF … >= 5.7 Then 1 ELSE 0 …)
GBU Ugly Count  SUM(IF … <  5.7 Then 1 ELSE 0 …)
GBU NR Count    SUM(IF … =  0   Then 1 ELSE 0 …)
GBU All Count   SUM(IF … Then 1 …)          ← all bands
```
Separate counters rather than a filtered count let all four totals sit in one view without
each re-filtering the data.

```
IMDB Rating QA NR Text   IF [Number Of Votes] < 100 then 'Low Voted'
                         ELSEIF [IMDB Rating] <= 0 then 'Rated'
                         ELSEIF ISNULL([IMDB Rating]) then 'Not Rated' ELSE '' END
```
Explains *why* a film is unrated, shown in the trail tooltips.

### 5.2 Population scoping

```
Western Genre T|F   CONTAINS(LOWER([Genres (full list)]), LOWER('western'))   ← 26 of 26 sheets
Year Selector Filter  [Year Parameter] = [Year of Release]                     ← 6 sheets
Year = Year P T|F     IF [Year of Release] = [Year Parameter] Then True ELSE False END
Selected Movie T|F    IF [Movie Title] = [Title] Then TRUE ELSE FALSE END      ← 4 sheets
```

### 5.3 Sunburst geometry

```
*Angle          2 * PI() * ([Year of Release] - 1906) / ({MAX([Year of Release])} - 1906)
*Angle Adjusted ( [*Angle] * (([*Angle, End] - [*Angle, Start]) / 360)
                + ([*Angle, Start] * 2 * PI() / 360) )
*Index          INDEX()-1
*X cos          ([*Index] + [*Hole]) * COS(MIN([*Angle Adjusted]))
*Y sin          ([*Index] + [*Hole]) * SIN(MIN([*Angle Adjusted]))
```

The same trigonometric chain the author uses elsewhere, with two differences that produce the
sunset rather than a galaxy: the sweep runs **2° to 178°** (a half circle above the horizon
rather than a full ring), and `*Hole` is **30** — a large inner radius that becomes the sun
disc itself. `1906` is the pinned origin year.

### 5.4 The timeline

```
Western Timeline Year     IF [Western Genre T|F] = True Then [Year of Release] ELSE 1903 END
Western Timeline Year T|F IF [Western Genre T|F] = True THEN TRUE ELSE False END

Event Year      IF [Western Timeline Year] In (1903, 1927, 1930, 1940, 1960, 1964, 1990)
                Then [Year of Release] ELSE 1903 END
                /* 1903 First Western Movie (The Great Train Robbery) … */
Event Year T|F  same membership test returning TRUE/FALSE

Iconic Movie Year     IF [Western Timeline Year] In (
                        1903, // The Great Train Robbery
                        1920, // The Mark of Zorro
                        1924, // Greed
                        1925, // The Gold Rush
                        1928, // The Wind
                        1939, // Stagecoach  … )
Iconic Movie Year T|F same membership test

Year Color  IF [Event Year T|F] = TRUE THEN 'Event'
            ELSEIF [Iconic Movie Year T|F] = TRUE THEN 'Iconic Movie'
            ELSEIF [Western Timeline Year T|F] = TRUE THEN 'Western Release' END
Year Size   IF [Event Year T|F] = TRUE THEN 2
            ELSEIF [Iconic Movie Year T|F] = TRUE THEN 1
            ELSEIF [Western Timeline Year T|F] = TRUE THEN 1 ELSE 0 END
```

The curated year lists carry inline comments naming each film or event — the author's own
annotations, preserved in the workbook, which is what makes the timeline maintainable.

### 5.5 Era and decade banding

```
Western Era        IF [Year of Release] <= 1926 Then 'Silent Era'
                   ELSEIF <= 1930 Then 'Sound Era'
                   ELSEIF <= 1939 Then 'Depression Era'
                   ELSEIF <= 1959 Then 'Golden Age'
                   ELSEIF <= 1989 Then 'Revivals' ELSE 'Neo Western' END
Western Era Years  the same bands returning their year ranges as labels
Decade String      Left(Str([Year of Release]),3) + "0's"
```

### 5.6 The GBU Trail grid

The trail arranges each year's films into four graded columns of ranked rows, using three
paired X/Y coordinate fields:

```
*RankU_IMDB QA         RANK_UNIQUE(min([IMDB Rating QA]),'desc')    ← 7 sheets

X Rank Movie By Year      FLOAT(IF [*RankU_IMDB QA] = 1 THEN 1 ELSEIF = 2 THEN 1 … END)
Y Rank Movie By Year      FLOAT(IF [*RankU_IMDB QA] = 1 THEN 0 ELSEIF = 2 THEN 1 … END)
X Rank Movie By Year ALL  FLOAT(IF [*RankU_IMDB QA] <= 120 THEN 1 ELSEIF <= 240 THEN 2
                                ELSEIF <= 360 THEN 3 ELSEIF <= 480 THEN 4 … END)
Y Rank Movie By Year ALL  IF [*RankU_IMDB QA] = 1 THEN 1 ELSEIF = 2 THEN 2 … END
X / Y Rank Movie By Year NR   the same pattern with 100-row columns for the Not Rated list
```

`X` supplies the column, `Y` the row. Different bucket sizes per variant (120 rows per column
for the full list, 100 for Not Rated) are what keep each list the same physical height on the
dashboard despite very different counts.

### 5.7 Title and text handling

```
Count Title Char      LEN([Title])
Title Abbv Part 1     IF [Count Title Char] > 24 THEN " "+LEFT([Title],24) ELSE " "+[Title] END
Title Abbv Part 2     IF [Count Title Char] > 24 AND <= 48
                      THEN " "+LTRIM(RIGHT([Title], LEN([Title])-24))
                      ELSEIF [Count Title Char] > 48 THEN " "+LTRIM(RIGHT(LEFT(…)))  END
Title Abbv (28 Char)  special-cases two specific over-long titles, then falls back to LEFT(…,24)
Plot Abbv (Char 800)  IF LEN([Plot]) > 800 THEN LEFT([Plot],800)+'...' ELSE [Plot] END
*Txt_Tagline          IF ISNULL([Tagline]) then '' ELSE '◄'+REGEXP_REPLACE([Tagline],'"','')+'►' END
*Genres (full list text)  REGEXP_REPLACE([Genres (full list)], "," , ", ")
```

The two-part title split is a manual word-wrap: Tableau will not wrap a dimension label inside
a fixed-width mark, so the title is broken into two fields stacked as two rows.

### 5.8 People

```
Director   {FIXED [Title]: MIN( if [What did they do ?] = 'director' then [Person Name] end )}
Actor      if [What did they do ?] IN ('actor','actress') then [Person Name] end
*Rank_Director  RANK(min([Person Name]),'asc')
```

### 5.9 Awards and counts

```
*Best Picture Score  IF [Best Picture] = 'Winner' then 2 ELSEIF 'Nominated' then 1 ELSE 0 END
Number of Movies     // a count distincy of the number of Titles in the dataset
                     COUNTD([Title Id])
```

### 5.10 Chart-mode booleans (Dynamic Zone Visibility controls)

| Field | Formula | Zones controlled |
|---|---|---:|
| **T\|F Sun Radial Chart** | `IF [Chart] = 'R' Then TRUE ELSE FALSE END` | **21** |
| **T\|F GBU Snake Chart** | `IF [Chart] = 'S' Then TRUE ELSE FALSE END` | 5 |
| **T\|F Timeline Chart** | `IF [Chart] = 'T' Then TRUE ELSE FALSE END` | 3 |
| **T\|F Once Upon a Time In the West** | `IF [Movie Title] = 'Once Upon a Time in the West' Then TRUE ELSE FALSE END` | 1 |

The last one is unusual and worth noting: a zone bound to a **specific film title**, so a
bespoke panel appears only when the default film is selected.

```
*Index x-1  (INDEX()-1)
*YoR        INDEX()+Min([Year of Release])
z           'z'                      ← single-member colour constant
```

---

## 6. Worksheet specifications

26 worksheets, prefixed by dashboard region.

| Prefix | Sheets | Role |
|---|---|---|
| **A1** | `A1-Western Sun`, `A1-Western Sunlines` | The sunburst — the sun disc and the radiating film rays. `Sunlines` is the parameter-action source for year selection |
| **A2** | `A2-Sun GOOD`, `A2-Sun KPI BAD`, `A2-River KPI UGLY`, `A2-River KPI NR` | The four GBU totals beside the sunburst (1,053 / 898 / 938 / 5,456) |
| **A3** | `A3-Era`, `A3-Decade`, `A3-BP Card`, `A3-Nom Card` | Right-hand panel: Western Movie Eras bars, Distribution by Decade bars, Oscar Best Picture (3★) and Nominations (20▲) cards |
| **B1** | `B1-Snake GBU`, `B1-Snake NR` | The GBU Trail grids for the selected year |
| **B2** | `B2-River KPI GOOD/BAD/UGLY/NR` | Per-year counts above each trail column (26 / 31 / 34 / 45) |
| **D1** | `D1-Western Timeline` | The Western Movie History timeline |
| **E1** | `E1-Snake GBU ALL` | The full-collection snake chart (Chart = `S`) |
| **F1** | `F1-Timeline` | The timeline chart mode (Chart = `T`) |
| **G1–G3** | `G1-Actors`, `G2-Directors`, `G3-Movie Plot`, `G3-Movie Specs` | Selected Movie panel: top 10 cast, directors, plot text, and the specs block |
| **H1** | `H1-R Button`, `H1-S Button`, `H1-T Button` | The three chart-mode icons |

---

## 7. Dashboard specification

| Property | Value |
|---|---|
| Name | `GBU Western` |
| Canvas | 1600 × 2026 px |
| Ground | Cream / parchment |
| Accent | Deep red-brown title and panel borders |

**Regions**

```
Masthead         title, subtitle, "Released from 1900 - 2022 | Western Genre | GBU Movie Score"
                 three chart-mode icons top-right ("Hover Over to Display a Chart")
Main stage       Western Movie Collection by Year — the sunburst, with the four GBU totals
                 down the left and Eras / Decade / Oscar cards down the right
Selected Year    "1968 GBU Trail" — four graded columns of films for the chosen year
Selected Movie   plot, tagline, directors, top 10 cast, and movie details
Timeline         Western Movie History, 1894 – 2021, with event / iconic / release markers
Footer           "Project 2025 Iron Viz Qualifier | Designed by John Johansson |
                  Sources Tableau + IMDb, Wikipedia | Hashtag #IronViz #DataPlusMovies"
```

---

## 8. Interactivity

### 8.1 Parameter actions (7)

| Caption | Source sheet | Trigger | Payload | Target |
|---|---|---|---|---|
| `Change Year with Sun` | `A1-Western Sunlines` | on-select | `Year of Release` | **Year Parameter** |
| `Change Movie Title` | *(trail / sunburst)* | on-select | `ATTR(Title)` | **Movie Title** |
| `Change Movie Image` | *(trail / sunburst)* | on-select | `ATTR(Image Url (Title))` | **Movie Image URL** |
| `Change Movie Page URL` | *(trail / sunburst)* | on-select | `ATTR(IMDB Url (title))` | **Movie Page URL** |
| `Change to Radial` | `H1-R Button` | **on-hover** | `'R'` | **Chart** |
| `Change to Snake` | `H1-S Button` | **on-hover** | `'S'` | **Chart** |
| `Change to Timeline` | `H1-T Button` | **on-hover** | `'T'` | **Chart** |

**Clicking one film fires four actions simultaneously** — title, image URL, page URL and
(via the sunburst) year. That is what lets a single click repoint the whole lower half of the
dashboard, load the correct poster, and arm the IMDb link. It is also the reason the three URL
parameters need their vast domains.

Chart switching is hover-driven, which the masthead states explicitly: *"Hover Over to Display
a Chart"*.

### 8.2 URL action (1)

| Caption | Target |
|---|---|
| `Go to IMDb Movie Page` | `<Movie Page URL parameter>` |

A parameterised URL action: the link target is whatever the last clicked film wrote into
`Movie Page URL`, so the button always points at the film currently on display.

### 8.3 Dynamic Zone Visibility (30 bindings)

| Field | Zones |
|---|---:|
| `T\|F Sun Radial Chart` | 21 |
| `T\|F GBU Snake Chart` | 5 |
| `T\|F Timeline Chart` | 3 |
| `T\|F Once Upon a Time In the West` | 1 |

**Resulting states**

| `Chart` | Visible |
|---|---|
| `R` Sun Radial *(default)* | The sunburst plus its 20 supporting zones — totals, eras, decades, Oscar cards |
| `S` GBU Snake | The full-collection snake chart and its 4 supporting zones |
| `T` Timeline | The timeline chart and its 2 supporting zones |

---

## 9. Design system

| Token | Use |
|---|---|
| Cream / parchment | Dashboard ground — the aged-paper western feel |
| Deep red-brown | Titles, panel borders, the "THE BAD" grade |
| Gold / amber | The sun disc, "THE GOOD" grade |
| Purple-grey | "THE UGLY" grade |
| Slate blue-grey | "NOT RATED" rays — the dominant colour of the sunburst by volume |

The colour assignment does real work: because Not Rated is 5,456 of 8,345 films, the sunburst
reads mostly as muted grey rays with the graded films picked out in warm colour — so the eye
lands on exactly the films the dashboard is about.

**Typography** — Calibri, set at workbook level. The masthead uses a heavy serif-adjacent
treatment in deep red for the title.

---

## 10. Rebuild / maintenance runbook

**Refreshing the data**

1. The GBU thresholds (6.3 and 5.7) are hard-coded in five places: `GBU Calc`, `*GBU Color`,
   `GBU Score`, and the four `GBU * Count` fields. Change them as a set or the grades and the
   counters will disagree.
2. The 100-vote quality gate in `IMDB Rating QA` defines the Not Rated bucket. Raising it moves
   films from the graded bands into Not Rated.
3. `*Angle` divides by a pinned origin year of `1906`; update if the collection start moves.

**The three URL parameters**

`Movie Title`, `Movie Image URL` and `Movie Page URL` carry 453,767 / 377,391 / 513,122 members
and are the reason the package is 1.1 GB. They must enumerate every possible click target,
because a parameter action can only write a value the domain already contains. If the dashboard
were restricted to Westerns only at the data source level, these domains could shrink by roughly
98 % — the current breadth exists because no data source filter is applied.

**Extending the timeline**

`Event Year`, `Event Year T|F`, `Iconic Movie Year` and `Iconic Movie Year T|F` carry curated
year lists with inline comments naming each film. Add the year to all four, keeping the comment
convention so the next maintainer can see what each year marks. `Year Color` and `Year Size`
then classify it automatically.

**Adding a chart mode**

1. Add a member to the `Chart` parameter.
2. Add a `T|F <Name> Chart` boolean.
3. Duplicate a `H1-* Button` sheet and add a hover parameter action.
4. Bind the new mode's zones to the new boolean.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | IMDB movies and people Extract |
| Connection class | `federated` |
| Tables / relations | 1 · joins: 0 |
| Physical columns materialised | 33 |
| Source fields in data pane | 30 |
| Calculated fields (used / total) | 64 / 103 |
| Parameters | 11 |
| Data source filters | 0 (none) |

### Source fields

30 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Title detail

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Genres (full list)** | string · Dimension | 12,492 | yes | 26 | Comma-separated genre list, e.g. `Action,Adventure,Sci-Fi`. Tested with `CONTAINS` rather than split. The |
| **Certificate (US)** | string · Dimension | 27 | yes | 7 | US content rating — G, PG, PG-13, R and so on. What rating certificate was this film given in USA? |
| **Language** | string · Dimension | 147 | yes | 7 | Primary language of the film. This is the primary language spoken in the title |
| **Runtime (Minutes)**<br>`runtime (minutes)` | integer · Measure (Sum) | 536 | yes | 7 | Running time in minutes. The running time of this title in minutes. |
| **Certificate (GB)** | string · Dimension | 20 | yes | 1 | UK BBFC certificate — U, PG, 12A, 15, 18. What rating certificate was this film given in Great Britain? |
| **Color**<br>`color` | string · Dimension | 5 | yes | 0 | Colour information — Color, Black and White, or a combination. Was the film black & white, colour, or a mix? *(not used in any sheet)* |
| **Genres (1st)**<br>`genres (1st)` | string · Dimension | 26 | yes | 0 | First-listed genre — IMDb's primary classification. The *(not used in any sheet)* |
| **Genres (2nd)**<br>`genres (2nd)` | string · Dimension | 27 | yes | 0 | Second-listed genre. The *(not used in any sheet)* |
| **Genres (3rd)**<br>`genres (3rd)` | string · Dimension | 28 | yes | 0 | Third-listed genre. The *(not used in any sheet)* |

#### Title identity

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Title Id**<br>`titleId` | string · Dimension | 873,804 | yes | 19 | IMDb title key in `tt0000000` form. The film-level key — all film counts use `COUNTD`. The unique identifier used by IMDB |
| **Year of Release** | integer · Dimension | 123 | yes | 17 | Release year. Drives the radial angle, timeline position and decade grouping. The year of the earliest release of this title globally. |
| **Title** | string · Dimension | 764,786 | yes | 9 | Film title as displayed on IMDb. The original title text of the title, normally what the title is known as in its original country of release. |

#### Ratings

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **IMDB Rating**<br>`imdb rating` | real · Measure (Sum) | 119 | yes | 19 | IMDb weighted user rating, 1–10. The colour encoding across the radial and timeline views. The IMDb Rating for the title. The rating is between 1 and 10 and given to one decimal place. |
| **Number Of Votes**<br>`number of votes` | integer · Measure (Sum) | 50,161 | yes | 17 | Count of IMDb user votes. The size encoding everywhere, and the quality gate for inclusion. A single IMDb user can cast a maximum of one vote. This field can be missing when we do not yet have an IMDb rating for the title in question.   This can occur either because it does not yet have enough votes, or it has not yet been released.   A TV series rating is not the weighted average of the ratings of individual episodes. Instead, customers vote separately for the rating of the series as a whole via each title’s series page. |

#### Links

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Image Url (Title)**<br>`image_url (title)` | string · Dimension | 772,043 | yes | 7 | Poster image URL, served from Amazon media. A URL linking to the primary image associated with this title, such as a movie poster or still frame. |
| **IMDB Url (title)**<br>`imdbUrl (title)` | string · Dimension | 873,804 | yes | 7 | Deep link to the film's IMDb page. A full URL to see the name or title on www.imdb.com. |
| **IMDB Url (Person)**<br>`imdbUrl (Person)` | string · Dimension | 981,391 | yes | 0 | Deep link to the person's IMDb page. A full URL to see this person on www.imdb.com. *(not used in any sheet)* |

#### Origin

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Country** | string · Dimension | 263 | yes | 7 | Country of production. This is the country of the primary production company associated with this title. ( |
| **Continent** | string · Dimension | 6 | yes | 0 | Continent of production. *(not used in any sheet)* |

#### Text

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Tagline**<br>`tagline` | string · Dimension | 354,880 | yes | 7 | Marketing tagline. Wrapped in guillemets by `*Txt_Tagline`. A tagline is a short description or comment on a title that is often displayed on posters. |

#### People

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Person Name** | string · Dimension | 970,154 | yes | 2 | Cast or crew member's name. |
| **What did they do ?** | string · Dimension | 3 | yes | 2 | Credit role — `director`, `actor`, `actress`. The row's reason for existing. What was their job in the movie. We include only |
| **Billing (position in cast list)** | integer · Measure (Sum) | 51 | yes | 1 | Cast-list order; 1 is top billing. The Billing represents the position in the cast list this person appeared. Number 1 represents top billing. For this dataset we only have the top 50 people in each title. |
| **Who did they play ?** | string · Dimension | 615,787 | yes | 1 | Character name for acting credits. The name of they character they played in this movie/show |
| **Person Name ID** | string · Dimension | 981,391 | yes | 0 | IMDb person key in `nm0000000` form. Basis of distinct people counts. The IMDB id for the person. *(not used in any sheet)* |

#### Awards

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Best Picture** | string · Dimension | 3 | yes | 2 | Academy Award Best Picture status: `Winner`, `Nominated`, or null. Was the picture |

#### Production

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Production Companies (1st)** | string · Dimension | 199,664 | yes | 0 | First-listed production company. The *(not used in any sheet)* |
| **Production Companies (2nd)** | string · Dimension | 124,376 | yes | 0 | Second-listed production company. The *(not used in any sheet)* |
| **Production Companies (3rd)** | string · Dimension | 83,928 | yes | 0 | Third-listed production company. The *(not used in any sheet)* |
| **Production Companies (List)** | string · Dimension | 386,624 | yes | 0 | Comma-separated list of production companies. The full list of Production Companies.   Where a movie has multiple Production Companies, there is no specific order in where they appear in the list. For example. *(not used in any sheet)* |

### Calculated fields in use

64 of the workbook's 103 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| ***Angle**<br>`Calculation_222646726344683528` | real · Measure | LOD | Year of Release | *Angle Adjusted | 3 |
| ***Angle Adjusted**<br>`Year of Release, Angle (copy)_1009861854723629084` | real · Dimension | Basic | *Angle | *X cos, *X cos Small, *Y sin, *Y sin Small | 2 |
| ***Best Picture Score**<br>`Calculation_1523061119493234693` | integer · Measure | Basic | Best Picture | — | 2 |
| ***GBU Color**<br>`Calculation_452893239485493254` | string · Measure | Basic | IMDB Rating QA Fixed | — | 7 |
| ***Genres (full list text)**<br>`Genres (full list) (copy)_1261570865657135110` | string · Dimension | Basic | Genres (full list) | — | 7 |
| ***Index**<br>`Calculation_1261570865630474240` | integer · Measure | Table calc | — | *X cos, *X cos Small, *Y sin, *Y sin Small | 2 |
| ***Index x-1**<br>`*Index (copy)_1666331882452328452` | integer · Measure | Table calc | — | Index Snake | 1 |
| ***Rank_Director**<br>`*Rank_Actor (copy)_2555511322843721788` | integer · Measure | Table calc | Person Name | — | 1 |
| ***RankU_IMDB QA**<br>`*Rank_Unique_TitleID (copy)_1523061119489306627` | integer · Measure | Basic | IMDB Rating QA | X Rank Movie By Year, X Rank Movie By Year ALL, X Rank Movie By Year NR, Y Rank Movie By Year … | 7 |
| ***Txt_Tagline**<br>`Calculation_1261570865762828296` | string · Dimension | Basic | Tagline | — | 7 |
| ***X cos**<br>`*X cos (copy)_1261570865648197635` | real · Measure | Basic | *Index, *Angle Adjusted | — | 2 |
| ***Y sin**<br>`*Y sin (copy)_1261570865648214020` | real · Measure | Basic | *Index, *Angle Adjusted | — | 2 |
| ***YoR**<br>`*Index *-1 (copy)_1666331882467725322` | integer · Measure | Table calc | Year of Release | — | 1 |
| **1894**<br>`Event Year (copy)_2555511322184810497` | string · Dimension | Basic | — | — | 1 |
| **1899**<br>`Silent Movie List 1890 (copy)_2555511322191106054` | string · Dimension | Basic | — | — | 1 |
| **1901**<br>`1900 (copy)_2555511322208915470` | string · Dimension | Basic | — | — | 1 |
| **1903**<br>`Silent Movie List 1900 (copy)_2555511322191441927` | string · Dimension | Basic | — | — | 1 |
| **1903 Year**<br>`Calculation_2555511322177974272` | integer · Measure | Basic | — | — | 1 |
| **1904**<br>`Silent Movie List 1900 (copy)_2555511322191720457` | string · Dimension | Basic | — | — | 1 |
| **1905**<br>`1904 (copy)_2555511322619203624` | string · Dimension | Basic | — | — | 1 |
| **Actor**<br>`Director (copy)_3177852492969316362` | string · Dimension | Basic | What did they do ?, Person Name | — | 1 |
| **Count Title Char**<br>`Calculation_3177852493524463639` | integer · Measure | Basic | Title | Title Abbv (28 Char), Title Abbv Part 1, Title Abbv Part 2 | 3 |
| **Decade String**<br>`Calculation_3177852492968398857` | string · Dimension | Basic | Year of Release | — | 1 |
| **Director**<br>`Calculation_452893240144031751` | string · Dimension | LOD | Title, What did they do ?, Person Name | — | 1 |
| **Event Year**<br>`Event Year (copy)_3177852494554783796` | integer · Dimension | Basic | Western Timeline Year, Year of Release | — | 1 |
| **Event Year T\|F**<br>`Year of Release (copy)_3177852494506057769` | boolean · Dimension | Basic | Western Timeline Year | Year Color, Year Size | 1 |
| **GBU All Count**<br>`GBU Bad Count (copy)_2555511322503086112` | integer · Measure | Basic | IMDB Rating QA Fixed | — | 1 |
| **GBU Bad Count**<br>`GBU Good Count (copy)_3177852493662453795` | integer · Measure | Basic | IMDB Rating QA Fixed | — | 2 |
| **GBU Calc**<br>`*GBU (copy)_3177852493505671188` | string · Dimension | Basic | IMDB Rating QA Fixed | — | 11 |
| **GBU Good Count**<br>`GBU Calc (copy)_3177852493662322722` | integer · Measure | Basic | IMDB Rating QA Fixed | — | 2 |
| **GBU NR Count**<br>`GBU Ugly Count (copy)_3177852493662830629` | integer · Measure | Basic | IMDB Rating QA Fixed | — | 2 |
| **GBU Score**<br>`GBU Calc (copy)_2555511322752679978` | string · Dimension | Basic | IMDB Rating QA Fixed | — | 1 |
| **GBU Ugly Count**<br>`GBU Bad Count (copy)_3177852493662756900` | integer · Measure | Basic | IMDB Rating QA Fixed | — | 2 |
| **Iconic Movie Year**<br>`Event Year (copy)_3177852494562037816` | integer · Dimension | Basic | Western Timeline Year, Year of Release | — | 1 |
| **Iconic Movie Year T\|F**<br>`Iconic Movie Year (copy)_3177852494699941946` | boolean · Dimension | Basic | Western Timeline Year | Year Color, Year Size | 1 |
| **IMDB Rating QA**<br>`IMDB Rating (copy)_452893241986580562` | real · Measure | Basic | Number Of Votes, IMDB Rating | *RankU_IMDB QA, IMDB Rating QA Fixed, Western & Rated T\|F | 17 |
| **IMDB Rating QA Fixed**<br>`IMDB Rating QA (copy)_3177852493498081298` | real · Measure | LOD | Title Id, IMDB Rating QA | *GBU Color, Count QA Movies, GBU All Count, GBU Bad Count … | 17 |
| **IMDB Rating QA NR Text**<br>`IMDB Rating QA (copy)_2555511322531172387` | string · Dimension | Basic | Number Of Votes, IMDB Rating | — | 1 |
| **Number of Movies**<br>`Calculation_659777350463983618` | integer · Measure | Basic | Title Id | — | 8 |
| **Plot Abbv (Char 800)**<br>`Plot (copy)_2555511322865565758` | string · Dimension | Basic | — | — | 1 |
| **Selected Movie T\|F**<br>`Calculation_2555511322825121848` | boolean · Dimension | Basic | Title | — | 4 |
| **Title Abbv (28 Char)**<br>`Title Abbv Part 1 (copy)_2555511322754424875` | string · Dimension | Basic | Title, Count Title Char | — | 1 |
| **Title Abbv Part 1**<br>`Title Abbv (copy 2)_3177852493640831006` | string · Dimension | Basic | Count Title Char, Title | — | 3 |
| **Title Abbv Part 2**<br>`Title Abbv Part 1 (copy)_3177852493641035807` | string · Dimension | Basic | Count Title Char, Title | — | 2 |
| **T\|F GBU Snake Chart**<br>`T\|F Sun Radial Chart (copy)_2555511322762203181` | boolean · Dimension | Basic | — | — | 1 |
| **T\|F Once Upon a Time In the West**<br>`T\|F Timeline Chart (copy)_2555511322972287043` | boolean · Dimension | Basic | — | — | 1 |
| **T\|F Sun Radial Chart**<br>`Calculation_2555511322761662508` | boolean · Dimension | Basic | — | — | 1 |
| **T\|F Timeline Chart**<br>`T\|F Sun Radial Chart (copy)_2555511322762395694` | boolean · Dimension | Basic | — | — | 1 |
| **Western Era**<br>`Year of Release (copy)_3177852492947607556` | string · Dimension | Basic | Year of Release | — | 1 |
| **Western Era Years**<br>`Western Era (copy)_2555511322471653403` | string · Dimension | Basic | Year of Release | — | 1 |
| **Western Genre T\|F**<br>`Contains Genre? (copy)_452893242114383975` | boolean · Dimension | Basic | Genres (full list) | Western Timeline Year, Western Timeline Year T\|F | 26 |
| **Western Timeline Year**<br>`Calculation_3177852494545141811` | integer · Dimension | Basic | Western Genre T\|F, Year of Release | Event Year, Event Year T\|F, Iconic Movie Year, Iconic Movie Year T\|F | 1 |
| **Western Timeline Year T\|F**<br>`Western Timeline Year (copy)_3177852494558871607` | boolean · Dimension | Basic | Western Genre T\|F | Year Color, Year Size | 1 |
| **X Rank Movie By Year**<br>`Y Rank Movie By Year (copy)_452893242007031900` | real · Measure | Basic | *RankU_IMDB QA | — | 1 |
| **X Rank Movie By Year ALL**<br>`X Rank Movie By Year (copy)_2555511322611384358` | real · Measure | Basic | *RankU_IMDB QA | — | 1 |
| **X Rank Movie By Year NR**<br>`X Rank Movie By Year (copy)_2555511322527899682` | real · Measure | Basic | *RankU_IMDB QA | — | 1 |
| **Y Rank Movie By Year**<br>`Calculation_452893242006937691` | real · Measure | Basic | *RankU_IMDB QA | — | 1 |
| **Y Rank Movie By Year ALL**<br>`Y Rank Movie By Year (copy)_2555511322611425319` | integer · Measure | Basic | *RankU_IMDB QA | — | 1 |
| **Y Rank Movie By Year NR**<br>`Y Rank Movie By Year (copy)_2555511322526593057` | integer · Measure | Basic | *RankU_IMDB QA | — | 1 |
| **Year = Year P T\|F**<br>`Calculation_2555511322955112514` | boolean · Dimension | Basic | Year of Release | — | 4 |
| **Year Color**<br>`Event Year (copy)_3177852494509088810` | string · Dimension | Basic | Event Year T\|F, Iconic Movie Year T\|F, Western Timeline Year T\|F | — | 1 |
| **Year Selector Filter**<br>`Year Selector Color (copy)_452893242884649071` | boolean · Dimension | Basic | Year of Release | — | 6 |
| **Year Size**<br>`Year Size (copy)_3177852494703108156` | integer · Dimension | Basic | Event Year T\|F, Iconic Movie Year T\|F, Western Timeline Year T\|F | — | 1 |
| **z**<br>`Calculation_2555511322472562718` | string · Dimension | Basic | — | — | 4 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| ***Hole Small**<br>`*Hole (copy)_2236600186040856580` | real · any | `5.` | any value |
| **Movie Image URL**<br>`Chart (copy)_2555511322747777065` | string · list | `"https://m.media-amazon.com/images/M/MV5BODQ3NDExOGYtMzI3Mi00NWRlLTkwNjAtNjc4MDgzZGJiZTA1XkEyXkFqcGdeQXVyMjUzOTY1NTc@._V1_.jpg"` | list of 377,391 — "https://m.media-amazon.com/images/M/MV5BM2E0MDUzYzAtNGI0ZC00YzMwLTlmZDQtMjMzMjJjZTVhOGFlXkEyXkFqcGdeQXVyNTIxNzMwNw@@._V1_.jpg", "https://m.media-amazon.com/images/M/MV5BM2E0MGE1ZmUtYzVmNy00NDdmLTgwOTYtMTg5ZjQ1NjYzMDJhXkEyXkFqcGdeQXVyNDk5MDQ2Mw@@._V1_.jpg", "https://m.media-amazon.com/images/M/MV5BM2E0MGFiNDMtMzkyMS00NWJhLWJjNGUtYzhmMWE3YmY2MjY4XkEyXkFqcGdeQXVyNzYyMDEwNTM@._V1_.jpg", "https://m.media-amazon.com/images/M/MV5BM2E0MGI2NWEtMWM5MC00MGYzLWFhMTUtZmZmOTMxOTc1OWEzXkEyXkFqcGdeQXVyNzMzMjU5NDY@._V1_.jpg", "https://m.media-amazon.com/images/M/MV5BM2E0MGIwNmItZjMxMS00OWYxLWFmZjYtOGRkYmJkOTVlNDAwXkEyXkFqcGdeQXVyMTQxNzMzNDI@._V1_.jpg", "https://m.media-amazon.com/images/M/MV5BM2E0MGZmN2QtYWUzZi00NjdlLTg1NTAtNGQzMTcxYTNjMzU0XkEyXkFqcGdeQXVyODE5NzE3OTE@._V1_.jpg" … |
| **Movie Page URL**<br>`Movie Image URL (copy)_2555511322768597039` | string · list | `"https://www.imdb.com/title/tt0064116/"` | list of 513,122 — "https://www.imdb.com/title/tt0000502/", "https://www.imdb.com/title/tt0000574/", "https://www.imdb.com/title/tt0000591/", "https://www.imdb.com/title/tt0000615/", "https://www.imdb.com/title/tt0000630/", "https://www.imdb.com/title/tt0000675/" … |
| **Year Parameter**<br>`Parameter 1` | integer · list | `1968` | list of 121 — 1902, 1903, 1904, 1905, 1906, 1907 … |
| **Production Company**<br>`Parameter 3` | string · any | `"Marvel Studios"` | any value |
| ***Hole**<br>`Parameter 4` | real · any | `30.` | any value |
| ***Angle, Start**<br>`Parameter 5` | integer · range | `2` | range 2 to 180 |
| **Chart**<br>`Parameter 7` | string · list | `"R"` | list of 3 — "R" → Sun Radial, "S" → GBU Snake, "T" → Timeline |
| **Select Movie Genre**<br>`Production Company (copy)_2043508340865777664` | string · list | `"Western"` | list of 21 — "All" → In Total, "Action", "Adventure", "Animation", "Biography", "Comedy" |
| ***Angle, End**<br>`Start Angle (copy)_1009861854720749594` | integer · range | `178` | range 0 to — |
| **Movie Title**<br>`Year Parameter (copy)_3177852493965852711` | string · list | `"Once Upon a Time in the West"` | list of 453,767 — "0 kara no kaze", "0 no teikô", "0 Uhr 15, Zimmer 9", "0_1_0", "0-18 or A Message from the Sky", "0-41*" … |

### How the numbers are computed

**Level-of-detail expressions — 3.** These fix their own level of detail, so they return the same value regardless of what dimensions are in the view:

> `*Angle`, `Director`, `IMDB Rating QA Fixed`

**Table calculations — 4.** These compute across the rendered table, so their result depends on what is in the view at the time:

> `*Index x-1`, `*YoR`, `*Rank_Director`, `*Index`

The workbook file records only each table calculation's `ordering-type`. The addressing and partitioning described elsewhere in this document are **derived from each sheet's level-of-detail shelf**, not read from the file.

## Appendix A — Calculated fields excluded from this documentation

39 calculated fields exist in the data pane but are not referenced by any worksheet or dashboard
logic. They comprise abandoned ranking variants, alternative GBU threshold experiments, unused
radial-geometry copies, and inherited helpers from the author's other IMDb workbooks.

---

## Appendix B — Package inventory

| File | Size |
|---|---:|
| Workbook XML | large — dominated by three enumerated URL/title parameter domains |
| Embedded IMDb extract | included |
| **Total `.twbx`** | **1.1 GB** |

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas and settings are quoted verbatim from the workbook
definition; rendered figures are read from the published view.*
