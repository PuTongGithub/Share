# JAVA中的反射机制（一）

在写代码的时候我遇到了一个这样的需求：提供一个方法，把一个任意类型的对象实体转换为Map<String, Object>。面对这样一个需求我自然就想到了利用java中的反射来实现，同时借这个机会我也来梳理一下java中的反射机制。



### 什么是反射

> - Java反射机制作用在程序运行状态中
> - 对于**任意一个类**，都能知道这个类的所以**属性和方法**；
> - 对于**任何一个对象**，都能够调用它的任何一个**方法和属性**；
> - 这样**动态获取**新的以及**动态调用**对象方法的功能就叫做**反射**。



### 如何使用反射

一般情况下我们通过以下这些类来使用反射：

```java
java.lang.Class;
java.lang.reflect.Constructor;
java.lang.reflect.Field;
java.lang.reflect.Method;
java.lang.reflect.Modifier;
java.lang.reflect.Array;
```

#### Class类

Class类可以说是使用反射的基础，`java.lang.reflect`中的所有类都没有public构造方法，所有这些类的实例都是通过Class类获取。

在我们程序里写的每一种类，都会在编译使产生一个唯一的Class类对象，这个对象就保存在同名的.class文件里。

我们一般通过以下这几种方式获取到一个Class类实例：

```java
Class c1 = Test.class;	//这说明任何一个类都有一个隐含的静态成员变量class，这种方式是通过获取类的静态成员变量class得到的()
Class c = Double.TYPE;   //等价于 double.class
Class c2 = test.getClass();		// test是Test类的一个对象，这种方式是通过一个类的对象的getClass()方法获得的 (对于基本类型无法使用这种方法)
Class c3 = Class.forName("me.reflect.Test");	//这种方法是Class类调用forName方法，通过一个类的全量限定名获得（基本类型无法使用此方法）
```

另外还有一些其他的方法获取一个Class类实例：

```java
Class.getClasses()	//得到目标类中的所有的public内部类以及public内部接口所对应的Class对象
Class.getDeclaredClasses()	//同getClasses()，但不局限于public修饰，只要是目标类中声明的内部类和接口均可
Class.getDeclaringClass()	//得到目标属性所在类对应的Class对象
Class.getEnclosingClass()	//得到目标类所在的外围类的Class对象
Class.getSuperclass()	//获得给定类的父类Class
java.lang.reflect.Field.getDeclaringClass()
java.lang.reflect.Method.getDeclaringClass()
java.lang.reflect.Constructor.getDeclaringClass()
```

#### Field

在Class类里提供了4个方法是我们获取到Field对象:

```java
Class c = Test.getClass();	//Test为一个自定义的类型
c.getDeclaredField(String name);	//获取指定的变量（只要是声明的变量都能获得，包括private）
c.getField(String name);	//获取指定的变量（只能获得public）
c.getDeclaredFields();		//获取所有声明的变量（包括private）
c.getFields();		//获取所有的public变量
```

通过Field对象我们可以获取变量的**类型**、**修饰符**、**注解**、**变量名**、**变量值**，我们还可以**修改变量值**。下面举例相关方法：

```java
Field f = c.getField("testString");
f.getType();	//获取类型
f.getModifiers();	//获取修饰符
f.getAnnotations();		//获取注解
f.getName();	//获取名字

Test t = new Test("test");
f.get(t);	//获取值
f.set(t, "testString");		//设置值
```

#### Method

在Class类里同样提供了4中方法获取Method对象:

```java
Class c = Test.getClass();	//Test为一个自定义的类型
c.getDeclaredMethod(String name, Class<?>... parameterTypes);	//根据方法名获得指定的方法， 参数name为方法名，参数parameterTypes为方法的参数类型，如 getDeclaredMethod(“doTest”, String.class)
c.getMethod(String name, Class<?>... parameterTypes);	//根据方法名获取指定的public方法，参数说明同上
c.getDeclaredMethods();	//获取所有声明方法
c.getMethods();	//获取所有的public方法
```

通过Method对象我们可以获取**方法返回类型**、**方法参数类型**、**方法声明抛出的异常类型**、**方法名称**、**方法修饰符**，还可以**通过反射调用方法**。下面举例相关方法：

