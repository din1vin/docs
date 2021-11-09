### 1.

```shell
Exception in thread "main" org.apache.hudi.exception.HoodieIOException: Failed to get instance of org.apache.hadoop.fs.FileSystem
	at org.apache.hudi.common.fs.FSUtils.getFs(FSUtils.java:98)
	at org.apache.hudi.DefaultSource.createRelation(DefaultSource.scala:81)
	at org.apache.hudi.DefaultSource.createRelation(DefaultSource.scala:63)
	at org.apache.spark.sql.execution.datasources.DataSource.resolveRelation(DataSource.scala:340)
	at org.apache.spark.sql.DataFrameReader.loadV1Source(DataFrameReader.scala:239)
	at org.apache.spark.sql.DataFrameReader.load(DataFrameReader.scala:227)
	at org.apache.spark.sql.DataFrameReader.load(DataFrameReader.scala:174)
	at com.taptap.singletest.TapDBUserTapTap$.main(TapDBUserTapTap.scala:23)
	at com.taptap.singletest.TapDBUserTapTap.main(TapDBUserTapTap.scala)
Caused by: java.io.IOException: No FileSystem for scheme: oss
	at org.apache.hadoop.fs.FileSystem.getFileSystemClass(FileSystem.java:2586)
	at org.apache.hadoop.fs.FileSystem.createFileSystem(FileSystem.java:2593)
	at org.apache.hadoop.fs.FileSystem.access$200(FileSystem.java:91)
	at org.apache.hadoop.fs.FileSystem$Cache.getInternal(FileSystem.java:2632)
	at org.apache.hadoop.fs.FileSystem$Cache.get(FileSystem.java:2614)
	at org.apache.hadoop.fs.FileSystem.get(FileSystem.java:370)
	at org.apache.hadoop.fs.Path.getFileSystem(Path.java:296)
	at org.apache.hudi.common.fs.FSUtils.getFs(FSUtils.java:96)
	... 8 more
```

解决方案:

```java
SparkSession.
    ...
.config("spark.hadoop.fs.oss.impl", "org.apache.hadoop.fs.aliyun.oss.AliyunOSSFileSystem")
```

指定oss接口.



### 2.

```shell
Caused by: java.lang.ClassNotFoundException: org.apache.avro.LogicalType
```

缺少avro依赖,加上这两个就可以了

```groovy
implementation 'org.apache.parquet:parquet-avro:1.9.0'
implementation 'org.apache.avro:avro:1.9.0'
```