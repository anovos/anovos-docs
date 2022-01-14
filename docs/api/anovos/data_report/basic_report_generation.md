# Module <code>basic_report_generation</code>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
import subprocess
from pathlib import Path
import datapane as dp
import pandas as pd
import plotly.express as px
from anovos.data_analyzer.association_evaluator import correlation_matrix, variable_clustering, IV_calculation, \
IG_calculation
from anovos.data_analyzer.quality_checker import duplicate_detection, nullRows_detection, nullColumns_detection, \
outlier_detection, IDness_detection, biasedness_detection, invalidEntries_detection
from anovos.data_analyzer.stats_generator import global_summary, measures_of_counts, measures_of_centralTendency, \
measures_of_cardinality, measures_of_dispersion, measures_of_percentiles, measures_of_shape
from anovos.shared.utils import ends_with
global_theme = px.colors.sequential.Plasma
global_theme_r = px.colors.sequential.Plasma_r
global_plot_bg_color = &#39;rgba(0,0,0,0)&#39;
global_paper_bg_color = &#39;rgba(0,0,0,0)&#39;
default_template = dp.HTML(
&#39;&lt;html&gt;&lt;img src=&#34;https://mobilewalla-anovos.s3.amazonaws.com/anovos.png&#34; style=&#34;height:100px;display:flex;margin:auto;float:right&#34;&gt;&lt;/img&gt;&lt;/html&gt;&#39;), dp.Text(
&#34;# ML-Anovos Report&#34;)
def stats_args(path, func):
&#34;&#34;&#34;
Args:
path:
func:
Returns:
&#34;&#34;&#34;
output = {}
mainfunc_to_args = {&#39;biasedness_detection&#39;: [&#39;stats_mode&#39;],
&#39;IDness_detection&#39;: [&#39;stats_unique&#39;],
&#39;outlier_detection&#39;: [&#39;stats_unique&#39;],
&#39;correlation_matrix&#39;: [&#39;stats_unique&#39;],
&#39;nullColumns_detection&#39;: [&#39;stats_unique&#39;, &#39;stats_mode&#39;, &#39;stats_missing&#39;],
&#39;variable_clustering&#39;: [&#39;stats_unique&#39;, &#39;stats_mode&#39;]}
args_to_statsfunc = {&#39;stats_unique&#39;: &#39;measures_of_cardinality&#39;, &#39;stats_mode&#39;: &#39;measures_of_centralTendency&#39;,
&#39;stats_missing&#39;: &#39;measures_of_counts&#39;}
for arg in mainfunc_to_args.get(func, []):
output[arg] = {&#39;file_path&#39;: (ends_with(path) + args_to_statsfunc[arg] + &#34;.csv&#34;),
&#39;file_type&#39;: &#39;csv&#39;, &#39;file_configs&#39;: {&#39;header&#39;: True, &#39;inferSchema&#39;: True}}
return output
def anovos_basic_report(spark, idf, id_col=&#39;&#39;, label_col=&#39;&#39;, event_label=&#39;&#39;, output_path=&#39;.&#39;, local_or_emr=&#39;local&#39;,
print_impact=True):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
id_col: ID column (Default value = &#39;&#39;)
label_col: Label/Target column (Default value = &#39;&#39;)
event_label: Value of (positive) event (i.e label 1) (Default value = &#39;&#39;)
output_path: File Path for saving metrics and basic report (Default value = &#39;.&#39;)
local_or_emr: local&#34; (default), &#34;emr&#34;
&#34;emr&#34; if the files are read from or written in AWS s3
print_impact:
(Default value = True)
Returns:
&#34;&#34;&#34;
global num_cols
global cat_cols
SG_funcs = [global_summary, measures_of_counts, measures_of_centralTendency, measures_of_cardinality,
measures_of_dispersion, measures_of_percentiles, measures_of_shape]
QC_rows_funcs = [duplicate_detection, nullRows_detection]
QC_cols_funcs = [nullColumns_detection, outlier_detection, IDness_detection, biasedness_detection,
invalidEntries_detection]
AA_funcs = [correlation_matrix, variable_clustering]
AT_funcs = [IV_calculation, IG_calculation]
all_funcs = SG_funcs + QC_rows_funcs + QC_cols_funcs + AA_funcs + AT_funcs
if local_or_emr == &#34;local&#34;:
local_path = output_path
else:
local_path = &#34;report_stats&#34;
Path(local_path).mkdir(parents=True, exist_ok=True)
for func in all_funcs:
if func in SG_funcs:
stats = func(spark, idf)
elif func in (QC_rows_funcs + QC_cols_funcs):
extra_args = stats_args(output_path, func.__name__)
stats = func(spark, idf, **extra_args)[1]
elif func in AA_funcs:
extra_args = stats_args(output_path, func.__name__)
stats = func(spark, idf, drop_cols=id_col, **extra_args)
elif label_col:
if func in AT_funcs:
stats = func(spark, idf, label_col=label_col, event_label=event_label)
else:
continue
stats.toPandas().to_csv(ends_with(local_path) + func.__name__ + &#34;.csv&#34;, index=False)
if local_or_emr == &#39;emr&#39;:
bash_cmd = &#34;aws s3 cp &#34; + ends_with(local_path) + func.__name__ + &#34;.csv &#34; + ends_with(output_path)
output = subprocess.check_output([&#39;bash&#39;, &#39;-c&#39;, bash_cmd])
if print_impact:
print(func.__name__, &#34;:\n&#34;)
stats = spark.read.csv(ends_with(output_path) + func.__name__ + &#34;.csv&#34;, header=True, inferSchema=True)
stats.show()
def remove_u_score(col):
&#34;&#34;&#34;
Args:
col:
Returns:
&#34;&#34;&#34;
col_ = col.split(&#34;_&#34;)
bl = []
for i in col_:
if i == &#34;nullColumns&#34; or i == &#34;nullRows&#34;:
bl.append(&#34;Null&#34;)
else:
bl.append(i[0].upper() + i[1:])
return &#34; &#34;.join(bl)
global_summary_df = pd.read_csv(ends_with(local_path) + &#34;global_summary.csv&#34;)
rows_count = int(global_summary_df[global_summary_df.metric.values == &#34;rows_count&#34;].value.values[0])
catcols_count = int(global_summary_df[global_summary_df.metric.values == &#34;catcols_count&#34;].value.values[0])
numcols_count = int(global_summary_df[global_summary_df.metric.values == &#34;numcols_count&#34;].value.values[0])
columns_count = int(global_summary_df[global_summary_df.metric.values == &#34;columns_count&#34;].value.values[0])
catcols_name = &#34;,&#34;.join(list(global_summary_df[global_summary_df.metric.values == &#34;catcols_name&#34;].value.values))
numcols_name = &#34;,&#34;.join(list(global_summary_df[global_summary_df.metric.values == &#34;numcols_name&#34;].value.values))
l1 = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(&#34;*This section summarizes the dataset with key statistical metrics.*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Global Summary&#34;), \
dp.Group(dp.Text(&#34; Total Number of Records: **&#34; + str(f&#34;{rows_count:,d}&#34;) + &#34;**&#34;),
dp.Text(&#34; Total Number of Attributes: **&#34; + str(columns_count) + &#34;**&#34;),
dp.Text(&#34; Number of Numerical Attributes : **&#34; + str(numcols_count) + &#34;**&#34;),
dp.Text(&#34; Numerical Attributes Name : **&#34; + str(numcols_name) + &#34;**&#34;),
dp.Text(&#34; Number of Categorical Attributes : **&#34; + str(catcols_count) + &#34;**&#34;),
dp.Text(&#34; Categorical Attributes Name : **&#34; + str(catcols_name) + &#34;**&#34;), rows=6), rows=8)
l2 = dp.Text(&#34;### Statistics by Metric Type&#34;)
SG_content = []
for i in SG_funcs:
if i.__name__ != &#39;global_summary&#39;:
SG_content.append(dp.DataTable(pd.read_csv(ends_with(local_path) + str(i.__name__) + &#34;.csv&#34;).round(3),
label=remove_u_score(i.__name__)))
l3 = dp.Group(dp.Select(blocks=SG_content, type=dp.SelectType.TABS), dp.Text(&#34;# &#34;))
tab1 = dp.Group(l1, dp.Text(&#34;# &#34;), l2, l3, dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;),
label=&#34;Descriptive Statistics&#34;)
QCcol_content = []
for i in QC_cols_funcs:
QCcol_content.append([dp.Text(&#34;### &#34; + str(remove_u_score(i.__name__))),
dp.DataTable(pd.read_csv(ends_with(local_path) + str(i.__name__) + &#34;.csv&#34;).round(3)),
dp.Text(&#34;#&#34;), dp.Text(&#34;#&#34;)])
QCrow_content = []
for i in QC_rows_funcs:
if i.__name__ == &#34;duplicate_detection&#34;:
stats = pd.read_csv(ends_with(local_path) + str(i.__name__) + &#34;.csv&#34;).round(3)
unique_rows_count = &#34; No. Of Unique Rows: **&#34; + str(
format(int(stats[stats[&#34;metric&#34;] == &#34;unique_rows_count&#34;].value.values), &#34;,&#34;)) + &#34;**&#34;
total_rows_count = &#34; No. of Rows: **&#34; + str(
format(int(stats[stats[&#34;metric&#34;] == &#34;rows_count&#34;].value.values), &#34;,&#34;)) + &#34;**&#34;
duplicate_rows_count = &#34; No. of Duplicate Rows: **&#34; + str(
format(int(stats[stats[&#34;metric&#34;] == &#34;duplicate_rows&#34;].value.values), &#34;,&#34;)) + &#34;**&#34;
duplicate_rows_pct = &#34; Percentage of Duplicate Rows: **&#34; + str(
float(stats[stats[&#34;metric&#34;] == &#34;duplicate_pct&#34;].value.values * 100.0)) + &#34; %&#34; + &#34;**&#34;
QCrow_content.append([dp.Text(&#34;### &#34; + str(remove_u_score(i.__name__))),
dp.Group(dp.Text(total_rows_count), dp.Text(unique_rows_count), dp.Text(duplicate_rows_count),
dp.Text(duplicate_rows_pct), rows=4), dp.Text(&#34;#&#34;),dp.Text(&#34;#&#34;)])
else:
QCrow_content.append([dp.Text(&#34;### &#34; + str(remove_u_score(i.__name__))),
dp.DataTable(pd.read_csv(ends_with(local_path) + str(i.__name__) + &#34;.csv&#34;).round(3)),
dp.Text(&#34;#&#34;), dp.Text(&#34;#&#34;)])
QCcol_content = [item for sublist in QCcol_content for item in sublist]
QCrow_content = [item for sublist in QCrow_content for item in sublist]
tab2 = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(&#34;*This section identifies the data quality issues at both row and column level.*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Select(blocks=[
dp.Group(dp.Text(&#34;# &#34;), dp.Group(*QCcol_content), rows=2, label=&#34;Column Level&#34;), \
dp.Group(dp.Text(&#34;# &#34;), dp.Group(*QCrow_content), rows=2, label=&#34;Row Level&#34;)], \
type=dp.SelectType.TABS), \
dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), \
label=&#34;Quality Check&#34;)
AA_content = []
for i in (AA_funcs + AT_funcs):
if i.__name__ == &#34;correlation_matrix&#34;:
stats = pd.read_csv(ends_with(local_path) + str(i.__name__) + &#34;.csv&#34;).round(3)
feats_order = list(stats[&#34;attribute&#34;].values)
stats = stats.round(3)
fig = px.imshow(stats[feats_order], y=feats_order, color_continuous_scale=global_theme, aspect=&#34;auto&#34;)
fig.layout.plot_bgcolor = global_plot_bg_color
fig.layout.paper_bgcolor = global_paper_bg_color
AA_content.append(
dp.Group(dp.Text(&#34;##&#34;), dp.DataTable(stats[[&#34;attribute&#34;] + feats_order]), dp.Plot(fig), rows=3,
label=remove_u_score(i.__name__)))
elif i.__name__ == &#34;variable_clustering&#34;:
stats = pd.read_csv(ends_with(local_path) + str(i.__name__) + &#34;.csv&#34;).round(3).sort_values(by=[&#39;Cluster&#39;],
ascending=True)
fig = px.sunburst(stats, path=[&#39;Cluster&#39;, &#39;Attribute&#39;], values=&#39;RS_Ratio&#39;,
color_discrete_sequence=global_theme)
fig.layout.plot_bgcolor = global_plot_bg_color
fig.layout.paper_bgcolor = global_paper_bg_color
fig.layout.autosize = True
AA_content.append(
dp.Group(dp.Text(&#34;##&#34;), dp.DataTable(stats), dp.Plot(fig), rows=3, label=remove_u_score(i.__name__)))
else:
if label_col:
stats = pd.read_csv(ends_with(local_path) + str(i.__name__) + &#34;.csv&#34;).round(3)
col_nm = [x for x in list(stats.columns) if &#34;attribute&#34; not in x]
stats = stats.sort_values(col_nm[0], ascending=True)
fig = px.bar(stats, x=col_nm[0], y=&#39;attribute&#39;, orientation=&#39;h&#39;, color_discrete_sequence=global_theme)
fig.layout.plot_bgcolor = global_plot_bg_color
fig.layout.paper_bgcolor = global_paper_bg_color
fig.layout.autosize = True
AA_content.append(
dp.Group(dp.Text(&#34;##&#34;), dp.DataTable(stats), dp.Plot(fig), label=remove_u_score(i.__name__),
rows=3))
tab3 = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section analyzes the interaction between different attributes and/or the relationship between an attribute &amp; the binary target variable.*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Association Matrix &amp; Plot&#34;), \
dp.Select(blocks=AA_content, type=dp.SelectType.DROPDOWN), \
dp.Text(&#34;### &#34;), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;## &#34;), \
label=&#34;Attribute Associations&#34;)
basic_report = dp.Report(default_template[0], default_template[1], \
dp.Select(blocks=[tab1, tab2, tab3], type=dp.SelectType.TABS)) \
.save(ends_with(local_path) + &#34;basic_report.html&#34;, open=True)
if local_or_emr == &#39;emr&#39;:
bash_cmd = &#34;aws s3 cp &#34; + ends_with(local_path) + &#34;basic_report.html &#34; + ends_with(output_path)
output = subprocess.check_output([&#39;bash&#39;, &#39;-c&#39;, bash_cmd])
```
</details>
## Functions
<dl>
<dt id="anovos.data_report.basic_report_generation.anovos_basic_report"><code class="name flex">
<span>def <span class="ident">anovos_basic_report</span></span>(<span>spark, idf, id_col='', label_col='', event_label='', output_path='.', local_or_emr='local', print_impact=True)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>spark</code></strong></dt>
<dd>Spark Session</dd>
<dt><strong><code>idf</code></strong></dt>
<dd>Input Dataframe</dd>
<dt><strong><code>id_col</code></strong></dt>
<dd>ID column (Default value = '')</dd>
<dt><strong><code>label_col</code></strong></dt>
<dd>Label/Target column (Default value = '')</dd>
<dt><strong><code>event_label</code></strong></dt>
<dd>Value of (positive) event (i.e label 1) (Default value = '')</dd>
<dt><strong><code>output_path</code></strong></dt>
<dd>File Path for saving metrics and basic report (Default value = '.')</dd>
<dt><strong><code>local_or_emr</code></strong></dt>
<dd>local" (default), "emr"</dd>
</dl>
<p>"emr" if the files are read from or written in AWS s3
print_impact:
(Default value = True)</p>
<p>Returns:</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def anovos_basic_report(spark, idf, id_col=&#39;&#39;, label_col=&#39;&#39;, event_label=&#39;&#39;, output_path=&#39;.&#39;, local_or_emr=&#39;local&#39;,
print_impact=True):
&#34;&#34;&#34;
Args:
spark: Spark Session
idf: Input Dataframe
id_col: ID column (Default value = &#39;&#39;)
label_col: Label/Target column (Default value = &#39;&#39;)
event_label: Value of (positive) event (i.e label 1) (Default value = &#39;&#39;)
output_path: File Path for saving metrics and basic report (Default value = &#39;.&#39;)
local_or_emr: local&#34; (default), &#34;emr&#34;
&#34;emr&#34; if the files are read from or written in AWS s3
print_impact:
(Default value = True)
Returns:
&#34;&#34;&#34;
global num_cols
global cat_cols
SG_funcs = [global_summary, measures_of_counts, measures_of_centralTendency, measures_of_cardinality,
measures_of_dispersion, measures_of_percentiles, measures_of_shape]
QC_rows_funcs = [duplicate_detection, nullRows_detection]
QC_cols_funcs = [nullColumns_detection, outlier_detection, IDness_detection, biasedness_detection,
invalidEntries_detection]
AA_funcs = [correlation_matrix, variable_clustering]
AT_funcs = [IV_calculation, IG_calculation]
all_funcs = SG_funcs + QC_rows_funcs + QC_cols_funcs + AA_funcs + AT_funcs
if local_or_emr == &#34;local&#34;:
local_path = output_path
else:
local_path = &#34;report_stats&#34;
Path(local_path).mkdir(parents=True, exist_ok=True)
for func in all_funcs:
if func in SG_funcs:
stats = func(spark, idf)
elif func in (QC_rows_funcs + QC_cols_funcs):
extra_args = stats_args(output_path, func.__name__)
stats = func(spark, idf, **extra_args)[1]
elif func in AA_funcs:
extra_args = stats_args(output_path, func.__name__)
stats = func(spark, idf, drop_cols=id_col, **extra_args)
elif label_col:
if func in AT_funcs:
stats = func(spark, idf, label_col=label_col, event_label=event_label)
else:
continue
stats.toPandas().to_csv(ends_with(local_path) + func.__name__ + &#34;.csv&#34;, index=False)
if local_or_emr == &#39;emr&#39;:
bash_cmd = &#34;aws s3 cp &#34; + ends_with(local_path) + func.__name__ + &#34;.csv &#34; + ends_with(output_path)
output = subprocess.check_output([&#39;bash&#39;, &#39;-c&#39;, bash_cmd])
if print_impact:
print(func.__name__, &#34;:\n&#34;)
stats = spark.read.csv(ends_with(output_path) + func.__name__ + &#34;.csv&#34;, header=True, inferSchema=True)
stats.show()
def remove_u_score(col):
&#34;&#34;&#34;
Args:
col:
Returns:
&#34;&#34;&#34;
col_ = col.split(&#34;_&#34;)
bl = []
for i in col_:
if i == &#34;nullColumns&#34; or i == &#34;nullRows&#34;:
bl.append(&#34;Null&#34;)
else:
bl.append(i[0].upper() + i[1:])
return &#34; &#34;.join(bl)
global_summary_df = pd.read_csv(ends_with(local_path) + &#34;global_summary.csv&#34;)
rows_count = int(global_summary_df[global_summary_df.metric.values == &#34;rows_count&#34;].value.values[0])
catcols_count = int(global_summary_df[global_summary_df.metric.values == &#34;catcols_count&#34;].value.values[0])
numcols_count = int(global_summary_df[global_summary_df.metric.values == &#34;numcols_count&#34;].value.values[0])
columns_count = int(global_summary_df[global_summary_df.metric.values == &#34;columns_count&#34;].value.values[0])
catcols_name = &#34;,&#34;.join(list(global_summary_df[global_summary_df.metric.values == &#34;catcols_name&#34;].value.values))
numcols_name = &#34;,&#34;.join(list(global_summary_df[global_summary_df.metric.values == &#34;numcols_name&#34;].value.values))
l1 = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(&#34;*This section summarizes the dataset with key statistical metrics.*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Global Summary&#34;), \
dp.Group(dp.Text(&#34; Total Number of Records: **&#34; + str(f&#34;{rows_count:,d}&#34;) + &#34;**&#34;),
dp.Text(&#34; Total Number of Attributes: **&#34; + str(columns_count) + &#34;**&#34;),
dp.Text(&#34; Number of Numerical Attributes : **&#34; + str(numcols_count) + &#34;**&#34;),
dp.Text(&#34; Numerical Attributes Name : **&#34; + str(numcols_name) + &#34;**&#34;),
dp.Text(&#34; Number of Categorical Attributes : **&#34; + str(catcols_count) + &#34;**&#34;),
dp.Text(&#34; Categorical Attributes Name : **&#34; + str(catcols_name) + &#34;**&#34;), rows=6), rows=8)
l2 = dp.Text(&#34;### Statistics by Metric Type&#34;)
SG_content = []
for i in SG_funcs:
if i.__name__ != &#39;global_summary&#39;:
SG_content.append(dp.DataTable(pd.read_csv(ends_with(local_path) + str(i.__name__) + &#34;.csv&#34;).round(3),
label=remove_u_score(i.__name__)))
l3 = dp.Group(dp.Select(blocks=SG_content, type=dp.SelectType.TABS), dp.Text(&#34;# &#34;))
tab1 = dp.Group(l1, dp.Text(&#34;# &#34;), l2, l3, dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;),
label=&#34;Descriptive Statistics&#34;)
QCcol_content = []
for i in QC_cols_funcs:
QCcol_content.append([dp.Text(&#34;### &#34; + str(remove_u_score(i.__name__))),
dp.DataTable(pd.read_csv(ends_with(local_path) + str(i.__name__) + &#34;.csv&#34;).round(3)),
dp.Text(&#34;#&#34;), dp.Text(&#34;#&#34;)])
QCrow_content = []
for i in QC_rows_funcs:
if i.__name__ == &#34;duplicate_detection&#34;:
stats = pd.read_csv(ends_with(local_path) + str(i.__name__) + &#34;.csv&#34;).round(3)
unique_rows_count = &#34; No. Of Unique Rows: **&#34; + str(
format(int(stats[stats[&#34;metric&#34;] == &#34;unique_rows_count&#34;].value.values), &#34;,&#34;)) + &#34;**&#34;
total_rows_count = &#34; No. of Rows: **&#34; + str(
format(int(stats[stats[&#34;metric&#34;] == &#34;rows_count&#34;].value.values), &#34;,&#34;)) + &#34;**&#34;
duplicate_rows_count = &#34; No. of Duplicate Rows: **&#34; + str(
format(int(stats[stats[&#34;metric&#34;] == &#34;duplicate_rows&#34;].value.values), &#34;,&#34;)) + &#34;**&#34;
duplicate_rows_pct = &#34; Percentage of Duplicate Rows: **&#34; + str(
float(stats[stats[&#34;metric&#34;] == &#34;duplicate_pct&#34;].value.values * 100.0)) + &#34; %&#34; + &#34;**&#34;
QCrow_content.append([dp.Text(&#34;### &#34; + str(remove_u_score(i.__name__))),
dp.Group(dp.Text(total_rows_count), dp.Text(unique_rows_count), dp.Text(duplicate_rows_count),
dp.Text(duplicate_rows_pct), rows=4), dp.Text(&#34;#&#34;),dp.Text(&#34;#&#34;)])
else:
QCrow_content.append([dp.Text(&#34;### &#34; + str(remove_u_score(i.__name__))),
dp.DataTable(pd.read_csv(ends_with(local_path) + str(i.__name__) + &#34;.csv&#34;).round(3)),
dp.Text(&#34;#&#34;), dp.Text(&#34;#&#34;)])
QCcol_content = [item for sublist in QCcol_content for item in sublist]
QCrow_content = [item for sublist in QCrow_content for item in sublist]
tab2 = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(&#34;*This section identifies the data quality issues at both row and column level.*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Select(blocks=[
dp.Group(dp.Text(&#34;# &#34;), dp.Group(*QCcol_content), rows=2, label=&#34;Column Level&#34;), \
dp.Group(dp.Text(&#34;# &#34;), dp.Group(*QCrow_content), rows=2, label=&#34;Row Level&#34;)], \
type=dp.SelectType.TABS), \
dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), \
label=&#34;Quality Check&#34;)
AA_content = []
for i in (AA_funcs + AT_funcs):
if i.__name__ == &#34;correlation_matrix&#34;:
stats = pd.read_csv(ends_with(local_path) + str(i.__name__) + &#34;.csv&#34;).round(3)
feats_order = list(stats[&#34;attribute&#34;].values)
stats = stats.round(3)
fig = px.imshow(stats[feats_order], y=feats_order, color_continuous_scale=global_theme, aspect=&#34;auto&#34;)
fig.layout.plot_bgcolor = global_plot_bg_color
fig.layout.paper_bgcolor = global_paper_bg_color
AA_content.append(
dp.Group(dp.Text(&#34;##&#34;), dp.DataTable(stats[[&#34;attribute&#34;] + feats_order]), dp.Plot(fig), rows=3,
label=remove_u_score(i.__name__)))
elif i.__name__ == &#34;variable_clustering&#34;:
stats = pd.read_csv(ends_with(local_path) + str(i.__name__) + &#34;.csv&#34;).round(3).sort_values(by=[&#39;Cluster&#39;],
ascending=True)
fig = px.sunburst(stats, path=[&#39;Cluster&#39;, &#39;Attribute&#39;], values=&#39;RS_Ratio&#39;,
color_discrete_sequence=global_theme)
fig.layout.plot_bgcolor = global_plot_bg_color
fig.layout.paper_bgcolor = global_paper_bg_color
fig.layout.autosize = True
AA_content.append(
dp.Group(dp.Text(&#34;##&#34;), dp.DataTable(stats), dp.Plot(fig), rows=3, label=remove_u_score(i.__name__)))
else:
if label_col:
stats = pd.read_csv(ends_with(local_path) + str(i.__name__) + &#34;.csv&#34;).round(3)
col_nm = [x for x in list(stats.columns) if &#34;attribute&#34; not in x]
stats = stats.sort_values(col_nm[0], ascending=True)
fig = px.bar(stats, x=col_nm[0], y=&#39;attribute&#39;, orientation=&#39;h&#39;, color_discrete_sequence=global_theme)
fig.layout.plot_bgcolor = global_plot_bg_color
fig.layout.paper_bgcolor = global_paper_bg_color
fig.layout.autosize = True
AA_content.append(
dp.Group(dp.Text(&#34;##&#34;), dp.DataTable(stats), dp.Plot(fig), label=remove_u_score(i.__name__),
rows=3))
tab3 = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section analyzes the interaction between different attributes and/or the relationship between an attribute &amp; the binary target variable.*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Association Matrix &amp; Plot&#34;), \
dp.Select(blocks=AA_content, type=dp.SelectType.DROPDOWN), \
dp.Text(&#34;### &#34;), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;## &#34;), \
label=&#34;Attribute Associations&#34;)
basic_report = dp.Report(default_template[0], default_template[1], \
dp.Select(blocks=[tab1, tab2, tab3], type=dp.SelectType.TABS)) \
.save(ends_with(local_path) + &#34;basic_report.html&#34;, open=True)
if local_or_emr == &#39;emr&#39;:
bash_cmd = &#34;aws s3 cp &#34; + ends_with(local_path) + &#34;basic_report.html &#34; + ends_with(output_path)
output = subprocess.check_output([&#39;bash&#39;, &#39;-c&#39;, bash_cmd])
```
</details>
</dd>
<dt id="anovos.data_report.basic_report_generation.stats_args"><code class="name flex">
<span>def <span class="ident">stats_args</span></span>(<span>path, func)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>path</code></strong></dt>
<dd>&nbsp;</dd>
<dt><strong><code>func</code></strong></dt>
<dd>&nbsp;</dd>
</dl>
<p>Returns:</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def stats_args(path, func):
&#34;&#34;&#34;
Args:
path:
func:
Returns:
&#34;&#34;&#34;
output = {}
mainfunc_to_args = {&#39;biasedness_detection&#39;: [&#39;stats_mode&#39;],
&#39;IDness_detection&#39;: [&#39;stats_unique&#39;],
&#39;outlier_detection&#39;: [&#39;stats_unique&#39;],
&#39;correlation_matrix&#39;: [&#39;stats_unique&#39;],
&#39;nullColumns_detection&#39;: [&#39;stats_unique&#39;, &#39;stats_mode&#39;, &#39;stats_missing&#39;],
&#39;variable_clustering&#39;: [&#39;stats_unique&#39;, &#39;stats_mode&#39;]}
args_to_statsfunc = {&#39;stats_unique&#39;: &#39;measures_of_cardinality&#39;, &#39;stats_mode&#39;: &#39;measures_of_centralTendency&#39;,
&#39;stats_missing&#39;: &#39;measures_of_counts&#39;}
for arg in mainfunc_to_args.get(func, []):
output[arg] = {&#39;file_path&#39;: (ends_with(path) + args_to_statsfunc[arg] + &#34;.csv&#34;),
&#39;file_type&#39;: &#39;csv&#39;, &#39;file_configs&#39;: {&#39;header&#39;: True, &#39;inferSchema&#39;: True}}
return output
```
</details>
</dd>
</dl>