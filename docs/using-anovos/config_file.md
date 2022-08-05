# Configuring Workloads

_Anovos_ workloads can be described by a YAML configuration file.

Such a configuration file defines:

- the input dataset(s)
- the analyses and transformations to be performed on the data
- the output files and dataset(s)
- the reports to be generated

Defining workloads this way allows users to make full use of _Anovos_ capabilities
while maintaining an easy-to-grasp overview.
Since each configuration file fully describes one workload,  these files can be
shared,  versioned,  and run across different compute environments.

In the following,  we'll describe in detail each of the sections in an _Anovos_
configuration file.
If you'd rather see a full example right away,  have a look at
[this example](https://github.com/anovos/anovos/blob/main/config/configs.yaml).

Note that each section of the configuration file maps to a module of _Anovos_.
You'll find links to the respective sections of the
[API Documentation](../api/_index.md) that provide much more detailed information
on each modules' capabilities than we can squeeze into this guide.

## 📑 `input_dataset`

This configuration block describes how the input dataset is loaded and prepared
using the [`data_ingest.data_ingest`](../api/data_ingest/data_ingest.md) module.
Each _Anovos_ configuration file must contain exactly one `input_dataset` block.

Note that the subsequent operations are performed in the order given here:
First,  columns are deleted,  then selected,  then renamed,  and then recast.

### `read_dataset`

🔎 _Corresponds to [`data_ingest.read_dataset`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.read_dataset)_

- `file_path`: The file (or directory) path to read the input dataset from.
   It can be a local path,  an [📖 S3 path](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-bucket-intro.html)
  (when running on AWS),  a path to a file resource on Google Colab (see
   [📖 this tutorial](https://neptune.ai/blog/google-colab-dealing-with-files) for
   an overview),  or a path on the [📖 Databricks File System](https://docs.microsoft.com/de-de/azure/databricks/data/databricks-file-system)
   (when running on Azure).

- `file_type`: The file format of the input data. Currently,  _Anovos_ supports
   CSV (`csv`),  Parquet (`parquet`),  and Avro (`avro`).
   (Please note that if you're using Avro data sources,  you need to add the external
   package `org.apache.spark:spark-avro` when submitting the Spark job.)

- `file_configs` (optional): Options to pass to the respective Spark file reader,
   e.g.,  delimiters,  schemas,  headers. In the case of a CSV file,  this might look
   like:

   ```yaml
   file_configs:
     delimiter: ", "
     header: True
     inferSchema: True
   ```

   For more information on available configuration options,  see the following external
   documentation:

  - [Read CSV files](https://sparkbyexamples.com/pyspark/pyspark-read-csv-file-into-dataframe/)
  - [Read Parquet files](https://sparkbyexamples.com/pyspark/pyspark-read-and-write-parquet-file/)
  - [Read Avro files](https://sparkbyexamples.com/spark/read-write-avro-file-spark-dataframe/)

### `delete_column`

🔎 _Corresponds to [`data_ingest.delete_column`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.delete_column)_

List of column names (list of strings or string of column names separated by `|`)
to be deleted from the loaded input data.

🤓  _Example:_

```yaml
delete_column: ['unnecessary',  'obsolete',  'outdated']
```

### `select_column`

🔎 _Corresponds to [`data_ingest.select_column`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.select_column)_

List of column names (list of strings or string of column names separated by `|`)
to be selected for further processing.

🤓  _Example:_

```yaml
select_column: ['feature1',  'feature2',  'feature3',  'label']
```

### `rename_column`

🔎 _Corresponds to [`data_ingest.rename_column`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.rename_column)_

- `list_of_cols`: List of the names of columns (list of strings or string of column names separated by `|`)
  to be renamed.

- `list_of_newcols`: The new column names. The first element in `list_of_cols` will be renamed
  to the first name in `list_of_newcols` and so on.

🤓  _Example:_

```yaml
rename_column:
  list_of_cols: ['very_long_column_name',  'price']
  list_of_newcols: ['short_name',  'label']
```

This will rename the column `very_long_column_name` to `short_name` and the column `price` to `label`.

### `recast_column`

🔎 _Corresponds to [`data_ingest.recast_column`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.recast_column)_

- `list_of_cols`: List of the names of columns (list of strings or string of column names separated by `|`)
  to be cast to a different type.

- `list_of_dtypes`: The new datatypes. The first element in `list_of_cols` will be recast
  to the first type in `list_of_dtypes` and so on. See
  [📖 the Spark documentation](https://spark.apache.org/docs/latest/sql-ref-datatypes.html)
  for a list of valid datatypes.
  Note that this field is case-insensitive.

🤓  _Example:_

```yaml
recast_column:
  list_of_cols: ['price',  'quantity']
  list_of_dtypes: ['double',  'int']
```

## 📑 `concatenate_dataset`

🔎 _Corresponds to [`data_ingest.concatenate_dataset`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.concatenate_dataset)_

This configuration block describes how to combine multiple loaded dataframes into a single one.

### `method`

There are two different methods to concatenate dataframes:

- `index`: Concatenate by column index,  i.e.,  the first column of the first dataframe is
  matched with the first column of the second dataframe and so forth.
- `name`: Concatenate by column name,  i.e.,  columns of the same name are matched.

Note that in both cases,  the first dataframe will define both the names and the order of the
columns in the final dataframe.
If the subsequent dataframes have too few columns (`index`) or are missing named columns (`name´)
for the concatenation to proceed,  an error will be raised.

🤓  _Example:_

```yaml
method: name
```

### `dataset1`

#### `read_dataset`

🔎 _Corresponds to [`data_ingest.read_dataset`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.read_dataset)_

- `file_path`: The file (or directory) path to read the other concatenating input dataset from.
  It can be a local path,  an [📖 S3 path](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-bucket-intro.html)
  (when running on AWS),  a path to a file resource on Google Colab (see
  [📖 this tutorial](https://neptune.ai/blog/google-colab-dealing-with-files) for
  an overview),  or a path on the
  [📖 Databricks File System](https://docs.microsoft.com/de-de/azure/databricks/data/databricks-file-system)
  (when running on Azure).

- `file_type`: The file format of the other concatenating input data. Currently,  _Anovos_ supports
  CSV (`csv`),  Parquet (`parquet`),  and Avro (`avro`).
  (Please note that if you're using Avro data sources,  you need to add the external
  package `org.apache.spark:spark-avro` when submitting the Spark job.)

- `file_configs` (optional): Options to pass to the respective Spark file reader,
  e.g.,  delimiters,  schemas,  headers.

#### `delete_column`

🔎 _Corresponds to [`data_ingest.delete_column`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.delete_column)_

List of column names (list of strings or string of column names separated by `|`)
to be deleted from the loaded input data.

#### `select_column`

🔎 _Corresponds to [`data_ingest.select_column`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.select_column)_

List of column names (list of strings or string of column names separated by `|`)
to be selected for further processing.

#### `rename_column`

🔎 _Corresponds to [`data_ingest.rename_column`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.rename_column)_

- `list_of_cols`: List of the names of columns (list of strings or string of column names separated by `|`)
  to be renamed.

- `list_of_newcols`: The new column names. The first element in `list_of_cols` will be renamed
  to the first name in `list_of_newcols` and so on.

#### `recast_column`

🔎 _Corresponds to [`data_ingest.recast_column`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.recast_column)_

- `list_of_cols`: List of the names of columns (list of strings or string of column names separated by `|`)
  to be cast to a different type.

- `list_of_dtypes`: The new datatypes. The first element in `list_of_cols` will be recast
  to the first type in `list_of_dtypes` and so on. See
  [📖 the Spark documentation](https://spark.apache.org/docs/latest/sql-ref-datatypes.html)
  for a list of valid datatypes.
  Note that this field is case-insensitive.

### `dataset2`,  `dataset3`,  …

Additional datasets are configured in the same manner as `dataset1`.

## 📑 `join_dataset`

🔎 _Corresponds to [`data_ingest.join_dataset`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.join_dataset)_

This configuration block describes how multiple dataframes are joined into a single one.

### `join_cols`

The key of the column(s) to join on.

In the case that the key consists of multiple columns,  they can be passed as a list of strings or
a single string where the column names are separated by `|`.

🤓  _Example:_

```yaml
join_cols: id_column
```

### `join_type`

The type of join to perform: `inner`,  `full`,  `left`,  `right`,  `left_semi`,  or `left_anti`.

For a general introduction to joins,  see
[📖 this tutorial](https://sparkbyexamples.com/spark/spark-sql-dataframe-join/).

🤓  _Example:_

```yaml
join_type: inner
```

### `dataset1`

#### `read_dataset`

🔎 _Corresponds to [`data_ingest.read_dataset`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.read_dataset)_

- `file_path`: The file (or directory) path to read the other joining input dataset from.
  It can be a local path,  an [📖 S3 path](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-bucket-intro.html)
  (when running on AWS),  a path to a file resource on Google Colab (see
  [📖 this tutorial](https://neptune.ai/blog/google-colab-dealing-with-files) for
  an overview),  or a path on the
  [📖 Databricks File System](https://docs.microsoft.com/de-de/azure/databricks/data/databricks-file-system)
  (when running on Azure).

- `file_type`: The file format of the other joining input data. Currently,  _Anovos_ supports
  CSV (`csv`),  Parquet (`parquet`),  and Avro (`avro`).
  (Please note that if you're using Avro data sources,  you need to add the external
  package `org.apache.spark:spark-avro` when submitting the Spark job.)

- `file_configs` (optional): Options to pass to the respective Spark file reader,
  e.g.,  delimiters,  schemas,  headers.

#### `delete_column`

🔎 _Corresponds to [`data_ingest.delete_column`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.delete_column)_

List of column names (list of strings or string of column names separated by `|`)
to be deleted from the loaded input data.

#### `select_column`

🔎 _Corresponds to [`data_ingest.select_column`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.select_column)_

List of column names (list of strings or string of column names separated by `|`)
to be selected for further processing.

#### `rename_column`

🔎 _Corresponds to [`data_ingest.rename_column`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.rename_column)_

- `list_of_cols`: List of the names of columns (list of strings or string of column names separated by `|`)
  to be renamed.

- `list_of_newcols`: The new column names. The first element in `list_of_cols` will be renamed
  to the first name in `list_of_newcols` and so on.

#### `recast_column`

🔎 _Corresponds to [`data_ingest.recast_column`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.recast_column)_

- `list_of_cols`: List of the names of columns (list of strings or string of column names separated by `|`)
  to be cast to a different type.

- `list_of_dtypes`: The new datatypes. The first element in `list_of_cols` will be recast
  to the first type in `list_of_dtypes` and so on. See
  [📖 the Spark documentation](https://spark.apache.org/docs/latest/sql-ref-datatypes.html)
  for a list of valid datatypes.
  Note that this field is case-insensitive.

### `dataset2`,  `dataset3`,  …

Additional datasets are configured in the same manner as `dataset1`.

## 📑 `timeseries_analyzer`

🔎 _Corresponds to [`data_analyzer.ts_analyzer`](../api/data_analyzer/ts_analyzer.md)_

Configuration for the time series analyzer.

- `auto_detection`: Can be set to `True` or `False`.
  If `True`,  it attempts to automatically infer the date/timestamp format in the input dataset.

- `id_col`: Name of the ID column in the input dataset.

- `tz_offset`: The timezone offset of the timestamps in the input dataset.
  Can be set to either `local`,  `gmt`,  or `utc`. The default setting is `local`.

- `inspection`: Can be set to `True` or `False`.
  If `True`,  the time series elements undergo an inspection.

- `analysis_level`: Can be set to `daily`,  `weekly`,  or `hourly`. The default setting is `daily`.
  If set to `daily`,  the daily view is populated.
  If set to `hourly`,  the view is shown at a day part level.
  If set to `weekly`,  the display it per individual weekdays (1-7) as captured.

- `max_days`: Maximum number of days up to which the data will be aggregated.
  If the dataset contains a timestamp/date field with very high number of unique dates
  (e.g.,  20 years worth of daily data),  this option can be used to reduce the timespan that is analyzed.

🤓  _Example:_

```yaml
timeseries_analyzer:
    auto_detection: True
    id_col: 'id_column'
    tz_offset: 'local'
    inspection: True
    analysis_level: 'daily'
    max_days: 3600
```

## 📑 `anovos_basic_report`

🔎 _Corresponds to [`data_report.basic_report_generation`](../api/data_report/basic_report_generation.md)_

The basic report consists of a summary of the outputs of the
[stats_generator](../api/data_analyzer/stats_generator.md),
[quality_checker](../api/data_analyzer/quality_checker.md),  and
[association evaluator](../api/data_analyzer/association_evaluator.md)
See the [📖 documentation for data reports](data-reports/overview.md) for more details.

The basic report can be customized using the following options:

### `basic_report`

If `True`,  a basic report is generated after completion of the
[data_analyzer](../api/data_analyzer/_index.md) modules.

If `False`,  no report is generated.
Nevertheless,  all the computed statistics and metrics will be available in the final report.

### `report_args`

- `id_col`: The name of the ID column in the input dataset.

- `label_col`: The name of the label or target column in the input dataset.

- `event_lable`: The value of the event (label `1`/`true`) in the label column.

- `output_path`: Path where the basic report is saved.
  It can be a local path,  an [📖 S3 path](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-bucket-intro.html)
  (when running on AWS),  a path to a file resource on Google Colab (see
  [📖 this tutorial](https://neptune.ai/blog/google-colab-dealing-with-files) for
  an overview),  or a path on the [📖 Databricks File System](https://docs.microsoft.com/de-de/azure/databricks/data/databricks-file-system)
  (when running on Azure).

🤓  _Example:_

```yaml
report_args:
  id_col: id_column
  label_col: label_col
  event_label: 'class1'
  output_path: report_stats
```

## 📑 `stats_generator`

🔎 _Corresponds to [`data_analyzer.stats_generator`](../api/data_analyzer/stats_generator.md)_

This module generates descriptive statistics of the ingested data.
Descriptive statistics are split into different metric types.
Each function corresponds to one metric type.

### `metric`

List of metrics to calculate for the input dataset.
Available options are:

- [📖  `global_summary`](../api/data_analyzer/stats_generator.md#anovos.data_analyzer.stats_generator.global_summary)
- [📖  `measures_of_count`](../api/data_analyzer/stats_generator.md#anovos.data_analyzer.stats_generator.measures_of_count)
- [📖  `measures_of_centralTendency`](../api/data_analyzer/stats_generator.md#anovos.data_analyzer.stats_generator.measures_of_centralTendency)
- [📖  `measures_of_cardinality`](../api/data_analyzer/stats_generator.md#anovos.data_analyzer.stats_generator.measures_of_cardinality)
- [📖  `measures_of_dispersion`](../api/data_analyzer/stats_generator.md#anovos.data_analyzer.stats_generator.measures_of_dispersion)
- [📖  `measures_of_percentiles`](../api/data_analyzer/stats_generator.md#anovos.data_analyzer.stats_generator.measures_of_percentiles)
- [📖  `measures_of_shape`](../api/data_analyzer/stats_generator.md#anovos.data_analyzer.stats_generator.measures_of_shape)

🤓  _Example:_

```yaml
metric: ['global_summary',  'measures_of_counts',  'measures_of_cardinality',  'measures_of_dispersion']
```

### `metric_args`

- `list_of_cols`: List of column names (list of strings or string of column names separated by `|`)
  to compute the metrics for.
  Alternatively,  if set to `"all"`,  all columns are included.

- `drop_cols`:  List of column names (list of strings or string of column names separated by `|`)
  to exclude from metrics computation.
  This option is especially useful if `list_of_cols` is set to `"all"`,  as it allows computing metrics
  for all except a few columns without having to specify a potentially very long list of column names to include.

🤓  _Example:_

```yaml
metric_args:
  list_of_cols: all
  drop_cols: ['id_column']
```

## 📑 `quality_checker`

🔎 _Corresponds to [`data_analyzer.quality_checker`](../api/data_analyzer/quality_checker.md)_

This module assesses the data quality along different dimensions.
Quality metrics are computed at both the row and column level.
Further,  the module includes appropriate treatment options to fix several common quality issues.

### `duplicate_detection`

🔎 _Corresponds to [`quality_checker.duplicate_detection`](../api/data_analyzer/quality_checker.md#anovos.data_analyzer.quality_checker.duplicate_detection)_

- `list_of_cols`: List of column names (list of strings or string of column names separated by `|`)
  to consider when searching for duplicates.
  Alternatively,  if set to `"all"`,  all columns are included.

- `drop_cols`: List of column names (list of strings or string of column names separated by `|`)
  to be excluded from duplicate detection.

- `treatment`: If `False`,  duplicates are detected and reported.
  If `True`,  duplicate rows are removed from the input dataset.

🤓  _Example:_

```yaml
duplicate_detection:
  list_of_cols: all
  drop_cols: ['id_column']
  treatment: True
```

### `nullRows_detection`

🔎 _Corresponds to [`quality_checker.nullRows_detection`](../api/data_analyzer/quality_checker.md#anovos.data_analyzer.quality_checker.nullRows_detection)_

- `list_of_cols`: List of column names (list of strings or string of column names separated by `|`)
  to consider during `null` rows detection.
  Alternatively,  if set to `"all"`,  all columns are included.

- `drop_cols`: List of column names (list of strings or string of column names separated by `|`)
  to exclude from `null` rows detection.

- `treatment`: If `False`,  `null` rows are detected and reported.
  If `True`,  rows where more than `treatment_threshold` columns are `null` are removed from the input dataset.

- `treatment_threshold`: It takes a value between `0` and `1` (default `0.8`) that specifies which fraction of
  columns has to be `null` for a row to be considered a `null` row.
  If the threshold is `0`,  rows with any missing value will be flagged as `null`.
  If the threshold is `1`,  only rows where all values are missing will be flagged as `null`.

🤓  _Example:_

```yaml
nullRows_detection:
  list_of_cols: all
  drop_cols: []
  treatment: True
  treatment_threshold: 0.75
```

### `invalidEntries_detection`

🔎 _Corresponds to [`quality_checker.invalidEntries_detection`](../api/data_analyzer/quality_checker.md#anovos.data_analyzer.quality_checker.invalidEntries_detection)_

- `list_of_cols`: List of column names (list of strings or string of column names separated by `|`)
  to be considered during invalid entries' detection.
  Alternatively,  if set to `"all"`,  all columns are included.

- `drop_cols`: List of column names (list of strings or string of column names separated by `|`)
  to exclude from invalid entries' detection.

- `treatment`: If `False`,  invalid entries are detected and reported.
  If `True`,  invalid entries are replaced with `null`.

- `output_mode`: Can be either `"replace"` or `"append"`.
 If set to `"replace"`,  the original columns will be replaced with the treated columns.
 If set to `"append"`,  the original columns will be kept and the treated columns will be appended to the dataset.
 The appended columns will be named as the original column with a suffix `"_cleaned"`
 (e.g.,  the column `"cost_of_living_cleaned"` corresponds to the original column `"cost_of_living"`).

🤓  _Example:_

```yaml
invalidEntries_detection:
  list_of_cols: all
  drop_cols: ['id_column']
  treatment: True
  output_mode: replace
```

### `IDness_detection`

🔎 _Corresponds to [`quality_checker.IDness_detection`](../api/data_analyzer/quality_checker.md#anovos.data_analyzer.quality_checker.IDness_detection)_

- `list_of_cols`: List of column names (list of strings or string of column names separated by `|`)
  to be considered for IDness detection.
  Alternatively,  if set to `"all"`,  all columns are included.

- `drop_cols`: List of column names (list of strings or string of column names separated by `|`)
  to exclude from IDness detection.

- `treatment`: If `False`,  columns with high IDness are detected and reported.
  If `True`,  columns with an IDness above `treatment_threshold` are removed.

- `treatment_threshold`: A value between `0` and `1` (default `1.0`).

🤓  _Example:_

```yaml
IDness_detection:
  list_of_cols: all
  drop_cols: ['id_column']
  treatment: True
  treatment_threshold: 0.9
```

### `biasedness_detection`

🔎 _Corresponds to [`quality_checker.biasedness_detection`](../api/data_analyzer/quality_checker.md#anovos.data_analyzer.quality_checker.biasedness_detection)_

- `list_of_cols`: List of column names (list of strings or string of column names separated by `|`)
  to be considered for biasedness detection.
  Alternatively,  if set to `"all"`,  all columns are included.

- `drop_cols`: List of column names (list of strings or string of column names separated by `|`)
  to exclude from biasedness detection.

- `treatment`: If `False`,  columns with high IDness are detected and reported.
  If `True`,  columns with a bias above `treatment_threshold` are removed.

- `treatment_threshold`: A value between `0` and `1` (default `1.0`).

🤓  _Example:_

```yaml
biasedness_detection:
  list_of_cols: all
  drop_cols: ['label_col']
  treatment: True
  treatment_threshold: 0.98
```

### `outlier_detection`

🔎 _Corresponds to [`quality_checker.outlier_detection`](../api/data_analyzer/quality_checker.md#anovos.data_analyzer.quality_checker.outlier_detection)_

- `list_of_cols`: List of column names (list of strings or string of column names separated by `|`)
  to be considered for outlier detection.
  Alternatively,  if set to `"all"`,  all columns are included.

  ⚠ _Note that any column that contains just a single value or only null values is not subjected to outlier detection_
  _even if it is selected under this argument._

- `drop_cols`: List of column names (list of strings or string of column names separated by `|`)
  to exclude from outlier detection.

- `detection_side`: Whether outliers should be detected on the `"upper"`,  the `"lower"`,  or `"both"` sides.

- `detection_configs`: A map that defines the input parameters for different outlier detection methods.
  Possible keys are:
  - `pctile_lower` (default `0.05`)
  - `pctile_upper` (default `0.95`)
  - `stdev_lower` (default `3.0`)
  - `stdev_upper` (default `3.0`)
  - `IQR_lower` (default `1.5`)
  - `IQR_upper` (default `1.5`)
  - `min_validation` (default `2`)
  For details,  see [📖 the `outlier_detection` API documentation](../api/data_analyzer/quality_checker.md#anovos.data_analyzer.quality_checker.outlier_detection)

- `treatment`: If `False`,  outliers are detected and reported.
  If `True`,  outliers are treated with the specified `treatment_method`.

- `treatment_method`: Specifies how outliers are treated.
  Possible options are `"null_replacement"`,  `"row_removal"`,  `"value_replacement"`.

- `pre_existing_model`: If `True`,  the file specified under `model_path` with lower/upper bounds is loaded.
  If no such file exists,  set to `False` (the default).

- `model_path`: The path to the file with lower/upper bounds.
  It can be a local path,  an [📖 S3 path](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-bucket-intro.html)
  (when running on AWS),  a path to a file resource on Google Colab (see
  [📖 this tutorial](https://neptune.ai/blog/google-colab-dealing-with-files) for
  an overview),  or a path on the [📖 Databricks File System](https://docs.microsoft.com/de-de/azure/databricks/data/databricks-file-system)
  (when running on Azure).
  If `pre_existing_model` is `True`,  the pre-saved will be loaded from this location.
  If `pre_existing_model` is `False`,  a file with lower/upper bounds will be saved at this location.
  By default,  it is set to `NA`,  indicating that there is neither a pre-saved file nor should such a file be generated.

- `output_mode`: Can be either `"replace"` or `"append"`.
  If set to `"replace"`,  the original columns will be replaced with the treated columns.
  If set to `"append"`,  the original columns will be kept and the treated columns will be appended to the dataset.
  The appended columns will be named as the original column with a suffix `"_outliered"`
  (e.g.,  the column `"cost_of_living_outliered"` corresponds to the original column `"cost_of_living"`).

🤓  _Example:_

```yaml
outlier_detection:
  list_of_cols: all
  drop_cols: ['id_column', 'label_col']
  detection_side: upper
  detection_configs:
    pctile_lower: 0.05
    pctile_upper: 0.90
    stdev_lower: 3.0
    stdev_upper: 3.0
    IQR_lower: 1.5
    IQR_upper: 1.5
    min_validation: 2
  treatment: True
  treatment_method: value_replacement
  pre_existing_model: False
  model_path: NA
  output_mode: replace
```

### `nullColumns_detection`

🔎 _Corresponds to [`quality_checker.nullColumns_detection`](../api/data_analyzer/quality_checker.md#anovos.data_analyzer.quality_checker.nullColumns_detection)_

- `list_of_cols`: List of column names (list of strings or string of column names separated by `|`)
  to be considered for `null` columns detection.
  Alternatively,  if set to `"all"`,  all columns are included.
  If set to `"missing"` (the default) only columns with missing values are included.
  One of the use cases where `"all"` may be preferable over `"missing"` is when the user wants to save the
  imputation model for future use.
  This can be useful,  for example,  if a column may not have missing values in the training dataset but
  missing values are acceptable in the test dataset.

- `drop_cols`: List of column names (list of strings or string of column names separated by `|`)
  to be excluded from `null` columns detection.

- `treatment`: If `False`,  `null` columns are detected and reported.
  If `True`,  missing values are treated with the specified `treatment_method`.

- `treatment_method`: Specifies how `null` columns are treated.
  Possible values are `"MMM"`,  "`row_removal"`,  or `"column_removal"`.

- `treatment_configs`: Additional parameters for the `treatment_method`.
  If `treatment_method` is `"column_removal"`,  the key `treatment_threshold` can be used to define the fraction of
  missing values above which a column is flagged as a `null` column and remove.
  If `treatment_method` is `"MMM"`,  possible keys are the parameters of the
  [`imputation_MMM`](../api/data_analyzer/quality_checker.md#anovos.data_analyzer.quality_checker.imputation_MMM)
  function.

🤓  _Example:_

```yaml
nullColumns_detection:
  list_of_cols: all
  drop_cols: ['id_column', 'label_col']
  treatment: True
  treatment_method: MMM
  treatment_configs:
    method_type: median
    pre_existing_model: False
    model_path: NA
    output_mode: replace
```

## 📑 `association_evaluator`

🔎 _Corresponds to [`data_analyzer.association_evaluator`](../api/data_analyzer/association_evaluator.md)_

This block configures the association evaluator that focuses on understanding the
interaction between different attributes or the relationship between an attribute
and a binary target variable.

### `correlation_matrix`

🔎 _Corresponds to [`association_evaluator.correlation_matrix`](../api/data_analyzer/association_evaluator.md#anovos.data_analyzer.association_evaluator.correlation_matrix)_

- `list_of_cols`: List of column names (list of strings or string of column names separated by `|`)
  to include in the correlation matrix.
  Alternatively,  when set to `all`,  all columns are included.

- `drop_cols`: List of column names (list of strings or string of column names separated by `|`)
  to be excluded from the correlation matrix.
  This is especially useful when almost all columns should be included in the correlation matrix:
  Set `list_of_cols` to `all` and drop the few excluded columns.

🤓  _Example:_

```yaml
correlation_matrix:
  list_of_cols: all
  drop_cols: ['id_column']
```

### `IV_calculation`

🔎 _Corresponds to [`association_evaluator.IV_calculation`](../api/data_analyzer/association_evaluator.md#anovos.data_analyzer.association_evaluator.IV_calculation)_

- `list_of_cols`: List of column names (list of strings or string of column names separated by `|`)
  to include in the IV calculation.

- `drop_cols`: List of column names (list of strings or string of column names separated by `|`)
  to exclude from IV calculation.

- `label_col`: Name of label or target column in the input dataset.

- `event_label`: Value of event (label `1`/`true`) in the label column.

- `encoding_configs`: Detailed configuration of the binning step.

  - `bin_method`: The binning method. Defaults to `equal_frequency`.
  - `bin_size`: The bin size. Defaults to `10`.
  - `monotonicity_check`: If set to `1`,  dynamically computes the `bin_size` such that monotonicity is ensured.
    Can be a computationally expensive calculation. Defaults to `0`.

🤓  _Example:_

```yaml
IV_calculation:
  list_of_cols: all
  drop_cols: id_column
  label_col: label_col
  event_label: 'class1'
  encoding_configs:
    bin_method: equal_frequency
    bin_size: 10
    monotonicity_check: 0
```

### `IG_calculation`

🔎 _Corresponds to [`association_evaluator.IG_calculation`](../api/data_analyzer/association_evaluator.md#anovos.data_analyzer.association_evaluator.IG_calculation)_

- `list_of_cols`: List of column names (list of strings or string of column names separated by `|`)
  to consider for IG calculation.

- `drop_cols`: List of column names (list of strings or string of column names separated by `|`)
  to exclude from IG calculation.

- `label_col`: Name of label or target column in the input dataset.

- `event_label`: Value of event (label `1`/`true`) in the label column.

- `encoding_configs`: Detailed configuration of the binning step.

  - `bin_method`: The binning method. Defaults to `equal_frequency`.
  - `bin_size`: The bin size. Defaults to `10`.
  - `monotonicity_check`: If set to `1`,  dynamically computes the `bin_size` such that monotonicity is ensured.
    Can be a computationally expensive calculation. Defaults to `0`.

🤓  _Example:_

```yaml
IG_calculation:
  list_of_cols: all
  drop_cols: id_column
  label_col: label_col
  event_label: 'class1'
  encoding_configs:
    bin_method: equal_frequency
    bin_size: 10
    monotonicity_check: 0
```

### `variable_clustering`

🔎 _Corresponds to [`association_evaluator.variable_clustering`](../api/data_analyzer/association_evaluator.md#anovos.data_analyzer.association_evaluator.variable_clustering)_

- `list_of_cols`: List of column names (list of strings or string of column names separated by `|`)
  to include for variable clustering

- `drop_cols`: List of column names (list of strings or string of column names separated by `|`)
  to exclude from variable clustering.

🤓  _Example:_

```yaml
variable_clustering:
  list_of_cols: all
  drop_cols: id_column|label_col
```

## 📑 `drift_detector`

🔎 _Corresponds to [`drift.detector`](../api/drift/detector.md)_

This block configures the drift detector module that provides a range of methods
to detect drift within and between datasets.

### `drift_statistics`

🔎 _Corresponds to [`drift.detector.statistics`](../api/drift/detector.md#anovos.drift.detector.statistics)_

#### `configs`

- `list_of_cols`: List of columns to check drift (list or string of col names separated by `|`)
  to include in the drift statistics.
  Can be set to `all` to include all non-array columns (except those given in `drop_cols`).

- `drop_cols`: List of columns to be dropped (list or string of col names separated by `|`)
  to exclude from the drift statistics.

- `method_type`: Method(s) to apply to detect drift  (list or string of methods separated by `|`).
  Possible values are [`PSI`](https://www.listendata.com/2015/05/population-stability-index.html),
  [`JSD`](https://medium.com/datalab-log/measuring-the-statistical-similarity-between-two-samples-using-jensen-shannon-and-kullback-leibler-8d05af514b15),
  [`HD`](https://www.tifr.res.in/~prahladh/teaching/2011-12/comm/lectures/l12.pdf),
  and [`KS`](https://www.itl.nist.gov/div898/handbook/eda/section3/eda35g.htm).
  If set to `all`,  all available metrics are calculated.

- `threshold`: Threshold above which attributes are flagged as exhibiting drift.

- `bin_method`: The binning method.
  Possible values are `equal_frequency` and `equal_range`.

- `bin_size`: The bin size.
  We recommend setting it to `10` to `20` for `PSI` and above `100` for all other metrics.

- `pre_existing_source`: Set to `true` if a pre-computed binning model as well as frequency
  counts and attributes are available.
  `false` otherwise.

- `source_path`: If `pre_existing_source` is `true`,  this described from where the pre-computed
  data is loaded.

- `drift_statistics_folder`. drift_statistics folder must contain the output from `attribute_binning` and
  `frequency_counts`. If `pre_existing_source` is False,  this can be used for saving the details.
  Default folder "NA" is used for saving the intermediate output.

🤓  _Example:_

```yaml
configs:
  list_of_cols: all
  drop_cols: ['id_column', 'label_col']
  method_type: all
  threshold: 0.1
  bin_method: equal_range
  bin_size: 10
  pre_existing_source: False
  source_path: NA
```

#### `source_dataset`

The reference/baseline dataset.

##### `read_dataset`

- `file_path`: The file (or directory) path to read the source dataset from.
  It can be a local path,  an [📖 S3 path](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-bucket-intro.html)
  (when running on AWS),  a path to a file resource on Google Colab (see
  [📖 this tutorial](https://neptune.ai/blog/google-colab-dealing-with-files) for
  an overview),  or a path on the [📖 Databricks File System](https://docs.microsoft.com/de-de/azure/databricks/data/databricks-file-system)
  (when running on Azure).

- `file_type`: The file format of the source data. Currently,  _Anovos_ supports
  CSV (`csv`),  Parquet (`parquet`),  and Avro (`avro`).
  (Please note that if you're using Avro data sources,  you need to add the external
  package `org.apache.spark:spark-avro` when submitting the Spark job.)

- `file_configs` (optional): Options to pass to the respective Spark file reader,
  e.g.,  delimiters,  schemas,  headers.

##### `delete_column`

List of column names (list of strings or string of column names separated by `|`)
to be deleted from the loaded input data.

##### `select_column`

List of column names (list of strings or string of column names separated by `|`)
to be selected for further processing.

##### `rename_column`

- `list_of_cols`: List of the names of columns (list of strings or string of column names separated by `|`)
  to be renamed.

- `list_of_newcols`: The new column names. The first element in `list_of_cols` will be renamed
  to the first name in `list_of_newcols` and so on.

##### `recast_column`

- `list_of_cols`: List of the names of columns (list of strings or string of column names separated by `|`)
  to be cast to a different type.

- `list_of_dtypes`: The new datatypes. The first element in `list_of_cols` will be recast
  to the first type in `list_of_dtypes` and so on. See the
  [📖 Spark documentation](https://spark.apache.org/docs/latest/sql-ref-datatypes.html)
  for a list of valid datatypes.
  Note that this field is case-insensitive.

### `stability_index`

🔎 _Corresponds to [`detector.stability_index_computation`](../api/drift/detector.md#anovos.drift.detector.stability_index_computation)_

#### `configs`

- `metric_weightages`: A dictionary where the keys are the metric names (`mean`,  `stdev`,  `kurtosis`)
  and the values are the weight of the metric (between `0` and `1`).
  All weights must sum to `1`.

- `existing_metric_path`: Location of previously computed metrics of historical datasets
 (`idx`,  `attribute`,  `mean`,  `stdev`,  `kurtosis` where `idx` is index number of
 the historical datasets in chronological order).

- `appended_metric_path`: The path where the input dataframe metrics are saved after they
  have been appended to the historical metrics.

- `threshold`: The threshold above which attributes are flagged as unstable.

🤓  _Example:_

```yaml
configs:
  metric_weightages:
    mean: 0.5
    stddev: 0.3
    kurtosis: 0.2
  existing_metric_path: ''
  appended_metric_path: 'si_metrics'
  threshold: 2
```

#### `dataset1`

##### `read_dataset`

_Corresponds to [`data_ingest.read_dataset`](../api/data_ingest/data_ingest.md#anovos.data_ingest.data_ingest.read_dataset)_

- `file_path`: The file (or directory) path to read the other joining input dataset from.
  It can be a local path,  an [📖 S3 path](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-bucket-intro.html)
  (when running on AWS),  a path to a file resource on Google Colab (see
  [📖 this tutorial](https://neptune.ai/blog/google-colab-dealing-with-files) for
  an overview),  or a path on the
  [📖 Databricks File System](https://docs.microsoft.com/de-de/azure/databricks/data/databricks-file-system)
  (when running on Azure).

- `file_type`: The file format of the other joining input data. Currently,  _Anovos_ supports
  CSV (`csv`),  Parquet (`parquet`),  and Avro (`avro`).
  (Please note that if you're using Avro data sources,  you need to add the external
  package `org.apache.spark:spark-avro` when submitting the Spark job.)

- `file_configs` (optional): Options to pass to the respective Spark file reader,
  e.g.,  delimiters,  schemas,  headers.

#### `dataset2`,  `dataset3`,  …

Additional datasets are configured in the same manner as `dataset1`.

## 📑 `report_preprocessing`

🔎 _Corresponds to [`data_report.report_preprocessing`](../api/data_report/report_preprocessing.md)_

This configuration block describes the data pre–processing necessary for report generation.

### `master_path`

The path where all outputs are saved.

🤓  _Example:_

```yaml
master_path: 'report_stats'
```

### `charts_to_objects`

🔎 _Corresponds to [`report_preprocessing.charts_to_objects`](../api/data_report/report_preprocessing.md#anovos.data_report.report_preprocessing.charts_to_objects)_

This is the core function of the report preprocessing stage.
It saves the chart data in the form of objects that are used by the subsequent report generation scripts.

See the
[intermediate report documentation](../using-anovos/data-reports/intermediate_report.md) for more details.

- `list_of_cols`: List of column names (list of strings or string of column names separated by `|`)
  to include in preprocessing.

- `drop_cols`: List of column names (list of strings or string of column names separated by `|`)
  to exclude from preprocessing.

- `label_col`: Name of the label or target column in the input dataset.

- `event_label`: Value of the event (label `1`/`true`) in the label column.

- `bin_method`: The binning method.
  Possible values are `equal_frequency` and `equal_range`.

- `bin_size`: The bin size.
  We recommend setting it to `10` to `20` for `PSI` and above `100` for all other metrics.

- `drift_detector`: Indicates whether data drift has already analyzed. Defaults to `False`.

- `outlier_charts`: Indicates whether outlier charts should be included. Defaults to `False`.

- `source_path`: The source data path for drift analysis.
  If it has not been computed or is not required,  set it to the default value `"NA"`.

🤓  _Example:_

```yaml
charts_to_objects:
  list_of_cols: all
  drop_cols: id_column
  label_col: label_col
  event_label: 'class1'
  bin_method: equal_frequency
  bin_size: 10
  drift_detector: True
  outlier_charts: False
  source_path: "NA"
```

## 📑 `report_generation`

🔎 _Corresponds to [`data_report.report_generation`](../api/data_report/report_generation.md)_

This configuration block controls the generation of the actual report,  i.e.,  the data that is included and
the layout.
See the [report generation documentation](data-reports/final_report.md) for more details.

- `master_path`: The path to the preprocessed data generated during the [`report_preprocessing`](#report_preprocessing) step.

- `id_col`: The ID column present in the input dataset

- `label_col`: Name of label or target column in the input dataset.

- `corr_threshold`: The threshold above which attributes are considered to be correlated and thus,  redundant.
  Its value is between `0` and `1`.

- `iv_threshold`: The threshold above which attributes are considered ot be significant.
  Its value is between `0` and `1`.

| **Information Value** | **Variable's Predictiveness** |
|-----------------------|-------------------------------|
| <0.02                 | Not useful for prediction     |
| 0.02 to 0.1           | Weak predictive power         |
| 0.1 to 0.3            | Medium predictive power       |
| 0.3 to 0.5            | Strong predictive power       |
| >0.5                  | Suspicious predictive power   |

- `drift_threshold_model`: The threshold above which an attribute is flagged as exhibiting drift.
  Its value is between `0` and `1`.

- `dataDict_path`: The path to the data dictionary containing the exact names and definitions of the attributes.
  This information is used in the report to aid comprehensibility.

- `metricDict_path`: Path to the metric dictionary.

- `final_report_path`: The path where final report will be saved.
  It can be a local path,  an [📖 S3 path](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-bucket-intro.html)
  (when running on AWS),  a path to a file resource on Google Colab (see
  [📖 this tutorial](https://neptune.ai/blog/google-colab-dealing-with-files) for
  an overview),  or a path on the
  [📖 Databricks File System](https://docs.microsoft.com/de-de/azure/databricks/data/databricks-file-system)
  (when running on Azure).

🤓  _Example:_

```yaml
report_generation:
  master_path: 'report_stats'
  id_col: 'id_column'
  label_col: 'label_col'
  corr_threshold: 0.4
  iv_threshold: 0.02
  drift_threshold_model: 0.1
  dataDict_path: 'data/income_dataset/data_dictionary.csv'
  metricDict_path: 'data/metric_dictionary.csv'
  final_report_path: 'report_stats'
```

## 📑 `transformers`

🔎 _Corresponds to [`data_transformer.transformers`](../api/data_transformer/transformers.md)_

This block configures the [`data_transformer`](../api/data_transformer/_index.md) module that supports
numerous pre-processing and transformation functions,  such as binning,  encoding,  scaling,  and imputation.

### `numerical_mathops`

This group of functions is used to perform mathematical transformations of numerical attributes.

#### `feature_transformation`

🔎 _Corresponds to [`transformers.feature_transformation`](../api/data_transformer/transformers.md#anovos.data_transformer.transformers.feature_transformation)_

- `list_of_cols`: The numerical columns (list of strings or string of column names separated by `|`)
  to transform.
  Can be set to `"all"` to include all numerical columns.

- `drop_cols`: The numerical columns (list of strings or string of column names separated by `|`)
  to exclude from feature transformation.

- `method_type`: The method to apply to use for transformation.
  The default method is `sqrt` ($\sqrt{x}$).
  Possible values are:
  - `ln`
  - `log10`
  - `log2`
  - `exp`
  - `powOf2` ($2^x$)
  - `powOf10` ($10^x$)
  - `powOfN` ($N^x$)Z
  - `sqrt` ($\sqrt{x}$)
  - `cbrt` ($\sqrt[3]{x}$)
  - `sq` ($x^2$)
  - `cb` ($x^3$)
  - `toPowerN` ($x^N$)
  - `sin`
  - `cos`
  - `tan`
  - `asin`
  - `acos`
  - `atan`
  - `radians`
  - `remainderDivByN` ($x % N$)
  - `factorial` ($x!$)
  - `mul_inv` ($1/x$)
  - `floor`
  - `ceil`
  - `roundN` (round to `N` decimal places)

- `N`: `None` by default. If `method_type` is `powOfN`,  `toPowerN`,  `remainderDivByN`,  or `roundN`,
  `N` will be used as the required constant.

🤓  _Example 1:_

```yaml
feature_transformation:
  list_of_cols: all
  drop_cols: []
  method_type: sqrt
```

🤓  _Example 2:_

```yaml
feature_transformation:
  list_of_cols: ['capital-gain', 'capital-loss']
  drop_cols: []
  method_type: sqrt

feature_transformation:
  list_of_cols: ['age','education_num']
  drop_cols: []
  method_type: sq
```

#### `boxcox_transformation`

🔎 _Corresponds to [`transformers.boxcox_transformation`](../api/data_transformer/transformers.md#anovos.data_transformer.transformers.boxcox_transformation)_

- `list_of_cols`:  The columns (list of strings or string of column names separated by `|`) to transform.
  Can be set to `"all"` to include all columns.

- `drop_cols`: The columns (list of strings or string of column names separated by `|`)
  to exclude from Box-Cox transformation.

- `boxcox_lambda`: The $\lambda$ value for the Box-Cox transformation.
  It can be given as a
  - list where each element represents the value of $\lambda$ for a single attribute.
  The length of the list must be the same as the number of columns to transform.
  - number that is used for all attributes.
  If no value is given (the default),  a search for the best $\lambda$ will be conducted among the following values:
  `[1, -1, 0.5, -0.5, 2, -2, 0.25, -0.25, 3, -3, 4, -4, 5, -5]`.
  The search is conducted independently for each column.

🤓  _Example 1:_

```yaml
boxcox_transformation:
  list_of_cols: num_feature1|num_feature2
  drop_cols: []
```

🤓  _Example 2:_

```yaml
boxcox_transformation:
  list_of_cols: num_feature3|num_feature4
  drop_cols: []
  boxcox_lambda: [-2, -1]
```

### `numerical_binning`

This group of functions is used to transform numerical attributes into discrete (integer or categorical) attribute.

#### `attribute_binning`

🔎 _Corresponds to [`transformers.attribute_binning`](../api/data_transformer/transformers.md#anovos.data_transformer.transformers.attribute_binning)_

- `list_of_cols`:  The numerical columns (list of strings or string of column names separated by `|`) to transform.
  Can be set to `"all"` to include all numerical columns.

- `drop_cols`: The columns (list of strings or string of column names separated by `|`)
  to exclude from attribute binning.

- `method_type`: The binning method. Possible values are `equal_frequency` and `equal_range`.
  With `equal_range`, each bin is of equal size/width and with `equal_frequency`,
  each bin contains an equal number of rows.
  Defaults to `equal_range`.

- `bin_size`: The number of bins. Defaults to `10`.

- `bin_dtype`: The `dtype` of the transformed column. Possible values are `numerical` and `categorical`.
  With `numerical`, the values reflect the bin number (`1`, `2`, …).
  With `categorical` option, the values are a string that describes the minimal and maximal value of the bin.
  Defaults to `numerical`.

🤓  _Example:_

```yaml
attribute_binning:
  list_of_cols: num_feature1|num_feature2
  drop_cols: []
  method_type: equal_frequency
  bin_size: 10
  bin_dtype: numerical
```

#### `monotonic_binning`

🔎 _Corresponds to [`transformers.monotonic_binning`](../api/data_transformer/transformers.md#anovos.data_transformer.transformers.monotonic_binning)_

- `list_of_cols`: The numerical columns (list of strings or string of column names separated by `|`) to transform.
  Can be set to `"all"` to include all numerical columns.

- `drop_cols`: The columns (list of strings or string of column names separated by `|`)
  to exclude from monotonic binning.

- `method_type`: The binning method. Possible values are `equal_frequency` and `equal_range`.
  With `equal_range`, each bin is of equal size/width and with `equal_frequency`,
  each bin contains an equal number of rows.
  Defaults to `equal_range`.

- `bin_size`: The number of bins. Defaults to `10`.

- `bin_dtype`: The `dtype` of the transformed column. Possible values are `numerical` and `categorical`.
  With `numerical`, the values reflect the bin number (`1`, `2`, …).
  With `categorical` option, the values are a string that describes the minimal and maximal value of the bin.
  Defaults to `numerical`.

🤓  _Example:_

```yaml
attribute_binning:
  list_of_cols: num_feature1|num_feature2
  drop_cols: []
  label_col: ["label_col"]
  event_label: ["class1"]
  method_type: equal_frequency
  bin_size: 10
  bin_dtype: numerical
```

### `numerical_expression`

#### `expression_parser`

🔎 _Corresponds to [`transformers.expression_parser`](../api/data_transformer/transformers.md#anovos.data_transformer.transformers.expression_parser)_

This function can be used to evaluate a list of SQL expressions and output the result as new features.
Columns used in the SQL expression must be available in the dataset.

- `list_of_expr`: List of expressions to evaluate as new features e.g.,  ["expr1", "expr2"].
  Alternatively, expressions can be specified in a string format, where different expressions are separated
  by pipe delimiter “|” e.g.,  "expr1|expr2".

- `postfix`: postfix for new feature name.Naming convention "f" + expression_index + postfix e.g. with postfix
  of "new",  new added features are named as f0new,  f1new etc. (Default value = "").

🤓  _Example 1:_

```yaml
expression_parser:
  list_of_expr: 'log(age) + 1.5|sin(capital-gain)+cos(capital-loss)'
```

🤓  _Example 2:_

```yaml
expression_parser:
  list_of_expr: ['log(age) + 1.5', 'sin(capital-gain)+cos(capital-loss)']
```

Both _Example 1_ and _Example 2_ generate 2 new features: _log(age) + 1.5_ and _sin(capital-gain)+cos(capital-loss)_.
The newly generated features will be appended to the dataframe as new columns: f0 and f1.

### `categorical_outliers`

This function assigns less frequently seen values in a categorical column to a new category `others`.

#### `outlier_categories`

🔎 _Corresponds to [`transformers.outlier_categories`](../api/data_transformer/transformers.md#anovos.data_transformer.transformers.outlier_categories)_

- `list_of_cols`: The categorical columns (list of strings or string of column names separated by `|`) to transform.
  Can be set to `"all"` to include all categorical columns.

- `drop_cols`: The columns (list of strings or string of column names separated by `|`)
  to exclude from outlier transformation.

- `coverage`: The minimum fraction of rows that remain in their original category, given as a value between `0` and `1`.
  For example, with a coverage of `0.8`, the categories that 80% of the rows belong to remain and the more seldom
  occurring categories are mapped to `others`.
  The default value is `1.0`, which means that no rows are changed to `others`.

- `max_category`: Even if coverage is less, only (max_category - 1) categories will be mapped to actual name and
  rest to others. Caveat is when multiple categories have same rank, then #categories can be more than max_category.
  Defaults to `50`.

🤓  _Example 1:_

```yaml
outlier_categories:
  list_of_cols: all
  drop_cols: ['id_column', 'label_col']
  coverage: 0.9
  max_category: 20
```

🤓  _Example 2:_

```yaml
outlier_categories:
  list_of_cols: ["cat_feature1", "cat_feature2"]
  drop_cols: []
  coverage: 0.8
  max_category: 10

outlier_categories:
  list_of_cols: ["cat_feature3", "cat_feature4"]
  drop_cols: []
  coverage: 0.9
  max_category: 15
```

### `categorical_encoding`

This group of transformers functions used to converting a categorical attribute into numerical attribute(s).

#### `cat_to_num_unsupervised`

🔎 _Corresponds to [`transformers.cat_to_num_unsupervised`](../api/data_transformer/transformers.md#anovos.data_transformer.transformers.cat_to_num_unsupervised)_

- `list_of_cols`: The categorical columns (list of strings or string of column names separated by `|`) to encode.
  Can be set to `"all"` to include all categorical columns.

- `drop_cols`: The columns (list of strings or string of column names separated by `|`)
  to exclude from categorical encoding.

- `method_type`: The encoding method. Set to `1` for label encoding and to `0` for one-hot encoding.
  With label encoding, each categorical value is assigned a unique integer based on the ordering specified through `index_order`.
  With one-hot encoding, each categorical value will be represented by a binary column.
  Defaults to `1` (label encoding).

- `index_order`: The order assigned to the categorical values when `method_type` is set to `1` (label encoding).
  Possible values are:
  - `frequencyDesc` (default): Order by descending frequency.
  - `frequencyAsc`: Order by ascending frequency.
  - `alphabetDesc`: Order alphabetically (descending).
  - `alphabetAsc`: Order alphabetically (ascending).

- `cardinality_threshold`: Columns with a cardinality above this threshold are excluded from enconding.
  Defaults to `100`.

🤓  _Example 1:_

```yaml
cat_to_num_unsupervised:
  list_of_cols: all
  drop_cols: ['id_column']
  method_type: 0
  cardinality_threshold: 10
```

🤓  _Example 2:_

```yaml
cat_to_num_unsupervised:
  list_of_cols: ["cat_feature1", "cat_feature2"]
  drop_cols: []
  method_type: 0
  cardinality_threshold: 10

cat_to_num_unsupervised:
  list_of_cols: ["cat_feature3", "cat_feature4"]
  drop_cols: []
  method_type: 1
```

#### `cat_to_num_supervised`

🔎 _Corresponds to [`transformers.cat_to_num_supervised`](../api/data_transformer/transformers.md#anovos.data_transformer.transformers.cat_to_num_supervised)_

- `list_of_cols`: The categorical columns (list of strings or string of column names separated by `|`) to encode.
  Can be set to `"all"` to include all categorical columns.

- `drop_cols`: The columns (list of strings or string of column names separated by `|`)
  to exclude from categorical encoding.

- `label_col`: The label/target column. Defaults to `label`.

- `event_label`: Value of the (positive) event (i.e, label `1`/`true`). Defaults to `1`.

🤓  _Example:_

```yaml
cat_to_num_supervised:
  list_of_cols: cat_feature1 | cat_feature2
  drop_cols: ['id_column']
  label_col: income
  event_label: '>50K'
```

### `numerical_rescaling`

Group of functions to rescale numerical attributes.

#### `normalization`

🔎 _Corresponds to [`transformers.normalization`](../api/data_transformer/transformers.md#anovos.data_transformer.transformers.normalization)_

- `list_of_cols`: The numerical columns (list of strings or string of column names separated by `|`) to normalize.
  Can be set to `"all"` to include all numerical columns.

- `drop_cols`: The columns (list of strings or string of column names separated by `|`)
  to exclude from normalization.

🤓  _Example:_

```yaml
normalization:
  list_of_cols: ["num_feature1", "num_feature2"]
  drop_cols: []
```

#### `z_standardization`

🔎 _Corresponds to [`transformers.z_standardization`](../api/data_transformer/transformers.md#anovos.data_transformer.transformers.z_standardization)_

- `list_of_cols`: The numerical columns (list of strings or string of column names separated by `|`) to standardize.
  Can be set to `"all"` to include all numerical columns.

- `drop_cols`: The columns (list of strings or string of column names separated by `|`)
  to exclude from standardization.

🤓  _Example:_

```yaml
z_standardization:
  list_of_cols: ["num_feature1", "num_feature2"]
  drop_cols: []
```

#### `IQR_standardization`

🔎 _Corresponds to [`transformers.IQR_standardization`](../api/data_transformer/transformers.md#anovos.data_transformer.transformers.IQR_standardization)_

- `list_of_cols`: The numerical columns (list of strings or string of column names separated by `|`) to standardize.
  Can be set to `"all"` to include all numerical columns.

- `drop_cols`: The columns (list of strings or string of column names separated by `|`)
  to exclude from standardization.

🤓  _Example:_

```yaml
IQR_standardization:
  list_of_cols: ["num_feature1", "num_feature2", "num_feature3"]
  drop_cols: []
```

### `numerical_latentFeatures`

Group of functions to generate latent features to reduce the dimensionality of the input dataset.

#### `PCA_latentFeatures`

🔎 _Corresponds to [`transformers.PCA_latentFeatures`](../api/data_transformer/transformers.md#anovos.data_transformer.transformers.PCA_latentFeatures)_

- `list_of_cols`:  The numerical columns (list of strings or string of column names separated by `|`) to standardize.
  Can be set to `"all"` to include all numerical columns.

- `drop_cols`: The columns (list of strings or string of column names separated by `|`)
  to exclude from latent features computation.

- `explained_variance_cutoff`: The required explained variance cutoff.
  Determines the number of encoded columns in the output.
  If `N` is the smallest integer such that the top `N` encoded columns explain more than the given variance threshold,
  these `N` columns be selected.
  Defaults to `0.95`.

- `standardization`:  If `True` (the default), standardization is applied. `False` otherwise.

- `standardization_configs`: The arguments for the
  [`z_standardization`](../api/data_transformer/transformers.md#anovos.data_transformer.transformers.z_standardization)
  function in dictionary format.
  Defaults to `{"pre_existing_model": False}`

- `imputation`: If `True`, imputation is applied. `False` (the default) otherwise.

- `imputation_configs`: Configuration for imputation in dictionary format.
  The name of the imputation function is specified with the key `imputation_name` (defaults to `imputation_MMM`).
  Arguments for the imputation function can be passed using additional keys.

🤓  _Example 1:_

```yaml
PCA_latentFeatures:
  list_of_cols: ["num_feature1", "num_feature2", "num_feature3"]
  explained_variance_cutoff: 0.95
  standardization: False
  imputation: True
```

🤓  _Example 2:_

```yaml
PCA_latentFeatures:
  list_of_cols: ["num_feature1", "num_feature2", "num_feature3"]
  explained_variance_cutoff: 0.8
  standardization: False
  imputation: True

PCA_latentFeatures:
  list_of_cols: ["num_feature4", "num_feature5", "num_feature6"]
  explained_variance_cutoff: 0.6
  standardization: True
  imputation: True
```

#### `autoencoder_latentFeatures`

🔎 _Corresponds to [`transformers.autoencoder_latentFeatures`](../api/data_transformer/transformers.md#anovos.data_transformer.transformers.autoencoder_latentFeatures)_

- `list_of_cols`:  The numerical columns (list of strings or string of column names separated by `|`) to standardize.
  Can be set to `"all"` to include all numerical columns.

- `drop_cols`: The columns (list of strings or string of column names separated by `|`)
  to exclude from latent features computation.

- `reduction_params`: Determines the number of resulting encoded features.
  If `reduction_params` is below `1`, `reduction_params * "number of columns"` columns will be generated.
  Else, `reduction_params` columns will be generated.
  Defaults to `0.5`, i.e., the number of columns in the result is half the of number of columns in the input.

- `sample_size`: Maximum number of rows used for training the autoencoder model. Defaults to `500000` (`5e5`).

- `epochs`: The number of epochs to train the autoencoder model. Defaults to `100`.

- `batch_size`: The batch size for autoencoder model training. Defaults to `256`.

- `standardization`:  If `True` (the default), standardization is applied. `False` otherwise.

- `standardization_configs`: The arguments for the
  [`z_standardization`](../api/data_transformer/transformers.md#anovos.data_transformer.transformers.z_standardization)
  function in dictionary format.
  Defaults to `{"pre_existing_model": False}`

- `imputation`: If `True`, imputation is applied. `False` (the default) otherwise.

- `imputation_configs`: Configuration for imputation in dictionary format.
  The name of the imputation function is specified with the key `imputation_name` (defaults to `imputation_MMM`).
  Arguments for the imputation function can be passed using additional keys.

🤓  _Example 1:_

```yaml
autoencoder_latentFeatures:
  list_of_cols: ["num_feature1", "num_feature2", "num_feature3"]
  reduction_params: 0.5
  sample_size: 10000
  epochs: 20
  batch_size: 256
```

🤓  _Example 2:_

```yaml
autoencoder_latentFeatures:
  list_of_cols: ["num_feature1", "num_feature2"]
  reduction_params: 0.5
  sample_size: 10000
  epochs: 20
  batch_size: 256

autoencoder_latentFeatures:
  list_of_cols: ["num_feature3", "num_feature4", "num_feature5", "num_feature6", "num_feature7"]
  reduction_params: 0.8
  sample_size: 10000
  epochs: 100
  batch_size: 256
```

## 📑 `write_intermediate`

- `file_path`: Path where intermediate datasets (after selecting, dropping, renaming, and recasting of columns)
  for quality checker operations,  join dataset and concatenate dataset will be saved.

- `file_type`: (CSV,  Parquet or Avro). file format of intermediate dataset

- `file_configs` (optional): Rest of the valid configuration can be passed through this options e.g., repartition,
  mode, compression, header, delimiter, inferSchema etc. This might look like:

  🤓  _Example:_

  ```yaml
  file_configs:
     mode: overwrite
     header: True
     delimiter: ", "
     inferSchema: True
  ```

  For more information on available configuration options,  see the following external
  documentation:

  - [📖 Writing CSV files](https://sparkbyexamples.com/pyspark/pyspark-read-csv-file-into-dataframe/)
  - [📖 Writing Parquet files](https://sparkbyexamples.com/pyspark/pyspark-read-and-write-parquet-file/)
  - [📖 Writing Avro files](https://sparkbyexamples.com/spark/read-write-avro-file-spark-dataframe/)

## 📑 `write_main`

- `file_path`: Path where final cleaned input dataset will be saved.

- `file_type`: (CSV,  Parquet or Avro). file format of final dataset

- `file_configs` (optional): Rest of the valid configuration can be passed through this options e.g.,
  repartition,  mode,  compression,  header,  delimiter,  inferSchema etc. This might look like:

  🤓  _Example:_

  ```yaml
  file_configs:
       mode: overwrite
       header: True
       delimiter: ", "
       inferSchema: True
  ```

  For more information on available configuration options, see the following external
  documentation:

  - [📖 Writing CSV files](https://sparkbyexamples.com/pyspark/pyspark-read-csv-file-into-dataframe/)
  - [📖 Writing Parquet files](https://sparkbyexamples.com/pyspark/pyspark-read-and-write-parquet-file/)
  - [📖 Writing Avro files](https://sparkbyexamples.com/spark/read-write-avro-file-spark-dataframe/)

## 📑 `write_stats`

- `file_path`: Path where all tables/stats of anovos modules (data drift & data analyzer) will be saved.

- `file_type`: (CSV,  Parquet or Avro). file format of final dataset

- `file_configs` (optional): Rest of the valid configuration can be passed through this options e.g., repartition,
  mode,  compression,  header,  delimiter,  inferSchema etc. This might look like:

  ```yaml
  file_configs:
     mode: overwrite
     header: True
     delimiter: ", "
     inferSchema: True
  ```

  For more information on available configuration options,  see the following external
  documentation:

  - [📖 Writing CSV files](https://sparkbyexamples.com/pyspark/pyspark-read-csv-file-into-dataframe/)
  - [📖 Writing Parquet files](https://sparkbyexamples.com/pyspark/pyspark-read-and-write-parquet-file/)
  - [📖 Writing Avro files](https://sparkbyexamples.com/spark/read-write-avro-file-spark-dataframe/)

## 📑 `write_feast_features`

🔎 _Corresponds to [`feature_store.feast_exporter`](../api/feature_store/feast_exporter.md)_

📖 _For details, see the_
_[Feature Store Integration documentation](https://sparkbyexamples.com/pyspark/pyspark-read-csv-file-into-dataframe/)_

- `file_path`:

- `entity`:
    - `name`:
    - `description`:
    - `id_col`:

- `file_source`:
    - `description`:
    - `owner`:
    - `timestamp_col`:
    - `create_timestamp_col`:

- `feature_view`:
    - `name`:
    - `owner`:
    - `ttl_in_seconds`:
    - `create_timestamps`:

- `feature_service` (optional):
    - `name`:

- `file_configs`:
    - `mode`:

🤓  _Example:_
