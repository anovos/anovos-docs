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

    a.  method: index or name. This needs to be entered as a keyword
        argument. The "index" method involves concatenating the
        dataframes by the column index. IF the sequence of column is not
        fixed among the dataframe, this method should be avoided. The
        "name" method involves concatenating by columns names. The 1st
        dataframe passed under idfs will define the final columns in the
        concatenated dataframe. It will throw an error if any column in
        the 1st dataframe is not available in any of other dataframes.

    b.  dataset1

        i.  read_dataset

            1.  file_path: file (or directory) path where the other input data is saved. 
                File path can be a local path, s3 path (when running with AWS cloud services),
                google colab path (when running with open source platform: Google Colab),
                azure dbfs or azure blob storage (when running with Azure databricks).
                Note: For azure dbfs path should be like "dbfs:/directory_name" and 
                for azure blob storage path should be like "dbfs:/mnt/directory_name".

            2.  file_type: (CSV, Parquet or Avro). file format of the
                input data. Currently, we support CSV, Parquet or Avro.
                Note: Avro data source requires an external package to run,
                which can be configured with spark-submit options
                (--packages org.apache.spark:spark-avro_2.11:2.4.0).

            3.  file_configs (optional): Rest of the valid configuration
                can be passed through this key e.g., delimiter, InferSchema, header.
                Examples
                    delimiter: ","
                    header: True
                    inferSchema: True

        ii. delete_column: (list format or string of col names separated
            by |). It specifies the columns required to be deleted from
            the other input dataframe.

        iii. select_column: (list format or string of col names
            separated by |). It specifies the columns required to be
            selected from the other input dataframe.

        iv. rename_column

            1.  list_of_cols: (list format or string of col names
                separated by |). It is used to specify the columns
                required to be renamed in the other input dataframe.

            2.  list_of_newcols: It is used to specify the new column
                name, i.e., the first element in list_of_cols will be
                the original column name, and the corresponding first
                column in list_of_newcols will be the new column name.

        v.  recast_column

            1.  list_of_cols: (list format or string of col names
                separated by |). It is used to specify the columns
                required to be recast in the other input dataframe.

            2.  list_of_dtypes: It is used to specify the datatype,
                i.e., the first element in list_of_cols will column
                name, and the corresponding element in list_of_dtypes
                will be new datatype such as float, integer, string,
                double, decimal, etc. (case insensitive).

    c.  dataset2: same as dataset1

## `join_dataset`

    a.  Join_cols: Key column(s) to join all dataframes together. In
        case of multiple columns to join, they can be passed in a list
        format or a single text format where different column names are
        separated by pipe delimiter "|"

    b.  Join_type: "inner", "full", "left", "right", "left_semi",
        "left_anti"

    c.  dataset1

        i.  read_dataset

            1.  file_path: file (or directory) path where the other
                input data that need to be joined is saved.
                File path can be a local path, s3 path (when running with AWS cloud services),
                google colab path (when running with open source platform: Google Colab),
                azure dbfs or azure blob storage (when running with Azure databricks).
                Note: For azure dbfs path should be like "dbfs:/directory_name" and 
                for azure blob storage path should be like "dbfs:/mnt/directory_name".

            2.  file_type: (CSV, Parquet or Avro). file format of the
                other input data (joining dataset).

            3.  file_configs (optional): Rest of the valid configuration
                can be passed through this key e.g., delimiter, InferSchema, header.
                Examples
                    delimiter: ","
                    header: True
                    inferSchema: True

        ii. delete_column: (list format or string of col names separated
            by |). It specifies the columns required to be deleted from
            the other input dataframe.

        iii. select_column: (list format or string of col names
            separated by |). It specifies the columns required to be
            selected from the other input dataframe.

        iv. rename_column

            1.  list_of_cols: (list format or string of col names
                separated by |). It is used to specify the columns
                required to be renamed in the other input dataframe.

            2.  list_of_newcols: It is used to specify the new column
                name, i.e., the first element in list_of_cols will be
                the original column name, and the corresponding first
                column in list_of_newcols will be the new column name.

        v.  recast_column

            1.  list_of_cols: (list format or string of col names
                separated by |). It is used to specify the columns
                required to be recast in the other input dataframe.

            2.  list_of_dtypes: It is used to specify the datatype,
                i.e., the first element in list_of_cols will column
                name, and the corresponding element in list_of_dtypes
                will be new datatype such as float, integer, string,
                double, decimal, etc. (case insensitive).

    d.  dataset2: same configuration as dataset1

