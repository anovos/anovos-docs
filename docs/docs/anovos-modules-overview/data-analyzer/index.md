## Module ANOVOS.stats_generator 	 

This module generates all the descriptive statistics related to the ingested data. Descriptive statistics are broken down into different metric types, and each function corresponds to one metric type.
- global_summary
- measures_of_counts
- measures_of_centralTendency
- measures_of_cardinality
- measures_of_dispersion
- measures_of_percentiles  
- measures_of_shape

Columns subjected to this analysis can be controlled by the right combination of arguments - list_of_cols and drop_cols. All the above functions require the following arguments:

- **idf**: Input dataframe 
- **list_of_cols**: This argument, in a list format, is used to specify the columns which are subjected to the analysis in the input dataframe. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”. The user can also use “all” as an input to this argument to consider all columns. This is super useful instead of specifying all column names manually. 
- **drop_cols**: In a list format, this argument is used to specify the columns that need to be dropped from list_of_cols. Instead of a list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”. It is most useful when coupled with the “all” value of list_of_cols, when we need to consider all columns except a few handful of them.
- **print_impact**: This argument is to print out the statistics. 

### global_summary

The global summary function computes the following universal statistics/metrics and returns a Spark DataFrame with schema – metric, value. 
- No. of rows
- No. of columns
- No. of categorical columns along with column names
- No. of numerical columns along with the column names
- No. of non-numerical non-categorical columns such as date type, array type etc. along with column names 

### measures_of_counts

The Measures of Counts function computes different count metrics for each column (interchangeably called an attribute in the document). It returns a Spark DataFrame with schema – attribute, fill_count, fill_pct, missing_count, missing_pct, nonzero_count, nonzero_pct. 
- Fill Count/Rate is defined as number of rows with non-null values in a column both in terms of absolute count and its proportion to row count. It leverages count statistic from summary7 functionality of Spark SQL. 
- Missing Count/Rate is defined as null (or missing) values seen in a column both in terms of absolute count and its proportion to row count. It is directly derivable from Fill Count/Rate.  
- Non Zero Count/Rate is defined as non-zero values seen in a numerical column both in terms of absolute count and its proportion to row count. For categorical column, it will show null value. Also, it uses a subfunction nonzeroCount_computation, which is later called under measures_of_counts. Under the hood, it leverage Multivariate Statistical Summary8 of Spark MLlib. 

### measures_of_centralTendency 

The Measures of Central Tendency function provides summary statistics that represents the centre point or most likely value of an attribute. It returns a Spark DataFrame with schema – attribute, mean, median, mode, mode_pct. 

- Mean is arithmetic average of a column i.e. sum of all values seen in the column divided by the number of rows. It leverage mean statistic from summary functionality of Spark SQL. 
- Median is 50th percentile or middle value in a column when the values are arranged in ascending or descending order. It leverage ‘50%’ statistic from summary functionality of Spark SQL. 
- Mode is most frequently seen value in a column. Mode is calculated only for discrete columns (categorical + Integer/Long columns) 
- Mode Pct is defined as % of rows seen with Mode value. Mode Pct is calculated only for discrete columns (categorical + Integer/Long columns) 

### measures_of_counts 

The Measures of Counts function computes different count metric for each column (interchangeably called as attribute in the document). It returns a Spark DataFrame with schema – attribute, fill_count, fill_pct, missing_count, missing_pct, nonzero_count, nonzero_pct. 

- Fill Count/Rate is defined as the number of rows with non-null values in a column in terms of absolute count and its proportion to row count. It leverages count statistics from the summary7 functionality of Spark SQL.
- Missing Count/Rate is defined as null (or missing) values seen in a column in terms of absolute count and its proportion to row count. It is directly derivable from Fill Count/Rate.
- Non Zero Count/Rate is defined as non-zero values seen in a numerical column in terms of absolute count and its proportion to row count. For categorical columns, it will show a null value. Also, it uses a subfunction nonzeroCount_computation, which is later called under measures_of_counts. Under the hood, it leverages Multivariate Statistical Summary8 of Spark MLlib.

