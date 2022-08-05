# <code>feature_explorer</code>
<p>Feature explorer helps list down the potential features from our corpus based
on user defined industry or/and use case.</p>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
"""Feature explorer helps list down the potential features from our corpus based
on user defined industry or/and use case.
"""
import numpy as np
import pandas as pd
from sentence_transformers import util

from anovos.feature_recommender.featrec_init import (
    feature_exploration_prep,
    get_column_name,
    model_fer,
)

df_input_fer = feature_exploration_prep()
(
    feature_name_column,
    feature_desc_column,
    industry_column,
    usecase_column,
) = get_column_name(df_input_fer)


def list_all_industry():
    """
    Lists down all the Industries that are supported in Feature Recommender module.

    Returns
    -------
    DataFrame of all the supported industries as part of feature exploration/recommendation
    """
    odf_uni = df_input_fer.iloc[:, 2].unique()
    odf = pd.DataFrame(odf_uni, columns=["Industry"])
    return odf


def list_all_usecase():
    """
    Lists down all the Use cases that are supported in Feature Recommender module.

    Returns
    -------
    DataFrame of all the supported usecases as part of feature exploration/recommendation
    """
    odf_uni = df_input_fer.iloc[:, 3].unique()
    odf = pd.DataFrame(odf_uni, columns=["Usecase"])
    return odf


def list_all_pair():
    """
    Lists down all the Industry/Use case pairs that are supported in Feature Recommender module.

    Returns
    -------
    DataFrame of all the supported Industry/Usecase pairs as part of feature exploration/recommendation
    """
    odf = df_input_fer.iloc[:, [2, 3]].drop_duplicates(keep="last", ignore_index=True)
    return odf


