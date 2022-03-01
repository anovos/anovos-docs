# Using Anovos at Scale

_Anovos_ is built for feature engineering and data processing at scale.
The library was built for and tested on [Mobilewalla's](https://www.mobilewalla.com/) mobile engagement data
with the following attributes:

| Property                   | Value       |
|----------------------------|-------------|
| Size                       | 50 GB       |
| No. of Rows                | 384,694,946 |
| No. of Columns             | 35          |
| No. of Numerical Columns   | 4           |
| No. of Categorical Columns | 31          |

## ⏱ Benchmark

To benchmark _Anovos_' performance, we ran a pipeline on this dataset.

The entire pipeline was optimized such that the computed statistics could be reused by other functions as much as possible.
For example, the modes (the most frequently seen values) computed by the [`measures_of_centralTendency`](../api/data_analyzer/stats_generator.md#measures_of_centraltendency)
function were also used for imputation while treating null values in a column with [`nullColumns_detection`](../api/data_analyzer/quality_checker.md#nullcolumns_detection)
or detecting a columns' biasedness using [`biasedness_detection`](../api/data_analyzer/quality_checker.md#biasedness_detection).

Hence, the time recorded for a function in the benchmark might differ significantly from the time taken by the same function
when running in isolation.

Further, Apache Spark does its own set of optimizations of transformations under the hood while running multiple
functions together, which further adds to the time difference.

| Function                                                                                             | Time (minutes) |
|------------------------------------------------------------------------------------------------------|----------------|
| [`global_summary`](../api/data_analyzer/stats_generator.md#global_summary)                           | 1              |
| [`measures_of_counts`](../api/data_analyzer/stats_generator.md#measures_of_counts)                   | 6              |
| [`measures_of_centralTendency`](../api/data_analyzer/stats_generator.md#measures_of_centraltendency) | 38             |
| [`measures_of_cardinality`](../api/data_analyzer/stats_generator.md#measures_of_cardinality)         | 65             |
| [`measures_of_percentiles`](../api/data_analyzer/stats_generator.md#measures_of_percentiles)         | 2              |
| [`measures_of_dispersion`](../api/data_analyzer/stats_generator.md#measures_of_dispersion)           | 2              |
| [`measures_of_shape`](../api/data_analyzer/stats_generator.md#measures_of_shape)                     | 3              |
| [`duplicate_detection`](../api/data_analyzer/quality_checker.md#duplicate_detection)                 | 6              |
| [`nullRows_detection`](../api/data_analyzer/quality_checker.md#nullrows_detection)                   | 3              |
| [`invalidEntries_detection`](../api/data_analyzer/quality_checker.md#invalidentries_detection)       | 17             |
| [`IDness_detection`](../api/data_analyzer/quality_checker.md#idness_detection)                       | 2              |
| [`biasedness_detection`](../api/data_analyzer/quality_checker.md#biasedness_detection)               | 2              |
| [`outlier_detection`](../api/data_analyzer/quality_checker.md#outlier_detection)                     | 4              |
| [`nullColumns_detection`](../api/data_analyzer/quality_checker.md#nullcolumns_detection)             | 2              |
| [`variable_clustering`](../api/data_analyzer/association_evaluator.md#variable_clustering)           | 2              |
| [`IV_calculation`](../api/data_analyzer/association_evaluator.md#iv_calculation)\*                   | 16             |
| [`IG_calculation`](../api/data_analyzer/association_evaluator.md#ig_calculation)\*                   | 12             |
| \* A binary categorical column was selected as a target variable to test this function.              |                |

To see if the library works with large number of attributes, we horizontally scale tested on different dataset with the following attributes:

| Property                   | Value       |
|----------------------------|-------------|
| Size                       | 15 GB       |
| No. of Rows                | 40,507,005  |
| No. of Columns             | 284         |
| No. of Numerical Columns   | 252         |
| No. of Categorical Columns | 23          |

| Function                                                                                             | Time (minutes) |
|------------------------------------------------------------------------------------------------------|----------------|
| [`global_summary`](../api/data_analyzer/stats_generator.md#global_summary)                           | 0.2            |
| [`measures_of_counts`](../api/data_analyzer/stats_generator.md#measures_of_counts)                   | 3              |
| [`measures_of_centralTendency`](../api/data_analyzer/stats_generator.md#measures_of_centraltendency) | 9              |
| [`measures_of_cardinality`](../api/data_analyzer/stats_generator.md#measures_of_cardinality)         | 12             |
| [`measures_of_percentiles`](../api/data_analyzer/stats_generator.md#measures_of_percentiles)         | 7              |
| [`measures_of_dispersion`](../api/data_analyzer/stats_generator.md#measures_of_dispersion)           | 9              |
| [`measures_of_shape`](../api/data_analyzer/stats_generator.md#measures_of_shape)                     | 5              |
| [`duplicate_detection`](../api/data_analyzer/quality_checker.md#duplicate_detection)                 | 2              |
| [`nullRows_detection`](../api/data_analyzer/quality_checker.md#nullrows_detection)                   | 4              |
| [`invalidEntries_detection`](../api/data_analyzer/quality_checker.md#invalidentries_detection)       | 9              |
| [`IDness_detection`](../api/data_analyzer/quality_checker.md#idness_detection)                       | 2              |
| [`biasedness_detection`](../api/data_analyzer/quality_checker.md#biasedness_detection)               | 2              |
| [`outlier_detection`](../api/data_analyzer/quality_checker.md#outlier_detection)                     | 85             |
| [`nullColumns_detection`](../api/data_analyzer/quality_checker.md#nullcolumns_detection)             | 3              |
| [`cat_to_num_unsupervised`](../api/data_transformer/transformers.md#cat_to_num_unsupervised)         | 4              |
| [`cat_to_num_supervised`](../api/data_transformer/transformers.md#cat_to_num_supervised)             | 2              |
| [`z_standardization`](../api/data_transformer/transformers.md#z_standardization)                     | 6              |
| [`IQR_standardization`](../api/data_transformer/transformers.md#IQR_standardization)                 | 3              |
| [`normalization`](../api/data_transformer/transformers.md#normalization)                             | 6              |
| [`PCA_latentFeatures`](../api/data_transformer/transformers.md#PCA_latentFeatures)                   | 20             |




## Limitations

For current performance limitations, see the dedicated overview of [_Anovos_' limitations](limitations.md#performance).
