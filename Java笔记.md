# Java

[TOC]

[Java函数式编程](https://www.bilibili.com/video/BV1Gh41187uR?p=29)

## Java基础

### 输入

```java
Scanner input = new Scanner(System.in);
String s = input.next();
System.out.println(s);
```

### 引用赋值

```java
int []arr1 = {1,2,3};
int []arr2 = arr1;
arr2[0] = 1;
arr1[0]	// 1
```

独立空间复制

```java
int[] arr1 = {1,2,3};
int[] arr2 = new int[arr1.length];

```

### 可变参数

> 解决了部分方法的重载问题

```java
public static void main(String[] args) {
    num(6,8,6,10);
}

public static void num(int a, int...num){
    System.out.println(a);
    for(int i:num){
        System.out.print(i+"\t");
    }
}
```

* 可变参数的实现相当于传入了一个数组
* 可变参数必须要在固定参数之后

### 注解

> 注解（annotation）是一种引用类型，编译后生成xxx.class

[修饰符列表] @interface 注解类型名{

}

注解怎么使用，用在什么地方？

​	语法格式：@注解类型名

​	注解可以出现在方法、属性、宏、接口上

```java
@MyAnnotation
public class AnnotationTest {

    @MyAnnotation
    private String name;

    @MyAnnotation
    public void say(@MyAnnotation int s,@MyAnnotation String k){

    }
}

@MyAnnotation
interface ook{
    
}
```

JDK内置的注解：

`@Deprecated`  

`@override`  只能注解方法，编译器会进行检查，如果不是重写父类的方法就会报错。

**元注解：**

注解上的注解被称为元注解。比如 

`@Target`用于标注注解出现在哪个位置。 `@Target(ElementType.METHOD)`

`@Retention` 用于被标注的注解最终保存在哪个位置。`@Retention(RetentionPolicy.SOURCE)` 保留在java源文件中；`@Retention(RetentionPolicy.CLASS)` 保存在class文件中； `@Retention(RetentionPolicy.RUNTIME)` 保存在class文件中，并且可以被反射机制读取。

### equals和==

```java
public static void main(String[] args) {
    String a = "123";
    String b = "123";
    System.out.println(a==b);   //true
    System.out.println(a.equals(b));    //true

    a = new String("123");
    b = new String("123");
    System.out.println(a==b);   //false
    System.out.println(a.equals(b));    //true
    System.out.println(ao==a);    //false
    System.out.println(ao.equals(a));    //true

    ArrayList arr = new ArrayList();
    arr.add("e");arr.add("123");arr.add(123);
    System.out.println("ENd:" + (arr.contains(a) && arr.contains(ao)));
}
```

自定义对象equals

```java
public class ContainsDemo {
    public static void main(String[] args) {
        User u1 = new User("A");
        ArrayList a = new ArrayList();
        a.add(u1);
        User u2 = new User("A");
        a.contains(u2);     // false 因为没有重写equals方法
    }
}

class User{
    private String name;
    public User(String name) {this.name = name;}
}
```

改正为：

```java
public class ContainsDemo {
    public static void main(String[] args) {
        User u1 = new User("A");
        ArrayList a = new ArrayList();
        a.add(u1);
        User u2 = new User("A");
        System.out.println(a.contains(u2));     // true
    }
}

class User{
    private String name;
    public User(String name) {this.name = name;}

    @Override
    public boolean equals(Object obj) {
        if(obj instanceof User){
            User o = (User)obj;
            return this.name == o.name;
        }return false;
    }
}
```

### String类

> 属于不可变的类型，直接存储在“方法区”的字符串常量池中。

```java
String a = "xyz";
String b = "xyz";
System.out.println(a == b);	//true
String a1 = new String("xyz");
String a2 = new String("xyz");
System.out.println(a1 == a2);	//false
System.out.println(a1.equals(b));	//true

System.out.println("xyz".equals(a));	//一般使用这一种方式比较防止空指针异常
```

创建字符串的常用方法：

`String(byte [], @pre offset, @pre length)` 通过byte数组构造字符串

`String(char [], @pre offset, @pre count)`

`contains()`  `endsWith()` `equalsIgnoreCase(String s)`  `getBytes(byte)`

`indexOf(String s)` `matches(String regex)`  `replace(String oldChar, String newChar)`

`replaceAll(String regex, String replacement)`  `split(String regex)`

`substring(int begin, int end)  [begin, end)`

`valueOf()`  将非字符串类型的变量转化为字符串。`println()`的底层实现就是valueOf

<h4>StringBuffer和StringBuilder</h4>
> 如果业务中要使用到频繁的拼接字符串，就会占用大量的方法区，造成内存空间的浪费。

底层实现是使用的`byte[]`

* StringBuilder兼容StringBuffer，但是StringBuilder不是线程安全的，缓冲区需要保证线程安全。

* 为了减小底层的多次扩容，可以指定初始容量。

### Arrays工具类

```java
public static void main(String[] args) {
    int []arr = {9,2,5,6,4,2};

    System.out.println(Arrays.toString(arr));   //tostring
    Arrays.sort(arr);   //sort  [2, 2, 4, 5, 6, 9]
    System.out.println(Arrays.toString(arr));   //tostring
    System.out.println(Arrays.binarySearch(arr, 5));

    //数组复制
    int[] newArr = Arrays.copyOf(arr, 3);
    System.out.println(Arrays.toString(newArr));    //[2, 2, 4]
    int[] newArr2 = Arrays.copyOfRange(arr, 1, 5);  // 区间复制 [1, 5)
    System.out.println(Arrays.toString(newArr2));   //[2, 4, 5, 6]
    System.arraycopy(src,srcPos , des,desPos ,length );	//也可以使用system的这个复制

    //数组比较
    int[] arr3 = {1,2,3};
    int[] arr4 = {1,2,3};
    System.out.println(Arrays.equals(arr3, arr4));  //true
    System.out.println(arr3 == arr4);       //false 相当于arr3.equals(arr4) 比较的是地址的值

    //数组填充
    Arrays.fill(arr3, 10);
    System.out.println(Arrays.toString(arr3));
}
```

### 集合(Collection, Iterator, List, Set, Map, 泛型)

> 都在java.uil.*下

所有以单个元素存储的集合的超级父接口是`java.util.Collection`，继承了`Iterable`类表示都是可迭代可遍历的，使用迭代器`Iterator`来进行处理，其方法有`hasNext()` `next()` `remove()`

所有以键值对存储的超级父接口都是`java.util.Map`。

Collection：List，Set，Queue，Deque，SortedSet等

Map：HashMap，HashTable（线程安全）

<h4>Collection</h4>
**List**

> 父类Collection，存储元素有序可重复有下标。

ArrayList：底层采用数组结构（非线程安全）

​	`size()`获取数组的长度，在创建ArrayList的时候预先估计一下给定的容量，减小多次分配。

```java
public class Collection01 {
    public static void main(String[] args) {
        ArrayList a = new ArrayList();
        a.add(1);
        a.add(3.2);
        a.add(true);
        System.out.println(a.size());
        a.clear();
        a.add("e");
        a.add("god");
        a.set(1, 6);
        a.add("GOod");
        a.remove("e");
        System.out.println(a.isEmpty());
        Object[] array = a.toArray();   // 转化为数组

        // 迭代

        Iterator it = a.iterator();	//适用于所有的Set，List，!!!集合只要改变，迭代器必须重写!!!
        while(it.hasNext()){
            // 删除时候只能使用 it.remove()，来删除当前元素
            System.out.print(it.next()+",");
        }
    }
}
```



LinkedList：采用双向链表

Vector：底层采用数组结构（线程安全 所有方法含synchronized，但是效率较低使用少）

**Set**

> 父类Collection，无序不重复没有下标

HashSet：底层实际上是HashMap，实际存储到HashMap。等同于放在HashMap的Key部分了。

TreeSet：底层是TreeMap

`contains(Object o)`：底层是有`equals` 方法实现的，因此对于自己定义的类需要实现重写equals方法。

`remove(Object o)`：底层同样调用了`equals` 方法。

TreeSet Demo:

```java
public class GenericDemo {
    public static void main(String[] args) {
        Set<Integer> s = new TreeSet<>();
    
        s.add(10);
        s.add(21);
        s.add(24);
        s.add(13);
        s.add(14);
        s.add(20);

        for (int i : s) {
            System.out.print(i + ",");
        }
    }
}

// 10,13,14,20,21,24
```



<h4>Map</h4>
HashMap：底层哈希表，非线程安全。

TreeMap：底层是二叉树，key可以按照大小顺序排序

```java
public static void main(String[] args) {	//使用迭代器迭代map
    Map<Integer, String> map = new HashMap<>();
    map.put(2, "Good");
    map.put(1, "God");
    map.put(3, "Bike");

    Set<Integer> keys = map.keySet();
    Iterator<Integer> it = keys.iterator();
    while (it.hasNext()) {
        int k = it.next();
        String v = map.get(k);
        System.out.println(v);
    }
}

// foreach类型
public static void main(String[] args) {
    Map<Integer, String> map = new HashMap<>();
    map.put(2, "Good");
    map.put(1, "God");
    map.put(3, "Bike");
    for (int i : map.keySet()) {
        String s = map.get(i);
        System.out.println(s);
    }
}
```



<h4>泛型</h4>
> Generic，存储类型统一零 ，不需要向下转型了。

`ArrayList<User> uList = new ArrayList<User>();`

钻石表达式（JDK8）：`ArrayList<User> uList = new ArrayList<>();`	后面的“User”可以不用写。

### 枚举





### 反射

> java.lang.reflect.* 。可以通过java的反射机制操作（读和修改）字节码文件

相关类：

* java.lang.Class，代表字节码文件
* java.lang.reflect.Method，方法字节码
* java.lang.reflect.Constructor，构造器字节码
* java.lang.reflect.Field，属性字节码

<h4>获取Class的三种方式：</h4>
* `Class.forName()`

  ```java
  Class c1 = Class.forName("java.lang.String");
  ```

* `obj.getClass()`

  ```java
  public static void main(String[] args) throws ClassNotFoundException {
      Class c1 = Class.forName("java.lang.String");
      //c1 就代表了字节文件或者 String 类
      String s = "";
      Class c2 = s.getClass();
      System.out.println(c1 == c2);	//true
  }
  ```

* `${class}.class`

  `String.class`，`Integer.class`，`Date.class`，`double.class`

<h4>实例化反射Class</h4>
* 为什么使用反射实例化class文件更加灵活？

  在编写代码的时候可以跟去读取的流文件来得到具体的类名。在想创建其他对象的时候只需要修改文件（比如properties文件）的内容即可，不需要再修改代码重新编译。更加灵活。符合开闭原则。

* 如果只是像让代码的静态代码执行，只需要`class.forName()`即可。更不需要`newInstance()`

方式一：

```java
public static void main(String[] args) throws ClassNotFoundException {
    Class c = Class.forName("com.yz.reflect.User");
    try {
        Object o = c.newInstance();
        System.out.println(o);
    } catch (InstantiationException e) {e.printStackTrace();} 
    catch (IllegalAccessException e) {e.printStackTrace();}
}
```

这种方法需要使用`obj.newInstance()`来实例化，必须要保证无参数构造方法存在才可以。

### 文件IO流

流的分类：输入流输出流，字符流字节流。再java.io中。

四大家族：字节流：`java.io.InputStream`，`java.io.OutputStream`，字符流：`java.io.Reader`，`java.io.Writer`

所有的流都实现了`java.io.Closeable` ，所有的输出流还实现了`java.io.Flushable`

需要掌握的流：

* 文件专属：`java.io.FileInputStream` `java.io.FileOutputStream` `java.io.FileReader` `java.io.FileWriter`
* 转换流（字节$\iff$字符流）`java.io.InputStreamReader` `java.io.InputStreamWriter`
* 缓冲流：`java.io.BufferdReader` `java.io.BufferdWriter` `java.io.BufferdInputStream` ``java.io.BufferdOutputStream`
* 数据流专属：`java.io.DataInputStream` `java.io.DataOutputStream`
* 标准输出流：`java.io.PrintWriter` `java.io.PrintStream`
* 对象专属流：`java.io.ObjectInputStream` `java.io.ObjectOutputStream`

<h4>文件流</h4>
**读取文件字节流：**

```java
public class FileInputStream01 {
    public static void main(String[] args) {
        FileInputStream fis = null;

        try {
            fis = new FileInputStream("C:/prac/Java/JavaLearnNotes/projects/Threads/src/a.txt");
            // 一次性读
		   int cnt = 0;
            byte []data = new byte[4];
            while((cnt = fis.read(data)) != -1){
                System.out.println(new String(data, 0, cnt));	//转成字符串必须要指定长度，否则会出现覆盖的情况
            }
            
            // 一次读取一个字节
            int data;
            while((data = fis.read()) != -1)	
                System.out.println(data);
        } catch (IOException e) {e.printStackTrace();}
        finally{
            try {fis.close();}
            catch (IOException e) {e.printStackTrace();}
        }
    }
}
```

其他函数：`available():`剩下没有读的字节数量 和 `skip(int n):`跳过n个字节不读。

**写出文件流：**

`new FileOutputStream(String filename, Boolean append)` append 可以决定是否为追加模式。

```java
public class FileOutput {
    public static void main(String[] args) {
        FileOutputStream fo = null;

        try {
            fo = new FileOutputStream("myfile");
            byte []data = {97,99,101,103};
            fo.write(data);
        } catch (IOException e) {e.printStackTrace();} 
        finally{
            try {fo.close();}
            catch (IOException e) {e.printStackTrace();}
        }
    }
}
```



<h4>查看文件路径</h4>
> src目录就是工程的根目录

```java
public static void main(String[] args) {
    //getResource()  是获取类目录（src目录）下的文件
    String path = Thread.currentThread().getContextClassLoader().getResource("a.txt").getPath();
    System.out.println(path);
}
```

或者资源绑定器(绑定xxx.properties)：比如获取src目录下的user.properties  `ResourceBundle rb = ResourceBundle.getBundle("user")`

## 面向对象	

### 基本创建对象

```java
public class Person {
    public static void main(String[] args) {
        Army army = new Army();
        System.out.println(army);
    }
}

class Army {
    private String name;
    private int age, weight;
}
```

* 只有第一次创建对象的时候创建类`loadClass()`，第二次创建对象的时候就不会再创建类。

### 成员变量和局部变量

```java
class Army {
    private String name;    // global var
    private int age, weight;
    public int say(){
        int num;    // loval var
        {
            int k;  // loval var
        }
        return age + weight;
    }
}
```

成员变量有初始值，局部变量没有初始值。

### 构造器

重载构造器之后，系统不会默认分配构造器。

```java
class Person{
    String name;
    int age, height;
    public Person(){}
    public Person(String name){
        this.name = name;
    }
    public Person(String name, int age){
        this(name);
        this.age = age;
    }
    public Person(String name, int age, int height){
        this(name, age);
        this.height = height;
    }
}
```

* 同一个类中的构造器可以相互this调用，但是this构造器必须放在第一行。

### static关键字

**修饰属性**

```java
public class Demo {
    static int cnt = 0;
    int age;

    public Demo(){}

    public Demo(int age) {
        this.age = age;
    }
    public static void main(String[] args) {
        Demo d1 = new Demo();
        Demo d2 = new Demo();
        d1.age = 1;
        d2.age = 2;
        d1.cnt = 20;
        d2.cnt = 30;
        System.out.println(d1.age);
        System.out.println(d2.cnt); //直接通过类名访问  Demo.cnt
    }
}
```

* 在加载类的同时会将静态的内容（静态域）也加载到方法区。被所有对象共享

**修饰方法：**

```java
public class Demo {
    static int cnt = 0;
    int age;

    public Demo(){}

    public Demo(int age) {
        this.age = age;
    }

    public static String say(){
        // this.Demo  不可以
        // this.age	不可以
        // Demo.cnt 可以
        return "school" ;
    }
}
```

* 非静态可以访问静态
* 静态方法不可以访问静态属性和方法

### 代码块

> 分为：普通块，构造块，静态块，同步块（多线程）

```java
public class CodeBlock {
    int age;
    static int height;

    {//构造块（很少用）
        System.out.println("构造块");
    }

    static {//静态块
        System.out.println("静态块");
    }

    public CodeBlock(int age) {
        this.age = age;
        {// 普通块
            System.out.println("普通块");
        }
    }

    public static void main(String[] args) {
        CodeBlock codeBlock = new CodeBlock(20);
    }
}

//静态块
//构造块
//普通块
```

执行顺序：静态块 -> 构造块 -> 普通块

静态块：只在类加载的时候执行一次。一般用于创建工厂、数据库的初始化信息。

### 包和import

包（package）可以解决重名问题和权限问题

* 全部小写，中间用“.”隔开，公司域名倒着写com.tencent，具体模块com.tencent.login

* 但是不能使用系统中的关键字nul,con,com1-com9

* 必须在非注释的第一行，否则之间包名调用`com.yz.Demo demo = com.yz.Demo();`

* java.lang下的包直接用

* 包之间没有包含与被包含的责任

  ```java
  tencent
  丨Person
  丨一login
      丨一Administrator
  
  import com.tencent.*;
  import com.tencent.login.*;
  ```

* 静态导入：

  ```java
  import static java.lang.Math.*;	//导入java.lang下Math类中所有的静态方法
  
  public class CodeBlock {
      public static void main(String[] args) {
          System.out.println(round(5.6));	//直接可以用round
      }
  }
  ```

### 面向对象三大特性之一——封装

> 高内聚：内部细节自己完成。低耦合：仅对外暴露少量的方法。“private”。封装：encapsulation

```java
public class Girl {
    private int age;

    public int getAge() {
        return age;
    }
}
```

### 面向对象三大特性之一——继承

> Inheritance. 继承 is - a 关系。

```java
public class Person {
    private int age, height;
    private String name;

    public int getAge() {return age;}

    public void setAge(int age) {this.age = age;}
}

class Girl extends Person{
    private int brain;

    public int getBrain() {return brain;}

    public void setBrain(int brain) {this.brain = brain;}

    public static void main(String[] args) {Girl girl = new Girl();}
}
```

* 子类无法访问private属性和方法，但是仍可以使用暴露的方法对属性进行修改。
* 便于代码扩展，是多态的前提
* 不允许多继承，但继承具有传递性

**权限修饰符：**

|           | 同一个类 | 同一个包 | 子类 | 所有类 |
| :-------: | :------: | :------: | :--: | :----: |
|  private  |    *     |          |      |        |
|  default  |    *     |    *     |      |        |
| protected |    *     |    *     |  *   |        |
|  public   |    *     |    *     |  *   |   *    |

**重载和重写：**

重写：发生在子类和父类之间的方法。参数的个数，类型必须相同。但是加入返回值是引用类型，父类的返回值是子类的返回值的父类，则可以。父类的权限修饰符要低于子类。

重载：发生在同一个类中，方法名相同，形参列表不同。

### super关键字

```java
public class Girl extends Person{
    private int brain;

    public int getBrain() {
        return brain;
    }

    public void setBrain(int brain) {
        this.brain = brain;
        super.getAge();		// 默认情况下super默认不写。当子类中后者局部变量也含相同变量名
    }

    public static void main(String[] args) {
        Girl girl = new Girl();
    }
}
```

一般执行流程：子类构造器`Girl()`，父类构造器`Person()`

错误❌：

```java
public class Girl extends Person{
    private int brain;
    
    public Girl(int id, String name, int age){
        super(id, name);	//❌	
        this(age);		//❌	因为构造器只能放在第一行，两者冲突，因此只能存在一个构造器
    }

    public static void main(String[] args) {
        Girl girl = new Girl();
    }
}
```

### Object类

**`toString()`**

```java
public class Girl extends Person{
    private int brain;

    public static void main(String[] args) {
        Girl girl = new Girl();
        System.out.println(girl);
    }
}
//com.yz3.Girl@1540e19d
```

**`equals()`**

底层依旧使用的是`==`，需要重写equals方法

```java
public class Girl extends Person{
    private int brain;

    public static void main(String[] args) {
        Girl girl = new Girl();
        System.out.println(girl);
    }

    @Override
    public boolean equals(Object obj) {	// 初级版 需要instanceof改进 
        Girl girl = (Girl) obj;
        return this.brain == girl.brain && this.getAge() == girl.getAge();
    }
}
```

`instanceof`

```java
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Girl girl = (Girl) o;
        return brain == girl.brain;
    }
