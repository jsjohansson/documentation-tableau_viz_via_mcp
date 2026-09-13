# All In The Wins — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `All In The Wins` — *Road to the World Series* |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/AllIntheWins/RoadtotheWorldSeries |
| Recognition | Tableau **Viz of the Day** (`#VOTD`) · `#SportsVizSunday` |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `AllIntheWins.twbx` → `All In The Wins  #VOTD.twb`, decompiled and parsed field-by-field |
| Explicit exclusion | 11 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic**. They are listed by name only in Appendix B. |
| Companion file | `documentation_viz_AllIntheWins_README.md` — same facts, condensed overview format |

---

## 1. Executive summary

**All In The Wins** profiles the **World Series champion and runner-up teams of the last five
seasons (2021–2025)**, tracing each team's full 162-game regular season and its post-season
run to the title.

The organising idea is a **season calendar grid**: every game of the regular season is a
rounded square, laid out with week number across and day of the week down, coloured by win or
loss. One glance gives the shape of a season — streaks as runs of green, slumps as clusters of
grey — in a way a line chart cannot. A winged donut reports the win ratio, and four split
bars break it down by home/away and day/night.

Below that, the post-season is drawn as a ladder of best-of series (Wild Card → Division →
League Championship → World Series), each a row of game squares showing the series result.
The lower third carries full batting and pitching statistics for the selected team.

Selection is entirely hover-driven: five World Series banners across the top pick the season,
a Champion / Runner-Up toggle picks which of the two finalists to view, and a Batting /
Pitching toggle swaps the statistics table. There are no filter controls on the canvas — five
parameter actions and four Dynamic Zone Visibility bindings do all of it.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | 38 |
| Dashboards | 1 (`Road to the World Series`) |
| Data sources | 1 (4 Excel connections, 8 relations, 7 joins) |
| Parameters | 4 |
| Total fields in data pane | 201 (147 base + 54 calculated) |
| Calculated fields **used** | 43 named + 60 in-view ad-hoc |
| Calculated fields unused (not documented) | 11 |
| Dashboard canvas | 1600 × 1800 px, fixed |
| Global font | Arial |
| Dashboard actions | 5 parameter actions |
| Dynamic Zone Visibility bindings | 4 |

### 1.2 Headline figures rendered by the dashboard

Default state — 2025 season, Champion team:

| Element | Value |
|---|---|
| Team | **Los Angeles Dodgers** — 2025 World Series Champions |
| Record | **93 Wins · 69 Losses** |
| Win Ratio | **57.4 %** (93 / 162) |
| Home win rate | 64.2 % |
| Away win rate | 50.6 % |
| Day win rate | 62.2 % |
| Night win rate | 55.6 % |

**Post-season ladder**

| Series | Format | Result |
|---|---|---|
| Wild Card Series | 2-of-3 to win | 2 – 0 |
| Division Series | 3-of-5 to win | 3 – 1 |
| League Championship Series | 4-of-7 to win | 4 – 0 |
| World Series | 4-of-7 to win | 4 – 3 |

**Season batting totals**

| Runs | Hits | 2B | 3B | HR | RBI | SB | TB | BB | SO |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 825 | 1,384 | 257 | 21 | 244 | 791 | 88 | 2,415 | 580 | 1,353 |

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `17069458` |
| LUID | `87639880-55b3-471d-83d4-f29f168d964c` |
| Repository URL | `AllIntheWins` |
| Default view | `Road to the World Series` |
| Revision | 2.4 |
| First published | 2025-06-06 21:13 UTC |
| Last published / updated | 2025-11-11 19:21 UTC |
| Published size | 2,833,030 bytes (2.7 MB) |
| View count | 14,069 |
| Favourites | 54 |
| Data download allowed | Yes |
| Data credit | Baseball-Reference · logos from SportsLogos |

**Published description**

> #VizOfTheDay exploring the Win Rates of the World Series Teams in the last 4 years. Hover
> over the baseball diamond icons to see Viz instructions.
> #MLB #Baseball #VOTD #SportsVizSunday #Sports #WingedDonut #ShapeChart

---

## 3. Data architecture

### 3.1 Connection and model

| Property | Value |
|---|---|
| Data source | `Games_Champions (Baseball_Reference)` |
| Connection class | `excel-direct` × 4, behind a `federated` wrapper |
| Relations | 8 |
| Joins | **7** |
| Custom SQL | None |
| Materialisation | Extract, 82 columns |

**Named connections**

| Connection | Supplies |
|---|---|
| `Games_Baseball_Reference` | Regular-season game log |
| `Team Names` | Team identity and World Series outcome |
| `Post Season Games` | Playoff game log |
| `Stats_Baseball_Reference` | Batting and pitching statistics |

**Relations**

| Table | Role |
|---|---|
| `Team Groups` | One row per team-season; carries `League Champion` (Champion / Runner-Up) and `Wild Card Bye` |
| `Games_Champions` | Regular-season games — 162 rows per team-season |
| `PS Games` | Post-season games with series identity |
| `Batting` | Per-player batting lines |
| `Pitching` | Per-player pitching lines |
| `Team Name Opponite` / `Team Name Opponite1` | Opponent name lookups for the regular and post seasons |

`Unique` is the join key that ties every table to a team-season.

### 3.2 Grain

The joined model is **mixed-grain**, which is the single most important fact about this
workbook. One row can represent a game, a batter, or a pitcher depending on which branch of
the join it came from.

Three consequences visible throughout:

- Game counts use `COUNT([Gm#])` with an `IF` guard rather than a plain row count, so only
  rows that genuinely are games are counted.
- Every game-count calculation names its condition explicitly (`IF [W/L Group] = 'Win' AND
  [Home/Away Game] = 'Home' THEN [Gm#] ELSE NULL END`) so the counting is scoped rather than
  inherited from context.
- Two **team-season filters** (§6.1) are applied on 33–34 sheets to reduce the joined data to
  a single team-season before anything is measured.

### 3.3 Data source filters

**None.** Scoping is done entirely by the `Team Selector Filter` and `Year Selector Filter`
booleans applied at sheet level.

### 3.4 Materialised schema (82 columns)

Grouped by source table:

| Group | Columns |
|---|---|
| **Team Groups** | `Unique`, `Year`, `League Champion`, `Team Location`, `Team Name Short`, `Wild Card Bye` |
| **Regular-season games** | `Gm#`, `Tm`, `Z2` (home/away marker), `Opp`, `W/L`, `R`, `RA`, `W - L`, `D/N`, `W`, `L`, `Day Name`, `2 Games`, `2 Games Order`, `Year`, `Game Date`, `Week Number` |
| **Post-season games** | `Series`, `Post Gm#`, `Post Series Gm#`, `Series #`, `Opp`, `W/L`, `R`, `RA`, `W - L_S`, `Day Name`, `Game Date` |
| **Opponent lookups** | `TeamID Opponite`, `Team Location Opponite`, `Team Name Opponite Short` |
| **Batting** | `Rk`, `Player`, `Pos`, `G`, `AB`, `R`, `H`, `2B`, `3B`, `HR`, `RBI`, `SB`, `BB`, `SO`, `BA`, `SLG`, `OPS`, `TB`, `HBP` |
| **Pitching** | `Rk`, `Player`, `ERA`, `G`, `SHO`, `IP`, `H`, `R`, `HR`, `BB`, `SO`, `HBP`, `BK`, `BF`, `FIP`, `WHIP` |

The source exposes 106 columns; **50 are not materialised** — advanced sabermetrics and
administrative fields (`WAR`, `ERA+`, `OPS+`, `Rbat+`, `cLI`, `rOBA`, `Attendance`, `Awards`,
`Streak`, `Time`, `Orig. Scheduled` and similar) retained in the source but excluded from the
extract.

### 3.5 Data-pane organisation

