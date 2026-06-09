# PySpark & Cassandra MovieLens Data Pipeline Project (STQD6324 Data Management)

## This repository contains the second assignment for the STQD6324 Data Management course at the National University of Malaysia (UKM). The project implements an end-to-end distributed Big Data ETL and relational analytical pipeline using Apache Spark (PySpark) on top of a Hadoop Distributed File System (HDFS) and a NoSQL destination layer powered by Apache Cassandra.

##  Data Pipeline Workflow 

The end-to-end engineering implementation processes unstructured transactional dataset records into a NoSQL analytics layer. The engineering pipeline stages are frameworked as follows:

---

###  1. Distributed Storage & Data Ingestion
* **Storage Layer** `➔` Raw tabular text files (`u.user`, `u.data`, `u.item`) from the MovieLens 100k dataset are structured inside cluster storage directories in the Hadoop Distributed File System (HDFS).
* **Distributed Stream** `➔` `sc.textFile()` reads raw blocks directly from HDFS file paths and parallelizes them into memory instances as initial parallel streams (**Text RDDs**).

###  2. Data Parsing & Schema Typing
* **Tokenization Engine** `➔` Custom Python `lambda` map split operations parse internal file record boundary separations (`|` characters for user demographics and movie features vs `\t`/whitespace characters for rating logs).
* **Explicit Casts** `➔` Tokenized field arrays are programmatically cast into explicit primitive data types (`int` for identifier and age metrics, `float` for continuous rating evaluations).

###  3. Schema Mapping & Catalyst Registration
* **DataFrame Structuring** `➔` Parsed RDD row streams are organized into structured, strongly-typed **PySpark DataFrames** using `spark.createDataFrame()`.
* **Temporary Views** `➔` DataFrames execute logical namespace binding via `createOrReplaceTempView()`, establishing optimized in-memory query targets (`users`, `ratings`, `movies`) inside the Spark Catalyst Optimizer.

---

###  4. Relational SQL Multi-Task Analytics (Tasks i - v)
*The Core Query Engine executes relational computations across the optimization views to solve 5 separate analytic tasks:*

* **`Tasks i & ii` Ratings Aggregation & Ranking** `➔` Computes movie global metrics using `ROUND(AVG(r.rating), 2)` grouped by title fields, joined with movie dimensions, and sorted via `ORDER BY avg_rating DESC` to output the **Top 10 Highest Rated Movies** meeting threshold constraints.
* **`Task iii` High-Frequency Core Profile Analysis** `➔` Isolates active users with **$\ge 50$ ratings** via a CTE `HAVING COUNT(movie_id) >= 50` predicate. Converts horizontal genre flags into vertical relational features via `explode(map(...))`, and resolves the absolute target preference utilizing a window ranking function: `ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY genre_rating_count DESC)`.
* **`Task iv` Micro-Market Demographic Filter** `➔` Evaluates single-predicate conditional queries to capture young user cohorts using the filter profile: `WHERE age < 20`.
* **`Task v` Niche Operational Segmentation** `➔` Evaluates multi-variable logical expressions to isolate specific demographics based on the rule profile: `WHERE occupation = 'scientist' AND age BETWEEN 30 AND 40`.

---

###  5. NoSQL Persistence Layer Execution
* **Driver Pipeline** `➔` Interfaced via the Datastax Spark-Cassandra driver to translate Spark DataFrame blocks into native NoSQL database protocol transactions.
* **Append Transactions** `➔` The data flow commits memory blocks from the analytical worker nodes and cross-persists them into 5 target Cassandra schemas under the `movielens` keyspace context using `.mode("append")`.

###  6. Reverse Closed-Loop Data Integrity Verification
* **Data Flow Readback** `➔` Triggers a distinct reverse lookup ingestion via `spark.read.format("org.apache.spark.sql.cassandra")` to pull database table states back into the runtime environment.
* **Parity Inspection** `➔` Evaluates transaction completeness and schema compliance by executing `.show()` commands on reloaded data stacks, closing the integration validation loop.

---

###  Pipeline Matrix & Target Schema Mapping
*The table below defines the end-to-end data lifecycle binding the in-memory Spark computation abstractions to the physical NoSQL production tables, along with their respective verification tasks:*

| Operational Analytical Stage | Source Data-Engine DataFrame | Destination Cassandra Table | Target Verification Operator |
| :--- | :--- | :--- | :--- |
| **User Baseline Demographics** | `df_users` | `movielens.users` | `validate_users.show(5)` |
| **Ranked Rating Quality (Tasks i & ii)** | `top_movies_df` | `movielens.top_movies` | `validate_top_movies.orderBy("avg_rating", ascending=False).show(10)` |
| **Active User Behavior Analysis (Task iii)** | `favorite_genre_df` | `movielens.user_favorite_genre` | `validate_fav_genre.show(5)` |
| **Demographic Filter Block A (Task iv)** | `young_users_df` | `movielens.young_users` | `validate_young.show(5)` |
| **Demographic Filter Block B (Task v)** | `scientists_filtered_df` | `movielens.middle_aged_scientists` | `validate_scientists.show(5)` |
