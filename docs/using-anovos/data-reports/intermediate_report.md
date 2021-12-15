This section largely covers the data pre–processing. The primary function which is used to address all the subsequent modules is **charts_to_objects**. It precisely helps in saving the chart data in form of objects, which is eventually read by the final report generation script. The objects saved are specifically used at the modules shown at the Report based on the user input. Wide variations of chart are used for showcasing the data trends through Bar Plot, Histogram, Violin Plot, Heat Map, Gauge Chart, Line Chart, etc.

Following arguments are specified in the primary function **charts_to_objects**:

- **spark**: Spark session
- **idf**: Input Dataframe
- **list_of_cols**: This argument, in a list format, is used to specify the columns which are subjected to the analysis in the input dataframe. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”. The user can also use “all” as an input to this argument to consider all columns. This is super useful instead of specifying all column names manually.
- **drop_cols**: This argument, in a list format, is used to specify the columns which needs to be dropped from list_of_cols. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”. It is most useful when used coupled with “all” value of list_of_cols, when we need to consider all columns except few handful of them.
- **label_col**: Name of label or target column in the input dataset. By default, the label_col is set as None to accommodate unsupervised use case.
- **event_label**: Value of event (label 1) in the label column. By default, the event_label is kept as 1 unless otherwise specified explicitly.
- **bin_method**: equal_frequency or equal_range. The bin method is set as “equal_range” and is being further referred to the attribute binning function where the necessary aggregation / binning is done. 
- **bin_size**: The maximum number of categories which the user wants to retain is to be set here. By default the size is kept as 10 beyond which remaining records would be grouped under “others”.
- **coverage**: Minimum % of rows mapped to actual category name and rest will be mapped to others. The default value kept is 1.0 which is the maximum at 100%.
- **drift_detector**: This argument takes Boolean type input – True or False. It indicates whether the drift component is already analyzed or not. By default it is kept as False.
- **source_path**: The source data path which is needed for drift analysis. If it’s not computed / out of scope, the default value of "NA" is considered.
- **master_path**: The path which will contain the data of intermediate output in terms of json chart objects, csv file (pandas df)
- **run_type**: local or EMR. Option to specify whether the execution happen locally or in EMR way as it requires reading & writing to s3.

The two form of output generated from this are **chart objects** and **data frame**. There are some secondary functions used alongside as a part of **charts_to_objects** processing.
