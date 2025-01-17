
---
title: "OrigoMigo HR Attrition Statistical Analysis"
output:
  html_document:
    df_print: paged
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
library(ggplot2)
library(corrplot)
library(plotly)
```

# Introduction

At OrigoMigo Inc., recent layoffs have raised concerns among employees about potential biases in the selection process. This analysis seeks to explore the HR data to identify patterns or biases, address specific accusations, and provide actionable insights. The ultimate goal is to reassure stakeholders and help OrigoMigo refine its HR practices.

This analysis uses the fictional IBM HR Attrition dataset from Kaggle (https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset). The dataset was created by IBM data scientists to mimic real-world HR scenarios. For this analysis, **attrition** is interpreted as employees being laid off.

# Load Data

```{r}
# Load dataset
hrdata <- read.csv("HR-Employee-Attrition.csv")

# Preview data
head(hrdata)
summary(hrdata)
```

# Exploratory Data Analysis (EDA)

## Business Context
The first step in understanding potential biases is to explore the data distribution. For OrigoMigo, knowing the demographic and numerical trends in the data is crucial for identifying areas of concern.

### Attrition Distribution
Understanding attrition rates is key to assessing whether layoffs were disproportionately impacting certain groups.

```{r}
ggplot(hrdata, aes(x = Attrition, fill = Attrition)) +
  geom_bar() +
  ggtitle("Attrition Distribution") +
  xlab("Attrition") +
  ylab("Count") +
  theme_minimal()
```

### Age Distribution by Attrition
Analyzing the age distribution of employees who left versus those who stayed helps assess claims of ageism.

```{r}
ggplot(hrdata, aes(x = Age, fill = Attrition)) +
  geom_histogram(position = "dodge", bins = 20) +
  ggtitle("Age Distribution by Attrition") +
  xlab("Age") +
  ylab("Count") +
  theme_minimal()
```

# Correlation Analysis

## Business Context
At OrigoMigo, understanding how key variables relate to one another can provide insights into systemic patterns. For instance, does experience correlate with higher income, and do these relationships influence layoffs?

```{r}
# Select relevant columns
correlation_columns <- c("Age", "DailyRate", "DistanceFromHome", "Education", 
                         "HourlyRate", "MonthlyIncome", "MonthlyRate", 
                         "NumCompaniesWorked", "TotalWorkingYears", "TrainingTimesLastYear")

# Compute correlation matrix
cor_matrix <- cor(hrdata[, correlation_columns], use = "complete.obs")

# Visualize correlation matrix
corrplot(cor_matrix, method = "circle", type = "upper", title = "Correlation Matrix",
         tl.col = "black", tl.cex = 0.8)
```

# Statistical Testing

## Testing Age and Attrition

### Business Context
A specific claim from a former employee is that layoffs were biased against older employees. This analysis tests whether there is a statistically significant difference in ages between employees who were laid off and those who were not.

### Null Hypothesis
H₀: There is no significant difference in the ages of employees who were laid off versus those who were not.

### Boxplot

```{r}
ggplot(hrdata, aes(x = Attrition, y = Age, fill = Attrition)) +
  geom_boxplot() +
  ggtitle("Age Distribution by Attrition") +
  xlab("Attrition") +
  ylab("Age") +
  theme_minimal()
```

**Interpretation:** The boxplot shows that the median age of employees who were laid off is lower than that of those who stayed. Additionally, the range of ages is smaller for those who left, with a notable absence of older employees in this group compared to those who stayed.

### T-Test

```{r}
# Split data into groups
yes_group <- hrdata[(hrdata$Attrition == "Yes"), "Age"]
no_group <- hrdata[(hrdata$Attrition == "No"), "Age"]

# Perform Welch Two Sample T-Test
t_test_age <- t.test(yes_group, no_group)
print(t_test_age)
```

**Interpretation:** With a p-value of `r t_test_age$p.value`, which is less than the threshold of 0.05, we reject the null hypothesis. This indicates a statistically significant difference in ages between the two groups. The evidence suggests younger employees were more likely to be laid off, countering the claim of age bias against older employees.

## Testing Employee Number and Attrition

### Business Context
Another accusation is that newer employees (indicated by lower employee numbers) were targeted. This test examines whether tenure was a factor in layoffs.

### Null Hypothesis
H₀: There is no significant difference in the employee numbers of those who were laid off versus those who were not.

### Boxplot

```{r}
ggplot(hrdata, aes(x = Attrition, y = EmployeeNumber, fill = Attrition)) +
  geom_boxplot() +
  ggtitle("Employee Number Distribution by Attrition") +
  xlab("Attrition") +
  ylab("Employee Number") +
  theme_minimal()
