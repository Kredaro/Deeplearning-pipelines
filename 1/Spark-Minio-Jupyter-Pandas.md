# Minio-Sparks-Jupyter-Pandas
Minio is open-source cloud object storage server. It follows Amazon S3 protocol and at times referred as Open-Source Amazon S3 alternative, which anyone can host on their own machines. It was developed with the focus to be efficient storage and retrieval of objects. 
As the requirements of user change, it is deemed to evolve to meet those requirements. These change in requirements were driven by emerging big data, analytics and machine learning workflows. This lead to the creation of S3 Select API, which gives SQL query powers to the object storage. After which, Minio has rolled out its implemention of Select API. 

## Minio S3 Select API support
The typical data flow prior to the release of the Select API would look like this:

 * Applications download the whole object, using `GetObject()` :
 * Load the object into local memory.
 * Start the query process while the object resides in memory.

With the S3 Select API, applications can now download specific subset of an object — only the subset that satisfies given Select query. This directly translates into efficiency and performance:
 * Reduced bandwidth requirements
 * Optimizes compute resources and memory
 * With the smaller memory footprint, more jobs can be run in parallel — with same compute resources
 * As jobs finish faster, there is better utilization of analysts and domain experts


## Setting up system configuration (VM/Instance)
The system configuration selected for the task is as mentioned below :

 * 10 cores CPUs
 * 40 GB memory (RAM)
 * 200 GB Disk Size (ROM)


## Downloading Minio Server and Client
### Minio Server
Follow the steps below to setup Minio Server : 
```sh
# Downloading Minio binary and copying to /opt
sudo wget -O /opt/minio https://dl.minio.io/server/minio/release/linux-amd64/minio
# Changing the file permission of binary to mark as executable
sudo chmod +x /opt/minio
# Creating a symbolic link to /usr/local/bin to make the file executable from any path
sudo ln -s /opt/minio /usr/local/bin/
# Making the data directory for storing the objects/data for minio server
mkdir ./data
# Running the Minio server with the data directory parameter
minio server ./data
```

### Minio Client
Follow the steps below to setup Minio Client :
```sh
# Downloading minio binary and copying to /opt
sudo wget -O /opt/mc https://dl.minio.io/client/mc/release/linux-amd64/mc
# Changing the file permission of binary to mark as executable
sudo chmod +x /opt/mc
# Creatin a symbolic link to /usr/local/bin to make the file executable from any path
sudo ln -s /opt/mc /usr/local/bin
# Executing the command with help parameter to ensure that installation was a success
mc --help
```


## Loading Sample Data
Follow the steps below to load Sample Data using Minio Client :
```sh
# Downloading the sample data of TotalPopulationBySex.csv from UN
curl "https://esa.un.org/unpd/wpp/DVD/Files/1_Indicators%20(Standard)/CSV_FILES/WPP2017_TotalPopulationBySex.csv" > TotalPopulation.csv
# Compressing the csv file
gzip TotalPopulation.csv
# Creating a new bucket
mc mb data/mycsvbucket
# Copying the compressed file inside bucket
mc cp TotalPopulation.csv.gz data/mycsvbucket/sampledata/
```

## Setting up Python Environment in the machine
Follow the steps below to setup Python v3.6 Environment :
```sh
# Adding the PPA for python-3.6
sudo add-apt-repository ppa:jonathonf/python-3.6
# Updating the archive list from the repository
sudo apt-get update
# Installing python3.6 and pip
sudo apt-get install -y python3.6 python3-pip
# Installing python3.6-venv
sudo apt-get install -y python3.6-venv
```

## Setting up Virtual Environment for Running Python
Follow the steps below to setup virtual environment and installing dependencies :
```sh
# Creating Virtual Environment for Python v3.6
python3.6 -m venv ./myvenv
# Activating the virtual environment
source ./myvenv/bin/activate
# Verify versions of python and pip
python --version
pip --version
# Installing Boto3 - AWS SDK for Python
pip install boto3
# Installing other dependencies
pip install -r requirements.txt
```

