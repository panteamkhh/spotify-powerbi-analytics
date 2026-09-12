# 🔍 Data Validation & Chart Audit

Independent verification of the numbers and visuals in `spotify-dashboard.pbix`, reproduced directly from [`../data/raw/spotify.csv`](../data/raw/spotify.csv) with Python/pandas.

**Verdict:** the headline KPIs and the main-dashboard visuals are **numerically correct**. Three issues were found: a **swapped Low/Medium label** on the popularity donut, an **inconsistent deduplication base** between `spotify_clean` (~89,741) and `genre_summary`/`audio_features_long` (~89,378), and the documented duplicate-handling figures did not match the raw data. The detail-page visuals also used **wrong aggregations** (`Sum` instead of `Average`) and are excluded from the README gallery.

> Reproduce with: `python -c "import pandas as pd; d=pd.read_csv('data/raw/spotify.csv'); ..."` — see the checks listed below.

---

## 1. Verified metrics

### Raw dataset

| Metric | Verified value | Notes |
|---|---:|---|
| Rows | **114,000** | exactly 114 genres × 1,000 tracks |
| Columns | 21 | incl. the export's `Unnamed: 0` index |
| Distinct artists | **31,437** | → KPI `31.43K` ✅ |
| Distinct genres | **114** | drops to 113 after dedup (`songwriter`) |
| Fully identical rows | **0** | no two rows are byte-identical |
| Avg. popularity | 33.24 | std 22.31, median 35 |
| Min / max popularity | 0 / 100 | 101 distinct values |

### Cleaned fact table (`spotify_clean`)

| Metric | Verified value | Notes |
|---|---:|---|
| Rows | **89,741** | one row per unique `track_id` → KPI `89.74K` ✅ |
| Removed | **24,259** | 21.3% of raw |
| Distinct artists | 31,437 | KPI `31.43K` ✅ |
| Distinct genres | 113 | KPI `113` ✅ |
| Avg. popularity | **33.20** | KPI `33.20` ✅ |
| Std. dev. | 20.58 | lower than raw because duplicates inflated spread |
| Median | 33 | |

### Popularity buckets (cleaned base, 89,741)

| Category | Rule | Count | Share |
|---|---|---:|---:|
| Low | popularity 0–30 | 41,654 | 46.4% |
| Medium | popularity 31–69 | 44,961 | 50.1% |
| High | popularity 70–100 | 3,126 | 3.5% |

These are exactly the three magnitudes shown on the donut (`44.96K`, `41.65K`, `3.13K`) — see issue **#1**.

### Audio features (cleaned base)

| Feature | Avg. score |
|---|---:|
| energy | 0.634 |
| danceability | 0.562 |
| valence | 0.470 |
| acousticness | 0.329 |
| liveness | 0.217 |
| instrumentalness | 0.174 |
| speechiness | 0.088 |
| tempo | 122.05 BPM |

✅ Matches both the `Average Audio Feature Score` visual and `avg-audio-feature-score.png`. `tempo` is correctly kept off the 0–1 comparison.

### Genre ranking by average popularity (cleaned base)

| Rank | Genre | Avg. popularity |
|---:|---|---:|
| 1 | k-pop | 59.36 |
| 2 | pop-film | 59.10 |
| 3 | metal | 56.42 |
| 4 | chill | 53.74 |
| 5 | latino | 51.79 |
| 6 | sad | 51.11 |
| 7 | grunge | 50.59 |

✅ Matches the main dashboard's `Average of popularity by track_genre` bar chart (k-pop → grunge, in order).

---

## 2. Chart-by-chart audit

### Main dashboard — `screenshots/dashboard-overview.png`

