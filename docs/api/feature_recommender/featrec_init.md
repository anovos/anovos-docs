# <code>featrec_init</code>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
import os
import copy
from re import finditer

import pandas as pd
from sentence_transformers import SentenceTransformer
from torch.hub import _get_torch_home
import site


def detect_model_path():
    """

    Returns
    -------
    Local Feature Explorer and Recommender semantic model path (if the model is pre-downloaded)
    """
    transformers_path = os.getenv("SENTENCE_TRANSFORMERS_HOME")
    if transformers_path is None:
        try:
            torch_home = _get_torch_home()
        except ImportError:
            torch_home = os.path.expanduser(
                os.getenv(
                    "TORCH_HOME",
                    os.path.join(os.getenv("XDG_CACHE_HOME", "~/.cache"), "torch"),
                )
            )
        transformers_path = os.path.join(torch_home, "sentence_transformers")
    model_path = os.path.join(
        transformers_path, "sentence-transformers_all-mpnet-base-v2"
    )
    return model_path


def model_download():
    print("Starting the Semantic Model download")
    SentenceTransformer("all-mpnet-base-v2")
    print("Model downloading finished")


class _TransformerModel:
    def __init__(self):
        self._model = None

    @property
    def model(self) -> SentenceTransformer:
        if self._model is None:
            model_path = detect_model_path()
            if os.path.exists(model_path):
                self._model = SentenceTransformer(model_path)
            else:
                raise FileNotFoundError(
                    "Model has not been downloaded. Please use model_download() function to download the model first"
                )
        return self._model


model_fer = _TransformerModel()


def init_input_fer():
    """

    Returns
    -------
    Loading the Feature Explorer and Recommender (FER) Input DataFrame (FER corpus)
    """
    local_path = os.path.join(
        os.path.dirname(os.path.abspath(__file__)), "data", "flatten_fr_db.csv"
    )
    if os.path.exists(local_path):
        input_path_fer = local_path
    else:
        site_path = site.getsitepackages()[0]
        input_path_fer = os.path.join(
            site_path, "anovos/feature_recommender/data/flatten_fr_db.csv"
        )
    df_input_fer = pd.read_csv(input_path_fer)
    return df_input_fer


def get_column_name(df):
    """

    Parameters
    ----------
    df
        Input DataFrame

    Returns
    -------
    feature_name_column
        Column name of Feature Name in the input DataFrame (string)
    feature_desc_column
        Column name of Feature Description in the input DataFrame (string)
    industry_column
        Column name of Industry in the input DataFrame (string)
    usecase_column
        Column name of Usecase in the input DataFrame (string)
    """
    feature_name_column = str(df.columns.tolist()[0])
    feature_desc_column = str(df.columns.tolist()[1])
    industry_column = str(df.columns.tolist()[2])
    usecase_column = str(df.columns.tolist()[3])
    return (
        feature_name_column,
        feature_desc_column,
        industry_column,
        usecase_column,
    )


def camel_case_split(input):
    """

    Parameters
    ----------
    input
        Input (string) which requires cleaning

    Returns
    -------

    """
    processed_input = ""
    matches = finditer(".+?(?:(?<=[a-z])(?=[A-Z])|(?<=[A-Z])(?=[A-Z][a-z])|$)", input)
    for m in matches:
        processed_input += str(m.group(0)) + str(" ")
    return processed_input


def recommendation_data_prep(df, name_column, desc_column):
    """

    Parameters
    ----------
    df
        Input DataFrame
    name_column
        Column name of Input DataFrame attribute/ feature name (string)
    desc_column
        Column name of Input DataFrame attribute/ feature description (string)


    Returns
    -------
    list_corpus
        List of prepared data for Feature Recommender functions
    return df_prep
        Processed DataFrame for Feature Recommender functions

    """
    if not isinstance(df, pd.DataFrame):
        raise TypeError("Invalid input for df")
    if name_column not in df.columns and name_column is not None:
        raise TypeError("Invalid input for name_column")
    if desc_column not in df.columns and desc_column is not None:
        raise TypeError("Invalid input for desc_column")
    if name_column is None and desc_column is None:
        raise TypeError("Need at least one input for either name_column or desc_column")
    df_prep = copy.deepcopy(df)
    if name_column is None:
        df_prep[desc_column] = df_prep[desc_column].astype(str)
        df_prep_com = df_prep[desc_column]
    elif desc_column is None:
        df_prep[name_column] = df_prep[name_column].astype(str)
        df_prep_com = df_prep[name_column]
    else:
        df_prep[name_column] = df_prep[name_column].str.replace("_", " ")
        df_prep[name_column] = df_prep[name_column].astype(str)
        df_prep[desc_column] = df_prep[desc_column].astype(str)
        df_prep_com = df_prep[[name_column, desc_column]].agg(" ".join, axis=1)
    df_prep_com = df_prep_com.replace({"[^A-Za-z0-9 ]+": " "}, regex=True)
    for i in range(len(df_prep_com)):
        df_prep_com[i] = df_prep_com[i].strip()
        df_prep_com[i] = camel_case_split(df_prep_com[i])
    list_corpus = df_prep_com.to_list()
    return list_corpus, df_prep