```

### 面向对象三大特性之一——多态

> 多态和属性无关，多态指的是方法的多态，而不是属性的多态。

```java
public class Animal {
    public void shout(){System.out.println("小动物叫");}
}

class Cat extends Animal{
    @Override
    public void shout() {System.out.println("猫猫叫");}
}

class Dog extends Animal{
    @Override
    public void shout() {System.out.println("狗狗叫");}
}

class Girl{
    public void play(Animal a){a.shout();}
}

class Test{
    public static void main(String[] args) {
        Girl girl = new Girl();
        Animal dog = new Dog();
        girl.play(dog);

        Animal cat = new Cat();
        girl.play(cat);
    }
}
```

现在的代码扩展性好，当小女孩想和不同动物玩的时候就不需要再改代码了。符合开闭原则：扩展是开放的，修改是关闭的。

多态的要素：继承，重写，父类引用指向子类对象`父类 v = new 子类()`。

**向上转型和向下转型：**

```java
Animal a = new Dog();	// 多态
Dog d = (Dog)a;	//向下转型
```

### 简单工厂设计模式

传参即可返回子类，方法必须是static

### final关键字

修饰变量：

```java
final Dog d = new Dog("10");
//d = new Dog("20");  //error
d.name = "30";	//okay
```

`final`只管`d`的地址不被改变。作为函数参数的时候不可以进行改变。相当于C++中的`const`

修饰方法：final修饰的方法，子类不可以继承

修饰类：final修饰的类，该类不可以被继承，里面的方法也可以不用final修饰。比如`Math`类。

其中Math类中有一个细节`private Math(){}`。其构造方法为private类型，就是不让任何人去随意实例化。

### 抽象类和抽象方法

```java
public abstract class Person {
    public abstract void say();

    public void look(){
        System.out.println("Looking for u");
    }
}
```

* 含抽象方法的类必须是抽象类
* 子类必须重写抽象父类中的抽象方法，如果不重写，则子类必须是抽象类
* 抽象类不可以创建对象

**问题**：

* 抽象类中是否有构造器？有。构造器的作用给子类创建对象的时候先用super调用父类构造器。
* 抽象类不能被final修饰，因为抽象类必须要被继承才有作用。

### 接口

* 接口（interface）没有构造器。类和接口的关系是实现关系（has-a关系）。

* Java只有单继承，但有多实现。
* 也可以多态

```java
public interface Person {
    int NUM = 10;
    void say();
    int getHeight();
}

class Student implements Person{
    @Override
    public void say() {System.out.println("ni hao");}

    @Override
    public int getHeight() {return 0;}
}
```

JDK1.8之后新增内容：

* default修饰符新增的内容，default可以修饰非抽象方法。
* 可以写静态方法，但是静态方法不能重写

### 内部类

> 成员内部类和局部内部类（方法内、块内、构造器内）

**成员内部类：**

```java
public class Person {
    int age;
    String name;
    static double height = 10.8;
    public class InnerClass{
        int age;
        public void say(){
            System.out.println(age);    // InnerClass.this.age
            System.out.println(Person.this.age);
            System.out.println(height);
        }
    }

    static class Ic2{
        public void go(){
            System.out.println(height);
//            System.out.println(age);      error
        }
    }

    public void stand(){
        InnerClass innerClass = new InnerClass();
        innerClass.say();
    }
}

class Test{
    public static void main(String[] args) {
        Person person = new Person();
        Person.InnerClass innerClass = person.new InnerClass();	// 创建非静态内部类

        Person.Ic2 ic2 = new Person.Ic2();		// 创建静态内部类
    }
}
```

**局部内部类：**

* 在局部内部类中访问的变量必须是final修饰的

* 如果类只使用一次的话，可以使用内部类（安卓常用）

* 匿名内部类

  ```java
  public class Person {
      int age;
      String name;
      static double height = 10.8;
      
      public Comparable say(){
          return new Comparable() {	// 返回匿名内部类对象
              @Override
              public int compareTo(Object o) {
                  return 0;
              }
          };
      }
  }
  ```

## 多线程

### 多线程创建

线程创建的三种方式：Thread类、Runnable接口、Callable接口

**创建方式一：继承Thread类**

* 继承Thread类并且重写`run()`方法，创建对象调用`start()`，由线程执行`run()`方法。(不直接使用`run()`方法，否则相当于程序调用，为一个线程)

```java
public class Thread1 extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 200; i++) {
            System.out.println("线程"+ i);
        }
    }

    public static void main(String[] args) {
        Thread1 thread1 = new Thread1();
        thread1.start();
        for (int i = 0; i < 200; i++) {
            System.out.println("Main Thread" + i);
        }
    }
}
```

**创建方式二：实现Runnable接口**

```java
public class RunnableDemo implements Runnable{
    @Override
    public void run() {
        for (int i = 0; i < 30; i++) {
            System.out.println("Runnable== " + i);
        }
    }

    public static void main(String[] args) {
        // 创建实现类的对象
        RunnableDemo runnableDemo = new RunnableDemo();
        // 创建线程对象，通过线程对象开启“代理”
        Thread thread = new Thread(runnableDemo);
        thread.start();

        for (int i = 0; i < 30; i++) {
            System.out.println("Main Thread" + i);
        }
    }
}
```

<font color='#00ff00'>避免单继承的局限性，灵活方便。</font>

**创建方式三：实现Callable接口**

流程：

1. 创建执行服务	`ExecutorService ser = Executors.newFixedThreadPool(3);`
2. 提交执行     `Future<Boolean> r1 = ser.submit(t1);`
3. 获取结果      `Boolean rs1 = r1.get();`
4. 关闭服务      `ser.shutdownNow();`

```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class CallableDemo implements Callable<Boolean>{
   
    private String name;
    @Override
    public Boolean call() {
        for (int i = 0; i < 10; i++) {
            System.out.println("Callable： " + name + " ->  " + i);
        }
        return true;
    }
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        CallableDemo t1 = new CallableDemo("小红");
        CallableDemo t2 = new CallableDemo("小黄");
        CallableDemo t3 = new CallableDemo("小韩");

        //创建服务
        ExecutorService ser = Executors.newFixedThreadPool(3);  //3个线程池

        //提交执行
        Future<Boolean> r1 = ser.submit(t1);
        Future<Boolean> r2 = ser.submit(t2);
        Future<Boolean> r3 = ser.submit(t3);

        //获取结果
        Boolean rs1 = r1.get();
        Boolean rs2 = r2.get();
        Boolean rs3 = r3.get();

        //关闭服务
        ser.shutdownNow();

        System.out.println(rs1 + " = " + rs2  + " = " + rs3);

    }

    public CallableDemo(String name) {
        this.name = name;
    }
    
}
```

Callable的好处：

* 可以定义返回值
* 可以抛出异常

### 线程不安全情况

> 类型：多个线程拿同一个资源

以买火车票为案例。

```java
public class Tickets implements Runnable{
    private int ticketsNum = 10;
    @Override
    public void run() {
        while (true){
            if (ticketsNum<=0)break;
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "==> 抢到了第" + ticketsNum-- + "张票。");
        }
    }

    public static void main(String[] args) {
        Tickets tickets = new Tickets();

        new Thread(tickets, "小红").start();
        new Thread(tickets, "小明").start();
        new Thread(tickets, "小黄").start();

    }
}
```

输出：

```
小黄==> 抢到了第8张票。
小明==> 抢到了第10张票。
小红==> 抢到了第9张票。
小明==> 抢到了第7张票。
小黄==> 抢到了第6张票。
小红==> 抢到了第5张票。
小红==> 抢到了第4张票。
小黄==> 抢到了第4张票。
小明==> 抢到了第4张票。
小红==> 抢到了第1张票。
小黄==> 抢到了第3张票。
小明==> 抢到了第2张票。
```

出现了线程不安全的情况，数据紊乱。

### 龟兔赛跑模型

> 两个对象共用一个资源

```java
public class Race implements Runnable{
    private String winner;

    @Override
    public void run() {
        for (int i = 0; i <= 100; i++) {
            if (Thread.currentThread().getName().equals("rabbit") && i % 10 == 0){
                try {
                    Thread.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            if (isGameOver(i))break;
            System.out.println(Thread.currentThread().getName() + "跑了" + i + "步");
        }

    }

    public boolean isGameOver(int steps){
        if (winner != null){
            return true;
        }else{
            if (steps>=100){
                winner = Thread.currentThread().getName();
                System.out.println("胜利者是" + winner);
                return true;
            }
        }
        return false;
    }

    public static void main(String[] args) {
        Race race = new Race();

        new Thread(race, "rabbit").start();
        new Thread(race, "tortoise").start();
    }
}
```

### 静态代理

> static proxy

```java
public class StaticProxy {
    public static void main(String[] args) {
        WeddingCompant w = new WeddingCompant(new You());
        w.happyMarry();

        // 相当于这个
        new Thread(()->System.out.println("Good")).start();	// 两个方法开启的方式类似
        new WeddingCompant(new You()).happyMarry();			//
    }
}


interface Marry{void happyMarry();}

class You implements Marry{
    @Override
    public void happyMarry() {System.out.println("Happy marry");}
}

class WeddingCompant implements Marry{
    private Marry target;

    @Override
    public void happyMarry() {
        before();	//做原对象不能做的行为，进行细节补充
        target.happyMarry();
        after();
    }

    public void before(){System.out.println("Before wedding preparation");}

    public void after(){System.out.println("After wedding charge.");}

    public WeddingCompant(Marry target) {this.target = target;}
}
```

### lambda表达式

> 避免匿名内部类过多，

**函数式接口：**对于一个类，如果只含有一个抽象方法，则其是一个函数式接口。可以直接继承。

```java
public class TestLambda {

    public static void main(String[] args) {
        Bribe b = new Bribe(){
            @Override
            public void marry(String name) {
                System.out.println("匿名内部类" + name);
            }
        };

        b.marry("Karry");

        b = (name) -> System.out.println("Lambda 表达式" + name);	// 可以简化类型
        b.marry("Mate");
    }
}

interface Bribe{
    void marry(String name);
}
```

### 线程状态

<img src="https://i.loli.net/2021/03/29/AwWB4RUqr5zQKn3.png" alt="image-20210329144927183.png" style="zoom:67%;" />

**状态：**

* 创建状态：`new`创建线程

* 就绪状态：调用`start()`方法，线程会立即进入就绪状态，但不意味着立即执行

* 运行状态：线程获得了CPU处理机资源

* 阻塞状态：线程调用`sleep() wait()`或者`同步锁定`的时候线程就会进入阻塞状态。阻塞解除之后继续进入就绪状态。

* 死亡状态：线程运行完毕或者终止。

**线程方法：**

* 停止线程

  > 不推荐使用`destroy()` `resume()` `stop()`等方法；推荐使用标志位进行中止让自己停下来
  ```java
  public class ThreadStop implements Runnable{
      private Boolean flag = true;
      
      @Override
      public void run() {
          int i = 0;
          while(flag)System.out.println("Thread： " + i++);
      }
  
      public void stop(){flag = false;}
  
      public static void main(String[] args) {
          ThreadStop ts = new ThreadStop();
          new Thread(ts).start();
  
          for (int i = 0; i < 1000; i++) {
              System.out.println("Main Thread " + i);
          }
          ts.stop();
      }
  }
  ```

* 线程休眠

  > `sleep()`函数，模拟网络延时（放大问题的发生率，可能引起线程不安全），模拟倒计时

  **每个对象都有一把锁，但是`sleep()` 不会释放锁。**

* 线程礼让（这个需要深入）

  > `yield()`函数。礼让线程，让当前执行的线程暂停但不阻塞转为就绪状态。**但礼让不一定会成功**

  ```java
  public class ThreadYield{
  
      public static void main(String[] args) {
          my ty = new my();
  
          new Thread(ty, "A").start();
          new Thread(ty, "B").start();
      }
  }
  
  
  class my implements Runnable{
      @Override
      public void run() {
          System.out.println(Thread.currentThread().getName() + "线程开始");
          Thread.yield();
          System.out.println(Thread.currentThread().getName() + "线程结束");
      }
  }
  ```

* 线程插队JOIN

  > `join()`合并线程，待此线程执行完毕之后再执行其他线程，其他线程阻塞。十分霸道

  ```java
  public class ThreadJoin implements Runnable{
      @Override
      public void run() {
          for (int i = 0; i < 10; i++) {
              System.out.println("线程VIP JOIN" + i);
          }
      }
  
      public static void main(String[] args) throws InterruptedException {
          ThreadJoin tj = new ThreadJoin();
          Thread t = new Thread(tj);
          t.start();
  
          for (int i = 0; i < 20; i++) {
              if(i == 15)t.join();
              System.out.println("主线程 " + i);
          }
      }
  }
  ```

  输出如下：

  ```
  主线程 0
  线程VIP JOIN0
  主线程 1
  线程VIP JOIN1
  主线程 2
  线程VIP JOIN2
  主线程 3
  主线程 4
  主线程 5
  主线程 6
  主线程 7
  主线程 8
  主线程 9
  主线程 10
  主线程 11
  主线程 12
  主线程 13
  主线程 14
  线程VIP JOIN3		到15的时候插入，直到VIP线程死亡为止
  线程VIP JOIN4
  线程VIP JOIN5
  线程VIP JOIN6
  线程VIP JOIN7
  线程VIP JOIN8
  线程VIP JOIN9
  主线程 15
  主线程 16
  主线程 17
  主线程 18
  主线程 19
  ```

### 观测线程状态

NEW，RUNNABLE，BLOCKED，WAITING，TIMED_WAITING，TERMINATED

```java
import java.lang.Thread.State;

public class ThreadStatus {
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(() -> {
            for (int i = 0; i < 1; i++) {
                try {
                    Thread.sleep(200);
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
            System.out.println("Over");
        });

        State state = t.getState();
        System.out.println(state); // NEW

        t.start();
        state = t.getState();
        System.out.println(state);  // RUNNABLE

        while(state != State.TERMINATED){
            Thread.sleep(20);
            state = t.getState();
            System.out.println(state);
        }

    }
}
```

输出：

```
NEW
RUNNABLE
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
TIMED_WAITING
Over
TERMINATED
```

### 线程优先级

设置方法：`setPriority(int)` `getPriority()`

优先级数字：用1~10表示。常量表示`Thread.MIN_PRIORITY`  `Thread.NORM_PRIORITY`  `Thread.MAX_PRIORITY`

```java
public class ThreadPriority {
    public static void main(String[] args) {
        Pr pobj = new Pr();

        Thread t1 = new Thread(pobj);
        Thread t2 = new Thread(pobj);
        Thread t3 = new Thread(pobj);
        Thread t4 = new Thread(pobj);

        t1.start();

        t2.setPriority(3);
        t2.start();

        t3.setPriority(Thread.MAX_PRIORITY);
        t3.start();

        t4.setPriority(Thread.MIN_PRIORITY);
        t4.start();
    }
}

class Pr implements Runnable{
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() +" Priority -> " + Thread.currentThread().getPriority());
    }
}
```

输出：

```
Thread-0 Priority -> 5
Thread-3 Priority -> 1
Thread-1 Priority -> 3
Thread-2 Priority -> 10
```

### 守护线程

> Daemon

线程分为用户线程和守护线程。虚拟机必须确保用户线程执行完毕，不需要等待守护线程执行完毕。

```java
public class ThreadDaemon {
    public static void main(String[] args) {
        DaemonProcess daemon = new DaemonProcess();
        Thread daemont = new Thread(daemon);
        daemont.setDaemon(true);
        daemont.start();

        new Thread(new AccProcess()).start();
    }
}

class DaemonProcess implements Runnable{
    @Override
    public void run() {
        while(true){
            System.out.println("Daemon running");
        }
    }
}

class AccProcess implements Runnable{
    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println("Person good.");
        }

