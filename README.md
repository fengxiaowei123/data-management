Distributed Classification on the Iris Dataset using PySpark MLlib

This repository contains the complete implementation of a distributed machine learning pipeline to classify botanical species of the Iris dataset using PySpark MLlib. The project is submitted for the STQD6324 Data Management course (Assignment 1, Semester 2 2025/2026).

📌 Project Overview

The primary objective of this project is to develop and compare three distributed machine learning classifiers utilizing Apache Spark's MLlib. The pipeline encapsulates the entire machine learning lifecycle, including:

Robust programmatic ingestion of public web data.

Structured distributed feature engineering and data transformation.

Systematized hyperparameter tuning via grid search and 3-fold cross-validation.

Comprehensive performance evaluation contrasting parametric and non-parametric algorithms.

📊 Dataset & Preprocessing Methodology

1. Dataset Description

The model is trained on the classic Iris Flower Dataset obtained from the UCI Machine Learning Repository. It comprises 150 samples across 3 classes (50 samples each):

Iris setosa

Iris versicolor

Iris virginica

Each sample possesses 4 physical features: sepal_length, sepal_width, petal_length, and petal_width.

2. Preprocessing & Distributed Pipeline

To feed the data into Spark's distributed estimators, the following workflow is engineered:

Label Encoding (StringIndexer): Transforms the textual botanical category labels into index floats (0.0, 1.0, 2.0).

Feature Assembly (VectorAssembler): Consolidates the four numeric feature columns into a single Spark dense vector column named "features".

Dataset Partitioning: The assembled dataset is split into 75% training (for model training and cross-validation) and 25% testing (for out-of-sample evaluation) using a fixed random seed of 42 to guarantee complete reproducibility.

⚙️ Model Tuning & Optimization

Three machine learning estimators are constructed, tuned, and evaluated:

Logistic Regression (LR): Tuned over regularization parameter regParam [0.01, 0.1, 1.0] and ElasticNet mixing parameter elasticNetParam [0.0, 0.5, 1.0].

Decision Tree (DT): Tuned over tree depth maxDepth [3, 5, 7] and continuous feature binning resolution maxBins [10, 20].

Random Forest (RF): Tuned over tree counts numTrees [10, 20, 50] and tree depth maxDepth [3, 5].

All classifiers are optimized using a 3-fold Stratified Cross-Validation (CrossValidator) scheme.

📈 Summary of Results & Key Findings

1. Empirical Metrics On Testing Partition

The optimized configurations converged to identical outstanding classification scores on the independent test dataset:

Model Classifier

Accuracy

F1-Score

Weighted Precision

Weighted Recall

Random Forest

0.9211

0.9224

0.9447

0.9211

Logistic Regression

0.9211

0.9224

0.9447

0.9211

Decision Tree

0.9211

0.9224

0.9447

0.9211

2. Analytical Discussion & Insights

The Majority-Class Trap Avoided: F1-Scores (0.9224) closely tracking and slightly exceeding the accuracy confirm that the classifiers did not fall into any class imbalances, proving the high quality of the 75%/25% data split.

Why Random Forest is Recommended: Despite the numerical tie across all three models due to the high linear separability of the small dataset, Random Forest is chosen as the optimal choice. It provides exceptional variance control through Bootstrap Aggregating (Bagging) and implicit regularization from its optimal hyperparameters (numTrees=10, maxDepth=5), preventing overfitting to localized noise.

🚀 How to Reproduce the Analysis

Follow these steps to run the pipeline locally or on a distributed Spark cluster.

Prerequisites

Make sure your system has Java Development Kit (JDK 8 or 11) installed and configured in your environment path.

1. Clone the Repository

git clone [https://github.com/your-username/your-repo-name.git](https://github.com/your-username/your-repo-name.git)
cd your-repo-name


2. Set Up Virtual Environment (Optional but Recommended)

python -m venv venv
source venv/bin/activate  # On Windows use: venv\Scripts\activate


3. Install Dependencies

Install PySpark and other visualization utilities listed in the runtime requirements:

pip install pyspark pandas matplotlib seaborn


4. Execute the Pipeline

You can run the notebook using Jupyter:

jupyter notebook


Open Data Management Assignment 1.ipynb and execute all cells sequentially (Cell -> Run All).

📂 Repository Structure

├── README.md                           <- This project summary and deployment documentation
└── Data Management Assignment 1.ipynb <- Complete PySpark implementation and performance plots
