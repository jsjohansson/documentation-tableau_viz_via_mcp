<img width="2400" height="3000" alt="MCP_Explain" src="https://github.com/user-attachments/assets/946ad98e-8822-4421-b401-6b158f102fce" />
# Tableau Portfolio Documentation

Technical documentation for every workbook published at
[public.tableau.com/app/profile/john.johansson](https://public.tableau.com/app/profile/john.johansson) —
22 documents, 742 KB, each one a complete specification of how its workbook is built.

These are reference documents, not summaries. Each is detailed enough that a competent Tableau
developer could rebuild the workbook without opening it: every data source and join, every field,
every calculation in use with its formula, every worksheet's shelves and encodings, the dashboard
zone tree, every action, and the design system.

---

## Contents

1. [How a file maps to a viz](#1-how-a-file-maps-to-a-viz)
2. [The MCP connection](#2-the-mcp-connection)
3. [What the MCP can and cannot reach](#3-what-the-mcp-can-and-cannot-reach)
4. [The extraction pipeline](#4-the-extraction-pipeline)
5. [What a document contains](#5-what-a-document-contains)
6. [Document inventory](#6-document-inventory)
7. [Adding or refreshing a document](#7-adding-or-refreshing-a-document)
8. [Known limits](#8-known-limits)

---

## 1. How a file maps to a viz

```
documentation_viz_<PostedName>.md
```

`<PostedName>` is the **Tableau Public repository name** — the segment after `/viz/` in the live
URL, with Tableau's auto-generated numeric trailer removed:

```
https://public.tableau.com/app/profile/john.johansson/viz/SuperstoreShippingMetrics/Superstore
                                                          └───────────┬──────────┘
                                                   documentation_viz_SuperstoreShippingMetrics.md
```

It is deliberately *not* the display title and *not* the local `.twbx` filename, both of which
drift. The posted name is the one identifier that is stable and resolvable.

**The trailer rule.** When a repository name collides with an existing one, Tableau appends a long
generated number — `_17728329067570`. It carries no meaning, changes nothing about which viz is
which, and makes filenames unreadable. Strip it:

```
HRAttritionDashboard_17728329067570   →   documentation_viz_HRAttritionDashboard.md
```

To rebuild the live URL for a workbook whose posted name was trailered, take the URL recorded in
§0 of the document itself, which is always the full published form.

### Examples — the #VOTD workbooks

| Published title | Live viz | File |
|---|---|---|
| Superstore Shipping Metrics \| #VOTD | `/viz/SuperstoreShippingMetrics/Superstore` | `documentation_viz_SuperstoreShippingMetrics.md` |
| Movies Across Time & Space (Re-Viz) \| #VOTD | `/viz/MoviesAcrossTimeSpaceRe-Viz/MoviesAcrossTimeSpace` | `documentation_viz_MoviesAcrossTimeSpaceRe-Viz.md` |
| All In The Wins \| #VOTD | `/viz/AllIntheWins/RoadtotheWorldSeries` | `documentation_viz_AllIntheWins.md` |
| Viz Design Catalog \| #VOTD | `/viz/VizDesignCatalog/Vol_I` | `documentation_viz_VizDesignCatalog.md` |

None of the four needed the trailer rule — their repository names are already clean. The two that
did are `HRAttritionDashboard_17728329067570` and `NewYearsResolutionsMakeoverMonday_17054389050480`,
both shortened to their readable stem while their documents keep the full live URL in §0.

**One exception to one-file-per-viz.** `VizDesignCatalog` is a single workbook published as four
separate vizzes (Vol. I–IV). It is documented once, under Vol. I's name, with the sibling URLs
recorded inside the document.

---

## 2. The MCP connection

Everything here is sourced through the **Tableau Public MCP server** — a local Model Context
Protocol server that wraps Tableau Public's public endpoints and exposes them as callable tools.
No credentials are involved: it reads the same public surface a browser would, which is why it
works against a published profile and stops at the boundary of what the author has chosen to share.

The server exposes three groups of tools.

### Discovery — find the workbook

| Tool | Returns |
|---|---|
| `get_workbooks_list` | A user's full portfolio, paginated |
| `search_visualizations` | Search across Tableau Public |
| `get_related_workbooks`, `get_viz_of_day`, `get_featured_authors` | Discovery feeds |
| `get_user_profile`, `get_user_profile_basic`, `get_user_profile_categories` | Author profile |
| `get_favorites`, `get_followers`, `get_following` | Social graph |

### Metadata and rendering — what the viz *is*

| Tool | Returns | Used for |
|---|---|---|
| `get_workbook_details` | IDs, publish/update dates, revision, size, view count, favourites, description, attribution, `allowDataAccess` | §0 Document control, §2 Publication metadata |
| `get_workbook_contents` | Visible sheet and dashboard list with repo URLs, author links | Sheet inventory, cross-checking the XML |
| `get_workbook_image` | A rendered PNG/JPEG of any view, size- and quality-controlled | **The real KPI values**, visual hierarchy, which sheets are decorative |
| `get_workbook_thumbnail` | Small preview | Quick identification |

`get_workbook_image` is the one people skip, and it is the one that makes a document concrete.
The XML tells you how a workbook is *built*; the render tells you what it actually *shows*. It is
how you learn that a sheet named `A-Radial Center1` is a glow effect rather than a chart, and it
is where the headline figures quoted in each document come from — real rendered numbers, not
placeholders.

### Package access — what the viz is *made of*

| Tool | Returns |
|---|---|
| `download_workbook_twbx` | The packaged `.twbx`, after checking the `allowDataAccess` flag |
| `unpack_twbx` | Extraction path plus a categorised inventory (`twb`, `data`, `image`, `other`) |
| `get_twbx_workbook_structure` | Data sources, worksheets, dashboards, parameters at a glance |
| `get_twbx_calculated_fields` | Formulas, data types, dependencies |
| `get_twbx_calculation_dependencies` | Dependency graph with depth levels and an ASCII tree |
| `get_twbx_lod_expressions` | LOD expressions, parsed and explained |
| `get_twbx_data_profile` | Column names from CSV/Excel/JSON in the package |

A `.twbx` is a zip. `download_workbook_twbx` → `unpack_twbx` yields `mainTwbPath`, and the `.twb`
inside it is the complete workbook definition in XML. **That file is the source of record for
everything structural in these documents.**

---

## 3. What the MCP can and cannot reach

This is the honest boundary, and it shapes what the documents can claim.

**Reachable without the package** — publication metadata, the sheet list, and a rendered image of
any view. Enough for a design and narrative write-up.

**Reachable only with the package** — every field, calculation, parameter, filter, action, layout
coordinate and colour token. If the author has disabled data download, `allowDataAccess` is
`false`, `download_workbook_twbx` refuses, and no amount of image analysis recovers this. Field
metadata is never inferred from a picture.

**Not reachable at all** — the underlying data rows. The `.hyper` extract inside the package needs
the Tableau Hyper API to read, and `get_twbx_data_profile` says so explicitly for `.hyper` and
`.tde`. These documents describe *structure and definition*, never record-level data.

Two workbooks here — **U.S. Gas Infrastructure** and **Franky's Adventure Supply Sales** — had data
access disabled and were documented from the local repository XML instead, with MCP supplying
metadata and renders. The pipeline reads a bare `.twb` identically to a `.twbx`; it never opens the
extract. For U.S. Gas that is the difference between parsing a 28 MB XML file and handling a
438 MB package for exactly the same result.

---

## 4. The extraction pipeline

```
  Tableau Public
        │
        │  ① get_workbook_details ─────┐
        │     get_workbook_contents    ├──► metadata, sheet list, live URL
        │                              │
        │  ② get_workbook_image ───────┴──► rendered view: real KPI values,
        │                                    visual hierarchy, decorative sheets
        │
        │  ③ download_workbook_twbx
        ▼
     .twbx  ──④ unpack_twbx──►  workbook.twb  (XML — the source of record)
                                      │
                                      │  ⑤ extract_twb.py
                                      ▼
                          workbook.json  +  digest.md
                                      │
                                      │  ⑥ gen_data_dictionary.py
                                      ▼
                    documentation_viz_<PostedName>.md
```

Steps ⑤ and ⑥ come from the `tableau-workbook-docs` skill
(`~/.claude/skills/tableau-workbook-docs/`). The skill exists because the generic
`get_twbx_*` tools, useful as they are, do not reach several things a complete document needs, and
one of them times out on large workbooks — a workbook with a big list parameter can carry a 25 MB+
XML file.

**What the dedicated extractor resolves that a naive XML read does not:**

| Construct | Where it actually lives |
|---|---|
| **Dynamic Zone Visibility** | `<datagraph>`, as a GUID-linked node graph. **Absent from `<dashboards>` entirely** |
| **Parameter actions** | `<edit-parameter-action>` elements — searching for `action` misses every one |
| **Phantom actions** | `<actions>` also contains `<datasources>` and `<datasource-dependencies>`; counting children over-reports |
| **Which calculations are used** | Each worksheet's `datasource-dependencies`, plus zone-visibility bindings and transitive references |
| **Sheet-local ad-hoc calculations** | Inline in worksheets; never appear in the data pane, but are load-bearing |
| **Captions vs internal names** | Internal names carry suffixes like `(copy)_2196584473075757` and can be *shifted* relative to their caption |
| **Shelf references** | `[ds].[min:imdb rating:qk]` must be resolved to `MIN(IMDB Rating)` |
| **Categorical colour maps** | The *data source* style block, not the worksheet |
| **Top-N filters** | Encoded as nested `groupfilter function="end"` + `function="order"` |

A controlled test of this is written up in
[`README/comparison_skill_vs_no_skill_SuperstoreShippingMetrics.md`](README/comparison_skill_vs_no_skill_SuperstoreShippingMetrics.md):
same workbook, same inputs, with and without the skill. The skilled run recovered 15 of 15
structural facts in one command; the unskilled run recovered 6, and — the point of the exercise —
had no way of knowing it was incomplete.

---

## 5. What a document contains

Every file follows the same structure, so they stay comparable:

| § | Section | Contents |
|---|---|---|
| 0 | Document control | Subject, author, live URL, scope, source of record, exclusions |
| 1 | Executive summary | What it shows, the signature technique, workbook-at-a-glance counts, headline figures |
| 2 | Publication metadata | IDs, dates, revision, views, favourites, attribution *(MCP-sourced)* |
| 3 | Data architecture | Connection, data model, extract, grain, data source filters, schema, data-pane organisation |
| 4 | Field dictionary | Base fields, what each is used for, fields present but unused |
| 5 | Parameters | Domain, current value, and whether each is a live control or a geometry constant |
| 6 | Calculated fields in use | Grouped by role, formula verbatim, purpose, consuming sheets |
| 7 | Worksheet specifications | Shelves, mark layers, encodings, filters, fixed axes, formatting |
| 8 | Dashboard specification | Canvas, zone tree with the author's own container names, floating layers, z-order |
| 9 | Interactivity | Every action typed; the Dynamic Zone Visibility map; resulting view states |
| 10 | Design system | Palette with hex and role, colour/size encodings, type scale, layout craft |
| 11 | Calculation dependency map | Source fields through to consuming sheets |
| — | **Data Dictionary** | Metadata reference: every source field grouped by subject with a written explanation of what it records, plus calculated-field lineage and parameter domains |
| 13 | Rebuild / maintenance runbook | Numbered procedures for refreshing, re-tuning and extending |
| A–C | Appendices | Worksheet index, excluded calculations, package inventory |

**Only calculations actually in use get full treatment.** Unused ones are listed by name in
Appendix B so a maintainer knows they exist without the document drowning in dead code.

---

## 6. Document inventory

| Posted name | Workbook | KB | Source fields | Calcs used | Params |
|---|---|---:|---:|---:|---:|
| `U_S_GasDistributionInfrastructure` | U.S. Gas Infrastructure | 108 | 279 | 129 | 3 |
| `MoviesAcrossTimeSpaceRe-Viz` | Movies Across Time & Space (Re-Viz) | 76 | 31 | 50 | 11 |
| `AllIntheWins` | All In The Wins | 61 | 140 | 43 | 4 |
| `VizDesignCatalog` | Viz Design Catalog (Vol. I–IV) | 61 | 25 | 113 | 9 |
| `SuperstoreShippingMetrics` | Superstore Shipping Metrics | 60 | 19 | 92 | 8 |
| `TheGoodTheBadAndTheUglyWesternMovieCollection` | The Good, The Bad And The Ugly | 42 | 30 | 64 | 11 |
| `AIChatbotSupportAnalytics` | AI Chatbot Support Analytics | 32 | 8 | 56 | 4 |
| `MoviesAcrossTimeSpace` | Movies Across Time & Space (original) | 30 | 30 | 34 | 8 |
| `BarDesignGuide` | Bar Design Guide | 29 | 29 | 54 | 4 |
| `HRAttritionDashboard` | HR Attrition Dashboard | 29 | 35 | 37 | 4 |
| `USPresidentialElectionsMakeoverMonday` | US Presidential Elections | 29 | 22 | 42 | 2 |
| `FrankysAdventureSupplySales` | Franky's Adventure Supply Sales | 26 | 8 | 32 | 2 |
| `B2VB` | No Limits Scatter Plot | 24 | 30 | 19 | 12 |
| `RunTheJewels-AudioVibeWaves` | Run The Jewels — Audio Vibe Waves | 21 | 37 | 17 | 2 |
| `B2VBHBCU` | HBCU Catalog | 20 | 29 | 15 | 1 |
| `AverageEuropeanUniversityCostsMakeoverMonday` | Average European University Costs | 16 | 7 | 8 | 1 |
| `TaylorSwiftAlbumSoundWave` | Taylor Swift Album Sound Wave | 15 | 8 | 10 | 1 |
| `WorkforceProductivityReportMakeoverMonday` | Workforce Productivity Report | 14 | 7 | 7 | 1 |
| `GenerativeAISearchTrendsMakeoverMonday` | Generative AI Search Trends | 13 | 4 | 7 | 1 |
| `OdetoDuBois` | Ode to Du Bois | 13 | 2 | 8 | 1 |
| `B2VBRecordedinLiterature` | Recorded in Literature | 12 | 5 | 3 | 0 |
| `NewYearsResolutionsMakeoverMonday` | New Year's Resolutions | 11 | 7 | 5 | 0 |

**Supporting files**

| File | What it is |
|---|---|
| `usage_report_documentation_runs.md` | Cost of documenting manually vs. with the skill, across three runs |
| `README/comparison_skill_vs_no_skill_SuperstoreShippingMetrics.md` | Controlled single-workbook comparison, inputs held constant |
| `README/documentation_viz_*_README.md` | Condensed overviews for the first four workbooks documented |

---

## 7. Adding or refreshing a document

```
1. Acquire     get_workbook_details / get_workbook_contents / download_workbook_twbx / unpack_twbx
               (or point at the local .twb in My Tableau Repository)
2. Extract     python <skill>/scripts/extract_twb.py "<path>.twb" --out <scratch>/extract
3. Render      get_workbook_image  → read it; quote the real figures
4. Write       follow references/document-template.md
5. Dictionary  python <skill>/scripts/gen_data_dictionary.py <scratch>/extract/workbook.json \
                      "documentation_viz_<PostedName>.md"
```

Step 5 is safe to re-run: it replaces the existing Data Dictionary rather than appending a second
one. Name the file from the posted repository name, never the display title.

When a new data source appears, extend `scripts/field_descriptions.py` so its columns get real
explanations. Anything the knowledge base does not recognise renders as an em-dash — deliberately.
A wrong description in a metadata reference is worse than a missing one.

---

## 8. Known limits

- **`get_twbx_calculated_fields` times out on large workbooks.** The bundled extractor parses the
  XML directly instead, which is why it handles a 28 MB definition without complaint.
- **`get_workbooks_list` has returned empty**, and `get_workbook_details` intermittently returns
  nothing for names that worked minutes earlier. When the MCP is degraded, fall back to the live
  URLs recorded inside each document — they were captured from the MCP when the document was written.
- **`allowDataAccess = false` blocks the package.** Use the local repository XML if you have it;
  otherwise the document is limited to design and narrative, and should say so.
- **Extract row counts are often absent.** Many workbooks carry no `rows-inserted` attribute, so
  that line is omitted rather than guessed.
- **Table-calculation addressing is inferred.** Workbooks record only `ordering-type`; partitioning
  and addressing are derived from each sheet's level-of-detail shelf, and the documents say so.
- **Open correction:** the `joins` figure in the Data Dictionary is wrong across these documents.
  The extractor inferred joins from the relation count; in fact all 35 workbook files in the
  repository contain **zero physical join clauses** — the whole portfolio uses Tableau's logical
  layer (relationships). The extractor is fixed and now reports the data model explicitly; the
  documents have not yet been regenerated.

---

*Compiled from the Tableau Public MCP server and the workbook XML. Structure and definitions are
read from the workbook files; published figures and rendered values are read from Tableau Public.*
