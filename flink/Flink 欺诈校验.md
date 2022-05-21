# Flink 欺诈校验 V1

## 项目构建

```shell
mvn archetype:generate \
-DarchetypeGroupId=org.apache.flink \
-DarchetypeArtifactId=flink-walkthrough-datastream-java \
-DarchetypeVersion=1.11.0 \
-DgroupId=frauddetection  \
-DartifactId=frauddetection \  
-Dversion=0.1 \
-Dpackage=spendreport \
-DinteractiveMode=false \
```

## Pom文件

```pom
<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>frauddetection</groupId>
    <artifactId>frauddetection</artifactId>
    <version>0.1</version>
    <packaging>jar</packaging>

    <name>Flink Walkthrough DataStream Java</name>
    <url>https://flink.apache.org</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <flink.version>1.12.0</flink.version>
        <target.java.version>1.8</target.java.version>
        <scala.binary.version>2.11</scala.binary.version>
        <maven.compiler.source>${target.java.version}</maven.compiler.source>
        <maven.compiler.target>${target.java.version}</maven.compiler.target>
        <log4j.version>2.12.1</log4j.version>
    </properties>

    <repositories>
        <repository>
            <id>apache.snapshots</id>
            <name>Apache Development Snapshot Repository</name>
            <url>https://repository.apache.org/content/repositories/snapshots/</url>
            <releases>
                <enabled>false</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-walkthrough-common_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <!-- This dependency is provided, because it should not be packaged into the JAR file. -->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
            <scope>provided</scope>
        </dependency>

        <!-- Add connector dependencies here. They must be in the default scope (compile). -->

        <!-- Example:

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        -->

        <!-- Add logging framework, to produce console output when running in the IDE. -->
        <!-- These dependencies are excluded from the application JAR by default. -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j-impl</artifactId>
            <version>${log4j.version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>${log4j.version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>${log4j.version}</version>
            <scope>runtime</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>

            <!-- Java Compiler -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>${target.java.version}</source>
                    <target>${target.java.version}</target>
                </configuration>
            </plugin>

            <!-- We use the maven-shade plugin to create a fat jar that contains all necessary dependencies. -->
            <!-- Change the value of <mainClass>...</mainClass> if your program entry point changes. -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.0.0</version>
                <executions>
                    <!-- Run shade goal on package phase -->
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <artifactSet>
                                <excludes>
                                    <exclude>org.apache.flink:force-shading</exclude>
                                    <exclude>com.google.code.findbugs:jsr305</exclude>
                                    <exclude>org.slf4j:*</exclude>
                                    <exclude>org.apache.logging.log4j:*</exclude>
                                </excludes>
                            </artifactSet>
                            <filters>
                                <filter>
                                    <!-- Do not copy the signatures in the META-INF folder.
                                    Otherwise, this might cause SecurityExceptions when using the JAR. -->
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>spendreport.FraudDetectionJob</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>

        <pluginManagement>
            <plugins>

                <!-- This improves the out-of-the-box experience in Eclipse by resolving some warnings. -->
                <plugin>
                    <groupId>org.eclipse.m2e</groupId>
                    <artifactId>lifecycle-mapping</artifactId>
                    <version>1.0.0</version>
                    <configuration>
                        <lifecycleMappingMetadata>
                            <pluginExecutions>
                                <pluginExecution>
                                    <pluginExecutionFilter>
                                        <groupId>org.apache.maven.plugins</groupId>
                                        <artifactId>maven-shade-plugin</artifactId>
                                        <versionRange>[3.0.0,)</versionRange>
                                        <goals>
                                            <goal>shade</goal>
                                        </goals>
                                    </pluginExecutionFilter>
                                    <action>
                                        <ignore/>
                                    </action>
                                </pluginExecution>
                                <pluginExecution>
                                    <pluginExecutionFilter>
                                        <groupId>org.apache.maven.plugins</groupId>
                                        <artifactId>maven-compiler-plugin</artifactId>
                                        <versionRange>[3.1,)</versionRange>
                                        <goals>
                                            <goal>testCompile</goal>
                                            <goal>compile</goal>
                                        </goals>
                                    </pluginExecutionFilter>
                                    <action>
                                        <ignore/>
                                    </action>
                                </pluginExecution>
                            </pluginExecutions>
                        </lifecycleMappingMetadata>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>
```

## 分析

