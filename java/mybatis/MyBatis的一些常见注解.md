# MyBatis的一些常见注解

在使用SpringBoot的时候我们通常都会尽量利用注解而不是配置文件来完成一些相关配置内容，MyBatis同样也提供了丰富的注解方式来尽可能的替代配置。本篇文章将简单举例一些MyBatis使用时的常用注解。

### @MapperScan, @Mapper

`@MapperScan`用来指定自动扫描Mapper的包路径，与配置项"mybatis.type-aliases-package"的作用是一样的。在指定的包路径下所有带有`@Mapper`注解的类都将会被MyBatis作为Mapper来加载。

在SpringBoot启动类上添加`@MapperScan`注解：

```java
@SpringBootApplication  
@MapperScan("com.me.mapper")  
public class App {  
    public static void main(String[] args) {  
       SpringApplication.run(App.class, args);  
    }  
}  
```

在指定包内的Mapper类上添加`@Mapper`注解：

```java
@Mapper  
public interface DemoMapper {  
    public void save(Demo demo);  
}  
```

### @Param

通过`@Param`注解可以让我们在Mapper类的方法中传递参数时指定参数名，示例如下：

```java
int insert(@Param("name") String name, @Param("age") Integer age);
```

这样，使用`@Param`定义的参数`name`即对应了SQL中的`#{name}`。

### @Select, @Insert, @Update, @Delete

正如注解名所示，这些注解便是用来定义增删改查操作的，具体实例如下：

```java
public interface UserMapper {

    @Select("SELECT * FROM user WHERE name = #{name}")
    User findByName(@Param("name") String name);

    @Insert("INSERT INTO user(name, age) VALUES(#{name}, #{age})")
    int insert(@Param("name") String name, @Param("age") Integer age);

    @Update("UPDATE user SET age=#{age} WHERE name=#{name}")
    void update(User user);

    @Delete("DELETE FROM user WHERE id =#{id}")
    void delete(Long id);
}
```

### @Result, @Results

对于查询操作，当我们需要进行结果映射的时候，我们便可以使用`@Result`与`@Results`注解来实现：

```java
@Results({
    @Result(property = "name", column = "name"),
    @Result(property = "age", column = "age")
})
@Select("SELECT name, age FROM user")
List<User> findAll();
```

除了上述提到的这些注解外，MyBatis还提供了更多作用更为复杂的注解供使用，具体详情可以查看[MyBatis官方文档](http://www.mybatis.org/mybatis-3/zh/java-api.html)。



###### 参考内容链接

1. [Spring Boot MyBatis注解：@MapperScan和@Mapper](https://blog.csdn.net/m0_37450089/article/details/81153713)
2. [Spring Boot中使用MyBatis注解配置详解](http://blog.didispace.com/mybatisinfo/)
3. [MyBatis官方文档](http://www.mybatis.org/mybatis-3/zh/java-api.html)

