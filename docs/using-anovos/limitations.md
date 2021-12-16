# Current Limitations of Anovos
 
- Anovos currently only supports numerical and categorical columns. Other data types such as date, time stamp, array, struct array etc may be supported in the later releases.
- Computing mode and/or distinct count are most expensive operations in Anovos. We aim to further optimize them in the upcoming releases.
- Correlation Matrix may throw memory issues if very high cardinality categorical features are involved – a limitation that was propagated from phik library. Therefore, it is highly recommended to drop columns with high IDness.
- Geospatial columns like geohash or lat/long can only be analysed as categorical (geohash) or numerical (lat/long). Functionalities specific to geospatial features will be supported in the later releases.
- Invalid Entries detection currently looks for only selected suspicious patterns. We aim to add more flexibility to this function with user provided pattern i.e. RegEx support. Also, this function may result in some false positives detection. Therefore will need caution while using the inbuilt treatment.
- Anovos currently relies on Spark automatic schema detection. In case, some numerical columns were deliberately saved as string, they will remain categorical column in the subsequent dataframe read (exception with csv file format).
    - Stability Index can be calculated only for numerical columns
    - Exception and error handling needs to be tightened.
    - Report module will be subjected to further optimization.
