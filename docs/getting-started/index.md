

# Getting Started with Anovos

*Anovos* provides data scientists and ML engineers with powerful and
versatile tools for feature engineering.

In this guide, you will learn how to set up *Anovos* and get to know what it can do.


## Setting up and verifying the Python and Spark environment

Anovos builds on [Apache Spark](https://spark.apache.org/), a highly scalable engine for data engineering, so an installation of Spark is required to run any Anovos code. The Python bindings for Spark (known as pyspark) also need to be installed in a compatible version.

If you are first starting out with Anovos and are not yet familiar with Spark, we recommend you execute this guide through the provided anovos-demo Docker container, which provides a full Spark setup along with compatible versions of Python and a Jupyter notebook environment.

[Setting up Anovos on
Local](https://docs.anovos.ai/docs/setting-up-your-environment/anovos-on-local/)

The currently version of Anovos is specifically built
for Spark 2.4.x, Python 3.7.x, and Java 8 (OpenJDK 1.8.x). You can
verify that you're running the correct versions by executing the
following lines:

<div class="cell code">

``` python
!python --version
```

</div>

<div class="cell code">

``` python
!spark-submit --version
```
</div>

Let's also check that a compatible version of `pyspark` is available
within our Python environment:


<div class="cell code">

``` python
import pyspark
pyspark.__version__
```

</div>

If you haven't done so already, let's install *Anovos* into the
currently active Python environment:

<div class="cell code">

``` python
!pip install anovos
```

</div>

Since Anovos relies on Spark behind the scenes for most heavy lifting, we need to pass an instantiated
[`SparkSession`](https://spark.apache.org/docs/2.4.8/api/python/pyspark.sql.html?highlight=sparksession#pyspark.sql.SparkSession)
to many of the function calls.

[Setting up Spark Session for
Anovos](https://docs.anovos.ai/docs/setting-up-your-environment/anovos-on-local/)

For the purposes of this guide, we'll use the pre-configured
`SparkSession` instance provided by
[`anovos.shared.spark`](https://github.com/anovos/anovos/blob/main/src/main/anovos/shared/spark.py):


<div class="cell code">

``` python
from anovos.shared.spark import spark
```
</div>

(Don't worry if this import takes some time and prints a lot of output.
You should see that settings are loaded, dependencies are added, and the
logger is configured.)

## Loading data

Data ingestion is the first step in any feature engineering project.
*Anovos* builds on [Spark's data loading
capabilites](https://spark.apache.org/docs/2.4.8/api/python/pyspark.sql.html)
and can handle different common file formats such as CSV, Parquet, and
Avro.

*Anovos*
[`data_ingest`](https://docs.anovos.ai/docs/anovos-modules-overview/data-ingest/)
module provides all data ingestion functionality. It includes functions
to merge multiple datasets as well as to select subsets of the loaded
data.

Let's load the classic [Adult Income
dataset](https://archive.ics.uci.edu/ml/datasets/adult) in CSV format,
which we'll use throughout this guide:

<div class="cell code">

``` python
from anovos.data_ingest.data_ingest import read_dataset

df = read_dataset(
    spark,  # Remember: The first argument of Anovos functions is always an instantiated SparkSession
    file_path='../data/income_dataset/csv',
    file_type='csv',
    file_configs={'header': 'True', 'delimiter': ',', 'inferSchema': 'True'}
)
```

</div>

Note that `df` is a standard Spark
[`DataFrame`](https://spark.apache.org/docs/2.4.8/api/python/pyspark.sql.html?highlight=sparksession#pyspark.sql.DataFrame):


<div class="cell code">

``` python
type(df)
```

</div>

Thus, you can use all the built-in methods you might be familiar with,
e.g.

<div class="cell code">

``` python
df.printSchema()
```

</div>


The [Adult Income
dataset's](https://archive.ics.uci.edu/ml/datasets/adult) more than 48k
entries each describes a person along with the information whether they
earn more than $50k per year.

In this guide, we will work with just a few of its columns:


<div class="cell code">

``` python
from anovos.data_ingest.data_ingest import select_column

df = select_column(df, list_of_cols=['age', 'education', 'education-num', 'occupation', 'hours-per-week', 'income'])
df = df.withColumn('income', (df['income'] == '>50K').cast('integer'))  # convert label to integer
df.printSchema()
```

</div>


## Learning about the data

Before we can start to engineer features, we need to understand the
data. For example, we need to verify that the data has
sufficient quality or if there are missing values.

*Anovos*'
[`data_analyzer`](https://docs.anovos.ai/docs/anovos-modules-overview/data-analyzer/)
module provides three submodules for this purpose:

  - The functions of
    [`data_analyzer.quality_checker`](https://docs.anovos.ai/docs/anovos-modules-overview/quality-checker/)
    allow us to detect and fix issues like empty rows or duplicate
    entries
  - [`data_analyzer.stats_generator`](https://docs.anovos.ai/docs/anovos-modules-overview/data-analyzer/)
    offers functions to caculate various statistical properties
  - [`data_analyzer.association_evaluator`](https://docs.anovos.ai/docs/anovos-modules-overview/association-evaluator/)
    enables us to examine a dataset for correlations between columns



### Assess the data quality

Once a new dataset is loaded, its quality should be assessed. Does the
dataset contain all the columns we expect? Do we have duplicate values?
Did we ingest the expected number of unique data points? *Anovos*'
[`data_analyzer.quality_checker`](https://docs.anovos.ai/docs/anovos-modules-overview/quality-checker/)
module provides convenient functions to answer these questions.

For time's sake, we'll assume that our dataset does not
exhibit any of these problems. Instead, we'll move on to more advanced quality assessments and check for outliers in the data. Outliers are data points that deviate significantly from the others. These points can be problematic when training ML models or during inference because there is very little information about the ranges in the dataset, leading to a high degree of uncertainty.

We immediately see that most people work a standard 40-hour week, while a significant minority reports anywhere between 20 and 60 hours. However, some individuals work very few hours, and others reported almost 100 hours per week, more than twice the median. These are the groups detected by the outlier detector.

We cannot only detect that there are outliers, but we can also deal with them right away. For example, let's remove the rows where individuals reported an excessive amount of hours worked per week:

How we deal with outliers depends on their origin and the application
context.

<div class="cell code">

``` python
from anovos.data_analyzer.quality_checker import outlier_detection

output_df, metric_df = outlier_detection(spark, df, detection_side='both', print_impact=True)
```

</div>

This tells us that we have little data for older people, which might be
an issue later when we are trying to predict the income of this group,
so we should keep this in mind.

We can ignore the value for `education-num`, a categorical column for
which the calculations performed by the
[`outlier_detection`](https://docs.anovos.ai/docs/anovos-modules-overview/quality-checker/#outlier_detection)
function are meaningless: In its standard configuration, it checks for
values that fulfill at least two of the following criteria: They belong
to the smallest or largest 5% of values, they deviate from the mean by
more than 3 standard deviations, or they lie below `Q1 - 1.5*IQR` or
above `Q3 + 1.5*IQR`, where `Q1` and `Q3` are the first and third
quartile, respectively, and `IQR` is the [Interquartile
Range](https://en.wikipedia.org/wiki/Interquartile_range). (For more
details and information on utilizing additional methods for outlier
detection, see [the
documentation](https://docs.anovos.ai/docs/anovos-modules-overview/quality-checker/#outlier_detection).

To better understand this, let's examine the hours worked per week
reported by the surveyed people:


<div class="cell code">

``` python
import plotly.express as px

px.histogram(df.toPandas(), x='hours-per-week')
```

</div>

We immediately see that the vast majority of people works a standard
40-hour week, while a significant minority reports anywhere between 20
and 60 hours. However, there are also individuals that work very few
hours as well as people that reported almost 100 hours per week, more
than twice the median. These are the groups detected by the outlier
detector.

We cannot only detect that there are outliers, we can also deal with
them right away. For example, let's remove the rows where individuals
reported an excessive amount of hours worked per week:


<div class="cell code">

``` python
df, metric_df = outlier_detection(spark, df, detection_side='both', list_of_cols=['hours-per-week'], treatment=True, treatment_method='row_removal')
```

</div>

Let's check that we have indeed reduced the dataset to entries where the
number of hours worked per week lies within a common range:

<div class="cell code">

``` python
px.histogram(df.toPandas(), x='hours-per-week')
```

</div>

### Understand how your data is distributed

When familiarizing ourselves with a dataset, one of the first steps is to understand the ranges of values each column contains and how the values are distributed. At the very least, we should learn the minimal and maximal values and examine the distribution within that range (e.g., by computing the mean, median, and standard deviation).

This information is vital to know. For many popular ML models, each feature column should be scaled to the same order of magnitude. Further, ML models will generally only work well for ranges of feature values trained on, so we should check that the data points they see during inference lie within the ranges we find.

*Anovos*'
[`data_analyzer.stats_generator`](https://docs.anovos.ai/docs/anovos-modules-overview/data-analyzer/)
module provides an easy way to quickly calculate a set of common
properties for the columns in a dataset:

<div class="cell code">

``` python
from anovos.data_analyzer.stats_generator import global_summary

global_summary(spark, df).toPandas()
```

</div>


In most real-world use cases, we are faced with incomplete datasets. For
example, customer records might be missing values because we have not
had a chance to ask the customer about them. In other cases, there might
have been a broken sensor, leading to missing values in a certain time
period.

In any case, we should know which columns in our dataset might exhibit
such issues in order to consider this during feature selection and
modeling. We can use the
[`data_analyzer.stats_generator`](https://docs.anovos.ai/docs/anovos-modules-overview/data-analyzer/)
module to check this:

<div class="cell code">

``` python
from anovos.data_analyzer.stats_generator import measures_of_counts

measures_of_counts(spark, df).toPandas()
```

</div>

There are various ways to deal with missing or unknown values. For
example, we could replace missing entries in our dataset with the mean
or the median of the respective column. If a column contains mostly null
values, it might also be an option to drop it entirely. If a dataset
contains a lot of unknown values across all of its columns, it will
likely be necessary to design a model that can explicitly handle this
situation.

Whatever is appropriate in a given scenario, *Anovos* offers convient
functions for this purpose in its
[`data_transformer`](https://docs.anovos.ai/docs/anovos-modules-overview/data_transformer/)
module, which we will have a look at later in this guide.


### Detect correlations within the dataset

In machine learning, we are often interested in predicting a class or
value (the *label*) from a set of *features*. To decide which features
to use in a specific scenario, it is often helpful to determine which
columns in a dataset are correlated with the label column. In other
words, we would like to find out which columns hold "predictive power".

*Anovos*'
[`data_analyzer.association_evaluator`](https://docs.anovos.ai/docs/anovos-modules-overview/association-evaluator/)
provides several functions for this purpose.

A commonly used tool is a correlation matrix. It visualizes the degree
of pairwise correlation between multiple columns at once.


<div class="cell code">

``` python
from anovos.data_analyzer.association_evaluator import correlation_matrix

correlation_matrix(spark, df, list_of_cols=['age', 'education-num', 'income']).toPandas()
```

</div>

From the matrix we see that age and education correlate with income: The
older or the higher educated a person, the higher the likelihood that
they earn above $50k. However, education and income exhibit a higher
degree of correlation than age and income.


### Examine Drift

Further, we have the drift detection. Drift is a problem for ML models.
If over time the distribution changes compared to that of the training,
validation, and test data, model performance might degrade. If you're
new to this topic, [this introductory blog
post](https://towardsdatascience.com/an-introduction-to-machine-learning-engineering-for-production-part-1-2247bbca8a61)
provides a first overview.

*Anovos* provides an entire module dedicated to detecting various kinds
of data drift. We recommend you check how the data you're planning to
use evolves over time prior to starting feature engineering.

As we only have one dataset, we will artificially create a dataset that
has drift by duplicating our dataset and shifting the age column,
artificially aging the population an entire decade:

<div class="cell code">

``` python
df_shifted = df.withColumn('age', df['age']+10)
```

</div>

With the
[`drift_detector.drift_statistics`](https://docs.anovos.ai/docs/anovos-modules-overview/data_drift_and_stability_index/#drift_statistics)
function we can compare a given dataset to a baseline. Let's try this:

<div class="cell code">

``` python
from anovos.data_drift.drift_detector import drift_statistics

drift_statistics(spark, df_shifted, df).toPandas()
```

</div>

We see that the drift detector has flagged the `age` column as
exhibiting drift. The calculated [Population Stability
Index](https://medium.com/model-monitoring-psi/population-stability-index-psi-ab133b0a5d42)
signals that the column's value distribution differs significantly from
that of the baseline.


## Transform the data

Up to this point, we have only analyzed the dataset and removed entire
entries based on these analyses. Now it's time to apply changes to the
dataset to make it more suitable for future ML model training.

Above, we discovered that there are missing values in all five feature
columns of the dataset. If we would like to later use an ML model that
cannot handle missing values, it might be a sensible option to replace
missing values with the feature column's average value.

*Anovos*'
[`data_transformer`](https://docs.anovos.ai/docs/anovos-modules-overview/data_transformer/)
module provides a handy utility function for this and similar
transformations:


<div class="cell code">

``` python
from anovos.data_transformer.transformers import imputation_MMM

transformed_df = imputation_MMM(spark, df, list_of_cols=['age', 'hours-per-week'], method_type='mean')
```

</div>


Let's check that we indeed replaced all missing values in the `age` and
`hours-per-week` columns:


<div class="cell code">

``` python
measures_of_counts(spark, transformed_df).toPandas()
```

</div>


Note that the data transformation capabilites of *Anovos* are currently
limited. Future versions of the library will offer capabilites like auto
encoders and methods for dimensionality reduction. For more information,
see the [Anovos Product
Roadmap](https://docs.anovos.ai/docs/anovos-roadmap/).


# Generate a report

Documenting datasets is an important component of any data governance
strategy. Thus, *Anovos* integrates [datapane](https://datapane.com/), a
library to create interactive reports.

A basic report can be generated with just one line of code:


<div class="cell code">

``` python
from anovos.data_report.basic_report_generation import anovos_basic_report

anovos_basic_report(spark, transformed_df, output_path='./report')
```

</div>

Once the report generation is finished, you can download and view the
generated `basic_report.html` stored in [./report](./report/) in any
browser.

Please note that due to Jupyter's security settings, it is currently not
possible to view it directly from within the Jupyter environment spun up
by the Docker container.

Of course, the format and content of the basic report will likely not be
sufficient for your organization's specific needs. Hence, *Anovos*
allows you to conveniently configure and create custom reports using the
functions in the
[`data_report.report_generation`](https://docs.anovos.ai/docs/data-reports/final-report-generation/)
submodule.


## Store the data

Once we've prepared and documented the data, it's time to store it so we
can use it to train and evaluate ML models.

Similar to data ingestion, data storage in *Anovos* is handled through
[Spark's versatile capabilities](). Using the
`data_ingest.write_dataset` function, we can write our processed
DataFrame to disk:

<div class="cell code">

``` python
from anovos.data_ingest.data_ingest import write_dataset

write_dataset(transformed_df, file_path="./export.csv", file_type="csv", file_configs={"header": "True", "delimiter": ","})
```

</div>

For now, this final step of data preparation (and this introductory
guide) is where *Anovos*' capabilities end. However, over the course of
the upcoming releases we will extend *Anovos* to include adapters for
popular AutoML solutions and Feature Stores, allowing you to seamlessly
move to model trainin and serving as well as data monitoring. For more
details and to see what else is ahead, see the [Anovos Product
Roadmap](https://docs.anovos.ai/docs/anovos-roadmap/).


## What's next?

In this guide, you've had a glimpse at the different capabilities
offered by *Anovos*. Of course, we've just scratched the surface and
there is much more to see and explore:

  - The [Anovos documentation](https://docs.anovos.ai/) is a great place
    to get an overview of the available functionality.
  - To see how different parts of *Anovos* can be used in practice, have
    a look at the [Jupyter
    notebooks](https://github.com/anovos/anovos/tree/main/examples/notebooks)
    our team has prepared for each of the modules.
  - Finally, to understand how *Anovos* can be integrated into your
    Spark ecosystem, see these
    [hints](https://docs.anovos.ai/docs/anovos-on-aws-emr/getting-started/)


<div class="cell code">

``` python
```

</div>
