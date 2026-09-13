# US Presidential Elections — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `US Presidential Elections \| #MakeoverMonday` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/USPresidentialElectionsMakeoverMonday/ElectionsResults |
| Challenge | **#MakeoverMonday 2024 Week 33** |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `USPresidentialElectionsMakeoverMonday.twbx`, decompiled and parsed field-by-field |
| Explicit exclusion | 15 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic**. Listed in Appendix A. |

---

## 1. Executive summary

**US Presidential Elections** compares the Electoral College result against the popular vote for
every presidential election from **1976 to 2020**.

The workbook's technical centrepiece — and the thing the author calls out in the published
description — is a **hex map built entirely from field calculations**. There is no shapefile, no
spatial join and no background image: two `CASE` statements assign every state an X and Y
coordinate, and four offset variants of those coordinates stack four sheets into a single
composite hexagon carrying the state code, electoral votes, popular votes and winning margin.

Below the map, a twelve-row history table places the two parties either side of a shared
central axis, with crown (♚) and star (★) glyphs marking Electoral College and popular-vote
winners. The four elections where those two diverge — 2016, 2000, and the near-misses — read
immediately from the mismatch between the crown column and the star column.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | 13 |
| Dashboards | 1 (`Elections Results`) |
| Data sources | 1 (2 tables, 2 joins) |
| Parameters | 2 |
| Calculated fields **used** | 42 named + 51 in-view ad-hoc |
| Calculated fields unused (not documented) | 15 |
| Dashboard actions | 1 parameter |
| Dynamic Zone Visibility bindings | 0 |
| Published size | 933,655 bytes (912 KB) |

### 1.2 Headline figures rendered by the dashboard

Election year **2020** (default):

| Metric | Democrat | Republican |
|---|---|---|
| Candidate | **BIDEN** | TRUMP |
| Electoral Votes | **♚ 306** | 232 |
| Popular Votes | **★ 81M** | 74M |
| EC Vote Percentage | **56.9 %** | 43.1 % |

**The twelve elections**

| Year | Dem EC | Dem Pop | Democrat | Republican | Rep Pop | Rep EC |
|---|---:|---:|---|---|---:|---:|
| 2020 | ♚306 | ★81M | BIDEN | TRUMP | 74M | 232 |
| 2016 | 237 | ★66M | CLINTON | TRUMP | 63M | ♚304 |
| 2012 | ♚332 | ★66M | OBAMA | ROMNEY | 61M | 206 |
| 2008 | ♚365 | ★69M | OBAMA | MCCAIN | 60M | 173 |
| 2004 | 251 | 59M | KERRY | BUSH | ★62M | ♚286 |
| 2000 | 266 | ★51M | GORE | BUSH | 50M | ♚271 |
| 1996 | ♚379 | ★47M | CLINTON | DOLE | 39M | 159 |
| 1992 | ♚370 | ★45M | CLINTON | BUSH | 39M | 168 |
| 1988 | 111 | 42M | DUKAKIS | BUSH | ★49M | ♚426 |
| 1984 | 13 | 37M | MONDALE | REAGAN | ★54M | ♚525 |
| 1980 | 49 | 35M | CARTER | REAGAN | ★44M | ♚489 |
| 1976 | ♚297 | ★41M | CARTER | FORD | 39M | 240 |

**2016 and 2000 are the two elections where the crown and the star sit on opposite sides** —
the Electoral College winner lost the popular vote. The dashboard is built to make exactly that
comparison legible.

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `15209677` |
| LUID | `d7deafea-2025-4eea-8987-7094139640b3` |
| Repository URL | `USPresidentialElectionsMakeoverMonday` |
| Default view | `Elections Results` |
| Revision | 2.6 |
| First published | 2024-08-20 |
| Last published | 2024-11-25 |
| View count | 5,234 |
| Favourites | 105 |
| Sources | Kaggle & US National Archives |

**Published description**

> US Presidential Elections | #MakeoverMonday #presidential #elections
> Check out the #hexmap build made with only field calculations.
> #gradient #color #layers

---

## 3. Data architecture

| Property | Value |
|---|---|
| Relations | 3 — a collection joining `1976-2020-president.csv` and `1976-2020 Results` |
| Joins | **2** |
| Custom SQL | None |
| Data source filters | None |
| Columns | 24 |

**The two tables**