def feature_exploration_prep():
    """

    Returns
    -------
    df_input_fer
        DataFrame used in Feature Exploration functions
    """
    df_input_fer = init_input_fer()
    df_input_fer = df_input_fer.rename(columns=lambda x: x.strip().replace(" ", "_"))
    return df_input_fer


def feature_recommendation_prep():
    """

    Returns
    -------
    list_train_fer
        List of prepared data for Feature Recommendation functions
    df_red_fer
        DataFrame used in Feature Recommendation functions
    list_embedding_train_fer
        List of embedding tensor for Feature Recommendation functions
    """
    df_input_fer = init_input_fer()
    (
        feature_name_column,
        feature_desc_column,
        industry_column,
        usecase_column,
    ) = get_column_name(df_input_fer)
    df_groupby_fer = (
        df_input_fer.groupby([feature_name_column, feature_desc_column])
        .agg(
            {
                industry_column: lambda x: ", ".join(set(x.dropna())),
                usecase_column: lambda x: ", ".join(set(x.dropna())),
            }
        )
        .reset_index()
    )
    list_train_fer, df_rec_fer = recommendation_data_prep(
        df_groupby_fer, feature_name_column, feature_name_column
    )

    return list_train_fer, df_rec_fer


class EmbeddingsTrainFer:
    def __init__(self, list_train_fer):
        self.list_train_fer = list_train_fer
        self._embeddings = None

    @property
    def get(self):
        if self._embeddings is None:
            self._embeddings = model_fer.model.encode(
                self.list_train_fer, convert_to_tensor=True
            )
        return self._embeddings
