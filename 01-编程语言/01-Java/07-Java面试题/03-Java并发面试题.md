## 小马哥 Java 并发面试题

### 线程创建

#### 有哪些方法创建线程？【基本版】

在 Java 中，一切皆对象。Thread 也是一个对象，要知道有哪些方法可以创建线程，首先要知道有哪些方法可以创建对象：

1. 使用 `new` 关键字创建对象
2. 使用 Class 类的 newInstance 方法(反射机制)
3. 使用 Constructor 类的 newInstance 方法(反射机制)
4. 使用 clone 方法创建对象
5. 使用(反)序列化机制创建对象

在这些创建对象的方法中，针对 Thread 对象，方法4中 `java.lang.Thread#clone`会抛出 `CloneNotSupportedException`，源码如下：

```java
/**
 * Throws CloneNotSupportedException as a Thread can not be meaningfully
 * cloned. Construct a new Thread instead.
 *
 * @throws  CloneNotSupportedException
 *          always
 */
@Override
protected Object clone() throws CloneNotSupportedException {
    throw new CloneNotSupportedException();
}
```

方法5，由于 Thread 对象并没有实现 `Serializable`接口，所以限制了通过反序列化创建 Thread 的方式

综上，Java 中创建线程有如下3种方式：

1. 使用 `new` 关键字创建 `Thread`，方法调用顺序如下：

   `java.lang.Thread#Thread()` ---> `java.lang.Thread#init(java.lang.ThreadGroup, java.lang.Runnable, java.lang.String, long)` ---> `java.lang.Thread#init(java.lang.ThreadGroup, java.lang.Runnable, java.lang.String, long, java.security.AccessControlContext, boolean)`

   调用顺序源码如下：

   ```java
   /**
    * Allocates a new {@code Thread} object. This constructor has the same
    * effect as {@linkplain #Thread(ThreadGroup,Runnable,String) Thread}
    * {@code (null, null, gname)}, where {@code gname} is a newly generated
    * name. Automatically generated names are of the form
    * {@code "Thread-"+}<i>n</i>, where <i>n</i> is an integer.
    */
   public Thread() {
       init(null, null, "Thread-" + nextThreadNum(), 0);
   }
   
   // ...
   
   /**
    * Initializes a Thread with the current AccessControlContext.
    * @see #init(ThreadGroup,Runnable,String,long,AccessControlContext,boolean)
    */
   private void init(ThreadGroup g, Runnable target, String name,
                     long stackSize) {
       init(g, target, name, stackSize, null, true);
   }
   
   // ...
   
   /**
    * Initializes a Thread.
    *
    * @param g the Thread group
    * @param target the object whose run() method gets called
    * @param name the name of the new Thread
    * @param stackSize the desired stack size for the new thread, or
    *        zero to indicate that this parameter is to be ignored.
    * @param acc the AccessControlContext to inherit, or
    *            AccessController.getContext() if null
    * @param inheritThreadLocals if {@code true}, inherit initial values for
    *            inheritable thread-locals from the constructing thread
    */
   private void init(ThreadGroup g, Runnable target, String name,
                     long stackSize, AccessControlContext acc,
                     boolean inheritThreadLocals) {
       if (name == null) {
           throw new NullPointerException("name cannot be null");
       }
   
       this.name = name;
   
       Thread parent = currentThread();
       SecurityManager security = System.getSecurityManager();
       if (g == null) {
           /* Determine if it's an applet or not */
   
           /* If there is a security manager, ask the security manager
                  what to do. */
           if (security != null) {
               g = security.getThreadGroup();
           }
   
           /* If the security doesn't have a strong opinion of the matter
                  use the parent thread group. */
           if (g == null) {
               g = parent.getThreadGroup();
           }
       }
   
       /* checkAccess regardless of whether or not threadgroup is
              explicitly passed in. */
       g.checkAccess();
   
       /*
            * Do we have the required permissions?
            */
       if (security != null) {
           if (isCCLOverridden(getClass())) {
               security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
           }
       }
   
       g.addUnstarted();
   
       this.group = g;
       this.daemon = parent.isDaemon();
       this.priority = parent.getPriority();
       if (security == null || isCCLOverridden(parent.getClass()))
           this.contextClassLoader = parent.getContextClassLoader();
       else
           this.contextClassLoader = parent.contextClassLoader;
       this.inheritedAccessControlContext =
           acc != null ? acc : AccessController.getContext();
       this.target = target;
       setPriority(priority);
       if (inheritThreadLocals && parent.inheritableThreadLocals != null)
           this.inheritableThreadLocals =
           ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
       /* Stash the specified stack size in case the VM cares */
       this.stackSize = stackSize;
   
       /* Set thread ID */
       tid = nextThreadID();
   }
   ```

2. 通过 `java.lang.reflect.Constructor#newInstance`方法创建 `Thread`

3. 通过 `java.lang.Class#newInstance`方法创建 `Thread`

