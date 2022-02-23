# <code>feature_exploration</code>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
import pandas as pd
import numpy as np
from sentence_transformers import SentenceTransformer
from sentence_transformers import util

model_fer = SentenceTransformer("all-mpnet-base-v2")
input_path_fer = "https://raw.githubusercontent.com/anovos/anovos/main/data/feature_recommender/flatten_fr_db.csv"
df_input_fer = pd.read_csv(input_path_fer)
df_input_fer = df_input_fer.rename(columns=lambda x: x.strip().replace(" ", "_"))
feature_name_column = str(df_input_fer.columns.tolist()[0])
feature_desc_column = str(df_input_fer.columns.tolist()[1])
industry_column = str(df_input_fer.columns.tolist()[2])
usecase_column = str(df_input_fer.columns.tolist()[3])
source_column = str(df_input_fer.columns.tolist()[4])


def list_all_industry():
    """:return: DataFrame of all the supported industries as part of feature exploration/recommendation"""
    odf_uni = df_input_fer.iloc[:, 2].unique()
    odf = pd.DataFrame(odf_uni, columns=["Industry"])
    return odf


def list_all_usecase():
    """:return: DataFrame of all the supported usecases as part of feature exploration/recommendation"""
    odf_uni = df_input_fer.iloc[:, 3].unique()
    odf = pd.DataFrame(odf_uni, columns=["Usecase"])
    return odf


def list_all_pair():
    """:return: DataFrame of all the supported Industry/Usecase pairs as part of feature exploration/recommendation"""
    odf = df_input_fer.iloc[:, [2, 3]].drop_duplicates(keep="last", ignore_index=True)
    return odf


def process_usecase(usecase, semantic):
    """

    Parameters
    ----------
    usecase
        Input usecase (string)
    semantic
        Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------

    """
    if type(semantic) != bool:
        raise TypeError("Invalid input for semantic")
    if type(usecase) != str:
        raise TypeError("Invalid input for usecase")
    usecase = usecase.lower().strip()
    usecase = usecase.replace("[^A-Za-z0-9 ]+", " ")
    all_usecase = list_all_usecase()["Usecase"].to_list()
    if semantic and usecase not in all_usecase:
        all_usecase_embeddings = model_fer.encode(all_usecase, convert_to_tensor=True)
        usecase_embeddings = model_fer.encode(usecase, convert_to_tensor=True)
        cos_scores = util.pytorch_cos_sim(usecase_embeddings, all_usecase_embeddings)[0]
        first_match_index = int(np.argpartition(-cos_scores, 0)[0])
        processed_usecase = all_usecase[first_match_index]
        print(
            "Given input Usecase is not available. Showing the most semantically relevant Usecase result: ",
            processed_usecase,
        )
    else:
        processed_usecase = usecase
    return processed_usecase


def process_industry(industry, semantic):
    """

    Parameters
    ----------
    industry
        Input industry (string)
    semantic
        Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------

    """
    if type(semantic) != bool:
        raise TypeError("Invalid input for semantic")
    if type(industry) != str:
        raise TypeError("Invalid input for industry")
    industry = industry.lower().strip()
    industry = industry.replace("[^A-Za-z0-9 ]+", " ")
    all_industry = list_all_industry()["Industry"].to_list()
    if semantic and industry not in all_industry:
        all_industry_embeddings = model_fer.encode(all_industry, convert_to_tensor=True)
        industry_embeddings = model_fer.encode(industry, convert_to_tensor=True)
        cos_scores = util.pytorch_cos_sim(industry_embeddings, all_industry_embeddings)[
            0
        ]
        first_match_index = int(np.argpartition(-cos_scores, 0)[0])
        processed_industry = all_industry[first_match_index]
        print(
            "Given input Industry is not available. Showing the most semantically relevant Industry result: ",
            processed_industry,
        )
    else:
        processed_industry = industry
    return processed_industry


def list_usecase_by_industry(industry, semantic=True):
    """

    Parameters
    ----------
    industry
        Input industry (string)
    semantic
        Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------

    """
    industry = process_industry(industry, semantic)
    odf = pd.DataFrame(df_input_fer.loc[df_input_fer.iloc[:, 2] == industry].iloc[:, 3])
    odf = odf.drop_duplicates(keep="last", ignore_index=True)
    return odf


def list_industry_by_usecase(usecase, semantic=True):
    """

    Parameters
    ----------
    usecase
        Input usecase (string)
    semantic
        Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------

    """
    usecase = process_usecase(usecase, semantic)
    odf = pd.DataFrame(df_input_fer.loc[df_input_fer.iloc[:, 3] == usecase].iloc[:, 2])
    odf = odf.drop_duplicates(keep="last", ignore_index=True)
    return odf


