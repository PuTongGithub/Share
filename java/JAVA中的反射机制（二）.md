# JAVA中的反射机制（二）

在`java.lang.reflect`包中，除了[JAVA中的反射机制（一）](https://github.com/PuTongGithub/Share/blob/master/java/JAVA中的反射机制%EF%BC%88一%EF%BC%89.md)里提到的那些类以外，还有一个比较常用的类没有介绍，它就是`java.long.reflect.Proxy`。Proxy类常常搭配`java.lang.reflect.InvocationHandler`接口来实现动态代理功能。



### 什么是动态代理

首先我们先复习一下java里一个对象的创建过程：

![java对象的创建过程](https://pic2.zhimg.com/80/v2-9cd31ab516bd967e1b8e68736931f8ba_hd.jpg)

①在编译的时候，编译器会根据一个类型的定义生成一个对应且唯一的class文件，这个class文件中包含了这个自定义类型的Class类信息；

②在运行程序的时候这个class文件会被ClassLoader加载到内存里的方法区中，并执行完成这个类里的静态代码块和静态初始化语句；

③当程序执行这个自定义对象的new操作时，程序会在堆中申请所需的内存空间，调用自己以及父类的构造器，创建一个空白对象，最后执行构造器内语句完成对象创建工作。

这时，我们如果想要一个对指定类的代理类（参考设计模式中的“代理模式”），我们通常会采用静态代理的方式来实现：

![静态代理](https://pic2.zhimg.com/80/v2-28223a1c03c1800052a5dfe4e6cb8c53_hd.jpg)

但是这种利用静态代理实现的方式要求我们为每一个目标类都编写一个对应的代理类，如果我们面临的需求是对多个不同的类做相同的代理工作（例如：在不同方法调用前后都添加一个日志记录操作），那这种方式显然会需要很大的工作量才能完成。

如果，我们可以做到利用一个统一的代理类去代理不同的目标类，以此来完成代理工作，那么这将会大大降低我们的工作量。这种利用同一个代理类去代理不同的目标类的方法就叫做动态代理。



### 如何实现动态代理

在`java.lang.reflect`包中所为我们提供的Proxy类与InvocationHandler接口便是用来实现动态代理的，下面我们将通过示例代码来体验一下如何实现动态代理。

首先创建需要被代理的目标类以及其接口：

```java
package test.proxy;

public interface Animal {

    void say();
    
}
```

```java
package test.proxy;

public class Dog implements Animal {

    @Override
    public void say() {
        System.out.println("BARK!");
    }

}
```

然后我们需要实现一个针对目标接口的调用处理器：

```java
package test.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class AnimalProxyImpl implements InvocationHandler {

    private Animal animal;
    
    public AnimalProxyImpl(Animal animal) {
        this.animal = animal;
    }
    
    /**
     * 代理类方法调用处理
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 目标类方法调用前的处理
        System.out.println("Befor say()");
        // 调用目标类方法，获取方法返回值
        Object object = method.invoke(animal, args);
        // 目标类方法调用后的处理
        System.out.println("After say()");
        // 返回目标方法调用返回值
        return object;
    }

}
```

接下来针对这个目标类我们使用动态代理来完成方法调用：

```java
package test.main;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

import test.proxy.Animal;
import test.proxy.AnimalProxyImpl;
import test.proxy.Dog;

public class ProxyMain {

    public static void main(String[] args) {
        Dog dog = new Dog();
        
        System.out.println("目标类直接调用方法：");
        dog.say();
        
        // 目标类的ClassLoader
        ClassLoader classLoader = dog.getClass().getClassLoader();
        // 需要被代理的目标类的所有实现接口
        Class<?>[] classes = dog.getClass().getInterfaces();
        // 针对目标类接口实现的调用处理器
        InvocationHandler invocationHandler = new AnimalProxyImpl(dog);
        // 利用Proxy类提供的方法创建代理对象
        Object proxyObject = Proxy.newProxyInstance(classLoader, classes, invocationHandler);
        
        System.out.println("通过代理类调用方法：");
        // 将代理对象类型强制转换为目标类接口类型后调用其方法
        ((Animal) proxyObject).say();
    }
    
}
```

运行示例代码我们可以在控制台看到以下输出：

```
目标类直接调用方法：
BARK!
通过代理类调用方法：
Befor say()
BARK!
After say()
```

也就是说，我们在通过代理对象调用目标类方法的时候，实际上是调用了我们实现的调用处理器中的invoke方法：

![invoke方法说明](https://iblobs.oss-cn-beijing.aliyuncs.com/20190721000.png)

因此，静态代理与动态代理调用目标类方法的区别可以通过下图表示：

![静态代理与动态代理调用目标类方法区别](https://pic4.zhimg.com/80/v2-b5fc8b279a6152889afdfedbb0f611cc_hd.jpg)



### 附加说明

通过`java.lang.reflect`包我们可以方便的实现动态代理，但是这种方式的动态代理实现方式仍然存在一定的局限性：

> JDK Proxy 只能代理实现接口的类（即使是extends继承类也是不可以代理的）。

也就是说，上面所提供的示例代码里面，如果我们把

```java
((Animal) proxyObject).say();
```

替换为

```java
((Dog) proxyObject).say();
```

我们将会得到一条报错信息

```
Exception in thread "main" java.lang.ClassCastException: com.sun.proxy.$Proxy0 cannot be cast to test.proxy.Dog
	at test.main.ProxyMain.main(ProxyMain.java:29)
```

因此在实际使用动态代理的时候，我们可能会考虑使用其他的动态代理框架（如Cglib），以获得更高的灵活性以及性能。



###### 参考内容链接

1. [Java 动态代理作用是什么？](https://www.zhihu.com/question/20794107)——[bravo1988](https://www.zhihu.com/people/huangsunting)的回答
2. [细说Spring——AOP详解（动态代理实现AOP）](blog.csdn.net/q982151756)
3. [Java提高班（六）反射和动态代理（JDK Proxy和Cglib）](https://cloud.tencent.com/developer/article/1377435)
4. [Java™ Platform, Standard Edition 8 API Specification](https://docs.oracle.com/javase/8/docs/api/)