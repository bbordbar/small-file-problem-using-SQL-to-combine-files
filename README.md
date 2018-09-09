### Small files problem in Hive and Impala: using SQL to combine files

When working with Hive and Impala tables sometimes large number of small files are created. A quick way of consolidating the files is to run the following query. 

```
insert overwrite table_name select * from table_name;
```

 
#### Small file problem
A _small file_ on HDFS is a file which is significantly smaller than the default size of a file (typically 128 MB). To handle a large number of small files, namenode would require substantial memory. 

> Because the namenode holds filesystem metadata in memory, the limit to the number of files in a filesystem is governed by the amount of memory on the namenode. As a rule of thumb, each file, directory, and block takes about 150 bytes. So, for example, if you had one million files, each taking one block, you would need at least 300 MB of memory. Although storing millions of files is feasible, billions is beyond the capability of current hardware.
[Hadoop The Definitive Guide](https://www.amazon.co.uk/Hadoop-Definitive-Guide-Tom-White) 

#### Simulating the problem
We will be using the [Cloudera QuickStart VM](https://www.cloudera.com/documentation/enterprise/latest/topics/cloudera_quickstart_vm.html) which has all required hadoop services such as Hive, Impala, and Hue pre-installed.  You can download  the version of [QuickStarts Virtual Machine](https://www.cloudera.com/downloads/quickstart_vms/5-13.html) used in this  document or download any newly released version of the QuickStart. 

For the following you can use Beeline shell, Impala shell or Hue. I will use Impala shell. Invoke the Impala shell with `impala-shell` command and create a database called `smallfilesdb` as follows. 

![creating a database called smallfilesdb](/files/createdatabase.png) 

Use sqoop to import a file. The following will sqoop the mysql table `customers` and create a hive table with the name `customers` inside the database `smallfilesdb`. We deliberately create 15 small files with passing `-m 15`.

```
[cloudera@quickstart ~]$ sqoop import  \
 --connect jdbc:mysql://quickstart:3306/retail_db \
 --username retail_dba  \
--password cloudera \
 --table customers \
 --hive-import \
 --hive-table customers \
 --hive-database smallfilesdb \
 -m 15
```

Validate to make sure that the table is created, either in Impala shell, Beeline shell or Hue. Don't forget to run `invalidate metadata` first if you working with Impala. 

![validate table is create](/files/validate.png)

Check the size of the created files. You can run `describe formatted customers` to find the location of the files for the Hive/Impala table `customers`. In the following
`-du -h` is for disk usage in human readable format. We can see a large number of smile files in the directory: ![many small files](/files/smallfiles.png) 
To consolidate the files and get rid of the small files  in the `customers` table run the following query in the Impala shell. 

```
insert overwrite customers select * from customers;
```

We can see there is only one large file. 
![no small files](/files/nosmallfiles.png)