上述3种创建线程的示例代码如下：

```java
public static void main(String[] args) throws Exception {
    // 通过 new 关键字创建 Thread 对象
    Thread thread0 = new Thread();
    System.out.println(thread0.getName());

    // 通过 java.lang.reflect.Constructor.newInstance 方法创建 Thread 对象
    Constructor<Thread> constructor = Thread.class.getDeclaredConstructor();
    constructor.setAccessible(true);
    Thread thread1 = constructor.newInstance();
    System.out.println(thread1.getName());

    // 通过 java.lang.Class.newInstance 方法创建 Thread 对象
    Class threadClazz = Class.forName("java.lang.Thread");
    Thread thread2 = (Thread) threadClazz.newInstance();
    System.out.println(thread2.getName());
}
```

输出结果如下所示：

```java
E:\software\01-java-jdk\java8\jdk8\bin\java.exe ...
    
Thread-0
Thread-1
Thread-2

Process finished with exit code 0
```

不过，小马哥说，反射创建对象是可以被关闭的，不属于完全可用。

#### 如何通过 Java 创建进程【进阶版】

可以通过 `java.lang.Runtime` 类来创建一个 Java 的进程，示例代码如下：

```java
public static void main(String[] args) throws Exception {
    // 获取 Java Runtime
    Runtime runtime = Runtime.getRuntime();

    // 打开 windows 上的默认浏览器，并打开 http://www.baidu.com 页面
    Process process1 = runtime.exec("cmd /k start http://www.baidu.com");
    process1.exitValue();

//        // 唤醒 windows 上的计算器进程
//        Process process2 = runtime.exec("calc");
//        process2.exitValue();
}
```



#### 如何销毁一个线程？【劝退版】

在 Java 中，正在执行的线程是没有办法销毁的，但是当 `java.lang.Thread#isAlive` native方法返回 `false` 时，实际底层的 Thread 已经被销毁了。在JVM的实现中，当 Java 的 `java.lang.Thread#run`方法执行完毕，会调用 JVM 的 c++ 实现类 thread.cpp 中的 `thread_main_inner()`方法，该方法末尾，有如下源码：

```c++
void JavaThread::thread_main_inner() {
    // ...

	// 线程退出
    this->exit(false);
    delete this;
}
```



#### 为什么不建议使用 Thread 的 `java.lang.Thread#stop()`,`java.lang.Thread#stop(java.lang.Throwable)`,`java.lang.Thread#suspend`,`java.lang.Thread#resume`方法？

参考官方文档 [Java-Thread-Primitive-Deprication](https://docs.oracle.com/javase/6/docs/technotes/guides/concurrency/threadPrimitiveDeprecation.html)

---

### 线程执行

#### 如何通过 Java API 启动线程？【基本版】

#### 当有线程 T1、T2 以及 T3，如何实现 T1 -> T2 -> T3 的执行顺序？【进阶版】

#### 以上问题请至少提供另外一种实现？【劝退版】

---

### 线程中止

#### 如何停止一个线程？【基本版】

#### 为什么 Java 要放弃 Thread 的 stop() 方法？【进阶版】

#### 请说明 Thread interrupt()、isInterrupted() 以及 interrupted() 的区别及意义？【劝退版】



---

#### synchronized 与 Lock 有什么区别？用新的 Lock 有什么好处？请举例说说

- 原始构成

  > synchronized 是关键字，属于 JVM 层面。
  >
  >     monitorenter(底层是通过 monitor 对象来完成，其实 wait/notify 等方法也依赖于 monitor 对象，只有在同步块或方法中才能调 wait/notify 等方法)
  >
  >     monitorexit
  >
  >
  >
  > Lock 是具体类(java.util.concurrent.locks.Lock)，是 api 层面的锁

- 使用方法

  > synchronized 不需要用户去手动释放锁，当 synchronized 代码执行完后系统会自动让线程释放对锁的占用
  >
  >
  >
  > ReentrantLock 则需要用户去手动释放锁，若没有主动释放锁，就有可能导致出现死锁现象。
  >
  > 需要 lock() 和 unlock() 方法配合 try/finally 语句块来完成。

- 等待是否可中断

  > synchronized 不可中断，除非抛出异常或者正常运行完成
  >
  >
  >
  > ReentrantLock 可中断：
  >
  > 1. 设置超时方法 tryLock(long timeout, TimeUnit unit)
  > 2. lockInterruptibly()放代码块中，调用 interrupt() 方法可中断

- 加锁是否公平

  > synchronized 非公平锁
  >
  >
  >
  > ReentrantLock 两者都可以，默认非公平锁，构造方法可以传入 boolean 值，true 为公平锁，false 为非公平锁

- 锁绑定多个条件 Condition

  > synchronized 没有
  >
  > ReentrantLock 用来实现分组唤醒需要唤醒的线程们，可以精确唤醒，而不是像 synchronized 要么随机唤醒一个线程要么唤醒全部线程。

