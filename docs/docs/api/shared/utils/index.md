<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, minimum-scale=1" />
<meta name="generator" content="pdoc 0.10.0" />
<title>anovos.shared.utils API documentation</title>
<meta name="description" content="" />
<link rel="preload stylesheet" as="style" href="https://cdnjs.cloudflare.com/ajax/libs/10up-sanitize.css/11.0.1/sanitize.min.css" integrity="sha256-PK9q560IAAa6WVRRh76LtCaI8pjTJ2z11v0miyNNjrs=" crossorigin>
<link rel="preload stylesheet" as="style" href="https://cdnjs.cloudflare.com/ajax/libs/10up-sanitize.css/11.0.1/typography.min.css" integrity="sha256-7l/o7C8jubJiy74VsKTidCy1yBkRtiUGbVkYBylBqUg=" crossorigin>
<link rel="stylesheet preload" as="style" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/10.1.1/styles/github.min.css" crossorigin>
<style>:root{--highlight-color:#fe9}.flex{display:flex !important}body{line-height:1.5em}#content{padding:20px}#sidebar{padding:30px;overflow:hidden}#sidebar > *:last-child{margin-bottom:2cm}.http-server-breadcrumbs{font-size:130%;margin:0 0 15px 0}#footer{font-size:.75em;padding:5px 30px;border-top:1px solid #ddd;text-align:right}#footer p{margin:0 0 0 1em;display:inline-block}#footer p:last-child{margin-right:30px}h1,h2,h3,h4,h5{font-weight:300}h1{font-size:2.5em;line-height:1.1em}h2{font-size:1.75em;margin:1em 0 .50em 0}h3{font-size:1.4em;margin:25px 0 10px 0}h4{margin:0;font-size:105%}h1:target,h2:target,h3:target,h4:target,h5:target,h6:target{background:var(--highlight-color);padding:.2em 0}a{color:#058;text-decoration:none;transition:color .3s ease-in-out}a:hover{color:#e82}.title code{font-weight:bold}h2[id^="header-"]{margin-top:2em}.ident{color:#900}pre code{background:#f8f8f8;font-size:.8em;line-height:1.4em}code{background:#f2f2f1;padding:1px 4px;overflow-wrap:break-word}h1 code{background:transparent}pre{background:#f8f8f8;border:0;border-top:1px solid #ccc;border-bottom:1px solid #ccc;margin:1em 0;padding:1ex}#http-server-module-list{display:flex;flex-flow:column}#http-server-module-list div{display:flex}#http-server-module-list dt{min-width:10%}#http-server-module-list p{margin-top:0}.toc ul,#index{list-style-type:none;margin:0;padding:0}#index code{background:transparent}#index h3{border-bottom:1px solid #ddd}#index ul{padding:0}#index h4{margin-top:.6em;font-weight:bold}@media (min-width:200ex){#index .two-column{column-count:2}}@media (min-width:300ex){#index .two-column{column-count:3}}dl{margin-bottom:2em}dl dl:last-child{margin-bottom:4em}dd{margin:0 0 1em 3em}#header-classes + dl > dd{margin-bottom:3em}dd dd{margin-left:2em}dd p{margin:10px 0}.name{background:#eee;font-weight:bold;font-size:.85em;padding:5px 10px;display:inline-block;min-width:40%}.name:hover{background:#e0e0e0}dt:target .name{background:var(--highlight-color)}.name > span:first-child{white-space:nowrap}.name.class > span:nth-child(2){margin-left:.4em}.inherited{color:#999;border-left:5px solid #eee;padding-left:1em}.inheritance em{font-style:normal;font-weight:bold}.desc h2{font-weight:400;font-size:1.25em}.desc h3{font-size:1em}.desc dt code{background:inherit}.source summary,.git-link-div{color:#666;text-align:right;font-weight:400;font-size:.8em;text-transform:uppercase}.source summary > *{white-space:nowrap;cursor:pointer}.git-link{color:inherit;margin-left:1em}.source pre{max-height:500px;overflow:auto;margin:0}.source pre code{font-size:12px;overflow:visible}.hlist{list-style:none}.hlist li{display:inline}.hlist li:after{content:',\2002'}.hlist li:last-child:after{content:none}.hlist .hlist{display:inline;padding-left:1em}img{max-width:100%}td{padding:0 .5em}.admonition{padding:.1em .5em;margin-bottom:1em}.admonition-title{font-weight:bold}.admonition.note,.admonition.info,.admonition.important{background:#aef}.admonition.todo,.admonition.versionadded,.admonition.tip,.admonition.hint{background:#dfd}.admonition.warning,.admonition.versionchanged,.admonition.deprecated{background:#fd4}.admonition.error,.admonition.danger,.admonition.caution{background:lightpink}</style>
<style media="screen and (min-width: 700px)">@media screen and (min-width:700px){#sidebar{width:30%;height:100vh;overflow:auto;position:sticky;top:0}#content{width:70%;max-width:100ch;padding:3em 4em;border-left:1px solid #ddd}pre code{font-size:1em}.item .name{font-size:1em}main{display:flex;flex-direction:row-reverse;justify-content:flex-end}.toc ul ul,#index ul{padding-left:1.5em}.toc > ul > li{margin-top:.5em}}</style>
<style media="print">@media print{#sidebar h1{page-break-before:always}.source{display:none}}@media print{*{background:transparent !important;color:#000 !important;box-shadow:none !important;text-shadow:none !important}a[href]:after{content:" (" attr(href) ")";font-size:90%}a[href][title]:after{content:none}abbr[title]:after{content:" (" attr(title) ")"}.ir a:after,a[href^="javascript:"]:after,a[href^="#"]:after{content:""}pre,blockquote{border:1px solid #999;page-break-inside:avoid}thead{display:table-header-group}tr,img{page-break-inside:avoid}img{max-width:100% !important}@page{margin:0.5cm}p,h2,h3{orphans:3;widows:3}h1,h2,h3,h4,h5,h6{page-break-after:avoid}}</style>
<script defer src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/10.1.1/highlight.min.js" integrity="sha256-Uv3H6lx7dJmRfRvH8TH6kJD1TSK1aFcwgx+mdg3epi8=" crossorigin></script>
<script>window.addEventListener('DOMContentLoaded', () => hljs.initHighlighting())</script>
</head>
<body>
<main>
<article id="content">
<header>
<h1 class="title">Module <code>anovos.shared.utils</code></h1>
</header>
<section id="section-intro">
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre><code class="python">from itertools import chain

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
      fixed_col: Values in this column will be converted into columns as header.&lt;br&gt;\
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
      (list1, list2, list3) &lt;br&gt;\
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
    return x[0]</code></pre>
</details>
</section>
<section>
</section>
<section>
</section>
<section>
<h2 class="section-title" id="header-functions">Functions</h2>
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
<p>(list1, list2, list3) <br>
3 lists - each corresponding to numerical, categorical, and others columns</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre><code class="python">def attributeType_segregation(idf):
    &#34;&#34;&#34;

    Args:
      idf: Input Dataframe

    Returns:
      (list1, list2, list3) &lt;br&gt;\
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
    return num_cols, cat_cols, other_cols</code></pre>
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
<pre><code class="python">def ends_with(string, end_str=&#34;/&#34;):
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
    return string + end_str</code></pre>
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
<pre><code class="python">def flatten_dataframe(idf, fixed_cols):
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
    return odf</code></pre>
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
<pre><code class="python">def get_dtype(idf, col):
    &#34;&#34;&#34;

    Args:
      idf: Input Dataframe
      col: Column Name for datatype detection

    Returns:
      data type

    &#34;&#34;&#34;
    return [dtype for name, dtype in idf.dtypes if name == col][0]</code></pre>
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
<pre><code class="python">def pairwise_reduce(op, x):
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
    return x[0]</code></pre>
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
<dd>Values in this column will be converted into columns as header.<br>
Ideally all values should be unique</dd>
</dl>
<h2 id="returns">Returns</h2>
<p>Transposed dataframe</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre><code class="python">def transpose_dataframe(idf, fixed_col):
    &#34;&#34;&#34;

    Args:
      idf: Input Dataframe
      fixed_col: Values in this column will be converted into columns as header.&lt;br&gt;\
    Ideally all values should be unique

    Returns:
      Transposed dataframe

    &#34;&#34;&#34;
    idf_flatten = flatten_dataframe(idf, fixed_cols=[fixed_col])
    odf = idf_flatten.groupBy(&#39;key&#39;).pivot(fixed_col).agg(F.first(&#39;value&#39;))
    return odf</code></pre>
</details>
</dd>
</dl>
</section>
<section>
</section>
</article>
<nav id="sidebar">
<h1>Index</h1>
<div class="toc">
<ul></ul>
</div>
<ul id="index">
<li><h3>Super-module</h3>
<ul>
<li><code><a title="anovos.shared" href="index.html">anovos.shared</a></code></li>
</ul>
</li>
<li><h3><a href="#header-functions">Functions</a></h3>
<ul class="">
<li><code><a title="anovos.shared.utils.attributeType_segregation" href="#anovos.shared.utils.attributeType_segregation">attributeType_segregation</a></code></li>
<li><code><a title="anovos.shared.utils.ends_with" href="#anovos.shared.utils.ends_with">ends_with</a></code></li>
<li><code><a title="anovos.shared.utils.flatten_dataframe" href="#anovos.shared.utils.flatten_dataframe">flatten_dataframe</a></code></li>
<li><code><a title="anovos.shared.utils.get_dtype" href="#anovos.shared.utils.get_dtype">get_dtype</a></code></li>
<li><code><a title="anovos.shared.utils.pairwise_reduce" href="#anovos.shared.utils.pairwise_reduce">pairwise_reduce</a></code></li>
<li><code><a title="anovos.shared.utils.transpose_dataframe" href="#anovos.shared.utils.transpose_dataframe">transpose_dataframe</a></code></li>
</ul>
</li>
</ul>
</nav>
</main>
<footer id="footer">
<p>Generated by <a href="https://pdoc3.github.io/pdoc" title="pdoc: Python API documentation generator"><cite>pdoc</cite> 0.10.0</a>.</p>
</footer>
</body>
</html>
