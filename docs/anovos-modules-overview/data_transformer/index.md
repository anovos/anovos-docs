## Module ANOVOS.transformers	

In the latest release of Anovos (V0.2), data transformer module supports selected pre-processing functions such as binning, encoding, scaling, imputation, to name a few, which were required for statistics generation and quality checks. List of functions supported through this modules are:

- attribute_binning
- monotonic_binning
- cat_to_num_unsupervised
- cat_to_num_supervised
- z_standardization
- IQR_standardization
- normalization
- imputation_MMM
- imputation_sklearn
- imputation_matrixFactorization
- auto_imputation
- autoencoder_latentFeatures
- PCA_latentFeatures
- feature_transformation
- boxcox_transformation
- outlier_categories
- expression_parser

Columns which are subjected to these analysis can be controlled by right combination of arguments - list_of_cols and drop_cols. Most of the functions have following common arguments:

- *idf*: Input dataframe
- *list_of_cols*: This argument, in a list format, is used to specify the columns which are subjected to the analysis in the input dataframe. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”. The user can also use “all” as an input to this argument to consider all valid columns. This is super useful instead of specifying all column names manually.
- *drop_cols*: This argument, in a list format, is used to specify the columns which needs to be dropped from list_of_cols. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”. It is most useful when used coupled with “all” value of list_of_cols, when we need to consider all columns except few handful of them.
- *output_mode*: replace or append. “replace” option replaces original columns with transformed column, whereas “append” option append transformed column to the input dataset. 
- *print_impact*: This argument is to print out the statistics.

### attribute_binning

Attribute binning (or discretization) is a method of numerical attribute into discrete (integer or categorical values) using pre-defined number of bins. This data pre-processing technique is used to reduce the effects of minor observation errors. Also, Binning introduces non-linearity and tends to improve the performance of the model. In this function, we are focussing on unsupervised way of binning i.e. without considering the target variable into account - Equal Range Binning, Equal Frequency Binning. In Equal Range method, each bin is of equal size/width and computed as:

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

This function constitutes supervised way of binning the numerical attribute into discrete (integer or categorical values) attribute. Instead of pre-defined fixed number of bins, number of bins are dynamically computed to ensure the monotonic nature of bins i.e. % event should increase or decrease with the bin. Monotonic nature of bins is evaluated by looking at spearman rank correlation, which should be either +1 or -1, between the bin index and % event. In case, the monotonic nature is not attained, user defined fixed number of bins are used for the binning.

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

In label encoding, each categorical value is assigned a unique integer based on alphabetical or frequency ordering (both ascending & descending options are available – can be selected by index_order argument). One of the pitfalls of using this technique is that the model may learn some spurious relationship, which doesn't exist or might not make any logical sense in the real world settings. In one-hot encoding, every unique value in the attribute will be added as a feature in a form of dummy/binary attribute. However, using this method on high cardinality attributes can further aggravate the dimensionality issue.

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

### cat_to_num_supervised

This is a supervised method to convert a categorical attribute into a numerical attribute. It takes a label/target column to indicate whether the event is positive or negative. For each column, the positive event rate for each categorical value is used as the encoded numerical value.

For example, there are 3 distinct values in a categorical attribute X: X1, X2 and X3. Within the input dataframe, there are 
- 15 positive events and 5 negative events with X==X1;
- 10 positive events and 40 negative events with X==X2;
- 20 positive events and 20 negative events with X==X3.

Thus, value X1 is mapped to 15/(15+5) = 0.75, value X2 is mapped to 10/(10+40) = 0.2 and value X3 is mapped to 20/(20+20) = 0.5. This mapping will be applied to all values in attribute X. This encoding method can avoiding creating too many dummy variables which may cause dimensionality issue and it also works with categorical attributes without an order or rank.

- *spark*: Spark Session
- *idf*
- *list_of_cols*: If 'all' is passed for this argument, then only categorical attributes are selected.
- *drop_cols*
- *label_col*: Label/Target column
- *event_label:* Value of (positive) event (i.e label 1)
- *pre_existing_model*: Boolean argument - True or False. True if model (original and mapped numerical value for each column) exists already, False Otherwise.
- *model_path*: If pre_existing_model is True, this argument is path for referring the pre-saved model. If pre_existing_model is False, this argument can be used for saving the model. Default "NA" means there is neither pre-existing model nor there is a need to save one.
- *output_mode*: "append" option appends transformed column with the naming convention - "{original.column.name}_encoded".
- *print_impact*

