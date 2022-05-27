# Setting up Anovos on Azure Databricks

[Azure Databricks](https://azure.microsoft.com/services/databricks/)
is a hosted version of [Apache Spark](https://spark.apache.org/) on [Microsoft Azure](https://azure.microsoft.com/).
It is a convenient way to handle big data workloads of Spark without having to set up and maintain your own cluster.

To learn more about Azure Databricks, have a look at
[the official documentation](https://databricks.com/introducing-azure-databricks)
or the following introductory tutorials:
- [A beginner’s guide to Azure Databricks](https://www.sqlshack.com/a-beginners-guide-to-azure-databricks/)
- [Azure Databricks Hands-on](https://medium.com/@jcbaey/azure-databricks-hands-on-6ed8bed125c7)

Currently, _Anovos_ supports two ways of running workflows on Azure Databricks:

1. Processing datasets stored directly on DBFS
2. Processing datasets stored on [Azure Blob Storage](https://azure.microsoft.com/services/storage/blobs/)

Generally, we recommend the first option, as it requires slightly less configuration.
However, if you're already storing your datasets on Azure Blob Storage, mounting the respective containers
directly to DBFS allows you to directly process them with _Anovos_.

## _Anovos_ on Azure Databricks using DBFS

The following steps are required for running _Anovos_ workloads on Azure Databricks that process datasets stored
on DBFS.

### Step 1: Installing _Anovos_ on Azure Databricks
To make _Anovos_ available on Azure Databricks, you need to provide access to the _Anovos_ Python package.

The easiest way is to point Azure Databricks to the
[current release of _Anovos_ on the Python Package Index (PyPI)](https://pypi.org/project/anovos/).
(This is where `pip install anovos` goes to fetch _Anovos_ when installing from the terminal.)
This has the advantage that you will get a well-tested and stable version of _Anovos_.

Unless you need to make custom modifications to _Anovos_ or need access to new features or bugfixes that have
not been released yet, we recommend to go with this option.
In this case, you can directly go to Step 2.
(We will configure Azure Databricks to retrieve the correct _Anovos_ Python package as part of Step 4.)

#### Alternative: Manually uploading a wheel file

Instead of pointing Azure Databricks to the Python Package Index (PyPI),
you can make _Anovos_ available by downloading the respective wheel file
from PyPI yourself and manually uploading it to Azure Databricks.

You'll find the link to the latest wheel file on
[the "Download files" tab](https://pypi.org/project/anovos/#files).
If you'd like to use an older version, you can navigate to the respective version in the
[Release history](https://pypi.org/project/anovos/#history) and access the "Download files" tab
from there.

Download the _Anovos_ wheel file to your local machine and move on to Step 2.

#### Alternative: Use a development version of _Anovos_

If you would like to try the latest version of _Anovos_ on Azure Databricks
(or would like to make custom modifications to the library),
you can also create a wheel file yourself.

First, clone the _Anovos_ GitHub repository to your local machine:

```shell
git clone --depth 1 <https://github.com/anovos/anovos.git>
```

_**Note**: Using the `--branch` flag allows you to select a specific release of Anovos._
_For example, adding `--branch v0.2.2` will give you the state of the 0.2.2 release._
_If you omit the flag, you will get the latest development version of Anovos, which might not_
_be fully functional or exhibit unexpected behavior._

After cloning, go to the `anovos` directory that was automatically created in the process
and execute the following command to clean and prepare the environment:

```shell
make clean
```

It is a good practice to always run this command prior to generating a wheel file or another kind
of build artifact.

_**Note**: To be able to create a wheel file, `wheel`, `build`, and `setuptools` need to be installed_
_in the current Python environment. You can do so by running `pip install build wheel setuptools`._

Then, to create the wheel file, run the following command directly inside the `anovos` folder:

```shell
python -m build --wheel --outdir dist/ .
```

Once the process is finished, the folder `dist` will contain the wheel file.
It will have the file extension `*.whl` and might carry the latest version in its name.

_**Note:** The version in the file name will be that of the latest version of _Anovos_,_
_even if you cloned the repository yourself and used the latest state of the code._
_This is due to the fact that the version is only updated right before new release is published._
_To avoid confusion, it's a good practice to rename the wheel file to a custom name._

### Step 2: Prepare and copy the workflow configuration and data to DBFS

To run an _Anovos_ workflow, both the data to be processed and the workflow configuration
need to be stored on DBFS.

You can either use the UI or the CLI to copy files from your local machine to DBFS.
For detailed instructions, see the respective subsections below.

In this tutorial, we will use "income dataset" and an accompanying pre-defined workflow.

You can obtain these files by cloning the _Anovos_ GitHub repository:
```shell
git clone https://github.com/anovos/anovos.git
```

You'll find the dataset under `examples/data/income_dataset` and the configuration file
under `config/configs_income_azure.yaml`.

The `configs_income_azure.yaml` file contains the definition of the _Anovos_ workflow.
(To learn more about this file, see [📖 Configuring Workloads](../config_file.md).)

First, you should have a look at the configured input paths to make sure that _Anovos_ can find the data to be processed.
It is also important to check that the output paths are set to a location on DBFS that suits your needs.

For example, in the `input_dataset` block, you can see that by default the `file_path` is set to
`dbfs:/FileStore/tables/income_dataset/csv/`.
If you would like to store your data at a different location, you need to adapt this path accordingly.

Output paths are defined in several blocks.

The output path for the report data is specified as `master_path` in the blocks `report_preprocessing` and
`report_generation`.
The path for the report is specified as `final_report_path` in the `report_generation` block.
In this tutorial, by default, all these paths are set to `dbfs:/FileStore/tables/report_stats`.

The location where the processed data is stored is given by `file_path` in the blocks `write_main`,
`write_intermediate`, and `write_stats`.
In this tutorial, by default, these are set to sub-folders of `dbfs:/FileStore/tables/result`.

You can also make other changes to the workflow.
For example, you can define which columns from the input dataset are used in the workflow.
To try it yourself, find the `delete_column` configuration in the `input_dataset` block and add the column `workclass`
to the list of columns to be deleted:

```yaml
delete_column: ['logfnl','workclass']
```

To learn more about defining workflows through config files, see
[📖 Configuring Workloads](../config_file.md).

Once the `configs_income_azure.yaml` file is complete, you can copy this file and the dataset to DBFS.
For this, you can choose to either upload the files through the UI or use the Azure Databricks CLI.
We describe both options in the following sections.

In any case, make sure that you place the data files in the location defined in the configuration file.
You should also remember the location of the `configs_income_azure.yaml`, as you will need this information
in the subsequent steps.

(For this tutorial, we have decided to place all files under `dbfs:FileStore/tables/`.)

#### Copying files to DBFS using the UI

![https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image1.png](https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image1.png)

1. Launch the Azure Databricks workspace.
2. Enter the data menu.
3. Upload files by dragging files onto the marked area or click on it to upload using the file browser.

For more detailed instructions, see the
[Databricks documentation](https://docs.microsoft.com/en-us/azure/databricks/data/databricks-file-system#dbfs-and-local-driver-node-paths).

#### Copying files to DBFS using the CLI

1. Install `databricks-cli` into a local Python environment by running `pip install databricks-cli`.
2. Generate a personal access token for your Databricks workspace by going to _Settings_ > _User Settings_ >
   _Generate new token_. For details, see the [Databricks documentation](https://docs.microsoft.com/azure/databricks/dev-tools/api/latest/authentication).
3. Configure the CLI to access your workspace by running `databricks configure --token`.
4. Enter the URL of the databricks host (the domain of your workspace,
   usually of the pattern `https://<UNIQUE ID OF YOUR WORKSPACE>.azuredatabricks.net/`) and the token when prompted
   for it.
5. To verify the configuration, run `databricks fs ls` and check whether you are able to see the files stored on DBFS.
6. Then copy the files using the `dbfs cp` command:

For example:
```shell
dbfs cp configs_income_azure.yaml dbfs:/FileStore/tables/configs_income_azure.yaml
dbfs cp -r anovos/examples/data/income_dataset dbfs:/FileStore/tables/income_dataset
```

For more information on the Databricks CLI, see the
[Databricks documentation](https://docs.microsoft.com/en-us/azure/databricks/dev-tools/cli/).

### Step 3: Create a workflow script

To launch the workflow on Azure Databricks, we need a single Python script as the entry point.
Hence, we'll create a `main.py` script that invokes the _Anovos'_ workflow runner:
```python
import sys
from anovos import workflow

workflow.run(config_path=sys.argv[1], run_type="databricks")
```

Upload this script to DBFS as well using either of the methods described above.
Again, you can place this file at a location of your choosing.
In this tutorial, we have placed it at `dbfs:/FileStore/tables/scripts/main.py`.

### Step 4: Configure and launch an _Anovos_ workflow as a Databricks job

Once all files have been copied to DBFS, we can create an Azure Databricks job
that starts a cluster and launches the _Anovos_ workflow.

Here's an example of a job configuration:
![Job configuration](../../assets/azure_databricks_images/image2.png)

You can see that we set the "Type" to "Python" and have provided the path of the `main.py` script.
In the parameters section, we pass the DBFS path of the config file.

The cluster configuration comprises settings for the Databricks Runtime, the number of workers,
worker and driver types, as well as the cluster's scaling behavior.

Here's an example of a cluster configuration for this tutorials:
![Cluster configuration](../../assets/azure_databricks_images/image12.png)

For more detailed information, refer to the
[Databricks documentation](https://docs.microsoft.com/en-us/azure/databricks/clusters/configure#cluster-configurations).

To give the Databricks platform access to _Anovos_, click on "Advanced options" and select "Add dependent libraries".

If you chose the default way of obtaining _Anovos_ directly from the Python Package Index,
select "PyPI" as the "Library Source" and enter `anovos` as the "Package":
![Adding Anovos through PyPI](../../assets/azure_databricks_images/image8.png)

If you chose to provide your own wheel file, select "Upload" as the library source and follow the instructions.

In addition to the _Anovos_ wheel file, we need to provide the `histogrammar` package to Azure Databricks.
_Anovos_ uses this library internally to compute correlation matrices.

Following the same procedure as for _Anovos_, you can add `histogrammar` as a dependent library.
This time, we use "Maven" as the "Library Source".

Then, select `io.github.histogrammar:histogrammar-sparksql_2.12:1.0.20`
and `io.github.histogrammar:histogrammar_2.12:1.0.20` as the "Coordinates":
![Adding Histogrammar through Maven](../../assets/azure_databricks_images/image9.png)

(In case you're running _Anovos_ on Spark 2.4.x, you need to add
`io.github.histogrammar:histogrammar-sparksql_2.11:1.0.20` and `io.github.histogrammar:histogrammar_2.11:1.0.20`)

Once the job is configured, click "Create" to instantiate it.
Then, you'll see the full task configuration:
![Task Details](../../assets/azure_databricks_images/image10.png)

On the subsequent screen, click on "Run now" to launch the job:
![Active and completed runs](../../assets/azure_databricks_images/image5.png)

For more information on creating and maintaining jobs, see the
[Azure Databricks configuration](https://docs.microsoft.com/en-us/azure/databricks/jobs).

### Step 5: Retrieve the output

Once the job finishes successfully, it will show up under "Completed runs".

The intermediate data and the report data are saved at the `master_path` and the `final_report_path`
specified in the `configs_income_azure.yaml` file.
In this tutorial, we have set these paths to `dbfs:FileStore/tables/report_stats/`.

To retrieve the HTML report and the report data, you can either go to this path in the UI and copy the files,
or use the CLI to copy everything to your local machine:
```shell
dbfs cp -r dbfs:/FileStore/tables/report_stats/ ./
```

For more details regarding accessing files on DBFS, see the instructions on uploading files to DBFS in Step 2.

## Anovos on Azure Databricks using an Azure Blob Storage container mounted to DBFS

_We're currently working on this section._
_If you're interested in learning more about this option, [let us know!](../community/communication.md)_
