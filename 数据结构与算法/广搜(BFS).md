## 一、概念

> [广度优先搜索]([https://zh.wikipedia.org/wiki/%E5%B9%BF%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2](https://zh.wikipedia.org/wiki/广度优先搜索))
>
> **广度优先搜索算法**（BFS），是一种[图形搜索算法](https://zh.wikipedia.org/w/index.php?title=圖形搜索演算法&action=edit&redlink=1)。简单的说，BFS是从[根节点](https://zh.wikipedia.org/w/index.php?title=根節點&action=edit&redlink=1)开始，沿着树的宽度遍历树的[节点](https://zh.wikipedia.org/wiki/节点)。如果所有节点均被访问，则算法中止。广度优先搜索的实现一般采用队列实现。

## 二、原理

​        BFS是一种[盲目搜索法](https://zh.wikipedia.org/w/index.php?title=盲目搜尋法&action=edit&redlink=1)，目的是检查[图](https://zh.wikipedia.org/wiki/图)中的所有节点，以找寻结果。换句话说，它会彻底地搜索整张图，直到找到结果为止。
​    从算法的观点，所有因为展开节点而得到的子节点都会被加进一个[先进先出](https://zh.wikipedia.org/wiki/先進先出)的[队列](https://zh.wikipedia.org/wiki/队列)中。一般的实现里，其邻居节点尚未被检验过的节点会被放置在一个被称为 *open* 的容器中（例如队列或是[链表](https://zh.wikipedia.org/wiki/連結串列)），而被检验过的节点则被放置在被称为 *closed* 的容器中(例如哈希表)。

## 三、伪代码

1. 首先将根节点放入队列中。
2. 从队列中取出第一个节点，并检验它是否为目标。
   - 如果找到目标，则结束搜索并回传结果。
   - 否则将它所有尚未检验过的直接子节点加入队列中。
3. 若队列为空，表示整张图都检查过了——亦即图中没有欲搜索的目标。结束搜索并回传“找不到目标”。
4. 重复步骤2。

```java
// 计算从起点 start 到终点 target 的最近距离
int BFS(Node start, Node target) {
    Queue<Node> q; // 核心数据结构
    Set<Node> visited; // 避免走回头路

    q.offer(start); // 将起点加入队列
    visited.add(start);
    int step = 0; // 记录扩散的步数

    while (q not empty) {
        int sz = q.size();
        /* 将当前队列中的所有节点向四周扩散 */
        for (int i = 0; i < sz; i++) {
            Node cur = q.poll();
            /* 划重点：这里判断是否到达终点 */
            if (cur is target)
                return step;
            /* 将 cur 的相邻节点加入队列 */
            for (Node x : cur.adj())
                if (x not in visited) {
                    q.offer(x);
                    visited.add(x);
                }
        }
        /* 划重点：更新步数在这里 */
        step++;
    }
}
```