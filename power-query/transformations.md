# ⚙️ Power Query Transformations Log

This document records every transformation applied in Power Query during data preparation. It is the detailed, step-by-step log behind the summarized narrative in `../report.md` (Sections 2–3) and the phase overview in `../README.md`.

> **Terminology note:** earlier drafts of this log used "Stage 1/2/3." It's been aligned here to "Phase 1/2/3" to match the numbering used across `README.md` and `report.md`, so the same phase number means the same thing everywhere in the repo.

---

# 🟢 Phase 1 — Data Profiling

## Dataset Initialization

- Source rows: 114,000
- Columns: 21
- Empty rows: 0

---

## Data Profiling Features Enabled

- Column Quality
- Column Distribution
- Column Profile

---

## Data Type Validation

- All columns reviewed in Power Query Editor
- No data type issues detected
- Numeric and categorical fields correctly assigned

---

## Statistical Summary — Popularity

- Min: 0
- Max: 100
- Mean: 33.23
- Std Dev: 22.3
- Distinct: 101

---

## Duplicate Inspection Process

Duplicate detection was performed incrementally:

### Full Dataset Scan
- 31,438 duplicate records identified

### Secondary Check
- 29,491 duplicate matches identified

### Column-Level Inspection

- artists → 31,438 duplicates observed
- track_name → 29,491 duplicates observed
- album_name → 24,039 duplicates observed

---

## Final Deduplication Rule

Records were considered duplicates based on:

- artists
- track_name
- album_name

No single column was used alone — each was checked independently first so the combined rule was a deliberate choice, not a default.

---

# 🟡 Phase 2 — Data Cleaning & Transformation

## Remove Duplicates Step

- Input rows: 114,000
- Output rows: ~89,961
- Removed: 24,039 rows

Applied keys:
- artists
- track_name
- album_name

---

## Add Column — Conditional Logic

Column: `popularity_category`

Logic:

- 0–30 → Low
- 31–70 → Medium
- 71–100 → High

---

## Group By Operation

Field:
- track_genre

Aggregations:
- Count Rows → Track Count
- Average(popularity) → Avg Popularity

---

## Unpivot Transformation

Columns converted into attribute-value structure:

- danceability
- energy
- speechiness
- acousticness
- instrumentalness
- liveness
- valence
- tempo

---

## Output Schema After Unpivot

Fields:

- artists
- track_name
- Audio Feature (Attribute)
- Score (Value)

---

## Transformation Result

Final datasets produced:

- spotify_clean (clean structured dataset)
- genre_summary (aggregated genre-level insights)
- audio_features_long (long-format analytical dataset)

---

# 🔵 Phase 3 — Data Modeling

## Objective

A simplified star-schema style model was created to enable structured and efficient analysis in Power BI.

---

## Model Structure

The dataset was organized into three logical tables:

- `spotify_clean` → Core fact table (track-level data)
- `genre_summary` → Aggregated genre-level metrics
- `audio_features_long` → Long-format audio feature analysis

---

## Relationships

Two active relationships were created:

### 1. Genre Relationship

- `spotify_clean[track_genre]`
→ `genre_summary[track_genre]`

Purpose:
- Enables genre-level aggregation and filtering

---

### 2. Audio Feature Relationship

- `spotify_clean[track_id]`
→ `audio_features_long[track_id]`

Purpose:
- Enables track-level feature analysis across multiple audio dimensions

---

## Modeling Principles

- Star-schema inspired structure
- One-to-many relationships
- Single-direction filtering
- Separation of raw, aggregated, and analytical layers

## Merge & Append — Considered, Not Applied Here

Merge and Append were both evaluated as alternatives to the relationship model above and consciously set aside for the production tables, since flattening `spotify_clean`, `genre_summary`, and `audio_features_long` together would either duplicate genre-level values across matching rows or combine tables of different grain. That evaluation — including worked examples against this same dataset — is documented separately so it doesn't clutter this log:

- `merge-demo.md`
- `append-demo.md`

---

## Final Model Outcome

The final semantic model enables:

- Genre-based insights
- Track-level analysis
- Audio feature exploration
- Scalable dashboard development

---

# 🟣 Next Phase

After completing the data preparation and modeling process, the generated datasets were loaded into Power BI for dashboard development.

Dashboard implementation, KPI creation, and visualization design are documented separately in:

- `../report.md` (Section 4 — Dashboard Overview, Section 5 — Key Findings)
- `../README.md` (Phase 4 — Dashboard Development)
- `../dashboard/dashboard-notes.md` (design rationale)

No additional Power Query transformations were applied during the dashboard development stage. The analytical measures that sit on top of this model are DAX (not Power Query) and are documented separately in `../dax/measures.md` — keeping the transformation log focused on data preparation and the measure layer focused on filter-context calculations.

---
