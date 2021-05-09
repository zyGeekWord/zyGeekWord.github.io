---
title: HashMap
date: 2021-03-12 22:22:00
tags: Java
categories: 日常学习
---

我们在日常使用中，最常见的Map集合就是HashMap，但HashMap的存储结构是什么样的，这个在初学的时候很少会有人思考。下面我们便开始深入了解HashMap集合。

<!-- more -->

**数据结构：**

* HashMap是一个数组+链表+红黑树的结构组成。
* HashMap初始长度为16，之后每次自动扩容都是原来的两倍。
* HashMap是基于Hash表实现的，每个元素都是一个key-value，内部通过单链表解决冲突，容量不足时自动增长。
* HashMap时为线程安全的，只适用于单线程环境下
* HashMap实现了serializable接口，因此它支持序列化，实现了Cloneable，能被克隆。
* HashMap可以有一个key等于null，value等于null的数据，排在下标为第一个的位置
* HashMap在计算hash值中，将key的hashcode进行了2次hash，以获得更好的散列值，然后对table数组长度取模。

**hashMap是如何解决hash冲突问题的：**

* Map中有一个内部接口Map.Entry,其实它就是一个key-value，当系统决定存储HashMap的key-value时，完全没有考虑Entry中的value，仅仅只是将value当成key的附属，在HashMap没有出现hash冲突时，没有形成单链表时，HashMap查找元素很快，get()能够直接定位到元素，但是，当出现单链表之后，单个bucket里存储的不是一个Entry，而时一个Entery链，系统只能必须按顺序遍历每个Entry，直到找到Entry为止。
* HashMap数据结构图：
* ![文件无法预览。](https://aaaas.oss-cn-beijing.aliyuncs.com/qwerrrr.jpg?Expires=1618752477&OSSAccessKeyId=TMP.3KioeBxRkdq2NaeZHzdxqDq3XXHQAv7Y19EeLbdaExZfH2umzHyZ1tqjTei6GMUzFosMjRwCJqmPFyCR3oqszyWEVTFeRU&Signature=KDZZ7%2FRwP0anzXr8apycnAxS5SE%3D)

* table是hash数组，数组的每个元素的，欧式一个单链表的头节点，链表是用来解决冲突的，当多个不同的key映射到数组的同一位置时，就将其放入到单链表中。

* 在HashMap插入时，如果key.hash()相等时，如果判断是否覆盖呢？
  * 在hashcode相同后，会调用equales()进行判断，两者是否为同一对象。
  * equels()默认是对比内存地址，因此Java强调重写equels()时，重写hashcode()

红黑树时jdk8中对HashMap的一个变更，在JDK7之前，HashMap采用数组链表的形式存储数据，我们知道，优秀的hash算法应避免碰撞的发生，假如开发者使用了错误的hash算法O(1)级别的数组查询退化到O(n)级别的数组查询退化到O(n)级别的链表查询，因此JDK8中引入了红黑树，当一个结点的链表长度小于6时，链表会转换成红黑树，提高查询效率，当长度小于6时，又会退回到链表结构。

**HashMap为什么用的是Object类型？**

首先我们要想，hashmap的数据结构是什么？数组+链表+红黑树

存储元素时，用的是hash表存储，每存储一个对象，都会调用hashcode()方法算出hash值，这里就是重点，基本数据类型是无法调用hashcode()方法的，但是所有的包装类都可以，而最后Object是所有包装类的父类。

