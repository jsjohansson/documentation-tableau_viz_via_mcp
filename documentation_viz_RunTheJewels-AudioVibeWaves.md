# Run The Jewels — Audio Vibe Waves — Technical Documentation

**Tableau Public workbook · full specification of data, fields, calculations, parameters, worksheets, dashboard and design system**

---

## 0. Document control

| Item | Value |
|---|---|
| Subject workbook | `Run The Jewels - Audio Vibe Waves` |
| Author | John Johansson (`john.johansson`) |
| Live URL | https://public.tableau.com/app/profile/john.johansson/viz/RunTheJewels-AudioVibeWaves/RTJAudioVibeWaves |
| Documentation scope | Complete workbook internals — data model, field dictionary, parameters, **calculated fields that are actually used in sheets**, worksheet construction, dashboard layout, interactivity and design system |
| Source of record | `RunTheJewels-AudioVibeWaves.twbx`, decompiled and parsed field-by-field |
| Explicit exclusion | 6 calculated fields exist in the data pane but are **not referenced by any worksheet or dashboard logic**. Listed in Appendix A. |

---

## 1. Executive summary

**Audio Vibe Waves** profiles the Run The Jewels discography using Spotify's audio-feature API:
every track is rendered as a **sound wave** — nine coloured bars, one per audio attribute,
mirrored above and below a centre line so each track reads as a waveform signature.

Nine Spotify attributes are plotted on wildly different native scales — `loudness` runs from
−60 to 0 dB, `duration_ms` in milliseconds, `tempo` in BPM, and six others from 0 to 1. The
workbook's core engineering is the **normalisation chain** (§5.2) that brings all nine onto a
common 0–100 % scale so they can share one axis.

The dashboard also embeds a **live Spotify player**: clicking a track fires a parameter action
that repoints a web-page object at that track's Spotify URI, so the reader can hear what they
are looking at.

### 1.1 Workbook at a glance

| Metric | Value |
|---|---|
| Tableau file version | 18.1 |
| Worksheets | 11 |
| Dashboards | 1 (`RTJ Audio Vibe Waves`) |
| Data sources | 1 (4 tables, **4 joins**) |
| Parameters | 2 |
| Calculated fields **used** | 17 named + 15 in-view ad-hoc |
| Calculated fields unused | 6 |
| Dashboard actions | 1 parameter |
| Dynamic Zone Visibility bindings | 10 |
| Global font | Calibri |
| Published size | 1,663,740 bytes (1.6 MB) |

### 1.2 As rendered

| Element | Value |
|---|---|
| Selected album | **RTJ4 (Deluxe Edition)** — album 4 |
| Album detail | Deluxe Edition Mix · 2021 Release Year · **26 Tracks** |
| Attributes plotted | Instrumentalness · Acousticness · Speechiness · Popularity · Energy · Loudness · Danceability · Tempo · Valence · Liveness |
| Track list | Track number, name, edition, featured artist, length, and the track's wave |

Example tracks: *yankee and the brave (ep. 4)* 2:44 · *ooh la la* (Greg Nice & DJ Premier) 3:02 ·
*out of sight* (2 Chainz) 3:35 · *holy calamafuck* 3:97 · *goonies vs. E.T.* 3:05

---

## 2. Publication metadata

| Property | Value |
|---|---|
| Workbook ID | `14839491` |
| LUID | `63dce6ee-1708-48d0-be5d-0da0adf3df15` |
| Repository URL | `RunTheJewels-AudioVibeWaves` |
| Default view | `RTJ Audio Vibe Waves` |
| Revision | 1.1 |
| First published | 2024-06-04 |
| Last published | 2024-06-04 |
| View count | 373 |
| Source | Spotify API — Audio Features |
| Attribution | Inspired by **Shreya Arya** — *Taylor Swift - Spotify Audio Features \| Back 2 Viz Basics* |

**Published description**

