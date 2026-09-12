# Changelog

All notable changes to **Spotify Power BI Analytics** are documented in this file.
The format is inspired by [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and entries are grouped by date (the project is not yet version-tagged).

---

## [2026-09-12] — Data validation & chart audit

### Added
- `dashboard/data-validation.md` — independent QA of every KPI and visual, re-derived from the raw CSV with pandas, including a chart-by-chart audit and a Power BI fix checklist.

### Fixed
- Corrected the duplicate-handling figures: **24,259 rows removed**, leaving **89,741** unique tracks (was documented as 24,039 / 89,961).
- Corrected `popularity_category` to **Low 0–30 / Medium 31–69 / High 70–100** and documented the Low/Medium label swap plus the required sort column.
- Replaced the raw-only popularity stats with raw **and** cleaned values throughout the docs.

### Changed
- Removed the unsound detail-page visuals (`Sum` of averages, mis-aggregated scatter) from the README gallery.

---

## [2026-09-12] — Documentation & DAX overhaul

### Added
- `dax/measures.md` — a paste-ready DAX measure library organized into six display folders (core KPIs, popularity, genre, audio features, filter-aware UX, Top-N), including suggested format strings.
- Duration, explicit-content and popularity-threshold measures (`Average Duration (minutes)`, `Explicit Tracks`, `% Explicit Tracks`, `Tracks Above 70 Popularity`, `Tracks Below 30 Popularity`, `Hit Rate`).
- `data/README.md` — dataset data dictionary for all 21 source columns, plus grain and caveats.
- `LICENSE` (MIT).
- `.gitattributes` — declares binary assets (`.pbix`, images) and normalizes text line endings.
- README screenshot gallery, badge header, hero image, roadmap table, tech stack, and getting-started guide.

### Changed
- Rewrote `README.md` into a portfolio-focused layout for reviewers.
- Renamed `dashboard/screenshots/*` to web-friendly filenames.
- Documented the DAX layer across `report.md` (Section 3.4), `dashboard/dashboard-notes.md`, and `power-query/transformations.md`.

### Removed
- The detail dashboard view (table + scatter plot) and the duplicated dashboard image from the README.

---

## [2026-07-08] — Modeling & technique documentation

### Added
- `power-query/merge-demo.md` — Merge Queries technique reference (evaluated, not applied).
- `power-query/append-demo.md` — Append Queries technique reference (evaluated, not applied).

### Changed
- Aligned all documents to the 5-phase terminology ("Phase 1/2/3").
- Rewrote dashboard design notes.
- Sharpened the README header.

---

## [2026-07-02] — Dashboard blueprint

### Added
- Dashboard blueprint: KPI row and seven business-oriented visuals.
- Dashboard screenshots and design documentation.

---

## [2026-07-01] — Data model

### Added
- Relationships between the core tables:
  - `spotify_clean[track_genre]` → `genre_summary[track_genre]`
  - `spotify_clean[track_id]` → `audio_features_long[track_id]`
- Initial project reports and transformation logs.

---

## [2026-06-25] — Phase 1 & project setup

### Added
- Phase 1: data exploration, profiling, and duplicate inspection.
- Initial project structure and README.

### Changed
- Project renamed from `spotify-powerbi-mini-project` to `spotify-powerbi`.

---

## [2026-06-22] — Initial commit

### Added
- Initial repository and Phase 1 data-profiling notes.
