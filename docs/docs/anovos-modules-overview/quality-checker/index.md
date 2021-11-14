## Module **ANOVOS.quality_checker**

This submodule focus on assessing the data quality at both row level and column level and also provides an appropriate treatment option to fix those quality issues. 

Columns which are subjected to these analysis can be controlled by right combination of arguments - list_of_cols and drop_cols. All functions have following common arguments: 

- idf: Input dataframe 
- list_of_cols: This argument, in a list format, is used to specify the columns which are subjected to the analysis in the input dataframe. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”. The user can also use “all” as an input to this argument to consider all columns. This is super useful instead of specifying all column names manually. 
- drop_cols: This argument, in a list format, is used to specify the columns which needs to be dropped from list_of_cols. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”. It is most useful when used coupled with “all” value of list_of_cols, when we need to consider all columns except few handful of them. 
- print_impact: This argument is to print out the statistics. 

At row level, the following checks are done: 

- duplicate_detection 
- nullRows_detection 

At column level, the following checks are done: 

- nullColumns_detection 
- outlier_detection 
- IDness_detection 
- biasedness_detection 
- invalidEntries_detection 

### duplicate_detection 

As the name suggests, this function detects duplication in the input dataset. This means, for a pair of duplicate rows, the values in each column coincide. Duplication check is confined to the list of columns passed in the arguments. As the part of treatment, duplicated rows are removed. This function returns two dataframes in tuple format – 1st dataframe is input dataset after deduplication (if treated) and  2nd dataframe is of schema – metric, value and contains total number of rows and number of unique rows. 

- idf 
- list_of_cols 
- drop_cols 
- treatment: This argument takes Boolean type input – True or False. If true, duplicate rows are removed from the input dataset. 
- print_impact 

### nullRows_detection 

This function inspects the row quality and computes number of columns which are missing for a row. This metric is further aggregated to check how many columns are missing for how many rows (also at % level). Intuition is if too many columns are missing for a row, removing it from the modelling may give better results than relying on its imputed values. Therefore as the part of treatment, rows with missing columns above the specified threshold are removed. This function returns two dataframes in tuple format – 1st dataframe is input dataset after filtering rows with high number of missing columns (if treated) and 2nd dataframe is of schema – null_cols_count, row_count, row_pct, flagged.

| null_cols_count | row_count | row_pct | flagged |
| --- | --- | --- | --- |
| 5 | 11 | 3.0E-4 | 0 |
| 7 | 1306 | 0.0401 | 1 |



Interpretation: 1306 rows (4.01% of total rows) have 7 missing columns and flagged for removal because null\_cols\_count is above the threshold.


- idf 
- list_of_cols 
- drop_cols 
- treatment: This argument takes Boolean type input – True or False. If true, rows with high null columns (defined by treatment_threshold argument) are removed from the input dataset. 
- treatment_threshold: This argument takes value between 0 to 1 with default 0.8, which means 80% of columns allowed to be Null per row. If it is more than the threshold, then it is flagged and if treatment is True, then affected rows are removed. If threshold is 0, it means, rows with any missing value will be flagged. If threshold is 1, it means rows with all missing value will be flagged. 
- print_impact 

### nullColumns_detection 

This function inspects the column quality and computes number of rows which are missing for a column. This function also leverages statistics which were computed as the part of the State Generator module so that statistics are not computed twice if already available.  

As part of treatments, it currently supports 3 methods – Mean Median Mode (MMM), row_removal or column_removal (more methods to be added soon). MMM replaces null value by the measure of central tendency (mode for categorical features and mean/median for numerical features). row_removal removes all rows with any missing value (output of this treatment is same as nullRows_detection with treatment_threshold of 0). column_removal remove a column if %rows with missing value is above treatment_threshold. 

This function returns two dataframes in tuple format – 1st dataframe is input dataset after imputation (if treated else the original dataset) and  2nd dataframe is of schema – attribute, missing_count, missing_pct. 

