# Anovos on Azure Databricks Platform 

Azure Databricks is an implementation of Apache Spark on Microsoft Azure. It is a powerful chamber that handles big data workloads effortlessly and helps in both data wrangling and exploration. It lets you run large-scale Spark jobs from any Python, R, SQL, and Scala applications.

Following links and video will brief about introduction about Azure Databricks and the usage of this platform and its benefits.
- [A beginner’s guide to Azure Databricks](https://www.sqlshack.com/a-beginners-guide-to-azure-databricks/)
- [Azure Databricks Hands-on](https://medium.com/@jcbaey/azure-databricks-hands-on-6ed8bed125c7)
- [Introducing Azure Databricks](https://databricks.com/introducing-azure-databricks)

Currently we are supporting two ways of running Anovos on Azure Databricks Platform 

1.	Anovos on Azure Databricks using DBFS
2.	Anovos on Azure Databricks using Azure blob storage container by mounting to DBFS

## 1. Anovos on Azure Databricks using DBFS

**Following are the steps required for running anovos on Azure Databricks using DBFS:**

**Step1: Installing/Downloading Anovos**

Clone the Anovos Repository on local machine using command:

    git clone <https://github.com/anovos/anovos.git>

After cloning, go to anovos directory and execute the following command
to clean and build the latest modules in dist folder:

    make clean build

**Step2: Creating wheel file in local machine**

Note: Before creating this wheel file, you will need to install python
packages using command:

    pip install wheel setuptools

Then create wheel file using setup.py file that contains all python
scripts and dependencies using command:

    python setup.py bdist_wheel --universal

After the command finishes executing successfully, a folder name ‘dist’
contains the .whl file which is your wheel file. Wheel file contains all
the packages and scripts which are given in setup.py for running anovos
in azure databricks.

**Step3: Copy necessary files to Databricks File System (DBFS) in Azure
Databricks**
Copy the following files from local machine to DBFS directly from UI or from CLI commands:

**Note** Steps to copy from UI or from CLI commands are mentioned below in detail.

  i. dist/income_dataset (optional)

<!-- end list -->

  - This folder contains our demo dataset. This is sample dataset that is shown for reference.Users can copy their own dataset.

<!-- end list -->

  ii. dist/main.py

<!-- end list -->

  - This is sample script to show how different functions from Anovos
    module can be stitched together to create a workflow.

  - The users can create their own workflow script by importing the
    necessary functions.

  - This script takes input from a yaml configuration file

<!-- end list -->

  iii. config/configs_income_azure.yaml

<!-- end list -->

  - This is the sample yaml configuration file for running anovos in
    Azure Databricks which sets the argument for all functions.

  - Update configs.yaml for all input & output DBFS paths. All other
    changes depend upon the dataset being used.
    
    **Note**Attaching config file description link to get more information about updating input,output path and threshold settings according to use case.
    [config_file_description](https://github.com/anovos/anovos-docs/blob/anovos_config_file_desc/docs/using-anovos/config_file.md)

**Steps to copy files to DBFS using UI**

![https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image1.png](https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image1.png)

1.  Launch any workspace and go to data menu on databricks. After
    clicking data menu, options to upload file will appear and then by
    clicking ‘drop files to upload, or click to browse’, users will be
    able to upload the above required files from local machine.
    
    **Note** Attaching link that details about DBFS and Files upload interface
    [Files upload interface in DBFS](https://docs.microsoft.com/en-us/azure/databricks/data/databricks-file-system#dbfs-and-local-driver-node-paths)


**Databricks CLI configuration steps**

1.  Install *databricks-cli* using command – pip install databricks-cli

2.  Make sure the personal access tokens have already generated

3.  Copy the URL of databricks host and the personal access tokens

4.  Configure the CLI using command – databricks configure --token

**DBFS CLI Command for copy in Azure Databricks:**

dbfs cp -r source_folder_path destination_folder_path

eg. dbfs cp -r /home/user1/Desktop/dummy_folder
dbfs:/Filestore/tables/dummy_folder

**Step4: Creating jobs for running anovos by initiating cluster**
  - **Task Details**

  ![https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image2.png](https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image2.png)

<!-- end list -->

1.  **Task Name –** Give any task name relevant to your project

2.  **Type –** Python, location of main.py script file in DBFS

3.  **Cluster: -**
**Note**Attaching link that describes all the information related to cluster configurations.
[Configure clusters](https://docs.microsoft.com/en-us/azure/databricks/clusters/configure#cluster-configurations)
  **Cluster Configurations -**

  ![https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image3.png](https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image3.png)

i.  **Cluster mode –** Standard

ii.  **Databricks run time version –** Select the spark and scala version
    for creating cluster

> For running in python 3.7.x – scala 2.11, spark 2.x.x
> 
> For running in python 3.8.x – scala 2.12, spark 3.x.x

iii.  **Autopilot Options –** Enable autoscaling has to be on

iv.  **Worker Types –** General purpose (14GB Memory, 4 cores), min
    worker – 2, max worker-8

v.  **Driver Types -** General purpose (14GB Memory, 4 cores)

> Notes – Users can change this worker types and driver types
> configurations according to jobs complexity.

4.  **Parameters –** [ DBFS path to config.yaml , global_run_type]

Eg. - ["/dbfs/FileStore/tables/configs_income_azure.yaml
", "databricks"]

5.  **Dependent libraries -**

![https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image4.png](https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image4.png)

i.  Add wheel file (.whl file) by uploading from local machine
    (dist/.whl file)

> Library Source- Upload
> 
> Library Type- Python Whl

ii.  Add jars by uploading from local machine (jars/.jar file)

> Library Source- Upload
> 
> Library Type- Jar
> 
> Note: The another way is that we can also give location of DBFS path directly by clicking on
> DBFS/ADLS (Library Source) if required wheel file and jars are already
> uploaded there from local machine.

After setting all these required steps in task, click create and jobs
will be created successfully.

![https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image5.png](https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image5.png)

For running these jobs, click on run now and then jobs will be triggered
automatically.

Option for scheduling these jobs for running automatically are also available.

**Note:** Attaching links that will brief about creating jobs, running as well as scheduling jobs for reference.
- [Jobs](https://docs.microsoft.com/en-us/azure/databricks/jobs)
- [Jobs](https://docs.databricks.com/jobs.html)

Once the job finishes successfully, users will be able to see their status as succeeded. we can see that in the above images.

The reports and all the intermediate outputs are saved at the required
DBFS path. All the required DBFS path location is updated in
configs.yaml file.



## 2. Anovos on Azure Databricks using Azure blob storage container by mounting to DBFS

**Following are the steps required for running end to end anovos on Azure Databricks Platform using Azure blob storage container by mounting to DBFS:**

**Step1: Installing/Downloading Anovos**

Clone the Anovos Repository on local machine using command:

    git clone <https://github.com/anovos/anovos.git>

After cloning, go to anovos directory and execute the following command
to clean and build the latest modules in dist folder:

    make clean build

**Step2: Creating wheel file in local machine**

Note: Before creating this wheel file, you will need to install python
packages using command:

    pip install wheel setuptools

Then create wheel file using setup.py file that contains all python
scripts and dependencies using command:

    python setup.py bdist_wheel --universal

After the command finishes executing successfully, a folder name ‘dist’
contains the .whl file which is your wheel file. Wheel file contains all
the packages and scripts which are given in setup.py for running anovos
in azure databricks.

**Step3: Copy necessary files to Azure blob storage container from local machine
machine**

Copy the following files from local machine to Azureblob storage container directly from UI or from CLI commands:

- dist/income_dataset (optional)

  - This folder contains our demo dataset.This is sample dataset that is shown for reference.Users can copy their own dataset.

- dist/main.py

  - This is sample script to show how different functions from Anovos
    module can be stitched together to create a workflow.

  - This script takes input from a yaml configuration file

- dist/.whl file

  - This is wheel file that contains all the packages and required
    scripts that was built in step 2

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

**Step4: Mount a container of Azure Blob Storage as a dbfs path in Azure Databricks Platform**
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
[Azure Blob storage](https://docs.microsoft.com/en-us/azure/databricks/data/data-sources/azure/azure-storage#:~:text=Mount%20Azure%20Blob%20storage%20containers%20to%20DBFS,-You%20can%20mount&text=All%20users%20have%20read%20and,immediately%20access%20the%20mount%20point)

**Note:** 
Mounting needs to be done only one time when we are using the same mount_name for mounting in dbfs. No need to mount when we are running again using same mount_name as it is already mounted.
To unmount a mount point, use the following command in Azure databricks notebook:

    dbutils.fs.unmount("/mnt/<mount-name>")
    
**Step5: Updation of config file for all input and output path according to dbfs mount path**

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
    [config_file_description](https://github.com/anovos/anovos-docs/blob/anovos_config_file_desc/docs/using-anovos/config_file.md)

**Step6: Copy updated config file from local machine to azure blob storage container using UI or from azure command in similar way like in step 3 for other files.**

    - config/configs_income_azure_blob_mount.yaml
    
This is the sample yaml configuration file which sets the argument for all functions for running anovos in Azure Databricks

**Step7: Creating jobs for running anovos by initiating cluster on Azure databricks Platform**

**•Task Details**
![https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image7.png](https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image7.png)

**a.Task Name** – Give any task name relevant to your project 

**b.Type** – Python, DBFS mount path of main.py script file 

**c.Cluster**

**Note**Attaching link that describes all the information related to cluster configurations.
[Configure clusters](https://docs.microsoft.com/en-us/azure/databricks/clusters/configure#cluster-configurations)
                                                **Cluster Configurations**
                                                
![https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image3.png](https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image3.png)                                               
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

![https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image6.png](https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image6.png)

For running these jobs, click on run now and then jobs will be triggered automatically.
Option for scheduling these jobs for running automatically are also available.

**Note:** Attaching links that will brief about creating jobs, running as well as scheduling jobs for reference.
- [Jobs](https://docs.microsoft.com/en-us/azure/databricks/jobs)
- [Jobs](https://docs.databricks.com/jobs.html)

Once the job finishes successfully, users will be able to see status as succeeded.we can see that in the below images.

![https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image5.png](https://raw.githubusercontent.com/anovos/anovos-docs/azure_databricks_docs/docs/assets/azure_databricks_images/image5.png)

The reports and all the intermediate outputs are saved at the required Azure blob storage container automatically once jobs finishes successfully.