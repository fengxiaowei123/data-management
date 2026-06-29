# Spotify Music Data Cleaning and Multi-Dimensional Analysis Using HDFS, Hive, and Zeppelin

## 1. Project Overview

This project establishes a data cleaning and multi-dimensional analysis workflow for a large Spotify music dataset within a distributed computing environment. Created as the final project for the **STQD6324 Data Management** course, it addresses typical data quality challenges found in raw logs, such as incorrect string formats, missing fields, and misaligned columns.

By applying regular expressions and data type casting (`CAST`) at the storage layer, the project completes a comprehensive ETL pipeline. On top of this clean dataset, it uses Hive SQL window functions, data bucketing, and OLAP drill-down queries to examine key trends in streaming genres, historical emotions, loudness effects, and track popularity. The final outputs are presented via interactive dashboards in Apache Zeppelin to support music production and platform operations.

---

## 2. Tools and Technologies Used

| Tool / Technology | Purpose |
| --- | --- |
| **HDFS** | Storing the raw Spotify dataset in a reliable, distributed file system |
| **Apache Hive** | Creating internal/external tables, performing ETL cleaning, and executing SQL analysis |
| **Apache Zeppelin** | Serving as the main notebook environment to run Hive queries and render visual dashboards |
| **Git / GitHub** | Managing code versions, hosting documentation, and sharing project results |

---

## 3. Dataset Description

The dataset contains track metadata and various acoustic features from Spotify. Key fields analyzed include:

| Column | Description |
| --- | --- |
| `track_name` | Name of the song or track |
| `artist` | Name of the artist or performer |
| `genre` | Musical genre classification (e.g., POP, ROCK, RAP) |
| `release_year` | Original release year of the track |
| `popularity` | Track popularity score on the streaming platform |
| `tempo` | Speed of the track measured in beats per minute (BPM) |
| `loudness` | Audio loudness measured in decibels (dB) |
| `emotion` | Psychological emotion label predicted by acoustic traits (e.g., joy, sadness, anger) |

---

## 4. Data Cleaning Summary (ETL)

Before running analytical queries, the raw logs were cleaned using Hive to resolve format errors and special character anomalies:

* **Structural Alignment**: Regular expressions were used to remove formatting errors and fix misaligned column data.
* **Type Conversion**: Continuous audio metrics were bucketed into clear categories using explicit casting, such as `CAST(loudness AS INT)`.
* **Handling Missing Fields**: Rows with critical missing details were excluded using conditions like `WHERE release_year >= 1970 AND emotion IS NOT NULL`.

This process ensured a clean, high-quality dataset, making the subsequent statistical analyses reliable.

---

## 5. Project Workflow

The overall big data lifecycle and project workflow are organized as follows:

```text
Raw Spotify Dataset
         ↓
Upload to HDFS storage
         ↓
Create Hive external tables mapping raw logs
         ↓
Perform ETL cleaning using regex and CAST
         ↓
Run statistical analysis and window functions in Hive
         ↓
Execute queries within Apache Zeppelin paragraphs
         ↓
Render interactive charts directly via Zeppelin's visual engine
         ↓
Organize insights and upload final files to GitHub

```

---

## 6. Core Analytical Questions

The project focuses on answering five key industry questions:

1. **General Market Overview**: How do different music genres distribute in terms of popularity and average tempo?
2. **Historical Emotion Trends**: How have different emotional categories evolved in track volume since 1970?
3. **Intra-Genre Competition**: What does the popularity distribution look like for the top 3 tracks within major genres post-2010?
4. **Acoustic Correlations**: What is the relationship between audio loudness and track popularity?
5. **Vertical Style Drill-Down**: What is the internal emotional composition within the dominant `RAP` genre?

---

## 7. Analysis Results and Visualization Summary

Since detailed visual features, metric calculations, and industry interpretations are already thoroughly documented within the individual paragraphs of the Zeppelin notebook, this README provides a high-level index of the five core analytical scenarios:

| Scenario | Analysis Focus & Key Dimensions | Chart Type | Screenshot Reference |
| --- | --- | --- | --- |
| **5.1** | General overview of popularity and average tempo (Tempo) across music genres | Bar Chart | `screenshots/01_genre_popularity_tempo.png` |
| **5.2** | Historical trends of acoustic emotion dimensions from 1970 to 2024 | Line Chart | `screenshots/02_emotion_timeline.png` |
| **5.3** | Popularity distribution of the top 3 tracks within major genres post-2010 | Bar Chart | `screenshots/03_top3_intra_genre.png` |
| **5.4** | Correlation between physical audio loudness and track popularity | Scatter Plot | `screenshots/04_loudness_popularity.png` |
| **5.5** | Deep drill-down of emotional composition within the dominant `RAP` genre | Pie Chart | `screenshots/05_rap_emotion_pie.png` |

> 💡 **Note**: For the full data-cleaning source code, dynamic charts, and comprehensive text analysis, please directly import and view the `notebooks/P165401_STQD6324_FINAL.json` notebook file.

