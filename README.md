**spark-overflow**
===================

Collect a lots of Spark information, solution, debugging etc. Feel Free to open a PR to contribute what you see or your experience on Apache Spark.


# **Knowledge**
### Spark executor memory([ref](http://www.slideshare.net/AGrishchenko/apache-spark-architecture/57))
  <img src='http://image.slidesharecdn.com/sparkarchitecture-jdkievv04-151107124046-lva1-app6892/95/apache-spark-architecture-57-638.jpg?cb=1446900275'/>

### spark-submit --verbose([ref](http://www.slideshare.net/jcmia1/a-beginners-guide-on-troubleshooting-spark-applications?qid=25ed4f3f-fc2e-43b2-bc8a-7f78b21bdebb&v=&b=&from_search=34))
  - Always add ```--verbose``` options on ```spark-submit``` to print following information
    - All default properties
    - Command line options
    - Settings from spark conf file

### Spark Executor on YARN([ref](http://www.slideshare.net/AmazonWebServices/bdt309-data-science-best-practices-for-apache-spark-on-amazon-emr))
  - Following is the memory relation config on YARN
  - YARN container size - ```yarn.nodemanager.resource.memory-mb```
  - Memory Overhead - ```spark.yarn.executor.memoryOverhead```
  <img src='http://image.slidesharecdn.com/bdt309-151009173030-lva1-app6891/95/bdt309-data-science-best-practices-for-apache-spark-on-amazon-emr-49-638.jpg'/>

# **Tunning**
### Tune the shuffle partitions
  - Tune the number of spark.sql.shuffle.partitions

### Avoid using jets3t 1.9([ref](http://www.slideshare.net/databricks/spark-summit-eu-2015-lessons-from-300-production-users))
  - it's a jar default on hadoop 2.0
  - Inexplicably terrible performance

### Use reduceBykey() instead of groupByKey()
  - reduceByKey
<img src='http://image.slidesharecdn.com/stratasj-everydayimshuffling-tipsforwritingbettersparkprograms-150223113317-conversion-gate02/95/everyday-im-shuffling-tips-for-writing-better-spark-programs-strata-san-jose-2015-9-638.jpg'/>

  - groupByKey
<img src='http://image.slidesharecdn.com/stratasj-everydayimshuffling-tipsforwritingbettersparkprograms-150223113317-conversion-gate02/95/everyday-im-shuffling-tips-for-writing-better-spark-programs-strata-san-jose-2015-10-638.jpg'/>

### GC policy([ref](https://databricks.com/blog/2015/05/28/tuning-java-garbage-collection-for-spark-applications.html))
  - G1GC is a new feature you can Use
  - Used by -XX:+UseG1GC

### Join a large Table with a small table([ref](http://www.slideshare.net/databricks/strata-sj-everyday-im-shuffling-tips-for-writing-better-spark-programs))
  - Default is ShuffledHashJoin, problem is all the data of big one will be shuffled
  - Use BroadcasthashJoin
    - it will broadcast the small one to all workers
    - Set ```spark.sql.autoBroadcastJoinThreshold```

### Use ```forEachPartition```
  - If your task involve a large setup time, use ```forEachPartition``` instead
  - For example: DB connection, Remote Call etc.

### Data Serialization
  - Default Java Serialization is too slow
  - Use Kyro
    - conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer");

# **Solution**
### java.io.IOException: No space left on device
  - It's about /tmp is full, check ```spark.local.dir``` in ```spark-conf.default```
  - How to fix? Mount more disk space
    - spark.local.dir   /data/disk1/tmp,/data/disk2/tmp,/data/disk3/tmp,/data/disk4/tmp

### java.lang.OutOfMemoryError: GC overhead limit exceeded([ref](http://www.slideshare.net/jcmia1/a-beginners-guide-on-troubleshooting-spark-applications?qid=25ed4f3f-fc2e-43b2-bc8a-7f78b21bdebb&v=&b=&from_search=34))
  - Too much GC time, you can check on Spark metrics
  - How to fix?
    - Increase executor heap size by ```--executor-memory```
    - Increase ```spark.storage.memoryFraction```
    - Change GC policy(ex: use G1GC)

### shutting down ActorSystem [sparkDriver] java.lang.OutOfMemoryError: Java heap space([ref](http://www.slideshare.net/jcmia1/a-beginners-guide-on-troubleshooting-spark-applications?qid=25ed4f3f-fc2e-43b2-bc8a-7f78b21bdebb&v=&b=&from_search=34))
  - OOM on Spark driver
  - Usually happened when you fetch a huge data to driver(client)
  - Spark SQL and Streaming is a typical workload which need large heap on driver
  - How to fix?
    - Increase ```--driver-memory```

### java.lang.NoClassDefFoundError([ref](http://www.slideshare.net/jcmia1/a-beginners-guide-on-troubleshooting-spark-applications?qid=25ed4f3f-fc2e-43b2-bc8a-7f78b21bdebb&v=&b=&from_search=34))
  - Compiled ok, but got error on run-time
  - How to fix?
    - Use ```--jars``` to upload and place on the classpath of your application
    - Use ```--packages``` to include comma-sparated list of Maven coordinates of JARs.   
      EX: ```--packages com.google.code.gson:gson:2.6.2```   
      This example will add jar of gson to both executor and driver classpath

### Serialization stack error
  - Error message likes:   
  Exception in thread "main" org.apache.spark.SparkException: Job aborted due to stage failure: Task 0.0 in stage 0.0 (TID 0) had a not serializable result: com.spark.demo.MyClass
Serialization stack:   
    -object not serializable (class: com.spark.demo.MyClass, value: com.spark.demo.MyClass@6951e281)   
    -element of array (index: 0)   
    -array (class [Ljava.lang.Object;, size 6)
  - How to fix?
    - Make ```com.spark.demo.MyClass``` to implement ```java.io.Serializable```

### Spark-assembly.jar changed on src filesystem error([ref](http://stackoverflow.com/questions/30893995/spark-on-yarn-jar-upload-problems))
  - How to fix?
   1. Upload Spark-assembly.jar to hadoop
   2. Using ```--conf spark.yarn.jar``` when spark-submit or ```conf.set("spark.yarn.jar","hdfs://hostname:port/spark-assembly-upload-at-1st.jar")``` in application