        System.out.println("Good Bye...");
    }
}
```

### 线程同步

> 并发：多个线程操作同一个资源。

多个线程需要同时访问一个资源，这就需要使用排队的思想来解决这个问题。**线程同步**其实是一个等待机制，多个线程访问此对象的线程需要进入这个对象的**等待池**。而且还需要一种锁机制(synchronized)

### 线程不安全案例

**多人买票问题：**

```java
public class unsafeBuyTickets {
    public static void main(String[] args) {
        BuyTickets station = new BuyTickets();
        
        new Thread(station, "A").start();
        new Thread(station, "B").start();
        new Thread(station, "C").start();
    }
}

class BuyTickets implements Runnable{
    private int ticketsCnt = 10;
    private Boolean flag = true;

    @Override
    public void run() {
        while(true){
            if(!flag)break;
            buyTicket();
        }
    }

    public void buyTicket(){
        if(ticketsCnt<=0){
            flag = false;
            return;
        }
        try{Thread.sleep(100);}
        catch (InterruptedException e){e.printStackTrace();}
        System.out.println(Thread.currentThread().getName() + "抢到了第" + ticketsCnt-- + "张票");
    }
}
```

输出：

```
B抢到了第8张票
A抢到了第10张票
C抢到了第9张票
B抢到了第7张票
C抢到了第6张票
A抢到了第5张票
C抢到了第4张票
A抢到了第3张票
B抢到了第3张票
C抢到了第2张票
A抢到了第1张票
B抢到了第0张票
C抢到了第-1张票
```

会出现多人买到同一张票，或者出现负数的问题。多人同一个时间段内进入判断条件的时候变量`ticketCnt`还大于0，由于延时的因素，前面先运行的程序先取走了票，后面的程序未再进行检测，因此导致负数。

**不安全取钱问题：**

```java
package com.yz.unsafeDemo;

public class unsafeBank {
    public static void main(String[] args) {
        Account acc = new Account(100);
        Bank a = new Bank(50, acc);
        Bank b = new Bank(60, acc);

        new Thread(a, "A").start();
        new Thread(b, "B").start();
    }
}

class Account{
    private int money;
    public Account(int money) {this.money = money;}
    public int getMoney() {return money;}
    public void setMoney(int money) {this.money = money;}
}

class Bank implements Runnable{
    private int getMoney;
    Account acc;
    @Override
    public void run() {
        if(acc.getMoney() - getMoney <= 0)System.out.println("你的钱余额不足！");
        else{
            try {Thread.sleep(1000);} 	// 延时会放大错误的发生率
            catch (InterruptedException e) {e.printStackTrace();}
            acc.setMoney(acc.getMoney() - getMoney);
            System.out.println(Thread.currentThread().getName() + "已经取走钱" + getMoney + "元。剩余："+ acc.getMoney());
        }
    }

    public Bank(int getMoney, Account acc) {
        this.getMoney = getMoney;
        this.acc = acc;
    }
}
//A已经取走钱50元。剩余：-10
//B已经取走钱60元。剩余：-10
```

**不安全的集合**

```java
public class unsafeList {
    public static void main(String[] args) {
        List<String> l = new ArrayList<String>();
        for (int i = 0; i < 10000; i++) {
            new Thread(() -> {
                l.add("a");
            }).start();
        }
        System.out.println(l.size());
    }
}
//7686	延时后会变多
```

### ※同步方法和同步块

使用`synchronized`的关键字对于将要进行增删改查操作对象进行修饰。

修复多人取票问题：

```java
public synchronized void buyTicket(){
    if(ticketsCnt<=0){
        flag = false;
        return;
    }
    try{Thread.sleep(500);}
    catch (InterruptedException e){e.printStackTrace();}
    System.out.println(Thread.currentThread().getName() + "抢到了第" + ticketsCnt-- + "张票");
}
```

修复多人取钱问题：

​	对于进行增删改查的acc对象进行synchronized限制。

```java
@Override
    public void run() {
        synchronized(acc){
            if(acc.getMoney() - getMoney <= 0)System.out.println("你的钱余额不足！");
            else{
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
                acc.setMoney(acc.getMoney() - getMoney);
                System.out.println(Thread.currentThread().getName() + "已经取走钱" + getMoney + "元。剩余："+ acc.getMoney());
            }
        }
    }
```

修复不安全集合问题：

> 未修复

```java

```

### 死锁

死锁样例，两个对象都占有并且不想放弃对方想要的资源，就造成了死锁。

```java
public class DeadLock{
    public static void main(String[] args) {
        MakeUp g1 = new MakeUp("小莉", 0);
        MakeUp g2 = new MakeUp("小萌", 1);

        new Thread(g1).start();
        new Thread(g2).start();
    }
}

class Mirror {}
class LipStick {}

class MakeUp implements Runnable{
    static final Mirror mirror = new Mirror();
    static final LipStick lipStick = new LipStick();

    private String name;
    private int choice;

    public MakeUp(String name, int choice) {
        this.name = name;
        this.choice = choice;
    }

    @Override
    public void run() {
        try {mu();}
        catch (InterruptedException e) {e.printStackTrace();}
    }

    private void mu() throws InterruptedException {
        if (choice == 1){
            synchronized (mirror){
                System.out.println(name + "拿到了镜子");
                Thread.sleep(1000);
                synchronized (lipStick){
                    System.out.println(name + "拿到了口红。=化妆完毕");
                }
            }
        }else{
            synchronized (lipStick){
                System.out.println(name + "拿到了口红");
                Thread.sleep(2000);
                synchronized (mirror){
                    System.out.println(name + "拿到了镜子。=化妆完毕");
                }
            }

        }
    }
}
```

### 线程通信协作（生产者消费者模型）

## 函数式编程

为什么要学习函数式编程？

* 效率更高
* 消除嵌套地狱
* 可读性更高
* 易于并发编程

### Lambda表达式

一个函数的参数是`interface`，如果这个接口的方法只有一个函数，则可以使用lambda表达式进行代替。简单的形式在上节已经介绍过，这次主要是进阶的。

🔵lambda表达式的省略规则

1. 参数类型可以省略，只有一个参数的时候小括号也可以省略
2. 方法体中只要一句话的时候，返回值对于的大括号和return都可以省略

### Stream流

> Stream流**必须**要有终结操作，否则可能中间操作可能不会调用到；并且Stream流只能使用一次；不会修改源数据

案例：

```java
public static void main(String[] args) {
    List<Author> authors = getAuthors();
    authors.stream()
            .distinct()                     // 去重
            .filter(author -> author.getAge() <= 18)    // 筛选出小于18岁的
            .forEach(System.out::println);  // 每个打印
}
```

🔵创建流：

单列集合：

```java
List<Author> authors = getAuthors();
authors.stream()
```

数组：

```java
Integer a[] = {8,1,1,5,6,3,2,3,3};

Arrays.stream(a)    // 类型是IntStream
        .distinct()
        .filter(i->i>2)
        .forEach(System.out::println);

Stream.of(a)        // 类型是Stream<Integer>
        .distinct()
        .filter(i->i>2)
        .forEach(System.out::println);
```

> 注意：如果类型是基本类型`int`的话，使用`of`方法就可能无法使用`distinct()`和`filter()`方法了

双列集合：

> 双列对象需要转为`EntrySet`才能够使用stream

```java
Map<String, Integer> map = new HashMap<>();
map.put("1", 1);
map.put("3", 4);
map.put("2", 2);

val stream = map.entrySet().stream();
stream.filter(e -> e.getValue() > 2)
        .forEach(System.out::println);
```

🔵中间操作：

* `filter()`筛选符合条件的对象，删去不符合条件的对象

  ```java
  Arrays.stream(a)
          .filter(i->i>2)
          .forEach(System.out::println);
  ```

* `map()`操作，将数据元素进行操作或者转换

  返回将所有作者年龄加5的结果，可以多个map叠加。类型变换的时候Ideaj会有提示。

  ```java
  authors.stream()
          .map(author -> author.getAge() + 5)
          .forEach(System.out::println);
  ```

* `distinct()`方法，其依靠的是Object的`Equals()`方法来进行比对

  ```java
  Arrays.stream(a)
          .distinct()	// 去重
          .forEach(System.out::println);
  ```

* `sorted()`方法

  ```java
  authors.stream()
          .sorted((a, b) -> a.getAge() - b.getAge())
          .forEach(System.out::println);
  ```

  按年龄大小排序（升序）。

  如果调用空参`sorted`方法将需要对应的类取实现`Comparable`接口。

* `limit()`方法

  用来限制流的最大长度，超过的部分会被截取掉。

  这里设置最大长度为3.

  ```java
  authors.stream()
          .limit(3)
          .forEach(System.out::println);
  ```

* `skip(int n)`方法

  跳过流中的前n个元素，返回后面的元素流

  这里忽略了前三个元素

  ```java
  authors.stream()
          .skip(3)
          .forEach(System.out::println);
  ```

* `flatMap()`方法

  可以将一个对象转化为流中的多个对象

  ```java
  authors.stream()
          .flatMap(author -> author.getBooks().stream())	// Books也是List
          .distinct()
          .forEach(i->System.out.println(i.getName()));
  ```

* `peek()`方法

  用来遍历元素并且插入中间操作

  ```java
  long count = authors.stream()
          .flatMap(author -> author.getBooks().stream())
          .distinct()
          .peek(System.out::println)
          .count();
  ```

🔵终结操作

* `forEach()`方法

  对流中的对象进行遍历

  ```java
  authors.stream()
          .forEach(System.out::println);
  ```

* `count()`方法

  用处如其名

  ```java
  authors.stream()
          .count();
  ```

* `max/min`

  返回的结果是`Optional`类型的

  ```java
  Optional<Integer> max = authors.stream()
          .map(author -> author.getAge())
          .max((o1, o2) -> o1 - o2);
  
  System.out.println(max.get());
  ```

* `collect()`方法

  将当前的流转化为集合

  ```java
  List<Integer> collect = authors.stream()
          .map(Author::getAge)
          .collect(Collectors.toList());
  ```

  转化为set：`Collectors.toSet()`

  转化成map:

  > 注意：如果转成map的话，Key不能重复，如果重复会抛出异常`IllegalStateException`，如果需要考虑并发情况的话使用`toConcurrentMap`

  ```java
  Map<Long, String> map = authors.stream()
          .distinct()
          .collect(Collectors.toMap(Author::getId, Author::getName));
  ```

* `anyMatch()`是否存在一个元素满足条件，`allMatch()`是否所有元素满足条件，`noneMatch()`是否所有元素都不满足条件，

  ```java
  boolean b = authors.stream()
          .anyMatch(a -> a.getAge() > 29);
  System.out.println(b);
  
  boolean b = authors.stream()
      .allMatch(a -> a.getAge() < 50);
  System.out.println(b);
  ```

* `findAny()`寻找任意一个元素(不一定是第一个元素)，`findFirst()`返回第一个元素

* `reduce()`

  将上个元素计算的结果作为参数与下个流的元素做运算。

  > `reduce()`函数有三种形式：可以设置result的初值(idnetity)。
  >
  > MapReduce思想类似

  ```java
  // 求和
  authors.stream()
          .map(Author::getAge)
          .reduce((r, e) -> r + e);
  
  // 求最大值
  authors.stream()
      .map(Author::getAge)
      .reduce((r, e) -> r > e ? r : e);
  ```

### Optional

为了防止空指针异常过多冗余的判断，使用Optional优雅的处理异常。

🔵创建Optional对象

可以使用`ofNullable()`方法接收空值。如果**确定**绝对不是空的话，就使用`of`方法。

```java
public static void main(String[] args) {
    List<Author> authors = TestStream.getAuthors();
    Author author = authors.get(0);

    Optional<Author> author1 = Optional.ofNullable(author);
    author1.ifPresent(System.out::println);
}
```

安全消费获得的值使用方法`ifPresent()`方法，或者`get()`方法，后者遇到`null`值会抛出异常。

```java
System.out.println(author1.orElseGet(Author::new));
System.out.println(author1.orElse(new Author()));
```

更安全的获取可以使用方法`orElseGet`或者`orElse`来处理null值时候的默认值。

🔵Optional也支持`filter(),map()`方法。

### 函数式接口

常见函数式接口：

* `Consumer`，只接受处理，不返回，`BiConsumer`可以接收两个参数
* `Function<T,R>`，传入一种参数，返回一种参数，`BiFunction<T,R,U>`，可以接收两个参数
* `Predict<T>`判断型接口，返回结果为bool类型，`BiPredict`类似。
* `Supplier<T>`，生产者接口，不接收，只返回。

对于`Predict`接口，支持`and`和`or`方法连接，用于条件判断：

这个一般用在两个接口直接的使用，不适用实现类的直接调用。

```java
authors.stream()
        .filter(((Predicate<Author>) author -> author.getAge() > 18).and(author -> author.getId() > 2))
        .forEach(System.out::println);
