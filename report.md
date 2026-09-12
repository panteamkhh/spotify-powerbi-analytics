# Spotify Catalog Analytics — Project Report

**Type:** Data Analytics Case Study
**Tool Stack:** Power BI Desktop, Power Query (M), DAX
**Dataset:** Spotify Tracks (114,000 rows, 21 columns)
**Author's note:** first structured Power BI project; covers the full workflow from Power Query cleaning through data modeling, a documented DAX measure layer, and dashboard delivery.

---

## Executive Summary

This project takes a raw, heavily-duplicated Spotify tracks export and turns it into a governed analytical model and an interactive dashboard. The raw file holds 114,000 rows — exactly 114 genres × 1,000 tracks. Deduplicating to one row per unique `track_id` removed 24,259 rows (21.3%), leaving **89,741 tracks** that were reshaped into a cleaned fact table, a genre-level summary, and a long-format audio-feature table — connected through a relationship model rather than flattened joins, in order to keep each table at its natural grain. (The genre summary was aggregated on a slightly different composite key, a 0.4% base mismatch documented in [`dashboard/data-validation.md`](dashboard/data-validation.md).)

The resulting dashboard answers seven core business questions around popularity, genre performance, and audio characteristics. The headline finding: popularity is long-tailed and genre-dependent, and no single audio feature explains it on its own — see [Section 5](#5-key-findings) for the full breakdown.

---

## 1. Background & Objective

The goal of this project was twofold:

1. Build real fluency with the *pre-dashboard* half of Power BI — profiling, cleaning, and modeling — since this is where most analytical judgment calls actually happen, and where a raw export usually falls apart if skipped.
2. Produce a dashboard that could stand on its own as a business deliverable: a small set of clear KPIs, seven focused visuals, and a consistent design language, rather than an unstructured collection of charts.

The project also defines an explicit **DAX measure layer** (see [Section 3.4](#34-dax-measure-layer)) so that headline metrics recalculate against the current filter context, instead of being frozen as one-time Power Query aggregations. The measure library — organized into six display folders with suggested format strings — lives in [`dax/measures.md`](dax/measures.md).

---

## 2. Data Source & Scope

| Attribute | Detail |
|---|---|
| Source rows | 114,000 |
| Source columns | 21 |
| Blank rows | 0 |
| Data type issues | None detected — all columns validated in Power Query Editor |
| Profiling tools used | Column Quality, Column Distribution, Column Profile |

**Popularity field, at a glance (raw feed):**

| Metric | Value |
|---|---:|
| Min | 0 |
| Max | 100 |
| Mean | 33.24 |
| Mean (cleaned) | 33.20 |
| Std. Dev. | 22.31 |
| Distinct values | 101 |

This distribution — a mean well below the midpoint of the 0–100 scale — is the first signal that popularity in this catalog is not evenly spread, a point returned to in the findings below.

---

## 3. Methodology

### 3.1 Duplicate Resolution

Duplicates were investigated per column first, then resolved with a single rule that was checked against the source data:

- **No fully identical rows** exist in the export — every duplicate is a partial one.
- Rows that repeat a value, per column: `artists` **82,563** · `track_name` **40,398** · `album_name` **67,421** · `track_id` **24,259**.
- The production fact table is deduplicated to **one row per unique `track_id`**, which removed 24,259 rows (21.3%) and took the dataset from 114,000 to **89,741** rows.
- A composite key (`artists` + `track_name` + `album_name`) was also evaluated: it yields **89,378** rows and is the basis of `genre_summary`. The two keys differ by 363 rows; reconciling them is tracked in [`dashboard/data-validation.md`](dashboard/data-validation.md).

### 3.2 Feature Engineering & Reshaping

Two structural changes were made to support downstream analysis:

- **`popularity_category`** — a conditional column bucketing the continuous `popularity` score into Low (0–30), Medium (31–69), and High (70–100), so the KPI-level story doesn't require every viewer to interpret a raw 0–100 number. (An earlier build had the Low/Medium labels swapped; the corrected rule and a sort-order column are documented in [`dashboard/data-validation.md`](dashboard/data-validation.md).)
- **Unpivot of audio features** — the eight audio-feature columns (`danceability`, `energy`, `speechiness`, `acousticness`, `instrumentalness`, `liveness`, `valence`, `tempo`) were converted from wide to long format, producing a `(track, Audio Feature, Score)` structure. This is what later allows all eight features to be compared in a single visual instead of eight separate ones.

A `Group By` on `track_genre` produced genre-level Track Count and Average Popularity — the basis for the `genre_summary` table.

### 3.3 Data Modeling

Three tables came out of the steps above, each kept at its own grain rather than merged into one wide table:

| Table | Grain | Role |
|---|---|---|
| `spotify_clean` | One row per track | Fact table |
| `genre_summary` | One row per genre | Aggregated dimension |
| `audio_features_long` | One row per track × feature | Analytical / long-format table |

They're connected with two single-direction, one-to-many relationships:

- `spotify_clean[track_genre]` → `genre_summary[track_genre]`
- `spotify_clean[track_id]` → `audio_features_long[track_id]`

Merge and Append were both deliberately evaluated and set aside in favor of this relationship model — flattening any of these tables together would either duplicate genre-level values across every matching track row or mix tables of different grain into one, which would misstate any re-aggregation. That evaluation is documented separately in `power-query/merge-demo.md` and `power-query/append-demo.md` so the decision is auditable rather than assumed.

### 3.4 DAX Measure Layer

On top of the model, an explicit DAX measure library is defined in [`dax/measures.md`](dax/measures.md), organized into six folders:

| Folder | Example measures |
|---|---|
| Core Catalogue KPIs | `Total Tracks`, `Total Artists`, `Total Genres`, `Tracks per Artist` |
| Popularity Analysis | `Average Popularity`, `Median Popularity`, `Popularity Std Dev`, `% High Popularity`, `Popularity vs Catalog Average` |
| Genre Analysis | `Genre Rank by Popularity`, `% of Total Tracks`, `Genre Performance Flag` |
| Audio Feature Analysis | `Average Feature Score`, `Average Energy`, `Average Valence`, `Feature Score vs Catalog` |
| Dynamic Titles & Filter-Aware UX | `Selected Genre`, `Genre Insight`, `Popularity Card Title` |
| Top-N Helpers | `Top N Tracks (slicer-driven)`, `Artist Popularity Rank` |

The design principle is **filter-context awareness**: measures such as `Popularity vs Catalog Average` and `Genre Insight` recalculate against the current slicer selection, which a static Power Query column cannot do. The library is written against the exact model column names and is paste-ready into Power BI Desktop (instructions at the top of the file). The published `.pbix` screenshots predate the measure layer and use the implicit Power Query aggregations described above; the DAX file is the explicit semantic layer defined for the model.

---

## 4. Dashboard Overview

The dashboard page was planned around seven business questions before any visual was built, specifically to avoid mid-build redesign.

**KPI row:**

| KPI | Value |
|---|---:|
| Total Tracks | 89.74K |
| Total Artists | 31.43K |
| Total Genres | 113 |
| Average Popularity | 33.20 |

**Visuals:**

| # | Business Question | Visual Type | Source Table |
|---|---|---|---|
| Q1 | How is popularity distributed? | Donut Chart | `spotify_clean` |
| Q2 | Which genres perform best? | Horizontal Bar | `spotify_clean` |
| Q3 | Who are the top artists? | Horizontal Bar | `spotify_clean` |
| Q4 | What are the top tracks? | Table | `spotify_clean` |
| Q5 | How do audio features compare? | Horizontal Bar | `audio_features_long` |
| Q6 | Do audio features predict popularity? | Scatter Plot | `spotify_clean` |
| Q7 | How does each genre's audio profile differ? | Matrix | `audio_features_long` |

Design language: dark KPI cards with a Spotify-green accent on a light canvas, consistent typography, and a KPI-first / detail-second layout. Full rationale for color choices and per-visual design decisions is in `dashboard/dashboard-notes.md` — kept separate from this report so design iteration doesn't require touching the analytical write-up.

---

## 5. Key Findings

**Popularity is long-tailed, not evenly distributed.**
A cleaned mean of 33.20 against a 0–100 scale, with a standard deviation of ~20.6, means most tracks sit well below the midpoint and very few reach the 70+ "hit" range — only about 3.5% of the catalog.

**Genre is a real differentiator, not noise.**
Genre-level averages split clearly into groups that sit persistently above vs. below the 33.2 catalog-wide average — genre is one of the first useful filters in any popularity analysis on this data.

**Audio-feature profiles cluster by genre.**
The long-format `audio_features_long` table makes it possible to compare all eight features across genres in one matrix. Genres visibly cluster into recognizable profiles (e.g., higher energy paired with lower acousticness, or the reverse).

**No single audio feature explains popularity alone.**
The scatter plot shows loose, non-linear tendencies at best for any individual feature — consistent with popularity being driven by factors outside this dataset (artist reach, playlist placement, release timing).

**Deduplication changed the numbers that matter.**
Roughly 21% of the raw rows were duplicates. Any KPI or genre count computed before cleaning would have meaningfully overstated the true catalog size.

**Suggested reading order for the dashboard:** start at the KPI row for scale, use the Genre Analysis bar chart as the primary filter for everything else, then read the Scatter Plot and the Genre × Audio Feature matrix together — one shows *whether* a relationship exists, the other shows *where* it's coming from.

---

## 6. Limitations & Assumptions

- **DAX layer is documented, not yet loaded in the published `.pbix`** — the measure library in `dax/measures.md` is defined and paste-ready, but the exported `.pbix` and its screenshots still reflect the implicit Power Query aggregations. Loading the measures is what unlocks the dynamic, filter-context-aware calculations described in [Section 3.4](#34-dax-measure-layer).
- **No time dimension** — the source data has no reliable release-date field, so no trend-over-time analysis was attempted; the Q6 scatter plot is a snapshot, not a trend.
- **Deduplication base is not fully unified** — the fact table is keyed on unique `track_id` (89,741 rows) while `genre_summary` and `audio_features_long` were aggregated on a composite key (89,378 rows). The 0.4% mismatch means genre totals do not reconcile exactly with the headline `Total Tracks`; applying one key across all three tables is the recommended fix and is tracked in [`dashboard/data-validation.md`](dashboard/data-validation.md).
- **Popularity is a platform-provided score**, not independently validated — its underlying methodology is external to this project.

---

*Supporting technical references: [`dashboard/data-validation.md`](dashboard/data-validation.md) (independent QA of the numbers and visuals), `power-query/transformations.md` (full transformation log), `power-query/merge-demo.md` and `power-query/append-demo.md` (technique evaluations), `dax/measures.md` (DAX measure library), `dashboard/dashboard-notes.md` (design rationale).*
