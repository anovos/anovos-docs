## *Module* **ANOVOS.data_ingest**					
This module consists of functions to read the dataset as Spark Dataframe, concatenate/join with other functions (if required), and perform some basic ETL actions such as selecting, deleting, renaming and/or recasting columns. List of functions included in this module are: 
- read_dataset 
- write_dataset 
- concatenate_dataset 
- join_dataset 
- delete_column 
- select_column 
- rename_column 
- recast_column 

### read_dataset 
This function reads the input data path and return a Spark Dataframe. Under the hood, this function is based on generic Load functionality of Spark SQL1.  It requires following arguments: 

- **file_path**: file (or directory) path where the input data is saved. File path can be a local path or s3 path (when running with AWS cloud services) 
- **file_type**: file format of the input data. Currently, we support csv, parquet or avro. Avro data source requires an external package to run, which can be configured with spark-submit options (--packages org.apache.spark:spark-avro_2.11:2.4.0). 
- **file_configs** (optional): Rest of the valid configurations can be passed through this argument in a dictionary format. All the key/value pairs written in this argument are passed as options to DataFrameReader, which is created using SparkSession.read. 

### write_dataset
This function saves Spark Dataframe in the user provided output path. Like read_dataset, this function is based on generic Save functionality of Spark SQL.  It requires following arguments: 

- **idf**: Spark Dataframe to be saved 
- **file_path**: file (or directory) path where the output data is to be saved. File path can be a local path or s3 path (when running with AWS cloud services) 
- **file_type**: file format of the input data. Currently, we support csv, parquet or avro. Avro data source requires an external package to run, which can be configured with spark-submit options (--packages org.apache.spark:spark-avro_2.11:2.4.0). 
- **file_configs** (optional): Rest of the valid configuration can be passed through this argument in a dictionary format e.g., repartition, mode, compression, header, delimiter etc. All the key/value pairs written in this argument are passed as options to DataFrameWriter is available using Dataset.write operator. If number of repartitions mentioned through this argument is less than the existing Dataframe partitions, then coalesce operation2 is used instead of repartition operation to make the execution efficient. This is because the coalesce operation doesn’t require any shuffling like repartition which is known to be an expensive step. 

### concatenate_dataset 
This function combines multiple dataframes into a single dataframe. To make the operation efficient, pairwise concatenation is done on the dataframes, instead of adding one dataframe at a time to the bigger dataframe. This function is leveraging union3 functionality of Spark SQL. It requires following arguments: 

- ***idfs**: Varying number of dataframes to be concatenated 
- **method_type**: index or name. This argument needs to be entered as a keyword argument. “index” method involves concatenating the dataframes by column index. In case, sequence of column is not fixed among the dataframe, this method should be avoided. “name” method involves concatenating by columns names. 1st dataframe passed under idfs will define the final columns in the concatenated dataframe and it will throw error if any column in 1st dataframe is not available in any of other dataframes. 

### join_dataset 

This function joins multiple dataframes into a single dataframe by a joining key column. To make the operation efficient, pairwise joining is done on the dataframes, instead of joining one dataframe at a time to the bigger dataframe. This function is leveraging join4 functionality of Spark SQL. It requires following arguments: 

- ***idfs**: Varying number of all dataframes to be joined 
- **join_cols**: Key column(s) to join all dataframes together. In case of multiple columns to join, they can be passed in a list format or a single text format where different column names are separated by pipe delimiter “|” 
- **join_type**: “inner”, “full”, “left”, “right”, “left_semi”, “left_anti”

### delete_column 

This function is used to delete specific columns from the input data. It is executed using drop functionality5 of Spark SQL. It is advisable to use this function if number of columns to delete are lesser than number of columns to select, otherwise it is recommended to use select_column. It requires following arguments: 

- **idf**: Input dataframe 
- **list_of_cols**: This argument, in a list format, is used to specify the columns which are required to be deleted from the input dataframe. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|” 
- **print_impact**: This argument is to compare number of columns before and after the operation. 

### select_column 

This function is used to select specific columns from the input data. It is executed using select operation6 of spark dataframe. It is advisable to use this function if number of columns to select are lesser than number of columns to drop, otherwise it is recommended to use delete_column. It requires following arguments: 

- **idf**: Input dataframe 
- **list_of_cols**: This argument, in a list format, is used to specify the columns which are required to be deleted from the input dataframe. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|” 
- **print_impact**: This argument is to compare number of columns before and after the operation. 

### rename_column 

This function is used to rename columns of the input data. Multiple columns can be renamed, however, sequence in which they passed as argument is critical and needs to be consistent between list_of_cols and list_of_newcols. It requires following arguments: 

- **idf**: Input dataframe 
- **list_of_cols**: This argument, in a list format, is used to specify the columns which are required to be renamed in the input dataframe. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|” 
- **list_of_newcols**: This argument, in a list format, is used to specify the new column name i.e. first element in list_of_cols will be original column name and corresponding first column in list_of_newcols will be new column name. 
- **print_impact**: This argument is to compare column names before and after the operation. 

### recast_column 

This function is used to modify the datatype of columns. Multiple columns can be casted, however, sequence in which they passed as argument is critical and needs to be consistent between list_of_cols and list_of_dtypes. It requires following arguments: 

- **idf**: Input dataframe 
- **list_of_cols**: This argument, in a list format, is used to specify the columns which are required to be recast in the input dataframe. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|” 
- **list_of_dtypes**: This argument, in a list format, is used to specify the datatype i.e. first element in list_of_cols will column name and corresponding element in list_of_dtypes will be new datatype such as float, integer, string, double, decimal etc. (case insensitive). 
- **print_impact**: This argument is to compare schema before and after the operation. 



