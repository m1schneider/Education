# Assessing The Impact of Socioeconomic Factors on School Performance

## Overview

This project analyzes high school data and ACT scores to examine the relationship between socioeconomic factors and academic performance. A multiple linear regression model is used to assess the strength of these relationships and identify which socioeconomic variables are most strongly associated with academic performance.

The ACT is a standardized exam used by colleges and universities to measure academic readiness. The exam is graded on a scale of 1 to 36. According to summary statistics published by the ACT in 2023, 37% of high school graduates took the ACT, and the national average score was 19.5 (ACT, 2023). ACT scores can determine admission to a university, as well as influence the outcome of merit-based scholarships.

This study aims to determine whether ACT scores should continue to be used as a measure of student preparedness for college coursework or whether they are too heavily influenced by socioeconomic factors to be deemed useful.


---
## Data

The data for this project was from two sources. The school data was from the National Center for Education Statistics (NCES) and provided the data to determine if a school was a high school, based on the unique ID provided to each school. The other data source was from EdGap, which provided the data for each school's average ACT score, median household income, the percentage of students on a free/reduced price lunch, and the percentage of adults with a college degree. 


---

## Project Structure

```
├── data/                 # Raw and processed data
├── code/                 # Jupyter notebooks
├── reports/              # Generated reports and visualizations
├── requirements.txt      # Dependencies
└── README.md             # Project documentation
```




---

## Analysis
The data for this project came from the National Center for Education Statistics and EdGap. The initial datasets were imported and cleaned for data analysis purposes. The data types were adjusted, and only relevant features for this project were kept. A pairwise plot was used to visually identify any initial patterns between numerical variables. After the data types were changed and only relevant columns were kept, the columns were renamed using snake_case formatting, and the two datasets were joined using the school ID as the join key. The code block displaying how the two datasets were joined is reflected below:

```
df = edgap.merge(
    school_information,
    how = 'left', # use the left join
    on = 'id'     # use id as the key
)
```

The next step was to address the missing values in the data. After the data were initially imported, there were only three missing values for the ACT score. These values were dropped because they made up less than 1% of the total data, and ACT score was the response variable for this project. Once these three missing values were dropped, a total of 79 missing values remained across the DataFrame.

```
df = edgap.merge(
    school_information,
    how = 'left', # use the left join
    on = 'id'     # use id as the key
)
```

To address these missing values, an iterative imputer was used. An iterative imputer is valuable in this scenario because it uses the relationships between other variables to predict the missing values. The iterative imputer was used to impute the 79 remaining missing values across the dataset. This process of imputing the missing values is reflected in the code block below:

```
imputer = IterativeImputer() #models the variable with missing values as a function of other variables

numerical_predictors = df[predictor_variables].select_dtypes(include = 'number').columns.to_list()
print(numerical_predictors)

imputer.fit(df.loc[:, numerical_predictors]) # learn the relationships between the columns

```

The final portion of this project involved creating a multiple linear regression model. The model was created using the following variables: percent_lunch, percent_college, and median_income. A table was created to outline the coefficients for each variable in the model, and a second median_income data column was created by dividing the original column by 10,000. The intention was to make the coefficient easier to interpret in $10,000 units rather than $1 units. A residual plot was then created to assess how well the multiple linear regression model fit the data. The code block for the multiple regression model is reflected below:

```
model_10k = smf.ols(
    formula = 'average_act ~ median_income_10k + percent_lunch + percent_college',
    data = df).fit()
```


Lastly, a second multiple linear regression model was created using scaled predictor variables. The data were standardized to give each explanatory variable a mean of zero and a standard deviation of one. The resulting coefficients were exported to a table, and a statsmodels summary was printed.

---
## Results
Our model included three explanatory variables of interest: median household income, percentage of students receiving free/reduced price lunches, and the percentage of adults with a college degree. The model explained 62.6% of the variation in ACT scores, suggesting that socioeconomic factors are associated with academic performance. Two of the three explanatory variables of interest were statistically significant, with the percentage of students receiving free or reduced-price lunch showing the strongest negative association. Median household income had the weakest association and was not statistically significant (p = 0.3973). These results indicate that socioeconomic factors are associated with ACT performance. Given the significant influence ACT scores can have on a high school student's future college opportunities and academic support, these standardized testing measures should be reevaluated.


---
## Author
Michael Schneider