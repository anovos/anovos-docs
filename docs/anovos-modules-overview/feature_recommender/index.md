## **Module: Anovos.feature explorer & feature recommender**

Feature engineering has always played a crucial role in solving any Machine learning (ML) related problems. Features/Predictors decide whether the Machine Learning projects are successful or not. However, coming up with good & intuitive features is not an easy task. Identifying list of potential features to build is a very hard to come up and requires both expertise in domain knowledge and technical aspects of Machine learning. In fact, 80% of Data Scientists&#39; time are being spent on data wrangling and Feature Engineering tasks, and only 20% is for fine-tuning the model and testing out. Building features from scratch is a cold-start problem for any Data Scientists to figure out what features would be used to help them in creating their models.

There are many tools to help Data Scientist to narrow down the features, but they are either not scalable, or very comprehensive to understand and operate. Here, within ANOVOS V0.2, we launch an open-source tool, Feature Explorer and Recommender (FER) module, in order to help the Machine Learning community with these cold start Feature Engineering problems. 

With Feature Explorer and Recommender module, we mainly address two problems:

- Create a platform for Data Scientists to explore available/already used features based on their interest of Industries/Domain and Use cases
- Recommend better features for Data Scientists to address cold-start problems (based on the data they have in hand)

Feature Explorer and Recommender utilizes Semantic similarity based Language Modeling in Nature Language Processing (NLP). Semantic matching techniques aims to determine the similarity between words, lines, and sentences through multiple metrics.

In this module, we use [all-mpnet-base-v2](https://huggingface.co/sentence-transformers/all-mpnet-base-v2) for our semantic model. This model is built based upon Microsoft [Mpnet](https://arxiv.org/abs/2004.09297) base model, masked and permuted pre-training for language understanding. Its performance triumphs BERT, XLNet, RoBERTa for language modeling and text recognition. The importance features of this model are,

- Trained on more than 1 billion training pairs, including around 300 millions research paper citation pairs
- Fine-tuned using cosine similarity from sentence pairs, then apply cross entropy loss by comparing true pairs

Our solution consists of 3 main steps:

- Using the pretrained model, convert texual data into tensors (Text Embedding Technique)
- Compute similarity scores of each input attribute name & description across both corpuses (Anovos Feature Corpus & User Data Dictionary)
- Sort the results and get the matches for each input feature based on their scores

See below for the solution workflow of FER for further understanding of our solution.

![Solution Details Diagram](https://github.com/anovos/anovos-docs/blob/feature_recommender_docs/docs/assets/Feature_Recommender_Workflow.png)

As mentioned earlier, our solution design consists of 2 sub-modules in it, namely Feature Explorer & Feature Recommender. Below we detail the list of functions supported under each module.

### *Feature Explorer:*

Feature explorer helps list down the potential features from our corpus based on user defined industry or/and use case.

### list\_all\_industry

Argument: None

This function lists down all the Industries that are supported in Feature Recommender module.

### list\_all\_usecase

Argument: None

This function lists down all the Use cases that are supported in Feature Recommender module

### list\_all\_pair

Argument: None

This function lists down all the Industry/Use case pairs that are supported in Feature Recommender module

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

Feature recommender recommends features based on ingested data dictionary by the user. 

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
