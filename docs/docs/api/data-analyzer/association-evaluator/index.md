!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, minimum-scale=1" />
<meta name="generator" content="pdoc 0.10.0" />
<title>anovos.data_analyzer.association_evaluator API documentation</title>
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
<h1 class="title">Module <code>anovos.data_analyzer.association_evaluator</code></h1>
</header>
<section id="section-intro">
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre><code class="python"># coding=utf-8
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
      list_of_cols: List of columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;]. &lt;br&gt;\
    Alternatively, columns can be specified in a string format, &lt;br&gt;\
    where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. &lt;br&gt;\
    &#34;all&#34; can be passed to include all columns for analysis. &lt;br&gt;\
    Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in &lt;br&gt;\
    drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;) 
      drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;]. &lt;br&gt;\
    Alternatively, columns can be specified in a string format, &lt;br&gt;\
    where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
      stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format &lt;br&gt;\
    to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or &lt;br&gt;\
    uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
      print_impact:  (Default value = False)

    Returns:
      Dataframe [attribute,*col_names] &lt;br&gt;\
      Correlation between attribute X and Y can be found at an intersection of &lt;br&gt;\
      a) row with value X in ‘attribute’ column and column ‘Y’, or &lt;br&gt;\
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
    odf = spark.createDataFrame(odf_pd)

    if print_impact:
        odf.show(odf.count())

    return odf