| Folder | Contents |
|---|---|
| `Game Counts` | `Wins`, `Games`, `Losses`, `Games - Wins`, `Win / Games Ratio`, `W/L 1/2 Key` |
| `Home/Away Games` | The full home/away split family (13 fields) |
| `Day/Night Games` | The full day/night split family (11 fields) |
| `Playoff Season` | 23 post-season field aliases (`… (PS Games)`) |
| `Team Base` | Team Groups aliases (`… (Team Groups)`) |
| `Team Selector (PA)` | `Team Selector`, `Team Selector Filter`, `League Champion`, and the two unused variants |
| `Year Selector (PA)` | `Year Selector`, `Year Selector Filter`, `Year` |
| `Stats PA` | `Batting Stats T\|F`, `Pitching Stats T\|F` |

**Groups** — 27, all auto-generated by the parameter actions and tooltips.

---

## 4. Field dictionary — base fields

| Caption | Source | Type | Purpose |
|---|---|---|---|
| `Unique` | all tables | string | The team-season join key |
| `Year (Team Groups)` | Team Groups | integer | Season; compared against the Year parameter |
| `League Champion (Team Groups)` | Team Groups | string | `Champion` / `Runner-Up`; compared against the Team parameter |
| `Team Location` / `Team Name Short` | Team Groups | string | "Los Angeles" / "Dodgers" |
| `Wild Card Bye` | Team Groups | boolean | Whether the team skipped the Wild Card round |
| `Gm#` | Games | integer | Game number 1–162; the unit counted everywhere |
| `Tm` | Games | string | Team code; drives the logo shape encoding |
| `Z2` | Games | string | `@` marks an away game |
| `W/L` | Games | string | `W`, `L`, `W-wo`, `L-wo` (walk-off variants) |
| `D/N` | Games | string | Day or night game |
| `Day Name` | Games | string | Weekday — the grid's vertical axis |
| `Week Number` | Games | real | Season week 1–29 — the grid's horizontal axis |
| `2 Games` / `2 Games Order` | Games | string / integer | Doubleheader flag and which game of the pair |
| `Opp` | Games | string | Opponent code; drives opponent logos |
| `Series` | PS Games | string | Wild Card / Division / LCS / World Series |
| `Post Series Gm#` | PS Games | real | Game number within a series |
| `W/L (PS Games)` | PS Games | string | Post-season result |
| `W - L S` | PS Games | string | Running series score |
| Batting / Pitching stat columns | Stats | numeric | Player statistics tables |

---

## 5. Parameters

| Caption | Internal name | Type | Domain | Default | Role |
|---|---|---|---|---|---|
| **World Series Year** | `[Parameter 1]` | real list | `2021` · `2022` · `2023` · `2024` · `2025` | `2025` | Season selector — the five banners |
| **World Series Team** | `[Parameter 2]` | string list | `Champion` · `Runner-Up` | `Champion` | Which finalist to view |
| **Stats** | `[World Series Team (copy)_1184165244975685632]` | string list | `Batting` · `Pitching` | `Batting` | Which statistics table is shown |
| **Color** | `[Parameter 3]` | string list | `WL` Win/Lose · `HA` Home/Away · `DN` Day/Night · `A` Home Attendance | `WL` | Colour scheme for the season grid |

---

## 6. Calculated fields in use

43 named calculations plus 60 in-view ad-hoc calculations. Grouped by role.

### 6.1 Team-season scoping — the two most important fields

Because the model joins five tables at mixed grain, every sheet must first reduce the data to
one team-season. Two boolean filters do this, and between them they appear on nearly every
sheet in the workbook.

#### `Year Selector Filter` — used in **34 sheets**
```
[World Series Year] = [Year (Team Groups)]
```

#### `Team Selector Filter` — used in **33 sheets**
```
[World Series Team] = [League Champion (Team Groups)]
```

Applied together as `= TRUE`, they isolate a single team-season. Every count, ratio and
statistic downstream assumes this scoping has happened.

`Team Selector` and `Year Selector` are identical formulas used for a different job — they
colour the banner and toggle marks to show which option is active (2 sheets and 1 sheet
respectively).

### 6.2 Game outcome classification

| Field | Formula | Purpose |
|---|---|---|
| **Home/Away Game** | `IF [Z2] = '@' Then 'Away' ELSE 'Home' END` | Baseball-Reference marks away games with `@`; this turns that convention into a usable dimension. Used in 5 sheets |
| **Win/Loss Text** | `IF [W/L] IN ('W','W-wo') THEN 'Win' ELSEIF [W/L] IN ('L','L-wo') THEN 'Loss' ELSE Null END` | Collapses walk-off variants into plain Win/Loss |
| **Win/Loss Text Plural** | Same, returning `'Wins'` / `'Losses'` | Legend labels |
| **Win/Loss ▲** | `IF [W/L] IN ('W','W-wo') THEN '▲' ELSE Null END` | Glyph for wins |
| **Win/Loss ▼** | `IF [W/L] IN ('L','L-wo') THEN '▲' ELSE Null END` | Glyph for losses |
| **W/L 1/2 Key** | `FLOAT(IF [W/L Group] = 'Win' THEN 1 ELSE 2 END)` | Ordering key that places the Wins key left of the Losses key |

### 6.3 The game-count family

Every count follows one pattern: count `Gm#` under a condition, null otherwise. Naming the
condition explicitly is what keeps the counts correct across the mixed-grain join.

```
Games        COUNT([Gm#])
             // { FIXED [Tm], [Year]: MAX([Gm#])}   ← earlier LOD approach, retained as a comment
Wins         COUNT(IF [W/L Group] = 'Win' THEN [Gm#] ELSE NULL END)
             // { FIXED [Tm], [Year]: MAX([W])}     ← earlier LOD approach, retained
Games - Wins [Games] - [Wins]
Win / Games Ratio  [Wins] / [Games]
```

Both `Games` and `Wins` carry a commented-out `{FIXED}` version above the live formula,
recording the earlier LOD approach that was replaced by conditional counting once the
team-season filters made the simpler form correct.

**The four split families** repeat the same three-part shape:

| Split | Games | Wins | Win ratio | Loss ratio |
|---|---|---|---|---|
| Home | `COUNT(IF [Home/Away Game]='Home' THEN [Gm#] ELSE NULL END)` | `COUNT(IF [W/L Group]='Win' AND [Home/Away Game]='Home' THEN [Gm#] ELSE NULL END)` | `[Home Wins]/[Home Games]` | `([Home Games]-[Home Wins])/[Home Games]` |
| Away | same with `'Away'` | same with `'Away'` | `[Away Wins]/[Away Games]` | `([Away Games]-[Away Wins])/[Away Games]` |
| Day | `COUNT(IF [D/N]='D' THEN [Gm#] ELSE NULL END)` | `COUNT(IF [W/L Group]='Win' AND [D/N]='D' …)` | `[Day Wins]/[Day Games]` | `([Day Games]-[Day Wins])/[Day Games]` |
| Night | same with `'N'` | same with `'N'` | `[Night Wins]/[Night Games]` | `([Night Games]-[Night Wins])/[Night Games]` |

These twelve fields produce the four split bars reading 64.2 %, 50.6 %, 62.2 % and 55.6 % on
the published dashboard. Each bar is a win-ratio / loss-ratio pair, which is why both the
ratio and its complement are calculated rather than deriving one from the other in the view.

### 6.4 The season grid

| Field | Formula | Purpose |
|---|---|---|
| **Win/Lose Color** | `IF [2 Games Order] = 1 Then [W/L Group] ELSEIF [2 Games Order] = 2 Then [W/L Group]+' (2)' ELSEIF Contains([W/L (PS Games)],'W') Then 'Win' ELSE 'x' END` | The grid's colour key. Doubleheaders produce **two games on one calendar day**, so the second game gets a distinct `… (2)` colour and a different shape, letting one cell show both results |
| **2 Games Number Size** | `[2 Games Order]` | Drives the size encoding that visually separates the two halves of a doubleheader |
| **Doubleheader Txt** | `IF [2 Games] = 'Yes' Then ' (Doubleheader)' ELSE NULL END` | Tooltip suffix |
| **Day Name Abbv** | `[Day Name]` | Alias used as the grid's row header |
| **Home Games Txt** | `IF [Home/Away Game] = 'Home' THEN 'the' ELSE 'an' END` | Grammatical helper so tooltips read "at the home game" / "at an away game" |

