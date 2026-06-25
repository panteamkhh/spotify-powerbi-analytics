## 🟢 Phase 1 — Data Exploration Summary

### 📊 Dataset Overview
- Total rows (raw): 114,001
- Rows after cleaning empty records: 114,000
- Rows after duplicate check: 114,000 (no significant change)
- Total columns: 21
- Dataset type: Spotify tracks audio features dataset

---

### 📂 Column List
- track_id
- artists
- album_name
- track_name
- popularity
- duration_ms
- explicit
- danceability
- energy
- key
- loudness
- mode
- speechiness
- acousticness
- instrumentalness
- liveness
- valence
- tempo
- time_signature
- track_genre

---

### 🔍 Data Quality Assessment

#### Duplicate Analysis
- Small number of duplicate records detected
- Most duplicate groups had counts of 1–2
- No major duplication issue found

#### Data Types
- All columns have correct data types assigned

#### Missing Values
- No significant missing values detected after cleaning empty rows

---

### 📊 Popularity Analysis

- Min popularity: 0
- Max popularity: 100
- Average popularity: 33.23
- Standard deviation: 22.3

#### Distribution Insight
- Popularity distribution is highly skewed
- Many tracks fall in the low popularity range (0–20)
- Few tracks achieve very high popularity (80–100)

---

### 🎧 Key Observations

- Dataset is large and rich in audio features
- Strong variation exists in popularity values
- Audio features provide strong potential for correlation analysis
- Dataset is suitable for music analytics and dashboard development

---

### 🚀 Phase 1 Conclusion

Dataset is clean, well-structured, and ready for:
- Data transformation (Power Query)
- Feature engineering
- Dashboard development