### 1. Static Methods in interface require -target:jvm-1.8

#### 说明

gradle项目+scala 2.11+java8+flink 1.12

出错代码：

```scala
kafkaSource.assignTimestampsAndWatermarks(WatermarkStrategy
      .forBoundedOutOfOrderness[JSONObject](Duration.ofSeconds(10)))
```

报错信息：==Static Methods in interface require -target:jvm-1.8==

从报错看是scala尝试调用Java接口中的静态方法报的错。但是idea配置中编译打包都用的jdk1.8



修改以下IDEA配置无效：

![image-20210916161926056](https://i.loli.net/2021/09/16/5hdFoyGlBqTAsOr.png)

于是思考换一个突破口，在build.gradle文件添加以下配置,问题解决。

```groovy
project.tasks.compileScala.scalaCompileOptions.additionalParameters = ["-target:jvm-1.8"]
project.tasks.compileTestScala.scalaCompileOptions.additionalParameters = ["-target:jvm-1.8"]
```

