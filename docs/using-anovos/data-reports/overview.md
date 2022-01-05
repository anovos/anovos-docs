# Creating Data Reports with Anovos

_Anovos_ includes capabilities to generate comprehensive _Data Reports_ that describe a dataset
and its processing. Data reports are an important component of many data governance concepts.

## 📑 How are reports generated?

_Anovos_ generates reports in two steps:

1. The data that will be included in the report is generated using the functions of the 
   [`data_analyzer`](../../docs/anovos-modules-overview/data-analyzer/index.md) module.
   As all _Anovos_ data operations, this happens in a distributed fashion,
   fully utilizing the power of _Apache Spark_.
   We call the result the "intermediate report."

2. The generated data is processed and the final report is generated.

You can configure a report and trigger its generation in two ways:
Using the configuration file, or through individual modules.

## 📋 Generating data reports via the configuration file

*Here the user specifies the desired options / input parameters as done for other modules*

## 🦄 Generating data reports through individual modules

*Here the user can pick up the relevant functions specific to the reporting module and execute*
