# Module <code>stats_generator</code>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
# coding=utf-8
import warnings
from anovos.shared.utils import transpose_dataframe, attributeType_segregation
from pyspark.mllib.linalg import Vectors
from pyspark.mllib.stat import Statistics
from pyspark.sql import functions as F
from pyspark.sql import types as T
def global_summary(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=True):
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
print_impact:
(Default value = True)
Returns:
Dataframe [metric, value]
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
list_of_cols = idf.columns
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in idf.columns for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
row_count = idf.count()
col_count = len(list_of_cols)
num_cols, cat_cols, other_cols = attributeType_segregation(idf.select(list_of_cols))
numcol_count = len(num_cols)
catcol_count = len(cat_cols)
othercol_count = len(other_cols)
if print_impact:
print(&#34;No. of Rows: %s&#34; % &#34;{0:,}&#34;.format(row_count))
print(&#34;No. of Columns: %s&#34; % &#34;{0:,}&#34;.format(col_count))
print(&#34;Numerical Columns: %s&#34; % &#34;{0:,}&#34;.format(numcol_count))
if numcol_count &gt; 0:
print(num_cols)
print(&#34;Categorical Columns: %s&#34; % &#34;{0:,}&#34;.format(catcol_count))
if catcol_count &gt; 0:
print(cat_cols)
if othercol_count &gt; 0:
print(&#34;Other Columns: %s&#34; % &#34;{0:,}&#34;.format(othercol_count))
print(other_cols)
odf = spark.createDataFrame([[&#34;rows_count&#34;, str(row_count)], [&#34;columns_count&#34;, str(col_count)],
[&#34;numcols_count&#34;, str(numcol_count)], [&#34;numcols_name&#34;, &#39;, &#39;.join(num_cols)],
[&#34;catcols_count&#34;, str(catcol_count)], [&#34;catcols_name&#34;, &#39;, &#39;.join(cat_cols)],
[&#34;othercols_count&#34;, str(othercol_count)], [&#34;othercols_name&#34;, &#39;, &#39;.join(other_cols)]],
schema=[&#39;metric&#39;, &#39;value&#39;])
return odf
def missingCount_computation(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
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
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, missing_count, missing_pct]
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
idf_stats = idf.select(list_of_cols).summary(&#34;count&#34;)
odf = transpose_dataframe(idf_stats, &#39;summary&#39;) \
.withColumn(&#39;missing_count&#39;, F.lit(idf.count()) - F.col(&#39;count&#39;).cast(T.LongType())) \
.withColumn(&#39;missing_pct&#39;, F.round(F.col(&#39;missing_count&#39;) / F.lit(idf.count()), 4)) \
.select(F.col(&#39;key&#39;).alias(&#39;attribute&#39;), &#39;missing_count&#39;, &#39;missing_pct&#39;)
if print_impact:
odf.show(len(list_of_cols))
return odf
def nonzeroCount_computation(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of numerical columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, nonzero_count, nonzero_pct]
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
warnings.warn(&#34;No Non-Zero Count Computation - No numerical column(s) to analyze&#34;)
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;nonzero_count&#39;, T.StringType(), True),
T.StructField(&#39;nonzero_pct&#39;, T.StringType(), True)])
odf = spark.sparkContext.emptyRDD().toDF(schema)
return odf
tmp = idf.select(list_of_cols).fillna(0).rdd.map(lambda row: Vectors.dense(row))
nonzero_count = Statistics.colStats(tmp).numNonzeros()
odf = spark.createDataFrame(zip(list_of_cols, [int(i) for i in nonzero_count]),
schema=(&#34;attribute&#34;, &#34;nonzero_count&#34;)) \
.withColumn(&#34;nonzero_pct&#34;, F.round(F.col(&#39;nonzero_count&#39;) / F.lit(idf.count()), 4))
if print_impact:
odf.show(len(list_of_cols))
return odf
def measures_of_counts(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
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
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, fill_count, fill_pct, missing_count, missing_pct, nonzero_count, nonzero_pct]
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
num_cols = attributeType_segregation(idf.select(list_of_cols))[0]
if any(x not in idf.columns for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
odf = transpose_dataframe(idf.select(list_of_cols).summary(&#34;count&#34;), &#39;summary&#39;) \
.select(F.col(&#34;key&#34;).alias(&#34;attribute&#34;), F.col(&#34;count&#34;).cast(T.LongType()).alias(&#34;fill_count&#34;)) \
.withColumn(&#39;fill_pct&#39;, F.round(F.col(&#39;fill_count&#39;) / F.lit(idf.count()), 4)) \
.withColumn(&#39;missing_count&#39;, F.lit(idf.count()) - F.col(&#39;fill_count&#39;).cast(T.LongType())) \
.withColumn(&#39;missing_pct&#39;, F.round(1 - F.col(&#39;fill_pct&#39;), 4)) \
.join(nonzeroCount_computation(spark, idf, num_cols), &#34;attribute&#34;, &#34;full_outer&#34;)
if print_impact:
odf.show(len(list_of_cols))
return odf
def mode_computation(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of Discrete (Categorical + Integer) columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all discrete columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, mode, mode_rows]
In case there is tie between multiple values, one value is randomly picked as mode.
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
warnings.warn(&#34;No Mode Computation - No discrete column(s) to analyze&#34;)
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;mode&#39;, T.StringType(), True),
T.StructField(&#39;mode_rows&#39;, T.StringType(), True)])
odf = spark.sparkContext.emptyRDD().toDF(schema)
return odf
mode = [list(idf.select(i).dropna().groupby(i).count().orderBy(&#34;count&#34;, ascending=False).first() or [None, None])
for i in list_of_cols]
mode = [(str(i), str(j)) for i, j in mode]
odf = spark.createDataFrame(zip(list_of_cols, mode), schema=(&#34;attribute&#34;, &#34;metric&#34;)) \
.select(&#39;attribute&#39;, (F.col(&#39;metric&#39;)[&#34;_1&#34;]).alias(&#39;mode&#39;),
(F.col(&#39;metric&#39;)[&#34;_2&#34;]).cast(&#34;long&#34;).alias(&#39;mode_rows&#39;))
if print_impact:
odf.show(len(list_of_cols))
return odf
def measures_of_centralTendency(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
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
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, mean, median, mode, mode_rows, mode_pct]
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
odf = transpose_dataframe(idf.select(list_of_cols).summary(&#34;mean&#34;, &#34;50%&#34;, &#34;count&#34;), &#39;summary&#39;) \
.withColumn(&#39;mean&#39;, F.round(F.col(&#39;mean&#39;).cast(T.DoubleType()), 4)) \
.withColumn(&#39;median&#39;, F.round(F.col(&#39;50%&#39;).cast(T.DoubleType()), 4)) \
.withColumnRenamed(&#39;key&#39;, &#39;attribute&#39;) \
.join(mode_computation(spark, idf, list_of_cols), &#39;attribute&#39;, &#39;full_outer&#39;) \
.withColumn(&#39;mode_pct&#39;, F.round(F.col(&#39;mode_rows&#39;) / F.col(&#39;count&#39;).cast(T.DoubleType()), 4)) \
.select(&#39;attribute&#39;, &#39;mean&#39;, &#39;median&#39;, &#39;mode&#39;, &#39;mode_rows&#39;, &#39;mode_pct&#39;)
if print_impact:
odf.show(len(list_of_cols))
return odf
def uniqueCount_computation(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of Discrete (Categorical + Integer) columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all discrete columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, unique_values]
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
warnings.warn(&#34;No Unique Count Computation - No discrete column(s) to analyze&#34;)
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;unique_values&#39;, T.StringType(), True)])
odf = spark.sparkContext.emptyRDD().toDF(schema)
return odf
uniquevalue_count = idf.agg(*(F.countDistinct(F.col(i)).alias(i) for i in list_of_cols))
odf = spark.createDataFrame(zip(list_of_cols, uniquevalue_count.rdd.map(list).collect()[0]),
schema=(&#34;attribute&#34;, &#34;unique_values&#34;))
if print_impact:
odf.show(len(list_of_cols))
return odf
def measures_of_cardinality(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of Discrete (Categorical + Integer) columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all discrete columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, unique_values, IDness]
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
warnings.warn(&#34;No Cardinality Computation - No discrete column(s) to analyze&#34;)
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;unique_values&#39;, T.StringType(), True),
T.StructField(&#39;IDness&#39;, T.StringType(), True)])
odf = spark.sparkContext.emptyRDD().toDF(schema)
return odf
odf = uniqueCount_computation(spark, idf, list_of_cols) \
.join(missingCount_computation(spark, idf, list_of_cols), &#39;attribute&#39;, &#39;full_outer&#39;) \
.withColumn(&#39;IDness&#39;, F.round(F.col(&#39;unique_values&#39;) / (F.lit(idf.count()) - F.col(&#39;missing_count&#39;)), 4)) \
.select(&#39;attribute&#39;, &#39;unique_values&#39;, &#39;IDness&#39;)
if print_impact:
odf.show(len(list_of_cols))
return odf
def measures_of_dispersion(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of numerical columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, stddev, variance, cov, IQR, range]
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
warnings.warn(&#34;No Dispersion Computation - No numerical column(s) to analyze&#34;)
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;stddev&#39;, T.StringType(), True),
T.StructField(&#39;variance&#39;, T.StringType(), True),
T.StructField(&#39;cov&#39;, T.StringType(), True),
T.StructField(&#39;IQR&#39;, T.StringType(), True),
T.StructField(&#39;range&#39;, T.StringType(), True)])
odf = spark.sparkContext.emptyRDD().toDF(schema)
return odf
odf = transpose_dataframe(idf.select(list_of_cols).summary(&#34;stddev&#34;, &#34;min&#34;, &#34;max&#34;, &#34;mean&#34;, &#34;25%&#34;, &#34;75%&#34;), &#39;summary&#39;) \
.withColumn(&#39;stddev&#39;, F.round(F.col(&#39;stddev&#39;).cast(T.DoubleType()), 4)) \
.withColumn(&#39;variance&#39;, F.round(F.col(&#39;stddev&#39;) * F.col(&#39;stddev&#39;), 4)) \
.withColumn(&#39;range&#39;, F.round(F.col(&#39;max&#39;) - F.col(&#39;min&#39;), 4)) \
.withColumn(&#39;cov&#39;, F.round(F.col(&#39;stddev&#39;) / F.col(&#39;mean&#39;), 4)) \
.withColumn(&#39;IQR&#39;, F.round(F.col(&#39;75%&#39;) - F.col(&#39;25%&#39;), 4)) \
.select(F.col(&#39;key&#39;).alias(&#39;attribute&#39;), &#39;stddev&#39;, &#39;variance&#39;, &#39;cov&#39;, &#39;IQR&#39;, &#39;range&#39;)
if print_impact:
odf.show(len(list_of_cols))
return odf
def measures_of_percentiles(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of numerical columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, min, 1%, 5%, 10%, 25%, 50%, 75%, 90%, 95%, 99%, max]
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
warnings.warn(&#34;No Percentiles Computation - No numerical column(s) to analyze&#34;)
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;min&#39;, T.StringType(), True),
T.StructField(&#39;1%&#39;, T.StringType(), True),
T.StructField(&#39;5%&#39;, T.StringType(), True),
T.StructField(&#39;10%&#39;, T.StringType(), True),
T.StructField(&#39;25%&#39;, T.StringType(), True),
T.StructField(&#39;50%&#39;, T.StringType(), True),
T.StructField(&#39;75%&#39;, T.StringType(), True),
T.StructField(&#39;90%&#39;, T.StringType(), True),
T.StructField(&#39;95%&#39;, T.StringType(), True),
T.StructField(&#39;99%&#39;, T.StringType(), True),
T.StructField(&#39;max&#39;, T.StringType(), True)])
odf = spark.sparkContext.emptyRDD().toDF(schema)
return odf
stats = [&#34;min&#34;, &#34;1%&#34;, &#34;5%&#34;, &#34;10%&#34;, &#34;25%&#34;, &#34;50%&#34;, &#34;75%&#34;, &#34;90%&#34;, &#34;95%&#34;, &#34;99%&#34;, &#34;max&#34;]
odf = transpose_dataframe(idf.select(list_of_cols).summary(*stats), &#39;summary&#39;) \
.withColumnRenamed(&#34;key&#34;, &#34;attribute&#34;)
for i in odf.columns:
if i != &#34;attribute&#34;:
odf = odf.withColumn(i, F.round(F.col(i).cast(&#34;Double&#34;), 4))
odf = odf.select([&#39;attribute&#39;] + stats)
if print_impact:
odf.show(len(list_of_cols))
return odf
def measures_of_shape(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
list_of_cols: List of numerical columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
spark:
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, skewness, kurtosis]
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
warnings.warn(&#34;No Skewness/Kurtosis Computation - No numerical column(s) to analyze&#34;)
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;skewness&#39;, T.StringType(), True),
T.StructField(&#39;kurtosis&#39;, T.StringType(), True)])
odf = spark.sparkContext.emptyRDD().toDF(schema)
return odf
shapes = []
for i in list_of_cols:
s, k = idf.select(F.skewness(i), F.kurtosis(i)).first()
shapes.append([i, s, k])
odf = spark.createDataFrame(shapes, schema=(&#34;attribute&#34;, &#34;skewness&#34;, &#34;kurtosis&#34;)) \
.withColumn(&#39;skewness&#39;, F.round(F.col(&#34;skewness&#34;), 4)) \
.withColumn(&#39;kurtosis&#39;, F.round(F.col(&#34;kurtosis&#34;), 4))
if print_impact:
odf.show(len(list_of_cols))
return odf
```
</details>
## Functions
<dl>
<dt id="anovos.data_analyzer.stats_generator.global_summary"><code class="name flex">
<span>def <span class="ident">global_summary</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], print_impact=True)</span>
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
print_impact:
(Default value = True)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe [metric, value]</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def global_summary(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=True):
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
print_impact:
(Default value = True)
Returns:
Dataframe [metric, value]
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
list_of_cols = idf.columns
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
if any(x not in idf.columns for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
row_count = idf.count()
col_count = len(list_of_cols)
num_cols, cat_cols, other_cols = attributeType_segregation(idf.select(list_of_cols))
numcol_count = len(num_cols)
catcol_count = len(cat_cols)
othercol_count = len(other_cols)
if print_impact:
print(&#34;No. of Rows: %s&#34; % &#34;{0:,}&#34;.format(row_count))
print(&#34;No. of Columns: %s&#34; % &#34;{0:,}&#34;.format(col_count))
print(&#34;Numerical Columns: %s&#34; % &#34;{0:,}&#34;.format(numcol_count))
if numcol_count &gt; 0:
print(num_cols)
print(&#34;Categorical Columns: %s&#34; % &#34;{0:,}&#34;.format(catcol_count))
if catcol_count &gt; 0:
print(cat_cols)
if othercol_count &gt; 0:
print(&#34;Other Columns: %s&#34; % &#34;{0:,}&#34;.format(othercol_count))
print(other_cols)
odf = spark.createDataFrame([[&#34;rows_count&#34;, str(row_count)], [&#34;columns_count&#34;, str(col_count)],
[&#34;numcols_count&#34;, str(numcol_count)], [&#34;numcols_name&#34;, &#39;, &#39;.join(num_cols)],
[&#34;catcols_count&#34;, str(catcol_count)], [&#34;catcols_name&#34;, &#39;, &#39;.join(cat_cols)],
[&#34;othercols_count&#34;, str(othercol_count)], [&#34;othercols_name&#34;, &#39;, &#39;.join(other_cols)]],
schema=[&#39;metric&#39;, &#39;value&#39;])
return odf
```
</details>
</dd>
<dt id="anovos.data_analyzer.stats_generator.measures_of_cardinality"><code class="name flex">
<span>def <span class="ident">measures_of_cardinality</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of Discrete (Categorical + Integer) columns to analyse e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all discrete columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute, unique_values, IDness]</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def measures_of_cardinality(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of Discrete (Categorical + Integer) columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all discrete columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, unique_values, IDness]
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
warnings.warn(&#34;No Cardinality Computation - No discrete column(s) to analyze&#34;)
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;unique_values&#39;, T.StringType(), True),
T.StructField(&#39;IDness&#39;, T.StringType(), True)])
odf = spark.sparkContext.emptyRDD().toDF(schema)
return odf
odf = uniqueCount_computation(spark, idf, list_of_cols) \
.join(missingCount_computation(spark, idf, list_of_cols), &#39;attribute&#39;, &#39;full_outer&#39;) \
.withColumn(&#39;IDness&#39;, F.round(F.col(&#39;unique_values&#39;) / (F.lit(idf.count()) - F.col(&#39;missing_count&#39;)), 4)) \
.select(&#39;attribute&#39;, &#39;unique_values&#39;, &#39;IDness&#39;)
if print_impact:
odf.show(len(list_of_cols))
return odf
```
</details>
</dd>
<dt id="anovos.data_analyzer.stats_generator.measures_of_centralTendency"><code class="name flex">
<span>def <span class="ident">measures_of_centralTendency</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], print_impact=False)</span>
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
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute, mean, median, mode, mode_rows, mode_pct]</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def measures_of_centralTendency(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
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
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, mean, median, mode, mode_rows, mode_pct]
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
odf = transpose_dataframe(idf.select(list_of_cols).summary(&#34;mean&#34;, &#34;50%&#34;, &#34;count&#34;), &#39;summary&#39;) \
.withColumn(&#39;mean&#39;, F.round(F.col(&#39;mean&#39;).cast(T.DoubleType()), 4)) \
.withColumn(&#39;median&#39;, F.round(F.col(&#39;50%&#39;).cast(T.DoubleType()), 4)) \
.withColumnRenamed(&#39;key&#39;, &#39;attribute&#39;) \
.join(mode_computation(spark, idf, list_of_cols), &#39;attribute&#39;, &#39;full_outer&#39;) \
.withColumn(&#39;mode_pct&#39;, F.round(F.col(&#39;mode_rows&#39;) / F.col(&#39;count&#39;).cast(T.DoubleType()), 4)) \
.select(&#39;attribute&#39;, &#39;mean&#39;, &#39;median&#39;, &#39;mode&#39;, &#39;mode_rows&#39;, &#39;mode_pct&#39;)
if print_impact:
odf.show(len(list_of_cols))
return odf
```
</details>
</dd>
<dt id="anovos.data_analyzer.stats_generator.measures_of_counts"><code class="name flex">
<span>def <span class="ident">measures_of_counts</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], print_impact=False)</span>
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
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute, fill_count, fill_pct, missing_count, missing_pct, nonzero_count, nonzero_pct]</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def measures_of_counts(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
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
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, fill_count, fill_pct, missing_count, missing_pct, nonzero_count, nonzero_pct]
&#34;&#34;&#34;
if list_of_cols == &#39;all&#39;:
num_cols, cat_cols, other_cols = attributeType_segregation(idf)
list_of_cols = num_cols + cat_cols
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(drop_cols, str):
drop_cols = [x.strip() for x in drop_cols.split(&#39;|&#39;)]
list_of_cols = list(set([e for e in list_of_cols if e not in drop_cols]))
num_cols = attributeType_segregation(idf.select(list_of_cols))[0]
if any(x not in idf.columns for x in list_of_cols) | (len(list_of_cols) == 0):
raise TypeError(&#39;Invalid input for Column(s)&#39;)
odf = transpose_dataframe(idf.select(list_of_cols).summary(&#34;count&#34;), &#39;summary&#39;) \
.select(F.col(&#34;key&#34;).alias(&#34;attribute&#34;), F.col(&#34;count&#34;).cast(T.LongType()).alias(&#34;fill_count&#34;)) \
.withColumn(&#39;fill_pct&#39;, F.round(F.col(&#39;fill_count&#39;) / F.lit(idf.count()), 4)) \
.withColumn(&#39;missing_count&#39;, F.lit(idf.count()) - F.col(&#39;fill_count&#39;).cast(T.LongType())) \
.withColumn(&#39;missing_pct&#39;, F.round(1 - F.col(&#39;fill_pct&#39;), 4)) \
.join(nonzeroCount_computation(spark, idf, num_cols), &#34;attribute&#34;, &#34;full_outer&#34;)
if print_impact:
odf.show(len(list_of_cols))
return odf
```
</details>
</dd>
<dt id="anovos.data_analyzer.stats_generator.measures_of_dispersion"><code class="name flex">
<span>def <span class="ident">measures_of_dispersion</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of numerical columns to analyse e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute, stddev, variance, cov, IQR, range]</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def measures_of_dispersion(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of numerical columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, stddev, variance, cov, IQR, range]
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
warnings.warn(&#34;No Dispersion Computation - No numerical column(s) to analyze&#34;)
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;stddev&#39;, T.StringType(), True),
T.StructField(&#39;variance&#39;, T.StringType(), True),
T.StructField(&#39;cov&#39;, T.StringType(), True),
T.StructField(&#39;IQR&#39;, T.StringType(), True),
T.StructField(&#39;range&#39;, T.StringType(), True)])
odf = spark.sparkContext.emptyRDD().toDF(schema)
return odf
odf = transpose_dataframe(idf.select(list_of_cols).summary(&#34;stddev&#34;, &#34;min&#34;, &#34;max&#34;, &#34;mean&#34;, &#34;25%&#34;, &#34;75%&#34;), &#39;summary&#39;) \
.withColumn(&#39;stddev&#39;, F.round(F.col(&#39;stddev&#39;).cast(T.DoubleType()), 4)) \
.withColumn(&#39;variance&#39;, F.round(F.col(&#39;stddev&#39;) * F.col(&#39;stddev&#39;), 4)) \
.withColumn(&#39;range&#39;, F.round(F.col(&#39;max&#39;) - F.col(&#39;min&#39;), 4)) \
.withColumn(&#39;cov&#39;, F.round(F.col(&#39;stddev&#39;) / F.col(&#39;mean&#39;), 4)) \
.withColumn(&#39;IQR&#39;, F.round(F.col(&#39;75%&#39;) - F.col(&#39;25%&#39;), 4)) \
.select(F.col(&#39;key&#39;).alias(&#39;attribute&#39;), &#39;stddev&#39;, &#39;variance&#39;, &#39;cov&#39;, &#39;IQR&#39;, &#39;range&#39;)
if print_impact:
odf.show(len(list_of_cols))
return odf
```
</details>
</dd>
<dt id="anovos.data_analyzer.stats_generator.measures_of_percentiles"><code class="name flex">
<span>def <span class="ident">measures_of_percentiles</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of numerical columns to analyse e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute, min, 1%, 5%, 10%, 25%, 50%, 75%, 90%, 95%, 99%, max]</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def measures_of_percentiles(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of numerical columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, min, 1%, 5%, 10%, 25%, 50%, 75%, 90%, 95%, 99%, max]
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
warnings.warn(&#34;No Percentiles Computation - No numerical column(s) to analyze&#34;)
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;min&#39;, T.StringType(), True),
T.StructField(&#39;1%&#39;, T.StringType(), True),
T.StructField(&#39;5%&#39;, T.StringType(), True),
T.StructField(&#39;10%&#39;, T.StringType(), True),
T.StructField(&#39;25%&#39;, T.StringType(), True),
T.StructField(&#39;50%&#39;, T.StringType(), True),
T.StructField(&#39;75%&#39;, T.StringType(), True),
T.StructField(&#39;90%&#39;, T.StringType(), True),
T.StructField(&#39;95%&#39;, T.StringType(), True),
T.StructField(&#39;99%&#39;, T.StringType(), True),
T.StructField(&#39;max&#39;, T.StringType(), True)])
odf = spark.sparkContext.emptyRDD().toDF(schema)
return odf
stats = [&#34;min&#34;, &#34;1%&#34;, &#34;5%&#34;, &#34;10%&#34;, &#34;25%&#34;, &#34;50%&#34;, &#34;75%&#34;, &#34;90%&#34;, &#34;95%&#34;, &#34;99%&#34;, &#34;max&#34;]
odf = transpose_dataframe(idf.select(list_of_cols).summary(*stats), &#39;summary&#39;) \
.withColumnRenamed(&#34;key&#34;, &#34;attribute&#34;)
for i in odf.columns:
if i != &#34;attribute&#34;:
odf = odf.withColumn(i, F.round(F.col(i).cast(&#34;Double&#34;), 4))
odf = odf.select([&#39;attribute&#39;] + stats)
if print_impact:
odf.show(len(list_of_cols))
return odf
```
</details>
</dd>
<dt id="anovos.data_analyzer.stats_generator.measures_of_shape"><code class="name flex">
<span>def <span class="ident">measures_of_shape</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of numerical columns to analyse e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
spark:
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute, skewness, kurtosis]</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def measures_of_shape(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
list_of_cols: List of numerical columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
spark:
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, skewness, kurtosis]
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
warnings.warn(&#34;No Skewness/Kurtosis Computation - No numerical column(s) to analyze&#34;)
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;skewness&#39;, T.StringType(), True),
T.StructField(&#39;kurtosis&#39;, T.StringType(), True)])
odf = spark.sparkContext.emptyRDD().toDF(schema)
return odf
shapes = []
for i in list_of_cols:
s, k = idf.select(F.skewness(i), F.kurtosis(i)).first()
shapes.append([i, s, k])
odf = spark.createDataFrame(shapes, schema=(&#34;attribute&#34;, &#34;skewness&#34;, &#34;kurtosis&#34;)) \
.withColumn(&#39;skewness&#39;, F.round(F.col(&#34;skewness&#34;), 4)) \
.withColumn(&#39;kurtosis&#39;, F.round(F.col(&#34;kurtosis&#34;), 4))
if print_impact:
odf.show(len(list_of_cols))
return odf
```
</details>
</dd>
<dt id="anovos.data_analyzer.stats_generator.missingCount_computation"><code class="name flex">
<span>def <span class="ident">missingCount_computation</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], print_impact=False)</span>
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
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute, missing_count, missing_pct]</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def missingCount_computation(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
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
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, missing_count, missing_pct]
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
idf_stats = idf.select(list_of_cols).summary(&#34;count&#34;)
odf = transpose_dataframe(idf_stats, &#39;summary&#39;) \
.withColumn(&#39;missing_count&#39;, F.lit(idf.count()) - F.col(&#39;count&#39;).cast(T.LongType())) \
.withColumn(&#39;missing_pct&#39;, F.round(F.col(&#39;missing_count&#39;) / F.lit(idf.count()), 4)) \
.select(F.col(&#39;key&#39;).alias(&#39;attribute&#39;), &#39;missing_count&#39;, &#39;missing_pct&#39;)
if print_impact:
odf.show(len(list_of_cols))
return odf
```
</details>
</dd>
<dt id="anovos.data_analyzer.stats_generator.mode_computation"><code class="name flex">
<span>def <span class="ident">mode_computation</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of Discrete (Categorical + Integer) columns to analyse e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all discrete columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute, mode, mode_rows]
In case there is tie between multiple values, one value is randomly picked as mode.</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def mode_computation(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of Discrete (Categorical + Integer) columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all discrete columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, mode, mode_rows]
In case there is tie between multiple values, one value is randomly picked as mode.
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
warnings.warn(&#34;No Mode Computation - No discrete column(s) to analyze&#34;)
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;mode&#39;, T.StringType(), True),
T.StructField(&#39;mode_rows&#39;, T.StringType(), True)])
odf = spark.sparkContext.emptyRDD().toDF(schema)
return odf
mode = [list(idf.select(i).dropna().groupby(i).count().orderBy(&#34;count&#34;, ascending=False).first() or [None, None])
for i in list_of_cols]
mode = [(str(i), str(j)) for i, j in mode]
odf = spark.createDataFrame(zip(list_of_cols, mode), schema=(&#34;attribute&#34;, &#34;metric&#34;)) \
.select(&#39;attribute&#39;, (F.col(&#39;metric&#39;)[&#34;_1&#34;]).alias(&#39;mode&#39;),
(F.col(&#39;metric&#39;)[&#34;_2&#34;]).cast(&#34;long&#34;).alias(&#39;mode_rows&#39;))
if print_impact:
odf.show(len(list_of_cols))
return odf
```
</details>
</dd>
<dt id="anovos.data_analyzer.stats_generator.nonzeroCount_computation"><code class="name flex">
<span>def <span class="ident">nonzeroCount_computation</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of numerical columns to analyse e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute, nonzero_count, nonzero_pct]</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def nonzeroCount_computation(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of numerical columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all numerical columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, nonzero_count, nonzero_pct]
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
warnings.warn(&#34;No Non-Zero Count Computation - No numerical column(s) to analyze&#34;)
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;nonzero_count&#39;, T.StringType(), True),
T.StructField(&#39;nonzero_pct&#39;, T.StringType(), True)])
odf = spark.sparkContext.emptyRDD().toDF(schema)
return odf
tmp = idf.select(list_of_cols).fillna(0).rdd.map(lambda row: Vectors.dense(row))
nonzero_count = Statistics.colStats(tmp).numNonzeros()
odf = spark.createDataFrame(zip(list_of_cols, [int(i) for i in nonzero_count]),
schema=(&#34;attribute&#34;, &#34;nonzero_count&#34;)) \
.withColumn(&#34;nonzero_pct&#34;, F.round(F.col(&#39;nonzero_count&#39;) / F.lit(idf.count()), 4))
if print_impact:
odf.show(len(list_of_cols))
return odf
```
</details>
</dd>
<dt id="anovos.data_analyzer.stats_generator.uniqueCount_computation"><code class="name flex">
<span>def <span class="ident">uniqueCount_computation</span></span>(<span>spark, idf, list_of_cols='all', drop_cols=[], print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of Discrete (Categorical + Integer) columns to analyse e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
"all" can be passed to include all discrete columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')
drop_cols: List of columns to be dropped e.g., ["col1","col2"].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute, unique_values]</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def uniqueCount_computation(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], print_impact=False):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
list_of_cols: List of Discrete (Categorical + Integer) columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
&#34;all&#34; can be passed to include all discrete columns for analysis.
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
print_impact:
(Default value = False)
Returns:
Dataframe [attribute, unique_values]
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
warnings.warn(&#34;No Unique Count Computation - No discrete column(s) to analyze&#34;)
schema = T.StructType([T.StructField(&#39;attribute&#39;, T.StringType(), True),
T.StructField(&#39;unique_values&#39;, T.StringType(), True)])
odf = spark.sparkContext.emptyRDD().toDF(schema)
return odf
uniquevalue_count = idf.agg(*(F.countDistinct(F.col(i)).alias(i) for i in list_of_cols))
odf = spark.createDataFrame(zip(list_of_cols, uniquevalue_count.rdd.map(list).collect()[0]),
schema=(&#34;attribute&#34;, &#34;unique_values&#34;))
if print_impact:
odf.show(len(list_of_cols))
return odf
```
</details>
</dd>
</dl>