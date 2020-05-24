## 1、LRU

### 1.1 题目描述

​     [leetcode链接](https://leetcode.com/problems/lru-cache/)

>Design and implement a data structure for [Least Recently Used (LRU) cache](https://en.wikipedia.org/wiki/Cache_replacement_policies#LRU). It should support the following operations: `get` and `put`.

> `get(key)` - Get the value (will always be positive) of the key if the key exists in the cache, otherwise return -1.
> `put(key, value)` - Set or insert the value if the key is not already present. When the cache reached its capacity, it should invalidate the least recently used item before inserting a new item.

> The cache is initialized with a **positive** capacity.
>
> Could you do both operations in **O(1)** time complexity?
### 1.2 思路

get(key)和put(key)需要在0(1)时间内，自然而然想到使用Map。

同时为了实现最近最少使用的逻辑，所以需要维护数据的访问顺序。这里使用链表来实现。（因为更新访问顺序也需要O(1)，所以这里使用双向链表）

```java
class LRUCache {
    // 哨兵
    LinkNode head;
    // 哨兵
    LinkNode tail;
    // 初始容量
    int cap;
    // 实际大小
    int size;
    Map<Integer,LinkNode> dataMap = new HashMap<>();

    public LRUCache(int capacity) {
        this.head = new LinkNode();
        this.tail = new LinkNode();
        this.cap = capacity;
        this.size = 0;
        this.head.next = this.tail;
        this.tail.pre = this.head;
    }
    
    public int get(int key) {
        LinkNode node = dataMap.get(key);
        if(null != node){
            deleteNode(node);
            toHead(node);
            return node.data;
        }
        return -1;
    }
    
    public void put(int key, int value) {
        LinkNode oldNode = dataMap.get(key);
        if(oldNode != null){
            deleteNode(oldNode);
        }
        if(size+1>cap){
           deleteNode(tail.pre);
        }
        LinkNode node = new LinkNode(key,value,null,null);
        toHead(node);
        return;
        
    }
    
    private void toHead(LinkNode node){
        LinkNode headNext = head.next;
        node.next = headNext;
        headNext.pre = node;
        head.next = node;
        node.pre = head;
        
        dataMap.put(node.key,node);
        ++size;
    }
    
    private void deleteNode(LinkNode node){
        LinkNode nodeNext = node.next;
        LinkNode nodePre = node.pre;
        
        nodePre.next = nodeNext;
        nodeNext.pre = nodePre;
        
        node.next = null;
        node.next = null;
        dataMap.remove(node.key);
        --size;
    }
    
}

class LinkNode{
    int key;
    int data;
    LinkNode next;
    LinkNode pre;
    
    LinkNode(int key,int data,LinkNode next,LinkNode pre){
        this.key = key;
        this.data = data;
        this.next = next;
        this.pre = pre;
    }
    
    LinkNode(){
        
    }
}
```