## Source Code of Example Python Application using Select-API of S3
```python
#!/usr/bin/env/env python3
import boto3
import os

s3 = boto3.client('s3',
                  endpoint_url='http://localhost:9000',
                  aws_access_key_id=os.environ.get("AWS_ACCESS", 'minio'),
                  aws_secret_access_key=os.environ.get("AWS_SECRET", 'minio123'),
                  region_name='us-east-1')

r = s3.select_object_content(
    Bucket=os.environ.get("BUCKET_NAME",'mycsvbucket'),
    Key=os.environ.get("OBJECT_PATH", 'sampledata/TotalPopulation.csv.gz'),
    ExpressionType='SQL',
    Expression="select * from s3object s where s.Location like '%United States%'",
    InputSerialization={
        'CSV': {
            "FileHeaderInfo": "USE",
        },
        'CompressionType': 'GZIP',
    },
    OutputSerialization={'CSV': {}},
)

for event in r['Payload']:
    if 'Records' in event:
        records = event['Records']['Payload'].decode('utf-8')
        print(records)
    elif 'Stats' in event:
        statsDetails = event['Stats']['Details']
        print("Stats details bytesScanned: ")
        print(statsDetails['BytesScanned'])
        print("Stats details bytesProcessed: ")
        print(statsDetails['BytesProcessed'])
```