def list_feature_by_industry(industry, num_of_feat=100, semantic=True):
    """

    Parameters
    ----------
    industry
        Input industry (string)
    num_of_feat
        Number of features to be displayed in the output.
        Value can be either integer, or 'all' - display all features matched with the input. Default is 100.
    semantic
        Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------

    """
    if type(num_of_feat) != int or num_of_feat < 0:
        if num_of_feat != "all":
            raise TypeError("Invalid input for num_of_feat")
    industry = process_industry(industry, semantic)
    odf = df_input_fer.loc[df_input_fer.iloc[:, 2] == industry].drop_duplicates(
        keep="last", ignore_index=True
    )
    if len(odf) > 0:
        odf["count"] = odf.groupby(usecase_column)[usecase_column].transform("count")
        odf.sort_values("count", inplace=True, ascending=False)
        odf = odf.drop("count", axis=1)
        if num_of_feat != "all":
            odf = odf.head(num_of_feat).reset_index(drop=True)
        else:
            odf = odf.reset_index(drop=True)
    return odf


def list_feature_by_usecase(usecase, num_of_feat=100, semantic=True):
    """

    Parameters
    ----------
    usecase
        Input usecase (string)
    num_of_feat
        Number of features to be displayed in the output.
        Value can be either integer, or 'all' - display all features matched with the input.  Default is 100.
    semantic
        Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------

    """
    if type(num_of_feat) != int or num_of_feat < 0:
        if num_of_feat != "all":
            raise TypeError("Invalid input for num_of_feat")
    usecase = process_usecase(usecase, semantic)
    odf = df_input_fer.loc[df_input_fer.iloc[:, 3] == usecase].drop_duplicates(
        keep="last", ignore_index=True
    )
    if len(odf) > 0:
        odf["count"] = odf.groupby(industry_column)[industry_column].transform("count")
        odf.sort_values("count", inplace=True, ascending=False)
        odf = odf.drop("count", axis=1)
        if num_of_feat != "all":
            odf = odf.head(num_of_feat).reset_index(drop=True)
        else:
            odf = odf.reset_index(drop=True)
    return odf


def list_feature_by_pair(industry, usecase, num_of_feat=100, semantic=True):
    """

    Parameters
    ----------
    industry
        Input industry (string)
    usecase
        Input usecase (string)
    num_of_feat
        Number of features to be displayed in the output.
        Value can be either integer, or 'all' - display all features matched with the input.  Default is 100.
    semantic
        Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------

    """
    if type(num_of_feat) != int or num_of_feat < 0:
        if num_of_feat != "all":
            raise TypeError("Invalid input for num_of_feat")
    industry = process_industry(industry, semantic)
    usecase = process_usecase(usecase, semantic)
    if num_of_feat != "all":
        odf = (
            df_input_fer.loc[
                (df_input_fer.iloc[:, 2] == industry)
                & (df_input_fer.iloc[:, 3] == usecase)
            ]
            .drop_duplicates(keep="last", ignore_index=True)
            .head(num_of_feat)
        )
    else:
        odf = df_input_fer.loc[
            (df_input_fer.iloc[:, 2] == industry) & (df_input_fer.iloc[:, 3] == usecase)
        ].drop_duplicates(keep="last", ignore_index=True)
    return odf
