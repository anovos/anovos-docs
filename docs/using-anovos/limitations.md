# Current Limitations of Anovos

The current Alpha release of _Anovos_ still has several limitations, the majority of which we'll address
in time for version 1.0. To learn more about what's on the horizon, check out our [roadmap](roadmap.md).
 
## 🔣 Data
- _Anovos_ currently only supports numerical and categorical columns.
  We plan to add support for additional data types such as dates, time stamps, (struct) arrays etc.
in the future.

- Geospatial columns like geohash or lat/long can only be analysed as categorical (geohash)
  or numerical (lat/long). Functionalities specific to geospatial features will be supported later releases.

- _Anovos_ currently relies on [Apache Spark's automatic schema detection](https://spark.apache.org/docs/2.2.1/sql-programming-guide.html#inferring-the-schema-using-reflection).
  In case some numerical columns were deliberately saved as string, they will show up as categorical columns when
  loaded into a DataFrame (except for CSV files).

## 🏎 Performance 
- Computing the [mode](../api/anovos/data_analyzer/stats_generator.md#measures_of_centraltendency) and/or 
  distinct value counts are the most expensive operations in _Anovos_.  
  We aim to further optimize them in the upcoming releases.

- Calculating a [correlation matrix](../api/anovos/data_analyzer/association_evaluator.md#correlation_matrix)
  may result in memory issues if very high cardinality categorical features are involved – a limitation that was
  propagated from the underlying `phik` library. Therefore, we recommend dropping these columns if not necessary.

- The [invalid entries detection](../api/anovos/quality_checker.md#invalidentries_detection)
  currently only looks for selected suspicious patterns. We aim to add more flexibility to this function with user
  provided patterns, i.e., regex support.
  Also, this function may yield false positives. Hence, be cautious when using the inbuilt treatment option.

## 🔩 Other
- The [stability index](../api/anovos/data_drift/drift_detector.md#stabilityindex_computation)
  can currently only be calculated for numerical columns.

- The exception and error handling is at times inconsistent. Please don't hesitate to
  [file an issue on GitHub](https://github.com/anovos/anovos/issues) if you encounter any problems.
