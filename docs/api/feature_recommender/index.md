# <code>feature_recommender</code>
<p>Feature engineering has always played a crucial role in solving any Machine learning (ML) related problems.</p>
<p>Features/Predictors decide whether the Machine Learning projects are successful or not. However, coming up with good
&amp; intuitive features is not an easy task. Identifying list of potential features to build is a very hard to come up
and requires both expertise in domain knowledge and technical aspects of Machine learning. In fact, 80% of Data
Scientists&#39; time are being spent on data wrangling and Feature Engineering tasks, and only 20% is for fine-tuning
the model and testing out. Building features from scratch is a cold-start problem for any Data Scientists to figure
out what features would be used to help them in creating their models.</p>
<p>There are many tools to help Data Scientist to narrow down the features, but they are either not scalable,
or very comprehensive to understand and operate. Here, within ANOVOS V0.2, we launch an open-source tool,
Feature Explorer and Recommender (FER) module, in order to help the Machine Learning community with these cold start
Feature Engineering problems.</p>
<p>With Feature Explorer and Recommender module, we mainly address two problems:</p>
<ul>
<li>
<p>Create a platform for Data Scientists to explore available/already used features based on their interest of
Industries/Domain and Use cases</p>
</li>
<li>
<p>Recommend better features for Data Scientists to address cold-start problems (based
on the data they have in hand)</p>
</li>
</ul>
<p>Feature Explorer and Recommender utilizes Semantic similarity based Language Modeling in Natural Language Processing
(NLP). Semantic matching techniques aims to determine the similarity between words, lines, and sentences through
multiple metrics.</p>
<p>In this module, we use <a href="https://huggingface.co/sentence-transformers/all-mpnet-base-v2">all-mpnet-base-v2</a>
for our semantic model. This model is built based upon Microsoft <a href="https://arxiv.org/abs/2004.09297">Mpnet</a> base model,
masked and permuted pre-training for language understanding. Its performance triumphs BERT, XLNet, RoBERTa for language
modeling and text recognition. The importance features of this model are,</p>
<ul>
<li>
<p>Trained on more than 1 billion training pairs, including around 300 millions research paper citation pairs</p>
</li>
<li>
<p>Fine-tuned using cosine similarity from sentence pairs, then apply cross entropy loss by comparing true pairs</p>
</li>
</ul>
<p>Our solution consists of 3 main steps:</p>
<ul>
<li>Using the pretrained model, convert textual data into tensors (Text Embedding Technique)</li>
<li>Compute similarity scores of each input attribute name &amp; description across both corpora
(Anovos Feature Corpus &amp; User Data Dictionary Corpus)</li>
<li>Sort the results and get the matches for each input feature based on their scores</li>
</ul>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
<pre>
```python
"""Feature engineering has always played a crucial role in solving any Machine learning (ML) related problems.

Features/Predictors decide whether the Machine Learning projects are successful or not. However, coming up with good
& intuitive features is not an easy task. Identifying list of potential features to build is a very hard to come up
and requires both expertise in domain knowledge and technical aspects of Machine learning. In fact, 80% of Data
Scientists&#39; time are being spent on data wrangling and Feature Engineering tasks, and only 20% is for fine-tuning
the model and testing out. Building features from scratch is a cold-start problem for any Data Scientists to figure
out what features would be used to help them in creating their models.

There are many tools to help Data Scientist to narrow down the features, but they are either not scalable,
or very comprehensive to understand and operate. Here, within ANOVOS V0.2, we launch an open-source tool,
Feature Explorer and Recommender (FER) module, in order to help the Machine Learning community with these cold start
Feature Engineering problems.

With Feature Explorer and Recommender module, we mainly address two problems:

- Create a platform for Data Scientists to explore available/already used features based on their interest of
Industries/Domain and Use cases

- Recommend better features for Data Scientists to address cold-start problems (based
on the data they have in hand)

Feature Explorer and Recommender utilizes Semantic similarity based Language Modeling in Natural Language Processing
(NLP). Semantic matching techniques aims to determine the similarity between words, lines, and sentences through
multiple metrics.

In this module, we use [all-mpnet-base-v2](https://huggingface.co/sentence-transformers/all-mpnet-base-v2)
for our semantic model. This model is built based upon Microsoft [Mpnet](https://arxiv.org/abs/2004.09297) base model,
masked and permuted pre-training for language understanding. Its performance triumphs BERT, XLNet, RoBERTa for language
 modeling and text recognition. The importance features of this model are,

- Trained on more than 1 billion training pairs, including around 300 millions research paper citation pairs

- Fine-tuned using cosine similarity from sentence pairs, then apply cross entropy loss by comparing true pairs

Our solution consists of 3 main steps:

- Using the pretrained model, convert textual data into tensors (Text Embedding Technique)
- Compute similarity scores of each input attribute name & description across both corpora
  (Anovos Feature Corpus & User Data Dictionary Corpus)
- Sort the results and get the matches for each input feature based on their scores
"""
```
</pre>
</details>
## Sub-modules
<dl>
<dt><code class="name"><a title="anovos.feature_recommender.featrec_init" href="featrec_init.html">anovos.feature_recommender.featrec_init</a></code></dt>
<dd>
<div class="desc"></div>
</dd>
<dt><code class="name"><a title="anovos.feature_recommender.feature_exploration" href="feature_exploration.html">anovos.feature_recommender.feature_exploration</a></code></dt>
<dd>
<div class="desc"><p>Feature explorer helps list down the potential features from our corpus based
on user defined industry or/and use case.</p></div>
</dd>
<dt><code class="name"><a title="anovos.feature_recommender.feature_recommendation" href="feature_recommendation.html">anovos.feature_recommender.feature_recommendation</a></code></dt>
<dd>
<div class="desc"><p>Feature recommender recommends features based on ingested data dictionary by the user.</p></div>
</dd>
</dl>