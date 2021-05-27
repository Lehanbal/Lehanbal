---
title: LRU的JAVA的双向链表实现
date: 2020-10-30 23:40:31
tags:
- 操作系统
- JAVA
- 数据结构
categories:
- 数据结构
---

LRU（Least Recently Used）最近最久未使用，一种页面置换算法。

这里使用了双向链表实现，head表示队尾当中最后的进程，而tail则表示刚刚被使用的进程。通过移动节点的位置模拟被使用的过程。



## 实现代码

```java
package Lehanbal.Algorithms.Base;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

public class LRUCache {
    /**
     * 节点
     */
    private class Node {
        public Node pre;
        public Node next;
        public int key;
        public int val;

        public Node(int k, int v) {
            this.key = k;
            this.val = v;
            this.pre = null;
            this.next = null;
        }

        @Override
        public String toString() {
            return "Node{" +
                    "key=" + key +
                    ", val=" + val +
                    '}';
        }
    }

    /**
     * 手写双端队列，队头是不常使用的元素，队尾是刚刚使用的元素
     */
    private class DequeList {
        public Node head;
        public Node tail;

        public DequeList() {
            this.head = null;
            this.tail = null;
        }

        /**
         * 将元素移动到尾部（刚刚使用过）
         * @param n
         */
        public void moveToTail(Node n){
            //如果元素为空或者已经在尾巴了，就不要动了
            if(n == null || n == tail) return;
            //如果元素在头的地方，那就让头节点往后面移动一位，并且将头节点前面的元素置空
            if(head == n){
                head = n.next;
                head.pre = null;
            }else{
                //如果在中间的位置，那就让元素的前驱结点直接指向后继节点（这里是为了让n节点的前驱后继都越过n，把n空出来）
                n.pre.next = n.next;
                n.next.pre = n.pre;
            }

            //把n放到tail后面，再移动tail到n的位置，注意写法，防止内存泄漏
            tail.next = n;
            n.pre = tail;
            n.next = null;
            tail = tail.next;
        }

        /**
         * 添加新节点
         * @param n
         */
        public void addToTail(Node n){
            if(n == null) return;
            if(head == null){
                head = n;
                tail = n;
            }else{
                tail.next = n;
                n.pre = tail;
                tail = tail.next;
            }
        }

        /**
         * 删除最近最久未使用节点
         * @return 返回删除的节点
         */
        public Node removeHead(){
            if(head == null) return null;
            Node n = head;
            if(head == tail){
                head = null;
                tail = null;
            }else{
                head = head.next;
                head.pre = null;
            }
            return n;
        }

        public List<Node> trave(){
            List<Node> res = new ArrayList<>();
            Node iter = tail;
            while(iter != null){
                res.add(iter);
                iter = iter.pre;
            }
            return res;
        }
    }

    private DequeList list;
    private HashMap<Integer, Node> map;
    private int capacity = 10;

    public LRUCache(int capacity){
        this.capacity = capacity;
        this.list = new DequeList();
        this.map = new HashMap<>();
    }

    public int get(int key){
        if(!map.containsKey(key)) return -1;
        Node n = map.get(key);
        list.moveToTail(n);
        return n.val;
    }

    public void put(int key, int val){
        if(!map.containsKey(key)){
            Node n = new Node(key, val);
            map.put(key, n);
            list.addToTail(n);

            if(map.size() > capacity){
                Node rm = list.removeHead();
                map.remove(rm.key);
            }
        }else{
            Node n = map.get(key);
            n.val = val;
            list.moveToTail(n);
        }
    }

    public void traveList(){
        List<Node> trave = list.trave();
        System.out.println(trave);
    }
}
```

测试方法：

```java
    @Test
    public void Test07(){
        LRUCache LRU = new LRUCache(5);
        LRU.put(1,1001);
        LRU.put(2,1002);
        LRU.put(3,1003);
        LRU.put(4,1004);
        LRU.put(5,1005);
        LRU.put(1,1002);
        LRU.put(7,1007);
        LRU.traveList();
        LRU.get(3);
        LRU.traveList();
    }
```

![image-20201030234723878](https://gitee.com/lehanbal/blog-image/raw/master/img/image-20201030234723878.png)