### z_standardization
standardization is commonly used in data pre-processing process. z_standardization standardizes the selected attributes of an input dataframe by normalizing each attribute to have standard deviation of 1 and mean of 0. For each attribute, the standard deviation (s) and mean (u) are calculated and a sample x will be standardized into (x-u)/s. If the standard deviation of an attribute is 0, it will be excluded in standardization and a warning will be shown. None values will be kept as None in the output dataframe.

- *spark*: Spark Session
- *idf*
- *list_of_cols*: "all" can be passed to include all numerical columns.
- *drop_cols*
- *pre_existing_model*: Boolean argument - True or False. True if model files (Mean/stddev for each feature) exists already, False otherwise
- *model_path*: If pre_existing_model is True, this argument is path for referring the pre-saved model. If pre_existing_model is False, this argument can be used for saving the model. Default "NA" means there is neither pre-existing model nor there is a need to save one.
- *output_mode*: "append" option appends transformed column with the naming convention - "{original.column.name}_scaled".
- *print_impact*

### IQR_standardization

IQR_standardization removes the median of an attribute and scales the attribute using its Interquartile Range(IQR), which is the quantile range between the 1st quartile and the 3rd quartile. IQR_standardization is similar to z_standardization but it is more robust to outliers. For each attribute, the IQR and median are calculated and a sample x will be standardized into (x-median)/IQR. If the 1st quartile and the 3rd quartile of an attribute are the same, it will be excluded in standardization and a warning will be shown. None values will be kept as None in the output dataframe.

- *spark*: Spark Session
- *idf*
- *list_of_cols*: "all" can be passed to include all numerical columns.
- *drop_cols*
- *pre_existing_model*: Boolean argument - True or False. True if model files (25/50/75 percentile for each feature) exists already, False Otherwise.
- *model_path*: If pre_existing_model is True, this argument is path for referring the pre-saved model. If pre_existing_model is False, this argument can be used for saving the model. Default "NA" means there is neither pre-existing model nor there is a need to save one.
- *output_mode*: "append" option appends transformed column with the naming convention "{original.column.name}_scaled".
- *print_impact*


### normalization

Normalization is a scaling technique which transforms each attribute to a range - [0, 1]. It uses the  pyspark.ml.feature.MinMaxScaler function. Thus, the rescaled value for attribute X is calculated as, Rescaled(x_i) = (x_i - X_min) / (X_max - X_min). If X_max equals to X_min, Rescaled(x_i) will be 0.5. In additiona, None values will be kept as None in the output dataframe.

- *idf*
- *list_of_cols*: "all" can be passed to include all numerical columns.
- *drop_cols*
- *pre_existing_model*: Boolean argument - True or False. True if normalization/scalar model exists already, False Otherwise
- *model_path*: If pre_existing_model is True, this argument is path for referring the pre-saved model. If pre_existing_model is False, this argument can be used for saving the model. Default "NA" means there is neither pre-existing model nor there is a need to save one.
- *output_mode*: "append" option appends transformed column with the naming convention - "{original.column.name}_scaled".
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

### imputation_sklearn

imputation_sklearn trains a sklearn imputer to handle missing values in numerical columns. It learns how to impute the missing value of a sample using the rest of the samples. Two methods are supported: “KNN” and “regression”. 

“KNN” option trains a sklearn.impute.KNNImputer which is based on k-Nearest Neighbors algorithm. The missing values of a sample are imputed using the mean of its 5 nearest neighbors in the training set. “regression” option trains a sklearn.impute.IterativeImputer which models attribute to impute as a function of rest of the attributes and imputes using the estimation. Imputation is performed in an iterative way from attributes with fewest number of missing values to most. All the hyperparameters used in the above mentioned imputers are their default values.

However, sklearn imputers are not scalable, which might be slow if the size of the input dataframe is large. Thus, an input sample_size (the default value is 500,000) can be set to control the number of samples to be used to train the imputer. If the total number of samples exceeds sample_size, the rest of the samples will be imputed using the trained imputer in a scalable manner. 

