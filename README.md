### Small files problem: using SQL through Hive and Impala to combine files

Contents

	1. write about small file problem, general stuff
	2. write about downloading quick start
	3. explain how to do it.
We explain a simple method of dealing with the small files problem in HDFS when working with hive and impala. 
#### Small file problem
A _small file_ on HDFS is a file which is signifantly smaller than the default size of a file (typically 128 MB). To handle a large number of small files, namenode would require substantial memeory. 

> Because the namenode holds filesystem metadata in memory, the limit to the number of files in a filesystem is governed by the amount of memory on the namenode. As a rule of thumb, each file, directory, and block takes about 150 bytes. So, for example, if you had one million files, each taking one block, you would need at least 300 MB of memory. Although storing millions of files is feasible, billions is beyond the capability of current hardware.
[Hadoop The Definitive Guide](https://www.amazon.co.uk/Hadoop-Definitive-Guide-Tom-White) 

#### Simulating the problem
We will be using the [Cloudera QuickStart VM](https://www.cloudera.com/documentation/enterprise/latest/topics/cloudera_quickstart_vm.html) which has all required hadoop services.  You can download  the version of [QuickStarts Virtual Machine](https://www.cloudera.com/downloads/quickstart_vms/5-13.html) used in this  document or download any newly released version of the QuickStart. 

Create a directory called smallfiles to deposit the files

```
[cloudera@quickstart ~]$ hdfs dfs -ls /user/cloudera
[cloudera@quickstart ~]$ hdfs dfs -mkdir /user/cloudera/smallfiles
[cloudera@quickstart ~]$ hdfs dfs -ls /user/cloudera
Found 1 items
drwxr-xr-x   - cloudera cloudera          0 2018-09-08 03:19 /user/cloudera/smallfiles
```

Use sqoop to import a file. We deliberately create 15 small files with passing `-m 15`.

```
[cloudera@quickstart ~]$ sqoop import --connect jdbc:mysql://quickstart:3306/retail_db -username retail_dba --password cloudera --table customers --hive-table customers --hive-database mycustomers --warehouse-dir /user//cloudera/smallfiles -m 15
```

Let us check the file sizes in the directory .

![many small files])(./files/smallfiles.png)
