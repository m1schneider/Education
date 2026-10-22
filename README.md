# Rainfall Comparison between Seattle and Philadelphia

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
The data for this project was gathered by NOAA, however, it had to be appropriately cleaned and organized for analysis. The data that we used was the precipitation data gathered by the two weather stations, and the data was stripped of non-relevant data and reorganized into a long format. This data was then exported an saved under the 'data' folder, and named 'clean_seattle_PHL_weather.csv'.


This cleaned data was then visualized and aggregated to find underlying patterns regarding the precipitation data in terms of rainfall per year and the proportion of days with rainfall. On days of the year where rainfall data was missing, the mean rainfall was added in its place.


The following is a section of the groupby statement that was able to return data on which cities where receiving more rainfall and at what times of the year. The first statement returns the average daily rainfall for both Seattle and Philadelphia, and the seconf statement return the average daily rainfall by month. The monthly breakdown allows us to see the annual trends that both cities experience and how they vary from one another.

```python
df[['city','precipitation']].groupby('city').mean()
```

```python
df[['month', 'precipitation', 'city']].groupby(['month','city']).mean()
```

Visualizing this data also allows us to see the annual trends on a month to month basis to get a better understanding of how both cities varied from on another. The code sections below visualized this data in the form of barplots. The first sections returns a barplot of the mean rainfall per month and the second section returns the proportion of days with rain by month. 

```python
plt.figure(figsize=(20,4))

sns.barplot(data=df, x='month', y='precipitation', hue='city')

plt.xlabel('Month', fontsize=15)
plt.ylabel('Precipitation (inches)', fontsize=15)
plt.tick_params(labelsize=15)
plt.xticks(ticks=range(12), labels=month_names)

plt.show()
```

```python
plt.figure(figsize=(20,4))

sns.barplot(data=df, x='month', y='any_precipitation', hue='city')

plt.xlabel(None)
plt.ylabel('Proportion days with precipitation', fontsize=13)
plt.xticks(ticks=range(12), labels=month_names)
plt.tick_params(labelsize=15)

plt.show()
```

An independent-samples t-test was conducted for each month to determine which months showed a statistically significant difference in mean daily rainfall at the 0.05 significance level. It was determined that the difference in mean daily rainfall was statistically significant in 8 out of 12 months, the code used to perform this analysis is provided below.

```python
from scipy import stats

significance_level = 0.05
significantly_different = np.zeros(12)

results = []

#perform a t-test for each month
for month in range (1,13):
    #Get precipitation data for Seattle and St.Louis for the current month
    sea_data = df.loc[(df['city'] == 'SEA') & (df['month'] == month), 'precipitation'   ]
    phl_data = df.loc[(df['city'] == 'PHL') & (df['month'] == month), 'precipitation'   ]

    #Equal variance = false, indicated we are using Welch's T-Test
    t_statistic, p_value = stats.ttest_ind(sea_data, phl_data, equal_var=False)

    # Record whether the difference is statistically significant
    significant = p_value < significance_level

    if significant:
        significantly_different[month - 1] = 1

    # Save results for the table
    results.append({
        'Month': month,
        't-statistic': t_statistic,
        'p-value': p_value,
        'Significant?': 'True' if significant else 'False'
    })
```



A two proportion z-test was conducted for each month to determine which months showed a statistically significant difference in their proportion of rainy days at the 0.05 significance level. It was determined that the difference in mean daily rainfall was statistically significant in 8 out of 12 months, the code used to perform this analysis is provided below.


```python

import numpy as np
import pandas as pd
from statsmodels.stats.proportion import proportions_ztest

significance_level = 0.05
significantly_different_proportion = np.zeros(12)

results_proportion = []

# Perform a z-test for each month
for month in range(1, 13):

    # Create a contingency table for Seattle and Philadelphia
    contingency_table = pd.crosstab(
        df.loc[df['month'] == month, 'city'],
        df.loc[df['month'] == month, 'any_precipitation']
    )

    # Ensure both Boolean columns exist
    contingency_table = contingency_table.reindex(
        columns=[False, True],
        fill_value=0
    )

    # Calculate days with precipitation for each city
    days_with_precipitation = contingency_table[True]

    # Calculate total days for each city
    total_counts = contingency_table.sum(axis=1)

    # Perform the hypothesis test
    zstat, p_value = proportions_ztest(
        count=days_with_precipitation.to_numpy(),
        nobs=total_counts.to_numpy(),
        alternative='two-sided'
    )

    # Record whether the proportions are significantly different
    significantly_different_proportion[month - 1] = (
        p_value < significance_level
    )

    # Store the results
    results_proportion.append({
        'Month': month,
        'Z-statistic': zstat,
        'P-value': p_value,
        'Significantly Different': p_value < significance_level
    })

# Convert results to a DataFrame
results_proportion = pd.DataFrame(results_proportion)

# Display a formatted table
results_proportion.style.format({
    'Z-statistic': '{:.2f}',
    'P-value': '{:.3e}'
});

```

---
## Results
Our model included three explanatory variables of interest: median household income, percentage of students receiving free/reduced price lunches, and the percentage of adults with a college degree. The model explained 62.6% of the variation in ACT scores, suggesting that socioeconomic factors are associated with academic performance. Two of the three explanatory variables of interest were statistically significant, with the percentage of students receiving free or reduced-price lunch showing the strongest negative association. Median household income had the weakest association and was not statistically significant (p = 0.3973). These results indicate that socioeconomic factors are associated with ACT performance. Given the significant influence ACT scores can have on a high school student's future college opportunities and academic support, these standardized testing measures should be reevaluated.


---
## Author
Michael Schneider