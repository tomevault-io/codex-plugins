# Copilot Instructions for Healthcare ML EDA Project

## Project Overview
This is an R-based exploratory data analysis (EDA) project for synthetic healthcare data. The main notebook (`Projet ML.ipynb`) analyzes patient data to understand distributions and relationships for heart disease risk prediction.

## Key Technologies & Dependencies
- **Language**: R
- **Core Packages**:
  - `tidyverse`: Data manipulation and piping
  - `ggplot2`: Data visualization
  - `corrplot`, `GGally`: Correlation analysis
  - `FactoMineR`, `factoextra`: Multivariate analysis
  - `gridExtra`: Multi-plot arrangements
  - `moments`: Statistical moments
  - `vcd`: Categorical data visualization

## Data Handling Patterns
- **Data Source**: `healthcare_synthetic_data.csv` (15,000 patients, 19 variables)
- **Unique Identifier**: `Patient_ID` column - always remove for analysis (`df %>% select(-Patient_ID)`)
- **Categorical Variables**: Encoded as integers (0/1/2/3) - convert to factors:
  ```r
  df[, c("Gender", "Smoking_Status", "Alcohol_Consumption",
         "Physical_Activity_Level", "Family_History", "Heart_Disease_Risk")] <- 
    lapply(df[, c("Gender", "Smoking_Status", "Alcohol_Consumption",
                  "Physical_Activity_Level", "Family_History", "Heart_Disease_Risk")], as.factor)
  ```
- **No Missing Data**: Dataset is clean; no imputation needed

## Visualization Conventions
- **Continuous Variables**: Histogram + density overlay using `geom_histogram(aes(y=..density..)) + geom_density(alpha=.2, col="blue")`
- **Categorical Variables**: Bar plots with counts using `geom_bar(fill="salmon") + geom_text(stat="count", aes(label=after_stat(count)))`
- **Multi-plots**: Use `grid.arrange()` for arranging multiple ggplot objects
- **Theme**: `theme_minimal()` with angled x-axis labels for categorical plots

## Variable Naming
- **Continuous**: `Age`, `Height_cm`, `Weight_kg`, `BMI`, `Systolic_BP`, `Diastolic_BP`, `Cholesterol_Total`, `Cholesterol_LDL`, `Cholesterol_HDL`, `Fasting_Blood_Sugar`, `Stress_Level`, `Sleep_Hours`
- **Categorical**: `Gender`, `Smoking_Status`, `Alcohol_Consumption`, `Physical_Activity_Level`, `Family_History`, `Heart_Disease_Risk`

## Workflow Patterns
1. Load packages at notebook start
2. Read CSV with `read.csv(..., stringsAsFactors = FALSE)`
3. Remove `Patient_ID` and convert categoricals to factors
4. Generate univariate plots for all continuous variables
5. Create bar plots for all categorical variables using `lapply()` and `grid.arrange()`

## Code Style
- Use tidyverse piping (`%>%`) for data manipulation
- Store plots in variables (e.g., `g1`, `g2`) before arranging
- Clean up plot variables with `rm()` after use
- Use `cat()` and `print()` for console output during exploration</content>
<parameter name="filePath">c:\Users\Dr console\Desktop\INSA\4A\Machine Learning\Projet-Machine-learning\.github\copilot-instructions.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AhmedBousbih) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