```

### 方法引用

进一步简化代码`::`，使用两个代码。

如果lambda表达式中只有一个方法一行代码，并且无参数或者参数对应的数量是相同的时候。

### 基本类型优化

对于基本类型会有自动装箱和拆箱的开销。

jdk提供了`mapToInt,mapToDouble`等方法来进行转化

### 并行流

```java
authors.stream()
        .parallel()
        .peek((r)->{
            System.out.println("Cur"+Thread.currentThread().getName());
        })
        .filter(author -> author.getAge() > 18)
        .forEach(System.out::println);
```

## Maven

学习视频：[BV1Ah411S7ZE](https://www.bilibili.com/video/BV1Ah411S7ZE)

### 概述

maven是一个项目管理工具，将项目开发和管理过程抽象出一个项目对象模型（POM, project object model）

<img src="https://i.loli.net/2021/08/10/R52yFofbxn819Nr.png" alt="image-20210810043856968" style="zoom:67%;" />

仓库：用于存储jar包

<img src="https://i.loli.net/2021/08/10/rCAQea5EuzOMlU3.png" alt="image-20210810044657523" style="zoom:50%;" />

坐标：用于定位资源的信息

* groupId：所属组织名称（比如：org.apache）
* artifactId：项目名称（SMS，Login）
* version：定义当前的版本号
* packaging：发布的类型`jar`

### 安装和配置环境变量

[Maven下载链接](https://maven.apache.org/download.cgi)

配置环境变量 MAVEN_HOME，并且加入到path中。

🔵Maven仓库配置：

**在其settings标签下进行设置默认下载位置：**

在maven根目录下的`conf/settings.xml`文件中进行配置

```xml
<!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
-->
<localRepository>E:/Notes/Java/.m2/repository</localRepository>
```

**修改maven仓库镜像地址：**

* 同样在`settings.xml`文件内，在`<mirrors></mirrors>`内添加一下配置

  ```xml
  <mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>aliyun</name>
    <url>https://maven.aliyun.com/repository/public</url>
  </mirror>
  ```

### maven的目录结构：



|             目录              |           目的            |
| :---------------------------: | :-----------------------: |
|          ${basedir}           | 存放`pom.xml`和其他子目录 |
|   ${basedir}/src/main/java    |      项目Java源代码       |
| ${basedir}/src/main/resourses | 项目的资源，property文件  |
|   ${basedir}/src/test/java    |  项目的测试类，JUnit代码  |
| ${basedir}/src/test/resourses |      测试使用的资源       |

```
C:.
|   pom.xml
|
\---src
    +---main
    |   +---java
    |   |   \---com
    |   |       \---yz
    |   |               App.java
    |   |
    |   \---resources
    \---test
        +---java
        |   \---com
        |       \---yz
        |               AppTest.java
        |
        \---resources
```

### maven命令

* mvn -version
* mvn compile  编译src/main/目录下的java文件
* mvn clean  将编译后的文件删除
* mvn test  运行test目录下的文件
* mvn package  打包源代码文件
* mvn install  将自己的项目安装到本地仓库中
* -D 指定属性  -P 指定profile，设置运行环境

**编译运行**

* 在项目目录下输入指令`mvn compile`，第一次等下文件下载完毕。
* 运行java文件：`mvn exec:java -Dexec.mainClass="com.yz.App"`(不带“.java”后缀名)

### maven依赖管理

🔵依赖配置：

```xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/junit/junit -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>

</dependencies>
```

🔵依赖传递：

分为直接依赖和间接依赖。

对于依赖之间的间接依赖可能使用到同一个包的不同版本，会有依赖冲突问题。

有三个原则：

* 路径优先：依赖中出现相同资源的时候，层级越深，优先级越低。

  <img src="https://i.loli.net/2021/08/10/XcqM5W2EZS3BePx.png" alt="image-20210810060316103" style="zoom: 67%;" />

  比如1度使用到了junit为1.0版本，2度中使用到了junit为2.0版本，则优先使用1.0版本的junit。

* 声明优先：多个同级依赖使用到相同的间接依赖的时候，先声明的覆盖靠后的。

* 特殊优先：同级依赖使用相同包的不同版本时候，后面覆盖前面

🔵可选依赖

配置`<option>true</option>`，不想让别人看到我的依赖（不透明）

```xml
 <dependency>
     <groupId>junit</groupId>
     <artifactId>junit</artifactId>
     <version>4.12</version>
     <option>true</option>
</dependency>
```

🔵排除依赖

主动断开依赖的依赖（不需要）

```xml
 <dependency>
     <groupId>junit</groupId>
     <artifactId>junit</artifactId>
     <version>4.12</version>
     <exclusions>
     	<exclusion>
          <groupId>log4j</groupId>
     		<artifactId>log4j</artifactId>
       </exclusion>
     </exclusions>
</dependency>
```

🔵依赖范围

`<scope></scope>`用于指定依赖jar包的作用范围，默认为`compile`

| scope  | 主代码 | 测试代码 | 打包     |
| :------: | :--: | :------: | :------: |
| compile | Y  | Y      | Y |
| test        |    | Y       |     |
| provided  | Y | Y |  |
| runtime    |    |        | Y  |

<h4 id="ylfb"></h4>

🔵将源代码中的资源文件也发布到release中，如`mybatisMapper.xml`文件等。

```xml
<build>
    <!--将源代码目录下的其他资源文件也编译到输出文件中-->
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```

### 生命周期和插件

<img src="E:\Notes\Java\2021Java\Java笔记.assets\image-20210810083851187.png" alt="image-20210810083851187" style="zoom:67%;" />

## Spring

> Spring框架分为四部分

视频：[BV1wy4y1D7JT ](https://www.bilibili.com/video/BV1nz4y1d7uy)

### IoC控制反转

> Inverse of Control是一种思想。对象的创建、赋值、管理都是由容器实现的。

java中创建对象的的方法：构造方法、反射、序列化、克隆、IoC、动态代理

IoC的技术实现：DI（Dependency Injection）依赖注入。底层使用的是反射机制。

### Maven项目

Maven-archetype-quickstart

**创建Spring项目：**

1. 创建maven项目

2. 加入maven依赖，spring依赖(version 5.2.5) ，juint依赖

   ```xml
   <properties>
       <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       <maven.compiler.source>1.8</maven.compiler.source>
       <maven.compiler.target>1.8</maven.compiler.target>
   </properties>
   
   <dependencies>
       <!--    juint单元测试-->
       <dependency>
           <groupId>junit</groupId>
           <artifactId>junit</artifactId>
           <version>4.11</version>
           <scope>test</scope>
       </dependency>
       <!--    spring 依赖-->
       <dependency>
           <groupId>org.springframework</groupId>
           <artifactId>spring-context</artifactId>
           <version>5.3.5</version>
       </dependency>
   </dependencies>
   ```

   

3. 创建类，创建spring的配置文件

   在`resources`目录下创建`beans.xml`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
       <!-- xmlns:p="http://www.springframework.org/schema/p" -->
   
       <!-- 使用spring来创建对象
           声明bean，告诉spring来创建某个类的对象
           id 是对象的自定义名称，spring通过名称来找到对应的对象
           class 是对象的全限定名称，不能是接口（反射）
           Spring是使用map来进行存储对象的 SpringMap.put("SomeService", new SomeServiceImpl())
       -->
   
       <bean id="SomeService" class="com.yz.spring01.services.impl.SomeServiceImpl" />
   
   </beans>
   <!-- 
   	beans是根标签，spring把java对象转为bean
   	xsd文件是约束文件
   -->
   ```

   1. 在Test中测试spring创建对象

   ```java
   @Test
   public void test02(){
       // 指定spring的配置文件
       String config = "beans.xml";
       //创建spring容器的对象，ApplicationContext表示spring容器，就可以获取对象了
       ApplicationContext ac = new ClassPathXmlApplicationContext(config);	// 执行所有bean的构造方法
       //从类路径下加载对象文件
       SomeService service = (SomeService)ac.getBean("SomeService");
       service.doSome();
   }
   ```


### Junit单元测试

1. 需要在`pom.xml`加入junit依赖：

    ```xml
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.11</version>
        <scope>test</scope>
    </dependency>
    ```

2. 在src/test/java中进行方法定义：

    ```java
    //public 没有参数、返回值，使用 @Test 进行标注
    @Test
    public void t5() {
        String conf = "ba01/applicationContext.xml";
        ApplicationContext ac = new ClassPathXmlApplicationContext(conf);
        Student s = (Student)ac.getBean("StudentBean");
        System.out.println(s);
    }
    ```

3. Run test

### 获取Spring中对象的信息

```java
@Test
public void test3(){
    String conf = "beans.xml";
    ApplicationContext ac = new ClassPathXmlApplicationContext(conf);

    int nums = ac.getBeanDefinitionCount();
    System.out.println("Beans中定义的对象数量： " + nums);

    System.out.println("容器中对象的名字为：");
    String[] names = ac.getBeanDefinitionNames();
    for (String name : names) {
        System.out.println(name);
    }
}
```

**Spring创建非自定义的对象**

比如说java自带的Date类

```java
@Test
public void Test04() {
    String conf = "beans.xml";
    ApplicationContext ac = new ClassPathXmlApplicationContext(conf);
    Date my = (Date) ac.getBean("MyDate");
    System.out.println(my);
}
```

### DI(基于XML的依赖注入)

> DI：表示创建对象，给对象属性赋值，dependency injection

1. 基于XML的DI：在spring的配置文件中使用标签和属性进行完成
2. 基于注解的DI：在spring中注解完成属性赋值

DI的分类：

* set设值注入：调用spring中的set方法
* 构造注入：创建对象在构造方法中实现注入。

<h4>基于XML</h4>
<h5>Set 注入</h5>

简单类型的xml注入：

首先定义类

```java
package com.yz.spring01.ba01;

public class Student {
    private String name;
    private int age;
    public void setName(String name) {
        this.name = name;
    }
    public void setAge(int age) {
        this.age = age;
    }
    @Override
    public String toString() {
        return "Student [age=" + age + ", name=" + name + "]";
    }  
}

```

然后再beans.xml文件中进行赋值：

原理：是调用对象实现的setter方法进行赋值。只要有set方法就能调用

```xml
<bean id="StudentBean" class="com.yz.spring01.ba01.Student">
    <property name="name" value="小萌"></property>
    <property name="age" value="18"></property>
</bean>
```

在Test中进行创建容器创建对象，输出即可。

**引用类型的赋值**

```xml
<bean id="StudentBean" class="com.yz.spring01.ba01.Student">
    <property name="name" value="小萌"></property>
    <property name="age" value="18"></property>
    <!-- 引用类型，ref设置为对应的bean id -->
    <property name="school" ref="MySchool"></property>  
</bean>

<bean id="MySchool" class="com.yz.spring01.ba01.School">
    <property name="name" value="SZU"></property>
</bean>
```

**构造注入：**

需要对应的类要实现构造方法，使用`<constructor-arg>`标签

```xml
<bean id="Student2" class="com.yz.spring01.ba01.Student">
    <constructor-arg index="0" value="萌萌"></constructor-arg>
    <constructor-arg index="1" value="18"></constructor-arg>
    <constructor-arg index="2" ref="MySchool"></constructor-arg>
</bean>
<!-- 也可以省略index属性 -->
```

**自动注入（autowire）：**

1. 引用类型自动注入-Byname：

   ```xml
   <bean id="MyStu" class="com.yz.spring01.ba03.Student" autowire="byName">
       <property name="age" value="19"/>
       <property name="name" value="萌萌子"/>
   </bean>
   
   <!-- byName 类型需与引用类型的id一致-->
   
   <bean id="school" class="com.yz.spring01.ba03.School">
       <property name="name" value="SZU[JNU]"/>
   </bean>
   ```

2. 自动注入-ByType（按类型注入）：

   保证同源（同类，父子类，接口实现类）关系

   ```xml
   <bean id="MyStu" class="com.yz.spring01.ba03.Student" autowire="byType">
       <property name="age" value="19"/>
       <property name="name" value="萌萌子"/>
   </bean>
   
   <!-- byType 类型无需对id进行一致-->
   
   <bean id="myschool" class="com.yz.spring01.ba03.School">
       <property name="name" value="SZU[JNU] == bytype"/>
   </bean>
   ```

### 多配置文件

为了防止配置文件过大，读取时间过长。

一般分为主配置文件和子配置文件。主配置文件是用来包含其他配置文件，不定义对象，用于导入其他配置文件。

```
配置文件目录
ba04
	spring-total.xml
	spring-student.xml
	spring-school.xml
```

student和schoold的xml文件只保存自己的bean，类似上述的配置。

在total主配置文件中进行配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--   classpath需要在target目录下的classes进行寻找 -->

    <import resource="classpath:ba04/spring-student.xml"/>
    <import resource="classpath:ba04/spring-school.xml"/>
    
    <!-- 可以进行包含关系进行通配符匹配 -->
    <!-- 但是主配置文件不能与这个文件名匹配，比如不可以是spring-total.xml  -->
    <import resource="classpath:ba04/spring-*.xml"/>
</beans>
```

### DI(基于注解的依赖注入)

1. 在maven中加入依赖spring-context，间接加入spring-aop的依赖
2. 在类中加入spring注解
3. 在spring配置文件中，加入组件扫描器的标签，说明注解在项目中的位置。

🔵注解的种类：

* `@Component`  `@Component(value="name")` `@Component("name")`，用在普通类的上面
* `@Repository`（用在持久层类上），放在dao层实现类上，表示创建dao对象
* `@Service`，放在service的实现类上，创建service对象
* `@Controller`，放在控制器的上面

后三种与`@Component`语法一样，但后三个还有额外的功能，用于给项目对象进行分层。

* `@Value`，用在简单类型的属性名或者是set方法上

* `@Autowired`，用在引用类型的属性上，默认为ByType方式，ByName方式需要配合`@Qualify("beanid")`

  默认`@Autowired(required=true)`，如果赋值失败会报错，如果为false，会将引用类型赋值为null。

* `@Resource`，用在引用类型的属性上，默认为ByName方式，无name会自动使用ByType

🔵添加注解：

```java
/**
 * @Component: 创建对象，相当于<bean></bean>
 * value是bean的id
 */
//@Component(value = "myStu")
//@Component      // id为类名的首字母小写
@Component("myStu")
public class Student {

    @Value("萌萌")
    private String name;
    private int age;
    @Autowired
    private School school;

    @Value("18")
    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" + "name='" + name + '\'' + ", age=" + age + ", school=" + school + '}';
    }
}

@Component("school")
class School {
    @Value("苏州大学")
    private String name;
    @Value("江苏苏州")
    private String region;

    @Override
    public String toString() {
        return "School{" + "name='" + name + '\'' + ", region='" + region + '\'' + '}';
    }
}
```

🔵设置XML扫描器：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--找到包和子包中所有的注解，按照注解创建对象，给属性赋值-->
    <context:component-scan base-package="com.yz.ba01"/>
    <!--导入多个包，用;或者,来分割多个包名-->
    <context:component-scan base-package="com.yz.ba02;com.yz.ba03,com.yz.ba04"/>
    <!--直接指定父包名，会扫描子包-->
    <context:component-scan base-package="com.yz"/>

</beans>
```

### XML和注解的对比

经常改变的使用XML，不经常该表的使用注解的方式。

注解的形式也可以使用动态修改的方法：

* 创建properties文件

  ```properties
  myname=mengmeng
  myage=18
  ```

* 在spring的配置xml文件下进行绑定

  ```xml
  <context:property-placeholder location="classpath:test.properties"/>
  ```

* 在文件中添加注解：

  使用`${myname}`的形式，可以达到xml的效果

  ```java
  @Component("myStu")
  public class Student {
  
      @Value("${myname}")
      private String name;
      private int age;
      @Autowired
      private School school;
  
      @Value("18")
      public void setAge(int age) {
          this.age = age;
      }
  
      @Override
      public String toString() {
          return "Student{" + "name='" + name + '\'' + ", age=" + age + ", school=" + school + '}';
      }
  }
  ```

