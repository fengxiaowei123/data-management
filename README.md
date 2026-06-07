# PySpark MLlib Iris Classification Project (STQD6324 Data Management)

## This repository contains the first assignment for the STQD6324 Data Management course at the National University of Malaysia (UKM). The project implements a distributed feature engineering and classification pipeline using PySpark MLlib on the classic Iris dataset, comparing the performance of three different algorithms after hyperparameter tuning.

##  Workflow Pipeline

Apache Spark operates on a lazy-evaluation paradigm; thus, our implementation focuses on constructing an end-to-end Machine Learning Pipeline. The end-to-end distributed data flow is structured as follows:

---

###  1. Data Ingestion & Schema Definition
* **Data Source** `➔` UCI Machine Learning Repository 
* **Internal Transfer** `➔` Python Pandas DataFrame
* **Target Engine** `➔` Programmatically converted into a distributed **PySpark DataFrame** with an explicitly enforced schema.

###  2. Distributed Feature Engineering
* **Categorical Encoding** `➔` `StringIndexer` converts text labels (e.g., Iris-setosa) into numerical class indices `[0.0, 1.0, 2.0]`.
* **Vector Assembly** `➔` `VectorAssembler` merges the 4 physical feature dimensions into a single multi-dimensional dense vector named `"features"` *(a strict prerequisite for Spark MLlib algorithm training)*.

###  3. Dataset Partitioning 
* **Configuration** `➔` Deterministic stratified random split using a fixed seed (`seed=42`).
* **Training Stream [75%]** `➔` **112 Samples** routed directly to the Parallel Hyperparameter Tuning Loop.
* **Testing Stream [25%]** `➔` **38 Samples** completely isolated and held out as a clean environment for out-of-sample validation.

---

###  4. Parallel Grid Tuning & Stratified Cross-Validation
*The 75% training split is consumed by `ParamGridBuilder` and `CrossValidator` (3-Fold CV) to thoroughly search the parameter space and optimize on F1-Score:*

* **`Algorithm A` Logistic Regression (LR)** `[Grid Params]` `regParam: [0.01, 0.1, 1.0]` (L2) | `elasticNetParam: [0.0, 0.5, 1.0]` (L1/L2 Ratio)  
    `[Optimal]` `regParam=0.01` | `elasticNetParam=0.0`
* **`Algorithm B` Decision Tree (DT)** `[Grid Params]` `maxDepth: [3, 5, 7]` (Tree Depth) | `maxBins: [10, 20]` (Continuous Binning)  
    `[Optimal]` `maxDepth=5` | `maxBins=20`
* **`Algorithm C` Random Forest (RF)** `[Grid Params]` `numTrees: [10, 20, 50]` (Ensemble Size) | `maxDepth: [3, 5]` (Subtree Depth)  
    `[Optimal]` `numTrees=10` | `maxDepth=5`

---

###  5. Out-of-Sample Inference & Blind Test
* **Execution** `➔` The best model configurations locked from the Cross-Validation phase are extracted and applied to perform predictions on the untouched 25% test partition (**38 instances**).
* **Evaluation** `➔` Evaluated via `MulticlassClassificationEvaluator`.

###  6. Final Performance Benchmarks & Convergence
*Due to the mathematical constraints of the borderline overlapping spaces between Versicolor and Virginica, all three classifiers reached an identical empirical performance ceiling of **35 / 38 correct classifications**:*

| Evaluation Metric | Consolidated Result | Mathematical Verification |
| :--- | :---: | :--- |
| **Accuracy** | `0.9211` | $\text{Accuracy} = \frac{35}{38} \approx 0.921052$ |
| **F1-Score** | `0.9224` | High alignment confirms full class-balance safety. |
| **Weighted Precision** | `0.9447` | Confirms exceptionally low false-positive rates. |
| **Weighted Recall** | `0.9211` | Perfectly tracks raw accuracy due to zero skewness. |


