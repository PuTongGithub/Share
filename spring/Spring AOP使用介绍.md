# Spring AOP使用介绍

我们经常会遇到一些可以使用面向切面编程（AOP）快捷实现的需求，如：方法耗时记录、操作日志记录、分布式锁等。Spring中提供了一个使用起来非常便捷的AOP框架供我们使用。本篇文章将梳理介绍Spring中基于注解实现AOP编程的相关内容。

### 引入依赖

若想要在Spring Boot中使用AOP相关内容，需要在POM文件中引入以下依赖：

```xml
<dependency>  
	<groupId>org.springframework.boot</groupId>  
	<artifactId>spring-boot-starter-aop</artifactId>  
</dependency>
```

### AOP术语

本篇文章假设读者已经理解了一点AOP的基本概念，这里将直接给出一些相关术语及说明。

| 术语                      | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| 切面（Aspect）            | 一个关注点的模块化内容，定义该关注点的一些处理逻辑内容，这个关注点可能会横切多个对象。 |
| 连接点（Joinpoint）       | 在程序执行过程中某个特定的点，比如某方法调用的时候或者处理异常的时候。在Spring AOP中，一个连接点总是表示一个方法的执行。 |
| 通知（Advice）            | 在切面的某个特定的连接点上执行的动作。其中包括了“around”、“before”和“after”等不同类型的通知（通知的类型将在后面部分进行讨论）。许多AOP框架（包括Spring）都使用拦截器做通知模型，维护了一个以连接点为中心的拦截器链。 |
| 切入点（Pointcut）        | 匹配连接点的断言。通知和一个切入点表达式关联，并在满足这个切入点的连接点上运行（例如，当执行某个特定名称的方法时）。切入点表达式如何和连接点匹配是AOP的核心：Spring缺省使用AspectJ切入点语法。 |
| 引入（Introduction）      | 用来给一个类型声明额外的方法或属性（也被称为连接类型声明（inter-type declaration））。Spring允许引入新的接口（以及一个对应的实现）到任何被代理的对象。例如，你可以使用引入来使一个bean实现IsModified接口，以便简化缓存机制。 |
| 目标对象（Target Object） | 被一个或者多个切面所通知的对象。也被称做被通知（advised）对象。既然Spring AOP是通过运行时代理实现的，这个对象永远是一个被代理（proxied）对象。 |
| AOP代理（AOP Proxy）      | AOP框架创建的对象，用来实现切面契约（例如通知方法执行等等）。在Spring中，AOP代理可以是JDK动态代理或者CGLIB代理。 |
| 织入（Weaving）           | 把切面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象。这些可以在编译时（例如使用AspectJ编译器），类加载时和运行时完成。Spring和其他纯Java AOP框架一样，在运行时完成织入。 |

Spring中可以使用的通知（Advice）类型说明：

> 前置通知（Before）：在某连接点之前执行的通知，但这个通知不能阻止连接点之前的执行流程（除非它抛出一个异常）。
>
> 后置通知（AfterReturning）：在某连接点正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。
>
> 异常通知（AfterThrowing）：在方法抛出异常退出时执行的通知。
>
> 最终通知（After）：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。
>
> 环绕通知（Around）：包围一个连接点的通知，可以完整的自定义连接点目标对象方法调用的所有过程。这是最强大的一种通知类型。

### Spring AOP示例

这里我们先直接给出一个使用了Spring的AOP框架完成的简单面向切面编程代码示例，然后我们再详细介绍其中的细节：

```java
/**
 * 定义切面描述
 */
@Aspect
@Configuration
public class TestAop {

    /*
     * 定义一个切入点
     */
    @Pointcut("execution(* com.test.service.CacheDemoService.find*(..))")
    public void excudeService(){}
    
    /*
     * 通过连接点切入
     */
    @Before("excudeService()")
    public void twiceAsOld1(JoinPoint joinPoint){
        System.err.println ("切面before执行了。。。。");

    }

}
```

#### 使用AspectJ切入点语法指定切入点（Pointcut）

在各通知（Advice）类型的连接点注解中定义的参数，即为切入点的表达式。Spring AOP切入点语法中使用到的基本符号如下：

> 通配符：
>
> *：匹配任何数量字符； 
>
> ..：在类型模式中匹配任何数量子包；而在方法参数模式中匹配任何数量参数。
>
> +：匹配指定类型的子类型；仅能作为后缀放在类型模式后边。
>
> 
>
> 组合表达式：
>
> &&：与
>
> ||：或
>
> !：非

Spring AOP支持多种切入点指示符：`execution`、`within`、`this`、`target`、`args`、`@target`、`@args`、`@within`、`@annotation`、`bean`。

切入点表达式遵循这样的格式：`execution([可见性] 返回类型 [声明类型].方法名(参数) [异常])`

关于这些指示符的切入点表达式示例如下：

匹配任意public方法：

```java
execution(public * *(..))
```

匹配任意方法名是“set”开头的方法：

```java
execution(* set*(..))
```

匹配`AccountService`类中的所有方法：

```java
execution(* com.xyz.service.AccountService.*(..))
```

匹配`service`包中的所有方法：

```java
execution(* com.xyz.service.*.*(..))
```

匹配`service`包以及其子包中的所有方法：

```java
execution(* com.xyz.service..*.*(..))
```

匹配`service`包中的所有方法：

```java
within(com.xyz.service.*)
```

匹配所有代理对象实现了`AccountService`接口的方法：

```java
this(com.xyz.service.AccountService)
```

匹配所有目标对象实现了`AccountService`接口的方法：

```java
target(com.xyz.service.AccountService)
```

