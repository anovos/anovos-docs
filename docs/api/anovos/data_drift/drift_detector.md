# Module <code>drift_detector</code>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
# coding=utf-8
from __future__ import division, print_function
import math
import numpy as np
import pandas as pd
import pyspark
from anovos.data_ingest.data_ingest import concatenate_dataset
from anovos.data_transformer.transformers import attribute_binning
from anovos.shared.utils import attributeType_segregation
from pyspark.sql import functions as F
from pyspark.sql import types as T
from scipy.stats import variation
def drift_statistics(spark, idf_target, idf_source, list_of_cols=&#39;all&#39;, drop_cols=[], method_type=&#39;PSI&#39;,
bin_method=&#39;equal_range&#39;,
bin_size=10, threshold=0.1, pre_existing_source=False, source_path=&#34;NA&#34;, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf_target: Input Dataframe
idf_source: Baseline/Source Dataframe. This argument is ignored if pre_existing_source is True.
list_of_cols: List of columns to check drift e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all (non-array) columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
method: PSI&#34;, &#34;JSD&#34;, &#34;HD&#34;, &#34;KS&#34;,&#34;all&#34;.
&#34;all&#34; can be passed to calculate all drift metrics.
One or more methods can be passed in a form of list or string where different metrics are separated
by pipe delimiter “|” e.g. [&#34;PSI&#34;, &#34;JSD&#34;] or &#34;PSI|JSD&#34;
bin_method: equal_frequency&#34;, &#34;equal_range&#34;.
In &#34;equal_range&#34; method, each bin is of equal size/width and in &#34;equal_frequency&#34;, each bin
has equal no. of rows, though the width of bins may vary. (Default value = &#39;equal_range&#39;)
bin_size: Number of bins for creating histogram (Default value = 10)
threshold: A column is flagged if any drift metric is above the threshold. (Default value = 0.1)
pre_existing_source: Boolean argument – True or False. True if the drift_statistics folder (binning model &amp;
frequency counts for each attribute) exists already, False Otherwise. (Default value = False)
source_path: If pre_existing_source is False, this argument can be used for saving the drift_statistics folder.
The drift_statistics folder will have attribute_binning (binning model) &amp; frequency_counts sub-folders.
If pre_existing_source is True, this argument is path for referring the drift_statistics folder.
Default &#34;NA&#34; for temporarily saving source dataset attribute_binning folder.
method_type:
(Default value = &#39;PSI&#39;)
print_impact:
(Default value = False)
Returns:
Output Dataframe [attribute, *metric, flagged]
Number of columns will be dependent on method argument. There will be one column for each drift method/metric.
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf_target)
list_of_cols = num_cols + cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in idf_target.columns for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if method_type == &#39;all&#39;:
method_type = [&#39;PSI&#39;, &#39;JSD&#39;, &#39;HD&#39;, &#39;KS&#39;]
if isinstance(method_type, str):
method_type = [x.strip() for x in method_type.split(&#39;|&#39;)]
if any(x not in (&#34;PSI&#34;, &#34;JSD&#34;, &#34;HD&#34;, &#34;KS&#34;) for x in method_type):
raise TypeError(&#39;Invalid input for method_type&#39;)
num_cols = attributeType_segregation(idf_target.select(list_of_cols))[0]
if not pre_existing_source:
source_bin = attribute_binning(spark, idf_source, list_of_cols=num_cols, method_type=bin_method,
bin_size=bin_size,
pre_existing_model=False, model_path=source_path + &#34;/drift_statistics&#34;)
source_bin.persist(pyspark.StorageLevel.MEMORY_AND_DISK).count()
target_bin = attribute_binning(spark, idf_target, list_of_cols=num_cols, method_type=bin_method, bin_size=bin_size,
pre_existing_model=True, model_path=source_path + &#34;/drift_statistics&#34;)
target_bin.persist(pyspark.StorageLevel.MEMORY_AND_DISK).count()
def hellinger_distance(p, q):
&#34;&#34;&#34;
Args:
p:
q:
Returns:
&#34;&#34;&#34;
hd = math.sqrt(np.sum((np.sqrt(p) - np.sqrt(q)) ** 2) / 2)
return hd
def PSI(p, q):
&#34;&#34;&#34;
Args:
p:
q:
Returns:
&#34;&#34;&#34;
psi = np.sum((p - q) * np.log(p / q))
return psi
def JS_divergence(p, q):
&#34;&#34;&#34;
Args:
p:
q:
Returns:
&#34;&#34;&#34;
def KL_divergence(p, q):
&#34;&#34;&#34;
Args:
p:
q:
Returns:
&#34;&#34;&#34;
kl = np.sum(p * np.log(p / q))
return kl
m = (p + q) / 2
pm = KL_divergence(p, m)
qm = KL_divergence(q, m)
jsd = (pm + qm) / 2
return jsd
def KS_distance(p, q):
&#34;&#34;&#34;
Args:
p:
q:
Returns:
&#34;&#34;&#34;
dstats = np.max(np.abs(np.cumsum(p) - np.cumsum(q)))
return dstats
output = {&#39;attribute&#39;: []}
output[&#34;flagged&#34;] = []
for method in method_type:
output[method] = []
for i in list_of_cols:
if pre_existing_source:
x = spark.read.csv(source_path + &#34;/drift_statistics/frequency_counts/&#34; + i, header=True, inferSchema=True)
else:
x = source_bin.groupBy(i).agg((F.count(i) / idf_source.count()).alias(&#39;p&#39;)).fillna(-1)
x.coalesce(1).write.csv(source_path + &#34;/drift_statistics/frequency_counts/&#34; + i, header=True,
mode=&#39;overwrite&#39;)
y = target_bin.groupBy(i).agg((F.count(i) / idf_target.count()).alias(&#39;q&#39;)).fillna(-1)
xy = x.join(y, i, &#39;full_outer&#39;).fillna(0.0001, subset=[&#39;p&#39;, &#39;q&#39;]).replace(0, 0.0001).orderBy(i)
p = np.array(xy.select(&#39;p&#39;).rdd.flatMap(lambda x: x).collect())
q = np.array(xy.select(&#39;q&#39;).rdd.flatMap(lambda x: x).collect())
output[&#39;attribute&#39;].append(i)
counter = 0
for idx, method in enumerate(method_type):
drift_function = {&#39;PSI&#39;: PSI, &#39;JSD&#39;: JS_divergence, &#39;HD&#39;: hellinger_distance, &#39;KS&#39;: KS_distance}
metric = float(round(drift_function[method](p, q), 4))
output[method].append(metric)
if counter == 0:
if metric &gt; threshold:
output[&#34;flagged&#34;].append(1)
counter = 1
if (idx == (len(method_type) - 1)) &amp; (counter == 0):
output[&#34;flagged&#34;].append(0)
odf = spark.createDataFrame(pd.DataFrame.from_dict(output, orient=&#39;index&#39;).transpose()) \
.select([&#39;attribute&#39;] + method_type + [&#39;flagged&#39;]).orderBy(F.desc(&#39;flagged&#39;))
if print_impact:
print(&#34;All Attributes:&#34;)
odf.show(len(list_of_cols))
print(&#34;Attributes meeting Data Drift threshold:&#34;)
drift = odf.where(F.col(&#39;flagged&#39;) == 1)
drift.show(drift.count())
return odf
def stabilityIndex_computation(spark, *idfs, list_of_cols=&#39;all&#39;, drop_cols=[],
metric_weightages={&#39;mean&#39;: 0.5, &#39;stddev&#39;: 0.3, &#39;kurtosis&#39;: 0.2},
existing_metric_path=&#39;&#39;, appended_metric_path=&#39;&#39;, threshold=1, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idfs: Variable number of input dataframes
list_of_cols: List of numerical columns to check stability e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
metric_weightages: Takes input in dictionary format with keys being the metric name - &#34;mean&#34;,&#34;stdev&#34;,&#34;kurtosis&#34;
and value being the weightage of the metric (between 0 and 1). Sum of all weightages must be 1. (Default value = {&#39;mean&#39;: 0.5)
existing_metric_path: This argument is path for referring pre-existing metrics of historical datasets and is
of schema [idx, attribute, mean, stdev, kurtosis].
idx is index number of historical datasets assigned in chronological order. (Default value = &#39;&#39;)
appended_metric_path: This argument is path for saving input dataframes metrics after appending to the
historical datasets&#39; metrics. (Default value = &#39;&#39;)
threshold: A column is flagged if the stability index is below the threshold, which varies between 0 to 4. (Default value = 1)
*idfs:
&#39;stddev&#39;: 0.3:
&#39;kurtosis&#39;: 0.2}:
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, mean_si, stddev_si, kurtosis_si, mean_cv, stddev_cv, kurtosis_cv, stability_index].
*_cv is coefficient of variation for each metric. *_si is stability index for each metric.
stability_index is net weighted stability index based on the individual metrics&#39; stability index.
&#34;&#34;&#34;
num_cols = attributeType_segregation(idfs[0])[0]
if list_of_cols == &#39;all&#39;:
list_of_cols = num_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in num_cols for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if round(metric_weightages.get(&#39;mean&#39;, 0) + metric_weightages.get(&#39;stddev&#39;, 0) + metric_weightages.get(&#39;kurtosis&#39;,
0), 3) != 1:
raise ValueError(
&#39;Invalid input for metric weightages. Either metric name is incorrect or sum of metric weightages is not 1.0&#39;)
if existing_metric_path:
existing_metric_df = spark.read.csv(existing_metric_path, header=True, inferSchema=True)
dfs_count = existing_metric_df.select(F.max(F.col(&#39;idx&#39;))).first()[0]
else:
schema = T.StructType([T.StructField(&#39;idx&#39;, T.IntegerType(), True),
T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;mean&#39;, T.DoubleType(), True),
T.StructField(&#39;stddev&#39;, T.DoubleType(), True),
T.StructField(&#39;kurtosis&#39;, T.DoubleType(), True)])
existing_metric_df = spark.sparkContext.emptyRDD().toDF(schema)
dfs_count = 0
metric_ls = []
for idf in idfs:
for i in list_of_cols:
mean, stddev, kurtosis = idf.select(F.mean(i), F.stddev(i), F.kurtosis(i)).first()
metric_ls.append([dfs_count + 1, i, mean, stddev, kurtosis + 3.0 if kurtosis else None])
dfs_count += 1
new_metric_df = spark.createDataFrame(metric_ls, schema=(&#39;idx&#39;, &#39;attribute&#39;, &#39;mean&#39;, &#39;stddev&#39;, &#39;kurtosis&#39;))
appended_metric_df = concatenate_dataset(existing_metric_df, new_metric_df)
if appended_metric_path:
appended_metric_df.coalesce(1).write.csv(appended_metric_path, header=True, mode=&#39;overwrite&#39;)
output = []
for i in list_of_cols:
i_output = [i]
for metric in [&#39;mean&#39;, &#39;stddev&#39;, &#39;kurtosis&#39;]:
metric_stats = appended_metric_df.where(F.col(&#39;attribute&#39;) == i).orderBy(&#39;idx&#39;) \
.select(metric).fillna(np.nan).rdd.flatMap(list).collect()
metric_cv = round(float(variation([a for a in metric_stats])), 4) or None
i_output.append(metric_cv)
output.append(i_output)
schema = T.StructType([T.StructField(&#34;attribute&#34;, T.StringType(), True),
T.StructField(&#34;mean_cv&#34;, T.FloatType(), True),
T.StructField(&#34;stddev_cv&#34;, T.FloatType(), True),
T.StructField(&#34;kurtosis_cv&#34;, T.FloatType(), True)])
odf = spark.createDataFrame(output, schema=schema)
def score_cv(cv, thresholds=[0.03, 0.1, 0.2, 0.5]):
&#34;&#34;&#34;
Args:
cv:
thresholds:
(Default value = [0.03)
0.1:
0.2:
0.5]:
Returns:
&#34;&#34;&#34;
if cv is None:
return None
else:
cv = abs(cv)
stability_index = [4, 3, 2, 1, 0]
for i, thresh in enumerate(thresholds):
if cv &lt; thresh:
return stability_index[i]
return stability_index[-1]
f_score_cv = F.udf(score_cv, T.IntegerType())
odf = odf.replace(np.nan, None).withColumn(&#39;mean_si&#39;, f_score_cv(F.col(&#39;mean_cv&#39;))) \
.withColumn(&#39;stddev_si&#39;, f_score_cv(F.col(&#39;stddev_cv&#39;))) \
.withColumn(&#39;kurtosis_si&#39;, f_score_cv(F.col(&#39;kurtosis_cv&#39;))) \
.withColumn(&#39;stability_index&#39;, F.round((F.col(&#39;mean_si&#39;) * metric_weightages.get(&#39;mean&#39;, 0) +
F.col(&#39;stddev_si&#39;) * metric_weightages.get(&#39;stddev&#39;, 0) +
F.col(&#39;kurtosis_si&#39;) * metric_weightages.get(&#39;kurtosis&#39;, 0)), 4)) \
.withColumn(&#39;flagged&#39;,
F.when((F.col(&#39;stability_index&#39;) &lt; threshold) | (F.col(&#39;stability_index&#39;).isNull()), 1).otherwise(
0))
if print_impact:
print(&#34;All Attributes:&#34;)
odf.show(len(list_of_cols))
print(&#34;Potential Unstable Attributes:&#34;)
unstable = odf.where(F.col(&#39;flagged&#39;) == 1)
unstable.show(unstable.count())
return odf
```
</details>
## Functions
<dl>
<dt id="anovos.data_drift.drift_detector.drift_statistics"><code class="name flex">
<span>def <span class="ident">drift_statistics</span></span>(<span>spark, idf_target, idf_source, list_of_cols='all', drop_cols=[], method_type='PSI', bin_method='equal_range', bin_size=10, threshold=0.1, pre_existing_source=False, source_path='NA', print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf_target</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>idf_source</code></strong></dt>
<dd>Baseline/Source Dataframe. This argument is ignored if pre_existing_source is True.</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of columns to check drift e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all (non-array) columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
method: PSI", "JSD", "HD", "KS","all".
"all" can be passed to calculate all drift metrics.
One or more methods can be passed in a form of list or string where different metrics are separated
by pipe delimiter “|” e.g. ["PSI", "JSD"] or "PSI|JSD"
bin_method: equal_frequency", "equal_range".
In "equal_range" method, each bin is of equal size/width and in "equal_frequency", each bin
has equal no. of rows, though the width of bins may vary. (Default value = 'equal_range')
bin_size: Number of bins for creating histogram (Default value = 10)
threshold: A column is flagged if any drift metric is above the threshold. (Default value = 0.1)
pre_existing_source: Boolean argument – True or False. True if the drift_statistics folder (binning model &amp;
frequency counts for each attribute) exists already, False Otherwise. (Default value = False)
source_path: If pre_existing_source is False, this argument can be used for saving the drift_statistics folder.
The drift_statistics folder will have attribute_binning (binning model) &amp; frequency_counts sub-folders.
If pre_existing_source is True, this argument is path for referring the drift_statistics folder.
Default "NA" for temporarily saving source dataset attribute_binning folder.
method_type:
(Default value = 'PSI')
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Output Dataframe [attribute, *metric, flagged]
Number of columns will be dependent on method argument. There will be one column for each drift method/metric.</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def drift_statistics(spark, idf_target, idf_source, list_of_cols=&#39;all&#39;, drop_cols=[], method_type=&#39;PSI&#39;,
bin_method=&#39;equal_range&#39;,
bin_size=10, threshold=0.1, pre_existing_source=False, source_path=&#34;NA&#34;, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf_target: Input Dataframe
idf_source: Baseline/Source Dataframe. This argument is ignored if pre_existing_source is True.
list_of_cols: List of columns to check drift e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all (non-array) columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
method: PSI&#34;, &#34;JSD&#34;, &#34;HD&#34;, &#34;KS&#34;,&#34;all&#34;.
&#34;all&#34; can be passed to calculate all drift metrics.
One or more methods can be passed in a form of list or string where different metrics are separated
by pipe delimiter “|” e.g. [&#34;PSI&#34;, &#34;JSD&#34;] or &#34;PSI|JSD&#34;
bin_method: equal_frequency&#34;, &#34;equal_range&#34;.
In &#34;equal_range&#34; method, each bin is of equal size/width and in &#34;equal_frequency&#34;, each bin
has equal no. of rows, though the width of bins may vary. (Default value = &#39;equal_range&#39;)
bin_size: Number of bins for creating histogram (Default value = 10)
threshold: A column is flagged if any drift metric is above the threshold. (Default value = 0.1)
pre_existing_source: Boolean argument – True or False. True if the drift_statistics folder (binning model &amp;
frequency counts for each attribute) exists already, False Otherwise. (Default value = False)
source_path: If pre_existing_source is False, this argument can be used for saving the drift_statistics folder.
The drift_statistics folder will have attribute_binning (binning model) &amp; frequency_counts sub-folders.
If pre_existing_source is True, this argument is path for referring the drift_statistics folder.
Default &#34;NA&#34; for temporarily saving source dataset attribute_binning folder.
method_type:
(Default value = &#39;PSI&#39;)
print_impact:
(Default value = False)
Returns:
Output Dataframe [attribute, *metric, flagged]
Number of columns will be dependent on method argument. There will be one column for each drift method/metric.
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf_target)
list_of_cols = num_cols + cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in idf_target.columns for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if method_type == &#39;all&#39;:
method_type = [&#39;PSI&#39;, &#39;JSD&#39;, &#39;HD&#39;, &#39;KS&#39;]
if isinstance(method_type, str):
method_type = [x.strip() for x in method_type.split(&#39;|&#39;)]
if any(x not in (&#34;PSI&#34;, &#34;JSD&#34;, &#34;HD&#34;, &#34;KS&#34;) for x in method_type):
raise TypeError(&#39;Invalid input for method_type&#39;)
num_cols = attributeType_segregation(idf_target.select(list_of_cols))[0]
if not pre_existing_source:
source_bin = attribute_binning(spark, idf_source, list_of_cols=num_cols, method_type=bin_method,
bin_size=bin_size,
pre_existing_model=False, model_path=source_path + &#34;/drift_statistics&#34;)
source_bin.persist(pyspark.StorageLevel.MEMORY_AND_DISK).count()
target_bin = attribute_binning(spark, idf_target, list_of_cols=num_cols, method_type=bin_method, bin_size=bin_size,
pre_existing_model=True, model_path=source_path + &#34;/drift_statistics&#34;)
target_bin.persist(pyspark.StorageLevel.MEMORY_AND_DISK).count()
def hellinger_distance(p, q):
&#34;&#34;&#34;
Args:
p:
q:
Returns:
&#34;&#34;&#34;
hd = math.sqrt(np.sum((np.sqrt(p) - np.sqrt(q)) ** 2) / 2)
return hd
def PSI(p, q):
&#34;&#34;&#34;
Args:
p:
q:
Returns:
&#34;&#34;&#34;
psi = np.sum((p - q) * np.log(p / q))
return psi
def JS_divergence(p, q):
&#34;&#34;&#34;
Args:
p:
q:
Returns:
&#34;&#34;&#34;
def KL_divergence(p, q):
&#34;&#34;&#34;
Args:
p:
q:
Returns:
&#34;&#34;&#34;
kl = np.sum(p * np.log(p / q))
return kl
m = (p + q) / 2
pm = KL_divergence(p, m)
qm = KL_divergence(q, m)
jsd = (pm + qm) / 2
return jsd
def KS_distance(p, q):
&#34;&#34;&#34;
Args:
p:
q:
Returns:
&#34;&#34;&#34;
dstats = np.max(np.abs(np.cumsum(p) - np.cumsum(q)))
return dstats
output = {&#39;attribute&#39;: []}
output[&#34;flagged&#34;] = []
for method in method_type:
output[method] = []
for i in list_of_cols:
if pre_existing_source:
x = spark.read.csv(source_path + &#34;/drift_statistics/frequency_counts/&#34; + i, header=True, inferSchema=True)
else:
x = source_bin.groupBy(i).agg((F.count(i) / idf_source.count()).alias(&#39;p&#39;)).fillna(-1)
x.coalesce(1).write.csv(source_path + &#34;/drift_statistics/frequency_counts/&#34; + i, header=True,
mode=&#39;overwrite&#39;)
y = target_bin.groupBy(i).agg((F.count(i) / idf_target.count()).alias(&#39;q&#39;)).fillna(-1)
xy = x.join(y, i, &#39;full_outer&#39;).fillna(0.0001, subset=[&#39;p&#39;, &#39;q&#39;]).replace(0, 0.0001).orderBy(i)
p = np.array(xy.select(&#39;p&#39;).rdd.flatMap(lambda x: x).collect())
q = np.array(xy.select(&#39;q&#39;).rdd.flatMap(lambda x: x).collect())
output[&#39;attribute&#39;].append(i)
counter = 0
for idx, method in enumerate(method_type):
drift_function = {&#39;PSI&#39;: PSI, &#39;JSD&#39;: JS_divergence, &#39;HD&#39;: hellinger_distance, &#39;KS&#39;: KS_distance}
metric = float(round(drift_function[method](p, q), 4))
output[method].append(metric)
if counter == 0:
if metric &gt; threshold:
output[&#34;flagged&#34;].append(1)
counter = 1
if (idx == (len(method_type) - 1)) &amp; (counter == 0):
output[&#34;flagged&#34;].append(0)
odf = spark.createDataFrame(pd.DataFrame.from_dict(output, orient=&#39;index&#39;).transpose()) \
.select([&#39;attribute&#39;] + method_type + [&#39;flagged&#39;]).orderBy(F.desc(&#39;flagged&#39;))
if print_impact:
print(&#34;All Attributes:&#34;)
odf.show(len(list_of_cols))
print(&#34;Attributes meeting Data Drift threshold:&#34;)
drift = odf.where(F.col(&#39;flagged&#39;) == 1)
drift.show(drift.count())
return odf
```
</details>
</dd>
<dt id="anovos.data_drift.drift_detector.stabilityIndex_computation"><code class="name flex">
<span>def <span class="ident">stabilityIndex_computation</span></span>(<span>spark, *idfs, list_of_cols='all', drop_cols=[], metric_weightages={'mean': 0.5, 'stddev': 0.3, 'kurtosis': 0.2}, existing_metric_path='', appended_metric_path='', threshold=1, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idfs</code></strong></dt>
<dd>Variable number of input dataframes</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of numerical columns to check stability e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
metric_weightages: Takes input in dictionary format with keys being the metric name - "mean","stdev","kurtosis"
and value being the weightage of the metric (between 0 and 1). Sum of all weightages must be 1. (Default value = {'mean': 0.5)
existing_metric_path: This argument is path for referring pre-existing metrics of historical datasets and is
of schema [idx, attribute, mean, stdev, kurtosis].
idx is index number of historical datasets assigned in chronological order. (Default value = '')
appended_metric_path: This argument is path for saving input dataframes metrics after appending to the
historical datasets' metrics. (Default value = '')
threshold: A column is flagged if the stability index is below the threshold, which varies between 0 to 4. (Default value = 1)
*idfs:
'stddev': 0.3:
'kurtosis': 0.2}:
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute, mean_si, stddev_si, kurtosis_si, mean_cv, stddev_cv, kurtosis_cv, stability_index].
<em>_cv is coefficient of variation for each metric. </em>_si is stability index for each metric.
stability_index is net weighted stability index based on the individual metrics' stability index.</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def stabilityIndex_computation(spark, *idfs, list_of_cols=&#39;all&#39;, drop_cols=[],
metric_weightages={&#39;mean&#39;: 0.5, &#39;stddev&#39;: 0.3, &#39;kurtosis&#39;: 0.2},
existing_metric_path=&#39;&#39;, appended_metric_path=&#39;&#39;, threshold=1, print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idfs: Variable number of input dataframes
list_of_cols: List of numerical columns to check stability e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
metric_weightages: Takes input in dictionary format with keys being the metric name - &#34;mean&#34;,&#34;stdev&#34;,&#34;kurtosis&#34;
and value being the weightage of the metric (between 0 and 1). Sum of all weightages must be 1. (Default value = {&#39;mean&#39;: 0.5)
existing_metric_path: This argument is path for referring pre-existing metrics of historical datasets and is
of schema [idx, attribute, mean, stdev, kurtosis].
idx is index number of historical datasets assigned in chronological order. (Default value = &#39;&#39;)
appended_metric_path: This argument is path for saving input dataframes metrics after appending to the
historical datasets&#39; metrics. (Default value = &#39;&#39;)
threshold: A column is flagged if the stability index is below the threshold, which varies between 0 to 4. (Default value = 1)
*idfs:
&#39;stddev&#39;: 0.3:
&#39;kurtosis&#39;: 0.2}:
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, mean_si, stddev_si, kurtosis_si, mean_cv, stddev_cv, kurtosis_cv, stability_index].
*_cv is coefficient of variation for each metric. *_si is stability index for each metric.
stability_index is net weighted stability index based on the individual metrics&#39; stability index.
&#34;&#34;&#34;
num_cols = attributeType_segregation(idfs[0])[0]
if list_of_cols == &#39;all&#39;:
list_of_cols = num_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in num_cols for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
if round(metric_weightages.get(&#39;mean&#39;, 0) + metric_weightages.get(&#39;stddev&#39;, 0) + metric_weightages.get(&#39;kurtosis&#39;,
0), 3) != 1:
raise ValueError(
&#39;Invalid input for metric weightages. Either metric name is incorrect or sum of metric weightages is not 1.0&#39;)
if existing_metric_path:
existing_metric_df = spark.read.csv(existing_metric_path, header=True, inferSchema=True)
dfs_count = existing_metric_df.select(F.max(F.col(&#39;idx&#39;))).first()[0]
else:
schema = T.StructType([T.StructField(&#39;idx&#39;, T.IntegerType(), True),
T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;mean&#39;, T.DoubleType(), True),
T.StructField(&#39;stddev&#39;, T.DoubleType(), True),
T.StructField(&#39;kurtosis&#39;, T.DoubleType(), True)])
existing_metric_df = spark.sparkContext.emptyRDD().toDF(schema)
dfs_count = 0
metric_ls = []
for idf in idfs:
for i in list_of_cols:
mean, stddev, kurtosis = idf.select(F.mean(i), F.stddev(i), F.kurtosis(i)).first()
metric_ls.append([dfs_count + 1, i, mean, stddev, kurtosis + 3.0 if kurtosis else None])
dfs_count += 1
new_metric_df = spark.createDataFrame(metric_ls, schema=(&#39;idx&#39;, &#39;attribute&#39;, &#39;mean&#39;, &#39;stddev&#39;, &#39;kurtosis&#39;))
appended_metric_df = concatenate_dataset(existing_metric_df, new_metric_df)
if appended_metric_path:
appended_metric_df.coalesce(1).write.csv(appended_metric_path, header=True, mode=&#39;overwrite&#39;)
output = []
for i in list_of_cols:
i_output = [i]
for metric in [&#39;mean&#39;, &#39;stddev&#39;, &#39;kurtosis&#39;]:
metric_stats = appended_metric_df.where(F.col(&#39;attribute&#39;) == i).orderBy(&#39;idx&#39;) \
.select(metric).fillna(np.nan).rdd.flatMap(list).collect()
metric_cv = round(float(variation([a for a in metric_stats])), 4) or None
i_output.append(metric_cv)
output.append(i_output)
schema = T.StructType([T.StructField(&#34;attribute&#34;, T.StringType(), True),
T.StructField(&#34;mean_cv&#34;, T.FloatType(), True),
T.StructField(&#34;stddev_cv&#34;, T.FloatType(), True),
T.StructField(&#34;kurtosis_cv&#34;, T.FloatType(), True)])
odf = spark.createDataFrame(output, schema=schema)
def score_cv(cv, thresholds=[0.03, 0.1, 0.2, 0.5]):
&#34;&#34;&#34;
Args:
cv:
thresholds:
(Default value = [0.03)
0.1:
0.2:
0.5]:
Returns:
&#34;&#34;&#34;
if cv is None:
return None
else:
cv = abs(cv)
stability_index = [4, 3, 2, 1, 0]
for i, thresh in enumerate(thresholds):
if cv &lt; thresh:
return stability_index[i]
return stability_index[-1]
f_score_cv = F.udf(score_cv, T.IntegerType())
odf = odf.replace(np.nan, None).withColumn(&#39;mean_si&#39;, f_score_cv(F.col(&#39;mean_cv&#39;))) \
.withColumn(&#39;stddev_si&#39;, f_score_cv(F.col(&#39;stddev_cv&#39;))) \
.withColumn(&#39;kurtosis_si&#39;, f_score_cv(F.col(&#39;kurtosis_cv&#39;))) \
.withColumn(&#39;stability_index&#39;, F.round((F.col(&#39;mean_si&#39;) * metric_weightages.get(&#39;mean&#39;, 0) +
F.col(&#39;stddev_si&#39;) * metric_weightages.get(&#39;stddev&#39;, 0) +
F.col(&#39;kurtosis_si&#39;) * metric_weightages.get(&#39;kurtosis&#39;, 0)), 4)) \
.withColumn(&#39;flagged&#39;,
F.when((F.col(&#39;stability_index&#39;) &lt; threshold) | (F.col(&#39;stability_index&#39;).isNull()), 1).otherwise(
0))
if print_impact:
print(&#34;All Attributes:&#34;)
odf.show(len(list_of_cols))
print(&#34;Potential Unstable Attributes:&#34;)
unstable = odf.where(F.col(&#39;flagged&#39;) == 1)
unstable.show(unstable.count())
return odf
```
</details>
</dd>
</dl>