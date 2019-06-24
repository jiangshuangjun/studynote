# 小马哥 Java 并发面试题

## 线程创建

### 有哪些方法创建线程？【基本版】

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

1. 使用 `new` 关键字创建 `Thread`

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

### 如何通过 Java 创建进程【进阶版】

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



### 如何销毁一个线程？【劝退版】

在 Java 中，正在执行的线程是没有办法销毁的，但是当 `java.lang.Thread#isAlive` native 方法返回 `false` 时，实际底层的 Thread 已经被销毁了。在 JVM 的实现中，当 Java 的 `java.lang.Thread#run` 方法执行完毕，会调用 JVM 的 c++ 实现类 thread.cpp 中的 `thread_main_inner()` 方法，该方法末尾，有如下源码：

```c++
void JavaThread::thread_main_inner() {
    // ...

	// 线程退出
    this->exit(false);
    delete this;
}
```

---

## 线程执行

### 如何通过 Java API 启动线程？【基本版】

通过 `java.lang.Thread#start` -> `java.lang.Thread#start0` 方法，其中 `java.lang.Thread#start0` 是一个 native 方法，会去调用操作系统的线程

### 当有线程 T1、T2 以及 T3，如何实现 T1 -> T2 -> T3 的执行顺序？【进阶版】

- 方法一：利用 `java.lang.Thread#join()` 控制线程必须执行完成

  ```java
  public static void main(String[] args) throws Exception {
      Runnable runnable = () -> System.out.printf("线程[%s] 正在执行...\n", Thread.currentThread().getName());
          
      Thread t1 = new Thread(runnable, "t1");
      Thread t2 = new Thread(runnable, "t2");
      Thread t3 = new Thread(runnable, "t3");
  
      t1.start();
      t1.join();
  
      t2.start();
      t2.join();
  
      t3.start();
      t3.join();
  }
  ```

### 以上问题请至少提供另外一种实现？【劝退版】

- 方法二：利用 `java.lang.Thread#isAlive` 自旋，`java.lang.Thread#isAlive` 方法是一个 native 方法

  ```java
  public static void main(String[] args) throws Exception {
      Runnable runnable = () -> System.out.printf("线程[%s] 正在执行...\n", Thread.currentThread().getName());
  
      Thread t1 = new Thread(runnable, "t1");
      Thread t2 = new Thread(runnable, "t2");
      Thread t3 = new Thread(runnable, "t3");
  
      t1.start();
      while (t1.isAlive()) {}
  
      t2.start();
      while (t2.isAlive()) {}
  
      t3.start();
      while (t3.isAlive()) {}
  }
  ```

- 方法三：利用 `java.lang.Thread#isAlive` 自旋加 `java.lang.Thread#sleep(long)` 方法

  ```java
  public static void main(String[] args) throws Exception {
      Runnable runnable = () -> System.out.printf("线程[%s] 正在执行...\n", Thread.currentThread().getName());
  
      Thread t1 = new Thread(runnable, "t1");
      Thread t2 = new Thread(runnable, "t2");
      Thread t3 = new Thread(runnable, "t3");
  
      t1.start();
      while (t1.isAlive()) {
          Thread.sleep(0);
      }
  
      t2.start();
      while (t2.isAlive()) {
          Thread.sleep(0);
      }
  
      t3.start();
      while (t3.isAlive()) {
          Thread.sleep(0);
      }
  }
  ```

- 方法四：通过 `java.lang.Object#wait()` 和 `java.lang.Thread#isAlive` 方法

  ```java
  public static void main(String[] args) throws Exception {
      Runnable runnable = () -> System.out.printf("线程[%s] 正在执行...\n", Thread.currentThread().getName());
  
      Thread t1 = new Thread(runnable, "t1");
      Thread t2 = new Thread(runnable, "t2");
      Thread t3 = new Thread(runnable, "t3");
  
      threadStartAndWait(t1);
      threadStartAndWait(t2);
      threadStartAndWait(t3);
  }
  
  private static void threadStartAndWait(Thread thread) {
      if (Thread.State.NEW.equals(thread.getState())) {
          thread.start();
      }
  
      while (thread.isAlive()) {
          synchronized (thread) {
              try {
                  thread.wait();
              } catch (Exception e) {
                  throw new RuntimeException(e);
              }
          }
      }
  }
  ```

---

## 线程中止

### 如何停止一个线程？【基本版】

停止一个线程除了让它执行完毕或者异常退出，没有别的办法，但有办法中断线程的逻辑执行。中断线程的执行有如下两种思路：