### AOP面向切面编程

> aspect-oriented programming

增加功能就是切面，一般都是非业务功能。

🔵动态代理：

> 在不修改原有代码的基础上，增加功能，减少重复代码，专注于业务逻辑。

在程序执行的过程中，创建代理对象，通过代理对象的执行方法，给目标类的方法增加功能。

分为JDK动态代理和CGLIB动态代理。

AOP让开发人员使用一种统一的方法来进行动态代理。

🔵AOP的实现框架

1. spring框架中的aop，主要在事务中使用，开发中很少使用spring的aop实现，比较笨重
2. aspenctJ：一个开源做AOP的框架，spring中集成了aspectJ框架。实现方式有两种方法：XML的配置文件和使用注解的方式来进行AOP，aspectJ有5种注解。

### JDK动态代理

1. 创建目标类，比如someServiceImpl中的DoSome和DoOther添加功能
2. 创建`InvocationHandler`接口的实现类，在这个类中给目标方法添加功能
3. 使用JDK中的proxy类，创建代理对象，实现创建对象的能力。

有这样一个需求，需要在`SomeServiceImpl`中每个方法的业务上，执行之前打印运行时间，执行之后开启事务。如果在`SomeServiceImpl`中每个方法都加上这两个业务，就会显得十分冗余和麻烦。我们要使用解耦合的方法，减少开发量，达到高效开发的目的。

```java
public interface SomeService {
    void DoSome();
    void DoOther();
}

class SomeServiceImpl implements SomeService {
    @Override
    public void DoSome() {
        System.out.println("Dome service impl");
    }

    @Override
    public void DoOther() {
        System.out.println("Do other service impl");
    }
    
    // ....
}

```

这里借助proxy来完成此功能。

实现`InvocationHandler`的类：

```java
public class MyInvocationHandler implements InvocationHandler {

    private Object target;  // SomeServiceImpl类

    public MyInvocationHandler(Object target) {	// 使用构造方法来接受对应的类
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 通过代理对象执行方法的时候，会调用这个invoke
        Object res = null;
        // 在目标方法执行前的动作
        System.out.println("Time:" + new Date());
        // 执行目标类的方法，通过这个method实现
        method.invoke(target, args);
        // 在目标方法执行后的动作
        System.out.println("Status: OK.");
        // 返回目标方法的执行结果
        System.out.println(method.getName());
        return res;
    }
}
```

在执行的时候借助proxy创建实例并且执行方法`Proxy.newProxyInstance`即可：

```java
public class Main {
    public static void main(String[] args) {
        // 创建目标
        SomeService target = new SomeServiceImpl();
        // 创建InvovationHandler对象
        InvocationHandler handler = new MyInvocationHandler(target);
        // 重新生成SomeService对象，proxy创建代理
        SomeService proxy = (SomeService) Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(), handler);
        // 执行方法
        proxy.DoOther();
    }
}
```

新需求：如果只需要给`DoSome()`方法添加上面两个业务，其他的方法不需要添加。

只需要在`invoke`函数中添加一个判断方法名的语句即可：

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    // 通过代理对象执行方法的时候，会调用这个invoke
    Object res = null;
    String methodName = method.getName();
    if ("DoSome".equals(methodName)){
        // 在目标方法执行前的动作
        System.out.println("Time:" + new Date());
        // 执行目标类的方法，通过这个method实现
        method.invoke(target, args);
        // 在目标方法执行后的动作
        System.out.println("Status: OK.");
        // 返回目标方法的执行结果
    }
    return res;
}
```

### aspectJ的使用：

aspectJ实现aop有两种方式：

* XML文件配置：用来配置全局事务
* 使用注解，一般情况下都使用注解

🔵aspectj的执行时间（也叫做`advice`）：

* `@Before`
* `@AfterReturning`
* `Around`
* `AfterThrowing`
* `After`

🔵切面的执行位置，使用切入点表达式

```java
execution(modifiers? ret-type declaring-type?name(param) throws?)
// execution(访问权限， 方法返回值， 方法声明（参数） 异常类型)
```

* modifiers表示访问权限类型
* **ret-type **表示返回值类型，必需
* declaring-type 表示包名和类名
* **name(param)** 表示函数名（参数类型和个数），必需
* throws 表示抛出异常的类型
* ?表示可选部分

`*`表示通配符，`..`表示函数任意多个参数或者表示当前包和子包，`+`表示当前类接口和子类

比如：

* `execution(public * *(..))`表示任意公共方法
* `execution(* set*(..))` 表示任意一set开头的方法

🔵加入依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.5</version>
</dependency>

<!--        aspectj依赖-->
<!-- https://mvnrepository.com/artifact/org.springframework/spring-aspects -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>5.3.9</version>
</dependency>
```

🔵`@Before`的使用

方法中有`JointPoint`参数：作用是可以在通知方法中获取方法执行中的信息，例如方法名称，方法实参

```java
public interface SomeService {
    void DoSome(String name, int age);
    void DoOther();
}

class SomeServiceImpl implements SomeService {
    @Override
    public void DoSome(String name, int age) {
        System.out.println(age + "=======DoSome========" + name);
    }

    @Override
    public void DoOther() {
        System.out.println("=========DoOther========");
    }
}

```

创建切面类：

> 切面类的方法要求：1.必须是公共方法 2. 没有返回值 3. 方法可以有参数，也可以无参数

`@Aspect`用来声明这个类是切面类。

`@Before(value)`中的value使用切入点表达式，来匹配对应包下的方法。

```java
@Aspect     // 表明是切面类
public class MyAspect {
    @Before("execution(public void com.yz.service.impl.SomeServiceImpl.DoSome(..))")
    public void myBefore(){
        System.out.println("Before Methods");
    }
    
    @Before("execution(* com.yz.*..DoSome(..))")
    public void myBefore2(JoinPoint jp){
        //类别：class com.yz.service.impl.SomeServiceImpl
        System.out.println("类别："+jp.getTarget().getClass());  // 获取目标的类
        // 参数：[NIHA, 20]
        System.out.println("参数："+Arrays.toString(jp.getArgs()));
        //签名：void com.yz.service.SomeService.DoSome(String,int)
        System.out.println("签名："+ jp.getSignature());
        System.out.println("Before Methods2");
    }
}
```

在Spring的配置文件中声明这两个Bean，Spring会将所有的对象加载到内存中，通过切入点表达式来匹配切面类对应的方法，实现proxy代理。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="SomeService" class="com.yz.service.impl.SomeServiceImpl"/>
    <bean id="MyAspect" class="com.yz.service.MyAspect"/>
    <aop:aspectj-autoproxy/>
</beans>
```

测试：

```java
public class test {
    @Test
    public void test01(){
        String config = "ac.xml";
        ApplicationContext ac = new ClassPathXmlApplicationContext(config);
        // 自动创建为proxy类
        SomeService proxy = (SomeService) ac.getBean("SomeService");
        proxy.DoSome("NIHA", 20);
        proxy.DoOther();
        System.out.println(proxy.getClass().getName()); //com.sun.proxy.$Proxy8
    }
}
```

🔵`@AfterReturning(value, retValue)`的使用

`retValue`必须和通知方法的形参名相同，能够获取到目标方法的返回值。

```java
public class User {
    private String name;
    private Integer age;
    // Get Set toString Constructor omit..
}

public interface SomeService {
    User DoAfReturning();
}

public class SomeServiceImpl implements SomeService {
    @Override
    public User DoAfReturning() {
        System.out.println("=========DoAfterReturning========");
        return new User("MM", 17);
    }
}
```

Spring指定AOP：

```xml
<bean id="SomeService" class="com.yz.service.impl.SomeServiceImpl"/>
<bean id="MyAspect" class="com.yz.service.MyAspect"/>
<aop:aspectj-autoproxy/>
```

添加切面类：

这里划线处的名称必须一致。

![image-20210811185448571](https://i.loli.net/2021/08/11/knojdHGuF9EfvZA.png)

如果AfterReturning返回的结果是**引用类型**，则中途改变参数的属性值，会影响测试输出结果。

> 如果想使用JoinPoint，可以改为`public void myAfterReturning(JoinPoint jp, Object pointcut)`，其中`JoinPoint`必须为参数的第一位。

```java
@Aspect     // 表明是切面类
public class MyAspect {
    @AfterReturning(value = "execution(* com.yz.service.*..DoAfReturning())", returning = "res")
    public void myAfterReturning(Object res){
        System.out.println("得到目标方法返回值" + res);
        User u = (User) res;
        u.setAge(20);
    }
}
```

测试：

```java
@Test
public void test02(){
    String config = "ac.xml";
    ApplicationContext ac = new ClassPathXmlApplicationContext(config);
    SomeService proxy = (SomeService) ac.getBean("SomeService");
    User u = proxy.DoAfReturning();
    // 这里相当于运行的myAfterReturning(u);
    System.out.println("得到返回值"+u);
}
=========DoAfterReturning========
得到目标方法返回值User{name='MM', age=17}
得到返回值User{name='MM', age=20}  17 变 20岁了。
```

🔵`@Around(value)`环绕通知的使用

> 可以在目标方法前和后面都可以使用，能够修改目标方法的执行结果。相当于JDK动态代理
>
> 参数：ProceedingJoinPoint，父类是JoinPoint。等同于JDK动态代理的Method
>
> 功能强大，不只能修改引用类型，还能改数值类型。

```java
@Around("execution(* com.yz.service.*..DoAround())")
public Object myAround(ProceedingJoinPoint pjp) throws Throwable {

    Object o = null;
    System.out.println("Before");
    o = pjp.proceed();  // 相当 method.invoke();，并且用o接受目标方法的返回值
    System.out.println("After");
    return 11;	// 返回11
}

// Do Around 的值，返回值为0
@Override
public int DoAround() {
    System.out.println("=========DoAround========");
    return 0;
}
```

test:

```java
@Test
public void test03(){
    String config = "ac.xml";
    ApplicationContext ac = new ClassPathXmlApplicationContext(config);
    SomeService proxy = (SomeService) ac.getBean("SomeService");
    Object u = proxy.DoAround();
    System.out.println("得到返回值"+u);	// 将目标方法的返回值改为 11
}
/*
Before
=========DoAround========
After
得到返回值11
*/
```

🔵`@AfterThrowing(value, throwing)`异常通知的使用

> 参数有切入点表达式和throwing自定义的变量，在抛出异常和调用
>
> 方法参数有`JoinPoint` `Exception`

```java
@AfterThrowing(value = "execution(* com.yz.service.*..DoAround())", throwing = "ex")
public void myAfterThrowing(Exception ex){
    System.out.println("发生异常，发送邮件");
}
```

🔵`@After(value, throwing)`最终通知的使用

> 一般是做资源清除的工作，这个代码无论目标方法发生异常或者其他总会被执行，相当于`finally`。

```java
@Override
public void DoAfter() {
    System.out.println("Do After");
}
```

🔵`@Pointcut`切入点表达式：

> 项目中如果有多个相同的切入点表达式，增加复用使用此标签

使用`@Pointcut`定义在一个方法上面，此时这个方法就是切入点表达式的别名。

```java
@Pointcut("execution(* com.yz.service.*..DoAfter())")
public void mypt(){
    //无需代码
}

@After("mypt()")
public void myAfter(){
    System.out.println("Job clear.");
}
```

### 使用cgLib进行动态代理：

> 不需要编写接口类，cglib效率较高

直接对可继承的类进行代理。[CGLIB](https://www.bilibili.com/video/BV1nz4y1d7uy?p=66)

也可以在spring配置文件中直接明确指定使用CGLIB进行代理：

```xml
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

## MyBatis