- ==程序是如何加载的==
  
  `FraudDetectionJob入口类`
  
  ```java
  package spendreport;
  
  import org.apache.flink.streaming.api.datastream.DataStream;
  import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
  import org.apache.flink.walkthrough.common.sink.AlertSink;
  import org.apache.flink.walkthrough.common.entity.Alert;
  import org.apache.flink.walkthrough.common.entity.Transaction;
  import org.apache.flink.walkthrough.common.source.TransactionSource;
  
  /**
   * Skeleton code for the datastream walkthrough
   */
  public class FraudDetectionJob {
      public static void main(String[] args) throws Exception {
          StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
  
      //新建输入流,从自定义的Transaction读入
          DataStream<Transaction> transactions = env
              .addSource(new TransactionSource())
              .name("transactions");
  
      //处理算子
          DataStream<Alert> alerts = transactions
              .keyBy(Transaction::getAccountId) //通过id分组,相同id的流会被分配到同一个slot上
              .process(new FraudDetector()) //对每个分组进行欺诈校验
              .name("fraud-detector");
  
      //输出
          alerts
              .addSink(new AlertSink())
              .name("send-alerts");
  
          env.execute("Fraud Detection");
      }
  }
  ```

- ==数据源==

```java
package org.apache.flink.walkthrough.common.source;

import org.apache.flink.annotation.Public;
import org.apache.flink.streaming.api.functions.source.FromIteratorFunction;
import org.apache.flink.walkthrough.common.entity.Transaction;

import java.io.Serializable;
import java.util.Iterator;

/**
 * A stream of transactions.
 */
@Public
public class TransactionSource extends FromIteratorFunction<Transaction> {

    private static final long serialVersionUID = 1L;

    public TransactionSource() {
        super(new RateLimitedIterator<>(TransactionIterator.unbounded())); // 限速流来源于TransactionIterator
    }

  //内部类限速迭代器
    private static class RateLimitedIterator<T> implements Iterator<T>, Serializable {

        private static final long serialVersionUID = 1L;

        private final Iterator<T> inner;

    //构造方法私有,只能在TransactionSource类中才能被实例化
        private RateLimitedIterator(Iterator<T> inner) {
            this.inner = inner;
        }

        @Override
        public boolean hasNext() {
            return inner.hasNext();
        }

        @Override 
    //重写FromIteratorFunction方法的next方法来不断生成数据流
        public T next() {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            return inner.next(); //无界流从inner data中循环生成Transaction对象
        }
    }
}     
```

- ==封装起来的数据流迭代器==
  
  ```java
  package org.apache.flink.walkthrough.common.source;
  
  import org.apache.flink.walkthrough.common.entity.Transaction;
  
  import java.io.Serializable;
  import java.sql.Timestamp;
  import java.util.Arrays;
  import java.util.Iterator;
  import java.util.List;
  
  /**
   * An iterator of transaction events.
   */
  final class TransactionIterator implements Iterator<Transaction>, Serializable {
  
      private static final long serialVersionUID = 1L;
  
      private static final Timestamp INITIAL_TIMESTAMP = Timestamp.valueOf("2019-01-01 00:00:00");
  
      private static final long SIX_MINUTES = 6 * 60 * 1000;
  
      private final boolean bounded;
  
      private int index = 0;
  
      private long timestamp;
  
      static TransactionIterator bounded() {
          return new TransactionIterator(true);
      }
  
      static TransactionIterator unbounded() {
          return new TransactionIterator(false);
      }
  
      private TransactionIterator(boolean bounded) {
          this.bounded = bounded;
          this.timestamp = INITIAL_TIMESTAMP.getTime();
      }
  
      @Override
      public boolean hasNext() {
          if (index < data.size()) {
              return true;
          } else if (!bounded) { //如果是unbounded,当index到data.size()时从头再次重新生成
              index = 0;
              return true;
          } else {
              return false;
          }
      }
  
      @Override
      public Transaction next() {
          Transaction transaction = data.get(index++);
          transaction.setTimestamp(timestamp);
          timestamp += SIX_MINUTES;
          return transaction;
      }
  
    //真正进入flink的数据源
      private static List<Transaction> data = Arrays.asList(
          new Transaction(1, 0L, 188.23),
          new Transaction(2, 0L, 374.79),
          new Transaction(3, 0L, 112.15),
          new Transaction(4, 0L, 478.75),
          new Transaction(5, 0L, 208.85),
          new Transaction(1, 0L, 379.64),
          new Transaction(2, 0L, 351.44),
          new Transaction(3, 0L, 320.75),
          new Transaction(4, 0L, 259.42),
          new Transaction(5, 0L, 273.44),
          new Transaction(1, 0L, 267.25),
          new Transaction(2, 0L, 397.15),
          new Transaction(3, 0L, 0.219),
          new Transaction(4, 0L, 231.94),
          new Transaction(5, 0L, 384.73),
          new Transaction(1, 0L, 419.62),
          new Transaction(2, 0L, 412.91),
          new Transaction(3, 0L, 0.77),
          new Transaction(4, 0L, 22.10),
          new Transaction(5, 0L, 377.54),
          new Transaction(1, 0L, 375.44),
          new Transaction(2, 0L, 230.18),
          new Transaction(3, 0L, 0.80),
          new Transaction(4, 0L, 350.89),
          new Transaction(5, 0L, 127.55),
          new Transaction(1, 0L, 483.91),
          new Transaction(2, 0L, 228.22),
          new Transaction(3, 0L, 871.15),
          new Transaction(4, 0L, 64.19),
          new Transaction(5, 0L, 79.43),
          new Transaction(1, 0L, 56.12),
          new Transaction(2, 0L, 256.48),
          new Transaction(3, 0L, 148.16),
          new Transaction(4, 0L, 199.95),
          new Transaction(5, 0L, 252.37),
          new Transaction(1, 0L, 274.73),
          new Transaction(2, 0L, 473.54),
          new Transaction(3, 0L, 119.92),
          new Transaction(4, 0L, 323.59),
          new Transaction(5, 0L, 353.16),
          new Transaction(1, 0L, 211.90),
          new Transaction(2, 0L, 280.93),
          new Transaction(3, 0L, 347.89),
          new Transaction(4, 0L, 459.86),
          new Transaction(5, 0L, 82.31),
          new Transaction(1, 0L, 373.26),
          new Transaction(2, 0L, 479.83),
          new Transaction(3, 0L, 454.25),
          new Transaction(4, 0L, 83.64),
          new Transaction(5, 0L, 292.44));
  }
  ```

