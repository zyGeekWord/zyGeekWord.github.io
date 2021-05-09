---
title: 数据库连接池之Druid--配置篇
date: 2021-03-23 21:46:12
tags: Java
categories: 日常学习
---

日常开发中数据库是我们都要使用的，使用数据库也就意味着必须建立项目和数据之间的联系，所以我们需要使用数据库连接池，目前我选择使用druid连接池，该连接池是阿里开源的，功能非常强大，最大的特点是可以监控sql和可视化界面，目前各种数据库连接池的特点如下图所示

![image-20210330202641268](https://aaaas.oss-cn-beijing.aliyuncs.com/image-20210330202641268.png)

**步入正题**

<!-- more -->

druid有两种配置方式，阿里开源框架中为了springboot提供了封装好的引用包和druid原生的包，分别为

```xml
<!-- 此依赖为druid原生的包 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.22</version>
</dependency>

<!-- 此依赖为springboot--druid的包 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>

```

## 1. 引用springboot-druid的配置

引用springboot-druid包后，当前包中已经为我们配置好了配置文件的映射方法，所以我们只需要在配置文件中添加配置就好，配置文件如下所示

```xml
spring.datasource.druid.driver-class-name=com.mysql.jdbc.Driver
#以下为druid增加的配置
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
# 初始化时建立物理连接的个数
spring.datasource.druid.initial-size=50
# 最小连接池数量
spring.datasource.druid.min-idle=50
# 最大连接池数量
spring.datasource.druid.max-active=10
# 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁
spring.datasource.druid.max-wait=60000
# 有两个含义：
#1) Destroy线程会检测连接的间隔时间，如果连接空闲时间大于等于minEvictableIdleTimeMillis则关闭物理连接。
#2) testWhileIdle的判断依据，详细看testWhileIdle属性的说明
spring.datasource.druid.time-between-eviction-runs-millis=60000
# 连接保持空闲而不被驱逐的最小时间
spring.datasource.druid.min-evictable-idle-time-millis=30000
# 用来检测连接是否有效的SQL,如果未null,testOnBorrow,testOnReturn,testWhileIdle不会起作用
spring.datasource.druid.validation-query=SELECT 1
# 检测链接是否有效的超时时间,单位秒;底层调用jdbc Statement对象的void setQueryTimeout(int seconds)方法
spring.datasource.druid.validation-query-timeout=10
# 建议配置为true,不影响性能,并且保证安全性.申请连接的时候检测,如果空闲时间大于timeBetweenEvictionRunsMills,执行validationQuery检测连接是否有效.
spring.datasource.druid.test-while-idle=true
# 申请连接时执行validationQuery检测连接是否有效,做了这配置会降低性能
spring.datasource.druid.test-on-borrow=false
# 归还连接时执行validationQuery检测连接是否有效,做了这配置会降低性能
spring.datasource.druid.test-on-return=false
# 是否缓存preparedStatement,即PSCache,PSCache对支持游标的数据库性能提升巨大,如oracle,mysql下建议关闭
spring.datasource.druid.pool-prepared-statements=false
# 要启用PSCache,必须配置大于0,当大于0时,上面配置自动修改为true;Druid中不会存在Oracle下PSCache占用内存过多的问题,可以把数值配置大一些,如100
spring.datasource.druid.max-pool-prepared-statement-per-connection-size=20
spring.datasource.druid.filter.stat.enabled=true
spring.datasource.druid.filter.stat.merge-sql=true
spring.datasource.druid.filter.stat.slow-sql-millis=5000
#WebStatFilter监控配置
# 这个教程很保姆,参考https://blog.csdn.net/qq_45173404/article/details/109075810
spring.datasource.druid.web-stat-filter.enabled=true
spring.datasource.druid.web-stat-filter.url-pattern=/*
spring.datasource.druid.web-stat-filter.exclusions=*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*
spring.datasource.druid.web-stat-filter.session-stat-enable=true
spring.datasource.druid.web-stat-filter.session-stat-max-count=100
spring.datasource.druid.stat-view-servlet.enabled=true
spring.datasource.druid.stat-view-servlet.url-pattern=/druid/*
spring.datasource.druid.stat-view-servlet.reset-enable=true
#页面登录用户名
spring.datasource.druid.stat-view-servlet.login-username=admin
#页面登录用户密码
spring.datasource.druid.stat-view-servlet.login-password=admin
```

## 2. 在我们引用原生的druid的情况下，我们需要手写druid的配置类，配置读取配置文件的格式

 配置类如下所述

```java
import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.support.http.StatViewServlet;
import com.alibaba.druid.support.http.WebStatFilter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
public class DruidConfig {
    /**
     *  主要实现WEB监控的配置处理
     */
    @Bean
    public ServletRegistrationBean druidServlet() {
        // 现在要进行druid监控的配置处理操作
        ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(
                new StatViewServlet(), "/druid/*");
        // 白名单,多个用逗号分割， 如果allow没有配置或者为空，则允许所有访问
        servletRegistrationBean.addInitParameter("allow", "127.0.0.1,172.29.32.54");
        // 黑名单,多个用逗号分割 (共同存在时，deny优先于allow)
        servletRegistrationBean.addInitParameter("deny", "192.168.1.110");
        // 控制台管理用户名
        servletRegistrationBean.addInitParameter("loginUsername", "admin");
        // 控制台管理密码
        servletRegistrationBean.addInitParameter("loginPassword", "eju1314");
        // 是否可以重置数据源，禁用HTML页面上的“Reset All”功能
        servletRegistrationBean.addInitParameter("resetEnable", "false");
        return servletRegistrationBean ;
    }
@Bean
public FilterRegistrationBean filterRegistrationBean() {
    FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean() ;
    filterRegistrationBean.setFilter(new WebStatFilter());
    //所有请求进行监控处理
    filterRegistrationBean.addUrlPatterns("/*"); 
    //添加不需要忽略的格式信息
    filterRegistrationBean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*.css,/druid/*");
    return filterRegistrationBean ;
}

@Bean
@ConfigurationProperties(prefix = "spring.datasource")
public DataSource druidDataSource() {
    return new DruidDataSource();
}
}
```
配置完配置类后，我们的配置文件中配置如下所示

```xml
spring.datasource.name=dev
# 基本属性
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test?autoReconnect=true&useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&zeroDateTimeBehavior=convertToNull&useSSL=false
spring.datasource.username=root
spring.datasource.password=123456
# 可以不配置，根据url自动识别，建议配置
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
#=================================以下为druid增加的配置==========================
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
# 初始化连接池个数
spring.datasource.initialSize=5
# 最小连接池个数——》已经不再使用，配置了也没效果
spring.datasource.minIdle=2
 # 最大连接池个数
spring.datasource.maxActive=20
# 配置获取连接等待超时的时间，单位毫秒，缺省启用公平锁，并发效率会有所下降
spring.datasource.maxWait=60000
 # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
spring.datasource.timeBetweenEvictionRunsMillis=60000
# 配置一个连接在池中最小生存的时间，单位是毫秒
spring.datasource.minEvictableIdleTimeMillis=300000
# 用来检测连接是否有效的sql，要求是一个查询语句。
# 用来检测连接是否有效的sql，要求是一个查询语句。
spring.datasource.validationQuery=SELECT 1 FROM DUAL
# 建议配置为true，不影响性能，并且保证安全性。
# 申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。
spring.datasource.testWhileIdle=true
# 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能
spring.datasource.testOnBorrow=false
# 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能
spring.datasource.testOnReturn=false
# 打开PSCache，并且指定每个连接上PSCache的大小
spring.datasource.poolPreparedStatements=true
spring.datasource.maxPoolPreparedStatementPerConnectionSize=20
# 通过别名的方式配置扩展插件，多个英文逗号分隔，常用的插件有： 
# 监控统计用的filter:stat
# 日志用的filter:log4j
# 防御sql注入的filter:wall
spring.datasource.filters=stat,wall,log4j
# 通过connectProperties属性来打开mergeSql功能；慢SQL记录
spring.datasource.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
# 合并多个DruidDataSource的监控数据
spring.datasource.useGlobalDataSourceStat=true
```