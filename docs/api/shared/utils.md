# <code>utils</code>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
from itertools import chain

from pyspark.sql import functions as F


def flatten_dataframe(idf, fixed_cols):
    """

    Parameters
    ----------
    idf
        Input Dataframe
    fixed_cols
        All columns except in this list will be melted/unpivoted

    Returns
    -------

    """
    valid_cols = [e for e in idf.columns if e not in fixed_cols]
    key_and_val = F.create_map(
        list(chain.from_iterable([[F.lit(c), F.col(c)] for c in valid_cols]))
    )
    odf = idf.select(*fixed_cols, F.explode(key_and_val))
    return odf


def transpose_dataframe(idf, fixed_col):
    """

    Parameters
    ----------
    idf
        Input Dataframe
    fixed_col
        Values in this column will be converted into columns as header.
        Ideally all values should be unique

    Returns
    -------

    """
    idf_flatten = flatten_dataframe(idf, fixed_cols=[fixed_col])
    odf = idf_flatten.groupBy("key").pivot(fixed_col).agg(F.first("value"))
    return odf


def attributeType_segregation(idf):
    """

    Parameters
    ----------
    idf
        Input Dataframe

    Returns
    -------

    """
    cat_cols = []
    num_cols = []
    other_cols = []

    for i in idf.dtypes:
        if i[1] == "string":
            cat_cols.append(i[0])
        elif (i[1] in ("double", "int", "bigint", "float", "long")) | (
            i[1].startswith("decimal")
        ):
            num_cols.append(i[0])
        else:
            other_cols.append(i[0])
    return num_cols, cat_cols, other_cols


def get_dtype(idf, col):
    """

    Parameters
    ----------
    idf
        Input Dataframe
    col
        Column Name for datatype detection

    Returns
    -------

    """
    return [dtype for name, dtype in idf.dtypes if name == col][0]


def ends_with(string, end_str="/"):
    """

    Parameters
    ----------
    string
        "s3:mw-bucket"
    end_str
        return: "s3:mw-bucket/" (Default value = "/")

    Returns
    -------

    """
    string = str(string)
    if string.endswith(end_str):
        return string
    return string + end_str


def pairwise_reduce(op, x):
    """

    Parameters
    ----------
    op
        Operation
    x
        Input list

    Returns
    -------

    """
    while len(x) > 1:
        v = [op(i, j) for i, j in zip(x[::2], x[1::2])]
        if len(x) > 1 and len(x) % 2 == 1:
            v[-1] = op(v[-1], x[-1])
        x = v
    return x[0]


def output_to_local(output_path):
    """

    Parameters
    ----------
    output_path :
        input_path. e.g. dbfs:/sample_path

    Returns
    -------
    type
        path after removing ":" and appending "/" . e.g. /dbfs/sample_path

    """
    punctuations = ":"
    for x in output_path:
        if x in punctuations:
            local_path = output_path.replace(x, "")
            local_path = "/" + local_path
    return local_path
```
</pre>
</details>
## Functions
<dl>
<dt id="anovos.shared.utils.attributeType_segregation"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">attributeType_segregation</span></span>(<span class="n">idf)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def attributeType_segregation(idf):
    """

    Parameters
    ----------
    idf
        Input Dataframe

    Returns
    -------

    """
    cat_cols = []
    num_cols = []
    other_cols = []

    for i in idf.dtypes:
        if i[1] == "string":
            cat_cols.append(i[0])
        elif (i[1] in ("double", "int", "bigint", "float", "long")) | (
            i[1].startswith("decimal")
        ):
            num_cols.append(i[0])
        else:
            other_cols.append(i[0])
    return num_cols, cat_cols, other_cols
```
</pre>
</details>
</dd>
<dt id="anovos.shared.utils.ends_with"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">ends_with</span></span>(<span class="n">string, end_str='/')</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>string</code></strong></dt>
<dd>"s3:mw-bucket"</dd>
<dt><strong><code>end_str</code></strong></dt>
<dd>return: "s3:mw-bucket/" (Default value = "/")</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def ends_with(string, end_str="/"):
    """

    Parameters
    ----------
    string
        "s3:mw-bucket"
    end_str
        return: "s3:mw-bucket/" (Default value = "/")

    Returns
    -------

    """
    string = str(string)
    if string.endswith(end_str):
        return string
    return string + end_str
