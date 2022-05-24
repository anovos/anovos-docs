# Setting up Anovos on Azure Databricks 

[Azure Databricks](https://azure.microsoft.com/services/databricks/)
is a hosted version of [Apache Spark](https://spark.apache.org/) on [Microsoft Azure](https://azure.microsoft.com/).
It is a convenient way to handle big data workloads of Spark without having to set up and maintain your own cluster.

To learn more about Azure Databricks, have a look at
[the official documentation](https://databricks.com/introducing-azure-databricks)
or the following introductoray tutorials:
- [A beginner’s guide to Azure Databricks](https://www.sqlshack.com/a-beginners-guide-to-azure-databricks/)
- [Azure Databricks Hands-on](https://medium.com/@jcbaey/azure-databricks-hands-on-6ed8bed125c7)

Currently, _Anovos_ supports two ways of running workflows on Azure Databricks: 

1.	Using DBFS directly
2.	Mounting an Azure blob storage container to DBFS

**TODO: What do we recommend in which case?**

## Anovos on Azure Databricks using DBFS

The following are required for running _Anovos_ workloads on Azure Databricks using DBFS.

### Step 1: Download Anovos wheel file from PyPI

To make _Anovos_ available on Azure Databricks, you need to upload a wheel file.
The easiest way to obtain it is to download directly from PyPI.
This ensures that you have the latest stable and well-tested version of _Anovos_.

You'll find the link to the latest wheel file on
[the "Download files" tab](https://pypi.org/project/anovos/#files).
If you'd like to use an older version, you can navigate to the respective version in the
[Release history](https://pypi.org/project/anovos/#history) and access the "Download files" tab
from there.

**TODO: Do users really need to download the wheel? In the screenshot below, there is an option
to use packages directly from PyPI**

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
__

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
python -m build --wheel --universal --outdir dist/ .
```

Once the process is finished, the folder `dist` will contain the wheel file.
It will have the file extension `*.whl` and might carry the latest version in its name.

_**Note:** The version in the file name will be that of the latest version of _Anovos_,_
_even if you cloned the repository yourself and used the latest state of the code._
_This is due to the fact that the version is only updated right before new release is published._
_To avoid confusion, it's a good practice to rename the wheel file to a custom name._

### Step 2: Copy the data and workflow configuration to DBFS

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

**TODO: What exactly do users need to update here to run the example?**
Update configs.yaml for all input & output DBFS paths. All other  
changes depend upon the dataset being used.

**TODO: In which folder(s) should the data and config file be placed?
The screenshot below shows the config file at FileStore/tables/**

To learn more about defining workflows through config files, see the
[config file documentation](../config_file.md).

#### Copying files to DBFS using the UI

![https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image1.png](https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image1.png)

1. Launch the workspace on Databricks.
2. Enter the data menu
3. Upload files by dragging files onto the marked area or click on it to upload using the file browser.

For more detailed instructions, see the
[Databricks documentation](https://docs.microsoft.com/en-us/azure/databricks/data/databricks-file-system#dbfs-and-local-driver-node-paths)

#### Copying files to DBFS using the CLI

1. Install `databricks-cli` into a local Python environment by running `pip install databricks-cli`.
2. Generate a personal access token for your Databricks workspace **TODO: Where/how do I create such a token?**
3. Configure the CLI to access your workspace by running `databricks configure --token`.
4. Copy the files using the `dbfs cp` command.

For example:
```shell
dbfs cp -r /home/user1/Desktop/dummy_folder dbfs:/Filestore/tables/dummy_folder
```

For more information on the Databricks CLI, see the
[Databricks documentation](https://docs.microsoft.com/en-us/azure/databricks/dev-tools/cli/).

### Step 3: Create a workflow script

To launch the workflow on Azure Databricks, we need a single Python script as the entry point.
Hence, we'll create a `main.py` script that invokes the _Anovos'_ workflow runner with the proper run type:
```python
import sys
from anovos import workflow

workflow.run(config_path=sys.argv[1], run_type="databricks")
```

Upload this script to DBFS as well.
**TODO: Which location should it be placed in? The screenshot below shows it at FileStore/tables/scripts/**

### Step 4: Configure and launch an _Anovos_ workflow as a Databricks job

Once all files have been copied to DBFS, we can create an Azure Databricks job
that starts a cluster and launches the _Anovos_ workflow.

Here's an example of a job configuration:

![Job configuration](../../assets/azure_databricks_images/image2.png)

The cluster configuration comprises settings for the Spark version, the number of workers and worker types,
as well as the scaling behavior.
For more detailed information, refer to the
[Databricks documentation](https://docs.microsoft.com/en-us/azure/databricks/clusters/configure#cluster-configurations).

For purposes of this tutorial, you can use the following example configuration:

![Cluster configuration](../../assets/azure_databricks_images/image3.png)

To give the Databricks platform access to _Anovos_, click on "Advanced options" and select "Add dependent libraries".
In the configuration dialogue, upload the _Anovos_ wheel.
**TODO: Why don't we use the "PyPI" option visible in the screenshot?**

![Add dependent library](../../assets/azure_databricks_images/image4.png)

**TODO: Which Jars need to be uploaded and why? Can't we just get them from Maven?**
ii.  Add jars by uploading from local machine (jars/.jar file)

> Library Source- Upload
> 
> Library Type- Jar
> 
> Note: The another way is that we can also give location of DBFS path directly by clicking on
> DBFS/ADLS (Library Source) if required wheel file and jars are already
> uploaded there from local machine.

Once the job is configured, click "Create" to instantiate it.
On the subsequent screen, click on "Run now" to launch the job:

![Active and completed runs](../../assets/azure_databricks_images/image5.png)

For more information on creating and maintaining jobs, see the
[Azure Databricks configuration](https://docs.microsoft.com/en-us/azure/databricks/jobs).

### Step 5: Retrieve the output

Once the job finishes successfully, it will show up under "Completed runs".

**TODO: Be more specific: Where do I go and what will I find? What should I do with that data?**
The intermediate data and the report data are saved at the master_path and the final_report_path in DBFS as specified by the user inside the configs.yaml file 

## Anovos on Azure Databricks using an Azure Blob Storage container mounted to DBFS

**TODO: Update to simplified workflow similar to the preceding section**

### Step 3: Copy the files to an Azure Blob Storage container 

Copy the following files from local machine to Azureblob storage container directly from UI or from CLI commands in order to run Anovos using Azure Blob Storage container:

- dist/income_dataset (optional)

  - This folder contains our demo dataset.This is sample dataset that is shown for reference.Users can copy their own dataset.

- dist/main.py

  - This is sample script to show how different functions from Anovos
    module can be stitched together to create a workflow.

  - This script takes input from a yaml configuration file

- dist/.whl file

  - This is wheel file that contains all the packages and required
    scripts that was taken or built in step 2

- jars/.jar file

  - This is jar file used for installing histogram packages for running
    anovos

> For running in spark 2.4.x, copy following jar file to azure blob
> storage container from local machine: a)
> [histogrammar-sparksql\_2.11-1.0.20.jar](https://github.com/anovos/anovos/blob/main/jars/histogrammar-sparksql_2.11-1.0.20.jar)
> b)
> [histogrammar\_2.11-1.0.20.jar](https://github.com/anovos/anovos/blob/main/jars/histogrammar_2.11-1.0.20.jar)
> 
> For running in spark 3.x.x, copy following jar file to azure blob
> storage container from local machine : a)
> [histogrammar-sparksql\_2.12-1.0.20.jar](https://github.com/anovos/anovos/blob/main/jars/histogrammar-sparksql_2.12-1.0.20.jar)
> b)
> [histogrammar\_2.12-1.0.20.jar](https://github.com/anovos/anovos/blob/main/jars/histogrammar_2.12-1.0.20.jar)

Note: we can copy directly from UI by clicking upload button on azure
blob storage container or using command line.

The syntax to upload a file using command line are as follows:

azcopy copy " **SourceFile**" "**storage_account_name**.**blob**.core.windows.net/**containername**?**SAStoken**"
    
Note: Attaching link that details about transfering data with AzCopy command line utility and file storage for reference.
[Transfer data with AzCopy and file storage](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-files)

### Step 4: Mount a container of Azure Blob Storage as a dbfs path in Azure Databricks

For accessing files from azure blob storage container for running anovos in Azure databricks platform, we need to mount that container in dbfs path.

Mounting Azure blob storage container by executing the following commands in Azure databricks notebook by starting the cluster.

    dbutils.fs.mount(
        source = "wasbs://<container-name>@<storage-account-name>.blob.core.windows.net",
        mount_point = "/mnt/<mount-name>",
        extra_configs = {"fs.azure.account.key.<storage-account-name>.blob.core.windows.net":"<storage-account-key>"})

here, 
- **storage-account-name:** is the name of your Azure Blob storage account.
- **container-name:** is the name of a container in your Azure Blob storage account.
- **mount-name:** is a DBFS path representing where the Blob storage container or a folder inside the container will be mounted in DBFS.
- **storage-account-key:** is the access key for that storage account

Attaching some links to get more information about mounting azure blob storage container in dbfs path.
- [Azure Blob storage](https://docs.microsoft.com/en-us/azure/databricks/data/data-sources/azure/azure-storage#:~:text=Mount%20Azure%20Blob%20storage%20containers%20to%20DBFS,-You%20can%20mount&text=All%20users%20have%20read%20and,immediately%20access%20the%20mount%20point)

**Note:** 
Mounting needs to be done only one time when we are using the same mount_name for mounting in dbfs. No need to mount when we are running again using same mount_name as it is already mounted.
To unmount a mount point, use the following command in Azure databricks notebook:

    dbutils.fs.unmount("/mnt/<mount-name>")
    
### Step 5: Update config file for all input and output path according to dbfs mount path

Once mounting is completed, the data is present in the required dbfs path where we have given in mount_point. All the operations happened during running anovos by using this mount dbfs path and that automatically get updated in azure blob storage container too.

Config.yaml file that is available in local machine needs to be updated accordingly using path which we have given in mount_point.
Input and Output Path should be updated everywhere in config file that starts like this 

    For Pyspark operations - "dbfs:/mnt/mount-name/folder_name/"
    For Python operations – "/dbfs/mnt/mount-name/folder_name/"

**Example:**
```yaml
  read_dataset:
    file_path: "dbfs:/mnt/anovos1/income_dataset/csv/"
    file_type: csv
```

here mount-name refers to anovos1 and income_dataset is the folder name that is present in azure blob storage container.

**Note** Attaching config file description link to get more information about updating input,output path and threshold settings according to use case.
    [config_file_description](../config_file.md)

### Step 6: Copy updated config file from local machine to Azure Blob Storage container

using UI or from azure command in similar way like in step 3 for other files

    - config/configs_income_azure_blob_mount.yaml
    
This is the sample yaml configuration file which sets the argument for all functions for running anovos in Azure Databricks

### Step 7: Creating jobs for running anovos by initiating cluster on Azure Databricks
Once you have copied all files in DBFS, then you can create jobs for running anovos by starting cluster and provides all the task details like task name, type, cluster configurations, Parameters and Dependent Libraries. Below shows one sample example for creating jobs for reference.

**•Task Details**
![Task details](../../assets/azure_databricks_images/image7.png)

**a.Task Name** – Give any task name relevant to your project 

**b.Type** – Python, DBFS mount path of main.py script file 

**c.Cluster**

**Note** Attaching link that describes all the information related to cluster configurations.
[Configure clusters](https://docs.microsoft.com/en-us/azure/databricks/clusters/configure#cluster-configurations)
                                                **Cluster Configurations**
                                                
![Configure new cluster](../../assets/azure_databricks_images/image3.png)                                               
- **Cluster mode** – Standard
- **Databricks run time version** – Select the spark and scala version for creating cluster
    For running in python 3.7.x – scala 2.11, spark 2.x.x
    For running in python 3.8.x – scala 2.12, spark 3.x.x
- **Autopilot Options** – Enable autoscaling should be on 
- **Worker Types** – General purpose (14GB Memory, 4 cores), min worker – 2, max worker-8
- **Driver Types** - General purpose (14GB Memory, 4 cores)

**Notes** – Users can change this worker types and driver types configurations according to jobs complexity

**d.Parameters** – [ mounted DBFS path to config.yaml ,  run_type]
      Eg. - ["/dbfs/mnt/anovos1/configs_income_mount.yaml","databricks"]
      
**e.Dependent libraries-**
Provide mounted DBFS path of wheel file and jar file by clicking on DBFS/ADLS if required wheel file and jars are already         uploaded in the azure blob storage container that is required for running anovos

**Example:**

      Wheel file path – dbfs:/mnt/anovos1/jars/anovos-0.1.1-py2.py3-none-any.whl
      Jar file path -  dbfs:/mnt/anovos1/jars/histogrammar_2.12-1.0.20.jar
                            dbfs:/mnt/anovos1/jars/histogrammar-sparksql_2.12-1.0.20.jar

After setting all these required steps in task, click create and jobs will be created successfully and then users will be able to see like this in tasks.

![](../../assets/azure_databricks_images/image6.png)

For running these jobs, click on run now and then jobs will be triggered automatically.
Option for scheduling these jobs for running automatically are also available.

**Note:** Attaching links that will brief about creating jobs, running as well as scheduling jobs for reference.
- [Jobs](https://docs.microsoft.com/en-us/azure/databricks/jobs)
- [Jobs](https://docs.databricks.com/jobs.html)

Once the job finishes successfully, users will be able to see status as succeeded.we can see that in the below images.

![Active and completed runs](../../assets/azure_databricks_images/image5.png)

The intermediate data and the report data are saved at the master_path and the final_report_path as specified by the user inside the configs.yaml file and these outputs will be stored in Azure blob storage container automatically once jobs finishes successfully.