Attaching the documentation link of data ingest module to understand more about above operations(read, delete, select, join, concatenate, etc): [Data Ingest](https://github.com/anovos/anovos-docs/blob/main/docs/anovos-modules-overview/data-ingest/index.md)

## `anovos_basic_report`

    a.  Basic_report: This takes Boolean type input -- True or False. If
        True, basic report is generated after completion of data analyzer, association evaluator and quality checker modules which have descriptive
        statistics(global_summary, measures_of_count,
        measures_of_centralTendency, measures_of_cardinality,
        measures_of_dispersion, measures_of_percentiles,
        measures_of_shape), quality checker(nullRows_detection,
        nullColumns_detection, duplicate_detection, IDness_detection,
        biasedness_detection, invalidEntries_detection,
        outlier_detection), attribute association (correlation_matrix,
        IV_calculation, IG_calculation, variable_clustering).

    Attaching the documentation link of modules to get better idea what these modules actually do and thier output: [Data Analyzer](https://github.com/anovos/anovos-docs/blob/main/docs/anovos-modules-overview/data-analyzer/index.md), [Quality Checker](https://github.com/anovos/anovos-docs/blob/main/docs/anovos-modules-overview/quality-checker/index.md), [Association Evaluator](https://github.com/anovos/anovos-docs/blob/main/docs/anovos-modules-overview/association-evaluator/index.md), [Data Drift and Stability Index](https://github.com/anovos/anovos-docs/blob/main/docs/anovos-modules-overview/data_drift_and_stability_index/index.md) 

    b.  Report_args

        i.  Id_col: Name of Id column in the input dataset

        ii. Label_col: Name of label or target column in the input
            dataset

        iii. Event_lable: Value of event (label 1) in the label column

        iv. Output_path: Path where basic report is saved. File path can
            be a local path or s3 path (when running with AWS cloud
            services)

## `stats_generator`

    a.  Metric: list of different metrics used to generate descriptive
        statistics [global_summary, measures_of_count,
        measures_of_centralTendency, measures_of_cardinality,
        measures_of_dispersion, measures_of_percentiles,
        measures_of_shape]

    b.  Metric_args

        i.  List_of_cols: (list format or string of col names separated
            by |). It is used to specify the columns which are
            subjected to the analysis in the input dataframe. The user
            can also use "all" as an input to this argument to consider
            all columns. This is super useful instead of specifying all
            column names manually.

        ii. Drop_cols: (list format or string of col names separated by
            |). It is used to specify the columns that need to be
            dropped from list_of_cols. It is most useful when coupled
            with the "all" value of list_of_cols, when we need to
            consider all columns except a few handful of them.

## `quality_checker`

    a.  duplicate_detection

        i.  list_of_cols: (list format or string of col names separated
            by |). It is used to specify the columns which are
            subjected to the duplicate detection

        ii. drop_cols: (list format or string of col names separated by
            |). It is used to specify the columns that need to be
            dropped from list_of_cols before duplicate detection

        iii. treatment: It takes Boolean type input -- True or False. If
            true, duplicate rows are removed from the input dataset.

    b.  nullRows_detection

        i.  list_of_cols: (list format or string of col names separated
            by |). It is used to specify the columns which are
            subjected to the null rows detection

        ii. drop_cols: (list format or string of col names separated by
            |). It is used to specify the columns that need to be
            dropped from list_of_cols before null rows detection

        iii. treatment: This takes Boolean type input -- True or False.
            If true, rows with high null columns (defined by
            treatment_threshold argument) are removed from the input
            dataset.

        iv. treatment_threshold: It takes a value between 0 to 1 with
            default 0.8, which means 80% of columns are allowed to be
            Null per row. If it is more than the threshold, then it is
            flagged and if treatment is True, then affected rows are
            removed. If the threshold is 0, it means rows with any
            missing value will be flagged. If the threshold is 1, it
            means rows with all missing values will be flagged.

    c.  invalidEntries_detection

        i.  list_of_cols: (list format or string of col names separated
            by |). It is used to specify the columns which are
            subjected to the invalid entries' detection

        ii. drop_cols: (list format or string of col names separated by
            |). It is used to specify the columns that need to be
            dropped from list_of_cols before invalid entries' detection

        iii. treatment: This takes Boolean type input -- True or False.
             If true, invalid values are replaced as null and treated as
             missing.

        iv. output_mode: replace or append. "replace" option replaces
            original columns with treated column, whereas "append"
            option append treated column to the input dataset. All
            treated columns are appended with the naming convention -
            "{original.column.name}_cleaned"

    d.  IDness_detection

        i.  List_of_cols: (list format or string of col names separated
            by |). It is used to specify the columns which are
            subjected to the Idness detection

        ii. Drop_cols: (list format or string of col names separated by
            |). It is used to specify the columns that need to be
            dropped from list_of_cols before Idness detection

        iii. Treatment: This takes Boolean type input -- True or False.
             If true, columns above IDness threshold are removed.

        iv. Treatment_threshold: This takes value between 0 to 1 with
            default 1.0.

    e.  Biasedness_detection

        i.  List_of_cols: (list format or string of col names separated
            by |). It is used to specify the columns which are
            subjected to the biasedness detection

        ii. Drop_cols: (list format or string of col names separated by
            |). It is used to specify the columns that need to be
            dropped from list_of_cols before biasedness detection

        iii. Treatment: This takes Boolean type input -- True or False.
             If true, columns above biasedness threshold are removed.

        iv. Treatment_threshold: This takes value between 0 to 1 with
            default 1.0.

    f.  Outlier_detection

        i.  List_of_cols: (list format or string of col names separated
            by |). It is used to specify the columns which are
            subjected to the outlier detection

            Note: Any attribute with single value or all null values are not
            subjected to outlier detection even if it is selected under this
            argument.

        ii. Drop_cols: (list format or string of col names separated by |). It
            is used to specify the columns that need to be dropped from
            list_of_cols before outlier detection

        iii. Detection_side: upper, lower, both

        iv. Detection_configs: It takes input in dictionary format with keys
            (representing upper and lower bound for different outlier
            identification methodologies) - pctile_lower (default 0.05),
            pctile_upper (default 0.95), stdev_lower (default 3.0), stdev_upper
            (default 3.0), IQR_lower (default 1.5), IQR_upper (default 1.5),
            min_validation (default 2)

        v.  Treatment: takes Boolean type input -- True or False. If true,
            specified treatment method is applied.

        vi. Treatment_method: null_replacement, row_removal, value_replacement

        vii. Pre_existing_model: It takes Boolean type input -- True or False.
            True if the file with upper/lower permissible values exists
            already, False Otherwise.

        viii. Model_path: If pre_existing_model is True, this is path for
            pre-saved model file. If pre_existing_model is False, this field
            can be used for saving the model file. Default NA means there is
            neither pre-saved model file nor there is a need to save one.

        ix. Output_mode: replace or append. "replace" option replaces original
            columns with treated column, whereas "append" option append treated
            column to the input dataset. All treated columns are appended with
            the naming convention - "{original.column.name}_outliered".

    g.  nullColumns_detection

        i.  list_of_cols: "all" can be passed to include all (non-array)
            columns for analysis. "missing" (default) can be passed to
            include only those columns with missing values. One of the use
            cases where "all" may be preferable over "missing" is when
            the user wants to save the imputation model for future use e.g.
            a column may not have missing value in the training dataset.
            Still, missing values may possibly appear in the prediction
            dataset.

        ii. drop_cols: (list format or string of col names separated by |).
            It is used to specify the columns that need to be dropped from
            list_of_cols before null column detection

        iii. treatment: takes Boolean type input -- True or False. If true,
             missing values are treated as per treatment_method argument

        iv. treatment_method: MMM, row_removal or column_removal

        v.  treatment_configs: It takes input in dictionary format with keys
            'treatment_threshold' for column_removal treatment, or all
            arguments corresponding to imputation_MMM function.

## `association_evaluator`

    a.  correlation_matrix

        i.  list_of_cols: (list format or string of col names separated
            by |). It is used to specify the columns which are
            subjected for generating correlation matrix. The user can
            also use "all" as an input to this argument to consider all
            columns. This is super useful instead of specifying all
            column names manually.

        ii. drop_cols: (list format or string of col names separated by
            |). It is used to specify the columns which needs to be
            dropped from list_of_cols. It is most useful when used
            coupled with "all" value of list_of_cols, when we need to
            consider all columns except few handful of them.

    b.  IV_calculation

        i.  List_of_cols: (list format or string of col names separated
            by |). It is used to specify the columns which are
            subjected to IV calculation.

        ii. Drop_cols: (list format or string of col names separated by
            |). It is used to specify the columns that need to be
            dropped from list_of_cols before IV calculation

        iii. Label_col: Name of label or target column in the input
             dataset

        iv. Event_lable: Value of event (label 1) in the label column

        v.  Encoding_configs: It takes input in dictionary format with
            keys related to binning operation - 'bin_method' (default
            'equal_frequency'), 'bin_size' (default 10) and
            'monotonicity_check' (default 0). monotonicity_check of 1
            will dynamically calculate the bin_size ensuring monotonic
            nature and can be expensive operation.

    c.  IG_calculation

        i.  List_of_cols: (list format or string of col names separated
            by |). It is used to specify the columns which are
            subjected to IG calculation

        ii. Drop_cols: (list format or string of col names separated by
            |). It is used to specify the columns that need to be
            dropped from list_of_cols before IG calculation

        iii. Label_col: Name of label or target column in the input
             dataset

        iv. Event_lable: Value of event (label 1) in the label column

        v.  Encoding_configs: It takes input in dictionary format with
            keys related to binning operation - 'bin_method' (default
            'equal_frequency'), 'bin_size' (default 10) and
            'monotonicity_check' (default 0). monotonicity_check of 1
            will dynamically calculate the bin_size ensuring monotonic
            nature and can be expensive operation.

    d.  Variable_clustering

        i.  List_of_cols: (list format or string of col names separated
            by |). It is used to specify the columns which are
            subjected to variable clustering

        ii. Drop_cols: (list format or string of col names separated by
            |). It is used to specify the columns that need to be
            dropped from list_of_cols before variable clustering.

# `drift_detector`

    a.  drift_statistics

        i.  configs

            1.  list_of_cols: List of columns to check drift (list or
                string of col names separated by |). Use 'all' - to
                include all non-array columns (excluding drop_cols).

            2.  drop_cols: List of columns to be dropped (list or string
                of col names separated by |)

            3.  method_type: PSI, JSD, HD, KS (list or string of methods
                separated by |). Use 'all' - to calculate all metrics.

            4.  Threshold: To flag attributes meeting drift threshold

            5.  bin_method: equal_frequency or equal_range

            6.  bin_size: 10 - 20 (recommended for PSI), >100 (other
                method types)

            7.  pre_existing_source: True if binning model & frequency
                counts/attribute exists already, False Otherwise.

            8.  source_path: If pre_existing_source is True, this is
                path for the source dataset details - drift_statistics
                folder. drift_statistics folder must contain
                attribute_binning & frequency_counts folders. If
                pre_existing_source is False, this can be used for
                saving the details. Default "NA" for temporarily
                saving source dataset attribute_binning folder

        ii. source_dataset

            1.  read_dataset

                a.  file_path: file (or directory) path where the source data is saved.
                    File path can be a local path, s3 path (when running with AWS cloud services),
                    google colab path (when running with open source platform: Google Colab),
                    azure dbfs or azure blob storage (when running with Azure databricks).
                    Note: For azure dbfs path should be like "dbfs:/directory_name" and 
                    for azure blob storage path should be like "dbfs:/mnt/directory_name".

                b.  file_type: (CSV, Parquet or Avro). file format of
                    the source dataset.

                c.  file_configs (optional): Rest of the valid
                    configuration can be passed through this key e.g.,
                    delimiter, InferSchema, header.
                    Examples
                        delimiter: ","
                        header: True
                        inferSchema: True

            2.  delete_column: It specifies the columns required to be
                deleted from the source dataframe. Alternatively,
                instead of list, columns can be specified in a single
                text format where different column names are separated
                by pipe delimiter "|"

            3.  select_column: It specifies the columns required to be
                selected from the source dataframe. Alternatively,
                instead of list, columns can be specified in a single
                text format where different column names are separated
                by pipe delimiter "|"

            4.  rename_column

                a.  list_of_cols: It is used to specify the columns
                    required to be renamed in the source dataframe.
                    Alternatively, instead of a list, columns can be
                    specified in a single text format where different
                    column names are separated by pipe delimiter "|"

                b.  list_of_newcols: It is used to specify the new
                    column name, i.e., the first element in list_of_cols
                    will be the original column name, and the
                    corresponding first column in list_of_newcols will
                    be the new column name.

            5.  recast_column:

                a.  list_of_cols: It is used to specify the columns
                    required to be recast in the source dataframe.
                    Alternatively, instead of a list, columns can be
                    specified in a single text format where different
                    column names are separated by pipe delimiter "|"

                b.  list_of_dtypes: It is used to specify the datatype,
                    i.e., the first element in list_of_cols will column
                    name, and the corresponding element in
                    list_of_dtypes will be new datatype such as float,
                    integer, string, double, decimal, etc. (case
                    insensitive).

    b.  stabilityIndex_compuation

        i.  configs

            1.  metric_weightages: A dictionary with key being the
                metric name (mean, stdev, kurtosis) and value being the
                weightage of the metric (between 0 and 1). Sum of all
                weightages must be 1.

            2.  existing_metric_path: path for pre-existing metrics of
                historical datasets <idx, attribute, mean, stdev,
                kurtosis>. idx is index number of historical datasets
                assigned in chronological order

            3.  appended_metric_path: path for saving input dataframes
                metrics after appending to the historical datasets'
                metrics.

            4.  threshold: To flag unstable attributes meeting the
                threshold.

        ii. dataset1

            1.  read_dataset

                a.  file_path: file (or directory) path where the historical dataset is saved.
                    File path can be a local path, s3 path (when running with AWS cloud services),
                    google colab path (when running with open source platform: Google Colab),
                    azure dbfs or azure blob storage (when running with Azure databricks).
                    Note: For azure dbfs path should be like "dbfs:/directory_name" and 
                    for azure blob storage path should be like "dbfs:/mnt/directory_name".

                b.  file_type: (CSV, Parquet or Avro). file format of
                    the historical dataset.

                c.  file_configs (optional): Rest of the valid
                    configuration can be passed through this key e.g.,
                    delimiter, InferSchema, header.

                    Examples
                        delimiter: ","
                        header: True
                        inferSchema: True

        iii. dataset2: same configuration as dataset1

## `report_preprocessing`

    a.  master_path: Path where all modules output is saved

    b.  charts_to_objects

        i.  list_of_cols: (list format or string of col names separated
            by |). It is used to specify the columns which are
            subjected to the analysis in the input dataframe.

        ii. drop_cols: (list format or string of col names separated by
            |). It is used to specify the columns which needs to be
            dropped from list_of_cols

        iii. lable_col: Name of label or target column in the input
             dataset

        iv. event_label: Value of event (label 1) in the label column

        v.  bin_method: equal_frequency or equal_range

        vi. bin_size: 10 - 20 (recommended for PSI), >100 (other method
            types)

        vii. drift_detector: It takes Boolean type input -- True or
            False. It indicates whether the drift component is already
            analyzed or not. By default it is kept as False.

        viii. source_path: The source data path which is needed for
            drift analysis. If it's not computed / out of scope, the
            default value of "NA" is considered.

## `report_generation`

    a.  master_path: The path which contains the data of intermediate
        output in terms of json chart objects, csv file (pandas df).
        Note: In case of azure databricks, azure dbfs path should be like "/dbfs/directory_name" and 
        for azure blob storage path should be like "/dbfs/mnt/directory_name" 
        beacause in report generation all the operations happen in python.

    b.  id_col: The ID column is accepted to ensure & restrict
        unnecessary analysis to be performed on the same lable_col: Name
        of label or target column in the input dataset

    c.  corr_threshold: The threshold chosen beyond which the attributes
        are found to be redundant. It should be between 0 to 1.

    d.  iv_threshold: The threshold beyond which the attributes are
        found to be significant in terms of model. It takes value
        between 0 to 1.

    |**Information Value**|   **Variable Predictiveness**|
    |--- | ---|
    |Less than 0.02    |      Not useful for prediction|
    |0.02 to 0.1       |     Weak predictive Power|
    |0.1 to 0.3        |      Medium predictive Power|
    |0.3 to 0.5        |      Strong predictive Power|
    |>0.5              |    Suspicious Predictive Power|

    e.  drift_threshold_model: The threshold beyond which the attribute can
        be flagged as 1 or drifted as measured across different drift
        metrices specified by the user. It takes value between 0 to 1.

    f.  dataDict_path: The path containing the exact name, definition
        mapping of the attributes. This is eventually used to populate at
        the report for easy referencing. Note: In case of azure databricks, 
        azure dbfs path should be like "/dbfs/directory_name" and 
        For azure blob storage path should be like "/dbfs/mnt/directory_name" 
        beacause in report generation all the operations happen in python.

    g.  metricDict_path: Path of metric dictionary.
        Note: In case of azure databricks, azure dbfs path should be like "/dbfs/directory_name" and 
        For azure blob storage path should be like "/dbfs/mnt/directory_name" 
        beacause in report generation all the operations happen in python.

    h.  final_report_path: Path where final report will be saved. File path can
        be a local path or s3 path (when running with AWS cloud services),
        azure dbfs or azure blob storage (when running with Azure databricks).
        Note: azure dbfs path should be like "/dbfs/directory_name" and 
        For azure blob storage path should be like "/dbfs/mnt/directory_name" 
        beacause in report generation all the operations happen in python.

## `write_intermediate`

    a.  file_path: Path where intermediate datasets (after selecting,
        dropping, renaming, and recasting of columns) for quality
        checker operations, join dataset and concatenate dataset will be
        saved. File path can be a local path, s3 path (when running with AWS cloud services),
        google colab path (when running with open source platform: Google Colab),
        azure dbfs or azure blob storage (when running with Azure databricks).
        Note: For azure dbfs path should be like "dbfs:/directory_name" and 
        for azure blob storage path should be like "dbfs:/mnt/directory_name".

    b.  file_type: (CSV, Parquet or Avro). file format of intermediate
        dataset

    c.  file_configs (optional): Rest of the valid configuration can be
        passed through this key e.g., repartition, mode, compression,
        header, delimiter etc.

        Examples
            mode: overwrite
            header: True
            delimiter: ","
            inferSchema: True

## `write_main`

    a.  file_path: Path where final cleaned input dataset will be saved.
                   File path can be a local path, s3 path (when running with AWS cloud services),
                   google colab path (when running with open source platform: Google Colab),
                   azure dbfs or azure blob storage (when running with Azure databricks).
                   Note: For azure dbfs path should be like "dbfs:/directory_name" and 
                   for azure blob storage path should be like "dbfs:/mnt/directory_name".

    b.  file_type: (CSV, Parquet or Avro). file format of final dataset

    c.  file_configs (optional): Rest of the valid configuration can be
        passed through this key e.g., repartition, mode, compression,
        header, delimiter etc.

        Examples
            mode: overwrite
            header: True
            delimiter: ","
            inferSchema: True

## `write_stats`

    a.  file_path: Path where all tables/stats of anovos modules (data
        drift & data analyzer) will be saved.
        File path can be a local path, s3 path (when running with AWS cloud services),
        google colab path (when running with open source platform: Google Colab),
        azure dbfs or azure blob storage (when running with Azure databricks).
        Note: For azure dbfs path should be like "dbfs:/directory_name" and 
        for azure blob storage path should be like "dbfs:/mnt/directory_name".

    b.  file_type: (CSV, Parquet or Avro). file format of final dataset

    c.  file_configs (optional): Rest of the valid configuration can be
        passed through this key e.g., repartition, mode, compression,
        header, delimiter, inferSchema etc.

        Examples
            mode: overwrite
            header: True
            delimiter: ","
            inferSchema: True

Attaching some links to get more information about file configuration while writing dataset: [write csv files](https://sparkbyexamples.com/pyspark/pyspark-read-csv-file-into-dataframe/), [write parquet files](https://sparkbyexamples.com/pyspark/pyspark-read-and-write-parquet-file/), [write json files](https://sparkbyexamples.com/pyspark/pyspark-read-json-file-into-dataframe/), [write avro files](https://sparkbyexamples.com/spark/read-write-avro-file-spark-dataframe/)
