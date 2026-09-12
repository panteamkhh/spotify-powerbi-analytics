# 🧮 DAX Measure Library

Production-ready DAX measures for the `spotify-dashboard.pbix` semantic model.

These measures are written against the **exact column names in the model** (`spotify_clean`, `genre_summary`, `audio_features_long`). They are **paste-ready**: open Power BI Desktop → *Modeling → New measure*, then paste each definition below. Recommended home table: a dedicated `_Measures` table (Enter Data → empty table) to keep measures grouped and out of the data tables.

> **Display folders** — the headings below (`01 · Core KPIs`, `02 · Popularity`, …) map to the *Display folder* property in Power BI, so the measure list stays navigable once pasted.

---

## 01 · Core Catalogue KPIs

```dax
Total Tracks = COUNTROWS ( spotify_clean )
```

```dax
Total Artists = DISTINCTCOUNT ( spotify_clean[artists] )
```

```dax
Total Albums = DISTINCTCOUNT ( spotify_clean[album_name] )
```

```dax
Total Genres = DISTINCTCOUNT ( spotify_clean[track_genre] )
```

```dax
Tracks per Artist =
DIVIDE ( [Total Tracks], [Total Artists] )
```

---

## 02 · Popularity Analysis

```dax
Average Popularity = AVERAGE ( spotify_clean[popularity] )
```

```dax
Median Popularity = MEDIAN ( spotify_clean[popularity] )
```

```dax
Min Popularity = MIN ( spotify_clean[popularity] )
```

```dax
Max Popularity = MAX ( spotify_clean[popularity] )
```

```dax
Popularity Std Dev = STDEV.S ( spotify_clean[popularity] )
```

```dax
High Popularity Tracks =
CALCULATE ( [Total Tracks], spotify_clean[popularity_category] = "high" )
```

```dax
Medium Popularity Tracks =
CALCULATE ( [Total Tracks], spotify_clean[popularity_category] = "medium" )
```

```dax
Low Popularity Tracks =
CALCULATE ( [Total Tracks], spotify_clean[popularity_category] = "low" )
```

```dax
% High Popularity =
DIVIDE ( [High Popularity Tracks], [Total Tracks] )
```

```dax
Average Popularity (High) =
CALCULATE ( [Average Popularity], spotify_clean[popularity_category] = "high" )
```

```dax
Popularity vs Catalog Average =
[Average Popularity] - CALCULATE ( [Average Popularity], ALL ( spotify_clean ) )
```

> **Why this matters:** `Popularity vs Catalog Average` is *filter-context aware* — it recalculates against the 33.2 catalog mean for whatever the current slicer selection is. This is exactly the kind of calculation a static Power Query column cannot express.

---

## 03 · Genre Analysis

```dax
% of Total Tracks =
DIVIDE (
    [Total Tracks],
    CALCULATE ( [Total Tracks], REMOVEFILTERS ( spotify_clean[track_genre] ) )
)
```

```dax
Genre Rank by Popularity =
RANKX (
    ALL ( spotify_clean[track_genre] ),
    [Average Popularity],
    ,
    DESC,
    DENSE
)
```

```dax
Top Genre by Popularity =
VAR TopGenres =
    TOPN ( 1, ALL ( spotify_clean[track_genre] ), [Average Popularity], DESC )
RETURN
    CONCATENATEX ( TopGenres, spotify_clean[track_genre], ", " )
```

```dax
Genre vs Catalog Average =
VAR CatalogAvg =
    CALCULATE ( [Average Popularity], ALL ( spotify_clean[track_genre] ) )
RETURN
    IF (
        ISBLANK ( [Average Popularity] ),
        BLANK (),
        [Average Popularity] - CatalogAvg
    )
```

```dax
Genre Performance Flag =
VAR CatalogAvg =
    CALCULATE ( [Average Popularity], ALL ( spotify_clean[track_genre] ) )
RETURN
    SWITCH (
        TRUE (),
        [Average Popularity] > CatalogAvg, "▲ Above catalog average",
        [Average Popularity] < CatalogAvg, "▼ Below catalog average",
        "— On par"
    )
```

