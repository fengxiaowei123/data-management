# PySpark And Cassandra MovieLens Data Pipeline Project (STQD6324 Data Management)

## This repository contains the second assignment for the STQD6324 Data Management course at the National University of Malaysia (UKM). The project implements an end-to-end distributed Big Data ETL and relational analytical pipeline using Apache Spark (PySpark) on top of a Hadoop Distributed File System (HDFS) and a NoSQL destination layer powered by Apache Cassandra.

##  Data Pipeline Workflow

Since Apache Spark operates on a lazy-evaluation paradigm, the core focus of this project is constructing a well-structured and reproducible data flow. The step-by-step pipeline execution and distributed data lifecycle are frameworked as follows:

---

### 1. Distributed Storage And Data Ingestion
* **Raw Data Hosting** `➔` Unstructured flat text files (`u.user`, `u.data`, `u.item`) from the MovieLens 100k dataset are uploaded and hosted across cluster directories within the Hadoop Distributed File System (HDFS).
* **Parallel Stream Generation** `➔` `sc.textFile()` is invoked within the Jupyter Notebook environment to read raw data blocks directly from HDFS paths, converting the raw text into in-memory distributed **Text RDDs** for parallel computation.

### 2. Data Parsing And Schema Typing
* **Text Splitting And Tokenization** `➔` To handle variant file separation rules, custom Python `lambda` split functions perform line-by-line parsing (`|` delimiter for user demographics and movie catalogs vs whitespace/tabs for transactional rating records).
* **Explicit Type-Casting** `➔` Parsed token arrays undergo field extraction and are programmatically cast into explicit primitive data types (`int` for user identity/age metrics, `float` for movie evaluation ratings) to ensure analytical data integrity.

### 3. Relational DataFrame Mapping And Catalyst Registration
* **Structured DataFrame Compilation** `➔` The cleaned and typed RDD rows are bound with an explicitly enforced Schema structure and organized into relational **PySpark DataFrames** using `spark.createDataFrame()`.
* **Temporary View Registration** `➔` DataFrames execute logical namespace binding via `createOrReplaceTempView()`, establishing optimized in-memory query targets (`users`, `ratings`, `movies`) to exploit the Spark Catalyst Optimizer for downstream execution.

---

### 4. Relational Multi-Task Analytics via Spark SQL
*The core SQL engine executes structured query operations across the registered temporary views to resolve the 5 analytical tasks required by the assignment:*

* **Movie Ratings Aggregation And Ranking (Tasks i And ii)** `➔` Connects the ratings and movie views via a relational `JOIN` operator, computing global statistics using `ROUND(AVG(r.rating), 2)` grouped by movie titles. After filtering low-frequency outliers via a Common Table Expression (CTE), results are globally sorted via `ORDER BY avg_rating DESC` to capture the **Top 10 Highest Rated Movies**.
* **Active User Core Preference Extraction (Task iii)** `➔` Isolates active power-consumers utilizing a CTE group-by predicate: `HAVING COUNT(movie_id) >= 50`. It then unpivots the horizontal 19-dimensional genre flag matrix into vertical relational features via `explode(map(...))`, and resolves the primary user interest utilizing a window ranking function: `ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY genre_rating_count DESC)`.
* **Young User Demographic Filtering (Task iv)** `➔` Evaluates a single-predicate conditional query to capture young consumer cohorts directly from the user baseline profile via `WHERE age < 20`.
* **Middle-Aged Scientist Market Segmentation (Task v)** `➔` Combines multivariable logical expressions to perform cross-dimensional filtering, isolating specific high-value user cohorts based on the target profile: `WHERE occupation = 'scientist' AND age BETWEEN 30 AND 40`.

---

### 5. Cassandra NoSQL Persistence Layer Execution
* **Cross-Platform Driver Coupling** `➔` Connects the PySpark execution kernel natively with the Cassandra distributed database layer utilizing the integrated `Spark-Cassandra Connector` driver dependencies.
* **Data Append Transactions** `➔` Processed analytical DataFrames are flushed from worker memory nodes and appended directly into 5 target physical tables under the `movielens` keyspace context using `.write.format("org.apache.spark.sql.cassandra")` with `.mode("append")`.

### 6. Reverse Data-Flow Readback And Integrity Verification
* **Reverse Database Lookup** `➔` To guarantee that no structural records were dropped or compressed during the persistence phase, a reverse data flow ingestion is triggered via `spark.read` to load tables back into Spark.
* **Sample Assertion Display** `➔` Executes `.show()` inspection tasks on the reloaded data instances (e.g., `validate_users`) to visually confirm transactional completeness and complete the end-to-end integration loop.

---

###  Pipeline Matrix And Target Schema Mapping
*The table below mappings the complete analytical data lifecycle, aligning in-memory Spark DataFrame abstractions to physical Cassandra target tables alongside their respective verification operators:*

| Operational Analytical Stage | Source Data-Engine DataFrame | Destination Cassandra Table | Target Verification Operator |
| :--- | :--- | :--- | :--- |
| **Baseline User Demographics** | `df_users` | `movielens.users` | `validate_users.show(5)` |
| **Ranked Rating Quality (Tasks i And ii)** | `top_movies_df` | `movielens.top_movies` | `validate_top_movies.orderBy("avg_rating", ascending=False).show(10)` |
| **Active User Behavior Analysis (Task iii)** | `favorite_genre_df` | `movielens.user_favorite_genre` | `validate_fav_genre.show(5)` |
| **Demographic Filter Block A (Task iv)** | `young_users_df` | `movielens.young_users` | `validate_young.show(5)` |
| **Demographic Filter Block B (Task v)** | `scientists_filtered_df` | `movielens.middle_aged_scientists` | `validate_scientists.show(5)` |