- *spark*: Spark Session
- *idf*
- *list_of_cols*: "all" can be passed to include all numerical columns. "missing" can be used to only include numerical columns with any missing value.
- *drop_cols*
- *method_type*: "KNN", "regression". "KNN" option trains a sklearn.impute.KNNImputer. "regression" option trains a sklearn.impute.IterativeImputer.
- *sample_size*: Maximum rows for training the sklearn imputer  
- *pre_existing_model*: Boolean argument - True or False. True if imputation model exists already, False otherwise.
- *model_path*: If pre_existing_model is True, this argument is path for referring the pre-saved model. If pre_existing_model is False, this argument can be used for saving the model. Default "NA" means there is neither pre-existing model nor there is a need to save one.
- *output_mode*: "append" option appends transformed column with the naming convention - "{original.column.name}_imputed".
- *stats_missing*: Takes arguments for read_dataset (data_ingest module) function in a dictionary format to read pre-saved statistics on missing count/pct i.e. if measures_of_counts or missingCount_computation (data_analyzer.stats_generator module) has been computed & saved before.
- *emr_mode*: Boolean argument - True or False. True if it is run on EMR, False otherwise.
- *print_impact*

### imputation_matrixFactorization

imputation_matrixFactorization uses collaborative filtering technique to impute missing values. Collaborative filtering is commonly used in recommender systems to fill the missing user-item entries and PySpark provides an implementation using alternating least squares (ALS) algorithm, which is used in this function. To fit our problem into the ALS model, each attribute is treated as an item and an id columns needs to be specified by the user to generate the user-item pairs. Subsequently, all user-item pairs with known values will be used to train the ALS model and the trained model can be used to predict the user-item pairs with missing values.

- *spark*: Spark Session
- *idf*
- *list_of_cols*: "all" can be passed to include all numerical columns.  "missing" can be used to only include numerical columns with any missing value.
- *drop_cols*
- *id_col*: name of the column representing ID. "" (by default) can be used if there is no ID column.
- *output_mode*: "append" option appends transformed column with the naming convention - "{original.column.name}_imputed".
- *stats_missing*: Takes arguments for read_dataset (data_ingest module) function in a dictionary format to read pre-saved statistics on missing count/pct i.e. if measures_of_counts or missingCount_computation (data_analyzer.stats_generator module) has been computed & saved before.
- *print_impact*

### auto_imputation

auto_imputation tests for 5 imputation methods using the other imputation functions provided in this module and returns the one with the best performance. The 5 methods are: (1) imputation_MMM with method_type="mean" (2) imputation_MMM with method_type="median" (3) imputation_sklearn with method_type="KNN" (4) imputation_sklearn with method_type="regression" (5) imputation_matrixFactorization

Samples without missing values in attributes to impute are used for testing by removing some % of values and impute them again using the above 5 methods. RMSE/attribute_mean is used as the evaluation metric for each attribute to reduce the effect of unit difference among attributes. The final error of a method is calculated by the sum of (RMSE/attribute_mean) for all numerical attributes to impute and the method with the least error will be selected.

The above testing is only applicable for numerical attributes. If categorical attributes are included, they will be automatically imputed using imputation_MMM. In addition, if there is only one numerical attribute to impute, only method (1) and (2) will be tested because the rest of the methods require more than one column.

- *spark*: Spark Session
- *idf*
- *list_of_cols*: "all" can be passed to include all columns. "missing" can be used to only include columns with any missing value.
- *drop_cols*
- *id_col*: name of the column representing ID. "" (by default) can be used if there is no ID column.
- *null_pct*: proportion of the valid input data to be replaced by None to form the test data
- *stats_missing*: Takes arguments for read_dataset (data_ingest module) function in a dictionary format to read pre-saved statistics on missing count/pct i.e. if measures_of_counts or missingCount_computation (data_analyzer.stats_generator module) has been computed & saved before.
- *output_mode*: "append" option appends transformed column with the naming convention - "{original.column.name}_imputed".
- *print_impact*

### autoencoder_latentFeatures
Many machine learning models suffer from "the curse of dimensionality" when the number of features is too large. autoencoder_latentFeatures is able to reduce the dimensionality by compressing input attributes to a smaller number of latent features.

