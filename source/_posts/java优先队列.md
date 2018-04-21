---
title: Java优先队列 PiorityQueue
author: Kairou Zeng
date: 2017/10/15
tags: [java,优先队列]
---

`PriorityQueue`是个基于优先级堆的极大优先级队列,PriorityQueue的排序不是普通的排序，而是堆排序。

排序方式：
- 自然排序：根据元素的自然顺序来指定排序(参阅Comparable)，也就是数字默认是小的在队列头，字符串则按字典序排列，也可以根据 Comparator来指定，这取决于使用哪种构造方法。
- 定制排序：根据Comparator来指定，使用Comparator类来重写compare(Object o1,Object o2)方法来实现定制排序

PriorityQueue对元素采用的是堆排序，头是按指定排序方式的最小元素。堆排序只能保证根是最大（最小），整个堆并不是有序的。


优先级队列不允许null元素，依靠自然排序的优先级队列还不允许插入不可比较的对象(这样做可能导致ClassCastException)

```java
import java.util.PriorityQueue;  
public class PriorityQueueTest3   
{  
    public static void main(String[] args)   
    {  
        PriorityQueue pq = new PriorityQueue();  
        pq.offer(6);  
        pq.offer(-3);  
        pq.offer(0);  
        pq.offer(9);  
        System.out.println(pq);  
    }  
} 
```
堆排序只会保证第一个元素(根节点)的元素是当前优先队列里面最小(或者最大)的元素，而且每一次变化之后，比如offer()或者poll()之后，都会进行堆重排，所以如果想要按从小到大的顺序取出元素，那么需要用一个for循环,如下：
```java
import java.util.PriorityQueue;  
public class PriorityQueueTest3   
{  
    public static void main(String[] args)   
    {  
        PriorityQueue pq = new PriorityQueue();  
        pq.offer(6);  
        pq.offer(-3);  
        pq.offer(0);  
        pq.offer(9);  
        int len = pq.size();//这里之所以取出.size()大小，因为每一次poll()之后size大小都会变化，所以不能作为for循环的判断条件  
        for(int i=0;i<len;i++){  
            System.out.print(pq.poll()+" ");  
        }  
        System.out.println();  
    }  
}
```
