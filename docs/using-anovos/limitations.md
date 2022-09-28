# Current Limitations of Anovos

The current 1.0 release of _Anovos_ still has some limitations, which we will address in the upcoming releases.
To learn more about what's on the horizon, check out our [roadmap](roadmap.md).

## 🔣 Data

- _Anovos_ currently supports numerical, categorical, geospatial, and datetime/timestamp columns
  (at the cross-sectional and transactional level).
  We plan to add support for additional data types such as (struct) arrays in the future.

- _Anovos_ currently relies on
  [Apache Spark's automatic schema detection](https://spark.apache.org/docs/2.2.1/sql-programming-guide.html#inferring-the-schema-using-reflection).
  In case some numerical columns were deliberately saved as string, they will show up as categorical columns when
  loaded into a DataFrame (except for CSV files).

## 🏎 Performance

- Computing the [mode](../api/data_analyzer/stats_generator.md#measures_of_centraltendency) and/or
  distinct value counts are the most expensive operations in _Anovos_.
  We aim to further optimize them in the upcoming releases.

- Correlation matrix only supports numerical data.
  Support for categorical data has been removed due to performance concerns and will return in a later release.

- The [invalid entries detection](../api/data_analyzer/quality_checker.md#invalidentries_detection)
  may yield false positives.
  Hence, be cautious when using the inbuilt treatment option.

- The categorical encoding functions
  [`cat_to_num_supervised`](../api/data_transformer/transformers.md#cat_to_num_supervised)
  and
  [`cat_to_num_unsupervised`](../api/data_transformer/transformers.md#cat_to_num_unsupervised)
  may exhibit poor performance and scaling behavior with very high-cardinality columns.
  Therefore, it is recommended to reduce cardinality before subjecting them to encoding
  or specifying an appropriate threshold to drop them from the analysis while encoding.

- The sample size for constructing the imputation models in
  [`imputation_sklearn`](../api/data_transformer/transformers.md#imputation_sklearn)
  or creating latent features through
  [`autoencoder_latentFeatures`](../api/data_transformer/transformers.md#autoencoder_latentFeatures)
  should be selected with caution, taking into the consideration the dataset size and the number of columns.
  This sample dataset is converted into a Pandas DataFrame and subsequent operations are run on a single node (driver).
  If the sample dataset is too large to fit into the driver's memory, this will result in a memory overflow error.

## 🔩 Other

- The [`stability index`](../api/drift/detector.md#stabilityindex_computation)
  can currently only be calculated for numerical columns.

- Due to incompatibilities in TensorFlow and Docker, _Anovos_ may not run well on Apple's M1 chips.
  You can find out more here:
  - [Docker documentation on running on Apple hardware](https://docs.docker.com/desktop/mac/apple-silicon/)
  - [Pythonspeed article on Docker build problems on Macs](https://pythonspeed.com/articles/docker-build-problems-mac/)
  - [Installing TensorFlow on M1-chip-based Macs](https://caffeinedev.medium.com/how-to-install-tensorflow-on-m1-mac-8e9b91d93706)

- The exception and error handling within _Anovos_ is at times inconsistent. Please don't hesitate to
  [file an issue on GitHub](https://github.com/anovos/anovos/issues) if you encounter any problems.
