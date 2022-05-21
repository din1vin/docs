# Flink 学习笔记

## 一、 原理初探

### 1.角色分工

![Flink Role](/Users/dinl/Pictures/实用/flink.svg)

* **Flink Program(Client)**:
  * 将程序代码转换成DataFlow
  * 优化生成DAG图
  * Client提交job到JobManager



* **JobManager**
  * 负责整个任务的状态管理,checkPoint触发
  * 任务分发,管理节点任务状态
  * 心跳信息



* **TaskManager(Worker)**
  * 工作的节点
  * Task Slot处理DataStream

Flink系统作业的提交和调度都是利用AKKA的Actor通信(Actor System).

### 2. 执行流程

**Flink on YARN**

1. Flink用户代码上传Jar和配置到HDFS
2. 注册申请资源
3. 启动ApplicationMaster和JobManager
4. 启动TaskManager
5. 加载Jar,依赖和配置,运行任务

### 3. 重要概念

* DataFlow: Flink程序执行的时候会被映射成一个数据流模型
* Operator:数据流模型中的每一个操作被称作Operator,Operator分为: Source/Transform/Sink,多个OneToOne的Operator会被合并到同一个OperatorChain中
* Partition: 数据流模型是分布式和并行的,执行中会形成1~n个分区
* SubTask: 多个分区任务可以并行,每一个都是独立运行在一个线程中的,也就是一个SubTask子任务
* Parallelism: 并行度,就是可以同时执行的子任务数/分区数

* TaskSlot: 任务槽,每个TaskManager就是一个JVM进程,TaskSlot就是这个JVM进程中的最小资源分配单位(线程),一个TaskManager中有多少TaskSlot就意味着能支持多少个并发的Task处理
* Bounded Stream: 有界流,明确知道会结束的数据,批处理数据
* Unbounded Stream: 无界流,不断产生没有尽头的流



### 4. Flink执行流程图

* StreamGraph: 最初的程序执行逻辑流程,也就是算子之间的前后顺序--在Client上生成
* JobGraph: 将OneToOne Operator合并为OperatorChain--在Client上生成
* ExecutionGraph: 将JobGraph根据代码中设置的并行度和请求的资源进行规划--在JobManager上面生成
* 物理执行图: 将ExecutionGraph的并行计划落实到TaskManager上,SubTask落到具体TaskSlot



## 二、流批一体API

Flink 1.12已经实现流批一体,后期不再维护DataSetAPI,实现真正的流批一体.

flink默认的**RuntimeExecutionMode.STREAMING**,可以通过setRunTimeMode()方法指定BATCH模式

### 1. Source

#### I. 基于文件

* readTextFile(String path) :  调用readFile(new Path(path),path,FileProcessingMode.PROCESS_ONCE, -1,BasicTypeInfo.STRING_TYPE_INFO)

* readFile(FileInputFormat<OUT> fileInputFormat,String path, FileProcessingMode watchType,long interval,TypeInformation<OUT> typeInfo):

  `-fileInputFormat`: The input format used to create the data stream filePath 

  `–path` :the path of the file, as a URI (e.g., "file:///some/local/file" or "hdfs://host:port/file/path")

​       `-watchType`: The mode in which the source should operate, i.e. monitor path and react to new data, or process once and exit

​	   `-interval` :In the case of periodic path monitoring, this specifies the interval (in millis) between consecutive path scans

​	   `typeInformation`: Information on the type of the elements in the output stream

#### II. 基于集合

* fromElements(Object.. obs): 调用了fromCollection(Arrays.asList(obs),obs[0].type)
* fromCollection(Collection<OUT> data,TypeInformation<OUT> typeInfo): 调用addSource()将集合中的数据当做有界流并指定并行度为1,返回DataStreamSource.
* fromSequence(long start,long end): 从开始到结束,创建有状态的长整形序列,waterMark策略设置为noWaterMakers

#### III. 基于Socket

*socketTextStream(String ip,int Port): 基于socket端口的文件流

#### IV. 自定义Source

Flink提供了多种数据源接口,实现接口就可以实现自定义数据源,不同接口的功能如下:

* SourceFunction: 非并行数据源(并行度=1)
* RichSourceFunction: 多功能非并行数据源(并行度=1),可以获取RuntimeContext,
* ParallelSourceFunction: 并行数据源(并行度>=1)
* RichParallelSourceFunction:多功能并行数据源(并行度>=1),可以获取RuntimeContext



#### V. MySQL数据源