欺诈校验规则(同一个用户上一个小于1.0下一个大于500的订单,会被检测出来):

​        最直观的思路是维持一个flag,检测每笔订单的金额,订单小于1时将flag置为true,否则置为false,当检测到大于500的订单并且flag为true的时候,该笔订单即是欺诈订单,但是有如下问题:

1. 订单会被分发到分布式系统的不同slot上,如果A用户有一笔小于1的订单,紧接着B用户买了一笔500以上的订单,会被认为B用户的这笔订单异常,显然不符合检查逻辑 

2. 为了维护每个用户的flag,可以尝试维护一个id->flag的映射Map,通过内存中的Map再使用第1步中的逻辑判断欺诈订单,但是常规的类成员变 量是无法做到容错处理的，当任务失败重启后，之前的状态信息将会丢失。 这样的话，如 果程序曾出现过失败重启的情况，将会漏掉一些欺诈报警。

3. 为了应对前面的问题,Flink提供了一套支持容错状态的原语,使用最基本的原语`ValueState`,`ValueState` 是一种 `keyed state`,只能在KeyBy操作之后使用;
   
   ValueState是一个包装类，类似于 Java 标准库里边
    的 AtomicReference 和 AtomicLong。 它提供了三个用于交互的方法。update 用于更新 状态，value 用于获取状态值，还有 clear 用于清空状态。 如果一个 key 还没有状态， 例如当程序刚启动或者调用过 ValueState#clear 方法时，ValueState#value 将会返 回 null。 如果需要更新状态，需要调用 ValueState#update 方法，直接更改 ValueState#value 的返回值可能不会被系统识别。

```java
package spendreport;

import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.util.Collector;
import org.apache.flink.walkthrough.common.entity.Alert;
import org.apache.flink.walkthrough.common.entity.Transaction;

/**
 * Skeleton code for implementing a fraud detector.
 */
public class FraudDetector extends KeyedProcessFunction<Long, Transaction, Alert> {

    private static final long serialVersionUID = 1L;

    private static final double SMALL_AMOUNT = 1.00;
    private static final double LARGE_AMOUNT = 500.00;
    private static final long ONE_MINUTE = 60 * 1000;

    private transient ValueState<Boolean> flagState;

  //使用open完成使用前的注册,相当于初始化一个Boolean的状态
    @Override
    public void open(Configuration parameters) throws Exception {
        ValueStateDescriptor<Boolean> flagDescriptor = new ValueStateDescriptor<>("flag", Types.BOOLEAN);
        flagState = getRuntimeContext().getState(flagDescriptor);
    }

  //
    @Override
    public void processElement(
            Transaction transaction,
            Context context,
            Collector<Alert> collector) throws Exception {
        Boolean lastTransactionWasSmall = flagState.value();
        if(lastTransactionWasSmall!=null){
            if(transaction.getAmount()>LARGE_AMOUNT){
        //只有在lastTransactionWasSmall 并且本订单金额大于500时新建Alert
                Alert alert = new Alert();
                alert.setId(transaction.getAccountId());
                collector.collect(alert);//只有被collector收集的alert才会流到下一步
            }
            //clean state
            flagState.clear();
        }

        if(transaction.getAmount()<SMALL_AMOUNT){
            //本金额订单小与1,将flag置为True
            flagState.update(true);
        }

    }
}
```

