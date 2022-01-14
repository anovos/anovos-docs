# Module <code>spark</code>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
from os import environ
import __main__
from pyspark import SQLContext
from pyspark.sql import SparkSession
def init_spark(app_name=&#39;anovos&#39;, master=&#39;local[*]&#39;, jar_packages=[],
py_files=[], spark_configs={}):
&#34;&#34;&#34;
Args:
app_name: Name of Spark app. (Default value = &#39;anovos&#39;)
master: Cluster connection details
Defaults to local[*] which means to run Spark locally with as many worker threads as logical cores on the machine.
jar_packages: List of Spark JAR package names. (Default value = [])
files: List of files to send to Spark cluster (master and workers).
spark_config: Dictionary of config key-value pairs.
py_files:
(Default value = [])
spark_configs:
(Default value = {})
Returns:
A tuple of references to the Spark Session, Spark Context &amp; SQL Context.
&#34;&#34;&#34;
# detect execution environment
flag_repl = not (hasattr(__main__, &#39;__file__&#39;))
flag_debug = &#39;DEBUG&#39; in environ.keys()
if not (flag_repl or flag_debug):
spark_builder = SparkSession.builder.appName(app_name)
else:
spark_builder = SparkSession.builder.master(master).appName(app_name)
# create Spark JAR packages string
spark_jars_packages = &#39;,&#39;.join(list(jar_packages))
spark_builder.config(&#39;spark.jars.packages&#39;, spark_jars_packages)
spark_files = &#39;,&#39;.join(list(py_files))
spark_builder.config(&#39;spark.files&#39;, spark_files)
# add other config params
for key, val in spark_configs.items():
spark_builder.config(key, val)
# create spark session and contexts
spark = spark_builder.getOrCreate()
sc = spark.sparkContext
sqlContext = SQLContext(sc)
return spark, sc, sqlContext
configs = {&#39;app_name&#39;: &#39;Anovos_pipeline&#39;,
&#39;jar_packages&#39;: [&#34;io.github.histogrammar:histogrammar_2.11:1.0.20&#34;,
&#34;io.github.histogrammar:histogrammar-sparksql_2.11:1.0.20&#34;,
&#34;org.apache.spark:spark-avro_2.11:2.4.0&#34;],
&#39;py_files&#39;: [],
&#39;spark_configs&#39;: {&#39;spark.sql.session.timeZone&#39;: &#39;GMT&#39;,
&#39;spark.python.profile&#39;: &#39;false&#39;}}
spark, sc, sqlContext = init_spark(**configs)
```
</details>
## Functions
<dl>
<dt id="anovos.shared.spark.init_spark"><code class="name flex">
<span>def <span class="ident">init_spark</span></span>(<span>app_name='anovos', master='local[*]', jar_packages=[], py_files=[], spark_configs={})</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>app_name</code></strong></dt>
<dd>Name of Spark app. (Default value = 'anovos')</dd>
<dt><strong><code>master</code></strong></dt>
<dd>Cluster connection details</dd>
</dl>
<p>Defaults to local[*] which means to run Spark locally with as many worker threads as logical cores on the machine.
jar_packages: List of Spark JAR package names. (Default value = [])
files: List of files to send to Spark cluster (master and workers).
spark_config: Dictionary of config key-value pairs.
py_files:
(Default value = [])
spark_configs:
(Default value = {})</p>
<h2 id="returns">Returns</h2>
<p>A tuple of references to the Spark Session, Spark Context &amp; SQL Context.</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def init_spark(app_name=&#39;anovos&#39;, master=&#39;local[*]&#39;, jar_packages=[],
py_files=[], spark_configs={}):
&#34;&#34;&#34;
Args:
app_name: Name of Spark app. (Default value = &#39;anovos&#39;)
master: Cluster connection details
Defaults to local[*] which means to run Spark locally with as many worker threads as logical cores on the machine.
jar_packages: List of Spark JAR package names. (Default value = [])
files: List of files to send to Spark cluster (master and workers).
spark_config: Dictionary of config key-value pairs.
py_files:
(Default value = [])
spark_configs:
(Default value = {})
Returns:
A tuple of references to the Spark Session, Spark Context &amp; SQL Context.
&#34;&#34;&#34;
# detect execution environment
flag_repl = not (hasattr(__main__, &#39;__file__&#39;))
flag_debug = &#39;DEBUG&#39; in environ.keys()
if not (flag_repl or flag_debug):
spark_builder = SparkSession.builder.appName(app_name)
else:
spark_builder = SparkSession.builder.master(master).appName(app_name)
# create Spark JAR packages string
spark_jars_packages = &#39;,&#39;.join(list(jar_packages))
spark_builder.config(&#39;spark.jars.packages&#39;, spark_jars_packages)
spark_files = &#39;,&#39;.join(list(py_files))
spark_builder.config(&#39;spark.files&#39;, spark_files)
# add other config params
for key, val in spark_configs.items():
spark_builder.config(key, val)
# create spark session and contexts
spark = spark_builder.getOrCreate()
sc = spark.sparkContext
sqlContext = SQLContext(sc)
return spark, sc, sqlContext
```
</details>
</dd>
</dl>