匹配所有单一参数且该参数类型为`Serializable`的方法，并将该参数传入增强方法中：

```java
args(java.io.Serializable)
```

匹配带有`@Transactional`注解的目标对象：

```java
@target(org.springframework.transaction.annotation.Transactional)
```

匹配带有`@Transactional`注解的类中的所有方法：

```java
@within(org.springframework.transaction.annotation.Transactional)
```

匹配所有带有`@Transactional`注解的方法：

```java
@annotation(org.springframework.transaction.annotation.Transactional)
```

匹配所有单一参数且该参数带有`@Classified`注解的方法：

```java
@args(com.xyz.security.Classified)
```

匹配所有名为`tradeService`的Spring bean：

```java
bean(tradeService)
```

匹配所有名字以"Service"结尾的Spring bean：

```java
bean(*Service)
```

切入点表达式可以直接写在通知（Advice）注解的参数中，也可以通过`@Pointcut`注解定义，如同示例中所示。

#### JoinPoint对象与Around方法

完成了切入点的定义之后，就可以实际来编写这个切入点对应所需执行的增强方法了。

增强方法可以是无参的，也可以带有被切入点指示符`args`定义传入的指定参数，另外增强方法还可以带有一个`org.aspectj.lang.JoinPoint`类型的参数（如示例中所示），JoinPoint对象中封装了一些被切面方法的信息，JoinPoint常用方法见下表。

| 方法名                    | 功能描述                                                     |
| ------------------------- | ------------------------------------------------------------ |
| Signature getSignature(); | 获取封装了署名信息的对象,在该对象中可以获取到目标方法名,所属类的Class等信息 |
| Object[] getArgs();       | 获取传入目标方法的参数对象                                   |
| Object getTarget();       | 获取被代理的对象                                             |
| Object getThis();         | 获取代理对象                                                 |

如果这个切入点对应的通知（Advice）类型是Around类型，那么方法中需要传入一个`org.aspectj.lang.ProceedingJoinPoint`类型的参数，它是JoinPoint类型的子类，多了两个方法。

| 方法名                          | 功能描述                     |
| ------------------------------- | ---------------------------- |
| Object proceed()；              | 执行目标方法                 |
| Object proceed(Object[] var1)； | 传入的新的参数去执行目标方法 |

使用详情见下面所示的代码示例：

```java
@Aspect
@Component
public class aopAspect {
    /**
     * 定义一个切入点表达式,用来确定哪些类需要代理
     * execution(* aopdemo.*.*(..))代表aopdemo包下所有类的所有方法都会被代理
     */
    @Pointcut("execution(* aopdemo.*.*(..))")
    public void declareJoinPointerExpression() {}

    /**
     * 前置方法,在目标方法执行前执行
     * @param joinPoint 封装了代理方法信息的对象
     */
    @Before("declareJoinPointerExpression()")
    public void beforeMethod(JoinPoint joinPoint){
        System.out.println("目标方法名为:" + joinPoint.getSignature().getName());
        System.out.println("目标方法所属类的简单类名:" +        joinPoint.getSignature().getDeclaringType().getSimpleName());
        System.out.println("目标方法所属类的类名:" + joinPoint.getSignature().getDeclaringTypeName());
        System.out.println("目标方法声明类型:" + Modifier.toString(joinPoint.getSignature().getModifiers()));
        //获取传入目标方法的参数
        Object[] args = joinPoint.getArgs();
        for (int i = 0; i < args.length; i++) {
            System.out.println("第" + (i+1) + "个参数为:" + args[i]);
        }
        System.out.println("被代理的对象:" + joinPoint.getTarget());
        System.out.println("代理对象自己:" + joinPoint.getThis());
    }

    /**
     * 环绕方法,可自定义目标方法执行的时机
     * @param pjd JoinPoint的子接口,添加了
     *            Object proceed() throws Throwable 执行目标方法
     *            Object proceed(Object[] var1) throws Throwable 传入的新的参数去执行目标方法
     *            两个方法
     * @return 此方法需要返回值,返回值视为目标方法的返回值
     */
    @Around("declareJoinPointerExpression()")
    public Object aroundMethod(ProceedingJoinPoint pjd){
        Object result = null;
        try {
            //前置通知
            System.out.println("目标方法执行前...");
            //执行目标方法
            //result = pjd.proeed();
            //用新的参数值执行目标方法
            result = pjd.proceed(new Object[]{"newSpring","newAop"});
            //后置通知
            System.out.println("目标方法执行后...");
            return result;
        } catch (Throwable e) {
            //异常通知
            System.out.println("执行目标方法异常后...");
            throw new RuntimeException(e);
        } finally {
            //返回通知
            System.out.println("目标方法返回结果后...");
        }
    }
}
```



###### 参考内容链接

1. [Spring Framework官方指南](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop-pointcuts)
2. [【spring-boot】spring aop 面向切面编程初接触](https://www.cnblogs.com/lic309/p/4079194.html)
3. [Spring 之AOP AspectJ切入点语法详解（最全面、最详细。）](https://blog.csdn.net/zhengchao1991/article/details/53391244)
4. [Spring-AOP @AspectJ切点函数之@within()和@target](https://blog.csdn.net/yangshangwei/article/details/77846974)
5. [Spring AOP中使用args表达式访问目标方法的参数](https://blog.csdn.net/confirmaname/article/details/9739899)
6. [Spring AOP target() vs this()](https://stackoverflow.com/questions/11924685/spring-aop-target-vs-this)
7. [Spring AOP中JoinPoint的用法](https://www.jianshu.com/p/90881bfc3241)
8. [使用Spring进行面向切面编程](https://blog.csdn.net/yuqinying112/article/details/7244281)

