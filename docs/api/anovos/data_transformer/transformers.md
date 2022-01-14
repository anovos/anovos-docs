# Module <code>transformers</code>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
# coding=utf-8
import warnings
import pyspark
from anovos.data_analyzer.stats_generator import missingCount_computation, uniqueCount_computation
from anovos.data_ingest.data_ingest import read_dataset
from anovos.shared.utils import attributeType_segregation, get_dtype
from pyspark.ml import Pipeline, PipelineModel
from pyspark.ml.feature import Imputer, ImputerModel
from pyspark.ml.feature import StringIndexer, OneHotEncoderEstimator
from pyspark.ml.linalg import DenseVector
from pyspark.sql import functions as F
from pyspark.sql import types as T
from pyspark.sql.window import Window
from scipy import stats
def attribute_binning(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], method_type=&#34;equal_range&#34;, bin_size=10,
bin_dtype=&#34;numerical&#34;,
pre_existing_model=False, model_path=&#34;NA&#34;, output_mode=&#34;replace&#34;, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of numerical columns to transform e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
bin_method: equal_frequency&#34;, &#34;equal_range&#34;.
In &#34;equal_range&#34; method, each bin is of equal size/width and in &#34;equal_frequency&#34;, each bin has
equal no. of rows, though the width of bins may vary.
bin_size: Number of bins. (Default value = 10)
bin_dtype: numerical&#34;, &#34;categorical&#34;.
With &#34;numerical&#34; option, original value is replaced with an Integer (1,2,…) and
with &#34;categorical&#34; option, original replaced with a string describing min and max value allowed
in the bin (&#34;minval-maxval&#34;). (Default value = &#34;numerical&#34;)
pre_existing_model: Boolean argument – True or False. True if binning model exists already, False Otherwise. (Default value = False)
model_path: If pre_existing_model is True, this argument is path for referring the pre-saved model.
If pre_existing_model is False, this argument can be used for saving the model.
Default &#34;NA&#34; means there is neither pre-existing model nor there is a need to save one.
output_mode: replace&#34;, &#34;append&#34;.
“replace” option replaces original columns with transformed column. “append” option append transformed
column to the input dataset with a postfix &#34;_binned&#34; e.g. column X is appended as X_binned. (Default value = &#34;replace&#34;)
method_type:
(Default value = &#34;equal_range&#34;)
print_impact:
(Default value = False)
Returns:
Binned Dataframe
&#34;&#34;&#34;
num_cols = attributeType_segregation(idf)[0]
if list_of_cols == &#39;all&#39;:
list_of_cols = num_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in num_cols for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if len(list_of_cols) == 0:
warnings.warn(&#34;No Binning Performed - No numerical column(s) to transform&#34;)
return idf
if method_type not in (&#34;equal_frequency&#34;, &#34;equal_range&#34;):
raise TypeError(&#39;Invalid input for method_type&#39;)
if bin_size &lt; 2:
raise TypeError(&#39;Invalid input for bin_size&#39;)
if output_mode not in (&#39;replace&#39;, &#39;append&#39;):
raise TypeError(&#39;Invalid input for output_mode&#39;)
if pre_existing_model:
df_model = spark.read.parquet(model_path + &#34;/attribute_binning&#34;)
bin_cutoffs = []
for i in list_of_cols:
mapped_value = df_model.where(F.col(&#39;attribute&#39;) == i).select(&#39;parameters&#39;) \
.rdd.flatMap(lambda x: x).collect()[0]
bin_cutoffs.append(mapped_value)
else:
if method_type == &#34;equal_frequency&#34;:
pctile_width = 1 / bin_size
pctile_cutoff = []
for j in range(1, bin_size):
pctile_cutoff.append(j * pctile_width)
bin_cutoffs = idf.approxQuantile(list_of_cols, pctile_cutoff, 0.01)
else:
bin_cutoffs = []
for i in list_of_cols:
max_val = (idf.select(F.col(i)).groupBy().max().rdd.flatMap(lambda x: x).collect() + [None])[0]
min_val = (idf.select(F.col(i)).groupBy().min().rdd.flatMap(lambda x: x).collect() + [None])[0]
bin_cutoff = []
if max_val:
bin_width = (max_val - min_val) / bin_size
for j in range(1, bin_size):
bin_cutoff.append(min_val + j * bin_width)
bin_cutoffs.append(bin_cutoff)
if model_path != &#34;NA&#34;:
df_model = spark.createDataFrame(zip(list_of_cols, bin_cutoffs), schema=[&#39;attribute&#39;, &#39;parameters&#39;])
df_model.write.parquet(model_path + &#34;/attribute_binning&#34;, mode=&#39;overwrite&#39;)
def bucket_label(value, index):
&#34;&#34;&#34;
Args:
value:
index:
Returns:
&#34;&#34;&#34;
if value is None:
return None
for j in range(0, len(bin_cutoffs[index])):
if value &lt;= bin_cutoffs[index][j]:
if bin_dtype == &#34;numerical&#34;:
return j + 1
else:
if j == 0:
return &#34;&lt;= &#34; + str(round(bin_cutoffs[index][j], 4))
else:
return str(round(bin_cutoffs[index][j - 1], 4)) + &#34;-&#34; + str(round(bin_cutoffs[index][j], 4))
else:
next
if bin_dtype == &#34;numerical&#34;:
return len(bin_cutoffs[0]) + 1
else:
return &#34;&gt; &#34; + str(round(bin_cutoffs[index][len(bin_cutoffs[0]) - 1], 4))
if bin_dtype == &#34;numerical&#34;:
f_bucket_label = F.udf(bucket_label, T.IntegerType())
else:
f_bucket_label = F.udf(bucket_label, T.StringType())
odf = idf
for idx, i in enumerate(list_of_cols):
odf = odf.withColumn(i + &#34;_binned&#34;, f_bucket_label(F.col(i), F.lit(idx)))
if idx % 5 == 0:
odf.persist(pyspark.StorageLevel.MEMORY_AND_DISK).count()
if output_mode == &#39;replace&#39;:
for col in list_of_cols:
odf = odf.drop(col).withColumnRenamed(col + &#34;_binned&#34;, col)
if print_impact:
if output_mode == &#39;replace&#39;:
output_cols = list_of_cols
else:
output_cols = [(i + &#34;_binned&#34;) for i in list_of_cols]
uniqueCount_computation(spark, odf, output_cols).show(len(output_cols))
return odf
def monotonic_binning(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], label_col=&#39;label&#39;, event_label=1,
bin_method=&#34;equal_range&#34;, bin_size=10, bin_dtype=&#34;numerical&#34;, output_mode=&#34;replace&#34;):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of numerical columns to transform e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
label_col: Label/Target column (Default value = &#39;label&#39;)
event_label: Value of (positive) event (i.e label 1) (Default value = 1)
bin_method: equal_frequency&#34;, &#34;equal_range&#34;.
In &#34;equal_range&#34; method, each bin is of equal size/width and in &#34;equal_frequency&#34;, each bin has
equal no. of rows, though the width of bins may vary. (Default value = &#34;equal_range&#34;)
bin_size: Default number of bins in case monotonicity is not achieved.
bin_dtype: numerical&#34;, &#34;categorical&#34;.
With &#34;numerical&#34; option, original value is replaced with an Integer (1,2,…) and
with &#34;categorical&#34; option, original replaced with a string describing min and max value allowed
in the bin (&#34;minval-maxval&#34;). (Default value = &#34;numerical&#34;)
output_mode: replace&#34;, &#34;append&#34;.
“replace” option replaces original columns with transformed column. “append” option append transformed
column to the input dataset with a postfix &#34;_binned&#34; e.g. column X is appended as X_binned. (Default value = &#34;replace&#34;)
Returns:
Binned Dataframe
&#34;&#34;&#34;
num_cols = attributeType_segregation(idf)[0]
if list_of_cols == &#39;all&#39;:
list_of_cols = num_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in (drop_cols + [label_col])]))
if any(x not in num_cols for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
attribute_binning(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], method_type=&#34;equal_range&#34;, bin_size=10,
pre_existing_model=False, model_path=&#34;NA&#34;, output_mode=&#34;replace&#34;, print_impact=False)
odf = idf
for col in list_of_cols:
n = 20
r = 0
while n &gt; 2:
tmp = attribute_binning(spark, idf, [col], drop_cols=[], method_type=bin_method, bin_size=n,
output_mode=&#39;append&#39;) \
.select(label_col, col, col + &#39;_binned&#39;) \
.withColumn(label_col, F.when(F.col(label_col) == event_label, 1).otherwise(0)) \
.groupBy(col + &#39;_binned&#39;).agg(F.avg(col).alias(&#39;mean_val&#39;),
F.avg(label_col).alias(&#39;mean_label&#39;)).dropna()
r, p = stats.spearmanr(tmp.toPandas()[[&#39;mean_val&#39;]], tmp.toPandas()[[&#39;mean_label&#39;]])
if r == 1.0:
odf = attribute_binning(spark, odf, [col], drop_cols=[], method_type=bin_method, bin_size=n,
bin_dtype=bin_dtype, output_mode=output_mode)
break
n = n - 1
r = 0
if r &lt; 1.0:
odf = attribute_binning(spark, odf, [col], drop_cols=[], method_type=bin_method, bin_size=bin_size,
bin_dtype=bin_dtype, output_mode=output_mode)
return odf
def cat_to_num_unsupervised(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], method_type=1, index_order=&#39;frequencyDesc&#39;, cardinality_threshold=100,
pre_existing_model=False, model_path=&#34;NA&#34;, output_mode=&#39;replace&#39;, stats_unique={}, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of categorical columns to transform e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all categorical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
method_type: 1 for Label Encoding or 0 for One hot encoding.
In label encoding, each categorical value is assigned a unique integer based on alphabetical
or frequency ordering (both ascending &amp; descending options are available that can be selected by
index_order argument).
In one-hot encoding, every unique value in the column will be added in a form of dummy/binary column. (Default value = 1)
index_order: frequencyDesc&#34;, &#34;frequencyAsc&#34;, &#34;alphabetDesc&#34;, &#34;alphabetAsc&#34;.
Valid only for Label Encoding method_type. (Default value = &#39;frequencyDesc&#39;)
cardinality_threshold: Defines threshold to skip columns with higher cardinality values from encoding. Default value is 100.
pre_existing_model: Boolean argument – True or False. True if encoding model exists already, False Otherwise. (Default value = False)
model_path: If pre_existing_model is True, this argument is path for referring the pre-saved model.
If pre_existing_model is False, this argument can be used for saving the model.
Default &#34;NA&#34; means there is neither pre existing model nor there is a need to save one.
output_mode: replace&#34;, &#34;append&#34;.
“replace” option replaces original columns with transformed column. “append” option append transformed
column to the input dataset with a postfix &#34;_index&#34; e.g. column X is appended as X_index. (Default value = &#39;replace&#39;)
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)
Returns:
Encoded Dataframe
&#34;&#34;&#34;
cat_cols = attributeType_segregation(idf)[1]
if list_of_cols == &#39;all&#39;:
list_of_cols = cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in cat_cols for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if len(list_of_cols) == 0:
warnings.warn(&#34;No Encoding Computation - No categorical column(s) to transform&#34;)
return idf
if method_type not in (0, 1):
raise TypeError(&#39;Invalid input for method_type&#39;)
if index_order not in (&#39;frequencyDesc&#39;, &#39;frequencyAsc&#39;, &#39;alphabetDesc&#39;, &#39;alphabetAsc&#39;):
raise TypeError(&#39;Invalid input for Encoding Index Order&#39;)
if output_mode not in (&#39;replace&#39;, &#39;append&#39;):
raise TypeError(&#39;Invalid input for output_mode&#39;)
if pre_existing_model:
pipelineModel = PipelineModel.load(model_path + &#34;/cat_to_num_unsupervised/indexer&#34;)
else:
stages = []
for i in list_of_cols:
stringIndexer = StringIndexer(inputCol=i, outputCol=i + &#39;_index&#39;,
stringOrderType=index_order, handleInvalid=&#39;keep&#39;)
stages += [stringIndexer]
pipeline = Pipeline(stages=stages)
pipelineModel = pipeline.fit(idf)
odf_indexed = pipelineModel.transform(idf)
if method_type == 0:
list_of_cols_vec = []
list_of_cols_idx = []
for i in list_of_cols:
list_of_cols_vec.append(i + &#34;_vec&#34;)
list_of_cols_idx.append(i + &#34;_index&#34;)
if pre_existing_model:
encoder = OneHotEncoderEstimator.load(model_path + &#34;/cat_to_num_unsupervised/encoder&#34;)
else:
encoder = OneHotEncoderEstimator(inputCols=list_of_cols_idx, outputCols=list_of_cols_vec,
handleInvalid=&#39;keep&#39;)
odf_encoded = encoder.fit(odf_indexed).transform(odf_indexed)
odf = odf_encoded
def vector_to_array(v):
&#34;&#34;&#34;
Args:
v:
Returns:
&#34;&#34;&#34;
v = DenseVector(v)
new_array = list([int(x) for x in v])
return new_array
f_vector_to_array = F.udf(vector_to_array, T.ArrayType(T.IntegerType()))
if stats_unique != {}:
stats_df = read_dataset(spark, **stats_unique)
skipped_cols = []
for i in list_of_cols:
if stats_unique == {}:
uniq_cats = idf.select(i).distinct().count()
else:
uniq_cats = stats_df.where(F.col(&#34;attribute&#34;) == i).select(&#34;unique_values&#34;).rdd.flatMap(lambda x: x).collect()[0]
if uniq_cats &gt; cardinality_threshold:
skipped_cols.append(i)
odf = odf.drop(i + &#39;_vec&#39;, i + &#39;_index&#39;)
continue
odf_schema = odf.schema
odf_schema = odf_schema.add(T.StructField(&#34;tmp&#34;,T.ArrayType(T.IntegerType())))
for j in range(0, uniq_cats):
odf_schema = odf_schema.add(T.StructField(i + &#34;_&#34; + str(j),T.IntegerType()))
odf = odf.withColumn(&#34;tmp&#34;, f_vector_to_array(i + &#39;_vec&#39;)).rdd.map(lambda x: (*x, *x[&#34;tmp&#34;])).toDF(schema=odf_schema)
if output_mode == &#39;replace&#39;:
odf = odf.drop(i, i + &#39;_vec&#39;, i + &#39;_index&#39;, &#39;tmp&#39;)
else:
odf = odf.drop(i + &#39;_vec&#39;, i + &#39;_index&#39;, &#39;tmp&#39;)
if skipped_cols:
warnings.warn(
&#34;Columns dropped from one-hot encoding due to high cardinality: &#34; + (&#39;,&#39;).join(skipped_cols))
else:
odf = odf_indexed
for i in list_of_cols:
odf = odf.withColumn(i + &#39;_index&#39;, F.when(F.col(i).isNull(), None)
.otherwise(F.col(i + &#39;_index&#39;).cast(T.IntegerType())))
if output_mode == &#39;replace&#39;:
for i in list_of_cols:
odf = odf.drop(i).withColumnRenamed(i + &#39;_index&#39;, i)
odf = odf.select(idf.columns)
if (pre_existing_model == False) &amp; (model_path != &#34;NA&#34;):
pipelineModel.write().overwrite().save(model_path + &#34;/cat_to_num_unsupervised/indexer&#34;)
if method_type == 0:
encoder.write().overwrite().save(model_path + &#34;/cat_to_num_unsupervised/encoder&#34;)
if (print_impact == True) &amp; (method_type == 1):
print(&#34;Before&#34;)
idf.describe().where(F.col(&#39;summary&#39;).isin(&#39;count&#39;, &#39;min&#39;, &#39;max&#39;)).show()
print(&#34;After&#34;)
odf.describe().where(F.col(&#39;summary&#39;).isin(&#39;count&#39;, &#39;min&#39;, &#39;max&#39;)).show()
if (print_impact == True) &amp; (method_type == 0):
print(&#34;Before&#34;)
idf.printSchema()
print(&#34;After&#34;)
odf.printSchema()
return odf
def imputation_MMM(spark, idf, list_of_cols=&#34;missing&#34;, drop_cols=[], method_type=&#34;median&#34;, pre_existing_model=False,
model_path=&#34;NA&#34;,
output_mode=&#34;replace&#34;, stats_missing={}, stats_mode={}, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of columns to impute e.g., [&#34;col1&#34;,&#34;col2&#34;].
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
method_type: median&#34;, &#34;mean&#34; (valid only for for numerical columns attributes).
Mode is only option for categorical columns. (Default value = &#34;median&#34;)
pre_existing_model: Boolean argument – True or False. True if imputation model exists already, False otherwise. (Default value = False)
model_path: If pre_existing_model is True, this argument is path for referring the pre-saved model.
If pre_existing_model is False, this argument can be used for saving the model.
Default &#34;NA&#34; means there is neither pre-existing model nor there is a need to save one.
output_mode: replace&#34;, &#34;append&#34;.
“replace” option replaces original columns with transformed column. “append” option append transformed
column to the input dataset with a postfix &#34;_imputed&#34; e.g. column X is appended as X_imputed. (Default value = &#34;replace&#34;)
stats_missing: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on missing count/pct i.e. if measures_of_counts or
missingCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
stats_mode: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on most frequently seen values i.e. if measures_of_centralTendency or
mode_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)
Returns:
Imputed Dataframe
&#34;&#34;&#34;
if stats_missing == {}:
missing_df = missingCount_computation(spark, idf)
else:
missing_df = read_dataset(spark, **stats_missing).select(&#39;attribute&#39;, &#39;missing_count&#39;, &#39;missing_pct&#39;)
missing_cols = missing_df.where(F.col(&#39;missing_count&#39;) &gt; 0).select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
if str(pre_existing_model).lower() == &#39;true&#39;:
pre_existing_model = True
elif str(pre_existing_model).lower() == &#39;false&#39;:
pre_existing_model = False
else:
raise TypeError(&#39;Non-Boolean input for pre_existing_model&#39;)
if (len(missing_cols) == 0) &amp; (pre_existing_model == False) &amp; (model_path == &#34;NA&#34;):
return idf
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
warnings.warn(&#34;No Imputation performed- No column(s) to impute&#34;)
return idf
if any(x not in idf.columns for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if method_type not in (&#39;mode&#39;, &#39;mean&#39;, &#39;median&#39;):
raise TypeError(&#39;Invalid input for method_type&#39;)
if output_mode not in (&#39;replace&#39;, &#39;append&#39;):
raise TypeError(&#39;Invalid input for output_mode&#39;)
num_cols, cat_cols, other_cols = attributeType_segregation(idf.select(list_of_cols))
odf = idf
if len(num_cols) &gt; 0:
# Checking for Integer/Decimal Type Columns &amp; Converting them into Float/Double Type
recast_cols = []
recast_type = []
for i in num_cols:
if get_dtype(idf, i) not in (&#39;float&#39;, &#39;double&#39;):
odf = odf.withColumn(i, F.col(i).cast(T.DoubleType()))
recast_cols.append(i + &#34;_imputed&#34;)
recast_type.append(get_dtype(idf, i))
# For mode imputation
if method_type == &#39;mode&#39;:
if stats_mode == {}:
parameters = [str((idf.select(i).dropna().groupby(i).count().orderBy(&#34;count&#34;, ascending=False).first()
or [None])[0]) for i in num_cols]
else:
mode_df = read_dataset(spark, **stats_mode).replace(&#39;None&#39;, None)
mode_df_cols = list(mode_df.select(&#39;attribute&#39;).toPandas()[&#39;attribute&#39;])
parameters = []
for i in num_cols:
if i not in mode_df_cols:
parameters.append(str((idf.select(i).dropna().groupby(i).count().orderBy(&#34;count&#34;, ascending=False).first()
or [None])[0]))
else:
parameters.append(mode_df.where(F.col(&#39;attribute&#39;) == i).select(&#39;mode&#39;).rdd.flatMap(list).collect()[0])
for index, i in enumerate(num_cols):
odf = odf.withColumn(i + &#34;_imputed&#34;, F.when(F.col(i).isNull(), parameters[index]).otherwise(F.col(i)))
else: #For mean, median imputation
# Building new imputer model or uploading the existing model
if pre_existing_model == True:
imputerModel = ImputerModel.load(model_path + &#34;/imputation_MMM/num_imputer-model&#34;)
else:
imputer = Imputer(strategy=method_type, inputCols=num_cols,
outputCols=[(e + &#34;_imputed&#34;) for e in num_cols])
imputerModel = imputer.fit(odf)
# Applying model
# odf = recast_column(imputerModel.transform(odf), recast_cols, recast_type)
odf = imputerModel.transform(odf)
for i, j in zip(recast_cols, recast_type):
odf = odf.withColumn(i, F.col(i).cast(j))
# Saving model if required
if (pre_existing_model == False) &amp; (model_path != &#34;NA&#34;):
imputerModel.write().overwrite().save(model_path + &#34;/imputation_MMM/num_imputer-model&#34;)
if len(cat_cols) &gt; 0:
if pre_existing_model:
df_model = spark.read.csv(model_path + &#34;/imputation_MMM/cat_imputer&#34;, header=True, inferSchema=True)
parameters = []
for i in cat_cols:
mapped_value = \
df_model.where(F.col(&#39;attribute&#39;) == i).select(&#39;parameters&#39;).rdd.flatMap(lambda x: x).collect()[0]
parameters.append(mapped_value)
else:
if stats_mode == {}:
parameters = [str((idf.select(i).dropna().groupby(i).count().orderBy(&#34;count&#34;, ascending=False).first()
or [None])[0]) for i in cat_cols]
else:
mode_df = read_dataset(spark, **stats_mode).replace(&#39;None&#39;, None)
parameters = [mode_df.where(F.col(&#39;attribute&#39;) == i).select(&#39;mode&#39;).rdd.flatMap(list).collect()[0] for i
in cat_cols]
for index, i in enumerate(cat_cols):
odf = odf.withColumn(i + &#34;_imputed&#34;, F.when(F.col(i).isNull(), parameters[index]).otherwise(F.col(i)))
# Saving model File if required
if (pre_existing_model == False) &amp; (model_path != &#34;NA&#34;):
df_model = spark.createDataFrame(zip(cat_cols, parameters), schema=[&#39;attribute&#39;, &#39;parameters&#39;])
df_model.repartition(1).write.csv(model_path + &#34;/imputation_MMM/cat_imputer&#34;, header=True, mode=&#39;overwrite&#39;)
for i in (num_cols + cat_cols):
if i not in missing_cols:
odf = odf.drop(i + &#34;_imputed&#34;)
elif output_mode == &#39;replace&#39;:
odf = odf.drop(i).withColumnRenamed(i + &#34;_imputed&#34;, i)
if print_impact:
if output_mode == &#39;replace&#39;:
odf_print = missing_df.select(&#39;attribute&#39;, F.col(&#34;missing_count&#34;).alias(&#34;missingCount_before&#34;)) \
.join(missingCount_computation(spark, odf, list_of_cols) \
.select(&#39;attribute&#39;, F.col(&#34;missing_count&#34;).alias(&#34;missingCount_after&#34;)), &#39;attribute&#39;, &#39;inner&#39;)
else:
output_cols = [(i + &#34;_imputed&#34;) for i in [e for e in (num_cols + cat_cols) if e in missing_cols]]
odf_print = missing_df.select(&#39;attribute&#39;, F.col(&#34;missing_count&#34;).alias(&#34;missingCount_before&#34;)) \
.join(missingCount_computation(spark, odf, output_cols) \
.withColumnRenamed(&#39;attribute&#39;, &#39;attribute_after&#39;) \
.withColumn(&#39;attribute&#39;, F.expr(&#34;substring(attribute_after, 1, length(attribute_after)-8)&#34;)) \
.drop(&#39;missing_pct&#39;), &#39;attribute&#39;, &#39;inner&#39;)
odf_print.show(len(list_of_cols))
return odf
def outlier_categories(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], coverage=1.0, max_category=50,
pre_existing_model=False, model_path=&#34;NA&#34;, output_mode=&#39;replace&#39;, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of categorical columns to transform e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all categorical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
coverage: Defines the minimum % of rows that will be mapped to actual category name and the rest to be mapped
to others and takes value between 0 to 1. Coverage of 0.8 can be interpreted as top frequently seen
categories are considered till it covers minimum 80% of rows and rest lesser seen values are mapped to others. (Default value = 1.0)
max_category: Even if coverage is less, only (max_category - 1) categories will be mapped to actual name and rest to others.
Caveat is when multiple categories have same rank, then #categories can be more than max_category. (Default value = 50)
pre_existing_model: Boolean argument – True or False. True if the model with the outlier/other values
for each attribute exists already to be used, False Otherwise. (Default value = False)
model_path: If pre_existing_model is True, this argument is path for the pre-saved model.
If pre_existing_model is False, this field can be used for saving the model.
Default &#34;NA&#34; means there is neither pre-existing model nor there is a need to save one.
output_mode: replace&#34;, &#34;append&#34;.
“replace” option replaces original columns with transformed column. “append” option append transformed
column to the input dataset with a postfix &#34;_outliered&#34; e.g. column X is appended as X_outliered. (Default value = &#39;replace&#39;)
print_impact:
(Default value = False)
Returns:
Dataframe after outlier treatment
&#34;&#34;&#34;
cat_cols = attributeType_segregation(idf)[1]
if list_of_cols == &#39;all&#39;:
list_of_cols = cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in cat_cols for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if len(list_of_cols) == 0:
warnings.warn(&#34;No Outlier Categories Computation - No categorical column(s) to transform&#34;)
return idf
if (coverage &lt;= 0) | (coverage &gt; 1):
raise TypeError(&#39;Invalid input for Coverage Value&#39;)
if max_category &lt; 2:
raise TypeError(&#39;Invalid input for Maximum No. of Categories Allowed&#39;)
if output_mode not in (&#39;replace&#39;, &#39;append&#39;):
raise TypeError(&#39;Invalid input for output_mode&#39;)
if pre_existing_model == True:
df_model = spark.read.csv(model_path + &#34;/outlier_categories&#34;, header=True, inferSchema=True)
else:
for index, i in enumerate(list_of_cols):
window = Window.partitionBy().orderBy(F.desc(&#39;count_pct&#39;))
df_cats = idf.groupBy(i).count().dropna() \
.withColumn(&#39;count_pct&#39;, F.col(&#39;count&#39;) / F.sum(&#39;count&#39;).over(Window.partitionBy())) \
.withColumn(&#39;rank&#39;, F.rank().over(window)) \
.withColumn(&#39;cumu&#39;, F.sum(&#39;count_pct&#39;).over(window.rowsBetween(Window.unboundedPreceding, 0))) \
.withColumn(&#39;lag_cumu&#39;, F.lag(&#39;cumu&#39;).over(window)).fillna(0) \
.where(~((F.col(&#39;cumu&#39;) &gt;= coverage) &amp; (F.col(&#39;lag_cumu&#39;) &gt;= coverage))) \
.where(F.col(&#39;rank&#39;) &lt;= (max_category - 1)) \
.select(F.lit(i).alias(&#39;attribute&#39;), F.col(i).alias(&#39;parameters&#39;))
if index == 0:
df_model = df_cats
else:
df_model = df_model.union(df_cats)
odf = idf
for i in list_of_cols:
parameters = df_model.where(F.col(&#39;attribute&#39;) == i).select(&#39;parameters&#39;).rdd.flatMap(lambda x: x).collect()
if output_mode == &#39;replace&#39;:
odf = odf.withColumn(i, F.when((F.col(i).isin(parameters)) | (F.col(i).isNull()), F.col(i)).otherwise(
&#34;others&#34;))
else:
odf = odf.withColumn(i + &#34;_outliered&#34;,
F.when((F.col(i).isin(parameters)) | (F.col(i).isNull()), F.col(i)).otherwise(
&#34;others&#34;))
# Saving model File if required
if (pre_existing_model == False) &amp; (model_path != &#34;NA&#34;):
df_model.repartition(1).write.csv(model_path + &#34;/outlier_categories&#34;, header=True, mode=&#39;overwrite&#39;)
if print_impact:
if output_mode == &#39;replace&#39;:
output_cols = list_of_cols
else:
output_cols = [(i + &#34;_outliered&#34;) for i in list_of_cols]
uniqueCount_computation(spark, idf, list_of_cols).select(&#39;attribute&#39;,
F.col(&#34;unique_values&#34;).alias(
&#34;uniqueValues_before&#34;)).show(
len(list_of_cols))
uniqueCount_computation(spark, odf, output_cols).select(&#39;attribute&#39;,
F.col(&#34;unique_values&#34;).alias(
&#34;uniqueValues_after&#34;)).show(
len(list_of_cols))
return odf
```
</details>
## Functions
<dl>
<dt id="anovos.data_transformer.transformers.attribute_binning"><code class="name flex">
<span>def <span class="ident">attribute_binning</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], method_type='equal_range', bin_size=10, bin_dtype='numerical', pre_existing_model=False, model_path='NA', output_mode='replace', print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of numerical columns to transform e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
bin_method: equal_frequency", "equal_range".
In "equal_range" method, each bin is of equal size/width and in "equal_frequency", each bin has
equal no. of rows, though the width of bins may vary.
bin_size: Number of bins. (Default value = 10)
bin_dtype: numerical", "categorical".
With "numerical" option, original value is replaced with an Integer (1,2,…) and
with "categorical" option, original replaced with a string describing min and max value allowed
in the bin ("minval-maxval"). (Default value = "numerical")
pre_existing_model: Boolean argument – True or False. True if binning model exists already, False Otherwise. (Default value = False)
model_path: If pre_existing_model is True, this argument is path for referring the pre-saved model.
If pre_existing_model is False, this argument can be used for saving the model.
Default "NA" means there is neither pre-existing model nor there is a need to save one.
output_mode: replace", "append".
“replace” option replaces original columns with transformed column. “append” option append transformed
column to the input dataset with a postfix "_binned" e.g. column X is appended as X_binned. (Default value = "replace")
method_type:
(Default value = "equal_range")
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Binned Dataframe</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def attribute_binning(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], method_type=&#34;equal_range&#34;, bin_size=10,
bin_dtype=&#34;numerical&#34;,
pre_existing_model=False, model_path=&#34;NA&#34;, output_mode=&#34;replace&#34;, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of numerical columns to transform e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
bin_method: equal_frequency&#34;, &#34;equal_range&#34;.
In &#34;equal_range&#34; method, each bin is of equal size/width and in &#34;equal_frequency&#34;, each bin has
equal no. of rows, though the width of bins may vary.
bin_size: Number of bins. (Default value = 10)
bin_dtype: numerical&#34;, &#34;categorical&#34;.
With &#34;numerical&#34; option, original value is replaced with an Integer (1,2,…) and
with &#34;categorical&#34; option, original replaced with a string describing min and max value allowed
in the bin (&#34;minval-maxval&#34;). (Default value = &#34;numerical&#34;)
pre_existing_model: Boolean argument – True or False. True if binning model exists already, False Otherwise. (Default value = False)
model_path: If pre_existing_model is True, this argument is path for referring the pre-saved model.
If pre_existing_model is False, this argument can be used for saving the model.
Default &#34;NA&#34; means there is neither pre-existing model nor there is a need to save one.
output_mode: replace&#34;, &#34;append&#34;.
“replace” option replaces original columns with transformed column. “append” option append transformed
column to the input dataset with a postfix &#34;_binned&#34; e.g. column X is appended as X_binned. (Default value = &#34;replace&#34;)
method_type:
(Default value = &#34;equal_range&#34;)
print_impact:
(Default value = False)
Returns:
Binned Dataframe
&#34;&#34;&#34;
num_cols = attributeType_segregation(idf)[0]
if list_of_cols == &#39;all&#39;:
list_of_cols = num_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in num_cols for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if len(list_of_cols) == 0:
warnings.warn(&#34;No Binning Performed - No numerical column(s) to transform&#34;)
return idf
if method_type not in (&#34;equal_frequency&#34;, &#34;equal_range&#34;):
raise TypeError(&#39;Invalid input for method_type&#39;)
if bin_size &lt; 2:
raise TypeError(&#39;Invalid input for bin_size&#39;)
if output_mode not in (&#39;replace&#39;, &#39;append&#39;):
raise TypeError(&#39;Invalid input for output_mode&#39;)
if pre_existing_model:
df_model = spark.read.parquet(model_path + &#34;/attribute_binning&#34;)
bin_cutoffs = []
for i in list_of_cols:
mapped_value = df_model.where(F.col(&#39;attribute&#39;) == i).select(&#39;parameters&#39;) \
.rdd.flatMap(lambda x: x).collect()[0]
bin_cutoffs.append(mapped_value)
else:
if method_type == &#34;equal_frequency&#34;:
pctile_width = 1 / bin_size
pctile_cutoff = []
for j in range(1, bin_size):
pctile_cutoff.append(j * pctile_width)
bin_cutoffs = idf.approxQuantile(list_of_cols, pctile_cutoff, 0.01)
else:
bin_cutoffs = []
for i in list_of_cols:
max_val = (idf.select(F.col(i)).groupBy().max().rdd.flatMap(lambda x: x).collect() + [None])[0]
min_val = (idf.select(F.col(i)).groupBy().min().rdd.flatMap(lambda x: x).collect() + [None])[0]
bin_cutoff = []
if max_val:
bin_width = (max_val - min_val) / bin_size
for j in range(1, bin_size):
bin_cutoff.append(min_val + j * bin_width)
bin_cutoffs.append(bin_cutoff)
if model_path != &#34;NA&#34;:
df_model = spark.createDataFrame(zip(list_of_cols, bin_cutoffs), schema=[&#39;attribute&#39;, &#39;parameters&#39;])
df_model.write.parquet(model_path + &#34;/attribute_binning&#34;, mode=&#39;overwrite&#39;)
def bucket_label(value, index):
&#34;&#34;&#34;
Args:
value:
index:
Returns:
&#34;&#34;&#34;
if value is None:
return None
for j in range(0, len(bin_cutoffs[index])):
if value &lt;= bin_cutoffs[index][j]:
if bin_dtype == &#34;numerical&#34;:
return j + 1
else:
if j == 0:
return &#34;&lt;= &#34; + str(round(bin_cutoffs[index][j], 4))
else:
return str(round(bin_cutoffs[index][j - 1], 4)) + &#34;-&#34; + str(round(bin_cutoffs[index][j], 4))
else:
next
if bin_dtype == &#34;numerical&#34;:
return len(bin_cutoffs[0]) + 1
else:
return &#34;&gt; &#34; + str(round(bin_cutoffs[index][len(bin_cutoffs[0]) - 1], 4))
if bin_dtype == &#34;numerical&#34;:
f_bucket_label = F.udf(bucket_label, T.IntegerType())
else:
f_bucket_label = F.udf(bucket_label, T.StringType())
odf = idf
for idx, i in enumerate(list_of_cols):
odf = odf.withColumn(i + &#34;_binned&#34;, f_bucket_label(F.col(i), F.lit(idx)))
if idx % 5 == 0:
odf.persist(pyspark.StorageLevel.MEMORY_AND_DISK).count()
if output_mode == &#39;replace&#39;:
for col in list_of_cols:
odf = odf.drop(col).withColumnRenamed(col + &#34;_binned&#34;, col)
if print_impact:
if output_mode == &#39;replace&#39;:
output_cols = list_of_cols
else:
output_cols = [(i + &#34;_binned&#34;) for i in list_of_cols]
uniqueCount_computation(spark, odf, output_cols).show(len(output_cols))
return odf
```
</details>
</dd>
<dt id="anovos.data_transformer.transformers.cat_to_num_unsupervised"><code class="name flex">
<span>def <span class="ident">cat_to_num_unsupervised</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], method_type=1, index_order='frequencyDesc', cardinality_threshold=100, pre_existing_model=False, model_path='NA', output_mode='replace', stats_unique={}, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of categorical columns to transform e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all categorical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
method_type: 1 for Label Encoding or 0 for One hot encoding.
In label encoding, each categorical value is assigned a unique integer based on alphabetical
or frequency ordering (both ascending &amp; descending options are available that can be selected by
index_order argument).
In one-hot encoding, every unique value in the column will be added in a form of dummy/binary column. (Default value = 1)
index_order: frequencyDesc", "frequencyAsc", "alphabetDesc", "alphabetAsc".
Valid only for Label Encoding method_type. (Default value = 'frequencyDesc')
cardinality_threshold: Defines threshold to skip columns with higher cardinality values from encoding. Default value is 100.
pre_existing_model: Boolean argument – True or False. True if encoding model exists already, False Otherwise. (Default value = False)
model_path: If pre_existing_model is True, this argument is path for referring the pre-saved model.
If pre_existing_model is False, this argument can be used for saving the model.
Default "NA" means there is neither pre existing model nor there is a need to save one.
output_mode: replace", "append".
“replace” option replaces original columns with transformed column. “append” option append transformed
column to the input dataset with a postfix "_index" e.g. column X is appended as X_index. (Default value = 'replace')
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Encoded Dataframe</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def cat_to_num_unsupervised(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], method_type=1, index_order=&#39;frequencyDesc&#39;, cardinality_threshold=100,
pre_existing_model=False, model_path=&#34;NA&#34;, output_mode=&#39;replace&#39;, stats_unique={}, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of categorical columns to transform e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all categorical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
method_type: 1 for Label Encoding or 0 for One hot encoding.
In label encoding, each categorical value is assigned a unique integer based on alphabetical
or frequency ordering (both ascending &amp; descending options are available that can be selected by
index_order argument).
In one-hot encoding, every unique value in the column will be added in a form of dummy/binary column. (Default value = 1)
index_order: frequencyDesc&#34;, &#34;frequencyAsc&#34;, &#34;alphabetDesc&#34;, &#34;alphabetAsc&#34;.
Valid only for Label Encoding method_type. (Default value = &#39;frequencyDesc&#39;)
cardinality_threshold: Defines threshold to skip columns with higher cardinality values from encoding. Default value is 100.
pre_existing_model: Boolean argument – True or False. True if encoding model exists already, False Otherwise. (Default value = False)
model_path: If pre_existing_model is True, this argument is path for referring the pre-saved model.
If pre_existing_model is False, this argument can be used for saving the model.
Default &#34;NA&#34; means there is neither pre existing model nor there is a need to save one.
output_mode: replace&#34;, &#34;append&#34;.
“replace” option replaces original columns with transformed column. “append” option append transformed
column to the input dataset with a postfix &#34;_index&#34; e.g. column X is appended as X_index. (Default value = &#39;replace&#39;)
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)
Returns:
Encoded Dataframe
&#34;&#34;&#34;
cat_cols = attributeType_segregation(idf)[1]
if list_of_cols == &#39;all&#39;:
list_of_cols = cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in cat_cols for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if len(list_of_cols) == 0:
warnings.warn(&#34;No Encoding Computation - No categorical column(s) to transform&#34;)
return idf
if method_type not in (0, 1):
raise TypeError(&#39;Invalid input for method_type&#39;)
if index_order not in (&#39;frequencyDesc&#39;, &#39;frequencyAsc&#39;, &#39;alphabetDesc&#39;, &#39;alphabetAsc&#39;):
raise TypeError(&#39;Invalid input for Encoding Index Order&#39;)
if output_mode not in (&#39;replace&#39;, &#39;append&#39;):
raise TypeError(&#39;Invalid input for output_mode&#39;)
if pre_existing_model:
pipelineModel = PipelineModel.load(model_path + &#34;/cat_to_num_unsupervised/indexer&#34;)
else:
stages = []
for i in list_of_cols:
stringIndexer = StringIndexer(inputCol=i, outputCol=i + &#39;_index&#39;,
stringOrderType=index_order, handleInvalid=&#39;keep&#39;)
stages += [stringIndexer]
pipeline = Pipeline(stages=stages)
pipelineModel = pipeline.fit(idf)
odf_indexed = pipelineModel.transform(idf)
if method_type == 0:
list_of_cols_vec = []
list_of_cols_idx = []
for i in list_of_cols:
list_of_cols_vec.append(i + &#34;_vec&#34;)
list_of_cols_idx.append(i + &#34;_index&#34;)
if pre_existing_model:
encoder = OneHotEncoderEstimator.load(model_path + &#34;/cat_to_num_unsupervised/encoder&#34;)
else:
encoder = OneHotEncoderEstimator(inputCols=list_of_cols_idx, outputCols=list_of_cols_vec,
handleInvalid=&#39;keep&#39;)
odf_encoded = encoder.fit(odf_indexed).transform(odf_indexed)
odf = odf_encoded
def vector_to_array(v):
&#34;&#34;&#34;
Args:
v:
Returns:
&#34;&#34;&#34;
v = DenseVector(v)
new_array = list([int(x) for x in v])
return new_array
f_vector_to_array = F.udf(vector_to_array, T.ArrayType(T.IntegerType()))
if stats_unique != {}:
stats_df = read_dataset(spark, **stats_unique)
skipped_cols = []
for i in list_of_cols:
if stats_unique == {}:
uniq_cats = idf.select(i).distinct().count()
else:
uniq_cats = stats_df.where(F.col(&#34;attribute&#34;) == i).select(&#34;unique_values&#34;).rdd.flatMap(lambda x: x).collect()[0]
if uniq_cats &gt; cardinality_threshold:
skipped_cols.append(i)
odf = odf.drop(i + &#39;_vec&#39;, i + &#39;_index&#39;)
continue
odf_schema = odf.schema
odf_schema = odf_schema.add(T.StructField(&#34;tmp&#34;,T.ArrayType(T.IntegerType())))
for j in range(0, uniq_cats):
odf_schema = odf_schema.add(T.StructField(i + &#34;_&#34; + str(j),T.IntegerType()))
odf = odf.withColumn(&#34;tmp&#34;, f_vector_to_array(i + &#39;_vec&#39;)).rdd.map(lambda x: (*x, *x[&#34;tmp&#34;])).toDF(schema=odf_schema)
if output_mode == &#39;replace&#39;:
odf = odf.drop(i, i + &#39;_vec&#39;, i + &#39;_index&#39;, &#39;tmp&#39;)
else:
odf = odf.drop(i + &#39;_vec&#39;, i + &#39;_index&#39;, &#39;tmp&#39;)
if skipped_cols:
warnings.warn(
&#34;Columns dropped from one-hot encoding due to high cardinality: &#34; + (&#39;,&#39;).join(skipped_cols))
else:
odf = odf_indexed
for i in list_of_cols:
odf = odf.withColumn(i + &#39;_index&#39;, F.when(F.col(i).isNull(), None)
.otherwise(F.col(i + &#39;_index&#39;).cast(T.IntegerType())))
if output_mode == &#39;replace&#39;:
for i in list_of_cols:
odf = odf.drop(i).withColumnRenamed(i + &#39;_index&#39;, i)
odf = odf.select(idf.columns)
if (pre_existing_model == False) &amp; (model_path != &#34;NA&#34;):
pipelineModel.write().overwrite().save(model_path + &#34;/cat_to_num_unsupervised/indexer&#34;)
if method_type == 0:
encoder.write().overwrite().save(model_path + &#34;/cat_to_num_unsupervised/encoder&#34;)
if (print_impact == True) &amp; (method_type == 1):
print(&#34;Before&#34;)
idf.describe().where(F.col(&#39;summary&#39;).isin(&#39;count&#39;, &#39;min&#39;, &#39;max&#39;)).show()
print(&#34;After&#34;)
odf.describe().where(F.col(&#39;summary&#39;).isin(&#39;count&#39;, &#39;min&#39;, &#39;max&#39;)).show()
if (print_impact == True) &amp; (method_type == 0):
print(&#34;Before&#34;)
idf.printSchema()
print(&#34;After&#34;)
odf.printSchema()
return odf
```
</details>
</dd>
<dt id="anovos.data_transformer.transformers.imputation_MMM"><code class="name flex">
<span>def <span class="ident">imputation_MMM</span></span>(<span>spark, idf, list_of_cols='missing', drop_cols=[], method_type='median', pre_existing_model=False, model_path='NA', output_mode='replace', stats_missing={}, stats_mode={}, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of columns to impute e.g., ["col1","col2"].</dd>
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
method_type: median", "mean" (valid only for for numerical columns attributes).
Mode is only option for categorical columns. (Default value = "median")
pre_existing_model: Boolean argument – True or False. True if imputation model exists already, False otherwise. (Default value = False)
model_path: If pre_existing_model is True, this argument is path for referring the pre-saved model.
If pre_existing_model is False, this argument can be used for saving the model.
Default "NA" means there is neither pre-existing model nor there is a need to save one.
output_mode: replace", "append".
“replace” option replaces original columns with transformed column. “append” option append transformed
column to the input dataset with a postfix "_imputed" e.g. column X is appended as X_imputed. (Default value = "replace")
stats_missing: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on missing count/pct i.e. if measures_of_counts or
missingCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
stats_mode: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on most frequently seen values i.e. if measures_of_centralTendency or
mode_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Imputed Dataframe</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def imputation_MMM(spark, idf, list_of_cols=&#34;missing&#34;, drop_cols=[], method_type=&#34;median&#34;, pre_existing_model=False,
model_path=&#34;NA&#34;,
output_mode=&#34;replace&#34;, stats_missing={}, stats_mode={}, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of columns to impute e.g., [&#34;col1&#34;,&#34;col2&#34;].
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
method_type: median&#34;, &#34;mean&#34; (valid only for for numerical columns attributes).
Mode is only option for categorical columns. (Default value = &#34;median&#34;)
pre_existing_model: Boolean argument – True or False. True if imputation model exists already, False otherwise. (Default value = False)
model_path: If pre_existing_model is True, this argument is path for referring the pre-saved model.
If pre_existing_model is False, this argument can be used for saving the model.
Default &#34;NA&#34; means there is neither pre-existing model nor there is a need to save one.
output_mode: replace&#34;, &#34;append&#34;.
“replace” option replaces original columns with transformed column. “append” option append transformed
column to the input dataset with a postfix &#34;_imputed&#34; e.g. column X is appended as X_imputed. (Default value = &#34;replace&#34;)
stats_missing: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on missing count/pct i.e. if measures_of_counts or
missingCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
stats_mode: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on most frequently seen values i.e. if measures_of_centralTendency or
mode_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)
Returns:
Imputed Dataframe
&#34;&#34;&#34;
if stats_missing == {}:
missing_df = missingCount_computation(spark, idf)
else:
missing_df = read_dataset(spark, **stats_missing).select(&#39;attribute&#39;, &#39;missing_count&#39;, &#39;missing_pct&#39;)
missing_cols = missing_df.where(F.col(&#39;missing_count&#39;) &gt; 0).select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
if str(pre_existing_model).lower() == &#39;true&#39;:
pre_existing_model = True
elif str(pre_existing_model).lower() == &#39;false&#39;:
pre_existing_model = False
else:
raise TypeError(&#39;Non-Boolean input for pre_existing_model&#39;)
if (len(missing_cols) == 0) &amp; (pre_existing_model == False) &amp; (model_path == &#34;NA&#34;):
return idf
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
warnings.warn(&#34;No Imputation performed- No column(s) to impute&#34;)
return idf
if any(x not in idf.columns for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if method_type not in (&#39;mode&#39;, &#39;mean&#39;, &#39;median&#39;):
raise TypeError(&#39;Invalid input for method_type&#39;)
if output_mode not in (&#39;replace&#39;, &#39;append&#39;):
raise TypeError(&#39;Invalid input for output_mode&#39;)
num_cols, cat_cols, other_cols = attributeType_segregation(idf.select(list_of_cols))
odf = idf
if len(num_cols) &gt; 0:
# Checking for Integer/Decimal Type Columns &amp; Converting them into Float/Double Type
recast_cols = []
recast_type = []
for i in num_cols:
if get_dtype(idf, i) not in (&#39;float&#39;, &#39;double&#39;):
odf = odf.withColumn(i, F.col(i).cast(T.DoubleType()))
recast_cols.append(i + &#34;_imputed&#34;)
recast_type.append(get_dtype(idf, i))
# For mode imputation
if method_type == &#39;mode&#39;:
if stats_mode == {}:
parameters = [str((idf.select(i).dropna().groupby(i).count().orderBy(&#34;count&#34;, ascending=False).first()
or [None])[0]) for i in num_cols]
else:
mode_df = read_dataset(spark, **stats_mode).replace(&#39;None&#39;, None)
mode_df_cols = list(mode_df.select(&#39;attribute&#39;).toPandas()[&#39;attribute&#39;])
parameters = []
for i in num_cols:
if i not in mode_df_cols:
parameters.append(str((idf.select(i).dropna().groupby(i).count().orderBy(&#34;count&#34;, ascending=False).first()
or [None])[0]))
else:
parameters.append(mode_df.where(F.col(&#39;attribute&#39;) == i).select(&#39;mode&#39;).rdd.flatMap(list).collect()[0])
for index, i in enumerate(num_cols):
odf = odf.withColumn(i + &#34;_imputed&#34;, F.when(F.col(i).isNull(), parameters[index]).otherwise(F.col(i)))
else: #For mean, median imputation
# Building new imputer model or uploading the existing model
if pre_existing_model == True:
imputerModel = ImputerModel.load(model_path + &#34;/imputation_MMM/num_imputer-model&#34;)
else:
imputer = Imputer(strategy=method_type, inputCols=num_cols,
outputCols=[(e + &#34;_imputed&#34;) for e in num_cols])
imputerModel = imputer.fit(odf)
# Applying model
# odf = recast_column(imputerModel.transform(odf), recast_cols, recast_type)
odf = imputerModel.transform(odf)
for i, j in zip(recast_cols, recast_type):
odf = odf.withColumn(i, F.col(i).cast(j))
# Saving model if required
if (pre_existing_model == False) &amp; (model_path != &#34;NA&#34;):
imputerModel.write().overwrite().save(model_path + &#34;/imputation_MMM/num_imputer-model&#34;)
if len(cat_cols) &gt; 0:
if pre_existing_model:
df_model = spark.read.csv(model_path + &#34;/imputation_MMM/cat_imputer&#34;, header=True, inferSchema=True)
parameters = []
for i in cat_cols:
mapped_value = \
df_model.where(F.col(&#39;attribute&#39;) == i).select(&#39;parameters&#39;).rdd.flatMap(lambda x: x).collect()[0]
parameters.append(mapped_value)
else:
if stats_mode == {}:
parameters = [str((idf.select(i).dropna().groupby(i).count().orderBy(&#34;count&#34;, ascending=False).first()
or [None])[0]) for i in cat_cols]
else:
mode_df = read_dataset(spark, **stats_mode).replace(&#39;None&#39;, None)
parameters = [mode_df.where(F.col(&#39;attribute&#39;) == i).select(&#39;mode&#39;).rdd.flatMap(list).collect()[0] for i
in cat_cols]
for index, i in enumerate(cat_cols):
odf = odf.withColumn(i + &#34;_imputed&#34;, F.when(F.col(i).isNull(), parameters[index]).otherwise(F.col(i)))
# Saving model File if required
if (pre_existing_model == False) &amp; (model_path != &#34;NA&#34;):
df_model = spark.createDataFrame(zip(cat_cols, parameters), schema=[&#39;attribute&#39;, &#39;parameters&#39;])
df_model.repartition(1).write.csv(model_path + &#34;/imputation_MMM/cat_imputer&#34;, header=True, mode=&#39;overwrite&#39;)
for i in (num_cols + cat_cols):
if i not in missing_cols:
odf = odf.drop(i + &#34;_imputed&#34;)
elif output_mode == &#39;replace&#39;:
odf = odf.drop(i).withColumnRenamed(i + &#34;_imputed&#34;, i)
if print_impact:
if output_mode == &#39;replace&#39;:
odf_print = missing_df.select(&#39;attribute&#39;, F.col(&#34;missing_count&#34;).alias(&#34;missingCount_before&#34;)) \
.join(missingCount_computation(spark, odf, list_of_cols) \
.select(&#39;attribute&#39;, F.col(&#34;missing_count&#34;).alias(&#34;missingCount_after&#34;)), &#39;attribute&#39;, &#39;inner&#39;)
else:
output_cols = [(i + &#34;_imputed&#34;) for i in [e for e in (num_cols + cat_cols) if e in missing_cols]]
odf_print = missing_df.select(&#39;attribute&#39;, F.col(&#34;missing_count&#34;).alias(&#34;missingCount_before&#34;)) \
.join(missingCount_computation(spark, odf, output_cols) \
.withColumnRenamed(&#39;attribute&#39;, &#39;attribute_after&#39;) \
.withColumn(&#39;attribute&#39;, F.expr(&#34;substring(attribute_after, 1, length(attribute_after)-8)&#34;)) \
.drop(&#39;missing_pct&#39;), &#39;attribute&#39;, &#39;inner&#39;)
odf_print.show(len(list_of_cols))
return odf
```
</details>
</dd>
<dt id="anovos.data_transformer.transformers.monotonic_binning"><code class="name flex">
<span>def <span class="ident">monotonic_binning</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], label_col='label', event_label=1, bin_method='equal_range', bin_size=10, bin_dtype='numerical', output_mode='replace')</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of numerical columns to transform e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
label_col: Label/Target column (Default value = 'label')
event_label: Value of (positive) event (i.e label 1) (Default value = 1)
bin_method: equal_frequency", "equal_range".
In "equal_range" method, each bin is of equal size/width and in "equal_frequency", each bin has
equal no. of rows, though the width of bins may vary. (Default value = "equal_range")
bin_size: Default number of bins in case monotonicity is not achieved.
bin_dtype: numerical", "categorical".
With "numerical" option, original value is replaced with an Integer (1,2,…) and
with "categorical" option, original replaced with a string describing min and max value allowed
in the bin ("minval-maxval"). (Default value = "numerical")
output_mode: replace", "append".
“replace” option replaces original columns with transformed column. “append” option append transformed
column to the input dataset with a postfix "_binned" e.g. column X is appended as X_binned. (Default value = "replace")</p>
<h2 id="returns">Returns</h2>
<p>Binned Dataframe</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def monotonic_binning(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], label_col=&#39;label&#39;, event_label=1,
bin_method=&#34;equal_range&#34;, bin_size=10, bin_dtype=&#34;numerical&#34;, output_mode=&#34;replace&#34;):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of numerical columns to transform e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
label_col: Label/Target column (Default value = &#39;label&#39;)
event_label: Value of (positive) event (i.e label 1) (Default value = 1)
bin_method: equal_frequency&#34;, &#34;equal_range&#34;.
In &#34;equal_range&#34; method, each bin is of equal size/width and in &#34;equal_frequency&#34;, each bin has
equal no. of rows, though the width of bins may vary. (Default value = &#34;equal_range&#34;)
bin_size: Default number of bins in case monotonicity is not achieved.
bin_dtype: numerical&#34;, &#34;categorical&#34;.
With &#34;numerical&#34; option, original value is replaced with an Integer (1,2,…) and
with &#34;categorical&#34; option, original replaced with a string describing min and max value allowed
in the bin (&#34;minval-maxval&#34;). (Default value = &#34;numerical&#34;)
output_mode: replace&#34;, &#34;append&#34;.
“replace” option replaces original columns with transformed column. “append” option append transformed
column to the input dataset with a postfix &#34;_binned&#34; e.g. column X is appended as X_binned. (Default value = &#34;replace&#34;)
Returns:
Binned Dataframe
&#34;&#34;&#34;
num_cols = attributeType_segregation(idf)[0]
if list_of_cols == &#39;all&#39;:
list_of_cols = num_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in (drop_cols + [label_col])]))
if any(x not in num_cols for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
attribute_binning(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], method_type=&#34;equal_range&#34;, bin_size=10,
pre_existing_model=False, model_path=&#34;NA&#34;, output_mode=&#34;replace&#34;, print_impact=False)
odf = idf
for col in list_of_cols:
n = 20
r = 0
while n &gt; 2:
tmp = attribute_binning(spark, idf, [col], drop_cols=[], method_type=bin_method, bin_size=n,
output_mode=&#39;append&#39;) \
.select(label_col, col, col + &#39;_binned&#39;) \
.withColumn(label_col, F.when(F.col(label_col) == event_label, 1).otherwise(0)) \
.groupBy(col + &#39;_binned&#39;).agg(F.avg(col).alias(&#39;mean_val&#39;),
F.avg(label_col).alias(&#39;mean_label&#39;)).dropna()
r, p = stats.spearmanr(tmp.toPandas()[[&#39;mean_val&#39;]], tmp.toPandas()[[&#39;mean_label&#39;]])
if r == 1.0:
odf = attribute_binning(spark, odf, [col], drop_cols=[], method_type=bin_method, bin_size=n,
bin_dtype=bin_dtype, output_mode=output_mode)
break
n = n - 1
r = 0
if r &lt; 1.0:
odf = attribute_binning(spark, odf, [col], drop_cols=[], method_type=bin_method, bin_size=bin_size,
bin_dtype=bin_dtype, output_mode=output_mode)
return odf
```
</details>
</dd>
<dt id="anovos.data_transformer.transformers.outlier_categories"><code class="name flex">
<span>def <span class="ident">outlier_categories</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], coverage=1.0, max_category=50, pre_existing_model=False, model_path='NA', output_mode='replace', print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of categorical columns to transform e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all categorical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
coverage: Defines the minimum % of rows that will be mapped to actual category name and the rest to be mapped
to others and takes value between 0 to 1. Coverage of 0.8 can be interpreted as top frequently seen
categories are considered till it covers minimum 80% of rows and rest lesser seen values are mapped to others. (Default value = 1.0)
max_category: Even if coverage is less, only (max_category - 1) categories will be mapped to actual name and rest to others.
Caveat is when multiple categories have same rank, then #categories can be more than max_category. (Default value = 50)
pre_existing_model: Boolean argument – True or False. True if the model with the outlier/other values
for each attribute exists already to be used, False Otherwise. (Default value = False)
model_path: If pre_existing_model is True, this argument is path for the pre-saved model.
If pre_existing_model is False, this field can be used for saving the model.
Default "NA" means there is neither pre-existing model nor there is a need to save one.
output_mode: replace", "append".
“replace” option replaces original columns with transformed column. “append” option append transformed
column to the input dataset with a postfix "_outliered" e.g. column X is appended as X_outliered. (Default value = 'replace')
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe after outlier treatment</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def outlier_categories(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], coverage=1.0, max_category=50,
pre_existing_model=False, model_path=&#34;NA&#34;, output_mode=&#39;replace&#39;, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of categorical columns to transform e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all categorical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
coverage: Defines the minimum % of rows that will be mapped to actual category name and the rest to be mapped
to others and takes value between 0 to 1. Coverage of 0.8 can be interpreted as top frequently seen
categories are considered till it covers minimum 80% of rows and rest lesser seen values are mapped to others. (Default value = 1.0)
max_category: Even if coverage is less, only (max_category - 1) categories will be mapped to actual name and rest to others.
Caveat is when multiple categories have same rank, then #categories can be more than max_category. (Default value = 50)
pre_existing_model: Boolean argument – True or False. True if the model with the outlier/other values
for each attribute exists already to be used, False Otherwise. (Default value = False)
model_path: If pre_existing_model is True, this argument is path for the pre-saved model.
If pre_existing_model is False, this field can be used for saving the model.
Default &#34;NA&#34; means there is neither pre-existing model nor there is a need to save one.
output_mode: replace&#34;, &#34;append&#34;.
“replace” option replaces original columns with transformed column. “append” option append transformed
column to the input dataset with a postfix &#34;_outliered&#34; e.g. column X is appended as X_outliered. (Default value = &#39;replace&#39;)
print_impact:
(Default value = False)
Returns:
Dataframe after outlier treatment
&#34;&#34;&#34;
cat_cols = attributeType_segregation(idf)[1]
if list_of_cols == &#39;all&#39;:
list_of_cols = cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in cat_cols for x in list_of_cols):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if len(list_of_cols) == 0:
warnings.warn(&#34;No Outlier Categories Computation - No categorical column(s) to transform&#34;)
return idf
if (coverage &lt;= 0) | (coverage &gt; 1):
raise TypeError(&#39;Invalid input for Coverage Value&#39;)
if max_category &lt; 2:
raise TypeError(&#39;Invalid input for Maximum No. of Categories Allowed&#39;)
if output_mode not in (&#39;replace&#39;, &#39;append&#39;):
raise TypeError(&#39;Invalid input for output_mode&#39;)
if pre_existing_model == True:
df_model = spark.read.csv(model_path + &#34;/outlier_categories&#34;, header=True, inferSchema=True)
else:
for index, i in enumerate(list_of_cols):
window = Window.partitionBy().orderBy(F.desc(&#39;count_pct&#39;))
df_cats = idf.groupBy(i).count().dropna() \
.withColumn(&#39;count_pct&#39;, F.col(&#39;count&#39;) / F.sum(&#39;count&#39;).over(Window.partitionBy())) \
.withColumn(&#39;rank&#39;, F.rank().over(window)) \
.withColumn(&#39;cumu&#39;, F.sum(&#39;count_pct&#39;).over(window.rowsBetween(Window.unboundedPreceding, 0))) \
.withColumn(&#39;lag_cumu&#39;, F.lag(&#39;cumu&#39;).over(window)).fillna(0) \
.where(~((F.col(&#39;cumu&#39;) &gt;= coverage) &amp; (F.col(&#39;lag_cumu&#39;) &gt;= coverage))) \
.where(F.col(&#39;rank&#39;) &lt;= (max_category - 1)) \
.select(F.lit(i).alias(&#39;attribute&#39;), F.col(i).alias(&#39;parameters&#39;))
if index == 0:
df_model = df_cats
else:
df_model = df_model.union(df_cats)
odf = idf
for i in list_of_cols:
parameters = df_model.where(F.col(&#39;attribute&#39;) == i).select(&#39;parameters&#39;).rdd.flatMap(lambda x: x).collect()
if output_mode == &#39;replace&#39;:
odf = odf.withColumn(i, F.when((F.col(i).isin(parameters)) | (F.col(i).isNull()), F.col(i)).otherwise(
&#34;others&#34;))
else:
odf = odf.withColumn(i + &#34;_outliered&#34;,
F.when((F.col(i).isin(parameters)) | (F.col(i).isNull()), F.col(i)).otherwise(
&#34;others&#34;))
# Saving model File if required
if (pre_existing_model == False) &amp; (model_path != &#34;NA&#34;):
df_model.repartition(1).write.csv(model_path + &#34;/outlier_categories&#34;, header=True, mode=&#39;overwrite&#39;)
if print_impact:
if output_mode == &#39;replace&#39;:
output_cols = list_of_cols
else:
output_cols = [(i + &#34;_outliered&#34;) for i in list_of_cols]
uniqueCount_computation(spark, idf, list_of_cols).select(&#39;attribute&#39;,
F.col(&#34;unique_values&#34;).alias(
&#34;uniqueValues_before&#34;)).show(
len(list_of_cols))
uniqueCount_computation(spark, odf, output_cols).select(&#39;attribute&#39;,
F.col(&#34;unique_values&#34;).alias(
&#34;uniqueValues_after&#34;)).show(
len(list_of_cols))
return odf
```
</details>
</dd>
</dl>