To be more specific, it trains a neural network model using TensorFlow library. The neural network contains an encoder and a decoder, where the encoder learns to represent the input using smaller number of latent features controlled by the input *reduction_params* and the decoder learns to reproduce the input using the latent features. In the end, only the encoder will be kept and the latent features generated by the encoder will be added to the output dataframe.

However, the neural network model is not trained in a scalable manner, which might not be able to handle large input dataframe. Thus, an input *sample_size* (the default value is 500,000) can be set to control the number of samples to be used to train the model. If the total number of samples exceeds *sample_size*, the rest of the samples will be predicted using the fitted encoder. 

Standardization is highly recommended if the input attributes are not of the same scale. Otherwise, the model might not converge smoothly. Inputs *standardization* and *standardization_configs* can be set accordingly to perform standardization within the function. In addition, if a sample contains missing values in the model input, the output values for its latent features will all be None. Thus data imputation is also recommended if missing values exist, which can be done within the function by setting inputs *imputation* and *imputation_configs*.

- *spark*: Spark Session
- *idf*
- *list_of_cols*: "all" can be passed to include all numerical columns.
- *drop_cols*
- *reduction_params*: Determines the number of encoded features in the result. If reduction_params < 1, int(reduction_params * <number of columns>) columns will be generated. Else, reduction_params columns will be generated.
- *sample_size*: Maximum rows for training the autoencoder model using tensorflow
- *batch_size*: Integer - number of samples per gradient update when fitting the tensorflow model.
- *pre_existing_model*: Boolean argument - True or False. True if model exists already, False Otherwise.
- *model_path*: If pre_existing_model is True, this argument is path for referring the pre-saved model. If pre_existing_model is False, this argument can be used for saving the model. Default "NA" means there is neither pre-existing model nor there is a need to save one.
- *standardization*: Boolean argument - True or False. True, if the standardization required.
- *standardization_configs*: z_standardization function arguments in dictionary format.
- *imputation*: Boolean argument - True or False. True, if the imputation required.
- *imputation_configs*: Takes input in dictionary format. Imputation function name is provided with key "imputation_name". Optional arguments pertaining to that imputation function can be provided with argument name as key.
- *stats_missing*: Takes arguments for read_dataset (data_ingest module) function in a dictionary format to read pre-saved statistics on missing count/pct i.e. if measures_of_counts or missingCount_computation (data_analyzer.stats_generator module) has been computed & saved before.
- *emr_mode*: Boolean argument - True or False. True if it is run on EMR, False otherwise.
- *output_mode*: "replace" option replaces original columns with transformed columns: latent_<col_index>. "append" option append transformed columns with format latent_<col_index> to the input dataset, e.g. latent_0, latent_1 will be appended if reduction_params=2.
- *print_impact*

### PCA_latentFeatures

Similar to autoencoder_latentFeatures, PCA_latentFeatures also generates latent features which reduces the dimensionality of the input dataframe but through a different technique: Principal Component Analysis (PCA). PCA algorithm produces principal components such that it can describe most of the remaining variance and all the principal components are orthogonal to each other. The final number of generated principal components is controlled by the input *explained_variance_cutoff*. In other words, the number of selected principal components, k, is the minimum value such that top k components (latent features) can explain at least *explained_variance_cutoff* of the total variance.

Standardization is highly recommended if the input attributes are not of the same scale. Otherwise the generated latent features might be dominated by the attributes with larger variance. Inputs *standardization* and *standardization_configs* can be set accordingly to perform standardization within the function. In addition, data imputation is also recommended if missing values exist, which can be done within the function by setting inputs *imputation* and *imputation_configs*.

- *spark*: Spark Session
- *idf*
- *list_of_cols*: "all" can be passed to include all numerical columns.
- *drop_cols*
- *explained_variance_cutoff*: Determines the number of encoded columns in the output. If N is the smallest  integer such that top N encoded columns explain more than explained_variance_cutoff variance, these N columns will be selected.
- *pre_existing_model*: Boolean argument - True or False. True if model exists already, False Otherwise
- *model_path*: If pre_existing_model is True, this argument is path for referring the pre-saved model. If pre_existing_model is False, this argument can be used for saving the model. Default "NA" means there is neither pre-existing model nor there is a need to save one.
- *standardization*: Boolean argument - True or False. True, if the standardization required.
- *standardization_configs*: z_standardization function arguments in dictionary format.
- *imputation*: Boolean argument - True or False. True, if the imputation required.
- *imputation_configs*: Takes input in dictionary format. Imputation function name is provided with key "imputation_name". optional arguments pertaining to that imputation function can be provided with argument name as key. 
- *stats_missing*: Takes arguments for read_dataset (data_ingest module) function in a dictionary format to read pre-saved statistics on missing count/pct i.e. if measures_of_counts or missingCount_computation (data_analyzer.stats_generator module) has been computed & saved before.
- *output_mode*: "replace" option replaces original columns with transformed columns: latent_<col_index>. "append" option append transformed columns with format latent_<col_index> to the input dataset, e.g. latent_0, latent_1.
- *print_impact*

