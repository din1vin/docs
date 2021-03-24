# Flink DataStream API(一): 基本操作

## 从WordCount开始

类似于学习任何变成语言的Hello World一样,大数据框架的Demo通常从Word Count开始,看一看Flink 是怎么做Word Count的吧~

```java
//DataStrem Api Word Count
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

public class WindowWordCount {

    public static void main(String[] args) throws Exception {
				//获取Stream执行环境,通常只需要getExcutionEnviroment()方法内部,根据运行环境自动选择Local或Remote
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment(); 

        DataStream<String> socketStream = env
                .socketTextStream("localhost", 9999);
      					
        DataStream<Tuple2<String,Integet>> countStream = socketStram
          			.flatMap(new Splitter()) 
        				.keyBy(value -> value.f0)  
        				.window(TumblingProcessingTimeWindows.of(Time.seconds(5)))
        				.sum(1);

        dataStream.print();

        env.execute("Window WordCount");
    }

  	//自定义Spliter实现FlatMapFunction
    public static class Splitter implements FlatMapFunction<String, Tuple2<String, Integer>> {
        @Override
        public void flatMap(String sentence, Collector<Tuple2<String, Integer>> out) throws Exception {
            for (String word: sentence.split(" ")) {
                out.collect(new Tuple2<String, Integer>(word, 1));
            }
        }
    }

}
```



Flink api 编程一般可以分为三个部分:

- 1. 数据来源
- 2. Transaction算子,转换逻辑

* 3. 输出(落地)

 

## 数据源

源是flink程序处理的数据来源;



### 内置数据源

#### 基于文件

* readTextFile(path): 读取符合TextInputFormat规范的文件,每行一条,生成String类型的数据源;

* readFile(fileInputFormat, path): 按照指定格式读取数据源;

  

#### 基于socket

* socketTextStream: 从socket接口读取数据,元素可以被定界符分隔

#### 从集合生成

* fromCollection(Collection): 从Java的集合java.util.Collection类生成,所有元素必须类型相同;
* fromCollection(Iterator,Class): 从迭代器生成,class用来指定迭代器返回的类型;

* fromElements(T ...): 直接从元素队列生成,元素必须类型相同
* fromParallelConllection(SplitableIterator, class): 从并行的迭代器生成数据流,class用来指定迭代器返回的类型
* generateSequence(from,to) - 并行的从给定区间创建数据队列



### 自定义数据源

#### 非并行的数据源-- implements SourceFunction

实现SourceFunction接口,需要重载里面的run方法和cancel 方法,自定义好的接口通过env.addSource()方法接入Flink;

```java
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.SourceFunction;

import java.util.Random;

public class MyFlinkSource implements SourceFunction<Tuple2<Long,Integer>> {
    private boolean run = true;
    @Override
    public void run(SourceContext ctx) throws Exception {
        while(run){
            ctx.collect(new Tuple2<Long,Integer>(System.currentTimeMillis(),new Random().nextInt(100)));
            Thread.sleep(1000);
        }
    }

    @Override
    public void cancel() {
        this.run = false;
    }
  
  	public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<Tuple2<Long, Integer>> tuple2DataStreamSource = env.addSource(new MyFlinkSource());

        tuple2DataStreamSource.print();
        env.execute();

    }
}
```



## ETL





![image-20210226161850376](/Users/dinl/Library/Application Support/typora-user-images/image-20210226161850376.png)



上图是flink官方对于DataSteam之间的转换关系图,通过对数据流的一系列转换,打到计算目的:



### 无状态转换

#### map

> map是基本转换算子,对数据流中的每条数据进行操作,不改变数据流的条数;

#### flatmap

> map方法只适用于一对转换,flatmap可以实现一对多转换,比如wordcount里面的将一行word按照空格进行拆分;

#### connect

> connect 方法可以将不同流进行合并,让DataStream变成ConnectedDataStream,ConnectedDataStream无法直接输出,要使用coMap方法才能转回DataStream,该方法常用于控制流过滤等场景

#### Keyed Streams

