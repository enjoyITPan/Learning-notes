## 一、OOM（内存溢出）

1、查看gc.log 看是不是频繁发生full gc

2、设置参数在OOM时自动dump内存数据：

```java
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/admin/logs/java.hprof -Xloggc:/home/admin/logs/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps
```

3、手机dump堆内存快照

```java
jmap -dump:format=b,file=heapdump.phrof pid
```

4、dump文件分析工具：

​	4.1、jvisualvm

​	4.2、eclipse的MAT插件

​	4.3、zprofiler

## 二、CPU彪高

1、top 命令查看占用cpu高的进程,获取到pid

2、top -Hp pid 获取占用cpu最高的线程

3、jstack -l pid （dump线程堆栈信息）

4、需要将第二步的pid转为16进制，到dump线程堆信息里grep。

5、查看是否有死锁发生、查看Runable的线程数

## 参考资料

- [JVM问题分析处理手册](https://zhuanlan.zhihu.com/p/43435903)
- [谈谈线上CPU100%排查套路](https://www.cnblogs.com/rjzheng/p/10315250.html)