| Table | Supplies | Join key |
|---|---|---|
| `1976-2020-president.csv` | State-level popular vote — `candidatevotes`, `totalvotes`, `party_detailed`, `party_simplified`, `writein` | `Join (V)` |
| `1976-2020 Results` | Electoral College outcome — `Electoral Votes`, `Total Electoral Votes`, `Candidate`, `Party` | `Join (EC)` |

Joining popular-vote data to electoral-college data is what makes the central comparison
possible at all — neither source alone carries both measures.

### 3.1 Grain

One row per state × year × candidate. State-level aggregates therefore require explicit party
filtering rather than simple sums, which is why the calculation library (§5) is dominated by
`IF [Party Simplified] = "DEMOCRAT" THEN … END` patterns.

---

## 4. Parameters

| Caption | Type | Domain | Default | Role |
|---|---|---|---|---|
| **Year Parameter** | integer list | 12 members, 1976 – 2020 | `2020` | Which election the hex map shows |
| **Vote Type Parameter** | string list | `EC` (Electoral College Vote) · `P` (Popular Vote) | `EC` | Which measure the map's percentage toggle displays |

`Vote Type Parameter` is exposed as a native radio control on the dashboard ("Voting Results %
Toggle"); `Year Parameter` is driven by a parameter action on the history table.

---

## 5. Calculated fields in use

42 named calculations plus 51 in-view ad-hoc.

### 5.1 The hex map — built from calculations alone

The workbook's headline technique. Every state's position is hard-coded in two `CASE`
statements keyed on the two-letter state code:

```
X (HexMap Base)  Case [State Po] When 'AL' Then 7.5 When 'AK' Then 0.5
                                 When 'AZ' Then 3   When 'AR' Then 6
                                 When 'CA' Then 2   When 'CO' Then 3.5
                                 When 'CT' Then 11  When 'DE' Then 9.5
                                 When 'DC' Then …   END

Y (HexMap Base)  Case [State Po] When 'AL' Then 6 When 'AK' Then 0
                                 When 'AZ' Then 5 When 'AR' Then 5
                                 When 'CA' Then 5 When 'CO' Then 4
                                 When 'CT' Then 3 When 'DE' Then 4
                                 When 'DC' Then 5 … END
```

Half-step X values (7.5, 3.5, 9.5) are what interlock alternate rows into a honeycomb rather
than a square grid.

**Four offset variants** then stack four sheets into one composite hexagon:

```
X-0.25  [X (HexMap Base)] - 0.25     ← left half of the hex
X+0.25  [X (HexMap Base)] + 0.25     ← right half
Y+0.25  [Y (HexMap Base)] + 0.25     ← upper band
```

Each published hexagon therefore carries four layers: a background hex, a cover hex, the
electoral-vote figure and the popular-vote percentage — positioned by the base coordinate and
its three offsets. No spatial file is involved anywhere.

```
Min -1  -1        ← used in 7 sheets as an axis anchor
X1      'X1'      ← single-member dimension for a fixed colour
```

### 5.2 Party vote aggregation

Because the grain is state × year × candidate, party totals must be filtered explicitly:

```
*Democrat Votes    sum(if [Party Simplified] = "DEMOCRAT"   then [candidatevotes] end)
*Republican Votes  sum(if [Party Simplified] = "REPUBLICAN" then [candidatevotes] end)
*Democrat ECV      SUM(IF [Party Simplified] = "DEMOCRAT"   then [Electoral Votes] end)
*Republican ECV    SUM(IF [Party Simplified] = "REPUBLICAN" then [Electoral Votes] end)
*Votes             ({FIXED [Year],[Candidate]: SUM([candidatevotes])})
candidatevotes (M) [candidatevotes]      ← alias for a second, differently formatted use
```

### 5.3 State winner and loser

Each state resolves to a winner and a loser, with matching votes, share and electoral votes:

```
*State Winner        IF ([*Democrat Votes]) > ([*Republican Votes]) THEN 'D'
                     ELSEIF < THEN 'R' ELSE '' end
*State Winner Name   … THEN 'Democrat' … 'Republican' …
*State Winner Votes  IF [*State Winner]='D' THEN ([*Democrat Votes])
                     elseif 'R' THEN ([*Republican Votes]) ELSE 0 end
*State Winner Vote % IF [*State Winner]='D' THEN ([*Democrat Votes]/SUM([candidatevotes]))
                     elseif 'R' THEN ([*Republican Votes]/SUM([candidatevotes])) ELSE 0 end
*State Winner ECV    IF [*State Winner]='D' THEN SUM(IIF([Party]='D',[Electoral Votes],0))
                     elseif 'R' THEN SUM(IIF([Party]='R',[Electoral Votes],0)) ELSE 0 end

*State Loser, *State Loser Name, *State Loser Votes,
*State Loser Vote %, *State Loser ECV     ← the exact mirror set
```

Building winner and loser as parallel field sets rather than deriving one from the other means
a tooltip can show both sides of a state without a second pass over the data.

```
*Party Color  IF ([*Democrat Votes]) > ([*Republican Votes]) THEN 'Democrats'
              elseif < THEN 'Republicans' ELSE 'Other' end
```

### 5.4 National outcome

```
*Elected Party  CASE [Year] WHEN 2020 THEN "DEMOCRAT"   WHEN 2016 THEN "REPUBLICAN"
                            WHEN 2012 THEN "DEMOCRAT"   WHEN 2008 THEN "DEMOCRAT"
                            WHEN 2004 THEN "REPUBLICAN" WHEN 2000 THEN "REPUBLICAN" … END

*Elected Candidate  IF [*Elected Party] = [Party Simplified] THEN "♚" ELSE "" END
*Elected Candidate: DBox / RBox   the same test returning TRUE/FALSE for the two box layers

*ECV (Winner)  IF ([*Democrat ECV]) > ([*Republican ECV]) THEN [*Democrat ECV]
               elseif < THEN [*Republican ECV] ELSE 0 end
```

`*Elected Party` is a **hard-coded lookup** of who actually took office each year. It is
independent of the vote data, which is deliberate: it lets the dashboard mark the true winner
(♚) even in years where the popular-vote leader lost.

```
*Popular Vote       IF {FIXED [Year]:MAX([*Votes])} = [*Votes] Then '★' ELSE NULL END
*Popular Vote DBox / RBox   the same test returning TRUE/FALSE
```

The `{FIXED [Year]:MAX([*Votes])}` comparison finds the popular-vote leader per year. Placing
the crown and the star in separate columns is what makes the 2016 and 2000 divergences visible
at a glance.

### 5.5 The mirrored percentage bars

```
*Candidate Votes %          SUM([candidatevotes])/TOTAL(SUM([candidatevotes]))
*Candidate ECVotes %        SUM([Electoral Votes])/TOTAL(SUM([Electoral Votes]))

*Candidate Votes % (2-Bar)   IF MAX([Party Simplified])="DEMOCRAT"   THEN -[*Candidate Votes %]
                             ELSEIF MAX([Party Simplified])="REPUBLICAN" THEN [*Candidate Votes %]
                             ELSE 0 END
*Candidate ECVotes % (2-Bar) same construction on the EC measure
```

Negating the Democrat share is what pushes it left of the central axis, producing the paired
bars in the history table (56.9 % / 43.1 % for 2020).

### 5.6 Candidate name handling

```
*Candidate Clean      IF [Candidate]="MITT, ROMNEY" THEN "ROMNEY, MITT" ELSE [Candidate] END
*Candidate Last Name  TRIM( SPLIT( [*Candidate Clean], ",", 1 ) )
```

A single data-quality fix: one source record has the name reversed. Correcting it in a wrapper
field rather than editing the source keeps the fix visible and reversible — and `SPLIT` on the
comma then reliably yields the surname for every candidate.

### 5.7 Filters and formatting

```
*Year Selector Filter      [Year Parameter] = [Year]       ← 4 sheets
*Year Selector             [Year Parameter] = [Year]       ← highlights the active row
*Voting Type Abbv Filter   CASE [Vote Type Parameter] WHEN 'EC' Then 'EC' WHEN 'P' Then 'P' END
EC Votes   IF [Electoral Votes] != [Total Electoral Votes]
           THEN STR([Electoral Votes]) +"/"+ STR([Total Electoral Votes])
           ELSEIF = THEN STR([Electoral Votes]) END
```

`EC Votes` prints `3/4`-style fractions only where a state split its electoral votes, and a
plain number otherwise — which is how Maine and Nebraska are handled without cluttering the
other 48 hexagons.

---

## 6. Worksheet specifications

13 worksheets, numbered by dashboard region.

| Sheet | Role |
|---|---|
| `1-Year` | The twelve-row year column; the parameter-action source |
| `2-D Candidate` | Democrat candidate name column |
| `2-D ECVote` | Democrat electoral votes, with ♚ where elected |
| `2-D PopVote` | Democrat popular votes, with ★ where leading |
| `3-Bar ECVote` | The mirrored Electoral College percentage bars |
| `3-Bar PopVote` | The mirrored popular-vote percentage bars |
| `4-R Candidate` | Republican candidate name column |
| `4-R EC Vote` | Republican electoral votes with ♚ |
| `4-R Pop Vote` | Republican popular votes with ★ |
| `5-Hex Background` | Hex layer 1 — the base hexagon, coloured by winning party |
| `5-Hex Cover` | Hex layer 2 — the overlay carrying the state code |
| `5-Hex EC Vote` | Hex layer 3 — the electoral-vote figure (offset by `Y+0.25`) |
| `5-Hex Pop %` | Hex layer 4 — the winning margin percentage |

The four `5-Hex *` sheets are floated on top of one another at matching coordinates; together
they compose each state hexagon.

---

## 7. Dashboard specification

| Property | Value |
|---|---|
| Name | `Elections Results` |
| Canvas | 1600 × 1626 px |
| Ground | Near-white with a subtle blue-to-red gradient behind the map |
| Border | Navy frame with a red-to-blue split rule beneath the masthead |

**Regions**

```
Masthead   Republican elephant | "US Presidential Elections | 1976 - 2020" | Democrat donkey
           subtitle "Electoral College Vote vs. Popular Vote"
           legend: Democrat Win · Republican Win · Electoral College Votes of Winner ·
                   Popular Vote Percentage of Winner
Map        "Election Year 2020" over the calculated hex map, 51 hexagons
Controls   "Voting Results % Toggle" radio (Electoral College Vote / Popular Vote)
           "Hover over the Year to Change the Election Year Map"
Table      twelve rows: Year | Dem EC | Dem Pop | Democrat | bars | Republican | Rep Pop | Rep EC
Footer     "Project Makeover Monday | Challenge 2024 Week 33 | Designed by John Johansson |
            Sources Kaggle & US National Archives"
```

The **gradient** named in the description runs behind the map area — blue on the left, red on
the right — reinforcing the two-party split without colouring the map itself.

---

## 8. Interactivity

### 8.1 Parameter action (1)

| Caption | Source sheet | Trigger | Payload | Target |
|---|---|---|---|---|
| `Change Year 2 1` | `1-Year` | hover | `Year` | **Year Parameter** |

Hovering any row of the history table repoints the hex map to that election. The instruction
line beneath the map states it: *"Hover over the Year to Change the Election Year Map"*.

### 8.2 Native controls

| Control | Parameter |
|---|---|
| "Voting Results % Toggle" radio | `Vote Type Parameter` |

### 8.3 Dynamic Zone Visibility

**None.** The map's measure switch is handled by `*Voting Type Abbv Filter` as a sheet filter
rather than by zone swapping.

---

## 9. Design system

| Token | Use |
|---|---|
| Navy blue | Democrat party colour, frame, masthead text |
| Deep red | Republican party colour |
| Pale blue / pale red | Losing-party fills, low-margin states |
| Gold / amber | Electoral College vote figures, ♚ crown glyph |
| White | Dashboard ground, hex interiors |

**Glyph conventions**

| Glyph | Meaning |
|---|---|
| ♚ | Electoral College winner — took office |
| ★ | Popular-vote winner |

**Colour encoding** — hexagons are filled by winning party with intensity carrying the margin,
so safe states read strongly and swing states read pale. The margin percentage printed in each
hexagon gives the exact figure.

---

## 10. Rebuild / maintenance runbook

**Adding the 2024 election**

1. Append the new rows to both source tables, preserving the `Join (V)` / `Join (EC)` keys.
2. Add `2024` to the `Year Parameter` domain.
3. **Add a `WHEN 2024 THEN "…"` branch to `*Elected Party`.** This is a hard-coded lookup and
   will not update itself — without it the ♚ crown will not appear for the new year.
4. Everything else (state winners, margins, mirrored bars, popular-vote stars) derives from the
   data automatically.

**Editing the hex map**

`X (HexMap Base)` and `Y (HexMap Base)` hold all 51 positions. To move a state, edit both — and
remember the three offset fields (`X-0.25`, `X+0.25`, `Y+0.25`) derive from the base, so the
four layers stay aligned automatically.

Because the map is pure calculation, it has no geographic dependencies: it will render
identically on any machine, with no spatial file to package.

**Data quality**

`*Candidate Clean` exists to fix one reversed name (`MITT, ROMNEY`). If new source data arrives
with similar issues, extend that field rather than editing the source, so the correction stays
visible.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | 1976-2020-president |
| Connection class | `federated` |
| Tables / relations | 3 · joins: 2 |
| Physical columns materialised | 24 |
| Source fields in data pane | 22 |
| Calculated fields (used / total) | 42 / 57 |
| Parameters | 2 |
| Data source filters | 0 (none) |

### Source fields

22 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Candidate

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Candidate**<br>`candidate` | string · Dimension | 216 | yes | 13 | Candidate name from the Electoral College table. |
| **Party Simplified**<br>`party_simplified` | string · Dimension | 4 | yes | 13 | Party reduced to DEMOCRAT, REPUBLICAN, LIBERTARIAN or OTHER. The comparison key. |
| **Party Simplified (group)** | string · Dimension | — | — | 9 | Author-defined grouping built on **Party Simplified**. Party reduced to DEMOCRAT, REPUBLICAN, LIBERTARIAN or OTHER. The comparison key. |
| **Party** | string · Dimension | 2 | yes | 2 | Party code — D or R. |
| **Candidate1**<br>`Candidate` | string · Dimension | 18 | yes | 0 | Candidate name from the Electoral College table. The trailing digit marks a second copy of the column, brought in by a join. *(not used in any sheet)* |
| **Party Detailed**<br>`party_detailed` | string · Dimension | 153 | yes | 0 | Full party name as filed. *(not used in any sheet)* |
| **Party Simplified (group) (copy)**<br>`Party Simplified (group) (copy)_1434959482711031835` | string · Dimension | — | — | 0 | Author-defined grouping built on **Party Simplified**. Party reduced to DEMOCRAT, REPUBLICAN, LIBERTARIAN or OTHER. The comparison key. *(not used in any sheet)* |
| **Writein**<br>`writein` | boolean · Dimension | 3 | yes | 0 | Whether the candidate was a write-in. *(not used in any sheet)* |

#### Election

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Year**<br>`year` | integer · Dimension | 12 | yes | 13 | Election year. |
| **Office**<br>`office` | string · Dimension | 1 | yes | 0 | Office contested — President throughout. *(not used in any sheet)* |
| **Year1**<br>`Year` | integer · Dimension | 12 | yes | 0 | Election year. The trailing digit marks a second copy of the column, brought in by a join. *(not used in any sheet)* |

#### Votes

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **candidatevotes** | integer · Measure (Sum) | 2,365 | yes | 9 | Popular votes cast for this candidate in this state. |
| **Totalvotes**<br>`totalvotes` | integer · Measure (Sum) | 673 | yes | 0 | Total popular votes cast in the state. *(not used in any sheet)* |

#### Geography

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **State Po**<br>`state_po` | string · Dimension | 51 | yes | 4 | Two-letter state code. The key for the calculated hex map. |
| **State**<br>`state` | string · Dimension | 51 | yes | 1 | State name. |
| **State Abbv** | string · Dimension | 51 | yes | 1 | Two-letter state code. |
| **State Cen**<br>`state_cen` | integer · Measure (Sum) | 51 | yes | 0 | Census Bureau state code. *(not used in any sheet)* |
| **State Fips**<br>`state_fips` | integer · Measure (Sum) | 51 | yes | 0 | FIPS state code. *(not used in any sheet)* |
| **State Ic**<br>`state_ic` | integer · Measure (Sum) | 51 | yes | 0 | ICPSR state code. *(not used in any sheet)* |
| **State1**<br>`State` | string · Dimension | 51 | yes | 0 | State name. The trailing digit marks a second copy of the column, brought in by a join. *(not used in any sheet)* |

#### Album

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Version**<br>`version` | date · Dimension | 1 | yes | 0 | Edition qualifier. Null means the original release. *(not used in any sheet)* |

#### Metadata

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Notes**<br>`notes` | string · Dimension | 1 | yes | 0 | Source notes. *(not used in any sheet)* |

### Calculated fields in use

42 of the workbook's 57 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| ***Candidate Clean**<br>`Calculation_3526318558648238080` | string · Dimension | Basic | Candidate | *Candidate Last Name | 7 |
| ***Candidate ECVotes %**<br>`*Candidate Votes % (copy)_1765129631787495440` | real · Measure | Table calc | — | *Candidate ECVotes % (2-Bar) | 1 |
| ***Candidate ECVotes % (2-Bar)**<br>`*Candidate Votes % (2-Bar) (copy)_1765129631787597841` | real · Measure | Basic | Party Simplified, *Candidate ECVotes % | — | 1 |
| ***Candidate Last Name**<br>`Calculation_1434959482673651734` | string · Dimension | Basic | *Candidate Clean | *Candidate Last Name (2-Bar) | 7 |
| ***Candidate Votes %**<br>`*Democrat Votes % (copy)_1434959482672238612` | real · Measure | Table calc | candidatevotes | *Candidate Votes % (2-Bar) | 1 |
| ***Candidate Votes % (2-Bar)**<br>`*Candidate Votes % (copy)_1434959482672554005` | real · Measure | Basic | Party Simplified, *Candidate Votes % | — | 1 |
| ***Democrat ECV**<br>`Calculation_3171941562954674192` | integer · Measure | Basic | Party Simplified | *ECV (Winner), *Party Color (ECV) | 1 |
| ***Democrat Votes**<br>`Calculation_3171941562942390280` | integer · Measure | Basic | Party Simplified, candidatevotes | *Party Color, *State Loser, *State Loser Name, *State Loser Vote % … | 4 |
| ***ECV (Winner)**<br>`*Party Color (copy) (copy)_3171941562955370515` | integer · Measure | Basic | *Democrat ECV, *Republican ECV | — | 1 |
| ***Elected Candidate**<br>`*Elected Party (copy)_3171941562937012226` | string · Dimension | Basic | *Elected Party, Party Simplified | — | 2 |
| ***Elected Candidate: DBox**<br>`*Elected Candidate (copy)_1434959482713165860` | boolean · Dimension | Basic | *Elected Party, Party Simplified | — | 1 |
| ***Elected Candidate: RBox**<br>`*Elected Candidate: DBox (copy)_1434959482718310454` | boolean · Dimension | Basic | *Elected Party, Party Simplified | — | 1 |
| ***Elected Party**<br>`Calculation_2357915930215763968` | string · Dimension | Basic | Year | *Elected Candidate, *Elected Candidate: DBox, *Elected Candidate: RBox | 2 |
| ***Party Color**<br>`Calculation_3171941562943135754` | string · Measure | Basic | *Democrat Votes, *Republican Votes | — | 2 |
| ***Popular Vote**<br>`*x votes (copy)_3171941562939637764` | string · Dimension | LOD | Year, *Votes | — | 4 |
| ***Popular Vote DBox**<br>`*Popular Vote (copy)_1434959482713112611` | boolean · Dimension | LOD | Year, *Votes | — | 1 |
| ***Popular Vote RBox**<br>`*Popular Vote DBox (copy)_1434959482719133751` | boolean · Dimension | LOD | Year, *Votes | — | 1 |
| ***Republican ECV**<br>`*Democrat ECV (copy)_3171941562954874897` | integer · Measure | Basic | Party Simplified | *ECV (Winner), *Party Color (ECV) | 1 |
| ***Republican Votes**<br>`*Democrat Votes (copy)_3171941562942476297` | integer · Measure | Basic | Party Simplified, candidatevotes | *Party Color, *State Loser, *State Loser Name, *State Loser Vote % … | 4 |
| ***State Loser**<br>`*State Winner (copy)_1765129631775715334` | string · Measure | Basic | *Democrat Votes, *Republican Votes | *State Loser ECV, *State Loser Vote %, *State Loser Votes | 1 |
| ***State Loser ECV**<br>`*State Winner ECV (copy)_1765129631780892684` | integer · Measure | Basic | *State Loser, Party | — | 1 |
| ***State Loser Name**<br>`*State Winner Name (copy)_1765129631776763914` | string · Measure | Basic | *Democrat Votes, *Republican Votes | — | 1 |
| ***State Loser Vote %**<br>`*State Winner Vote % (copy)_1765129631775801351` | real · Measure | Basic | *State Loser, *Democrat Votes, candidatevotes, *Republican Votes | — | 1 |
| ***State Loser Votes**<br>`*State Winner Votes (copy)_1765129631775911944` | integer · Measure | Basic | *State Loser, *Democrat Votes, *Republican Votes | — | 1 |
| ***State Winner**<br>`*Votes of Winner (State) (copy)_1434959482653708296` | string · Measure | Basic | *Democrat Votes, *Republican Votes | *State Winner ECV, *State Winner Vote %, *State Winner Votes | 4 |
| ***State Winner ECV**<br>`*State Winner (copy)_1765129631777394699` | integer · Measure | Basic | *State Winner, Party | *State Winner ECV-OtherVotes | 2 |
| ***State Winner Name**<br>`*State Winner (copy)_1765129631776661513` | string · Measure | Basic | *Democrat Votes, *Republican Votes | — | 1 |
| ***State Winner Vote %**<br>`*State Winner Votes (copy)_1434959482654707721` | real · Measure | Basic | *State Winner, *Democrat Votes, candidatevotes, *Republican Votes | — | 2 |
| ***State Winner Votes**<br>`*Party Color (copy)_1434959482648281093` | integer · Measure | Basic | *State Winner, *Democrat Votes, *Republican Votes | — | 2 |
| ***Votes**<br>`Calculation_3171941562938822659` | integer · Measure | LOD | Year, Candidate, candidatevotes | *Popular Vote, *Popular Vote DBox, *Popular Vote RBox | 4 |
| ***Voting Type Abbv Filter**<br>`*Voting Type (copy)_1765129631797805079` | string · Dimension | Basic | — | — | 2 |
| ***Year Selector**<br>`Calculation_1765129631767425025` | boolean · Dimension | Basic | Year | — | 1 |
| ***Year Selector Filter**<br>`*Year Selector (copy)_1765129631768895492` | boolean · Dimension | Basic | Year | — | 4 |
| **candidatevotes (M)**<br>`candidatevotes (copy)_1765129631789785106` | integer · Measure | Basic | candidatevotes | — | 4 |
| **EC Votes**<br>`Calculation_1434959482644398082` | string · Dimension | Basic | — | — | 1 |
| **Min -1**<br>`Min -1 (copy)_1434959482716368947` | integer · Measure | Basic | — | — | 7 |
| **X (HexMap Base)**<br>`Calculation_2357915930219773954` | real · Measure | Basic | State Po | X+0.25, X-0.25 | 4 |
| **X+0.25**<br>`X-0.25 (copy)_1434959482646179843` | real · Measure | Basic | X (HexMap Base) | — | 1 |
| **X-0.25**<br>`X (copy)_1808476776226762752` | real · Measure | Basic | X (HexMap Base) | — | 1 |
| **X1**<br>`Calculation_1765129631745941504` | string · Dimension | Basic | — | — | 1 |
| **Y  (HexMap Base)**<br>`Calculation_2357915930219671553` | integer · Measure | Basic | State Po | Y+0.25, Y-0.05 | 4 |
| **Y+0.25**<br>`X-25 (copy)_1808476776229109761` | real · Measure | Basic | Y  (HexMap Base) | — | 2 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| **Year Parameter**<br>`Parameter 1` | integer · list | `2020` | list of 12 — 1976, 1980, 1984, 1988, 1992, 1996 |
| **Vote Type Parameter**<br>`Year Parameter (copy)_1765129631795494932` | string · list | `"EC"` | list of 2 — "EC" → Electoral College Vote, "P" → Popular Vote |

### How the numbers are computed

**Level-of-detail expressions — 4.** These fix their own level of detail, so they return the same value regardless of what dimensions are in the view:

> `*Popular Vote DBox`, `*Popular Vote RBox`, `*Popular Vote`, `*Votes`

**Table calculations — 2.** These compute across the rendered table, so their result depends on what is in the view at the time:

> `*Candidate ECVotes %`, `*Candidate Votes %`

The workbook file records only each table calculation's `ordering-type`. The addressing and partitioning described elsewhere in this document are **derived from each sheet's level-of-detail shelf**, not read from the file.

## Appendix A — Calculated fields excluded from this documentation

15 calculated fields exist in the data pane but are not referenced by any worksheet or dashboard
logic:

`*Candidate EC Adjust` · `*Candidate Last Name (2-Bar)` · `*Candidate V Adjust` ·
`*RepublicanVotes %` · `*Democrat Votes %` · `*P` · `*Party Color (ECV)` ·
`*State Winner ECV-OtherVotes` · `*Voting Type` · `0` · `*EC True` · `*EC` ·
`*Max Votes by State` · `% of Votes` · `Y-0.05`

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas and settings are quoted verbatim from the workbook
definition; rendered figures are read from the published view.*
