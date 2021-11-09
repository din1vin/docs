# S





# park/Pyspark 代码片段



## 1. Spark/Pyspark 访问 oss/hdfs 文件系统

在某些情况下，spark周期调度任务要轮询oss或者hdfs文件夹，判断文件是否存在，存在新文件才启动ETL任务，这个时候就需要列举oss/hdfs文件夹，那么spark如何获取oss/HDFS文件系统呢？

以OSS为例：

### 1.1 python

```python
from pyspark.sql import SparkSession

spark = Spark.builder.appName("spark oss").getOrCreate()
sc = spark.sparkContext
conf = sc._jsc.hadoopConfiguration()
conf.set("fs.oss.accessKeyId","xxx")
conf.set("fs.oss.accessKeySecret","xxx")
conf.set("fs.oss.endpoint","xxx")
conf.set("fs.oss.impl","org.apache.hadoop.fs.aliyun.oss.AliyunOSSFileSystem")

path = sc._jvm.org.apache.hadoop.fs.Path("oss://xxxx")
fs = sc._jvm.org.apache.hadoop.fs.FileSystem.get(path.toUri(),conf)
# 这个fs就是FileSystem对象，就像Java中的FileSystem那么用就可以
# 判断文件是否存在
exist = fs.exists(path)
```



### 1.2 Scala

```scala
import org.apache.hadoop.fs.{FileSystem, Path}
import org.apache.hadoop.conf.Configuration
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
				.appName("spark oss")
				.getOrCreate()
val conf = new Configuration()
conf.set("fs.oss.accessKeyId","xxx")
conf.set("fs.oss.accessKeySecret","xxx")
conf.set("fs.oss.endpoint","xxx")
conf.set("fs.oss.impl","org.apache.hadoop.fs.aliyun.oss.AliyunOSSFileSystem")
val path = new Path("oss://tapdb-ods-for-dla/dla_dataset_v4/")
val fs = FileSystem.get(path.toUri, conf)
// 同上 获得文件系统fs
```



## 2. spark通配符过滤部分文件

### 2.1 Scala 

```scala
import org.apache.hadoop.fs.{Path,PathFilter}
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
				.appName("spark path filter")
				.getOrCreate()
val sc = spark.sparkContext
sc.hadoopConfiguration.setClass("mapreduce.input.pathFilter.class",classOf[MyFilter],classOf[PathFilter])

class MyFilter extends PathFilter{
  override def accept(path:Path):Boolean = !path.getName.endsWith(".tmp")
}

```

