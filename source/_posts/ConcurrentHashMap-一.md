---
title: ConcurrentHashMap(一)
date: 2021-04-09 21:41:18
tags: Java
categories: 日常
---

当我们提到Map集合的时候，我们第一想法都是HashMap和HashTable，但是HashMap和HashTable都有自己的缺点。

首先HashMap它跟HashTable比，HashMap因为没有加锁，所以它的效率很高，但同时，它在多线程环境下，它无法保证数据同步，反之HashTable加了锁，但是同时也使它的效率降低，所以，在jdk1.5中加入了ConcurrentHashMap，但是ConcurrentHashMap发扬光大的还是在jdk1.7和1.8版本中，这两个版本的ConcurrentHashMap有着非常巨大的区别，本章我们先初步介绍这两个版本的ConcurrentHashMap。

## jdk1.7