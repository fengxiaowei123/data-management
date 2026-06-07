PySpark MLlib Iris Classification Project (STQD6324 Data Management)

This repository contains the first assignment for the STQD6324 Data Management course at the National University of Malaysia (UKM). The project implements a distributed feature engineering and classification pipeline using PySpark MLlib on the classic Iris dataset, comparing the performance of three different algorithms after hyperparameter tuning.

Workflow

Apache Spark operates on a lazy-evaluation paradigm; thus, our implementation focuses on constructing an end-to-end Machine Learning Pipeline. The data flow of this project is illustrated below:

  +-----------------------------------------------------------------------+
  |                           Data Ingestion & Schema Definition          |
  |     Online ingestion from UCI ML Repository -> Pandas -> Spark DataFrame|
  +-----------------------------------------------------------------------+
                                      |
                                      v
  +-----------------------------------------------------------------------+
  |                          Spark ML Feature Engineering                 |
  |  [Text Labels]  --->  StringIndexer   --->  [Numeric Index 0.0/1.0/2.0]|
  |  [Raw Features] --->  VectorAssembler  --->  [Dense Vector "features"] |
  +-----------------------------------------------------------------------+
                                      |
                                      v
  +-----------------------------------------------------------------------+
  |                              Dataset Partitioning                     |
  |                 75% Training Split   |   25% Testing Split            |
  +-----------------------------------------------------------------------+
                        |                             |
                        v                             v
  +--------------------------------------------+  +-----------------------+
  |              Parallel Grid Tuning & Evaluation |  |                       |
  |  - ParamGridBuilder (Hyperparameter Grid)  |  |   Out-of-Sample Test  |
  |  - CrossValidator (3-fold Cross Validation)|  |       Predictions     |
  |  - MulticlassClassificationEvaluator       |  |                       |
  +--------------------------------------------+  +-----------------------+
                        |                             |
                        +------------->---------------+
                                      |
                                      v
  +-----------------------------------------------------------------------+
  |                             Metrics & Visualization                   |
  |       Compute & print Accuracy, F1-Score, Precision, and Recall       |
  +-----------------------------------------------------------------------+


Data Preparation & Preprocessing

1. Dataset Description

The dataset is programmatically fetched online from the UCI Machine Learning Repository, containing a total of 150 instances.
The physical dimensions of sepals and petals (4 features in total) serve as the model input vector:


$$\mathbf{x} = [\text{sepal\_length}, \text{sepal\_width}, \text{petal\_length}, \text{petal\_width}]^T$$


The target variable is the flower species: Setosa (linearly separable), Versicolor, and Virginica (the latter two have partially overlapping feature distributions).

2. Preprocessing Steps

StringIndexer: Converts categorical string labels (e.g., Iris-setosa) into numerical indices [0.0, 1.0, 2.0].

VectorAssembler: Combines the 4 independent feature columns into a single dense vector column named "features" (a strict prerequisite for Spark MLlib model training).

Dataset Partitioning: The data is partitioned into a 75% training split and a 25% testing split with a fixed random seed (seed=42). The split yields exactly 112 training samples and 38 testing samples, ensuring full reproducibility across executions.

Model Tuning & Parameter Grid

To discover the optimal configuration and prevent models from overfitting on the training partition, we leveraged CrossValidator to perform 3-fold cross-validation combined with a grid search (ParamGridBuilder):

Model

Core Mechanism

Hyperparameter Grid

Optimal Configuration

Logistic Regression (LR)

Parametric model searching for the optimal linear decision hyperplane.

regParam: [0.01, 0.1, 1.0] (L2 Regularisation)elasticNetParam: [0.0, 0.5, 1.0] (L1/L2 ratio)

regParam=0.01



elasticNetParam=0.0

Decision Tree (DT)

Non-parametric model recursively splitting feature space based on Information Gain.

maxDepth: [3, 5, 7] (Tree depth limit)maxBins: [10, 20] (Continuous feature binning)

maxDepth=5



maxBins=20

Random Forest (RF)

Ensemble method training multiple decorrelated trees via Bootstrap Aggregating (Bagging).

numTrees: [10, 20, 50] (Ensemble tree count)maxDepth: [3, 5] (Maximum subtree depth)

numTrees=10



maxDepth=5

Test Partition Results & Discussion

1. Test Performance Metrics ($N=38$)

Following parameter tuning, the evaluations on the 25% independent test partition converged to the following results:

Model Classifier

Accuracy

F1-Score

Weighted Precision

Weighted Recall

Correct Classifications

Random Forest (RF)

0.9211

0.9224

0.9447

0.9211

35 / 38

Logistic Regression (LR)

0.9211

0.9224

0.9447

0.9211

35 / 38

Decision Tree (DT)

0.9211

0.9224

0.9447

0.9211

35 / 38

2. Empirical Performance Discussion

Why do all three models yield identical metrics?

You will notice that all three classifiers—despite having completely different mathematical foundations—produced the exact same Accuracy of 0.9211.
This is because our test split contains exactly 38 samples:


$$\text{Accuracy} = \frac{35}{38} \approx 0.921052 \approx \mathbf{0.9211}$$


This mathematical breakdown reveals that every single model misclassified exactly 3 samples out of the 38 test samples. Given the small size of the Iris dataset, different algorithms (whether parametric linear boundaries or non-parametric recursive partitions) easily reach an empirical performance ceiling. The 3 misclassified samples are borderline cases located where Versicolor and Virginica overlap in physical space, making them naturally hard to separate.

F1-Score and Accuracy Correspondence

