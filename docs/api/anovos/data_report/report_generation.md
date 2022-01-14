# Module <code>report_generation</code>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
import json
import os
import subprocess
import datapane as dp
import numpy as np
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
from anovos.shared.utils import ends_with
global_theme = px.colors.sequential.Plasma
global_theme_r = px.colors.sequential.Plasma_r
global_plot_bg_color = &#39;rgba(0,0,0,0)&#39;
global_paper_bg_color = &#39;rgba(0,0,0,0)&#39;
default_template = dp.HTML(
&#39;&lt;html&gt;&lt;img src=&#34;https://mobilewalla-anovos.s3.amazonaws.com/anovos.png&#34; style=&#34;height:100px;display:flex;margin:auto;float:right&#34;&gt;&lt;/img&gt;&lt;/html&gt;&#39;), dp.Text(
&#34;# ML-Anovos Report&#34;)
def remove_u_score(col):
&#34;&#34;&#34;
Args:
col: Analysis column containing &#34;_&#34; present gets replaced along with upper case conversion
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
def line_chart_gen_stability(df1, df2, col):
&#34;&#34;&#34;
Args:
df1: Analysis dataframe pertaining to summarized stability metrics
df2: Analysis dataframe pertaining to historical data
col: Analysis column
Returns:
&#34;&#34;&#34;
def val_cat(val):
&#34;&#34;&#34;
Args:
val:
Returns:
&#34;&#34;&#34;
if val &gt;= 3.5:
return &#34;Very Stable&#34;
elif val &gt;= 3 and val &lt; 3.5:
return &#34;Stable&#34;
elif val &gt;= 2 and val &lt; 3:
return &#34;Marginally Stable&#34;
elif val &gt;= 1 and val &lt; 2:
return &#34;Unstable&#34;
elif val &gt;= 0 and val &lt; 1:
return &#34;Very Unstable&#34;
else:
return &#34;Out of Range&#34;
val_si = list(df2[df2[&#34;attribute&#34;] == col].stability_index.values)[0]
f1 = go.Figure()
f1.add_trace(go.Indicator(
mode=&#34;gauge+number&#34;,
value=val_si,
gauge={
&#39;axis&#39;: {&#39;range&#39;: [None, 4], &#39;tickwidth&#39;: 1, &#39;tickcolor&#39;: &#34;black&#34;},
&#39;bgcolor&#39;: &#34;white&#34;,
&#39;steps&#39;: [
{&#39;range&#39;: [0, 1], &#39;color&#39;: px.colors.sequential.Reds[7]},
{&#39;range&#39;: [1, 2], &#39;color&#39;: px.colors.sequential.Reds[6]},
{&#39;range&#39;: [2, 3], &#39;color&#39;: px.colors.sequential.Oranges[4]},
{&#39;range&#39;: [3, 3.5], &#39;color&#39;: px.colors.sequential.BuGn[7]},
{&#39;range&#39;: [3.5, 4], &#39;color&#39;: px.colors.sequential.BuGn[8]}],
&#39;threshold&#39;: {
&#39;line&#39;: {&#39;color&#39;: &#34;black&#34;, &#39;width&#39;: 3},
&#39;thickness&#39;: 1,
&#39;value&#39;: val_si},
&#39;bar&#39;: {&#39;color&#39;: global_plot_bg_color}},
title={&#39;text&#39;: &#34;Order of Stability: &#34; + val_cat(val_si)}))
f1.update_layout(height=400, font={&#39;color&#39;: &#34;black&#34;, &#39;family&#39;: &#34;Arial&#34;})
f5 = &#34;Stability Index for &#34; + str(col.upper())
if len(df1.columns) &gt; 0:
df1 = df1[df1[&#34;attribute&#34;] == col]
f2 = px.line(df1, x=&#39;idx&#39;, y=&#39;mean&#39;, markers=True,
title=&#39;CV of Mean is &#39; + str(list(df2[df2[&#34;attribute&#34;] == col].mean_cv.values)[0]))
f2.update_traces(line_color=global_theme[2], marker=dict(size=14))
f2.layout.plot_bgcolor = global_plot_bg_color
f2.layout.paper_bgcolor = global_paper_bg_color
f3 = px.line(df1, x=&#39;idx&#39;, y=&#39;stddev&#39;, markers=True,
title=&#39;CV of Stddev is &#39; + str(list(df2[df2[&#34;attribute&#34;] == col].stddev_cv.values)[0]))
f3.update_traces(line_color=global_theme[6], marker=dict(size=14))
f3.layout.plot_bgcolor = global_plot_bg_color
f3.layout.paper_bgcolor = global_paper_bg_color
f4 = px.line(df1, x=&#39;idx&#39;, y=&#39;kurtosis&#39;, markers=True,
title=&#39;CV of Kurtosis is &#39; + str(list(df2[df2[&#34;attribute&#34;] == col].kurtosis_cv.values)[0]))
f4.update_traces(line_color=global_theme[4], marker=dict(size=14))
f4.layout.plot_bgcolor = global_plot_bg_color
f4.layout.paper_bgcolor = global_paper_bg_color
return dp.Group(dp.Text(&#34;#&#34;), dp.Text(f5), dp.Plot(f1),
dp.Group(dp.Plot(f2), dp.Plot(f3), dp.Plot(f4), columns=3), rows=4, label=col)
else:
return dp.Group(dp.Text(&#34;#&#34;), dp.Text(f5), dp.Plot(f1), rows=3, label=col)
def data_analyzer_output(master_path, avl_recs_tab, tab_name):
&#34;&#34;&#34;
Args:
master_path: Path containing all the output from analyzed data
avl_recs_tab: Available file names from the analysis tab
tab_name: Analysis tab from association_evaluator / quality_checker / stats_generator
Returns:
&#34;&#34;&#34;
df_list = []
txt_list = []
plot_list = []
df_plot_list = []
avl_recs_tab = [x for x in avl_recs_tab if &#34;global_summary&#34; not in x]
for index, i in enumerate(avl_recs_tab):
data = pd.read_csv(ends_with(master_path) + str(i) + &#34;.csv&#34;)
if len(data.index) == 0:
continue
if tab_name == &#34;quality_checker&#34;:
if i == &#34;duplicate_detection&#34;:
duplicate_recs = pd.read_csv(ends_with(master_path) + str(i) + &#34;.csv&#34;).round(3)
unique_rows_count = &#34; No. Of Unique Rows: **&#34; + str(
format(int(duplicate_recs[duplicate_recs[&#34;metric&#34;] == &#34;unique_rows_count&#34;].value.values),
&#34;,&#34;)) + &#34;**&#34;
rows_count = &#34; No. of Rows: **&#34; + str(
format(int(duplicate_recs[duplicate_recs[&#34;metric&#34;] == &#34;rows_count&#34;].value.values), &#34;,&#34;)) + &#34;**&#34;
duplicate_rows = &#34; No. of Duplicate Rows: **&#34; + str(
format(int(duplicate_recs[duplicate_recs[&#34;metric&#34;] == &#34;duplicate_rows&#34;].value.values), &#34;,&#34;)) + &#34;**&#34;
duplicate_pct = &#34; Percentage of Duplicate Rows: **&#34; + str(
float(duplicate_recs[duplicate_recs[&#34;metric&#34;] == &#34;duplicate_pct&#34;].value.values * 100.0)) + &#34; %&#34; + &#34;**&#34;
df_list.append([dp.Text(&#34;### &#34; + str(remove_u_score(i))),
dp.Group(dp.Text(rows_count), dp.Text(unique_rows_count), dp.Text(duplicate_rows),
dp.Text(duplicate_pct), rows=4), dp.Text(&#34;#&#34;), dp.Text(&#34;#&#34;)])
elif i == &#34;outlier_detection&#34;:
df_list.append([dp.Text(&#34;### &#34; + str(remove_u_score(i))),
dp.DataTable(pd.read_csv(ends_with(master_path) + str(i) + &#34;.csv&#34;).round(3)),
&#34;outlier_charts_placeholder&#34;])
else:
df_list.append([dp.Text(&#34;### &#34; + str(remove_u_score(i))),
dp.DataTable(pd.read_csv(ends_with(master_path) + str(i) + &#34;.csv&#34;).round(3)),
dp.Text(&#34;#&#34;), dp.Text(&#34;#&#34;)])
elif tab_name == &#34;association_evaluator&#34;:
for j in avl_recs_tab:
if j == &#34;correlation_matrix&#34;:
df_list_ = pd.read_csv(ends_with(master_path) + str(j) + &#34;.csv&#34;).round(3)
feats_order = list(df_list_[&#34;attribute&#34;].values)
df_list_ = df_list_.round(3)
fig = px.imshow(df_list_[feats_order], y=feats_order, color_continuous_scale=global_theme,
aspect=&#34;auto&#34;)
fig.layout.plot_bgcolor = global_plot_bg_color
fig.layout.paper_bgcolor = global_paper_bg_color
#
fig.update_layout(title_text=str(&#34;Correlation Plot &#34;))
df_plot_list.append(
dp.Group(dp.Text(&#34;##&#34;), dp.DataTable(df_list_[[&#34;attribute&#34;] + feats_order]), dp.Plot(fig),
rows=3, label=remove_u_score(j)))
elif j == &#34;variable_clustering&#34;:
df_list_ = pd.read_csv(ends_with(master_path) + str(j) + &#34;.csv&#34;).round(3).sort_values(
by=[&#39;Cluster&#39;], ascending=True)
fig = px.sunburst(df_list_, path=[&#39;Cluster&#39;, &#39;Attribute&#39;], values=&#39;RS_Ratio&#39;,
color_discrete_sequence=global_theme)
#
fig.update_layout(title_text=str(&#34;Distribution of homogenous variable across Clusters&#34;))
fig.layout.plot_bgcolor = global_plot_bg_color
fig.layout.paper_bgcolor = global_paper_bg_color
#
fig.update_layout(title_text=str(&#34;Variable Clustering Plot &#34;))
fig.layout.autosize = True
df_plot_list.append(
dp.Group(dp.Text(&#34;##&#34;), dp.DataTable(df_list_), dp.Plot(fig), rows=3, label=remove_u_score(j)))
else:
try:
df_list_ = pd.read_csv(ends_with(master_path) + str(j) + &#34;.csv&#34;).round(3)
col_nm = [x for x in list(df_list_.columns) if &#34;attribute&#34; not in x]
df_list_ = df_list_.sort_values(col_nm[0], ascending=True)
fig = px.bar(df_list_, x=col_nm[0], y=&#39;attribute&#39;, orientation=&#39;h&#39;,
color_discrete_sequence=global_theme)
fig.layout.plot_bgcolor = global_plot_bg_color
fig.layout.paper_bgcolor = global_paper_bg_color
#
fig.update_layout(title_text=str(&#34;Representation of &#34; + str(remove_u_score(j))))
fig.layout.autosize = True
df_plot_list.append(
dp.Group(dp.Text(&#34;##&#34;), dp.DataTable(df_list_), dp.Plot(fig), label=remove_u_score(j),
rows=3))
except:
pass
if len(avl_recs_tab) == 1:
df_plot_list.append(dp.Group(dp.DataTable(pd.DataFrame(columns=[&#39; &#39;], index=range(1)), label=&#34; &#34;),
dp.Plot(blank_chart, label=&#34; &#34;), label=&#34; &#34;))
else:
pass
return df_plot_list
else:
df_list.append(dp.DataTable(pd.read_csv(ends_with(master_path) + str(i) + &#34;.csv&#34;).round(3),
label=remove_u_score(avl_recs_tab[index])))
if tab_name == &#34;quality_checker&#34; and len(avl_recs_tab) == 1:
return df_list[0], [dp.Text(&#34;#&#34;), dp.Plot(blank_chart)]
elif tab_name == &#34;stats_generator&#34; and len(avl_recs_tab) == 1:
return [df_list[0], dp.DataTable(pd.DataFrame(columns=[&#39; &#39;], index=range(1)), label=&#34; &#34;)]
else:
return df_list
def drift_stability_ind(missing_recs_drift, drift_tab, missing_recs_stability, stability_tab):
&#34;&#34;&#34;missing_recs_drift: Missing files from the drift tab
drift_tab: &#34;drift_statistics&#34;
missing_recs_stability: Missing files from the stability tab
stability_tab:&#34;stabilityIndex_computation, stabilityIndex_metrics&#34;
Args:
missing_recs_drift:
drift_tab:
missing_recs_stability:
stability_tab:
Returns:
&#34;&#34;&#34;
if len(missing_recs_drift) == len(drift_tab):
drift_ind = 0
else:
drift_ind = 1
if len(missing_recs_stability) == len(stability_tab):
stability_ind = 0
elif (&#34;stabilityIndex_metrics&#34; in missing_recs_stability) and (
&#34;stabilityIndex_computation&#34; not in missing_recs_stability):
stability_ind = 0.5
else:
stability_ind = 1
return drift_ind, stability_ind
def chart_gen_list(master_path, chart_type, type_col=None):
&#34;&#34;&#34;
Args:
master_path: Path containing all the charts same as the other files from data analyzed output
chart_type: Files containing only the specific chart names for the specific chart category
type_col: None. Default value is kept as None
Returns:
&#34;&#34;&#34;
plot_list = []
for i in chart_type:
col_name = i[i.find(&#34;_&#34;) + 1:]
if type_col == &#34;numerical&#34;:
if col_name in numcols_name.replace(&#34; &#34;, &#34;&#34;).split(&#34;,&#34;):
plot_list.append(dp.Plot(go.Figure(json.load(open(ends_with(master_path) + i))), label=col_name))
else:
pass
elif type_col == &#34;categorical&#34;:
if col_name in catcols_name.replace(&#34; &#34;, &#34;&#34;).split(&#34;,&#34;):
plot_list.append(dp.Plot(go.Figure(json.load(open(ends_with(master_path) + i))), label=col_name))
else:
pass
else:
plot_list.append(dp.Plot(go.Figure(json.load(open(ends_with(master_path) + i))), label=col_name))
return plot_list
def executive_summary_gen(master_path, label_col, ds_ind, id_col, iv_threshold, corr_threshold, print_report=False):
&#34;&#34;&#34;
Args:
master_path: Path containing the input files.
label_col: Label column.
ds_ind: Drift stability indicator in list form.
id_col: ID column.
iv_threshold: IV threshold beyond which attributes can be called as significant.
corr_threshold: Correlation threshold beyond which attributes can be categorized under correlated.
print_report: Printing option flexibility. Default value is kept as False.
Returns:
&#34;&#34;&#34;
try:
obj_dtls = json.load(open(ends_with(master_path) + &#34;freqDist_&#34; + str(label_col)))
text_val = list(list(obj_dtls.values())[0][0].items())[8][1]
x_val = list(list(obj_dtls.values())[0][0].items())[11][1]
y_val = list(list(obj_dtls.values())[0][0].items())[13][1]
label_fig_ = go.Figure(data=[go.Pie(labels=x_val, values=y_val, textinfo=&#39;label+percent&#39;,
insidetextorientation=&#39;radial&#39;, pull=[0, 0.1], marker_colors=global_theme)])
label_fig_.update_traces(textposition=&#39;inside&#39;, textinfo=&#39;percent+label&#39;)
label_fig_.update_layout(legend=dict(orientation=&#34;h&#34;, x=0.5, yanchor=&#34;bottom&#34;, xanchor=&#34;center&#34;))
label_fig_.layout.plot_bgcolor = global_plot_bg_color
label_fig_.layout.paper_bgcolor = global_paper_bg_color
except:
label_fig_ = None
a1 = &#34;The dataset contains
**&#34; + str(f&#34;{rows_count:,d}&#34;) + &#34;** records and **&#34; + str(
numcols_count + catcols_count) + &#34;** attributes (**&#34; + str(numcols_count) + &#34;** numerical + **&#34; + str(
catcols_count) + &#34;** categorical).&#34;
if label_col is None:
a2 = dp.Group(dp.Text(&#34;- There is **no** target variable in the dataset&#34;), dp.Text(&#34;- Data Diagnosis:&#34;), rows=2)
else:
if label_fig_ is None:
a2 = dp.Group(dp.Text(&#34;- Target variable is **&#34; + str(label_col) + &#34;** &#34;), dp.Text(&#34;- Data Diagnosis:&#34;),
rows=2)
else:
a2 = dp.Group(dp.Text(&#34;- Target variable is **&#34; + str(label_col) + &#34;** &#34;), dp.Plot(label_fig_),
dp.Text(&#34;- Data Diagnosis:&#34;), rows=3)
try:
x1 = list(pd.read_csv(ends_with(master_path) + &#34;measures_of_dispersion.csv&#34;).query(&#34;`cov`&gt;1&#34;).attribute.values)
if len(x1) &gt; 0:
x1_1 = [&#34;High Variance&#34;, x1]
else:
x1_1 = [&#34;High Variance&#34;, None]
except:
x1_1 = [&#34;High Variance&#34;, None]
try:
x2 = list(pd.read_csv(ends_with(master_path) + &#34;measures_of_shape.csv&#34;).query(&#34;`skewness`&gt;0&#34;).attribute.values)
if len(x2) &gt; 0:
x2_1 = [&#34;Positive Skewness&#34;, x2]
else:
x2_1 = [&#34;Positive Skewness&#34;, None]
except:
x2_1 = [&#34;Positive Skewness&#34;, None]
try:
x3 = list(pd.read_csv(ends_with(master_path) + &#34;measures_of_shape.csv&#34;).query(&#34;`skewness`&lt;0&#34;).attribute.values)
if len(x3) &gt; 0:
x3_1 = [&#34;Negative Skewness&#34;, x3]
else:
x3_1 = [&#34;Negative Skewness&#34;, None]
except:
x3_1 = [&#34;Negative Skewness&#34;, None]
try:
x4 = list(pd.read_csv(ends_with(master_path) + &#34;measures_of_shape.csv&#34;).query(&#34;`kurtosis`&gt;0&#34;).attribute.values)
if len(x4) &gt; 0:
x4_1 = [&#34;High Kurtosis&#34;, x4]
else:
x4_1 = [&#34;High Kurtosis&#34;, None]
except:
x4_1 = [&#34;High Kurtosis&#34;, None]
try:
x5 = list(pd.read_csv(ends_with(master_path) + &#34;measures_of_shape.csv&#34;).query(&#34;`kurtosis`&lt;0&#34;).attribute.values)
if len(x5) &gt; 0:
x5_1 = [&#34;Low Kurtosis&#34;, x5]
else:
x5_1 = [&#34;Low Kurtosis&#34;, None]
except:
x5_1 = [&#34;Low Kurtosis&#34;, None]
try:
x6 = list(
pd.read_csv(ends_with(master_path) + &#34;measures_of_counts.csv&#34;).query(&#34;`fill_pct`&lt;0.7&#34;).attribute.values)
if len(x6) &gt; 0:
x6_1 = [&#34;Low Fill Rates&#34;, x6]
else:
x6_1 = [&#34;Low Fill Rates&#34;, None]
except:
x6_1 = [&#34;Low Fill Rates&#34;, None]
try:
biasedness_df = pd.read_csv(ends_with(master_path) + &#34;biasedness_detection.csv&#34;)
if &#34;treated&#34; in biasedness_df:
x7 = list(df.query(&#34;`treated`&gt;0&#34;).attribute.values)
else:
x7 = list(df.query(&#34;`flagged`&gt;0&#34;).attribute.values)
if len(x7) &gt; 0:
x7_1 = [&#34;High Biasedness&#34;, x7]
else:
x7_1 = [&#34;High Biasedness&#34;, None]
except:
x7_1 = [&#34;High Biasedness&#34;, None]
try:
x8 = list(pd.read_csv(ends_with(master_path) + &#34;outlier_detection.csv&#34;).attribute.values)
if len(x8) &gt; 0:
x8_1 = [&#34;Outliers&#34;, x8]
else:
x8_1 = [&#34;Outliers&#34;, None]
except:
x8_1 = [&#34;Outliers&#34;, None]
try:
corr_matrx = pd.read_csv(ends_with(master_path) + &#34;correlation_matrix.csv&#34;)
corr_matrx = corr_matrx[list(corr_matrx.attribute.values)]
corr_matrx = corr_matrx.where(np.triu(np.ones(corr_matrx.shape), k=1).astype(np.bool))
to_drop = [column for column in corr_matrx.columns if any(corr_matrx[column] &gt; corr_threshold)]
if len(to_drop) &gt; 0:
x9_1 = [&#34;High Correlation&#34;, to_drop]
else:
x9_1 = [&#34;High Correlation&#34;, None]
except:
x9_1 = [&#34;High Correlation&#34;, None]
try:
x10 = list(pd.read_csv(ends_with(master_path) + &#34;IV_calculation.csv&#34;).query(
&#34;`iv`&gt;&#34; + str(iv_threshold)).attribute.values)
if len(x10) &gt; 0:
x10_1 = [&#34;Significant Attributes&#34;, x10]
else:
x10_1[&#34;Significant Attributes&#34;, None]
except:
x10_1 = [&#34;Significant Attributes&#34;, None]
blank_list_df = []
for i in [x1_1, x2_1, x3_1, x4_1, x5_1, x6_1, x7_1, x8_1, x9_1, x10_1]:
try:
for j in i[1]:
blank_list_df.append([i[0], j])
except:
blank_list_df.append([i[0], &#39;NA&#39;])
list_n = []
x1 = pd.DataFrame(blank_list_df, columns=[&#34;Metric&#34;, &#34;Attribute&#34;])
x1[&#39;Value&#39;] = &#39;✔&#39;
all_cols = (catcols_name.replace(&#34; &#34;, &#34;&#34;) + &#34;,&#34; + numcols_name.replace(&#34; &#34;, &#34;&#34;)).split(&#34;,&#34;)
remainder_cols = list(set(all_cols) - set(x1.Attribute.values))
total_metrics = set(list(x1.Metric.values))
for i in remainder_cols:
for j in total_metrics:
list_n.append([j, i])
x2 = pd.DataFrame(list_n, columns=[&#34;Metric&#34;, &#34;Attribute&#34;])
x2[&#34;Value&#34;] = &#39;✘&#39;
x = x1.append(x2, ignore_index=True)
x = x.drop_duplicates().pivot(index=&#39;Attribute&#39;, columns=&#39;Metric&#39;, values=&#39;Value&#39;).fillna(&#39;✘&#39;).reset_index()[
[&#34;Attribute&#34;, &#34;Outliers&#34;, &#34;Significant Attributes&#34;, &#34;Positive Skewness&#34;, &#34;Negative Skewness&#34;, &#34;High Variance&#34;,
&#34;High Correlation&#34;, &#34;High Kurtosis&#34;, &#34;Low Kurtosis&#34;]]
x = x[x.Attribute.values != &#34;NA&#34;]
if ds_ind[0] == 1 and ds_ind[1] &gt;= 0.5:
a5 = &#34;Data Health based on Drift Metrics &amp; Stability Index : &#34;
report = dp.Group(dp.Text(&#34;# &#34;), \
dp.Text(&#34;**Key Report Highlights**&#34;), \
dp.Text(&#34;- &#34; + a1), a2, dp.DataTable(x), dp.Text(&#34;- &#34; + a5), \
dp.Group(dp.BigNumber(heading=&#34;# Drifted Attributes&#34;,
value=str(str(drifted_feats) + &#34; out of &#34; + str(len_feats))), \
dp.BigNumber(heading=&#34;% Drifted Attributes&#34;,
value=str(np.round((100 * drifted_feats / len_feats), 2)) + &#34;%&#34;), \
dp.BigNumber(heading=&#34;# Unstable Attributes&#34;,
value=str(len(unstable_attr)) + &#34; out of &#34; + str(
len(total_unstable_attr)), change=&#34;numerical&#34;,
is_upward_change=True), \
dp.BigNumber(heading=&#34;% Unstable Attributes&#34;, value=str(
np.round(100 * len(unstable_attr) / len(total_unstable_attr), 2)) + &#34;%&#34;),
columns=4), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), label=&#34;Executive Summary&#34;)
if ds_ind[0] == 0 and ds_ind[1] &gt;= 0.5:
a5 = &#34;Data Health based on Stability Index : &#34;
report = dp.Group(dp.Text(&#34;# &#34;), \
dp.Text(&#34;**Key Report Highlights**&#34;), \
dp.Text(&#34;# &#34;), dp.Text(&#34;- &#34; + a1), a2, dp.DataTable(x), dp.Text(&#34;- &#34; + a5), \
dp.Group(dp.BigNumber(heading=&#34;# Unstable Attributes&#34;,
value=str(len(unstable_attr)) + &#34; out of &#34; + str(
len(total_unstable_attr)), change=&#34;numerical&#34;,
is_upward_change=True), \
dp.BigNumber(heading=&#34;% Unstable Attributes&#34;, value=str(
np.round(100 * len(unstable_attr) / len(total_unstable_attr), 2)) + &#34;%&#34;),
columns=2), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), label=&#34;Executive Summary&#34;)
if ds_ind[0] == 1 and ds_ind[1] == 0:
a5 = &#34;Data Health based on Drift Metrics : &#34;
report = dp.Group(dp.Text(&#34;# &#34;), \
dp.Text(&#34;**Key Report Highlights**&#34;), \
dp.Text(&#34;# &#34;), dp.Text(&#34;- &#34; + a1), a2, dp.DataTable(x), dp.Text(&#34;- &#34; + a5), \
dp.Group(dp.BigNumber(heading=&#34;# Drifted Attributes&#34;,
value=str(str(drifted_feats) + &#34; out of &#34; + str(len_feats))), \
dp.BigNumber(heading=&#34;% Drifted Attributes&#34;,
value=str(np.round((100 * drifted_feats / len_feats), 2)) + &#34;%&#34;),
columns=2), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), label=&#34;Executive Summary&#34;)
if ds_ind[0] == 0 and ds_ind[1] == 0:
report = dp.Group(dp.Text(&#34;# &#34;), \
dp.Text(&#34;**Key Report Highlights**&#34;), \
dp.Text(&#34;# &#34;), dp.Text(&#34;- &#34; + a1), a2, dp.DataTable(x), dp.Text(&#34;# &#34;),
label=&#34;Executive Summary&#34;)
if print_report:
dp.Report(default_template[0], default_template[1], report).save(
ends_with(master_path) + &#34;executive_summary.html&#34;, open=True)
return report
def wiki_generator(master_path, dataDict_path=None, metricDict_path=None, print_report=False):
&#34;&#34;&#34;
Args:
master_path: Path containing the input files.
dataDict_path: Data dictionary path. Default value is kept as None.
metricDict_path: Metric dictionary path. Default value is kept as None.
print_report: Printing option flexibility. Default value is kept as False.
Returns:
&#34;&#34;&#34;
try:
datatype_df = pd.read_csv(ends_with(master_path) + &#34;data_type.csv&#34;)
except:
datatype_df = pd.DataFrame(columns=[&#39;attribute&#39;, &#39;data_type&#39;], index=range(1))
try:
data_dict = pd.read_csv(dataDict_path).merge(datatype_df, how=&#34;outer&#34;, on=&#34;attribute&#34;)
except:
data_dict = datatype_df
try:
metric_dict = pd.read_csv(metricDict_path)
except:
metric_dict = pd.DataFrame(columns=[&#39;Section Category&#39;, &#39;Section Name&#39;, &#39;Metric Name&#39;, &#39;Metric Definitions&#39;],
index=range(1))
report = dp.Group(dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*A quick reference to the attributes from the dataset (Data Dictionary) and the metrics computed in the report (Metric Dictionary).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Select(blocks= \
[dp.Group(dp.Group(dp.Text(&#34;## &#34;), dp.DataTable(data_dict)),
label=&#34;Data Dictionary&#34;), \
dp.Group(dp.Text(&#34;##&#34;), dp.DataTable(metric_dict), label=&#34;Metric Dictionary&#34;)], \
type=dp.SelectType.TABS), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;),
label=&#34;Wiki&#34;)
if print_report:
dp.Report(default_template[0], default_template[1], report).save(
ends_with(master_path) + &#34;wiki_generator.html&#34;, open=True)
return report
def descriptive_statistics(master_path, SG_tabs, avl_recs_SG, missing_recs_SG, all_charts_num_1_, all_charts_cat_1_,
print_report=False):
&#34;&#34;&#34;
Args:
master_path: Path containing the input files.
SG_tabs: measures_of_counts&#39;,&#39;measures_of_centralTendency&#39;,&#39;measures_of_cardinality&#39;,&#39;measures_of_percentiles&#39;,&#39;measures_of_dispersion&#39;,&#39;measures_of_shape&#39;,&#39;global_summary&#39;
avl_recs_SG: Available files from the SG_tabs (Stats Generator tabs)
missing_recs_SG: Missing files from the SG_tabs (Stats Generator tabs)
all_charts_num_1_: Numerical charts (histogram) all collated in a list format supported as per datapane objects
all_charts_cat_1_: Categorical charts (barplot) all collated in a list format supported as per datapane objects
print_report: Printing option flexibility. Default value is kept as False.
Returns:
&#34;&#34;&#34;
if &#34;global_summary&#34; in avl_recs_SG:
cnt = 0
else:
cnt = 1
if len(missing_recs_SG) + cnt == len(SG_tabs):
return &#34;null_report&#34;
else:
if &#34;global_summary&#34; in avl_recs_SG:
l1 = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(&#34;*This section summarizes the dataset with key statistical metrics and distribution plots.*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Global Summary&#34;), \
dp.Group(dp.Text(&#34; Total Number of Records: **&#34; + f&#39;{rows_count:,}&#39; + &#34;**&#34;),
dp.Text(&#34; Total Number of Attributes: **&#34; + str(columns_count) + &#34;**&#34;),
dp.Text(&#34; Number of Numerical Attributes : **&#34; + str(numcols_count) + &#34;**&#34;),
dp.Text(&#34; Numerical Attributes Name : **&#34; + str(numcols_name) + &#34;**&#34;),
dp.Text(&#34; Number of Categorical Attributes : **&#34; + str(catcols_count) + &#34;**&#34;),
dp.Text(&#34; Categorical Attributes Name : **&#34; + str(catcols_name) + &#34;**&#34;), rows=6), rows=8)
else:
l1 = dp.Text(&#34;# &#34;)
if len(data_analyzer_output(master_path, avl_recs_SG, &#34;stats_generator&#34;)) &gt; 0:
l2 = dp.Text(&#34;### Statistics by Metric Type&#34;)
l3 = dp.Group(dp.Select(blocks=data_analyzer_output(master_path, avl_recs_SG, &#34;stats_generator&#34;),
type=dp.SelectType.TABS), dp.Text(&#34;# &#34;))
else:
l2 = dp.Text(&#34;# &#34;)
l3 = dp.Text(&#34;# &#34;)
if len(all_charts_num_1_) == 0 and len(all_charts_cat_1_) == 0:
l4 = 1
elif len(all_charts_num_1_) == 0 and len(all_charts_cat_1_) &gt; 0:
l4 = dp.Text(&#34;# &#34;), dp.Text(&#34;### Attribute Visualization&#34;), \
dp.Select(blocks=all_charts_cat_1_, type=dp.SelectType.DROPDOWN), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;)
elif len(all_charts_num_1_) &gt; 0 and len(all_charts_cat_1_) == 0:
l4 = dp.Text(&#34;# &#34;), dp.Text(&#34;### Attribute Visualization&#34;), \
dp.Select(blocks=all_charts_num_1_, type=dp.SelectType.DROPDOWN), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;)
else:
l4 = dp.Text(&#34;# &#34;), dp.Text(&#34;### Attribute Visualization&#34;), \
dp.Group(dp.Select(blocks= \
[dp.Group(dp.Select(blocks=all_charts_num_1_, type=dp.SelectType.DROPDOWN),
label=&#34;Numerical&#34;), \
dp.Group(dp.Select(blocks=all_charts_cat_1_, type=dp.SelectType.DROPDOWN),
label=&#34;Categorical&#34;)], \
type=dp.SelectType.TABS)), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;)
if l4 == 1:
report = dp.Group(l1, dp.Text(&#34;# &#34;), l2, l3, dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), label=&#34;Descriptive Statistics&#34;)
else:
report = dp.Group(l1, dp.Text(&#34;# &#34;), l2, l3, *l4, dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), label=&#34;Descriptive Statistics&#34;)
if print_report:
dp.Report(default_template[0], default_template[1], report).save(
ends_with(master_path) + &#34;descriptive_statistics.html&#34;, open=True)
return report
def quality_check(master_path, QC_tabs, avl_recs_QC, missing_recs_QC, all_charts_num_3_, print_report=False):
&#34;&#34;&#34;
Args:
master_path: Path containing the input files.
QC_tabs: nullColumns_detection&#39;,&#39;IDness_detection&#39;,&#39;biasedness_detection&#39;,&#39;invalidEntries_detection&#39;,&#39;duplicate_detection&#39;,&#39;nullRows_detection&#39;,&#39;outlier_detection&#39;
avl_recs_QC: Available files from the QC_tabs (Quality Checker tabs)
missing_recs_QC: Missing files from the QC_tabs (Quality Checker tabs)
all_charts_num_3_: Numerical charts (outlier charts) all collated in a list format supported as per datapane objects
print_report: Printing option flexibility. Default value is kept as False.
Returns:
&#34;&#34;&#34;
c_ = []
r_ = []
if len(missing_recs_QC) == len(QC_tabs):
return &#34;null_report&#34;
else:
row_wise = [&#34;duplicate_detection&#34;, &#34;nullRows_detection&#34;]
col_wise = [&#39;nullColumns_detection&#39;, &#39;IDness_detection&#39;, &#39;biasedness_detection&#39;, &#39;invalidEntries_detection&#39;,
&#39;outlier_detection&#39;]
row_wise_ = [p for p in row_wise if p in avl_recs_QC]
col_wise_ = [p for p in col_wise if p in avl_recs_QC]
len_row_wise = len([p for p in row_wise if p in avl_recs_QC])
len_col_wise = len([p for p in col_wise if p in avl_recs_QC])
if len_row_wise == 0:
c = data_analyzer_output(master_path, col_wise_, &#34;quality_checker&#34;)
for i in c:
for j in i:
if j == &#39;outlier_charts_placeholder&#39; and len(all_charts_num_3_) &gt; 1:
c_.append(dp.Select(blocks=all_charts_num_3_, type=dp.SelectType.DROPDOWN))
elif j == &#39;outlier_charts_placeholder&#39; and len(all_charts_num_3_) == 0:
c_.append(dp.Plot(blank_chart))
else:
c_.append(j)
report = dp.Group(dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section identifies the data quality issues at both row and column level.*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Group(*c_), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), rows=8, label=&#34;Quality Check&#34;)
elif len_col_wise == 0:
r = data_analyzer_output(master_path, row_wise_, &#34;quality_checker&#34;)
for i in r:
for j in i:
r_.append(j)
report = dp.Group(dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section identifies the data quality issues at both row and column level.*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Group(*r_), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), rows=8, label=&#34;Quality Check&#34;)
else:
c = data_analyzer_output(master_path, col_wise_, &#34;quality_checker&#34;)
for i in c:
for j in i:
if j == &#39;outlier_charts_placeholder&#39; and len(all_charts_num_3_) &gt; 1:
c_.append(dp.Select(blocks=all_charts_num_3_, type=dp.SelectType.DROPDOWN))
elif j == &#39;outlier_charts_placeholder&#39; and len(all_charts_num_3_) == 0:
c_.append(dp.Plot(blank_chart))
else:
c_.append(j)
r = data_analyzer_output(master_path, row_wise_, &#34;quality_checker&#34;)
for i in r:
for j in i:
r_.append(j)
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(&#34;*This section identifies the data quality issues at both row and column level.*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Select(blocks=[
dp.Group(dp.Text(&#34;# &#34;), dp.Group(*c_), rows=2, label=&#34;Column Level&#34;), \
dp.Group(dp.Text(&#34;# &#34;), dp.Group(*r_), rows=2, label=&#34;Row Level&#34;)], \
type=dp.SelectType.TABS), \
dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), \
label=&#34;Quality Check&#34;)
if print_report:
dp.Report(default_template[0], default_template[1], report).save(
ends_with(master_path) + &#34;quality_check.html&#34;, open=True)
return report
def attribute_associations(master_path, AE_tabs, avl_recs_AE, missing_recs_AE, label_col, all_charts_num_2_,
all_charts_cat_2_, print_report=False):
&#34;&#34;&#34;
Args:
master_path: Path containing the input files.
AE_tabs: correlation_matrix&#39;,&#39;IV_calculation&#39;,&#39;IG_calculation&#39;,&#39;variable_clustering&#39;
avl_recs_AE: Available files from the AE_tabs (Association Evaluator tabs)
missing_recs_AE: Missing files from the AE_tabs (Association Evaluator tabs)
label_col: label column
all_charts_num_2_: Numerical charts (histogram) all collated in a list format supported as per datapane objects
all_charts_cat_2_: Categorical charts (barplot) all collated in a list format supported as per datapane objects
print_report: Printing option flexibility. Default value is kept as False.
Returns:
&#34;&#34;&#34;
if (len(missing_recs_AE) == len(AE_tabs)) and ((len(all_charts_num_2_) + len(all_charts_cat_2_)) == 0):
return &#34;null_report&#34;
else:
if len(all_charts_num_2_) == 0 and len(all_charts_cat_2_) == 0:
l = dp.Text(&#34;##&#34;)
else:
if len(all_charts_num_2_) &gt; 0 and len(all_charts_cat_2_) == 0:
l = dp.Group(dp.Text(&#34;### Attribute to Target Association&#34;), dp.Text(
&#34;*Bivariate Distribution considering the event captured across different attribute splits (or categories)*&#34;), \
dp.Select(blocks=all_charts_num_2_, type=dp.SelectType.DROPDOWN), label=&#34;Numerical&#34;)
elif len(all_charts_num_2_) == 0 and len(all_charts_cat_2_) &gt; 0:
l = dp.Group(dp.Text(&#34;### Attribute to Target Association&#34;), dp.Text(
&#34;*Bivariate Distribution considering the event captured across different attribute splits (or categories)*&#34;), \
dp.Select(blocks=all_charts_cat_2_, type=dp.SelectType.DROPDOWN), label=&#34;Categorical&#34;)
else:
l = dp.Group(dp.Text(&#34;### Attribute to Target Association&#34;),
dp.Select(blocks=[
dp.Group(dp.Select(blocks=all_charts_num_2_, type=dp.SelectType.DROPDOWN),
label=&#34;Numerical&#34;),
dp.Group(dp.Select(blocks=all_charts_cat_2_, type=dp.SelectType.DROPDOWN),
label=&#34;Categorical&#34;)], type=dp.SelectType.TABS),
dp.Text(
&#34;*Event Rate is defined as % of event label (i.e. label 1) in a bin or a categorical value of an attribute.*&#34;),
dp.Text(&#34;# &#34;))
if len(missing_recs_AE) == len(AE_tabs):
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section analyzes the interaction between different attributes and/or the relationship between an attribute &amp; the binary target variable.*&#34;), \
dp.Text(&#34;## &#34;), \
l, \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;## &#34;), \
label=&#34;Attribute Associations&#34;)
else:
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section analyzes the interaction between different attributes and/or the relationship between an attribute &amp; the binary target variable.*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Association Matrix &amp; Plot&#34;), \
dp.Select(blocks=data_analyzer_output(master_path, avl_recs_AE, tab_name=&#34;association_evaluator&#34;),
type=dp.SelectType.DROPDOWN), \
dp.Text(&#34;### &#34;), \
dp.Text(&#34;## &#34;), \
l, \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;## &#34;), \
label=&#34;Attribute Associations&#34;)
if print_report:
dp.Report(default_template[0], default_template[1], report).save(
ends_with(master_path) + &#34;attribute_associations.html&#34;, open=True)
return report
def data_drift_stability(master_path, ds_ind, id_col, drift_threshold_model, all_drift_charts_, print_report=False):
&#34;&#34;&#34;
Args:
master_path: Path containing the input files.
ds_ind: Drift stability indicator in list form.
id_col: ID column
drift_threshold_model: threshold which the user is specifying for tagging an attribute to be drifted or not
all_drift_charts_: Charts (histogram/barplot) all collated in a list format supported as per datapane objects
print_report: Printing option flexibility. Default value is kept as False.
Returns:
&#34;&#34;&#34;
line_chart_list = []
if ds_ind[0] &gt; 0:
fig_metric_drift = go.Figure()
fig_metric_drift.add_trace(go.Scatter(x=list(drift_df[drift_df.flagged.values == 1][metric_drift[0]].values),
y=list(drift_df[drift_df.flagged.values == 1].attribute.values),
marker=dict(color=global_theme[1], size=14),
mode=&#34;markers&#34;,
name=metric_drift[0]))
fig_metric_drift.add_trace(go.Scatter(x=list(drift_df[drift_df.flagged.values == 1][metric_drift[1]].values),
y=list(drift_df[drift_df.flagged.values == 1].attribute.values),
marker=dict(color=global_theme[3], size=14),
mode=&#34;markers&#34;,
name=metric_drift[1]))
fig_metric_drift.add_trace(go.Scatter(x=list(drift_df[drift_df.flagged.values == 1][metric_drift[2]].values),
y=list(drift_df[drift_df.flagged.values == 1].attribute.values),
marker=dict(color=global_theme[5], size=14),
mode=&#34;markers&#34;,
name=metric_drift[2]))
fig_metric_drift.add_trace(go.Scatter(x=list(drift_df[drift_df.flagged.values == 1][metric_drift[3]].values),
y=list(drift_df[drift_df.flagged.values == 1].attribute.values),
marker=dict(color=global_theme[7], size=14),
mode=&#34;markers&#34;,
name=metric_drift[3]))
fig_metric_drift.add_vrect(x0=0, x1=drift_threshold_model, fillcolor=global_theme[7], opacity=0.1,
layer=&#34;below&#34;, line_width=1),
fig_metric_drift.update_layout(legend=dict(orientation=&#34;h&#34;, x=0.5, yanchor=&#34;top&#34;, xanchor=&#34;center&#34;))
fig_metric_drift.layout.plot_bgcolor = global_plot_bg_color
fig_metric_drift.layout.paper_bgcolor = global_paper_bg_color
fig_metric_drift.update_xaxes(showline=True, linewidth=2, gridcolor=px.colors.sequential.Greys[1])
fig_metric_drift.update_yaxes(showline=True, linewidth=2, gridcolor=px.colors.sequential.Greys[2])
#
Drift Chart - 2
fig_gauge_drift = go.Figure(go.Indicator(
domain={&#39;x&#39;: [0, 1], &#39;y&#39;: [0, 1]},
value=drifted_feats,
mode=&#34;gauge+number&#34;,
title={&#39;text&#39;: &#34;&#34;},
gauge={&#39;axis&#39;: {&#39;range&#39;: [None, len_feats]},
&#39;bar&#39;: {&#39;color&#39;: px.colors.sequential.Reds[7]},
&#39;steps&#39;: [
{&#39;range&#39;: [0, drifted_feats], &#39;color&#39;: px.colors.sequential.Reds[8]}, \
{&#39;range&#39;: [drifted_feats, len_feats], &#39;color&#39;: px.colors.sequential.Greens[8]}], \
&#39;threshold&#39;: {&#39;line&#39;: {&#39;color&#39;: &#39;black&#39;, &#39;width&#39;: 3}, &#39;thickness&#39;: 1, &#39;value&#39;: len_feats}}))
fig_gauge_drift.update_layout(font={&#39;color&#39;: &#34;black&#34;, &#39;family&#39;: &#34;Arial&#34;})
def drift_text_gen(drifted_feats, len_feats):
&#34;&#34;&#34;
Args:
drifted_feats: count of attributes drifted
len_feats: count of attributes passed for analysis
Returns:
&#34;&#34;&#34;
if drifted_feats == 0:
text = &#34;*Drift barometer does not indicate any drift in the underlying data. Please refer to the metric values as displayed in the above table &amp; comparison plot for better understanding*&#34;
elif drifted_feats == 1:
text = &#34;*Drift barometer indicates that &#34; + str(drifted_feats) + &#34; out of &#34; + str(
len_feats) + &#34; (&#34; + str(np.round((100 * drifted_feats / len_feats),
2)) + &#34;%) attributes has been drifted from its source behaviour.*&#34;
elif drifted_feats &gt; 1:
text = &#34;*Drift barometer indicates that &#34; + str(drifted_feats) + &#34; out of &#34; + str(
len_feats) + &#34; (&#34; + str(np.round((100 * drifted_feats / len_feats),
2)) + &#34;%) attributes have been drifted from its source behaviour.*&#34;
else:
text = &#34;&#34;
return text
else:
pass
if ds_ind[0] == 0 and ds_ind[1] == 0:
return &#34;null_report&#34;
elif ds_ind[0] == 0 and ds_ind[1] &gt; 0.5:
for i in total_unstable_attr:
line_chart_list.append(line_chart_gen_stability(df1=df_stability, df2=df_si_, col=i))
report = dp.Group(dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section examines the dataset stability wrt the baseline dataset (via computing drift statistics) and/or wrt the historical datasets (via computing stability index).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Data Stability Analysis&#34;), \
dp.DataTable(df_si), \
dp.Select(blocks=line_chart_list, type=dp.SelectType.DROPDOWN), \
dp.Group(dp.Text(&#34;**Stability Index Interpretation:**&#34;), dp.Plot(plot_index_stability),
rows=2), \
label=&#34;Drift &amp; Stability&#34;)
elif ds_ind[0] == 1 and ds_ind[1] == 0:
if len(all_drift_charts_) &gt; 0:
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section examines the dataset stability wrt the baseline dataset (via computing drift statistics) and/or wrt the historical datasets (via computing stability index).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Data Drift Analysis&#34;), \
dp.DataTable(drift_df), \
dp.Text(
&#34;*An attribute is flagged as drifted if any drift metric is found to be above the threshold of &#34; + str(
drift_threshold_model) + &#34;.*&#34;), \
dp.Text(&#34;##&#34;), \
dp.Select(blocks=all_drift_charts_, type=dp.SelectType.DROPDOWN), \
dp.Text(
&#34;*Source &amp; Target datasets were compared to see the % deviation at decile level for numerical attributes and at individual category level for categorical attributes*&#34;), \
dp.Text(&#34;###
&#34;), \
dp.Text(&#34;###
&#34;), \
dp.Text(&#34;### Data Health&#34;), \
dp.Group(dp.Plot(fig_metric_drift), dp.Plot(fig_gauge_drift), columns=2), \
dp.Group(dp.Text(&#34;*Representation of attributes across different computed Drift Metrics*&#34;),
dp.Text(drift_text_gen(drifted_feats, len_feats)), columns=2), \
label=&#34;Drift &amp; Stability&#34;)
else:
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section examines the dataset stability wrt the baseline dataset (via computing drift statistics) and/or wrt the historical datasets (via computing stability index).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Data Drift Analysis&#34;), \
dp.DataTable(drift_df), \
dp.Text(
&#34;*An attribute is flagged as drifted if any drift metric is found to be above the threshold of &#34; + str(
drift_threshold_model) + &#34;.*&#34;), \
dp.Text(&#34;##&#34;), \
dp.Text(&#34;###
&#34;), \
dp.Text(&#34;### Data Health&#34;), \
dp.Group(dp.Plot(fig_metric_drift), dp.Plot(fig_gauge_drift), columns=2), \
dp.Group(dp.Text(&#34;*Representation of attributes across different computed Drift Metrics*&#34;),
dp.Text(drift_text_gen(drifted_feats, len_feats)), columns=2), \
label=&#34;Drift &amp; Stability&#34;)
elif ds_ind[0] == 1 and ds_ind[1] &gt;= 0.5:
for i in total_unstable_attr:
line_chart_list.append(line_chart_gen_stability(df1=df_stability, df2=df_si_, col=i))
if len(all_drift_charts_) &gt; 0:
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section examines the dataset stability wrt the baseline dataset (via computing drift statistics) and/or wrt the historical datasets (via computing stability index).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Data Drift Analysis&#34;), \
dp.DataTable(drift_df), \
dp.Text(
&#34;*An attribute is flagged as drifted if any drift metric is found to be above the threshold of &#34; + str(
drift_threshold_model) + &#34;.*&#34;), \
dp.Text(&#34;##&#34;), \
dp.Select(blocks=all_drift_charts_, type=dp.SelectType.DROPDOWN), \
dp.Text(
&#34;*Source &amp; Target datasets were compared to see the % deviation at decile level for numerical attributes and at individual category level for categorical attributes*&#34;), \
dp.Text(&#34;###
&#34;), \
dp.Text(&#34;###
&#34;), \
dp.Text(&#34;### Data Health&#34;), \
dp.Group(dp.Plot(fig_metric_drift), dp.Plot(fig_gauge_drift), columns=2), \
dp.Group(dp.Text(&#34;*Representation of attributes across different computed Drift Metrics*&#34;),
dp.Text(drift_text_gen(drifted_feats, len_feats)), columns=2), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;### Data Stability Analysis&#34;), \
dp.DataTable(df_si), \
dp.Select(blocks=line_chart_list, type=dp.SelectType.DROPDOWN), \
dp.Group(dp.Text(&#34;**Stability Index Interpretation:**&#34;), dp.Plot(plot_index_stability), rows=2), \
label=&#34;Drift &amp; Stability&#34;)
else:
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section examines the dataset stability wrt the baseline dataset (via computing drift statistics) and/or wrt the historical datasets (via computing stability index).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Data Drift Analysis&#34;), \
dp.DataTable(drift_df), \
dp.Text(
&#34;*An attribute is flagged as drifted if any drift metric is found to be above the threshold of &#34; + str(
drift_threshold_model) + &#34;.*&#34;), \
dp.Text(&#34;##&#34;), \
dp.Text(&#34;### Data Health&#34;), \
dp.Group(dp.Plot(fig_metric_drift), dp.Plot(fig_gauge_drift), columns=2), \
dp.Group(dp.Text(&#34;*Representation of attributes across different computed Drift Metrics*&#34;),
dp.Text(drift_text_gen(drifted_feats, len_feats)), columns=2), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;### Data Stability Analysis&#34;), \
dp.DataTable(df_si), \
dp.Select(blocks=line_chart_list, type=dp.SelectType.DROPDOWN), \
dp.Group(dp.Text(&#34;**Stability Index Interpretation:**&#34;), dp.Plot(plot_index_stability), rows=2), \
label=&#34;Drift &amp; Stability&#34;)
elif ds_ind[0] == 0 and ds_ind[1] &gt;= 0.5:
for i in total_unstable_attr:
line_chart_list.append(line_chart_gen_stability(df1=df_stability, df2=df_si_, col=i))
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section examines the dataset stability wrt the baseline dataset (via computing drift statistics) and/or wrt the historical datasets (via computing stability index).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Data Stability Analysis&#34;), \
dp.DataTable(df_si), \
dp.Select(blocks=line_chart_list, type=dp.SelectType.DROPDOWN), \
dp.Group(dp.Text(&#34;**Stability Index Interpretation:**&#34;), dp.Plot(plot_index_stability), rows=2), \
label=&#34;Drift &amp; Stability&#34;)
else:
for i in total_unstable_attr:
line_chart_list.append(line_chart_gen_stability(df1=df_stability, df2=df_si_, col=i))
if len(all_drift_charts_) &gt; 0:
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section examines the dataset stability wrt the baseline dataset (via computing drift statistics) and/or wrt the historical datasets (via computing stability index).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Data Drift Analysis&#34;), \
dp.DataTable(drift_df), \
dp.Text(
&#34;*An attribute is flagged as drifted if any drift metric is found to be above the threshold of &#34; + str(
drift_threshold_model) + &#34;.*&#34;), \
dp.Text(&#34;##&#34;), \
dp.Select(blocks=all_drift_charts_, type=dp.SelectType.DROPDOWN), \
dp.Text(
&#34;*Source &amp; Target datasets were compared to see the % deviation at decile level for numerical attributes and at individual category level for categorical attributes*&#34;), \
dp.Text(&#34;###
&#34;), \
dp.Text(&#34;###
&#34;), \
dp.Text(&#34;### Data Health&#34;), \
dp.Group(dp.Plot(fig_metric_drift), dp.Plot(fig_gauge_drift), columns=2), \
dp.Group(dp.Text(&#34;*Representation of attributes across different computed Drift Metrics*&#34;),
dp.Text(drift_text_gen(drifted_feats, len_feats)), columns=2), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;### Data Stability Analysis&#34;), \
dp.DataTable(df_si), \
dp.Select(blocks=line_chart_list, type=dp.SelectType.DROPDOWN), \
dp.Group(dp.Text(&#34;**Stability Index Interpretation:**&#34;), dp.Plot(plot_index_stability), rows=2), \
label=&#34;Drift &amp; Stability&#34;)
else:
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section examines the dataset stability wrt the baseline dataset (via computing drift statistics) and/or wrt the historical datasets (via computing stability index).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Data Drift Analysis&#34;), \
dp.DataTable(drift_df), \
dp.Text(
&#34;*An attribute is flagged as drifted if any drift metric is found to be above the threshold of &#34; + str(
drift_threshold_model) + &#34;.*&#34;), \
dp.Text(&#34;##&#34;), \
dp.Text(&#34;### Data Health&#34;), \
dp.Group(dp.Plot(fig_metric_drift), dp.Plot(fig_gauge_drift), columns=2), \
dp.Group(dp.Text(&#34;*Representation of attributes across different computed Drift Metrics*&#34;),
dp.Text(drift_text_gen(drifted_feats, len_feats)), columns=2), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;### Data Stability Analysis&#34;), \
dp.DataTable(df_si), \
dp.Select(blocks=line_chart_list, type=dp.SelectType.DROPDOWN), \
dp.Group(dp.Text(&#34;**Stability Index Interpretation:**&#34;), dp.Plot(plot_index_stability), rows=2), \
label=&#34;Drift &amp; Stability&#34;)
if print_report:
dp.Report(default_template[0], default_template[1], report).save(
ends_with(master_path) + &#34;data_drift_stability.html&#34;, open=True)
return report
def anovos_report(master_path, id_col=&#39;&#39;, label_col=&#39;&#39;, corr_threshold=0.4, iv_threshold=0.02,
drift_threshold_model=0.1, dataDict_path=&#34;.&#34;,
metricDict_path=&#34;.&#34;, run_type=&#34;local&#34;, final_report_path=&#34;.&#34;):
&#34;&#34;&#34;
Args:
master_path: Path containing the input files.
id_col: ID column (Default value = &#39;&#39;)
label_col: label column (Default value = &#39;&#39;)
corr_threshold: Correlation threshold beyond which attributes can be categorized under correlated. (Default value = 0.4)
iv_threshold: IV threshold beyond which attributes can be called as significant. (Default value = 0.02)
drift_threshold_model: threshold which the user is specifying for tagging an attribute to be drifted or not (Default value = 0.1)
dataDict_path: Data dictionary path. Default value is kept as None.
metricDict_path: Metric dictionary path. Default value is kept as None.
run_type: local or emr option. Default is kept as local
final_report_path: Path where the report will be saved. (Default value = &#34;.&#34;)
Returns:
&#34;&#34;&#34;
if run_type == &#39;emr&#39;:
bash_cmd = &#34;aws s3 cp --recursive &#34; + ends_with(master_path) + &#34; &#34; + ends_with(&#34;report_stats&#34;)
master_path = &#34;report_stats&#34;
output = subprocess.check_output([&#39;bash&#39;, &#39;-c&#39;, bash_cmd])
if &#34;global_summary.csv&#34; not in os.listdir(master_path):
print(&#34;Minimum supporting data is unavailable, hence the Report could not be generated.&#34;)
return None
global global_summary_df
global numcols_name
global catcols_name
global rows_count
global columns_count
global numcols_count
global catcols_count
global blank_chart
global df_si_
global df_si
global unstable_attr
global total_unstable_attr
global drift_df
global metric_drift
global drift_df
global len_feats
global drift_df_stats
global drifted_feats
global df_stability
global n_df_stability
global stability_interpretation_table
global plot_index_stability
SG_tabs = [&#39;measures_of_counts&#39;, &#39;measures_of_centralTendency&#39;, &#39;measures_of_cardinality&#39;,
&#39;measures_of_percentiles&#39;, &#39;measures_of_dispersion&#39;, &#39;measures_of_shape&#39;, &#39;global_summary&#39;]
QC_tabs = [&#39;nullColumns_detection&#39;, &#39;IDness_detection&#39;, &#39;biasedness_detection&#39;, &#39;invalidEntries_detection&#39;,
&#39;duplicate_detection&#39;, &#39;nullRows_detection&#39;, &#39;outlier_detection&#39;]
AE_tabs = [&#39;correlation_matrix&#39;, &#39;IV_calculation&#39;, &#39;IG_calculation&#39;, &#39;variable_clustering&#39;]
drift_tab = [&#39;drift_statistics&#39;]
stability_tab = [&#39;stabilityIndex_computation&#39;, &#39;stabilityIndex_metrics&#39;]
avl_SG, avl_QC, avl_AE = [], [], []
stability_interpretation_table = pd.DataFrame(
[[&#39;0-1&#39;, &#39;Very Unstable&#39;], [&#39;1-2&#39;, &#39;Unstable&#39;], [&#39;2-3&#39;, &#39;Marginally Stable&#39;], [&#39;3-3.5&#39;, &#34;Stable&#34;],
[&#39;3.5-4&#39;, &#39;Very Stable&#39;]], columns=[&#39;StabilityIndex&#39;, &#39;StabilityOrder&#39;])
plot_index_stability = go.Figure(data=[go.Table(
header=dict(values=list(stability_interpretation_table.columns), fill_color=px.colors.sequential.Greys[2],
align=&#39;center&#39;, font=dict(size=12)),
cells=dict(
values=[stability_interpretation_table.StabilityIndex, stability_interpretation_table.StabilityOrder],
line_color=px.colors.sequential.Greys[2], fill_color=&#39;white&#39;, align=&#39;center&#39;, height=25),
columnwidth=[2, 10])])
plot_index_stability.update_layout(margin=dict(l=20, r=700, t=20, b=20))
blank_chart = go.Figure()
blank_chart.update_layout(autosize=False, width=10, height=10)
blank_chart.layout.plot_bgcolor = global_plot_bg_color
blank_chart.layout.paper_bgcolor = global_paper_bg_color
blank_chart.update_xaxes(visible=False)
blank_chart.update_yaxes(visible=False)
global_summary_df = pd.read_csv(ends_with(master_path) + &#34;global_summary.csv&#34;)
rows_count = int(global_summary_df[global_summary_df.metric.values == &#34;rows_count&#34;].value.values[0])
catcols_count = int(global_summary_df[global_summary_df.metric.values == &#34;catcols_count&#34;].value.values[0])
numcols_count = int(global_summary_df[global_summary_df.metric.values == &#34;numcols_count&#34;].value.values[0])
columns_count = int(global_summary_df[global_summary_df.metric.values == &#34;columns_count&#34;].value.values[0])
if catcols_count &gt; 0:
catcols_name = &#34;,&#34;.join(list(global_summary_df[global_summary_df.metric.values == &#34;catcols_name&#34;].value.values))
else:
catcols_name = &#34;&#34;
if numcols_count &gt; 0:
numcols_name = &#34;,&#34;.join(list(global_summary_df[global_summary_df.metric.values == &#34;numcols_name&#34;].value.values))
else:
numcols_name = &#34;&#34;
all_files = os.listdir(master_path)
eventDist_charts = [x for x in all_files if &#34;eventDist&#34; in x]
stats_files = [x for x in all_files if &#34;.csv&#34; in x]
freq_charts = [x for x in all_files if &#34;freqDist&#34; in x]
outlier_charts = [x for x in all_files if &#34;outlier&#34; in x]
drift_charts = [x for x in all_files if &#34;drift&#34; in x and &#34;.csv&#34; not in x]
all_charts_num_1_ = chart_gen_list(master_path, chart_type=freq_charts, type_col=&#34;numerical&#34;)
all_charts_num_2_ = chart_gen_list(master_path, chart_type=eventDist_charts, type_col=&#34;numerical&#34;)
all_charts_num_3_ = chart_gen_list(master_path, chart_type=outlier_charts, type_col=&#34;numerical&#34;)
all_charts_cat_1_ = chart_gen_list(master_path, chart_type=freq_charts, type_col=&#34;categorical&#34;)
all_charts_cat_2_ = chart_gen_list(master_path, chart_type=eventDist_charts, type_col=&#34;categorical&#34;)
all_drift_charts_ = chart_gen_list(master_path, chart_type=drift_charts)
for x in [all_charts_num_1_, all_charts_num_2_, all_charts_num_3_, all_charts_cat_1_, all_charts_cat_2_,
all_drift_charts_]:
if len(x) == 1:
x.append(dp.Plot(blank_chart, label=&#34; &#34;))
else:
x
mapping_tab_list = []
for i in stats_files:
if i.split(&#34;.csv&#34;)[0] in SG_tabs:
mapping_tab_list.append([i.split(&#34;.csv&#34;)[0], &#34;Descriptive Statistics&#34;])
elif i.split(&#34;.csv&#34;)[0] in QC_tabs:
mapping_tab_list.append([i.split(&#34;.csv&#34;)[0], &#34;Quality Check&#34;])
elif i.split(&#34;.csv&#34;)[0] in AE_tabs:
mapping_tab_list.append([i.split(&#34;.csv&#34;)[0], &#34;Attribute Associations&#34;])
elif i.split(&#34;.csv&#34;)[0] in drift_tab or i.split(&#34;.csv&#34;)[0] in stability_tab:
mapping_tab_list.append([i.split(&#34;.csv&#34;)[0], &#34;Data Drift &amp; Data Stability&#34;])
else:
mapping_tab_list.append([i.split(&#34;.csv&#34;)[0], &#34;null&#34;])
xx = pd.DataFrame(mapping_tab_list, columns=[&#34;file_name&#34;, &#34;tab_name&#34;])
xx_avl = list(set(xx.file_name.values))
for i in SG_tabs:
if i in xx_avl:
avl_SG.append(i)
for j in QC_tabs:
if j in xx_avl:
avl_QC.append(j)
for k in AE_tabs:
if k in xx_avl:
avl_AE.append(k)
missing_SG = list(set(SG_tabs) - set(avl_SG))
missing_QC = list(set(QC_tabs) - set(avl_QC))
missing_AE = list(set(AE_tabs) - set(avl_AE))
missing_drift = list(set(drift_tab) - set(xx[xx.tab_name.values == &#34;Data Drift &amp; Data Stability&#34;].file_name.values))
missing_stability = list(
set(stability_tab) - set(xx[xx.tab_name.values == &#34;Data Drift &amp; Data Stability&#34;].file_name.values))
ds_ind = drift_stability_ind(missing_drift, drift_tab, missing_stability, stability_tab)
if ds_ind[0] &gt; 0:
drift_df = pd.read_csv(ends_with(master_path) + &#34;drift_statistics.csv&#34;).sort_values(by=[&#39;flagged&#39;],
ascending=False)
metric_drift = list(drift_df.drop([&#34;attribute&#34;, &#34;flagged&#34;], 1).columns)
drift_df = drift_df[drift_df.attribute.values != id_col]
len_feats = drift_df.shape[0]
drift_df_stats = drift_df[drift_df.flagged.values == 1] \
.melt(id_vars=&#34;attribute&#34;, value_vars=metric_drift) \
.sort_values(by=[&#39;variable&#39;, &#39;value&#39;], ascending=False)
drifted_feats = drift_df[drift_df.flagged.values == 1].shape[0]
if ds_ind[1] &gt; 0.5:
df_stability = pd.read_csv(ends_with(master_path) + &#34;stabilityIndex_metrics.csv&#34;)
df_stability[&#39;idx&#39;] = df_stability[&#39;idx&#39;].astype(str).apply(lambda x: &#39;df&#39; + x)
n_df_stability = str(df_stability[&#34;idx&#34;].nunique())
df_si_ = pd.read_csv(ends_with(master_path) + &#34;stabilityIndex_computation.csv&#34;)
df_si = df_si_[[&#34;attribute&#34;, &#34;stability_index&#34;, &#34;mean_si&#34;, &#34;stddev_si&#34;, &#34;kurtosis_si&#34;, &#34;flagged&#34;]]
unstable_attr = list(df_si_[df_si_.flagged.values == 1].attribute.values)
total_unstable_attr = list(df_si_.attribute.values)
elif ds_ind[1] == 0.5:
df_si_ = pd.read_csv(ends_with(master_path) + &#34;stabilityIndex_computation.csv&#34;)
df_si = df_si_[[&#34;attribute&#34;, &#34;stability_index&#34;, &#34;mean_si&#34;, &#34;stddev_si&#34;, &#34;kurtosis_si&#34;, &#34;flagged&#34;]]
unstable_attr = list(df_si_[df_si_.flagged.values == 1].attribute.values)
total_unstable_attr = list(df_si_.attribute.values)
df_stability = pd.DataFrame()
n_df_stability = &#34;the&#34;
else:
pass
tab1 = executive_summary_gen(master_path, label_col, ds_ind, id_col, iv_threshold, corr_threshold)
tab2 = wiki_generator(master_path, dataDict_path=dataDict_path, metricDict_path=metricDict_path)
tab3 = descriptive_statistics(master_path, SG_tabs, avl_SG, missing_SG, all_charts_num_1_, all_charts_cat_1_)
tab4 = quality_check(master_path, QC_tabs, avl_QC, missing_QC, all_charts_num_3_)
tab5 = attribute_associations(master_path, AE_tabs, avl_AE, missing_AE, label_col, all_charts_num_2_,
all_charts_cat_2_)
tab6 = data_drift_stability(master_path, ds_ind, id_col, drift_threshold_model, all_drift_charts_)
final_tabs_list = []
for i in [tab1, tab2, tab3, tab4, tab5, tab6]:
if i == &#34;null_report&#34;:
pass
else:
final_tabs_list.append(i)
if run_type == &#34;local&#34;:
final_report = dp.Report(default_template[0], default_template[1], \
dp.Select(blocks=final_tabs_list, type=dp.SelectType.TABS)) \
.save(ends_with(final_report_path) + &#34;ml_anovos_report.html&#34;, open=True)
else:
final_report = dp.Report(default_template[0], default_template[1], \
dp.Select(blocks=final_tabs_list, type=dp.SelectType.TABS)) \
.save(&#34;ml_anovos_report.html&#34;, open=True)
bash_cmd = &#34;aws s3 cp ml_anovos_report.html &#34; + ends_with(final_report_path)
output = subprocess.check_output([&#39;bash&#39;, &#39;-c&#39;, bash_cmd])
print(&#34;Report generated successfully at the specified location&#34;)
```
</details>
## Functions
<dl>
<dt id="anovos.data_report.report_generation.anovos_report"><code class="name flex">
<span>def <span class="ident">anovos_report</span></span>(<span>master_path, id_col='', label_col='', corr_threshold=0.4, iv_threshold=0.02, drift_threshold_model=0.1, dataDict_path='.', metricDict_path='.', run_type='local', final_report_path='.')</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>master_path</code></strong></dt>
<dd>Path containing the input files.</dd>
<dt><strong><code>id_col</code></strong></dt>
<dd>ID column (Default value = '')</dd>
<dt><strong><code>label_col</code></strong></dt>
<dd>label column (Default value = '')</dd>
<dt><strong><code>corr_threshold</code></strong></dt>
<dd>Correlation threshold beyond which attributes can be categorized under correlated. (Default value = 0.4)</dd>
<dt><strong><code>iv_threshold</code></strong></dt>
<dd>IV threshold beyond which attributes can be called as significant. (Default value = 0.02)</dd>
<dt><strong><code>drift_threshold_model</code></strong></dt>
<dd>threshold which the user is specifying for tagging an attribute to be drifted or not (Default value = 0.1)</dd>
<dt><strong><code>dataDict_path</code></strong></dt>
<dd>Data dictionary path. Default value is kept as None.</dd>
<dt><strong><code>metricDict_path</code></strong></dt>
<dd>Metric dictionary path. Default value is kept as None.</dd>
<dt><strong><code>run_type</code></strong></dt>
<dd>local or emr option. Default is kept as local</dd>
<dt><strong><code>final_report_path</code></strong></dt>
<dd>Path where the report will be saved. (Default value = ".")</dd>
</dl>
<p>Returns:</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def anovos_report(master_path, id_col=&#39;&#39;, label_col=&#39;&#39;, corr_threshold=0.4, iv_threshold=0.02,
drift_threshold_model=0.1, dataDict_path=&#34;.&#34;,
metricDict_path=&#34;.&#34;, run_type=&#34;local&#34;, final_report_path=&#34;.&#34;):
&#34;&#34;&#34;
Args:
master_path: Path containing the input files.
id_col: ID column (Default value = &#39;&#39;)
label_col: label column (Default value = &#39;&#39;)
corr_threshold: Correlation threshold beyond which attributes can be categorized under correlated. (Default value = 0.4)
iv_threshold: IV threshold beyond which attributes can be called as significant. (Default value = 0.02)
drift_threshold_model: threshold which the user is specifying for tagging an attribute to be drifted or not (Default value = 0.1)
dataDict_path: Data dictionary path. Default value is kept as None.
metricDict_path: Metric dictionary path. Default value is kept as None.
run_type: local or emr option. Default is kept as local
final_report_path: Path where the report will be saved. (Default value = &#34;.&#34;)
Returns:
&#34;&#34;&#34;
if run_type == &#39;emr&#39;:
bash_cmd = &#34;aws s3 cp --recursive &#34; + ends_with(master_path) + &#34; &#34; + ends_with(&#34;report_stats&#34;)
master_path = &#34;report_stats&#34;
output = subprocess.check_output([&#39;bash&#39;, &#39;-c&#39;, bash_cmd])
if &#34;global_summary.csv&#34; not in os.listdir(master_path):
print(&#34;Minimum supporting data is unavailable, hence the Report could not be generated.&#34;)
return None
global global_summary_df
global numcols_name
global catcols_name
global rows_count
global columns_count
global numcols_count
global catcols_count
global blank_chart
global df_si_
global df_si
global unstable_attr
global total_unstable_attr
global drift_df
global metric_drift
global drift_df
global len_feats
global drift_df_stats
global drifted_feats
global df_stability
global n_df_stability
global stability_interpretation_table
global plot_index_stability
SG_tabs = [&#39;measures_of_counts&#39;, &#39;measures_of_centralTendency&#39;, &#39;measures_of_cardinality&#39;,
&#39;measures_of_percentiles&#39;, &#39;measures_of_dispersion&#39;, &#39;measures_of_shape&#39;, &#39;global_summary&#39;]
QC_tabs = [&#39;nullColumns_detection&#39;, &#39;IDness_detection&#39;, &#39;biasedness_detection&#39;, &#39;invalidEntries_detection&#39;,
&#39;duplicate_detection&#39;, &#39;nullRows_detection&#39;, &#39;outlier_detection&#39;]
AE_tabs = [&#39;correlation_matrix&#39;, &#39;IV_calculation&#39;, &#39;IG_calculation&#39;, &#39;variable_clustering&#39;]
drift_tab = [&#39;drift_statistics&#39;]
stability_tab = [&#39;stabilityIndex_computation&#39;, &#39;stabilityIndex_metrics&#39;]
avl_SG, avl_QC, avl_AE = [], [], []
stability_interpretation_table = pd.DataFrame(
[[&#39;0-1&#39;, &#39;Very Unstable&#39;], [&#39;1-2&#39;, &#39;Unstable&#39;], [&#39;2-3&#39;, &#39;Marginally Stable&#39;], [&#39;3-3.5&#39;, &#34;Stable&#34;],
[&#39;3.5-4&#39;, &#39;Very Stable&#39;]], columns=[&#39;StabilityIndex&#39;, &#39;StabilityOrder&#39;])
plot_index_stability = go.Figure(data=[go.Table(
header=dict(values=list(stability_interpretation_table.columns), fill_color=px.colors.sequential.Greys[2],
align=&#39;center&#39;, font=dict(size=12)),
cells=dict(
values=[stability_interpretation_table.StabilityIndex, stability_interpretation_table.StabilityOrder],
line_color=px.colors.sequential.Greys[2], fill_color=&#39;white&#39;, align=&#39;center&#39;, height=25),
columnwidth=[2, 10])])
plot_index_stability.update_layout(margin=dict(l=20, r=700, t=20, b=20))
blank_chart = go.Figure()
blank_chart.update_layout(autosize=False, width=10, height=10)
blank_chart.layout.plot_bgcolor = global_plot_bg_color
blank_chart.layout.paper_bgcolor = global_paper_bg_color
blank_chart.update_xaxes(visible=False)
blank_chart.update_yaxes(visible=False)
global_summary_df = pd.read_csv(ends_with(master_path) + &#34;global_summary.csv&#34;)
rows_count = int(global_summary_df[global_summary_df.metric.values == &#34;rows_count&#34;].value.values[0])
catcols_count = int(global_summary_df[global_summary_df.metric.values == &#34;catcols_count&#34;].value.values[0])
numcols_count = int(global_summary_df[global_summary_df.metric.values == &#34;numcols_count&#34;].value.values[0])
columns_count = int(global_summary_df[global_summary_df.metric.values == &#34;columns_count&#34;].value.values[0])
if catcols_count &gt; 0:
catcols_name = &#34;,&#34;.join(list(global_summary_df[global_summary_df.metric.values == &#34;catcols_name&#34;].value.values))
else:
catcols_name = &#34;&#34;
if numcols_count &gt; 0:
numcols_name = &#34;,&#34;.join(list(global_summary_df[global_summary_df.metric.values == &#34;numcols_name&#34;].value.values))
else:
numcols_name = &#34;&#34;
all_files = os.listdir(master_path)
eventDist_charts = [x for x in all_files if &#34;eventDist&#34; in x]
stats_files = [x for x in all_files if &#34;.csv&#34; in x]
freq_charts = [x for x in all_files if &#34;freqDist&#34; in x]
outlier_charts = [x for x in all_files if &#34;outlier&#34; in x]
drift_charts = [x for x in all_files if &#34;drift&#34; in x and &#34;.csv&#34; not in x]
all_charts_num_1_ = chart_gen_list(master_path, chart_type=freq_charts, type_col=&#34;numerical&#34;)
all_charts_num_2_ = chart_gen_list(master_path, chart_type=eventDist_charts, type_col=&#34;numerical&#34;)
all_charts_num_3_ = chart_gen_list(master_path, chart_type=outlier_charts, type_col=&#34;numerical&#34;)
all_charts_cat_1_ = chart_gen_list(master_path, chart_type=freq_charts, type_col=&#34;categorical&#34;)
all_charts_cat_2_ = chart_gen_list(master_path, chart_type=eventDist_charts, type_col=&#34;categorical&#34;)
all_drift_charts_ = chart_gen_list(master_path, chart_type=drift_charts)
for x in [all_charts_num_1_, all_charts_num_2_, all_charts_num_3_, all_charts_cat_1_, all_charts_cat_2_,
all_drift_charts_]:
if len(x) == 1:
x.append(dp.Plot(blank_chart, label=&#34; &#34;))
else:
x
mapping_tab_list = []
for i in stats_files:
if i.split(&#34;.csv&#34;)[0] in SG_tabs:
mapping_tab_list.append([i.split(&#34;.csv&#34;)[0], &#34;Descriptive Statistics&#34;])
elif i.split(&#34;.csv&#34;)[0] in QC_tabs:
mapping_tab_list.append([i.split(&#34;.csv&#34;)[0], &#34;Quality Check&#34;])
elif i.split(&#34;.csv&#34;)[0] in AE_tabs:
mapping_tab_list.append([i.split(&#34;.csv&#34;)[0], &#34;Attribute Associations&#34;])
elif i.split(&#34;.csv&#34;)[0] in drift_tab or i.split(&#34;.csv&#34;)[0] in stability_tab:
mapping_tab_list.append([i.split(&#34;.csv&#34;)[0], &#34;Data Drift &amp; Data Stability&#34;])
else:
mapping_tab_list.append([i.split(&#34;.csv&#34;)[0], &#34;null&#34;])
xx = pd.DataFrame(mapping_tab_list, columns=[&#34;file_name&#34;, &#34;tab_name&#34;])
xx_avl = list(set(xx.file_name.values))
for i in SG_tabs:
if i in xx_avl:
avl_SG.append(i)
for j in QC_tabs:
if j in xx_avl:
avl_QC.append(j)
for k in AE_tabs:
if k in xx_avl:
avl_AE.append(k)
missing_SG = list(set(SG_tabs) - set(avl_SG))
missing_QC = list(set(QC_tabs) - set(avl_QC))
missing_AE = list(set(AE_tabs) - set(avl_AE))
missing_drift = list(set(drift_tab) - set(xx[xx.tab_name.values == &#34;Data Drift &amp; Data Stability&#34;].file_name.values))
missing_stability = list(
set(stability_tab) - set(xx[xx.tab_name.values == &#34;Data Drift &amp; Data Stability&#34;].file_name.values))
ds_ind = drift_stability_ind(missing_drift, drift_tab, missing_stability, stability_tab)
if ds_ind[0] &gt; 0:
drift_df = pd.read_csv(ends_with(master_path) + &#34;drift_statistics.csv&#34;).sort_values(by=[&#39;flagged&#39;],
ascending=False)
metric_drift = list(drift_df.drop([&#34;attribute&#34;, &#34;flagged&#34;], 1).columns)
drift_df = drift_df[drift_df.attribute.values != id_col]
len_feats = drift_df.shape[0]
drift_df_stats = drift_df[drift_df.flagged.values == 1] \
.melt(id_vars=&#34;attribute&#34;, value_vars=metric_drift) \
.sort_values(by=[&#39;variable&#39;, &#39;value&#39;], ascending=False)
drifted_feats = drift_df[drift_df.flagged.values == 1].shape[0]
if ds_ind[1] &gt; 0.5:
df_stability = pd.read_csv(ends_with(master_path) + &#34;stabilityIndex_metrics.csv&#34;)
df_stability[&#39;idx&#39;] = df_stability[&#39;idx&#39;].astype(str).apply(lambda x: &#39;df&#39; + x)
n_df_stability = str(df_stability[&#34;idx&#34;].nunique())
df_si_ = pd.read_csv(ends_with(master_path) + &#34;stabilityIndex_computation.csv&#34;)
df_si = df_si_[[&#34;attribute&#34;, &#34;stability_index&#34;, &#34;mean_si&#34;, &#34;stddev_si&#34;, &#34;kurtosis_si&#34;, &#34;flagged&#34;]]
unstable_attr = list(df_si_[df_si_.flagged.values == 1].attribute.values)
total_unstable_attr = list(df_si_.attribute.values)
elif ds_ind[1] == 0.5:
df_si_ = pd.read_csv(ends_with(master_path) + &#34;stabilityIndex_computation.csv&#34;)
df_si = df_si_[[&#34;attribute&#34;, &#34;stability_index&#34;, &#34;mean_si&#34;, &#34;stddev_si&#34;, &#34;kurtosis_si&#34;, &#34;flagged&#34;]]
unstable_attr = list(df_si_[df_si_.flagged.values == 1].attribute.values)
total_unstable_attr = list(df_si_.attribute.values)
df_stability = pd.DataFrame()
n_df_stability = &#34;the&#34;
else:
pass
tab1 = executive_summary_gen(master_path, label_col, ds_ind, id_col, iv_threshold, corr_threshold)
tab2 = wiki_generator(master_path, dataDict_path=dataDict_path, metricDict_path=metricDict_path)
tab3 = descriptive_statistics(master_path, SG_tabs, avl_SG, missing_SG, all_charts_num_1_, all_charts_cat_1_)
tab4 = quality_check(master_path, QC_tabs, avl_QC, missing_QC, all_charts_num_3_)
tab5 = attribute_associations(master_path, AE_tabs, avl_AE, missing_AE, label_col, all_charts_num_2_,
all_charts_cat_2_)
tab6 = data_drift_stability(master_path, ds_ind, id_col, drift_threshold_model, all_drift_charts_)
final_tabs_list = []
for i in [tab1, tab2, tab3, tab4, tab5, tab6]:
if i == &#34;null_report&#34;:
pass
else:
final_tabs_list.append(i)
if run_type == &#34;local&#34;:
final_report = dp.Report(default_template[0], default_template[1], \
dp.Select(blocks=final_tabs_list, type=dp.SelectType.TABS)) \
.save(ends_with(final_report_path) + &#34;ml_anovos_report.html&#34;, open=True)
else:
final_report = dp.Report(default_template[0], default_template[1], \
dp.Select(blocks=final_tabs_list, type=dp.SelectType.TABS)) \
.save(&#34;ml_anovos_report.html&#34;, open=True)
bash_cmd = &#34;aws s3 cp ml_anovos_report.html &#34; + ends_with(final_report_path)
output = subprocess.check_output([&#39;bash&#39;, &#39;-c&#39;, bash_cmd])
print(&#34;Report generated successfully at the specified location&#34;)
```
</details>
</dd>
<dt id="anovos.data_report.report_generation.attribute_associations"><code class="name flex">
<span>def <span class="ident">attribute_associations</span></span>(<span>master_path, AE_tabs, avl_recs_AE, missing_recs_AE, label_col, all_charts_num_2_, all_charts_cat_2_, print_report=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>master_path</code></strong></dt>
<dd>Path containing the input files.</dd>
<dt><strong><code>AE_tabs</code></strong></dt>
<dd>correlation_matrix','IV_calculation','IG_calculation','variable_clustering'</dd>
<dt><strong><code>avl_recs_AE</code></strong></dt>
<dd>Available files from the AE_tabs (Association Evaluator tabs)</dd>
<dt><strong><code>missing_recs_AE</code></strong></dt>
<dd>Missing files from the AE_tabs (Association Evaluator tabs)</dd>
<dt><strong><code>label_col</code></strong></dt>
<dd>label column</dd>
<dt><strong><code>all_charts_num_2_</code></strong></dt>
<dd>Numerical charts (histogram) all collated in a list format supported as per datapane objects</dd>
<dt><strong><code>all_charts_cat_2_</code></strong></dt>
<dd>Categorical charts (barplot) all collated in a list format supported as per datapane objects</dd>
<dt><strong><code>print_report</code></strong></dt>
<dd>Printing option flexibility. Default value is kept as False.</dd>
</dl>
<p>Returns:</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def attribute_associations(master_path, AE_tabs, avl_recs_AE, missing_recs_AE, label_col, all_charts_num_2_,
all_charts_cat_2_, print_report=False):
&#34;&#34;&#34;
Args:
master_path: Path containing the input files.
AE_tabs: correlation_matrix&#39;,&#39;IV_calculation&#39;,&#39;IG_calculation&#39;,&#39;variable_clustering&#39;
avl_recs_AE: Available files from the AE_tabs (Association Evaluator tabs)
missing_recs_AE: Missing files from the AE_tabs (Association Evaluator tabs)
label_col: label column
all_charts_num_2_: Numerical charts (histogram) all collated in a list format supported as per datapane objects
all_charts_cat_2_: Categorical charts (barplot) all collated in a list format supported as per datapane objects
print_report: Printing option flexibility. Default value is kept as False.
Returns:
&#34;&#34;&#34;
if (len(missing_recs_AE) == len(AE_tabs)) and ((len(all_charts_num_2_) + len(all_charts_cat_2_)) == 0):
return &#34;null_report&#34;
else:
if len(all_charts_num_2_) == 0 and len(all_charts_cat_2_) == 0:
l = dp.Text(&#34;##&#34;)
else:
if len(all_charts_num_2_) &gt; 0 and len(all_charts_cat_2_) == 0:
l = dp.Group(dp.Text(&#34;### Attribute to Target Association&#34;), dp.Text(
&#34;*Bivariate Distribution considering the event captured across different attribute splits (or categories)*&#34;), \
dp.Select(blocks=all_charts_num_2_, type=dp.SelectType.DROPDOWN), label=&#34;Numerical&#34;)
elif len(all_charts_num_2_) == 0 and len(all_charts_cat_2_) &gt; 0:
l = dp.Group(dp.Text(&#34;### Attribute to Target Association&#34;), dp.Text(
&#34;*Bivariate Distribution considering the event captured across different attribute splits (or categories)*&#34;), \
dp.Select(blocks=all_charts_cat_2_, type=dp.SelectType.DROPDOWN), label=&#34;Categorical&#34;)
else:
l = dp.Group(dp.Text(&#34;### Attribute to Target Association&#34;),
dp.Select(blocks=[
dp.Group(dp.Select(blocks=all_charts_num_2_, type=dp.SelectType.DROPDOWN),
label=&#34;Numerical&#34;),
dp.Group(dp.Select(blocks=all_charts_cat_2_, type=dp.SelectType.DROPDOWN),
label=&#34;Categorical&#34;)], type=dp.SelectType.TABS),
dp.Text(
&#34;*Event Rate is defined as % of event label (i.e. label 1) in a bin or a categorical value of an attribute.*&#34;),
dp.Text(&#34;# &#34;))
if len(missing_recs_AE) == len(AE_tabs):
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section analyzes the interaction between different attributes and/or the relationship between an attribute &amp; the binary target variable.*&#34;), \
dp.Text(&#34;## &#34;), \
l, \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;## &#34;), \
label=&#34;Attribute Associations&#34;)
else:
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section analyzes the interaction between different attributes and/or the relationship between an attribute &amp; the binary target variable.*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Association Matrix &amp; Plot&#34;), \
dp.Select(blocks=data_analyzer_output(master_path, avl_recs_AE, tab_name=&#34;association_evaluator&#34;),
type=dp.SelectType.DROPDOWN), \
dp.Text(&#34;### &#34;), \
dp.Text(&#34;## &#34;), \
l, \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;## &#34;), \
label=&#34;Attribute Associations&#34;)
if print_report:
dp.Report(default_template[0], default_template[1], report).save(
ends_with(master_path) + &#34;attribute_associations.html&#34;, open=True)
return report
```
</details>
</dd>
<dt id="anovos.data_report.report_generation.chart_gen_list"><code class="name flex">
<span>def <span class="ident">chart_gen_list</span></span>(<span>master_path, chart_type, type_col=None)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>master_path</code></strong></dt>
<dd>Path containing all the charts same as the other files from data analyzed output</dd>
<dt><strong><code>chart_type</code></strong></dt>
<dd>Files containing only the specific chart names for the specific chart category</dd>
<dt><strong><code>type_col</code></strong></dt>
<dd>None. Default value is kept as None</dd>
</dl>
<p>Returns:</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def chart_gen_list(master_path, chart_type, type_col=None):
&#34;&#34;&#34;
Args:
master_path: Path containing all the charts same as the other files from data analyzed output
chart_type: Files containing only the specific chart names for the specific chart category
type_col: None. Default value is kept as None
Returns:
&#34;&#34;&#34;
plot_list = []
for i in chart_type:
col_name = i[i.find(&#34;_&#34;) + 1:]
if type_col == &#34;numerical&#34;:
if col_name in numcols_name.replace(&#34; &#34;, &#34;&#34;).split(&#34;,&#34;):
plot_list.append(dp.Plot(go.Figure(json.load(open(ends_with(master_path) + i))), label=col_name))
else:
pass
elif type_col == &#34;categorical&#34;:
if col_name in catcols_name.replace(&#34; &#34;, &#34;&#34;).split(&#34;,&#34;):
plot_list.append(dp.Plot(go.Figure(json.load(open(ends_with(master_path) + i))), label=col_name))
else:
pass
else:
plot_list.append(dp.Plot(go.Figure(json.load(open(ends_with(master_path) + i))), label=col_name))
return plot_list
```
</details>
</dd>
<dt id="anovos.data_report.report_generation.data_analyzer_output"><code class="name flex">
<span>def <span class="ident">data_analyzer_output</span></span>(<span>master_path, avl_recs_tab, tab_name)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>master_path</code></strong></dt>
<dd>Path containing all the output from analyzed data</dd>
<dt><strong><code>avl_recs_tab</code></strong></dt>
<dd>Available file names from the analysis tab</dd>
<dt><strong><code>tab_name</code></strong></dt>
<dd>Analysis tab from association_evaluator / quality_checker / stats_generator</dd>
</dl>
<p>Returns:</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def data_analyzer_output(master_path, avl_recs_tab, tab_name):
&#34;&#34;&#34;
Args:
master_path: Path containing all the output from analyzed data
avl_recs_tab: Available file names from the analysis tab
tab_name: Analysis tab from association_evaluator / quality_checker / stats_generator
Returns:
&#34;&#34;&#34;
df_list = []
txt_list = []
plot_list = []
df_plot_list = []
avl_recs_tab = [x for x in avl_recs_tab if &#34;global_summary&#34; not in x]
for index, i in enumerate(avl_recs_tab):
data = pd.read_csv(ends_with(master_path) + str(i) + &#34;.csv&#34;)
if len(data.index) == 0:
continue
if tab_name == &#34;quality_checker&#34;:
if i == &#34;duplicate_detection&#34;:
duplicate_recs = pd.read_csv(ends_with(master_path) + str(i) + &#34;.csv&#34;).round(3)
unique_rows_count = &#34; No. Of Unique Rows: **&#34; + str(
format(int(duplicate_recs[duplicate_recs[&#34;metric&#34;] == &#34;unique_rows_count&#34;].value.values),
&#34;,&#34;)) + &#34;**&#34;
rows_count = &#34; No. of Rows: **&#34; + str(
format(int(duplicate_recs[duplicate_recs[&#34;metric&#34;] == &#34;rows_count&#34;].value.values), &#34;,&#34;)) + &#34;**&#34;
duplicate_rows = &#34; No. of Duplicate Rows: **&#34; + str(
format(int(duplicate_recs[duplicate_recs[&#34;metric&#34;] == &#34;duplicate_rows&#34;].value.values), &#34;,&#34;)) + &#34;**&#34;
duplicate_pct = &#34; Percentage of Duplicate Rows: **&#34; + str(
float(duplicate_recs[duplicate_recs[&#34;metric&#34;] == &#34;duplicate_pct&#34;].value.values * 100.0)) + &#34; %&#34; + &#34;**&#34;
df_list.append([dp.Text(&#34;### &#34; + str(remove_u_score(i))),
dp.Group(dp.Text(rows_count), dp.Text(unique_rows_count), dp.Text(duplicate_rows),
dp.Text(duplicate_pct), rows=4), dp.Text(&#34;#&#34;), dp.Text(&#34;#&#34;)])
elif i == &#34;outlier_detection&#34;:
df_list.append([dp.Text(&#34;### &#34; + str(remove_u_score(i))),
dp.DataTable(pd.read_csv(ends_with(master_path) + str(i) + &#34;.csv&#34;).round(3)),
&#34;outlier_charts_placeholder&#34;])
else:
df_list.append([dp.Text(&#34;### &#34; + str(remove_u_score(i))),
dp.DataTable(pd.read_csv(ends_with(master_path) + str(i) + &#34;.csv&#34;).round(3)),
dp.Text(&#34;#&#34;), dp.Text(&#34;#&#34;)])
elif tab_name == &#34;association_evaluator&#34;:
for j in avl_recs_tab:
if j == &#34;correlation_matrix&#34;:
df_list_ = pd.read_csv(ends_with(master_path) + str(j) + &#34;.csv&#34;).round(3)
feats_order = list(df_list_[&#34;attribute&#34;].values)
df_list_ = df_list_.round(3)
fig = px.imshow(df_list_[feats_order], y=feats_order, color_continuous_scale=global_theme,
aspect=&#34;auto&#34;)
fig.layout.plot_bgcolor = global_plot_bg_color
fig.layout.paper_bgcolor = global_paper_bg_color
#
fig.update_layout(title_text=str(&#34;Correlation Plot &#34;))
df_plot_list.append(
dp.Group(dp.Text(&#34;##&#34;), dp.DataTable(df_list_[[&#34;attribute&#34;] + feats_order]), dp.Plot(fig),
rows=3, label=remove_u_score(j)))
elif j == &#34;variable_clustering&#34;:
df_list_ = pd.read_csv(ends_with(master_path) + str(j) + &#34;.csv&#34;).round(3).sort_values(
by=[&#39;Cluster&#39;], ascending=True)
fig = px.sunburst(df_list_, path=[&#39;Cluster&#39;, &#39;Attribute&#39;], values=&#39;RS_Ratio&#39;,
color_discrete_sequence=global_theme)
#
fig.update_layout(title_text=str(&#34;Distribution of homogenous variable across Clusters&#34;))
fig.layout.plot_bgcolor = global_plot_bg_color
fig.layout.paper_bgcolor = global_paper_bg_color
#
fig.update_layout(title_text=str(&#34;Variable Clustering Plot &#34;))
fig.layout.autosize = True
df_plot_list.append(
dp.Group(dp.Text(&#34;##&#34;), dp.DataTable(df_list_), dp.Plot(fig), rows=3, label=remove_u_score(j)))
else:
try:
df_list_ = pd.read_csv(ends_with(master_path) + str(j) + &#34;.csv&#34;).round(3)
col_nm = [x for x in list(df_list_.columns) if &#34;attribute&#34; not in x]
df_list_ = df_list_.sort_values(col_nm[0], ascending=True)
fig = px.bar(df_list_, x=col_nm[0], y=&#39;attribute&#39;, orientation=&#39;h&#39;,
color_discrete_sequence=global_theme)
fig.layout.plot_bgcolor = global_plot_bg_color
fig.layout.paper_bgcolor = global_paper_bg_color
#
fig.update_layout(title_text=str(&#34;Representation of &#34; + str(remove_u_score(j))))
fig.layout.autosize = True
df_plot_list.append(
dp.Group(dp.Text(&#34;##&#34;), dp.DataTable(df_list_), dp.Plot(fig), label=remove_u_score(j),
rows=3))
except:
pass
if len(avl_recs_tab) == 1:
df_plot_list.append(dp.Group(dp.DataTable(pd.DataFrame(columns=[&#39; &#39;], index=range(1)), label=&#34; &#34;),
dp.Plot(blank_chart, label=&#34; &#34;), label=&#34; &#34;))
else:
pass
return df_plot_list
else:
df_list.append(dp.DataTable(pd.read_csv(ends_with(master_path) + str(i) + &#34;.csv&#34;).round(3),
label=remove_u_score(avl_recs_tab[index])))
if tab_name == &#34;quality_checker&#34; and len(avl_recs_tab) == 1:
return df_list[0], [dp.Text(&#34;#&#34;), dp.Plot(blank_chart)]
elif tab_name == &#34;stats_generator&#34; and len(avl_recs_tab) == 1:
return [df_list[0], dp.DataTable(pd.DataFrame(columns=[&#39; &#39;], index=range(1)), label=&#34; &#34;)]
else:
return df_list
```
</details>
</dd>
<dt id="anovos.data_report.report_generation.data_drift_stability"><code class="name flex">
<span>def <span class="ident">data_drift_stability</span></span>(<span>master_path, ds_ind, id_col, drift_threshold_model, all_drift_charts_, print_report=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>master_path</code></strong></dt>
<dd>Path containing the input files.</dd>
<dt><strong><code>ds_ind</code></strong></dt>
<dd>Drift stability indicator in list form.</dd>
<dt><strong><code>id_col</code></strong></dt>
<dd>ID column</dd>
<dt><strong><code>drift_threshold_model</code></strong></dt>
<dd>threshold which the user is specifying for tagging an attribute to be drifted or not</dd>
<dt><strong><code>all_drift_charts_</code></strong></dt>
<dd>Charts (histogram/barplot) all collated in a list format supported as per datapane objects</dd>
<dt><strong><code>print_report</code></strong></dt>
<dd>Printing option flexibility. Default value is kept as False.</dd>
</dl>
<p>Returns:</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def data_drift_stability(master_path, ds_ind, id_col, drift_threshold_model, all_drift_charts_, print_report=False):
&#34;&#34;&#34;
Args:
master_path: Path containing the input files.
ds_ind: Drift stability indicator in list form.
id_col: ID column
drift_threshold_model: threshold which the user is specifying for tagging an attribute to be drifted or not
all_drift_charts_: Charts (histogram/barplot) all collated in a list format supported as per datapane objects
print_report: Printing option flexibility. Default value is kept as False.
Returns:
&#34;&#34;&#34;
line_chart_list = []
if ds_ind[0] &gt; 0:
fig_metric_drift = go.Figure()
fig_metric_drift.add_trace(go.Scatter(x=list(drift_df[drift_df.flagged.values == 1][metric_drift[0]].values),
y=list(drift_df[drift_df.flagged.values == 1].attribute.values),
marker=dict(color=global_theme[1], size=14),
mode=&#34;markers&#34;,
name=metric_drift[0]))
fig_metric_drift.add_trace(go.Scatter(x=list(drift_df[drift_df.flagged.values == 1][metric_drift[1]].values),
y=list(drift_df[drift_df.flagged.values == 1].attribute.values),
marker=dict(color=global_theme[3], size=14),
mode=&#34;markers&#34;,
name=metric_drift[1]))
fig_metric_drift.add_trace(go.Scatter(x=list(drift_df[drift_df.flagged.values == 1][metric_drift[2]].values),
y=list(drift_df[drift_df.flagged.values == 1].attribute.values),
marker=dict(color=global_theme[5], size=14),
mode=&#34;markers&#34;,
name=metric_drift[2]))
fig_metric_drift.add_trace(go.Scatter(x=list(drift_df[drift_df.flagged.values == 1][metric_drift[3]].values),
y=list(drift_df[drift_df.flagged.values == 1].attribute.values),
marker=dict(color=global_theme[7], size=14),
mode=&#34;markers&#34;,
name=metric_drift[3]))
fig_metric_drift.add_vrect(x0=0, x1=drift_threshold_model, fillcolor=global_theme[7], opacity=0.1,
layer=&#34;below&#34;, line_width=1),
fig_metric_drift.update_layout(legend=dict(orientation=&#34;h&#34;, x=0.5, yanchor=&#34;top&#34;, xanchor=&#34;center&#34;))
fig_metric_drift.layout.plot_bgcolor = global_plot_bg_color
fig_metric_drift.layout.paper_bgcolor = global_paper_bg_color
fig_metric_drift.update_xaxes(showline=True, linewidth=2, gridcolor=px.colors.sequential.Greys[1])
fig_metric_drift.update_yaxes(showline=True, linewidth=2, gridcolor=px.colors.sequential.Greys[2])
#
Drift Chart - 2
fig_gauge_drift = go.Figure(go.Indicator(
domain={&#39;x&#39;: [0, 1], &#39;y&#39;: [0, 1]},
value=drifted_feats,
mode=&#34;gauge+number&#34;,
title={&#39;text&#39;: &#34;&#34;},
gauge={&#39;axis&#39;: {&#39;range&#39;: [None, len_feats]},
&#39;bar&#39;: {&#39;color&#39;: px.colors.sequential.Reds[7]},
&#39;steps&#39;: [
{&#39;range&#39;: [0, drifted_feats], &#39;color&#39;: px.colors.sequential.Reds[8]}, \
{&#39;range&#39;: [drifted_feats, len_feats], &#39;color&#39;: px.colors.sequential.Greens[8]}], \
&#39;threshold&#39;: {&#39;line&#39;: {&#39;color&#39;: &#39;black&#39;, &#39;width&#39;: 3}, &#39;thickness&#39;: 1, &#39;value&#39;: len_feats}}))
fig_gauge_drift.update_layout(font={&#39;color&#39;: &#34;black&#34;, &#39;family&#39;: &#34;Arial&#34;})
def drift_text_gen(drifted_feats, len_feats):
&#34;&#34;&#34;
Args:
drifted_feats: count of attributes drifted
len_feats: count of attributes passed for analysis
Returns:
&#34;&#34;&#34;
if drifted_feats == 0:
text = &#34;*Drift barometer does not indicate any drift in the underlying data. Please refer to the metric values as displayed in the above table &amp; comparison plot for better understanding*&#34;
elif drifted_feats == 1:
text = &#34;*Drift barometer indicates that &#34; + str(drifted_feats) + &#34; out of &#34; + str(
len_feats) + &#34; (&#34; + str(np.round((100 * drifted_feats / len_feats),
2)) + &#34;%) attributes has been drifted from its source behaviour.*&#34;
elif drifted_feats &gt; 1:
text = &#34;*Drift barometer indicates that &#34; + str(drifted_feats) + &#34; out of &#34; + str(
len_feats) + &#34; (&#34; + str(np.round((100 * drifted_feats / len_feats),
2)) + &#34;%) attributes have been drifted from its source behaviour.*&#34;
else:
text = &#34;&#34;
return text
else:
pass
if ds_ind[0] == 0 and ds_ind[1] == 0:
return &#34;null_report&#34;
elif ds_ind[0] == 0 and ds_ind[1] &gt; 0.5:
for i in total_unstable_attr:
line_chart_list.append(line_chart_gen_stability(df1=df_stability, df2=df_si_, col=i))
report = dp.Group(dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section examines the dataset stability wrt the baseline dataset (via computing drift statistics) and/or wrt the historical datasets (via computing stability index).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Data Stability Analysis&#34;), \
dp.DataTable(df_si), \
dp.Select(blocks=line_chart_list, type=dp.SelectType.DROPDOWN), \
dp.Group(dp.Text(&#34;**Stability Index Interpretation:**&#34;), dp.Plot(plot_index_stability),
rows=2), \
label=&#34;Drift &amp; Stability&#34;)
elif ds_ind[0] == 1 and ds_ind[1] == 0:
if len(all_drift_charts_) &gt; 0:
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section examines the dataset stability wrt the baseline dataset (via computing drift statistics) and/or wrt the historical datasets (via computing stability index).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Data Drift Analysis&#34;), \
dp.DataTable(drift_df), \
dp.Text(
&#34;*An attribute is flagged as drifted if any drift metric is found to be above the threshold of &#34; + str(
drift_threshold_model) + &#34;.*&#34;), \
dp.Text(&#34;##&#34;), \
dp.Select(blocks=all_drift_charts_, type=dp.SelectType.DROPDOWN), \
dp.Text(
&#34;*Source &amp; Target datasets were compared to see the % deviation at decile level for numerical attributes and at individual category level for categorical attributes*&#34;), \
dp.Text(&#34;###
&#34;), \
dp.Text(&#34;###
&#34;), \
dp.Text(&#34;### Data Health&#34;), \
dp.Group(dp.Plot(fig_metric_drift), dp.Plot(fig_gauge_drift), columns=2), \
dp.Group(dp.Text(&#34;*Representation of attributes across different computed Drift Metrics*&#34;),
dp.Text(drift_text_gen(drifted_feats, len_feats)), columns=2), \
label=&#34;Drift &amp; Stability&#34;)
else:
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section examines the dataset stability wrt the baseline dataset (via computing drift statistics) and/or wrt the historical datasets (via computing stability index).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Data Drift Analysis&#34;), \
dp.DataTable(drift_df), \
dp.Text(
&#34;*An attribute is flagged as drifted if any drift metric is found to be above the threshold of &#34; + str(
drift_threshold_model) + &#34;.*&#34;), \
dp.Text(&#34;##&#34;), \
dp.Text(&#34;###
&#34;), \
dp.Text(&#34;### Data Health&#34;), \
dp.Group(dp.Plot(fig_metric_drift), dp.Plot(fig_gauge_drift), columns=2), \
dp.Group(dp.Text(&#34;*Representation of attributes across different computed Drift Metrics*&#34;),
dp.Text(drift_text_gen(drifted_feats, len_feats)), columns=2), \
label=&#34;Drift &amp; Stability&#34;)
elif ds_ind[0] == 1 and ds_ind[1] &gt;= 0.5:
for i in total_unstable_attr:
line_chart_list.append(line_chart_gen_stability(df1=df_stability, df2=df_si_, col=i))
if len(all_drift_charts_) &gt; 0:
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section examines the dataset stability wrt the baseline dataset (via computing drift statistics) and/or wrt the historical datasets (via computing stability index).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Data Drift Analysis&#34;), \
dp.DataTable(drift_df), \
dp.Text(
&#34;*An attribute is flagged as drifted if any drift metric is found to be above the threshold of &#34; + str(
drift_threshold_model) + &#34;.*&#34;), \
dp.Text(&#34;##&#34;), \
dp.Select(blocks=all_drift_charts_, type=dp.SelectType.DROPDOWN), \
dp.Text(
&#34;*Source &amp; Target datasets were compared to see the % deviation at decile level for numerical attributes and at individual category level for categorical attributes*&#34;), \
dp.Text(&#34;###
&#34;), \
dp.Text(&#34;###
&#34;), \
dp.Text(&#34;### Data Health&#34;), \
dp.Group(dp.Plot(fig_metric_drift), dp.Plot(fig_gauge_drift), columns=2), \
dp.Group(dp.Text(&#34;*Representation of attributes across different computed Drift Metrics*&#34;),
dp.Text(drift_text_gen(drifted_feats, len_feats)), columns=2), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;### Data Stability Analysis&#34;), \
dp.DataTable(df_si), \
dp.Select(blocks=line_chart_list, type=dp.SelectType.DROPDOWN), \
dp.Group(dp.Text(&#34;**Stability Index Interpretation:**&#34;), dp.Plot(plot_index_stability), rows=2), \
label=&#34;Drift &amp; Stability&#34;)
else:
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section examines the dataset stability wrt the baseline dataset (via computing drift statistics) and/or wrt the historical datasets (via computing stability index).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Data Drift Analysis&#34;), \
dp.DataTable(drift_df), \
dp.Text(
&#34;*An attribute is flagged as drifted if any drift metric is found to be above the threshold of &#34; + str(
drift_threshold_model) + &#34;.*&#34;), \
dp.Text(&#34;##&#34;), \
dp.Text(&#34;### Data Health&#34;), \
dp.Group(dp.Plot(fig_metric_drift), dp.Plot(fig_gauge_drift), columns=2), \
dp.Group(dp.Text(&#34;*Representation of attributes across different computed Drift Metrics*&#34;),
dp.Text(drift_text_gen(drifted_feats, len_feats)), columns=2), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;### Data Stability Analysis&#34;), \
dp.DataTable(df_si), \
dp.Select(blocks=line_chart_list, type=dp.SelectType.DROPDOWN), \
dp.Group(dp.Text(&#34;**Stability Index Interpretation:**&#34;), dp.Plot(plot_index_stability), rows=2), \
label=&#34;Drift &amp; Stability&#34;)
elif ds_ind[0] == 0 and ds_ind[1] &gt;= 0.5:
for i in total_unstable_attr:
line_chart_list.append(line_chart_gen_stability(df1=df_stability, df2=df_si_, col=i))
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section examines the dataset stability wrt the baseline dataset (via computing drift statistics) and/or wrt the historical datasets (via computing stability index).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Data Stability Analysis&#34;), \
dp.DataTable(df_si), \
dp.Select(blocks=line_chart_list, type=dp.SelectType.DROPDOWN), \
dp.Group(dp.Text(&#34;**Stability Index Interpretation:**&#34;), dp.Plot(plot_index_stability), rows=2), \
label=&#34;Drift &amp; Stability&#34;)
else:
for i in total_unstable_attr:
line_chart_list.append(line_chart_gen_stability(df1=df_stability, df2=df_si_, col=i))
if len(all_drift_charts_) &gt; 0:
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section examines the dataset stability wrt the baseline dataset (via computing drift statistics) and/or wrt the historical datasets (via computing stability index).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Data Drift Analysis&#34;), \
dp.DataTable(drift_df), \
dp.Text(
&#34;*An attribute is flagged as drifted if any drift metric is found to be above the threshold of &#34; + str(
drift_threshold_model) + &#34;.*&#34;), \
dp.Text(&#34;##&#34;), \
dp.Select(blocks=all_drift_charts_, type=dp.SelectType.DROPDOWN), \
dp.Text(
&#34;*Source &amp; Target datasets were compared to see the % deviation at decile level for numerical attributes and at individual category level for categorical attributes*&#34;), \
dp.Text(&#34;###
&#34;), \
dp.Text(&#34;###
&#34;), \
dp.Text(&#34;### Data Health&#34;), \
dp.Group(dp.Plot(fig_metric_drift), dp.Plot(fig_gauge_drift), columns=2), \
dp.Group(dp.Text(&#34;*Representation of attributes across different computed Drift Metrics*&#34;),
dp.Text(drift_text_gen(drifted_feats, len_feats)), columns=2), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;### Data Stability Analysis&#34;), \
dp.DataTable(df_si), \
dp.Select(blocks=line_chart_list, type=dp.SelectType.DROPDOWN), \
dp.Group(dp.Text(&#34;**Stability Index Interpretation:**&#34;), dp.Plot(plot_index_stability), rows=2), \
label=&#34;Drift &amp; Stability&#34;)
else:
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section examines the dataset stability wrt the baseline dataset (via computing drift statistics) and/or wrt the historical datasets (via computing stability index).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Data Drift Analysis&#34;), \
dp.DataTable(drift_df), \
dp.Text(
&#34;*An attribute is flagged as drifted if any drift metric is found to be above the threshold of &#34; + str(
drift_threshold_model) + &#34;.*&#34;), \
dp.Text(&#34;##&#34;), \
dp.Text(&#34;### Data Health&#34;), \
dp.Group(dp.Plot(fig_metric_drift), dp.Plot(fig_gauge_drift), columns=2), \
dp.Group(dp.Text(&#34;*Representation of attributes across different computed Drift Metrics*&#34;),
dp.Text(drift_text_gen(drifted_feats, len_feats)), columns=2), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;## &#34;), \
dp.Text(&#34;### Data Stability Analysis&#34;), \
dp.DataTable(df_si), \
dp.Select(blocks=line_chart_list, type=dp.SelectType.DROPDOWN), \
dp.Group(dp.Text(&#34;**Stability Index Interpretation:**&#34;), dp.Plot(plot_index_stability), rows=2), \
label=&#34;Drift &amp; Stability&#34;)
if print_report:
dp.Report(default_template[0], default_template[1], report).save(
ends_with(master_path) + &#34;data_drift_stability.html&#34;, open=True)
return report
```
</details>
</dd>
<dt id="anovos.data_report.report_generation.descriptive_statistics"><code class="name flex">
<span>def <span class="ident">descriptive_statistics</span></span>(<span>master_path, SG_tabs, avl_recs_SG, missing_recs_SG, all_charts_num_1_, all_charts_cat_1_, print_report=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>master_path</code></strong></dt>
<dd>Path containing the input files.</dd>
<dt><strong><code>SG_tabs</code></strong></dt>
<dd>measures_of_counts','measures_of_centralTendency','measures_of_cardinality','measures_of_percentiles','measures_of_dispersion','measures_of_shape','global_summary'</dd>
<dt><strong><code>avl_recs_SG</code></strong></dt>
<dd>Available files from the SG_tabs (Stats Generator tabs)</dd>
<dt><strong><code>missing_recs_SG</code></strong></dt>
<dd>Missing files from the SG_tabs (Stats Generator tabs)</dd>
<dt><strong><code>all_charts_num_1_</code></strong></dt>
<dd>Numerical charts (histogram) all collated in a list format supported as per datapane objects</dd>
<dt><strong><code>all_charts_cat_1_</code></strong></dt>
<dd>Categorical charts (barplot) all collated in a list format supported as per datapane objects</dd>
<dt><strong><code>print_report</code></strong></dt>
<dd>Printing option flexibility. Default value is kept as False.</dd>
</dl>
<p>Returns:</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def descriptive_statistics(master_path, SG_tabs, avl_recs_SG, missing_recs_SG, all_charts_num_1_, all_charts_cat_1_,
print_report=False):
&#34;&#34;&#34;
Args:
master_path: Path containing the input files.
SG_tabs: measures_of_counts&#39;,&#39;measures_of_centralTendency&#39;,&#39;measures_of_cardinality&#39;,&#39;measures_of_percentiles&#39;,&#39;measures_of_dispersion&#39;,&#39;measures_of_shape&#39;,&#39;global_summary&#39;
avl_recs_SG: Available files from the SG_tabs (Stats Generator tabs)
missing_recs_SG: Missing files from the SG_tabs (Stats Generator tabs)
all_charts_num_1_: Numerical charts (histogram) all collated in a list format supported as per datapane objects
all_charts_cat_1_: Categorical charts (barplot) all collated in a list format supported as per datapane objects
print_report: Printing option flexibility. Default value is kept as False.
Returns:
&#34;&#34;&#34;
if &#34;global_summary&#34; in avl_recs_SG:
cnt = 0
else:
cnt = 1
if len(missing_recs_SG) + cnt == len(SG_tabs):
return &#34;null_report&#34;
else:
if &#34;global_summary&#34; in avl_recs_SG:
l1 = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(&#34;*This section summarizes the dataset with key statistical metrics and distribution plots.*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;### Global Summary&#34;), \
dp.Group(dp.Text(&#34; Total Number of Records: **&#34; + f&#39;{rows_count:,}&#39; + &#34;**&#34;),
dp.Text(&#34; Total Number of Attributes: **&#34; + str(columns_count) + &#34;**&#34;),
dp.Text(&#34; Number of Numerical Attributes : **&#34; + str(numcols_count) + &#34;**&#34;),
dp.Text(&#34; Numerical Attributes Name : **&#34; + str(numcols_name) + &#34;**&#34;),
dp.Text(&#34; Number of Categorical Attributes : **&#34; + str(catcols_count) + &#34;**&#34;),
dp.Text(&#34; Categorical Attributes Name : **&#34; + str(catcols_name) + &#34;**&#34;), rows=6), rows=8)
else:
l1 = dp.Text(&#34;# &#34;)
if len(data_analyzer_output(master_path, avl_recs_SG, &#34;stats_generator&#34;)) &gt; 0:
l2 = dp.Text(&#34;### Statistics by Metric Type&#34;)
l3 = dp.Group(dp.Select(blocks=data_analyzer_output(master_path, avl_recs_SG, &#34;stats_generator&#34;),
type=dp.SelectType.TABS), dp.Text(&#34;# &#34;))
else:
l2 = dp.Text(&#34;# &#34;)
l3 = dp.Text(&#34;# &#34;)
if len(all_charts_num_1_) == 0 and len(all_charts_cat_1_) == 0:
l4 = 1
elif len(all_charts_num_1_) == 0 and len(all_charts_cat_1_) &gt; 0:
l4 = dp.Text(&#34;# &#34;), dp.Text(&#34;### Attribute Visualization&#34;), \
dp.Select(blocks=all_charts_cat_1_, type=dp.SelectType.DROPDOWN), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;)
elif len(all_charts_num_1_) &gt; 0 and len(all_charts_cat_1_) == 0:
l4 = dp.Text(&#34;# &#34;), dp.Text(&#34;### Attribute Visualization&#34;), \
dp.Select(blocks=all_charts_num_1_, type=dp.SelectType.DROPDOWN), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;)
else:
l4 = dp.Text(&#34;# &#34;), dp.Text(&#34;### Attribute Visualization&#34;), \
dp.Group(dp.Select(blocks= \
[dp.Group(dp.Select(blocks=all_charts_num_1_, type=dp.SelectType.DROPDOWN),
label=&#34;Numerical&#34;), \
dp.Group(dp.Select(blocks=all_charts_cat_1_, type=dp.SelectType.DROPDOWN),
label=&#34;Categorical&#34;)], \
type=dp.SelectType.TABS)), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;)
if l4 == 1:
report = dp.Group(l1, dp.Text(&#34;# &#34;), l2, l3, dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), label=&#34;Descriptive Statistics&#34;)
else:
report = dp.Group(l1, dp.Text(&#34;# &#34;), l2, l3, *l4, dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), label=&#34;Descriptive Statistics&#34;)
if print_report:
dp.Report(default_template[0], default_template[1], report).save(
ends_with(master_path) + &#34;descriptive_statistics.html&#34;, open=True)
return report
```
</details>
</dd>
<dt id="anovos.data_report.report_generation.drift_stability_ind"><code class="name flex">
<span>def <span class="ident">drift_stability_ind</span></span>(<span>missing_recs_drift, drift_tab, missing_recs_stability, stability_tab)</span>
</code></dt>
<dd>
<div class="desc"><p>missing_recs_drift: Missing files from the drift tab
drift_tab: "drift_statistics"
missing_recs_stability: Missing files from the stability tab
stability_tab:"stabilityIndex_computation, stabilityIndex_metrics"</p>
<h2 id="args">Args</h2>
<dl>
<dt><strong><code>missing_recs_drift</code></strong></dt>
<dd>&nbsp;</dd>
<dt><strong><code>drift_tab</code></strong></dt>
<dd>&nbsp;</dd>
<dt><strong><code>missing_recs_stability</code></strong></dt>
<dd>&nbsp;</dd>
<dt><strong><code>stability_tab</code></strong></dt>
<dd>&nbsp;</dd>
</dl>
<p>Returns:</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def drift_stability_ind(missing_recs_drift, drift_tab, missing_recs_stability, stability_tab):
&#34;&#34;&#34;missing_recs_drift: Missing files from the drift tab
drift_tab: &#34;drift_statistics&#34;
missing_recs_stability: Missing files from the stability tab
stability_tab:&#34;stabilityIndex_computation, stabilityIndex_metrics&#34;
Args:
missing_recs_drift:
drift_tab:
missing_recs_stability:
stability_tab:
Returns:
&#34;&#34;&#34;
if len(missing_recs_drift) == len(drift_tab):
drift_ind = 0
else:
drift_ind = 1
if len(missing_recs_stability) == len(stability_tab):
stability_ind = 0
elif (&#34;stabilityIndex_metrics&#34; in missing_recs_stability) and (
&#34;stabilityIndex_computation&#34; not in missing_recs_stability):
stability_ind = 0.5
else:
stability_ind = 1
return drift_ind, stability_ind
```
</details>
</dd>
<dt id="anovos.data_report.report_generation.executive_summary_gen"><code class="name flex">
<span>def <span class="ident">executive_summary_gen</span></span>(<span>master_path, label_col, ds_ind, id_col, iv_threshold, corr_threshold, print_report=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>master_path</code></strong></dt>
<dd>Path containing the input files.</dd>
<dt><strong><code>label_col</code></strong></dt>
<dd>Label column.</dd>
<dt><strong><code>ds_ind</code></strong></dt>
<dd>Drift stability indicator in list form.</dd>
<dt><strong><code>id_col</code></strong></dt>
<dd>ID column.</dd>
<dt><strong><code>iv_threshold</code></strong></dt>
<dd>IV threshold beyond which attributes can be called as significant.</dd>
<dt><strong><code>corr_threshold</code></strong></dt>
<dd>Correlation threshold beyond which attributes can be categorized under correlated.</dd>
<dt><strong><code>print_report</code></strong></dt>
<dd>Printing option flexibility. Default value is kept as False.</dd>
</dl>
<p>Returns:</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def executive_summary_gen(master_path, label_col, ds_ind, id_col, iv_threshold, corr_threshold, print_report=False):
&#34;&#34;&#34;
Args:
master_path: Path containing the input files.
label_col: Label column.
ds_ind: Drift stability indicator in list form.
id_col: ID column.
iv_threshold: IV threshold beyond which attributes can be called as significant.
corr_threshold: Correlation threshold beyond which attributes can be categorized under correlated.
print_report: Printing option flexibility. Default value is kept as False.
Returns:
&#34;&#34;&#34;
try:
obj_dtls = json.load(open(ends_with(master_path) + &#34;freqDist_&#34; + str(label_col)))
text_val = list(list(obj_dtls.values())[0][0].items())[8][1]
x_val = list(list(obj_dtls.values())[0][0].items())[11][1]
y_val = list(list(obj_dtls.values())[0][0].items())[13][1]
label_fig_ = go.Figure(data=[go.Pie(labels=x_val, values=y_val, textinfo=&#39;label+percent&#39;,
insidetextorientation=&#39;radial&#39;, pull=[0, 0.1], marker_colors=global_theme)])
label_fig_.update_traces(textposition=&#39;inside&#39;, textinfo=&#39;percent+label&#39;)
label_fig_.update_layout(legend=dict(orientation=&#34;h&#34;, x=0.5, yanchor=&#34;bottom&#34;, xanchor=&#34;center&#34;))
label_fig_.layout.plot_bgcolor = global_plot_bg_color
label_fig_.layout.paper_bgcolor = global_paper_bg_color
except:
label_fig_ = None
a1 = &#34;The dataset contains
**&#34; + str(f&#34;{rows_count:,d}&#34;) + &#34;** records and **&#34; + str(
numcols_count + catcols_count) + &#34;** attributes (**&#34; + str(numcols_count) + &#34;** numerical + **&#34; + str(
catcols_count) + &#34;** categorical).&#34;
if label_col is None:
a2 = dp.Group(dp.Text(&#34;- There is **no** target variable in the dataset&#34;), dp.Text(&#34;- Data Diagnosis:&#34;), rows=2)
else:
if label_fig_ is None:
a2 = dp.Group(dp.Text(&#34;- Target variable is **&#34; + str(label_col) + &#34;** &#34;), dp.Text(&#34;- Data Diagnosis:&#34;),
rows=2)
else:
a2 = dp.Group(dp.Text(&#34;- Target variable is **&#34; + str(label_col) + &#34;** &#34;), dp.Plot(label_fig_),
dp.Text(&#34;- Data Diagnosis:&#34;), rows=3)
try:
x1 = list(pd.read_csv(ends_with(master_path) + &#34;measures_of_dispersion.csv&#34;).query(&#34;`cov`&gt;1&#34;).attribute.values)
if len(x1) &gt; 0:
x1_1 = [&#34;High Variance&#34;, x1]
else:
x1_1 = [&#34;High Variance&#34;, None]
except:
x1_1 = [&#34;High Variance&#34;, None]
try:
x2 = list(pd.read_csv(ends_with(master_path) + &#34;measures_of_shape.csv&#34;).query(&#34;`skewness`&gt;0&#34;).attribute.values)
if len(x2) &gt; 0:
x2_1 = [&#34;Positive Skewness&#34;, x2]
else:
x2_1 = [&#34;Positive Skewness&#34;, None]
except:
x2_1 = [&#34;Positive Skewness&#34;, None]
try:
x3 = list(pd.read_csv(ends_with(master_path) + &#34;measures_of_shape.csv&#34;).query(&#34;`skewness`&lt;0&#34;).attribute.values)
if len(x3) &gt; 0:
x3_1 = [&#34;Negative Skewness&#34;, x3]
else:
x3_1 = [&#34;Negative Skewness&#34;, None]
except:
x3_1 = [&#34;Negative Skewness&#34;, None]
try:
x4 = list(pd.read_csv(ends_with(master_path) + &#34;measures_of_shape.csv&#34;).query(&#34;`kurtosis`&gt;0&#34;).attribute.values)
if len(x4) &gt; 0:
x4_1 = [&#34;High Kurtosis&#34;, x4]
else:
x4_1 = [&#34;High Kurtosis&#34;, None]
except:
x4_1 = [&#34;High Kurtosis&#34;, None]
try:
x5 = list(pd.read_csv(ends_with(master_path) + &#34;measures_of_shape.csv&#34;).query(&#34;`kurtosis`&lt;0&#34;).attribute.values)
if len(x5) &gt; 0:
x5_1 = [&#34;Low Kurtosis&#34;, x5]
else:
x5_1 = [&#34;Low Kurtosis&#34;, None]
except:
x5_1 = [&#34;Low Kurtosis&#34;, None]
try:
x6 = list(
pd.read_csv(ends_with(master_path) + &#34;measures_of_counts.csv&#34;).query(&#34;`fill_pct`&lt;0.7&#34;).attribute.values)
if len(x6) &gt; 0:
x6_1 = [&#34;Low Fill Rates&#34;, x6]
else:
x6_1 = [&#34;Low Fill Rates&#34;, None]
except:
x6_1 = [&#34;Low Fill Rates&#34;, None]
try:
biasedness_df = pd.read_csv(ends_with(master_path) + &#34;biasedness_detection.csv&#34;)
if &#34;treated&#34; in biasedness_df:
x7 = list(df.query(&#34;`treated`&gt;0&#34;).attribute.values)
else:
x7 = list(df.query(&#34;`flagged`&gt;0&#34;).attribute.values)
if len(x7) &gt; 0:
x7_1 = [&#34;High Biasedness&#34;, x7]
else:
x7_1 = [&#34;High Biasedness&#34;, None]
except:
x7_1 = [&#34;High Biasedness&#34;, None]
try:
x8 = list(pd.read_csv(ends_with(master_path) + &#34;outlier_detection.csv&#34;).attribute.values)
if len(x8) &gt; 0:
x8_1 = [&#34;Outliers&#34;, x8]
else:
x8_1 = [&#34;Outliers&#34;, None]
except:
x8_1 = [&#34;Outliers&#34;, None]
try:
corr_matrx = pd.read_csv(ends_with(master_path) + &#34;correlation_matrix.csv&#34;)
corr_matrx = corr_matrx[list(corr_matrx.attribute.values)]
corr_matrx = corr_matrx.where(np.triu(np.ones(corr_matrx.shape), k=1).astype(np.bool))
to_drop = [column for column in corr_matrx.columns if any(corr_matrx[column] &gt; corr_threshold)]
if len(to_drop) &gt; 0:
x9_1 = [&#34;High Correlation&#34;, to_drop]
else:
x9_1 = [&#34;High Correlation&#34;, None]
except:
x9_1 = [&#34;High Correlation&#34;, None]
try:
x10 = list(pd.read_csv(ends_with(master_path) + &#34;IV_calculation.csv&#34;).query(
&#34;`iv`&gt;&#34; + str(iv_threshold)).attribute.values)
if len(x10) &gt; 0:
x10_1 = [&#34;Significant Attributes&#34;, x10]
else:
x10_1[&#34;Significant Attributes&#34;, None]
except:
x10_1 = [&#34;Significant Attributes&#34;, None]
blank_list_df = []
for i in [x1_1, x2_1, x3_1, x4_1, x5_1, x6_1, x7_1, x8_1, x9_1, x10_1]:
try:
for j in i[1]:
blank_list_df.append([i[0], j])
except:
blank_list_df.append([i[0], &#39;NA&#39;])
list_n = []
x1 = pd.DataFrame(blank_list_df, columns=[&#34;Metric&#34;, &#34;Attribute&#34;])
x1[&#39;Value&#39;] = &#39;✔&#39;
all_cols = (catcols_name.replace(&#34; &#34;, &#34;&#34;) + &#34;,&#34; + numcols_name.replace(&#34; &#34;, &#34;&#34;)).split(&#34;,&#34;)
remainder_cols = list(set(all_cols) - set(x1.Attribute.values))
total_metrics = set(list(x1.Metric.values))
for i in remainder_cols:
for j in total_metrics:
list_n.append([j, i])
x2 = pd.DataFrame(list_n, columns=[&#34;Metric&#34;, &#34;Attribute&#34;])
x2[&#34;Value&#34;] = &#39;✘&#39;
x = x1.append(x2, ignore_index=True)
x = x.drop_duplicates().pivot(index=&#39;Attribute&#39;, columns=&#39;Metric&#39;, values=&#39;Value&#39;).fillna(&#39;✘&#39;).reset_index()[
[&#34;Attribute&#34;, &#34;Outliers&#34;, &#34;Significant Attributes&#34;, &#34;Positive Skewness&#34;, &#34;Negative Skewness&#34;, &#34;High Variance&#34;,
&#34;High Correlation&#34;, &#34;High Kurtosis&#34;, &#34;Low Kurtosis&#34;]]
x = x[x.Attribute.values != &#34;NA&#34;]
if ds_ind[0] == 1 and ds_ind[1] &gt;= 0.5:
a5 = &#34;Data Health based on Drift Metrics &amp; Stability Index : &#34;
report = dp.Group(dp.Text(&#34;# &#34;), \
dp.Text(&#34;**Key Report Highlights**&#34;), \
dp.Text(&#34;- &#34; + a1), a2, dp.DataTable(x), dp.Text(&#34;- &#34; + a5), \
dp.Group(dp.BigNumber(heading=&#34;# Drifted Attributes&#34;,
value=str(str(drifted_feats) + &#34; out of &#34; + str(len_feats))), \
dp.BigNumber(heading=&#34;% Drifted Attributes&#34;,
value=str(np.round((100 * drifted_feats / len_feats), 2)) + &#34;%&#34;), \
dp.BigNumber(heading=&#34;# Unstable Attributes&#34;,
value=str(len(unstable_attr)) + &#34; out of &#34; + str(
len(total_unstable_attr)), change=&#34;numerical&#34;,
is_upward_change=True), \
dp.BigNumber(heading=&#34;% Unstable Attributes&#34;, value=str(
np.round(100 * len(unstable_attr) / len(total_unstable_attr), 2)) + &#34;%&#34;),
columns=4), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), label=&#34;Executive Summary&#34;)
if ds_ind[0] == 0 and ds_ind[1] &gt;= 0.5:
a5 = &#34;Data Health based on Stability Index : &#34;
report = dp.Group(dp.Text(&#34;# &#34;), \
dp.Text(&#34;**Key Report Highlights**&#34;), \
dp.Text(&#34;# &#34;), dp.Text(&#34;- &#34; + a1), a2, dp.DataTable(x), dp.Text(&#34;- &#34; + a5), \
dp.Group(dp.BigNumber(heading=&#34;# Unstable Attributes&#34;,
value=str(len(unstable_attr)) + &#34; out of &#34; + str(
len(total_unstable_attr)), change=&#34;numerical&#34;,
is_upward_change=True), \
dp.BigNumber(heading=&#34;% Unstable Attributes&#34;, value=str(
np.round(100 * len(unstable_attr) / len(total_unstable_attr), 2)) + &#34;%&#34;),
columns=2), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), label=&#34;Executive Summary&#34;)
if ds_ind[0] == 1 and ds_ind[1] == 0:
a5 = &#34;Data Health based on Drift Metrics : &#34;
report = dp.Group(dp.Text(&#34;# &#34;), \
dp.Text(&#34;**Key Report Highlights**&#34;), \
dp.Text(&#34;# &#34;), dp.Text(&#34;- &#34; + a1), a2, dp.DataTable(x), dp.Text(&#34;- &#34; + a5), \
dp.Group(dp.BigNumber(heading=&#34;# Drifted Attributes&#34;,
value=str(str(drifted_feats) + &#34; out of &#34; + str(len_feats))), \
dp.BigNumber(heading=&#34;% Drifted Attributes&#34;,
value=str(np.round((100 * drifted_feats / len_feats), 2)) + &#34;%&#34;),
columns=2), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), label=&#34;Executive Summary&#34;)
if ds_ind[0] == 0 and ds_ind[1] == 0:
report = dp.Group(dp.Text(&#34;# &#34;), \
dp.Text(&#34;**Key Report Highlights**&#34;), \
dp.Text(&#34;# &#34;), dp.Text(&#34;- &#34; + a1), a2, dp.DataTable(x), dp.Text(&#34;# &#34;),
label=&#34;Executive Summary&#34;)
if print_report:
dp.Report(default_template[0], default_template[1], report).save(
ends_with(master_path) + &#34;executive_summary.html&#34;, open=True)
return report
```
</details>
</dd>
<dt id="anovos.data_report.report_generation.line_chart_gen_stability"><code class="name flex">
<span>def <span class="ident">line_chart_gen_stability</span></span>(<span>df1, df2, col)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>df1</code></strong></dt>
<dd>Analysis dataframe pertaining to summarized stability metrics</dd>
<dt><strong><code>df2</code></strong></dt>
<dd>Analysis dataframe pertaining to historical data</dd>
<dt><strong><code>col</code></strong></dt>
<dd>Analysis column</dd>
</dl>
<p>Returns:</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def line_chart_gen_stability(df1, df2, col):
&#34;&#34;&#34;
Args:
df1: Analysis dataframe pertaining to summarized stability metrics
df2: Analysis dataframe pertaining to historical data
col: Analysis column
Returns:
&#34;&#34;&#34;
def val_cat(val):
&#34;&#34;&#34;
Args:
val:
Returns:
&#34;&#34;&#34;
if val &gt;= 3.5:
return &#34;Very Stable&#34;
elif val &gt;= 3 and val &lt; 3.5:
return &#34;Stable&#34;
elif val &gt;= 2 and val &lt; 3:
return &#34;Marginally Stable&#34;
elif val &gt;= 1 and val &lt; 2:
return &#34;Unstable&#34;
elif val &gt;= 0 and val &lt; 1:
return &#34;Very Unstable&#34;
else:
return &#34;Out of Range&#34;
val_si = list(df2[df2[&#34;attribute&#34;] == col].stability_index.values)[0]
f1 = go.Figure()
f1.add_trace(go.Indicator(
mode=&#34;gauge+number&#34;,
value=val_si,
gauge={
&#39;axis&#39;: {&#39;range&#39;: [None, 4], &#39;tickwidth&#39;: 1, &#39;tickcolor&#39;: &#34;black&#34;},
&#39;bgcolor&#39;: &#34;white&#34;,
&#39;steps&#39;: [
{&#39;range&#39;: [0, 1], &#39;color&#39;: px.colors.sequential.Reds[7]},
{&#39;range&#39;: [1, 2], &#39;color&#39;: px.colors.sequential.Reds[6]},
{&#39;range&#39;: [2, 3], &#39;color&#39;: px.colors.sequential.Oranges[4]},
{&#39;range&#39;: [3, 3.5], &#39;color&#39;: px.colors.sequential.BuGn[7]},
{&#39;range&#39;: [3.5, 4], &#39;color&#39;: px.colors.sequential.BuGn[8]}],
&#39;threshold&#39;: {
&#39;line&#39;: {&#39;color&#39;: &#34;black&#34;, &#39;width&#39;: 3},
&#39;thickness&#39;: 1,
&#39;value&#39;: val_si},
&#39;bar&#39;: {&#39;color&#39;: global_plot_bg_color}},
title={&#39;text&#39;: &#34;Order of Stability: &#34; + val_cat(val_si)}))
f1.update_layout(height=400, font={&#39;color&#39;: &#34;black&#34;, &#39;family&#39;: &#34;Arial&#34;})
f5 = &#34;Stability Index for &#34; + str(col.upper())
if len(df1.columns) &gt; 0:
df1 = df1[df1[&#34;attribute&#34;] == col]
f2 = px.line(df1, x=&#39;idx&#39;, y=&#39;mean&#39;, markers=True,
title=&#39;CV of Mean is &#39; + str(list(df2[df2[&#34;attribute&#34;] == col].mean_cv.values)[0]))
f2.update_traces(line_color=global_theme[2], marker=dict(size=14))
f2.layout.plot_bgcolor = global_plot_bg_color
f2.layout.paper_bgcolor = global_paper_bg_color
f3 = px.line(df1, x=&#39;idx&#39;, y=&#39;stddev&#39;, markers=True,
title=&#39;CV of Stddev is &#39; + str(list(df2[df2[&#34;attribute&#34;] == col].stddev_cv.values)[0]))
f3.update_traces(line_color=global_theme[6], marker=dict(size=14))
f3.layout.plot_bgcolor = global_plot_bg_color
f3.layout.paper_bgcolor = global_paper_bg_color
f4 = px.line(df1, x=&#39;idx&#39;, y=&#39;kurtosis&#39;, markers=True,
title=&#39;CV of Kurtosis is &#39; + str(list(df2[df2[&#34;attribute&#34;] == col].kurtosis_cv.values)[0]))
f4.update_traces(line_color=global_theme[4], marker=dict(size=14))
f4.layout.plot_bgcolor = global_plot_bg_color
f4.layout.paper_bgcolor = global_paper_bg_color
return dp.Group(dp.Text(&#34;#&#34;), dp.Text(f5), dp.Plot(f1),
dp.Group(dp.Plot(f2), dp.Plot(f3), dp.Plot(f4), columns=3), rows=4, label=col)
else:
return dp.Group(dp.Text(&#34;#&#34;), dp.Text(f5), dp.Plot(f1), rows=3, label=col)
```
</details>
</dd>
<dt id="anovos.data_report.report_generation.quality_check"><code class="name flex">
<span>def <span class="ident">quality_check</span></span>(<span>master_path, QC_tabs, avl_recs_QC, missing_recs_QC, all_charts_num_3_, print_report=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>master_path</code></strong></dt>
<dd>Path containing the input files.</dd>
<dt><strong><code>QC_tabs</code></strong></dt>
<dd>nullColumns_detection','IDness_detection','biasedness_detection','invalidEntries_detection','duplicate_detection','nullRows_detection','outlier_detection'</dd>
<dt><strong><code>avl_recs_QC</code></strong></dt>
<dd>Available files from the QC_tabs (Quality Checker tabs)</dd>
<dt><strong><code>missing_recs_QC</code></strong></dt>
<dd>Missing files from the QC_tabs (Quality Checker tabs)</dd>
<dt><strong><code>all_charts_num_3_</code></strong></dt>
<dd>Numerical charts (outlier charts) all collated in a list format supported as per datapane objects</dd>
<dt><strong><code>print_report</code></strong></dt>
<dd>Printing option flexibility. Default value is kept as False.</dd>
</dl>
<p>Returns:</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def quality_check(master_path, QC_tabs, avl_recs_QC, missing_recs_QC, all_charts_num_3_, print_report=False):
&#34;&#34;&#34;
Args:
master_path: Path containing the input files.
QC_tabs: nullColumns_detection&#39;,&#39;IDness_detection&#39;,&#39;biasedness_detection&#39;,&#39;invalidEntries_detection&#39;,&#39;duplicate_detection&#39;,&#39;nullRows_detection&#39;,&#39;outlier_detection&#39;
avl_recs_QC: Available files from the QC_tabs (Quality Checker tabs)
missing_recs_QC: Missing files from the QC_tabs (Quality Checker tabs)
all_charts_num_3_: Numerical charts (outlier charts) all collated in a list format supported as per datapane objects
print_report: Printing option flexibility. Default value is kept as False.
Returns:
&#34;&#34;&#34;
c_ = []
r_ = []
if len(missing_recs_QC) == len(QC_tabs):
return &#34;null_report&#34;
else:
row_wise = [&#34;duplicate_detection&#34;, &#34;nullRows_detection&#34;]
col_wise = [&#39;nullColumns_detection&#39;, &#39;IDness_detection&#39;, &#39;biasedness_detection&#39;, &#39;invalidEntries_detection&#39;,
&#39;outlier_detection&#39;]
row_wise_ = [p for p in row_wise if p in avl_recs_QC]
col_wise_ = [p for p in col_wise if p in avl_recs_QC]
len_row_wise = len([p for p in row_wise if p in avl_recs_QC])
len_col_wise = len([p for p in col_wise if p in avl_recs_QC])
if len_row_wise == 0:
c = data_analyzer_output(master_path, col_wise_, &#34;quality_checker&#34;)
for i in c:
for j in i:
if j == &#39;outlier_charts_placeholder&#39; and len(all_charts_num_3_) &gt; 1:
c_.append(dp.Select(blocks=all_charts_num_3_, type=dp.SelectType.DROPDOWN))
elif j == &#39;outlier_charts_placeholder&#39; and len(all_charts_num_3_) == 0:
c_.append(dp.Plot(blank_chart))
else:
c_.append(j)
report = dp.Group(dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section identifies the data quality issues at both row and column level.*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Group(*c_), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), rows=8, label=&#34;Quality Check&#34;)
elif len_col_wise == 0:
r = data_analyzer_output(master_path, row_wise_, &#34;quality_checker&#34;)
for i in r:
for j in i:
r_.append(j)
report = dp.Group(dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*This section identifies the data quality issues at both row and column level.*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Group(*r_), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), rows=8, label=&#34;Quality Check&#34;)
else:
c = data_analyzer_output(master_path, col_wise_, &#34;quality_checker&#34;)
for i in c:
for j in i:
if j == &#39;outlier_charts_placeholder&#39; and len(all_charts_num_3_) &gt; 1:
c_.append(dp.Select(blocks=all_charts_num_3_, type=dp.SelectType.DROPDOWN))
elif j == &#39;outlier_charts_placeholder&#39; and len(all_charts_num_3_) == 0:
c_.append(dp.Plot(blank_chart))
else:
c_.append(j)
r = data_analyzer_output(master_path, row_wise_, &#34;quality_checker&#34;)
for i in r:
for j in i:
r_.append(j)
report = dp.Group(
dp.Text(&#34;# &#34;), \
dp.Text(&#34;*This section identifies the data quality issues at both row and column level.*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Select(blocks=[
dp.Group(dp.Text(&#34;# &#34;), dp.Group(*c_), rows=2, label=&#34;Column Level&#34;), \
dp.Group(dp.Text(&#34;# &#34;), dp.Group(*r_), rows=2, label=&#34;Row Level&#34;)], \
type=dp.SelectType.TABS), \
dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), \
label=&#34;Quality Check&#34;)
if print_report:
dp.Report(default_template[0], default_template[1], report).save(
ends_with(master_path) + &#34;quality_check.html&#34;, open=True)
return report
```
</details>
</dd>
<dt id="anovos.data_report.report_generation.remove_u_score"><code class="name flex">
<span>def <span class="ident">remove_u_score</span></span>(<span>col)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>col</code></strong></dt>
<dd>Analysis column containing "_" present gets replaced along with upper case conversion</dd>
</dl>
<p>Returns:</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def remove_u_score(col):
&#34;&#34;&#34;
Args:
col: Analysis column containing &#34;_&#34; present gets replaced along with upper case conversion
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
```
</details>
</dd>
<dt id="anovos.data_report.report_generation.wiki_generator"><code class="name flex">
<span>def <span class="ident">wiki_generator</span></span>(<span>master_path, dataDict_path=None, metricDict_path=None, print_report=False)</span>
</code></dt>
<dd>
<div class="desc"><h2 id="args">Args</h2>
<dl>
<dt><strong><code>master_path</code></strong></dt>
<dd>Path containing the input files.</dd>
<dt><strong><code>dataDict_path</code></strong></dt>
<dd>Data dictionary path. Default value is kept as None.</dd>
<dt><strong><code>metricDict_path</code></strong></dt>
<dd>Metric dictionary path. Default value is kept as None.</dd>
<dt><strong><code>print_report</code></strong></dt>
<dd>Printing option flexibility. Default value is kept as False.</dd>
</dl>
<p>Returns:</p></div>
<details class="source">
<summary>
<span>Expand source code</span>
</summary>
```python
def wiki_generator(master_path, dataDict_path=None, metricDict_path=None, print_report=False):
&#34;&#34;&#34;
Args:
master_path: Path containing the input files.
dataDict_path: Data dictionary path. Default value is kept as None.
metricDict_path: Metric dictionary path. Default value is kept as None.
print_report: Printing option flexibility. Default value is kept as False.
Returns:
&#34;&#34;&#34;
try:
datatype_df = pd.read_csv(ends_with(master_path) + &#34;data_type.csv&#34;)
except:
datatype_df = pd.DataFrame(columns=[&#39;attribute&#39;, &#39;data_type&#39;], index=range(1))
try:
data_dict = pd.read_csv(dataDict_path).merge(datatype_df, how=&#34;outer&#34;, on=&#34;attribute&#34;)
except:
data_dict = datatype_df
try:
metric_dict = pd.read_csv(metricDict_path)
except:
metric_dict = pd.DataFrame(columns=[&#39;Section Category&#39;, &#39;Section Name&#39;, &#39;Metric Name&#39;, &#39;Metric Definitions&#39;],
index=range(1))
report = dp.Group(dp.Text(&#34;# &#34;), \
dp.Text(
&#34;*A quick reference to the attributes from the dataset (Data Dictionary) and the metrics computed in the report (Metric Dictionary).*&#34;), \
dp.Text(&#34;# &#34;), \
dp.Text(&#34;# &#34;), \
dp.Select(blocks= \
[dp.Group(dp.Group(dp.Text(&#34;## &#34;), dp.DataTable(data_dict)),
label=&#34;Data Dictionary&#34;), \
dp.Group(dp.Text(&#34;##&#34;), dp.DataTable(metric_dict), label=&#34;Metric Dictionary&#34;)], \
type=dp.SelectType.TABS), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;), dp.Text(&#34;# &#34;),
label=&#34;Wiki&#34;)
if print_report:
dp.Report(default_template[0], default_template[1], report).save(
ends_with(master_path) + &#34;wiki_generator.html&#34;, open=True)
return report
```
</details>
</dd>
</dl>