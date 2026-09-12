# 📦 Dataset — Spotify Tracks

Source file: [`raw/spotify.csv`](raw/spotify.csv) · **114,000 rows × 21 columns** (~20 MB).

This is the **raw export** used by the project, kept untouched so the Power Query transformations in [`../power-query/transformations.md`](../power-query/transformations.md) can be reproduced from the original state. The cleaned outputs (`spotify_clean`, `genre_summary`, `audio_features_long`) live inside the `.pbix` model, not as separate files.

> **Source:** public *Spotify Tracks Dataset* (Kaggle). Popularity is a Spotify-provided score and its methodology is external to this project.

---

## Grain

One row **per track × genre** — the same `track_id` can legitimately appear more than once under different `track_genre` values. The cleaned fact table keeps **one row per unique `track_id`** (89,741 rows); an alternative composite key (`artists` + `track_name` + `album_name`) yields 89,378 rows and is the basis of the genre summary. See [`../dashboard/data-validation.md`](../dashboard/data-validation.md) for the reconciliation.

---

## Data Dictionary

| # | Column | Type | Range / Example | Notes |
|---:|---|---|---|---|
| 1 | `Unnamed: 0` | integer | `0 … 113999` | Row index from the original export. Renamed `Column1` in the model and excluded from analysis. |
| 2 | `track_id` | text | `5SuOikwiRyPMVoIQDJUgSV` | Spotify track identifier. Not unique across rows (see grain). |
| 3 | `artists` | text | `Gen Hoshino` | Primary artist(s); can contain multiple names. |
| 4 | `album_name` | text | `Comedy` | Album title. |
| 5 | `track_name` | text | `Comedy` | Track title. |
| 6 | `popularity` | integer | `0 … 100` | Platform popularity score. Raw mean **33.24** (std 22.31); cleaned mean **33.20** (std 20.58). |
| 7 | `duration_ms` | integer | `149610` | Track length in milliseconds. |
| 8 | `explicit` | boolean | `TRUE` / `FALSE` | Explicit-content flag. |
| 9 | `danceability` | decimal | `0.0 … 1.0` | How suitable a track is for dancing. |
| 10 | `energy` | decimal | `0.0 … 1.0` | Perceptual intensity and activity. |
| 11 | `key` | integer | `0 … 11` | Musical key (pitch class). |
| 12 | `loudness` | decimal | `-60 … 0` dB | Overall loudness. |
| 13 | `mode` | integer | `0` / `1` | `0` = minor, `1` = major. |
| 14 | `speechiness` | decimal | `0.0 … 1.0` | Presence of spoken words. |
| 15 | `acousticness` | decimal | `0.0 … 1.0` | Confidence the track is acoustic. |
| 16 | `instrumentalness` | decimal | `0.0 … 1.0` | Likelihood of no vocals. |
| 17 | `liveness` | decimal | `0.0 … 1.0` | Presence of a live audience. |
| 18 | `valence` | decimal | `0.0 … 1.0` | Musical positivity (0 = sad, 1 = happy). |
| 19 | `tempo` | decimal | BPM (~60 … 200) | Estimated tempo. Different scale from the 0–1 features. |
| 20 | `time_signature` | integer | `3 … 7` | Beats per bar. |
| 21 | `track_genre` | text | `acoustic` | Genre label — the dimension used for `genre_summary`. |

---

## Known Caveats

- **No release date** — the export has no reliable date field, so no time-series / trend analysis is possible. The dashboard is a snapshot.
- **Duplicate-heavy** — 114,000 rows collapse to 89,741 unique `track_id` values (24,259 duplicates, 21.3%), removed in Phase 2.
- **`tempo` is off-scale** relative to the other audio features, so it is excluded from 0–1 feature averages (see [`../dax/measures.md`](../dax/measures.md), Section 04).
- **`Unnamed: 0`** is an artifact of the export, not analytical data.

For the cleaning steps applied to this raw file, see [`../power-query/transformations.md`](../power-query/transformations.md).
