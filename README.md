# 🎧 Spotify Power BI Analytics

![Power BI](https://img.shields.io/badge/Power%20BI-Analytics-F2C811?logo=powerbi)
![Status](https://img.shields.io/badge/Status-Completed-success)
![Type](https://img.shields.io/badge/Project-Data%20Analytics-blue)

---

## 📌 Project Overview

An end-to-end **Power BI analytics project** built on Spotify dataset.

The project focuses on transforming raw music data into structured analytical datasets through:

- Data profiling & quality assessment
- Data cleaning & deduplication
- Feature engineering
- Data transformation (including unpivot modeling)
- Preparing data for dashboard-level insights
- Interactive dashboard development

---

## 🧭 Project Workflow (Stages)

### 🟢 Stage 1 — Data Profiling
- Data quality check (21 columns)
- Missing values analysis
- Duplicate detection (multi-level logic)
- Statistical overview of popularity

### 🟡 Stage 2 — Data Cleaning & Transformation
- Duplicate removal using business keys
- Feature engineering (popularity categorization)
- Genre-level aggregation
- Unpivot transformation for audio features

### 🔵 Stage 3 — Data Modeling

- Designed a simplified star schema structure using three tables:
  - `spotify_clean` (fact table)
  - `genre_summary` (genre-level aggregation)
  - `audio_features_long` (feature-level analysis)

- Created relationships:
  - `spotify_clean[track_genre]` → `genre_summary[track_genre]`
  - `spotify_clean[track_id]` → `audio_features_long[track_id]`

- Applied one-to-many, single-direction filtering for clean data flow

- Model prepared for dashboard development and future DAX calculations

### 🟣 Stage 4 — Dashboard Development

- Designed an interactive Power BI dashboard
- Built 4 KPI cards for executive summary
- Created 7 business-oriented visualizations:
  - Popularity Distribution
  - Genre Analysis
  - Top Artists
  - Top Tracks
  - Audio Feature Comparison
  - Audio Features vs Popularity
  - Genre vs Audio Features
- Applied a consistent dark theme and business-focused layout

---

## 📊 Datasets Created

| Dataset | Description |
|--------|-------------|
| `spotify_clean` | Cleaned dataset after deduplication |
| `genre_summary` | Genre-level aggregated metrics |
| `audio_features_long` | Unpivoted feature-level dataset |

---

## 💡 Key Insights

- Most tracks fall in low-to-medium popularity range
- Genre popularity varies significantly
- Audio features enable deeper track-level analysis
- Relationships between audio features and popularity can be explored visually
- Interactive dashboards improve analytical exploration

---

## 🛠️ Tools Used

- Power BI Desktop
- Power Query (M Language)
- Excel / CSV Dataset
- Git & GitHub

---

## 📁 Repository Structure

```text
spotify-powerbi-analytics/
│
├── data/
│   └── spotify.csv
│
├── power-query/
│   ├── append-demo.md
│   ├── merge-demo.md
│   └── transformations.md
│
├── dashboard/
│   ├── spotify-dashboard.pbix
│   ├── screenshots/
│   └── dashboard-notes.md
│
├── report.md
└── README.md