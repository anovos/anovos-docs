# Anovos Product Roadmap

_Anovos_ is built and released as an open source project based on our experience using massive data sets
to produce predictive features. At [Mobilewalla](https://www.mobilewalla.com), we process terabytes of 
mobile engagement signals daily to mine consumer behavior and use features from that data to build distributed
machine learning models to solve the respective business problems.

In this journey, we faced lots of challenges by not having a comprehensive and scalable library.
After realizing the unavailability of such libraries, we designed and implemented _Anovos_ as an
open source library for every data scientists’ use. 

![https://anovos.github.io/anovos-docs/assets/roadmap.png](https://anovos.github.io/anovos-docs/assets/roadmap.png)

## The Roadmap
We plan to bring full functionality to _Anovos_ over the course of three major releases: Alpha, Beta, and versio 1.0.

### Alpha Release

The Alpha release of _Anovos_ has all the essential data ingestion and comprehensive data analysis functionalities,
as well as the data pre-processing and cleaning mechanisms. It also has some key differentiating functionalities,
like data drift and stability computations, which are crucial in deciding the need for model refreshing/tweaking
options. Another benefit of Anovos is a dynamic visualization component configured based on data ingestion pipelines. Every data metric computed from the Anovos ingestion process can be visualized and utilized for CXO level decision-making.

### Beta Release

In the Beta release of _Anovos_, the library will support ingesting from cloud service providers
like MS Azure and will have mechanisms to read/write different file formats such as Avro and nested Json.
It will also enable ingesting various data types (see the above figure for the details).

The key differentiating functionality of beta release would be the “Feature Wiki” for data scientists
and end-users to resolve their cold-start problems, which will immensely reduce their literature search time.

The Beta release will also have another explain each attributes’ contribution in feature building which we 
call the “Attribute -> Feature” mapper.

### Version 1.0
We'll release the GA release of _Anovos_ in March 2022 with the functionalities to support an end-to-end
machine learning workflow. It will be able to store the generated features in an open source feature store,
like Feast. It will also support running open source based Auto ML models and ML workflow integration.

This release will also include a mechanism to explain the model behavior by having the respective Shapley values. 