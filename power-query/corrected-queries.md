# ✅ Corrected Pipeline — Paste-Ready M Queries

This is the **corrected** Power Query pipeline, aligned with the verified findings in [`../dashboard/data-validation.md`](../dashboard/data-validation.md). It fixes the two data-level issues that cannot be patched from outside Power BI:

1. a **single deduplication key** across all three tables (composite `artists` + `track_name` + `album_name` → 89,378 rows), so `Total Tracks` reconciles with the genre totals;
2. the correct **`popularity_category`** boundaries (Low 0–30 / Medium 31–69 / High 70–100) plus a sort column.

Create three blank queries in Power BI Desktop (*Home → Transform Data → New Source → Blank Query → Advanced Editor*) and paste these one at a time. The order matters: `genre_summary` and `audio_features_long` reference `spotify_clean`.

---

## Query 1 — `spotify_clean` (fact table)

```m
let
    Source = Csv.Document(
        File.Contents("data\raw\spotify.csv"),
        [Delimiter = ",", Columns = 21, Encoding = 65001, QuoteStyle = QuoteStyle.Csv]
    ),
    Promoted = Table.PromoteHeaders(Source, [PromoteAllScalars = true]),
    RemovedIndex = Table.RemoveColumns(Promoted, {"Unnamed: 0"}),

    // One row per unique business key -> 89,378 rows
    Deduped = Table.Distinct(RemovedIndex, {"artists", "track_name", "album_name"}),

    // Corrected category: Low 0-30, Medium 31-69, High 70-100
    AddedCategory = Table.AddColumn(
        Deduped, "popularity_category",
        each if [popularity] <= 30 then "low"
             else if [popularity] <= 69 then "medium"
             else "high",
        type text
    ),
    // Sort column for the donut legend (Low -> Medium -> High)
    AddedOrder = Table.AddColumn(
        AddedCategory, "popularity_category_order",
        each if [popularity] <= 30 then 1
             else if [popularity] <= 69 then 2
             else 3,
        Int64.Type
    ),
    Typed = Table.TransformColumnTypes(AddedOrder, {
        {"track_id", type text},
        {"artists", type text},
        {"album_name", type text},
        {"track_name", type text},
        {"popularity", Int64.Type},
        {"duration_ms", Int64.Type},
        {"explicit", type logical},
        {"danceability", type number},
        {"energy", type number},
        {"key", Int64.Type},
        {"loudness", type number},
        {"mode", Int64.Type},
        {"speechiness", type number},
        {"acousticness", type number},
        {"instrumentalness", type number},
        {"liveness", type number},
        {"valence", type number},
        {"tempo", type number},
        {"time_signature", Int64.Type},
        {"track_genre", type text}
    })
in
    Typed
```

After loading, set **Sort by column** on `popularity_category` → `popularity_category_order`.

---

## Query 2 — `genre_summary` (one row per genre)

```m
let
    Source = spotify_clean,
    Grouped = Table.Group(
        Source,
        {"track_genre"},
        {
            {"track_count", each Table.RowCount(_), Int64.Type},
            {"average_popularity", each List.Average([popularity]), type number}
        }
    )
in
    Grouped
```

`track_count` now sums to **89,378**, matching `spotify_clean`, and `average_popularity` is the true per-genre average (plot it with **Average**, never **Sum**).

---

## Query 3 — `audio_features_long` (long format)

```m
let
    Source = spotify_clean,
    Keep = Table.SelectColumns(
        Source,
        {"track_id", "track_name", "artists", "track_genre", "popularity_category",
         "danceability", "energy", "speechiness", "acousticness",
         "instrumentalness", "liveness", "valence", "tempo"}
    ),
    Unpivoted = Table.UnpivotOtherColumns(
        Keep,
        {"track_id", "track_name", "artists", "track_genre", "popularity_category"},
        "Audio Feature",
        "Score"
    )
in
    Unpivoted
```

---

## Relationships after the rebuild

- `spotify_clean[track_genre]` → `genre_summary[track_genre]` (one-to-many)
- `spotify_clean[track_id]` → `audio_features_long[track_id]` (one-to-many)

## Visual fixes that remain manual

These are field settings inside the report canvas, not the data model:

| Visual | Change |
|---|---|
| KPI card | Rename `Count of track_id` → `Unique Tracks` |
| Popularity donut | Sort legend by `popularity_category_order` |
| Detail scatter | Y axis → `popularity` (raw value), not `Count of popularity` |
| Audio Feature × Genre matrix | Value → **Average of `Score`** |
| Track-count chart | Sort descending by `track_count` |

Full context and the verification numbers: [`../dashboard/data-validation.md`](../dashboard/data-validation.md).
