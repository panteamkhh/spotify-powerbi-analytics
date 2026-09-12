# 🖥️ Dashboard Design Notes

Supplementary notes for `spotify-dashboard.pbix`, expanding on the Phase 4 summary in `../report.md`.

---

## 🎯 Design Goals

- Read in under 10 seconds by someone who has never seen the dataset
- One page, no scrolling, no tab-hopping between multiple report pages
- KPI cards first, detail visuals second — executive-summary layout, not an exploratory notebook

---

##  Visual Theme

| Element | Choice |
|---|---|
| Background | Light neutral canvas with **dark (near-black) KPI cards and chart containers** for contrast |
| Accent palette | Spotify-inspired green (`#1DB954`), with 2–3 supporting neutrals |
| Typography | Single consistent font family across all visuals; larger weight for KPI numbers, lighter weight for labels |
| Chart borders | Minimal / none — separation via spacing and background contrast instead of boxes |
| Data labels | Shown only where they aid reading (e.g., top bars); suppressed where they'd clutter (e.g., scatter plot) |

**Rationale:** dark containers over a light canvas keep the four KPI numbers as the most prominent element without turning the whole page into a low-contrast dark theme, and the single green accent reads as a product dashboard rather than a default Power BI report.

---

## 🧮 DAX Measures

The dashboard's explicit measure layer is documented in [`../dax/measures.md`](../dax/measures.md). Key measures mapped to the visuals:

| Visual | Measure(s) |
|---|---|
| KPI · Total Tracks / Artists / Genres | `Total Tracks`, `Total Artists`, `Total Genres` |
| KPI · Average Popularity | `Average Popularity` |
| Q1 · Popularity Distribution | `High/Medium/Low Popularity Tracks`, `% High Popularity` |
| Q2 · Genre Analysis | `Genre Rank by Popularity`, `Genre Performance Flag` |
| Q3 · Top Artists | `Artist Popularity Rank` |
| Q5 · Audio Feature Comparison | `Average Feature Score` + per-feature measures |
| Q6 · Energy vs Popularity | `Popularity vs Catalog Average` |
| Dynamic labels | `Selected Genre`, `Genre Insight` |

> The published `.pbix` screenshots use implicit Power Query aggregations; pasting the measures above upgrades the cards and titles to filter-context-aware versions.


---

##  KPI Card Row

| KPI | Value | Source |
|---|---:|---|
| Total Tracks | 89.74K | `spotify_clean` (post-deduplication) |
| Total Artists | 31.43K | `spotify_clean` |
| Total Genres | 113 | `spotify_clean` / `genre_summary` |
| Average Popularity | 33.20 | `spotify_clean` |

Placed top-of-page, left-to-right, so scale (tracks/artists/genres) is established before any detail visual is read.

---

##  Interactivity

- **Cross-filtering:** clicking a genre in the Genre Analysis bar chart filters every other visual on the page (Popularity Distribution, Top Artists, Top Tracks, Audio Feature Comparison)
- **Tooltips:** default tooltips retained on the scatter plot (Q6) to surface track name/artist on hover, since the visual has no data labels
- **Sort control:** Top Artists and Top Tracks are sorted descending by default, with the field well left open for the viewer to re-sort by a different measure if needed

---

##  Visual-by-Visual Notes

**Q1 — Popularity Distribution (Donut):** buckets `popularity_category` (Low/Medium/High) rather than raw popularity, so the shape of the catalog is legible at a glance instead of a dense histogram.

**Q2 — Genre Analysis (Horizontal Bar):** horizontal orientation chosen because genre names are long strings — vertical bars would force rotated axis labels, which are harder to scan.

**Q3 — Top Artists (Horizontal Bar):** capped to a top-N view (not the full 31.43K artists) — the point is ranking, not a complete listing.

**Q4 — Top Tracks (Table):** kept as a table rather than a chart since the ask here is precise lookup (exact track/artist/score), not pattern recognition.

**Q5 — Audio Feature Comparison (Horizontal Bar, from `audio_features_long`):** the unpivoted long format is what makes a single bar chart able to compare all 8 audio features side-by-side, instead of needing 8 separate visuals.

**Q6 — Audio Features vs Popularity (Scatter):** deliberately left without a trend line — the underlying relationship isn't strongly linear (see `../report.md`, Phase 5), and a trend line would overstate the relationship.

**Q7 — Genre vs Audio Features (Matrix, from `audio_features_long`):** matrix chosen over a chart because it's a genre × feature grid — this is inherently tabular, and conditional-formatting-style shading (if enabled) does the pattern-spotting work that a chart would otherwise need multiple small multiples to do.

---

##  Screenshots

See `screenshots/` for exported page views. Current exports:

| File | View |
|---|---|
| `dashboard-overview.png` | Executive page — KPI row + popularity & genre |
| `dashboard-detail.png` | Detail page — top tracks, genre matrix, scatter |
| `model-view.png` | Semantic model / relationships |
| `avg-audio-feature-score.png` | Audio feature comparison |
| `energy-vs-popularity.png` | Energy vs. popularity scatter |
| `genre-vs-popularity.png` | Genre performance |
| `tracks-by-genre.png` | Catalog depth by genre |

(Add updated exports here whenever the dashboard visuals change materially.)


---

