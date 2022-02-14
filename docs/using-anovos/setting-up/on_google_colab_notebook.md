**Anovos on Google Colab Notebook**

**Step1: Installing all the dependencies in colab notebook for setting
pyspark environment**

i)  Installing Java using command:

> !apt-get install openjdk-8-jdk-headless -qq > /dev/null

ii)  Installing spark using command:

> !wget -q <https://archive.apache.org/dist/spark/spark-2.4.7/spark-2.4.7-bin-hadoop2.7.tgz>
> 
> Note: Used java8 and spark 2.4.7 version (change the version as per
> compatibility if needed)

iii)  Unzip the spark file to the current folder using command:
    
    !tar xf spark-2.4.7-bin-hadoop2.7.tgz

iv)  Setting java and spark folder to our system path environment using
    following command:
    
    Import os
    
    os.environ\["JAVA\_HOME"\] = "/usr/lib/jvm/java-8-openjdk-amd64"
    
    os.environ\["SPARK\_HOME"\] = "/content/spark-2.4.7-bin-hadoop2.7"

v)  Installing findspark using command:
    
    !pip install -q findspark

vi)  Installing pyspark using command:
    
    !pip install pyspark==2.4.7

**Step2: Installing/Downloading Anovos and its dependencies**

Clone the Anovos Repository on google colab notebook using command:

!git clone <https://github.com/anovos/anovos.git>

After cloning, go to the newly created anovos directory and execute the
following command in notebook itself to clean and build the latest
modules:

cd anovos

!make clean build

Then**,** Install the anovos related dependencies using command:

!pip install -r requirements.txt

**Note**: Anovos directory is present under Files/content/ in notebook

**Step3: Updation of necessary files according to use case**

Go to dist folder(ie, under Files/content/anovos/dist/) in the notebook
and update the following

  - > Update the input and output paths in configs.yaml and configure
    > the data set. You might also want to adapt the threshold settings
    > to your needs.

  - > Adapt the main.py sample script. It demonstrates how different
    > functions from *Anovos* can be stitched together to create a
    > workflow.

  - > If necessary, update spark-submit.sh. This is the shell script
    > used to run the Spark application via spark-submit.

**Step 4: Triggering workflow run**

Once everything is configured, you can start your workflow run using the
aforementioned script using command:

cd dist

!nohup ./spark-submit.sh > run.txt &

While the job is running, you can check the logs written to stdout using
command:

!tail -f run.txt

Once the run completes, the reports and all the intermediate outputs are
saved at the required path.
