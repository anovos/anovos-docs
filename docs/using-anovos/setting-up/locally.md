# Setting up _Anovos_ locally

## 💿 Software Prerequisites

The current Beta release of _Anovos_ requires Spark, Python, and Java to be set up.
We test for and officially support the following combinations:

- [Spark 2.4.8](https://spark.apache.org/docs/2.4.8/),
  [Python 3.7](https://www.python.org/downloads/release/python-378/), and
  [Java 8](https://www.oracle.com/java/technologies/downloads/#java8)

- [Spark 3.1.3](https://spark.apache.org/docs/3.1.3/),
  [Python 3.9](https://www.python.org/downloads/release/python-3910/), and
  [Java 11](https://www.oracle.com/java/technologies/downloads/#java11)

- [Spark 3.2.1](https://spark.apache.org/docs/3.2.1/),
  [Python 3.9](https://www.python.org/downloads/release/python-3910/), and
  [Java 11](https://www.oracle.com/java/technologies/downloads/#java11)

The following tutorials can be helpful in setting up Apache Spark:

- [Installing Apache Spark on Mac OSX](https://kevinvecmanis.io/python/pyspark/install/2019/05/31/Installing-Apache-Spark.html)
- [Installing Apache Spark and using PySpark on Windows](https://towardsdatascience.com/installing-apache-pyspark-on-windows-10-f5f0c506bea1)

💡 _For the foreseeable future, _Anovos_ will support Spark 2.4.x, 3.1.x, and 3.2.x._
   _To see which precise combinations we're currently testing, see_
   _[this workflow configuration](https://github.com/anovos/anovos/blob/main/.github/workflows/full-demo.yml#L21)._

## Installing Anovos

_Anovos_ can be installed and used in one of two ways:

- Cloning [the GitHub repository](https://github.com/anovos/anovos) and running via `spark-submit`.
- Installing through `pip` and importing it into your own Python scripts.

### Clone the GitHub repository to use Anovos with `spark-submit`

Clone [the _Anovos_ repository](https://github.com/anovos/anovos) to your local environment using the command:
```shell
git clone https://github.com/anovos/anovos.git
```

For production use, you'll always want to clone a specific version, e.g.,
```shell
git clone -b v0.1.0 --depth 1 https://github.com/anovos/anovos
```
to just get the code for version `0.1.0`.

Afterwards, go to the newly created `anovos` directory and execute the following command to clean and build
the latest modules:
```shell
make clean build
```

Next, install _Anovos_' dependencies by running
```shell
pip install -r requirements.txt
```

and go to the `dist/` folder. There, you should

- Update the input and output paths in `configs.yaml` and configure the data set.
  You might also want to adapt the threshold settings to your needs.

- Adapt the `main.py` sample script. It demonstrates how different functions from _Anovos_ can be stitched
  together to create a workflow.

- If necessary, update `spark-submit.sh`. This is the shell script used to run the Spark application via `spark-submit`.

Once everything is configured, you can start your workflow run using the aforementioned script:
```shell
nohup ./spark-submit.sh > run.txt &
```

While the job is running, you can check the logs written to `stdout` using
```shell
tail -f run.txt
```

Once the run completes, the script will attempt to automatically open the final report
(`report_stats/ml_anovos_report.html`) in your web browser.

### 🐍 Install through `pip` to use Anovos within your Python applications

To install _Anovos_, simply run
```shell
pip install anovos
```

Then, you can import _Anovos_ as a module into your Python applications using
```python
import anovos
```

To trigger Spark workloads from Python, you have to ensure that the necessary external packages
are included in the [`SparkSession`](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.SparkSession.html).

For this, you can either use the pre-configured `SparkSession` provided by _Anovos_:

```python
from anovos.shared.spark import spark
```

If you need to use your own custom `SparkSession`, make sure to include the following dependencies:

- [io.github.histogrammar:histogrammar_2.11:1.0.20](https://repo1.maven.org/maven2/io/github/histogrammar/histogrammar_2.11/1.0.20/)
- [io.github.histogrammar:histogrammar-sparksql_2.11:1.0.20](https://repo1.maven.org/maven2/io/github/histogrammar/histogrammar-sparksql_2.11/1.0.20/)
- [org.apache.spark:spark-avro_2.11:2.4.0](https://mvnrepository.com/artifact/org.apache.spark/spark-avro_2.11/2.4.0)
