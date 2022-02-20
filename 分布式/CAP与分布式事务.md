## 一、CAP

- **Consistency（可用性）**
意思是只要收到用户的请求，服务器就必须给出回应。用户可以选择向 G1 或 G2 发起读操作。不管是哪台服务器，只要收到请求，就必须告诉用户，到底是 v0 还是 v1，否则就不满足可用性。

- **Availability（一致性）**
意思是，写操作之后的读操作，必须返回该值。举例来说，某条记录是 v0，用户向 G1 发起一个写操作，将其改为 v1。之后用户访问服务器的任一节点，返回的值都必须是v1。这就需要更改记录时，需要所有的节点返回修改成功，才能告诉用户修改成功。

- **Partition tolerance（分区容错）**
大多数分布式系统都分布在多个子网络。每个子网络就叫做一个区（partition）。分区容错的意思是，区间通信可能失败。比如，一台服务器放在中国，另一台服务器放在美国，这就是两个区，它们之间可能无法通信。

一般来说，分区容错无法避免，因此可以认为 CAP 的 P 总是成立。CAP 定理告诉我们，剩下的 C 和 A 无法同时做到。

### Consistency 和 Availability 的矛盾

一致性和可用性，为什么不可能同时成立？答案很简单，因为可能通信失败（即出现分区容错）。

如果保证 G2 的一致性，那么 G1 必须在写操作时，锁定 G2 的读操作和写操作。只有数据同步后，才能重新开放读写。锁定期间，G2 不能读写，没有可用性。

如果保证 G2 的可用性，那么势必不能锁定 G2，所以一致性不成立。

综上所述，G2 无法同时做到一致性和可用性。系统设计时只能选择一个目标。如果追求一致性，那么无法保证所有节点的可用性；如果追求所有节点的可用性，那就没法做到一致性。

## 二、分布式事务

### 2.1 2PC（同步阻塞、单点问题、太过保守）

<img src="https://gitee.com/nieyunshu/picture/raw/master/img/20220219233848.jpg" alt="Xnip2022-02-19_23-37-57" style="zoom:50%;" />

<img src="https://gitee.com/nieyunshu/picture/raw/master/img/20220220014841.jpg" alt="Xnip2022-02-20_01-48-34" style="zoom:50%;" />

<img src="https://gitee.com/nieyunshu/picture/raw/master/img/20220220014927.jpg" alt="Xnip2022-02-20_01-49-19" style="zoom:50%;" />

### 2.2 3PC

<img src="https://gitee.com/nieyunshu/picture/raw/master/img/20220220015339.jpg" alt="Xnip2022-02-20_01-53-31" style="zoom:50%;" />

<img src="https://gitee.com/nieyunshu/picture/raw/master/img/20220220015441.jpg" alt="Xnip2022-02-20_01-54-10" style="zoom:50%;" />

<img src="https://gitee.com/nieyunshu/picture/raw/master/img/20220220015555.jpg" alt="Xnip2022-02-20_01-55-36" style="zoom:50%;" />

### 2.3 TCC

TCC事务的处理流程与3PC两阶段提交类似，不过3PC通常都是在跨库的DB层面，而TCC本质上就是一个应用层面的3PC，需要通过业务逻辑来实现。这种分布式事务的实现方式的优势在于，可以让**应用自己定义数据库操作的粒度，使得降低锁冲突、提高吞吐量成为可能**。

而不足之处则在于对应用的侵入性非常强，业务逻辑的每个分支都需要实现try、confirm、cancel三个操作。此外，其实现难度也比较大，需要按照网络状态、系统故障等不同的失败原因实现不同的回滚策略。为了满足一致性的要求，confirm和cancel接口还必须实现幂等。

## 三、MySql XA

https://www.cnblogs.com/zhoujinyi/p/5257558.html

## 四、参考资料

[2pc、3pc](https://segmentfault.com/a/1190000004474543)

[TCC](https://www.cnblogs.com/duanxz/p/5226316.html)

[TCC](https://houbb.github.io/2018/09/02/sql-distribute-transaction-tcc)

[小米分布式事务实践](https://xiaomi-info.github.io/2020/01/02/distributed-transaction/)