> #SpotifyAPI - Audio Features | Album Sound Waves - RTJ Edition
> Click on a track to hear it on the player.
> #SoundWave #VisibilityControls #Dumbbell #SpotifyPlayer

---

## 3. Data architecture

| Property | Value |
|---|---|
| Relations | 5 — a collection joining `Albums`, `All Album Tracks`, `Tracks`, `Track Details Short` |
| Joins | **4** |
| Data source filters | None |
| Columns | 30+ |

| Table | Supplies |
|---|---|
| `Albums` | Album name, version, release date, total tracks, cover art URL, Spotify URI |
| `All Album Tracks` | Track listing and ordering |
| `Tracks` | Audio features in attribute-name / attribute-value form |
| `Track Details Short` | Track IDs used by the player action |

### 3.1 Grain

The audio features arrive **pivoted** — one row per track × attribute, with `Attribute Names`
and `Attribute Values` columns. That shape is what allows nine attributes to be plotted from a
single measure, and it is why the normalisation logic (§5.2) is written as a long `IF` chain
keyed on the attribute name rather than as nine separate fields.

---

## 4. Parameters

| Caption | Type | Domain | Default | Role |
|---|---|---|---|---|
| **Album** | string list | 9 members | `RTJ4 (Deluxe Edition)` | Which album is displayed |
| **Song** | string | any | `5yBGniyWqeALcoBpXktoby` | The selected track's Spotify ID — written by the click action, read by the player |

---

## 5. Calculated fields in use

17 named calculations plus 15 in-view ad-hoc.

### 5.1 Album filtering

```
Album Filter  IF [Album] = [Album Name] then TRUE else FALSE end     ← 9 sheets
```

### 5.2 Normalising nine incompatible scales

The workbook's central problem. Spotify's audio features use unrelated units, so they must be
mapped onto a common range before they can share an axis.

```
Attribute Values Normalized
    if     [Attribute Names] = 'loudness'    then ((-60 - [Attribute Values]) * -1)
    ELSEIF [Attribute Names] = 'duration_ms' then ([Attribute Values] / 60000)
    else   [Attribute Values] END
```

Two attributes need rescaling: `loudness` is a negative decibel value from −60 upward, inverted
here into a positive 0–60 range; `duration_ms` becomes minutes. The remaining seven already sit
on usable scales.

```
Max A Score   if [Attribute Names] = 'acousticness'  then 1
              elseif 'danceability' then 1
              elseif 'duration_ms'  then 12
              elseif …                                          ← the ceiling per attribute

Min A Score   the matching floor per attribute (mostly 0)

Score %       [Attribute Values Normalized] / [Max A Score]      ← 8 sheets
Score %*-1    [Score %] * -1                                     ← 4 sheets
```

`Max A Score` supplies a per-attribute denominator — 1 for the 0-to-1 features, 12 for duration
in minutes, and so on — so `Score %` lands every attribute in 0–1 regardless of its native unit.
`Score %*-1` is the mirrored half: plotting both on a dual axis is what produces the symmetric
waveform.

### 5.3 The wave ordering

```
*SoundWave  If     [Attribute Names] = 'instrumentalness' Then 1
            ELSEIF [Attribute Names] = 'acousticness'     Then 2
            ELSEIF [Attribute Names] = 'speechiness'      Then 3
            ELSEIF …
```

Assigns each attribute a fixed position along the wave, so every track's signature is directly
comparable — the third bar is always speechiness, for every track on every album.

### 5.4 Album colour coding — the DZV controls

Five booleans group the nine albums into five colour families:

```
*Case Green   CASE [Album] When 'Run The Jewels' THEN TRUE
                           When 'Run The Jewels Instrumentals' THEN TRUE ELSE FALSE END
*Case Red     When 'Meow the Jewels' THEN TRUE When 'Run the Jewels 2' THEN TRUE ELSE FALSE END
*Case Blue    When 'Run The Jewels 3' THEN TRUE ELSE FALSE END
*Case Pink    When 'RTJ4' THEN TRUE When 'RTJ CU4TRO' THEN TRUE ELSE FALSE END
*Case Yellow  When 'RTJ4 (Deluxe Edition)' THEN TRUE When 'Spotify Sessions' THEN TRUE ELSE FALSE END
```