- **idf**
- **list_of_cols**: "all" can be passed to include all (non-array) columns for analysis. "missing" (default) can be passed to include only those columns with missing values. One of the use cases where "all" may be preferable over "missing" is when the user wants to save the imputation model for the future use e.g. a column may not have missing value in the training dataset but missing values may possibly appear in the prediction dataset. 
- **drop_cols**
- **treatment**: This argument takes Boolean type input – True or False. If true, missing values are treated as per treatment_method argument
- **treatment_method**: MMM, row_removal or column_removal 
- **treatment_configs**: This argument takes input in dictionary format with keys – ‘treatment_threshold’ for column_removal treatment, or all arguments corresponding to imputation_MMM function. 
- **stats_missing**: Arguments corresponding to read_dataset function in dictionary format, to read output from measures_of_counts function of stats generator 
- **stats_unique**: Arguments corresponding to read_dataset function in dictionary format, to read output from measures_of_cardinality function of stats generator 
- **stats_mode**: Arguments corresponding to read_dataset function in dictionary format, to read output from measures_of_centralTendency function of stats generator 
**print_impact**

### outlier_detection 

In Machine Learning, outlier detection is the identification of values that deviates drastically from the rest of the attribute values. An outlier may be caused simply by chance, measurement error or inherent heavy-tailed distribution. This function identify extreme values in both directions (or any direction provided by the user via detection_side argument). Outlier is identified by 3 different methodologies and tagged an outlier only if it is validated by at least 2 methodologies (can be changed by the user via min_validation under detection_configs argument). 

- Percentile Method: In this methodology, a value higher than a certain (default 95th) percentile value is considered as an outlier. Similarly, a value lower than a certain (default 5th) percentile value is considered as an outlier. 
- Standard Deviation Method: In this methodology, if a value is certain number of standard deviations (default 3) away from the mean, then it is identified as an outlier. 
- Interquartile Range (IQR) Method: A value which is below Q1 – 1.5 IQR or above Q3 + 1.5 IQR are identified as outliers, where Q1 is first quantile/25th percentile, Q3 is third quantile/75th percentile and IQR is difference between third quantile & first quantile. 

This function also leverages statistics which were computed as the part of the State Generator module so that statistics are not computed twice if already available. 

As part of treatments available, outlier values can be replaced by null so that it can be imputed by a reliable imputation methodology (null_replacement). It can also be replaced by maximum or minimum permissible by above methodologies (value_replacement). Lastly, rows can be removed if it is identified with any outlier (row_removal). 

This function returns two dataframes in tuple format – 1st dataframe is input dataset after treating outlier (the original dataset if no treatment) and  2nd dataframe is of schema – attribute, lower_outliers, upper_outliers. If outliers are checked only for upper end, then lower_outliers column will be shown all zero. Similarly if checked only for lower end, then upper_outliers will be zero for all attributes. 

- **idf** 
- **list_of_cols**: Any attribute with single value or all null values are not subjected to outlier detection even if it is selected under this argument. 
- **drop_cols**
- **detection_side**: upper, lower, both 
- **detection_configs**: This argument takes input in dictionary format with keys (representing upper and lower bound for different outlier identification methodologies) - pctile_lower (default 0.05), pctile_upper (default 0.95), stdev_lower (default 3.0), stdev_upper (default 3.0), IQR_lower (default 1.5), IQR_upper (default 1.5), min_validation (default 2) 
- **treatment**: This argument takes Boolean type input – True or False. If true, specified treatment method is applied. 
- **treatment_method**: null_replacement, row_removal, value_replacement 
- **pre_existing_model**: This argument takes Boolean type input – True or False. True if the file with upper/lower permissible values exists already, False Otherwise. 
- **model_path**: If pre_existing_model is True, this argument is path for pre-saved model file. If pre_existing_model is False, this field can be used for saving the model file. Default NA means there is neither pre-saved model file nor there is a need to save one. 
- **output_mode**: replace or append. “replace” option replaces original columns with treated column, whereas “append” option append treated column to the input dataset. All treated columns are appended with the naming convention - "{original.column.name}_outliered". 
- **stats_unique**: Arguments corresponding to read_dataset function in dictionary format, to read output from measures_of_cardinality function of stats generator 
- **print_impact**