```java
public class MysqlSource extends RichParallelSourceFunction<Student> {
    private boolean flag = true;
    private PreparedStatement ps;
    private Connection conn;

    @Override
    public void open(Configuration parameters) throws Exception {
        conn = DriverManager.getConnection("url", "user", "passwd");
        ps = conn.prepareStatement("select id,name,age from student");
    }

    @Override
    public void run(SourceContext<Student> ctx) throws Exception {
        while (flag) {
            ResultSet resultSet = ps.executeQuery();
            while (resultSet.next()) {
                ctx.collect(new Student(resultSet.getInt("id"),
                        resultSet.getString("name"),
                        resultSet.getInt("age")));
            }
            Thread.sleep(5 * 1000);
        }
    }

    @Override
    public void cancel() {
        flag = false;
    }

    @Override
    public void close() throws Exception {
        if (null != conn) conn.close();
        if (null != ps) ps.close();
    }
}

@Data
@AllArgsConstructor
@NoArgsConstructor
class Student {
    private int id;
    private String name;
    private int age;
}
```

### 2. Transformation

#### **DataStream基本操作**

##### I. map ( SingleOutputStreamOperator)

> ==一对一映射==,需要实现MapFunction接口,可以使用lambda表达式

##### II. flatmap(SingleOutputStreamOperator)

> ==一对多映射==,需要实现FlatMapFunction接口,

##### III. filter (SingleOutputStreamOperator)

> 过滤,去掉不满足过滤条件的数据,需要实现FilterFunction接口,可以使用lambda表达式

##### IV. keyBy (KeyedStream)

> 按key分区,相同分区的数据会被放到同一个taskSlot里面,lambda表达式只能用于单属性keyBy,多属性元组keyBy需要实现KeySelector接口

------

#### **KeyedStream基本操作**

##### I. reduce(SingleOutputStreamOperator)

> 滚动聚合,将前面流数据的结果聚合到后面流上,再依次向后传递,

##### II. max/min(SingleOutputStreamOperator)

> 滚动更新max指定的属性,每个keyedStream里面取第一条流,更新max指定字段的最大值/最小值

##### III. maxBy/minBy(SingleOutputStreamOperator)

> 以整条流为单位,保留该字段最大的那整条流的信息,相当于滚动取流

##### IV. sum(SingleOutputStreamOperator)

> 特殊的reduce,相当于ReduceFunction中函数为(x,y)->x+y

##### V. process(SingleOutputStreamOperator)

------

#### **流的合并**

##### I. union(DataStream)

> union算子可以合并==多个相同类型==的流,并生成同类型的数据流,即可以将多个DataStream[T]合并成一个新的DataStream[T],数据顺序按照==先进先出==顺序 

##### II. connect(ConnectedStream)

> 连接两个数据流生成ConnectedStream,两条流的类型可以不一致,a.connect(b),a是左流first,b是右流last.

#### **流的拆分和选择**

##### I. split(DataStream)

> 将一个流分成多个流,在1.12版本被废弃

##### II. select(DataStream)

> 获取分流后对应的流,也在1.12版本被废弃

##### III. Side output

> ```java
> public static void main(String[] args) throws Exception {
>     StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
>     DataStreamSource<Integer> input = env.fromElements(1, 2, 3, 4, 5, 6);
>     OutputTag<Integer> oddTag = new OutputTag<>("奇数", Types.INT);
>     OutputTag<Integer> evenTag = new OutputTag<>("偶数", Types.INT);
> 
>     SingleOutputStreamOperator<Integer> sideOutPut = input.process(new ProcessFunction<Integer, Integer>() {
>         @Override
>         public void processElement(Integer value, Context ctx, Collector<Integer> out) throws Exception {
>             if (value % 2 == 0) {
>                 ctx.output(evenTag, value);
>             } else {
>                 ctx.output(oddTag, value);
>             }
>         }
>     });
>     DataStream<Integer> oddStream = sideOutPut.getSideOutput(oddTag);
>     DataStream<Integer> evenStream = sideOutPut.getSideOutput(evenTag);
>     oddStream.print("奇数");
>     evenStream.print("偶数");
>     env.execute("sideOut");
> }
> ```

### 3. Sink

#### I. print()

> 打印到控制台

#### II. printToErr()

> 打印到控制台用红色输出

#### III. writeAsText()

> 保存到text文件,在1.12中被废弃,改用addSink()替代.

#### IIII. addSink()

> 重写invoke方法,涉及到连接资源的关闭,extends RichSinkFunction

### 4. Connectors

#### I. JDBC Connector

> ```java
> ds.addSink(JdbcSink(sql,(ps,t)->{
>   ps.setInt(1,t.id),
>   ps.setString(2,t.name),
>   ps.setInt(3,t.age)
> },
>   new JdbcConnctionOptions.JdbcConnectionOptionsBuilder()
>                    .withUrl(url)
>                    .withUsername(username)
>                    .withPassword(pswd)
>                    .withDriverName("com.mysql.jdbc.Driver")
>                    .build()
>           )
> );
> ```

#### II. Kafka Connector

