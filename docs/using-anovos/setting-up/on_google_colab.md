# Setting Up Anovos on Google Colab

[Colab](https://colab.research.google.com/) is an offer by Google Research 
that provides access to cloud-hosted [Jupyter notebooks](https://jupyter.org/) 
for collaborating on and sharing data science work.

Colab offers substantial compute resources even in its free tier and is integrated with
[Google Drive](https://drive.google.com/), 
making it an excellent place to explore libraries like _Anovos_ without setting up anything
on your local machine.

If you're not yet familiar with Google Colab, the following selection of introductory tutorials
are an excellent starting point to familiarize yourself with this platform:

- [Google Colab — The Beginner’s Guide](https://medium.com/lean-in-women-in-tech-india/google-colab-the-beginners-guide-5ad3b417dfa) (Lean In Women In Tech India)
- [How to use Google Colab](https://www.geeksforgeeks.org/how-to-use-google-colab/) (GeeksforGeeks)
- [Google Colab Tutorial for Data Scientists](https://www.datacamp.com/community/tutorials/tutorial-google-colab-for-data-scientists) (Datacamp)

## Step-by-step Instructions for Using _Anovos_ on Google Colab

The following four steps will guide you through the entire setup of _Anovos_
on Google Colab.

The instructions assume that you're starting out with a fresh, empty notebook environment.

### Step 1: Installing Spark dependencies

_Anovos_ builds on [Apache Spark](https://spark.apache.org/), which is not available by default in Google Colab.
Hence, before we can start working _Anovos_, we need to install Spark and set up a Spark environment.

Since Spark is a Java application, we start out by installing the Java Development Kit:

```shell
!apt-get install openjdk-8-jdk-headless -qq > /dev/null
```

Then, we can download Spark:

```shell
!wget https://archive.apache.org/dist/spark/spark-2.4.8/spark-2.4.8-bin-hadoop2.7.tgz
```

_**Note**: In this tutorial, we use Java 8 and Spark 2.4.8. You can use more recent versions as well._
_See the [list of currently supported versions](locally.md#software-prerequisites) to learn about available options._
   
Next, unzip the downloaded Spark archive to the current folder:

```shell
!tar xf spark-2.4.8-bin-hadoop2.7.tgz
```

Now we'll let the Colab notebook know where Java and Spark can be found by setting the corresponding
environment variables:

```python
import os
os.environ["JAVA_HOME"] = "/usr/lib/jvm/java-8-openjdk-amd64"
os.environ["SPARK_HOME"] = "/content/spark-2.4.8-bin-hadoop2.7"
```

To access Spark through Python, we need the `pyspark` library as well as the `findspark` utility:

```shell    
!pip install findspark pyspark==2.4.8
```
_**Note**: Make sure that the version of `pyspark` matches the Spark versions you downloaded._

## Step 2: Installing _Anovos_ and its dependencies

Clone the _Anovos_ GitHub repository to Google Colab:

```shell
!git clone --branch v0.2.2 https://github.com/anovos/anovos.git
```
_**Note**: Using the `--branch` flag allows you to select the desired release of Anovos._
_If you omit the flag, you will get the latest development version of Anovos, which might not_
_be fully functional or exhibit unexpected behavior._

After cloning, let's enter the newly created _Anovos_ directory:
```shell
cd anovos
```
As indicated by the output shown, _Anovos_ was placed in the folder `/content/anovos`,
which you can also access through the sidebar:

![You can view and access Anovos' files through the sidebar](../../assets/google_colab_notebook_images/image1_colab.png)

The next step is to build _Anovos_:
```shell
!make clean build
```
    
As the final step before we can start working with _Anovos_,
we need to install the required Python dependencies:

```shell
!pip install -r requirements.txt
```
    
## Step 3: Configuring an _Anovos_ workflow

_Anovos_ workflows are configured through a YAML configuration file.
To learn more, have a look at the exhaustive [Configuring Workflows](../config_file.md) documentation.

But don't worry: We'll guide you through the necessary steps!

First, open the file viewer in the sidebar and download the `configs.yaml` file from the `dist` folder
by right-clicking on the file and selecting _Download_:
![Download the `configs.yaml` file from the `dist` folder](../../assets/google_colab_notebook_images/image2_colab.png)

After downloading the `configs.yaml` file, you can now adapt the workflow it describes to your needs.

For example, you can define which columns from the input dataset are used in the workflow.
To try it yourself, find the `delete_column` configuration in the `input_dataset` block and add
the column `workclass` to the list of columns to be deleted:

```yaml
input_dataset:
...
  delete_column: ['logfnl','workclass']
...
```

You can learn more about this and all other configuration options in the [Configuring Workflows](../config_file.md)
documentation.
Each configuration block is associated with one of the various [_Anovos_ modules and functions](../../api/_index.md).

Once you adapted the `configs.yaml` file, you can upload it again by right-clicking on the `dist` folder
and selecting _Upload_:
![Upload the `configs.yaml` file to the `dist` folder](../../assets/google_colab_notebook_images/image3_colab.png)

## Step 4: Trigger a workflow run

Once the workflow configuration has been uploaded, you can run your workflow.
_Anovos_ workflows are triggered by executing the `spark-submit.sh` file that you'll find in the `dist` folder.
This script contains the configuration for the Spark executor.

To change the number of executors, the executor's memory, driver memory, and other parameters,
you can edit this file.

For example, in case of a very large dataset of several GB in size,
you might want to allocate more memory to the _Anovos_ workflow.
Let's go ahead and change the executor memory from the pre-defined `20g` to `32g`:
```bash
spark-submit \
...
--executor-memory 32g \
...
```
To make this or any other change, you need to download and upload the `spark-submit.sh` file similarly 
to the `configs.yaml` file as described in the previous section.

Once the adapted `spark-submit.sh` has been uploaded, we can trigger the _Anovos_ workflow run by
entering the `dist` directory and running `spark-submit.sh`:
```shell
cd dist
!nohup ./spark-submit.sh > run.txt &
```
The `nohup` command together with the `&` at the end of line ensures that the workflow is executed
in the background, allowing us to continue working in the Colab notebook.

To see what your workflow is doing, have a look at `run.txt`, where all logs are collected:
```shell
!tail -f run.txt
```
Once the run completes, the reports generated by _Anovos_ and all intermediate outputs are
stored at the specified path.

The intermediate data and the report data are saved at the `master_path` and the `final_report_path`
as specified by the user inside the `configs.yaml` file.
By default, these are set to `report_stats` and you should find all output files in this folder:
```shell
!cd report_stats
!ls -l
```
To view the HTML report, you'll have to download the `basic_report.html` file to your local machine,
using the same steps you took to download the `configs.yaml` and `spark-submit.sh` files.

## What's next?

In this tutorial, you've learned the basics of running _Anovos_ workflows on Google Colab.

- To learn all about the different modules and functions of _Anovos_, have a look at the 
  [API documentation](../../api/_index.md).
- The [Configuring Workflows](../config_file.md) documentation contains a complete list of all
  possible configuration options.
