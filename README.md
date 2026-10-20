# Rainfall Comparison between Seattle and Philadelphia

## Overview

This project compares rainfall data from Seattle and Philadelphia to determine which city receives more rainfall annually and experiences a greater proportion of rainy days throughout the year. The null hypotheses are that the mean daily rainfall and the proportion of rainy days are equal between the two cities.

---
## Data

The data analyzed in this project was sourced from the National Oceanic and Atmospheric Administration (NOAA) through the NOAA Climate Data Online (CDO) portal. The data is from two weather stations, one in Seattle and one in Philadelphia. The data spans a total of five years, from January 1, 2018, to December 31, 2022.
The data can also be accessed through the following link:
https://www.ncei.noaa.gov/cdo-web/


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
Seattle and Philadelphia were the focus locations of this project, and the key questions that were answered were which city has a higher mean daily rainfall in terms of inches and which city sees a higher proportion of rainy days throughout the year. We also presented a null hypothesis that both cities have the same daily mean rainfall, and that both cities have the same proportion of rainy days throughout the year. 
  
In terms of mean annual rainfall, Philadelphia averaged 18.5% more rain throughout the year. In terms of the proportion of days with rain, on the other hand, Seattle averaged 52.4% more days with rain throughout the year compared to Philadelphia. These were yearly averages, and there was a lot of variety each month of the year.  

When we looked at the monthly patterns in the data, the difference in mean daily rainfall was statistically significant in 8 out of 12 months. The only months that were not statistically significant were February, March, April, and October. This allowed us to reject our null hypothesis that the average daily rainfall in Seattle and Philadelphia was equal for 8 out of 12 months.

When we looked at the monthly patterns in the proportion of rainy days, the difference in the proportions of rainy days was statistically significant in 8 out of 12 months. The months that were not statistically significant were May, June, August, and September. This allowed us to reject our null hypothesis that the proportions of rainy days were equal in Seattle and Philadelphia for 8 out of the 12 months of the year.

This study focused on analyzing precipitation data in Philadelphia and Seattle. However, a limitation of the study is that our data covered only a 5-year period. This may be insufficient to predict future weather patterns, as well as to generalize historical weather patterns. That being said, this study was able to reject our null hypotheses that the mean daily rainfall and the proportion of rainy days were equal in both cities in 8 out of the 12 months.


---
## Author
Michael Schneider