### feature_transformation

As the name indicates, feature_transformation performs mathematical transformation over selected attributes. The following methods are supported for an input attribute x: ln(x), log10(x), log2(x), e^x, 2^x, 10^x, N^x, square and cube root of x, x^2, x^3, x^N, trigonometric transformations of x (sin, cos, tan, asin, acos, atan), radians, x%N, x!, 1/x, floor and ceiling of x and x rounded to N decimal places. Some transformations only work with positive or non-negative input values such as log and square root and an error will be returned if violated.

- *idf*
- *list_of_cols*: "all" can be passed to include all numerical columns.
- *drop_cols*
- *method_type*: "ln", "log10", "log2", "exp", "powOf2" (2^x), "powOf10" (10^x), "powOfN" (N^x), "sqrt" (square root), "cbrt" - (cube root), "sq" (square), "cb" (cube), "toPowerN" (x^N), "sin", "cos", "tan", "asin", "acos", "atan", "radians", "remainderDivByN" (x%N), "factorial" (x!), "mul_inv" (1/x), "floor", "ceil", "roundN" (round to N decimal places)
- *N*: None by default. If method_type is "powOfN", "toPowerN", "remainderDivByN" or "roundN", N will be used as the required constant.
- *output_mode*: "append" option appends transformed column with the naming convention - "{original.column.name}_{transformation.name}".
- *print_impact*

### boxcox_transformation

Some machine learning algorithms require the input data to follow normal distributions. Thus, when the input data is too skewed, boxcox_transformation can be used to transform it into a more normal-like distribution. The transformed value of a sample x depends on a coefficient lambda: (1) if lambda = 0, x is transformed into log(x); (2) if lambda !=0, x is transformed into (x^lambda-1)/lambda. The value of lambda can be either specified by the user or automatically selected within the function. If lambda needs to be selected within the function, a range of values (0, 1, -1, 0.5, -0.5, 2, -2, 0.25, -0.25, 3, -3, 4, -4, 5, -5) will be tested and the lambda, which optimizes the KolmogorovSmirnovTest by PySpark with the theoretical distribution being normal, will be used to perform the transformation. Different lambda values can be assigned to different attributes but one attribute can only be assigned one lambda value.

- *idf*
- *list_of_cols*: "all" can be passed to include all numerical columns.
- *drop_cols*
- *boxcox_lambda*: Lambda value for box_cox transormation. If boxcox_lambda is not None, it will be directly used for the transformation. It can be a (1) list: each element represents a lambda value for an attribute and the length of the list must be the same as the number of columns to transform. (2) int/float: all attributes will be assigned the same lambda value. Else, search for the best lambda among [1,-1,0.5,-0.5,2,-2,0.25,-0.25,3,-3,4,-4,5,-5] for each column and apply the transformation
- *output_mode*: "append" option appends transformed column with the naming convention - "{original.column.name}__bxcx_{lambda}".
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


### expression_parser
expression_parser can be used to evaluate a list of SQL expressions and output the result as new features. It is able to handle column names containing special characters such as “.”, “-”, “@”, “^”, etc, by converting them to “_” first before the evaluation and convert them back to the original names before returning the output dataframe.

- *idf*
- *list_of_expr*: List of expressions to evaluate as new features e.g., ["expr1","expr2"]. Alternatively, expressions can be specified in a string format, where different expressions are separated by pipe delimiter "|" e.g., "expr1|expr2".
- *postfix*: postfix for new feature name.Naming convention "f" + expression_index + postfix e.g. with postfix of "new", new added features are named as f0new, f1new etc.
- *print_impact*
