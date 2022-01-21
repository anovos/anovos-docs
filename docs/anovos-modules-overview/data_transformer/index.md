## Module ANOVOS.transformers	

In this release, data transformer module supports selected pre-processing functions such binning, encoding, to name a few, which were required for statistics generation and quality checks. However, in the future releases, more exhaustive transformations will be included. List of functions included in this modules are:

- attribute_binning
- monotonic_binning
- cat_to_num_unsupervised
- imputation_MMM
- outlier_categories

Columns which are subjected to these analysis can be controlled by right combination of arguments - list_of_cols and drop_cols. All functions have following common arguments:

- *idf*: Input dataframe
- *list_of_cols*: This argument, in a list format, is used to specify the columns which are subjected to the analysis in the input dataframe. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”. The user can also use “all” as an input to this argument to consider all valid columns. This is super useful instead of specifying all column names manually.
- *drop_cols*: This argument, in a list format, is used to specify the columns which needs to be dropped from list_of_cols. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”. It is most useful when used coupled with “all” value of list_of_cols, when we need to consider all columns except few handful of them.
- *output_mode*: replace or append. “replace” option replaces original columns with transformed column, whereas “append” option append transformed column to the input dataset. 
- *print_impact*: This argument is to print out the statistics.

### attribute_binning

Attribute binning (or discretization) is a method of numerical attribute into discrete (integer or categorical values) using pre-defined number of bins. This data pre-processing technique is used to reduce the effects of minor observation errors. Also, Binning introduces non-linearity and tends to improve the performance of the model. In this function, we are focussing on unsupervised way of binning i.e. without considering the target variable into account - Equal Range Binning, Equal Frequency Binning. In Equal Range method, each bin is of equal size/width and computed as:

w = max- min / no. of bins 

*bins cutoff=[min, min+w,min+2w…..,max-w,max]*

whereas in Equal Frequency binning method, bins are created in such a way that each bin has equal no. of rows, though the width of bins may vary from each other.

w = 1 / no. of bins

*bins cutoff=[min, wthpctile, 2wthpctile….,max ]*

- *idf*
- *list_of_cols:* If ‘all’ is passed for this argument, then only numerical attributes are selected.
- *drop_cols*
- *method_type: equal_frequency, equal_range*
- *bin_size: Number of bins*
- *bin_dtype:* numerical, categorical*.* Original value is replaced with Integer (1,2,…) with ‘numerical’ input and replaced with string describing min and max value observed in the bin ("minval-maxval")
- *pre_existing_model:* This argument takes Boolean type input – True or False. True if the file with bin cutoff values exists already, False Otherwise.
- *model_path:* If pre_existing_model is True, this argument is path for pre-saved model file. If pre_existing_model is False, this field can be used for saving the model file. Default NA means there is neither pre-saved model file nor there is a need to save one.
- *output_mode:* All transformed columns are appended with the naming convention - "{original.column.name}_binned".
- *print_impact*

### monotonic_binning

This function constitutes supervised way of binning the numerical attribute into discrete (integer or categorical values) attribute. Instead of pre-defined fixed number of bins, number of bins are computed dynamically ensuring the monotonic nature of bins i.e. % event should increase or decrease with the bin. Monotonic nature of bins is evaluated by looking at spearman rank correlation, which should be either +1 or -1, between the bin index and % event. In case, the monotonic nature is not attained, user defined fixed number of bins are used for the binning.

- *idf*
- *list_of_cols:* If 'all' is passed for this argument, then only numerical attributes are selected.
- *drop_cols*
- *label_col*: Name of label or target column in the input dataset
- *event_label*: Value of event (label 1) in the label column
- *method_type*: equal_frequency, equal_range_
- *bin_size*: Number of bins_
- *bin_dtype:* numerical, categorical_._ Original value is replaced with Integer (1,2,…) with 'numerical' input and replaced with string describing min and max value observed in the bin '&lt;bin_cutoffi &gt;- &lt;bin_cutoffi+1&gt;'
- *output_mode:* All transformed columns are appended with the naming convention -  "{original.column.name}_binned".

### cat_to_num_unsupervised

