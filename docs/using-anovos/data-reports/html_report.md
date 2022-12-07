# Generating HTML Reports with Anovos

The final output is generated in form of HTML report. This has 6 sections viz. Executive Summary, Wiki, Descriptive Statistics, Quality Check, Attribute Associations, Data Drift & Data Stability at most which can be seen on the basis of user input. We’ve tried to detail each section based on the analysis performed on a publicly available dataset.


### Executive Summary

![](https://anovos.github.io/anovos-docs/assets/html-reports/executive-report-1.png)

The **"Executive Summary"** gives an overall summary of the key statistics from the analyzed data. 

- **1 & 2** specifies about the dimensions of data & nature of use case whether target variable is involved or not.
![](https://anovos.github.io/anovos-docs/assets/html-reports/executive-report-2.png)
- **3 & 4** covers the overall view of the data in a nutshell across some of the key metrices : Outliers, Significant Attribute, Positive Skewness, Negative Skewness, High Variance, High Correlation, High Kurtosis, Low Kurtosis.
![](https://anovos.github.io/anovos-docs/assets/html-reports/executive-report-3.png)

### Wiki

![](https://anovos.github.io/anovos-docs/assets/html-reports/wiki-1.png)

The **"Wiki"** section has two different sections consisting of:

- **Data Dictionary**: This section contains the details of the attributes present in the data frame. The user, if, specifies the attribute wise definition at a specific path, then the details of the same will be populated along with the data type. Else, only the attribute wise datatype will be seen. This gives a schema - attribute, definition (description), data_type

![](https://anovos.github.io/anovos-docs/assets/html-reports/wiki-2.png)

- **Metric Dictionary**: Details about the different sections of the report, along with the definitions of the metrics used in them. This could be a quick reference for the user. 
![](https://anovos.github.io/anovos-docs/assets/html-reports/wiki-3.png)

### Descriptive Statistics

![](https://anovos.github.io/anovos-docs/assets/html-reports/descriptive-statistics-1.png)

The **Descriptive Statistics** gives specific information about the data elements and their individual metrics. Descriptive Statistics consists of the following modules:

- **Global Summary**: Details about the data dimensions and the attribute wise information. 
![](https://anovos.github.io/anovos-docs/assets/html-reports/descriptive-statistics-2.png)

- **Statistics By Metric Type** includes the following modules:
    - **Measures of Counts** : Details about the attribute wise count, fill rate, etc.   
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/descriptive-statistics-3.png)
    - **Measures of Central Tendency**: Details about the measurement of central tendency in terms of mean, median, mode.
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/descriptive-statistics-4.png)
    - **Measures of Cardinality**: Details about the uniqueness in categories for each attribute.
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/descriptive-statistics-5.png)
    - **Measures of Percentiles**: Indicates the different attribute value associated against the range of percentile cut offs. This helps to understand the spread of attributes. 
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/descriptive-statistics-6.png)
    - **Measures of Dispersion**:  Explains how much the data is dispersed through metrics like Standard Deviation, Variance, Covariance, IQR and range for each attribute.
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/descriptive-statistics-7.png)
    - **Measures of Shape**: Describe  distribution (or pattern) for different attributes through metrics like skewness and kurtosis.
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/descriptive-statistics-8.png)

- **Attribute Visualizations** includes the following modules:
    - **Numeric**: Visualizing the distributions of Numerical attributes using Histograms
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/attribute-visualization-1.png)

    - **Categorical**: Visualizing the distributions of Categorical attributes using Barplot
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/attribute-visualization-2.png)

### Quality Check 

![](https://anovos.github.io/anovos-docs/assets/html-reports/quality-check-1.png)

The **Quality Check** section consists of a qualitative inspection of the data at a row & columnar level. The Quality Check consists of the following modules:

- **Column Level**
    - **Null Columns Detections** – Detect the sparsity of the datasets, e.g., count and percentage of missing value of attributes
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/quality-check-2.png)

    - **Outlier Detection** – Used to detect and visualize the outlier present in numerical attributes of the datasets
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/quality-check-3.png)

    - **Violin Plot** - Displays the spread of numerical attributes
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/quality-check-4.png)

    - **IDness Detection** - IDness is calculated as Unique Values divided by non-null values seen in a column
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/quality-check-5.png)

    - **Biasedness Detection** 
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/quality-check-6.png)

    - **Invalid Entries Detection**
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/quality-check-7.png)

- **Row Level**
    - **Duplicate Detection** – Measures the number of rows in the datasets that have same value for each attribute
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/quality-check-8.png)
    - **NullRows Detection** - Measures the count/percentage of rows which have missing/null attributes
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/quality-check-9.png)

