**Spark-overflow**
===================

A collection of Spark related information, solutions, debugging tips and tricks, etc. PR are always welcome! Share what you know about Apache Spark.


# **Knowledge**
### Spark executor memory([Reference Link](http://www.slideshare.net/AGrishchenko/apache-spark-architecture/57))
  <img src='http://image.slidesharecdn.com/sparkarchitecture-jdkievv04-151107124046-lva1-app6892/95/apache-spark-architecture-57-638.jpg?cb=1446900275'/>

### spark-submit --verbose([Reference Link](http://www.slideshare.net/jcmia1/a-beginners-guide-on-troubleshooting-spark-applications?qid=25ed4f3f-fc2e-43b2-bc8a-7f78b21bdebb&v=&b=&from_search=34))
  - Always add `--verbose options` on `spark-submit` to print the following information:
    - All default properties.
    - Command line options.
    - Settings from spark conf file.

### Spark Executor on YARN([Reference Link](http://www.slideshare.net/AmazonWebServices/bdt309-data-science-best-practices-for-apache-spark-on-amazon-emr))
  Following is the memory relation config on YARN:
  - YARN container size - `yarn.nodemanager.resource.memory-mb`.
  - Memory Overhead - `spark.yarn.executor.memoryOverhead`.

  <img src='http://image.slidesharecdn.com/bdt309-151009173030-lva1-app6891/95/bdt309-data-science-best-practices-for-apache-spark-on-amazon-emr-49-638.jpg'/>

  - An example on how to set up Yarn and launch spark jobs to use a specific numbers of executors ([Reference Link](http://stackoverflow.com/questions/29940711/apache-spark-setting-executor-instances-does-not-change-the-executors))  

# **Tunnings**
### Tune the shuffle partitions
  - Tune the numbers of `spark.sql.shuffle.partitions`.

### Avoid using jets3t 1.9([Reference Link](http://www.slideshare.net/databricks/spark-summit-eu-2015-lessons-from-300-production-users))
  - It's a jar default on Hadoop 2.0.
  - Inexplicably terrible performance.

### Use `reduceBykey()` instead of `groupByKey()`
  - reduceByKey

<img src='http://image.slidesharecdn.com/stratasj-everydayimshuffling-tipsforwritingbettersparkprograms-150223113317-conversion-gate02/95/everyday-im-shuffling-tips-for-writing-better-spark-programs-strata-san-jose-2015-9-638.jpg'/>

  - groupByKey

<img src='http://image.slidesharecdn.com/stratasj-everydayimshuffling-tipsforwritingbettersparkprograms-150223113317-conversion-gate02/95/everyday-im-shuffling-tips-for-writing-better-spark-programs-strata-san-jose-2015-10-638.jpg'/>

### GC policy([Reference Link](https://databricks.com/blog/2015/05/28/tuning-java-garbage-collection-for-spark-applications.html))
  - G1GC is a new feature that you can use.
  - Used by -XX:+UseG1GC.

### Join a large Table with a small table([Reference Link](http://www.slideshare.net/databricks/strata-sj-everyday-im-shuffling-tips-for-writing-better-spark-programs))
  - By default it's using `ShuffledHashJoin`, the problem here is that all the data of big ones will be shuffled.
  - Use `BroadcasthashJoin`:
    - It will broadcast the small one to all workers.
    - Set `spark.sql.autoBroadcastJoinThreshold`.

### Use `forEachPartition`
  - If your task involves a large setup time, use `forEachPartition` instead.
  - For example: DB connection, Remote Call, etc.

### Data Serialization
  - The default Java Serialization is too slow.
  - Use Kyro:
    - conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer");

# **Solutions**
### java.io.IOException: No space left on device
  - The `/tmp` is probably full, check `spark.local.dir` in `spark-conf.default`.
  - How to fix it?
    - Mount more disk space:  
      `spark.local.dir   /data/disk1/tmp,/data/disk2/tmp,/data/disk3/tmp,/data/disk4/tmp`

### java.lang.OutOfMemoryError: GC overhead limit exceeded([ref](http://www.slideshare.net/jcmia1/a-beginners-guide-on-troubleshooting-spark-applications?qid=25ed4f3f-fc2e-43b2-bc8a-7f78b21bdebb&v=&b=&from_search=34))
  - Too much GC time, you can check that on Spark metrics.
  - How to fix it?
    - Increase executor heap size by `--executor-memory`.
    - Increase `spark.storage.memoryFraction`.
    - Change GC policy(ex: use G1GC).

### shutting down ActorSystem [sparkDriver] java.lang.OutOfMemoryError: Java heap space([ref](http://www.slideshare.net/jcmia1/a-beginners-guide-on-troubleshooting-spark-applications?qid=25ed4f3f-fc2e-43b2-bc8a-7f78b21bdebb&v=&b=&from_search=34))
  - OOM on Spark driver.
  - This usually happens when you fetch a huge data to driver(client).
  - Spark SQL and Streaming is a typical workload which needs large heap on driver
  - How to fix?
    - Increase `--driver-memory`.

### java.lang.NoClassDefFoundError([ref](http://www.slideshare.net/jcmia1/a-beginners-guide-on-troubleshooting-spark-applications?qid=25ed4f3f-fc2e-43b2-bc8a-7f78b21bdebb&v=&b=&from_search=34))
  - Compiled okay, but got error on run-time.
  - How to fix it?
    - Use `--jars` to upload and place on the `classpath` of your application.
    - Use `--packages` to include comma-sparated list of Maven coordinates of JARs.   
      EX: `--packages com.google.code.gson:gson:2.6.2`   
      This example will add a jar of gson to both executor and driver `classpath`.

### Serialization stack error
  - Error message:   
  Exception in thread "main" org.apache.spark.SparkException: Job aborted due to stage failure: Task 0.0 in stage 0.0 (TID 0) had a not serializable result: com.spark.demo.MyClass
Serialization stack:   
    - Object is not serializable (class: com.spark.demo.MyClass, value: com.spark.demo.MyClass@6951e281)   
    - Element of array (index: 0)   
    - Array (class [Ljava.lang.Object;, size 6)
  - How to fix it?
    - Make `com.spark.demo.MyClass` to implement `java.io.Serializable`.

### java.io.FileNotFoundException: spark-assembly.jar does not exist
  - How to fix it?
   1. Upload Spark-assembly.jar to Hadoop.
   2. Set `spark.yarn.jar`, there are two ways to configure it:
      - Add `--conf spark.yarn.jar` when launching spark-submit.
      - Set `spark.yarn.jar` on `SparkConf` in your spark driver.

### java.io.IOException: Resource spark-assembly.jar changed on src filesystem ([Reference Link](http://stackoverflow.com/questions/30893995/spark-on-yarn-jar-upload-problems))
  - Spark-assembly.jar exists in HDFS, but still get assembly jar changed error.
  - How to fix it?
   1. Upload Spark-assembly.jar to Hadoop.
   2. Set `spark.yarn.jar`, there are two ways to configure it:
      - Add `--conf spark.yarn.jar` when launching spark-submit.
      - Set `spark.yarn.jar` on `SparkConf` in your spark driver.

 ### How to find the size of dataframe in Spark
  - In Java, you can use [org.apache.spark.util.SizeEstimator](http://stackoverflow.com/a/35008549).
  - In Pyspark, one way to do it is to persist the dataframe to disk, then go to the SparkUI Storage tab and see the size.
