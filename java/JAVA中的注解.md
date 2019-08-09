# JAVA中的注解

在 Java SE 5.0 版本中引入的注解被spring boot框架广泛使用，它为spring框架带来了极为轻便的去XML化配置方法。注解可以放在一个类、一个方法甚至是一个对象上，为被注解的内容带来一些额外的提升，这个提升可以是一些注释性的内容，可以是指定地编译器工作，有的注解甚至还可以影响程序运行期的行为。本篇文章将整理介绍Java中的注解如何使用。



### 注解的使用

同`classs` 和 `interface` 类似，如果需要定义一个注解，我们可以使用`@interface`关键字来做到：

```java
public @interface TestAnnotation {
}
```

而使用一个注解只需要在它的名字前加上`@`后，放在在需要的地方就可以了：

```java
@TestAnnotation
public class Test {
}
```

当然，如果想让一个注解做更多的事情，按照我们的想法去工作，那我们还需要做很多事情，下面我们来逐一介绍。



### 元注解

元注解有5个（Java 8），它们通常被放在一个自定义注解的类上，是用来定义一个注解的基本属性的。

#### @Retention

Retention意为保留期，这个注解被用来定义自定义注解的生存期，具体取值如下：

> RetentionPolicy.SOURCE  注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视。
> RetentionPolicy.CLASS  注解只被保留到编译进行的时候，它并不会被加载到 JVM 中。
> RetentionPolicy.RUNTIME  注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，所以在程序运行时可以获取到它们。

使用示例：

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
}
```

#### @Target

这个注解用来指定自定义注解的使用位置，具体取值如下：

> ElementType.ANNOTATION_TYPE  可以给一个注解进行注解
> ElementType.CONSTRUCTOR  可以给构造方法进行注解
> ElementType.FIELD  可以给属性进行注解
> ElementType.LOCAL_VARIABLE  可以给局部变量进行注解
> ElementType.METHOD   可以给方法进行注解
> ElementType.PACKAGE  可以给一个包进行注解
> ElementType.PARAMETER  可以给一个方法内的参数进行注解
> ElementType.TYPE  可以给一个类型进行注解，比如类、接口、枚举

可以为一个注解同时指定多个使用位置，例如：

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
public @interface Deprecated {
}
```

#### @Inherited

这个注解用来定义自定义注解的继承性：如果某个类使用了@Xxx注解（定义@Xxx注解时使用了@Inherited修饰）修饰，则这个类的子类将自动被@Xxx修饰。

#### @Documented

被该元注解修饰的注解类将会被javadoc工具提取成文档。

#### @Repeatable

Repeatable是可重复的意思，被这个注解修饰的注解类可以在同一个位置重复使用，具体可以通过一个示例来理解：

```java
@Repeatable(Persons.class)
@interface Person{
	String role default "";
}

@interface Persons {
	Person[]  value();
}

@Person(role="coder")
@Person(role="UI")
@Person(role="PM")
public class SuperMan{
	
}

// SuperMan的注解等同于
// @Persons({@Person(role="coder"),@Person(role="UI"),@Person(role="PM")})
```



#### 注解的属性

自定义的注解中可以有变量，但是没有方法，这些变量以“无形参方法”形式来定义，可以带有默认值，示例如下：

```java
@Documented
@Target(ElementType.METHOD)
@Inherited
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotataion{
    String name();
    String website() default "hello";
    int revision() default 1;
}
```

定义了变量的注解就可以在使用注解时指定变量值了：

```java
@TestAnnotation(name="test",revision=3,website="hello annotation")
public void test {
}
```

如果一个注解内仅仅只有一个名字为 value 的属性时，应用这个注解时可以直接接属性值填写到括号内。

```java
public @interface Check {
	String value();
}

@Check("hi")
int a;
```



### 注解解析

利用反射，我们就可以获取到某个元素的注解以及该注解中所携带的信息。

假设我们有一个自定义注解类：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
	
	public int id() default -1;
	
	public String msg() default "Hi";

}
```

我们利用一个测试类来测试它：

```java
@TestAnnotation(msg = "hello!")
public class Test {
    
	public static void main(String[] args) {
		boolean hasAnnotation = Test.class.isAnnotationPresent(TestAnnotation.class);		
		if ( hasAnnotation ) { 
            TestAnnotation testAnnotation = Test.class.getAnnotation(TestAnnotation.class);
			
			System.out.println("id:"+testAnnotation.id());
			System.out.println("msg:"+testAnnotation.msg());
		}
	}
}
```

运行的结果将是：

```
id:-1
msg:hello!
```

至此Java注解的基本内容就介绍完了，另外Java中还有一些内置的注解，注解也还有很多潜力可以开发，点击参考内容链接了解更多内容。



###### 参考内容连接

1. [Java注解（Annotation）详解](https://www.jianshu.com/p/596d389282a0)
2. [秒懂，Java 注解 （Annotation）你可以这样学](https://blog.csdn.net/briblue/article/details/73824058)
3. [Java 注解（Annotation）：带你一步步探索神秘的注解（Annotation）](https://cloud.tencent.com/developer/article/1394522)