# Anovos Product Roadmap

_Anovos_ is built and released as an open source project based on our experience in handling massive data sets
to produce predictive features. At [Mobilewalla](https://www.mobilewalla.com), we process terabytes of 
mobile engagement signals daily to mine consumer behavior and use features from that data to build distributed
machine learning models to solve a wide range of business problems.

On this journey, we faced lots of challenges due to the lack of a comprehensive and scalable library.
After realizing the unavailability of such libraries, we designed and implemented _Anovos_ as an
open source library for every data scientists’ use. 

![https://anovos.github.io/anovos-docs/assets/roadmap.png](https://anovos.github.io/anovos-docs/assets/roadmap.png)

## 🛣 The Roadmap
We plan to bring fully functional _Anovos_ over the course of three major releases: V0.1,V0.2 and version 1.0.

### V0.1 (November 2021)

The V0.1 release of _Anovos_ had all the essential data ingestion and comprehensive data analysis functionalities,
as well as the data pre-processing and cleaning mechanisms. It also included some key differentiating functionalities,
like data drift and stability computations, which are crucial in deciding the need for model refresh/tweak
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
- **Runtime Environment support**
      - Local 
      - Docker-based
      - AWS EMR 
- **Report Visualization**
      - Comprehensive 360 degree view report of the ingested data (Numerical & Categorical)
          - Executive summary
          - Wiki
          - Descriptive statistics
          - Quality Checker
          - Attribute association
          - Data drift & stability

### V0.2 Release (March 2022)

In this release of _Anovos_, the library will support ingesting from cloud service providers
like MS Azure and will also have mechanisms to read/write different file formats such as Avro and nested Json.
It will also enable ingesting various data types (see the above figure for the details).

The key differentiating functionality of this release is that “Feature Explorer & Feature Recommender” for data scientists
and end-users to resolve their cold-start problems, which will immensely reduce their literature review time. 

The V0.2 release will also have another important capability named as Feature Stability estimator based on the composition of a given feature using set of attribute/s. This will greatly benefit data scientists to understand the potential feature instabilities which could harm the resillency of a ML model.

With the V0.2 release Anovos can be used in day to day practices of any Data Scientists or Data Analysts 

#### Details

- **Data Ingest**
    - Microsoft Azure Blob Storage integration
    - Support for Avro and nested JSON files
    - Support for additional data types: Time stamps columns 
    - Support for Timeseries data ingestion
- **Data Cleaning and Transformation**
    - Parsing
    - Merging
    - Converting/Coding
    - Derivations
    - Calculations
    - Imputations
    - Auto encoders
    - Dimension reduction
    - Date/Time related transformations
- **Feature Explorer / Feature recommender (Semantic search enabled)**
  To recommend potential features based on the industry, use case, and the ingested data dictionary
    - Industry specific use cases and respective features
        - Telco
        - BFSI
        - Retail
        - Healthcare
        - Transportation
        - Supply chain
    - Recommendations are enabled by Semantic search capability
    - Supported by pre-compiled feature corpus 
- **Feature Stability**
    - This will be an extension of the attribute stability capabilities of the V0.1 release
- **Extended Spark & Python support**
    - Compatibility with different Spark & Python versions
    - Apache Spark 2.4.x on Java 8 with Python 3.7.x
    - Apache Spark 3.1.x on Java 11 with Python 3.9.x
    - Apache Spark 3.2.x on Java 11 with Python 3.9.x
- **Runtime Environment support**
    - Microsoft Azure Databricks

### Version 1.0 (Summer of 2022)

We'll release version 1.0 of _Anovos_ in June 2022 with the functionalities to support an end-to-end
machine learning workflow. It will be able to store the generated features in an open source feature store,
like Feast. It will also support running open source based Auto ML models and ML workflow integration.

This release will also include a mechanism to explain the model behavior by having the respective Shapley values. 

#### Details

- **Feature Store Integration:** APIs to integrate _Anovos_ with existing OSS Feature Stores
- **Explainable AI:** SHAP value computations
- **Auto ML Integration:** APIs to integrate _Anovos_ with existing OSS Auto ML solutions
- **ML Flow** workflow integration