* ==数据源Keyby id== 
  
  | 1      | 2      | 3      | 4      | 5      |
  | ------ | ------ | ------ | ------ | ------ |
  | 188.23 | 374.79 | 112.15 | 478.75 | 208.85 |
  | 379.64 | 351.44 | 320.75 | 259.42 | 273.44 |
  | 267.25 | 397.15 | 0.219  | 231.94 | 384.73 |
  | 419.62 | 412.91 | 0.77   | 22.10  | 377.54 |
  | 375.44 | 230.18 | 0.80   | 350.89 | 127.55 |
  | 483.91 | 228.22 | 871.15 | 64.19  | 79.43  |
  | 56.12  | 256.48 | 148.16 | 199.95 | 252.37 |
  | 274.73 | 473.54 | 119.92 | 323.59 | 353.16 |
  | 211.90 | 280.93 | 347.89 | 459.86 | 82.31  |
  | 373.26 | 479.83 | 454.25 | 83.64  | 292.44 |

将案例中的数据源按照id keyby之后,再进行欺诈校验,欺诈校验认为小于1.0的订单之后下一单超过500的用户id会被校验出来(3号用户第5到第6笔订单不符合校验规则);

## 踩坑记录

1. 构建的源码无法直接运行
   
   > 错误: 无法初始化主类 spendreport.FraudDetectionJob
   > 原因: java.lang.NoClassDefFoundError: org/apache/flink/streaming/api/functions/source/SourceFunction

修改方式:

![image-20210119141926337](/Users/dinl/Library/Application Support/typora-user-images/image-20210119141926337.png)

# Flink 欺诈校验V2: 状态 + 时间=❤

骗子们在小额交易后不会等很久就进行大额消费，这样可以降低小额测试交易被发现的几率.假设你为欺诈检测器设置了一分钟的超时，对于上边的例子，用户3的大额消费订单只有在小额订单完成的一分钟内紧跟大额订单才算是欺诈校验.

就需要对上述程序做如下修改:

                 * 当标记状态被设置为 true 时，设置一个在当前时间一分钟后触发的定时器。
                 * 当定时器被触发时，重置标记状态。
                 * 当标记状态被重置时，删除定时器。

```java
package spendreport;

import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.util.Collector;
import org.apache.flink.walkthrough.common.entity.Alert;
import org.apache.flink.walkthrough.common.entity.Transaction;

import java.io.IOException;

/**
 * Skeleton code for implementing a fraud detector.
 */
public class FraudDetector extends KeyedProcessFunction<Long, Transaction, Alert> {

    private static final long serialVersionUID = 1L;
    private static final double SMALL_AMOUNT = 1.00;
    private static final double LARGE_AMOUNT = 500.00;
    private static final long ONE_MINUTE = 60 * 1000;

    private transient ValueState<Boolean> flagState;
    private transient ValueState<Long> timeSate;

    @Override
    public void open(Configuration parameters){
        ValueStateDescriptor<Boolean> flagDescriptor = new ValueStateDescriptor<>("flag", Types.BOOLEAN);
        flagState = getRuntimeContext().getState(flagDescriptor);

        ValueStateDescriptor<Long> timeDescriptor = new ValueStateDescriptor<>("timer-state",Types.LONG);
        timeSate = getRuntimeContext().getState(timeDescriptor);
    }

    @Override
    public void onTimer(long timestamp, OnTimerContext ctx, Collector<Alert> out) {
    //当定时器被触发时，重置标记状态。
        timeSate.clear();
        flagState.clear();
    }



    @Override
    public void processElement(
            Transaction transaction,
            Context context,
            Collector<Alert> collector) throws Exception {
        Boolean lastTransactionWasSmall = flagState.value();
        if(lastTransactionWasSmall!=null){
            if(transaction.getAmount()>LARGE_AMOUNT){
                Alert alert = new Alert();
                alert.setId(transaction.getAccountId());

                collector.collect(alert);
            }
            //clean state 当标记状态被重置时，删除定时器。
            cleanUp(context); 
        }

        if(transaction.getAmount()<SMALL_AMOUNT){
            //set state to True
            flagState.update(true);
            long timer = context.timerService().currentProcessingTime() + ONE_MINUTE;
            context.timerService().registerEventTimeTimer(timer); // 一分钟后调用onTimer,设置一分钟后的触发器
            timeSate.update(timer);
        }
    }

    private void cleanUp(Context ctx) throws IOException {
        Long timer = timeSate.value();
        ctx.timerService().deleteProcessingTimeTimer(timer);//删除定时器,防止定时到的时候误删状态
        //clean up all state
        timeSate.clear();
        flagState.clear();
    }
}
```