Each controls **two zones** — an X rule and a Y rule — for ten bindings total. Selecting an
album shows the accent rules in that album's colour, which is what makes the yellow framing in
the published view (RTJ4 Deluxe) change with the selection.

### 5.5 The player

```
*Default Track  '5yBGniyWqeALcoBpXktoby'
P               IF [Song] = [track id (Track Details Short)] then '♫' ELSE '' END
```

`P` places a ♫ glyph beside whichever track is currently loaded in the player — visible on
track 2 in the published view.

### 5.6 Display fields

```
Edition          IF ISNULL([Version]) Then 'Original' ELSE [Version] End
Featured Artist  IF ISNULL([Feature]) Then '' ELSE [Feature] End
Track Length     Round(([Track Duration Ms] / 60000), 2) * 100
```

---

## 6. Worksheet specifications

11 worksheets.

| Sheet | Role |
|---|---|
| `Album Art` | The album cover image tile |
| `Album #` | The large album number (4) |
| `Album Text` | Album name, edition, year and track count |
| `Album Wave` | The album-level average wave, top right |
| `Artist Wave` | The full-discography dumbbell chart in the masthead |
| `Tracks` | The track table — number, name, edition, featured artist, length |
| `Soundbar base` | The per-track wave marks |
| `Min-Max` | Axis anchor sheet holding the normalised range |
| `Tooltip Bar`, `Tooltip Bar Avg`, `Tooltip Bar Avg Artist` | Viz-in-tooltip sheets showing scoring detail on hover |

---

## 7. Dashboard specification

| Property | Value |
|---|---|
| Name | `RTJ Audio Vibe Waves` |
| Canvas | 1600 × 1200 px, fixed |
| Ground | Black |

**Regions**

```
Masthead    Run The Jewels logo · band photograph · "Audio Vibe Waves"
            right: the Album dumbbell chart — one row per album, nine attribute
            positions, with min/max percentages labelled
            credits: Designed by John Johansson · Source Spotify API ·
            Measures Spotify Audio Features · Spotify logo
Legend      nine colour-coded attribute names across one row
Album panel album art · album number · name, edition, year, track count ·
            album-average wave
Track table Track | Name | Edition | Featured Artist | Track Length | Track Wave
Player      embedded Spotify web player, repointed by the Song parameter
```

---

## 8. Interactivity

### 8.1 Parameter action (1)

| Caption | Trigger | Payload | Target |
|---|---|---|---|
| `Change Song 1 2 1 1 1 1` | on-select | the track's Spotify ID | **Song** |

Clicking a track writes its Spotify ID into `Song`; the embedded player object's URL is
parameterised on that value, so the track begins playing. The `P` calculation marks the loaded
track with ♫.

### 8.2 Dynamic Zone Visibility (10 bindings)

| Field | Zones |
|---|---|
| `*Case Green` | Green X rule · Green Y rule |
| `*Case Red` | Red X rule · Red Y rule |
| `*Case Blue` | Blue X rule · Blue Y rule |
| `*Case Pink` | Pink X rule · Pink Y rule |
| `*Case Yellow` | Yellow X rule · Yellow Y rule |

The album's accent colour is applied by showing one of five pre-built rule pairs rather than by
a colour calculation — the "#VisibilityControls" the description names.

---

## 9. Design system

| Token | Use |
|---|---|
| Black | Dashboard ground — a record-sleeve treatment |
| White | Logo, headings, track names |
| Yellow / gold | RTJ4 Deluxe accent (the published selection) |
| Green · Red · Blue · Pink | The other four album accent families |
| Nine-colour legend | Green instrumentalness · purple acousticness · cyan speechiness · blue popularity · teal energy · magenta loudness · green danceability · orange tempo · yellow valence · white liveness |

The nine attribute colours are the workbook's real palette: they appear in the legend, the
masthead dumbbells, the album average wave and every track wave, so a reader learns the mapping
once and reads every waveform after that.

**Typography** — Calibri, set at workbook level.

---

## 10. Rebuild / maintenance runbook

**Adding an album**

1. Add the album's tracks and audio features to the source tables.
2. Add the album to the `Album` parameter domain.
3. Assign it to one of the five `*Case <Colour>` booleans — or add a sixth colour family, which
   means a new boolean plus two new zones and two new DZV bindings.

**Adding an audio attribute**

Three fields must be edited together or the wave will misalign:

1. `*SoundWave` — give the attribute its position in the wave order.
2. `Max A Score` — supply its ceiling.
3. `Min A Score` — supply its floor.
4. `Attribute Values Normalized` — add a rescaling branch if its native unit is not already
   usable.

**The player**

The `Song` parameter holds a raw Spotify track ID. If the source IDs change, `*Default Track`
must be updated too, or the dashboard will load with an empty player.

---

## Data Dictionary

A metadata reference for every field the workbook exposes, written so it can be read on its own. **Source fields** are physical columns that arrive from the data source, each with an explanation of what the column actually records. **Calculated fields** are derived inside the workbook; only those a worksheet or dashboard rule actually references are catalogued here, and the rest are listed by name in the appendix.

### Overview

| Property | Value |
|---|---|
| Data source | RTJ Short |
| Connection class | `federated` |
| Tables / relations | 5 · joins: 4 |
| Physical columns materialised | 38 |
| Source fields in data pane | 37 |
| Calculated fields (used / total) | 17 / 23 |
| Parameters | 2 |
| Data source filters | 0 (none) |

### Source fields

37 physical columns, grouped by subject. **Sheets** counts the worksheets that reference the field directly — a zero means the column is carried for context or feeds a calculation rather than being placed on a shelf. **Distinct** and **Nulls** are read from the extract's own column metadata where the extract records them.

#### Audio

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Attribute Names** | string · Dimension | 11 | yes | 11 | Which audio feature this row carries — the pivot key. |
| **Track Duration Ms**<br>`track_duration_ms` | integer · Measure (Sum) | 96 | yes | 1 | Track length in milliseconds. Converted to minutes for display. |
| **Duration Ms**<br>`duration_ms` | integer · Measure (Sum) | 96 | yes | 0 | Track length in milliseconds. Normalised by dividing by 60000. *(not used in any sheet)* |
| **Explicit**<br>`explicit` | boolean · Dimension | 2 | yes | 0 | Explicit content flag. *(not used in any sheet)* |
| **Popularity**<br>`popularity` | integer · Measure (Sum) | 46 | yes | 0 | Spotify popularity score, 0 to 100, reflecting recent play volume. *(not used in any sheet)* |