The doubleheader handling is the detail that makes the calendar grid honest: a naive
week × weekday grid would silently overwrite one of the two games played on the same date.

### 6.5 Post-season

| Field | Formula | Purpose |
|---|---|---|
| **Wild Card Bye Text** | `IF [Wild Card Bye] = True Then 'Wild Card Bye' ELSE NUll END` | Prints the bye label for teams that skipped the Wild Card round |
| **1** | `1` | Unit constant sizing every post-season game square equally, so a series reads as a row of fixed cells regardless of scores |

### 6.6 Statistics toggle

| Field | Formula | Controls |
|---|---|---|
| **Batting Stats T\|F** | `[Stats] = 'Batting'` | Batting table container and the batter stat strip; also the toggle's own colour |
| **Pitching Stats T\|F** | `[Stats] = 'Pitching'` | Pitching table container and the pitcher stat strip; also the toggle's colour |

### 6.7 Helpers

| Field | Formula | Purpose |
|---|---|---|
| **i\*** / **i\* (copy)** | `'i'` | Single-member dimensions that force the information/guide icons to a fixed colour (`#06673e` and `#767f8b`) |
| **Pitcher** | `'Pitcher'` | Static label |
| **HRs Allowed** | `[Homeruns Allowed]` | Renamed alias for the pitching table |

---

## 7. Worksheet specifications

38 worksheets. Grouped by dashboard region; the numeric prefix on each sheet name is the
author's own region marker (`1-` selectors, `2-` season, `3-` donut, `4-` post-season,
`5-`/`6-` statistics).

### 7.1 Selectors (`1-` sheets, 9 sheets)

| Sheet | Mark | Construction |
|---|---|---|
| **1-Year PA** | Shape ×3 | Columns `Year`; Rows `MIN(0)` dual axis; shape `Tm` (team logo); colour `Year Selector`; text `Year` + `World Series Team`. Renders the five World Series banners across the header |
| **1-Team PA C** | Automatic | Columns `MIN(1)`; colour `Team Selector`; text `League Champion (Team Groups)`; LOD `Team Name`. The **Champion Team** button |
| **1-Team PA R** | Automatic | Same with `min(1)`. The **Runner-Up Team** button |
| **1-Stats Batting PA** | Shape | Colour `Batting Stats T\|F`; text `'Batting'`. Batting toggle |
| **1-Stats Pitching PA** | Shape | Colour `Pitching Stats T\|F`; text `'Pitching'`. Pitching toggle |
| **1-Guide A / B / C / 4** | Shape | The baseball-diamond help icons; `1-Guide 4` colours by `i* (copy)`. Their tooltips carry the viz instructions the published description refers to |

### 7.2 Season grid (`2-` sheets, 8 sheets)

| Sheet | Mark | Construction |
|---|---|---|
| **2-Games** | Shape | **The centrepiece.** Columns `Week Number` (1–29); Rows `Day Name Abbv` (Su–Sa); colour `Win/Lose Color`; size `2 Games Number Size`; LOD `Gm#`. Custom shapes `icons8-rounded-square-400.png` (filled) and `icons8-rounded-square-blank-400.png` (outline, for the second game of a doubleheader) |
| **2-Games Line** | Line | Columns `Week Number`; Rows Multiple Values; colour `:Measure Names`. The cumulative wins/losses trend lines above the grid |
| **2-Days** | Shape | Columns `MIN(1)`; Rows `Day Name Abbv`; colour `W/L Group`; text `CNT(Gm#)`. The per-weekday win counts down the left edge (15, 11, 15, 17, 8, 15, 12) |
| **2-Week** | Shape | Columns `Week Number`; Rows `MIN(1)`; colour `W/L Group`. The week-number header strip |
| **2-Keys** | Shape ×3 | Columns `W/L 1/2 Key` dual axis; text `CNT(Gm#)` + `Win/Loss Text Plural`. The **93 Wins / 69 Losses** key |
| **2-Team Name** | Text | Text `Team Location (Team Groups)` + `Team Name Short (Team Groups)` + `World Series Year`. The large "Los Angeles Dodgers" title |
| **2-Title** | Text | Same fields, used in a second position |
| **VS Logos** | Shape ×3 | Columns `MIN(-1)` + `MIN(1)` dual axis; shape `Tm` and `Opp`; text the two team names. The head-to-head logo pair |

### 7.3 Win-ratio donut and splits (5 sheets)

| Sheet | Mark | Construction |
|---|---|---|
| **3-Donut Wins** | Pie ×3 | Rows `min(0)` + `MIN(0)` dual axis; wedge size Multiple Values; colour `:Measure Names`; text `Games - Wins`, `Games`, `Win / Games Ratio`. The **winged donut** the description names — three stacked pie layers producing a ring with the ratio at its centre (57.4 %, 93 / 162) |
| **Home Wins** | Bar | Columns Multiple Values; colour `:Measure Names`; the 64.2 % / 35.8 % split bar |
| **Away Wins** | Bar | Same construction — 50.6 % / 49.4 % |
| **Day Wins** | Bar | Same — 62.2 % / 37.8 % |
| **Night Wins** | Bar | Same — 55.6 % / 44.4 % |
| **Wins \| Bar** | Bar | Columns `CNT(Gm#)`; Rows `Day Name Abbv`; colour `W/L Group` |

Each split bar places the win ratio and its complement side by side as two measures on one
bar, which is why §6.3 calculates both rather than deriving the remainder in the view.

### 7.4 Post-season ladder (`4-` sheets, 6 sheets)

| Sheet | Mark | Construction |
|---|---|---|
| **4-PS Games-1 … -4** | Shape | One sheet per series round. Columns `Post Series Gm#`; Rows `Series`; colour `W/L (PS Games)`; size `MIN(1)` (equal cells); text `W - L S`; LOD `Post Gm#1`. `4-PS Games-1` also prints `Team Name Short` |
| **4-PS VS Logos** | Shape ×3 | Columns `MIN(-1)` + `MIN(1)`; shape `Tm` and `Opp (PS Games)`; text the two team locations |
| **4-Wild Card Bye** | Text | Text `Wild Card Bye Text` — prints only when the team had a bye |

Four separate sheets rather than one with `Series` on rows: each round has a different
best-of format (2-of-3, 3-of-5, 4-of-7, 4-of-7), and separate sheets let each row carry its
own caption and cell count.

### 7.5 Statistics (`5-` and `6-` sheets, 6 sheets)

| Sheet | Mark | Construction |
|---|---|---|
| **5-Batting Stats** | Text | Columns `'x'`; Rows the full batting line — `Rank #` / `Player` / `Position` / `Games Played` / `Hits/At Bat` / `SLG` / `OPS` / `At Bat` / `Runs Scored` / `Hits` / `2B` / `3B` / `HR` / `RBI` / `Total Bases` / `Stolen Bases` / `Base Walks` / `Strikeouts` / `Hit Batter` |
| **5-Batting Total** | Text | The same structure rendering the **Team Totals** row |
| **5-Pitching Stats** | Text | The pitching equivalent — `Rank Number` / `Player` / `Pitcher` / `Games Played` / `Innings Pitched` / ERA / WHIP / FIP and counting stats |
| **5-Pitching Total** | Text | Pitching team totals |
| **6-Batter Stats** | Shape | Columns `:Measure Names`; Rows `MIN(1)`; colour `i*`; text Multiple Values. The green stat chips above the table — 825 Runs Scored, 1,384 Hits, 257 Doubles, 21 Triples, 244 Home Runs, 791 RBI, 88 Stolen Bases, 2,415 Total Bases, 580 Base Walks, 1,353 Strikeouts |
| **6-Pitcher Stats** | Shape | The pitching equivalent chip strip |

### 7.6 Reference sheets (4 sheets)

`Teams` (Rows `Year / Tm`, shape `Tm`), `Opp` (Rows `Opp`, shape `Opp`) — logo reference
sheets; `Sheet 38` — an empty scratch sheet.

---

## 8. Dashboard specification

### 8.1 Canvas

| Property | Value |
|---|---|
| Name | `Road to the World Series` |
| Sizing | **Fixed**, 1600 × 1800 px |
| Root background | `#cde0d8` (pale mint) |
| Preset index | 8 |

