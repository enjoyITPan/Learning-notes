## 一、分库分表策略

- 垂直切分

  垂直切分是指按照业务将表进行分类，分布到不同的数据库上面，这样也就将数据或者说压力分担到不同的库上面

- 水平切分

  垂直拆分后遇到单机瓶颈，可以使用水平拆分。相对于垂直拆分的区别是：垂直拆分是把不同的表拆到不同的数据库中，而水平拆分是把同一个表拆到不同的数据库中。

   相对于垂直拆分，水平拆分不是将表的数据做分类，而是按照某个字段的某种规则来分散到多个库之中，每个表中包含一部分数据。简单来说，我们可以将数据的水平切分理解为是按照数据行的切分，就是将表中 的某些行切分到一个数据库，而另外的某些行又切分到其他的数据库中，主要有分表，分库两种模式

- 垂直&水平切分

## 二、分库分表时机（即单表数据量多大时需要拆分）

​          单表行数超过 500 万行或者单表容量超过 2GB，才推荐进行分库分表。当数据量太大时，由于查询维度较多，即使添加从库、优化索引，做很多操作时性能仍下降严重。此时就要考虑对其进行切分了，切分的目的就在于减少数据库的负担，缩短查询时间。（mysql单表存储量推荐是百万级）

## 三、主从同步



## 四、DB扩容

单库单表-> 分库分表：

1. 确定分库分表键
2. 确定路由策略(一般采用hash 路由)
3. 确定分布式id生成方案
4. 通过binlog或者扫表的模式把存量数据迁移到分库分表
5. 开启双写，将增量数据写入两份表
6. 灰度切流，将部分请求的读写都转移到分库分表
7. 灰度验证通过后，增大灰度比例，直到100%
8. 删除单库单表

分库分表 -> 扩容

方案一：

每次扩容都按照2倍进行扩容(该方案只适合一个库一个表)。

例如：原始存在两个库，每个库1张表。分别是A，B库

原始路由方案：

id % (库数量) 确定表

现在新增A1,B1库

1、将A1变为A的从库，进行数据同步

2、将B1变为B的从库，进行数据同步

3、修改路由规则：

id % 2 == 0 => A 改为 id % 4 == 0 => A  id % 4 == 2 => A1

id % 2 == 1> B 改为 id % 4 == 1 => B  id % 4 == 3 => B1

这样就无需进行数据迁移，只需要在数据同步完成后，删除A，B中多余的数据就可以了

方案二：

对于一个库多个表的方案一就不适合了

这个时候可以先建立y个库，通过扫表或者binlog 拉去老库数据进行rehash到新库。

这个过程需要停服。也可以不停服进行双写，但是要处理好数据冲突。

数据同步完成后，修改路由配置。然后删除老库就可以了