[BV185411s7Ry](https://www.bilibili.com/video/BV185411s7Ry) P34

### 安装：

🔵需要安装mybatis和MySQL的驱动

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.7</version>
</dependency>

<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.26</version>
</dependency>
```

### 配置和简单查询：

> 坑：对于查询操作，对应的Model的构造函数必须含有空构造或者全构造函数，不然查询的时候会报错。

1. 首先配置MyBatis的配置文件：

   jdbc.properties配置文件：

   ```properties
   jdbc.mysql.driver=com.mysql.cj.jdbc.Driver
   jdbc.mysql.url=jdbc:mysql://localhost:3306/demo
   jdbc.mysql.username=root
   jdbc.mysql.passwd=pass
   ```

   MyBatis.xml文件

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   
   <!--读取jdbc.properties文件-->
   <properties resource="jdbc.properties"/>
   <configuration>
       <environments default="development">    <!--default表示默认选取的环境-->
           <environment id="development">   <!--环境名称-->
               <transactionManager type="JDBC"/>   <!--JDBC-->
               <dataSource type="POOLED">      <!--使用连接池-->
                   <property name="driver" value="${jdbc.mysql.driver}"/>
                   <property name="url" value="${jdbc.mysql.url}"/>
                   <property name="username" value="${jdbc.mysql.username}"/>
                   <property name="password" value="${jdbc.mysql.passwd}"/>
               </dataSource>
           </environment>
       </environments>
       <mappers>
           <!-- 映射文件 -->
           <mapper resource="com/yz/dao/MerchantDao.xml"/>
           <!--或者使用包名，全部导入包下的XML文件-->
           <package name="com.yz.dao"/>
       </mappers>
   </configuration>
   ```

   如果需要打印SQL语句，在`<configuration>`中添加：

   ```xml
   <settings>
       <setting name="logImpl" value="STDOUT_LOGGING"/>
   </settings>
   ```

2. 首先编写与数据库表中向对应的Java对象，并且设置get set方法，重写tostring：

   > 注意！这里的数值类型最好使用`Integer`进行代替。

   ```java
   public class Merchant {
       private long id;
       private String name;
       private byte age;
       private String address;
       private boolean is_credit;
       private long register_time;
       
       // get set toString方法，此处省略
   }
   ```

3. 编写一个简单的dao接口

   ```java
   public interface MerchantDao {
       // Query
       public List<Merchant> QueryMerchants();
   }
   ```

4. 在对应的MerchantDao.java的相同目录下创建MerchantDao.xml，编写一个简单的select语句

   这里的namespace是MerchantDao.java的包名类名引用路径（规范）

   select的id与dao接口对应的方法名一致，返回结果为对应的模型（规范）。

   注意：这里的XML文件需要配置POM文件中的将资源文件也拷贝到编译完的目录下才有效，<a href="#maven依赖管理">转到设置</a>。

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.yz.dao.MerchantDao">
       <select id="QueryMerchants" resultType="com.yz.model.Merchant">
           select * from merchant limit 1, 5
       </select>
   </mapper>
   ```

   

5. 测试并且运行：

   开启builder -> 创建工厂（读取配置）-> 开启会话 -> 读取数据 -> 关闭会话。

   ```java
   @Test
   public void test02() {
       // test mybatis
       String config = "mybatis.xml";
   
       InputStream instream = null;
       try {
           instream = Resources.getResourceAsStream(config);
       } catch (IOException e) {
           e.printStackTrace();
       }
       SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
       SqlSessionFactory fac = builder.build(instream);
       SqlSession session = fac.openSession();
       List<Merchant> list = null;
       try {
           list = session.selectList("com.yz.dao.MerchantDao.QueryMerchants");
       }catch (PersistenceException e){
           System.out.println("连接数据库失败");
           e.printStackTrace();
           return;
       }
       list.forEach(i -> System.out.println(i));
       session.close();
   }
   ```

### 增删改

> Mybatis默认**手动**提交事务为，因此需要最后进行手动`commit()`操作

Insert：

API:

```java
public interface MerchantDao {
    int insertMerchant(Merchant merchant);	// int返回行数
}
```

XML：

```xml
<insert id="InsertMerchant">
    insert into merchant(name, age, address, is_credit, register_time) values (#{name}, #{age}, #{address}, #{is_credit}, #{register_time})
</insert>
```

test:

```java
public void testInsert(){
    Merchant merchant = new Merchant("Mengmeng", (byte) 17, "SZU", true,(new Date().getTime()) / 1000);
    SqlSession session = getSession();
    int rows = session.insert("MerchantDao.InsertMerchant", merchant);
    System.out.println(rows);
    session.commit();
    session.close();
}
```

### 主要类的介绍：

1. Resources: Mybatis中的一个类，用于读取mybatis的配置文件

   `instream = Resources.getResourceAsStream("mybatis.xml");`

2. SqlSessionFactoryBuilder：用来创建SqlSessionFactory对象

3. SqlSessionFactory为一个接口，实现类为DefaultSqlSessionFactory，用于产生一个对话

   ```java
   fac.openSession();	// 非自动提交事务
   fac.openSession(true);	// 自动提交事务
   ```

4. SqlSession也是接口，实现类为DefaultSqlSession，定义了操作数据库的各个操作。

> SqlSession不是线程安全的

### DAO接口与DAO的XML文件的绑定

> 在之前的介绍可以看出，持久层的增删改查和DAO接口一点联系都没有

🔵方法一：

即使用一个实现类DaoImpl实现上述方法，并且使用mybatis的方式来完成上述操作。

```java
public interface MerchantDaoImpl implements MerchantDao{
    @Override
    int insertMerchant(Merchant merchant){
        SqlSession session = getSession();
        int rows = session.insert("MerchantDao.InsertMerchant", merchant);
        session.commit();
        return rows;
    }
}
```

🔵方法二：

使用Mybatis的动态代理机制：

这里getMapper，是利用XML文件中命名空间和对应标签id来定位SQL语句。如果不对应就会报错。

`MerchantDao.java`中`queryMerchants`对应类包路径是：`com.yz.dao.Merchant.queryMerchants`

则对应的命名空间为`com.yz.dao.Merchant`，对应的查询id必须是`queryMerchants`。

主要语句：`MerchantDao dao = session.getMapper(MerchantDao.class);`

```java
@Test
public void test0S(){
    SqlSession session = getSession();
    MerchantDao dao = session.getMapper(MerchantDao.class);
    List<Merchant> merchants = dao.queryMerchants();
    merchants.forEach(System.out::println);
}
```

XML：

```xml
<mapper namespace="com.yz.dao.MerchantDao">
    <select id="queryMerchants" resultType="com.yz.model.Merchant">
        select * from merchant
    </select>
</mapper>
```

### 传参

`paramType`可以不写，mybatis会通过反射自动推断数据类型。

🔵`$`和`#`占位符的区别

`${}`可以替换列名，但是不安全

`#{}`只可以替换值，不能替换列名，使用的是prepareStatement，安全。

🔵简单类型传参

Java基本类型和String都是简单类型

单参数传递里面用什么满足都行，多参数需要使用`@Param`或者`arg0` `arg1`顺序来标志参数。

API:

```java
public interface MerchantDao {
    List<Merchant> queryMerchantsByName(String name);
    List<Merchant> queryMerchantsByIdAndName(Integer id, String name);
}
```

XML

> 多参数顺序和个数可能由于业务的改变而改变

```xml
<select id="queryMerchantsByName" resultType="com.yz.model.Merchant">
    select * from merchant where name = #{qweqwe}
</select>

<select id="queryMerchantsByIdAndName" resultType="com.yz.model.Merchant">
    select * from merchant where id = #{arg0} or name like #{arg1}
</select>
```

==推荐==：多个参数也可以使用`@Param("name")`的形式，在XML中就可以标识参数了：

```java
List<Merchant> queryMerchantsByName2(@Param("name") String name);
```

```xml
<select id="queryMerchantsByName2" resultType="com.yz.model.Merchant">
    select * from merchant where name = #{name}
</select>
```

还可以使用map的key-value形式进行存储，不推荐。

🔵复杂类型传参

* 对象类型：

  Mapper中直接使用对象的属性名即可。

  ```java
  List<Merchant> queryMerchantsByName2(Merchant m);
  ```

  ```xml
  <select id="queryMerchantsByName2" resultType="com.yz.model.Merchant">
      select * from merchant where id = #{id}
  </select>
  ```

  

### 输出结果

🔵ResultType

1. 返回对象

   > 可以定义别名

   ```java
   <select id="queryMerchantsByName" resultType="com.yz.model.Merchant">
       select * from merchant where name = #{qweqwe}
   </select>
   ```

   如果很多返回都是`com.yz.model.Merchant`，就会很长。

   这里可以在mybatis的配置文件下使用`typeAlias`来进行配置别名

   mybatis.xml

   ```xml
   <typeAliases>
       <typeAlias type="com.yz.model.Merchant" alias="m"/>
   </typeAliases>
   ```

   mapper:

   ```xml
   <select id="queryId" resultType="m">
       select id from merchant
   </select>
   ```

   或者可以使用包的方式直接导入类名：

   ```xml
   <typeAliases>
       <package name="com.yz.model"/>
   </typeAliases>
   ```

   

2. 返回基本类型

   查询的结果是一列

   ```java
   <select id="queryMerchantsByName" resultType="int">
       select id from merchant where name = #{name}
   </select>
   ```

   

3. 返回map类型

   查询的结果是一行

   ```java
   <select id="queryMerchantsByName" resultType="map">
       select * from merchant where name = #{name} limit 1
   </select>
   ```

🔵ResultMap

> 当列名和属性名不一致的时候，使用ResultMap

```xml
<resultMap id="UserMap" type="com.yz.model.User">
    <!--        column是列名，property是java的属性名-->
    <id column="id" property="myid"/>
    <id column="name" property="myname"/>
</resultMap>

<select id="selectUser" resultMap="UserMap">
    select id,name from user
</select>
```

第二种方式

```xml
<select id="selectUser" resultType="com.yz.model.User">
    select id myid,name myname from user
</select>
```

### 模糊查询like

`"&"`和`#{name}`中间必须要有空格.

```xml
<select id="selectUser" resultType="com.yz.model.User">
     select id,name from user where name like "%" #{name} "%"
</select>
```

### 动态SQL

> SQL语句是变化的，根据不同的条件获取不同的SQL语句`<if> <where> <foreach>`

🔵`<if>`的使用：

```xml
<select id="queryMerchantsIf" resultType="m">
    select * from merchant where
    <if test="name != null and name != ''">
        name = #{name}
    </if>
    <if test="age > 0">
        and age = #{age}
    </if>
</select>
```

会出现`select * from merchant where and age = #{age}`的情况，出现语法错误，通常在where后面加一个`1 = 1`的恒等条件，解决此问题。

🔵`<where>`的使用：

> 使用来解决`<if>`会出现的问题。

```xml
<select id="queryMerchantsIf" resultType="m">
    select * from merchant
    <where>
        <if test="name != null and name != ''">
            name = #{name}
        </if>
        <if test="age > 0">
            and age = #{age}
        </if>
    </where>
</select>
```

使用where标签，出现if中的问题时候，会自动去除or和and

🔵`<foreach>`的使用：

> 用于循环java中的数组和集合，主要用在in语句中

当元素为简单类型时：

```java
List<Merchant> queryMerchantsFor(List<Integer> list);
```

```xml
<select id="queryMerchantsFor" resultType="m">
    select * from merchant where id in 
    <foreach collection="list" item="id" open="(" close=")" separator=",">
        #{id}
    </foreach>
</select>
```

当元素为对象时：

直接使用`.`指定对应的属性字段

```java
List<Merchant> queryMerchantsFor(List<User> list);
```

```xml
<select id="queryMerchantsFor" resultType="m">
    select * from merchant where id in ()
    <foreach collection="list" item="user" separator=",">
        #{user.id}
    </foreach>
    )
</select>
```



collection：表示数组或者集合的类型，list或者array

item：表示循环变量的id，用在下面

open：表示开始符号`(`

close：表示结束符号`)`

separator：表示分割符号：`,`

🔵代码片段复用：

> 增加程序的复用性

```xml
<sql id="selectMer">
    select * from merchant
</sql>

<select id="demo" resultType="m">
    <include refid="selectMer" /> where id = 1
</select>
```

## Spring集成MyBatis

> 集成像一个框架一样，原理ioc。会使用独立的连接池，代替mybatis的连接池。

注意：这个整合包的事务是自动提交的。

要让spring创建的对象：

1. 独立的连接池对象，使用阿里的druid连接池。
2. SqlSessionFactory对象
3. 创建dao对象

### 添加依赖：

添加依赖：spring，mybatis，mysql，spring事务，spring+mybatis集成依赖

```xml
<dependencies>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.11</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.9</version>
    </dependency>

    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.7</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.26</version>
    </dependency>

    <!--spring事务依赖-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>
        <version>5.3.9</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.3.9</version>
    </dependency>

    <!--mybatis-spring集成依赖-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.6</version>
    </dependency>
    <!--用于创建连接池-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.2.6</version>
    </dependency>
</dependencies>

<build>
    <!--将源代码目录下的其他资源文件也编译到输出文件中-->
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```

### 创建模型实体类：

```java
public class Merchant {
    private Integer id;
    private String name;
    private Integer age;
    private String address;
    private boolean is_credit;
    private Integer register_time;
    public Merchant(){}
    // getters and setters
}
```

### 构建mybatis配置：

dao类：

```java
public interface MerchantDao {
    int insertMerchant(Merchant m);
    List<Merchant> queryMerchants(Merchant m);
}
```

dao的xml文件：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.yz.dao.MerchantDao">
    <insert id="insertMerchant">
        insert into merchant(name, age, address, is_credit, register_time) VALUES (#{id}, #{age}, #{address}, #{is_credit}, #{register_time})
    </insert>

    <select id="queryMerchants" resultType="Merchant">
        select * from merchant
        <where>
            <if test="id > 0">
                id = ${id}
            </if>
            <if test="name != null and name != ''">
                or name = #{name}
            </if>
        </where>
    </select>
</mapper>
```

mybatis配置：

> mybatis中的数据源配置迁移到spring中去配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>

    <typeAliases>
        <package name="com.yz.model"/>
    </typeAliases>

    <mappers>
        <package name="com.yz.dao"/>
    </mappers>
</configuration>
```

### 创建service

> 使用service也是通过spring的ioc构建，因此需要编写setters方法

```java
public interface MerchantService {
    int addMerchant(Merchant m);
    List<Merchant> queryMerchants(Merchant m);
}

class MerchantServiceImpl implements MerchantService {
    private MerchantDao dao;

    public void setDao(MerchantDao dao) {this.dao = dao;}

    @Override
    public int addMerchant(Merchant m) {
        return dao.insertMerchant(m);
    }

    @Override
    public List<Merchant> queryMerchants(Merchant m) {
        return dao.queryMerchants(m);
    }
}
```

### Spring配置：

properties配置

```properties
jdbc.mysql.driver=com.mysql.cj.jdbc.Driver
jdbc.mysql.url=jdbc:mysql://localhost:49154/demo
jdbc.mysql.username=root
jdbc.mysql.passwd=785611814
jdbc.mysql.maxActive=20
```

spring的配置：

1. 声明properties的配置文件
2. 声明数据源
3. 声明创建SqlSessionFactory
4. 声明dao对象
5. 声明service对象

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:property-placeholder location="classpath:jdbc.properties"/>
    <!--声明数据源-->
    <bean id="dsn" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <property name="url" value="${jdbc.mysql.url}"/>
        <property name="username" value="${jdbc.mysql.username}"/>
        <property name="password" value="${jdbc.mysql.passwd}"/>
        <property name="maxActive" value="${jdbc.mysql.maxActive}"/>
    </bean>

    <!--创建SqlSessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--set注入-->
        <property name="dataSource" ref="dsn"/>
        <!--configLocations是Resource类型，读取配置文件中的mapper-->
        <property name="configLocation" value="classpath:mybatis.xml"/>
    </bean>

    <!--创建dao对象，使用sqlsession的getMapper，MapperScannerConfigurer在内部自动调用getMapper创建每个dao的对象-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--指定创建dao的sqlsession-->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <!--指定dao包名，每个dao类都执行getMapper方法-->
        <!--生成的dao对象名是接口名的首字母小写UserDao.java -> userDao -->
        <property name="basePackage" value="com.yz.dao"/>
    </bean>

    <!--声明service-->
    <bean id="merchantService" class="com.yz.service.impl.MerchantServiceImpl">
        <property name="dao" ref="merchantDao"/>
    </bean>
</beans>
```

### Spring事务处理

> Spring的事务处理要放在service层，因为业务的方法会调用执行多个sql语句。

Spring中有一个事务管理器`PlatformTransactionManager`，定义了commit，rollback等方法。

mybatis对应的实现类为：`DataSourceTransactionManager`，Hibernate对应的是`HibernateTransactionManager`.

🔵如何说明事务的类型

* 隔离级别，4个值。`TransactionDefinition`中定义。
* 事务的超时时间。超过多少时间就回滚。
* 事务的传播行为，7种行为。`PROPAGATION_XXX`
  * `PROPAGATION_REQUIRED`，指定方法必须在事务内执行。若当前存在事务，就加入当前事务。（默认）
  * `PROPAGATION_SUPPORTS`，支持当前事务，如果不在事务中，也可以以非事务的方式执行。（查询语句）
  * `PROPAGATION_REQUIRES_NEW`，总是新创建一个事务，若存在事务则挂起，等新事务完成后恢复。

🔵提交事务，回滚事务的时期：

* 无异常时，自动commit
* ❗**存在**
* **`RuntimeException`的时候，会rollback**

🔵事务处理方案：

* 适用于中小项目的，注解方案

  spring框架使用aop来给事务增加功能，使用`@Transacational`注解增加事务，放在public的方法上面。

  🟣可选属性：

  * `propagation`，传播方式，默认`Propagation.REQUIRED`
  * `isolation`，隔离级别，默认为`Isolation.DEFAULT`，枚举类型
  * `readOnly`，是否只读（只使用查询），布尔值，默认为`false`
  * `timeout`，超时时间，默认-1
  * `rollbackFor`，指定需要回滚的异常类，类型`Class[]`，默认空数组，如果不是`RuntimeException`也会回滚。
  * `rollbackForClassName`，指定需要回滚的异常类类名，类型`String[]`，默认空数组。
  * `noRollbackFor`  `noRollbackForClassName`

  🟣使用步骤：

  * 声明事务管理器对象，开启事务注解驱动：

    ```xml
    <!--声明数据源-->
    <context:property-placeholder location="classpath:jdbc.properties"/>
    <bean id="dsn" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <property name="url" value="${jdbc.mysql.url}"/>
        <property name="username" value="${jdbc.mysql.username}"/>
        <property name="password" value="${jdbc.mysql.passwd}"/>
        <property name="maxActive" value="${jdbc.mysql.maxActive}"/>
    </bean>
    
    <!--声明事务管理器对象 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--指定连接的数据库，数据源-->
        <property name="dataSource" ref="dsn"/>
    </bean>
    
    <!--开启事务注解驱动，设置spring使用注解管理事务-->
    <tx:annotation-driven transaction-manager="transactionManager"/>
    ```

  * 加入注解

    ```java
    @Transactional(
        propagation = Propagation.REQUIRED,
        isolation = Isolation.DEFAULT,
        readOnly = false,
        rollbackFor = {RuntimeException.class}
    )
    @Override
    public int addMerchant(Merchant m) {
        if(m.getId() < 0){
            throw new RuntimeException("id < 0 exception");
        }
        return dao.insertMerchant(m);
    }
    ```

    

* 适合大型项目，有很多的类和方法，需要大量的配置事务，使用aspectj（==推荐==）

  🟣实现步骤：在XML文件中使用。

  * 加入aspectj依赖

  * 声明事务管理器对象

  * 声明方法需要的事务类型

    ```xml
    <tx:advice id="myAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <!--给具体的方法配置属性，name可以使用通配符-->
            <tx:method name="addMerchant" isolation="DEFAULT" propagation="REQUIRED" rollback-for="java.lang.RuntimeException"/>
        </tx:attributes>
    </tx:advice>
    
    <aop:config>
        <!--指定事务配置应用什么包和类中-->
        <aop:pointcut id="pc1" expression="execution(* *..service..*.*(..))"/>
        <!--关联point和advice-->
        <aop:advisor advice-ref="myAdvice" pointcut-ref="pc1"/>
    </aop:config>
    ```

    

  * 配置AOP

## SpringMVC

[BV1Ry4y1574R](https://www.bilibili.com/video/BV1Ry4y1574R) P80

### 配置和依赖

maven依赖：

```xml
<dependencies>
    <!--SpringMVC-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.2.16.RELEASE</version>
    </dependency>

    <!--日志-->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.5</version>
    </dependency>

    <!--Servlet API-->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>provided</scope>
    </dependency>

    <!--thymeleaf-->
    <dependency>
        <groupId>org.thymeleaf</groupId>
        <artifactId>thymeleaf-spring5</artifactId>
        <version>3.0.12.RELEASE</version>
    </dependency>

</dependencies>
```

### Web.xml文件配置

> web.xml文件默认在项目`webapp\WEB-INF`目录下，用于servlet配置

🔵默认配置方式（==不推荐==）

> 会将`xxx-servlet.xml`的配置文件放在WEB-INF目录下，而非resources目录下

```xml
<servlet>
    <servlet-name>SpringMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>SpringMVC</servlet-name>
    <!--设置springmvc核心控制器能处理的请求路径，不会匹配.jsp的请求-->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

🔵扩展配置方式（==推荐==）

```xml
<servlet>
    <servlet-name>SpringMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--配置spring的位置和文件名-->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc-servlet.xml</param-value>
    </init-param>
    <!--将前端控制器DispatcherServlet的初始化时间提前到服务器启动时-->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>SpringMVC</servlet-name>
    <!--设置springmvc核心控制器能处理的请求路径，不会匹配.jsp的请求-->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

### 编写简单的helloworld

🔵编写controller（POJO）

```java
@Controller
public class HelloController {
    @RequestMapping( "/")		// 设置访问路径
    public String index(){
        return "index";		// 返回html除去后缀名的文件名
    }
}
```

🔵spring配置：

> 视图模板即在`/WEB-INF/templates/`创建对应的html文件，然后根据`@RequestMapping`的目标方法返回对应的html文件名即可。

```xml
<!--配置注解扫描器-->
<context:component-scan base-package="com.yz"/>

<!--配置thymeleaf-->
<bean id="viewResolver" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
    <!--设置视图解析器的优先级-->
    <property name="order" value="1"/>
    <!--编码-->
    <property name="characterEncoding" value="UTF-8"/>

    <property name="templateEngine">
        <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
            <property name="templateResolver">
                <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
                    <!--视图前缀-->
                    <property name="prefix" value="/WEB-INF/templates/"/>
                    <!--视图后缀-->
                    <property name="suffix" value=".html"/>
                    <property name="templateMode" value="HTML"/>
                    <property name="characterEncoding" value="UTF-8"/>
                </bean>
            </property>
        </bean>
    </property>
</bean>
```

🔵自动添加上下文路径

java:

```java
@Controller
public class HelloController {

    @RequestMapping( "/")
    public String index(){
        return "index";
    }

    @RequestMapping( "/target")
    public String target(){
        return "target";
    }
}
```

html:

目标url为`localhost:8080/Springmvc/target`

```html
<a th:href="@{/target}">Go</a>	<!--自动添加/Springmvc/-->
```

### @RequestMapping注解

🔵标识类路径：

```java
@Controller
@RequestMapping("/api")
public class APIController {
    //http://localhost:8080/Springmvc/api/1
    @RequestMapping("1")
    public String a1(){return "api1";}
    //http://localhost:8080/Springmvc/api/2
    @RequestMapping("2")
    public String a2(){return "api2";}
}
```

🔵value属性`String[]`

```java
@Controller
public class APIController {
    //http://localhost:8080/Springmvc/api
    //http://localhost:8080/Springmvc/api2
    @RequestMapping(value={"api", "api2"})
    public String a1(){return "api";}
}
```

🔵method属性`enum[]`

> 默认情况下，get，post访问都可以，如果不匹配则返回405

```java
@Controller
public class APIController {
    @RequestMapping(value = "1", method = RequestMethod.GET)	//指定GET
    public String a1() {
        return "api1";
    }
	
    @RequestMapping(value = "2", method = {RequestMethod.GET, RequestMethod.POST})
    public String a2() {			// GET POSt都可以
        return "api2";
    }
}
```

也可以指定GET方法：

```java
@GetMapping("api2")
public String a2() {
    return "api2";
}
```

类似的有`@PostMapping()` `@DeleteMapping` `@PutMapping`

🔵params属性`String[]`

>指定参数满足四种类型`"item"`, `"!item"` `"item=a"` `"item!=b"`，不满足条件返回400，条件需同时满足。

```java
@GetMapping(value = "1", params = {"name", "!age", "dog=true", "cat!=false"})
public String a1() {
    return "api1";
}
```

name项必须存在，age项必须不能存在，dog项必须为true，cat项不能为false

thymeleaf的传参方式：

```html
<a th:href="@{/api(name='nick', age='15')}"></a>
```

🔵headers属性`String[]`

> 不满足返回404

用法类似`params`

```java
@GetMapping(value = "1", headers = {"User-Agent=Chrome"})
public String a1() {
    return "api1";
}
```

🔵RESTful API形式

```java
@GetMapping("/api2/{id}/{name}")
public String a2(@PathVariable("id") String id, @PathVariable("name") String name){
    System.out.println(id + name);
    return "api2";
}
```

### 获取请求体参数

🔵通过ServletAPI获取参数（==不推荐==）

```java
@GetMapping("API")
public String a3(HttpServletRequest req){
    String name = req.getParameter("name");
    return "api1";
}
```

🔵HTTP请求参数和变量名一致：

```java
@GetMapping("API")
public String a3(String name, int[] age){	// http://host/name?=a
    System.out.println(name);	// a
    System.out.println(Arrays.toString(age));
    return "api1";
}
```

网址：`http://host/?name=a`，输出`a`

网址：`http://host/?name=a&name=b&age=8&age=9`，输出`a,b  [8,9]`

🔵使用`@RequestParam("name")`

> 可以指定可选参数和默认参数，默认为必选参数

```java
@GetMapping("API")
public String a3(@RequestParam("Name", defaultValue="Jack") String name, @RequestParam(required = false) int[] age){	// http://host/name?=a
    System.out.println(name);	// a
    System.out.println(Arrays.toString(age));
    return "api1";
}
```

🔵使用实体类接受参数

需要请求参数名和POJO对象名一致。

```java
class User{
    String name;
    Integer age;
}

@GetMapping("API")
public String a3(User u){	// http://host/name?=a
    System.out.println(u.name);	// a
    System.out.println(Arrays.toString(u.age));
    return "api1";
}
```

### 获取请求头

同样拥有value，required，defaultValue三个属性

`@RequestHeader`   `@CookieValue`分别用来获取请求头参数和cookie值。

```java
1 @RequestMapping("/testCookie")
2 public String testCookie(@CookieValue(value="name",required=false) String name,
3         @CookieValue(value="age",required=false) Integer age){
4     System.out.println(name+","+age);
5     return "hello";
6 }
```

### 编码问题

GET请求编码需要配置tomcat，`URIEncoding`属性

使用javaweb的过滤器。

```xml
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <!--设置编码-->
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <!--是否强制响应编码-->
    <init-param>
        <param-name>forceResponseEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### 域对象共享数据

🔵使用Servlet向域中传递对象

```java
@RequestMapping( "/target")
public String target(HttpServletRequest req){
    req.setAttribute("Name", "Jack");
    return "target";
}
```

html：

```html
<p th:text="${Name}"></p>
```

🔵使用ModelAndView传递对象

```java
@RequestMapping( "/target")
public ModelAndView mav1(){
    ModelAndView mav = new ModelAndView();
    mav.addObject("name", "Lisa");
    mav.setViewName("index");       // index.html
    return mav;
}
```

🔵使用model传递对象

```java
@RequestMapping( "/target")
public String target(Model model){
    model.setAttribute("Name", "Jack");
    return "target";
}
```

🔵使用map传递对象

```java
@RequestMapping( "/target")
public String target(Map<String,Object> m){
    m.setAttribute("Name", "Jack");
    return "target";
}
```

🔵使用ModelMap传递对象

```java
@RequestMapping( "/target")
public String target(ModelMap m){
    m.setAttribute("Name", "Jack");
    return "target";
}
```

🔵Model, ModelMap, Map的关系

三者在SpringMVC请求域底层实现的类都是`BindingAwareModelMap`

🔵向Session中共享数据

```java
@RequestMapping( "/target")
public String target(HttpSession session){
    session.setAttribute("Name", "Jack");
    return "target";
}
```

🔵向application中共享数据

```java
@RequestMapping( "/target")
public String target(HttpSession session){
    ServletContext app = session.getServletContext();
    app.setAttribute("name", "Jack");
    return "target";
}
```

### SpringMVC视图

分为转发视图和重定向视图

🔵转发：

不需要解析两次。

```java
@RequestMapping( "/a")
public String target(){
    return "index";
}

@RequestMapping( "/b")
public String target(){
    return "forward:/a";
}
```

🔵重定向：

```java
@RequestMapping( "/a")
public String target(){
    return "index";
}

@RequestMapping( "/b")
public String target(){
    return "redirect:/a";
}
```

🔵视图控制器：

```xml
<mvc:view-controller path="/" view-name="index"/>
<!--开启mvc解析静态资源访问 -->
<mvc:default-servlet-handler/>
<!--开启mvc注解驱动 -->
<mvc:annotation-driven/>
```

这句话相当于：

```java
@RequestMapping( "/")
public String index(@RequestHeader("Host") String host){
    return "index";
}
```

### RESTfulAPI

🔵获取参数

```java
@GetMapping("/user/{id}")
public String b(@PathVariable("id") String id){
    return id;
}
```

### 报文信息转换器——HttpMessageConverter

`@RequestBody`  `@RequestEntity` `@ResponseBody` `@ResponseEntity`（后两个常用）

body是请求体，Entity是整体请求报文（头+体）

`@RequestBody`：获取的信息类似`name=jack&age=20`

`@RequestEntity` ：类型`RequestEntity<String>`，req.getHeaders(), req.getBody()

`@ResponseBody`：

返回一个字符串

```java
@RequestMapping( "/a")
@ResponseBody
public String target(){
    return "good, body";
}
```

返回一个对象：

> 导入jackson的json数据绑定jar包

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.4</version>
</dependency>
```

再加入mvc的注解驱动：`<mvc:annotation-driven/>`

java

```java
@RequestMapping( "/u")
@ResponseBody
public User u(){
    return new User("John", 66);
}
```

也可以返回数组：

```java
@RequestMapping( "/u")
@ResponseBody
public int[] u(){
    int [] arr = new int[]{1,1,3,5,7,9,12321,4};
    return arr;
}
```

`@ResponseEntity`：

一般用于控制器方法的返回值类型。其返回值即是返回到浏览器的响应报文。

🔵`@RestController`

`@RestController = @Controller + @ResponseBody`

为类中的每个方法加`@ResponseBody`注解。

### 文件上传

上传文件必须是POST请求，并且需要导入`commons-fileupload`依赖

```xml
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.4</version>
</dependency>
```

编写html上传器：

```html
<form th:href="@{/api/upload}" method="post" enctype="multipart/form-data">
    File: <input type="file" name="file" id=""/> <br>
    <input type="submit" value="Go">
</form>
```

编写java代码：

> SpringMVC为文件上传提供了一个`MultipartFile`用来专门解决文件的问题，但是不能直接将二进制转为Java对象，需要springmvc配置上传文件解析器

```java
@PostMapping("/upload")
@ResponseBody
public String upload(MultipartFile file, HttpSession session) throws IOException {
    String filename = file.getOriginalFilename();
    System.out.println("收到"+filename);
    ServletContext context = session.getServletContext();
    String realPath = context.getRealPath("img");
    File file1 = new File(realPath);
    if (!file1.exists()){	// 目录是否存在
        file1.mkdir();
    }
    String finalPath = realPath + File.separator + UUID.randomUUID().toString();
    file.transferTo(new File(finalPath));
    System.out.println("上传完毕" + finalPath);
    return "OK";
}
```

🔵配置上传文件解析器

这里的**id**必须叫`multipartResolver`

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"></bean>
```



## Springboot简介

核心技术和响应式编程

[BV19K4y1L7MT](https://www.bilibili.com/video/BV19K4y1L7MT)

> 简化Spring全家桶的脚手架

### 安装和配置：

maven的pom配置：

这里的父依赖包含了子依赖所有的信息，如果引入子依赖的时候就不需要进行指定版本号。

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.3</version>
</parent>

<dependencies>
    <!--导入web开发依赖-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <!--不需要指定版本号-->
    </dependency>
</dependencies>
```

编写主程序入口：

```java
// @SpringBootApplication 标志这是一个主程序类
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

编写一个网页controller

```java
@RestController
public class HelloController {
    @GetMapping("/")
    public String a(){
        return "Hello SpringBoot";
    }
}
```

开始的时候直接运行main函数即可

### 部署项目

在pom下加入打包jar的依赖

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

### 自动扫描注解

> 如果程序是在主程序同包或者子包下，就会自动扫描注解

如果在包外的程序也想使用注解，必须要在主程序指定包注解解析的范围：

```java
@SpringBootApplication(scanBasePackages = "com.yz")
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

### 底层注解：

🔵`@Configuration`注解

属性：`proxyBeanMethods`，默认为true。如果是引用类型的依赖，会自动检查容器中是否存在，存在则不再创建，类似单例模式；如果为false，不会去检查，直接重新创建一个对象，加载较快。

用于创建类似spring中的bean

```java
@Configuration
public class UserConfig {	// 配置类也是组件，代理对象
    @Bean
    public User User01(){
        return new User("Jack", 18);
    }
}
```

测试：

```java
@SpringBootApplication()
public class App {
    public static void main(String[] args) {
        //返回IOC容器
        ApplicationContext run = SpringApplication.run(App.class, args);
        User u = (User) run.getBean("User01");
        System.out.println(u);	// User{name='Jack', age=18}
    }
}
```

🔵`@Import`注解

> 会自动创建对应类的对象到IOC容器中，Bean名即为包类全限定名称

```java
@Import({User.class})
@SpringBootApplication()
public class App {
    public static void main(String[] args) {
        ApplicationContext run = SpringApplication.run(App.class, args);
        User u = (User) run.getBean("com.yz.bean.User");
        System.out.println(u);
    }
}
```

🔵`@Conditional`注解

> 满足条件时候，加入组件注入

`@ConditionalOnBean(name="xx")`：容器中拥有xx对象的时候才引入组件

<img src="https://i.loli.net/2021/08/17/XCjJLBQvHcG49un.png" alt="image-20210817022254960" style="zoom: 67%;" />

 🔵`@ImportResource`注解

> 从Spring的xml文件中导入组件，适用于新旧工程迁移

xml:

```xml
<bean id="user" class="com.yz.bean.User">
    <property name="name" value="小萌"></property>
    <property name="age" value="18"></property>
</bean>
```

java

```java
@ImportResource("classpath:spring.xml")
@SpringBootApplication(scanBasePackages = "com.yz")
public class App {
    public static void main(String[] args) {
        ApplicationContext run = SpringApplication.run(App.class, args);
        User u = (User) run.getBean("user");
        System.out.println(u);
    }
}
```

 🔵`@ConfigurationProperties`

> 从属性文件中导入信息到IOC容器中

首先需要在springboot的指定配置文件中加入信息：

```properties
user01.name=Jack
user01.age=20
```

方法一：`@Component` + `@ConfigurationProperties`

需要使用prefix指定前缀

```java
@Component
@ConfigurationProperties(prefix = "user01")
public class User {
    private String name;
    private Integer age;
	 // getters and setter
}
```

方法二：`@EnableConfigurationProperties` +  `@ConfigurationProperties`

> 需要在配置类(`@Configuration`)文件中使用

```java
@Configuration
@EnableConfigurationProperties({User.class})
public class UserConfig {

}
```
User类中：
```java
@ConfigurationProperties(prefix = "user01")
public class User {
    private String name;
    private Integer age;
	 // getters and setter
}
```

使用：

需要使用`@Autowired`注解

```java
@RestController
public class HelloController {
    @Autowired
    User user;

    @GetMapping("/user")
    public User b(){
        return user;
    }
}

```

### 自动配置原理

🔵`@SpringBootApplication`原理

```java
@SpringBootConfiguration	// 相当于@Configuration
@EnableAutoConfiguration
// 扫描包
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {/*...*/}
```

1. 利用Registerar来导入组件：

```java
public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
    AutoConfigurationPackages.register(registry, (String[])(new AutoConfigurationPackages.PackageImports(metadata)).getPackageNames().toArray(new String[0]));
}
```

2. 根据`META-INFO/spring.factories`来引入组件，但是其最终根据`@Conditional`来进行按需装配引入的。
3. 用户有自己配置的话，以用户为主。

可以在springboot的配置文件中加入`debug=true`来查看有哪些组件自动装配了。positive为生效，negative为不生效的。

### 开发小技巧

🔵Lombok

> 简化JavaBean开发，toString，getters和setters

倒是也没必要用

🔵dev-tools

倒是也没必要用

🔵Spring Initializr

<img src="E:\Notes\Java\2021Java\Java笔记.assets\image-20210817033843945.png" alt="image-20210817033843945" style="zoom: 80%;" />



### 单元测试Junit5

引入依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

测试：

```java
@SpringBootTest
class SpringDaoApplicationTests {
    @Autowired
    StringRedisTemplate redisTemplate;
    @Test
    void testRedis(){
        ValueOperations<String, String> stringValueOp = redisTemplate.opsForValue();
        stringValueOp.set("mykey", "vlure", 30);
    }

}
```

`@Test`是junit-api中的

🔵常用注解：

参考：[JUnit 5 Anotation](https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations)

* `@DisplayName`用于标注测试名

  ```java
  @DisplayName("Junit5 Test Class")
  public class Junit5Test {
  
      @DisplayName("Junit5 Test01")
      @Test
      public void test01(){
          System.out.println("test");
      }
  }
  ```

  

* `@BeforeEach`每个测试方法前都要运行

  ```java
  @BeforeEach
  public void before(){
      System.out.println("Before test preparation");
  }
  ```

  `@AfterEach`同理类似，`@BeforeAll`是在整个测试类运行之前，`@AfterAll`类似

* `@Disable`，用于禁用某个测试方法

* `@Timeout(value=5, unit = TimeUnit.Second)`

  测试超时则抛出异常：

  ```java
  @Timeout(value = 1,  unit = TimeUnit.SECONDS)
  @Test
  public void aVoid() throws InterruptedException {
      Thread.sleep(1100);
  }
  ```

  

* `@RepeatedTest(n)`，重复n次测试

  ```java
  @RepeatedTest(5)
  @Test
  public void c(){
  System.out.println(6);
  }
  ```

🔵断言机制

参考：[Assertions (JUnit 5.7.2 API)](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/Assertions.html)

检查业务逻辑是否正确：

```java
@Test
public void d(){
    int a = 2 + 3;
    Assertions.assertEquals(a, 5);
}
```

还有测试数组是否相等`assertArrayEquals`。

```java
@Test
public void e(){
    assertAll("test", ()-> assertTrue(true),
                 ()-> assertEquals(5, 5));
}
```

assertAll：全部成功才能成功。

`fail(msg)`：直接测试失败。

🔵前置机制（Assumption）

类似Assertion

🔵参数化测试`@ParameterizedTest`

参考：[JUnit 参数化测试](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests)

轮流输入1，2，3，4，5

```java
@ParameterizedTest
@ValueSource(ints = {1,2,3,4,5})
public void f(int i){
    System.out.println(i);
}
```

### 指标监控（Actuator）



## SpringBoot——Web开发

[BV19K4y1L7MT](https://www.bilibili.com/video/BV19K4y1L7MT?p=22) P77

### 简单功能设置

参考：[developing-web-applications](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc)

🔵静态资源访问

静态资源目录默认名`/static` (or `/public` or `/resources` or `/META-INF/resources`)

访问路径为：根目录+`/`+资源名称。

设置静态资源的前缀（以`/res`为前缀）：

```yaml
spring:
  mvc:
    static-path-pattern: /res/**	# http://localhost:8080/res/...
  web:
    resources:
      static-locations: [classpath:/static/]	# 指定静态资源的目录
```

也可以使用webjars

🔵欢迎页配置

springboot会自动寻找`index.html`文件和`favori.ico`。

但是如果配置`static-path-pattern`，会导致图标寻找自动失效。

🔵请求参数处理

开启浏览器的put，delete请求：

```yaml
spring:
  mvc:
    hiddenmethod:
      filter:
        enabled: true
```

获取参数的方式同SpringMVC  <a href="#获取请求体参数">Go</a>

### 配置拦截器

🔵首先要做一个拦截器并且继承`HandlerInterceptor`，按照需求重写其中的`preHandle` `postHandle` `afterCompletion`的方法。

```java
@Slf4j
public class LoginInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse rsp, Object handler) throws Exception {
        System.out.println("Interceptor - PreHandler");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest req, HttpServletResponse rsp, Object handler, ModelAndView mav) throws Exception {
        System.out.println("Interceptor - postHandler");

    }

    @Override
    public void afterCompletion(HttpServletRequest req, HttpServletResponse rsp, Object handler, Exception ex) throws Exception {
        System.out.println("Interceptor - afterCompletion");
    }
}
```

🔵然后将拦截器配置对应的生效路径：

> 注意这里静态资源也会拦截，前后端分离另说。

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/**")                 // 添加拦截规则
                .excludePathPatterns("/", "/setsession");    // 放行路径
    }
}
```

🔵测试：

```java
@GetMapping("getsession")
public String getSession(@CookieValue("age") String m){
    System.out.println("Controller");
    return m;
}
```

输出：

```
Interceptor - PreHandler
Controller
Interceptor - postHandler
Interceptor - afterCompletion
```

### 文件上传配置

类似<a href="#文件上传">SpringMVC文件上传</a>

单文件使用`MultipartFile`，多文件`MultipartFile[]`

```properties
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=100MB
```

用于设置单个文件上传大小和请求的总共文件大小。

### 异常处理

处理异常的文件夹`/error`放在`resources/templates/`或者`resources/public/`目录下，SpringB会自动解析`4xx.html`和`5xx.html`

### 使用原生servlet，filter

> 一种是使用`@WebServlet`等注解，另一种使用`@RegistrationBean`

🔵原生servlet：

> 但是未经过Springboot的拦截器

使用`@ServletComponentScan`指定servlet路径

```java
@ServletComponentScan(basePackages = "com.yz.servlet")
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        ApplicationContext run = SpringApplication.run(App.class, args);
    }
}
```

编写servlet继承HttpSevlet

```java
@WebServlet(urlPatterns = "/demoS")
public class DemoServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("my servlet");
    }
}
```

🔵原生filter：

```java
@WebFilter(urlPatterns = {"/*"})
public class DemoFilter extends HttpFilter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse rsp, FilterChain chain) throws IOException, ServletException {
        System.out.println("filter do");
        chain.doFilter(req, rsp);
    }

    @Override
    public void init() throws ServletException {
        System.out.println("filter init");
        super.init();
    }

    @Override
    public void destroy() {
        System.out.println("filter destroy");
        super.destroy();
    }
}
```

🟣使用`RegistrationBean`：

> 这样就可以忽略`@WebFilter` `@WebServlet`

```java
@Configuration
public class FilterServletConfig {

    @Bean
    public ServletRegistrationBean myServlet(){
        DemoServlet demoServlet = new DemoServlet();
        return new ServletRegistrationBean(demoServlet, "/my", "/ok");
    }
    
    @Bean
    public FilterRegistrationBean myfilter(){
        DemoFilter demoFilter = new DemoFilter();
        FilterRegistrationBean bean = new FilterRegistrationBean(demoFilter);
        bean.addUrlPatterns("/*");
        return bean;
    }
    
}
```

### 数据访问

🔵导入依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>

<!--MySQL驱动-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>

<!--使用druid starter-->
<!-- https://mvnrepository.com/artifact/com.alibaba/druid-spring-boot-starter -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.6</version>
</dependency>
```

🔵数据源配置：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/demo
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
```

🔵Druid的配置：

参考：[druid-spring-boot-starter](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter)

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:49154/demo
    username: root
    password: 785611814
    druid:
      stat-view-servlet:    # 监控页的配置
        enabled: true
        login-username: admin
        login-password: asdQWE123
        reset-enable: false
      web-stat-filter:
        enabled: true
        url-pattern: /*
        exclusions: '*.js, *.css, *.jpg, *.gif, *.ico, /druid/*'
      filters: 'stat,wall'    # 监控和防火墙
      filter:
        stat:
          log-slow-sql: true    # 开启慢查询记录
          slow-sql-millis: 1000 # 1秒
          enabled: true
        wall:
          enabled: true
          config:
            drop-table-allow: false   # 禁止删除表
```

🔵整合Mybatis进行数据操控

引入starter

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.0</version>
</dependency>
```

配置springboot中的mybatis配置文件：

```yaml
mybatis:
  configuration:
    map-underscore-to-camel-case: true		# 是否开启驼峰命名法
  mapper-locations: classpath:mappers/*.xml		# mapper目录
  type-aliases-package: com.yz.springdao.model	# 别名设置
```

可以创建mybatis全局文件（可选）和mapper文件（可选）

创建对应接口：

> 对应的接口需要使用`@Mapper`进行标注，也可以使用`@MapperScan`，简单的SQL语句可以直接使用注解的形式比如`@Select` `@Update`等，支持混合模式开发

```java
@Mapper
public interface UserDao {
    public User selectUserId(Integer id);

    @Select("select * from merchant where name = #{name}")
    public User selectUserName(String name);
}
```

对应的Mapper文件：

```xml
<mapper namespace="com.yz.springdao.dao.UserDao">
    <select id="selectUserId" resultType="User">
        select * from merchant where id = #{id}
    </select>
</mapper>
```

### 整合Mybatis-plus

❗需要学习一下Mybatis-plus

> 可以在依赖中省略mybatis的依赖了，只需要继承BaseMapper就可以CURD了

需要在Springboot应用上添加注解`MapperScan`，扫描mapper的路径：

```java
@SpringBootApplication
@MapperScan("asia.qaqaqqa.blogbackend.mapper")
public class BlogBackendApplication {
    public static void main(String[] args) {
        SpringApplication.run(BlogBackendApplication.class, args);
    }
}
```

其中的mapper-locations是默认配置好的：`/mapper/**/*.xml`

配置：

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.3</version>
</dependency>
```

配置yml：

```yaml
mybatis-plus:
  mapper-locations: classpath*:/mapper/**/*.xml
  type-aliases-package: asia.qaqaqqa.blogbackend.model
```

### 整合Redis

添加依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

使用：

```java
@Autowired
StringRedisTemplate redisTemplate;

@Test
void testRedis(){
    ValueOperations<String, String> stringValueOp = redisTemplate.opsForValue();
    stringValueOp.set("mykey", "vlure", 30);
}
```

## Spring Security

[BV1Cz4y1k7rd](https://www.bilibili.com/video/BV1Cz4y1k7rd)

[Spring Security | FULL COURSE - YouTube](https://www.youtube.com/watch?v=her_7pa0vrg)

### startup

1. spring security intro
2. Oauth2
3. Spring security oauth2
4. JWT

添加依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

添加Spring security依赖之后会为项目自动添加认证`/login`和`/logout`页面。

### 简单验证

方式一：

```java
@Configuration
@EnableWebSecurity
public class SercurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest()
                .authenticated()
                .and()
                .httpBasic();
    }
}
```

这种方式是以浏览器prompt的形式进行验证，验证成功会添加一个Header：

```
Authorization: Basic dXNlcjo2NjA1YmQxOS0xYzA4LTRkZTYtODkwOS1lNTkzY2U0YzFkZGE=
```

内容为：`Basic base64(username:password)`

<h4>添加白名单</h4>

使用`antMatchers`和`permitAll()`函数在前面。

```java
@Configuration
@EnableWebSecurity
public class SercurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/", "index", "/css/*", "/js/*")
                .permitAll()
                .anyRequest()
                .authenticated()
                .and()
                .httpBasic();
    }
}
```

### 密码加密与验证

```java
@Test
void password(){
    PasswordEncoder pe = new BCryptPasswordEncoder();
    String encoded = pe.encode("123");
    System.out.println(encoded);
    System.out.println(pe.matches("123", "$2a$10$wpPPru6SOA.56QQYiqtPu.iu9jLSO28sDB3ybN3uFEoXgrn41x1Vm"));
    System.out.println(pe.matches("123", encoded));
}
```

### 自定义用户

```java
@Configuration
@EnableWebSecurity
public class SercurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/", "index", "/css/*", "/js/*")
                .permitAll()
                .anyRequest()
                .authenticated()
                .and()
                .httpBasic();	// basic auth
    }

    @Override
    @Bean
    protected UserDetailsService userDetailsService() {
        UserDetails jack = User.builder()
                .username("jack")
                .password(passwordEncoder.encode("jack"))
                .roles("student")
                .build();
        
        UserDetails john = User.builder()
                .username("john")
                .password(passwordEncoder.encode("john"))
                .roles("admin")
                .build();
        return new InMemoryUserDetailsManager(jack, john);
    }
}
```

### 角色鉴权

这个是基于在`InMemoryUserDetailsManager`中创建`UserDetailsService`的用户角色`role`来确定的。

通过`role`来鉴权：`hasRole()`用来允许一个用户，`hasAnyRole()`可以允许多个用户

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/", "index", "/css/*", "/js/*").permitAll()
                .antMatchers("/api/**").hasAnyRole(ADMIN.name())
                .anyRequest()
                .authenticated()
                .and()
                .httpBasic();
    }
```

**授权**鉴权：

这里的授权是指`Authority()`，在添加用户的时候指定：

`authorities()`函数

```java
UserDetails john = User.builder()
        .username("john")
        .password(passwordEncoder.encode("john"))
        .roles(ADMIN.name())
        .authorities(ADMIN.getGrantedAuthorities())
        .build();
```

并且需要传入对应`GrantedAuthority`类的数组：

```java
public Set<SimpleGrantedAuthority> getGrantedAuthorities(){
    Set<SimpleGrantedAuthority> permissions = getPermissions().stream()
            .map(p -> new SimpleGrantedAuthority(p.getPermission()))
            .collect(Collectors.toSet());

    //permissions.add(new SimpleGrantedAuthority("ROLE_" + this.name()));
    return permissions;
}
```

鉴权时候通过`antMatchers`后的`hasAuthority`或者`hasAnyAuthority`来指定：

```java
.antMatchers(HttpMethod.POST, "/api/v1/*").hasAuthority(COURSE_READ.getPermission())
```

<h4>注解形式鉴权</h4>

需要在实现`WebSecurityConfigurerAdapter`的安全设置类上添加注解`@EnableGlobalMethodSecurity(prePostEnabled = true)`

在对应的请求上添加`@PreAuthorize`，设置条件`hasRole()`，`hasAnyRole()`，`hasAuthority`()，`hasAnyAuthority()`

> 注意这里的Role需要加前缀`ROLE_`

```java
@GetMapping("/{studentId}")
@PreAuthorize("hasRole('ROLE_ADMIN')")
public Student getStudent(@PathVariable("studentId") Integer studentId){
    return STUDENTS
            .stream()
            .filter(one -> studentId.equals(one.studentId))
            .findFirst()
            .orElseThrow(() -> new IllegalStateException("Student: " + studentId + " is not exists"));
}
```

### CSRF

> 简单的身份验证只能保证请求是发自某个用户的浏览器，却不能保证请求本身是用户自愿发出的

默认情况下，Spring Security会产生CSRF TOKEN

配置：

```java
http.csrf().csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
        .and()
```

### Auth 方式

之前学习的方式都是`Basic auth`方式，但是不能退出登录。大部分网站使用的是以表单(`form auth`)的形式进行提交。

默认情况下，Spring Security也是使用的`formLogin()`

```java
http
        .csrf().csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
        .and()
        .authorizeRequests()
        .antMatchers("/", "index", "/css/*", "/js/*").permitAll()
        .anyRequest()
        .authenticated()
        .and()
        .formLogin();
```

###  OAuth2

第三方认证





## 资料

* 资料百度云地址https://pan.baidu.com/s/1uQV8miKR5LM3u2ZfD5Q51Q，提取码tlnb
* 思维导图：https://pan.baidu.com/s/1HMWH8wDbmrBoGcPdvArfcw  提取码：6666 