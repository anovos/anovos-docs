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
For example, the modes (the most frequently seen values) computed by the [`measures_of_centralTendency`](../docs/anovos-modules-overview/data-analyzer/index.md#measures_of_centraltendency)
function were also used for imputation while treating null values in a column with [`nullColumns_detection`](../docs/anovos-modules-overview/quality-checker/index.md#nullcolumns_detection)
or detecting a columns' biasedness using [`biasedness_detection`](../docs/anovos-modules-overview/quality-checker/index.md#biasedness_detection).

Hence, the time recorded for a function in the benchmark might differ significantly from the time taken by the same function
when running in isolation.

Further, Apache Spark does its own set of optimizations of transformations under the hood while running multiple
functions together, which further adds to the time difference.

| Function                                                                                                            | Time (minutes) |
|---------------------------------------------------------------------------------------------------------------------|----------------|
| [`global_summary`](../docs/anovos-modules-overview/data-analyzer/index.md#global_summary)                           | 1              |
| [`measures_of_counts`](../docs/anovos-modules-overview/data-analyzer/index.md#measures_of_counts)                   | 6              |
| [`measures_of_centralTendency`](../docs/anovos-modules-overview/data-analyzer/index.md#measures_of_centraltendency) | 38             |
| [`measures_of_cardinality`](../docs/anovos-modules-overview/data-analyzer/index.md#measures_of_cardinality)         | 65             |
| [`measures_of_percentiles`](../docs/anovos-modules-overview/data-analyzer/index.md#measures_of_percentiles)         | 2              |
| [`measures_of_dispersion`](../docs/anovos-modules-overview/data-analyzer/index.md#measures_of_dispersion)           | 2              |
| [`measures_of_shape`](../docs/anovos-modules-overview/data-analyzer/index.md#measures_of_shape)                     | 3              |
| [`duplicate_detection`](../docs/anovos-modules-overview/quality-checker/index.md#duplicate_detection)               | 6              |
| [`nullRows_detection`](../docs/anovos-modules-overview/quality-checker/index.md#nullrows_detection)                 | 3              |
| [`invalidEntries_detection`](../docs/anovos-modules-overview/quality-checker/index.md#invalidentries_detection)     | 17             |
| [`IDness_detection`](../docs/anovos-modules-overview/quality-checker/index.md#idness_detection)                     | 2              |
| [`biasedness_detection`](../docs/anovos-modules-overview/quality-checker/index.md#biasedness_detection)             | 2              |
| [`outlier_detection`](../docs/anovos-modules-overview/quality-checker/index.md#outlier_detection)                   | 4              |
| [`nullColumns_detection`](../docs/anovos-modules-overview/quality-checker/index.md#nullcolumns_detection)           | 2              |
| [`variable_clustering`](../docs/anovos-modules-overview/association-evaluator/index.md#variable_clustering)         | 2              |
| [`IV_calculation`](../docs/anovos-modules-overview/association-evaluator/index.md#iv_calculation)\*                 | 16             |
| [`IG_calculation`](../docs/anovos-modules-overview/association-evaluator/index.md#ig_calculation)\*                 | 12             |
| \* A binary categorical column was selected as a target variable to test this function.                             |                |

## Limitations

For current performance limitations, see the dedicated overview of [_Anovos_' limitations](limitations.md#performance).
