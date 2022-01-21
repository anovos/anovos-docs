## Module **ANOVOS.association_evaluator** 				

This submodule focuses on understanding the interaction between different attributes and/or the relationship between an attribute & the binary target variable. 

Association between attributes is measured by: 

- correlation_matrix
- variable_clustering
- Association between an attribute and binary target is measured by:
	- IV_calculation
	- IG_calculation

Columns which are subjected to these analysis can be controlled by right combination of arguments - list_of_cols and drop_cols. All functions have following common arguments:

- **idf**: Input dataframe
- **ist_of_cols**: This argument, in a list format, is used to specify the columns which are subjected to the analysis in the input dataframe. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”. The user can also use “all” as an input to this argument to consider all columns. This is super useful instead of specifying all column names manually.
- **drop_cols**: This argument, in a list format, is used to specify the columns which needs to be dropped from list_of_cols. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”. It is most useful when used coupled with “all” value of list_of_cols, when we need to consider all columns except few handful of them.
- **print_impact**: This argument is to print out the statistics.

### correlation_matrix

This function calculates correlation coefficient statistical, which measures the strength of the relationship between the relative movements of two attributes. Pearson’s correlation coefficient is a standard approach of measuring correlation between two variables. However, it has some drawbacks: a) It works only with continuous variables, b) It only accounts for a linear relationship between variables, and c) It is sensitive to outliers. To avoid these issues, we are computing Phik (𝜙k), which is a new and practical correlation coefficient that works consistently between categorical, ordinal and interval variables, captures non-linear dependency and reverts to the Pearson correlation coefficient in case of a bivariate normal input distribution. The correlation coefficient is calculated for every pair of attributes and its value lies between 0 and 1, where 0 means there is no correlation between the two attributes and 1 means strong correlation. However, this methodology have drawbacks of its own as it is found to be more computational expensive especially when number of columns in the input dataset is on higher side (number of pairs to analyse increases exponentially with number of columns). Further, there is no indication of the direction of the relationship. More detail can be referred from the [source paper] [1].

[1]: https://arxiv.org/abs/1811.11440/     "source paper"

This function returns a correlation matrix dataframe of schema – attribute, <attribute_names>. Correlation between attribute X and Y can be found at intersection of a) row with value X in ‘attribute’ column and b) column ‘Y’ (or row with value Y in ‘attribute’ column and column ‘X’).

- **idf**
- **list_of_cols**
- **drop_cols**
- **stats_unique**: Arguments corresponding to read_dataset function in dictionary format, to read output from measures_of_cardinality function of stats generator.
- **print_impact**

### variable_clustering

Variable Clustering groups attributes that are as correlated as possible among themselves within a cluster and as uncorrelated as possible with attribute in other clusters. The function is leveraging [VarClusHi] [2] library to do variable clustering; however, this library is not implemented in a scalable manner due to which the analysis is done on a sample dataset. Further, it is found to be a bit computational expensive especially when number of columns in the input dataset is on higher side (number of pairs to analyse increases exponentially with number of columns).

[2]: https://github.com/jingtt/varclushi   "VarCluShi"

It returns a Spark Dataframe with schema – Cluster, Attribute, RS_Ratio. The attribute with the lowest (1 — RS_Ratio) can be chosen as a representative of the cluster while discarding the other attributes from that cluster. This can also help in achieving the dimension reduction, if required.

- **idf**
- **list_of_cols**
- **drop_cols**
- **sample_size**: Sample size used for performing variable cluster. Default is 100,000.
- **stats_unique**: Arguments corresponding to read_dataset function in dictionary format, to read output from measures_of_cardinality function of stats generator. This is used to remove single value columns from the analysis purpose.
- **stats_mode**: Arguments corresponding to read_dataset function in dictionary format, to read output from measures_of_centralTendency function of stats generator. This is used for MMM imputation as Variable Clustering doesn’t work with missing values.
- **print_impact**

### IV_calculation

Information Value (IV) is simple and powerful technique to conduct attribute relevance analysis. It measures how well an attribute is able to distinguish between a binary target variable i.e. label 0 from label 1, and hence helps in ranking attributes on the basis of their importance. In the heart of IV methodology are groups (bins) of observations. For categorical attributes, usually each category is a bin while numerical attributes need to be split into categories. 

IV = ∑ (% of non-events - % of events) * WOE
<br>where:
<br>WOE = In(% of non-events ➗ % of events)
<br>% of event = % label 1 in a bin
<br>% of non-event = % label 0 in a bin

General rule of thumb while creating the bins are that a) each bin should have at least 5% of the observations, b) the WOE should be monotonic, i.e. either growing or decreasing with the bins, and c) missing values should be binned separately. An article  from listendata.com can be referred for good understanding of IV & WOE concepts.

- **idf**
- **list_of_cols**
- **drop_cols**
- **label_col**: Name of label or target column in the input dataset
- **event_label**: Value of event (label 1) in the label column
- **encoding_configs**: This argument takes input in dictionary format with keys related to binning operation - 'bin_method' (default 'equal_frequency'),  'bin_size' (default 10) and 'monotonicity_check' (default 0). monotonicity_check of 1 will dynamically calculate the bin_size ensuring monotonic nature and can be expensive operation.
- **print_impact**

### IG_calculation

Information Gain (IG) is another powerful technique for feature selection analysis. Information gain is calculated by comparing the entropy of the dataset before and after a transformation (introduction of attribute in this particular case). Similar to IV calculation, each category is a bin for categorical attributes, while numerical attributes need to be split into categories. 

IG = Total Entropy – Entropy

Total Entropy= -%event*log⁡(%event)-(1-%event)*log⁡(1-%event)

Entropy = ∑(-%〖event〗_i*log⁡(%〖event〗_i )-(1-%〖event〗_i )*log⁡(1-%〖event〗_i)

- **idf**
- **list_of_cols**
- **drop_cols**
- **label_col**: Name of label or target column in the input dataset
- **event_label**: Value of event (label 1) in the label column
- **encoding_configs**: This argument takes input in dictionary format with keys related to binning operation - 'bin_method' (default 'equal_frequency'),  'bin_size' (default 10) and 'monotonicity_check' (default 0). monotonicity_check of 1 will dynamically calculate the bin_size ensuring monotonic nature and can be expensive operation.
- **print_impact**






