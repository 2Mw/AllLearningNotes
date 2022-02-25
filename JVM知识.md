# JVM

[BV1yE411Z7AP](https://www.bilibili.com/video/BV1yE411Z7AP?p=65) p119

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

当存在别的对象对一个对象引用的时候，就会让引用的计数加一，如果为0的时候就可以进行垃圾回收了。

对于引用计数法存在一个问题，对于两个循环引用的对象，A引用B，B引用A时，两个各自引用计数都是1，因此就不能进行回收。

Java因此不采用这种方法，python以前使用的是这种方法。

🔵可达性分析算法

Java采用的算法，Java首先需要判断根对象，jvm判断每个对象是否被根对象所引用，如果被引用则不可以被垃圾回收，未被引用则表示可以被回收。

什么是根对象（GC root）？

对于分析内存泄漏可以使用Eclipse开发的[Memory Analyzer(MAT)](https://www.eclipse.org/mat/)工具，来发现哪些对象可以作为GCRoot。

```sh
jps # 查看java进程id
jmap -dump:format=b,live,file=1.bin 21384	# 输出堆内存快照文件
jmap -dump:format=b,live,file=2.bin 21384	# 输出将指针设置为空后的快照
# 然后使用mat软件打开两个快照文件，查看gc root
```

![image-20220223111525676](E:\Notes\Java\JVM\JVM知识.assets\image-20220223111525676.png)

🔵Java中的四种引用

引用的接口类为`Reference`，其他分别为`StrongReference, SoftReference, WeakReference, PhantomReference`.

![image-20220223111636024](E:\Notes\Java\JVM\JVM知识.assets\image-20220223111636024.png)

图中的实现为强引用，虚线为其他引用。

1. 强引用

   只有所有的强引用连接全部断开，才会进行垃圾回收。

2. 软引用

   当A2都没有被强引用直接或者间接引用的时候，在内存不足的时候，都有可能被回收。

3. 弱引用

   当A3都没有被强引用直接或者间接引用的时候，不管内存够不够，都有可能被回收。可以配合引用队列进行使用。

4. 虚引用

   其**必须**配合引用队列进行使用，比如在回收ByteBuffer对象的时候，也需要回收其对应的直接内存。

5. 终结器引用

   使用对象都会继续`Object`父类，调用`finalize()`时，但是不推荐调用这个方法。

使用软引用代码：`-Xmx20m -XX:+PrintGCDetails -verbose:gc`

```java
@Slf4j(topic = "SoftReference")
public class SoftReferenceDemo {
    private static final int _4MB = 1024 * 1024 * 4;

    public static void main(String[] args) throws IOException {
        soft();
    }

    public static void soft() throws IOException {
        System.in.read();
        List<SoftReference<byte[]>> list = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            SoftReference<byte[]> ref = new SoftReference<>(new byte[_4MB]);
            log.info("{}", ref);
            list.add(ref);
            log.info("{}", list.size());
        }

        log.debug("循环结束");
        for (SoftReference ref : list) {
            log.info("{}", ref);
        }
        System.in.read();
    }
}
```

对于软引用可能会被垃圾回收的情况，可以将`SoftReference`关联到`ReferenceQueue`对象上，当被回收的时候可以通过检测来删除对象。

### 垃圾回收算法

🔵标记清除法

![image-20220223135714269](E:\Notes\Java\JVM\JVM知识.assets\image-20220223135714269.png)

首先将GCroot未进行引用的地址进行标记，然后进行清除。

优点就是速度快，但是会产生很多空间不连续会导致很多的空间碎片。

🔵标记整理法

在标记清除算法的基础上，在进行一个“紧凑”操作。

不会产生碎片，但是需要考虑到系统对象引用的修改，缺点就是速度较慢。

🔵复制算法

![image-20220223140204820](E:\Notes\Java\JVM\JVM知识.assets\image-20220223140204820.png)

复制回收算法使用于存活的内存区域较少的情况，即将内存区域分为两块FROM区域和TO区域。将原先FROM区域中存活的内存复制到TO区域中，清除FROM区域。最终交换两个区域，FROM区域变为TO区域，TO区域变为FROM区域。

缺点就是需要划分两倍的内存。

### 分代回收算法

![image-20220223140915693](E:\Notes\Java\JVM\JVM知识.assets\image-20220223140915693.png)

在具体的java垃圾回收算法中，不会只是用某一种算法，而是将几种算法有机的结合在一起。

Java的回收算法将内存区分为两个区域：新生代和老年代。

有的对象需要长时间进行使用，就放入到老年代中；对于用完就可以丢弃的，就放入到新生代中。

🔵回收流程

首次进行垃圾回收，会将对象存入到新生代的伊甸园中，一直存放到伊甸园满为止。当伊甸园已经存放不了下一个对象的时候就会触发一次垃圾回收，将GC ROOT引用的对象复制到幸存区TO区域，将伊甸园中的其余未引用的对象进行清除，然后将幸存区的对象进行设置生命周期加1，最后将FROM区域和TO区域的进行交换。

![image-20220223142103931](E:\Notes\Java\JVM\JVM知识.assets\image-20220223142103931.png)

当存放在幸存区中的寿命超过阈值15的时候，会将对应的内存晋升到老年代中。之前的GC操作可以称为`Minor GC`。如果新生代和老年代两个区域都放不下新的内存区域的时候，就会触发新的`Full GC`操作，会对新生代和老年代两个区域都进行垃圾回收。

`Minor GC`会触发Stop The World操作，即发生垃圾回收的时候，会暂停其他的用户线程，等待垃圾回收操作完成之后，才会进行恢复用户线程。因为在进行垃圾回收的时候，会发生内存地址发生变化的情况，如果不暂停线程的话会导致地址混乱的情况。

`Full GC`也会触发Stop The World操作，并且时间较长。

对于占用大内存的对象会直接存储老年代中。子线程中内存溢出报错不会停止主线程的运行。

### 相关的vm参数

|       含义        |                            参数                             |
| :---------------: | :---------------------------------------------------------: |
|    堆初始大小     |                            -Xms                             |
|    堆最大大小     |                            -Xmx                             |
|    新生代大小     |                            -Xmn                             |
| 幸存区比例(动态)  | -XX:InitialSurvivorRatio=ratio / -XX:+UseAdaptiveSizePolicy |
|    幸存区比例     |                   -XX:SurvivorRatio=ratio                   |
|     晋升阈值      |             -XX:MaxTenuringThreshold=threshold              |
|     晋升详情      |               -XX:+PrintTenuringDistribution                |
|      GC详情       |               -XX:+PrintGCDetails -verbose:gc               |
| Full GC前Minor GC |                  -XX:ScavengeBeforeFullGC                   |

### 垃圾回收器

* 串行垃圾回收器`Serial GC`

  适合单线程；也适合堆内存比较小，适合单人电脑。

  开启串行垃圾回收：`-XX:+UseSerialGC=Serial+SerialOld`

* 吞吐量优先`Parallel GC`

  适合多线程，堆内存较大，让单位时间内STW时间最短。但是会在短时间内使得CPU占用变高。

  开启吞吐量优先的开关：`-XX:UseParallelGC`

  `-XX:GCTimeRatio=ratio` 设置垃圾回收占用时间的比值

  `-XX:MaxGCPauseMillis=ms` 设置垃圾回收占用时间最大毫秒数

* 响应时间有限`ConcMarkSweep GC`

  适合多线程，堆内存较大，尽可能的让STW单次时间最短（可能STW次数很多）。

  开启响应时间GC：`-XX:+UseConcMarkSweepGC` 开启并发的标记清除GC算法。

### G1垃圾回收

JDK9之后默认使用垃圾回收器。

适用场景：

* 同时注重吞吐量和低延迟，默认的暂停时间是200ms。
* 适合超大的堆内存，会将堆大小分为多个大小相等的Region。
* 整体上是标记整理法，区域之间是复制算法。

相关参数：

* `-XX:UseG1GC`，JDK8需要显式开启
* `-XX:G1HeapRegionSize=size`，设置G1垃圾回收的 Region 大小
* `-XX:MaxGCPauseMillis=time`，设置暂停时间。

🔵G1回收阶段

<img src="E:\Notes\Java\JVM\JVM知识.assets\image-20220224192554562.png" alt="image-20220224192554562" style="zoom: 67%;" />

G1当老年代中内存不足的时候，如果G1垃圾回收器处理垃圾的速度大于用户产生垃圾的速度的时候不是 Full GC，如果小于用户产生垃圾的速度，就会退化为 Serial GC ，就进入了 Full GC 阶段。

重标记法remark

JDK8u20字符串去重

JDK8u40并发标记类卸载

JDK8u60回收巨型对象

### 垃圾回收调优

确定目标来选择合适的回收器，**低延迟**还是**高吞吐量**。

最快的GC就是不发生GC。

考虑以下几个问题：

* 数据是不是太多？
* 数据是否太臃肿？拿取数据的时候是否也顺便拿取了冗余的数据？
* 是否存在内存泄漏？比如向`static Map map`中频繁的加入和删除数据，一般情况下使用第三方缓存数据库来进行存储。

🔵新生代调优：

新生代特点：

* 所有new操作的内存分配非常廉价，TLASB(Thread-Local allocation buffer)
* 死亡对象的回收代价为0， From区域和To区域只要交换后，To区域就会直接清零。
* 大部分对象用过即死
* Minor GC的回收时间远远小于 Full GC。

新生代设置的不能太小也不能太大。如果设置太大，就会导致老年代空间变小，从而使得 Full GC 次数变多。

Oracle 建议新生代设置的大小为堆大小的25%-50%之间，新生代的设置能够容纳`并发量*(请求-响应)`的数据。

新生代也存在幸存区，其大小要满足能够保留`当前活跃对象+需要晋升对象`的数据，让短时间存活的对象不晋升，让长时间存活的对象尽量晋升。

🔵老年代的调优

* CMS的老年代内存越大越好
* 先不进行调优，如果没有出现 Full GC 就挺好了，否则先尝试调优新生代。
* 调试完新生代之后还会发生 Full GC 的时候，将老年代的大小调大 1/3 或者 1/4。

## 类加载和字节码

### 类文件结构

简单的 Hello World 样例编译后的字节码文件：

![image-20220225115051013](E:\Notes\Java\JVM\JVM知识.assets\image-20220225115051013.png)

```
CA FE BA BE 00 00 00 37 00 22 0A 00 06 00 14 09
```

🔵版本信息：前8个字节

* `CA FE BA BE` cafebabe 是用来标识的是java的class文件
* `00 00 00 37` 标识JDK的版本，这里标识JDK11，34标识jdk8

🔵常量池信息：

第8-9个字节：`00 22` 标识有多少个常量。

第 #1 常量：`0A 00 06 00 14`，0A表示Method信息，00 06 和 00 14 表示其引用了常量池中 #6 和 #20 项来获取这个方法的**所属类**和**方法名**。

第 #2 常量：`09 00 15 00 16`，09 表示Field属性类型，后面`00 15` 以及 `00 16` 表示引用了常量 #23 和 #24 的常量信息。

对于字符串常量：`01 00 0B 48 65 6C 6C 6F 20 57 6F 72 6C 64 `，其中 01 表示 utf8串，`00 0B`表示字符串的长度，`48 65 6C 6C 6F 20 57 6F 72 6C 64` 就表示的是`Hello World`。

其余类似。

🔵类的访问标识

```
00 21 00 05 00 06 00 00 00 00 00 02 00 01 00 07
```

`00 21` 表示是一个公共的类，`00 05` 表示类全限定名称 #5，`00 06` 表示父类的全限定名称 #6，`00 00` 表示接口的数量信息，`00 00` 表示属性信息，`00 02` 表示方法的数量。

对于类中的方法属性，其中还有字节码会标识**Java源码**中对应的行号。

更详情的内容可以参考书籍《The Java® Virtual Machine Specification》中的第四章The class File Format。

### 字节码指令

java提供javap指令来进行反编译：`javap -v xxx.class`

样例分析：

```java
public class First {
    public static void main(String[] args) {
        int a = 10;
        int b = a++ + ++a + a--;
        System.out.println(a);  // 11
        System.out.println(b);  // 34
    }
}
```

对应的字节码：

```
0: bipush        10
2: istore_1
3: iload_1
4: iinc          1, 1
7: iinc          1, 1
10: iload_1
11: iadd
12: iload_1
13: iinc          1, -1
16: iadd
17: istore_2
18: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
21: iload_1
22: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
25: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
28: iload_2
29: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
32: return
```

样例分析二：

```java
private static void t1() {
    int i = 0, x = 0;
    while (i < 10) {
        x = x ++;
        i++;
    }
    System.out.println(x + " " + i);    // 0 10
}
```

对于x一直为0有字节码：

    10: iload_2
    11: iinc          2, 1
    14: istore_2

JVM先将LocalVariableTable中slot为2的`x`进行操作栈，然后对LocalVariableTable中的x进行自增变为1，执行到`istore_2` 的时候又将操作栈中的值重新赋值给slot中的数据，因此数值还是为0。

对应字节码：

```
0: iconst_0
1: istore_1
2: iconst_0
3: istore_2
4: iload_1
5: bipush        10
7: if_icmpge     21
10: iload_2
11: iinc          2, 1
14: istore_2
15: iinc          1, 1
18: goto          4
21: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
24: iload_2
25: iload_1
26: invokedynamic #3,  0              // InvokeDynamic #0:makeConcatWithConstants:(II)Ljava/lang/String;
31: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
34: return
```

🔵static：

```java
public class StaticDemo {
    static int i = 1;

    static { i = 2; }

    static { i = 3; }
}
```

在JDK8中会将静态代码块和静态变量的赋值全部按照上下顺序合并为`<cinit>`方法。

🔵非静态成员变量和代码块

```java
package com.jvm.bytecode;

public class Constructor {
    private int a = 1;

    {
        b = 3;
    }

    private int b = 2;

    {
        a = 4;
    }

    public Constructor(int a, int b) {
        this.a = a;
        this.b = b;
    }

    public static void main(String[] args) {
        Constructor ins = new Constructor(9, 99);
        System.out.println(ins.a + " " + ins.b);    // 9 99
    }
}
```

对于非静态代码块和属性中的赋值，jvm会将其重新整合成一个新的构造器，然后将人工构造器的代码放在新的构造器之后。
