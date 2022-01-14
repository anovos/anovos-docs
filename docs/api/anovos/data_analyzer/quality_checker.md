# Module <code>quality_checker</code>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
# coding=utf-8
import re
import warnings
from anovos.data_analyzer.stats_generator import uniqueCount_computation, missingCount_computation, mode_computation, \
measures_of_cardinality
from anovos.data_ingest.data_ingest import read_dataset
from anovos.data_transformer.transformers import imputation_MMM
from anovos.shared.utils import attributeType_segregation, transpose_dataframe, get_dtype
from pyspark.sql import functions as F
from pyspark.sql import types as T
def duplicate_detection(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], treatment=False, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of columns to inspect e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
treatment: Boolean argument – True or False. If True, duplicate rows are removed from the input dataframe. (Default value = False)
print_impact:
(Default value = False)
Returns:
Output Dataframe, Metric Dataframe)
Output Dataframe is de-duplicated dataframe if treated, else original input dataframe.
Metric Dataframe is of schema [metric, value] and contains metrics - number of rows, number of unique rows,
number of duplicate rows and percentage of duplicate rows in total.
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in idf.columns for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if str(treatment).lower() == &#39;true&#39;:
treatment = True
elif str(treatment).lower() == &#39;false&#39;:
treatment = False
else:
raise TypeError(&#39;Non-Boolean input for treatment&#39;)
odf_tmp = idf.drop_duplicates(subset=list_of_cols)
odf = odf_tmp if treatment else idf
odf_print = spark.createDataFrame([[&#34;rows_count&#34;, float(idf.count())], \
[&#34;unique_rows_count&#34;, float(odf_tmp.count())], \
[&#34;duplicate_rows&#34;, float(idf.count() - odf_tmp.count())], \
[&#34;duplicate_pct&#34;, round((idf.count() - odf_tmp.count())/idf.count(), 4)]], \
schema=[&#39;metric&#39;, &#39;value&#39;])
if print_impact:
print(&#34;No. of Rows: &#34; + str(idf.count()))
print(&#34;No. of UNIQUE Rows: &#34; + str(odf_tmp.count()))
print(&#34;No. of Duplicate Rows: &#34; + str(idf.count() - odf_tmp.count()))
print(&#34;Percentage of Duplicate Rows: &#34; + str(round((idf.count() - odf_tmp.count())/idf.count(),4)))
return odf, odf_print
def nullRows_detection(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], treatment=False, treatment_threshold=0.8,
print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of columns to inspect e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
treatment: Boolean argument – True or False. If True, rows with high no. of null columns (defined by
treatment_threshold argument) are removed from the input dataframe. (Default value = False)
treatment_threshold: Defines % of columns allowed to be Null per row and takes value between 0 to 1.
If % of null columns is above the threshold for a row, it is removed from the dataframe.
There is no row removal if the threshold is 1.0. And if the threshold is 0, all rows with
null value are removed. (Default value = 0.8)
print_impact:
(Default value = False)
Returns:
Output Dataframe, Metric Dataframe)
Output Dataframe is the dataframe after row removal if treated, else original input dataframe.
Metric Dataframe is of schema [null_cols_count, row_count, row_pct, flagged/treated]. null_cols_count is defined as
no. of missing columns in a row. row_count is no. of rows with null_cols_count missing columns.
row_pct is row_count divided by number of rows. flagged/treated is 1 if null_cols_count is more than
(threshold
X Number of Columns), else 0.
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in idf.columns for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if str(treatment).lower() == &#39;true&#39;:
treatment = True
elif str(treatment).lower() == &#39;false&#39;:
treatment = False
else:
raise TypeError(&#39;Non-Boolean input for treatment&#39;)
treatment_threshold = float(treatment_threshold)
if (treatment_threshold &lt; 0) | (treatment_threshold &gt; 1):
raise TypeError(&#39;Invalid input for Treatment Threshold Value&#39;)
def null_count(*cols):
&#34;&#34;&#34;
Args:
*cols:
Returns:
&#34;&#34;&#34;
return cols.count(None)
f_null_count = F.udf(null_count, T.LongType())
odf_tmp = idf.withColumn(&#34;null_cols_count&#34;, f_null_count(*list_of_cols)) \
.withColumn(&#39;flagged&#39;, F.when(F.col(&#34;null_cols_count&#34;) &gt; (len(list_of_cols) * treatment_threshold), 1) \
.otherwise(0))
if treatment_threshold == 1:
odf_tmp = odf_tmp.withColumn(&#39;flagged&#39;, F.when(F.col(&#34;null_cols_count&#34;) == len(list_of_cols), 1).otherwise(0))
odf_print = odf_tmp.groupBy(&#34;null_cols_count&#34;, &#34;flagged&#34;).agg(F.count(F.lit(1)).alias(&#39;row_count&#39;)) \
.withColumn(&#39;row_pct&#39;, F.round(F.col(&#39;row_count&#39;) / float(idf.count()), 4)) \
.select(&#39;null_cols_count&#39;, &#39;row_count&#39;, &#39;row_pct&#39;, &#39;flagged&#39;).orderBy(&#39;null_cols_count&#39;)
if treatment:
odf = odf_tmp.where(F.col(&#34;flagged&#34;) == 0).drop(*[&#34;null_cols_count&#34;, &#34;flagged&#34;])
odf_print = odf_print.withColumnRenamed(&#39;flagged&#39;, &#39;treated&#39;)
else:
odf = idf
if print_impact:
odf_print.show(odf.count())
return odf, odf_print
def nullColumns_detection(spark, idf, list_of_cols=&#39;missing&#39;, drop_cols=[], treatment=False,
treatment_method=&#39;row_removal&#39;,
treatment_configs={}, stats_missing={}, stats_unique={}, stats_mode={}, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of columns to inspect e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all (non-array) columns for analysis.
&#34;missing&#34; (default) can be passed to include only those columns with missing values.
One of the usecases where &#34;all&#34; may be preferable over &#34;missing&#34; is when the user wants to save
the imputation model for the future use e.g. a column may not have missing value in the training
dataset but missing values may possibly appear in the prediction dataset.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols.
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
treatment: Boolean argument – True or False. If True, missing values are treated as per treatment_method argument. (Default value = False)
treatment_method: MMM&#34;, &#34;row_removal&#34;, &#34;column_removal&#34; (more methods to be added soon).
MMM (Mean Median Mode) replaces null value by the measure of central tendency (mode for
categorical features and mean or median for numerical features).
row_removal removes all rows with any missing value.
column_removal remove a column if % of rows with missing value is above a threshold (defined
by key &#34;treatment_threshold&#34; under treatment_configs argument). (Default value = &#39;row_removal&#39;)
treatment_configs: Takes input in dictionary format.
For column_removal treatment, key ‘treatment_threshold’ is provided with a value between 0 to 1.
For MMM, arguments corresponding to imputation_MMM function (transformer module) are provided,
where each key is an argument from imputation_MMM function.
For row_removal, this argument can be skipped. (Default value = {})
stats_missing: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on missing count/pct i.e. if measures_of_counts or
missingCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
stats_mode: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on most frequently seen values i.e. if measures_of_centralTendency or
mode_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)
Returns:
Output Dataframe, Metric Dataframe)
Output Dataframe is the imputed dataframe if treated, else original input dataframe.
Metric Dataframe is of schema [attribute, missing_count, missing_pct]. missing_count is number of rows
with null values for an attribute and missing_pct is missing_count divided by number of rows.
&#34;&#34;&#34;
if stats_missing == {}:
odf_print = missingCount_computation(spark, idf)
else:
odf_print = read_dataset(spark, **stats_missing).select(&#39;attribute&#39;, &#39;missing_count&#39;, &#39;missing_pct&#39;)
missing_cols = odf_print.where(F.col(&#39;missing_count&#39;) &gt; 0).select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
if list_of_cols == &#34;missing&#34;:
list_of_cols = missing_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if len(list_of_cols) == 0:
warnings.warn(&#34;No Null Detection - No column(s) to analyze&#34;)
odf = idf
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;missing_count&#39;, T.StringType(), True),
T.StructField(&#39;missing_pct&#39;, T.StringType(), True)])
odf_print = spark.sparkContext.emptyRDD().toDF(schema)
return odf, odf_print
if any(x not in idf.columns for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if str(treatment).lower() == &#39;true&#39;:
treatment = True
elif str(treatment).lower() == &#39;false&#39;:
treatment = False
else:
raise TypeError(&#39;Non-Boolean input for treatment&#39;)
if treatment_method not in (&#39;MMM&#39;, &#39;row_removal&#39;, &#39;column_removal&#39;):
raise TypeError(&#39;Invalid input for method_type&#39;)
treatment_threshold = treatment_configs.pop(&#39;treatment_threshold&#39;, None)
if treatment_threshold:
treatment_threshold = float(treatment_threshold)
else:
if treatment_method == &#39;column_removal&#39;:
raise TypeError(&#39;Invalid input for column removal threshold&#39;)
odf_print = odf_print.where(F.col(&#39;attribute&#39;).isin(list_of_cols))
if treatment:
if treatment_threshold:
threshold_cols = odf_print.where(F.col(&#39;attribute&#39;).isin(list_of_cols)) \
.where(F.col(&#39;missing_pct&#39;) &gt; treatment_threshold) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
if treatment_method == &#39;column_removal&#39;:
odf = idf.drop(*threshold_cols)
if print_impact:
print(&#34;Removed Columns: &#34;, threshold_cols)
if treatment_method == &#39;row_removal&#39;:
remove_cols = odf_print.where(F.col(&#39;attribute&#39;).isin(list_of_cols)) \
.where(F.col(&#39;missing_pct&#39;) == 1.0) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
list_of_cols = [e for e in list_of_cols if e not in remove_cols]
if treatment_threshold:
list_of_cols = [e for e in threshold_cols if e not in remove_cols]
odf = idf.dropna(subset=list_of_cols)
if print_impact:
odf_print.show(len(list_of_cols))
print(&#34;Before Count: &#34; + str(idf.count()))
print(&#34;After Count: &#34; + str(odf.count()))
if treatment_method == &#39;MMM&#39;:
if stats_unique == {}:
remove_cols = uniqueCount_computation(spark, idf, list_of_cols).where(F.col(&#39;unique_values&#39;) &lt; 2) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
else:
remove_cols = read_dataset(spark, **stats_unique).where(F.col(&#39;unique_values&#39;) &lt; 2) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
list_of_cols = [e for e in list_of_cols if e not in remove_cols]
if treatment_threshold:
list_of_cols = [e for e in threshold_cols if e not in remove_cols]
odf = imputation_MMM(spark, idf, list_of_cols, **treatment_configs, stats_missing=stats_missing,
stats_mode=stats_mode, print_impact=print_impact)
else:
odf = idf
return odf, odf_print
def outlier_detection(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], detection_side=&#39;upper&#39;,
detection_configs={&#39;pctile_lower&#39;: 0.05, &#39;pctile_upper&#39;: 0.95,
&#39;stdev_lower&#39;: 3.0, &#39;stdev_upper&#39;: 3.0,
&#39;IQR_lower&#39;: 1.5, &#39;IQR_upper&#39;: 1.5,
&#39;min_validation&#39;: 2},
treatment=False, treatment_method=&#39;value_replacement&#39;, pre_existing_model=False,
model_path=&#34;NA&#34;, output_mode=&#39;replace&#39;, stats_unique={}, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of numerical columns to inspect e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
detection_side: upper&#34;, &#34;lower&#34;, &#34;both&#34;.
&#34;lower&#34; detects outliers in the lower spectrum of the column range, whereas &#34;upper&#34; detects
in the upper spectrum. &#34;Both&#34; detects in both upper and lower end of the spectrum. (Default value = &#39;upper&#39;)
detection_configs: Takes input in dictionary format with keys representing upper &amp; lower parameter for
three outlier detection methodologies.
a) Percentile Method: In this methodology, a value higher than a certain (default 0.95)
percentile value is considered as an outlier. Similarly, a value lower than a certain
(default 0.05) percentile value is considered as an outlier.
b) Standard Deviation Method: In this methodology, if a value is certain number of
standard deviations (default 3.0) away from the mean, then it is identified as an outlier.
c) Interquartile Range (IQR) Method: A value which is below Q1 – k * IQR or
above Q3 + k * IQR (default k is 1.5) are identified as outliers, where Q1 is first quartile/
25th percentile, Q3 is third quartile/75th percentile and IQR is difference between
third quartile &amp; first quartile.
If an attribute value is less (more) than its derived lower (upper) bound value,
it is considered as outlier by a methodology. A attribute value is considered as outlier
if it is declared as outlier by atleast &#39;min_validation&#39; methodologies (default 2).
treatment: Boolean argument – True or False. If True, outliers are treated as per treatment_method argument. (Default value = False)
treatment_method: null_replacement&#34;, &#34;row_removal&#34;, &#34;value_replacement&#34;.
In &#34;null_replacement&#34;, outlier values are replaced by null so that it can be imputed by a
reliable imputation methodology. In &#34;value_replacement&#34;, outlier values are replaced by
maximum or minimum permissible value by above methodologies. Lastly in &#34;row_removal&#34;, rows
are removed if it is found with any outlier. (Default value = &#39;value_replacement&#39;)
pre_existing_model: Boolean argument – True or False. True if the model with upper/lower permissible values
for each attribute exists already to be used, False otherwise. (Default value = False)
model_path: If pre_existing_model is True, this argument is path for the pre-saved model.
If pre_existing_model is False, this field can be used for saving the model.
Default &#34;NA&#34; means there is neither pre-existing model nor there is a need to save one.
output_mode: replace&#34;, &#34;append&#34;.
“replace” option replaces original columns with treated column. “append” option append treated
column to the input dataset with a postfix &#34;_outliered&#34; e.g. column X is appended as X_outliered. (Default value = &#39;replace&#39;)
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
&#39;pctile_upper&#39;: 0.95:
&#39;stdev_lower&#39;: 3.0:
&#39;stdev_upper&#39;: 3.0:
&#39;IQR_lower&#39;: 1.5:
&#39;IQR_upper&#39;: 1.5:
&#39;min_validation&#39;: 2}:
print_impact:
(Default value = False)
Returns:
Output Dataframe, Metric Dataframe)
Output Dataframe is the imputed dataframe if treated, else original input dataframe.
Metric Dataframe is of schema [attribute, lower_outliers, upper_outliers]. lower_outliers is no. of outliers
found in the lower spectrum of the attribute range and upper_outliers is outlier count in the upper spectrum.
&#34;&#34;&#34;
num_cols = attributeType_segregation(idf)[0]
if len(num_cols) == 0:
warnings.warn(&#34;No Outlier Check - No numerical column(s) to analyse&#34;)
odf = idf
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;lower_outliers&#39;, T.StringType(), True),
T.StructField(&#39;upper_outliers&#39;, T.StringType(), True)])
odf_print = spark.sparkContext.emptyRDD().toDF(schema)
return odf, odf_print
if list_of_cols == &#39;all&#39;:
list_of_cols = num_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
if stats_unique == {}:
remove_cols = uniqueCount_computation(spark, idf, list_of_cols).where(F.col(&#39;unique_values&#39;) &lt; 2) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
else:
remove_cols = read_dataset(spark, **stats_unique).where(F.col(&#39;unique_values&#39;) &lt; 2) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
list_of_cols = list(set([e for e in list_of_cols if e not in (drop_cols + remove_cols)]))
if any(x not in num_cols for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if detection_side not in (&#39;upper&#39;, &#39;lower&#39;, &#39;both&#39;):
raise TypeError(&#39;Invalid input for detection_side&#39;)
if treatment_method not in (&#39;null_replacement&#39;, &#39;row_removal&#39;, &#39;value_replacement&#39;):
raise TypeError(&#39;Invalid input for treatment_method&#39;)
if output_mode not in (&#39;replace&#39;, &#39;append&#39;):
raise TypeError(&#39;Invalid input for output_mode&#39;)
if str(treatment).lower() == &#39;true&#39;:
treatment = True
elif str(treatment).lower() == &#39;false&#39;:
treatment = False
else:
raise TypeError(&#39;Non-Boolean input for treatment&#39;)
if str(pre_existing_model).lower() == &#39;true&#39;:
pre_existing_model = True
elif str(pre_existing_model).lower() == &#39;false&#39;:
pre_existing_model = False
else:
raise TypeError(&#39;Non-Boolean input for pre_existing_model&#39;)
for arg in [&#39;pctile_lower&#39;, &#39;pctile_upper&#39;]:
if arg in detection_configs:
if (detection_configs[arg] &lt; 0) | (detection_configs[arg] &gt; 1):
raise TypeError(&#39;Invalid input for &#39; + arg)
recast_cols = []
recast_type = []
for i in list_of_cols:
if get_dtype(idf, i).startswith(&#39;decimal&#39;):
idf = idf.withColumn(i, F.col(i).cast(T.DoubleType()))
recast_cols.append(i)
recast_type.append(get_dtype(idf, i))
if pre_existing_model:
df_model = spark.read.parquet(model_path + &#34;/outlier_numcols&#34;)
params = []
for i in list_of_cols:
mapped_value = df_model.where(F.col(&#39;attribute&#39;) == i).select(&#39;parameters&#39;) \
.rdd.flatMap(lambda x: x).collect()[0]
params.append(mapped_value)
pctile_params = idf.approxQuantile(list_of_cols, [detection_configs.get(&#39;pctile_lower&#39;, 0.05),
detection_configs.get(&#39;pctile_upper&#39;, 0.95)], 0.01)
skewed_cols = []
for i, p in zip(list_of_cols, pctile_params):
if p[0] == p[1]:
skewed_cols.append(i)
else:
detection_configs[&#39;pctile_lower&#39;] = detection_configs[&#39;pctile_lower&#39;] or 0.0
detection_configs[&#39;pctile_upper&#39;] = detection_configs[&#39;pctile_upper&#39;] or 1.0
pctile_params = idf.approxQuantile(list_of_cols, [detection_configs[&#39;pctile_lower&#39;],
detection_configs[&#39;pctile_upper&#39;]], 0.01)
skewed_cols = []
for i, p in zip(list_of_cols, pctile_params):
if p[0] == p[1]:
skewed_cols.append(i)
detection_configs[&#39;stdev_lower&#39;] = detection_configs[&#39;stdev_lower&#39;] or detection_configs[&#39;stdev_upper&#39;]
detection_configs[&#39;stdev_upper&#39;] = detection_configs[&#39;stdev_upper&#39;] or detection_configs[&#39;stdev_lower&#39;]
stdev_params = []
for i in list_of_cols:
mean, stdev = idf.select(F.mean(i), F.stddev(i)).first()
stdev_params.append(
[mean - detection_configs[&#39;stdev_lower&#39;] * stdev, mean + detection_configs[&#39;stdev_upper&#39;] * stdev])
detection_configs[&#39;IQR_lower&#39;] = detection_configs[&#39;IQR_lower&#39;] or detection_configs[&#39;IQR_upper&#39;]
detection_configs[&#39;IQR_upper&#39;] = detection_configs[&#39;IQR_upper&#39;] or detection_configs[&#39;IQR_lower&#39;]
quantiles = idf.approxQuantile(list_of_cols, [0.25, 0.75], 0.01)
IQR_params = [[e[0] - detection_configs[&#39;IQR_lower&#39;] * (e[1] - e[0]),
e[1] + detection_configs[&#39;IQR_upper&#39;] * (e[1] - e[0])] for e in quantiles]
n = detection_configs[&#39;min_validation&#39;]
params = [[sorted([x[0], y[0], z[0]], reverse=True)[n - 1], sorted([x[1], y[1], z[1]])[n - 1]] for x, y, z in
list(zip(pctile_params, stdev_params, IQR_params))]
# Saving model File if required
if model_path != &#34;NA&#34;:
df_model = spark.createDataFrame(zip(list_of_cols, params), schema=[&#39;attribute&#39;, &#39;parameters&#39;])
df_model.coalesce(1).write.parquet(model_path + &#34;/outlier_numcols&#34;, mode=&#39;overwrite&#39;)
for i, j in zip(recast_cols, recast_type):
idf = idf.withColumn(i, F.col(i).cast(j))
def composite_outlier(*v):
&#34;&#34;&#34;
Args:
*v:
Returns:
&#34;&#34;&#34;
output = []
for idx, e in enumerate(v):
if e is None:
output.append(None)
continue
if detection_side in (&#39;upper&#39;, &#39;both&#39;):
if e &gt; params[idx][1]:
output.append(1)
continue
if detection_side in (&#39;lower&#39;, &#39;both&#39;):
if e &lt; params[idx][0]:
output.append(-1)
continue
output.append(0)
return output
f_composite_outlier = F.udf(composite_outlier, T.ArrayType(T.IntegerType()))
odf = idf.withColumn(&#34;outliered&#34;, f_composite_outlier(*list_of_cols))
odf.persist()
output_print = []
for index, i in enumerate(list_of_cols):
odf = odf.withColumn(i + &#34;_outliered&#34;, F.col(&#39;outliered&#39;)[index])
output_print.append(
[i, odf.where(F.col(i + &#34;_outliered&#34;) == -1).count(), odf.where(F.col(i + &#34;_outliered&#34;) == 1).count()])
if treatment &amp; (treatment_method in (&#39;value_replacement&#39;, &#39;null_replacement&#39;)):
if skewed_cols:
warnings.warn(
&#34;Columns dropped from outlier treatment due to highly skewed distribution: &#34; + (&#39;,&#39;).join(skewed_cols))
if i not in skewed_cols:
replace_vals = {&#39;value_replacement&#39;: [params[index][0], params[index][1]],
&#39;null_replacement&#39;: [None, None]}
odf = odf.withColumn(i + &#34;_outliered&#34;,
F.when(F.col(i + &#34;_outliered&#34;) == 1, replace_vals[treatment_method][1]) \
.otherwise(F.when(F.col(i + &#34;_outliered&#34;) == -1, replace_vals[treatment_method][0]) \
.otherwise(F.col(i))))
if output_mode == &#39;replace&#39;:
odf = odf.drop(i).withColumnRenamed(i + &#34;_outliered&#34;, i)
else:
odf = odf.drop(i + &#34;_outliered&#34;)
odf = odf.drop(&#34;outliered&#34;)
if treatment &amp; (treatment_method == &#39;row_removal&#39;):
if skewed_cols:
warnings.warn(
&#34;Columns dropped from outlier treatment due to highly skewed distribution: &#34; + (&#39;,&#39;).join(skewed_cols))
for index, i in enumerate(list_of_cols):
if i not in skewed_cols:
odf = odf.where((F.col(i + &#34;_outliered&#34;) == 0) | (F.col(i + &#34;_outliered&#34;).isNull())).drop(
i + &#34;_outliered&#34;)
else:
odf = odf.drop(i + &#34;_outliered&#34;)
if not (treatment):
odf = idf
odf_print = spark.createDataFrame(output_print, schema=[&#39;attribute&#39;, &#39;lower_outliers&#39;, &#39;upper_outliers&#39;])
if print_impact:
odf_print.show(len(list_of_cols))
return odf, odf_print
def IDness_detection(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], treatment=False, treatment_threshold=0.8,
stats_unique={},
print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of Discrete (Categorical + Integer) columns to inspect e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all categorical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
treatment: Boolean argument – True or False. If True, columns with high IDness (defined by
treatment_threshold argument) are removed from the input dataframe. (Default value = False)
treatment_threshold: Defines acceptable level of IDness (calculated as no. of unique values divided by no. of
non-null values) for a column and takes value between 0 to 1. Default threshold
of 0.8 can be interpreted as remove column if its unique values count is more than
80% of total rows (after excluding null values).
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)
Returns:
Output Dataframe, Metric Dataframe)
Output Dataframe is the dataframe after column removal if treated, else original input dataframe.
Metric Dataframe is of schema [attribute, unique_values, IDness, flagged/treated]. unique_values is no. of distinct
values in a column, IDness is unique_values divided by no. of non-null values. A column is flagged 1
if IDness is above the threshold, else 0.
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
for i in idf.select(list_of_cols).dtypes:
if (i[1] not in (&#39;string&#39;, &#39;int&#39;, &#39;bigint&#39;, &#39;long&#39;)):
list_of_cols.remove(i[0])
if any(x not in idf.columns for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if len(list_of_cols) == 0:
warnings.warn(&#34;No IDness Check - No discrete column(s) to analyze&#34;)
odf = idf
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;unique_values&#39;, T.StringType(), True),
T.StructField(&#39;IDness&#39;, T.StringType(), True),
T.StructField(&#39;flagged&#39;, T.StringType(), True)])
odf_print = spark.sparkContext.emptyRDD().toDF(schema)
return odf, odf_print
treatment_threshold = float(treatment_threshold)
if (treatment_threshold &lt; 0) | (treatment_threshold &gt; 1):
raise TypeError(&#39;Invalid input for Treatment Threshold Value&#39;)
if str(treatment).lower() == &#39;true&#39;:
treatment = True
elif str(treatment).lower() == &#39;false&#39;:
treatment = False
else:
raise TypeError(&#39;Non-Boolean input for treatment&#39;)
if stats_unique == {}:
odf_print = measures_of_cardinality(spark, idf, list_of_cols)
else:
odf_print = read_dataset(spark, **stats_unique).where(F.col(&#39;attribute&#39;).isin(list_of_cols))
odf_print = odf_print.withColumn(&#39;flagged&#39;, F.when(F.col(&#39;IDness&#39;) &gt;= treatment_threshold, 1).otherwise(0))
if treatment:
remove_cols = odf_print.where(F.col(&#39;flagged&#39;) == 1).select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
odf = idf.drop(*remove_cols)
odf_print = odf_print.withColumnRenamed(&#39;flagged&#39;,&#39;treated&#39;)
else:
odf = idf
if print_impact:
odf_print.show(len(list_of_cols))
if treatment:
print(&#34;Removed Columns: &#34;, remove_cols)
return odf, odf_print
def biasedness_detection(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], treatment=False, treatment_threshold=0.8,
stats_mode={},
print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of Discrete (Categorical + Integer) columns to inspect e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all discrete columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
treatment: Boolean argument – True or False. If True, columns with high biasedness (defined by
treatment_threshold argument) are removed from the input dataframe. (Default value = False)
treatment_threshold: Defines acceptable level of biasedness (frequency of most-frequently seen value)for
a column and takes value between 0 to 1. Default threshold of 0.8 can be interpreted as
remove column if the number of rows with most-frequently seen value is more than 80%
of total rows (after excluding null values).
stats_mode: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on most frequently seen values i.e. if measures_of_centralTendency or
mode_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)
Returns:
Output Dataframe, Metric Dataframe)
Output Dataframe is the dataframe after column removal if treated, else original input dataframe.
Metric Dataframe is of schema [attribute, mode, mode_rows, mode_pct, flagged/treated]. mode is the most frequently seen value,
mode_rows is number of rows with mode value and mode_pct is number of rows with mode value divided by non-null values.
A column is flagged 1 if mode_pct is above the threshold else 0.
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
for i in idf.select(list_of_cols).dtypes:
if (i[1] not in (&#39;string&#39;, &#39;int&#39;, &#39;bigint&#39;, &#39;long&#39;)):
list_of_cols.remove(i[0])
if any(x not in idf.columns for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if len(list_of_cols) == 0:
warnings.warn(&#34;No biasedness Check - No discrete column(s) to analyze&#34;)
odf = idf
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;mode&#39;, T.StringType(), True),
T.StructField(&#39;mode_rows&#39;, T.StringType(), True),
T.StructField(&#39;mode_pct&#39;, T.StringType(), True),
T.StructField(&#39;flagged&#39;, T.StringType(), True)])
odf_print = spark.sparkContext.emptyRDD().toDF(schema)
return odf, odf_print
if (treatment_threshold &lt; 0) | (treatment_threshold &gt; 1):
raise TypeError(&#39;Invalid input for Treatment Threshold Value&#39;)
if str(treatment).lower() == &#39;true&#39;:
treatment = True
elif str(treatment).lower() == &#39;false&#39;:
treatment = False
else:
raise TypeError(&#39;Non-Boolean input for treatment&#39;)
if stats_mode == {}:
odf_print = transpose_dataframe(idf.select(list_of_cols).summary(&#34;count&#34;), &#39;summary&#39;) \
.withColumnRenamed(&#39;key&#39;, &#39;attribute&#39;) \
.join(mode_computation(spark, idf, list_of_cols), &#39;attribute&#39;, &#39;full_outer&#39;) \
.withColumn(&#39;mode_pct&#39;, F.round(F.col(&#39;mode_rows&#39;) / F.col(&#39;count&#39;).cast(T.DoubleType()), 4)) \
.select(&#39;attribute&#39;, &#39;mode&#39;, &#39;mode_rows&#39;, &#39;mode_pct&#39;)
else:
odf_print = read_dataset(spark, **stats_mode).select(&#39;attribute&#39;, &#39;mode&#39;, &#39;mode_rows&#39;, &#39;mode_pct&#39;) \
.where(F.col(&#39;attribute&#39;).isin(list_of_cols))
odf_print = odf_print.withColumn(&#39;flagged&#39;,
F.when(
(F.col(&#39;mode_pct&#39;) &gt;= treatment_threshold) | (F.col(&#39;mode_pct&#39;).isNull()),
1).otherwise(0))
if treatment:
remove_cols = odf_print.where((F.col(&#39;mode_pct&#39;) &gt;= treatment_threshold) | (F.col(&#39;mode_pct&#39;).isNull())) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
odf = idf.drop(*remove_cols)
odf_print = odf_print.withColumnRenamed(&#39;flagged&#39;,&#39;treated&#39;)
else:
odf = idf
if print_impact:
odf_print.show(len(list_of_cols))
if treatment:
print(&#34;Removed Columns: &#34;, remove_cols)
return odf, odf_print
def invalidEntries_detection(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], treatment=False, treatment_method=&#39;null_replacement&#39;,
treatment_configs={}, stats_missing={}, stats_unique={}, stats_mode={}, output_mode=&#39;replace&#39;, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of Discrete (Categorical + Integer) columns to inspect e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all discrete columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
treatment: Boolean argument – True or False. If True, invalid values are replaced by Null. (Default value = False)
treatment_method: MMM&#34;, &#34;null_replacement&#34;, &#34;column_removal&#34; (more methods to be added soon).
MMM (Mean Median Mode) replaces invalid value by the measure of central tendency (mode for
categorical features and mean or median for numerical features).
null_replacement removes all values with any invalid values as null.
column_removal remove a column if % of rows with invalid value is above a threshold (defined
by key &#34;treatment_threshold&#34; under treatment_configs argument). (Default value = &#39;null_replacement&#39;)
treatment_configs: Takes input in dictionary format.
For column_removal treatment, key ‘treatment_threshold’ is provided with a value between 0 to 1.
For value replacement, by MMM, arguments corresponding to imputation_MMM function (transformer module) are provided,
where each key is an argument from imputation_MMM function.
For null_replacement, this argument can be skipped. (Default value = {})
output_mode: replace&#34;, &#34;append&#34;.
“replace” option replaces original columns with treated column. “append” option append treated
column to the input dataset with a postfix &#34;_invalid&#34; e.g. column X is appended as X_invalid. (Default value = &#39;replace&#39;)
stats_missing:
(Default value = {})
stats_unique:
(Default value = {})
stats_mode:
(Default value = {})
print_impact:
(Default value = False)
Returns:
Output Dataframe, Metric Dataframe)
Output Dataframe is the dataframe after treatment if applicable, else original input dataframe.
Metric Dataframe is of schema [attribute, invalid_entries, invalid_count, invalid_pct].
invalid_entries are all potential invalid values (separated by delimiter pipe “|”), invalid_count is no.
of rows which are impacted by invalid entries and invalid_pct is invalid_count divided by no of rows.
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
list_of_cols = []
for i in idf.dtypes:
if (i[1] in (&#39;string&#39;, &#39;int&#39;, &#39;bigint&#39;, &#39;long&#39;)):
list_of_cols.append(i[0])
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in idf.columns for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if len(list_of_cols) == 0:
warnings.warn(&#34;No Invalid Entries Check - No discrete column(s) to analyze&#34;)
odf = idf
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;invalid_entries&#39;, T.StringType(), True),
T.StructField(&#39;invalid_count&#39;, T.StringType(), True),
T.StructField(&#39;invalid_pct&#39;, T.StringType(), True)])
odf_print = spark.sparkContext.emptyRDD().toDF(schema)
return odf, odf_print
if output_mode not in (&#39;replace&#39;, &#39;append&#39;):
raise TypeError(&#39;Invalid input for output_mode&#39;)
if str(treatment).lower() == &#39;true&#39;:
treatment = True
elif str(treatment).lower() == &#39;false&#39;:
treatment = False
else:
raise TypeError(&#39;Non-Boolean input for treatment&#39;)
if treatment_method not in (&#39;MMM&#39;, &#39;null_replacement&#39;, &#39;column_removal&#39;):
raise TypeError(&#39;Invalid input for method_type&#39;)
treatment_threshold = treatment_configs.pop(&#39;treatment_threshold&#39;, None)
if treatment_threshold:
treatment_threshold = float(treatment_threshold)
else:
if treatment_method == &#39;column_removal&#39;:
raise TypeError(&#39;Invalid input for column removal threshold&#39;)
null_vocab = [&#39;&#39;, &#39; &#39;, &#39;nan&#39;, &#39;null&#39;, &#39;na&#39;, &#39;inf&#39;, &#39;n/a&#39;, &#39;not defined&#39;, &#39;none&#39;, &#39;undefined&#39;, &#39;blank&#39;]
specialChars_vocab = [&#34;&amp;&#34;, &#34;$&#34;, &#34;;&#34;, &#34;:&#34;, &#34;.&#34;, &#34;,&#34;, &#34;*&#34;, &#34;#&#34;, &#34;@&#34;, &#34;_&#34;, &#34;?&#34;, &#34;%&#34;, &#34;!&#34;, &#34;^&#34;, &#34;(&#34;, &#34;)&#34;, &#34;-&#34;, &#34;/&#34;, &#34;&#39;&#34;]
def detect(*v):
&#34;&#34;&#34;
Args:
*v:
Returns:
&#34;&#34;&#34;
output = []
for idx, e in enumerate(v):
if e is None:
output.append(None)
continue
e = str(e).lower().strip()
# Null &amp; Special Chars Search
if e in (null_vocab + specialChars_vocab):
output.append(1)
continue
# Consecutive Identical Chars Search
regex = &#34;\\b([a-zA-Z0-9])\\1\\1+\\b&#34;
p = re.compile(regex)
if (re.search(p, e)):
output.append(1)
continue
# Ordered Chars Search
l = len(e)
check = 0
if l &gt;= 3:
for i in range(1, l):
if ord(e[i]) - ord(e[i - 1]) != 1:
output.append(0)
check = 1
break
if check == 1:
continue
else:
output.append(1)
continue
else:
output.append(0)
continue
return output
f_detect = F.udf(detect, T.ArrayType(T.LongType()))
odf = idf.withColumn(&#34;invalid&#34;, f_detect(*list_of_cols))
odf.persist()
output_print = []
for index, i in enumerate(list_of_cols):
tmp = odf.withColumn(i + &#34;_invalid&#34;, F.col(&#39;invalid&#39;)[index])
invalid = tmp.where(F.col(i + &#34;_invalid&#34;) == 1).select(i).distinct().rdd.flatMap(lambda x: x).collect()
invalid = [str(x) for x in invalid]
invalid_count = tmp.where(F.col(i + &#34;_invalid&#34;) == 1).count()
output_print.append([i, &#39;|&#39;.join(invalid), invalid_count, round(invalid_count / idf.count(), 4)])
odf_print = spark.createDataFrame(output_print,
schema=[&#39;attribute&#39;, &#39;invalid_entries&#39;, &#39;invalid_count&#39;, &#39;invalid_pct&#39;])
if treatment:
if treatment_threshold:
threshold_cols = odf_print.where(F.col(&#39;attribute&#39;).isin(list_of_cols)) \
.where(F.col(&#39;invalid_pct&#39;) &gt; treatment_threshold) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
if treatment_method in (&#39;null_replacement&#39;, &#39;MMM&#39;):
for index, i in enumerate(list_of_cols):
if treatment_threshold:
if i not in threshold_cols:
odf = odf.drop(i + &#34;_invalid&#34;)
continue
odf = odf.withColumn(i + &#34;_invalid&#34;, F.when(F.col(&#39;invalid&#39;)[index] == 1, None).otherwise(F.col(i)))
if output_mode == &#39;replace&#39;:
odf = odf.drop(i).withColumnRenamed(i + &#34;_invalid&#34;, i)
else:
if odf_print.where(F.col(&#34;attribute&#34;) == i).select(&#39;invalid_pct&#39;).collect()[0][0] == 0.0:
odf = odf.drop(i + &#34;_invalid&#34;)
odf = odf.drop(&#34;invalid&#34;)
if treatment_method == &#39;column_removal&#39;:
odf = idf.drop(*threshold_cols)
if print_impact:
print(&#34;Removed Columns: &#34;, threshold_cols)
if treatment_method == &#39;MMM&#39;:
if stats_unique == {} or output_mode == &#39;append&#39;:
remove_cols = uniqueCount_computation(spark, odf, list_of_cols).where(F.col(&#39;unique_values&#39;) &lt; 2) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
else:
remove_cols = read_dataset(spark, **stats_unique).where(F.col(&#39;unique_values&#39;) &lt; 2) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
list_of_cols = [e for e in list_of_cols if e not in remove_cols]
if treatment_threshold:
list_of_cols = [e for e in threshold_cols if e not in remove_cols]
if output_mode == &#39;append&#39;:
if len(list_of_cols) &gt; 0:
list_of_cols = [e + &#39;_invalid&#39; for e in list_of_cols]
odf = imputation_MMM(spark, odf, list_of_cols, **treatment_configs, stats_missing=stats_missing,
stats_mode=stats_mode, print_impact=print_impact)
else:
odf = idf
if print_impact:
odf_print.show(len(list_of_cols))
return odf, odf_print
```
</details>
## Functions
<dl>
<dt id="anovos.data_analyzer.quality_checker.IDness_detection"><code class="name flex">
<span>def <span class="ident">IDness_detection</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], treatment=False, treatment_threshold=0.8, stats_unique={}, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of Discrete (Categorical + Integer) columns to inspect e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all categorical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
treatment: Boolean argument – True or False. If True, columns with high IDness (defined by
treatment_threshold argument) are removed from the input dataframe. (Default value = False)
treatment_threshold: Defines acceptable level of IDness (calculated as no. of unique values divided by no. of
non-null values) for a column and takes value between 0 to 1. Default threshold
of 0.8 can be interpreted as remove column if its unique values count is more than
80% of total rows (after excluding null values).
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Output Dataframe, Metric Dataframe)
Output Dataframe is the dataframe after column removal if treated, else original input dataframe.
Metric Dataframe is of schema [attribute, unique_values, IDness, flagged/treated]. unique_values is no. of distinct
values in a column, IDness is unique_values divided by no. of non-null values. A column is flagged 1
if IDness is above the threshold, else 0.</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def IDness_detection(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], treatment=False, treatment_threshold=0.8,
stats_unique={},
print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of Discrete (Categorical + Integer) columns to inspect e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all categorical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
treatment: Boolean argument – True or False. If True, columns with high IDness (defined by
treatment_threshold argument) are removed from the input dataframe. (Default value = False)
treatment_threshold: Defines acceptable level of IDness (calculated as no. of unique values divided by no. of
non-null values) for a column and takes value between 0 to 1. Default threshold
of 0.8 can be interpreted as remove column if its unique values count is more than
80% of total rows (after excluding null values).
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)
Returns:
Output Dataframe, Metric Dataframe)
Output Dataframe is the dataframe after column removal if treated, else original input dataframe.
Metric Dataframe is of schema [attribute, unique_values, IDness, flagged/treated]. unique_values is no. of distinct
values in a column, IDness is unique_values divided by no. of non-null values. A column is flagged 1
if IDness is above the threshold, else 0.
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
for i in idf.select(list_of_cols).dtypes:
if (i[1] not in (&#39;string&#39;, &#39;int&#39;, &#39;bigint&#39;, &#39;long&#39;)):
list_of_cols.remove(i[0])
if any(x not in idf.columns for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if len(list_of_cols) == 0:
warnings.warn(&#34;No IDness Check - No discrete column(s) to analyze&#34;)
odf = idf
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;unique_values&#39;, T.StringType(), True),
T.StructField(&#39;IDness&#39;, T.StringType(), True),
T.StructField(&#39;flagged&#39;, T.StringType(), True)])
odf_print = spark.sparkContext.emptyRDD().toDF(schema)
return odf, odf_print
treatment_threshold = float(treatment_threshold)
if (treatment_threshold &lt; 0) | (treatment_threshold &gt; 1):
raise TypeError(&#39;Invalid input for Treatment Threshold Value&#39;)
if str(treatment).lower() == &#39;true&#39;:
treatment = True
elif str(treatment).lower() == &#39;false&#39;:
treatment = False
else:
raise TypeError(&#39;Non-Boolean input for treatment&#39;)
if stats_unique == {}:
odf_print = measures_of_cardinality(spark, idf, list_of_cols)
else:
odf_print = read_dataset(spark, **stats_unique).where(F.col(&#39;attribute&#39;).isin(list_of_cols))
odf_print = odf_print.withColumn(&#39;flagged&#39;, F.when(F.col(&#39;IDness&#39;) &gt;= treatment_threshold, 1).otherwise(0))
if treatment:
remove_cols = odf_print.where(F.col(&#39;flagged&#39;) == 1).select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
odf = idf.drop(*remove_cols)
odf_print = odf_print.withColumnRenamed(&#39;flagged&#39;,&#39;treated&#39;)
else:
odf = idf
if print_impact:
odf_print.show(len(list_of_cols))
if treatment:
print(&#34;Removed Columns: &#34;, remove_cols)
return odf, odf_print
```
</details>
</dd>
<dt id="anovos.data_analyzer.quality_checker.biasedness_detection"><code class="name flex">
<span>def <span class="ident">biasedness_detection</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], treatment=False, treatment_threshold=0.8, stats_mode={}, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of Discrete (Categorical + Integer) columns to inspect e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all discrete columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
treatment: Boolean argument – True or False. If True, columns with high biasedness (defined by
treatment_threshold argument) are removed from the input dataframe. (Default value = False)
treatment_threshold: Defines acceptable level of biasedness (frequency of most-frequently seen value)for
a column and takes value between 0 to 1. Default threshold of 0.8 can be interpreted as
remove column if the number of rows with most-frequently seen value is more than 80%
of total rows (after excluding null values).
stats_mode: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on most frequently seen values i.e. if measures_of_centralTendency or
mode_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Output Dataframe, Metric Dataframe)
Output Dataframe is the dataframe after column removal if treated, else original input dataframe.
Metric Dataframe is of schema [attribute, mode, mode_rows, mode_pct, flagged/treated]. mode is the most frequently seen value,
mode_rows is number of rows with mode value and mode_pct is number of rows with mode value divided by non-null values.
A column is flagged 1 if mode_pct is above the threshold else 0.</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def biasedness_detection(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], treatment=False, treatment_threshold=0.8,
stats_mode={},
print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of Discrete (Categorical + Integer) columns to inspect e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all discrete columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
treatment: Boolean argument – True or False. If True, columns with high biasedness (defined by
treatment_threshold argument) are removed from the input dataframe. (Default value = False)
treatment_threshold: Defines acceptable level of biasedness (frequency of most-frequently seen value)for
a column and takes value between 0 to 1. Default threshold of 0.8 can be interpreted as
remove column if the number of rows with most-frequently seen value is more than 80%
of total rows (after excluding null values).
stats_mode: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on most frequently seen values i.e. if measures_of_centralTendency or
mode_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)
Returns:
Output Dataframe, Metric Dataframe)
Output Dataframe is the dataframe after column removal if treated, else original input dataframe.
Metric Dataframe is of schema [attribute, mode, mode_rows, mode_pct, flagged/treated]. mode is the most frequently seen value,
mode_rows is number of rows with mode value and mode_pct is number of rows with mode value divided by non-null values.
A column is flagged 1 if mode_pct is above the threshold else 0.
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
for i in idf.select(list_of_cols).dtypes:
if (i[1] not in (&#39;string&#39;, &#39;int&#39;, &#39;bigint&#39;, &#39;long&#39;)):
list_of_cols.remove(i[0])
if any(x not in idf.columns for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if len(list_of_cols) == 0:
warnings.warn(&#34;No biasedness Check - No discrete column(s) to analyze&#34;)
odf = idf
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;mode&#39;, T.StringType(), True),
T.StructField(&#39;mode_rows&#39;, T.StringType(), True),
T.StructField(&#39;mode_pct&#39;, T.StringType(), True),
T.StructField(&#39;flagged&#39;, T.StringType(), True)])
odf_print = spark.sparkContext.emptyRDD().toDF(schema)
return odf, odf_print
if (treatment_threshold &lt; 0) | (treatment_threshold &gt; 1):
raise TypeError(&#39;Invalid input for Treatment Threshold Value&#39;)
if str(treatment).lower() == &#39;true&#39;:
treatment = True
elif str(treatment).lower() == &#39;false&#39;:
treatment = False
else:
raise TypeError(&#39;Non-Boolean input for treatment&#39;)
if stats_mode == {}:
odf_print = transpose_dataframe(idf.select(list_of_cols).summary(&#34;count&#34;), &#39;summary&#39;) \
.withColumnRenamed(&#39;key&#39;, &#39;attribute&#39;) \
.join(mode_computation(spark, idf, list_of_cols), &#39;attribute&#39;, &#39;full_outer&#39;) \
.withColumn(&#39;mode_pct&#39;, F.round(F.col(&#39;mode_rows&#39;) / F.col(&#39;count&#39;).cast(T.DoubleType()), 4)) \
.select(&#39;attribute&#39;, &#39;mode&#39;, &#39;mode_rows&#39;, &#39;mode_pct&#39;)
else:
odf_print = read_dataset(spark, **stats_mode).select(&#39;attribute&#39;, &#39;mode&#39;, &#39;mode_rows&#39;, &#39;mode_pct&#39;) \
.where(F.col(&#39;attribute&#39;).isin(list_of_cols))
odf_print = odf_print.withColumn(&#39;flagged&#39;,
F.when(
(F.col(&#39;mode_pct&#39;) &gt;= treatment_threshold) | (F.col(&#39;mode_pct&#39;).isNull()),
1).otherwise(0))
if treatment:
remove_cols = odf_print.where((F.col(&#39;mode_pct&#39;) &gt;= treatment_threshold) | (F.col(&#39;mode_pct&#39;).isNull())) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
odf = idf.drop(*remove_cols)
odf_print = odf_print.withColumnRenamed(&#39;flagged&#39;,&#39;treated&#39;)
else:
odf = idf
if print_impact:
odf_print.show(len(list_of_cols))
if treatment:
print(&#34;Removed Columns: &#34;, remove_cols)
return odf, odf_print
```
</details>
</dd>
<dt id="anovos.data_analyzer.quality_checker.duplicate_detection"><code class="name flex">
<span>def <span class="ident">duplicate_detection</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], treatment=False, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of columns to inspect e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
treatment: Boolean argument – True or False. If True, duplicate rows are removed from the input dataframe. (Default value = False)
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Output Dataframe, Metric Dataframe)
Output Dataframe is de-duplicated dataframe if treated, else original input dataframe.
Metric Dataframe is of schema [metric, value] and contains metrics - number of rows, number of unique rows,
number of duplicate rows and percentage of duplicate rows in total.</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def duplicate_detection(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], treatment=False, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of columns to inspect e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
treatment: Boolean argument – True or False. If True, duplicate rows are removed from the input dataframe. (Default value = False)
print_impact:
(Default value = False)
Returns:
Output Dataframe, Metric Dataframe)
Output Dataframe is de-duplicated dataframe if treated, else original input dataframe.
Metric Dataframe is of schema [metric, value] and contains metrics - number of rows, number of unique rows,
number of duplicate rows and percentage of duplicate rows in total.
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in idf.columns for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if str(treatment).lower() == &#39;true&#39;:
treatment = True
elif str(treatment).lower() == &#39;false&#39;:
treatment = False
else:
raise TypeError(&#39;Non-Boolean input for treatment&#39;)
odf_tmp = idf.drop_duplicates(subset=list_of_cols)
odf = odf_tmp if treatment else idf
odf_print = spark.createDataFrame([[&#34;rows_count&#34;, float(idf.count())], \
[&#34;unique_rows_count&#34;, float(odf_tmp.count())], \
[&#34;duplicate_rows&#34;, float(idf.count() - odf_tmp.count())], \
[&#34;duplicate_pct&#34;, round((idf.count() - odf_tmp.count())/idf.count(), 4)]], \
schema=[&#39;metric&#39;, &#39;value&#39;])
if print_impact:
print(&#34;No. of Rows: &#34; + str(idf.count()))
print(&#34;No. of UNIQUE Rows: &#34; + str(odf_tmp.count()))
print(&#34;No. of Duplicate Rows: &#34; + str(idf.count() - odf_tmp.count()))
print(&#34;Percentage of Duplicate Rows: &#34; + str(round((idf.count() - odf_tmp.count())/idf.count(),4)))
return odf, odf_print
```
</details>
</dd>
<dt id="anovos.data_analyzer.quality_checker.invalidEntries_detection"><code class="name flex">
<span>def <span class="ident">invalidEntries_detection</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], treatment=False, treatment_method='null_replacement', treatment_configs={}, stats_missing={}, stats_unique={}, stats_mode={}, output_mode='replace', print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of Discrete (Categorical + Integer) columns to inspect e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all discrete columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
treatment: Boolean argument – True or False. If True, invalid values are replaced by Null. (Default value = False)
treatment_method: MMM", "null_replacement", "column_removal" (more methods to be added soon).
MMM (Mean Median Mode) replaces invalid value by the measure of central tendency (mode for
categorical features and mean or median for numerical features).
null_replacement removes all values with any invalid values as null.
column_removal remove a column if % of rows with invalid value is above a threshold (defined
by key "treatment_threshold" under treatment_configs argument). (Default value = 'null_replacement')
treatment_configs: Takes input in dictionary format.
For column_removal treatment, key ‘treatment_threshold’ is provided with a value between 0 to 1.
For value replacement, by MMM, arguments corresponding to imputation_MMM function (transformer module) are provided,
where each key is an argument from imputation_MMM function.
For null_replacement, this argument can be skipped. (Default value = {})
output_mode: replace", "append".
“replace” option replaces original columns with treated column. “append” option append treated
column to the input dataset with a postfix "_invalid" e.g. column X is appended as X_invalid. (Default value = 'replace')
stats_missing:
(Default value = {})
stats_unique:
(Default value = {})
stats_mode:
(Default value = {})
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Output Dataframe, Metric Dataframe)
Output Dataframe is the dataframe after treatment if applicable, else original input dataframe.
Metric Dataframe is of schema [attribute, invalid_entries, invalid_count, invalid_pct].
invalid_entries are all potential invalid values (separated by delimiter pipe “|”), invalid_count is no.
of rows which are impacted by invalid entries and invalid_pct is invalid_count divided by no of rows.</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def invalidEntries_detection(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], treatment=False, treatment_method=&#39;null_replacement&#39;,
treatment_configs={}, stats_missing={}, stats_unique={}, stats_mode={}, output_mode=&#39;replace&#39;, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of Discrete (Categorical + Integer) columns to inspect e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all discrete columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
treatment: Boolean argument – True or False. If True, invalid values are replaced by Null. (Default value = False)
treatment_method: MMM&#34;, &#34;null_replacement&#34;, &#34;column_removal&#34; (more methods to be added soon).
MMM (Mean Median Mode) replaces invalid value by the measure of central tendency (mode for
categorical features and mean or median for numerical features).
null_replacement removes all values with any invalid values as null.
column_removal remove a column if % of rows with invalid value is above a threshold (defined
by key &#34;treatment_threshold&#34; under treatment_configs argument). (Default value = &#39;null_replacement&#39;)
treatment_configs: Takes input in dictionary format.
For column_removal treatment, key ‘treatment_threshold’ is provided with a value between 0 to 1.
For value replacement, by MMM, arguments corresponding to imputation_MMM function (transformer module) are provided,
where each key is an argument from imputation_MMM function.
For null_replacement, this argument can be skipped. (Default value = {})
output_mode: replace&#34;, &#34;append&#34;.
“replace” option replaces original columns with treated column. “append” option append treated
column to the input dataset with a postfix &#34;_invalid&#34; e.g. column X is appended as X_invalid. (Default value = &#39;replace&#39;)
stats_missing:
(Default value = {})
stats_unique:
(Default value = {})
stats_mode:
(Default value = {})
print_impact:
(Default value = False)
Returns:
Output Dataframe, Metric Dataframe)
Output Dataframe is the dataframe after treatment if applicable, else original input dataframe.
Metric Dataframe is of schema [attribute, invalid_entries, invalid_count, invalid_pct].
invalid_entries are all potential invalid values (separated by delimiter pipe “|”), invalid_count is no.
of rows which are impacted by invalid entries and invalid_pct is invalid_count divided by no of rows.
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
list_of_cols = []
for i in idf.dtypes:
if (i[1] in (&#39;string&#39;, &#39;int&#39;, &#39;bigint&#39;, &#39;long&#39;)):
list_of_cols.append(i[0])
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in idf.columns for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if len(list_of_cols) == 0:
warnings.warn(&#34;No Invalid Entries Check - No discrete column(s) to analyze&#34;)
odf = idf
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;invalid_entries&#39;, T.StringType(), True),
T.StructField(&#39;invalid_count&#39;, T.StringType(), True),
T.StructField(&#39;invalid_pct&#39;, T.StringType(), True)])
odf_print = spark.sparkContext.emptyRDD().toDF(schema)
return odf, odf_print
if output_mode not in (&#39;replace&#39;, &#39;append&#39;):
raise TypeError(&#39;Invalid input for output_mode&#39;)
if str(treatment).lower() == &#39;true&#39;:
treatment = True
elif str(treatment).lower() == &#39;false&#39;:
treatment = False
else:
raise TypeError(&#39;Non-Boolean input for treatment&#39;)
if treatment_method not in (&#39;MMM&#39;, &#39;null_replacement&#39;, &#39;column_removal&#39;):
raise TypeError(&#39;Invalid input for method_type&#39;)
treatment_threshold = treatment_configs.pop(&#39;treatment_threshold&#39;, None)
if treatment_threshold:
treatment_threshold = float(treatment_threshold)
else:
if treatment_method == &#39;column_removal&#39;:
raise TypeError(&#39;Invalid input for column removal threshold&#39;)
null_vocab = [&#39;&#39;, &#39; &#39;, &#39;nan&#39;, &#39;null&#39;, &#39;na&#39;, &#39;inf&#39;, &#39;n/a&#39;, &#39;not defined&#39;, &#39;none&#39;, &#39;undefined&#39;, &#39;blank&#39;]
specialChars_vocab = [&#34;&amp;&#34;, &#34;$&#34;, &#34;;&#34;, &#34;:&#34;, &#34;.&#34;, &#34;,&#34;, &#34;*&#34;, &#34;#&#34;, &#34;@&#34;, &#34;_&#34;, &#34;?&#34;, &#34;%&#34;, &#34;!&#34;, &#34;^&#34;, &#34;(&#34;, &#34;)&#34;, &#34;-&#34;, &#34;/&#34;, &#34;&#39;&#34;]
def detect(*v):
&#34;&#34;&#34;
Args:
*v:
Returns:
&#34;&#34;&#34;
output = []
for idx, e in enumerate(v):
if e is None:
output.append(None)
continue
e = str(e).lower().strip()
# Null &amp; Special Chars Search
if e in (null_vocab + specialChars_vocab):
output.append(1)
continue
# Consecutive Identical Chars Search
regex = &#34;\\b([a-zA-Z0-9])\\1\\1+\\b&#34;
p = re.compile(regex)
if (re.search(p, e)):
output.append(1)
continue
# Ordered Chars Search
l = len(e)
check = 0
if l &gt;= 3:
for i in range(1, l):
if ord(e[i]) - ord(e[i - 1]) != 1:
output.append(0)
check = 1
break
if check == 1:
continue
else:
output.append(1)
continue
else:
output.append(0)
continue
return output
f_detect = F.udf(detect, T.ArrayType(T.LongType()))
odf = idf.withColumn(&#34;invalid&#34;, f_detect(*list_of_cols))
odf.persist()
output_print = []
for index, i in enumerate(list_of_cols):
tmp = odf.withColumn(i + &#34;_invalid&#34;, F.col(&#39;invalid&#39;)[index])
invalid = tmp.where(F.col(i + &#34;_invalid&#34;) == 1).select(i).distinct().rdd.flatMap(lambda x: x).collect()
invalid = [str(x) for x in invalid]
invalid_count = tmp.where(F.col(i + &#34;_invalid&#34;) == 1).count()
output_print.append([i, &#39;|&#39;.join(invalid), invalid_count, round(invalid_count / idf.count(), 4)])
odf_print = spark.createDataFrame(output_print,
schema=[&#39;attribute&#39;, &#39;invalid_entries&#39;, &#39;invalid_count&#39;, &#39;invalid_pct&#39;])
if treatment:
if treatment_threshold:
threshold_cols = odf_print.where(F.col(&#39;attribute&#39;).isin(list_of_cols)) \
.where(F.col(&#39;invalid_pct&#39;) &gt; treatment_threshold) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
if treatment_method in (&#39;null_replacement&#39;, &#39;MMM&#39;):
for index, i in enumerate(list_of_cols):
if treatment_threshold:
if i not in threshold_cols:
odf = odf.drop(i + &#34;_invalid&#34;)
continue
odf = odf.withColumn(i + &#34;_invalid&#34;, F.when(F.col(&#39;invalid&#39;)[index] == 1, None).otherwise(F.col(i)))
if output_mode == &#39;replace&#39;:
odf = odf.drop(i).withColumnRenamed(i + &#34;_invalid&#34;, i)
else:
if odf_print.where(F.col(&#34;attribute&#34;) == i).select(&#39;invalid_pct&#39;).collect()[0][0] == 0.0:
odf = odf.drop(i + &#34;_invalid&#34;)
odf = odf.drop(&#34;invalid&#34;)
if treatment_method == &#39;column_removal&#39;:
odf = idf.drop(*threshold_cols)
if print_impact:
print(&#34;Removed Columns: &#34;, threshold_cols)
if treatment_method == &#39;MMM&#39;:
if stats_unique == {} or output_mode == &#39;append&#39;:
remove_cols = uniqueCount_computation(spark, odf, list_of_cols).where(F.col(&#39;unique_values&#39;) &lt; 2) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
else:
remove_cols = read_dataset(spark, **stats_unique).where(F.col(&#39;unique_values&#39;) &lt; 2) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
list_of_cols = [e for e in list_of_cols if e not in remove_cols]
if treatment_threshold:
list_of_cols = [e for e in threshold_cols if e not in remove_cols]
if output_mode == &#39;append&#39;:
if len(list_of_cols) &gt; 0:
list_of_cols = [e + &#39;_invalid&#39; for e in list_of_cols]
odf = imputation_MMM(spark, odf, list_of_cols, **treatment_configs, stats_missing=stats_missing,
stats_mode=stats_mode, print_impact=print_impact)
else:
odf = idf
if print_impact:
odf_print.show(len(list_of_cols))
return odf, odf_print
```
</details>
</dd>
<dt id="anovos.data_analyzer.quality_checker.nullColumns_detection"><code class="name flex">
<span>def <span class="ident">nullColumns_detection</span></span>(<span>spark, idf, list_of_cols='missing', drop_cols=[], treatment=False, treatment_method='row_removal', treatment_configs={}, stats_missing={}, stats_unique={}, stats_mode={}, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of columns to inspect e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all (non-array) columns for analysis.
"missing" (default) can be passed to include only those columns with missing values.
One of the usecases where "all" may be preferable over "missing" is when the user wants to save
the imputation model for the future use e.g. a column may not have missing value in the training
dataset but missing values may possibly appear in the prediction dataset.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols.
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
treatment: Boolean argument – True or False. If True, missing values are treated as per treatment_method argument. (Default value = False)
treatment_method: MMM", "row_removal", "column_removal" (more methods to be added soon).
MMM (Mean Median Mode) replaces null value by the measure of central tendency (mode for
categorical features and mean or median for numerical features).
row_removal removes all rows with any missing value.
column_removal remove a column if % of rows with missing value is above a threshold (defined
by key "treatment_threshold" under treatment_configs argument). (Default value = 'row_removal')
treatment_configs: Takes input in dictionary format.
For column_removal treatment, key ‘treatment_threshold’ is provided with a value between 0 to 1.
For MMM, arguments corresponding to imputation_MMM function (transformer module) are provided,
where each key is an argument from imputation_MMM function.
For row_removal, this argument can be skipped. (Default value = {})
stats_missing: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on missing count/pct i.e. if measures_of_counts or
missingCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
stats_mode: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on most frequently seen values i.e. if measures_of_centralTendency or
mode_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Output Dataframe, Metric Dataframe)
Output Dataframe is the imputed dataframe if treated, else original input dataframe.
Metric Dataframe is of schema [attribute, missing_count, missing_pct]. missing_count is number of rows
with null values for an attribute and missing_pct is missing_count divided by number of rows.</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def nullColumns_detection(spark, idf, list_of_cols=&#39;missing&#39;, drop_cols=[], treatment=False,
treatment_method=&#39;row_removal&#39;,
treatment_configs={}, stats_missing={}, stats_unique={}, stats_mode={}, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of columns to inspect e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all (non-array) columns for analysis.
&#34;missing&#34; (default) can be passed to include only those columns with missing values.
One of the usecases where &#34;all&#34; may be preferable over &#34;missing&#34; is when the user wants to save
the imputation model for the future use e.g. a column may not have missing value in the training
dataset but missing values may possibly appear in the prediction dataset.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols.
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
treatment: Boolean argument – True or False. If True, missing values are treated as per treatment_method argument. (Default value = False)
treatment_method: MMM&#34;, &#34;row_removal&#34;, &#34;column_removal&#34; (more methods to be added soon).
MMM (Mean Median Mode) replaces null value by the measure of central tendency (mode for
categorical features and mean or median for numerical features).
row_removal removes all rows with any missing value.
column_removal remove a column if % of rows with missing value is above a threshold (defined
by key &#34;treatment_threshold&#34; under treatment_configs argument). (Default value = &#39;row_removal&#39;)
treatment_configs: Takes input in dictionary format.
For column_removal treatment, key ‘treatment_threshold’ is provided with a value between 0 to 1.
For MMM, arguments corresponding to imputation_MMM function (transformer module) are provided,
where each key is an argument from imputation_MMM function.
For row_removal, this argument can be skipped. (Default value = {})
stats_missing: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on missing count/pct i.e. if measures_of_counts or
missingCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
stats_mode: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on most frequently seen values i.e. if measures_of_centralTendency or
mode_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)
Returns:
Output Dataframe, Metric Dataframe)
Output Dataframe is the imputed dataframe if treated, else original input dataframe.
Metric Dataframe is of schema [attribute, missing_count, missing_pct]. missing_count is number of rows
with null values for an attribute and missing_pct is missing_count divided by number of rows.
&#34;&#34;&#34;
if stats_missing == {}:
odf_print = missingCount_computation(spark, idf)
else:
odf_print = read_dataset(spark, **stats_missing).select(&#39;attribute&#39;, &#39;missing_count&#39;, &#39;missing_pct&#39;)
missing_cols = odf_print.where(F.col(&#39;missing_count&#39;) &gt; 0).select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
if list_of_cols == &#34;missing&#34;:
list_of_cols = missing_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if len(list_of_cols) == 0:
warnings.warn(&#34;No Null Detection - No column(s) to analyze&#34;)
odf = idf
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;missing_count&#39;, T.StringType(), True),
T.StructField(&#39;missing_pct&#39;, T.StringType(), True)])
odf_print = spark.sparkContext.emptyRDD().toDF(schema)
return odf, odf_print
if any(x not in idf.columns for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if str(treatment).lower() == &#39;true&#39;:
treatment = True
elif str(treatment).lower() == &#39;false&#39;:
treatment = False
else:
raise TypeError(&#39;Non-Boolean input for treatment&#39;)
if treatment_method not in (&#39;MMM&#39;, &#39;row_removal&#39;, &#39;column_removal&#39;):
raise TypeError(&#39;Invalid input for method_type&#39;)
treatment_threshold = treatment_configs.pop(&#39;treatment_threshold&#39;, None)
if treatment_threshold:
treatment_threshold = float(treatment_threshold)
else:
if treatment_method == &#39;column_removal&#39;:
raise TypeError(&#39;Invalid input for column removal threshold&#39;)
odf_print = odf_print.where(F.col(&#39;attribute&#39;).isin(list_of_cols))
if treatment:
if treatment_threshold:
threshold_cols = odf_print.where(F.col(&#39;attribute&#39;).isin(list_of_cols)) \
.where(F.col(&#39;missing_pct&#39;) &gt; treatment_threshold) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
if treatment_method == &#39;column_removal&#39;:
odf = idf.drop(*threshold_cols)
if print_impact:
print(&#34;Removed Columns: &#34;, threshold_cols)
if treatment_method == &#39;row_removal&#39;:
remove_cols = odf_print.where(F.col(&#39;attribute&#39;).isin(list_of_cols)) \
.where(F.col(&#39;missing_pct&#39;) == 1.0) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
list_of_cols = [e for e in list_of_cols if e not in remove_cols]
if treatment_threshold:
list_of_cols = [e for e in threshold_cols if e not in remove_cols]
odf = idf.dropna(subset=list_of_cols)
if print_impact:
odf_print.show(len(list_of_cols))
print(&#34;Before Count: &#34; + str(idf.count()))
print(&#34;After Count: &#34; + str(odf.count()))
if treatment_method == &#39;MMM&#39;:
if stats_unique == {}:
remove_cols = uniqueCount_computation(spark, idf, list_of_cols).where(F.col(&#39;unique_values&#39;) &lt; 2) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
else:
remove_cols = read_dataset(spark, **stats_unique).where(F.col(&#39;unique_values&#39;) &lt; 2) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
list_of_cols = [e for e in list_of_cols if e not in remove_cols]
if treatment_threshold:
list_of_cols = [e for e in threshold_cols if e not in remove_cols]
odf = imputation_MMM(spark, idf, list_of_cols, **treatment_configs, stats_missing=stats_missing,
stats_mode=stats_mode, print_impact=print_impact)
else:
odf = idf
return odf, odf_print
```
</details>
</dd>
<dt id="anovos.data_analyzer.quality_checker.nullRows_detection"><code class="name flex">
<span>def <span class="ident">nullRows_detection</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], treatment=False, treatment_threshold=0.8, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of columns to inspect e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
treatment: Boolean argument – True or False. If True, rows with high no. of null columns (defined by
treatment_threshold argument) are removed from the input dataframe. (Default value = False)
treatment_threshold: Defines % of columns allowed to be Null per row and takes value between 0 to 1.
If % of null columns is above the threshold for a row, it is removed from the dataframe.
There is no row removal if the threshold is 1.0. And if the threshold is 0, all rows with
null value are removed. (Default value = 0.8)
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Output Dataframe, Metric Dataframe)
Output Dataframe is the dataframe after row removal if treated, else original input dataframe.
Metric Dataframe is of schema [null_cols_count, row_count, row_pct, flagged/treated]. null_cols_count is defined as
no. of missing columns in a row. row_count is no. of rows with null_cols_count missing columns.
row_pct is row_count divided by number of rows. flagged/treated is 1 if null_cols_count is more than
(threshold
X Number of Columns), else 0.</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def nullRows_detection(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], treatment=False, treatment_threshold=0.8,
print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of columns to inspect e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
treatment: Boolean argument – True or False. If True, rows with high no. of null columns (defined by
treatment_threshold argument) are removed from the input dataframe. (Default value = False)
treatment_threshold: Defines % of columns allowed to be Null per row and takes value between 0 to 1.
If % of null columns is above the threshold for a row, it is removed from the dataframe.
There is no row removal if the threshold is 1.0. And if the threshold is 0, all rows with
null value are removed. (Default value = 0.8)
print_impact:
(Default value = False)
Returns:
Output Dataframe, Metric Dataframe)
Output Dataframe is the dataframe after row removal if treated, else original input dataframe.
Metric Dataframe is of schema [null_cols_count, row_count, row_pct, flagged/treated]. null_cols_count is defined as
no. of missing columns in a row. row_count is no. of rows with null_cols_count missing columns.
row_pct is row_count divided by number of rows. flagged/treated is 1 if null_cols_count is more than
(threshold
X Number of Columns), else 0.
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in idf.columns for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if str(treatment).lower() == &#39;true&#39;:
treatment = True
elif str(treatment).lower() == &#39;false&#39;:
treatment = False
else:
raise TypeError(&#39;Non-Boolean input for treatment&#39;)
treatment_threshold = float(treatment_threshold)
if (treatment_threshold &lt; 0) | (treatment_threshold &gt; 1):
raise TypeError(&#39;Invalid input for Treatment Threshold Value&#39;)
def null_count(*cols):
&#34;&#34;&#34;
Args:
*cols:
Returns:
&#34;&#34;&#34;
return cols.count(None)
f_null_count = F.udf(null_count, T.LongType())
odf_tmp = idf.withColumn(&#34;null_cols_count&#34;, f_null_count(*list_of_cols)) \
.withColumn(&#39;flagged&#39;, F.when(F.col(&#34;null_cols_count&#34;) &gt; (len(list_of_cols) * treatment_threshold), 1) \
.otherwise(0))
if treatment_threshold == 1:
odf_tmp = odf_tmp.withColumn(&#39;flagged&#39;, F.when(F.col(&#34;null_cols_count&#34;) == len(list_of_cols), 1).otherwise(0))
odf_print = odf_tmp.groupBy(&#34;null_cols_count&#34;, &#34;flagged&#34;).agg(F.count(F.lit(1)).alias(&#39;row_count&#39;)) \
.withColumn(&#39;row_pct&#39;, F.round(F.col(&#39;row_count&#39;) / float(idf.count()), 4)) \
.select(&#39;null_cols_count&#39;, &#39;row_count&#39;, &#39;row_pct&#39;, &#39;flagged&#39;).orderBy(&#39;null_cols_count&#39;)
if treatment:
odf = odf_tmp.where(F.col(&#34;flagged&#34;) == 0).drop(*[&#34;null_cols_count&#34;, &#34;flagged&#34;])
odf_print = odf_print.withColumnRenamed(&#39;flagged&#39;, &#39;treated&#39;)
else:
odf = idf
if print_impact:
odf_print.show(odf.count())
return odf, odf_print
```
</details>
</dd>
<dt id="anovos.data_analyzer.quality_checker.outlier_detection"><code class="name flex">
<span>def <span class="ident">outlier_detection</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], detection_side='upper', detection_configs={'pctile_lower': 0.05, 'pctile_upper': 0.95, 'stdev_lower': 3.0, 'stdev_upper': 3.0, 'IQR_lower': 1.5, 'IQR_upper': 1.5, 'min_validation': 2}, treatment=False, treatment_method='value_replacement', pre_existing_model=False, model_path='NA', output_mode='replace', stats_unique={}, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of numerical columns to inspect e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
detection_side: upper", "lower", "both".
"lower" detects outliers in the lower spectrum of the column range, whereas "upper" detects
in the upper spectrum. "Both" detects in both upper and lower end of the spectrum. (Default value = 'upper')
detection_configs: Takes input in dictionary format with keys representing upper &amp; lower parameter for
three outlier detection methodologies.
a) Percentile Method: In this methodology, a value higher than a certain (default 0.95)
percentile value is considered as an outlier. Similarly, a value lower than a certain
(default 0.05) percentile value is considered as an outlier.
b) Standard Deviation Method: In this methodology, if a value is certain number of
standard deviations (default 3.0) away from the mean, then it is identified as an outlier.
c) Interquartile Range (IQR) Method: A value which is below Q1 – k * IQR or
above Q3 + k * IQR (default k is 1.5) are identified as outliers, where Q1 is first quartile/
25th percentile, Q3 is third quartile/75th percentile and IQR is difference between
third quartile &amp; first quartile.
If an attribute value is less (more) than its derived lower (upper) bound value,
it is considered as outlier by a methodology. A attribute value is considered as outlier
if it is declared as outlier by atleast 'min_validation' methodologies (default 2).
treatment: Boolean argument – True or False. If True, outliers are treated as per treatment_method argument. (Default value = False)
treatment_method: null_replacement", "row_removal", "value_replacement".
In "null_replacement", outlier values are replaced by null so that it can be imputed by a
reliable imputation methodology. In "value_replacement", outlier values are replaced by
maximum or minimum permissible value by above methodologies. Lastly in "row_removal", rows
are removed if it is found with any outlier. (Default value = 'value_replacement')
pre_existing_model: Boolean argument – True or False. True if the model with upper/lower permissible values
for each attribute exists already to be used, False otherwise. (Default value = False)
model_path: If pre_existing_model is True, this argument is path for the pre-saved model.
If pre_existing_model is False, this field can be used for saving the model.
Default "NA" means there is neither pre-existing model nor there is a need to save one.
output_mode: replace", "append".
“replace” option replaces original columns with treated column. “append” option append treated
column to the input dataset with a postfix "_outliered" e.g. column X is appended as X_outliered. (Default value = 'replace')
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
'pctile_upper': 0.95:
'stdev_lower': 3.0:
'stdev_upper': 3.0:
'IQR_lower': 1.5:
'IQR_upper': 1.5:
'min_validation': 2}:
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Output Dataframe, Metric Dataframe)
Output Dataframe is the imputed dataframe if treated, else original input dataframe.
Metric Dataframe is of schema [attribute, lower_outliers, upper_outliers]. lower_outliers is no. of outliers
found in the lower spectrum of the attribute range and upper_outliers is outlier count in the upper spectrum.</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def outlier_detection(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], detection_side=&#39;upper&#39;,
detection_configs={&#39;pctile_lower&#39;: 0.05, &#39;pctile_upper&#39;: 0.95,
&#39;stdev_lower&#39;: 3.0, &#39;stdev_upper&#39;: 3.0,
&#39;IQR_lower&#39;: 1.5, &#39;IQR_upper&#39;: 1.5,
&#39;min_validation&#39;: 2},
treatment=False, treatment_method=&#39;value_replacement&#39;, pre_existing_model=False,
model_path=&#34;NA&#34;, output_mode=&#39;replace&#39;, stats_unique={}, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of numerical columns to inspect e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
detection_side: upper&#34;, &#34;lower&#34;, &#34;both&#34;.
&#34;lower&#34; detects outliers in the lower spectrum of the column range, whereas &#34;upper&#34; detects
in the upper spectrum. &#34;Both&#34; detects in both upper and lower end of the spectrum. (Default value = &#39;upper&#39;)
detection_configs: Takes input in dictionary format with keys representing upper &amp; lower parameter for
three outlier detection methodologies.
a) Percentile Method: In this methodology, a value higher than a certain (default 0.95)
percentile value is considered as an outlier. Similarly, a value lower than a certain
(default 0.05) percentile value is considered as an outlier.
b) Standard Deviation Method: In this methodology, if a value is certain number of
standard deviations (default 3.0) away from the mean, then it is identified as an outlier.
c) Interquartile Range (IQR) Method: A value which is below Q1 – k * IQR or
above Q3 + k * IQR (default k is 1.5) are identified as outliers, where Q1 is first quartile/
25th percentile, Q3 is third quartile/75th percentile and IQR is difference between
third quartile &amp; first quartile.
If an attribute value is less (more) than its derived lower (upper) bound value,
it is considered as outlier by a methodology. A attribute value is considered as outlier
if it is declared as outlier by atleast &#39;min_validation&#39; methodologies (default 2).
treatment: Boolean argument – True or False. If True, outliers are treated as per treatment_method argument. (Default value = False)
treatment_method: null_replacement&#34;, &#34;row_removal&#34;, &#34;value_replacement&#34;.
In &#34;null_replacement&#34;, outlier values are replaced by null so that it can be imputed by a
reliable imputation methodology. In &#34;value_replacement&#34;, outlier values are replaced by
maximum or minimum permissible value by above methodologies. Lastly in &#34;row_removal&#34;, rows
are removed if it is found with any outlier. (Default value = &#39;value_replacement&#39;)
pre_existing_model: Boolean argument – True or False. True if the model with upper/lower permissible values
for each attribute exists already to be used, False otherwise. (Default value = False)
model_path: If pre_existing_model is True, this argument is path for the pre-saved model.
If pre_existing_model is False, this field can be used for saving the model.
Default &#34;NA&#34; means there is neither pre-existing model nor there is a need to save one.
output_mode: replace&#34;, &#34;append&#34;.
“replace” option replaces original columns with treated column. “append” option append treated
column to the input dataset with a postfix &#34;_outliered&#34; e.g. column X is appended as X_outliered. (Default value = &#39;replace&#39;)
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
&#39;pctile_upper&#39;: 0.95:
&#39;stdev_lower&#39;: 3.0:
&#39;stdev_upper&#39;: 3.0:
&#39;IQR_lower&#39;: 1.5:
&#39;IQR_upper&#39;: 1.5:
&#39;min_validation&#39;: 2}:
print_impact:
(Default value = False)
Returns:
Output Dataframe, Metric Dataframe)
Output Dataframe is the imputed dataframe if treated, else original input dataframe.
Metric Dataframe is of schema [attribute, lower_outliers, upper_outliers]. lower_outliers is no. of outliers
found in the lower spectrum of the attribute range and upper_outliers is outlier count in the upper spectrum.
&#34;&#34;&#34;
num_cols = attributeType_segregation(idf)[0]
if len(num_cols) == 0:
warnings.warn(&#34;No Outlier Check - No numerical column(s) to analyse&#34;)
odf = idf
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;lower_outliers&#39;, T.StringType(), True),
T.StructField(&#39;upper_outliers&#39;, T.StringType(), True)])
odf_print = spark.sparkContext.emptyRDD().toDF(schema)
return odf, odf_print
if list_of_cols == &#39;all&#39;:
list_of_cols = num_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
if stats_unique == {}:
remove_cols = uniqueCount_computation(spark, idf, list_of_cols).where(F.col(&#39;unique_values&#39;) &lt; 2) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
else:
remove_cols = read_dataset(spark, **stats_unique).where(F.col(&#39;unique_values&#39;) &lt; 2) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
list_of_cols = list(set([e for e in list_of_cols if e not in (drop_cols + remove_cols)]))
if any(x not in num_cols for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if detection_side not in (&#39;upper&#39;, &#39;lower&#39;, &#39;both&#39;):
raise TypeError(&#39;Invalid input for detection_side&#39;)
if treatment_method not in (&#39;null_replacement&#39;, &#39;row_removal&#39;, &#39;value_replacement&#39;):
raise TypeError(&#39;Invalid input for treatment_method&#39;)
if output_mode not in (&#39;replace&#39;, &#39;append&#39;):
raise TypeError(&#39;Invalid input for output_mode&#39;)
if str(treatment).lower() == &#39;true&#39;:
treatment = True
elif str(treatment).lower() == &#39;false&#39;:
treatment = False
else:
raise TypeError(&#39;Non-Boolean input for treatment&#39;)
if str(pre_existing_model).lower() == &#39;true&#39;:
pre_existing_model = True
elif str(pre_existing_model).lower() == &#39;false&#39;:
pre_existing_model = False
else:
raise TypeError(&#39;Non-Boolean input for pre_existing_model&#39;)
for arg in [&#39;pctile_lower&#39;, &#39;pctile_upper&#39;]:
if arg in detection_configs:
if (detection_configs[arg] &lt; 0) | (detection_configs[arg] &gt; 1):
raise TypeError(&#39;Invalid input for &#39; + arg)
recast_cols = []
recast_type = []
for i in list_of_cols:
if get_dtype(idf, i).startswith(&#39;decimal&#39;):
idf = idf.withColumn(i, F.col(i).cast(T.DoubleType()))
recast_cols.append(i)
recast_type.append(get_dtype(idf, i))
if pre_existing_model:
df_model = spark.read.parquet(model_path + &#34;/outlier_numcols&#34;)
params = []
for i in list_of_cols:
mapped_value = df_model.where(F.col(&#39;attribute&#39;) == i).select(&#39;parameters&#39;) \
.rdd.flatMap(lambda x: x).collect()[0]
params.append(mapped_value)
pctile_params = idf.approxQuantile(list_of_cols, [detection_configs.get(&#39;pctile_lower&#39;, 0.05),
detection_configs.get(&#39;pctile_upper&#39;, 0.95)], 0.01)
skewed_cols = []
for i, p in zip(list_of_cols, pctile_params):
if p[0] == p[1]:
skewed_cols.append(i)
else:
detection_configs[&#39;pctile_lower&#39;] = detection_configs[&#39;pctile_lower&#39;] or 0.0
detection_configs[&#39;pctile_upper&#39;] = detection_configs[&#39;pctile_upper&#39;] or 1.0
pctile_params = idf.approxQuantile(list_of_cols, [detection_configs[&#39;pctile_lower&#39;],
detection_configs[&#39;pctile_upper&#39;]], 0.01)
skewed_cols = []
for i, p in zip(list_of_cols, pctile_params):
if p[0] == p[1]:
skewed_cols.append(i)
detection_configs[&#39;stdev_lower&#39;] = detection_configs[&#39;stdev_lower&#39;] or detection_configs[&#39;stdev_upper&#39;]
detection_configs[&#39;stdev_upper&#39;] = detection_configs[&#39;stdev_upper&#39;] or detection_configs[&#39;stdev_lower&#39;]
stdev_params = []
for i in list_of_cols:
mean, stdev = idf.select(F.mean(i), F.stddev(i)).first()
stdev_params.append(
[mean - detection_configs[&#39;stdev_lower&#39;] * stdev, mean + detection_configs[&#39;stdev_upper&#39;] * stdev])
detection_configs[&#39;IQR_lower&#39;] = detection_configs[&#39;IQR_lower&#39;] or detection_configs[&#39;IQR_upper&#39;]
detection_configs[&#39;IQR_upper&#39;] = detection_configs[&#39;IQR_upper&#39;] or detection_configs[&#39;IQR_lower&#39;]
quantiles = idf.approxQuantile(list_of_cols, [0.25, 0.75], 0.01)
IQR_params = [[e[0] - detection_configs[&#39;IQR_lower&#39;] * (e[1] - e[0]),
e[1] + detection_configs[&#39;IQR_upper&#39;] * (e[1] - e[0])] for e in quantiles]
n = detection_configs[&#39;min_validation&#39;]
params = [[sorted([x[0], y[0], z[0]], reverse=True)[n - 1], sorted([x[1], y[1], z[1]])[n - 1]] for x, y, z in
list(zip(pctile_params, stdev_params, IQR_params))]
# Saving model File if required
if model_path != &#34;NA&#34;:
df_model = spark.createDataFrame(zip(list_of_cols, params), schema=[&#39;attribute&#39;, &#39;parameters&#39;])
df_model.coalesce(1).write.parquet(model_path + &#34;/outlier_numcols&#34;, mode=&#39;overwrite&#39;)
for i, j in zip(recast_cols, recast_type):
idf = idf.withColumn(i, F.col(i).cast(j))
def composite_outlier(*v):
&#34;&#34;&#34;
Args:
*v:
Returns:
&#34;&#34;&#34;
output = []
for idx, e in enumerate(v):
if e is None:
output.append(None)
continue
if detection_side in (&#39;upper&#39;, &#39;both&#39;):
if e &gt; params[idx][1]:
output.append(1)
continue
if detection_side in (&#39;lower&#39;, &#39;both&#39;):
if e &lt; params[idx][0]:
output.append(-1)
continue
output.append(0)
return output
f_composite_outlier = F.udf(composite_outlier, T.ArrayType(T.IntegerType()))
odf = idf.withColumn(&#34;outliered&#34;, f_composite_outlier(*list_of_cols))
odf.persist()
output_print = []
for index, i in enumerate(list_of_cols):
odf = odf.withColumn(i + &#34;_outliered&#34;, F.col(&#39;outliered&#39;)[index])
output_print.append(
[i, odf.where(F.col(i + &#34;_outliered&#34;) == -1).count(), odf.where(F.col(i + &#34;_outliered&#34;) == 1).count()])
if treatment &amp; (treatment_method in (&#39;value_replacement&#39;, &#39;null_replacement&#39;)):
if skewed_cols:
warnings.warn(
&#34;Columns dropped from outlier treatment due to highly skewed distribution: &#34; + (&#39;,&#39;).join(skewed_cols))
if i not in skewed_cols:
replace_vals = {&#39;value_replacement&#39;: [params[index][0], params[index][1]],
&#39;null_replacement&#39;: [None, None]}
odf = odf.withColumn(i + &#34;_outliered&#34;,
F.when(F.col(i + &#34;_outliered&#34;) == 1, replace_vals[treatment_method][1]) \
.otherwise(F.when(F.col(i + &#34;_outliered&#34;) == -1, replace_vals[treatment_method][0]) \
.otherwise(F.col(i))))
if output_mode == &#39;replace&#39;:
odf = odf.drop(i).withColumnRenamed(i + &#34;_outliered&#34;, i)
else:
odf = odf.drop(i + &#34;_outliered&#34;)
odf = odf.drop(&#34;outliered&#34;)
if treatment &amp; (treatment_method == &#39;row_removal&#39;):
if skewed_cols:
warnings.warn(
&#34;Columns dropped from outlier treatment due to highly skewed distribution: &#34; + (&#39;,&#39;).join(skewed_cols))
for index, i in enumerate(list_of_cols):
if i not in skewed_cols:
odf = odf.where((F.col(i + &#34;_outliered&#34;) == 0) | (F.col(i + &#34;_outliered&#34;).isNull())).drop(
i + &#34;_outliered&#34;)
else:
odf = odf.drop(i + &#34;_outliered&#34;)
if not (treatment):
odf = idf
odf_print = spark.createDataFrame(output_print, schema=[&#39;attribute&#39;, &#39;lower_outliers&#39;, &#39;upper_outliers&#39;])
if print_impact:
odf_print.show(len(list_of_cols))
return odf, odf_print
```
</details>
</dd>
</dl>