## Module **ANOVOS.datetime**
Datetime module supports various transformations related to columns of date and timestamp type. All available functions in this release can be classified into the following 4 categories:

Conversion:
- Between Timestamp and Epoch (`timestamp_to_unix` and `unix_to_timestamp`)
- Between Timestamp and String (`timestamp_to_string` and `string_to_timestamp`)
- Between Date Formats (`dateformat_conversion`)
- Between Time Zones (`timezone_conversion`)

Calculation:
- Time difference - [Timestamp 1 - Timestamp 2] (`time_diff`)
- Time elapsed - [Current - Given Timestamp] (`time_elapsed`)
- Adding/subtracting time units (`adding_timeUnits`)
- Aggregate features at X granularity level (`aggregator`)
- Aggregate features with window frame (`window_aggregator`)
- Lagged features - lagged date and time diff from the lagged date (`lagged_ts`)

Extraction:
- Time component extraction (`timeUnits_extraction`)
- Start/end of month/year/quarter (`start_of_month`, `end_of_month`, `start_of_year`, `end_of_year`, `start_of_quarter` and `end_of_quarter`)

Binary features:
- Timestamp comparison (`timestamp_comparison`)
- Is start/end of month/year/quarter nor not (`is_monthStart`, `is_monthEnd`, `is_yearStart`, `is_yearEnd`, `is_quarterStart`, `is_quarterEnd`)
- Is first half of the year/selected hours/leap year/weekend or not (`is_yearFirstHalf`, `is_selectedHour`, `is_leapYear` and `is_weekend`)

Columns which are subjected to these analysis can be controlled by arguments list_of_cols. Most functions have the following common arguments:

- *idf*: Input dataframe
- *list_of_cols*: This argument, in a list format, is used to specify the columns which are subjected to the analysis in the input dataframe. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”.
- *output_mode*: "replace" or "append". “replace” option replaces original columns with transformed column, whereas “append” option append transformed column to the input dataset. 



### timestamp_to_unix
Convert timestamp columns in a specified time zone to Unix time stamp in seconds or milliseconds.
- *spark*: Spark Session
- *idf*
- *list_of_cols*
- *precision*: "ms", "s". "ms" option returns the number of milliseconds from the unix epoch (1970-01-01 00:00:00 UTC). "s" option returns the number of seconds from the unix epoch.
- *tz*: "local", "gmt", "utc". Timezone of the input column(s)
- *output_mode*: "replace" (by default) or "append".
 

### unix_to_timestamp
Convert the number of seconds or milliseconds from unix epoch (1970-01-01 00:00:00 UTC) to a timestamp column in the specified time zone.
- *spark*: Spark Session
- *idf*
- *list_of_cols*
- *precision*: "ms", "s". "ms" treats the input columns as the number of milliseconds from the unix epoch (1970-01-01 00:00:00 UTC). "s" treats the input columns as the number of seconds from the unix epoch.
- *tz*: "local", "gmt", "utc". Timezone of the output column(s)
- *output_mode*: "replace" (by default) or "append".
        
### timezone_conversion
Convert timestamp columns from the given timezone (given_tz) to the output timezone (output_tz).
- *spark*: Spark Session
- *idf*
- *list_of_cols*
- *given_tz*: Timezone of the input column(s). If "local", the timezone of the spark session will be used.
- *output_tz*: Timezone of the output column(s). If "local", the timezone of the spark session will be used.
- *output_mode*: "replace" (by default) or "append".
        
      
### string_to_timestamp
Convert time string columns with given input format ("%Y-%m-%d %H:%M:%S", by default) to TimestampType or DateType columns.
- *spark*: Spark Session
- *idf*
- *list_of_cols*
- *input_format*: Format of the input column(s) in string.
- *output_type*: "ts", "dt". "ts" option returns result in T.TimestampType(). "dt" option returns result in T.DateType()
- *output_mode*: "replace" (by default) or "append".
        
      
### timestamp_to_string
Convert timestamp/date columns to time string columns with given output format ("%Y-%m-%d %H:%M:%S", by default)
- *spark*: Spark Session
- *idf*
- *list_of_cols* Columns must be of Datetime type or String type in "%Y-%m-%d %H:%M:%S" format.
- *output_format*: Format of the output column(s)
- *output_mode*: "replace" (by default) or "append".
        
      
### dateformat_conversion
Convert time string columns with given input format ("%Y-%m-%d %H:%M:%S", by default) to time string columns with given output format ("%Y-%m-%d %H:%M:%S", by default).
- *spark*: Spark Session
- *idf*
- *list_of_cols*
- *input_format*: Format of the input column(s) in string
- *output_format*: Format of the output column(s) in string
- *output_mode*: "replace" (by default) or "append".