### measures_of_centralTendency 
The Measures of Central Tendency function provides summary statistics representing the attribute's center point or most likely value. It returns a Spark DataFrame with schema – attribute, mean, median, mode, mode_pct.

- Mean is the arithmetic average of a column i.e., the sum of all values seen in the column divided by the number of rows. It leverage mean statistic from the summary functionality of Spark SQL.
- Median is the 50th percentile or the middle value in a column when the values are arranged in ascending or descending order. It leverage '50%' statistic from the summary functionality of Spark SQL. 
- Mode is the most frequently seen value in a column. Mode is calculated only for discrete columns (categorical + Integer/Long columns).
- Mode Pct is defined as % of rows seen with Mode value. Mode Pct is calculated only for discrete columns (categorical + Integer/Long columns)

### measures_of_cardinality 
The Measures of Cardinality function provides statistics that are related to unique values seen in an attribute. These statistics are calculated only for discrete columns (categorical + Integer/Long columns). It returns a Spark Dataframe with schema – attribute, unique_values, IDness.

- Unique Value is defined as a distinct value count of a column. It relies on a subfunction uniqueCount_computation for its computation and leverages the countDistinct9 functionality of Spark SQL.
- IDness is calculated as Unique Values divided by non-null values seen in a column. Non-null values count is used instead of total count because too many null values can give misleading results even if the column have all unique values (except null). It uses subfunctions - uniqueCount_computation and missingCount_computation.

### measures_of_dispersion  
The Measures of Dispersion function provides statistics that describe the spread of a numerical attribute. Alternatively, these statistics are also known as measures of spread. It returns a Spark DataFrame with schema – attribute, stddev, variance, cov, IQR, range.

- Standard Deviation (stddev) measures how concentrated an attribute is around the mean or average and mathematically computed as  below. It leverages ‘stddev’ statistic from summary functionality of Spark SQL.

        s= X- X2n -1 

        where:

        ` `X is an attribute value
        X is attribute mean
        n is no. of rows

- Variance is the squared value of Standard Deviation. 
- Coefficient of Variance (cov) is computed as ratio of Standard Deviation & Mean. It leverages ‘stddev’ and ‘mean’ statistic from the summary functionality of Spark SQL. 
- Interquartile Range (IQR): It describes the difference between the third quartile (75th percentile) and the first quartile (25th percentile), telling us about the range where middle half values are seen. It leverage ‘25%’ and ‘75%’ statistics from the summary functionality of Spark SQL. 
- Range is simply the difference between the maximum value and the minimum value. It leverage ‘min’ and ‘max’ statistics from the summary functionality of Spark SQL. 

### measures_of_percentiles 

he Measures of Percentiles function provides statistics at different percentiles. Nth percentile can be interpreted as N% of rows having values lesser than or equal to Nth percentile value. It is prominently used for quick detection of skewness or outlier. Alternatively, these statistics are also known as measures of position. These statistics are computed only for numerical attributes.

It returns a Spark Dataframe with schema – attribute, min, 1%, 5%, 10%, 25%, 50%, 75%, 90%, 95%, 99%, max. It leverage ‘N%’ statistics from summary functionality of Spark SQL where N is 0 for min and 100 for max.

### measures_of_shape 

The Measures of Shapes function provides statistics related to the shape of an attribute's distribution. Alternatively, these statistics are also known as measures of the moment and are computed only for numerical attributes. It returns a Spark Dataframe with schema – attribute, skewness, kurtosis.

- Skewness describes how much-skewed values are relative to a perfect bell curve observed in normal distribution and the direction of skew. If the majority of the values are at the left and the right tail is longer, we say that the distribution is skewed right or positively skewed; if the peak is toward the right and the left tail is longer, we say that the distribution is skewed left or negatively skewed. It leverage skewness10 functionality of Spark SQL.
- (Excess) Kurtosis describes how tall and sharp the central peak is relative to a perfect bell curve observed in the normal distribution. The reference standard is a normal distribution, which has a kurtosis of 3. In token of this, often, the excess kurtosis is presented: excess kurtosis is simply kurtosis−3. Higher (positive) values indicate a higher, sharper peak; lower (negative) values indicate a less distinct peak. It leverages kurtosis11 functionality of Spark SQL.

 