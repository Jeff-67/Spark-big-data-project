# Abstract
Managing the yield of wafer is one of the most important tasks to the semiconductor manufacturers. A lot of efforts for enhancing the yield of wafer have been conducted in both industries and academia. Thanks to the advance of IoT and data analytics techniques, huge amount of process operational data, such as indices of process parameters, equipment condition data, or historical data of manufacturing process, is collected and analyzed in realtime. In this project I will analyze the big data of Wafer manufaturing for troubleshooting and fault detection by using PySpark in several steps.
# Guide line
> [Wafer processing introduction](#wafer-processing-introduction) 

> [Spark and Hadoop introduction](#spark-and-hadoop-introduction)

> [Wafer processing Data introduction](#wafer-processing-data-introduction) 

> [Data analyze](#data-analyze)

- Pearson Correlation
  
- Box plot(by median gap)
  
- PLSR

> [Result](#result)

> [Conclusion](#conclusion)

## Wafer processing introduction
Semiconductor manufacturing is one of the most complex works that has hundreds of process steps, several kinds of wafers, machinery, re-entrant flow, and innumerable process parameters, so it takes few months for completing the whole processes accordingly. Also, since semiconductor manufacturing process is very sensitive on stream, yield management is one of the most important issues directly connected to survival of a company.Here I briefly introduced the manufacturing process of wafer.

- Wafer Manufaturing
    
    The profound process of fabricating silicon wafers with IC design by masks and semiconductors processing machine tools.
    
    ![screen shot 2018-11-26 at 7 17 55 pm](https://user-images.githubusercontent.com/36265245/49011072-28738e80-f1b0-11e8-92b1-e4d77744a9ea.png)
    
- Wafer Processing
  - Wafer fabrication
  
      The process is roughly divided into several stations, each of which corresponds to several stages. Each stage corresponds to a certain type of machine processing (there are several machines of various types in the factory). A type of machine will be used in many stages and all the wafer must be processed in sequence from start to finish.
  - Wafer probe
  
      A wafer prober is a machine used to test integrated circuits. In this project, we will foucus on the parameter called WAT(Wafer Acceptance Test)
 
  ![screen shot 2018-11-26 at 7 30 15 pm](https://user-images.githubusercontent.com/36265245/49011603-d03d8c00-f1b1-11e8-917f-974b4f5daa9b.png)

## Spark and Hadoop introduction
- What is Spark and Hadoop?

  Hadoop is a extremely powerful tool for distributed, scalable and economical data storage, processing and analysis. The data of semiconductor manufaturing is really tremendous thus I choose Hadoop to store them and use Pyspark to analyze.
  
  ![screen shot 2018-11-27 at 1 36 43 pm](https://user-images.githubusercontent.com/36265245/49060681-8607e980-f249-11e8-9d95-c20c73de7c78.png)![screen shot 2018-11-27 at 1 38 55 pm](https://user-images.githubusercontent.com/36265245/49060756-d41ced00-f249-11e8-8a58-7e3a947ec583.png)

- Simple command

  > Copy file data.csv from local disk to the user’s directory in HDFS

  ```shell
  $ hdfs dfs –put data.csv data.csv
  ```
  > Get a directory listing of the user’s home directory in HDFS

  ```shell
  $ hdfs dfs ls
  ```
  > Display the contents of the HDFS file /user/semiconductor/data.csv

  ```shell
  $ hdfs dfs –cat /user/semiconductor/data.csv
  ```
- Set up Pyspark environment

  ```python

  #!/usr/bin/env python
  # coding: utf-8

  # In[ ]:


  import os
  import sys

  #this part is used for pyspark submit
  os.environ['PYSPARK_SUBMIT_ARGS']='--verbose --master=yarn --queue test pyspark-shell'

  os.environ['JAVA_HOME']='/usr/lib/jvm/java-8-openjdk-amd64/'
  os.environ['YARN_CONF_DIR']='/etc/alternatives/hadoop-conf/'

  #this line is used for spark1.6
  #os.environ['SPARK_HOME']='/opt/cloudera/parcels/CDH/lib/spark'

  #this line is used for spark2.2
  os.environ['SPARK_HOME']='/opt/cloudera/parcels/SPARK2-2.2.0.cloudera2-1.cdh5.12.0.p0.232957/lib/spark2'

  # this line is used for python2.7
  #os.environ['PYSPARK_PYTHON']='/usr/bin/python'

  #this line is used for python3.5
  os.environ['PYSPARK_PYTHON']='/usr/bin/python3'

  spark_home = os.environ.get('SPARK_HOME', None)
  sys.path.insert(0, os.path.join(spark_home, 'python'))
  sys.path.insert(0, os.path.join(spark_home, 'python/lib/py4j-0.10.4-src.zip'))  
  #execfile(os.path.join(spark_home, 'python/pyspark/shell.py'))
  exec(open(os.path.join(spark_home, 'python/pyspark/shell.py')).read())
  ```
## Wafer processing Data introduction 
Let's recall our misson again: 
 
Find out the relevant factors which affect yield; therefore, we can delete some irrelevant data manually at first.

> In the directory wat_data:

| <a>**wat_data/Parameter_set**</a> | <a>**wat_data/Wat_root_cause**</a> 
| :---: |:---:| 
|![screen shot 2018-11-27 at 2 41 15 pm](https://user-images.githubusercontent.com/36265245/49063500-8f498400-f252-11e8-9201-8a957bea8896.png)    | ![screen shot 2018-11-27 at 2 41 26 pm](https://user-images.githubusercontent.com/36265245/49063501-907ab100-f252-11e8-9e6a-c0e591eaa604.png)

  The aa column in Parameter_set are arbitrary numbers and so does the Range column. The other column in wat_root_cause isn't really meaningful to the process as well. 

### Here we move to the two of most important data

> In the directory wat_data/wat:

   Just like we mentioned, WAT is a test key whick will affect yield.

| <a>**wat.header**</a> | <a>**wat.first_raw**</a> 
| :---: |:---:| 
|![screen shot 2018-11-27 at 3 00 05 pm](https://user-images.githubusercontent.com/36265245/49065321-a68b7000-f258-11e8-8450-1a6dda26a69a.png)    | ![screen shot 2018-11-27 at 3 00 19 pm](https://user-images.githubusercontent.com/36265245/49065322-a7240680-f258-11e8-9f8d-019da7aabcda.png)

> In the directory FDC_data/stageXX:

   SVID(Status Variables Identification) is the physical data collected by sensors embedded in the advanced machines during the manufacturing process. To state the physical nature of certain SVID, we usually transform SVID into Fault Detection and Classification parameters (FDC parameters) using statistical indicators.

| <a>**stageXX.header and stageXX.row**</a> | 
| :---: |
|![screen shot 2018-11-27 at 3 25 38 pm](https://user-images.githubusercontent.com/36265245/49066944-99bd4b00-f25d-11e8-877c-27e96531ee6d.png)   | 

  From [this picture](#wafer-processing-introduction#-wafer-processing) we will know what those columns(toolid, chamberid, process, stage) represent.


## Data analyze
## Result
## Conclusion
