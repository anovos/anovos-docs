# Module <code>data_ingest</code>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
# coding=utf-8
from anovos.shared.utils import pairwise_reduce
from pyspark.sql import DataFrame
from pyspark.sql import functions as F
def read_dataset(spark, file_path, file_type, file_configs={}):
&#34;&#34;&#34;
Args:
spark: Spark Session
file_path: Path to input data (directory or filename).
Compatible with local path and s3 path (when running in AWS environment).
file_type: csv&#34;, &#34;parquet&#34;, &#34;avro&#34;.
Avro data source requires an external package to run, which can be configured with
spark-submit (--packages org.apache.spark:spark-avro_2.11:2.4.0).
file_configs: This argument is passed in a dictionary format as key/value pairs
e.g. {&#34;header&#34;: &#34;True&#34;,&#34;delimiter&#34;: &#34;|&#34;,&#34;inferSchema&#34;: &#34;True&#34;} for csv files.
All the key/value pairs in this argument are passed as options to DataFrameReader,
which is created using SparkSession.read. (Default value = {})
Returns:
Dataframe
&#34;&#34;&#34;
odf = spark.read.format(file_type).options(**file_configs).load(file_path)
return odf
def write_dataset(idf, file_path, file_type, file_configs={}, column_order=[]):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
file_path: Path to output data (directory or filename).
Compatible with local path and s3 path (when running in AWS environment).
file_type: csv&#34;, &#34;parquet&#34;, &#34;avro&#34;.
Avro data source requires an external package to run, which can be configured with
spark-submit (--packages org.apache.spark:spark-avro_2.11:2.4.0).
file_configs: This argument is passed in dictionary format as key/value pairs.
Some of the potential keys are header, delimiter, mode, compression, repartition.
compression options - uncompressed, gzip (doesn&#39;t work with avro), snappy (only valid for parquet)
mode options - error (default), overwrite, append
repartition - None (automatic partitioning) or an integer value ()
e.g. {&#34;header&#34;:&#34;True&#34;,&#34;delimiter&#34;:&#34;,&#34;,&#39;compression&#39;:&#39;snappy&#39;,&#39;mode&#39;:&#39;overwrite&#39;,&#39;repartition&#39;:&#39;10&#39;}.
column_order: list of columns in the order in which Dataframe is to be written. If None or [] is specified, then the default order is applied.
Returns:
None (Dataframe saved)
&#34;&#34;&#34;
if not column_order:
column_order = idf.columns
else:
if not isinstance(column_order, list):
raise TypeError(&#39;Invalid input type for column_order argument&#39;)
if len(column_order) != len(idf.columns):
raise ValueError(&#39;Count of column(s) specified in column_order argument do not match Dataframe&#39;)
diff_cols = [x for x in column_order if x not in set(idf.columns)]
if diff_cols:
raise ValueError(&#39;Column(s) specified in column_order argument not found in Dataframe: &#39; + str(diff_cols))
mode = file_configs[&#39;mode&#39;] if &#39;mode&#39; in file_configs else &#39;error&#39;
repartition = int(file_configs[&#39;repartition&#39;]) if &#39;repartition&#39; in file_configs else None
if repartition is None:
idf.select(column_order).write.format(file_type).options(**file_configs).save(file_path, mode=mode)
else:
exist_parts = idf.rdd.getNumPartitions()
req_parts = int(repartition)
if req_parts &gt; exist_parts:
idf.select(column_order).repartition(req_parts).write.format(file_type).options(**file_configs).save(file_path, mode=mode)
else:
idf.select(column_order).coalesce(req_parts).write.format(file_type).options(**file_configs).save(file_path, mode=mode)
def concatenate_dataset(*idfs, method_type=&#39;name&#39;):
&#34;&#34;&#34;
Args:
dfs: All dataframes to be concatenated (with the first dataframe columns)
method_type: index&#34;, &#34;name&#34;.
This argument needs to be passed as a keyword argument.
&#34;index&#34; method concatenates by column index positioning, without shuffling columns.
&#34;name&#34; concatenates after shuffling and arranging columns as per the first dataframe.
First dataframe passed under idfs will define the final columns in the concatenated dataframe,
and will throw error if any column in first dataframe is not available in any of other dataframes. (Default value = &#39;name&#39;)
*idfs:
Returns:
Concatenated Dataframe
&#34;&#34;&#34;
if (method_type not in [&#39;index&#39;, &#39;name&#39;]):
raise TypeError(&#39;Invalid input for concatenate_dataset method&#39;)
if method_type == &#39;name&#39;:
odf = pairwise_reduce(lambda idf1, idf2: idf1.union(idf2.select(idf1.columns)), idfs)
# odf = reduce(DataFrame.unionByName, idfs) # only if exact no. of columns
else:
odf = pairwise_reduce(DataFrame.union, idfs)
return odf
def join_dataset(*idfs, join_cols, join_type):
&#34;&#34;&#34;
Args:
idfs: All dataframes to be joined
join_cols: Key column(s) to join all dataframes together.
In case of multiple columns to join, they can be passed in a list format or
a string format where different column names are separated by pipe delimiter “|”.
join_type: inner&#34;, “full”, “left”, “right”, “left_semi”, “left_anti”
*idfs:
Returns:
Joined Dataframe
&#34;&#34;&#34;
if isinstance(join_cols, str):
join_cols = [x.strip() for x in join_cols.split(&#39;|&#39;)]
odf = pairwise_reduce(lambda idf1, idf2: idf1.join(idf2, join_cols, join_type), idfs)
return odf
def delete_column(idf, list_of_cols, print_impact=False):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
list_of_cols: List of columns to delete e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
print_impact:
(Default value = False)
Returns:
Dataframe after dropping columns
&#34;&#34;&#34;
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
list_of_cols = list(set(list_of_cols))
odf = idf.drop(*list_of_cols)
if print_impact:
print(&#34;Before: \nNo. of Columns- &#34;, len(idf.columns))
print(idf.columns)
print(&#34;After: \nNo. of Columns- &#34;, len(odf.columns))
print(odf.columns)
return odf
def select_column(idf, list_of_cols, print_impact=False):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
list_of_cols: List of columns to select e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
print_impact:
(Default value = False)
Returns:
Dataframe with the selected columns
&#34;&#34;&#34;
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
list_of_cols = list(set(list_of_cols))
odf = idf.select(list_of_cols)
if print_impact:
print(&#34;Before: \nNo. of Columns-&#34;, len(idf.columns))
print(idf.columns)
print(&#34;\nAfter: \nNo. of Columns-&#34;, len(odf.columns))
print(odf.columns)
return odf
def rename_column(idf, list_of_cols, list_of_newcols, print_impact=False):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
list_of_cols: List of old column names e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
list_of_newcols: List of corresponding new column names e.g., [&#34;newcol1&#34;,&#34;newcol2&#34;].
Alternatively, new column names can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;newcol1|newcol2&#34;.
First element in list_of_cols will be original column name,
and corresponding first column in list_of_newcols will be new column name.
print_impact:
(Default value = False)
Returns:
Dataframe with revised column names
&#34;&#34;&#34;
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(list_of_newcols, str):
list_of_newcols = [x.strip() for x in list_of_newcols.split(&#39;|&#39;)]
mapping = dict(zip(list_of_cols, list_of_newcols))
odf = idf.select([F.col(i).alias(mapping.get(i, i)) for i in idf.columns])
if print_impact:
print(&#34;Before: \nNo. of Columns- &#34;, len(idf.columns))
print(idf.columns)
print(&#34;After: \nNo. of Columns- &#34;, len(odf.columns))
print(odf.columns)
return odf
def recast_column(idf, list_of_cols, list_of_dtypes, print_impact=False):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
list_of_cols: List of columns to cast e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
list_of_dtypes: List of corresponding datatypes e.g., [&#34;type1&#34;,&#34;type2&#34;].
Alternatively, datatypes can be specified in a string format,
where they are separated by pipe delimiter “|” e.g., &#34;type1|type2&#34;.
First element in list_of_cols will column name and corresponding element in list_of_dtypes
will be new datatypes such as &#34;float&#34;, &#34;integer&#34;, &#34;long&#34;, &#34;string&#34;, &#34;double&#34;, decimal&#34; etc.
Datatypes are case insensitive e.g. float or Float are treated as same.
print_impact:
(Default value = False)
Returns:
Dataframe with revised datatypes
&#34;&#34;&#34;
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(list_of_dtypes, str):
list_of_dtypes = [x.strip() for x in list_of_dtypes.split(&#39;|&#39;)]
odf = idf
for i, j in zip(list_of_cols, list_of_dtypes):
odf = odf.withColumn(i, F.col(i).cast(j))
if print_impact:
print(&#34;Before: &#34;)
idf.printSchema()
print(&#34;After: &#34;)
odf.printSchema()
return odf
```
</details>
## Functions
<dl>
<dt id="anovos.data_ingest.data_ingest.concatenate_dataset"><code class="name flex">
<span>def <span class="ident">concatenate_dataset</span></span>(<span>*idfs, method_type='name')</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>dfs</code></strong></dt>
<dd>All dataframes to be concatenated (with the first dataframe columns)</dd>
<dt><strong><code>method_type</code></strong></dt>
<dd>index", "name".</dd>
</dl>
<p>This argument needs to be passed as a keyword argument.
"index" method concatenates by column index positioning, without shuffling columns.
"name" concatenates after shuffling and arranging columns as per the first dataframe.
First dataframe passed under idfs will define the final columns in the concatenated dataframe,
and will throw error if any column in first dataframe is not available in any of other dataframes. (Default value = 'name')
*idfs: </p>
<h2 id="returns">Returns</h2>
<p>Concatenated Dataframe</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def concatenate_dataset(*idfs, method_type=&#39;name&#39;):
&#34;&#34;&#34;
Args:
dfs: All dataframes to be concatenated (with the first dataframe columns)
method_type: index&#34;, &#34;name&#34;.
This argument needs to be passed as a keyword argument.
&#34;index&#34; method concatenates by column index positioning, without shuffling columns.
&#34;name&#34; concatenates after shuffling and arranging columns as per the first dataframe.
First dataframe passed under idfs will define the final columns in the concatenated dataframe,
and will throw error if any column in first dataframe is not available in any of other dataframes. (Default value = &#39;name&#39;)
*idfs:
Returns:
Concatenated Dataframe
&#34;&#34;&#34;
if (method_type not in [&#39;index&#39;, &#39;name&#39;]):
raise TypeError(&#39;Invalid input for concatenate_dataset method&#39;)
if method_type == &#39;name&#39;:
odf = pairwise_reduce(lambda idf1, idf2: idf1.union(idf2.select(idf1.columns)), idfs)
# odf = reduce(DataFrame.unionByName, idfs) # only if exact no. of columns
else:
odf = pairwise_reduce(DataFrame.union, idfs)
return odf
```
</details>
</dd>
<dt id="anovos.data_ingest.data_ingest.delete_column"><code class="name flex">
<span>def <span class="ident">delete_column</span></span>(<span>idf, list_of_cols, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of columns to delete e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe after dropping columns</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def delete_column(idf, list_of_cols, print_impact=False):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
list_of_cols: List of columns to delete e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
print_impact:
(Default value = False)
Returns:
Dataframe after dropping columns
&#34;&#34;&#34;
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
list_of_cols = list(set(list_of_cols))
odf = idf.drop(*list_of_cols)
if print_impact:
print(&#34;Before: \nNo. of Columns- &#34;, len(idf.columns))
print(idf.columns)
print(&#34;After: \nNo. of Columns- &#34;, len(odf.columns))
print(odf.columns)
return odf
```
</details>
</dd>
<dt id="anovos.data_ingest.data_ingest.join_dataset"><code class="name flex">
<span>def <span class="ident">join_dataset</span></span>(<span>*idfs, join_cols, join_type)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>idfs</code></strong></dt>
<dd>All dataframes to be joined</dd>
<dt><strong><code>join_cols</code></strong></dt>
<dd>Key column(s) to join all dataframes together.</dd>
</dl>
<p>In case of multiple columns to join, they can be passed in a list format or
a string format where different column names are separated by pipe delimiter “|”.
join_type: inner", “full”, “left”, “right”, “left_semi”, “left_anti”
*idfs: </p>
<h2 id="returns">Returns</h2>
<p>Joined Dataframe</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def join_dataset(*idfs, join_cols, join_type):
&#34;&#34;&#34;
Args:
idfs: All dataframes to be joined
join_cols: Key column(s) to join all dataframes together.
In case of multiple columns to join, they can be passed in a list format or
a string format where different column names are separated by pipe delimiter “|”.
join_type: inner&#34;, “full”, “left”, “right”, “left_semi”, “left_anti”
*idfs:
Returns:
Joined Dataframe
&#34;&#34;&#34;
if isinstance(join_cols, str):
join_cols = [x.strip() for x in join_cols.split(&#39;|&#39;)]
odf = pairwise_reduce(lambda idf1, idf2: idf1.join(idf2, join_cols, join_type), idfs)
return odf
```
</details>
</dd>
<dt id="anovos.data_ingest.data_ingest.read_dataset"><code class="name flex">
<span>def <span class="ident">read_dataset</span></span>(<span>spark, file_path, file_type, file_configs={})</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>file_path</code></strong></dt>
<dd>Path to input data (directory or filename).</dd>
</dl>
<p>Compatible with local path and s3 path (when running in AWS environment).
file_type: csv", "parquet", "avro".
Avro data source requires an external package to run, which can be configured with
spark-submit (&ndash;packages org.apache.spark:spark-avro_2.11:2.4.0).
file_configs: This argument is passed in a dictionary format as key/value pairs
e.g. {"header": "True","delimiter": "|","inferSchema": "True"} for csv files.
All the key/value pairs in this argument are passed as options to DataFrameReader,
which is created using SparkSession.read. (Default value = {})</p>
<h2 id="returns">Returns</h2>
<p>Dataframe</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def read_dataset(spark, file_path, file_type, file_configs={}):
&#34;&#34;&#34;
Args:
spark: Spark Session
file_path: Path to input data (directory or filename).
Compatible with local path and s3 path (when running in AWS environment).
file_type: csv&#34;, &#34;parquet&#34;, &#34;avro&#34;.
Avro data source requires an external package to run, which can be configured with
spark-submit (--packages org.apache.spark:spark-avro_2.11:2.4.0).
file_configs: This argument is passed in a dictionary format as key/value pairs
e.g. {&#34;header&#34;: &#34;True&#34;,&#34;delimiter&#34;: &#34;|&#34;,&#34;inferSchema&#34;: &#34;True&#34;} for csv files.
All the key/value pairs in this argument are passed as options to DataFrameReader,
which is created using SparkSession.read. (Default value = {})
Returns:
Dataframe
&#34;&#34;&#34;
odf = spark.read.format(file_type).options(**file_configs).load(file_path)
return odf
```
</details>
</dd>
<dt id="anovos.data_ingest.data_ingest.recast_column"><code class="name flex">
<span>def <span class="ident">recast_column</span></span>(<span>idf, list_of_cols, list_of_dtypes, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of columns to cast e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
list_of_dtypes: List of corresponding datatypes e.g., ["type1","type2"].
Alternatively, datatypes can be specified in a string format,
where they are separated by pipe delimiter “|” e.g., "type1|type2".
First element in list_of_cols will column name and corresponding element in list_of_dtypes
will be new datatypes such as "float", "integer", "long", "string", "double", decimal" etc.
Datatypes are case insensitive e.g. float or Float are treated as same.
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe with revised datatypes</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def recast_column(idf, list_of_cols, list_of_dtypes, print_impact=False):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
list_of_cols: List of columns to cast e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
list_of_dtypes: List of corresponding datatypes e.g., [&#34;type1&#34;,&#34;type2&#34;].
Alternatively, datatypes can be specified in a string format,
where they are separated by pipe delimiter “|” e.g., &#34;type1|type2&#34;.
First element in list_of_cols will column name and corresponding element in list_of_dtypes
will be new datatypes such as &#34;float&#34;, &#34;integer&#34;, &#34;long&#34;, &#34;string&#34;, &#34;double&#34;, decimal&#34; etc.
Datatypes are case insensitive e.g. float or Float are treated as same.
print_impact:
(Default value = False)
Returns:
Dataframe with revised datatypes
&#34;&#34;&#34;
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(list_of_dtypes, str):
list_of_dtypes = [x.strip() for x in list_of_dtypes.split(&#39;|&#39;)]
odf = idf
for i, j in zip(list_of_cols, list_of_dtypes):
odf = odf.withColumn(i, F.col(i).cast(j))
if print_impact:
print(&#34;Before: &#34;)
idf.printSchema()
print(&#34;After: &#34;)
odf.printSchema()
return odf
```
</details>
</dd>
<dt id="anovos.data_ingest.data_ingest.rename_column"><code class="name flex">
<span>def <span class="ident">rename_column</span></span>(<span>idf, list_of_cols, list_of_newcols, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of old column names e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
list_of_newcols: List of corresponding new column names e.g., ["newcol1","newcol2"].
Alternatively, new column names can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "newcol1|newcol2".
First element in list_of_cols will be original column name,
and corresponding first column in list_of_newcols will be new column name.
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe with revised column names</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def rename_column(idf, list_of_cols, list_of_newcols, print_impact=False):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
list_of_cols: List of old column names e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
list_of_newcols: List of corresponding new column names e.g., [&#34;newcol1&#34;,&#34;newcol2&#34;].
Alternatively, new column names can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;newcol1|newcol2&#34;.
First element in list_of_cols will be original column name,
and corresponding first column in list_of_newcols will be new column name.
print_impact:
(Default value = False)
Returns:
Dataframe with revised column names
&#34;&#34;&#34;
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
if isinstance(list_of_newcols, str):
list_of_newcols = [x.strip() for x in list_of_newcols.split(&#39;|&#39;)]
mapping = dict(zip(list_of_cols, list_of_newcols))
odf = idf.select([F.col(i).alias(mapping.get(i, i)) for i in idf.columns])
if print_impact:
print(&#34;Before: \nNo. of Columns- &#34;, len(idf.columns))
print(idf.columns)
print(&#34;After: \nNo. of Columns- &#34;, len(odf.columns))
print(odf.columns)
return odf
```
</details>
</dd>
<dt id="anovos.data_ingest.data_ingest.select_column"><code class="name flex">
<span>def <span class="ident">select_column</span></span>(<span>idf, list_of_cols, print_impact=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>list_of_cols</code></strong></dt>
<dd>List of columns to select e.g., ["col1","col2"].</dd>
</dl>
<p>Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., "col1|col2".
print_impact:
(Default value = False)</p>
<h2 id="returns">Returns</h2>
<p>Dataframe with the selected columns</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def select_column(idf, list_of_cols, print_impact=False):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
list_of_cols: List of columns to select e.g., [&#34;col1&#34;,&#34;col2&#34;].
Alternatively, columns can be specified in a string format,
where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;.
print_impact:
(Default value = False)
Returns:
Dataframe with the selected columns
&#34;&#34;&#34;
if isinstance(list_of_cols, str):
list_of_cols = [x.strip() for x in list_of_cols.split(&#39;|&#39;)]
list_of_cols = list(set(list_of_cols))
odf = idf.select(list_of_cols)
if print_impact:
print(&#34;Before: \nNo. of Columns-&#34;, len(idf.columns))
print(idf.columns)
print(&#34;\nAfter: \nNo. of Columns-&#34;, len(odf.columns))
print(odf.columns)
return odf
```
</details>
</dd>
<dt id="anovos.data_ingest.data_ingest.write_dataset"><code class="name flex">
<span>def <span class="ident">write_dataset</span></span>(<span>idf, file_path, file_type, file_configs={}, column_order=[])</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>file_path</code></strong></dt>
<dd>Path to output data (directory or filename).</dd>
</dl>
<p>Compatible with local path and s3 path (when running in AWS environment).
file_type: csv", "parquet", "avro".
Avro data source requires an external package to run, which can be configured with
spark-submit (&ndash;packages org.apache.spark:spark-avro_2.11:2.4.0).
file_configs: This argument is passed in dictionary format as key/value pairs.
Some of the potential keys are header, delimiter, mode, compression, repartition.
compression options - uncompressed, gzip (doesn't work with avro), snappy (only valid for parquet)
mode options - error (default), overwrite, append
repartition - None (automatic partitioning) or an integer value ()
e.g. {"header":"True","delimiter":",",'compression':'snappy','mode':'overwrite','repartition':'10'}.
column_order: list of columns in the order in which Dataframe is to be written. If None or [] is specified, then the default order is applied.</p>
<h2 id="returns">Returns</h2>
<p>None (Dataframe saved)</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def write_dataset(idf, file_path, file_type, file_configs={}, column_order=[]):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
file_path: Path to output data (directory or filename).
Compatible with local path and s3 path (when running in AWS environment).
file_type: csv&#34;, &#34;parquet&#34;, &#34;avro&#34;.
Avro data source requires an external package to run, which can be configured with
spark-submit (--packages org.apache.spark:spark-avro_2.11:2.4.0).
file_configs: This argument is passed in dictionary format as key/value pairs.
Some of the potential keys are header, delimiter, mode, compression, repartition.
compression options - uncompressed, gzip (doesn&#39;t work with avro), snappy (only valid for parquet)
mode options - error (default), overwrite, append
repartition - None (automatic partitioning) or an integer value ()
e.g. {&#34;header&#34;:&#34;True&#34;,&#34;delimiter&#34;:&#34;,&#34;,&#39;compression&#39;:&#39;snappy&#39;,&#39;mode&#39;:&#39;overwrite&#39;,&#39;repartition&#39;:&#39;10&#39;}.
column_order: list of columns in the order in which Dataframe is to be written. If None or [] is specified, then the default order is applied.
Returns:
None (Dataframe saved)
&#34;&#34;&#34;
if not column_order:
column_order = idf.columns
else:
if not isinstance(column_order, list):
raise TypeError(&#39;Invalid input type for column_order argument&#39;)
if len(column_order) != len(idf.columns):
raise ValueError(&#39;Count of column(s) specified in column_order argument do not match Dataframe&#39;)
diff_cols = [x for x in column_order if x not in set(idf.columns)]
if diff_cols:
raise ValueError(&#39;Column(s) specified in column_order argument not found in Dataframe: &#39; + str(diff_cols))
mode = file_configs[&#39;mode&#39;] if &#39;mode&#39; in file_configs else &#39;error&#39;
repartition = int(file_configs[&#39;repartition&#39;]) if &#39;repartition&#39; in file_configs else None
if repartition is None:
idf.select(column_order).write.format(file_type).options(**file_configs).save(file_path, mode=mode)
else:
exist_parts = idf.rdd.getNumPartitions()
req_parts = int(repartition)
if req_parts &gt; exist_parts:
idf.select(column_order).repartition(req_parts).write.format(file_type).options(**file_configs).save(file_path, mode=mode)
else:
idf.select(column_order).coalesce(req_parts).write.format(file_type).options(**file_configs).save(file_path, mode=mode)
```
</details>
</dd>
</dl>