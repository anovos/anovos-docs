Anovos on Azure Databricks

**Step1: Installing/Downloading Anovos**

Clone the Anovos Repository on local environment using command:

git clone <https://github.com/anovos/anovos.git>

After cloning, go to anovos directory and execute the following command
to clean and build the latest modules in dist folder:

make clean build

**Step2: Creating wheel file in local environment**

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

Copy the following files to DBFS directly from UI or from CLI commands:

  i. dist/income_dataset (optional)

<!-- end list -->

  - This folder contains our demo dataset

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

**Steps to copy files to DBFS using UI**

1.  Launch any workspace and go to data menu on databricks. After
    clicking data menu, options to upload file will appear and then by
    clicking ‘drop files to upload, or click to browse’, users will be
    able to upload the above required files from local environment.


**Databricks CLI configuration steps**

1.  Install *databricks-cli* using command – pip install databricks-cli

2.  Make sure the personal access tokens have already generated

3.  Copy the URL of databricks host and the personal access tokens

4.  Configure the CLI using command – databricks configure --token

**DBFS CLI Command for copy in Azure Databricks:**

dbfs cp -r source_folder_path destination_folder_path

eg. dbfs cp -r /home/user1/Desktop/dummy_folder
dbfs:/Filestore/tables/dummy_folder

**Step3: Creating jobs for running anovos by initiating cluster **
  - **Task Details**

<!-- end list -->

1.  **Task Name –** Give any task name relevant to your project

2.  **Type –** Python, location of main.py script file in DBFS

3.  **Cluster: **
**Cluster Configurations**

![Graphical user interface, text, application Description automatically
generated](media/image3.png)

i.  **Cluster mode –** Standard

ii.  **Databricks run time version –** Select the spark and scala version
    for creating cluster

> For running in python 3.7 – scala 2.11, spark 2.x.x
> 
> For running in python 3.8 – scala 2.12, spark 3.x.x

iii.  **Autopilot Options –** Enable autoscaling should be on

iv.  **Worker Types –** General purpose (14GB Memory, 4 cores), min
    worker – 2, max worker-8

v.  **Driver Types -** General purpose (14GB Memory, 4 cores)

> Notes – Users can change this worker types and driver types
> configurations according to jobs complexity

4.  **Parameters –** [ DBFS path to config.yaml , global_run_type]

Eg. - ["/dbfs/FileStore/tables/configs_income_azure.yaml
","databricks"]

5.  **Dependent libraries- **

i.  Add wheel file (.whl file) by uploading from local environment
    (dist/.whl file)

> Library Source- Upload
> 
> Library Type- Python Whl

ii.  Add jars by uploading from local environment (jars/.jar file)

> Library Source- Upload
> 
> Library Type- Jar
> 
> Note: we can also give location of DBFS path directly by clicking on
> DBFS/ADLS (Library Source) if required wheel file and jars are already
> uploaded there.

After setting all these required steps in task, click create and jobs
will be created successfully.

For running these jobs, click on run now and then jobs will be triggered
automatically.

Option for scheduling these jobs for running automatically are also
available.

Once the job finishes successfully, users will be able to see their logs
and status as succeeded.

The reports and all the intermediate outputs are saved at the required
DBFS path. All the required DBFS path location is updated in
configs.yaml file.