---

## 04 · Audio Feature Analysis

```dax
Average Feature Score = AVERAGE ( audio_features_long[Score] )
```

```dax
Average Energy =
CALCULATE ( [Average Feature Score], audio_features_long[Audio Feature] = "energy" )
```

```dax
Average Danceability =
CALCULATE ( [Average Feature Score], audio_features_long[Audio Feature] = "danceability" )
```

```dax
Average Valence =
CALCULATE ( [Average Feature Score], audio_features_long[Audio Feature] = "valence" )
```

```dax
Average Acousticness =
CALCULATE ( [Average Feature Score], audio_features_long[Audio Feature] = "acousticness" )
```

```dax
Average Instrumentalness =
CALCULATE ( [Average Feature Score], audio_features_long[Audio Feature] = "instrumentalness" )
```

```dax
Average Liveness =
CALCULATE ( [Average Feature Score], audio_features_long[Audio Feature] = "liveness" )
```

```dax
Average Speechiness =
CALCULATE ( [Average Feature Score], audio_features_long[Audio Feature] = "speechiness" )
```

```dax
Feature Score vs Catalog =
[Average Feature Score]
    - CALCULATE ( [Average Feature Score], ALL ( audio_features_long[Audio Feature] ) )
```

> **Scale note:** `tempo` is expressed in BPM (~60–200) while the other features are 0–1. It is deliberately excluded from `Average Feature Score` comparisons so a single 122-BPM average cannot flatten the 0–1 bars. Analyze `tempo` on its own axis.

---

## 05 · Dynamic Titles & Filter-Aware UX

```dax
Selected Genre =
SELECTEDVALUE ( spotify_clean[track_genre], "All genres" )
```

```dax
Filter Status =
IF (
    ISFILTERED ( spotify_clean[track_genre] ),
    "Filtered view",
    "All data"
)
```

```dax
Popularity Card Title =
"Average Popularity · " & [Selected Genre]
```

```dax
Genre Insight =
VAR CurrentGenre = SELECTEDVALUE ( spotify_clean[track_genre] )
VAR GenreAvg = [Average Popularity]
VAR CatalogAvg =
    CALCULATE ( [Average Popularity], ALL ( spotify_clean[track_genre] ) )
RETURN
    IF (
        ISBLANK ( CurrentGenre ),
        "Select a genre to see its performance against the catalog average.",
        CurrentGenre & " averages " & FORMAT ( GenreAvg, "0.0" )
            & " popularity — "
            & IF ( GenreAvg >= CatalogAvg, "above", "below" )
            & " the catalog average of " & FORMAT ( CatalogAvg, "0.0" ) & "."
    )
```

---

## 06 · Top-N Helpers

```dax
Top N Tracks (slicer-driven) =
VAR N = SELECTEDVALUE ( 'TopN'[TopN], 10 )
RETURN
    CALCULATE ( [Total Tracks], KEEPFILTERS ( TOPN ( N, spotify_clean, [Average Popularity], DESC ) ) )
```

```dax
Artist Popularity Rank =
RANKX ( ALL ( spotify_clean[artists] ), [Average Popularity], , DESC, DENSE )
```

---

## Suggested measure formats

| Measure | Format string |
|---|---|
| `Total Tracks` / `Total Artists` / `Total Albums` / `Total Genres` | `#,##0` |
| `Average Popularity` / `Median Popularity` / `Min/Max Popularity` | `0.00` |
| `% High Popularity` | `0.0%` |
| `Average Feature Score` + feature measures | `0.00` |
| `Popularity vs Catalog Average` / `Genre vs Catalog Average` | `+0.00;-0.00;0.00` |

---

## How this layer differs from Power Query

Power Query computes values **once, at refresh time, at the table's grain** — they cannot respond to a slicer. The measures above evaluate **per visual, per filter selection**, which is what makes cross-filtering across the genre bar chart, the popularity donut, and the audio-feature bars behave correctly.

See [`../report.md`](../report.md) Section 3.4 for how this layer fits the overall model.
