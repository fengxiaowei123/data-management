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

```mermaid
graph TD
    %% 全局色彩与样式美化
    classDef default fill:#F8F9FA,stroke:#CBD5E0,stroke-width:2px,font-family:sans-serif,font-size:13px;
    classDef core fill:#EBF8FF,stroke:#3182CE,stroke-width:2px;
    classDef split fill:#E6FFFA,stroke:#319795,stroke-width:2px;
    classDef model fill:#F7FAFC,stroke:#4A5568,stroke-width:1.5px;
    classDef tune fill:#F5E6FF,stroke:#805AD5,stroke-width:2px;
    classDef out fill:#FFF5F5,stroke:#E53E3E,stroke-width:1.5px;
    classDef metric fill:#FFFAF0,stroke:#DD6B20,stroke-width:2px;

    %% 1. 数据摄入
    STEP1["<b>1. Data Ingestion & Schema Definition</b><br/>Online Ingestion (UCI ML Repository) ➔ Pandas ➔ PySpark DataFrame<br/><i>(150 instances | 4 continuous features | 3 classes)</i>"]:::core
    
    %% 2. 特征工程
    STEP2["<b>2. PySpark ML Feature Pipelines</b><br/>• <b>StringIndexer</b>: Categorical labels ➔ Numeric indices [0.0, 1.0, 2.0]<br/>• <b>VectorAssembler</b>: 4 features ➔ Dense Vector <b>'features'</b>"]:::core
    
    %% 3. 数据集切分
    STEP3{"<b>3. Dataset Partitioning</b><br/>Stratified Split (seed=42)"}:::split
    
    %% 4. 三大模型并行训练分支
    subgraph MODELS [" 4. Parallel Estimators Training Pool (75% Train Split: 112 Samples) "]
        fill:#F7FAFC
        stroke:#CBD5E0
        LR["<b>Logistic Regression (LR)</b><br/>• regParam: [0.01, 0.1, 1.0]<br/>• elasticNetParam: [0.0, 0.5, 1.0]"]:::model
        DT["<b>Decision Tree (DT)</b><br/>• maxDepth: [3, 5, 7]<br/>• maxBins: [10, 20]"]:::model
        RF["<b>Random Forest (RF)</b><br/>• numTrees: [10, 20, 50]<br/>• maxDepth: [3, 5]"]:::model
    end

    %% 5. 分布式交叉验证
    STEP5["<b>5. Distributed CrossValidator</b><br/>• 3-Fold Stratified Cross-Validation<br/>• Grid Search Grid Tuning via ParamGridBuilder<br/>• Evaluator: MulticlassClassificationEvaluator (Optimize: F1-Score)"]:::tune

    %% 6. 测试集盲测
    STEP6["<b>6. Out-of-Sample Test Inference</b><br/>Apply Best Optimized Parameters to Unseen Partition<br/><i>(25% Test Split: 38 Samples)</i>"]:::out

    %% 7. 最终完全一致的评估成果
    subgraph METRICS [" 7. Final Identical Multi-Class Evaluation Metrics (Correct: 35 / 38) "]
        M1["<b>Accuracy</b><br/>0.9211"]:::metric
        M2["<b>F1-Score</b><br/>0.9224"]:::metric
        M3["<b>Weighted Precision</b><br/>0.9447"]:::metric
        M4["<b>Weighted Recall</b><br/>0.9211"]:::metric
    end

    %% 流程连接关系拓扑
    STEP1 --> STEP2
    STEP2 --> STEP3
    
    STEP3 -- "75% Data Stream" --> LR
    STEP3 -- "75% Data Stream" --> DT
    STEP3 -- "75% Data Stream" --> RF
    
    LR --> STEP5
    DT --> STEP5
    RF --> STEP5
    
    STEP5 -- "Extract Best Models Config" --> STEP6
    STEP3 -. "25% Independent Bypass" .-> STEP6
    
    STEP6 --> M1
    STEP6 --> M2
    STEP6 --> M3
    STEP6 --> M4
