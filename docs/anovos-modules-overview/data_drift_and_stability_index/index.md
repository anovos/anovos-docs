Module **ANOVOS.drift_detector**

### drift_statistics
When the performance of a deployed machine learning model degrades in production, one potential reason is that the data used in training and prediction are not following the same distribution.

Data drift mainly includes the following manifestations:

- Covariate shift: training and test data follow different distributions. For example, An algorithm predicting income that is trained on younger population but tested on older population.
- Prior probability shift: change of prior probability. For example in a spam classification problem, the proportion of spam emails changes from 0.2 in training data to 0.6 in testing data.
- Concept shift: the distribution of the target variable changes given fixed input values. For example in the same spam classification problem, emails tagged as spam in training data are more likely to be tagged as non-spam in testing data.

In our module, we mainly focus on covariate shift detection.

In summary, given 2 datasets, source and target datasets, we would like to quantify the drift of some numerical attributes from source to target datasets.
The whole process can be broken down into 2 steps: (1) convert each attribute of interest in source and target datasets into source and target probability distributions. (2) calculate the statistical distance between source and target distributions for each attribute.

In the first step, attribute_binning is firstly performed to bin the numerical attributes of the source dataset, which requires two input variables: bin_method and bin_size. The same binning method is applied on the target dataset to align two results. The probability distributions are computed by dividing the frequency of each bin by the total frequency.

In the second step, 4 choices of statistical metrics are provided to measure the data drift of an attribute from source to target distribution: Population Stability Index (PSI), Jensen-Shannon Divergence (JSD), Hellinger Distance (HD) and Kolmogorov-Smirnov Distance (KS).

They are calculated as below:
For two discrete probability distributions *P=(p_1,…,p_k)* and *Q=(q_1,…,q_k),*

