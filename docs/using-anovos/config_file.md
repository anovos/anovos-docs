# Configuring Workloads

_Anovos_ workloads can be described by a YAML configuration file.

Such a configuration file defines:

- the input dataset(s)
- the analyses and transformations to be performed on the data
- the output files and dataset(s)
- the reports to be generated

Defining workloads this way allows users to make full use of _Anovos_ capabilities
while maintaining an easy-to-grasp overview.
Since each configuration file fully describes one workload, these files can be 
shared, versioned, and run across different compute environments.

In the following, we'll describe in detail each of the sections in an _Anovos_ 
configuration file.
If you'd rather see a full example right away, have a look at 
[this example](https://github.com/anovos/anovos/blob/main/config/configs.yaml).

## `input_dataset`

This configuration block describes how the input dataset is loaded and prepared.
Each _Anovos_ configuration file must contain exactly one `input_dataset` block.

Note that the subsequent operations are performed in the order given here:
First, columns are deleted, then selected, then renamed, and then recast.

### `read_dataset`

- `file_path`: The file (or directory) path to read the input dataset from.
   It can be a local path, an [S3 path](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-bucket-intro.html)
  (when running on AWS), a path to a file resource on Google Colab (see
   [this tutorial](https://neptune.ai/blog/google-colab-dealing-with-files)for
   an overview), or a path on the [Databricks File System](https://docs.microsoft.com/de-de/azure/databricks/data/databricks-file-system)
   (when running on Azure).

- `file_type`: The file format of the input data. Currently, _Anovos_ supports
   CSV (`csv`), Parquet (`parquet`), and Avro (`avro`).
   (Please note that if you're using Avro data sources, you need to add the external
   package `org.apache.spark:spark-avro` when submitting the Spark job.)

- `file_configs` (optional): Options to pass to the respective Spark file reader,
   e.g., delimiters, schemas, headers. In the case of a CSV file, this might look
   like:
   ```yaml
   file_configs:
     delimiter: ","
     header: True
     inferSchema: True
   ```
   For more information on available configuration options, see the following external
   documentation:
   
     - [Read CSV files](https://sparkbyexamples.com/pyspark/pyspark-read-csv-file-into-dataframe/)
     - [Read Parquet files](https://sparkbyexamples.com/pyspark/pyspark-read-and-write-parquet-file/)
     - [Read Avro files](https://sparkbyexamples.com/spark/read-write-avro-file-spark-dataframe/)
  
### `delete_column`

List of column names (list of strings or string of column names separated by `|`)
to be deleted from the loaded input data.

_Example:_
```yaml
delete_column: ['unnecessary', 'obsolete', 'outdated']
```

### `select_column`

List of column names (list of strings or string of column names separated by `|`)
to be selected for further processing.

_Example:_
```yaml
select_column: ['feature1', 'feature2', 'feature3', 'label']
```

### `rename_column`

- `list_of_cols`: List of the names of columns (list of strings or string of column names separated by `|`)
  to be renamed.

- `list_of_newcols`: The new column names. The first element in `list_of_cols` will be renamed
  to the first name in `list_of_newcols` and so on.

_Example:_
```yaml
rename_column:
  list_of_cols: ['very_long_column_name', 'price']
  list_of_newcols: ['short_name', 'label']
```

This will rename the column `very_long_column_name` to `short_name` and the column `price` to `label`.

### `recast_column`

- `list_of_cols`: List of the names of columns (list of strings or string of column names separated by `|`)
  to be cast to a different type.

- `list_of_dtypes`: The new datatypes. The first element in `list_of_cols` will be recast
  to the first type in `list_of_dtypes` and so on. See
  [the Spark documentation](https://spark.apache.org/docs/latest/sql-ref-datatypes.html)
  for a list of valid datatypes.
  Note that this field is case-insensitive.

_Example:_
```yaml
recast_column:
  list_of_cols: ['price', 'quantity']
  list_of_dtypes: ['double', 'int']
```

## `concatenate_dataset`

This configuration block describes how to combine multiple dataframes into a single dataframe.
There can be varying number of input dataframes that can be concatenated and for loading other
concatenating input datasets, we need to write its subsequent operations separately 
for each dataset like `dataset1`, `dataset2`, `dataset3` and so on.

### `method`

`index` or `name`. This needs to be entered as a keyword argument.
The "index" method involves concatenating the dataframes by the column index.
If the sequence of column is not fixed among the dataframe, this method should be avoided.
The "name" method involves concatenating by column names.

The first dataframe passed will define the final columns in the concatenated dataframe.
It will throw an error if any column in the first dataframe is not available in any of other dataframes.

_Example:_
```yaml
method: name
```

### `dataset1`

#### `read_dataset`

- `file_path`: The file (or directory) path to read the other concatenating input dataset from.
   It can be a local path, an [S3 path](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-bucket-intro.html)
   (when running on AWS), a path to a file resource on Google Colab (see
   [this tutorial](https://neptune.ai/blog/google-colab-dealing-with-files) for
   an overview), or a path on the
   [Databricks File System](https://docs.microsoft.com/de-de/azure/databricks/data/databricks-file-system)
   (when running on Azure).

- `file_type`: The file format of the other concatenating input data. Currently, _Anovos_ supports
   CSV (`csv`), Parquet (`parquet`), and Avro (`avro`).
   (Please note that if you're using Avro data sources, you need to add the external
   package `org.apache.spark:spark-avro` when submitting the Spark job.)

- `file_configs` (optional): Options to pass to the respective Spark file reader,
   e.g., delimiters, schemas, headers.
  
#### `delete_column`

List of column names (list of strings or string of column names separated by `|`)
to be deleted from the loaded input data.

#### `select_column`

List of column names (list of strings or string of column names separated by `|`)
to be selected for further processing.

#### `rename_column`

- `list_of_cols`: List of the names of columns (list of strings or string of column names separated by `|`)
  to be renamed.

- `list_of_newcols`: The new column names. The first element in `list_of_cols` will be renamed
  to the first name in `list_of_newcols` and so on.

#### `recast_column`

- `list_of_cols`: List of the names of columns (list of strings or string of column names separated by `|`)
  to be cast to a different type.

- `list_of_dtypes`: The new datatypes. The first element in `list_of_cols` will be recast
  to the first type in `list_of_dtypes` and so on. See
  [the Spark documentation](https://spark.apache.org/docs/latest/sql-ref-datatypes.html)
  for a list of valid datatypes.
  Note that this field is case-insensitive.

### `dataset2`, `dataset3`, ...

Additional datasets are configured in the same manner as `dataset1`.

## `join_dataset`

This configuration block describes how multiple dataframes are joined into a single dataframe by a key column.

There can be a varying number of input dataframes.
Each dataset to be joined is configured separately as `dataset1`, `dataset2`, and so on.

### `Join_cols`

Key column(s) to join all dataframes together.
In the case that the key consists of multiple columns, they can be passed as a list of strings or
a single string where the column names are separated by `|`.

_Example:_
```yaml
join_cols: id_column
```

### `join_type`

The type of join: `inner`, `full`, `left`, `right`, `left_semi`, `left_anti`

For an introduction, see [this tutorial](https://sparkbyexamples.com/spark/spark-sql-dataframe-join/).

_Example:_
```yaml
join_type: inner
```

### `dataset1`

#### `read_dataset`

- `file_path`: The file (or directory) path to read the other joining input dataset from.
   It can be a local path, an [S3 path](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-bucket-intro.html)
  (when running on AWS), a path to a file resource on Google Colab (see
   [this tutorial](https://neptune.ai/blog/google-colab-dealing-with-files)for
   an overview), or a path on the
  [Databricks File System](https://docs.microsoft.com/de-de/azure/databricks/data/databricks-file-system)
   (when running on Azure).

- `file_type`: The file format of the other joining input data. Currently, _Anovos_ supports
   CSV (`csv`), Parquet (`parquet`), and Avro (`avro`).
   (Please note that if you're using Avro data sources, you need to add the external
   package `org.apache.spark:spark-avro` when submitting the Spark job.)

- `file_configs` (optional): Options to pass to the respective Spark file reader,
   e.g., delimiters, schemas, headers.
  
#### `delete_column`

List of column names (list of strings or string of column names separated by `|`)
to be deleted from the loaded input data.

#### `select_column`

List of column names (list of strings or string of column names separated by `|`)
to be selected for further processing.

#### `rename_column`

- `list_of_cols`: List of the names of columns (list of strings or string of column names separated by `|`)
  to be renamed.

- `list_of_newcols`: The new column names. The first element in `list_of_cols` will be renamed
  to the first name in `list_of_newcols` and so on.

#### `recast_column`

- `list_of_cols`: List of the names of columns (list of strings or string of column names separated by `|`)
  to be cast to a different type.

- `list_of_dtypes`: The new datatypes. The first element in `list_of_cols` will be recast
  to the first type in `list_of_dtypes` and so on. See
  [the Spark documentation](https://spark.apache.org/docs/latest/sql-ref-datatypes.html)
  for a list of valid datatypes.
  Note that this field is case-insensitive.

### `dataset2`, `dataset3`, ...

Additional datasets are configured in the same manner as `dataset1`.

## `timeseries_analyzer`

Configuration for the  [time series analyzer](../api/data_analyzer/ts_analyzer.md).

- `auto_detection`: Can be set to `True` or `False`.
  If `True`, it attempt to automatically infer the date/timestamp format in the input dataset.

- `id_col`: Name of ID column in the input dataset.

- `tz_offset`: The timezone offset of the timestamps in the input dataset.
   Can be set to either `local`, `gmt`, or `utc`. The default setting is `local`.

- `inspection`: Can be set to `True` or `False`. If `True`, the time series elements undergo an inspection.

- `analysis_level`: Can be set to `daily`, `weekly`, or `hourly`. The default setting is `daily`.
   If set to `daily`, the daily view is populated.
   If set to `hourly`, the view is shown at a day part level.  
   If set to `weekly`, the display it per individual weekdays (1-7) as captured.

- `max_days`: Maximum number of days up to which the data will be aggregated.
   If the dataset contains a timestamp/date field with very high number of unique dates
   (e.g., 20 years worth of daily data), this option can be used to reduce the timespan that is analyzed.

_Example:_
```yaml
auto_detection: True
id_col: 'id_column'
tz_offset: 'local'
inspection: True
analysis_level: 'daily'
max_days: 3600
```

## `anovos_basic_report`

Anovos basic report consists of basic report which is generated after completion of data analyzer, association evaluator and quality checker modules. See the [Data Report Doc](https://github.com/anovos/anovos-docs/tree/main/docs/using-anovos/data-reports) for more details. In this basic report user will get basic descriptive stats and count  like descriptive statistics(global_summary, measures_of_count, measures_of_centralTendency, measures_of_cardinality, measures_of_dispersion, measures_of_percentiles, measures_of_shape), quality checker(nullRows_detection, nullColumns_detection, duplicate_detection, IDness_detection, biasedness_detection, invalidEntries_detection, outlier_detection), attribute association (correlation_matrix, IV_calculation, IG_calculation, variable_clustering). If user does not want basic report to be generated, they can keep basic report equals to False. 

### `basic_report`

This takes Boolean type input -- `True` or `False`. If True, basic report is generated after completion of data analyzer, association evaluator and quality checker modules. If False, basic report is not generated after completion of data analyzer, association evaluator and quality checker modules. But all the stats and counts are avaialble in final report independent of the basic report option.

### `report_args`

- `Id_col`: Name of Id column in the input dataset

- `Label_col`: Name of label or target column in the input dataset

- `Event_lable`: Value of event (label 1) in the label column

- `Output_path`: Path where basic report is saved. File path can be a local path or s3 path (when running with AWS cloud services)

_Example:_
```yaml
report_args:
  id_col: id_column
  label_col: label_col
  event_label: 'class1'
  output_path: report_stats
```

## `stats_generator`

This module generates all the descriptive statistics related to the ingested data. Descriptive statistics are split into different metric types, and each function corresponds to one metric type. - global_summary - measures_of_counts - measures_of_centralTendency - measures_of_cardinality - measures_of_dispersion - measures_of_percentiles - measures_of_shape. See the 
[Stats Generator Doc](https://github.com/anovos/anovos-docs/blob/main/docs/api/data_analyzer/stats_generator.md) for understanding its functions in detail.
### `Metric`

list of different metrics used to generate descriptive statistics [global_summary, measures_of_count, measures_of_centralTendency, measures_of_cardinality, measures_of_dispersion, measures_of_percentiles, measures_of_shape]

_Example:_
```yaml
metric: ['global_summary','measures_of_counts','measures_of_centralTendency','measures_of_cardinality',
          'measures_of_percentiles','measures_of_dispersion','measures_of_shape']
```

### `Metric_args`

- `List_of_cols`: (list format or string of col names separated by `|`). It is used to specify the columns which are subjected to the analysis in the input dataframe. The user can also use "all" as an input to this argument to consider all columns. This is super useful instead of specifying all column names manually.

- `Drop_cols`: (list format or string of col names separated by `|`). It is used to specify the columns that need to be dropped from list_of_cols. It is most useful when coupled with the `all` value of list_of_cols, when we need to consider all columns except a few handful of them.

_Example:_
```yaml
metric_args:
  list_of_cols: all
  drop_cols: ['id_column']
```

## `quality_checker`

This submodule focuses on assessing the data quality at both row-level and column-level and also provides an appropriate treatment option to fix quality issues. see the
[Quality Checker Dic](https://github.com/anovos/anovos-docs/blob/main/docs/api/data_analyzer/quality_checker.md) for understanding its functions in detail.
### `duplicate_detection`

- `list_of_cols`: (list format or string of col names separated by `|`). It is used to specify the columns which are subjected to the duplicate detection

- `drop_cols`: (list format or string of col names separated by `|`). It is used to specify the columns that need to be dropped from list_of_cols before duplicate detection

- `treatment`: It takes Boolean type input -- `True` or `False`. If true, duplicate rows are removed from the input dataset.

_Example:_
```yaml
duplicate_detection:
  list_of_cols: all
  drop_cols: ['id_column']
  treatment: True
```

### `nullRows_detection`

- `list_of_cols`: (list format or string of col names separated by `|`). It is used to specify the columns which are subjected to the null rows detection

- `drop_cols`: (list format or string of col names separated by `|`). It is used to specify the columns that need to be dropped from list_of_cols before null rows detection

- `treatment`: This takes Boolean type input -- `True` or `False`. If true, rows with high null columns (defined by treatment_threshold argument) are removed from the input dataset.

- `treatment_threshold`: It takes a value between `0` to `1` with default 0.8, which means 80% of columns are allowed to be Null per row. If it is more than the threshold, then it is flagged and if treatment is True, then affected rows are removed. If the threshold is 0, it means rows with any missing value will be flagged. If the threshold is 1, it means rows with all missing values will be flagged.

_Example:_
```yaml
nullRows_detection:
  list_of_cols: all
  drop_cols: []
  treatment: True
  treatment_threshold: 0.75
```

### `invalidEntries_detection`

- `list_of_cols`: (list format or string of col names separated by `|`). It is used to specify the columns which are subjected to the invalid entries' detection

- `drop_cols`: (list format or string of col names separated by `|`). It is used to specify the columns that need to be dropped from list_of_cols before invalid entries' detection

- `treatment`: This takes Boolean type input -- `True` or `False`. If true, invalid values are replaced as null and treated as missing.

- `output_mode`: `replace` or `append`. "replace" option replaces original columns with treated column, whereas "append" option append treated column to the input dataset. All treated columns are appended with the naming convention `{original.column.name}_cleaned`

_Example:_
```yaml
invalidEntries_detection:
  list_of_cols: all
  drop_cols: ['id_column']
  treatment: True
  output_mode: replace
```

### `IDness_detection`

- `List_of_cols`: (list format or string of col names separated by `|`). It is used to specify the columns which are subjected to the Idness detection

- `Drop_cols`: (list format or string of col names separated by `|`). It is used to specify the columns that need to be dropped from list_of_cols before Idness detection

- `Treatment`: This takes Boolean type input -- `True` or `False`. If true, columns above IDness threshold are removed.

- `Treatment_threshold`: This takes value between `0` to `1` with default 1.0.

_Example:_
```yaml
IDness_detection:
  list_of_cols: all
  drop_cols: ['id_column']
  treatment: True
  treatment_threshold: 0.9
```

### `Biasedness_detection`

- `List_of_cols`: (list format or string of col names separated by `|`). It is used to specify the columns which are subjected to the biasedness detection

- `Drop_cols`: (list format or string of col names separated by `|`). It is used to specify the columns that need to be dropped from list_of_cols before biasedness detection

- `Treatment`: This takes Boolean type input -- `True` or `False`. If true, columns above biasedness threshold are removed.

- `Treatment_threshold`: This takes value between `0` to `1` with default 1.0.

_Example:_
```yaml
biasedness_detection:
  list_of_cols: all
  drop_cols: ['label_col']
  treatment: True
  treatment_threshold: 0.98
```

### `Outlier_detection`

- `List_of_cols`: (list format or string of col names separated by `|`). It is used to specify the columns which are subjected to the outlier detection

```
Note: Any attribute with single value or all null values are not subjected to outlier detection even if it is selected under this argument.
```

- `Drop_cols`: (list format or string of col names separated by `|`). It is used to specify the columns that need to be dropped from list_of_cols before outlier detection

- `Detection_side`: 'upper', 'lower', 'both'

- `Detection_configs`: It takes input in dictionary format with keys (representing upper and lower bound for different outlier identification methodologies) - pctile_lower (default 0.05), pctile_upper (default 0.95), stdev_lower (default 3.0), stdev_upper (default 3.0), IQR_lower (default 1.5), IQR_upper (default 1.5), min_validation (default 2)

- `Treatment`: takes Boolean type input -- `True` or `False`. If true, specified treatment method is applied.

- `Treatment_method`: 'null_replacement', 'row_removal', 'value_replacement'

- `Pre_existing_model`: It takes Boolean type input -- `True` or `False`. True if the file with upper/lower permissible values exists already, False Otherwise.

- `Model_path`: If pre_existing_model is `True`, this is path for pre-saved model file. If pre_existing_model is `False`, this field can be used for saving the model file. Default NA means there is neither pre-saved model file nor there is a need to save one.

- `Output_mode`: `replace` or `append`. "replace" option replaces original columns with treated column, whereas "append" option append treated column to the input dataset. All treated columns are appended with the naming convention - `{original.column.name}_outliered`.

_Example:_
```yaml
outlier_detection:
  list_of_cols: all
  drop_cols: ['id_column','label_col']
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

- `list_of_cols`: `all` can be passed to include all (non-array) columns for analysis. `missing` (default) can be passed to include only those columns with missing values. One of the use cases where "all" may be preferable over "missing" is when the user wants to save the imputation model for future use e.g. a column may not have missing value in the training dataset. Still, missing values may possibly appear in the prediction dataset.

- `drop_cols`: (list format or string of col names separated by `|`). It is used to specify the columns that need to be dropped from list_of_cols before null column detection

- `treatment`: takes Boolean type input -- `True` or `False`. If true, missing values are treated as per treatment_method argument

- `treatment_method`: 'MMM', 'row_removal' or 'column_removal'

- `treatment_configs`: It takes input in dictionary format with keys `treatment_threshold` for column_removal treatment, or all arguments corresponding to imputation_MMM function.

_Example:_
```yaml
nullColumns_detection:
  list_of_cols: all
  drop_cols: ['id_column','label_col']
  treatment: True
  treatment_method: MMM
  treatment_configs:
    method_type: median
    pre_existing_model: False
    model_path: NA
    output_mode: replace
```

## `association_evaluator`

This submodule focuses on understanding the interaction between different attributes and/or the relationship between an attribute & the binary target variable. see the 
[Association Evaluator Doc](https://github.com/anovos/anovos-docs/blob/main/docs/api/data_analyzer/association_evaluator.md) for understanding its functions.

### `correlation_matrix`

- `list_of_cols`: (list format or string of col names separated by `|`). It is used to specify the columns which are subjected for generating correlation matrix. The user can also use `all` as an input to this argument to consider all columns. This is super useful instead of specifying all column names manually.

- `drop_cols`: (list format or string of col names separated by `|`). It is used to specify the columns which needs to be dropped from list_of_cols. It is most useful when used coupled with `all` value of list_of_cols, when we need to consider all columns except few handful of them.

_Example:_
```yaml
correlation_matrix:
  list_of_cols: all
  drop_cols: ['id_column']
```

### `IV_calculation`

- `List_of_cols`: (list format or string of col names separated by `|`). It is used to specify the columns which are subjected to IV calculation.

- `Drop_cols`: (list format or string of col names separated by `|`). It is used to specify the columns that need to be dropped from list_of_cols before IV calculation

- `Label_col`: Name of label or target column in the input dataset

- `Event_lable`: Value of event (label 1) in the label column

- `Encoding_configs`: It takes input in dictionary format with keys related to binning operation - `bin_method` (default 'equal_frequency'), `bin_size` (default 10) and `monotonicity_check` (default 0). monotonicity_check of 1 will dynamically calculate the bin_size ensuring monotonic nature and can be expensive operation.

_Example:_
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

- `List_of_cols`: (list format or string of col names separated by `|`). It is used to specify the columns which are subjected to IG calculation

- `Drop_cols`: (list format or string of col names separated by `|`). It is used to specify the columns that need to be dropped from list_of_cols before IG calculation

- `Label_col`: Name of label or target column in the input dataset

- `Event_lable`: Value of event (label 1) in the label column

- `Encoding_configs`: It takes input in dictionary format with keys related to binning operation - 'bin_method' (default 'equal_frequency'), 'bin_size' (default 10) and 'monotonicity_check' (default 0). monotonicity_check of 1 will dynamically calculate the bin_size ensuring monotonic nature and can be expensive operation.

_Example:_
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

### `Variable_clustering`

- `List_of_cols`: (list format or string of col names separated by `|`). It is used to specify the columns which are subjected to variable clustering

- `Drop_cols`: (list format or string of col names separated by `|`). It is used to specify the columns that need to be dropped from list_of_cols before variable clustering.

_Example:_
```yaml
variable_clustering:
  list_of_cols: all
  drop_cols: id_column|label_col
```

## `drift_detector`

### `drift_statistics`

#### `configs`

- `list_of_cols`: List of columns to check drift (list or string of col names separated by `|`). Use `all` - to include all non-array columns (excluding drop_cols).

- `drop_cols`: List of columns to be dropped (list or string of col names separated by `|`)

- `method_type`: 'PSI', 'JSD', 'HD', 'KS' (list or string of methods separated by `|`). Use `all` - to calculate all metrics.

- `Threshold`: To flag attributes meeting drift threshold

- `bin_method`: 'equal_frequency' or 'equal_range'

- `bin_size`: 10 - 20 (recommended for PSI), >100 (other method types)

- `pre_existing_source`: True if binning model & frequency counts/attribute exists already, False Otherwise.

- `source_path`: If pre_existing_source is True, this is path for the source dataset details - drift_statistics folder. drift_statistics folder must contain attribute_binning & frequency_counts folders. If pre_existing_source is False, this can be used for saving the details. Default "NA" for temporarily saving source dataset attribute_binning folder.

_Example:_
```yaml
configs:
  list_of_cols: all
  drop_cols: ['id_column','label_col']
  method_type: all
  threshold: 0.1
  bin_method: equal_range
  bin_size: 10
  pre_existing_source: False
  source_path: NA
```

#### `source_dataset`

##### `read_dataset`

- `file_path`: The file (or directory) path to read the source dataset from.
   It can be a local path, an [S3 path](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-bucket-intro.html)
  (when running on AWS), a path to a file resource on Google Colab (see
   [this tutorial](https://neptune.ai/blog/google-colab-dealing-with-files)for
   an overview), or a path on the [Databricks File System](https://docs.microsoft.com/de-de/azure/databricks/data/databricks-file-system)
   (when running on Azure).

- `file_type`: The file format of the source data. Currently, _Anovos_ supports
   CSV (`csv`), Parquet (`parquet`), and Avro (`avro`).
   (Please note that if you're using Avro data sources, you need to add the external
   package `org.apache.spark:spark-avro` when submitting the Spark job.)

- `file_configs` (optional): Options to pass to the respective Spark file reader,
   e.g., delimiters, schemas, headers.

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
  to the first type in `list_of_dtypes` and so on. See
  [the Spark documentation](https://spark.apache.org/docs/latest/sql-ref-datatypes.html)
  for a list of valid datatypes.
  Note that this field is case-insensitive.

### `stability_index`

#### `configs`

- `metric_weightages`: A dictionary with key being the metric name (mean, stdev, kurtosis) and value being the weightage of the metric (between 0 and 1). Sum of all weightages must be 1.

- `existing_metric_path`: path for pre-existing metrics of historical datasets <idx, attribute, mean, stdev, kurtosis>. idx is index number of historical datasets assigned in chronological order

- `appended_metric_path`: path for saving input dataframes metrics after appending to the historical datasets' metrics.

- `threshold`: To flag unstable attributes meeting the threshold.

_Example:_
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

- `file_path`: file (or directory) path where the historical dataset is saved.

- `file_type`: (CSV, Parquet or Avro). file format of the historical dataset.

- `file_configs` (optional): Options to pass to the respective Spark file reader,
   e.g., delimiters, schemas, headers.

```
Note: There can be multiple historical datasets, for loading the other datasets, it's configuration should be written in same as dataset1
```

#### `dataset2`
  
  same configuration as dataset1

## `report_preprocessing`

This section largely covers the data pre–processing. The primary function which is used to address all the subsequent modules is charts_to_objects. It precisely helps in saving the chart data in form of objects, which is eventually read by the final report generation script. The objects saved are specifically used at the modules shown at the Report based on the user input. See the [Intermediate Report Doc](https://github.com/anovos/anovos-docs/blob/main/docs/using-anovos/data-reports/intermediate_report.md) for more details.

### `master_path` 

  Path where all modules output is saved

_Example:_
```yaml
master_path: 'report_stats'
```

### `charts_to_objects`

- `list_of_cols`: (list format or string of col names separated by `|`). It is used to specify the columns which are subjected to the analysis in the input dataframe.

- `drop_cols`: (list format or string of col names separated by `|`). It is used to specify the columns which needs to be dropped from list_of_cols

- `lable_col`: Name of label or target column in the input dataset

- `event_label`: Value of event (label 1) in the label column

- `bin_method`: equal_frequency or equal_range

- `bin_size`: 10 - 20 (recommended for PSI), >100 (other method types)

- `drift_detector`: It takes Boolean type input -- `True` or `False`. It indicates whether the drift component is already analyzed or not. By default it is kept as False.

- `outlier_charts`: It takes Boolean type input -- `True` or `False`. Outlier chart provides the flexibility to the user whether the user wants outlier charts or not. If True, outlier_chart will be generated.

- `source_path`: The source data path which is needed for drift analysis. If it's not computed / out of scope, the default value of "NA" is considered.

_Example:_
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

## `report_generation`

This section covers the final execution part where primarily the output generated by the previous step is being fetched upon and structured in the desirable UI layout. See the [Final report generation Doc](https://github.com/anovos/anovos-docs/blob/main/docs/using-anovos/data-reports/final_report.md) for more details.

- `master_path`: The path which contains the data of intermediate output in terms of json chart objects, csv file (pandas df).

- `id_col`: The ID column is accepted to ensure & restrict unnecessary analysis to be performed on the same lable_col: Name of label or target column in the input dataset

- `corr_threshold`: The threshold chosen beyond which the attributes are found to be redundant. It should be between 0 to 1.

 `iv_threshold`: The threshold beyond which the attributes are found to be significant in terms of model. It takes value between 0 to 1.

| **Information Value**  | **Variable Predictiveness**  |
|------------------------|------------------------------|
| Less than 0.02         | Not useful for prediction    |
| 0.02 to 0.1            | Weak predictive Power        |
| 0.1 to 0.3             | Medium predictive Power      |
| 0.3 to 0.5             | Strong predictive Power      |
| >0.5                   | Suspicious Predictive Power  |

- `drift_threshold_model`: The threshold beyond which the attribute can be flagged as 1 or drifted as measured across different drift metrices specified by the user. It takes value between 0 to 1.

- `dataDict_path`: The path containing the exact name, definition mapping of the attributes. This is eventually used to populate at the report for easy referencing. 

- `metricDict_path`: Path of metric dictionary.

- `final_report_path`: Path where final report will be saved. File path can be a local path or s3 path (when running with AWS cloud services), azure dbfs or azure blob storage (when running with Azure databricks). Note: azure dbfs path should be like "/dbfs/directory_name" and For azure blob storage path should be like "/dbfs/mnt/directory_name" beacause in report generation all the operations happen in python.

_Example:_
```yaml
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

## `transformers`:

The data transformer module supports selected pre-processing & transformation functions, such as binning, encoding, scaling, imputation, to name a few, which are required for statistics generation and quality checks. See the
[Transformers Doc](https://github.com/anovos/anovos-docs/blob/main/docs/api/data_transformer/transformers.md) for understanding its functions in detail.

### `numerical_mathops`

This group of functions used to perform mathematical transformation over numerical attributes. Users must use only one function at a time and section of other function should be commented like if user want to use feature transformation then all Section of boxcox transformation should be commented.


#### `feature_transformation`:

- `list_of_cols`: List of numerical columns to encode e.g., ["col1","col2"]. "all" can be passed to include all numerical columns for analysis.

- `drop_cols`: List of columns to be dropped e.g., ["col1","col2"].

- `method_type`: "ln", "log10", "log2", "exp", "powOf2" (2^x), "powOf10" (10^x), "powOfN" (N^x), "sqrt" (square root), "cbrt" (cube root), "sq" (square), "cb" (cube), "toPowerN" (x^N), "sin", "cos", "tan", "asin", "acos", "atan", "radians", "remainderDivByN" (x%N), "factorial" (x!), "mul_inv" (1/x), "floor", "ceil", "roundN" (round to N decimal places) (Default value = "sqrt")

- `N`: None by default. If method_type is "powOfN", "toPowerN", "remainderDivByN" or "roundN", N will be used as the required constant.

_Example:_
```yaml
feature_transformation:
  list_of_cols: all
  drop_cols: []
  method_type: sqrt
```

#### `boxcox_transformation`:

- `list_of_cols`: List of numerical columns to encode e.g., ["col1","col2"].

- `drop_cols`: List of columns to be dropped e.g., ["col1","col2"].

- `boxcox_lambda`: Lambda value for box_cox transormation.
  If boxcox_lambda is not None, it will be directly used for the transformation. It can be a
    1. `list`: each element represents a lambda value for an attribute and the length of the list must be the same as the number of columns to transform.
    2. `int/float`: all attributes will be assigned the same lambda value.
    Else, search for the best lambda among [1,-1,0.5,-0.5,2,-2,0.25,-0.25,3,-3,4,-4,5,-5] for each column and apply the transformation (Default value = None)

_Example:_
```yaml
list_of_cols: num_feature1|num_feature2
drop_cols: []
```

### `numerical_binning`

This group of functions used to tranform the numerical attribute into discrete (integer or categorical values) attribute. Users must use only one function at a time and section of other function should be commented like if user want to use attribute binning method then all Section of monotonic binning method should be commented.

#### `attribute_binning`

- `list_of_cols`: List of numerical columns to transform e.g., ["col1","col2"]

- `drop_cols`: List of columns to be dropped e.g., ["col1","col2"]

- `method_type`: equal_frequency", "equal_range". In "equal_range" method, each bin is of equal size/width and in "equal_frequency", each bin has equal no. of rows, though the width of bins may vary. (Default value = "equal_range")

- `bin_size`: Number of bins. (Default value = 10)

- `bin_dtype`: "numerical", "categorical". With "numerical" option, original value is replaced with an Integer (1,2,…) and with "categorical" option, original replaced with a string describing min and max value allowed in the bin ("minval-maxval"). (Default value = "numerical").

_Example:_
```yaml
attribute_binning:
  list_of_cols: num_feature1|num_feature2
  drop_cols: []
  method_type: equal_frequency
  bin_size: 10
  bin_dtype: numerical
```

#### `monotonic_binning`

- `list_of_cols`: List of numerical columns to transform e.g., ["col1","col2"]

- `drop_cols`: List of columns to be dropped e.g., ["col1","col2"]

- `method_type`: equal_frequency", "equal_range". In "equal_range" method, each bin is of equal size/width and in "equal_frequency", each bin has equal no. of rows, though the width of bins may vary. (Default value = "equal_range")

- `bin_size`: Number of bins. (Default value = 10)

- `bin_dtype`: "numerical", "categorical". With "numerical" option, original value is replaced with an Integer (1,2,…) and with "categorical" option, original replaced with a string describing min and max value allowed in the bin ("minval-maxval"). (Default value = "numerical").

_Example:_
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


### `numerical_expression`:

#### `expression_parser`

- `list_of_expr`: List of expressions to evaluate as new features e.g., ["expr1","expr2"]. Alternatively, expressions can be specified in a string format, where different expressions are separated by pipe delimiter “|” e.g., "expr1|expr2".

- `postfix`: postfix for new feature name.Naming convention "f" + expression_index + postfix e.g. with postfix of "new", new added features are named as f0new, f1new etc. (Default value = "").

_Example:_
```yaml
expression_parser:
  list_of_expr: 'log(age) + 1.5|sin(capital-gain)+cos(capital-loss)'
```

### `categorical_outliers`

This function replaces less frequently seen values (called as outlier values in the current context) in a categorical column by 'others'.

#### `outlier_categories`

- `list_of_cols`: List of categorical columns to transform e.g., ["col1","col2"].

- `drop_cols`: List of columns to be dropped e.g., ["col1","col2"]

- `coverage`: Defines the minimum % of rows that will be mapped to actual category name and the rest to be mapped to others and takes value between 0 to 1. Coverage of 0.8 can be interpreted as top frequently seen categories are considered till it covers minimum 80% of rows and rest lesser seen values are mapped to others. (Default value = 1.0)

- `max_category`: Even if coverage is less, only (max_category - 1) categories will be mapped to actual name and rest to others. Caveat is when multiple categories have same rank, then #categories can be more than max_category. (Default value = 50).

_Example:_
```yaml
outlier_categories:
  list_of_cols: ["cat_feature1","cat_feature2"]
  drop_cols: ['id_column','label_col']
  coverage: 0.9
  max_category: 20
```

### `categorical_encoding`

This group of transformers functions used to converting a categorical attribute into numerical attribute(s). Users should use only one function at a time and section of other function should be commented like if user want to use categorical to numerical conversion using unsupervised method then all Section of categorical to numerical conversion using supervised method should be commented.

#### `cat_to_num_unsupervised`

- `list_of_cols`: List of categorical columns to transform e.g., ["col1","col2"]

- `drop_cols`: List of columns to be dropped e.g., ["col1","col2"].

- `method_type`: 1 for Label Encoding or 0 for One hot encoding. 
  In label encoding, each categorical value is assigned a unique integer based on alphabetical or frequency ordering (both ascending & descending options are available that can be selected by index_order argument). 

  In one-hot encoding, every unique value in the column will be added in a form of dummy/binary column. (Default value = 1)

- `index_order`: frequencyDesc", "frequencyAsc", "alphabetDesc", "alphabetAsc". Valid only for Label Encoding method_type. (Default value = "frequencyDesc")

- `cardinality_threshold`: Defines threshold to skip columns with higher cardinality values from encoding. Default value is 100.

_Example:_
```yaml
cat_to_num_unsupervised:
  list_of_cols: ["cat_feature1","cat_feature2"]
  drop_cols: ['id_column']
  method_type: 0
  cardinality_threshold: 110
```

#### `cat_to_num_supervised`

- `list_of_cols`: List of catigorical columns to transform e.g., ["col1","col2"].

- `drop_cols`: List of columns to be dropped e.g., ["col1","col2"].

- `label_col`: Label/Target column (Default value = "label")

- `event_label`: Value of (positive) event (i.e label 1) (Default value = 1).

_Example:_
```yaml
cat_to_num_supervised:
  list_of_cols: cat_feature1 | cat_feature2
  drop_cols: ['id_column']
  label_col: income
  event_label: '>50K'
```

### `numerical_rescaling`

This group of transformers functions used to rescale attribute(s). Users should use only one function at a time and section of other functions should be commented like if user want to use normalization method then all Sections of Z-standarization and IQR-standarization should be commented.

#### `normalization`

- `list_of_cols`: List of numerical columns to transform e.g., ["col1","col2"]. "all" can be passed to include all numerical columns for analysis.

- `drop_cols`: List of columns to be dropped e.g., ["col1","col2"].

_Example:_
```yaml
normalization:
  list_of_cols: ["num_feature1","num_feature2"]
  drop_cols: []
```

#### `z_standarization`

- `list_of_cols`: List of numerical columns to transform e.g., ["col1","col2"]. "all" can be passed to include all numerical columns for analysis.

- `drop_cols`: List of columns to be dropped e.g., ["col1","col2"].

_Example:_
```yaml
z_standardization:
  list_of_cols: ["num_feature1","num_feature2"]
  drop_cols: []
```

#### `IQR_standarization`

- `list_of_cols`: List of numerical columns to transform e.g., ["col1","col2"].

- `drop_cols`: List of columns to be dropped e.g., ["col1","col2"].

_Example:_
```yaml
IQR_standardization:
  list_of_cols: ["num_feature1","num_feature2","num_feature3"]
  drop_cols: []
```

### `numerical_latentFeatures`

This group of transformer functions used to generate latent features which reduces the dimensionality of the input dataframe. Users should use only one function at a time and section of other function should be commented like if user want to use PCA method then all Section of AutoEncoder method should be commented.

#### `PCA_latentFeatures`

- `list_of_cols`: List of numerical columns to encode e.g., ["col1","col2"]. "all" can be passed to include all numerical columns for analysis.

- `drop_cols`: List of columns to be dropped e.g., ["col1","col2"].

- `explained_variance_cutoff`: Determines the number of encoded columns in the output. If N is the smallest integer such that top N encoded columns explain more than explained_variance_cutoff variance, these N columns will be selected. (Default value = 0.95)

- `standardization`:  Boolean argument – True or False. True, if the standardization required. (Default value = True)

- `imputation`: Boolean argument – True or False. True, if the imputation required. (Default value = False)

- `imputation_configs`: Takes input in dictionary format. Imputation function name is provided with key "imputation_name". optional arguments pertaining to that imputation function can be provided with argument name as key. (Default value = {"imputation_function": "imputation_MMM"})

- `stats_missing`: Takes arguments for read_dataset (data_ingest module) function in a dictionary format to read pre-saved statistics on missing count/pct i.e. if measures_of_counts or missingCount_computation (data_analyzer.stats_generator module) has been computed & saved before. (Default value = {}).

_Example:_
```yaml
PCA_latentFeatures:
  list_of_cols: ["num_feature1","num_feature2","num_feature3"]
  explained_variance_cutoff: 0.95
  standardization: False
  imputation: True
```

#### `autoencoder_latentFeatures`

- `list_of_cols`:  List of numerical columns to encode e.g., ["col1","col2"]. "all" can be passed to include all numerical columns for analysis.

- `drop_cols`: List of columns to be dropped e.g., ["col1","col2"].

- `reduction_params`: Determines the number of encoded features in the result. If reduction_params < 1, int(reduction_params * (number of columns)) columns will be generated. Else, reduction_params columns will be generated. (Default value = 0.5)

- `sample_size`: Maximum rows for training the autoencoder model using tensorflow. (Default value = 500000)

- `epochs`: Integer - number of epochs to train the tensorflow model. (Default value = 100)

- `batch_size`: Integer - number of samples per gradient update when fitting the tensorflow model. (Default value = 256)

- `standardization`: Boolean argument – True or False. True, if the standardization required. (Default value = True)

- `standardization_configs`: z_standardization function arguments in dictionary format. (Default value = {"pre_existing_model": False)

- `imputation`: Boolean argument – True or False. True, if the imputation required. (Default value = False)

- `imputation_configs`: Takes input in dictionary format. Imputation function name is provided with key "imputation_name". optional arguments pertaining to that imputation function can be provided with argument name as key. (Default value = {"imputation_function": "imputation_MMM"})

- `stats_missing`: Takes arguments for read_dataset (data_ingest module) function in a dictionary format to read pre-saved statistics on missing count/pct i.e. if measures_of_counts or missingCount_computation (data_analyzer.stats_generator module) has been computed & saved before. (Default value = {})

_Example:_
```yaml
autoencoder_latentFeatures:
  list_of_cols: ["num_feature1","num_feature2","num_feature3"]
  reduction_params: 0.5
  sample_size: 10000 
  epochs: 20
  batch_size: 256
```

## `write_intermediate`

- `file_path`: Path where intermediate datasets (after selecting, dropping, renaming, and recasting of columns) for quality checker operations, join dataset and concatenate dataset will be saved.

- `file_type`: (CSV, Parquet or Avro). file format of intermediate dataset

- `file_configs` (optional): Rest of the valid configuration can be passed through this options e.g., repartition, mode, compression, header, delimiter, inferSchema etc. This might look like: 
   ```yaml
   file_configs:
      mode: overwrite
      header: True
      delimiter: ","
      inferSchema: True
   ```
   For more information on available configuration options, see the following external
   documentation:
   
     - [Write CSV files](https://sparkbyexamples.com/pyspark/pyspark-read-csv-file-into-dataframe/)
     - [Write Parquet files](https://sparkbyexamples.com/pyspark/pyspark-read-and-write-parquet-file/)
     - [Write Avro files](https://sparkbyexamples.com/spark/read-write-avro-file-spark-dataframe/)

## `write_main`

- `file_path`: Path where final cleaned input dataset will be saved.

- `file_type`: (CSV, Parquet or Avro). file format of final dataset

- `file_configs` (optional): Rest of the valid configuration can be passed through this options e.g., repartition, mode, compression, header, delimiter, inferSchema etc. This might look like: 
   ```yaml
   file_configs:
      mode: overwrite
      header: True
      delimiter: ","
      inferSchema: True
   ```
   For more information on available configuration options, see the following external
   documentation:
   
     - [Write CSV files](https://sparkbyexamples.com/pyspark/pyspark-read-csv-file-into-dataframe/)
     - [Write Parquet files](https://sparkbyexamples.com/pyspark/pyspark-read-and-write-parquet-file/)
     - [Write Avro files](https://sparkbyexamples.com/spark/read-write-avro-file-spark-dataframe/)

## `write_stats`

- `file_path`: Path where all tables/stats of anovos modules (data drift & data analyzer) will be saved.

- `file_type`: (CSV, Parquet or Avro). file format of final dataset

- `file_configs` (optional): Rest of the valid configuration can be passed through this options e.g., repartition, mode, compression, header, delimiter, inferSchema etc. This might look like: 

   ```yaml
   file_configs:
      mode: overwrite
      header: True
      delimiter: ","
      inferSchema: True
   ```

   For more information on available configuration options, see the following external
   documentation:
   
     - [Write CSV files](https://sparkbyexamples.com/pyspark/pyspark-read-csv-file-into-dataframe/)
     - [Write Parquet files](https://sparkbyexamples.com/pyspark/pyspark-read-and-write-parquet-file/)
     - [Write Avro files](https://sparkbyexamples.com/spark/read-write-avro-file-spark-dataframe/)
