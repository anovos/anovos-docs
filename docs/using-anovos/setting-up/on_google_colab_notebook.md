**Anovos on Google Colab Notebook**

Colab notebooks are Jupyter notebooks that run in the cloud and are highly integrated with Google Drive, making them easy to set up, access, and share.Google Colab is a cloud-based Jupyter notebook environment from Google Research. With its simple and easy-to-use interface, Colab helps you get started with your data science journey with almost no setup.

The following articles will give information about introduction to Google Colab Notebook and the usage of this platform and its benefits.
[Google Colab — The Beginner’s Guide](https://medium.com/lean-in-women-in-tech-india/google-colab-the-beginners-guide-5ad3b417dfa)
[How to use Google Colab](https://www.geeksforgeeks.org/how-to-use-google-colab/)
[Google Colab Tutorial for Data Scientists](https://www.datacamp.com/community/tutorials/tutorial-google-colab-for-data-scientists?utm_source=adwords_ppc&utm_medium=cpc&utm_campaignid=1455363063&utm_adgroupid=65083631748&utm_device=c&utm_keyword=&utm_matchtype=&utm_network=g&utm_adpostion=&utm_creative=332602034358&utm_targetid=aud-299261629574:dsa-429603003980&utm_loc_interest_ms=&utm_loc_physical_ms=9061848&gclid=Cj0KCQiA3-yQBhD3ARIsAHuHT64SOfn_qff9_FDLEdg40qL4YDdBjJUJI6mApoxPcns96oLIwGaeSBAaArgkEALw_wcB)

**Following are the steps required for running anovos on google colab notebook :**

**Step1: Installing all the dependencies in colab notebook for setting
pyspark environment**

Anovos builds on Spark, which is not available by default in Google Colab. Hence, before we can start working with steps for running Anovos,we need to install all the dependencies in colab notebook for setting pyspark environment.

i)  Installing Java using command:

    !apt-get install openjdk-8-jdk-headless -qq > /dev/null

ii)  Installing spark using command:

    !wget -q <https://archive.apache.org/dist/spark/spark-2.4.7/spark-2.4.7-bin-hadoop2.7.tgz>

**Note**: Used java8 and spark 2.4.7 version (change the version as per compatibility if needed)
   
   To use Anovos, you need compatible versions of Apache Spark, Java and Python.

   Currently, we officially support the following combinations:

       Apache Spark 2.4.x on Java 8 with Python 3.7.x
       Apache Spark 3.1.x on Java 11 with Python 3.9.x or python 3.8.x
       Apache Spark 3.2.x on Java 11 with Python 3.9.x

iii)  Unzip the spark file to the current folder using command:
    
    !tar xf spark-2.4.7-bin-hadoop2.7.tgz

iv)  Setting java and spark folder to our system path environment using
    following command:
    
    Import os
    
    os.environ["JAVA_HOME"] = "/usr/lib/jvm/java-8-openjdk-amd64"
    
    os.environ["SPARK_HOME"] = "/content/spark-2.4.7-bin-hadoop2.7"

v)  Installing findspark using command:
    
    !pip install findspark

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
    
**Note**: Anovos directory is present under Files/content/ in notebook. 
The above articles [Google Colab Tutorial for Data Scientists](https://www.datacamp.com/community/tutorials/tutorial-google-colab-for-data-scientists?utm_source=adwords_ppc&utm_medium=cpc&utm_campaignid=1455363063&utm_adgroupid=65083631748&utm_device=c&utm_keyword=&utm_matchtype=&utm_network=g&utm_adpostion=&utm_creative=332602034358&utm_targetid=aud-299261629574:dsa-429603003980&utm_loc_interest_ms=&utm_loc_physical_ms=9061848&gclid=Cj0KCQiA3-yQBhD3ARIsAHuHT64SOfn_qff9_FDLEdg40qL4YDdBjJUJI6mApoxPcns96oLIwGaeSBAaArgkEALw_wcB) will brief about Working with data from various sources.This part will describe about Cloning a GitHub repo into Colab instance and after cloning where the directory is present in notebooks. Sharing the images also for reference.
![https://github.com/anovos/anovos-docs/blob/google_colab_notebook_docs/docs/assets/google_colab_notebook_images/image1_colab.png](https://github.com/anovos/anovos-docs/blob/google_colab_notebook_docs/docs/assets/google_colab_notebook_images/image1_colab.png)

Further install the anovos related dependencies using command:

    !pip install -r requirements.txt
    
**Step3: Updation of necessary files according to use case**

Go inside dist folder(ie, present under Files/content/anovos/dist/) in the notebook and update the following.

**Note**:Firstly download the required below files in local machine which needs to be updated and then update it according to need and use case.we can download files like this way:
![https://github.com/anovos/anovos-docs/blob/google_colab_notebook_docs/docs/assets/google_colab_notebook_images/image2_colab.png](https://github.com/anovos/anovos-docs/blob/google_colab_notebook_docs/docs/assets/google_colab_notebook_images/image2_colab.png)


And then finally again upload updated files inside dist folder like this way:

![https://github.com/anovos/anovos-docs/blob/google_colab_notebook_docs/docs/assets/google_colab_notebook_images/image3_colab.png](https://github.com/anovos/anovos-docs/blob/google_colab_notebook_docs/docs/assets/google_colab_notebook_images/image3_colab.png)


  - > Update the input and output paths in configs.yaml and configure
    > the data set. You might also want to adapt the threshold settings
    > to your needs.
    
    **Note:** Attaching config file description link to get more information about updating input,output path and threshold settings according to use case.
    [config_file_description](https://github.com/anovos/anovos-docs/blob/anovos_config_file_desc/docs/using-anovos/config_file.md)

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
saved at the required path (ie, present under Files/content/anovos/dist/)in notebook.
