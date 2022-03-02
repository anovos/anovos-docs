# Current Limitations of Anovos

The current V0.2.0 release of _Anovos_ still has some limitations, which we will address in the upcoming releases. 
To learn more about what's on the horizon, check out our [roadmap](roadmap.md).
 
## 🔣 Data
- _Anovos_ currently only supports numerical, categorical and datetime/timestamp related columns (cross sectional & transactional level).
  We plan to add support for additional data types such as (struct) arrays, geo spatical etc. in the future.

- Geospatial columns like geohash or lat/long can only be analysed as categorical (geohash)
  or numerical (lat/long). Functionalities specific to geospatial features will be supported in the later releases.

- _Anovos_ currently relies on [Apache Spark's automatic schema detection](https://spark.apache.org/docs/2.2.1/sql-programming-guide.html#inferring-the-schema-using-reflection).
  In case some numerical columns were deliberately saved as string, they will show up as categorical columns when
  loaded into a DataFrame (except for CSV files).

## 🏎 Performance 
- Computing the [mode](../api/data_analyzer/stats_generator.md#measures_of_centraltendency) and/or 
  distinct value counts are the most expensive operations in _Anovos_.  
  We aim to further optimize them in the upcoming releases.

- Calculating a [correlation matrix](../api/data_analyzer/association_evaluator.md#correlation_matrix)
  may result in memory issues if very high cardinality categorical features are involved – a limitation that was
  propagated from the underlying `phik` library. Therefore, we recommend dropping these columns if not necessary.

- The [invalid entries detection](../api/data_analyzer/quality_checker.md#invalidentries_detection)
  may yield false positives. Hence, be cautious when using the inbuilt treatment option.

- The categorical encoding functions - [cat_to_num_supervised](../api/data_transformer/transformers.md#cat_to_num_supervised) & [cat_to_num_unsupervised](../api/data_transformer/transformers.md#cat_to_num_unsupervised) - may show some performance issues 
  with very high cardinality columns. Therefore, it is recommended to reduce cardinality before subjecting them to encoding 
  or put an appropriate threshold to drop them from the analysis while encoding.

- Sample size for constructing the imputation models in [imputation_sklearn](../api/data_transformer/transformers.md#imputation_sklearn) or creating latent features through [autoencoder_latentFeatures](../api/data_transformer/transformers.md#autoencoder_latentFeatures) should be selected with caution, taking into the consideration 
  the dataset size and the number of columns. This sample dataset is converted into pandas dataframe which run operations on a single node (driver).
  If sample dataset is too large to fit into the driver, it may throw error or cause severe performance issue.


## 🔩 Other
- The [stability index](../api/drift/detector.md#stabilityindex_computation)
  can currently only be calculated for numerical columns.

- The exception and error handling is at times inconsistent. Please don't hesitate to
  [file an issue on GitHub](https://github.com/anovos/anovos/issues) if you encounter any problems.
