# Anovos Product Roadmap

_Anovos_ is built and released as an open source project based on our experience using massive data sets
to produce predictive features. At [Mobilewalla](https://www.mobilewalla.com), we process terabytes of 
mobile engagement signals daily to mine consumer behavior and use features from that data to build distributed
machine learning models to solve the respective business problems.

In this journey, we faced lots of challenges by not having a comprehensive and scalable library.
After realizing the unavailability of such libraries, we designed and implemented _Anovos_ as an
open source library for every data scientists’ use. 

![https://anovos.github.io/anovos-docs/assets/roadmap.png](https://anovos.github.io/anovos-docs/assets/roadmap.png)

## 🛣 The Roadmap
We plan to bring full functionality to _Anovos_ over the course of three major releases: Alpha, Beta, and versio 1.0.

### Alpha Release

The Alpha release of _Anovos_ has all the essential data ingestion and comprehensive data analysis functionalities,
as well as the data pre-processing and cleaning mechanisms. It also has some key differentiating functionalities,
like data drift and stability computations, which are crucial in deciding the need for model refreshing/tweaking
options. Another benefit of _Anovos_ is a dynamic visualization component configured based on data ingestion pipelines.
Every data metric computed from the _Anovos_ ingestion process can be visualized and utilized for CXO level decision-making.

#### Details

- **Data Ingest**
  - AWS S3 Storage integration
  - Read and write to/from local files
  - Column selection and renaming
  - Support for Parquet and CSV files
  - Support for numerical and categorical data types
- **Data Analyzer and Diagnostics**
  - Frequency analysis
  - Attribute/feature vs. target
  - Attribute/feature interaction/association
- **Data Preprocessing and Cleaning**
  - Outlier detection (IQR/Standardization)
  - Treatment of invalid values
  - Missing attributes analysis
- **Data Health and Monitoring**
  - Data drift identification (Hellinger Distance, KS, JSD, and PSI)
  - Attribute stability analysis
  - Overall data quality analysis

### Beta Release

In the Beta release of _Anovos_, the library will support ingesting from cloud service providers
like MS Azure and will have mechanisms to read/write different file formats such as Avro and nested Json.
It will also enable ingesting various data types (see the above figure for the details).

The key differentiating functionality of beta release would be the “Feature Wiki/ Feature Recommender” for data scientists
and end-users to resolve their cold-start problems, which will immensely reduce their literature review time.

The Beta release will also have another explainer of each attributes’ contribution in feature building which we 
call the “Attribute -> Feature” mapper.

#### Details

- **Data Ingest**
  - Microsoft Azure Blob Storage integration
  - Support for Avro and nested JSON files
  - Support for additional data types: Time series, time stamps, RegEx
- **Data Cleaning and Transformation**
  - Parsing
  - Merging
  - Converting/Coding
  - Derivations
  - Calculations
  - Imputations
  - Auto encoders
  - Dimension reduction
- **Feature Wiki**
  - Industry specific use cases and respective features
    - Telco
    - BFSI
    - Healthcare
- **Attribute to Feature Mapping**

### Version 1.0

We'll release version 1.0 of _Anovos_ in June 2022 with the functionalities to support an end-to-end
machine learning workflow. It will be able to store the generated features in an open source feature store,
like Feast. It will also support running open source based Auto ML models and ML workflow integration.

This release will also include a mechanism to explain the model behavior by having the respective Shapley values. 

#### Details

- **Feature Store Integration:** APIs to integrate _Anovos_ with existing OSS Feature Stores
- **Explainable AI:** SHAP value computations
- **Auto ML Integration:** APIs to integrate _Anovos_ with existing OSS Auto ML solutions
- **ML Flow** workflow integration