| # | Visual | Status | Notes |
|---|---|---|---|
| 1 | Total Tracks `89.74K` | ✅ correct | = unique `track_id`. Label reads `Count of track_id`; it is the row count of a one-row-per-track table — consider renaming to *Unique Tracks*. |
| 2 | Total Artists `31.43K` | ✅ correct | distinct count of `artists` |
| 3 | Total Genres `113` | ✅ correct | cleaned count (raw = 114) |
| 4 | Average Popularity `33.20` | ✅ correct | cleaned mean |
| 5 | Popularity Distribution (donut) | ⚠ **fix** | Counts are right, **Low and Medium labels are swapped** (see issue #1) |
| 6 | Avg. popularity by genre | ✅ correct | ranking verified |
| 7 | Count of artists by popularity | ⚠ review | For a *catalog* distribution the Y axis should count **tracks**; if "how many artists fall in each popularity band" is intended, keep but rename to make the grain explicit |
| 8 | Average Audio Feature Score | ✅ correct | tempo correctly excluded |

### Detail page — `screenshots/dashboard-detail.png` / `genre-vs-popularity.png`

| Visual | Status | Problem | Fix in Power BI |
|---|---|---|---|
| `Sum of average_popularity by track_genre` | ❌ wrong | `Sum` of a per-genre average; ranking (blues ≈ 31) contradicts the main dashboard (k-pop ≈ 59) | Set the value to **Average of `average_popularity`**, or plot **Average of `popularity`** from `spotify_clean` |
| `Count of popularity by energy` (scatter) | ❌ wrong | Y is an implicit `Count` of `popularity` — meaningless on a scatter | Put **`popularity`** (no aggregation) on Y, `energy` on X |
| Audio Feature × Genre matrix | ❌ wrong | Totals are `Sum of Score` | Set value to **Average of `Score`** |
| `Sum of track_count by track_genre` | ⚠ review | Values match the cleaned counts, but the sort appears wrong (party shown on top while ambient = 999) | Re-sort the visual by `Sum of track_count` **descending** |

Because these visuals are not analytically sound, they were **removed from the README gallery**.

---

## 3. Issues & fixes

### Issue #1 — Popularity donut: Low/Medium labels swapped

The donut shows `low = 44.96K` and `medium = 41.65K`. The data says the opposite:

- popularity **0–30** (Low) = **41,654** → the donut labels this *medium*
- popularity **31–69** (Medium) = **44,961** → the donut labels this *low*

So the two slices are mislabeled (or their legend/colors are crossed). The `71–100` boundary is also off by one: the donut's high slice (3.13K) matches popularity **≥ 70**, not ≥ 71.

**Corrected rule:**

| Category | Range |
|---|---|
| Low | 0–30 |
| Medium | 31–69 |
| High | 70–100 |

**In Power Query**, edit the `popularity_category` conditional column so `0–30 → "low"`, `31–69 → "medium"`, `70–100 → "high"`, then add a **sort column** (`low = 1, medium = 2, high = 3`) and sort the legend by it.

**In DAX** (if built as a calculated column instead):

```dax
popularity_category =
SWITCH (
    TRUE (),
    spotify_clean[popularity] <= 30, "low",
    spotify_clean[popularity] <= 69, "medium",
    "high"
)
```

```dax
popularity_category_order =
SWITCH (
    spotify_clean[popularity_category],
    "low", 1,
    "medium", 2,
    "high", 3
)
```

Then set **Sort by column** for `popularity_category` → `popularity_category_order` so the donut legend reads Low → Medium → High.

### Issue #2 — Inconsistent deduplication base

Two different cleaned populations are in play:

| Table | Base | Rows |
|---|---|---:|
| `spotify_clean` (KPIs, donut, scatter) | unique `track_id` | **89,741** |
| `genre_summary`, `audio_features_long` | composite key (`artists`+`track_name`+`album_name`) | **89,378** |

Evidence: the KPI/donut totals sum to 89,741, while `genre_summary` track counts match the composite key exactly (party 819, pop-film 815, piano 783). The two bases differ by **363 rows (0.4%)**, so genre totals will not reconcile exactly with the headline `Total Tracks`.

**Fix:** pick one key and apply it everywhere. Recommended: deduplicate `spotify_clean` on the **composite key** (`artists` + `track_name` + `album_name`), then build `genre_summary` and `audio_features_long` from that same cleaned table. This makes `Total Tracks = 89,378` and every genre count add up to it.

### Issue #3 — Documentation numbers did not match the data

The report's duplicate figures were not reproducible. Corrected below:

| Document claim | Actual |
|---|---|
| Removed **24,039** rows | 24,259 (unique `track_id`) / 24,622 (composite key) |
| Remaining **~89,961** rows | 89,741 / 89,378 |
| Duplicate scan **31,438 / 29,491** | artists 82,563 · track_name 40,398 · album_name 67,421 · track_id 24,259 repeated rows |

All affected documents have been updated to the verified values above.

---

## 4. Sign-off checklist

- [ ] Fix `popularity_category` boundaries and **Low/Medium label swap** (Issue #1)
- [ ] Sort the donut legend Low → Medium → High
- [ ] Unify the deduplication key across all three tables (Issue #2)
- [ ] Detail page: change `Sum` → `Average` for popularity/score; fix the scatter Y axis; re-sort the track-count chart
- [ ] Rename `Count of track_id` → `Unique Tracks` for clarity
- [ ] Re-export screenshots after the fixes above

---

*Method: all figures recomputed from the raw CSV with pandas 3.x. No values were taken from the dashboard without re-deriving them from source.*