```

**Interpretation:** The boxplot shows that the distribution of employee numbers for both groups is relatively similar. While there are slight differences in the range and median, no dramatic disparities are evident.

### T-Test

```{r}
# Split data
yes_group_enum <- hrdata[(hrdata$Attrition == "Yes"), "EmployeeNumber"]
no_group_enum <- hrdata[(hrdata$Attrition == "No"), "EmployeeNumber"]

# Perform T-Test
t_test_enum <- t.test(yes_group_enum, no_group_enum)
print(t_test_enum)
```

**Interpretation:** The p-value of `r t_test_enum$p.value` exceeds 0.05, meaning we fail to reject the null hypothesis. This indicates no statistically significant difference in employee numbers between the two groups, countering the claim that newer employees were disproportionately targeted.

# Predictive Modeling

## Business Context
OrigoMigo wants to better understand salary dynamics to attract and retain talent. This section uses regression models to explore predictors of MonthlyIncome.

### Simple Linear Regression: Age and Monthly Income

```{r}
age_model <- lm(MonthlyIncome ~ Age, data = hrdata)
simple_model_summary <- summary(age_model)
simple_model_summary
```

**Interpretation:** The p-value for the Age predictor is `r simple_model_summary$coefficients[2,4]`, which is below 0.05, indicating that Age significantly affects MonthlyIncome. However, the R-squared value of `r simple_model_summary$r.squared` suggests that only 25% of the variance in MonthlyIncome is explained by Age alone. This highlights the model’s limitations in predictive power.

### Multiple Linear Regression: Age and TotalWorkingYears

```{r}
age_workingyears_model <- lm(MonthlyIncome ~ TotalWorkingYears + Age, data = hrdata)
multi_model_summary <- summary(age_workingyears_model)
multi_model_summary
```

**Interpretation:** The p-values for both TotalWorkingYears (`r multi_model_summary$coefficients[2,4]`) and Age (`r multi_model_summary$coefficients[3,4]`) are below 0.05, indicating their significance in the model. The R-squared value of `r multi_model_summary$r.squared` shows that this model explains 60% of the variance in MonthlyIncome, making it substantially more predictive than the simple linear model. Notably, TotalWorkingYears emerges as a stronger predictor than Age.

### Single Predictor: TotalWorkingYears

```{r}
total_workingyears_model <- lm(MonthlyIncome ~ TotalWorkingYears, data = hrdata)
workingyears_model_summary <- summary(total_workingyears_model)
workingyears_model_summary
```

**Interpretation:** With a p-value of `r workingyears_model_summary$coefficients[2,4]` and an R-squared value of `r workingyears_model_summary$r.squared`, TotalWorkingYears alone accounts for 60% of the variance in MonthlyIncome. This confirms it as the most powerful individual predictor in this dataset.

# Conclusions and Recommendations

1. **Age and Attrition:** Younger employees were more likely to be laid off, disproving claims of ageism against older employees.
2. **Employee Tenure:** No significant difference in employee numbers was observed between groups, countering the claim that newer employees were targeted.
3. **Predictive Insights:** TotalWorkingYears is the most significant predictor of MonthlyIncome, followed by Age.

**Recommendations:**
- Communicate these findings to the HR team to address employee concerns transparently.
- Use TotalWorkingYears as a key feature in salary-related decisions.
- Investigate other factors like job role or satisfaction to refine understanding of attrition trends.

# Caveats and Assumptions

1. **Attrition Definition:** Attrition is treated as equivalent to employees being laid off. In reality, attrition may also include voluntary resignations.
2. **Fictional Dataset:** The dataset is not real and was created for educational purposes. Thus, findings should not be generalized to real-world companies.
3. **Legal Disclaimer:** This analysis is purely academic and does not account for legal frameworks surrounding layoffs, such as age discrimination laws.
4. **Simplified Relationships:** The analysis assumes linear relationships between variables for modeling purposes, which may oversimplify real-world complexities.

# Future Work

- Analyze trends over time to monitor attrition patterns.
- Use clustering to group employees based on satisfaction and performance metrics.
- Explore more advanced machine learning models for attrition prediction.
