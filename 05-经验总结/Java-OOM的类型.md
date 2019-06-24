# Java OOM 的类型

<span id = "0">

- [背景知识 : 相关 JVM 参数](#1)

- [java.lang.StackOverflowError](#2)
- [java.lang.OutOfMemoryError:Java heap space](#3)
- [java.lang.OutOfMemoryError:GC overhead limit exceeded](#4)
- [java.lang.OutOfMemoryError:Direct buffer memory](#5)
- [java.lang.OutOfMemoryError:unable to create new native thread](#6)
- [java.lang.OutOfMemoryError:Metaspace](#7)

---

<span id = "1">

<br/>

## 背景知识 : 相关 JVM 参数

### 查看线程栈默认大小

- jdk12 官方描述

  > ```java
  > -Xss size
  > ```
  >
  > Sets the thread stack size (in bytes). Append the letter `k` or `K` to indicate KB, `m` or `M` to indicate MB, and `g` or `G` to indicate GB. The default value depends on the platform:
  >
  > - Linux/x64 (64-bit): 1024 KB
  > - OS X (64-bit): 1024 KB
  > - Oracle Solaris/x64 (64-bit): 1024 KB
  > - Windows: The default value depends on virtual memory
  >
  > The following examples set the thread stack size to 1024 KB in different units:
  >
  > ```java
  > -Xss1m
  > -Xss1024k
  > -Xss1048576
  > ```
  >
  > This option is similar to `-XX:ThreadStackSize`.

- jdk 各版本默认栈大小描述

  - [jdk8 : -Xsssize](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#CBBFHAJA)
  - [jdk9 : -Xss size](https://docs.oracle.com/javase/9/tools/java.htm#JSWOR624)
  - [jdk10 : -Xss size](https://docs.oracle.com/javase/10/tools/java.htm#JSWOR624)
  - [jdk11 : -Xss size](https://docs.oracle.com/en/java/javase/11/tools/java.html#GUID-3B1CE181-CD30-4178-9602-230B800D4FAE)
  - [jdk12 : -Xss size](https://docs.oracle.com/en/java/javase/12/tools/java.html#GUID-3B1CE181-CD30-4178-9602-230B800D4FAE)

### 设置线程栈大小

- 使用 -Xss 参数

  > 设置单个线程栈的大小，一般默认为 512K ~ 1024K
  >
  > 等价于 -XX:ThreadStackSize

### 堆内存 JVM 参数

- -Xms
  - 等价于 -XX:InitialHeapSize
  - 初始化堆内存大小，默认为系统内存的 1/64
- -Xmx
  - 等价于 -XX:MaxHeapSize
  - 最大堆内存大小，默认为系统内存的 1/4

### 堆外内存

- 设置堆外内存大小

  > -XX:MaxDirectMemorySize=5m

- 查看堆外内存大小，默认为物理内存的 1/4

  ```java
  // 查询当前虚拟机可用的最大堆外内存
  System.out.println("配置的 MaxDirectMemory : " + (sun.misc.VM.maxDirectMemory() / (double) (1024 * 1024)) + " MB");
  ```

<span id = "2">

<br/>

## java.lang.StackOverError

示例代码如下：

```java
/**
 * @author jiangsj
 * JVM 参数 : -Xss128k
 */
public class StackOverflowErrorDemo {

    private static int count = 0;

    public static void main(String[] args) {
        try {
            stackOverflowError();
        } catch (Throwable e) {
            System.out.println("递归调用次数 : " + count);
            e.printStackTrace();
            throw e;
        }
    }

    private static void stackOverflowError() {
        count++;
        stackOverflowError();
    }

}
```

运行结果如下：

```java
E:\software\01-java-jdk\java8\jdk8\bin\java.exe -Xss128k ...
递归调用次数 : 1100
java.lang.StackOverflowError
	at study.throwable.error.oom.StackOverflowErrorDemo.stackOverflowError(StackOverflowErrorDemo.java:22)
	at study.throwable.error.oom.StackOverflowErrorDemo.stackOverflowError(StackOverflowErrorDemo.java:23)
（部分内容被省略...）

Process finished with exit code 0
```

<span id = "3">

<br/>

## java.lang.OutOfMemoryError:Java heap space

示例代码如下：

```java
/**
 * @author jiangsj
 * JVM 参数 : -Xms10m -Xmx10m
 */
public class JavaHeapSpaceDemo {

    public static void main(String[] args) {
        // new 一个 11MB 的对象，直接撑爆最大堆内存
        Byte[] bytes = new Byte[11 * 1024 * 1024];
    }

}
```

运行结果如下：

```java
E:\software\01-java-jdk\java8\jdk8\bin\java.exe -Xms10m -Xmx10m ...
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at study.throwable.error.oom.JavaHeapSpaceDemo.main(JavaHeapSpaceDemo.java:12)

Process finished with exit code 1
```

<span id = "4">

<br/>

## java.lang.OutOfMemoryError:GC overhead limit exceeded

示例代码如下：

```java
/**
 * @author jiangsj
 * JVM 参数 : -Xms10m -Xmx10m -XX:+PrintGCDetails -XX:MaxDirectMemorySize=5m
 *
 * GC 回收时间过长时会抛出 : java.lang.OutOfMemoryError: GC overhead limit exceeded。
 *
 * 过长的定义是：
 * 超过 98% 的时间用来做 GC 并且回收了不到 2% 的堆内存，连续多次 GC 都只回收了不到 2% 的极端情况。
 *
 * 假如不抛出 GC overhead limit exceeded 错误会发生什么情况呢？
 * 那就是 GC 清理的这么点内存很快会再次填满，迫使 GC 再次执行，这样会形成恶性循环，CPU 使用率一直是 100%，而 GC 却没有任何成果。
 */
public class GCOverheadLimitExceededDemo {

    public static void main(String[] args) {
        int i = 0;
        List<String> list = new ArrayList<>();

        try {
            while (true) {
                list.add(String.valueOf(++i).intern());
            }
        } catch (Throwable e) {
            System.out.println(" i = " + i);
            e.printStackTrace();
            throw e;
        }
    }

}
```

运行结果如下：

```java
E:\software\01-java-jdk\java8\jdk8\bin\java.exe -Xms10m -Xmx10m -XX:+PrintGCDetails -XX:MaxDirectMemorySize=5m ...
[GC (Allocation Failure) [PSYoungGen: 2048K->480K(2560K)] 2048K->949K(9728K), 0.0015550 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
(部分重复 Allocation Failure 内容省略...)
...
(部分重复 Full GC 内容省略...)
[Full GC (Ergonomics) [PSYoungGen: 2047K->2047K(2560K)] [ParOldGen: 7101K->7101K(7168K)] 9149K->9149K(9728K), [Metaspace: 3239K->3239K(1056768K)], 0.0359657 secs] [Times: user=0.19 sys=0.00, real=0.04 secs] 
 i = 145848
[Full GC (Ergonomics) [PSYoungGen: 2047K->0K(2560K)] [ParOldGen: 7144K->654K(7168K)] 9192K->654K(9728K), [Metaspace: 3269K->3269K(1056768K)], 0.0074738 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
java.lang.OutOfMemoryError: GC overhead limit exceeded
	at java.lang.Integer.toString(Integer.java:403)
	at java.lang.String.valueOf(String.java:3099)
	at study.throwable.error.oom.GCOverheadLimitExceededDemo.main(GCOverheadLimitExceededDemo.java:26)
Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded
	at java.lang.Integer.toString(Integer.java:403)
	at java.lang.String.valueOf(String.java:3099)
	at study.throwable.error.oom.GCOverheadLimitExceededDemo.main(GCOverheadLimitExceededDemo.java:26)
Heap
 PSYoungGen      total 2560K, used 183K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 2048K, 8% used [0x00000000ffd00000,0x00000000ffd2dd38,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
 ParOldGen       total 7168K, used 654K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 9% used [0x00000000ff600000,0x00000000ff6a38a8,0x00000000ffd00000)
 Metaspace       used 3383K, capacity 4500K, committed 4864K, reserved 1056768K
  class space    used 364K, capacity 388K, committed 512K, reserved 1048576K

Process finished with exit code 1
```

<span id = "5">

<br/>

## java.lang.OutOfMemoryError:Direct buffer memory

示例代码如下：

运行结果如下：

<span id = "6">

<br/>

## java.lang.OutOfMemoryError:unable to create new native thread

示例代码如下：

运行结果如下：

<span id = "7">

<br/>

## java.lang.OutOfMemoryError:Metaspace

示例代码如下：

运行结果如下：

