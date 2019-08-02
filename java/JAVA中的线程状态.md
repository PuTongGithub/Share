# JAVA中的线程状态

在`java.lang.Thread`中有一个枚举类`State`，这个枚举类标识了Java中一个Thread的6种状态：NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED。本篇文章将介绍这6种状态的具体含义以及它们之间的转移路径。不过在介绍它们之前，首先我们将回顾一下早期操作系统中对于进程的状态管理。

### 操作系统中的进程状态

早期操作系统中对于进程的状态管理可见下图所示：

![操作系统进程状态](https://upload-images.jianshu.io/upload_images/1485056-9d17cf698a3f0dbc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

#### 创建状态

当操作系统需要启动一个新进程时，首先它会为这个新进程创建管理信息、完成资源分配、完成初始化相关操作。

#### 就绪状态

当一个新进程准备完毕后，就进入到了就绪队列之中，即为就绪状态。另外，如果一个进程已分配到除CPU以外的所有资源，它的状态便是就绪状态。

#### 执行状态

就绪进程被操作系统调度得到CPU之后，进程便进入执行状态。同一时刻可能会有多个进程处于执行状态（取决于操作系统）。

#### 阻塞状态

正在执行中的进程由于某些事件导致无法继续执行时，它便进入了阻塞状态以等待这些事件的完成。导致阻塞状态出现的典型事件举例：请求I/O，申请缓冲空间等。

#### 终止状态

当一个进程正常结束运行，或是被无法处理的错误强行终止时，该进程便进入了终止状态，等待操作系统完成善后工作。



### Java中的线程状态

了解了操作系统对于进程的状态控制之后，我们再来看Java中定义的6种线程状态分别是什么含义。首先我们需要明确一点：

>  Java中的线程状态是**虚拟机状态**，它**不反映**任何操作系统的线程状态。

Java中的线程状态更多是服务于监控的，也因此状态中的RUNNABLE实际上包含了就绪与执行两种程序状态。

#### NEW

完成初始化但是未启动的Thread。

#### RUNNABLE

可执行的Thread。注意：在JVM中该线程可能处于正在执行状态，但是在操作系统层面上这一线程有可能还需要等待来自于操作系统的其他资源。

#### BLOCKED

这种状态是指一个阻塞线程在等待获取monitor锁。当线程等待进入一个synchronized块/方法，或尝试重进入一个synchronized锁/方法时，线程便在等待获取monitor锁。

#### WAITING

一个线程在等待另外一个线程执行一个动作的时候，这个线程便处于WAITING状态。当线程调用以下方法后，便进入了WAITING状态：

> 1. Object#wait() 而且不加超时参数
> 2. Thread#join() 而且不加超时参数
> 3. LockSupport#park()

#### TIMED_WAITING

一个线程在一个特定的等待时间内等待另一个线程完成一个动作时，这个线程便处于TIMED_WAITING状态。当线程调用以下方法后，便进入了TIMED_WAITING状态：

> 1. Thread#sleep()
> 2. Object#wait() 并加了超时参数
> 3. Thread#join() 并加了超时参数
> 4. LockSupport#parkNanos()
> 5. LockSupport#parkUntil()

#### TERMINATED

线程执行完毕后，便进入了TERMINATED状态。

#### 六种线程状态的状态转移图

![线程状态转移图](https://iblobs.oss-cn-beijing.aliyuncs.com/thread_state.PNG)



###### 参考内容链接

1. [java线程运行怎么有第六种状态？](https://www.zhihu.com/question/56494969)——[Dawell](https://www.zhihu.com/people/dawell)的回答
2. [操作系统之进程的几种状态](https://www.jianshu.com/p/ac9ce2afd126)
3. [BLOCKED,WAITING,TIMED_WAITING有什么区别？-用生活的例子解释](https://segmentfault.com/a/1190000010973341)
4. java源码——java.lang.Thread类