![img](https://ververica.cn/wp-content/uploads/2019/08/%E5%B9%BB%E7%81%AF%E7%89%8711.png)

##### keyBy

> 将一个流,根据其中的一些属性进行分区,不同类别的流会被放到不同的slot里面去执行,可以物理的理解成对数据流水平切分,每个slot处理不同的类别;



##### 滚动更新 max/min/sum 

>max跟min(int filedPosition,String fieldName),max会更新指定position的字段信息,其他同key下的保留**第一个流**的信息不变,比如:

> 文本流:
>
> ```shell
> 1,Alice,Chinese,60
> 1,Alice,Math,77
> 1,Alice,History,89
> 2,Bob,Chinese,78
> 2,Bob,English,100
> 3,Tom,English,88
> 3,Tom,Chinese,66
> 4,Jerry,Chinese,67
> 4,Jerry,Math,29
> ```
>
> 根据cid分组keyby,滚动更新grade的最高值:
>
> ```java
> public class KeyedStreamTest {
>     public static void main(String[] args) throws Exception {
>         StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
>         env.setParallelism(1);
> 
>         DataStreamSource<String> source = env.readTextFile("/Users/dinl/Data/code/personal/flink/try-flink/src/main/resources/my_text_source.txt");
> 
>         DataStream stream = source
>                 .map(Person::new)
>                 .keyBy(Person::getCid)
>                 .max("grade");
>         stream.print();
>         env.execute();
>     }
> }
> ```
>
> 输出:
>
> ```shell
> Person{cid=1, name='Alice', subject='Chinese', grade=60}
> Person{cid=1, name='Alice', subject='Chinese', grade=77}
> Person{cid=1, name='Alice', subject='Chinese', grade=89}
> Person{cid=2, name='Bob', subject='Chinese', grade=78}
> Person{cid=2, name='Bob', subject='Chinese', grade=100}
> Person{cid=3, name='Tom', subject='English', grade=88}
> Person{cid=3, name='Tom', subject='English', grade=88}
> Person{cid=4, name='Jerry', subject='Chinese', grade=67}
> Person{cid=4, name='Jerry', subject='Chinese', grade=67}
> ```
>
> sum同理,只会滚动更新grade的总值;



##### 滚动取值 maxBy/minBy

> maxBy跟max的区别是: max是在每个keyedStream里面取第一条流,更新max指定字段的最大值,而maxBy则会以整条流为单位,保留该字段最大的那整条流的信息,相当于滚动取值,上述例子maxby运行结果是:
>
> ```shell
> Person{cid=1, name='Alice', subject='Chinese', grade=60}
> Person{cid=1, name='Alice', subject='Math', grade=77}
> Person{cid=1, name='Alice', subject='History', grade=89}
> Person{cid=2, name='Bob', subject='Chinese', grade=78}
> Person{cid=2, name='Bob', subject='English', grade=100}
> Person{cid=3, name='Tom', subject='English', grade=88}
> Person{cid=3, name='Tom', subject='English', grade=88}
> Person{cid=4, name='Jerry', subject='Chinese', grade=67}
> Person{cid=4, name='Jerry', subject='Chinese', grade=67}
> ```

##### 滚动更新算子 reduce

> reduce可以实现自定义的聚合,但并非MapReduce那种多条流聚合成一条流,而是会计算与上一条流的聚合方式,1+2-> update(2), 2+3->update(3)...
>
> 把第一条跟第二条流的聚合结果,更新到第二条流上,然后用新的第二条流跟第三条流聚合,结果更新到第三条上....
>
> 见例子:
>
> ```java
> public class KeyedStreamTest {
>  public static void main(String[] args) throws Exception {
>      StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
>      env.setParallelism(1);
> 
>      DataStreamSource<String> source = env.readTextFile("/Users/dinl/Data/code/personal/flink/try-		      flink/src/main/resources/my_text_source.txt");
> 
>      DataStream stream = source
>              .map(Person::new)
>              .keyBy(Person::getCid)
>              .reduce((o1, o2) ->
>                      new Person(o1.getCid(),
>                              o1.getName(),
>                              o1.getSubject() + "," + o2.getSubject(), //把两个科目用逗号连起来
>                              o1.getGrade() + o2.getGrade())); //分数直接相加
>      stream.print();
>      env.execute();
>  }
> 
> }
> ```
>
> 上述reduce的运行结果是:
>
> ```shell
> Person{cid=1, name='Alice', subject='Chinese', grade=60}
> Person{cid=1, name='Alice', subject='Chinese,Math', grade=137}
> Person{cid=1, name='Alice', subject='Chinese,Math,History', grade=226}
> Person{cid=2, name='Bob', subject='Chinese', grade=78}
> Person{cid=2, name='Bob', subject='Chinese,English', grade=178}
> Person{cid=3, name='Tom', subject='English', grade=88}
> Person{cid=3, name='Tom', subject='English,Chinese', grade=154}
> Person{cid=4, name='Jerry', subject='Chinese', grade=67}
> Person{cid=4, name='Jerry', subject='Chinese,Math', grade=96}
> ```

### 有状态转换

Flink为什么要参与状态管理?

​	flink为管理状态提供了一些引人注目的特性:

		* **本地性** : 为了保证访问速度,flink把状态存储在本地内存中;
		* **持久性** : flink状态是容错的,可以按照一定时间间隔产生checkpoint,并且在任务失败后进行恢复;
  * **纵向扩展性**: flink状态可以存储在RocksDB实例中,可以通过增加本地磁盘来扩展存储空间;
  * **横向扩展性**: flink状态可以随着集群扩展重新分布;
  * **可查询性**: flink状态可以通过状态查询API给外界反馈;

#### keyed

为了保证状态初始化,flink对于上述MapFunction/FilterFunction/FlatMapFunction提供了"rich"变体;

如RichFlatMapFunction,其中增加了以下方法,包括

		* open(Configuration c) :仅在算子初始化时调用一次,可以用来加载一些静态数据,或者建立外部服务的链接;
		* close() : 是生命周期中的最后一个调用的方法，清理状态。
		* getRuntimeContext() : 是程序创建和访问flink状态的途径;



example: 保留每个组第一个流

```java
public class MyFlatMapper extends RichFlatMapFunction<Person, Person> {
    ValueState<Boolean> keyHasBeenSeen;

    @Override
    public void open(Configuration conf) {
        ValueStateDescriptor<Boolean> desc = new ValueStateDescriptor<>("keyHasBeenSeen", Types.BOOLEAN);
        keyHasBeenSeen = getRuntimeContext().getState(desc);
    }

    @Override
    public void flatMap(Person value, Collector<Person> out) throws IOException {
      	//当key没有见过的时候collect一条
        if(keyHasBeenSeen.value()==null){
            out.collect(value);
          	//collect完之后将状态置为true
            keyHasBeenSeen.update(true);
        }
    }
}


public class KeyedStreamTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        DataStreamSource<String> source = env.readTextFile("/Users/dinl/Data/code/personal/flink/try-flink/src/main/resources/my_text_source.txt");
        DataStream stream = source
                .map(Person::new)
                .keyBy(Person::getCid)
                .flatMap(new MyFlatMapper());

        stream.print();
        env.execute();
    }

}
```

使用上述例子运行该代码,输出如下:

>```shell
>Person{cid=1, name='Alice', subject='Chinese', grade=60}
>Person{cid=2, name='Bob', subject='Chinese', grade=78}
>Person{cid=3, name='Tom', subject='English', grade=88}
>Person{cid=4, name='Jerry', subject='Chinese', grade=67}
>```

#### No-keyed

在没有键的上下文中我们也可以使用 Flink 管理的状态。这也被称作算子的状态。它包含 的接口是很不一样的，由于对用户定义的函数来说使用 non-keyed state 是不太常见的， 所以这里就介绍了。这个特性最常用于 source 和 sink 的实现。

## ConnectedStream Options

将两种流connect即可得到ConnectedStream,对于ConnectedStream,需要调用coFunction(CoMapFuction,CoFlatMapFunction,RichCoFlatMapFunction...),下面从一个指定流过滤来说明:

```java
public class WordFilter {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        //从socket汇入的主流
        DataStream<String> socketInput = env
                .fromElements("Flink","Marvel","Disney","Hollywood","Java")
                .keyBy(x -> x);

        //用来过滤主流的控制流
        DataStream<String> control = env
                .fromElements("Flink","Java")
                .keyBy(x -> x);

        control
                .connect(socketInput)
                .flatMap(new ControlFilter())
                .print();

        env.execute("Control Filter");
    }
}

class ControlFilter extends RichCoFlatMapFunction<String, String, String> {
    ValueState<Boolean> isFilter;

    @Override
    public void open(Configuration parameters) {
        isFilter = getRuntimeContext().getState(new ValueStateDescriptor<>("isFilter", Types.BOOLEAN));
    }

    //左流(控制流)flatMap方法,将左流的key设置为isFilter
    @Override
    public void flatMap1(String value, Collector<String> out) throws Exception {
        isFilter.update(true);
    }

    //右流(主输入流)flatMap方法,将右流中尚未更新的key输出
    @Override
    public void flatMap2(String value, Collector<String> out) throws Exception {
        if(null==isFilter.value()){
            out.collect(value);
        }
    }
}
```

在这个例子中,只有两个流键一致才能连接, 当keyed stream被连接时,他们被发到同一个实例上,RichCoFlatMapFunction在状态中存了一个布尔类型的变量,这个变量被两个流共享,流的左右顺序取决去用哪个流connect,`A`.connect(`B`)那A就是左流,B就是右流;

