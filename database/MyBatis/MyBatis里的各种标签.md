# MyBatis里的各种标签

本文将着重梳理MyBatis的XML映射文件中所用到的各类标签的具体作用，首先从几个顶级元素开始。

#### select

查询语句是MyBatis最常用的元素之一，select示例：

```xml
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
```

该语句的标识符为"selectPerson"，语句接受了一个int类型的参数"#{id}"，并返回一个HashMap类型的对象，其中的键是列名，值便是结果行中的对应值。

当然，除了示例中的属性外，还有其它的属性可以使用：

| **属性**      | **描述**                                                     |
| ------------- | ------------------------------------------------------------ |
| id            | 在命名空间中唯一的标识符，可以被用来引用这条语句。           |
| parameterType | 将会传入这条语句的参数类型的全名或别名。这个属性是可选的，因为MyBatis 可以通过类型处理器推断出具体传入参数的类型，默认值为未设置（unset）。 |
| resultType    | 语句返回类型的全名或别名。注意如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身。属性resultType和resultMap不能同时使用。 |
| resultMap     | 外部 resultMap 的命名引用。使用结果集映射标签来映射查询结果。属性resultType和resultMap不能同时使用。 |
| useCache      | 将其设置为 true 后，将会导致本条语句的结果被二级缓存缓存起来，默认值：true。 |
| flushCache    | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：false。 |
| databaseId    | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。 |
| resultSets    | 这个设置仅对多结果集的情况适用。它将列出语句执行后返回的结果集并给每个结果集一个名称，名称是逗号分隔的。 |

#### insert, update, delete

数据变更语句示例：

```xml
<insert id="insertAuthor">
  insert into Author (id,username,password,email,bio)
  values (#{id},#{username},#{password},#{email},#{bio})
</insert>

<update id="updateAuthor">
  update Author set
    username = #{username},
    password = #{password},
    email = #{email},
    bio = #{bio}
  where id = #{id}
</update>

<delete id="deleteAuthor">
  delete from Author where id = #{id}
</delete>
```

数据变更参数所使用的属性都是近似的：

| **属性**         | **描述**                                                     |
| ---------------- | ------------------------------------------------------------ |
| id               | 命名空间中的唯一标识符，可被用来代表这条语句。               |
| parameterType    | 将会传入这条语句的参数类型的全名或别名。这个属性是可选的，因为MyBatis 可以通过类型处理器推断出具体传入参数的类型，默认值为未设置（unset）。 |
| flushCache       | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：true。 |
| useGeneratedKeys | （仅对 insert 和 update 有用）这会令 MyBatis 使用 JDBC 的getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系数据库管理系统的自动递增字段），默认值：false。 |
| keyProperty      | （仅对 insert 和 update 有用）唯一标记一个属性，MyBatis 会通过getGeneratedKeys 的返回值或者通过 insert 语句的 selectKey 子元素设置它的键值，默认值：未设置（unset）。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| keyColumn        | （仅对 insert 和 update 有用）通过生成的键值设置表中的列名，这个设置仅在某些数据库（像 PostgreSQL）是必须的，当主键列不是表中的第一列的时候需要设置。如果希望使用多个生成的列，也可以设置为逗号分隔的属性名称列表。 |
| databaseId       | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。 |

#### selectKey

selectKey元素可以用来灵活处理ID生成问题（一般不会这么做），示例代码如下：

```xml
<insert id="insertAuthor">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    select CAST(RANDOM()*1000000 as INTEGER) a from SYSIBM.SYSDUMMY1
  </selectKey>
  insert into Author
    (id, username, password, email,bio, favourite_section)
  values
    (#{id}, #{username}, #{password}, #{email}, #{bio}, #{favouriteSection,jdbcType=VARCHAR})
</insert>
```

selectKey属性说明：

| 属性          | 描述                                                         |
| :------------ | :----------------------------------------------------------- |
| keyProperty   | selectKey语句结果应该被设置的目标属性。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| keyColumn     | 匹配属性的返回结果集中的列名称。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| resultType    | 结果的类型。MyBatis通常可以推断出来，但是为了更加精确，写上也不会有什么问题。MyBatis 允许将任何简单类型用作主键的类型，包括字符串。如果希望作用于多个生成的列，则可以使用一个包含期望属性的Object或一个Map。 |
| order         | 这可以被设置为BEFORE或AFTER。如果设置为BEFORE，那么它会首先生成主键，设置keyProperty然后执行插入语句。如果设置为AFTER，那么先执行插入语句，然后是selectKey中的语句，这和Oracle数据库的行为相似，在插入语句内部可能有嵌入索引调用。 |
| statementType | 与前面相同，MyBatis 支持STATEMENT，PREPARED和CALLABLE语句的映射类型，分别代表 PreparedStatement 和 CallableStatement 类型。 |

