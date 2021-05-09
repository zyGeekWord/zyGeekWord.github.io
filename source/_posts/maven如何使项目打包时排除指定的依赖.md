---
title: maven如何使项目打包时排除指定的依赖
date: 2021-03-30 20:06:51
tags: Java
categories: 日常学习
---

打包时，需要将项目中的一些依赖排除出去，查询后使用代码如下

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.58.Final</version>
    <scope>provided</scope>
</dependency>
```

其中，使用了

```xml
<scope>provided</scope>
```

此命令行可以使下项目打包时不将该依赖打包进去。



