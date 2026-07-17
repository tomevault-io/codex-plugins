# Machine Learning Copilot Instructions

## Architecture Overview

This repository contains Jupyter notebooks for machine learning projects, focusing on predictive modeling with structured data. Each notebook follows a standard ML pipeline:

1. **Data Loading & Exploration**: Load CSV datasets, initial EDA with pandas and visualization
2. **Preprocessing**: Handle missing values, encode categorical features, scale numeric data
3. **Feature Engineering**: Create new features, select important variables using statistical tests
4. **Modeling**: Train regression/classification models using scikit-learn
5. **Evaluation**: Assess model performance with metrics and visualizations

Key projects:
- **Heart Disease Prediction**: Binary classification on heart disease dataset
- **Insurance Cost Prediction**: Regression modeling for medical insurance charges

## Development Workflow

### Setup
- Use Jupyter Notebook/Lab for interactive development
- Install dependencies: `pip install numpy pandas matplotlib seaborn scikit-learn scipy`
- Run notebooks in order, executing cells sequentially

### Data Handling
- Datasets stored as CSV files in repo root
- Load with `pd.read_csv('filename.csv')`
- Output files saved as checkpoints (.ipynb-checkpoint files)

## Code Patterns

### Data Manipulation
- Use pandas DataFrames for data operations
- Categorical encoding: `df['col'].map({'value': 0, 'value2': 1})` or `pd.get_dummies()`
- Numeric scaling: `StandardScaler()` from sklearn

### Visualization
- Seaborn for statistical plots: `sns.histplot()`, `sns.boxplot()`, `sns.heatmap()`
- Matplotlib for custom plots: `plt.figure()`, `plt.subplot()`
- Pair plots and correlation matrices for EDA

### Feature Selection
- Chi-square test for categorical features: `chi2_contingency()`
- Correlation analysis: `df.corr()`, heatmap visualization
- Variance inflation factor (VIF) for multicollinearity

### Modeling
- Train/test split: `train_test_split(X, y, test_size=0.2)`
- Model fitting: `model.fit(X_train, y_train)`
- Predictions: `model.predict(X_test)`
- Evaluation: `accuracy_score()`, `mean_squared_error()`, `r2_score()`

### Preprocessing Pipeline
- Combine steps with `ColumnTransformer` and `Pipeline`
- Handle mixed data types appropriately

## Key Files
- `insurance.ipynb`: Complete ML pipeline for insurance cost prediction
- `Heart.ipynb`: Heart disease classification project
- `insurance.csv`, `heart.csv`: Source datasets

## Conventions
- Use `warnings.filterwarnings('ignore')` at notebook start
- Cell execution order matters - run sequentially
- Save intermediate results as variables for reuse
- Comment key steps with markdown cells
- Use `df.head()`, `df.info()`, `df.describe()` for initial exploration
- Standard train/test split ratio: 80/20
- Random state for reproducibility: `random_state=42`

This repository demonstrates end-to-end machine learning workflows for tabular data, emphasizing data preprocessing, feature engineering, and model evaluation.

---
> Source: [sivamtech/machine_learning](https://github.com/sivamtech/machine_learning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-17 -->