def variable_clustering(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], sample_size=100000, stats_unique={}, stats_mode={},
                        print_impact=False):
    &#34;&#34;&#34;

    Args:
      spark: Spark Session
      idf: Input Dataframe
      list_of_cols: List of columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;]. &lt;br&gt;\
    Alternatively, columns can be specified in a string format, &lt;br&gt;\
    where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. &lt;br&gt;\
    &#34;all&#34; can be passed to include all columns for analysis. &lt;br&gt;\
    Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in &lt;br&gt;\
    drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
      drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;]. &lt;br&gt;\
    Alternatively, columns can be specified in a string format, &lt;br&gt;\
    where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
      sample_size: Maximum sample size (in terms of number of rows) taken for the computation. &lt;br&gt;\
    Sample dataset is extracted using random sampling. (Default value = 100000) 
      stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format &lt;br&gt;\
    to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or &lt;br&gt;\
    uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
      stats_mode: Takes arguments for read_dataset (data_ingest module) function in a dictionary format &lt;br&gt;\
    to read pre-saved statistics on most frequently seen values i.e. if measures_of_centralTendency or &lt;br&gt;\
    mode_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
      print_impact:  (Default value = False)

    Returns:
      Dataframe [Cluster, Attribute, RS_Ratio] &lt;br&gt;\
      Attributes similar to each other are grouped together with the same cluster id. &lt;br&gt;\
      Attribute with the lowest (1 — RS_Ratio) can be chosen as a representative of the cluster. &lt;br&gt;\

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
      list_of_cols: List of columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;]. &lt;br&gt;\
    Alternatively, columns can be specified in a string format, &lt;br&gt;\
    where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. &lt;br&gt;\
    &#34;all&#34; can be passed to include all columns for analysis. &lt;br&gt;\
    Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in &lt;br&gt;\
    drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
      drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;]. &lt;br&gt;\
    Alternatively, columns can be specified in a string format, &lt;br&gt;\
    where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
      label_col: Label/Target column (Default value = &#39;label&#39;)
      event_label: Value of (positive) event (i.e label 1) (Default value = 1)
      encoding_configs: Takes input in dictionary format. Default {} i.e. empty dict means no encoding is required. &lt;br&gt;\
    In case numerical columns are present and encoding is required, following keys shall be &lt;br&gt;\
    provided - &#34;bin_size&#34; i.e. no. of bins for converting the numerical columns to categorical, &lt;br&gt;\
    &#34;bin_method&#34; i.e. method of binning - &#34;equal_frequency&#34; or &#34;equal_range&#34; and &lt;br&gt;\
    &#34;monotonicity_check&#34; 1 for monotonic binning else 0. monotonicity_check of 1 will &lt;br&gt;\
    dynamically calculate the bin_size ensuring monotonic nature but can be expensive operation.
      print_impact:  (Default value = False)

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
      list_of_cols: List of columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;]. &lt;br&gt;\
    Alternatively, columns can be specified in a string format, &lt;br&gt;\
    where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. &lt;br&gt;\
    &#34;all&#34; can be passed to include all columns for analysis. &lt;br&gt;\
    Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in &lt;br&gt;\
    drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
      drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;]. &lt;br&gt;\
    Alternatively, columns can be specified in a string format, &lt;br&gt;\
    where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
      label_col: Label/Target column (Default value = &#39;label&#39;)
      event_label: Value of (positive) event (i.e label 1) (Default value = 1)
      encoding_configs: Takes input in dictionary format. Default {} i.e. empty dict means no encoding is required. &lt;br&gt;\
    In case numerical columns are present and encoding is required, following keys shall be &lt;br&gt;\
    provided - &#34;bin_size&#34; i.e. no. of bins for converting the numerical columns to categorical, &lt;br&gt;\
    &#34;bin_method&#34; i.e. method of binning - &#34;equal_frequency&#34; or &#34;equal_range&#34; and &lt;br&gt;\
    &#34;monotonicity_check&#34; 1 for monotonic binning else 0. monotonicity_check of 1 will &lt;br&gt;\
    dynamically calculate the bin_size ensuring monotonic nature but can be expensive operation.
      print_impact:  (Default value = False)

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

    return odf</code></pre>
</details>
</section>
<section>
</section>
<section>
</section>
<section>
<h2 class="section-title" id="header-functions">Functions</h2>
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
<dd>List of columns to analyse e.g., ["col1","col2"]. <br>
Alternatively, columns can be specified in a string format, <br>
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". <br>
"all" can be passed to include all columns for analysis. <br>
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in <br>
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')</dd>
<dt><strong><code>drop_cols</code></strong></dt>
<dd>List of columns to be dropped e.g., ["col1","col2"]. <br>
Alternatively, columns can be specified in a string format, <br>
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])</dd>
<dt><strong><code>label_col</code></strong></dt>
<dd>Label/Target column (Default value = 'label')</dd>
<dt><strong><code>event_label</code></strong></dt>
<dd>Value of (positive) event (i.e label 1) (Default value = 1)</dd>
<dt><strong><code>encoding_configs</code></strong></dt>
<dd>Takes input in dictionary format. Default {} i.e. empty dict means no encoding is required. <br>
In case numerical columns are present and encoding is required, following keys shall be <br>
provided - "bin_size" i.e. no. of bins for converting the numerical columns to categorical, <br>
"bin_method" i.e. method of binning - "equal_frequency" or "equal_range" and <br>
"monotonicity_check" 1 for monotonic binning else 0. monotonicity_check of 1 will <br>
dynamically calculate the bin_size ensuring monotonic nature but can be expensive operation.</dd>
<dt><strong><code>print_impact</code></strong></dt>
<dd>(Default value = False)</dd>
</dl>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute, ig]</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre><code class="python">def IG_calculation(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], label_col=&#39;label&#39;, event_label=1,
                   encoding_configs={&#39;bin_method&#39;: &#39;equal_frequency&#39;, &#39;bin_size&#39;: 10, &#39;monotonicity_check&#39;: 0},
                   print_impact=False):
    &#34;&#34;&#34;

    Args:
      spark: Spark Session
      idf: Input Dataframe
      list_of_cols: List of columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;]. &lt;br&gt;\
    Alternatively, columns can be specified in a string format, &lt;br&gt;\
    where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. &lt;br&gt;\
    &#34;all&#34; can be passed to include all columns for analysis. &lt;br&gt;\
    Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in &lt;br&gt;\
    drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
      drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;]. &lt;br&gt;\
    Alternatively, columns can be specified in a string format, &lt;br&gt;\
    where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
      label_col: Label/Target column (Default value = &#39;label&#39;)
      event_label: Value of (positive) event (i.e label 1) (Default value = 1)
      encoding_configs: Takes input in dictionary format. Default {} i.e. empty dict means no encoding is required. &lt;br&gt;\
    In case numerical columns are present and encoding is required, following keys shall be &lt;br&gt;\
    provided - &#34;bin_size&#34; i.e. no. of bins for converting the numerical columns to categorical, &lt;br&gt;\
    &#34;bin_method&#34; i.e. method of binning - &#34;equal_frequency&#34; or &#34;equal_range&#34; and &lt;br&gt;\
    &#34;monotonicity_check&#34; 1 for monotonic binning else 0. monotonicity_check of 1 will &lt;br&gt;\
    dynamically calculate the bin_size ensuring monotonic nature but can be expensive operation.
      print_impact:  (Default value = False)

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

    return odf</code></pre>
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
<dd>List of columns to analyse e.g., ["col1","col2"]. <br>
Alternatively, columns can be specified in a string format, <br>
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". <br>
"all" can be passed to include all columns for analysis. <br>
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in <br>
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')</dd>
<dt><strong><code>drop_cols</code></strong></dt>
<dd>List of columns to be dropped e.g., ["col1","col2"]. <br>
Alternatively, columns can be specified in a string format, <br>
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])</dd>
<dt><strong><code>label_col</code></strong></dt>
<dd>Label/Target column (Default value = 'label')</dd>
<dt><strong><code>event_label</code></strong></dt>
<dd>Value of (positive) event (i.e label 1) (Default value = 1)</dd>
<dt><strong><code>encoding_configs</code></strong></dt>
<dd>Takes input in dictionary format. Default {} i.e. empty dict means no encoding is required. <br>
In case numerical columns are present and encoding is required, following keys shall be <br>
provided - "bin_size" i.e. no. of bins for converting the numerical columns to categorical, <br>
"bin_method" i.e. method of binning - "equal_frequency" or "equal_range" and <br>
"monotonicity_check" 1 for monotonic binning else 0. monotonicity_check of 1 will <br>
dynamically calculate the bin_size ensuring monotonic nature but can be expensive operation.</dd>
<dt><strong><code>print_impact</code></strong></dt>
<dd>(Default value = False)</dd>
</dl>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute, iv]</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre><code class="python">def IV_calculation(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], label_col=&#39;label&#39;, event_label=1,
                   encoding_configs={&#39;bin_method&#39;: &#39;equal_frequency&#39;, &#39;bin_size&#39;: 10, &#39;monotonicity_check&#39;: 0},
                   print_impact=False):
    &#34;&#34;&#34;

    Args:
      spark: Spark Session
      idf: Input Dataframe
      list_of_cols: List of columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;]. &lt;br&gt;\
    Alternatively, columns can be specified in a string format, &lt;br&gt;\
    where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. &lt;br&gt;\
    &#34;all&#34; can be passed to include all columns for analysis. &lt;br&gt;\
    Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in &lt;br&gt;\
    drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
      drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;]. &lt;br&gt;\
    Alternatively, columns can be specified in a string format, &lt;br&gt;\
    where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
      label_col: Label/Target column (Default value = &#39;label&#39;)
      event_label: Value of (positive) event (i.e label 1) (Default value = 1)
      encoding_configs: Takes input in dictionary format. Default {} i.e. empty dict means no encoding is required. &lt;br&gt;\
    In case numerical columns are present and encoding is required, following keys shall be &lt;br&gt;\
    provided - &#34;bin_size&#34; i.e. no. of bins for converting the numerical columns to categorical, &lt;br&gt;\
    &#34;bin_method&#34; i.e. method of binning - &#34;equal_frequency&#34; or &#34;equal_range&#34; and &lt;br&gt;\
    &#34;monotonicity_check&#34; 1 for monotonic binning else 0. monotonicity_check of 1 will &lt;br&gt;\
    dynamically calculate the bin_size ensuring monotonic nature but can be expensive operation.
      print_impact:  (Default value = False)

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

    return odf</code></pre>
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
<dd>List of columns to analyse e.g., ["col1","col2"]. <br>
Alternatively, columns can be specified in a string format, <br>
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". <br>
"all" can be passed to include all columns for analysis. <br>
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in <br>
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all') </dd>
<dt><strong><code>drop_cols</code></strong></dt>
<dd>List of columns to be dropped e.g., ["col1","col2"]. <br>
Alternatively, columns can be specified in a string format, <br>
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])</dd>
<dt><strong><code>stats_unique</code></strong></dt>
<dd>Takes arguments for read_dataset (data_ingest module) function in a dictionary format <br>
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or <br>
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})</dd>
<dt><strong><code>print_impact</code></strong></dt>
<dd>(Default value = False)</dd>
</dl>
<h2 id="returns">Returns</h2>
<p>Dataframe [attribute,*col_names] <br>
Correlation between attribute X and Y can be found at an intersection of <br>
a) row with value X in ‘attribute’ column and column ‘Y’, or <br>
b) row with value Y in ‘attribute’ column and column ‘X’.</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre><code class="python">def correlation_matrix(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], stats_unique={}, print_impact=False):
    &#34;&#34;&#34;

    Args:
      spark: Spark Session
      idf: Input Dataframe
      list_of_cols: List of columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;]. &lt;br&gt;\
    Alternatively, columns can be specified in a string format, &lt;br&gt;\
    where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. &lt;br&gt;\
    &#34;all&#34; can be passed to include all columns for analysis. &lt;br&gt;\
    Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in &lt;br&gt;\
    drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;) 
      drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;]. &lt;br&gt;\
    Alternatively, columns can be specified in a string format, &lt;br&gt;\
    where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
      stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format &lt;br&gt;\
    to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or &lt;br&gt;\
    uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
      print_impact:  (Default value = False)

    Returns:
      Dataframe [attribute,*col_names] &lt;br&gt;\
      Correlation between attribute X and Y can be found at an intersection of &lt;br&gt;\
      a) row with value X in ‘attribute’ column and column ‘Y’, or &lt;br&gt;\
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
    odf = spark.createDataFrame(odf_pd)

    if print_impact:
        odf.show(odf.count())

    return odf</code></pre>
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
<dd>List of columns to analyse e.g., ["col1","col2"]. <br>
Alternatively, columns can be specified in a string format, <br>
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". <br>
"all" can be passed to include all columns for analysis. <br>
Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in <br>
drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = 'all')</dd>
<dt><strong><code>drop_cols</code></strong></dt>
<dd>List of columns to be dropped e.g., ["col1","col2"]. <br>
Alternatively, columns can be specified in a string format, <br>
where different column names are separated by pipe delimiter “|” e.g., "col1|col2". (Default value = [])</dd>
<dt><strong><code>sample_size</code></strong></dt>
<dd>Maximum sample size (in terms of number of rows) taken for the computation. <br>
Sample dataset is extracted using random sampling. (Default value = 100000) </dd>
<dt><strong><code>stats_unique</code></strong></dt>
<dd>Takes arguments for read_dataset (data_ingest module) function in a dictionary format <br>
to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or <br>
uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})</dd>
<dt><strong><code>stats_mode</code></strong></dt>
<dd>Takes arguments for read_dataset (data_ingest module) function in a dictionary format <br>
to read pre-saved statistics on most frequently seen values i.e. if measures_of_centralTendency or <br>
mode_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})</dd>
<dt><strong><code>print_impact</code></strong></dt>
<dd>(Default value = False)</dd>
</dl>
<h2 id="returns">Returns</h2>
<p>Dataframe [Cluster, Attribute, RS_Ratio] <br>
Attributes similar to each other are grouped together with the same cluster id. <br>
Attribute with the lowest (1 — RS_Ratio) can be chosen as a representative of the cluster. <br></p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre><code class="python">def variable_clustering(spark, idf, list_of_cols=&#39;all&#39;, drop_cols=[], sample_size=100000, stats_unique={}, stats_mode={},
                        print_impact=False):
    &#34;&#34;&#34;

    Args:
      spark: Spark Session
      idf: Input Dataframe
      list_of_cols: List of columns to analyse e.g., [&#34;col1&#34;,&#34;col2&#34;]. &lt;br&gt;\
    Alternatively, columns can be specified in a string format, &lt;br&gt;\
    where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. &lt;br&gt;\
    &#34;all&#34; can be passed to include all columns for analysis. &lt;br&gt;\
    Please note that this argument is used in conjunction with drop_cols i.e. a column mentioned in &lt;br&gt;\
    drop_cols argument is not considered for analysis even if it is mentioned in list_of_cols. (Default value = &#39;all&#39;)
      drop_cols: List of columns to be dropped e.g., [&#34;col1&#34;,&#34;col2&#34;]. &lt;br&gt;\
    Alternatively, columns can be specified in a string format, &lt;br&gt;\
    where different column names are separated by pipe delimiter “|” e.g., &#34;col1|col2&#34;. (Default value = [])
      sample_size: Maximum sample size (in terms of number of rows) taken for the computation. &lt;br&gt;\
    Sample dataset is extracted using random sampling. (Default value = 100000) 
      stats_unique: Takes arguments for read_dataset (data_ingest module) function in a dictionary format &lt;br&gt;\
    to read pre-saved statistics on unique value count i.e. if measures_of_cardinality or &lt;br&gt;\
    uniqueCount_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
      stats_mode: Takes arguments for read_dataset (data_ingest module) function in a dictionary format &lt;br&gt;\
    to read pre-saved statistics on most frequently seen values i.e. if measures_of_centralTendency or &lt;br&gt;\
    mode_computation (data_analyzer.stats_generator module) has been computed &amp; saved before. (Default value = {})
      print_impact:  (Default value = False)

    Returns:
      Dataframe [Cluster, Attribute, RS_Ratio] &lt;br&gt;\
      Attributes similar to each other are grouped together with the same cluster id. &lt;br&gt;\
      Attribute with the lowest (1 — RS_Ratio) can be chosen as a representative of the cluster. &lt;br&gt;\

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
<li><code><a title="anovos.data_analyzer" href="index.html">anovos.data_analyzer</a></code></li>
</ul>
</li>
<li><h3><a href="#header-functions">Functions</a></h3>
<ul class="">
<li><code><a title="anovos.data_analyzer.association_evaluator.IG_calculation" href="#anovos.data_analyzer.association_evaluator.IG_calculation">IG_calculation</a></code></li>
<li><code><a title="anovos.data_analyzer.association_evaluator.IV_calculation" href="#anovos.data_analyzer.association_evaluator.IV_calculation">IV_calculation</a></code></li>
<li><code><a title="anovos.data_analyzer.association_evaluator.correlation_matrix" href="#anovos.data_analyzer.association_evaluator.correlation_matrix">correlation_matrix</a></code></li>
<li><code><a title="anovos.data_analyzer.association_evaluator.variable_clustering" href="#anovos.data_analyzer.association_evaluator.variable_clustering">variable_clustering</a></code></li>
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
