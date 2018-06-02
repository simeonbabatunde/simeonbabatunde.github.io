---
title: "How to modify/reset incremental.last.value in a sqoop job"
hide_sidebar: true
---

<p align="center">
  <img src="https://media-exp2.licdn.com/media/AAEAAQAAAAAAAAklAAAAJDUzYjBlMGM3LTg5NWItNDYxMS05OTZhLTUzZWUzYWMwYjUyYQ.jpg"/>
</p>


After creating a sqoop job for incremental import of data from sqlserver to hadoop, there comes a request to re-dump the data. My initial thought was, is it possible to reset the **incremental.last.value** in a sqoop job.

Though, flushing the previously ingested data in hadoop seems to be effortless using the command

**`hdfs dfs -rm /location/to/dir/part*`**

The caveat shows up when trying to execute the sqoop job again. The job executes successfully without pulling any record because it sees no increment in the number of records. So, instead of deleting the sqoop job and recreating a new one, the way to go is to reset the **incremental.last.value** to 0.

This can be achieved by changing the value of the last record in the sqoop metastore. The steps involves:
1.  Navigate to the home directory using **`cd ~`**
2.  Locate the sqoop metastore using **`ls -a`**
3.  You'll see a directory named *.sqoop*, cd into it with **`cd .sqoop`**
4.  Edit the file using the command **`vi metastore.db.script`** or **`nano metastore.db.script`**
5.  Scroll all the way down to see the details of the most recent job executed. Then locate the line with **'incremental.last.value','xxxxxx','SqoopOptions')** , where **xxxxxx** represents the last record pulled. Change the value to 0 or whatever number you want you next job execution to start with.
6.  Save the file and execute the sqoop job again, I'm sure you'll be doing just fine.

**NOTE**

If you decided to use timestamp rather than id, you will see something like this in step 5: **"'incremental.last.value','2013-01-01 00:00:00.0','SqoopOptions')"**

You can modify the date and time to your taste.
