# Taylor Swift Album Sound Wave — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `Taylor Swift Album Sound Wave \| B2VB` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/TaylorSwiftAlbumSoundWave/TS-Rows |
| Challenge | **#B2VB 2024 Week 05** — "Play with Size" |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `TaylorSwiftAlbumSoundWave.twbx`, decompiled and parsed field-by-field |
| Explicit exclusion | 6 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic**. Listed in Appendix A. |

---

## 1. Executive summary

**Taylor Swift Album Sound Wave** renders every track of a selected Taylor Swift album as a
**sound wave** — ten coloured bars, one per Spotify audio attribute, mirrored above and below a
centre line.

This is the **earlier** of the author's two sound-wave workbooks, published three months before
*Run The Jewels — Audio Vibe Waves*. The normalisation engine is identical (§5.2) and was
clearly carried forward; what differs is the layout. Here the tracks are laid out as a **grid of
21 small multiples**, seven per row, so a whole album's worth of signatures can be compared at
once. The RTJ build instead uses a single-column track table with the wave as one column.

The challenge brief was "Play with Size", and the response is literal: size is the only encoding
that varies. Every wave uses the same ten colours in the same order, so the shape alone carries
the track's character.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | 11 |
| Dashboards | 1 (`TS-Rows`) |
| Data sources | 1 (single table, 8 columns) |
| Parameters | 1 |
| Calculated fields **used** | 10 named + 11 in-view ad-hoc |
| Calculated fields unused | 6 |
| Dashboard actions | **0** |
| Dynamic Zone Visibility bindings | 0 |
| Extract rows | 6,886 |
| Published size | 121,864 bytes (119 KB) |

### 1.2 As rendered

