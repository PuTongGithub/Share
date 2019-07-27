# MyBatis的配置文件

在引入MyBatis的时候，我们都需要指定一个mybatis-config.xml的路径，这个文件里就包含了MyBatis的一些配置信息。根据[MyBatis官网](www.mybatis.org/mybatis-3/zh/configuration.html)介绍，MyBatis的配置文件整体xml结构如下：

> configuration（配置）            
>
> - [properties（属性）](http://www.mybatis.org/mybatis-3/zh/configuration.html#properties)
> - [settings（设置）](http://www.mybatis.org/mybatis-3/zh/configuration.html#settings)
> - [typeAliases（类型别名）](http://www.mybatis.org/mybatis-3/zh/configuration.html#typeAliases)
> - [typeHandlers（类型处理器）](http://www.mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)
> - [objectFactory（对象工厂）](http://www.mybatis.org/mybatis-3/zh/configuration.html#objectFactory)
> - [plugins（插件）](http://www.mybatis.org/mybatis-3/zh/configuration.html#plugins)
> - [environments（环境配置）](http://www.mybatis.org/mybatis-3/zh/configuration.html#environments)
> - [databaseIdProvider（数据库厂商标识）](http://www.mybatis.org/mybatis-3/zh/configuration.html#databaseIdProvider)
> - [mappers（映射器）](http://www.mybatis.org/mybatis-3/zh/configuration.html#mappers)

本文将对这个配置文件进行一个简单的梳理。

### properties（属性）

属性标签可以设定一些引用自外部文件的参数值，使得配置文件可以动态适应多种配置。标签使用示例如下：

```xml
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```

配置完成后的属性值就可以被用来完成动态配置替换了：

```xml
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
  <property name="password" value="${password}"/>
</dataSource>
```

这些配置属性的加载顺序如下：

> - 在 properties 元素体内指定的属性首先被读取。           
> - 然后根据 properties 元素中的 resource 属性读取类路径下属性文件或根据           url 属性指定的路径读取属性文件，并覆盖已读取的同名属性。           
> - 最后读取作为方法参数传递的属性，并覆盖已读取的同名属性。 

### settings（设置）

这个标签用于设定一些MyBatis的重要参数，这里仅给出一个使用示例，若需要了解这些设置的具体含义，可以访问[settings（设置）](http://www.mybatis.org/mybatis-3/zh/configuration.html#settings)。示例如下：

```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

### typeAliase（类型别名）

typeAliase可以用来为一个Java类型的类型全名配置一个可以在MyBatis映射文档中设用的别名，示例如下：

```xml
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
  <typeAlias alias="Comment" type="domain.blog.Comment"/>
  <typeAlias alias="Post" type="domain.blog.Post"/>
  <typeAlias alias="Section" type="domain.blog.Section"/>
  <typeAlias alias="Tag" type="domain.blog.Tag"/>
</typeAliases>
```

另外这个标签还可以用来指定MyBatis的搜索包名。示例如下：

```xml
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```

有一些基本的Java类型已经被MyBatis默认支持了别名，详细内容如下：

| 别名       | 映射的类型 |
| :--------- | :--------- |
| _byte      | byte       |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| object     | Object     |
| map        | Map        |
| hashmap    | HashMap    |
| list       | List       |
| arraylist  | ArrayList  |
| collection | Collection |
| iterator   | Iterator   |

### typeHandlers（类型处理器）

如果想要为一些MyBatis不支持的类型进行查询结果集匹配，那就需要自己实现相关的类型处理器，然后在配置文件中的typeHandlers下引入即可：

```xml
<typeHandlers>
  <typeHandler handler="org.mybatis.example.ExampleTypeHandler"/>
</typeHandlers>
```

### objectFactor（对象工厂）

若设定了自定义的MyBatis对象工厂，则使用该标签引入即可：

```xml
<objectFactory type="org.mybatis.example.ExampleObjectFactory">
  <property name="someProperty" value="100"/>
</objectFactory>
```

### plugins（插件）

MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：

> - Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
> - ParameterHandler (getParameterObject, setParameters)
> - ResultSetHandler (handleResultSets, handleOutputParameters)
> - StatementHandler (prepare, parameterize, batch, update, query) 

通过 MyBatis 提供的强大机制，使用插件是非常简单的，只需实现 Interceptor 接口，并指定想要拦截的方法签名即可。

```java
// ExamplePlugin.java
@Intercepts({@Signature(
  type= Executor.class,
  method = "update",
  args = {MappedStatement.class,Object.class})})
public class ExamplePlugin implements Interceptor {
  private Properties properties = new Properties();
  public Object intercept(Invocation invocation) throws Throwable {
    // implement pre processing if need
    Object returnObject = invocation.proceed();
    // implement post processing if need
    return returnObject;
  }
  public void setProperties(Properties properties) {
    this.properties = properties;
  }
}
```

最后通过plugin标签引入即可：

```xml
<plugins>
  <plugin interceptor="org.mybatis.example.ExamplePlugin">
    <property name="someProperty" value="100"/>
  </plugin>
</plugins>
```

### environments, databaseIdProvider, mappers

剩余3个标签使用较少，在此并不详细列出，若想具体了解请访问[MyBatis官方文档](www.mybatis.org/mybatis-3/zh/configuration.html)。



###### 参考内容链接

1. [MyBatis官方文档](www.mybatis.org/mybatis-3/zh/configuration.html)