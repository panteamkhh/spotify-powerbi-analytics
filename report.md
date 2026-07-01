# 🎧 Spotify Power BI Project Report

A structured data analytics project focused on profiling, cleaning, and transforming Spotify dataset using Power BI.

---

# 🟢 Stage 1 — Data Profiling

## 📌 Dataset Overview

- Initial dataset: **114,000 rows**
- Columns: **21**
- Blank rows: **0**

---

## 📊 Data Types

- All columns validated
- No type issues detected

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

### Step 1 — Full Dataset Duplicates
- **31,438 duplicate records identified**

### Step 2 — Intermediate Deduplication Check
- **29,491 duplicates identified**

### Step 3 — Field-Level Duplicate Analysis

Duplicate detection was refined using individual key columns:

- `artists` → **31,438 duplicates**
- `track_name` → **29,491 duplicates**
- `album_name` → **24,039 duplicates**

👉 Final deduplication strategy was based on the combined logic of:
- artists
- track_name
- album_name

---

# 🟡 Stage 2 — Data Cleaning & Transformation

## 🧹 Duplicate Removal

- Rows before cleaning: **114,000**
- Rows after cleaning: **~89,961**
- Cleaning based on:
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

Audio feature columns transformed:

- danceability
- energy
- speechiness
- acousticness
- instrumentalness
- liveness
- valence
- tempo

---

## 📌 Final Output Structure (Long Format)

| artists | track_name | Audio Feature | Score |
|---------|------------|---------------|------|
| A | Song 1 | danceability | 0.72 |
| A | Song 1 | energy | 0.81 |

---

# 🎯 Final Outcome

You now have 3 structured datasets:

- spotify_clean → cleaned base dataset  
- genre_summary → aggregated insights  
- audio_features_long → unpivoted analytical dataset  

---

# 🚀 Project Status

The project is now at a stage where:

- ✔ Data Cleaning completed  
- ✔ Feature Engineering completed  
- ✔ Data Modeling is ready to begin  
