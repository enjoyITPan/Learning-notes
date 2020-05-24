## 一、MySQL分页查询优化(基于limit offset，size)

### 1、表结构：

```mysql
create table `test`(
  `id` int(11),
   `className` varchar(255) NOT NULL comment `班级名`,
   `name` varchar(255) NOT NULL comment `学生姓名`,
   primary KEY (`id`),
   KEY idx_className(`className`) USING BTREE
)
```

插入一千万条数据到表中，保证className相同的数据至少有100万条。

### 2、直接查询

**直接分页查询耗时70s**：

```mysql
select * from test where class_name = "班级一" limit 900000,10;
```

![](img/1716e93e53bffc6d.png)


看mysql执行计划，的确用到了索引，但是查询耗时长达70s。

一般分页查询有两种做法：

1、先查询出90万+10条记录的id，回表查询数据，再将90万+10条完整记录发给MySQL以便筛选最后10条；
2、先查询出90万+10条记录的id，筛选出最后10条记录的id再回表查询，最后返回10条完整记录给MySQL。
在回表次数很多（limit决定）的情况下，显然第二种方法是比较快的，但是MySQL默认采用了第一种。

### 3、优化方案

很明显mysql默认的limit处理方式问题在于回表次数太多了，那么如果降低了回表次数（减少IO次数），性能是否提升呢？

#### 3.1、通过利用覆盖索引直接获取id，这里可以直接走索引，性能较高。再将筛选完的id到主键索引查询。

```mysql
select * from test as t1 inner join (select id from test where class_name = "班级一" order by id desc limit 900000,10) t2 USING(id);
```

![](img/1716e948bcce8c11.png)

优化后的效果显著：从70秒变为了0.47秒。

#### 3.2、利用上一次的最大id筛选数据

```mysql
//上图的最大id是 998507,查询时让id > 998507,再直接limit 10就可以得到第91万页了
select * from test where class_name = "班级一" and id > 998507 limit 10;
```

![](img/1716e94dc0899eb8.png)

查询效率依旧很快，0.01秒返回结果