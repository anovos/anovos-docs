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

*Here the user specifies the desired options / input parameters as done for other modules*

- Basic Report : In case the user doesn't want an exhaustive report containing all the detailed elements of the [`data_analyzer`](../../docs/anovos-modules-overview/data-analyzer/index.md)[`association_evaluator`](../../docs/anovos-modules-overview/association-evaluator/index.md)[`data_drift_stability`](../../docs/anovos-modules-overview/data_drift_and_stability_index/index.md)[`quality_checker`](../../docs/anovos-modules-overview/quality-checker/index.md) output, the user may wish to choose a fairly comprehensive view which can be achieved through `anovos_basic_report`. To generate that, there's only a small block which needs to be updated at the config file. Setting the `basic_report` option as `True` can enable this functionality. Alongside, the user should explicity specify the input parameters details such as `id_col`, `label_col`, `event_label` & the `output_path`.

```yaml
anovos_basic_report:
  basic_report: True
  report_args:
    id_col: 
    label_col: 
    event_label: 
    output_path: 
```

- Overall Report : The detailed & the exhaustive view form can be achieved through the Overall Report. A structured & a well formatted output from the [`data_analyzer`](../../docs/anovos-modules-overview/data-analyzer/index.md)[`association_evaluator`](../../docs/anovos-modules-overview/association-evaluator/index.md)[`data_drift_stability`](../../docs/anovos-modules-overview/data_drift_and_stability_index/index.md)[`quality_checker`](../../docs/anovos-modules-overview/quality-checker/index.md) is displayed along with eye catching visualizations thereby making it easier to capture data trends across different cut points. There are precisely two sections which needed to be updated viz. `report_preprocessing` & `report_generation`.
- Inside `report_preprocessing` block we've the requirement for `master_path` which is precisely the location where all the reporting stats are saved as computed from the different modules. Likewise, the `charts_to_objects` also seeks for the details pertaining to the data & preprocessing options. 

```yaml
report_preprocessing:
  master_path: 
  charts_to_objects:
    list_of_cols: all
    drop_cols: 
    label_col: 
    event_label: 
    bin_method: 
    bin_size: 
    drift_detector: True
    source_path:
```

- On the other hand, inside `report_generation` block, we have the need for `master_path` which is the same as specified above. The user is also needed to specify `id_col`, `label_col`. Alongside some benchmarking thresholds like `corr_threshold`, `iv_threshold` & `drift_threshold_model` are needed for highlighting various association analysis checks. Some of the file locations are needed to be specified by the user such as `dataDict_path` & `metricDict_path` basis which some of the reporting sections are updated. Finally, the user can specify the `final_report_path` where the report would be saved.

```yaml
report_generation:
  master_path:
  id_col: 
  label_col: 
  corr_threshold: 
  iv_threshold: 
  drift_threshold_model: 
  dataDict_path: 
  metricDict_path: 
  final_report_path: 
```

For an example, see [the `configs.yaml` for the Anovos demo run](https://github.com/anovos/anovos/blob/main/config/configs.yaml).

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