```
</pre>
</details>
</dd>
<dt id="anovos.shared.utils.flatten_dataframe"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">flatten_dataframe</span></span>(<span class="n">idf, fixed_cols)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>fixed_cols</code></strong></dt>
<dd>All columns except in this list will be melted/unpivoted</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def flatten_dataframe(idf, fixed_cols):
    """

    Parameters
    ----------
    idf
        Input Dataframe
    fixed_cols
        All columns except in this list will be melted/unpivoted

    Returns
    -------

    """
    valid_cols = [e for e in idf.columns if e not in fixed_cols]
    key_and_val = F.create_map(
        list(chain.from_iterable([[F.lit(c), F.col(c)] for c in valid_cols]))
    )
    odf = idf.select(*fixed_cols, F.explode(key_and_val))
    return odf
```
</pre>
</details>
</dd>
<dt id="anovos.shared.utils.get_dtype"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">get_dtype</span></span>(<span class="n">idf, col)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>col</code></strong></dt>
<dd>Column Name for datatype detection</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def get_dtype(idf, col):
    """

    Parameters
    ----------
    idf
        Input Dataframe
    col
        Column Name for datatype detection

    Returns
    -------

    """
    return [dtype for name, dtype in idf.dtypes if name == col][0]
```
</pre>
</details>
</dd>
<dt id="anovos.shared.utils.output_to_local"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">output_to_local</span></span>(<span class="n">output_path)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<p>output_path :
input_path. e.g. dbfs:/sample_path</p>
<h2 id="returns">Returns</h2>
<dl>
<dt><code>type</code></dt>
<dd>path after removing ":" and appending "/" . e.g. /dbfs/sample_path</dd>
</dl></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def output_to_local(output_path):
    """

    Parameters
    ----------
    output_path :
        input_path. e.g. dbfs:/sample_path

    Returns
    -------
    type
        path after removing ":" and appending "/" . e.g. /dbfs/sample_path

    """
    punctuations = ":"
    for x in output_path:
        if x in punctuations:
            local_path = output_path.replace(x, "")
            local_path = "/" + local_path
    return local_path
```
</pre>
</details>
</dd>
<dt id="anovos.shared.utils.pairwise_reduce"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">pairwise_reduce</span></span>(<span class="n">op, x)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>op</code></strong></dt>
<dd>Operation</dd>
<dt><strong><code>x</code></strong></dt>
<dd>Input list</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def pairwise_reduce(op, x):
    """

    Parameters
    ----------
    op
        Operation
    x
        Input list

    Returns
    -------

    """
    while len(x) > 1:
        v = [op(i, j) for i, j in zip(x[::2], x[1::2])]
        if len(x) > 1 and len(x) % 2 == 1:
            v[-1] = op(v[-1], x[-1])
        x = v
    return x[0]
```
</pre>
</details>
</dd>
<dt id="anovos.shared.utils.transpose_dataframe"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">transpose_dataframe</span></span>(<span class="n">idf, fixed_col)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>fixed_col</code></strong></dt>
<dd>Values in this column will be converted into columns as header.
Ideally all values should be unique</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def transpose_dataframe(idf, fixed_col):
    """

    Parameters
    ----------
    idf
        Input Dataframe
    fixed_col
        Values in this column will be converted into columns as header.
        Ideally all values should be unique

    Returns
    -------

    """
    idf_flatten = flatten_dataframe(idf, fixed_cols=[fixed_col])
    odf = idf_flatten.groupBy("key").pivot(fixed_col).agg(F.first("value"))
    return odf
```
</pre>
</details>
</dd>
</dl>