```java
Method m = c.getMethod(“doTest”, String.class);

m.getReturnType();	//获取目标方法返回类型对应的Class对象
m.getGenericReturnType();	//获取目标方法返回类型对应的Type对象

m.getParameterTypes();	//获取目标方法各参数类型对应的Class对象
m.getGenericParameterTypes();	//获取目标方法各参数类型对应的Type对象

m.getExceptionTypes();	//获取目标方法抛出的异常类型对应的Class对象
m.getGenericExceptionTypes();	//获取目标方法抛出的异常类型对应的Type对象

m.getParameters();	//获取Parameter对象数组，通过Parameter对象可以获取到每一个参数的相关信息，注：.class文件中默认不存储方法参数名称，如果想要获取方法参数名称，需要在编译的时候加上-parameters参数。(构造方法的参数获取方法同样)

m.getModifiers();	//获取方法修饰符

Test t = new Test("test");
m.invoke(t, "doTest");	//反射通过Method的invoke()方法来调用目标方法。第一个参数为需要调用的目标类对象，如果方法为static的，则该参数为null。后面的参数都为目标方法的参数值，顺序与目标方法声明中的参数顺序一致。注：如果方法是private的，可以使用method.setAccessible(true)方法绕过权限检查
```

#### Constructor

在Class类里依然提供了4中方法获取Constructor对象：

```java
Class c = Test.getClass();	//Test为一个自定义的类型
c.getDeclaredConstructor(Class<?>... parameterTypes);	//获取指定构造函数，参数parameterTypes为构造方法的参数类型
c.getConstructor(Class<?>... parameterTypes);	//获取指定public构造函数，参数parameterTypes为构造方法的参数类型
c.getDeclaredConstructors();	//获取所有声明的构造方法
c.getConstructors();	//获取所有的public构造方法
```

Constructor对象的使用方式与Method类似，具体内容参考Method相关介绍。

通常我们有两种方式利用反射来创建一个对象：

```
java.lang.reflect.Constructor.newInstance()
Class.newInstance()
```

一般来讲，我们优先使用第一种方法.

注意：反射不支持自动封箱，传入参数时要小心（自动封箱是在编译期间的，而反射在运行期间）。

#### Array

java中的数组本质也是一个对象，与一般对象相比的不同之处只是在获取Class对象以及创建对象上。

若想使用`Class.forName()`方法获取一个数组类型的Class对象，需要使用以下这种方式：

```java
Class cDoubleArray = Class.forName("[D");    //相当于double[].class
Class cStringArray = Class.forName("[[Ljava.lang.String;");   //相当于String[][].class
```

数组类型的创建与初始化则利用`java.lang.Array`对象，示例如下：

```java
//创建数组， 参数componentType为数组元素的类型，后面不定项参数的个数代表数组的维度，参数值为数组长度
Array.newInstance(Class<?> componentType, int... dimensions);

//设置数组值，array为数组对象，index为数组的下标，value为需要设置的值
Array.set(Object array, int index, int value);

//获取数组的值，array为数组对象，index为数组的下标
Array.get(Object array, int index);
```



##### Object转换为Map对象方法

利用反射我们就可以实现一个方法，把任意java对象转换为一个Map对象了，代码示例如下：

```java
public static Map<String, Object> objectToMap(Object obj) throws Exception {
    if(obj == null){
        return null;
    }

    Map<String, Object> map = new HashMap<String, Object>();
    Field[] declaredFields = obj.getClass().getDeclaredFields();
    for (Field field : declaredFields) {
        field.setAccessible(true);
        map.put(field.getName(), field.get(obj));
    }
    return map;  
}
```



更多内容请参见[JAVA中的反射机制（二）](https://github.com/PuTongGithub/Share/blob/master/java/JAVA中的反射机制%EF%BC%88二%EF%BC%89.md)



###### 参考内容链接

1. [Java反射](https://www.jianshu.com/p/10c29883eac1)
2. [java反射机制深入理解剖析](https://www.w3cschool.cn/java/java-reflex.html)
3. [Java™ Platform, Standard Edition 8 API Specification](https://docs.oracle.com/javase/8/docs/api/)

