## 介绍

在日常业务中,spark常见的就是通过路径通配符*,{}等方式一次读取多个文件,一次批处理将这些文件做一个大job写入Hive或者ODPS,笔者最近在用Spark读取Hudi的文件时候发现了一个诡异的文件丢失Bug:

**一次读入所有文件夹会有部分文件夹丢失,一开始怀疑是这部分文件夹本身有损坏,但是用spark单独读取该文件夹的时候发现数据又不会丢失.**

既然一次job会丢数据,那么不妨按文件夹拆分job,每个job执行单个任务,常见就是for循环去遍历所有文件夹挨个执行,但是效率过低需要

**六个小时**,在资源不变的情况下用多线程提交,成功把时间缩短到了**一个小时**.与全文件读取效率相当.



## code

```scala
import java.util.concurrent.Executors
import scala.collection.JavaConverters.asScalaSetConverter
import scala.concurrent.duration.{Duration, HOURS}
import scala.concurrent.{Await, ExecutionContext, ExecutionContextExecutorService, Future}


def runJobDirByDir(conf:Config,prefix:String):Unit = {
	assert(prefix.startswith("oss://"),"a legal path should starts with [oss://]")
	//spark framework多个类要用到spark,所以用了ThreadLocal共享spark环境
	val spark = EnvUtils.take 
 	// 自己写的oss文件系统工具,作用是列举文件夹
	val ofs = new OssSystemUtil(conf)
	val projects = ofs.listDir(prefix)
	val pool = Executors.newFixedThreadPool(5)
	implicit val xc:ExecutionContextExecutorService = ExecutionContext.fromExecutorService(pool)
	//因为子线程的ThreadLocal里面没有sparkEnv,所以在这传进去
	val tasks = projects.map(x=> runFutureTask(spark,x)) 
	Await.result(Future.sequence(tasks.toSeq),Duration(5,MINUTE))
}

def runFutureTask(spark:SparkContext, dir:String)(implicit xc: ExecutionContext)
	:Future[Unit]= Future{
    //TODO run single dir 
}
```

