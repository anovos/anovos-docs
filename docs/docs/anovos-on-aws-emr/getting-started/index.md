### Step 1: Installing/Downloading Anovos

Clone the Anovos Repository on local environment using command:

`git clone https://github.com/anovos/anovos.git`
or 
`pip3 install "git+https://github.com/anovos/anovos.git"`

After cloning, go to anovos directory and execute the following command to clean and build the latest modules in dist folder:

`make clean build`

### Step 2: Copy necessary files to AWS S3
Copy the following files to AWS S3:

- `dist/anovos.zip`
    -	This file contains all Anovos modules
    -	Zipped version is mandatory for running importing the modules as –py-files
- `dist/income_dataset` (optional)
    -	This folder contains our demo dataset
- `dist/main.py`
    -	This is sample script to show how different functions from Anovos module can be stitched together to create a workflow.
    -	The users can create their own workflow script by importing the necessary functions.
    -	This script takes input from a yaml configuration file
- `dist/configs.yaml`
    -	This is the sample yaml configuration file which sets the argument for all functions.
    -	Update configs.yaml for all input & output s3 paths. All other changes depends upon the dataset being used.
    - bin/req_packages_anovos.sh
    -	This shell script is used to install all required packages to run Anovos on EMR 

AWS copy command:
`aws s3 cp --recursive  <local file path> <s3 path> --profile  <profile name>`

##  Step 3: Creating Cluster

-   Software Configuration 
    - Emr-5.33.0 
	- Hadoop-2.10.1 
	- Spark-2.4.7 
	- Hive-2.3.7 

- Spark Submit Details

- Deploy mode client

- Spark-submit options
- `--num-executors 1000`
- `--executor-cores 2` 
- `--executor-memory 20g` 
- `--driver-memory 20G` 
- `--driver-cores 4` 
- `--conf spark.driver.maxResultSize=15g` 
- `--conf spark.yarn.am.memoryOverhead=1000m`
- `--conf spark.executor.memoryOverhead=2000m` 
- `--conf spark.kryo.referenceTracking=false` 
- `--conf spark.network.timeout=18000s`
- `--conf spark.executor.heartbeatInterval=12000s`
- `--conf spark.dynamicAllocation.executorIdleTimeout=12000s` 
- `--conf spark.rpc.message.maxSize=1024` 
- `--conf spark.yarn.maxAppAttempts=1` 
- `--conf spark.speculation=false` 
- `--conf spark.kryoserializer.buffer.max=1024` 
- `--conf spark.executor.extraJavaOptions=-XX:+UseG1GC` 
- `--conf spark.driver.extraJavaOptions=-XX:+UseG1GC`
- `--packages org.apache.spark:spark-avro_2.11:2.4.0` 
- `--jars s3://mw.com.ds.kajanan/Vishnu/ml_ingest/jars/histogrammar-sparksql_2.11-1.0.20.jar,s3://mw.com.ds.kajanan/Vishnu/ml_ingest/jars/histogrammar_2.11-1.0.20.jar` 
- `--py-files s3://mw.com.ds.kajanan/Sumit/Anovos_testing/income_data_emr/com.zip`

Application location*: s3 path of main.py file

Arguments: 
<path to config.yaml>, <local_or_emr>
	
    - Bootstrap Actions
    script location : path of bootstrap shell script (req_packages_anovos.sh)

