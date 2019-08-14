# 在SpringBoot中使用MyBatis

MyBatis是一款普遍使用且功能强大的持久层框架。使用MyBatis可以避免操作繁琐的JDBC代码，通过配置XML文件或者使用注解，即可完成SQL语句的定制、管理，以及结果集的映射。本篇文章将简单整理如何在SpringBoot中引入MyBatis。

### 添加Maven依赖

首先我们需要将MyBatis相关包引入到工程中，这里我们使用Maven来进行依赖管理。除了SpringBoot相关的依赖以外，将如下MyBatis依赖添加至POM文件中即可。

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>{mybatis-spring-boot-starter-version}</version>
</dependency>
```

想要选择合适的依赖版本，请查看MyBatis的官方[GitHub](https://github.com/mybatis/spring-boot-starter)。

### application.properties中的相关配置

这里整理了一些需要放在properties配置文件中，与MyBatis有关的配置信息：

```properties
# 数据库连接基本信息
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=123456
# 驱动类配置可省略，spring boot可根据url检出驱动类型
# spring.datasource.driver-class-name=com.mysql.jdbc.Driver

# mybatis-config.xml文件位置
mybatis.config-location=classpath:mybatis-config.xml
# 指定mapper xml文件位置，仅在mapper xml与mapper class不在同一个目录下时有效
mybatis.mapper-locations=com/me/core/mapper/xml/*.xml
# 搭配@Mapper注解使用，指定自动扫描Mappers的包路径
mybatis.type-aliases-package=com.me.core.mapper
```

更多配置参数说明详见[说明文档](http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/index.html)。

### MyBatis的常用搭配

#### 数据库连接池

在SpringBoot2.0中，Hikari是默认的数据库连接池实现，因此我们不需要添加任何东西就可以直接使用它了，相关配置如下：

```properties
## Hikari连接池的设置
# 连接池中维护的最小空闲连接数
spring.datasource.hikari.minimum-idle=5
# 最大池大小
spring.datasource.hikari.maximum-pool-size=15
# 配置从池返回的连接的默认自动提交行为。默认值为true
spring.datasource.hikari.auto-commit=true
# 允许连接在连接池中空闲的最长时间（以毫秒为单位）
spring.datasource.hikari.idle-timeout=30000
# 池中连接关闭后的最长生命周期（以毫秒为单位）
spring.datasource.hikari.max-lifetime=900000
# 客户端等待连接池连接的最大毫秒数（使用中的连接永远不会退役，只有当它关闭时才会在最长生命周期后删除。）
spring.datasource.hikari.connection-timeout=15000
```

若想要配置多数据源，可以参考这篇文章来实现：[Spring Boot HikariCP 一 ——集成多数据源](https://blog.csdn.net/qq_35981283/article/details/78846892)。

在国内可能大家用的更多的是Druid，关于SpringBoot与Druid的集成可以参考这篇文章：[Spring Boot 集成 Druid + Mybatis](https://www.jianshu.com/p/9dc6f4418a06)。

#### MyBatis Plus

MyBatis Plus是一款MyBaits增强工具，在MyBatis的基础上进行了功能拓展与增强，详细内容可以参考官网介绍：[MyBatis-Plus](https://mp.baomidou.com/)。



###### 参考内容链接

1. [Spring Boot(六)：如何优雅的使用 Mybatis](http://www.ityouknow.com/springboot/2016/11/06/spring-boot-mybatis.html)
2. [IDEA不编译src的java目录下的xml文件问题及解决](https://segmentfault.com/a/1190000014014871)
3. [MyBatis-Spring-Boot 使用总结](https://www.cnblogs.com/larryzeal/p/5874107.html)
4. [Spring Boot 2.x HikariCP使用详解](http://www.leftso.com/blog/520)

