# Anovos Scale Testing

We tested Anovos modules on Mobilewalla's mobile engagement data with the following attributes:

| Data Size | 50 GB |
| --- | --- |
| No. of Rows | 384,694,946 |
| No. of Columns | 35 |
| No. of Numerical Columns | 4 |
| No. of Categorical Columns | 31 |

The entire pipeline was optimized so computed statistics can be reused by other functions e.g. mode (most frequently seen value) computed under statistics generator (**measures_of_centralTendency**) were also used for imputation while treating null values in a column (**nullColumns_detection**) or detecting biasedness in a column (**biasedness_detection**). The time taken by a function in a pipeline may differ significantly from the time taken by the same function on running in solitary. Also, Spark does its own set of optimizations transformations under the hood while running multiple functions together, which further adds to the time difference. For further detail, please refer to the  `main.py` script in the Github.

| **Function** | **Time (mins)** |
| --- | --- |
| global\_summary | 1 |
| measures\_of\_counts | 6 |
| measures\_of\_centralTendency | 38 |
| measures\_of\_cardinality | 65 |
| measures\_of\_percentiles | 2 |
| measures\_of\_dispersion | 2 |
| measures\_of\_shape | 3 |
| duplicate\_detection | 6 |
| nullRows\_detection | 3 |
| invalidEntries\_detection | 17 |
| IDness\_detection | 2 |
| biasedness\_detection | 2 |
| outlier\_detection | 4 |
| nullColumns\_detection | 2 |
| variable\_clustering | 2 |
| IV Calculation\* | 16 |
| IG Calculation\* | 12 |
`* A binary categorical column was selected as a target variable to test this function.`

Limitations:
- Computing mode and/or distinct count are most expensive operations in Anovos. We aim to further optimize them in the upcoming releases.
- Correlation Matrix may throw memory issues if very high cardinality categorical features are involved – a limitation that was propagated from phik library.




 

