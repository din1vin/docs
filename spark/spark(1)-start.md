# spark 基础

## 什么是RDD？

为什么非要从RDD开始呢？虽然RDD API使用频率越来越低，但是DataFrame跟DataSet API最终都会在spark内部转化成RDD的分布式计算，如果想对Spark计算有一个知其所以然的把控，就要对RDD有足够的了解.

**RDD-弹性分布式数据集**，是spark实现分布式计算的基石。Spark的所有特性都来自于RDD。深入理解RDD能更全面更系统的学习Spark的工作原理。

## RDD 的五个特性

以下是摘自Spark RDD源码中对于RDD的五个main properties 的描述。

```textile
* A list of partitions： 
    RDD是由多个分区组成的，分区是逻辑上的概念。RDD的计算是以分区为单位进行的。
* A function for computing each split ： 
    作用于每个分区数据的计算函数。
* A list of dependencies on other RDDs ： 
    RDD中保存了对于父RDD的依赖，根据依赖关系组成了Spark的DAG（有向无环图），实现了spark巧妙、容错的编程模型
* Optionally, a Partitioner for key-value RDDs (e.g. to say that the RDD is hash-partitioned)
    分区器针对键值型RDD而言的，将key传入分区器获取唯一的分区id。在shuffle中，分区器有很重要的体现。
* Optionally, a list of preferred locations to compute each split on (e.g. block locations for an HDFS file)
    根据数据本地性的特性，获取计算的首选位置列表，尽可能的把计算分配到靠近数据的位置，减少数据的网络传输
```

总结下来就是RDD描述了数据血缘（数据加工的步骤，DAG图的生成依据）、数据分区（分布式依据，数据由哪个Excutor计算）等。

除了上述描述所说的特性外，**Spark延迟计算**和Shuffle阶段划分，都跟RDD之间的“算子”有着密不可分的联系。

## shuffle划分，宽窄依赖

很多的博客会机械化的说“宽窄依赖是父子RDD的依赖关系。窄依赖描述的是一对一的关系，宽依赖描述的是多对一关系”，这句话不对，起码不全对，窄依赖也有多个父RDD对应一个子RDD的关系，甚至多对多的笛卡尔积，也属于窄依赖。

那宽窄依赖究竟怎么分呢？

`NarrowDependency` 为 parent RDD 的一个或多个分区的数据全部流入到 child RDD 的一个或多个分区，而 `ShuffleDependency` 则为 parent RDD 的每个分区的每一部分，分别流入到 child RDD 的不同分区。

这里强调了分区内部数据的整体或部分，分区内数据有按情况拆分，就属于shuffleDependency了。

## 延迟计算

在刚开始接触spark的时候，还会接触到懒加载，延迟计算这类概念，给人一种很奇怪的感觉，为什么有懒加载机制？懒加载是什么样子的机制，为什么算子要区分Transformation和Action？

spark是内存计算，而RDD只是一种数据形态转换的抽象，为了行程完整的DAG以及方便计算优化，spark设计成部分算子是修改或者定义DAG节点，即Transformation算子。而另一部分算子才能触发计算，那些能触发spark计算的算子，就是所谓的Action算子。

既然说到spark算子划分，顺便看一下spark的算子描述划分吧，这是吴磊老师对于spark算子划分的表格：

![](https://static001.geekbang.org/resource/image/a3/88/a3ec138b50456604bae8ce22cdf56788.jpg?wh=1599x1885)



action算子可以理解成DAG的终点，当开发者调用一个Action算子时，底层代码会调用sc.runJob方法，通过iter context构造job，具体一点就是： 

Spark DAGScheduler会**以action算子为起点，从后向前回溯DAG，并以Shuffle为边界划分Stage，基于Stage创建taskSets，并将taskSets交给TaskSheduler调度，然后交由ScheduleBackend调度资源完成具体的Task。**



以上加粗的文字，也可以用来回答“请简述一下spark任务提交过程”这类问题了。



## task是什么？

为了更好的认识task，我们先看看task有哪些属性：

* stageID： task所在的属性ID

* stageAttempId: 失败重试编号

* taskBinary: 任务代码

* partition： task对应的RDD分区

* locs： 本地倾向性

stageId、stageAttemptId 标记了 Task 与执行阶段 Stage 的所属关系；taskBinary 则封装了隶属于这个执行阶段的用户代码；partition 就是我们刚刚说的 RDD 数据分区；locs 属性以字符串的形式记录了该任务倾向的计算节点或是 Executor ID。

不难发现，taskBinary、partition 和 locs 这三个属性，一起描述了这样一件事情：Task 应该在哪里（locs）为谁（partition）执行什么任务（taskBinary）。



通常情况下一个rdd的一个partition对应一个task，textfile等文件源task取决于block数量。
