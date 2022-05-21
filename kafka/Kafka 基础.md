# Kafka 基础



## Kafka是什么



### message queue

kafka是一个分布式的消息队列,一个典型的kafka体系架构包括若干Producer,若干Broker,和若干个Consumer以及一个Zookeeper集群.

* Producer: 生产者,也就是发送消息的一方,生产者负责创建消息,将其投递到kafka中.
* Consumer: 消费者,也就是接受消息的一方,消费者连接到kafka上并接收消息,进而进行相应的业务逻辑处理.
* Broker: 服务代理节点,分布式的kafka一般有多个服务器.

### kafka的用途

* 异步通信:生产者发送的消息无需等待消费者的处理结果
* 解耦: 写入消息跟读取消息的服务之间互不影响
* 削峰: 消息队列看做是中间缓存, 允许高并发的消息延迟处理,削峰防止突然的高并发场景.
* 发布订阅:  天然的发布/订阅模型
* 可恢复性: 支持从指定offset开始重复消费,提高消费者处理的容错.
* 顺序排队: 相同Topic的相同partition内消息是有序的,即先写入的消息先被消费.
* ...

## Command Line命令

### Topic

**列举**

> kafka-topics.sh --zookeeper localhost:2181/kafka --list

**查看**

>kafka-topics.sh  --zookeeper localhost:2181/kafka --describe --topic test

注： describe可以列举多个kafka topic，如果不指定topic 可以看到所有topic的详细信息



**创建**

> kafka-topics.sh --zookeeper localhost:2181/kafka --create --topic test --partitions 4  --replication-factor 2



## 客户端开发

### Producer

```java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

public class KafkaProducerDemo {
    public static final String BROKERS = "localhost:9092";
    public static final String TOPIC = "test";

    public static Properties initConfig() {
        Properties prop = new Properties();
        prop.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, BROKERS);
        prop.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        prop.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        prop.put(ProducerConfig.CLIENT_ID_CONFIG, "producer.id.demo");
        return prop;
    }

    public static void main(String[] args) {
        Properties config = initConfig();
        KafkaProducer<String, String> producer = new KafkaProducer<>(config);
        ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC, "message 1");
        try {
            producer.send(record);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```



* 生产者端自定义分区器

  1. 在生产者的property里面指定分区器为自定义分区器.

  ```java
  prop.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, KafkaPartitionerCustom.class);
  ```

  2. 自定义分区器实现Partitioner接口

  ```java
  import org.apache.kafka.clients.producer.Partitioner;
  import org.apache.kafka.common.Cluster;
  
  import java.util.Map;
  
  public class KafkaPartitionerCustom implements Partitioner {
      @Override
      public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
          Integer partitionsCnt = cluster.partitionCountForTopic(topic);
          return key.toString().hashCode() % partitionsCnt;
      }
  
      @Override
      public void close() {
      }
  
      @Override
      public void configure(Map<String, ?> configs) {
      }
  }
  ```

  

### Consumer

```java
import java.time.Duration;
import java.util.Collections;
import java.util.Properties;
import java.util.concurrent.atomic.AtomicBoolean;

public class KafkaConsumerDemo {
    public static final String BROKER = "localhost:9092";
    public static final String TOPIC = "test";
    public static final String GROUP_ID = "group1";
    public static final AtomicBoolean isRunning = new AtomicBoolean(true);


    public static Properties initConfig() {
        Properties prop = new Properties();
        prop.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, BROKER);
        prop.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        prop.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        prop.put(ConsumerConfig.GROUP_ID_CONFIG, GROUP_ID);
        prop.put(ConsumerConfig.CLIENT_ID_CONFIG, "consumer.id.demo");
        return prop;
    }

    public static void main(String[] args) {
        Properties config = initConfig();
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(config);
        consumer.subscribe(Collections.singletonList(TOPIC));
        try {
            while (isRunning.get()) {
                ConsumerRecords<String, String> poll = consumer.poll(Duration.ofMillis(1000));
                for (ConsumerRecord<String, String> record : poll) {
                    System.out.println("topic = " + record.topic() + " \n" +
                            "header = " + record.headers() + " \n" +
                            "message = { " + record.value() + "}");
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```



kafka 查看消息堆积

```shell
kafka-consumer-groups.sh --bootstrap-server 172.20.4.77:9094 --describe --group h5-sign-uv
```