> Consumer(source):
>
> ```Java
> env.addSource(new FlinkKafkaConsumer<>("topic",new SimpleStringSchema(),properties))
> ```
>
> Producer(sink):
>
> ```java
> ds.addSink(new FlinkKafkaProducer<String>("broker_list",new SimpleStringSchema(), properties))
> ```

#### III. Redis Connector

[外部项目文档][https://bahir.apache.org/docs/flink/current/flink-streaming-redis/]

> ```java
> result.addSink(new RedisSink<Tuplue2<String,String>>( new FlinkJedisPool.Builder()
>                                                      .setHost()
>                                                      .build(),new MyRedisMapper() ))
> ```
>
> 

[https://bahir.apache.org/docs/flink/current/flink-streaming-redis/]: 

## 三、 Flink高级API

### Flink的四大基石

* Window: 开窗即用的滚动滑动窗口,时间计数窗口
* State: 丰富的StateAPI. ValueState , ListState, MapState, BoardcastState
* Time: 实现了WaterMark机制,乱序数据处理,迟到数据容忍
* CheckPoint: 基于Chandy-Lamport算法,实现了分布式一致性快照,提供了一致性语义

### 1. Window

对流数据进行一些聚合操作,需要用到window

* time-window: 时间窗口,根据时间划分窗口,如:每xx分钟统计最近xx分钟的数据
* count-window:数量窗口,根据数量划分窗口,如:每xx个数据,统计最近xx个数据.
* session-window: 会话窗口,设置一个超时时间,如果超过该时间没有数据到来,触发上个窗口计算

### 2. Time&&WaterMarker

单独用window的场景一般是ProcessingTime,实际中更受关注的时间语义是EventTime,使用WaterMarker的时候需要告诉Flink哪个时间是事件时间.

EventTime是Flink 1.12默认的时间语义,



```java
public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);
        DataStreamSource<Order> source = env.addSource(new MyOrderGenerator());
        SingleOutputStreamOperator<Order> orderSingleOutputStreamOperator = source
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Order>forBoundedOutOfOrderness(Duration.ofSeconds(3))  //forBoundedOutOfOrderness 表示乱序数据 最大允许延迟时间3秒
                        .withTimestampAssigner((order, tms) -> order.getTms()) //指定EventTime
                );

        OutputTag<Order> lateness = new OutputTag<Order>("Lateness", TypeInformation.of(Order.class));
        SingleOutputStreamOperator<Order> output = orderSingleOutputStreamOperator.keyBy(Order::getUid)
                .window(TumblingEventTimeWindows.of(Time.minutes(15)))
                .allowedLateness(Time.seconds(300))
                .sideOutputLateData(lateness)
                .sum("money");
        //获取迟到的数据流
        DataStream<Order> latenessOut = output.getSideOutput(lateness);
        env.execute("Window Test");
    }
```



### 3. State

无状态计算: map/FlatMap/Filter... 相同输入得到相同结果,就是无状态计算.

有状态计算:sum/reduce.... 相同的输入得到不同的结果.

#### Managed State & Raw State

|              | Managed State                                                | Raw State                            |
| ------------ | ------------------------------------------------------------ | ------------------------------------ |
| 状态管理方式 | Flink Runtime管理<br />自动存储,自动恢复<br />内存管理上有优化 | 用户自己管理<br />需要用户自己序列化 |
| 状态数据结构 | 已知的数据结构<br />value,list,map                           | 字节数组<br />byte[]                 |
| 推荐使用场景 | **大多数情况下均可使用**                                     | **自定义Operator可用**               |

#### Keyed State & Operator State

| Keyed State                                                  | Operator State                                     |
| ------------------------------------------------------------ | -------------------------------------------------- |
| 只能用在Keyed State                                          | 可用于所有算子,常用于Source,例如FlinkKafkaConsumer |
| 每个key对应一个State                                         | 一个Operator实例对应一个State                      |
| 并发改变,State随着key迁移                                    | 并发改变可用重新分配: 均匀分配/合并后每个得到全量  |
| 通过RuntimeContext访问                                       | 实现CheckpointedFunction 或 ListCheckpointed接口   |
| ValueState,ListState,RuducingState,AggregatingState,MapState | ListState                                          |

## 四、容错机制

### checkpoint  Vs state

* State

  维护/存储的是一个Operator运行的状态/历史值,在内存中维护.

  state数据默认保存在Java的堆内存中个,TaskManager节点的内存上.

  state可以被记录,在失败的情况下数据还可以恢复.

* checkpoint

  某一个时刻,flink所有的Operator的当前State的全局快照,一般存在磁盘上

  可以理解成checkpoint把state数据定时的持久化存储了

  

[https://bahir.apache.org/docs/flink/current/flink-streaming-redis/]: 