Our weighted F1-Score of 0.9224 closely tracks the raw accuracy of 0.9211. In multi-class classification, if a model suffers from class bias (e.g., the majority-class bias trap), the F1-score typically drops significantly below accuracy. The tight alignment confirms that our 75%/25% split was highly balanced, avoiding class skewness or generalisation failure.

Which model should we choose?

Although the test performance is numerically identical, for deployment in a distributed production environment, Random Forest (RF) remains our primary choice:

Logistic Regression is a parametric linear model assuming a flat, rigid decision boundary. When faced with complex non-linear perturbations or noisy data in real-world environments, a purely linear model will suffer from high systematic bias (underfitting).

Decision Tree is highly sensitive to minor variations in the training dataset, suffering from high variance. This makes standalone tree models structurally unstable on different data splits.

Random Forest utilises a Bagging mechanism to ensemble 10 decorrelated subtrees. By aggregating predictions, it drastically minimises the overall model variance. Furthermore, the optimal parameters chosen via our grid search (numTrees=10, maxDepth=5) act as effective implicit pruning and regularisation. Thus, Random Forest provides far superior robustness and generalisation safety.



## 📌 Workflow Pipeline

Apache Spark operates on a lazy-evaluation paradigm; thus, our implementation focuses on constructing an end-to-end Machine Learning Pipeline. The end-to-end distributed data flow is structured as follows:

---

### 📥 1. Data Ingestion & Schema Definition
* **Data Source** `➔` UCI Machine Learning Repository (Online HTTP Request)
* **Internal Transfer** `➔` Python Pandas DataFrame
* **Target Engine** `➔` Programmatically converted into a distributed **PySpark DataFrame** with an explicitly enforced schema.

### ⚙️ 2. Distributed Feature Engineering
* **Categorical Encoding** `➔` `StringIndexer` converts text labels (e.g., Iris-setosa) into numerical class indices `[0.0, 1.0, 2.0]`.
* **Vector Assembly** `➔` `VectorAssembler` merges the 4 physical feature dimensions into a single multi-dimensional dense vector named `"features"` *(a strict prerequisite for Spark MLlib algorithm training)*.

### 🔀 3. Dataset Partitioning (Zero-Leakage Branching)
* **Configuration** `➔` Deterministic stratified random split using a fixed seed (`seed=42`).
* **Training Stream [75%]** `➔` **112 Samples** routed directly to the Parallel Hyperparameter Tuning Loop.
* **Testing Stream [25%]** `➔` **38 Samples** completely isolated and held out as a clean environment for out-of-sample validation.

---

### 🏋️‍♂️ 4. Parallel Grid Tuning & Stratified Cross-Validation
*The 75% training split is consumed by `ParamGridBuilder` and `CrossValidator` (3-Fold CV) to thoroughly search the parameter space and optimize on F1-Score:*

* **`Algorithm A` Logistic Regression (LR)** `[Grid Params]` `regParam: [0.01, 0.1, 1.0]` (L2) | `elasticNetParam: [0.0, 0.5, 1.0]` (L1/L2 Ratio)  
    `[Optimal]` `regParam=0.01` | `elasticNetParam=0.0`
* **`Algorithm B` Decision Tree (DT)** `[Grid Params]` `maxDepth: [3, 5, 7]` (Tree Depth) | `maxBins: [10, 20]` (Continuous Binning)  
    `[Optimal]` `maxDepth=5` | `maxBins=20`
* **`Algorithm C` Random Forest (RF)** `[Grid Params]` `numTrees: [10, 20, 50]` (Ensemble Size) | `maxDepth: [3, 5]` (Subtree Depth)  
    `[Optimal]` `numTrees=10` | `maxDepth=5`

---

### 🔮 5. Out-of-Sample Inference & Blind Test
* **Execution** `➔` The best model configurations locked from the Cross-Validation phase are extracted and applied to perform predictions on the untouched 25% test partition (**38 instances**).
* **Evaluation** `➔` Evaluated via `MulticlassClassificationEvaluator`.

### 📊 6. Final Performance Benchmarks & Convergence
*Due to the mathematical constraints of the borderline overlapping spaces between Versicolor and Virginica, all three classifiers reached an identical empirical performance ceiling of **35 / 38 correct classifications**:*

| Evaluation Metric | Consolidated Result | Mathematical Verification |
| :--- | :---: | :--- |
| **Accuracy** | `0.9211` | $\text{Accuracy} = \frac{35}{38} \approx 0.921052$ |
| **F1-Score** | `0.9224` | High alignment confirms full class-balance safety. |
| **Weighted Precision** | `0.9447` | Confirms exceptionally low false-positive rates. |
| **Weighted Recall** | `0.9211` | Perfectly tracks raw accuracy due to zero skewness. |

---
## 📌 Workflow

Apache Spark operates on a lazy-evaluation paradigm; thus, our implementation focuses on constructing an end-to-end Machine Learning Pipeline. The data flow of this project is illustrated below:

* **Data Ingestion & Schema Definition**
    * Online ingestion from UCI ML Repository -> Pandas -> Spark DataFrame
* **Spark ML Feature Engineering**
    * **[Text Labels]** `--->` StringIndexer `--->` **[Numeric Index 0.0/1.0/2.0]**
    * **[Raw Features]** `--->` VectorAssembler `--->` **[Dense Vector "features"]**
* **Dataset Partitioning**
    * **75% Training Split** `➔` Parallel Grid Tuning & Evaluation
        * ParamGridBuilder (Hyperparameter Grid)
        * CrossValidator (3-fold Cross Validation)
        * MulticlassClassificationEvaluator
    * **25% Testing Split** `➔` Out-of-Sample Test Predictions
* **Metrics & Visualization**
    * Compute & print Accuracy, F1-Score, Precision, and Recall
