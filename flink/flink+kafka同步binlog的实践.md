## 场景说明

​	 业务部门的Mysql数据使用阿里DTS（数据传输工具）上报了binlog日志。对于数仓部门有两个需求

1. 从DTS将数据，只保留type(insert，delete，update)跟record到kafka方便多方消费；
2. 为了验证Kafka准确性，数仓部门也消费kafka数据落地到Hologres，对比hologres与mysql原表保证完全一致。

## 问题

1. DTS端并行度为1，但是ETL过程因为下游算子有多个并行度，有可能出现数据乱序（DTS->Kafka乱序）
2. 消费Kafka的时候由于Kafka有多个分区，同样存在乱序。



## DTS->kafka 局部有序

需要因为消费是通过type关联主键执行sql，所以只要保证相同id相邻数据不出现乱序即可。

dts单个并行度是有序，那么只要在让相同id的数据被路由到同一个TaskManager来处理，就可以避免TM之间传输先后造成的乱序，就可以局部有序。

这里主要使用了Flink自定义分区器，把相同主键分到一个TaskManager。

```scala
import org.apache.flink.api.common.functions.Partitioner

//定义一个自定义的分区器，按照传入的键分区，PK键是String类型。
object CustomPartitioner extends Partitioner[String] {
   //numPartitions是下游算子数量，相同key的hashCode相同，就可以保证相同key会被发往相同TM。
  override def partition(key: String, numPartitions: Int): Int = {
    key.hashCode % numPartitions
  }
}


dtsStream
//重点是这个 partitionCustom，需要两个参数，第一个是分区键的分区策略，第二个是如何从流里获取分区键
      .partitionCustom(CustomPartitioner, x=> MetaInfoUtil.getPkOrUk(x))
      ....  //Other ETL 
	  .addSink(kafkaProducer)
// kafkaSink也要保证相同id会被存到kafka的相同partition中（在Kafka的Schema指定）
    env.execute(jobName)
```

kafka指定分区策略Demo

```scala
package com.taptap.data.stream.util

import com.alibaba.fastjson.{JSON, JSONObject}
import com.alibaba.fastjson.serializer.SerializerFeature
import org.apache.flink.streaming.connectors.kafka.KafkaSerializationSchema
import org.apache.kafka.clients.producer.ProducerRecord
import org.apache.kafka.common.header.internals.{RecordHeader, RecordHeaders}

import java.lang

class DtsKafkaSerializationSchema extends KafkaSerializationSchema[JSONObject] {

  override def serialize(element: JSONObject, timestamp: lang.Long): ProducerRecord[Array[Byte], Array[Byte]] = {
    val key = MetaInfo.getPuOrUk
    val headers = element.getJSONObject("headers")
    val kafkaHeaders = new RecordHeaders()
    val binlogType = headers.getString("binlog.type")
    val binlogDatabase = headers.getString("binlog.database")
    val binlogTable = headers.getString("binlog.table")
      //把元信息放到Header里
    kafkaHeaders.add(new RecordHeader("binlog.type", binlogType.getBytes()))
    kafkaHeaders.add(new RecordHeader("binlog.database", binlogDatabase.getBytes()))
    kafkaHeaders.add(new RecordHeader("binlog.table", binlogTable.getBytes()))

    val message = element.getJSONObject("message")
    val messageJsonString = JSON.toJSONString(message,SerializerFeature.WriteMapNullValue)
    val record = new ProducerRecord(
      topicName,
      null,
      timestamp,
      key.getBytes(), // 在这里指定分区键，这里是PkorUK
      messageJsonString.getBytes(),
      kafkaHeaders
    )

    record
  }
}

```



## Kafka->hologres 窗口内排序

为了保证partition之间按照时间戳顺序，取了一个小窗口，组内排序。

```scala
//使用kafka数据源自带的watermarker，保证局部有序
kafkaSource.assignTimestampsAndWatermarks(WatermarkStrategy
      .forBoundedOutOfOrderness[JSONObject](Duration.ofSeconds(10)))

val insertTag: OutputTag[JSONObject] = new OutputTag[JSONObject]("insert")
val deleteTag: OutputTag[JSONObject] = new OutputTag[JSONObject]("delete")
val updateTag: OutputTag[JSONObject] = new OutputTag[JSONObject]("update")

//定义窗口内的processFunction
val myProcessAllFunction = new ProcessAllWindowFunction[JSONObject, JSONObject, TimeWindow] {
      override def process(context: Context, elements: Iterable[JSONObject], out: Collector[JSONObject]): Unit = {
        //按照时间戳排序
        val sorted = elements.toList.sortBy(_.getLong("timestamp"))
        for (i <- sorted) {
          val tp = i.getString("binlog.type")
          tp match {
            case "insert" => context.output(insertTag, i)
            case "update" => context.output(updateTag, i)
            case "delete" => context.output(deleteTag, i)
          }
        }
      }
  }

	//允许迟到数据
	val lateness = new OutputTag[JSONObject]("Lateness")

	//主要逻辑
    val sideOut: DataStream[JSONObject] = streamSource
      .windowAll(TumblingEventTimeWindows.of(Time.seconds(3)))
      .allowedLateness(Time.seconds(10))
      .sideOutputLateData(lateness)
      .process(myProcessAllFunction)
    val insertStream: DataStream[JSONObject] = sideOut.getSideOutput(insertTag)
    val updateStream = sideOut.getSideOutput(updateTag)
    val deleteStream = sideOut.getSideOutput(deleteTag)

	//处理迟到数据
	sideOut.getSideOutput(lateness)
      .windowAll(TumblingEventTimeWindows.of(Time.seconds(3)))
      .process(myProcessAllFunction)

	//根据类型落sink
	insertStream.addSink(insertSink)
    updateStream.addSink(updateSink)
    deleteStream.addSink(deleteSink)
    env.execute()


```



