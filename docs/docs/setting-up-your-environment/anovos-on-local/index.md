### Step 1: Software Prerequisites

Anovos requires Spark (2.4.x), Python (3.7.*) and Java(8) to be setup in order to run in local environment. Kindly look at the following links for setting up Spark, Python and Java in local: 

[Spark 2.4.8](https://spark.apache.org/docs/2.4.8/ )
[Python 3.7](https://www.python.org/downloads/release/python-378/)
[Java 8](https://www.oracle.com/java/technologies/downloads/#java8 ) 

 	 
[Installing Apache on Mac OSX](https://kevinvecmanis.io/python/pyspark/install/2019/05/31/Installing-Apache-Spark.html)

[Using PySpark on Windows](https://towardsdatascience.com/installing-apache-pyspark-on-windows-10-f5f0c506bea1)

### Step2 : Running on local 

Anovos can be run on local in two ways: 

1.  By cloning the repo and running it via spark-submit 
    - Clone the Anovos repository on local environment using command: 
    `git clone https://github.com/anovos/anovos.git`
    - After cloning, go to the Anovos directory and execute the following command to clean and build the latest modules in dist folder: 
    `make clean build` 
    - Then go to dist/ folder and  
        - Update configs.yaml for all input & output paths. All other changes depends upon the dataset being used. Also update configs.yaml for other threshold settings for different functionalities. 
        - Update main.py - This is sample script to show how different functions from Anovos module can be stitched together to create a workflow. 
        - Update spark-submit.sh – This is a sample shell script to run spark application via spark-submit. 
    - Run using spark submit shell script 
    `nohup ./spark-submit.sh > run.txt &`
    - Check stdout logs while running 
    `tail -f run.txt`
    - Once the run has completed, the script will automatically open the final generated report "report_stats/ml_anovos_report.html" on the browser. 

2. By installing Anovos and importing it as you need it
    - Install anovos on local using: 
    `pip3 install "git+https://github.com/anovos/anovos.git"` 
    or 
    `pip3 install anovos`
    - Import Anovos as a package in your spark application 
    `import anovos`
    - Ensure dependent external packages are included in SparkSession 
        - If using your own SparkSession, include the following dependent packages while initializing: 
            - "io.github.histogrammar:histogrammar_2.11:1.0.20", 
            - "io.github.histogrammar:histogrammar-sparksql_2.11:1.0.20", 
            - "org.apache.spark:spark-avro_2.11:2.4.0" 
            Dependent Package JAR links : 
            [https://repo1.maven.org/maven2/io/github/histogrammar/histogrammar-sparksql_2.11/1.0.20/](https://repo1.maven.org/maven2/io/github/histogrammar/histogrammar-sparksql_2.11/1.0.20/)
            [https://repo1.maven.org/maven2/io/github/histogrammar/histogrammar_2.11/1.0.20/](https://repo1.maven.org/maven2/io/github/histogrammar/histogrammar_2.11/1.0.20/)
            [https://mvnrepository.com/artifact/org.apache.spark/spark-avro_2.11/2.4](https://mvnrepository.com/artifact/org.apache.spark/spark-avro_2.11/2.4.0)
    - Alternatively, if creating a new SparkSession please use the
    pre-configured SparkSession instance provided by anovos.shared.spark: 
    `from anovos.shared.spark import spark` 

    

 