```
</pre>
</details>
## Functions
<dl>
<dt id="anovos.feature_recommender.feature_exploration.list_all_industry"><code class="name flex">
<span>def <span class="ident">list_all_industry</span></span>(<span>)</span>
</code></dt>
<dd>
<div class="desc"><p>:return: DataFrame of all the supported industries as part of feature exploration/recommendation</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def list_all_industry():
    """:return: DataFrame of all the supported industries as part of feature exploration/recommendation"""
    odf_uni = df_input_fer.iloc[:, 2].unique()
    odf = pd.DataFrame(odf_uni, columns=["Industry"])
    return odf
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.feature_exploration.list_all_pair"><code class="name flex">
<span>def <span class="ident">list_all_pair</span></span>(<span>)</span>
</code></dt>
<dd>
<div class="desc"><p>:return: DataFrame of all the supported Industry/Usecase pairs as part of feature exploration/recommendation</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def list_all_pair():
    """:return: DataFrame of all the supported Industry/Usecase pairs as part of feature exploration/recommendation"""
    odf = df_input_fer.iloc[:, [2, 3]].drop_duplicates(keep="last", ignore_index=True)
    return odf
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.feature_exploration.list_all_usecase"><code class="name flex">
<span>def <span class="ident">list_all_usecase</span></span>(<span>)</span>
</code></dt>
<dd>
<div class="desc"><p>:return: DataFrame of all the supported usecases as part of feature exploration/recommendation</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def list_all_usecase():
    """:return: DataFrame of all the supported usecases as part of feature exploration/recommendation"""
    odf_uni = df_input_fer.iloc[:, 3].unique()
    odf = pd.DataFrame(odf_uni, columns=["Usecase"])
    return odf
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.feature_exploration.list_feature_by_industry"><code class="name flex">
<span>def <span class="ident">list_feature_by_industry</span></span>(<span>industry, num_of_feat=100, semantic=True)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>industry</code></strong></dt>
<dd>Input industry (string)</dd>
<dt><strong><code>num_of_feat</code></strong></dt>
<dd>Number of features to be displayed in the output.
Value can be either integer, or 'all' - display all features matched with the input. Default is 100.</dd>
<dt><strong><code>semantic</code></strong></dt>
<dd>Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def list_feature_by_industry(industry, num_of_feat=100, semantic=True):
    """

    Parameters
    ----------
    industry
        Input industry (string)
    num_of_feat
        Number of features to be displayed in the output.
        Value can be either integer, or 'all' - display all features matched with the input. Default is 100.
    semantic
        Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------

    """
    if type(num_of_feat) != int or num_of_feat < 0:
        if num_of_feat != "all":
            raise TypeError("Invalid input for num_of_feat")
    industry = process_industry(industry, semantic)
    odf = df_input_fer.loc[df_input_fer.iloc[:, 2] == industry].drop_duplicates(
        keep="last", ignore_index=True
    )
    if len(odf) > 0:
        odf["count"] = odf.groupby(usecase_column)[usecase_column].transform("count")
        odf.sort_values("count", inplace=True, ascending=False)
        odf = odf.drop("count", axis=1)
        if num_of_feat != "all":
            odf = odf.head(num_of_feat).reset_index(drop=True)
        else:
            odf = odf.reset_index(drop=True)
    return odf
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.feature_exploration.list_feature_by_pair"><code class="name flex">
<span>def <span class="ident">list_feature_by_pair</span></span>(<span>industry, usecase, num_of_feat=100, semantic=True)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>industry</code></strong></dt>
<dd>Input industry (string)</dd>
<dt><strong><code>usecase</code></strong></dt>
<dd>Input usecase (string)</dd>
<dt><strong><code>num_of_feat</code></strong></dt>
<dd>Number of features to be displayed in the output.
Value can be either integer, or 'all' - display all features matched with the input.
Default is 100.</dd>
<dt><strong><code>semantic</code></strong></dt>
<dd>Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def list_feature_by_pair(industry, usecase, num_of_feat=100, semantic=True):
    """

    Parameters
    ----------
    industry
        Input industry (string)
    usecase
        Input usecase (string)
    num_of_feat
        Number of features to be displayed in the output.
        Value can be either integer, or 'all' - display all features matched with the input.  Default is 100.
    semantic
        Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------

    """
    if type(num_of_feat) != int or num_of_feat < 0:
        if num_of_feat != "all":
            raise TypeError("Invalid input for num_of_feat")
    industry = process_industry(industry, semantic)
    usecase = process_usecase(usecase, semantic)
    if num_of_feat != "all":
        odf = (
            df_input_fer.loc[
                (df_input_fer.iloc[:, 2] == industry)
                & (df_input_fer.iloc[:, 3] == usecase)
            ]
            .drop_duplicates(keep="last", ignore_index=True)
            .head(num_of_feat)
        )
    else:
        odf = df_input_fer.loc[
            (df_input_fer.iloc[:, 2] == industry) & (df_input_fer.iloc[:, 3] == usecase)
        ].drop_duplicates(keep="last", ignore_index=True)
    return odf
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.feature_exploration.list_feature_by_usecase"><code class="name flex">
<span>def <span class="ident">list_feature_by_usecase</span></span>(<span>usecase, num_of_feat=100, semantic=True)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>usecase</code></strong></dt>
<dd>Input usecase (string)</dd>
<dt><strong><code>num_of_feat</code></strong></dt>
<dd>Number of features to be displayed in the output.
Value can be either integer, or 'all' - display all features matched with the input.
Default is 100.</dd>
<dt><strong><code>semantic</code></strong></dt>
<dd>Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def list_feature_by_usecase(usecase, num_of_feat=100, semantic=True):
    """

    Parameters
    ----------
    usecase
        Input usecase (string)
    num_of_feat
        Number of features to be displayed in the output.
        Value can be either integer, or 'all' - display all features matched with the input.  Default is 100.
    semantic
        Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------

    """
    if type(num_of_feat) != int or num_of_feat < 0:
        if num_of_feat != "all":
            raise TypeError("Invalid input for num_of_feat")
    usecase = process_usecase(usecase, semantic)
    odf = df_input_fer.loc[df_input_fer.iloc[:, 3] == usecase].drop_duplicates(
        keep="last", ignore_index=True
    )
    if len(odf) > 0:
        odf["count"] = odf.groupby(industry_column)[industry_column].transform("count")
        odf.sort_values("count", inplace=True, ascending=False)
        odf = odf.drop("count", axis=1)
        if num_of_feat != "all":
            odf = odf.head(num_of_feat).reset_index(drop=True)
        else:
            odf = odf.reset_index(drop=True)
    return odf
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.feature_exploration.list_industry_by_usecase"><code class="name flex">
<span>def <span class="ident">list_industry_by_usecase</span></span>(<span>usecase, semantic=True)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>usecase</code></strong></dt>
<dd>Input usecase (string)</dd>
<dt><strong><code>semantic</code></strong></dt>
<dd>Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def list_industry_by_usecase(usecase, semantic=True):
    """

    Parameters
    ----------
    usecase
        Input usecase (string)
    semantic
        Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------

    """
    usecase = process_usecase(usecase, semantic)
    odf = pd.DataFrame(df_input_fer.loc[df_input_fer.iloc[:, 3] == usecase].iloc[:, 2])
    odf = odf.drop_duplicates(keep="last", ignore_index=True)
    return odf
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.feature_exploration.list_usecase_by_industry"><code class="name flex">
<span>def <span class="ident">list_usecase_by_industry</span></span>(<span>industry, semantic=True)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>industry</code></strong></dt>
<dd>Input industry (string)</dd>
<dt><strong><code>semantic</code></strong></dt>
<dd>Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def list_usecase_by_industry(industry, semantic=True):
    """

    Parameters
    ----------
    industry
        Input industry (string)
    semantic
        Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------

    """
    industry = process_industry(industry, semantic)
    odf = pd.DataFrame(df_input_fer.loc[df_input_fer.iloc[:, 2] == industry].iloc[:, 3])
    odf = odf.drop_duplicates(keep="last", ignore_index=True)
    return odf
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.feature_exploration.process_industry"><code class="name flex">
<span>def <span class="ident">process_industry</span></span>(<span>industry, semantic)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>industry</code></strong></dt>
<dd>Input industry (string)</dd>
<dt><strong><code>semantic</code></strong></dt>
<dd>Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def process_industry(industry, semantic):
    """

    Parameters
    ----------
    industry
        Input industry (string)
    semantic
        Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------

    """
    if type(semantic) != bool:
        raise TypeError("Invalid input for semantic")
    if type(industry) != str:
        raise TypeError("Invalid input for industry")
    industry = industry.lower().strip()
    industry = industry.replace("[^A-Za-z0-9 ]+", " ")
    all_industry = list_all_industry()["Industry"].to_list()
    if semantic and industry not in all_industry:
        all_industry_embeddings = model_fer.encode(all_industry, convert_to_tensor=True)
        industry_embeddings = model_fer.encode(industry, convert_to_tensor=True)
        cos_scores = util.pytorch_cos_sim(industry_embeddings, all_industry_embeddings)[
            0
        ]
        first_match_index = int(np.argpartition(-cos_scores, 0)[0])
        processed_industry = all_industry[first_match_index]
        print(
            "Given input Industry is not available. Showing the most semantically relevant Industry result: ",
            processed_industry,
        )
    else:
        processed_industry = industry
    return processed_industry
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.feature_exploration.process_usecase"><code class="name flex">
<span>def <span class="ident">process_usecase</span></span>(<span>usecase, semantic)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>usecase</code></strong></dt>
<dd>Input usecase (string)</dd>
<dt><strong><code>semantic</code></strong></dt>
<dd>Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def process_usecase(usecase, semantic):
    """

    Parameters
    ----------
    usecase
        Input usecase (string)
    semantic
        Input semantic (boolean) - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------

    """
    if type(semantic) != bool:
        raise TypeError("Invalid input for semantic")
    if type(usecase) != str:
        raise TypeError("Invalid input for usecase")
    usecase = usecase.lower().strip()
    usecase = usecase.replace("[^A-Za-z0-9 ]+", " ")
    all_usecase = list_all_usecase()["Usecase"].to_list()
    if semantic and usecase not in all_usecase:
        all_usecase_embeddings = model_fer.encode(all_usecase, convert_to_tensor=True)
        usecase_embeddings = model_fer.encode(usecase, convert_to_tensor=True)
        cos_scores = util.pytorch_cos_sim(usecase_embeddings, all_usecase_embeddings)[0]
        first_match_index = int(np.argpartition(-cos_scores, 0)[0])
        processed_usecase = all_usecase[first_match_index]
        print(
            "Given input Usecase is not available. Showing the most semantically relevant Usecase result: ",
            processed_usecase,
        )
    else:
        processed_usecase = usecase
    return processed_usecase
```
</pre>
</details>
</dd>
</dl>