#### Album

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Album Name**<br>`album_name` | string · Dimension | 9 | yes | 10 | Album name. |
| **Track**<br>`track_number` | integer · Dimension | 26 | yes | 2 | Position of the track on the album. |
| **Album Release Date**<br>`album_release_date` | date · Dimension | 9 | yes | 1 | Album release date. |
| **Album Total Tracks**<br>`album_total_tracks` | integer · Measure (Sum) | 6 | yes | 1 | Number of tracks on the album. |
| **album_version** | string · Dimension | 6 | yes | 1 | Edition marker — Deluxe, Instrumentals and similar. |
| **Album**<br>`album` | string · Dimension | 9 | yes | 0 | Album name. *(not used in any sheet)* |
| **Album Id**<br>`album_id` | string · Dimension | 9 | yes | 0 | Spotify album ID. *(not used in any sheet)* |
| **Album Images.Url**<br>`album_images.url` | string · Dimension | 9 | yes | 0 | Album cover art URL. *(not used in any sheet)* |
| **Album Name (group)** | string · Dimension | — | — | 0 | Author-defined grouping built on **Album Name**. Album name. *(not used in any sheet)* |
| **Album.Id**<br>`album.id` | string · Dimension | 9 | yes | 0 | Spotify album ID. *(not used in any sheet)* |
| **Album.Name**<br>`album.name` | string · Dimension | 9 | yes | 0 | Album name. *(not used in any sheet)* |
| **Album.Release Date**<br>`album.release_date` | string · Dimension | 9 | yes | 0 | Album release date. *(not used in any sheet)* |
| **Album.Total Tracks**<br>`album.total_tracks` | integer · Measure (Sum) | 6 | yes | 0 | Number of tracks on the album. *(not used in any sheet)* |
| **Artists.Id**<br>`artists.id` | string · Dimension | 1 | yes | 0 | Spotify artist ID. *(not used in any sheet)* |
| **Artists.Name**<br>`artists.name` | string · Dimension | 1 | yes | 0 | Artist name. *(not used in any sheet)* |
| **Tracks**<br>`track_number (Tracks)` | integer · Dimension | — | — | 0 | Position of the track on the album. Arrives from the **Tracks** table in the join, duplicating the same column on the left-hand side. *(not used in any sheet)* |

#### Catalog scaffold

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Name**<br>`track_name_clean_short` | string · Dimension | 83 | yes | 3 | Chart name as listed in the catalog. |

#### Identity

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Track Id**<br>`track_id` | string · Dimension | 118 | yes | 1 | Spotify track ID. |
| **track id (Track Details Short)**<br>`track_id (Track Details Short)` | string · Dimension | — | — | 1 | Spotify track ID. Arrives from the **Track Details Short** table in the join, duplicating the same column on the left-hand side. |
| **Id**<br>`id` | string · Dimension | 118 | yes | 0 | Spotify track ID. Written into the `Song` parameter by the click action. *(not used in any sheet)* |
| **Name OG**<br>`name` | string · Dimension | 101 | yes | 0 | Track title. *(not used in any sheet)* |
| **Track Name OG**<br>`track_name` | string · Dimension | 94 | yes | 0 | Track title. *(not used in any sheet)* |
| **Type**<br>`type` | string · Dimension | 2 | yes | 0 | Record classification. *(not used in any sheet)* |

#### Links

| Field | Type | Distinct | Nulls | Sheets | What it holds |
|---|---|---:|:-:|---:|---|
| **Album External Urls.Spotify**<br>`album_external_urls.spotify` | string · Dimension | 9 | yes | 0 | Public Spotify web link for the album. *(not used in any sheet)* |
| **Album Uri**<br>`album_uri` | string · Dimension | 9 | yes | 0 | Spotify album URI. *(not used in any sheet)* |
| **Album.External Urls.Spotify**<br>`album.external_urls.spotify` | string · Dimension | 9 | yes | 0 | Public Spotify web link for the album. *(not used in any sheet)* |
| **Album.Uri**<br>`album.uri` | string · Dimension | 9 | yes | 0 | Spotify album URI. *(not used in any sheet)* |
| **External Ids.Isrc**<br>`external_ids.isrc` | string · Dimension | 107 | yes | 0 | ISRC recording code: the international standard identifier for the recording. *(not used in any sheet)* |
| **External Urls.Spotify**<br>`external_urls.spotify` | string · Dimension | 118 | yes | 0 | Public Spotify web link for the track. *(not used in any sheet)* |
| **external urls.spotify (Tracks)**<br>`external_urls.spotify (Tracks)` | string · Dimension | — | — | 0 | Public Spotify web link for the track. Arrives from the **Tracks** table in the join, duplicating the same column on the left-hand side. *(not used in any sheet)* |
| **Track Uri**<br>`track_uri` | string · Dimension | 118 | yes | 0 | Spotify track URI. *(not used in any sheet)* |
| **Uri**<br>`uri` | string · Dimension | 118 | yes | 0 | Spotify URI, used to point the embedded player at a track. *(not used in any sheet)* |