| Element | Value |
|---|---|
| Selected album | **1989 (Taylor's Version)** |
| Tracks shown | 21, in three rows of seven |
| Attributes | Instrumentalness · Acousticness · Liveness · Energy · Danceability · Loudness · Popularity · Valence · Tempo · Speechiness |

An album-average wave sits in the top-left corner, labelled `ALBUM`, giving a reference
signature against which each track can be read.

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `14210922` |
| LUID | `1a9a584c-9a32-42ce-85b5-9b520ab5e943` |
| Repository URL | `TaylorSwiftAlbumSoundWave` |
| Default view | `TS-Rows` |
| Revision | 1.8 |
| First published | 2024-03-08 |
| Last published | 2024-03-08 |
| View count | 1,338 |
| Favourites | 4 |
| Source | Spotify · Measures: Spotify Scoring |

**Published description**

> #B2VB 2024W05 | 'Play with Size'| Spotify scoring metrics | Taylor Swift's discography |
> Eras Tour Color Scheme | Album & Track Sound Waves | Hover to see Scoring Details
> #TaylorSwift #Music #SoundWave

---

## 3. Data architecture

| Property | Value |
|---|---|
| Data source | `Taylor Swift Spotify` |
| Relations | 1 table |
| Joins | **None** |
| Data source filters | None |
| Columns | **8** |
| Rows | **6,886** |

| Column | Role |
|---|---|
| `id` | Spotify track ID |
| `name` | Track name |
| `album` | Album name |
| `release_date` | |
| `track_number` | Position on the album |
| `attribute names` | **Which audio feature** |
| `attribute values` | Its value |
| `uri` | Spotify URI |

### 3.1 Grain

One row per track × attribute — the pivoted shape. 6,886 rows across the full discography is
roughly 689 tracks × 10 attributes. This is what lets ten attributes plot from a single measure.

---

## 4. Parameters

| Caption | Type | Domain | Default | Role |
|---|---|---|---|---|
| **Album** | string list | **27 members** | `1989 (Taylor's Version)` | Which album is displayed |

Twenty-seven albums — the full discography including Taylor's Version re-recordings and deluxe
editions.

---

## 5. Calculated fields in use

10 named calculations plus 11 in-view ad-hoc.

### 5.1 Album filtering

```
Album Filter  IF [Album ] = [Album] then TRUE else FALSE end     ← 11 sheets
```

Applied on every sheet. Note the trailing space in the parameter name `[Album ]`, which
distinguishes it from the data field `[Album]`.

### 5.2 The normalisation chain

Identical in structure to the RTJ workbook — this is where the pattern originates.

```
Attribute Values Normalized
    if     [Attribute Names] = 'loudness'    then ((-60 - [Attribute Values]) * -1)
    ELSEIF [Attribute Names] = 'duration_ms' then ([Attribute Values] / 60000)
    else   [Attribute Values] END                                    ← 11 sheets

Max A Score   if [Attribute Names] = 'acousticness'  then 1
              elseif 'danceability' then 1
              elseif 'duration_ms'  then 12
              elseif …                                               ← per-attribute ceiling

Min A Score   the matching floor per attribute                       ← 8 sheets

Score %       [Attribute Values Normalized] / [Max A Score]          ← 11 sheets
Score %*-1    [Score %] * -1                                         ← 9 sheets
```

`loudness` arrives as a negative decibel value and is inverted into a positive range;
`duration_ms` becomes minutes; the rest already sit on 0–1. `Max A Score` then supplies a
per-attribute denominator so every attribute lands on a comparable scale.

`Score %` and `Score %*-1` on a dual axis produce the mirrored waveform — the single technique
that defines both sound-wave workbooks.

```
Attribute Values Text  if [Attribute Names] = 'duration_ms' then ([Attribute Values]/60000)
                       else [Attribute Values] END
Spotify Attribute      [Attribute Names]
```

`Attribute Values Text` is the tooltip variant — it converts duration to minutes for display but
leaves everything else at its native value, so the hover shows real Spotify figures rather than
normalised ones.

### 5.3 Track selection

```
Tracks         IF [Id] IN ('08J4bMHbILkwSIhbtrcvjN', '3EPgWM1zfTSzEc0z4AwWTM',
                           '28tIMaewZuGR8urkvFhQyA', '7N015NTIWGjQDddHUdHPoO', … )
Tracks (copy)  the same membership list
```

Used in 11 and 7 sheets respectively. A hard-coded list of Spotify track IDs selecting which
tracks appear — a frozen selection rather than a computed one, which keeps the grid at exactly
21 tiles regardless of what the album parameter returns.

---

## 6. Worksheet specifications

11 worksheets.

| Sheet | Role |
|---|---|
| `Sound Wave` | The base wave construction |
| `Sound Wave 1-7` | Tracks 1–7 — the first grid row |
| `Sound Wave 8-14` | Tracks 8–14 — second row |
| `Sound Wave 15-21` | Tracks 15–21 — third row |
| `Sound Wave 22-28` | Tracks 22–28 — fourth row, for longer albums |
| `Sound Wave 29-35` | Tracks 29–35 |
| `Sound Wave 36-42` | Tracks 36–42 |
| `Sound Wave 42-46` | Tracks 42–46 |
| `Sound Wave Album Avg` | The album-average reference wave, top-left |
| `Tooltip Bar`, `Tooltip Bar Avg` | Viz-in-tooltip sheets showing scoring detail on hover |

**Seven separate sheets for seven track ranges.** Rather than one sheet with track number on
columns wrapping to multiple rows — which Tableau cannot do natively — each row of the grid is
its own sheet, filtered to its track range and stacked on the dashboard. Seven ranges cover up
to 46 tracks, enough for the longest release in the discography.

---

## 7. Dashboard specification

| Property | Value |
|---|---|
| Name | `TS-Rows` |
| Canvas | 1600 × 900 px, fixed |
| Ground | Cream / off-white |

**Regions**

```
Top-left    "ALBUM" label with the album-average wave
Masthead    "TAYLOR SWIFT'S" · "'1989 | Taylor's Version' Album Sound Wave"
Top-right   Album parameter control
Legend      ten attribute names with colour swatches across one row
Grid        "TRACKS" — three visible rows of seven waves, each numbered
Footer      "Project Back to Viz Basics | Challenge 2024 Week 05 - Play with Size |
             Designed by John Johansson | Source Spotify | Measures Spotify Scoring"
```

Alternate columns carry a very light grey background band, which is what keeps the eye tracking
across a row of otherwise free-floating waveforms.

---

## 8. Interactivity

**No actions and no Dynamic Zone Visibility.**

| Mechanism | Effect |
|---|---|
| `Album` parameter control | Re-filters all 11 sheets via `Album Filter` |
| `Tooltip Bar` / `Tooltip Bar Avg` | Viz-in-tooltip on hover — the "Hover to see Scoring Details" the description promises |

The workbook predates the author's adoption of parameter actions and Dynamic Zone Visibility;
the RTJ successor three months later adds both.

---

## 9. Design system

The description names the scheme: **Eras Tour colours**.

| Attribute | Colour |
|---|---|
| Instrumentalness | pale sage |
| Acousticness | soft gold |
| Liveness | lilac |
| Energy | deep crimson |
| Danceability | pale blue |
| Loudness | warm grey |
| Popularity | pink |
| Valence | mauve |
| Tempo | tan |
| Speechiness | navy |

| Token | Use |
|---|---|
| Cream / off-white | Dashboard ground |
| Charcoal `#353535` | All text — set as a global colour |
| Pale grey | Alternate column bands |

**Typography** — the workbook sets a global **colour** (`#353535`) rather than a global font,
applied to both `worksheet` and `all` elements.

---

## 10. Rebuild / maintenance runbook

**Adding an album**

1. Add the tracks and their audio features to the source.
2. Add the album to the `Album` parameter domain (currently 27 members).
3. **Add the track IDs to `Tracks` and `Tracks (copy)`.** Both hold hard-coded Spotify ID lists;
   a new album's tracks will not appear until they are listed in both.

**Adding an attribute**

Edit `Max A Score` and `Min A Score` to give the attribute a ceiling and floor, and add a
rescaling branch to `Attribute Values Normalized` if its native unit is not already usable.
Then add its colour to the legend.

**Track counts above 46**

The grid covers tracks 1–46 across seven sheets. A longer release needs an eighth sheet
(`Sound Wave 47-53`), filtered to the new range and added to the dashboard's vertical flow.

**Relationship to the RTJ workbook**

`Attribute Values Normalized`, `Max A Score`, `Min A Score`, `Score %` and `Score %*-1` are
shared verbatim with `Run The Jewels - Audio Vibe Waves`. A fix to the normalisation logic in
one should be mirrored in the other.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | Taylor Swift Spotify (taylor_swift_spotify_clean) |
| Connection class | `federated` |
| Tables / relations | 1 · joins: 0 |
| Extract rows | 6,886 |
| Physical columns materialised | 8 |
| Source fields in data pane | 8 |
| Calculated fields (used / total) | 10 / 16 |
| Parameters | 1 |
| Data source filters | 0 (none) |

### Source fields

8 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Album

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Album**<br>`album` | string · Dimension | 27 | yes | 11 | Album name. |
| **Track Number**<br>`track_number` | integer · Dimension | 60 | yes | 11 | Position of the track on the album. |
| **Release Date**<br>`release_date` | date · Dimension | 22 | yes | 9 | Release date. |

#### Audio

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Attribute Names**<br>`attribute names` | string · Dimension | 11 | yes | 11 | Which audio feature this row carries — the pivot key. |
| **Attribute Values**<br>`attribute values` | real · Measure (Sum) | 1,792 | yes | 11 | The feature's raw value on its own native scale. |

#### Identity

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Id**<br>`id` | string · Dimension | 711 | yes | 11 | Spotify track ID. Written into the `Song` parameter by the click action. |

#### Catalog scaffold

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Name**<br>`name` | string · Dimension | 418 | yes | 9 | Chart name as listed in the catalog. |

#### Links

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Uri**<br>`uri` | string · Dimension | 711 | yes | 0 | Spotify URI, used to point the embedded player at a track. *(not used in any sheet)* |

### Calculated fields in use

10 of the workbook's 16 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| **Album Filter**<br>`Calculation_978970031638634496` | boolean · Dimension | Basic | Album | — | 11 |
| **Attribute Values Normalized**<br>`Attribute Values (copy)_1045116651141586954` | real · Measure | Basic | Attribute Names, Attribute Values | Score % | 11 |
| **Attribute Values Text**<br>`Attribute Values Normalized (copy)_978970031650717697` | real · Measure | Basic | Attribute Names, Attribute Values | — | 2 |
| **Max A Score**<br>`Min A Score (copy)_1045116651139645448` | integer · Measure | Basic | Attribute Names | Score % | 11 |
| **Min A Score**<br>`Calculation_1045116651138056199` | integer · Measure | Basic | Attribute Names | — | 8 |
| **Score %**<br>`Max A Score (copy)_1045116651140640777` | real · Measure | Basic | Attribute Values Normalized, Max A Score | Score %*-1 | 11 |
| **Score %*-1**<br>`TEST % (copy)_1045116651144765452` | real · Measure | Basic | Score % | — | 9 |
| **Spotify Attribute**<br>`Attribute Names (copy)_1045116651145641997` | string · Dimension | Basic | Attribute Names | — | 11 |
| **Tracks**<br>`Track Number (copy)_978970031660871682` | integer · Dimension | Basic | Id, Track Number | t_max points | 11 |
| **Tracks (copy)**<br>`Tracks (copy)_1359242725557686275` | integer · Dimension | Basic | Id, Track Number | — | 7 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| **Album**<br>`Parameter 1` | string · list | `"1989 (Taylor's Version)"` | list of 27 — "1989", "1989 (Deluxe Edition)" → 1989 \\| Deluxe Edition, "1989 (Taylor's Version)" → 1989 \\| Taylor's Version, "1989 (Taylor's Version) [Deluxe]" → 1989 \\| Taylor's Version \\| Deluxe, "evermore", "evermore (deluxe version)" → evermore \\| Deluxe Edition … |

## Appendix A — Calculated fields excluded from this documentation

6 calculated fields exist in the data pane but are not referenced by any worksheet or dashboard
logic:

`Index X-1` · `0` · `t_index` · `Index X+1` · `Tracks (copy 2)` · `t_max points`

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas and settings are quoted verbatim from the workbook
definition; rendered figures are read from the published view.*
