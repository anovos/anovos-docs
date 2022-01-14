# Module <code>utils</code>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
from itertools import chain
from pyspark.sql import functions as F
def flatten_dataframe(idf, fixed_cols):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
fixed_cols: All columns except in this list will be melted/unpivoted
Returns:
Flatten/Melted dataframe
&#34;&#34;&#34;
valid_cols = [e for e in idf.columns if e not in fixed_cols]
key_and_val = F.create_map(list(chain.from_iterable([[F.lit(c), F.col(c)] for c in valid_cols])))
odf = idf.select(*fixed_cols, F.explode(key_and_val))
return odf
def transpose_dataframe(idf, fixed_col):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
fixed_col: Values in this column will be converted into columns as header.
Ideally all values should be unique
Returns:
Transposed dataframe
&#34;&#34;&#34;
idf_flatten = flatten_dataframe(idf, fixed_cols=[fixed_col])
odf = idf_flatten.groupBy(&#39;key&#39;).pivot(fixed_col).agg(F.first(&#39;value&#39;))
return odf
def attributeType_segregation(idf):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
Returns:
list1, list2, list3)
3 lists - each corresponding to numerical, categorical, and others columns
&#34;&#34;&#34;
cat_cols = []
num_cols = []
other_cols = []
for i in idf.dtypes:
if i[1] == &#39;string&#39;:
cat_cols.append(i[0])
elif (i[1] in (&#39;double&#39;, &#39;int&#39;, &#39;bigint&#39;, &#39;float&#39;, &#39;long&#39;)) | (i[1].startswith(&#39;decimal&#39;)):
num_cols.append(i[0])
else:
other_cols.append(i[0])
return num_cols, cat_cols, other_cols
def get_dtype(idf, col):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
col: Column Name for datatype detection
Returns:
data type
&#34;&#34;&#34;
return [dtype for name, dtype in idf.dtypes if name == col][0]
def ends_with(string, end_str=&#34;/&#34;):
&#34;&#34;&#34;
Args:
string: s3:mw-bucket&#34;
end_str: return: &#34;s3:mw-bucket/&#34; (Default value = &#34;/&#34;)
Returns:
s3:mw-bucket/&#34;
&#34;&#34;&#34;
string = str(string)
if string.endswith(end_str):
return string
return string + end_str
def pairwise_reduce(op, x):
&#34;&#34;&#34;
Args:
op:
x:
Returns:
&#34;&#34;&#34;
while len(x) &gt; 1:
v = [op(i, j) for i, j in zip(x[::2], x[1::2])]
if len(x) &gt; 1 and len(x) % 2 == 1:
v[-1] = op(v[-1], x[-1])
x = v
return x[0]
```
</details>
## Functions
<dl>
<dt id="anovos.shared.utils.attributeType_segregation"><code class="name flex">
<span>def <span class="ident">attributeType_segregation</span></span>(<span>idf)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
</dl>
<h2 id="returns">Returns</h2>
<p>list1, list2, list3)
3 lists - each corresponding to numerical, categorical, and others columns</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def attributeType_segregation(idf):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
Returns:
list1, list2, list3)
3 lists - each corresponding to numerical, categorical, and others columns
&#34;&#34;&#34;
cat_cols = []
num_cols = []
other_cols = []
for i in idf.dtypes:
if i[1] == &#39;string&#39;:
cat_cols.append(i[0])
elif (i[1] in (&#39;double&#39;, &#39;int&#39;, &#39;bigint&#39;, &#39;float&#39;, &#39;long&#39;)) | (i[1].startswith(&#39;decimal&#39;)):
num_cols.append(i[0])
else:
other_cols.append(i[0])
return num_cols, cat_cols, other_cols
```
</details>
</dd>
<dt id="anovos.shared.utils.ends_with"><code class="name flex">
<span>def <span class="ident">ends_with</span></span>(<span>string, end_str='/')</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>string</code></strong></dt>
<dd>s3:mw-bucket"</dd>
<dt><strong><code>end_str</code></strong></dt>
<dd>return: "s3:mw-bucket/" (Default value = "/")</dd>
</dl>
<h2 id="returns">Returns</h2>
<p>s3:mw-bucket/"</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def ends_with(string, end_str=&#34;/&#34;):
&#34;&#34;&#34;
Args:
string: s3:mw-bucket&#34;
end_str: return: &#34;s3:mw-bucket/&#34; (Default value = &#34;/&#34;)
Returns:
s3:mw-bucket/&#34;
&#34;&#34;&#34;
string = str(string)
if string.endswith(end_str):
return string
return string + end_str
```
</details>
</dd>
<dt id="anovos.shared.utils.flatten_dataframe"><code class="name flex">
<span>def <span class="ident">flatten_dataframe</span></span>(<span>idf, fixed_cols)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>fixed_cols</code></strong></dt>
<dd>All columns except in this list will be melted/unpivoted</dd>
</dl>
<h2 id="returns">Returns</h2>
<p>Flatten/Melted dataframe</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def flatten_dataframe(idf, fixed_cols):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
fixed_cols: All columns except in this list will be melted/unpivoted
Returns:
Flatten/Melted dataframe
&#34;&#34;&#34;
valid_cols = [e for e in idf.columns if e not in fixed_cols]
key_and_val = F.create_map(list(chain.from_iterable([[F.lit(c), F.col(c)] for c in valid_cols])))
odf = idf.select(*fixed_cols, F.explode(key_and_val))
return odf
```
</details>
</dd>
<dt id="anovos.shared.utils.get_dtype"><code class="name flex">
<span>def <span class="ident">get_dtype</span></span>(<span>idf, col)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>col</code></strong></dt>
<dd>Column Name for datatype detection</dd>
</dl>
<h2 id="returns">Returns</h2>
<p>data type</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def get_dtype(idf, col):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
col: Column Name for datatype detection
Returns:
data type
&#34;&#34;&#34;
return [dtype for name, dtype in idf.dtypes if name == col][0]
```
</details>
</dd>
<dt id="anovos.shared.utils.pairwise_reduce"><code class="name flex">
<span>def <span class="ident">pairwise_reduce</span></span>(<span>op, x)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>op</code></strong></dt>
<dd>&nbsp;</dd>
<dt><strong><code>x</code></strong></dt>
<dd>&nbsp;</dd>
</dl>
<p>Returns:</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def pairwise_reduce(op, x):
&#34;&#34;&#34;
Args:
op:
x:
Returns:
&#34;&#34;&#34;
while len(x) &gt; 1:
v = [op(i, j) for i, j in zip(x[::2], x[1::2])]
if len(x) &gt; 1 and len(x) % 2 == 1:
v[-1] = op(v[-1], x[-1])
x = v
return x[0]
```
</details>
</dd>
<dt id="anovos.shared.utils.transpose_dataframe"><code class="name flex">
<span>def <span class="ident">transpose_dataframe</span></span>(<span>idf, fixed_col)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>fixed_col</code></strong></dt>
<dd>Values in this column will be converted into columns as header.</dd>
</dl>
<p>Ideally all values should be unique</p>
<h2 id="returns">Returns</h2>
<p>Transposed dataframe</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def transpose_dataframe(idf, fixed_col):
&#34;&#34;&#34;
Args:
idf: Input Dataframe
fixed_col: Values in this column will be converted into columns as header.
Ideally all values should be unique
Returns:
Transposed dataframe
&#34;&#34;&#34;
idf_flatten = flatten_dataframe(idf, fixed_cols=[fixed_col])
odf = idf_flatten.groupBy(&#39;key&#39;).pivot(fixed_col).agg(F.first(&#39;value&#39;))
return odf
```
</details>
</dd>
</dl>