This is unsupervised method of converting a categorical attribute into numerical attribute(s). This is among the most important transformations required for any modelling exercise, as most of the machine learning algorithms cannot process categorical values. It covers two popular encoding techniques – label encoding & one-hot encoding.

In label encoding, each categorical value is assigned a unique integer based on alphabetical or frequency ordering (both ascending & descending options are available – can be selected by index_order argument). One of the pitfalls of using this technique is that the model may learn some spurious relationship, which doesn't exist or make logical sense in the real world. In one-hot encoding, every unique value in the attribute will be added as a feature in a form of dummy/binary attribute. However, using this method on high cardinality attributes can further aggravate the dimensionality issue.

- *idf*
- *list_of_cols:* If 'all' is passed for this argument, then only categorical attributes are selected.
- *drop_cols*
- *method_type:* 1 (for Label Encoding) or 0 (for One hot encoding)
- *index_order:* frequencyDesc, frequencyAsc, alphabetDesc, alphabetAsc (Valid only for Label Encoding)
- *onehot_dropLast:* This argument takes Boolean type input – True or False. if True, it drops one last column in one hot encoding
- *pre_existing_model:* This argument takes Boolean type input – True or False. True if the encoding models exist already, False Otherwise.
- *model_path:* If pre_existing_model is True, this argument is path for pre-saved model. If pre_existing_model is False, this field can be used for saving the mode. Default NA means there is neither pre-saved model nor there is a need to save one.
- *output_mode:* All transformed columns are appended with the naming convention - "{original.column.name}_index" for label encoding. &amp; "{original.column.name}_{n}" for one hot encoding, n varies from 0 to unique value count.
- *print_impact*

### imputation_MMM

This function handles missing value related issues by substituting null values by the measure of central tendency (mode for categorical features and mean/median for numerical features). For numerical attributes, it leverages [Imputer](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.ml.feature.Imputer.html) functionality of Spark MLlib. Though, Imputer can be used for categorical attributes but this feature is available only in Spark3.x, therefore for categorical features, we compute mode or leverage mode computation from Measures of Central Tendency.

- *idf*
- *list_of_cols*: 'missing' can be used for this argument, in which case, it will analyse only those columns with any missing value.
- *drop_cols*
- *method_type*: median (default), mean. Valid only for Numerical attributes.
- *pre_existing_model*: This argument takes Boolean type input – True or False. True if the encoding models exist already, False Otherwise.
- *model_path*: If pre_existing_model is True, this argument is path for pre-saved model. If pre_existing_model is False, this field can be used for saving the mode. Default NA means there is neither pre-saved model nor there is a need to save one.
- *output_mode*: All transformed columns are appended with the naming convention - "{original.column.name}_imputed".
- *stats_missing*: Arguments corresponding to read_dataset function in dictionary format, to read output from measures_of_counts function of stats generator
- *stats_mode*: Arguments corresponding to read_dataset function in dictionary format, to read output from measures_of_centralTendency function of stats generator
- *print_impact*

### outlier_categories

This function replaces less frequently seen values (called as outlier values in the current context) in a categorical column by 'others'. Outlier values can be defined in two ways – a) Max N categories, where N is used defined value. In this method, top N-1 frequently seen categories are considered and rest are clubbed under single category 'others'. or Alternatively, b) Coverage – top frequently seen categories are considered till it covers minimum N% of rows and rest lesser seen values are mapped to mapped to others. Even if the Coverage is less, maximum category constraint is given priority. Further, there is a caveat that when multiple categories have same rank. Then, number of categorical values can be more than max_category defined by the user.

- *idf*
- *list_of_cols*: 'missing' can be used for this argument, in which case, it will analyse only those columns with any missing value.
- *drop_cols*
- *coverage*: Minimum % of rows mapped to actual category name and rest will be mapped to others
- *max_category*: Maximum number of categories allowed
- *pre_existing_model*: This argument takes Boolean type input – True or False. True if the file with outlier values exist already for each attribute, False Otherwise.
- *model_path*: If pre_existing_model is True, this argument is path for pre-saved model file. If pre_existing_model is False, this field can be used for saving the model file. Default NA means there is neither pre-saved model nor there is a need to save one.
- *output_mode*: All transformed columns are appended with the naming convention - "{original.column.name}_ outliered ".
- *print_impact*