```
</pre>
</details>
## Functions
<dl>
<dt id="anovos.feature_recommender.featrec_init.camel_case_split"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">camel_case_split</span></span>(<span class="n">input)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>input</code></strong></dt>
<dd>Input (string) which requires cleaning</dd>
</dl>
<h2 id="returns">Returns</h2></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def camel_case_split(input):
    """

    Parameters
    ----------
    input
        Input (string) which requires cleaning

    Returns
    -------

    """
    processed_input = ""
    matches = finditer(".+?(?:(?<=[a-z])(?=[A-Z])|(?<=[A-Z])(?=[A-Z][a-z])|$)", input)
    for m in matches:
        processed_input += str(m.group(0)) + str(" ")
    return processed_input
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.featrec_init.detect_model_path"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">detect_model_path</span></span>(<span class="n">)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="returns">Returns</h2>
<dl>
<dt><code>Local Feature Explorer and Recommender semantic model path (if the model is pre-downloaded)</code></dt>
<dd>&nbsp;</dd>
</dl></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def detect_model_path():
    """

    Returns
    -------
    Local Feature Explorer and Recommender semantic model path (if the model is pre-downloaded)
    """
    transformers_path = os.getenv("SENTENCE_TRANSFORMERS_HOME")
    if transformers_path is None:
        try:
            torch_home = _get_torch_home()
        except ImportError:
            torch_home = os.path.expanduser(
                os.getenv(
                    "TORCH_HOME",
                    os.path.join(os.getenv("XDG_CACHE_HOME", "~/.cache"), "torch"),
                )
            )
        transformers_path = os.path.join(torch_home, "sentence_transformers")
    model_path = os.path.join(
        transformers_path, "sentence-transformers_all-mpnet-base-v2"
    )
    return model_path
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.featrec_init.feature_exploration_prep"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">feature_exploration_prep</span></span>(<span class="n">)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="returns">Returns</h2>
<dl>
<dt><code>df_input_fer</code></dt>
<dd>DataFrame used in Feature Exploration functions</dd>
</dl></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def feature_exploration_prep():
    """

    Returns
    -------
    df_input_fer
        DataFrame used in Feature Exploration functions
    """
    df_input_fer = init_input_fer()
    df_input_fer = df_input_fer.rename(columns=lambda x: x.strip().replace(" ", "_"))
    return df_input_fer
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.featrec_init.feature_recommendation_prep"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">feature_recommendation_prep</span></span>(<span class="n">)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="returns">Returns</h2>
<dl>
<dt><code>list_train_fer</code></dt>
<dd>List of prepared data for Feature Recommendation functions</dd>
<dt><code>df_red_fer</code></dt>
<dd>DataFrame used in Feature Recommendation functions</dd>
<dt><code>list_embedding_train_fer</code></dt>
<dd>List of embedding tensor for Feature Recommendation functions</dd>
</dl></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def feature_recommendation_prep():
    """

    Returns
    -------
    list_train_fer
        List of prepared data for Feature Recommendation functions
    df_red_fer
        DataFrame used in Feature Recommendation functions
    list_embedding_train_fer
        List of embedding tensor for Feature Recommendation functions
    """
    df_input_fer = init_input_fer()
    (
        feature_name_column,
        feature_desc_column,
        industry_column,
        usecase_column,
    ) = get_column_name(df_input_fer)
    df_groupby_fer = (
        df_input_fer.groupby([feature_name_column, feature_desc_column])
        .agg(
            {
                industry_column: lambda x: ", ".join(set(x.dropna())),
                usecase_column: lambda x: ", ".join(set(x.dropna())),
            }
        )
        .reset_index()
    )
    list_train_fer, df_rec_fer = recommendation_data_prep(
        df_groupby_fer, feature_name_column, feature_name_column
    )

    return list_train_fer, df_rec_fer
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.featrec_init.get_column_name"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">get_column_name</span></span>(<span class="n">df)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>df</code></strong></dt>
<dd>Input DataFrame</dd>
</dl>
<h2 id="returns">Returns</h2>
<dl>
<dt><code>feature_name_column</code></dt>
<dd>Column name of Feature Name in the input DataFrame (string)</dd>
<dt><code>feature_desc_column</code></dt>
<dd>Column name of Feature Description in the input DataFrame (string)</dd>
<dt><code>industry_column</code></dt>
<dd>Column name of Industry in the input DataFrame (string)</dd>
<dt><code>usecase_column</code></dt>
<dd>Column name of Usecase in the input DataFrame (string)</dd>
</dl></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def get_column_name(df):
    """

    Parameters
    ----------
    df
        Input DataFrame

    Returns
    -------
    feature_name_column
        Column name of Feature Name in the input DataFrame (string)
    feature_desc_column
        Column name of Feature Description in the input DataFrame (string)
    industry_column
        Column name of Industry in the input DataFrame (string)
    usecase_column
        Column name of Usecase in the input DataFrame (string)
    """
    feature_name_column = str(df.columns.tolist()[0])
    feature_desc_column = str(df.columns.tolist()[1])
    industry_column = str(df.columns.tolist()[2])
    usecase_column = str(df.columns.tolist()[3])
    return (
        feature_name_column,
        feature_desc_column,
        industry_column,
        usecase_column,
    )
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.featrec_init.init_input_fer"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">init_input_fer</span></span>(<span class="n">)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="returns">Returns</h2>
<dl>
<dt><code>Loading the Feature Explorer and Recommender (FER) Input DataFrame (FER corpus)</code></dt>
<dd>&nbsp;</dd>
</dl></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def init_input_fer():
    """

    Returns
    -------
    Loading the Feature Explorer and Recommender (FER) Input DataFrame (FER corpus)
    """
    local_path = os.path.join(
        os.path.dirname(os.path.abspath(__file__)), "data", "flatten_fr_db.csv"
    )
    if os.path.exists(local_path):
        input_path_fer = local_path
    else:
        site_path = site.getsitepackages()[0]
        input_path_fer = os.path.join(
            site_path, "anovos/feature_recommender/data/flatten_fr_db.csv"
        )
    df_input_fer = pd.read_csv(input_path_fer)
    return df_input_fer
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.featrec_init.model_download"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">model_download</span></span>(<span class="n">)</span>
</code></dt>
<dd>
<div class="desc"></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def model_download():
    print("Starting the Semantic Model download")
    SentenceTransformer("all-mpnet-base-v2")
    print("Model downloading finished")
```
</pre>
</details>
</dd>
<dt id="anovos.feature_recommender.featrec_init.recommendation_data_prep"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">recommendation_data_prep</span></span>(<span class="n">df, name_column, desc_column)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="parameters">Parameters</h2>
<dl>
<dt><strong><code>df</code></strong></dt>
<dd>Input DataFrame</dd>
<dt><strong><code>name_column</code></strong></dt>
<dd>Column name of Input DataFrame attribute/ feature name (string)</dd>
<dt><strong><code>desc_column</code></strong></dt>
<dd>Column name of Input DataFrame attribute/ feature description (string)</dd>
</dl>
<h2 id="returns">Returns</h2>
<dl>
<dt><code>list_corpus</code></dt>
<dd>List of prepared data for Feature Recommender functions</dd>
<dt><code>return df_prep</code></dt>
<dd>Processed DataFrame for Feature Recommender functions</dd>
</dl></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def recommendation_data_prep(df, name_column, desc_column):
    """

    Parameters
    ----------
    df
        Input DataFrame
    name_column
        Column name of Input DataFrame attribute/ feature name (string)
    desc_column
        Column name of Input DataFrame attribute/ feature description (string)


    Returns
    -------
    list_corpus
        List of prepared data for Feature Recommender functions
    return df_prep
        Processed DataFrame for Feature Recommender functions

    """
    if not isinstance(df, pd.DataFrame):
        raise TypeError("Invalid input for df")
    if name_column not in df.columns and name_column is not None:
        raise TypeError("Invalid input for name_column")
    if desc_column not in df.columns and desc_column is not None:
        raise TypeError("Invalid input for desc_column")
    if name_column is None and desc_column is None:
        raise TypeError("Need at least one input for either name_column or desc_column")
    df_prep = copy.deepcopy(df)
    if name_column is None:
        df_prep[desc_column] = df_prep[desc_column].astype(str)
        df_prep_com = df_prep[desc_column]
    elif desc_column is None:
        df_prep[name_column] = df_prep[name_column].astype(str)
        df_prep_com = df_prep[name_column]
    else:
        df_prep[name_column] = df_prep[name_column].str.replace("_", " ")
        df_prep[name_column] = df_prep[name_column].astype(str)
        df_prep[desc_column] = df_prep[desc_column].astype(str)
        df_prep_com = df_prep[[name_column, desc_column]].agg(" ".join, axis=1)
    df_prep_com = df_prep_com.replace({"[^A-Za-z0-9 ]+": " "}, regex=True)
    for i in range(len(df_prep_com)):
        df_prep_com[i] = df_prep_com[i].strip()
        df_prep_com[i] = camel_case_split(df_prep_com[i])
    list_corpus = df_prep_com.to_list()
    return list_corpus, df_prep
```
</pre>
</details>
</dd>
</dl>
##
Classes
<dl>
<dt id="anovos.feature_recommender.featrec_init.EmbeddingsTrainFer"><code class="flex name class">
<span>class <span class="ident">EmbeddingsTrainFer</span></span>
<span>(</span><span>list_train_fer)</span>
</code></dt>
<dd>
<div class="desc"></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
class EmbeddingsTrainFer:
    def __init__(self, list_train_fer):
        self.list_train_fer = list_train_fer
        self._embeddings = None

    @property
    def get(self):
        if self._embeddings is None:
            self._embeddings = model_fer.model.encode(
                self.list_train_fer, convert_to_tensor=True
            )
        return self._embeddings
```
</pre>
</details>
<h3>Instance variables</h3>
<dl>
<dt id="anovos.feature_recommender.featrec_init.EmbeddingsTrainFer.get"><code class="name">var <span class="ident">get</span></code></dt>
<dd>
<div class="desc"></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
@property
def get(self):
    if self._embeddings is None:
        self._embeddings = model_fer.model.encode(
            self.list_train_fer, convert_to_tensor=True
        )
    return self._embeddings
```
</pre>
</details>
</dd>
</dl>
</dd>
</dl>