### 8.2 Layout tree

The dashboard is built as a tiled **background frame** with the content floating above it.

```
[18]  layout-basic                                   bg #cde0d8
└ [100] flow  "Background VC"
  ├ [101] empty
  ├ [102] empty                                      bg #ffffff
  ├ [104] empty
  ├ [116] flow
  │ ├ [110] flow  ├ [108] empty bg #000000
  │ │             └ [109] empty bg #ffffff
  │ ├ [117] empty
  │ └ [113] flow  ├ [111] empty bg #000000
  │               └ [112] empty bg #ffffff
  ├ [118] empty
  └ [105] flow    ├ [106] empty bg #000000
                  └ [107] empty bg #ffffff

FLOATING LAYERS (in z-order)
[128] worksheet  4-Wild Card Bye
[97]  empty                                          bg #cde0d8
[43]  flow       ├ [42] 4-PS Games-1
                 ├ [53] 4-PS Games-2
                 ├ [60] 4-PS Games-3
                 └ [127] flow → [61] 4-PS Games-4
[40]  text       "All In The Wins / Road to the World Series / 162 Games | 30 Teams | 1 Champion"
[7]   worksheet  3-Donut Wins
[63]  worksheet  2-Team Name
[64]  bitmap     (World Series trophy image)
[41]  worksheet  1-Year PA
[66]  text       "Post-Season Playoffs"
[67]  text       "Game Season Statistics"
[76]  text       "MLB 162 Game Season"
[81]  flow       ├ [82] 1-Stats Batting PA
                 └ [88] 1-Stats Pitching PA
[96]  flow       └ [91] flow → [92] "Home"  [93] "Away"
```

The black-and-white paired empty containers in the background frame are the **card borders**:
a black strip with a white strip beside it produces the crisp panel outlines visible around
each region. Tableau has no drop-shadow, so the effect is built from stacked spacers.

### 8.3 Text zones

| Content | Role |
|---|---|
| `All In The Wins` / `Road to the World Series` / `162 Games \| 30 Teams \| 1 Champion` | Masthead |
| `MLB 162 Game Season` | Section header, left panel |
| `Post-Season Playoffs` | Section header, right panel |
| `Game Season Statistics` | Section header, lower panel |
| `Home` / `Away` / `Day` / `Night` | Split-bar row labels |
| `Day` / `Week` | Season grid axis labels |
| `Designed by John Johansson \| Statistics Baseball-Reference \| Logos SportsLogos` | Footer credit |

---

## 9. Interactivity

### 9.1 Parameter actions (5)

Every control is hover-driven; there are no click targets and no parameter controls on the
canvas.

| Caption | Source sheet | Trigger | Source field | Target parameter |
|---|---|---|---|---|
| `Change Year` | `1-Year PA` | on-hover | `Year` (SUM) | **World Series Year** |
| `Change Team C` | `1-Team PA C` | on-hover | `League Champion (Team Groups)` (ATTR) | **World Series Team** |
| `Change Team R` | `1-Team PA R` | on-hover | `League Champion (Team Groups)` (ATTR) | **World Series Team** |
| `Change Bat` | `1-Stats Batting PA` | on-hover | `'Batting'` | **Stats** |
| `Change Pitch` | `1-Stats Pitching PA` | on-hover | `'Pitching'` | **Stats** |

All five keep their value on clear, so the selection persists after the pointer leaves.

### 9.2 Dynamic Zone Visibility (4 bindings)

| Controlling field | Zone | Contents |
|---|---|---|
| `Batting Stats T\|F` | 77 | The batting table container |
| `Batting Stats T\|F` | 139 | `6-Batter Stats` chip strip |
| `Pitching Stats T\|F` | 134 | The pitching table container |
| `Pitching Stats T\|F` | 141 | `6-Pitcher Stats` chip strip |

**Resulting states**

| `Stats` | Visible | Hidden |
|---|---|---|
| `Batting` *(default)* | Batting totals chips + batting table | Pitching chips + table |
| `Pitching` | Pitching chips + table | Batting chips + table |

Year and team selection work differently: rather than swapping zones, they change the
`Year Selector Filter` / `Team Selector Filter` booleans, which re-filter every sheet at
once. One dashboard, five seasons × two teams = ten states, with no duplicated layout.

---

## 10. Design system

### 10.1 Palette

| Token | Hex | Applied to |
|---|---|---|
| Field green (primary) | `#06673e` | Wins, active selectors, stat chips, series victories |
| Slate grey (negative) | `#767f8b` | Losses, inactive selectors, unselected banners |
| Pale mint (ground) | `#cde0d8` | Dashboard background |
| White | `#ffffff` | Panel fills |
| Black | `#000000` | Panel border strips |
| Secondary green | `#378564` | `Win (2)` — second game of a doubleheader |
| Navy | `#041e42` | Away games, Night games |
| Crimson | `#bf0d3e` | Home games, Day games |
| Light grey | `#b3b7b8` | `Loss (2)` |
| Amber | `#f28e2b` | `Win_1` variant |
| Gold | `#edc948` | `Win_2` variant |

The navy/crimson pair is a deliberate switch away from the green/grey scheme: when the season
grid is coloured by Home/Away or Day/Night rather than Win/Loss, it must read as a *category*
distinction rather than a *good/bad* one.

### 10.2 Categorical colour assignments

| Field | Mapping |
|---|---|
| `W/L Group` | Win `#06673e` · Loss `#767f8b` |
| `W/L (PS Games)` | W `#06673e` · L `#767f8b` |
| `Win/Lose Color` | Win `#06673e` · Win (2) `#378564` · Loss `#767f8b` · Loss (2) `#b3b7b8` · plus `Win_1` `#f28e2b`, `Win_2` `#edc948`, `Lose_1` `#4e79a7`, `Lose_2` `#76b7b2` |
| `Home/Away Game` | Away `#041e42` · Home `#bf0d3e` |
| `D/N` | N `#041e42` · D `#bf0d3e` |
| `2 Games` | No `#4e79a7` · Yes `#f28e2b` |
| `Batting/Pitching Stats T\|F`, `Team Selector`, `Year Selector` | true `#06673e` · false `#767f8b` |
| `i*` / `i* (copy)` | `#06673e` / `#767f8b` |

### 10.3 Shape encoding

| Field | Shapes |
|---|---|
| `Win/Lose Color` | `DL Shapes 400/icons8-rounded-square-400.png` (filled) and `icons8-rounded-square-blank-400.png` (outline, second game of a doubleheader) |
| `W/L Group` | `icons8-rounded-square-400.png` |
| `Tm` | Team logos — `Baseball Teams/Dodgers.png`, `Astros.png`, `Braves.png`, `Diamondbacks.png`, `Phillies.png`, `Rangers.png`, `Yankees.png`, `toronto_blue_jays.png` |
| `Opp` / `Opp (PS Games)` | The full 30-team logo set — `boston_red_sox.png`, `chicago_cubs.png`, `cleveland_guardians.png` and so on |

The rounded-square shape is what gives the season grid its distinctive look; a default square
mark would read as a heatmap rather than a calendar of discrete games.

### 10.4 Typography

| Level | Spec |
|---|---|
| Workbook default | **Arial** |
| Titles | Bold, 10 pt base, scaled per zone |
| Masthead | Large bold — "All In The Wins" in green, "Road to the World Series" in black |
| Team name | Very large, light weight — "Los Angeles Dodgers" |
| Stat chips | Large gold numerals over green fill |
| Section headers | Bold, left-aligned above each panel |

### 10.5 Iconography

Two custom shape sets from the author's local repository: `Baseball Teams/` (30 MLB logos) and
`DL Shapes 400/` (rounded square, filled and blank). Plus an embedded bitmap of the World
Series trophy in the masthead — the only true image asset on the dashboard. The shape files
are **not** packaged in the `.twbx`.

---

## 11. Calculation dependency map

