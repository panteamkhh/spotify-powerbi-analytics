# 🎧 Spotify Power BI Project Report

A structured data analytics project focused on profiling, cleaning, transformation, and modeling of Spotify dataset using Power BI.

---

# 🟢 Stage 1 — Data Profiling

## 📌 Dataset Overview

- Initial dataset: **114,000 rows**
- Columns: **21**
- Blank rows: **0**

---

## 📊 Data Types

- All columns validated in Power Query
- No data type issues detected
- Numeric and categorical fields correctly assigned

---

## 🔍 Profiling Tools Used

- Column Quality
- Column Distribution
- Column Profile

---

## 🎯 Popularity Analysis

| Metric          | Value |
|-----------------|------:|
| Min             | 0 |
| Max             | 100 |
| Average         | 33.23 |
| Std Dev         | 22.3 |
| Distinct Values | 101 |

---

## 🔁 Duplicate Analysis

Duplicate investigation was performed step-by-step using different granularities:

### Step 1 — Full Dataset Scan
- **31,438 duplicate records identified**

### Step 2 — Intermediate Validation
- **29,491 duplicates identified**

### Step 3 — Column-Level Inspection

- `artists` → **31,438 duplicates**
- `track_name` → **29,491 duplicates**
- `album_name` → **24,039 duplicates**

👉 Final deduplication strategy:
- artists
- track_name
- album_name

---

# 🟡 Stage 2 — Data Cleaning & Transformation

## 🧹 Duplicate Removal

- Rows before cleaning: **114,000**
- Rows after cleaning: **~89,961**
- Removed duplicates based on:
  - artists
  - track_name
  - album_name

---

## 🎯 Feature Engineering

### 📊 Conditional Column

Created column:

- `popularity_category`

Rules:

- 0–30 → Low
- 31–70 → Medium
- 71–100 → High

---

## 📊 Aggregation (Group By)

Grouped by:

- `track_genre`

Generated metrics:

- Track Count
- Average Popularity

---

## 🔄 Data Reshaping (Unpivot)

Audio feature columns transformed into long format:

- danceability
- energy
- speechiness
- acousticness
- instrumentalness
- liveness
- valence
- tempo

---

## 📌 Final Output Structure

| artists | track_name | Audio Feature | Score |
|---------|------------|---------------|------|
| A | Song 1 | danceability | 0.72 |
| A | Song 1 | energy | 0.81 |

---

## 🎯 Stage 2 Result

Generated datasets:

- spotify_clean → cleaned base dataset
- genre_summary → aggregated genre insights
- audio_features_long → long-format analytical dataset

---

# 🔵 Stage 3 — Data Modeling

## 🎯 Objective

A simplified star-schema style data model was designed to support efficient analysis and visualization in Power BI.

---

## 🧩 Data Model Structure

The model consists of three logical tables:

- `spotify_clean` → Core fact table (track-level data)
- `genre_summary` → Aggregated genre-level metrics
- `audio_features_long` → Unpivoted audio feature dataset

---

## 🔗 Relationships

### 1. Genre-Level Relationship

- `spotify_clean[track_genre]`
→ `genre_summary[track_genre]`

Purpose:
- Enables genre-based analysis and filtering

---

### 2. Track-Level Relationship

- `spotify_clean[track_id]`
→ `audio_features_long[track_id]`

Purpose:
- Enables detailed audio feature analysis per track

---

## 🧠 Modeling Principles

- Star-schema inspired structure
- Separation of raw, aggregated, and analytical layers
- One-to-many relationships
- Single-direction filtering
- Clean semantic model design

---

## 📊 Final Data Model Outcome

The model enables:

- Genre-level insights
- Track-level analysis
- Audio feature exploration
- Scalable dashboard development in Power BI

---