![https://anovos.github.io/anovos-docs/assets/drift_stats_formulae.png](https://anovos.github.io/anovos-docs/assets/drift_stats_formulae.png)

A threshold can be set to flag out drifted attributes. If multiple statistical metrics have been calculated, an attribute will be marked as drifted if any of its statistical metric is larger than the threshold.

This function can be used in many scenarios. For example:

1. Attribute level data drift can be analysed together with the attribute importance of a machine learning model. The more important an attribute is, the more attention it needs to be given if drift presents.
2. To analyse data drift over time, one can treat one dataset as the source / baseline dataset and multiple datasets as the target datasets. Drift analysis can be performed between the source dataset and each of the target dataset to quantify the drift over time.

---------

- *idf_target*: Input target Dataframe
- *idf_source*: Input source Dataframe
- *list_of_cols*: List of columns to check drift (list or string of col names separated by |). Use ‘all’ - to include all non-array columns (excluding drop_cols).
- *drop_cols*: List of columns to be dropped (list or string of col names separated by |)  
- method: PSI, JSD, HD, KS (list or string of methods separated by |). Use ‘all’ - to calculate all metrics.
- *bin_method*: equal_frequency or equal_range
- *bin_size*: 10 - 20 (recommended for PSI), >100 (other method types)
- *threshold*: To flag attributes meeting drift threshold
- *pre_existing_source*: True if binning model & frequency counts/attribute exists already, False Otherwise. 
- *source_path*: If pre_existing_source is True, this argument is path for the source dataset details - drift_statistics folder. drift_statistics folder must contain attribute_binning & frequency_counts folders. If pre_existing_source is False, this argument can be used for saving the details. Default "NA" for temporarily saving source dataset attribute_binning folder

### stabilityIndex_computation

The data stability is represented by a single metric to summarise the stability of an attribute over multiple time periods. For example, given 6 datasets collected in 6 consecutive time periods (D1, D2, …, D6), data stability index of an attribute measures how stable the attribute is from D1 to D6.

The major difference between data drift and data stability is that data drift analysis is only based on 2 datasets: source and target. However data stability analysis can be performed on multiple datasets. In addition, the source dataset is not required indicating that the stability index can be directly computed among multiple target datasets by comparing the statistical properties among them.

In summary, given N datasets representing different time periods, we would like to measure the stability of some numerical attributes from the first to the N-th dataset.

The whole process can be broken down into 2 steps: (1) Choose a few statistical metrics to describe the distribution of each attribute at each time period. (2) Compute attribute level stability by combining the stability of each statistical metric over time periods.

In the first step, we choose mean, standard deviation and kurtosis as the statistical metrics in our implementation. Intuitively, they represent different aspects of a distribution: mean measures central tendency, standard deviation measures dispersion and kurtosis measures shape of a distribution. Reasons of selecting those 3 metrics will be explained in a later section. With mean, standard deviation and kurtosis computed for each attribute at each time interval, we can form 3 arrays of size N for each attribute.

In the second step, Coefficient of Variation (CV) is used to measure the stability of each metric. CV represents the ratio of the standard deviation to the mean, which is a unitless statistic to compare the relative variation from one array to another. Considering the wide potential range of CV, the absolute value of CV is then mapped to an integer between 0 and 4 according to the table below, where 0 indicates highly unstable and 4 indicates highly stable. We call this integer a metric stability index.

| abs(CV) Interval | Metric Stability Index |
| --- | --- |
| [0, 0.03) | 4 |
| [0.03, 0.1) | 3 |
| [0.1, 0.2) | 2 |
| [0.2, 0.5) | 1 |
| [0.5, +inf) | 0 |


Finally, the attribute stability index (SI) is a weighted sum of 3 metric stability indexes, where we assign 50% for mean, 30% for standard deviation and 20% for kurtosis. The final output is a float between 0 and 4 and an attribute can be classified as one of the following categories: very unstable (0≤SI<1), unstable (1≤SI<2), marginally stable (2≤SI<3), stable (3≤SI<3.5) and very stable (3.5≤SI≤4).

For example, there are 6 samples of attribute X from T1 to T6. For each sample, we have computed the statistical metrics of X from T1 to T6: 

| idx | Mean | Standard deviation | Kurtosis |
| --- | --- | --- | --- |
| 1 | 11 | 2 | 3.9 |
| 2 | 12 | 1 | 4.2 |
| 3 | 15 | 3 | 4.0 |
| 4 | 10 | 2 | 4.1 |
| 5 | 11 | 1 | 4.2 |
| 6 | 13 | 0.5 | 4.0 |

Then we calculate the Coefficient of Variation for each array:

- CV of mean = CV([11, 12, 15, 10, 11, 13]) = 0.136
- CV of standard deviation = CV([2, 1, 3, 2, 1, 0.5]) = 0.529
- CV of kurtosis = CV([3.9, 4.2, 4.0, 4.1, 4.2, 4.0]) = 0.027

Metric stability indexes are then computed by mapping each CV value to an integer accordingly. As a result, metric stability index is 2 for mean, 0 for standard deviation and 4 for kurtosis.

Why mean is chosen over median?

- Dummy variables which take only the value 0 or 1 are frequently seen in machine learning features. Mean of a dummy variable represents the proportion of value 1 and median of a dummy variable is either 0 or 1 whichever is more frequent. However, CV may not work well when 0 appears in the array or the array contains both positive and negative values. For example, intuitively [0,0,0,0,0,1,0,0,0] is a stable array but its CV is 2.83 which is extremely high, but cv of [0.45,0.44,0.48,0.49,0.42,0.52,0.49,0.47,0.48] is 0.06 which is much more reasonable. Thus we decided to use mean instead of median. Although median is considered as a more robust choice, outlier treatment can be applied prior to data stability analysis to handle this issue.

Why kurtosis is chosen over skewness?

- Kurtosis is a positive value (note that we use kurtosis instead of excess kurtosis which) but skewness can range from –inf to +inf. Usually, if skewness is between -0.5 and 0.5, the distribution is approximately symmetric. Thus, if the skewness fluctuates around 0, the CV is highly likely to be high or invalid because the mean will be close to 0.

Stability index is preferred in the following scenario:

- Pairwise drift analysis can be performed between the source dataset and each of the target dataset to quantify the drift over time. However this can be time-consuming especially when the number of target dataset is large. In this case, measuring data stability instead of data drift would be a much faster alternative and the source/baseline dataset is not required as well

Troubleshooting

- If the attribute stability index appears to be nan, it may due to one of the following reasons:
    - One metric (likely to be kurtosis) is nan. For example, the kurtosis of a sample is nan If its standard deviation is 0.
	- The mean of a metric from the first to the N-th dataset is zero, causing the denominator of CV to be 0. For example, when mean of attribute X is always zero for all datasets, its stability index would be nan.

Limitation

- Limitation of CV: CV may not work well when 0 appears in the array or the array contains both positive and negative values.

------

- *idfs*: Input Dataframes (flexible)
- *list_of_cols*: Numerical columns (in list format or string separated by |). Use ‘all’ - to include all numerical columns (excluding drop_cols).
- *drop_cols*: List of columns to be dropped (list or string of col names separated by |)
- *metric_weightages*: A dictionary with key being the metric name (mean,stdev,kurtosis) and value being the weightage of the metric (between 0 and 1). Sum of all weightages must be 1.
- *existing_metric_path*: this argument is path for pre-existing metrics of historical datasets  <idx,attribute,mean,stdev,kurtosis>. idx is index number of historical datasets assigned in chronological order
- *appended_metric_path*: this argument is path for saving input dataframes metrics after appending to the historical datasets' metrics. 
- *threshold*: To flag unstable attributes meeting the threshold




