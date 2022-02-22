## **Module: Anovos.feature\_recommender**

Features have always been a crucial part in solving any Data Science related problems. They are the most important factors, deciding whether the Machine Learning projects are successful or not. However, coming up with good features is not a simple task. It is usually very difficult and requires very expert knowledge in both domains and Data Science technical aspects. In fact, 80% of Data Scientists&#39; time are being spent on data wrangling and Feature Engineering tasks, and only 20% is for fine-tuning the model and testing out. Starting from scratch, this is a cold-start problem for any Data Scientist to figure out what features would be used to help them in creating their models.

There are many tools to help Data Scientist to narrow down the features, but they are either not scalable, or very comprehensive to understand and operate. Here, within ANOVOS Beta Release, we launch an open-source tool, Feature Explorer and Recommender module, in order to help the Machine Learning community with these Feature Engineering problems.

With Feature Explorer and Recommender module, we address 2 main statements:

- Create a platform for Data Scientists to explore available features based on their interest of Industries and Use cases
- Recommend better features for Data Scientists to address cold-start problems

Feature Explorer and Recommender utilizes Semantic Matching method for Language Modeling and Nature Language Processing (NLP). Semanticmatching techniques aim to determine the similarity between words, lines, and sentences through multiple metrics.

In this module, we use all-mpnet-base-v2([https://huggingface.co/sentence-transformers/all-mpnet-base-v2](https://huggingface.co/sentence-transformers/all-mpnet-base-v2)) for our semantic model. This model is built upon Microsoft Mpnet-base base model, Masked and Permuted Pre-training for Language Understanding. Its performance triumphs BERT, XLNet, RoBERTa for language modeling and text recognition:

- Trained on more than 1 billion training pairs, including around 300 millions research paper citation pairs
- Fine-tuned using cosine similarity from sentence pairs, then apply cross entropy loss by comparing true pairs

Our solution consists of 3 main steps:

- Using the pretrained model, convert texture data into tensors (Text Embedding Technique)
- Compute similarity scores of each input feature across both corpuses
- Sort the results and get the matches for each input feature based on their scores

### *Feature Explorer:*

### list\_all\_industry

Argument: None

This function lists down all the Industries that are supported in Feature Recommender package.

### list\_all\_usecase

Argument: None

This function lists down all the Use cases that are supported in Feature Recommender Package

### list\_all\_pair

Argument: None

This function lists down all the Industry/Use case pairs that are supported in Feature Recommender Package

### list\_usecase\_by\_industry

Arguments:

- *industry*(string): Input industry from user
- *semantic*(boolean): Whether the input needs to go through semantic matching or not. Default is True

This function lists down all the Use cases that are supported in Feature Recommender Package based on the Input Industry

### list\_industry\_by\_usecase

Arguments:

- *usecase*(string): Input usecase from user
- *semantic*(boolean): Whether the input needs to go through semantic matching or not. Default is True

This function lists down all the Use cases that are supported in Feature Recommender Package based on the Input Industry

### list\_feature\_by\_industry

Arguments:

- *industry*(string): Input industry from user
- *num\_of\_feat*(int): Number of features displayed. Default is 100
- *semantic*(boolean): Whether the input needs to go through semantic matching or not. Default is True

This function lists down all the Features that are available in Feature Recommender Package based on the Input Industry

The output is returned in the form of a DataFrame. Columns are:

- Feature Name: Name of the suggested Feature
- Feature Description: Description of the suggested Feature
- Industry: Industry name of the suggested Feature
- Usecase: Usecase name of the suggested Feature
- Source: Source of the suggested Feature

The list of features is sorted by the Usecases&#39; Feature Popularity to the Input Industry.

### list\_feature\_by\_usecase

Arguments:

- *usecase*(string): Input industry from user
- *num\_of\_feat*(int): Number of features displayed. Default is 100
- *semantic*(boolean): Whether the input needs to go through semantic matching or not. Default is True

This function lists down all the Features that are available in Feature Recommender Package based on the Input Usecase

The output is returned in the form of a DataFrame. Columns are:

- Feature Name: Name of the suggested Feature
- Feature Description: Description of the suggested Feature
- Industry: Industry name of the suggested Feature
- Usecase: Usecase name of the suggested Feature
- Source: Source of the suggested Feature

The list of features is sorted by the Industries&#39; Feature Popularity to the Input Usecase.

### list\_feature\_by\_pair

Arguments:

- *industry*(string): Input industry from user
- *usecase*(string): Input usecase from user
- *num\_of\_feat*(int): Number of features displayed. Default is 100
- *semantic*(boolean): Whether the input needs to go through semantic matching or not. Default is True

This function lists down all the Features that are available in Feature Recommender Package based on the Input Industry/ Usecase pair

The output is returned in the form of a DataFrame. Columns are:

- Feature Name: Name of the suggested Feature
- Feature Description: Description of the suggested Feature
- Industry: Industry name of the suggested Feature
- Usecase: Usecase name of the suggested Feature
- Source: Source of the suggested Feature

### *Feature Recommender:*

### feature\_recommendation

Arguments:

- *df*(DataFrame): Input User Attribute Dictionary
- *name\_column*(string): Name of the column contains attribute names. Default is None
- *desc\_column*(string): Name of the column contains attribute description. Default is None
- *suggested\_industry*(string): Input goal industry from user. Default is all
- *suggested\_usecase*(string): Input goal usecase from user. Default is all
- *semantic*(boolean): Whether the input needs to go through semantic matching or not. Default is True
- *top\_n*(int): Number of most similar features displayed matched to input attributes. Default is 2
- *threshold*(float): Floor limit of the similarity score to be matched. Default is 0.3

This function recommends features to users based on their input attributes, and their goal industry and/or use case

The output is returned in the form of a DataFrame. Columns are:

- Input Attribute Name: Name of the input Attribute
- Input Attribute Description: Description of the input Attribute
- Recommended Feature Name: Name of the recommended Feature
- Recommended Feature Description: Description of the recommended Feature
- Feature Similarity Score: Semantic similarity score between input Attribute and recommended Feature
- Industry: Industry name of the recommended Feature
- Usecase: Usecase name of the recommended Feature
- Source: Source of the recommended Feature

### find\_attr\_by\_relevance

Arguments:

- *df*(DataFrame): Input User Attribute Dictionary
- *building\_corpus*(list): Input User Feature Corpus
- *name\_column*(string): Name of the column contains attribute names. Default is None
- *desc\_column*(string): Name of the column contains attribute description. Default is None
- *threshold*(float): Floor limit of the similarity score to be matched. Default is 0.3

This function is to provide a comprehensive mapping method from users&#39; input attributes to their own feature corpus, and therefore, help with the process of creating features in cold-start problems

The output is returned in the form of a DataFrame. Columns are:

- Input Feature Desc: Description of the input Feature
- Recommended Input Attribute Name: Name of the recommended Feature
- Recommended Input Attribute Description: Description of the recommended Feature
- Input Attribute Similarity Score: Semantic similarity score between input Attribute and recommended Feature

### sankey\_visualization

Arguments:

- *df*(DataFrame): Input DataFrame. This DataFrame needs to be output of feature\_recommendation or find\_attr\_by\_relevance, or in the same format.
- *industry\_included*(boolean): Whether the plot needs to include industry mapping or not. Default is False
- *usecase\_included*(boolean): Whether the plot needs to include usecase mapping or not. Default is False

This function is to visualize Feature Recommendation functions through Sankey plots