- 方法一：给线程加开关

  ```java
  public class ThreadDemo {
  
      public static void main(String[] args) {
          Action action = new Action();
  
          Thread t1 = new Thread(action, "t1");
  
          t1.start();
  
          action.setStopped(true);
  
          System.out.println(t1.getState());
      }
  
      private static class Action implements Runnable {
          // 线程安全问题，确保可见性（Happens-Before)
          private volatile boolean stopped = false;
  
          @Override
          public void run() {
              if (!stopped) {
                  // 执行动作
                  action();
              }
          }
  
          public void setStopped(boolean stopped) {
              this.stopped = stopped;
          }
      }
  
      private static void action() {
          System.out.printf("线程[%s] 正在执行...\n", Thread.currentThread().getName());
      }
  
  }
  ```

  运行结果如下：

  ```java
  E:\software\01-java-jdk\java8\jdk8\bin\java.exe ...
  RUNNABLE
  
  Process finished with exit code 0
  ```

- 方法二：使用 `java.lang.Thread#isInterrupted()` 和 `java.lang.Thread#interrupt` 方法

  ```java
  public static void main(String[] args) {
      Thread t1 = new Thread(()-> {
          if (!Thread.currentThread().isInterrupted()) {
              System.out.printf("线程[%s] 正在执行...\n", Thread.currentThread().getName());
          }
      }, "t1");
  
      t1.start();
  
      t1.interrupt();
  
      System.out.println(t1.getState());
  }
  ```

  运行结果如下：

  ```java
  E:\software\01-java-jdk\java8\jdk8\bin\java.exe ...
  RUNNABLE
  
  Process finished with exit code 0
  ```

### 为什么 Java 要放弃 Thread 的 stop() 方法？【进阶版】

参考官方文档 [Java-Thread-Primitive-Deprication](https://docs.oracle.com/javase/6/docs/technotes/guides/concurrency/threadPrimitiveDeprecation.html)

- Why is `Thread.stop` deprecated?

  > Because it is inherently unsafe. Stopping a thread causes it to unlock all the monitors that it has locked. (The monitors are unlocked as the `ThreadDeath` exception propagates up the stack.) If any of the objects previously protected by these monitors were in an inconsistent state, other threads may now view these objects in an inconsistent state. Such objects are said to be *damaged*. When threads operate on damaged objects, arbitrary behavior can result. This behavior may be subtle and difficult to detect, or it may be pronounced. Unlike other unchecked exceptions, `ThreadDeath` kills threads silently; thus, the user has no warning that his program may be corrupted. The corruption can manifest itself at any time after the actual damage occurs, even hours or days in the future.

- Why are `Thread.suspend` and `Thread.resume` deprecated?

  > `Thread.suspend` is inherently deadlock-prone. If the target thread holds a lock on the monitor protecting a critical system resource when it is suspended, no thread can access this resource until the target thread is resumed. If the thread that would resume the target thread attempts to lock this monitor prior to calling `resume`, deadlock results. Such deadlocks typically manifest themselves as "frozen" processes.

### 请说明 Thread interrupt()、isInterrupted() 以及 interrupted() 的区别及意义？【劝退版】

- `java.lang.Thread#interrupt`

  > 中断当前线程
  >
  >
  >
  > 如果当前线程正在在中断自己，并且调用了 `java.lang.Thread#checkAccess` 方法，可能会导致 `java.lang.SecurityException` 异常
  >
  >
  >
  > 如果当前线程在调用 `java.lang.Object#wait()`、`java.lang.Object#wait(long)`、`java.lang.Object#wait(long, int)`、`java.lang.Thread#join()`、`java.lang.Thread#join(long)`、`java.lang.Thread#join(long, int)`、`java.lang.Thread#sleep(long)`、`java.lang.Thread#sleep(long, int)` 方法被阻塞了，那么当前线程的中断状态会被清除，并且会收到一个 `java.lang.InterruptedException`
  >
  >
  >
  > 详细描述可以看该方法 JavaDoc 注释

- `java.lang.Thread#isInterrupted()`

  > 判断当先线程是否被中断过，线程调用当前方法不会影响该线程的中断状态
  >
  > 当线程处于中断状态时，`java.lang.Thread#isAlive` 方法会返回 `false`

- `java.lang.Thread#interrupted`

  > 判断当前线程是否被中断过，线程调用当前方法会清除该线程的中断状态。换句话说，如果线程处于被中断状态，线程连续调用当前方法两次时，第二次调用将返回 `false`
  >
  > 当线程处于中断状态时，`java.lang.Thread#isAlive` 方法会返回 `false`

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