---

## 8. Key Insights and Industry Explanations

By running distributed computing queries and data mining across these scenarios, the project highlights three core industry insights:

* **High Saturation at the Top**: Genres like `POP`, `ROCK`, and `RAP` heavily dominate total track volume and user popularity. Window function analysis shows that the popularity variance among the top 3 tracks within a major genre is typically under 1.5%. This indicates a highly saturated competitive landscape at the head-end, making it difficult for new tracks to break through.
* **The Physical Limit of the "Loudness War"**: Since the 1970s, the average mixing loudness across emotional categories has steadily increased toward the 0 dB limit. However, the scatter plot reveals a clear non-linear trend: once the average loudness crosses the critical **-4 dB** threshold, track popularity stops rising and begins to decline. Excessive volume amplification reduces the dynamic range, leading to listener fatigue.
* **The "Dual-Core" Emotional Core of Rap**: Drill-down analysis indicates that the content ecosystem of the `RAP` genre relies heavily on `anger` and `joy`. Together, these two strong emotional expressions account for over 70% of the genre's total track volume, driving its strong audience engagement and broad market adaptability.

---

## 9. Strategic Recommendations

* **Production Level (Mixing Standards)**: Music labels and mixing engineers should move away from blindly maximizing track volume. Streaming platforms are encouraged to fully implement standardized loudness normalization features that keep playback volume within a stable, balanced range of **-8 dB to -5 dB**. This approach preserves audio depth, reduces listener fatigue, and increases user session duration.
* **Operation Level (Recommendation Algorithms)**: Modern recommendation systems should look beyond traditional, coarse-grained genres and incorporate acoustic emotion vectors into their core algorithms. By aligning track suggestions with the listener's real-time mood, platforms can effectively surface high-value niche songs (such as low-volume but highly stable themes like `love` and `surprise`), helping smaller tracks gain visibility while giving the platform a unique competitive edge.

---

## 10. Repository Structure

```text
Spotify_Music_BigData_Analysis/
├── notebooks/
│   └── P165401_STQD6324_FINAL.json     # Exported Zeppelin notebook containing Hive SQL and layout configurations
├── screenshots/
│   ├── 01_genre_popularity_tempo.png   # Screenshot for Section 5.1 (Genre Analysis)
│   ├── 02_emotion_timeline.png         # Screenshot for Section 5.2 (Historical Trends)
│   ├── 03_top3_intra_genre.png         # Screenshot for Section 5.3 (Intra-Genre Rankings)
│   ├── 04_loudness_popularity.png       # Screenshot for Section 5.4 (Loudness Scatter Plot)
│   └── 05_rap_emotion_pie.png          # Screenshot for Section 5.5 (RAP Drill-Down)
└── README.md                           # Main documentation and system summary

```

---

## 11. How to Run the Project

### 11.1 Warehouse Setup and Hive Analysis

1. Ensure Hadoop services (NameNode, DataNode, ResourceManager) and Hive services are up and running in your cluster environment.
2. Upload the raw Spotify music attributes dataset into your designated HDFS data directory.
3. Open the Apache Zeppelin web interface and import the `notebooks/P165401_STQD6324_FINAL.json` file.
4. Execute the paragraphs sequentially. The initial data-cleaning paragraphs will automatically generate the clean structured table `default.spotify_cleaned`, which then serves as the data source for all subsequent analysis paragraphs.

### 11.2 Viewing the Visual Dashboards

1. After successfully running a Hive SQL query in Zeppelin, you can switch from the raw data table view to graphical views by clicking the chart icons at the top of the output block (e.g., line, bar, scatter, pie).
2. Open the chart `settings` panel. Drag your categorical fields (such as `genre` or `release_year`) into the **Keys** or **Groups** section, and drag your target aggregation metrics (such as `SUM(annual_track_volume)`) into the **Values** section to reproduce the exact dashboards shown in this project report.

---

## 12. Conclusion

This project implemented a digital music data management and multi-dimensional analysis workflow using Apache Hadoop, Apache Hive, and Apache Zeppelin.

The work began by addressing common data quality issues in the raw dataset, including incorrect string formatting, missing fields, and misaligned columns. By using regular expressions and explicit data type casting (`CAST`), basic data cleaning and normalization were completed at the storage layer. On top of this clean dataset, the project used analytical tools such as window functions, data bucketing, and OLAP drill-down queries to find correlations across different variables. Finally, these results were converted into clear, interactive dashboards using Zeppelin's built-in visualization engines.

By examining how different musical traits relate to popularity trends, this study analyzed the market presence of major genres, the historical changes in music loudness, and the emotional themes within specific music styles. These findings allowed us to propose practical recommendations for loudness standards and recommendation algorithms. Overall, this project demonstrates how structured data engineering and analysis can effectively turn raw logs into useful industry insights, providing valuable hands-on experience with big data architectures and visualization workflows.
