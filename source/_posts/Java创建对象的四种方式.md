---
title: Java创建对象的四种方式
date: 2021-03-11 22:24:07
tags: Java
categories: 日常学习
---

* new对象
* 反射，使用Java.lang.Class或者java.lang.reflect.Construrctor
* 调用对象clone()，注意clone() 有深度和浅度。
* 反序列化手段，调用java.io.ObjectInputStream对象的readObject()方法