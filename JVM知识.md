# JVM

[BV1yE411Z7AP](https://www.bilibili.com/video/BV1yE411Z7AP)

## JVM内存结构

![image-20220219184229305](E:\Notes\Java\JVM\JVM知识.assets\image-20220219184229305.png)

### 程序计数器

Program Counter Register程序计数器，用于存储下一条指令的地址。特点就是线程私有的，并且**不会出现内存溢出的**情况。

### 栈

> 分为JVM stack和本地方法栈

线程运行的时候所需的内存空间。可以指定`-Xss1m`来指定栈的大小，这里只分配1m大小的栈。

垃圾回收时只涉及到堆内存中的数据，不会涉及到栈内存中的数据，因为栈中的数据执行完毕之后会自动弹栈。

🔵栈内存溢出

可能发生溢出的情况：栈帧过多(递归)、栈帧过大等情况。

```java
@Slf4j(topic = "StackOverflowError")
public class StackOF {
    // StackOverflowError
    private static int count = 0;
    public static void main(String[] args) {
        try {
            m1();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            log.debug("{}", count); // -Xss256k
        }
    }

    private static void m1() {
        count++;
        m1();
    }
}
```

🔵线程运行诊断

一，占用CPU过高：

```sh
top
ps H -eo pid,tid,%cpu | grep pid
jstack tid
```

使用top定位进程id，使用`ps H`命令定位对应线程的id，再使用`jstack tid`找到对应线程占用过高的代码。

二，线程发生死锁

同样使用jstack工具来进行诊断。

🔵本地方法栈

即使用`native`进行修饰的方法，比如使用C/C++实现的方法。

### 堆

> Heap，是线程共享的，需要考虑线程安全问题。

通过`new`关键字创建的对象都会使用堆内存，存在堆内存溢出的问题。

🔵堆内存溢出

```java
@Slf4j(topic = "HeapOF")
public class HeapOF {
    public static void main(String[] args) {
        int i = 0;
        try {
            List<String> list = new ArrayList<>();
            String a = "Hello";
            while (true) {
                list.add(a);
                a = a + a;
                i++;
            }
        } catch (Throwable e) {
            e.printStackTrace();
            log.debug("{}", i);
        }
    }
}
```

可以通过设置`-Xmx`参数设置堆的大小。如果想要暴露程序是否存在堆内存溢出的可能性，因此在调试的时候可以将堆设置小一些。

🔵堆内存溢出诊断

可以使用`jmap -heap`或者jconsole来监测。

对于垃圾回收过后，内存占用还是很高的情况怎么处理？可以使用`jvisualvm`工具中的堆dump来分析其中的类的对象信息。

### 方法区

方法区是所有jvm线程共享的，用来存储每个类的常量池、属性以及方法数据，方法区也会到处内存溢出。

方法区的常量池就是一张常量表。

可以通过`javap -v *.class`来对字节码进行反编译。

```
Constant pool:
   #1 = Methodref          #6.#20         // java/lang/Object."<init>":()V
   #2 = Fieldref           #21.#22        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #23            // Hello World
   #4 = Methodref          #24.#25        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #26            // com/jvm/struction/ConstantPool
   #6 = Class              #27            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
   #10 = Utf8              LineNumberTable
```

### StringTable

常量池中的信息都会加载到运行时常量池中，执行代码`ldc #2`的时候才可以将对于的常量转化为字符串对象。

```java
public static void main(String[] args) {
    String a = "a";
    String b = "b";
    String ab = "ab";
    String s4 = a + b;
    String s5 = "a" + "b";

    System.out.println(ab == s4);   // false
    System.out.println(ab == s5);   // true
    ab.intern();
}
```

对于jdk8中，s4中的字节码代码相当于是`StringBuilder().append("a").append("b").toString`，对于s5的情况是javac编译器上做了优化，在StringTable中已经存在"ab"这个常量了，又由于String是不可变类，因此创建s5的时候会直接从常量池中查找。

常量池中的数据只是符号，只有在使用的时候才会转为对象。可以利用串池的特性，使用`intern()`方法可以将未放进串池中字符串放进去，返回串池中的符号。

    0: ldc           #2                  // String a
    2: astore_1
    3: ldc           #3                  // String b
    5: astore_2
    6: ldc           #4                  // String ab
    8: astore_3
    9: aload_1
    10: aload_2
    11: invokedynamic #5,  0              // InvokeDynamic #0:makeConcatWithConstants:(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
    16: astore        4
    18: ldc           #4                  // String ab
    20: astore        5

**问题**：

```java
public static void main(String[] args) {
    String s1 = "a";
    String s2 = "b";
    String s3 = "a" + "b";
    String s4 = s1 + s2;
    String s5 = "ab";
    String s6 = s4.intern();

    System.out.println(s3 == s4);   // false
    System.out.println(s3 == s5);   // true
    System.out.println(s3 == s6);   // true

    String x2 = new String("c") + new String("d");
    String x1 = "cd";
    x2.intern();
    System.out.println(x1 == x2);   // false
}
```

🔵StringTable在jvm中存放的位置

在jdk8中Stringtable是存放在堆内存中。jdk1.6之前是放在永久代中。

![image-20220222165119414](E:\Notes\Java\JVM\JVM知识.assets\image-20220222165119414.png)

🔵StringTable性能调优

1. 调整桶的个数

   StringTable的底层存储结构其实是一个`HashTable`，如果使用`-XX:StringTableSize=20000`来指定常量池桶(bucket)的大小，如果字符串数量较多的话，常量池越小花费的时间会越多，常量池越大花费的时间会越少，减少hash冲突。

2. 考虑将字符串对象是否入池

### 直接内存

直接内存（Direct memory）指系统内存

常见于NIO操作，用于数据缓冲区。分配回收的成本较高，但是读写性能高；不受JVM内存回收管理。

传统的读写一般用`Input/OutputStream`，使用系统直接内存为`ByteBuffer`类中的`allocateDirect()`，对于Java传统的读写需要首先将数据读取到系统的缓冲区，然后再读取到Java的缓冲区中，直接切换的开销过大。

<img src="E:\Notes\Java\JVM\JVM知识.assets\image-20220222185204810.png" alt="image-20220222185204810" style="zoom: 50%;" /><img src="E:\Notes\Java\JVM\JVM知识.assets\image-20220222185242489.png" alt="image-20220222185242489" style="zoom: 50%;" />

🔵但是其不受JVM控制，内存怎么回收？

其底层调用的`Unsafe`类来分配内存和释放内存，对于已经分配好的内存，如果检测到当前对象被gc回收掉之后，其`Cleaner`对象调用`Unsafe`类的方法来释放对应的直接内存。

显式调用垃圾回收：

```java
System.gc();
```

可以设置运行参数：`-XX:+DisableExplictGC`来禁用显式垃圾回收。

## 垃圾回收

### 判断垃圾

🔵引用计数法