### Attribute Associations
![](https://anovos.github.io/anovos-docs/assets/html-reports/attribute-association-1.png)

Association analysis done for Attributes based on different statistical checks

- **Association Matrix & Plot** is a Correlation Measure of the strength of relationship among each attribute by finding correlation coefficient having range -1.0 to 1.0. Visualization is shown through heat map to describe the strength of relationship among attributes.
![](https://anovos.github.io/anovos-docs/assets/html-reports/attribute-association-2.png)

- **Information Value Computation** is used to rank variables on the basis of their importance. Greater the value of IV, higher the attribute importance. IV less than 0.02 is not useful for prediction. Bar plot is used to show the significance in descending order.
![](https://anovos.github.io/anovos-docs/assets/html-reports/attribute-association-3.png)

- **Information Gain Computation** measures the reduction in entropy by splitting a dataset according to a given value of an attribute. Bar plot is used to show the significance in descending order.
![](https://anovos.github.io/anovos-docs/assets/html-reports/attribute-association-4.png)

- **Variable Clustering** divides the numerical attributes into disjoint or hierarchical clusters based on linear relationship of attributes. This also reports the RS_Ratio of the attributes.  
![](https://anovos.github.io/anovos-docs/assets/html-reports/attribute-association-5.png)

- **Attribute to Target Association** determines how the target variable is associated with the rest of the attributes. It gives the event rate trend across different attribute categories<br>
    **Numeric**
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/attribute-association-6.png)
    **Categorical**
    ![](https://anovos.github.io/anovos-docs/assets/html-reports/attribute-association-7.png)

### Data Drift & Data Stability

- **Data Drift Analysis** - It gives the 4 measures of data drift namely, Population Stability Index (PSI), Jensen-Shannon Divergence (JSD), Hellinger Distance (HD) and Kolmogorov-Smirnov Distance (KS).

![](https://anovos.github.io/anovos-docs/assets/html-reports/data-drift-analytics-1.png)


![](https://anovos.github.io/anovos-docs/assets/html-reports/data-drift-analytics-2.png)

- **Overall Data Health**

![](https://anovos.github.io/anovos-docs/assets/html-reports/data-drift-analytics-3.png)

- **Data Stability Analysis**

![](https://anovos.github.io/anovos-docs/assets/html-reports/data-drift-analytics-4.png)

![](https://anovos.github.io/anovos-docs/assets/html-reports/data-drift-analytics-5.png)

### Time Series Analyzer

![](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/time-series-1.png)

This section summarizes the information about timestamp features and how they are interactive with other attributes. An exhaustive diagnosis is done by looking at different time series components, how they could be useful in deriving insights for further downstream applications.

- **The Basic Landscaping**

The initial analysis details we records where we understand whether a particular field qualifies for Time Series check or not. 

![](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/time-series-2.png)

- **Time Stamp Data Diagnosis**

The landscaping & diagnosis work done on the fields which have been auto-detected as time series. Different statistics are taken out pertaining to the association of devices for `id_date` & `date_id` pair combination as specified. Additionally, vital stats are also produced. 

![](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/time-series-3.png)
![](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/time-series-4.png)

- **Visualization across the Shortlisted Timestamp Attributes**

The visualization below shows the typical time series plots generated based on the analysis attributes and the granularity of data preferred for analysis (`daily`, `weekly`, `hourly`). 

The decomposed view largely describes about some of the typical components of time series forecasting like Trend, Seasonal & Residual on top of the Observed series. Inspecting on the decomposed view of Time Series is supposedly one of the key steps from analysis point irrespective of the model used later.

![](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/time-series-5.png)

The stationarity & transformation view help us in determinining how much the data can be quantified (through KPSS & ADSS test) in terms of transformation needed to attain stationarity. Additionally, we're showing on how a post transformation view basis `Box-Cox-Transformation` can be further used in the downstream applications.

![](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/time-series-6.png)

### Geospatial Analyzer

This section helps to analyze the geospatial related data features which are automatically identified. 

! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/geospatial_1.png)

- **Descriptive Analysis by Location Attributes**

This section gives the summary of the Lat-Long features and Geohash fields with respect to the following subsections: 
**Overall Summary** gives the count of the following stats -
For Lat-Long-Stats: Distinct {Lat,Long} pair, Distinct latitude, Distinct Longitude, Most Common {Lat,Long} Pair, Most Common {Lat,Long} pair occurence.
For Geohash-Stats: Total number of Distinct Geohashes, The Precision level observed for the Geohashes, The Most Common Geohash. 
**Top 100 Lat Long** gives the count of the Top 100 frequently seen lat-long/geohash pairs in the dataset. 

! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/descriptive_analysis_by_location_1.png)
! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/descriptive_analysis_by_location_2.png)


- **Clustering Geospatial Field**
 - **Cluster Identification** 
! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/cluster_identification_1.png)
     **Elbow Curve** showing the optimal number of clusters based on the lat-long and the geohash features. The elbow method uses the sum of squared distance (SSE) to choose an ideal value of k based on the distance between the data points and their assigned clusters.The algorithm used for this is KMeans
     ! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/cluster_identification_2.png)

     **Distribution of Silhouette scores across different parameters**: A silhouette provides a graphical representation of how well each feature has been matched to its own cluster. The silhouette ranges from -1 to +1, where a high value indicates that the feature is well matched to its own cluster and poorly matched to its neighbouring clusters. Underlying algorithm used: DBSCAN
     ! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/cluster_identification_3.png)

 - **Cluster Distribution** gives proportion of observations for the dataset contained in each cluster
 ! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/cluster_distribution_1.png)
 ! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/cluster_distribution_2.png)
 ! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/cluster_distribution_3.png)  


 - **Visualization** 
! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/visualization_1.png)
 This sub section helps in visualization of the latitude-longitude pairs on a geospatial world map for both the underlying algorithms - K-Means and DBSCAN.
! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/visualization_2.png)
! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/visualization_3.png)

 - **Outlier Points** 
 ! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/outlier_1.png)
 This sub section gives the graphical represenation of the outlier points captured by cluster analysis. Two graphs will be generated for DBSCAN algorithm : for Euclidean and Haversine distance.
 ! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/outlier_2.png)
 ! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/outlier_3.png)


- **Visualization by Geospatial Fields** Under this section, there are two subsections - Lat-Long-Plot and Geohash-Plot.
! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/visualization_geospatial_latlong_2.png)
! [](https://raw.githubusercontent.com/anovos/anovos-docs/main/docs/assets/html-reports/visualization_geospatial_geohash_2.png)