# JUC并发编程

## 1. 什么是JUC?

juc(java.util.concurrent)包,包括:

* java.util.concurrent
* java.util.concurrent.atomic
* Java.util.concurrent.locks

**普通线程代码:**

Thread,Runnable 没有返回值,效率相比于Callable相对较低,功能也不如Callable高.



## 2. 线程和进程

**进程**: 一个程序,程序的一次执行过程,线程的集合,Java进程默认有两个线程(main,GC);.

**线程**: 资源调度的最小单位.Java开启线程的方法(Thread,Runable,Callable).



### 2.1 并发和并行

**并发**: 

**并行**: