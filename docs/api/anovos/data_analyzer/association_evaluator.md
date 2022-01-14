# Module <code>association_evaluator</code>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
# coding=utf-8
import itertools
import math
import pyspark
from anovos.data_analyzer.stats_generator import uniqueCount_computation
from anovos.data_ingest.data_ingest import read_dataset
from anovos.data_transformer.transformers import attribute_binning, monotonic_binning, cat_to_num_unsupervised, \
imputation_MMM
from anovos.shared.utils import attributeType_segregation
from phik.phik import spark_phik_matrix_from_hist2d_dict
from popmon.analysis.hist_numpy import get_2dgrid
from pyspark.sql import Window
from pyspark.sql import functions as F
from varclushi import VarClusHi
def correlation_matrix(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], stats_unique={}, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)
Returns:
Dataframe [attribute,*col_names]
Correlation between attribute X and Y can be found at an intersection of
a) row with value X in ‘attribute’ column and column ‘Y’, or
b) row with value Y in ‘attribute’ column and column ‘X’.
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
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
if any(x not in idf.columns for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
combis = [list(c) for c in itertools.combinations_with_replacement(list_of_cols, 2)]
hists = idf.select(list_of_cols).pm_make_histograms(combis)
grids = {k: get_2dgrid(h) for k, h in hists.items()}
odf_pd = spark_phik_matrix_from_hist2d_dict(spark.sparkContext, grids)
odf_pd[&#39;attribute&#39;] = odf_pd.index
list_of_cols.sort()
odf = spark.createDataFrame(odf_pd) \
.select([&#39;attribute&#39;] + list_of_cols).orderBy(&#39;attribute&#39;)
if print_impact:
odf.show(odf.count())
return odf
def variable_clustering(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], sample_size=100000, stats_unique={},
stats_mode={},
print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
sample_size: Maximum sample size (in terms of number of rows) taken for the computation.
Sample dataset is extracted using random sampling. (Default value = 100000)
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
stats_mode: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on most frequently seen values i.e. if measures_of_centralTendency or
mode_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)
Returns:
Dataframe [Cluster, Attribute, RS_Ratio]
Attributes similar to each other are grouped together with the same cluster id.
Attribute with the lowest (1 — RS_Ratio) can be chosen as a representative of the cluster.
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
idf_sample = idf.sample(False, min(1.0, float(sample_size) / idf.count()), 0)
idf_sample.persist(pyspark.StorageLevel.MEMORY_AND_DISK).count()
if stats_unique == {}:
remove_cols = uniqueCount_computation(spark, idf_sample, list_of_cols).where(F.col(&#39;unique_values&#39;) &lt; 2) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
else:
remove_cols = read_dataset(spark, **stats_unique).where(F.col(&#39;unique_values&#39;) &lt; 2) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
list_of_cols = [e for e in list_of_cols if e not in remove_cols]
idf_sample = idf_sample.select(list_of_cols)
num_cols, cat_cols, other_cols = attributeType_segregation(idf_sample)
for i in idf_sample.dtypes:
if i[1].startswith(&#39;decimal&#39;):
idf_sample = idf_sample.withColumn(i[0], F.col(i[0]).cast(&#39;double&#39;))
idf_encoded = cat_to_num_unsupervised(spark, idf_sample, list_of_cols=cat_cols, method_type=1)
idf_imputed = imputation_MMM(spark, idf_encoded, stats_mode=stats_mode)
idf_imputed.persist(pyspark.StorageLevel.MEMORY_AND_DISK).count()
idf_sample.unpersist()
idf_pd = idf_imputed.toPandas()
vc = VarClusHi(idf_pd, maxeigval2=1, maxclus=None)
vc.varclus()
odf_pd = vc.rsquare
odf = spark.createDataFrame(odf_pd).select(&#39;Cluster&#39;, F.col(&#39;Variable&#39;).alias(&#39;Attribute&#39;),
F.round(F.col(&#39;RS_Ratio&#39;), 4).alias(&#39;RS_Ratio&#39;))
if print_impact:
odf.show(odf.count())
return odf
def IV_calculation(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], label_col=&#39;label&#39;, event_label=1,
encoding_configs={&#39;bin_method&#39;: &#39;equal_frequency&#39;, &#39;bin_size&#39;: 10, &#39;monotonicity_check&#39;: 0},
print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
label_col: Label/Target column (Default value = &#39;label&#39;)
event_label: Value of (positive) event (i.e label 1) (Default value = 1)
encoding_configs: Takes input in dictionary format. Default {} i.e. empty dict means no encoding is required.
In case numerical columns are present and encoding is required, following keys shall be
provided - &#34;bin_size&#34; i.e. no. of bins for converting the numerical columns to categorical,
&#34;bin_method&#34; i.e. method of binning - &#34;equal_frequency&#34; or &#34;equal_range&#34; and
&#34;monotonicity_check&#34; 1 for monotonic binning else 0. monotonicity_check of 1 will
dynamically calculate the bin_size ensuring monotonic nature but can be expensive operation.
&#39;bin_size&#39;: 10:
&#39;monotonicity_check&#39;: 0}:
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, iv]
&#34;&#34;&#34;
if label_col not in idf.columns:
raise TypeError(&#39;Invalid input for Label Column&#39;)
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in (drop_cols + [label_col])]))
if any(x not in idf.columns for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if (idf.where(F.col(label_col) == event_label).count() == 0):
raise TypeError(&#39;Invalid input for Event Label Value&#39;)
num_cols, cat_cols, other_cols = attributeType_segregation(idf.select(list_of_cols))
if (len(num_cols) &gt; 0) &amp; bool(encoding_configs):
bin_size = encoding_configs[&#39;bin_size&#39;]
bin_method = encoding_configs[&#39;bin_method&#39;]
monotonicity_check = encoding_configs[&#39;monotonicity_check&#39;]
if monotonicity_check == 1:
idf_encoded = monotonic_binning(spark, idf, num_cols, [], label_col, event_label, bin_method, bin_size)
else:
idf_encoded = attribute_binning(spark, idf, num_cols, [], bin_method, bin_size)
idf_encoded.persist(pyspark.StorageLevel.MEMORY_AND_DISK).count()
else:
idf_encoded = idf
output = []
for col in list_of_cols:
df_iv = idf_encoded.groupBy(col, label_col).count() \
.withColumn(label_col, F.when(F.col(label_col) == event_label, 1).otherwise(0)) \
.groupBy(col).pivot(label_col).sum(&#39;count&#39;).fillna(0.5) \
.withColumn(&#39;event_pct&#39;, F.col(&#34;1&#34;) / F.sum(&#34;1&#34;).over(Window.partitionBy())) \
.withColumn(&#39;nonevent_pct&#39;, F.col(&#34;0&#34;) / F.sum(&#34;0&#34;).over(Window.partitionBy())) \
.withColumn(&#39;iv&#39;,
(F.col(&#39;nonevent_pct&#39;) - F.col(&#39;event_pct&#39;)) * F.log(
F.col(&#39;nonevent_pct&#39;) / F.col(&#39;event_pct&#39;)))
iv_value = df_iv.select(F.sum(&#39;iv&#39;)).collect()[0][0]
output.append([col, iv_value])
odf = spark.createDataFrame(output, [&#34;attribute&#34;, &#34;iv&#34;]) \
.withColumn(&#39;iv&#39;, F.round(F.col(&#39;iv&#39;), 4)).orderBy(F.desc(&#39;iv&#39;))
if print_impact:
odf.show(odf.count())
return odf
def IG_calculation(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], label_col=&#39;label&#39;, event_label=1,
encoding_configs={&#39;bin_method&#39;: &#39;equal_frequency&#39;, &#39;bin_size&#39;: 10, &#39;monotonicity_check&#39;: 0},
print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
label_col: Label/Target column (Default value = &#39;label&#39;)
event_label: Value of (positive) event (i.e label 1) (Default value = 1)
encoding_configs: Takes input in dictionary format. Default {} i.e. empty dict means no encoding is required.
In case numerical columns are present and encoding is required, following keys shall be
provided - &#34;bin_size&#34; i.e. no. of bins for converting the numerical columns to categorical,
&#34;bin_method&#34; i.e. method of binning - &#34;equal_frequency&#34; or &#34;equal_range&#34; and
&#34;monotonicity_check&#34; 1 for monotonic binning else 0. monotonicity_check of 1 will
dynamically calculate the bin_size ensuring monotonic nature but can be expensive operation.
&#39;bin_size&#39;: 10:
&#39;monotonicity_check&#39;: 0}:
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, ig]
&#34;&#34;&#34;
if label_col not in idf.columns:
raise TypeError(&#39;Invalid input for Label Column&#39;)
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in (drop_cols + [label_col])]))
if any(x not in idf.columns for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if (idf.where(F.col(label_col) == event_label).count() == 0):
raise TypeError(&#39;Invalid input for Event Label Value&#39;)
num_cols, cat_cols, other_cols = attributeType_segregation(idf.select(list_of_cols))
if (len(num_cols) &gt; 0) &amp; bool(encoding_configs):
bin_size = encoding_configs[&#39;bin_size&#39;]
bin_method = encoding_configs[&#39;bin_method&#39;]
monotonicity_check = encoding_configs[&#39;monotonicity_check&#39;]
if monotonicity_check == 1:
idf_encoded = monotonic_binning(spark, idf, num_cols, [], label_col, event_label, bin_method, bin_size)
else:
idf_encoded = attribute_binning(spark, idf, num_cols, [], bin_method, bin_size)
idf_encoded.persist(pyspark.StorageLevel.MEMORY_AND_DISK).count()
else:
idf_encoded = idf
output = []
total_event = idf.where(F.col(label_col) == event_label).count() / idf.count()
total_entropy = - (total_event * math.log2(total_event) + ((1 - total_event) * math.log2((1 - total_event))))
for col in list_of_cols:
idf_entropy = idf_encoded.withColumn(label_col, F.when(F.col(label_col) == event_label, 1).otherwise(0)) \
.groupBy(col).agg(F.sum(F.col(label_col)).alias(&#39;event_count&#39;),
F.count(F.col(label_col)).alias(&#39;total_count&#39;)).dropna() \
.withColumn(&#39;event_pct&#39;, F.col(&#39;event_count&#39;) / F.col(&#39;total_count&#39;)) \
.withColumn(&#39;segment_pct&#39;, F.col(&#39;total_count&#39;) / F.sum(&#39;total_count&#39;).over(Window.partitionBy())) \
.withColumn(&#39;entropy&#39;, - F.col(&#39;segment_pct&#39;) * ((F.col(&#39;event_pct&#39;) * F.log2(F.col(&#39;event_pct&#39;))) +
((1 - F.col(&#39;event_pct&#39;)) * F.log2(
(1 - F.col(&#39;event_pct&#39;))))))
entropy = idf_entropy.groupBy().sum(&#39;entropy&#39;).rdd.flatMap(lambda x: x).collect()[0]
ig_value = total_entropy - entropy if entropy else None
output.append([col, ig_value])
odf = spark.createDataFrame(output, [&#34;attribute&#34;, &#34;ig&#34;]) \
.withColumn(&#39;ig&#39;, F.round(F.col(&#39;ig&#39;), 4)).orderBy(F.desc(&#39;ig&#39;))
if print_impact:
odf.show(odf.count())
return odf
```
</details>
## Functions
<dl>
<dt id="anovos.data_analyzer.association_evaluator.IG_calculation"><code class="name flex">
<span>def <span class="ident">IG_calculation</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], label_col='label', event_label=1, encoding_configs={'bin_method': 'equal_frequency', 'bin_size': 10, 'monotonicity_check': 0}, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of columns to analyse e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
label_col: Label/Target column (Default value = 'label')
event_label: Value of (positive) event (i.e label 1) (Default value = 1)
encoding_configs: Takes input in dictionary format. Default {} i.e. empty dict means no encoding is required.
In case numerical columns are present and encoding is required, following keys shall be
provided - "bin_size" i.e. no. of bins for converting the numerical columns to categorical,
"bin_method" i.e. method of binning - "equal_frequency" or "equal_range" and
"monotonicity_check" 1 for monotonic binning else 0. monotonicity_check of 1 will
dynamically calculate the bin_size ensuring monotonic nature but can be expensive operation.
'bin_size': 10:
'monotonicity_check': 0}:
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute, ig]</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def IG_calculation(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], label_col=&#39;label&#39;, event_label=1,
encoding_configs={&#39;bin_method&#39;: &#39;equal_frequency&#39;, &#39;bin_size&#39;: 10, &#39;monotonicity_check&#39;: 0},
print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
label_col: Label/Target column (Default value = &#39;label&#39;)
event_label: Value of (positive) event (i.e label 1) (Default value = 1)
encoding_configs: Takes input in dictionary format. Default {} i.e. empty dict means no encoding is required.
In case numerical columns are present and encoding is required, following keys shall be
provided - &#34;bin_size&#34; i.e. no. of bins for converting the numerical columns to categorical,
&#34;bin_method&#34; i.e. method of binning - &#34;equal_frequency&#34; or &#34;equal_range&#34; and
&#34;monotonicity_check&#34; 1 for monotonic binning else 0. monotonicity_check of 1 will
dynamically calculate the bin_size ensuring monotonic nature but can be expensive operation.
&#39;bin_size&#39;: 10:
&#39;monotonicity_check&#39;: 0}:
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, ig]
&#34;&#34;&#34;
if label_col not in idf.columns:
raise TypeError(&#39;Invalid input for Label Column&#39;)
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in (drop_cols + [label_col])]))
if any(x not in idf.columns for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if (idf.where(F.col(label_col) == event_label).count() == 0):
raise TypeError(&#39;Invalid input for Event Label Value&#39;)
num_cols, cat_cols, other_cols = attributeType_segregation(idf.select(list_of_cols))
if (len(num_cols) &gt; 0) &amp; bool(encoding_configs):
bin_size = encoding_configs[&#39;bin_size&#39;]
bin_method = encoding_configs[&#39;bin_method&#39;]
monotonicity_check = encoding_configs[&#39;monotonicity_check&#39;]
if monotonicity_check == 1:
idf_encoded = monotonic_binning(spark, idf, num_cols, [], label_col, event_label, bin_method, bin_size)
else:
idf_encoded = attribute_binning(spark, idf, num_cols, [], bin_method, bin_size)
idf_encoded.persist(pyspark.StorageLevel.MEMORY_AND_DISK).count()
else:
idf_encoded = idf
output = []
total_event = idf.where(F.col(label_col) == event_label).count() / idf.count()
total_entropy = - (total_event * math.log2(total_event) + ((1 - total_event) * math.log2((1 - total_event))))
for col in list_of_cols:
idf_entropy = idf_encoded.withColumn(label_col, F.when(F.col(label_col) == event_label, 1).otherwise(0)) \
.groupBy(col).agg(F.sum(F.col(label_col)).alias(&#39;event_count&#39;),
F.count(F.col(label_col)).alias(&#39;total_count&#39;)).dropna() \
.withColumn(&#39;event_pct&#39;, F.col(&#39;event_count&#39;) / F.col(&#39;total_count&#39;)) \
.withColumn(&#39;segment_pct&#39;, F.col(&#39;total_count&#39;) / F.sum(&#39;total_count&#39;).over(Window.partitionBy())) \
.withColumn(&#39;entropy&#39;, - F.col(&#39;segment_pct&#39;) * ((F.col(&#39;event_pct&#39;) * F.log2(F.col(&#39;event_pct&#39;))) +
((1 - F.col(&#39;event_pct&#39;)) * F.log2(
(1 - F.col(&#39;event_pct&#39;))))))
entropy = idf_entropy.groupBy().sum(&#39;entropy&#39;).rdd.flatMap(lambda x: x).collect()[0]
ig_value = total_entropy - entropy if entropy else None
output.append([col, ig_value])
odf = spark.createDataFrame(output, [&#34;attribute&#34;, &#34;ig&#34;]) \
.withColumn(&#39;ig&#39;, F.round(F.col(&#39;ig&#39;), 4)).orderBy(F.desc(&#39;ig&#39;))
if print_impact:
odf.show(odf.count())
return odf
```
</details>
</dd>
<dt id="anovos.data_analyzer.association_evaluator.IV_calculation"><code class="name flex">
<span>def <span class="ident">IV_calculation</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], label_col='label', event_label=1, encoding_configs={'bin_method': 'equal_frequency', 'bin_size': 10, 'monotonicity_check': 0}, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of columns to analyse e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
label_col: Label/Target column (Default value = 'label')
event_label: Value of (positive) event (i.e label 1) (Default value = 1)
encoding_configs: Takes input in dictionary format. Default {} i.e. empty dict means no encoding is required.
In case numerical columns are present and encoding is required, following keys shall be
provided - "bin_size" i.e. no. of bins for converting the numerical columns to categorical,
"bin_method" i.e. method of binning - "equal_frequency" or "equal_range" and
"monotonicity_check" 1 for monotonic binning else 0. monotonicity_check of 1 will
dynamically calculate the bin_size ensuring monotonic nature but can be expensive operation.
'bin_size': 10:
'monotonicity_check': 0}:
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute, iv]</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def IV_calculation(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], label_col=&#39;label&#39;, event_label=1,
encoding_configs={&#39;bin_method&#39;: &#39;equal_frequency&#39;, &#39;bin_size&#39;: 10, &#39;monotonicity_check&#39;: 0},
print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
label_col: Label/Target column (Default value = &#39;label&#39;)
event_label: Value of (positive) event (i.e label 1) (Default value = 1)
encoding_configs: Takes input in dictionary format. Default {} i.e. empty dict means no encoding is required.
In case numerical columns are present and encoding is required, following keys shall be
provided - &#34;bin_size&#34; i.e. no. of bins for converting the numerical columns to categorical,
&#34;bin_method&#34; i.e. method of binning - &#34;equal_frequency&#34; or &#34;equal_range&#34; and
&#34;monotonicity_check&#34; 1 for monotonic binning else 0. monotonicity_check of 1 will
dynamically calculate the bin_size ensuring monotonic nature but can be expensive operation.
&#39;bin_size&#39;: 10:
&#39;monotonicity_check&#39;: 0}:
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, iv]
&#34;&#34;&#34;
if label_col not in idf.columns:
raise TypeError(&#39;Invalid input for Label Column&#39;)
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in (drop_cols + [label_col])]))
if any(x not in idf.columns for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if (idf.where(F.col(label_col) == event_label).count() == 0):
raise TypeError(&#39;Invalid input for Event Label Value&#39;)
num_cols, cat_cols, other_cols = attributeType_segregation(idf.select(list_of_cols))
if (len(num_cols) &gt; 0) &amp; bool(encoding_configs):
bin_size = encoding_configs[&#39;bin_size&#39;]
bin_method = encoding_configs[&#39;bin_method&#39;]
monotonicity_check = encoding_configs[&#39;monotonicity_check&#39;]
if monotonicity_check == 1:
idf_encoded = monotonic_binning(spark, idf, num_cols, [], label_col, event_label, bin_method, bin_size)
else:
idf_encoded = attribute_binning(spark, idf, num_cols, [], bin_method, bin_size)
idf_encoded.persist(pyspark.StorageLevel.MEMORY_AND_DISK).count()
else:
idf_encoded = idf
output = []
for col in list_of_cols:
df_iv = idf_encoded.groupBy(col, label_col).count() \
.withColumn(label_col, F.when(F.col(label_col) == event_label, 1).otherwise(0)) \
.groupBy(col).pivot(label_col).sum(&#39;count&#39;).fillna(0.5) \
.withColumn(&#39;event_pct&#39;, F.col(&#34;1&#34;) / F.sum(&#34;1&#34;).over(Window.partitionBy())) \
.withColumn(&#39;nonevent_pct&#39;, F.col(&#34;0&#34;) / F.sum(&#34;0&#34;).over(Window.partitionBy())) \
.withColumn(&#39;iv&#39;,
(F.col(&#39;nonevent_pct&#39;) - F.col(&#39;event_pct&#39;)) * F.log(
F.col(&#39;nonevent_pct&#39;) / F.col(&#39;event_pct&#39;)))
iv_value = df_iv.select(F.sum(&#39;iv&#39;)).collect()[0][0]
output.append([col, iv_value])
odf = spark.createDataFrame(output, [&#34;attribute&#34;, &#34;iv&#34;]) \
.withColumn(&#39;iv&#39;, F.round(F.col(&#39;iv&#39;), 4)).orderBy(F.desc(&#39;iv&#39;))
if print_impact:
odf.show(odf.count())
return odf
```
</details>
</dd>
<dt id="anovos.data_analyzer.association_evaluator.correlation_matrix"><code class="name flex">
<span>def <span class="ident">correlation_matrix</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], stats_unique={}, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of columns to analyse e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute,*col_names]
Correlation between attribute X and Y can be found at an intersection of
a) row with value X in ‘attribute’ column and column ‘Y’, or
b) row with value Y in ‘attribute’ column and column ‘X’.</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def correlation_matrix(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], stats_unique={}, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)
Returns:
Dataframe [attribute,*col_names]
Correlation between attribute X and Y can be found at an intersection of
a) row with value X in ‘attribute’ column and column ‘Y’, or
b) row with value Y in ‘attribute’ column and column ‘X’.
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
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
if any(x not in idf.columns for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
combis = [list(c) for c in itertools.combinations_with_replacement(list_of_cols, 2)]
hists = idf.select(list_of_cols).pm_make_histograms(combis)
grids = {k: get_2dgrid(h) for k, h in hists.items()}
odf_pd = spark_phik_matrix_from_hist2d_dict(spark.sparkContext, grids)
odf_pd[&#39;attribute&#39;] = odf_pd.index
list_of_cols.sort()
odf = spark.createDataFrame(odf_pd) \
.select([&#39;attribute&#39;] + list_of_cols).orderBy(&#39;attribute&#39;)
if print_impact:
odf.show(odf.count())
return odf
```
</details>
</dd>
<dt id="anovos.data_analyzer.association_evaluator.variable_clustering"><code class="name flex">
<span>def <span class="ident">variable_clustering</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], sample_size=100000, stats_unique={}, stats_mode={}, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of columns to analyse e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
sample_size: Maximum sample size (in terms of number of rows) taken for the computation.
Sample dataset is extracted using random sampling. (Default value = 100000)
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
stats_mode: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on most frequently seen values i.e. if measures_of_centralTendency or
mode_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe [Cluster, Attribute, RS_Ratio]
Attributes similar to each other are grouped together with the same cluster id.
Attribute with the lowest (1 — RS_Ratio) can be chosen as a representative of the cluster.</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def variable_clustering(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], sample_size=100000, stats_unique={},
stats_mode={},
print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
sample_size: Maximum sample size (in terms of number of rows) taken for the computation.
Sample dataset is extracted using random sampling. (Default value = 100000)
stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
stats_mode: Takes arguments for read_dataset (data_ingest module) function in a dictionary format
to read pre-saved statistics on most frequently seen values i.e. if measures_of_centralTendency or
mode_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
print_impact:
(Default value = False)
Returns:
Dataframe [Cluster, Attribute, RS_Ratio]
Attributes similar to each other are grouped together with the same cluster id.
Attribute with the lowest (1 — RS_Ratio) can be chosen as a representative of the cluster.
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
idf_sample = idf.sample(False, min(1.0, float(sample_size) / idf.count()), 0)
idf_sample.persist(pyspark.StorageLevel.MEMORY_AND_DISK).count()
if stats_unique == {}:
remove_cols = uniqueCount_computation(spark, idf_sample, list_of_cols).where(F.col(&#39;unique_values&#39;) &lt; 2) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
else:
remove_cols = read_dataset(spark, **stats_unique).where(F.col(&#39;unique_values&#39;) &lt; 2) \
.select(&#39;attribute&#39;).rdd.flatMap(lambda x: x).collect()
list_of_cols = [e for e in list_of_cols if e not in remove_cols]
idf_sample = idf_sample.select(list_of_cols)
num_cols, cat_cols, other_cols = attributeType_segregation(idf_sample)
for i in idf_sample.dtypes:
if i[1].startswith(&#39;decimal&#39;):
idf_sample = idf_sample.withColumn(i[0], F.col(i[0]).cast(&#39;double&#39;))
idf_encoded = cat_to_num_unsupervised(spark, idf_sample, list_of_cols=cat_cols, method_type=1)
idf_imputed = imputation_MMM(spark, idf_encoded, stats_mode=stats_mode)
idf_imputed.persist(pyspark.StorageLevel.MEMORY_AND_DISK).count()
idf_sample.unpersist()
idf_pd = idf_imputed.toPandas()
vc = VarClusHi(idf_pd, maxeigval2=1, maxclus=None)
vc.varclus()
odf_pd = vc.rsquare
odf = spark.createDataFrame(odf_pd).select(&#39;Cluster&#39;, F.col(&#39;Variable&#39;).alias(&#39;Attribute&#39;),
F.round(F.col(&#39;RS_Ratio&#39;), 4).alias(&#39;RS_Ratio&#39;))
if print_impact:
odf.show(odf.count())
return odf
```
</details>
</dd>
</dl>