### timeUnits_extraction
Extract the unit(s) of given timestamp columns as integer. Currently the following units are supported: hour, minute, second, dayofmonth, dayofweek, dayofyear, weekofyear, month, quarter, year. Multiple units can be calculated at the same time by inputting a list of units or a string of units separated by pipe delimiter “|”.
- *spark*: Spark Session
- *idf*
- *list_of_cols*
- *units*: List of unit(s) to extract. Alternatively, unit(s) can be specified in a string format, where different units are separated by pipe delimiter “|” e.g., "hour|minute". Supported units to extract: 'hour', 'minute', 'second', 'dayofmonth', 'dayofweek', 'dayofyear', 'weekofyear', 'month', 'quarter', 'year'. 'all' can be passed to compute all supported metrics.
- *output_mode*: "replace" or "append" (by default).
        

### time_diff
Calculate the time difference between 2 timestamp columns (Timestamp 1 - Timestamp 2) in a given unit. Currently the following units are supported: second, minute, hour, day, week, month, year.
- *idf*
- *ts1*, *ts2*: The two columns to calculate the difference between.
- *unit*: 'second', 'minute', 'hour', 'day', 'week', 'month', 'year'. Unit of the output values.
- *output_mode*: "replace" or "append" (by default).
        
### time_elapsed
Calculate time difference between the current and the given timestamp (Current - Given Timestamp) in a given unit. Currently the following units are supported: second, minute, hour, day, week, month, year.
- *spark*: Spark Session
- *idf*
- *list_of_cols*
- *unit*: 'second', 'minute', 'hour', 'day', 'week', 'month', 'year'. Unit of the output values.
- *output_mode*: "replace" or "append" (by default).
 
### adding_timeUnits
Add or subtract given time units to/from timestamp columns. Currently the following units are supported: second, minute, hour, day, week, month, year. Subtraction can be performed by setting a negative unit_value.
- *spark*: Spark Session
- *idf*
- *list_of_cols*
- *unit*: 'hour','minute','second','day','week','month','year'. Unit of the added value.
- *unit_value*: The value to be added to input column(s).
- *output_mode*: "replace" or "append" (by default).


### timestamp_comparison
Compare timestamp columns with a given timestamp/date value (comparison_value) of given format (comparison_format). Supported comparison types include greater_than, less_than, greaterThan_equalTo and lessThan_equalTo. The derived values are 1 if True and 0 if False.
- *spark*: Spark Session
- *idf*
- *list_of_cols*
- *comparison_type*: "greater_than", "less_than", "greaterThan_equalTo", "lessThan_equalTo". The comparison type of the transformation.
- *comparison_value*: The timestamp / date value to compare with in string.
- *comparison_format*: The format of comparison_value in string.
- *output_mode*: "replace" or "append" (by default).


### start_of_month
Extract the first day of the month of given timestamp/date columns.
- *idf*
- *list_of_cols*
- *output_mode*: "replace" or "append" (by default).
 
### is_monthStart
Check if values in given timestamp/date columns are the first day of a month. The derived values are 1 if True and 0 if False.
- *idf*
- *list_of_cols*
- *output_mode*: "replace" or "append" (by default).

### end_of_month
Extract the last day of the month of given timestamp/date columns.
- *idf*
- *list_of_cols*
- *output_mode*: "replace" or "append" (by default).
 
### is_monthEnd
Check if values in given timestamp/date columns are the last day of a month. The derived values are 1 if True and 0 if False.
- *idf*
- *list_of_cols*
- *output_mode*: "replace" or "append" (by default).

### start_of_year
Extract the first day of the year of given timestamp/date columns.
- *idf*
- *list_of_cols*
- *output_mode*: "replace" or "append" (by default).
 
### is_yearStart
Check if values in given timestamp/date columns are the first day of a year. The derived values are 1 if True and 0 if False.
- *idf*
- *list_of_cols*
- *output_mode*: "replace" or "append" (by default).

### end_of_year
Extract the last day of the year of given timestamp/date columns.
- *idf*
- *list_of_cols*
- *output_mode*: "replace" or "append" (by default).
 
### is_yearEnd
Check if values in given timestamp/date columns are the last day of a year. The derived values are 1 if True and 0 if False.
- *idf*
- *list_of_cols*
- *output_mode*: "replace" or "append" (by default).

### start_of_quarter
Extract the first day of the quarter of given timestamp/date columns.
- *idf*
- *list_of_cols*
- *output_mode*: "replace" or "append" (by default).
 
### is_quarterStart
Check if values in given timestamp/date columns are the first day of a quarter. The derived values are 1 if True and 0 if False.
- *idf*
- *list_of_cols*
- *output_mode*: "replace" or "append" (by default).