```
World Series Year param ──► Year Selector Filter ──────┐  (34 sheets)
World Series Team param ─► Team Selector Filter ───────┤  (33 sheets)
                                                       └─► scopes EVERY measure to one team-season
                        └─► Team Selector / Year Selector ──► banner + button colour

Z2 ──► Home/Away Game ──┬─► Home Games / Home Wins ──► Home Win / Games Ratio ──► Home Wins bar
                        └─► Away Games / Away Wins ──► Away Win / Games Ratio ──► Away Wins bar
D/N ────────────────────┬─► Day Games / Day Wins ────► Day Win / Games Ratio ──► Day Wins bar
                        └─► Night Games / Night Wins ► Night Win / Games Ratio ► Night Wins bar

W/L ──► W/L Group ──┬─► Wins ──┬─► Win / Games Ratio ──► 3-Donut Wins
                    │          └─► Games - Wins ──────► 3-Donut Wins
                    ├─► W/L 1/2 Key ──► 2-Keys ordering
                    └─► Win/Lose Color ◄── 2 Games Order ──► 2-Games grid
                                       └─► 2 Games Number Size

Stats param ──┬─► Batting Stats T|F ──► DZV: batting table + chips
              └─► Pitching Stats T|F ─► DZV: pitching table + chips

Wild Card Bye ──► Wild Card Bye Text ──► 4-Wild Card Bye
```

---

## 12. Rebuild / maintenance runbook

**Adding a new season**

1. Append the new season's rows to all four Excel sources, keeping `Unique` consistent as the
   team-season key across `Team Groups`, `Games_Champions`, `PS Games`, `Batting` and
   `Pitching`.
2. Add the year to the **World Series Year** parameter's allowable values.
3. Add a banner to `1-Year PA` — it is driven by the data, so a new team-season with
   `League Champion` populated appears automatically once the parameter lists the year.
4. Add the two teams' logos to `Baseball Teams/` in the local Tableau Repository if they are
   not already present, and map them on the `Tm` and `Opp` shape encodings.
5. Refresh the extract.

**Adding a colour scheme to the season grid**

The **Color** parameter already lists four schemes (Win/Lose, Home/Away, Day/Night, Home
Attendance). To wire an additional one, extend `Win/Lose Color` with the new branch and add
the corresponding colour assignments, keeping the doubleheader `… (2)` variants in step so the
second game of a pair stays distinguishable.

**Preserving doubleheader handling**

`Win/Lose Color` and `2 Games Number Size` both read `2 Games Order`. If either is edited,
check that a date with two games still renders two distinguishable marks — the grid is keyed
on week × weekday, so without this a doubleheader silently loses a game.

**Changing the post-season ladder**

Each round is its own sheet (`4-PS Games-1` … `-4`) because the best-of formats differ. Adding
a round means duplicating a sheet, filtering it to the new `Series` value, and adding it to
the floating container (zone 43) with its caption.

**Statistics tables**

`5-Batting Stats` and `5-Batting Total` share a row structure; edit them together so the
totals row keeps aligning with the player rows above it. The same applies to the pitching
pair.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | Games_Champions (Baseball_Reference) |
| Connection class | `federated` |
| Tables / relations | 8 · joins: 7 |
| Physical columns materialised | 56 |
| Source fields in data pane | 140 |
| Calculated fields (used / total) | 43 / 54 |
| Parameters | 4 |
| Data source filters | 0 (none) |

### Source fields

140 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Team-season

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Year (Team Groups)** | integer · Dimension | — | — | 35 | Season year. Arrives from the **Team Groups** table in the join, duplicating the same column on the left-hand side. |
| **Team Name Short** | string · Dimension | 8 | yes | 2 | Nickname — e.g. Dodgers. |
| **Year** | integer · Dimension | 5 | yes | 2 | Season year. |
| **League** | string · Dimension | — | yes | 0 | League: American or National. *(hidden, not materialised, not used in any sheet)* |
| **League (PS Games)** | string · Dimension | — | — | 0 | League: American or National. Arrives from the **PS Games** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **League (Team Groups)** | string · Dimension | — | — | 0 | League: American or National. Arrives from the **Team Groups** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **League Champion** | string · Dimension | 2 | yes | 0 | Whether the team was the `Champion` or `Runner-Up` that season. *(hidden, not used in any sheet)* |
| **League Champion (PS Games)** | string · Dimension | — | — | 0 | Whether the team was the `Champion` or `Runner-Up` that season. Arrives from the **PS Games** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **Team Location (PS Games)** | string · Dimension | — | — | 0 | City — e.g. Los Angeles. Arrives from the **PS Games** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **Team Name (PS Games)** | string · Dimension | — | — | 0 | Full team name. Arrives from the **PS Games** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **Team Name (Team Groups)** | string · Dimension | — | — | 0 | Full team name. Arrives from the **Team Groups** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **Team Name Short (PS Games)** | string · Dimension | — | — | 0 | Nickname — e.g. Dodgers. Arrives from the **PS Games** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **Team No** | integer · Measure (Sum) | — | yes | 0 | Sequence number assigned to the team for ordering. *(hidden, not materialised, not used in any sheet)* |
| **Year (Batting)** | integer · Dimension | — | — | 0 | Season year. Arrives from the **Batting** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **Year (Pitching)** | integer · Dimension | — | — | 0 | Season year. Arrives from the **Pitching** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **Year (PS Games)** | integer · Dimension | — | — | 0 | Season year. Arrives from the **PS Games** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |

#### Game

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Gm#** | integer · Dimension | 162 | yes | 12 | Game number within the 162-game season. The unit counted everywhere. |
| **W/L** | string · Dimension | 2 | yes | 11 | Result — W, L, W-wo, L-wo. The `-wo` variants are walk-offs. |
| **W/L Group** | string · Dimension | — | — | 11 | Result collapsed to Win or Loss, folding the walk-off variants in. |
| **W/L (PS Games)** | string · Dimension | — | — | 7 | Result — W, L, W-wo, L-wo. The `-wo` variants are walk-offs. Arrives from the **PS Games** table in the join, duplicating the same column on the left-hand side. |
| **Tm** | string · Dimension | 8 | yes | 4 | Team code. Drives the logo shape encoding. |
| **Week Number** | real · Dimension | 29 | yes | 4 | Season week, 1–29. The horizontal axis of the grid. |
| **D/N** | string · Dimension | 2 | yes | 3 | Day or night game. |
| **Home Runs**<br>`HR` | integer · Dimension | 39 | yes | 3 | Home runs hit in the game. |
| **Runs Allowed**<br>`R (Pitching)` | integer · Dimension | — | — | 3 | Runs allowed in the game. |
| **Runs Scored**<br>`R (Batting)` | integer · Measure | — | — | 3 | Runs scored by the team in the game. |
| **2 Games Order** | integer · Measure (Sum) | 2 | yes | 2 | Which game of a doubleheader — 1 or 2. Without it the grid would overwrite one of the pair. |
| **Opp** | string · Dimension | 21 | yes | 2 | Opponent team code. Drives opponent logos. |
| **Rank #**<br>`Rk` | integer · Dimension | 41 | yes | 2 | Division standing as a number. |
| **Rank Number**<br>`Rk (Pitching)` | integer · Dimension | — | — | 2 | Division standing as a number. |
| **2 Games** | string · Dimension | 2 | yes | 1 | Whether the date carried a doubleheader. |
| **Game Date** | date · Dimension | 105 | yes | 1 | Calendar date of the game. |
| **L** | integer · Measure (Sum) | 79 | yes | 1 | Cumulative losses. |
| **Opp (PS Games)** | string · Dimension | — | — | 1 | Opponent team code. Drives opponent logos. Arrives from the **PS Games** table in the join, duplicating the same column on the left-hand side. |
| **R** | integer · Measure (Sum) | 80 | yes | 1 | Runs scored by the team. |
| **RA** | integer · Measure (Sum) | 13 | yes | 1 | Runs allowed. |
| **W** | integer · Measure (Sum) | 102 | yes | 1 | Cumulative wins. |
| **Attendance** | integer · Measure (Sum) | — | yes | 0 | Announced attendance at the game. *(hidden, not materialised, not used in any sheet)* |
| **Date (OG)** | string · Dimension | — | yes | 0 | Date as printed in the source game log, before parsing. *(hidden, not materialised, not used in any sheet)* |
| **Date Calc** | date · Dimension | — | yes | 0 | Parsed game date derived from the source text. *(hidden, not materialised, not used in any sheet)* |
| **GB** | string · Dimension | — | yes | 0 | Games behind the division leader after this game. *(hidden, not materialised, not used in any sheet)* |
| **Inn** | integer · Measure (Sum) | — | yes | 0 | Innings the game lasted. Flags extra-inning games. *(hidden, not materialised, not used in any sheet)* |
| **L (Pitching)** | integer · Measure | — | — | 0 | Cumulative losses. Arrives from the **Pitching** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **L (PS Games)** | integer · Measure | — | — | 0 | Cumulative losses. Arrives from the **PS Games** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **Loss** | string · Dimension | — | yes | 0 | Losing pitcher of record. *(hidden, not materialised, not used in any sheet)* |
| **Orig. Scheduled** | string · Dimension | — | yes | 0 | Original date, where the game was rescheduled from a postponement. *(hidden, not materialised, not used in any sheet)* |
| **Rank** | integer · Measure (Sum) | — | yes | 0 | Division standing after this game. *(hidden, not materialised, not used in any sheet)* |
| **Save** | string · Dimension | — | yes | 0 | Pitcher credited with the save. *(hidden, not materialised, not used in any sheet)* |
| **Streak** | string · Dimension | — | yes | 0 | Running win or loss streak, carried as + / - characters in the source. *(hidden, not materialised, not used in any sheet)* |
| **Time** | datetime · Dimension | — | yes | 0 | Duration of the game. *(hidden, not materialised, not used in any sheet)* |
| **Tm (PS Games)** | string · Dimension | — | — | 0 | Team code. Drives the logo shape encoding. Arrives from the **PS Games** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **Tm ID** | string · Dimension | — | yes | 0 | Team identifier. *(hidden, not materialised, not used in any sheet)* |
| **W (Pitching)** | integer · Measure | — | — | 0 | Cumulative wins. Arrives from the **Pitching** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **W (PS Games)** | integer · Measure | — | — | 0 | Cumulative wins. Arrives from the **PS Games** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **W - L (PS Games)** | string · Dimension | — | — | 0 | Running win–loss record. Arrives from the **PS Games** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **W-L%** | real · Measure (Sum) | — | yes | 0 | Winning percentage to date. *(hidden, not materialised, not used in any sheet)* |
| **Win** | string · Dimension | — | yes | 0 | Winning pitcher of record. *(hidden, not materialised, not used in any sheet)* |
| **Z1** | string · Dimension | — | yes | 0 | Home/away marker column from the source; an @ means away. *(hidden, not materialised, not used in any sheet)* |