### Calculated fields in use

17 of the workbook's 23 calculations are referenced by a worksheet or by dashboard logic. **Built from** names the fields each formula reads; **feeds** names the calculations that read it in turn, so a field can be traced in either direction. The formulas themselves are reproduced in full earlier in this document.

| Field | Returns | Class | Built from | Feeds | Sheets |
|---|---|---|---|---|---:|
| ***Case Blue**<br>`*Case Yellow (copy)_1550364197140619271` | boolean · Dimension | Basic | — | — | 1 |
| ***Case Green**<br>`*Case Red (copy)_1550364197140807689` | boolean · Dimension | Basic | — | — | 1 |
| ***Case Pink**<br>`*Case Yellow (copy)_1550364197140205574` | boolean · Dimension | Basic | — | — | 1 |
| ***Case Red**<br>`*Case Blue (copy)_1550364197140721672` | boolean · Dimension | Basic | — | — | 1 |
| ***Case Yellow**<br>`Calculation_1550364197139566597` | boolean · Dimension | Basic | — | — | 1 |
| ***Default Track**<br>`Calculation_1349672534058897408` | string · Dimension | Basic | — | — | 1 |
| ***SoundWave**<br>`Calculation_2557200180332707844` | integer · Measure | Basic | Attribute Names | — | 4 |
| **Album Filter**<br>`Calculation_107241982667804673` | boolean · Dimension | Basic | Album Name | — | 9 |
| **Attribute Values Normalized**<br>`Calculation_311592814361624580` | real · Measure | Basic | Attribute Names | Score % | 8 |
| **Edition**<br>`Feature 2 (copy)_293578418653511685` | string · Dimension | Basic | — | — | 1 |
| **Featured Artist**<br>`Feature (copy)_2557200180341428233` | string · Dimension | Basic | — | — | 1 |
| **Max A Score**<br>`Calculation_311592814360309761` | integer · Measure | Basic | Attribute Names | Score % | 8 |
| **Min A Score**<br>`Calculation_311592814360432642` | integer · Measure | Basic | Attribute Names | — | 1 |
| **P**<br>`Calculation_1220475520159629312` | string · Dimension | Basic | track id (Track Details Short) | — | 1 |
| **Score %**<br>`Calculation_311592814361866246` | real · Measure | Basic | Attribute Values Normalized, Max A Score | Score %*-1 | 8 |
| **Score %*-1**<br>`Score % (copy)_311592814361935879` | real · Measure | Basic | Score % | — | 4 |
| **Track Length**<br>`Calculation_2557200180336320517` | real · Measure | Basic | Track Duration Ms | — | 1 |

### Parameters

| Parameter | Type | Current value | Allowable values |
|---|---|---|---|
| **Album**<br>`Parameter 1` | string · list | `"RTJ4 (Deluxe Edition)"` | list of 9 — "RTJ4" → 4 \\| RTJ4, "RTJ4 (Deluxe Edition)" → 4 \\| RTJ4 (Deluxe Edition), "RTJ CU4TRO" → 4 \\| RTJ CU4TRO, "Run The Jewels 3" → 3 \\| Run The Jewels 3, "Spotify Sessions" → S \\| Spotify Sessions, "Run the Jewels 2" → 2 \\| Run the Jewels 2 |
| **Song**<br>`Parameter 2` | string · list | `"5yBGniyWqeALcoBpXktoby"` | any value |

## Appendix A — Calculated fields excluded from this documentation

6 calculated fields exist in the data pane but are not referenced by any worksheet or dashboard
logic:

`Attribute Names (copy)` · `1` · `E` · `0` · `Attribute Values Text` · `Track Length  (copy)`

---

*Compiled from the published `.twbx` package via the Tableau Public MCP server and direct
analysis of the workbook XML. All formulas and settings are quoted verbatim from the workbook
definition; rendered figures are read from the published view.*