### end_of_quarter
Extract the last day of the quarter of given timestamp/date columns.
- *idf*
- *list_of_cols*
- *output_mode*: "replace" or "append" (by default).

### is_quarterEnd
Check if values in given timestamp/date columns are the last day of a quarter. The derived values are 1 if True and 0 if False.
- *idf*
- *list_of_cols*
- *output_mode*: "replace" or "append" (by default).

### is_yearFirstHalf
Check if values in given timestamp/date columns are in the first half of a year. The derived values are 1 if True and 0 if False.
- *idf*
- *list_of_cols*
- *output_mode*: "replace" or "append" (by default).

### is_selectedHour
Check if the hour component of given timestamp columns are between start hour (inclusive) and end hour (inclusive). The derived values are 1 if True and 0 if False. Start hour can be larger than end hour, for example, start_hour=22 and end_hour=3 can be used to check whether the hour component is in [22, 23, 0, 1, 2, 3].
- *idf*
- *list_of_cols*
- *start_hour*: the starting hour of the hour range (inclusive)
- *end_hour*: the ending hour of the hour range (inclusive)
- *output_mode*: "replace" or "append" (by default).

### is_leapYear
Check if values in given timestamp/date columns are in a leap year. The derived values are 1 if True and 0 if False.
- *idf*
- *list_of_cols*
- *output_mode*: "replace" or "append" (by default).

### is_weekend
Check if values in given timestamp/date columns are on weekends. The derived values are 1 if True and 0 if False.
- *idf*
- *list_of_cols*
- *output_mode*: "replace" or "append" (by default).



### aggregator
aggregator performs groupBy over the timestamp/date column and calcuates a list of aggregate metrics over all input columns. The timestamp column is firstly converted to the given granularity format ("%Y-%m-%d", by default) before applying groupBy and the conversion step can be skipped by setting granularity format to be an empty string.

The following aggregate metrics are supported: count, min, max, sum, mean, median, stddev, countDistinct, sumDistinct, collect_list, collect_set.
- *idf*
- *list_of_cols*
- *list_of_aggs*: List of aggregate metrics to compute e.g., ["f1","f2"]. Alternatively, metrics can be specified in a string format, where different metrics are separated by pipe delimiter “|” e.g., "f1|f2". Supported metrics: 'count', 'min', 'max', 'sum', 'mean', 'median', 'stddev' 'countDistinct', 'sumDistinct', 'collect_list', 'collect_set'.
- *time_col*: (Timestamp) Column to group by.
- *granularity_format*: Format to be applied to time_col before groupBy. The default value is '%Y-%m-%d', which means grouping by the date component of time_col. Alternatively, '' can be used if no formatting is necessary.


### window_aggregator
window_aggregator calcuates a list of aggregate metrics for all input columns over a window frame (expanding by default, or rolling type) ordered by the given timestamp column and partitioned by partition_col ("" by default, to indicate no partition). 

Window size needs to be provided as an integer for rolling window type. The following aggregate metrics are supported: count, min, max, sum, mean, median.
- *idf*
- *list_of_cols*
- *list_of_aggs*: List of aggregate metrics to compute e.g., ["f1","f2"]. Alternatively, metrics can be specified in a string format, where different metrics are separated by pipe delimiter “|” e.g., "f1|f2". Supported metrics: 'count','min','max','sum','mean','median'
- *order_col*: (Timestamp) Column to order window
- *window_type*: "expanding", "rolling". "expanding" option have a fixed lower bound (first row in the partition). "rolling" option have a fixed window size defined by the *window_size* param.
- *window_size*: window size for rolling window type. Integer value with value >= 1.
- *partition_col*: Rows partitioned by this column before creating window.
- *output_mode*: "replace" or "append" (by default).



### lagged_ts
lagged_ts returns the values that are *lag* rows before the current rows, and None if there is less than *lag* rows before the current rows. If output_type is "ts_diff", an additional column is generated with values being the time difference between the original timestamp and the lagged timestamp in given unit *tsdiff_unit*. Currently the following units are supported: second, minute, hour, day, week, month, year.
- *spark*: Spark Session
- *idf*
- *list_of_cols*
- *lag*: Integer - number of row(s) to extend.
- *output_type*: "ts", "ts_diff". "ts" option generats a lag column for each input column having the value that is <lag> rows before the current row, and None if there is less than <lag> rows before the current row. "ts_diff" option generates the lag column in the same way as the "ts" option. On top of that, it appends a column which represents the time_diff between the original and the lag column.
- *tsdiff_unit*: 'second', 'minute', 'hour', 'day', 'week', 'month', 'year'. Unit of the time_diff if output_type="ts_diff".
- *partition_col*: Rows partitioned by this column before creating window.
- *output_mode*: "replace" or "append" (by default).