#### Post-season

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Post Gm#1**<br>`Post Gm#` | integer · Dimension | 29 | yes | 5 | Game number across the whole post-season. The trailing digit marks a second copy of the column, brought in by a join. |
| **Post Series Gm#** | real · Measure (Sum) | 7 | yes | 5 | Game number within the series. |
| **Series** | string · Dimension | 4 | yes | 5 | Playoff round — Wild Card, Division, League Championship, World Series. |
| **W - L S**<br>`W - L_S` | string · Dimension | 25 | yes | 5 | Running series score. |
| **Series #** | integer · Dimension | 4 | yes | 4 | Series sequence number. |
| **L S**<br>`L_S` | integer · Measure (Sum) | — | yes | 0 | Losses in the series to date. *(hidden, not materialised, not used in any sheet)* |
| **Series Standings** | string · Dimension | — | yes | 0 | Running series standing. *(hidden, not materialised, not used in any sheet)* |
| **W S**<br>`W_S` | integer · Measure (Sum) | — | yes | 0 | Wins in the series to date. *(hidden, not materialised, not used in any sheet)* |

#### Batting

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Base Walks**<br>`BB` | integer · Dimension | 70 | yes | 3 | Base on balls. |
| **Doubles**<br>`2B` | integer · Measure (Sum) | 52 | yes | 3 | Doubles hit. |
| **Hits**<br>`H` | integer · Dimension | 116 | yes | 3 | Hits. |
| **Player**<br>`Player (Pitching)` | string · Dimension | — | — | 3 | Player name. |
| **RBI** | integer · Dimension | 96 | yes | 3 | Runs batted in. |
| **Stolen Bases**<br>`SB` | integer · Dimension | 34 | yes | 3 | Stolen bases. |
| **Strikeouts**<br>`SO` | integer · Dimension | 137 | yes | 3 | Strikeouts. |
| **Strikeouts**<br>`SO (Pitching)` | integer · Dimension | — | — | 3 | Strikeouts. |
| **Total Bases**<br>`TB` | integer · Dimension | 157 | yes | 3 | Total bases: singles, plus two per double, three per triple, four per home run. |
| **Triples**<br>`3B` | integer · Measure (Sum) | 18 | yes | 3 | Triples hit. |
| **Walks**<br>`BB (Pitching)` | integer · Dimension | — | — | 3 | Base on balls drawn. |
| **At Bat**<br>`AB` | integer · Dimension | 197 | yes | 2 | At bats. |
| **Games Played**<br>`G` | integer · Dimension | 73 | yes | 2 | Games played. |
| **Games Played**<br>`G (Pitching)` | integer · Dimension | — | — | 2 | Games played. |
| **Hit by Pitcher**<br>`HBP` | integer · Dimension | 24 | yes | 2 | Times hit by a pitch. |
| **Hits/At Bat**<br>`BA` | real · Dimension | 138 | yes | 2 | Batting average: hits divided by at bats. |
| **OPS** | real · Dimension | 202 | yes | 2 | On-base plus slugging. |
| **Position**<br>`Pos` | string · Dimension | 16 | yes | 2 | Fielding position. |
| **SLG** | real · Dimension | 177 | yes | 2 | Slugging percentage — total bases per at bat. |
| **Awards** | string · Dimension | — | yes | 0 | Awards and honours earned that season. *(hidden, not materialised, not used in any sheet)* |
| **Awards (Pitching)** | string · Dimension | — | — | 0 | Awards and honours earned that season. Arrives from the **Pitching** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **CS** | integer · Measure (Sum) | — | yes | 0 | Caught stealing. *(hidden, not materialised, not used in any sheet)* |
| **Gidp**<br>`GIDP` | integer · Measure (Sum) | — | yes | 0 | Grounded into double play. *(hidden, not materialised, not used in any sheet)* |
| **GS** | integer · Measure (Sum) | — | yes | 0 | Games started. *(hidden, not materialised, not used in any sheet)* |
| **IBB** | integer · Measure (Sum) | — | yes | 0 | Intentional walks. *(hidden, not materialised, not used in any sheet)* |
| **IBB (Pitching)** | integer · Measure | — | — | 0 | Intentional walks. Arrives from the **Pitching** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **OBP** | real · Measure (Sum) | — | yes | 0 | On-base percentage. *(hidden, not materialised, not used in any sheet)* |
| **Ops+**<br>`OPS+` | integer · Measure (Sum) | — | yes | 0 | OPS indexed to the league, where 100 is league average. *(hidden, not materialised, not used in any sheet)* |
| **PA** | integer · Measure (Sum) | — | yes | 0 | Plate appearances. *(hidden, not materialised, not used in any sheet)* |
| **Player-additional** | string · Dimension | — | yes | 0 | Baseball-Reference player key, disambiguating players who share a name. *(hidden, not materialised, not used in any sheet)* |
| **Player-additional (Pitching)** | string · Dimension | — | — | 0 | Baseball-Reference player key, disambiguating players who share a name. Arrives from the **Pitching** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **Pos**<br>`Pos (Pitching)` | string · Dimension | — | — | 0 | Fielding position. *(hidden, not used in any sheet)* |
| **Pos 1** | string · Dimension | — | yes | 0 | Primary fielding position. *(hidden, not materialised, not used in any sheet)* |
| **Rbat+** | integer · Measure (Sum) | — | yes | 0 | Batting runs indexed to the league, 100 being average. *(hidden, not materialised, not used in any sheet)* |
| **Sacrifice Bunt**<br>`SH` | integer · Dimension | — | yes | 0 | Sacrifice bunts. *(hidden, not materialised, not used in any sheet)* |
| **Sacrifice Flies**<br>`SF` | integer · Dimension | — | yes | 0 | Sacrifice flies. *(hidden, not materialised, not used in any sheet)* |
| **WAR** | real · Measure (Sum) | — | yes | 0 | Wins above replacement. *(hidden, not materialised, not used in any sheet)* |
| **WAR (Pitching)** | real · Measure | — | — | 0 | Wins above replacement. Arrives from the **Pitching** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |

#### Pitching

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Balks**<br>`BK` | integer · Dimension | 11 | yes | 3 | Balks. |
| **Batters Faced**<br>`BF` | integer · Dimension | 226 | yes | 3 | Batters faced. |
| **Hit Batter**<br>`HBP (Pitching)` | integer · Dimension | — | — | 3 | Batters hit by a pitch. |
| **Hits Allowed**<br>`H (Pitching)` | integer · Dimension | — | — | 3 | Hits allowed. |
| **Homeruns Allowed**<br>`HR (Pitching)` | integer · Dimension | — | — | 3 | Home runs allowed. |
| **Shutouts**<br>`SHO` | integer · Dimension | 3 | yes | 3 | Complete games with no runs allowed. |
| **ERA** | real · Dimension | 220 | yes | 2 | Earned run average — earned runs per nine innings. |
| **FIP** | real · Dimension | 240 | yes | 2 | Fielding independent pitching — ERA-scaled, stripped of fielding effects. |
| **Innings Pitched**<br>`IP` | real · Dimension | 201 | yes | 2 | Innings pitched. |
| **WHIP** | real · Dimension | 243 | yes | 2 | Walks and hits per inning pitched. |
| **BB9** | real · Measure (Sum) | — | yes | 0 | Walks per nine innings. *(hidden, not materialised, not used in any sheet)* |
| **C Li**<br>`cLI` | real · Measure (Sum) | — | yes | 0 | Average leverage index: how high-pressure the appearances were. *(hidden, not materialised, not used in any sheet)* |
| **CG** | integer · Measure (Sum) | — | yes | 0 | Complete games. *(hidden, not materialised, not used in any sheet)* |
| **ER** | integer · Measure (Sum) | — | yes | 0 | Earned runs allowed. *(hidden, not materialised, not used in any sheet)* |
| **Era+**<br>`ERA+` | integer · Measure (Sum) | — | yes | 0 | ERA indexed to the league, 100 being average; higher is better. *(hidden, not materialised, not used in any sheet)* |
| **GF** | integer · Measure (Sum) | — | yes | 0 | Games finished. *(hidden, not materialised, not used in any sheet)* |
| **H9** | real · Measure (Sum) | — | yes | 0 | Hits allowed per nine innings. *(hidden, not materialised, not used in any sheet)* |
| **HR9** | real · Measure (Sum) | — | yes | 0 | Home runs allowed per nine innings. *(hidden, not materialised, not used in any sheet)* |
| **R Oba**<br>`rOBA` | real · Measure (Sum) | — | yes | 0 | Opponent batting average against. *(hidden, not materialised, not used in any sheet)* |
| **So/Bb**<br>`SO/BB` | real · Measure (Sum) | — | yes | 0 | Strikeout-to-walk ratio. *(hidden, not materialised, not used in any sheet)* |
| **SO9** | real · Measure (Sum) | — | yes | 0 | Strikeouts per nine innings. *(hidden, not materialised, not used in any sheet)* |
| **SV** | integer · Measure (Sum) | — | yes | 0 | Saves. *(hidden, not materialised, not used in any sheet)* |
| **WP** | integer · Measure (Sum) | — | yes | 0 | Wild pitches. *(hidden, not materialised, not used in any sheet)* |

#### Opponent

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **League Opponite** | string · Dimension | — | yes | 0 | Opponent's league (source spelling retained). Marks inter-league games. *(hidden, not materialised, not used in any sheet)* |
| **League Opponite (Team Name Opponite1)** | string · Dimension | — | — | 0 | Opponent's league (source spelling retained). Marks inter-league games. Arrives from the **Team Name Opponite1** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **Team Name Opponite (Team Name Opponite)**<br>`Team Name Opponite` | string · Dimension | — | yes | 0 | Opponent's full team name (source spelling retained). Arrives from the **Team Name Opponite** table in the join, duplicating the same column on the left-hand side. *(hidden, not materialised, not used in any sheet)* |
| **Team Name Opponite (Team Name Opponite1)** | string · Dimension | — | — | 0 | Opponent's full team name (source spelling retained). Arrives from the **Team Name Opponite1** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **TeamID Opponite** | string · Dimension | 30 | yes | 0 | Opponent team key (source spelling retained). *(hidden, not used in any sheet)* |
| **TeamID Opponite (Team Name Opponite1)** | string · Dimension | — | — | 0 | Opponent team key (source spelling retained). Arrives from the **Team Name Opponite1** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |

#### Keys

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Unique** | string · Dimension | 10 | yes | 0 | Team-season key. Ties every table in the seven-join model together. *(hidden, not used in any sheet)* |
| **Unique (Batting)** | string · Dimension | — | — | 0 | Team-season key. Ties every table in the seven-join model together. Arrives from the **Batting** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **Unique (Pitching)** | string · Dimension | — | — | 0 | Team-season key. Ties every table in the seven-join model together. Arrives from the **Pitching** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **Unique (PS Games)** | string · Dimension | — | — | 0 | Team-season key. Ties every table in the seven-join model together. Arrives from the **PS Games** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |
| **Unique (Team Groups)** | string · Dimension | — | — | 0 | Team-season key. Ties every table in the seven-join model together. Arrives from the **Team Groups** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |

#### Demographics

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Age** | integer · Measure (Sum) | — | yes | 0 | Employee age in years. *(hidden, not materialised, not used in any sheet)* |
| **Age (Pitching)** | integer · Measure | — | — | 0 | Employee age in years. Arrives from the **Pitching** table in the join, duplicating the same column on the left-hand side. *(hidden, not used in any sheet)* |

### Calculated fields in use