#### sql, include, property

sql元素可以用来定义可重用的SQL代码段，当需要使用通过sql元素定义的代码段时，使用include元素配合property元素引入即可。使用示例如下：

```xml
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>

<select id="selectUsers" resultType="map">
  select
    <include refid="userColumns"><property name="alias" value="t1"/></include>,
    <include refid="userColumns"><property name="alias" value="t2"/></include>
  from some_table t1
    cross join some_table t2
</select>
```

参数值也可用于属性值中，示例如下：

```xml
<sql id="sometable">
  ${prefix}Table
</sql>

<sql id="someinclude">
  from
    <include refid="${include_target}"/>
</sql>

<select id="select" resultType="map">
  select
    field1, field2, field3
  <include refid="someinclude">
    <property name="prefix" value="Some"/>
    <property name="include_target" value="sometable"/>
  </include>
</select>
```

#### resultMap

使用resultMap可以方便的建立结果集映射。下面是一个简单示例：

```xml
<resultMap id="userResultMap" type="User">
  <id property="id" column="user_id" />
  <result property="username" column="user_name"/>
  <result property="password" column="hashed_password"/>
</resultMap>

<select id="selectUsers" resultMap="userResultMap">
  select user_id, user_name, hashed_password
  from some_table
  where id = #{id}
</select>
```

resultMap可以支持更为复杂的结果映射，但是由于不是那么常用，这里就不一一列出，详情点击[这里](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html)查看。

#### cache

MyBatis内置了一个方便的缓存机制，若想要启用全局的二级缓存，只需要再映射文件中加上cache元素即可：

```xml
<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
```

其属性含义如下：

| **属性**      | **描述**                                                     |
| ------------- | ------------------------------------------------------------ |
| eviction      | 设置缓存清除策略，具体可用策略见表后。                       |
| flushInterval | 刷新间隔，可设置为任意正整数，单位毫秒。默认为不设置刷新间隔，缓存仅仅会在调用语句时刷新。 |
| size          | 引用数目，可设置为任意正整数，默认值是1024。                 |
| readOnly      | 只读属性，可以被设置为 true 或 false。默认值是 false。       |

可用的清除策略有：

- LRU – 最近最少使用：移除最长时间不被使用的对象。           
- FIFO – 先进先出：按对象进入缓存的顺序来移除它们。           
- SOFT – 软引用：基于垃圾回收器状态和软引用规则移除对象。           
- WEAK – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象。

#### if， choose（when， otherwise）

判断语句通常用来动态控制SQL的内容，若判断条件未通过，该标签下的语句将不会在最终执行语句中出现。示例如下：

```xml
<select id="findActiveBlogWithTitleLike"
     resultType="Blog">
  SELECT * FROM BLOG
  WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
</select>
```

若需要实现比简单if更复杂的判断，可以使用choose标签：

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

#### where

在使用if标签的时候，经常会出现由于判断为假，结果WHERE后边紧跟了AND/OR，导致SQL语句运行出错的情况，我们可以使用where标签来解决这个问题：

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG
  <where>
    <if test="state != null">
         state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```

#### set

类似WHERE搭配if标签会出现的错误问题，同样也会发生在使用SET搭配if标签时，使用set标签可以去除掉标签内末尾的","：

```xml
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
```

#### trim

如果需要对前后缀做一些自定义的控制，使用trim标签即可实现：

```xml
<trim prefix="" suffix="" suffixOverrides="" prefixOverrides="">
	...
</trim>
```

> prefix:在trim标签内sql语句加上前缀。
> suffix:在trim标签内sql语句加上后缀。
> suffixOverrides:指定去除多余的后缀内容。
> prefixOverrides:指定去除多余的前缀内容。

#### foreach

需要对一个集合进行遍历的时候，我们通常使用foreach标签：

```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT *
  FROM POST P
  WHERE ID in
  <foreach item="item" index="index" collection="list"
      open="(" separator="," close=")">
        #{item}
  </foreach>
</select>
```

#### bind

使用bind标签可以让我们创建一个自定义变量：

```xml
<select id="selectBlogsLike" resultType="Blog">
  <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
  SELECT * FROM BLOG
  WHERE title LIKE #{pattern}
</select>
```



###### 参考内容链接

1. [mybatis](http://www.mybatis.org/mybatis-3/zh/java-api.html)