# Creating Data Reports with Anovos

_Anovos_ includes capabilities to generate comprehensive _Data Reports_ that describe a dataset
and its processing. Data reports are an important component of many data governance concepts.

## 📑 How are reports generated?

_Anovos_ generates reports in two steps:

1. The data that will be included in the report is generated using the functions of the 
   [`data_analyzer`](../../docs/anovos-modules-overview/data-analyzer/index.md)[`association_evaluator`](../../docs/anovos-modules-overview/association-evaluator/index.md)[`data_drift_stability`](../../docs/anovos-modules-overview/data_drift_and_stability_index/index.md)[`quality_checker`](../../docs/anovos-modules-overview/quality-checker/index.md) module.
   As all _Anovos_ data operations, this happens in a distributed fashion,
   fully utilizing the power of _Apache Spark_.
   We call the result the "intermediate report."

2. The generated data is processed and the final report is generated.

You can configure a report and trigger its generation in two ways:
Using the configuration file, or through individual modules.

## 📋 Generating data reports via the configuration file

### Basic Report

In case you do not need an exhaustive report that contains all the detailed outputs of the
[`data_analyzer`](../../docs/anovos-modules-overview/data-analyzer/index.md),
[`association_evaluator`](../../docs/anovos-modules-overview/association-evaluator/index.md),
[`data_drift_stability`](../../docs/anovos-modules-overview/data_drift_and_stability_index/index.md), and
[`quality_checker`](../../docs/anovos-modules-overview/quality-checker/index.md),
you can opt to generate a concise, but nevertheless fairly comprehensive basic report by adding the
`anovos_basic_report` configuration block to the configuration file.

Setting the `basic_report` option to `True` enables this functionality. 
You can further explicitly specify the input details such as `id_col`, `label_col`, and `event_label`
as well as the `output_path` for the report.

```yaml
anovos_basic_report:
  basic_report: True
  report_args:
    id_col: 
    label_col: 
    event_label: 
    output_path: 
```

### Full Report

The detailed and exhaustive full report contains a structured and well-formatted outputs of the
[`data_analyzer`](../../docs/anovos-modules-overview/data-analyzer/index.md),
[`association_evaluator`](../../docs/anovos-modules-overview/association-evaluator/index.md),
[`data_drift_stability`](../../docs/anovos-modules-overview/data_drift_and_stability_index/index.md), and
[`quality_checker`](../../docs/anovos-modules-overview/quality-checker/index.md).
These are displayed along with eye-catching visualizations that make it easy to capture data trends across different cut points.

The full report is configured through two blocks in the configuration file: `report_preprocessing` and  `report_generation`.

The `report_preprocessing` includes a mandatory `master_path` setting which specifies the location
the data generated for the report is stored as it is computed by the different modules.

The `charts_to_objects` sub-block specifies the parameters passed to the different preprocessing and analysis functions. 

```yaml
report_preprocessing:
  master_path:                  # path where the report is stored
  charts_to_objects:
    list_of_cols: all           # the columns to include in the report
    drop_cols:                  # the columns to drop
    label_col:                  # the label column
    event_label:                # the event label
    bin_method:                 # method used for binning (either "equal_frequency" or "equal_range")
    bin_size:                   # the number of bins
    drift_detector: True        # whether to analyze for data drift
    source_path:
```

The `report_generation` block, we have the need for `master_path` which is the same as specified above.

The user is also needed to specify `id_col`, `label_col`.
Alongside some benchmarking thresholds like `corr_threshold`, `iv_threshold` & `drift_threshold_model` are needed for highlighting various association analysis checks.

Some of the file locations are needed to be specified by the user such as `dataDict_path` & `metricDict_path` basis which some of the reporting sections are updated. 
Finally, the user can specify the `final_report_path` where the report would be saved.

```yaml
report_generation:
  master_path:                  # path where the report is stored
  id_col:
  label_col:                    # the label column
  corr_threshold:
  iv_threshold:
  drift_threshold_model:
  dataDict_path:
  metricDict_path:
  final_report_path:
```

For an example, see [the `configs.yaml` for the Anovos demo run](https://github.com/anovos/anovos/blob/main/config/configs.yaml).

To control if and how the raw data that is included in the report is saved,
you can add `write_`-blocks to your configuration file:

```yaml
write_intermediate:
  file_path: "intermediate_data"
  file_type: csv
  file_configs:
    mode: overwrite
    header: True
    delimiter: ","
    inferSchema: True

write_main:
  file_path: "output"
  file_type: parquet
  file_configs:
    mode: overwrite
    
write_stats:
  file_path: "stats"
  file_type: parquet
  file_configs:
    mode: overwrite
```

## 🦄 Generating data reports through individual modules

*Here the user can pick up the relevant functions specific to the reporting module and execute*
