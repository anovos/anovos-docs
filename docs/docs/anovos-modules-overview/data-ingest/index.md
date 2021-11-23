## *Module* **ANOVOS.data_ingest**					
This module consists of functions to read the dataset as Spark DataFrame, concatenate/join with other functions (if required), and perform some basic ETL actions such as selecting, deleting, renaming and/or recasting columns. List of functions included in this module are: 
- read_dataset 
- write_dataset 
- concatenate_dataset 
- join_dataset 
- delete_column 
- select_column 
- rename_column 
- recast_column 

### read_dataset 
This function reads the input data path and return a Spark DataFrame. Under the hood, this function is based on generic Load functionality of Spark SQL1.  It requires following arguments: 

- **file_path**: file (or directory) path where the input data is saved. File path can be a local path or s3 path (when running with AWS cloud services) 
- **file_type**: file format of the input data. Currently, we support CSV, Parquet or Avro. Avro data source requires an external package to run, which can be configured with spark-submit options (--packages org.apache.spark:spark-avro_2.11:2.4.0). 
- **file_configs** (optional): Rest of the valid configurations can be passed through this argument in a dictionary format. All the key/value pairs written in this argument are passed as options to DataFrameReader, which is created using SparkSession.read. 

### write_dataset
This function saves the Spark DataFrame in the user-provided output path. Like read_dataset, this function is based on the generic Save functionality of Spark SQL.  It requires the following arguments: 

- **idf**: Spark DataFrame to be saved 
- **file_path**: file (or directory) path where the output data is to be saved. File path can be a local path or s3 path (when running with AWS cloud services) 
- **file_type**: file format of the input data. Currently, we support CSV, Parquet or Avro. The Avro data source requires an external package to run, which can be configured with spark-submit options (--packages org.apache.spark:spark-avro_2.11:2.4.0). 
- **file_configs** (optional): The rest of the valid configuration can be passed through this argument in a dictionary format, e.g., repartition, mode, compression, header, delimiter, etc. All the key/value pairs written in this argument are passed as options to DataFrameWriter is available using Dataset.write operator. If the number of repartitions mentioned through this argument is less than the existing DataFrame partitions, then the coalesce operation2 is used instead of the repartition operation to make the execution work. This is because the coalesce operation doesn’t  

### concatenate_dataset 
This function combines multiple dataframes into a single dataframe. A pairwise concatenation is performed on the dataframes, instead of adding one dataframe at a time to the bigger dataframe. This function leverages union3 functionality of Spark SQL. It requires the following arguments:

- ***idfs**: Varying number of dataframes to be concatenated 
- **method_type**: index or name. This argument needs to be entered as a keyword argument. The “index” method involves concatenating the dataframes by the column index. IF the sequence of column is not fixed among the dataframe, this method should be avoided. The “name” method involves concatenating by columns names. The 1st dataframe passed under idfs will define the final columns in the concatenated dataframe. It will throw an error if any column in the 1st dataframe is not available in any of other dataframes. 

### join_dataset 

This function joins multiple dataframes into a single dataframe by a joining key column. Pairwise joining is done on the dataframes, instead of joining individual dataframes to the bigger dataframe. This function leverages join4 functionality of Spark SQL. It requires the following arguments:

- ***idfs**: Varying number of all dataframes to be joined 
- **join_cols**: Key column(s) to join all dataframes together. In case of multiple columns to join, they can be passed in a list format or a single text format where different column names are separated by pipe delimiter “|” 
- **join_type**: “inner”, “full”, “left”, “right”, “left_semi”, “left_anti”

### delete_column 

This function is used to delete specific columns from the input data. It is executed using drop functionality5 of Spark SQL. It is advisable to use this function if the number of columns to delete is lesser than the number of columns to select; otherwise, it is recommended to use select_column. It requires the following arguments:

- **idf**: Input dataframe 
- **list_of_cols**: This argument, in a list format, specifies the columns required to be deleted from the input dataframe. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”
- **print_impact**: This argument is to compare number of columns before and after the operation. 

### select_column 

This function is used to select specific columns from the input data. It is executed using select operation6 of spark dataframe. It is advisable to use this function if the number of columns to select is lesser than the number of columns to drop; otherwise, it is recommended to use delete_column. It requires the following arguments:

- **idf**: Input dataframe 
- **list_of_cols**: This argument, in a list format, specifies the columns required to be deleted from the input dataframe. Alternatively, instead of list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”
- **print_impact**: This argument is to compare number of columns before and after the operation. 

### rename_column 

This function is used to rename the columns of the input data. Multiple columns can be renamed; however, the sequence they passed as an argument is critical and must be consistent between list_of_cols and list_of_newcols. It requires the following arguments:

- **idf**: Input dataframe 
- **list_of_cols**: This argument, in a list format, is used to specify the columns required to be renamed in the input dataframe. Alternatively, instead of a list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”
- **list_of_newcols**: This argument, in a list format, is used to specify the new column name, i.e. the first element in list_of_cols will be the original column name, and the corresponding first column in list_of_newcols will be the new column name.
- **print_impact**: This argument is to compare column names before and after the operation. 

### recast_column 

This function is used to modify the datatype of columns. Multiple columns can be cast; however, the sequence they passed as argument is critical and needs to be consistent between list_of_cols and list_of_dtypes. It requires the following arguments:

- **idf**: Input dataframe 
- **list_of_cols**: This argument, in a list format, is used to specify the columns required to be recast in the input dataframe. Alternatively, instead of a list, columns can be specified in a single text format where different column names are separated by pipe delimiter “|”
- **list_of_dtypes**: This argument, in a list format, is used to specify the datatype, i.e. the first element in list_of_cols will column name, and the corresponding element in list_of_dtypes will be new datatype such as float, integer, string, double, decimal, etc. (case insensitive).
- **print_impact**: This argument is to compare schema before and after the operation. 



