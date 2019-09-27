写一个LRU算法的例子

```java
package jz;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;


public class LRUCacheTry {
    private Map<Integer,DlinkeNode> map = new ConcurrentHashMap<>();
    //实际空间
    private int size;
    //能够承载的容量
    private int capacity;
    DlinkeNode head;
    DlinkeNode tail;

    public LRUCacheTry(int capacity){
        this.size=0;
        this.capacity=capacity;
        head = new DlinkeNode();
        tail = new DlinkeNode();
        head.next = tail;
        tail.next = head;
    }

    /**
     * 定义Node类，也就是写一个双向链表，存入的key和value都是int，key和value可以根据题目改
     */
    class DlinkeNode{
        int key;
        int val;
        DlinkeNode prev;
        DlinkeNode next;
    }

    /**
     * 每次被get出来，都移动到头节点
     *
     * 移动的方式：从之前的位置删除，添加到头节点
     */
    private void moveToHead(DlinkeNode node){
        deleteNode(node);
        insertHead(node);
    }

    private void insertHead(DlinkeNode node){
        node.prev = head;
        node.next = head.next;

        head.next.prev = node;
        head.next = node;
    }

    private void deleteNode(DlinkeNode node){
        DlinkeNode prev = node.prev;
        DlinkeNode next = node.next;

        prev.next = next;
        next.prev = prev;

    }

    private DlinkeNode popTail(){
        DlinkeNode node = tail.prev;
        deleteNode(node);
        return node;
    }


    public int get(int key,int val){
        //从hashmap里面把node拿出来，毕竟map才是真正存储的，而链表只是用来删除的
        DlinkeNode node = map.get(key);
        //如果node没有，返回-1
        if (node==null){
            return -1;
        }
        return node.val;
    }

    public void put(int key,int val){
        DlinkeNode node = map.get(key);
        if (node == null){
            DlinkeNode newNode = new DlinkeNode();
            newNode.key = key;
            newNode.val = val;
            //放元素进去
            map.put(key,newNode);
            insertHead(newNode);
            ++size;

            //如果添加完之后发现超过capacity，就从队尾pop出来删掉。
            if (size>capacity){
                DlinkeNode node1 = popTail();
                map.remove(node1.key);
                size--;
            }
           
        }else {
            node.val = val;
            moveToHead(node);
        }
    }
}

```
