---
title: List
date: 2021-03-11 22:22:12
tags: Java
categories: 日常学习
---

首先，List集合中有几个子类，分别是ArraryList，Vector，LinkList。下面我先现主要说一下最常用的ArrayList。

#### ArrayList

线程不安全，为什么呢？通过源码我们可以看到，它没有加Synchronized线程锁。其次他是**有序**的，查询快，但不利于经常修改的场景。

<!-- more -->

**原因：**

ArrayList中数据在内存中是连续的成块的，所以查询时候，直接遍历内存即可，插入或者删除时，需要把该节点之后的所有数据全都遍历一遍，为什么说是连续成块的呢？因为ArrayList底层实现为数组。

![image-20210411225555431](https://aaaas.oss-cn-beijing.aliyuncs.com/image-20210411225555431.png)

ArraryList的扩容机制(自动扩容)

ArrayList源码中的grow()方法，在ArrayList执行插入时，如果容量不足，会扩容1.5倍

![image-20210411230127426](https://aaaas.oss-cn-beijing.aliyuncs.com/image-20210411230127426.png)

List的最顶层是Collection类

![image-20210411230238064](https://aaaas.oss-cn-beijing.aliyuncs.com/image-20210411230238064.png)

##### ArrayList集合的特点：

* ArrayList基于数组实现，是一个动态数组，可自动增长
* ArrayList线程不安全，因为没有加锁
* ArrayList实现了serializable接口，因此它支持序列化
* 它实现了RandomAccess接口，支持快速随机访问
* 实现了Cloneable接口，能被clone

#### LinkList

**定义**

```java
List<String> stringList = new LinkedList<>();
List<String> demo = new ArrayList<>();
List<String> def = new LinkedList<>(demo);
//可以直接传递已有集合
```

LinkList内部使用基于链表的数据结构存储，LinkList有一个内部类作为存放元素的单元，里面有三个属性，用来存放元素本身以及前后两个单元的引用，另外LinkList内部有一个header属性，用来表示起始位置。

LinkList是采用双向链表实现的，所以它也具有链表的特点，每一个元素的地址不连续，通过引用找到当前节点上的一个节点和下一个节点，即插入和删除效率比较高，而get和set则较为低效

**LinkList的扩容机制：**

它的扩容机制是添加多少增长多少。

#### ArrayList和LinkList的区别

* ArrayList是基于动态数组的数据结构，LinkList是基于链表的数据结构。
* 对于随机访问元素，ArrayList获取时间复杂度是O(1),但是删除数据确实开销很大的，因为需要重排数组中所有的数据，ArrayList想要get(index)时，直接返回index位置上的元素，而LinkList需要通过for循环进行查找，虽然LinkList已经在查找中做了优化，但还是较慢。
* 对于添加或删除操作，add()和remove()，LinkList更快，首先因为LinkList底层是双向链表结构，不需要改变数组大小，也没有自动扩容，这是ArrayList的缺点，它的时间复杂度为O(n),而LinkList的时间复杂度为O(1).ArrayList在插入数据时还需要更新索引，ArrayList在插入或删除时主要好事的是System.arraycopy()动作，它会移动index后面所有的元素，LinkList耗时的是要先通过for循环找到index，然后直接插入或删除，所以无法对比两者谁的性能更好，只能说更合适哪种。

#### 适用场景

大部分情况下，ArrayList更受欢迎，但是有些情况下LinkList更优秀。

* 当不需要随机读取第n个元素时。
* 不需要大量的插入或者删除时