43 of the workbook's 54 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| **1**<br>`Calculation_1787366119106260992` | integer · Measure | Basic | — | — | 4 |
| **2 Games Number Size**<br>`2 Games Number (copy)_2337649696840589312` | integer · Dimension | Basic | 2 Games Order | — | 2 |
| **Away Games**<br>`Home Games (copy)_378865334195449868` | integer · Measure | Basic | Home/Away Game, Gm# | Away Games - Away Wins, Away Games - Away Wins / Games Ratio, Away Win / Games Ratio | 2 |
| **Away Games - Away Wins / Games Ratio**<br>`Home Games - Home Wins / Games Ratio (copy)_378865334198153231` | real · Measure | Basic | Away Games, Away Wins | — | 2 |
| **Away Win / Games Ratio**<br>`Home Win / Games Ratio (copy)_378865334198153232` | real · Measure | Basic | Away Wins, Away Games | — | 2 |
| **Away Wins**<br>`Home Wins (copy)_378865334195355658` | integer · Measure | Basic | W/L Group, Home/Away Game, Gm# | Away Games - Away Wins, Away Games - Away Wins / Games Ratio, Away Win / Games Ratio | 2 |
| **Batting Stats T\|F**<br>`Team Selector Filter (copy)_2495275686544597002` | boolean · Dimension | Basic | — | — | 2 |
| **Day Games**<br>`Home Games (copy)_378865334203203603` | integer · Measure | Basic | D/N, Gm# | Day Games - Day Wins, Day Games - Day Wins / Games Ratio, Day Win / Games Ratio | 1 |
| **Day Games - Day Wins / Games Ratio**<br>`Home Games - Home Wins / Games Ratio (copy)_378865334203203601` | real · Measure | Basic | Day Games, Day Wins | — | 1 |
| **Day Name Abbv**<br>`Day Name (copy)_1447062870428999680` | string · Dimension | Basic | — | — | 4 |
| **Day Win / Games Ratio**<br>`Home Win / Games Ratio (copy)_378865334203203605` | real · Measure | Basic | Day Wins, Day Games | — | 1 |
| **Day Wins**<br>`Home Wins (copy)_378865334203203604` | integer · Measure | Basic | W/L Group, D/N, Gm# | Day Games - Day Wins, Day Games - Day Wins / Games Ratio, Day Win / Games Ratio | 1 |
| **Doubleheader Txt**<br>`2 Games Number Size (copy)_1981583853020196881` | string · Dimension | Basic | 2 Games | — | 1 |
| **Games**<br>`Team Wins in Year (copy)_2734810889226928129` | integer · Measure | LOD | Tm, Year, Gm# | Games - Wins, Win / Games Ratio | 2 |
| **Games - Wins**<br>`Team Win/Games Ratio (copy)_2734810889230036997` | integer · Measure | Basic | Games, Wins | — | 1 |
| **Home Games**<br>`Team Games in Year (copy)_378865334183559175` | integer · Measure | Basic | Home/Away Game, Gm# | Home Games - Home Wins, Home Games - Home Wins / Games Ratio, Home Win / Games Ratio | 2 |
| **Home Games - Home Wins / Games Ratio**<br>`Home Games - Home Wins (copy)_378865334197420046` | real · Measure | Basic | Home Games, Home Wins | — | 1 |
| **Home Games Txt**<br>`Home Games (copy)_1981583853018701840` | string · Dimension | Basic | Home/Away Game | — | 1 |
| **Home Win / Games Ratio**<br>`Win / Games Ratio (copy)_378865334196375565` | real · Measure | Basic | Home Wins, Home Games | — | 2 |
| **Home Wins**<br>`Team Wins in Year (copy)_378865334182043653` | integer · Measure | Basic | W/L Group, Home/Away Game, Gm# | Home Games - Home Wins, Home Games - Home Wins / Games Ratio, Home Win / Games Ratio | 2 |
| **Home/Away Game**<br>`Calculation_1177409843077935104` | string · Dimension | Basic | — | Away Game Count, Away Games, Away Wins, Home Games … | 5 |
| **HRs Allowed**<br>`Homeruns Allowed (copy)_2431662341868826625` | integer · Dimension | Basic | Homeruns Allowed | — | 1 |
| **i***<br>`Calculation_2043789823936708612` | string · Dimension | Basic | — | — | 2 |
| **i* (copy)**<br>`i* (copy)_2431662341876838402` | string · Dimension | Basic | — | — | 1 |
| **Night Games**<br>`Day Games (copy)_378865334204051479` | integer · Measure | Basic | D/N, Gm# | Night Games - Day Wins, Night Games - Night Wins / Games Ratio, Night Win / Games Ratio | 1 |
| **Night Games - Night Wins / Games Ratio**<br>`Day Games - Day Wins / Games Ratio (copy)_378865334204051481` | real · Measure | Basic | Night Games, Night Wins | — | 1 |
| **Night Win / Games Ratio**<br>`Day Win / Games Ratio (copy)_378865334204051482` | real · Measure | Basic | Night Wins, Night Games | — | 1 |
| **Night Wins**<br>`Day Wins (copy)_378865334204051483` | integer · Measure | Basic | W/L Group, D/N, Gm# | Night Games - Day Wins, Night Games - Night Wins / Games Ratio, Night Win / Games Ratio | 1 |
| **Pitcher**<br>`Calculation_2043789823941607429` | string · Dimension | Basic | — | — | 2 |
| **Pitching Stats T\|F**<br>`Batting Stats T\|F (copy)_2495275686545448971` | boolean · Dimension | Basic | — | — | 2 |
| **Team Selector**<br>`Calculation_574208969250160640` | boolean · Dimension | Basic | — | — | 2 |
| **Team Selector Filter**<br>`Team Selector (copy)_1984398602714161153` | boolean · Dimension | Basic | — | — | 33 |
| **W/L 1/2 Key**<br>`Wins (copy)_2495275686556168207` | real · Measure | Basic | W/L Group | — | 1 |
| **Wild Card Bye Text**<br>`Calculation_2495275686566592529` | string · Dimension | Basic | — | — | 1 |
| **Win / Games Ratio**<br>`Team Games in Year (copy)_2734810889227403268` | real · Measure | Basic | Wins, Games | — | 1 |
| **Win/Lose Color**<br>`Calculation_935341362772733953` | string · Dimension | Basic | 2 Games Order, W/L Group, W/L (PS Games) | — | 2 |
| **Win/Loss Text**<br>`Calculation_1981583853025480722` | string · Dimension | Basic | W/L | — | 1 |
| **Win/Loss Text Plural**<br>`Win/Loss Text (copy)_2495275686556852240` | string · Dimension | Basic | W/L | — | 1 |
| **Win/Loss ▲**<br>`Win/Loss ▲▼ (copy)_2636013173992570881` | string · Dimension | Basic | W/L | — | 2 |
| **Win/Loss ▼**<br>`Win/Loss Text (copy)_2636013173991845888` | string · Dimension | Basic | W/L | — | 1 |
| **Wins**<br>`Calculation_2734810889226309632` | integer · Measure | LOD | Tm, Year, W, W/L Group, Gm# | Games - Wins, Win / Games Ratio | 1 |
| **Year Selector**<br>`Calculation_574208969252069377` | boolean · Dimension | Basic | Year (Team Groups) | — | 1 |
| **Year Selector Filter**<br>`Year Selector (copy)_1984398602711621632` | boolean · Dimension | Basic | Year (Team Groups) | — | 34 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| **World Series Year**<br>`Parameter 1` | real · list | `2025.` | list of 5 — 2021.0, 2022.0, 2023., 2024., 2025. |
| **World Series Team**<br>`Parameter 2` | string · list | `"Champion"` | list of 2 — "Champion" → Champions, "Runner-Up" → Runner-Up |
| **Color**<br>`Parameter 3` | string · list | `"WL"` | list of 4 — "WL" → Win/Lose, "HA" → Home/Away, "DN" → Day/Night, "A" → Home Attendance |
| **Stats**<br>`World Series Team (copy)_1184165244975685632` | string · list | `"Batting"` | list of 2 — "Batting" → Batting, "Pitching" → Pitching |

### How the numbers are computed

**Level-of-detail expressions — 2.** These fix their own level of detail, so they return the same value regardless of what dimensions are in the view:

> `Wins`, `Games`

## Appendix A — Worksheet index

| Group | Sheets |
|---|---|
| Selectors | `1-Year PA`, `1-Team PA C`, `1-Team PA R`, `1-Stats Batting PA`, `1-Stats Pitching PA`, `1-Guide A`, `1-Guide B`, `1-Guide C`, `1-Guide 4` |
| Season grid | `2-Games`, `2-Games Line`, `2-Days`, `2-Week`, `2-Keys`, `2-Team Name`, `2-Title`, `VS Logos` |
| Win ratio | `3-Donut Wins`, `Home Wins`, `Away Wins`, `Day Wins`, `Night Wins`, `Wins \| Bar` |
| Post-season | `4-PS Games-1`, `-2`, `-3`, `-4`, `4-PS VS Logos`, `4-Wild Card Bye` |
| Statistics | `5-Batting Stats`, `5-Batting Total`, `5-Pitching Stats`, `5-Pitching Total`, `6-Batter Stats`, `6-Pitcher Stats` |
| Reference | `Teams`, `Opp`, `Sheet 38` |

## Appendix B — Calculated fields excluded from this documentation

11 calculated fields exist in the data pane but are not referenced by any worksheet or
dashboard logic:

`Unicodes` · `Night Games - Day Wins` · `Away Game Count` · `Away Games - Away Wins` ·
`Day Games - Day Wins` · `Day/Night Color` · `Team Runner-Up Filter` ·
`Home Games - Home Wins` · `Team Champion Filter` · `Losses` · `Home/Away Color`

## Appendix C — Package inventory

| File | Size |
|---|---:|
| `All In The Wins  #VOTD.twb` | ~2.4 MB |
| Embedded extract | included |
| Embedded bitmap (World Series trophy) | included |
| **Total `.twbx`** | **2.7 MB** |

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas, colour values and settings are quoted verbatim
from the workbook definition; rendered figures are read from the published view.*
