##  一、B+树索引适用的场景

假设在table表上有索引index(key1,key2,key3)

### 一、全值匹配

```mysql
select * from table where key1=xx and key2 = xx and key3 =xx
```

### 二、最左前缀匹配

```mysql
select * from table where key1=xx；//使用key1列索引
select * form table where key1=xx and key3=xx; //使用key1列索引，因为索引列必须连续
```

### 三、匹配列前缀

```mysql
mysql对于String列的比较是按照字符来比较的，例如Aaser（前）和Aaber（后）
所以对于"Aa%"可以使用索引，但是"%ber"就无法使用索引
```

### 四、范围查询

#### 4.1 指定单个范围匹配

```mysql
select * from table where key1 > 1 and key1 < 10;
//
由于索引是按照key1进行排序的，所以会先定位 key1 = 1 然后定位 key1 = 10;
然后直接从叶子节点key1=1 顺序取值直到key1=10;
```

####4.2 指定多个范围匹配

```mysql
select * from table where key1 > 1 and key2 > 10;
//
只能用到key1列索引，无法用到key2列索引，原因跟最左前缀原则类似。因为只有在key1是等值时，key2列才能发挥作用
```

####4.3 精准匹配某一列并范围匹配另一列

```mysql
select * from table where key1 = 1 and key2 > 10;
//可以用到 key1,key2列索引
```

####4.4 in查询

https://juejin.im/post/5e1fb63af265da3e2650dcc0

MySQL优化器决定使用某个索引执行查询的仅仅是因为：使用该索引时的成本足够低。

在计算查询成本的这一步骤中大家需要注意，对于包含IN子句条件的查询来说，需要依次分析一下每一个范围区间中的记录数量是多少。MySQL优化器针对IN子句对应的范围区间的多少而指定了不同的策略：

- 如果IN子句对应的范围区间比较少，那么将率先去访问一下存储引擎，看一下每个范围区间中的记录有多少条（如果范围区间的记录比较少，那么统计结果就是精确的，反之会采用一定的手段计算一个模糊的值，当然算法也比较麻烦，我们就不展开说了，小册里有说[MySQL是怎样运行的小册链接](https://juejin.im/book/5bffcbc9f265da614b11b731?referrer=5bff96c6e51d45452f2d6f95)），这种在查询真正执行前优化器就率先访问索引来计算需要扫描的索引记录数量的方式称之为index dive。
- 如果IN子句对应的范围区间比较多，这样就不能采用index dive的方式去真正的访问二级索引idx_key1（因为那将耗费大量的时间），而是需要采用之前在背地里产生的一些统计数据去估算匹配的二级索引记录有多少条（很显然根据统计数据去估算记录条数比index dive的方式精确性差了很多）。

####4.5 not in查询

同上

### 五、排序

####5.1 直接排序

```mysql
select key1,key2,key3 from table order by key1,key2 key3
//覆盖索引查询，直接从叶子节点的最左边开始顺序取值

select * from table order by key1,key2,key3
//无法使用索引，只能全表扫描
```

####5.2 带where的排序

```mysql
表结构：
CREATE TABLE `account` (
  `id` int(11) NOT NULL,
  `key1` varchar(255) DEFAULT NULL,
  `key2` varchar(255) DEFAULT NULL,
  `key3` varchar(255) DEFAULT NULL,
  `key4` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_key1_key2` (`key1`,`key2`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

前提：where条件里没有索引列

 情况一：(选择全部数据), 结果：没有用的索引
 select * from account where key3 = "a101" order by key1;
 
 情况二：(选择的数据在where里), 结果：没有用的索引
 select key3 from account where key3 = "a101" order by key1;

 情况三(选择的数据在索引里，前缀的), 结果：没有用的索引
 select key1 from account where key3 ="a101" order by key1;


 情况四(选择的数据在索引里，后缀的) , 结果：没有用的索引
 select key2 from account where key3 = order by key1;

 情况六(查询数据在主键索引里)，结果：可以用到索引（覆盖索引）
select id from account order by key1;
```

####5.3 排序列包含非同一个索引的列

有时候用来排序的多个列不是一个索引里的，这种情况也不能使用索引进行排序，比方说：

```mysql
SELECT * FROM person_info ORDER BY name, country LIMIT 10;
```

`name`和`country`并不属于一个联合索引中的列，所以无法使用索引进行排序。

####5.4 WHERE子句中出现非排序使用到的索引列

如果WHERE子句中出现了非排序使用到的索引列，那么排序依然是使用不到索引的。

##二、总结

一、`B+`树索引适用于下边这些情况：

- 全值匹配
- 匹配左边的列
- 匹配范围值
- 精确匹配某一列并范围匹配另外一列
- 用于排序
- 用于分组

二、在使用索引时需要注意下边这些事项：

- 只为用于搜索、排序或分组的列创建索引

- 为列的基数大的列创建索引

- 索引列的类型尽量小

- 可以只对字符串值的前缀建立索引

- 只有索引列在比较表达式中单独出现才可以适用索引

- 为了尽可能少的让`聚簇索引`发生页面分裂和记录移位的情况，建议让主键拥有自增属性。

- 定位并删除表中的重复和冗余索引

- 尽量使用`覆盖索引`进行查询，避免`回表`带来的性能损耗。

##三、特殊例子

####1、[类型隐式转化]( http://www.fordba.com/mysql-type-convert-analysis.html)

1.1、如果存储的是varchar类型的数字，使用数字去查询是无法用到索引的。（因为对于数字1既可以转为字符串“01”，也可以转为“001”，那么就无法精确匹配了）

 1.2、但是存储的是数字类型的字段，使用varchar类型去查是可以用的索引的。（字符串转数字只有一种可能性）














