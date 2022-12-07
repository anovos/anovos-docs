# <code>feature_retrieval</code>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
import sys
from datetime import datetime

import feast
import pandas as pd


def retrieve_historical_feature_demo(repo_path: str):
    income_entities = pd.DataFrame.from_dict(
        {
            "ifa": [
                "27a",
                "30a",
                "475a",
                "965a",
                "1678a",
                "1698a",
                "1807a",
                "1951a",
                "2041a",
                "2215a",
            ],
            "event_time": [
                datetime.now(),
                datetime.now(),
                datetime.now(),
                datetime.now(),
                datetime.now(),
                datetime.now(),
                datetime.now(),
                datetime.now(),
                datetime.now(),
                datetime.now(),
            ],
        }
    )

    fs = feast.FeatureStore(repo_path=repo_path)
    income_features_df = fs.get_historical_features(
        entity_df=income_entities,
        features=[
            "income_view:income",
            "income_view:latent_0",
            "income_view:latent_1",
            "income_view:latent_2",
            "income_view:latent_3",
        ],
    ).to_df()
    print(income_features_df.head())

    # train model from here ...

    feature_service = fs.get_feature_service("income_feature_service")
    income_features_by_service_df = fs.get_historical_features(
        features=feature_service, entity_df=income_entities
    ).to_df()
    print(income_features_by_service_df.head())


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Please, provide a path to anovos feature repo!")
        exit(1)
    path = sys.argv[1]
    retrieve_historical_feature_demo(repo_path=path)
```
</pre>
</details>
## Functions
<dl>
<dt id="anovos.feature_store.feature_retrieval.retrieve_historical_feature_demo"><code class="name flex hljs csharp">
<span class="k">def</span> <span class="nf"><span class="ident">retrieve_historical_feature_demo</span></span>(<span class="n">repo_path: str)</span>
</code></dt>
<dd>
<div class="desc"></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
def retrieve_historical_feature_demo(repo_path: str):
    income_entities = pd.DataFrame.from_dict(
        {
            "ifa": [
                "27a",
                "30a",
                "475a",
                "965a",
                "1678a",
                "1698a",
                "1807a",
                "1951a",
                "2041a",
                "2215a",
            ],
            "event_time": [
                datetime.now(),
                datetime.now(),
                datetime.now(),
                datetime.now(),
                datetime.now(),
                datetime.now(),
                datetime.now(),
                datetime.now(),
                datetime.now(),
                datetime.now(),
            ],
        }
    )

    fs = feast.FeatureStore(repo_path=repo_path)
    income_features_df = fs.get_historical_features(
        entity_df=income_entities,
        features=[
            "income_view:income",
            "income_view:latent_0",
            "income_view:latent_1",
            "income_view:latent_2",
            "income_view:latent_3",
        ],
    ).to_df()
    print(income_features_df.head())

    # train model from here ...

    feature_service = fs.get_feature_service("income_feature_service")
    income_features_by_service_df = fs.get_historical_features(
        features=feature_service, entity_df=income_entities
    ).to_df()
    print(income_features_by_service_df.head())
```
</pre>
</details>
</dd>
</dl>