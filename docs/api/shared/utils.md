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
    idf :
        Input Dataframe
    fixed_cols :
        All columns except in this list will be melted/unpivoted

    Returns
    -------
    type
        Flatten/Melted dataframe

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
    idf :
        Input Dataframe
    fixed_col :
        Values in this column will be converted into columns as header.
        Ideally all values should be unique

    Returns
    -------
    type
        Transposed dataframe

    """
    idf_flatten = flatten_dataframe(idf, fixed_cols=[fixed_col])
    odf = idf_flatten.groupBy("key").pivot(fixed_col).agg(F.first("value"))
    return odf


def attributeType_segregation(idf):
    """

    Parameters
    ----------
    idf :
        Input Dataframe

    Returns
    -------
    type
        list1, list2, list3)
        3 lists - each corresponding to numerical, categorical, and others columns

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
    idf :
        Input Dataframe
    col :
        Column Name for datatype detection

    Returns
    -------
    type
        data type

    """
    return [dtype for name, dtype in idf.dtypes if name == col][0]


def ends_with(string, end_str="/"):
    """

    Parameters
    ----------
    string :
        s3:mw-bucket"
    end_str :
        return: "s3:mw-bucket/" (Default value = "/")

    Returns
    -------
    type
        s3:mw-bucket/"

    """
    string = str(string)
    if string.endswith(end_str):
        return string
    return string + end_str


def pairwise_reduce(op, x):
    """

    Parameters
    ----------
    op :

    x :


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
## Functions
<dl>
<dt id="anovos.shared.utils.attributeType_segregation"><code class="name flex">
<span>def <span class="ident">attributeType_segregation</span></span>(<span>idf)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<p>idf :
Input Dataframe</p>
<h2 id="returns">Returns</h2>
<dl>
<dt><code>type</code></dt>
<dd>list1, list2, list3)
3 lists - each corresponding to numerical, categorical, and others columns</dd>
</dl></div>
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
    idf :
        Input Dataframe

    Returns
    -------
    type
        list1, list2, list3)
        3 lists - each corresponding to numerical, categorical, and others columns

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
<dt id="anovos.shared.utils.ends_with"><code class="name flex">
<span>def <span class="ident">ends_with</span></span>(<span>string, end_str='/')</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<p>string :
s3:mw-bucket"
end_str :
return: "s3:mw-bucket/" (Default value = "/")</p>
<h2 id="returns">Returns</h2>
<dl>
<dt><code>type</code></dt>
<dd>s3:mw-bucket/"</dd>
</dl></div>
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
    string :
        s3:mw-bucket"
    end_str :
        return: "s3:mw-bucket/" (Default value = "/")

    Returns
    -------
    type
        s3:mw-bucket/"

    """
    string = str(string)
    if string.endswith(end_str):
        return string
    return string + end_str
```
</pre>
</details>
</dd>
<dt id="anovos.shared.utils.flatten_dataframe"><code class="name flex">
<span>def <span class="ident">flatten_dataframe</span></span>(<span>idf, fixed_cols)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<p>idf :
Input Dataframe
fixed_cols :
All columns except in this list will be melted/unpivoted</p>
<h2 id="returns">Returns</h2>
<dl>
<dt><code>type</code></dt>
<dd>Flatten/Melted dataframe</dd>
</dl></div>
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
    idf :
        Input Dataframe
    fixed_cols :
        All columns except in this list will be melted/unpivoted

    Returns
    -------
    type
        Flatten/Melted dataframe

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
<dt id="anovos.shared.utils.get_dtype"><code class="name flex">
<span>def <span class="ident">get_dtype</span></span>(<span>idf, col)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<p>idf :
Input Dataframe
col :
Column Name for datatype detection</p>
<h2 id="returns">Returns</h2>
<dl>
<dt><code>type</code></dt>
<dd>data type</dd>
</dl></div>
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
    idf :
        Input Dataframe
    col :
        Column Name for datatype detection

    Returns
    -------
    type
        data type

    """
    return [dtype for name, dtype in idf.dtypes if name == col][0]
```
</pre>
</details>
</dd>
<dt id="anovos.shared.utils.pairwise_reduce"><code class="name flex">
<span>def <span class="ident">pairwise_reduce</span></span>(<span>op, x)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<p>op :</p>
<p>x :</p>
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
    op :

    x :


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
<dt id="anovos.shared.utils.transpose_dataframe"><code class="name flex">
<span>def <span class="ident">transpose_dataframe</span></span>(<span>idf, fixed_col)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<p>idf :
Input Dataframe
fixed_col :
Values in this column will be converted into columns as header.
Ideally all values should be unique</p>
<h2 id="returns">Returns</h2>
<dl>
<dt><code>type</code></dt>
<dd>Transposed dataframe</dd>
</dl></div>
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
    idf :
        Input Dataframe
    fixed_col :
        Values in this column will be converted into columns as header.
        Ideally all values should be unique

    Returns
    -------
    type
        Transposed dataframe

    """
    idf_flatten = flatten_dataframe(idf, fixed_cols=[fixed_col])
    odf = idf_flatten.groupBy("key").pivot(fixed_col).agg(F.first("value"))
    return odf
```
</pre>
</details>
</dd>
</dl>