def process_usecase(usecase: str, semantic: bool):
    """

    Parameters
    ----------
    usecase : str
        Input usecase
    semantic : bool
        Whether the input needs to go through semantic similarity or not. Default is True.

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
        all_usecase_embeddings = model_fer.model.encode(
            all_usecase, convert_to_tensor=True
        )
        usecase_embeddings = model_fer.model.encode(usecase, convert_to_tensor=True)
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


def process_industry(industry: str, semantic: bool):
    """

    Parameters
    ----------
    industry : str
        Input industry
    semantic : bool
        Whether the input needs to go through semantic similarity or not. Default is True.

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
        all_industry_embeddings = model_fer.model.encode(
            all_industry, convert_to_tensor=True
        )
        industry_embeddings = model_fer.model.encode(industry, convert_to_tensor=True)
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
    Lists down all the Use cases that are supported in Feature Recommender Package based on the Input Industry.

    Parameters
    ----------
    industry : str
        Input industry
    semantic : bool
        Input semantic - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------

    """
    industry = process_industry(industry, semantic)
    odf = pd.DataFrame(df_input_fer.loc[df_input_fer.iloc[:, 2] == industry].iloc[:, 3])
    odf = odf.drop_duplicates(keep="last", ignore_index=True)
    return odf


def list_industry_by_usecase(usecase, semantic=True):
    """
    Lists down all the Use cases that are supported in Feature Recommender Package based on the Input Industry.

    Parameters
    ----------
    usecase : str
        Input usecase
    semantic : bool
        Input semantic - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------

    """
    usecase = process_usecase(usecase, semantic)
    odf = pd.DataFrame(df_input_fer.loc[df_input_fer.iloc[:, 3] == usecase].iloc[:, 2])
    odf = odf.drop_duplicates(keep="last", ignore_index=True)
    return odf


def list_feature_by_industry(industry, num_of_feat=100, semantic=True):
    """
    Lists down all the Features that are available in Feature Recommender Package based on the Input Industry.

    Parameters
    ----------
    industry : str
        Input industry
    num_of_feat : int
        Number of features to be displayed in the output.
        Value can be either integer, or 'all' - display all features matched with the input. Default is 100.
    semantic : bool
        Input semantic - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------
    DataFrame
        Columns are:
        - Feature Name: Name of the suggested Feature
        - Feature Description: Description of the suggested Feature
        - Industry: Industry name of the suggested Feature
        - Usecase: Usecase name of the suggested Feature
        - Source: Source of the suggested Feature

        The list of features is sorted by the Usecases' Feature Popularity to the Input Industry.

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
    Lists down all the Features that are available in Feature Recommender Package based on the Input Usecase.

    Parameters
    ----------
    usecase : str
        Input usecase
    num_of_feat : int
        Number of features to be displayed in the output.
        Value can be either integer, or 'all' - display all features matched with the input.  Default is 100.
    semantic : bool
        Input semantic - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------
    DataFrame
        Columns are:

        - Feature Name: Name of the suggested Feature
        - Feature Description: Description of the suggested Feature
        - Industry: Industry name of the suggested Feature
        - Usecase: Usecase name of the suggested Feature
        - Source: Source of the suggested Feature

        The list of features is sorted by the Industries' Feature Popularity to the Input Usecase.

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
    Lists down all the Features that are available in Feature Recommender Package based
    on the Input Industry/Usecase pair

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
    DataFrame
        Columns are:

        - Feature Name: Name of the suggested Feature
        - Feature Description: Description of the suggested Feature
        - Industry: Industry name of the suggested Feature
        - Usecase: Usecase name of the suggested Feature
        - Source: Source of the suggested Feature

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
<dt id="anovos.feature_recommender.feature_explorer.list_all_industry"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">list_all_industry</span></span>(<span class="n">)</span>
</code></dt>
<dd>
<div class="desc"><p>Lists down all the Industries that are supported in Feature Recommender module.</p>
<h2 id="returns">Returns</h2>
<dl>
<dt><code>DataFrame</code> of <code>all the supported industries as part</code> of <code>feature exploration/recommendation</code></dt>
<dd>&nbsp;</dd>
</dl></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def list_all_industry():
    """
    Lists down all the Industries that are supported in Feature Recommender module.

    Returns
    -------
    DataFrame of all the supported industries as part of feature exploration/recommendation
    """
    odf_uni = df_input_fer.iloc[:, 2].unique()
    odf = pd.DataFrame(odf_uni, columns=["Industry"])
    return odf
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.feature_explorer.list_all_pair"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">list_all_pair</span></span>(<span class="n">)</span>
</code></dt>
<dd>
<div class="desc"><p>Lists down all the Industry/Use case pairs that are supported in Feature Recommender module.</p>
<h2 id="returns">Returns</h2>
<dl>
<dt><code>DataFrame</code> of <code>all the supported Industry/Usecase pairs as part</code> of <code>feature exploration/recommendation</code></dt>
<dd>&nbsp;</dd>
</dl></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def list_all_pair():
    """
    Lists down all the Industry/Use case pairs that are supported in Feature Recommender module.

    Returns
    -------
    DataFrame of all the supported Industry/Usecase pairs as part of feature exploration/recommendation
    """
    odf = df_input_fer.iloc[:, [2, 3]].drop_duplicates(keep="last", ignore_index=True)
    return odf
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.feature_explorer.list_all_usecase"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">list_all_usecase</span></span>(<span class="n">)</span>
</code></dt>
<dd>
<div class="desc"><p>Lists down all the Use cases that are supported in Feature Recommender module.</p>
<h2 id="returns">Returns</h2>
<dl>
<dt><code>DataFrame</code> of <code>all the supported usecases as part</code> of <code>feature exploration/recommendation</code></dt>
<dd>&nbsp;</dd>
</dl></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def list_all_usecase():
    """
    Lists down all the Use cases that are supported in Feature Recommender module.

    Returns
    -------
    DataFrame of all the supported usecases as part of feature exploration/recommendation
    """
    odf_uni = df_input_fer.iloc[:, 3].unique()
    odf = pd.DataFrame(odf_uni, columns=["Usecase"])
    return odf
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.feature_explorer.list_feature_by_industry"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">list_feature_by_industry</span></span>(<span class="n">industry, num_of_feat=100, semantic=True)</span>
</code></dt>
<dd>
<div class="desc"><p>Lists down all the Features that are available in Feature Recommender Package based on the Input Industry.</p>
<h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>industry</code></strong> :&ensp;<code>str</code></dt>
<dd>Input industry</dd>
<dt><strong><code>num_of_feat</code></strong> :&ensp;<code>int</code></dt>
<dd>Number of features to be displayed in the output.
Value can be either integer, or 'all' - display all features matched with the input. Default is 100.</dd>
<dt><strong><code>semantic</code></strong> :&ensp;<code>bool</code></dt>
<dd>Input semantic - Whether the input needs to go through semantic similarity or not. Default is True.</dd>
</dl>
<h2 id="returns">Returns</h2>
<dl>
<dt><code>DataFrame</code></dt>
<dd>
<p>Columns are:
- Feature Name: Name of the suggested Feature
- Feature Description: Description of the suggested Feature
- Industry: Industry name of the suggested Feature
- Usecase: Usecase name of the suggested Feature
- Source: Source of the suggested Feature</p>
<p>The list of features is sorted by the Usecases' Feature Popularity to the Input Industry.</p>
</dd>
</dl></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def list_feature_by_industry(industry, num_of_feat=100, semantic=True):
    """
    Lists down all the Features that are available in Feature Recommender Package based on the Input Industry.

    Parameters
    ----------
    industry : str
        Input industry
    num_of_feat : int
        Number of features to be displayed in the output.
        Value can be either integer, or 'all' - display all features matched with the input. Default is 100.
    semantic : bool
        Input semantic - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------
    DataFrame
        Columns are:
        - Feature Name: Name of the suggested Feature
        - Feature Description: Description of the suggested Feature
        - Industry: Industry name of the suggested Feature
        - Usecase: Usecase name of the suggested Feature
        - Source: Source of the suggested Feature

        The list of features is sorted by the Usecases' Feature Popularity to the Input Industry.

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
<dt id="anovos.feature_recommender.feature_explorer.list_feature_by_pair"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">list_feature_by_pair</span></span>(<span class="n">industry, usecase, num_of_feat=100, semantic=True)</span>
</code></dt>
<dd>
<div class="desc"><p>Lists down all the Features that are available in Feature Recommender Package based
on the Input Industry/Usecase pair</p>
<h2 id="parameters">Parameters</h2>
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
<h2 id="returns">Returns</h2>
<dl>
<dt><code>DataFrame</code></dt>
<dd>
<p>Columns are:</p>
<ul>
<li>Feature Name: Name of the suggested Feature</li>
<li>Feature Description: Description of the suggested Feature</li>
<li>Industry: Industry name of the suggested Feature</li>
<li>Usecase: Usecase name of the suggested Feature</li>
<li>Source: Source of the suggested Feature</li>
</ul>
</dd>
</dl></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def list_feature_by_pair(industry, usecase, num_of_feat=100, semantic=True):
    """
    Lists down all the Features that are available in Feature Recommender Package based
    on the Input Industry/Usecase pair

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
    DataFrame
        Columns are:

        - Feature Name: Name of the suggested Feature
        - Feature Description: Description of the suggested Feature
        - Industry: Industry name of the suggested Feature
        - Usecase: Usecase name of the suggested Feature
        - Source: Source of the suggested Feature

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
<dt id="anovos.feature_recommender.feature_explorer.list_feature_by_usecase"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">list_feature_by_usecase</span></span>(<span class="n">usecase, num_of_feat=100, semantic=True)</span>
</code></dt>
<dd>
<div class="desc"><p>Lists down all the Features that are available in Feature Recommender Package based on the Input Usecase.</p>
<h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>usecase</code></strong> :&ensp;<code>str</code></dt>
<dd>Input usecase</dd>
<dt><strong><code>num_of_feat</code></strong> :&ensp;<code>int</code></dt>
<dd>Number of features to be displayed in the output.
Value can be either integer, or 'all' - display all features matched with the input.
Default is 100.</dd>
<dt><strong><code>semantic</code></strong> :&ensp;<code>bool</code></dt>
<dd>Input semantic - Whether the input needs to go through semantic similarity or not. Default is True.</dd>
</dl>
<h2 id="returns">Returns</h2>
<dl>
<dt><code>DataFrame</code></dt>
<dd>
<p>Columns are:</p>
<ul>
<li>Feature Name: Name of the suggested Feature</li>
<li>Feature Description: Description of the suggested Feature</li>
<li>Industry: Industry name of the suggested Feature</li>
<li>Usecase: Usecase name of the suggested Feature</li>
<li>Source: Source of the suggested Feature</li>
</ul>
<p>The list of features is sorted by the Industries' Feature Popularity to the Input Usecase.</p>
</dd>
</dl></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def list_feature_by_usecase(usecase, num_of_feat=100, semantic=True):
    """
    Lists down all the Features that are available in Feature Recommender Package based on the Input Usecase.

    Parameters
    ----------
    usecase : str
        Input usecase
    num_of_feat : int
        Number of features to be displayed in the output.
        Value can be either integer, or 'all' - display all features matched with the input.  Default is 100.
    semantic : bool
        Input semantic - Whether the input needs to go through semantic similarity or not. Default is True.

    Returns
    -------
    DataFrame
        Columns are:

        - Feature Name: Name of the suggested Feature
        - Feature Description: Description of the suggested Feature
        - Industry: Industry name of the suggested Feature
        - Usecase: Usecase name of the suggested Feature
        - Source: Source of the suggested Feature

        The list of features is sorted by the Industries' Feature Popularity to the Input Usecase.

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
<dt id="anovos.feature_recommender.feature_explorer.list_industry_by_usecase"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">list_industry_by_usecase</span></span>(<span class="n">usecase, semantic=True)</span>
</code></dt>
<dd>
<div class="desc"><p>Lists down all the Use cases that are supported in Feature Recommender Package based on the Input Industry.</p>
<h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>usecase</code></strong> :&ensp;<code>str</code></dt>
<dd>Input usecase</dd>
<dt><strong><code>semantic</code></strong> :&ensp;<code>bool</code></dt>
<dd>Input semantic - Whether the input needs to go through semantic similarity or not. Default is True.</dd>
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
    Lists down all the Use cases that are supported in Feature Recommender Package based on the Input Industry.

    Parameters
    ----------
    usecase : str
        Input usecase
    semantic : bool
        Input semantic - Whether the input needs to go through semantic similarity or not. Default is True.

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
<dt id="anovos.feature_recommender.feature_explorer.list_usecase_by_industry"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">list_usecase_by_industry</span></span>(<span class="n">industry, semantic=True)</span>
</code></dt>
<dd>
<div class="desc"><p>Lists down all the Use cases that are supported in Feature Recommender Package based on the Input Industry.</p>
<h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>industry</code></strong> :&ensp;<code>str</code></dt>
<dd>Input industry</dd>
<dt><strong><code>semantic</code></strong> :&ensp;<code>bool</code></dt>
<dd>Input semantic - Whether the input needs to go through semantic similarity or not. Default is True.</dd>
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
    Lists down all the Use cases that are supported in Feature Recommender Package based on the Input Industry.

    Parameters
    ----------
    industry : str
        Input industry
    semantic : bool
        Input semantic - Whether the input needs to go through semantic similarity or not. Default is True.

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
<dt id="anovos.feature_recommender.feature_explorer.process_industry"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">process_industry</span></span>(<span class="n">industry: str, semantic: bool)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>industry</code></strong> :&ensp;<code>str</code></dt>
<dd>Input industry</dd>
<dt><strong><code>semantic</code></strong> :&ensp;<code>bool</code></dt>
<dd>Whether the input needs to go through semantic similarity or not. Default is True.</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def process_industry(industry: str, semantic: bool):
    """

    Parameters
    ----------
    industry : str
        Input industry
    semantic : bool
        Whether the input needs to go through semantic similarity or not. Default is True.

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
        all_industry_embeddings = model_fer.model.encode(
            all_industry, convert_to_tensor=True
        )
        industry_embeddings = model_fer.model.encode(industry, convert_to_tensor=True)
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
<dt id="anovos.feature_recommender.feature_explorer.process_usecase"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">process_usecase</span></span>(<span class="n">usecase: str, semantic: bool)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>usecase</code></strong> :&ensp;<code>str</code></dt>
<dd>Input usecase</dd>
<dt><strong><code>semantic</code></strong> :&ensp;<code>bool</code></dt>
<dd>Whether the input needs to go through semantic similarity or not. Default is True.</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def process_usecase(usecase: str, semantic: bool):
    """

    Parameters
    ----------
    usecase : str
        Input usecase
    semantic : bool
        Whether the input needs to go through semantic similarity or not. Default is True.

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
        all_usecase_embeddings = model_fer.model.encode(
            all_usecase, convert_to_tensor=True
        )
        usecase_embeddings = model_fer.model.encode(usecase, convert_to_tensor=True)
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