### IDness_detection 

IDness of an attribute is defined as the ratio of number of unique values seen in an attribute by number of non-null rows. It varies between 0 to 100% where IDness of 100% means there are as many unique values as number of rows (primary key in the input dataset). IDness is computed only for categorical features. This function leverages the statistics from Measures of Cardinality function and flag the columns if IDness is above a certain threshold. Such columns can be deleted from the modelling analysis if directed for a treatment. 

This function returns two dataframes in tuple format – 1st dataframe is input dataset after removing high IDness columns (the original dataset if no treatment) and  2nd dataframe is of schema – attribute, , unique_values, IDness. 

- **idf** 
- **list_of_cols**
- **drop_cols** 
- **treatment**: This argument takes Boolean type input – True or False. If true, columns above IDness threshold are removed. 
- **treatment_threshold**: This argument takes value between 0 to 1 with default 1.0. 
- **stats_unique**: Arguments corresponding to read_dataset function in dictionary format, to read output from measures_of_cardinality function of stats generator. 
- **print_impact**

### biasedness_detection 

This function flags column if they are biased or skewed towards one specific value and is equivalent to mode_pct computation from Measures of Central Tendency i.e. number of rows with mode value (most frequently seen value) divided by number of non-null values. It varies between 0 to 100% where biasedness of 100% means there is only a single value (other than null). The function flags a column if its biasedness is above a certain threshold. Such columns can be deleted from the modelling analysis, if required. 

This function returns two dataframes in tuple format – 1st dataframe is input dataset after removing high biased columns (the original dataset if no treatment) and  2nd dataframe is of schema – attribute, mode, mode_pct. 

- **idf**
- **list_of_cols** 
- **drop_cols** 
- **treatment**: This argument takes Boolean type input – True or False. If true, columns above biasedness threshold are removed. 
- **treatment_threshold**: This argument takes value between 0 to 1 with default 1.0. 
- **stats_mode**: Arguments corresponding to read_dataset function in dictionary format, to read output from measures_of_centralTendency function of stats generator. 
- **print_impact** 

###	invalidEntries_detection

This function checks for certain suspicious patterns in attributes’ values. These suspicious values can be replaced as null and treated as missing. Patterns that are considered for this quality check:

- Missing Values: The function checks for all text strings which directly or indirectly indicate the missing value in an attribute. Currently, we check the following string values - '', ' ', 'nan', 'null', 'na', 'inf', 'n/a', 'not defined', 'none', 'undefined', 'blank'. The function also check for special characters such as ?, *, to name a few.
- Repetitive Characters: Certain attributes’ values with repetitive characters may be default value or system error, rather than being a legit value etc xx, zzzzz, 99999 etc. Such values are flagged for the user to take an appropriate action. There may be certain false positive which are legit values.
- Consecutive Characters: Similar to repetitive characters, consecutive characters (at least 3 characters long) such as abc, 1234 etc may not be legit values, and hence flagged. There may be certain false positive which are legit values.

This function returns two dataframes in tuple format – 1st dataframe is input dataset after replacing flagged values as null (or the original dataset if no treatment) and  2nd dataframe is of schema – attribute, invalid_entries, invalid_count, invalid_pct. All potential invalid values (separated by delimiter pipe “|”) are shown under invalid_entries column. Total number of rows impacted by these entries for each attribute is shown under invalid_count. invalid_pct is invalid_count divided by number of rows

- **idf**
- **list_of_cols**
- **drop_cols**
- **treatment**: This argument takes Boolean type input – True or False. If true, columns above biasedness threshold are removed.
- **output_mode**: replace or append. “replace” option replaces original columns with treated column, whereas “append” option append treated column to the input dataset. All treated columns are appended with the naming convention - "{original.column.name}_cleaned".
- **print_impact**

