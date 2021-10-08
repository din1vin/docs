# Kafka 基础

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

