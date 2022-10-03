# Setting up _Anovos_ locally

There are several ways to setting up and running _Anovos_ locally:

- **[🐋 Running _Anovos_ workloads through Docker](#running-anovos-workloads-through-docker)**
  You can run _Anovos_ workloads defined in a [configuration file](../config_file.md)
  using the `anovos-worker` Docker image, without having to set up Spark or anything else.
  This is the recommended option for developing and testing _Anovos_ workloads without access to a Spark cluster.
  If you're just starting with _Anovos_ and don't have any special requirements, pick this option.
- **[🐍 Using _Anovos_ as a Python library](#using-anovos-as-a-python-library)**
  There are two ways to install _Anovos_ to use it as a Python library in your own code:
    - **[Installation through `pip`.](#installation-through-pip)**
      If you need more fine-grained control than the [configuration file](../config_file.md) offers, or you
      have a way to execute Spark jobs, this is likely the best option for you.
    - **[Cloning the GitHub repository.](#cloning-the-github-repository)**
      This is recommended if you would like to get access to the latest development version.
      It is also the way to go if you would like to build custom wheel files.

## 🐋 Running _Anovos_ workloads through Docker

### 💿 Software Prerequisites

Running _Anovos_ workloads through Docker requires Python 3.x, a Bash-compatible shell, and Docker.
(See [the Docker documentation](https://www.docker.com/products/docker-desktop)
for instructions how to install Docker on your machine.)

At the moment, you need to download two scripts from the _Anovos_ GitHub repository.
You can either download the scripts individually:

```bash
mkdir local && cd local
wget https://raw.githubusercontent.com/anovos/anovos/main/local/rewrite_configuration.py
wget https://raw.githubusercontent.com/anovos/anovos/main/local/run_workload.sh
chmod +x run_workload.sh
```

Or you can clone the entire _Anovos_ GitHub repository, which will also give you access
to example configurations:

```bash
git clone https://github.com/anovos/anovos.git
cd anovos/local
chmod +x run_workload.sh
```

In both cases, you will have a folder named `local` that contains the `run_workload.sh` shell script
and the `rewrite_configuration.py` Python script.

### Launching an _Anovos_ run

To run an _Anovos_ workload defined in a [configuration file](../config_file.md),
you need to execute `run_workload.sh` and pass the name of the configuration file as the first parameter.

All paths to input data in the configuration file have to be given relative to the directory you are calling
`run_workload.sh` from.
For example, the configuration for the basic demo workload
(available [here](https://github.com/anovos/anovos/blob/main/config/configs_basic.yaml))
defines

```yaml
input_dataset:
  read_dataset:
    file_path: "data/income_dataset/csv"
```

Hence, we need to ensure that the `data` directory is a subdirectory of the directory we are launching the workload
from.
Otherwise, the _Anovos_ process inside the `anovos-worker` Docker container won't be able to access it.

The following command will run the
[basic demo workflow](https://github.com/anovos/anovos/blob/main/config/configs_basic.yaml)
included in the _Anovos_ repository on Spark 3.2.2:

```bash
# enter the root folder of the repository
cd anovos
# place the input data that is processed by the basic demo at the location
# expected by the configuration
mkdir data & cp ./examples/data/income_dataset ./data
# launch the anovos-worker container
./local/run_workload.sh config/configs_basic.yaml 3.2.2
```

Once processing has finished, you will find the output in a folder `output` within the directory you called
`run_workload.sh` from.

💡 _Note that the `anovos-worker` images provided through Docker Hub do not support_
   _the [Feast](../feature_store.md) or [MLflow](../mlflow.md) integrations out of the box,_
   _as they require interaction with third-party components, access to network communication,_
   _and/or interaction with the file system beyond the pre-configured paths._
   _You can find the list of currently unsupported configuration blocks at the top of_
   _[`rewrite_configuration.py`](https://github.com/anovos/anovos/blob/main/local/rewrite_configuration.py)._
   _If you try to run an _Anovos_ workload that uses unsupported features,_
   _you will receive an error message and no `anovos-worker` container will be launched._
   _If you would like, you can build and configure a custom pre-processing and launch script by adapting_
   _the files in [anovos/local](https://github.com/anovos/anovos/tree/main/local) to your specific needs._
   _For example, for Feast you will likely want to configure an additional volume or bind mount in_
   _`run_workload.sh`, whereas MLflow requires some network configuration._

### Specifying the Spark and _Anovos_ versions

You can optionally define the _Anovos_ version by adding it as a third parameter:

```bash
./local/run_workload.sh config.yaml 3.2.2 1.0.1
```

This will use _Anovos_ 1.0.1 on Spark 3.2.2.
If no version for Anovos is given, the latest release available for the specified Spark version will be used.

Please note that the corresponding `anovos-worker` image has to be available on
[Docker Hub](https://hub.docker.com/u/anovos) for this to work out of the box.

If you need a specific configuration not available as a pre-built image,
you can follow the instructions [here](https://github.com/anovos/anovos/tree/main/local) to build
your own `anovos-worker` image.
In that case, you can then launch `run_workload.sh` without specifying the Spark or _Anovos_ version:

```bash
./local/run_workload.sh config.yaml
```

## 🐍 Using _Anovos_ as a Python library

### 💿 Software Prerequisites

_Anovos_ requires Spark, Python, and Java to be set up.
We test for and officially support the following combinations:

- [Spark 2.4.8](https://spark.apache.org/docs/2.4.8/),
  [Python 3.7](https://www.python.org/downloads/release/python-3713/), and
  [Java 8](https://www.oracle.com/java/technologies/downloads/#java8)

- [Spark 3.1.3](https://spark.apache.org/docs/3.1.3/),
  [Python 3.9](https://www.python.org/downloads/release/python-3914/), and
  [Java 11](https://www.oracle.com/java/technologies/downloads/#java11)

- [Spark 3.2.2](https://spark.apache.org/docs/3.2.2/),
  [Python 3.10](https://www.python.org/downloads/release/python-3107/), and
  [Java 11](https://www.oracle.com/java/technologies/downloads/#java11)

- [Spark 3.3.0](https://spark.apache.org/docs/3.3.0/),
  [Python 3.10](https://www.python.org/downloads/release/python-3107/), and
  [Java 11](https://www.oracle.com/java/technologies/downloads/#java11)

The following tutorials can be helpful in setting up Apache Spark:

- [Installing Apache Spark on Mac OSX](https://kevinvecmanis.io/python/pyspark/install/2019/05/31/Installing-Apache-Spark.html)
- [Installing Apache Spark and using PySpark on Windows](https://towardsdatascience.com/installing-apache-pyspark-on-windows-10-f5f0c506bea1)

💡 _For the foreseeable future, _Anovos_ will support Spark 3.1.x, 3.3.x, and 3.3.x._
   _We will phase out 2.4.x over the course of the next releases._
   _To see which precise combinations we're currently testing, see_
   _[this workflow configuration](https://github.com/anovos/anovos/blob/main/.github/workflows/full-demo.yml#L21)._

### Installation through `pip`

To install _Anovos_, simply run

```shell
pip install anovos
```

You can select a specific version of _Anovos_ by specifying the version:

```shell
pip install anovos==1.0.1
```

For more information on specifying package versions, see
[the `pip` documentation](https://pip.pypa.io/en/stable/topics/repeatable-installs/#pinning-the-package-versions).

Then, you can import _Anovos_ as a module into your Python applications using

```python
import anovos
```

To trigger Spark workloads from Python, you have to ensure that the necessary external packages
are included in the
[`SparkSession`](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.SparkSession.html).

For this, you can either use the pre-configured `SparkSession` provided by _Anovos_:

```python
from anovos.shared.spark import spark
```

If you need to use your own custom `SparkSession`, make sure to include the following dependencies:

- [io.github.histogrammar:histogrammar_2.11:1.0.20](https://repo1.maven.org/maven2/io/github/histogrammar/histogrammar_2.11/1.0.20/)
- [io.github.histogrammar:histogrammar-sparksql_2.11:1.0.20](https://repo1.maven.org/maven2/io/github/histogrammar/histogrammar-sparksql_2.11/1.0.20/)
- [org.apache.spark:spark-avro_2.11:2.4.0](https://mvnrepository.com/artifact/org.apache.spark/spark-avro_2.11/2.4.0)

### Cloning the GitHub repository

Clone [the _Anovos_ repository](https://github.com/anovos/anovos) to your local environment using the command:

```shell
git clone https://github.com/anovos/anovos.git
```

For production use, you'll always want to clone a specific version, e.g.,

```shell
git clone -b v1.0.1 --depth 1 https://github.com/anovos/anovos
```

to just get the code for version `1.0.1`.

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