[Read More about Select API Here](https://docs.minio.io/docs/minio-select-api-quickstart-guide.html)


## Minio Spark-Select
With MinIO Select API support now generally available, any application can leverage this API to offload query jobs to the MinIO server itself.

>However, an application like Spark, used by thousands of enterprises already, if integrated with Select API, would create tremendous impact on the data science landscape — making Spark jobs faster by an order of magnitude.

Technically, it makes perfect sense for Spark SQL to push down possible queries to MinIO, and load only the relevant subset of object to memory for further analysis. This will make Spark SQL faster, use lesser compute/memory resources and allow more Spark jobs to be run concurrently.

![Spark Before/After S3 Select](https://cdn-images-1.medium.com/max/2400/1*A7GwBVHEW_r1OQmLQFvfIg.png)

To support this, we recently released the Spark-Select project to integrate the Select API with Spark SQL. The Spark-Select project is available under Apache License V2.0 on

 * GitHub (https://github.com/minio/spark-select)
 * Spark packages (https://spark-packages.org/package/minio/spark-select).

Spark-Select currently supports JSON , CSV and Parquet file formats for query pushdowns. This means the object should be one of these types for the push down to work.

### Setting up Java Environment for Spark Shell
```sh
# Adding ppa to local repository
sudo add-apt-repository ppa:webupd8team/java
# Updating repository archives
sudo apt update
# Installing Oracle Java8
sudo apt install -y oracle-java8-installer
# Verifying the java installation
javac -version
# Setting Oracle Java8 as default (In case of multiple java versions)
sudo apt install -y oracle-java8-set-default
# Setting up environment variable (Also, add this to the `~/.bashrc` file to apply for next boot)
export JAVA_HOME=/usr/lib/jvm/java-8-oracle
export PATH=$PATH:$JAVA_HOME/bin
```

### Installation of Apache Spark and Hadoop
Steps to install Apache Spark is as follow :
```sh
# Download Spark v2.3.0 without Hadoop
wget http://archive.apache.org/dist/spark/spark-2.3.0/spark-2.3.0-bin-without-hadoop.tgz
# Extracting the compressed file
sudo tar -C /opt/ -xvf spark-2.3.0-bin-without-hadoop.tgz
# Setting up environment variable (Also, add this to the `~/.bashrc` file to apply for next boot)
export SPARK_HOME=/opt/spark-2.3.0-bin-without-hadoop
export PATH=$PATH:$SPARK_HOME/bin
```

Steps to install Apache Hadoop is as follow :
```sh
# Download Hadoop v2.8.2
wget https://archive.apache.org/dist/hadoop/core/hadoop-2.8.2/hadoop-2.8.2.tar.gz
# Extracting the compressed file
sudo tar -C /opt/ -xvf hadoop-2.8.2.tar.gz
# Setting up environment for Hadoop
export HADOOP_HOME=/opt/hadoop-2.8.2
export PATH=$PATH:$HADOOP_HOME/bin
export SPARK_DIST_CLASSPATH=$(hadoop classpath)
# 
export LD_LIBRARY_PATH=$HADOOP_HOME/lib/native
```

### Setting up Minio Server endpoint and credentials
Open the file `$HADOOP_HOME/etc/hadoop/core-site.xml` for editing. In the example xml file below, Minio server is running at http://127.0.0.1:9000 with access key **minio** and secret key **minio123**. Make sure to update relevant sections with valid Minio server endpoint and credentials.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
  <property>
    <name>fs.s3a.endpoint</name>
    <description>AWS S3 endpoint to connect to. An up-to-date list is
      provided in the AWS Documentation: regions and endpoints. Without this
      property, the standard region (s3.amazonaws.com) is assumed.
    </description>
    <value>http://127.0.0.1:9000</value>
  </property>

  <property>
    <name>fs.s3a.access.key</name>
    <description>AWS access key ID.</description>
    <value>minio</value>
  </property>

  <property>
    <name>fs.s3a.secret.key</name>
    <description>AWS secret key.</description>
    <value>minio123</value>
  </property>

  <property>
    <name>fs.s3a.path.style.access</name>
    <value>true</value>
    <description>Enable S3 path style access ie disabling the default virtual hosting behaviour.
      Useful for S3A-compliant storage providers as it removes the need to set up DNS for virtual hosting.
    </description>
  </property>

  <property>
    <name>fs.s3a.impl</name>
    <value>org.apache.hadoop.fs.s3a.S3AFileSystem</value>
    <description>The implementation class of the S3A Filesystem</description>
  </property>
</configuration>
```


### Minio Spark-Select with Spark Shell

Note: Make sure JAVA_HOME has been set before setting up Spark Shell.

Spark-Select can be integrated with Spark via spark-shell , pyspark , spark-submit etc. You can also add it as Maven dependency, sbt-spark-package or a jar import.

Let’s see an example of using `spark-select` with `spark-shell`.

 * Start Minio server and configure mc to interact with this server.
 * Create a bucket and upload a sample file :
```sh
curl "https://raw.githubusercontent.com/minio/spark-select/master/examples/people.csv" > people.csv
mc mb data/sjm-airlines
mc cp people.csv data/sjm-airlines
```
 * Download the sample code from `spark-select` :
```sh
curl "https://raw.githubusercontent.com/minio/spark-select/master/examples/csv.scala" > csv.scala
```

 * Downloading depedencies and adding it to `spark` :
```sh
# Creating jars folder
mkdir jars
# Changing current directory to ./jars
cd jars
# Downloading jar dependencies
wget http://central.maven.org/maven2/org/apache/hadoop/hadoop-aws/2.8.2/hadoop-aws-2.8.2.jar
wget http://central.maven.org/maven2/org/apache/httpcomponents/httpclient/4.5.3/httpclient-4.5.3.jar
wget http://central.maven.org/maven2/joda-time/joda-time/2.9.9/joda-time-2.9.9.jar
wget http://central.maven.org/maven2/com/amazonaws/aws-java-sdk-s3/1.11.524/aws-java-sdk-s3-1.11.524.jar
wget http://central.maven.org/maven2/com/amazonaws/aws-java-sdk-core/1.11.524/aws-java-sdk-core-1.11.524.jar
wget http://central.maven.org/maven2/com/amazonaws/aws-java-sdk/1.11.524/aws-java-sdk-1.11.524.jar
wget http://central.maven.org/maven2/com/amazonaws/aws-java-sdk-kms/1.11.524/aws-java-sdk-kms-1.11.524.jar
# Downloading Minio-Select jar dependency
wget http://central.maven.org/maven2/io/minio/spark-select_2.11/2.0/spark-select_2.11-2.0.jar
# Copying all the jars to $SPARK_HOME/jars/
cp *.jar $SPARK_HOME/jars/
```

 * Configure Spark with Minio. Detailed steps are available in [this document](https://github.com/minio/cookbook/blob/master/docs/apache-spark-with-minio.md).
 * While starting Spark, use --packages flag to add spark-select package :
```java
$SPARK_HOME/bin/spark-shell --master local[4] --packages io.minio:spark-select_2.11:2.0 
```
 * After spark-shell is successfully invoked, execute the csv.scala file :
```scala
scala> :load csv.scala
Loading examples/csv.scala...
import org.apache.spark.sql._
import org.apache.spark.sql.types._
defined object app

scala> app.main(Array())
+-------+---+
|   name|age|
+-------+---+
|Michael| 31|
|   Andy| 30|
| Justin| 19|
+-------+---+
()
+-------+---+
|   name|age|
+-------+---+
|Michael| 31|
|   Andy| 30|
+-------+---+
()
scala>
```
![Without loading csv.scala file](https://i.imgur.com/otUDSFI.jpg)

You can see, only the fields with value age > 19 are returned.
For more, checkout the [spark-select project](https://github.com/minio/spark-select).


## Spark-Shell using PySpark and Minio
Make sure all of the `aws-java-sdk` jars are present under `$SPARK_HOME/jars/` or added to the `spark.jars.packages` in *spark-defaults.conf* file, before executing the following commands :

```sh
# Execute pyspark with spark-select package by minio
pyspark --packages io.minio:spark-select_2.11:2.0
```

You should be seeing the following screen :
```
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 2.3.0
      /_/

Using Python version 2.7.12 (default, Nov 12 2018 14:36:49)
SparkSession available as 'spark'.
>>>
```

Let's execute following lines to use pyspark with minio select :
```python
>>> from pyspark.sql.types import *
>>> schema = StructType([StructField('name', StringType(), True),StructField('age', IntegerType(), True)])
>>> df = spark.read.format("minioSelectCSV").schema(schema).load("s3://sjm-airlines/people.csv")
>>> df.show()
+-------+---+
|   name|age|
+-------+---+
|Michael| 31|
|   Andy| 30|
| Justin| 19|
+-------+---+
>>> df.select("*").filter("age > 19").show()
+-------+---+
|   name|age|
+-------+---+
|Michael| 31|
|   Andy| 30|
+-------+---+
```

## Minio, Spark with Jupyter Notebook

### Setting up environment, installing jupyter and running
```sh
# Downloading shell script to install Jupyter using Anaconda
wget https://repo.anaconda.com/archive/Anaconda3-2018.12-Linux-x86_64.sh
# Making the shell script executable
chmod +x ./Anaconda3-2018.12-Linux-x86_64.sh
# Running the shell script with bash
bash Anaconda3-2018.12-Linux-x86_64.sh
# Create new conda environment with minimal environment with only python installed.
conda create -n myconda python=3
# Put your self inside this environment run
conda activate myconda
# Verify the Anaconda Python v3.x Terminal inside the environment. To exit, press CTRL+D or `exit()`
python
# Install Jupyter Notebook inside the environment
conda install jupyter
# Install findspark inside the environment using conda-forge channel
conda install -c conda-forge findspark
# (Optional) Setting up jupyter notebook password, enter the desired password (If not set, have to use randomly generated tokens each time)
jupyter notebook password
# Running Jupyter Notebook and making it available to public at port 8888
jupyter notebook --ip 0.0.0.0  --port 8888
```

If everything goes well, you should be seeing the following :
```sh
[I 06:50:01.156 NotebookApp] JupyterLab extension loaded from /home/prashant/anaconda3/lib/python3.7/site-packages/jupyterlab
[I 06:50:01.157 NotebookApp] JupyterLab application directory is /home/prashant/anaconda3/share/jupyter/lab
[I 06:50:01.158 NotebookApp] Serving notebooks from local directory: /home/prashant
[I 06:50:01.158 NotebookApp] The Jupyter Notebook is running at:
[I 06:50:01.158 NotebookApp] http://(instance-1 or 127.0.0.1):8888/
[I 06:50:01.158 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
```

#### Deactivate conda virtual environment
Example:
```sh
# Usage: conda deactivate <environment-name>
conda deactivate myconda
```

#### Converting python scripts(.py) file to jupyter notebook(.ipynb) file
Example:
```sh
# Installing p2j using python-pip
pip install p2j

# Generating .ipynb file out of some sample script.py using p2j
p2j script.py
```

#### Converting jupyter notebook(.ipynb) file to python scripts(.py) file
Example:
```sh
# Generating script.py file out of some sample .ipynb file using jupyter nbconvert
jupyter nbconvert script.ipynb
```

### Creating sample python file
Let's create a python file **spark-minio.py** with the codes below :

```python
# Import sys and print the python environment
import sys
print(sys.executable)
# Import findspark to find spark make it accessible at run time
import findspark
findspark.init()
# Import pyspark and its components
import pyspark
from pyspark.sql.types import *
from pyspark.sql import SparkSession

# Creating SparkSession
spark = SparkSession.builder.getOrCreate()
# Creating schema of the CSV fields
schema = StructType([StructField('name', StringType(), True),StructField('age', IntegerType(), True)])
# Creating a dataframe to use minioSelectCSV with the specified schema and load data from a CSV in S3 
df = spark.read.format("minioSelectCSV").schema(schema).load("s3://sjm-airlines/people.csv")
# Displaying all data in the CSV
df.show()
# Displaying all the data in the csv for which age is greater than 19
df.select("*").filter("age > 19").show()
```

Now, converting the python code(spark-minio.py) to jupyter notebook compatible file (.ipynb) :

```sh
# Generating spark-minio.ipynb file out of spark-minio.py
p2j spark-minio.py
```

### Running .ipynb file from the jupyter notebook UI
Let's open the UI running at http://(server-public-ip-address/localhost):8888/

Enter the jupyter notebok password (or the token) and then, you should be seeing something like this :
![Jupyter Notebook](https://i.imgur.com/4uIf5Ta.jpg)

Select **spark-minio.ipynb** file and click on run, if everything went right, you should be getting the screen below :
![Jupyter Notebook](https://i.imgur.com/X47Kv93.jpg)

### Running Some Live Examples
In Jupyter Notebook, go to File Tab > New Notebook > Python 3 (Or any other kernel). Try the following pyspark example on the data on present in Minio : 
```py
import findspark
findspark.init()
import pyspark
from pyspark.sql.types import *
from pyspark.sql import SparkSession
spark = SparkSession.builder.getOrCreate()
schema = StructType([StructField('name', StringType(), True),StructField('age', IntegerType(), True)])
df = spark.read.format("minioSelectCSV").schema(schema).load("s3://sjm-airlines/people", compression="gzip")
df.createOrReplaceTempView("people")
print("List of all people :")
df2.show()
print("People with age less than 20 :")
df2 = spark.sql("SELECT * FROM people where age>20")
df2.show()
```

If the steps are properly followed, you should be seeing following in the jupyter notebook:
```scala
List of all people :
+-------+---+
|   name|age|
+-------+---+
|Michael| 31|
|   Andy| 30|
+-------+---+

People with age less than 20 :
+-------+---+
|   name|age|
+-------+---+
|Michael| 31|
|   Andy| 30|
+-------+---+
```

For the next example, we are gonna use SQL query capability of Spark dataframe on comparatively big CSV with **13 header fields** and **2000251** entries. For the task, at first, we are gonna download the CSV with gzipped compression from the following link.
```sh
wget https://gist.github.com/coolboi567/d65254b1ac4b64c5969bd6309d8f8424/raw/955d6eccc4990b7fed0c421c96e7bd290bab952e/natality00.gz
```

Create a new bucket in Minio, here, we are naming the bucket `spark-experiment` and upload the downloaded file to that bucket.
You can use Minio UI for the task. Or, you can use Minio Client - `mc` for the same.
```sh
# Go to the `data` folder which Minio Server is pointing to 
cd ~/data
# Creating a new bucket
mc mb spark-experiment
# Copying the compressed file inside the bucket
mc cp ../natality00.gz spark-experiment
```

Now, let's try the following script in Jupyter Noteboook. You can either create new cell in the same old notebook or create a new notebook for running the script. 
```python
import findspark
findspark.init()
import pyspark
from pyspark.sql.types import *
from pyspark.context import SparkContext
from pyspark.sql.session import SparkSession

sc = SparkContext.getOrCreate()
spark = SparkSession(sc)

df = spark.read.format("csv").option("header", "true").load("s3a://spark-experiment/natality00.gz")
query="SELECT is_male, count(*) as count, AVG(weight_pounds) AS avg_weight FROM natality GROUP BY is_male"
df.createOrReplaceTempView("natality")
df2 = spark.sql(query)
df2.show()
```

Upon running the script in notebook, you should get the following output:
```scala
+-------+-------+-----------------+
|is_male|  count|       avg_weight|
+-------+-------+-----------------+
|  false| 975147| 7.17758067338709|
|   true|1025104|7.439839161360215|
+-------+-------+-----------------+
```

## Visualization with charts and graphs using Pandas

### Installation
Install Pandas using `conda`. PySpark dataframe requires `pandas >= 0.19.2` for executing any of the features by pandas.
```sh
# Installing pandas and matplotlib. Make sure inside the created conda virtual environment, when you are running the following command
conda install pandas matplotlib
```

### Example 1
Let's display some charts on the report that we got in the previous example. Let's create a new cell on same notebook rather than integrating the following snippet in the above code, to reduce the time to plot multiple charts on same report.
```python
df3 = df2.toPandas()
df3.plot(x='is_male', y='count', kind='bar')
df3.plot(x='is_male', y='avg_weight', kind='bar')
```
![Total Count](https://i.imgur.com/8OlZNCP.jpg "Total Count") ![Average Weight](https://i.imgur.com/Rym7DkS.jpg "Average Weight")

**Observation**: From the generated chart, we can observe that gender of the child doesn't have any signficant role neither in average weight of the child nor wide difference can be seen in total count of the two gender divisions.

### Example 2
Now, let us try another example. Let's create a new notebook for this. If you don't wish to create a new one, you can try on a new cell of the previous notebook.
```python
import findspark
findspark.init()
import pyspark
from pyspark.sql.types import *

from pyspark.context import SparkContext
from pyspark.sql.session import SparkSession
sc = SparkContext.getOrCreate()
spark = SparkSession(sc)

df = spark.read.format("csv").option("header", "true").load("s3a://spark-experiment/natality00.gz")
query="SELECT mother_age, count(*) as count, AVG(weight_pounds) AS avg_weight FROM natality GROUP BY mother_age"
df.createOrReplaceTempView("natality")
print("Based on mother_age, total count and average weight is as follow : ")
df2 = spark.sql(query)
df3 = df2.toPandas()
df4= df3.sort_values('mother_age')
print("***DONE***")
```

After running the program, when it prints `DONE`. Create a new cell below and run the following snippet:
```python
df4.plot(x='mother_age', y='count')
df4.plot(x='mother_age', y='avg_weight')
```

![Total Count](https://i.imgur.com/gTWkH8i.jpg "Total Count") ![Average Weight](https://i.imgur.com/Az67FC3.jpg "Average Weight")
**Observation**: We can observe that, most of the mothers are between 20-30 age range when they gave birth. While the average weight of the children is shows some decline in case of mothers at young age, it shows significant decrease in children's average weight in case of mothers at old age.

### Example 3
This one will be one interesting one. We will plot a chart with scatter graph.
```python
import findspark
findspark.init()
import pyspark
from pyspark.sql.types import *

from pyspark.context import SparkContext
from pyspark.sql.session import SparkSession
sc = SparkContext.getOrCreate()
spark = SparkSession(sc)

df = spark.read.format("csv").option("header", "true").load("s3a://spark-experiment/natality00.gz")
query="SELECT INT(gestation_weeks), COUNT(*) AS count, AVG(weight_pounds) AS avg_weight FROM natality GROUP BY gestation_weeks"
df.createOrReplaceTempView("natality")
print("Based on gestation_weeks, total count and average weight is as follow : ")
df2 = spark.sql(query)
df3 = df2.toPandas()
df4= df3.sort_values('gestation_weeks')
print("***DONE***")
```
Like we did before, after `DONE` is printed. Create a new cell below with the following snippet. Here, we are introducing `matplotlib axes object(ax)` and `dataframe.describe()`.
```python
import matplotlib.pyplot as plt
fig, ax = plt.subplots()
df4.plot(kind="scatter", x="gestation_weeks", y="avg_weight", s=100, c="count", cmap="RdYlGn", ax=ax);
df4.describe()
```
![Scatter Chart](https://i.imgur.com/AF6oCVk.jpg "Scatter Chart") ![DataFrame Describe](https://i.imgur.com/X8Cw0jF.jpg "DataFrame Describe")
**Observation**: From the scattar graph, it can be seen that maximum number of mothers' gestation period was 40 weeks and children born around this period are mostly of more weight than rest. We can also see that there are around 100k entries for which *gestation_weeks* is 99, which is not possible in reality. So, we can be sure that these are the dummy values.

Note: List of possible `cmap` i.e. *colormap* can be found [here](https://gist.github.com/coolboi567/ab86e34febe7dba1d05bf0b2b7f56611)
>>>>>>> 150aad4f9ff325a99dd6cb596733b08d2f49aaa6:1/Spark-Minio-Jupyter-Pandas.md
