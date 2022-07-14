# Rust笔记

书籍：[Rust 程序设计语言](https://kaisery.github.io/trpl-zh-cn/title-page.html)

[TOC]

## 一. 准备

### 1. 安装Rust

下载 Rustup：[Link](https://www.rust-lang.org/zh-CN/learn/get-started)

设置环境变量：

* `RUSTUP_HOME`，Rustup 二进制文件安装位置
* `CARGO_HOME`，Rust 包管理软件的安装位置

打开下载好的 rustup-init 可执行文件，直接安装即可。

安装完毕后检验：

* `rustc --version`
* `cargo --version`

### 2. 设置代理

由于中国地区 cargo 构建较慢，因此需要配置代理。

创建 `$CARGO_HOME/config ` 文件，添加以下配置([bfsu](https://mirrors.bfsu.edu.cn/)镜像)：

```toml
[source.crates-io]
replace-with = 'tuna'

[source.tuna]
registry = "https://mirrors.bfsu.edu.cn/git/crates.io-index.git"
```

### 3. Cargo 管理工具

Cargo 是 Rust 的构建工具和包管理器

- `cargo new proj` 可以创建项目
- `cargo build` 可以编译构建项目，`cargo build --release` 表示发布项目
- `cargo run` 可以运行项目
- `cargo test` 可以测试项目
- `cargo doc` 可以为项目构建文档
- `cargo publish` 可以将库发布到 [crates.io](https://crates.io/)。
- `cargo check` 在不生成二进制文件的情况下构建项目来检查错误

### 4. Hello Rust

首先使用 `cargo new hello-rust` 创建一个项目

之后会生成一个项目：

```
│  .gitignore
│  Cargo.lock
│  Cargo.toml
│
├─src
│      main.rs
└─target
    └─debug
        │  .cargo-lock
        │  hello-rust.d
        │  hello-rust.exe
        └  hello_rust.pdb
    └─release
```

* Cargo.toml 是 rust 的清单文件，包含项目的元数据和依赖库
* main.rs 是编写程序的文件
* debug / release 文件夹下包含编译后的程序

使用 `cargo run` 来运行程序结果

## 二. 初步

### 1. 变量与可变性

变量默认是不可变的(immutable)，如果需要是可变的需要添加关键字 `mut`

```rust
fn main() {
    let x = 5;
    print!("The values is {}", x);
    x = 6;	// Compile Error
    
    let mut y = 6;
    y = 7;
}
```

常量：

```rust
const a: u32 = 1;
```

变量隐藏(Shadowing)，这也是常量和不可变变量的区别，方便变量名的复用，可以改变变量的类型：

```rust
fn main() {
    let x = 5;
    let x = x + 1;	// Shadowing
    {
        let x = x * 2;	// Shadowing
    }
}
```

而对于 `mut` 就不能改变变量类型。

数据类型：

* 整数类型：

  | 长度    | 有符号  | 无符号  |
  | ------- | ------- | ------- |
  | 8-bit   | `i8`    | `u8`    |
  | 16-bit  | `i16`   | `u16`   |
  | 32-bit  | `i32`   | `u32`   |
  | 64-bit  | `i64`   | `u64`   |
  | 128-bit | `i128`  | `u128`  |
  | arch    | `isize` | `usize` |

  其中 `isize` 和 `usize` 表示依赖于运行的系统，如果是64位系统，则为64位。

  字面值：

  | 数字字面值                    | 例子          |
  | ----------------------------- | ------------- |
  | Decimal (十进制)              | `98_222`      |
  | Hex (十六进制)                | `0xff`        |
  | Octal (八进制)                | `0o77`        |
  | Binary (二进制)               | `0b1111_0000` |
  | Byte (单字节字符)(仅限于`u8`) | `b'A'`        |

  其中为了方便读数，中间可以添加 `_` 进行分割，比如 `98_222` 表示十进制 `98222`，`0b1111_0000` 表示二进制 `11110000`

* 浮点类型，默认类型为64位浮点数

  ```rust
  let x = 2.0;		// f64
  let y: f32 = 3.0;	// f32
  ```

* 布尔类型：

  ```rust
  let x = true;
  let t : bool = false;
  ```

* 字符与字符串类型：

  rust中char类型大小为4个字节，可以表示很多unicode值，很多文字比如中日韩甚至emoji文字。

  ```rust
  let emoji = '😂';
  println!("{}", emoji);
  let s = "Very Good";
  println!("{}", s);
  ```

* 复合类型

  Rust 中有两个符合类型：元组和数组

  ```rust
  let tup = ('a', "nihao", 1, 1.5);
  let (a,b, _, _) = tup;
  // 可以根据索引访问
  println!("{}, --- {} --- {}", tup.2, a, b);
  // 指定类型
  let tp: (i32, u32, f64) = (1,1,1.0)
  ```

  数组：

  ```rust
  let mut a = [1,2,3,4];
  let b: [u32; 6] = [5,5,5,5,5,5];
  a[0] = 1;
  ```

### 2. 函数

语句与表达式，函数有一系列语句和一个可选表达式构成，Rust是一门基于表达式的语言。语句是执行操作不返回值的指令，表达式是计算并产生一个值，注意**末尾没有分号**。

**函数作用域内，无法访问函数外部的变量，而其他大括号的作用域，可以访问大括号外部的变量**。

表达式：

```rust
{
    let x = 3;
    x + 1
}
```

函数：

```rust
fn add(a: i32, b: i32)->i32{
    a + b
}
```

### 3. 流程控制

if 语句：

```rust
let a = 10;
if a % 2 == 0 {
    println!("even")
} else if a % 3 == 0 {
    println!("divide 3 okay.")
}

let b = if a % 2 == 0 {"even"} else {"odd"};
println!("{}", b)
```

Loop 循环：

和java一样支持循环标签。

```rust
let mut count = 0;
'counting_up: loop {
    println!("count = {}", count);
    let mut remaining = 10;

    loop {
        println!("remaining = {}", remaining);
        if remaining == 9 {
            break;
        }
        if count == 2 {
            break 'counting_up;
        }
        remaining -= 1;
    }

    count += 1;
}
```

还支持从循环返回值：

```````rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);
}
```````

While 循环：

```rust
let a = [10, 20, 30, 40, 50];
let mut index = 0;

while index < 5 {
    println!("the value is: {}", a[index]);

    index += 1;
}
```

For 循环：

```rust
let a = [10, 20, 30, 40, 50];

for element in a {
    println!("the value is: {}", element);
}

for number in (1..4).rev() {
    println!("{}!", number);
}
println!("LIFTOFF!!!");
```

### 4. 所有权

所有程序都必须管理其运行时使用计算机内存的方式，一些语言具有垃圾回收机制，在程序运行的时候不断寻找不再使用的内存；在另一些语言中，程序员需要亲自分配和释放内存；在 Rust 中是使用**第三种方式**，通过所有权系统管理内存，编译器在编译时候根据一系列规则进行检查。

在 Rust 中，值存储在栈还是堆上会影响语言的行为。

入栈比在堆上分配内存要快，因为（入栈时）分配器无需为存储新数据去搜索内存空间；其位置总是在栈顶。相比之下，在堆上分配内存则需要更多的工作，这是因为分配器必须首先找到一块足够存放数据的内存空间，并接着做一些记录为下一次分配做准备。

访问堆上的数据要比访问栈上的要慢，必须通过指针进行访问。因此了解所有权的目的就是为了管理堆数据。

🔵所有权规则

1. Rust 中每一个值都有一个被称为所有者(owner)的变量。
2. 值在任何时刻有且只有一个所有者。
3. 当Owner离开作用域的时候，这个值就会被丢弃。

String 类型：

```rust
let s1 = String::from("Hello");
let s2 = "Hello";
```

这两者类型不同，一个是 `String` 类型(可变类型)，另一个是 `&str` 类型(不可变)。原因是二者内存分配方式不同：

`s2` 直接被硬编码到可执行文件中，这种快速而且高效。而 `String` 类型为了支持可变，可增长的文本片段，需要在堆上分配一块在编译时未知大小的内存来存储内容，这意味着：

* 必须在运行时想内存分配器(Memory allocator)分配内存
* 需要处理完 String 时将内存归还分配器的方法

对于第二点，有GC的语言中会自动处理。在没有GC语言中需要对其进行显式释放，而且会遇到很多回收难题，比如回收过早，重复回收，忘记回收等情况。在 Rust 中使用不同的策略：变量离开作用域之后会自动释放。

```rust
{
    let s = String::from("hello"); // 从此处起，s 是有效的
    // 使用 s
}   // 此作用域已结束，
// s 不再有效
```

在离开作用域的时候，Rust 会自动调用特殊函数 `drop`。

🔵变量——移动

对于动态分配的变量，如果将 `s` 的值赋值给 `s2`，如果按照之前语言的经验来看，二者都会指向同一块内存，但是由于 Rust 在执行作用域完后会释放内存，因此可能会导致二次释放内存的情况。

所以在 rust 中会使 `s` 变量失效。

```rust
let s = String::from("Hello");
let s2 = s;
println!("{}", s);	// 编译错误
```

🔵变量——拷贝

对于存放在堆上的变量，使用 clone 函数进行拷贝：

```rust
let s = String::from("Hello");
let s2 = s.clone();
println!("{}--{}", s, s2);	// 编译错误
```

🔵函数与所有权

当将 `s` 变量作为 `use_String` 函数的时候，`s` 会移动给 `st` 变量，从而导致 `s` 变量失效。

```rust
fn main() {
    let s = String::from("Hello");
    use_String(s);
    println!("{}", s);	// 编译错误
}

fn use_String(st : String) {
    println!("{}", st);
}
```

### 5. 引用与借用

参考：

* [Rust中的闭包与关键字move](https://zhuanlan.zhihu.com/p/341815515)

如果想在上述函数与所有权中还能继续使用字符串，应该使用引用。使用引用不改变所有权而且运行使用。

```rust
fn main() {
    let s = String::from("Hello");
    use_string(&s);
    println!("{}", s);
    // 打印变量的地址
    println!("{:p}", s);
}

fn use_string(s : &String) {
    println!("{}", s);
}
```

可变引用，如果想要对引用的值进行修改：

```rust
fn main() {
    let mut s = String::from("Hello");
    change(&mut s);
    println!("{}", s);
}

fn change(s : &mut String) {
    s.push_str(", good");
}
```

但是同一时间同一作用域只能有一个对数据的可变引用：

```rust
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;	// 编译错误
```

这种方法可以避免数据竞争，数据竞争由三种行为造成：

* 两个或多个指针同时访问同一个数据
* 至少有一个指针被用来写入数据
* 没有同步数据访问的机制

也不能在同一作用域中同时声明可变和不可变引用。

```rust
let mut s = String::from("hello");

let r1 = &s; // 没问题
let r2 = &s; // 没问题
let r3 = &mut s; // 大问题
```

正确使用：

```rust
let mut s = String::from("hello");

let r1 = &s; // 没问题
let r2 = &s; // 没问题
println!("{} and {}", r1, r2);
// 此位置之后 r1 和 r2 不再使用

let r3 = &mut s; // 没问题
println!("{}", r3);
```

悬垂引用(Dangling References)，在具有指针的语言中，很可能会出现变量内存已释放但是它的指针仍存在，生成的悬垂指针，比如下例子。

```rust
fn dangle() -> &String { // dangle 返回一个字符串的引用

    let s = String::from("hello"); // s 是一个新字符串

    &s // 返回字符串 s 的引用
} // 这里 s 离开作用域并被丢弃。其内存被释放。
  // 危险！
```

正确的做法：

```rust
fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```

变量的所有权被转移出去，因此值没有被释放。

### 6. slice 类型

slice运行引用集合中一段连续的元素序列而不是整个集合，没有元素的所有权。

```rust
fn main() {
    let mut s = String::from("Hello, world");
    let hello = &s[0..5];
    let h = &s[..5];
    let w = &s[6..];
    let all = &s[..];
    print!("{}, {}, {}, {}", hello, h, w, all);
}
```

类型 `&str` 就是一个 slice。

### 7. 结构体

结构体的定义与创建：

```rust
fn main() {
    let mut u1 = User {
        username: String::from("Nick"),
        email: String::from("nick@email.com"),
        age: 66,
    };

    println!("User info: name:{}, email:{}, age:{}", u1.username, u1.email, u1.age);

    u1.email = String::from("nick@gmail.com");
    print!("New Email: {}", u1.email);
}
```

使用字段简写语法：

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,		// email 字段名和参数名相同
        username,	// 字段名和参数名相同
        active: true,
        sign_in_count: 1,
    }
}
```

简写二：

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    // --snip--

    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}
```

匿名结构体（元组结构体）：

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

打印结构体角色：

```rust
#[derive(Debug)]
struct User {
    username: String,
    email: String, 
    age: i32,
}


fn main() {
    let mut u1 = User {
        username: String::from("Nick"),
        email: String::from("nick@email.com"),
        age: 66,
    };

    print!("{:?}\n\n", &u1);
    print!("{:#?}", &u1);
    dbg!(&u1);
}

//User { username: "Nick", email: "nick@email.com", age: 66 }

//User {
//    username: "Nick",
//    email: "nick@email.com",
//    age: 66,
//}

[src\main.rs:18] &u1 = User {
    username: "Nick",
    email: "nick@email.com",
    age: 66,
}
```

为结构体定义方法：

```rust
struct User {
    username: String,
    email: String, 
    age: i32,
}

impl User {
    fn getAge(&self) -> i32 {
        return self.age;
    }
}
```

### 8. 枚举与模式匹配

一般枚举类型：

```rust
#[derive(Debug)]
enum IPType {
    v4, v6(String),v8{x:i32, y:i32}
}

impl IPType {
    fn p(&self) {
        println!("ME");
    }
}

fn main() {
    let a = IPType::v4;
    dbg!(&a);
    let b = IPType::v6(String::from("::1"));
    dbg!(&b);
    let c = IPType::v8 { x: 1, y: 2 };
    dbg!(&c);
    c.p();
}
```

可以支持不同参数类型，以及支持结构体。

🔵Option 枚举

Option 枚举广泛应用在要么有值要么空值。Rust 中没有很多语言存在的空值功能，Rust 使用 `Option<T>` 类型来进行处理空值问题，Option 中含有两部分：`None` 和 `Some<T>` 。

```rust
enum Option<T> {
    None,
    Some(T),
}
```

两者都是 Option 枚举类型，Some 可以包含特定类型的数值，None 需要指定类型。

然而 `Option<T>` 和 `T` 类型无法进行运算，因此需要函数 `unwrap()` 进行转换。

```rust
fn test_option() {
    let a: Option<i32> = None;
    let b = Some(5);
    assert_eq!(a.is_none(), true);
    assert_eq!(b.unwrap()+1, 6);
}
```

### 9. Match 流程控制

相当于 switch 语句，如果像匹配剩余其他选项，则可以指定将值赋给一个变量，即这里的 other 变量。

```rust
fn test_match() {
    #[derive(Debug)]
    enum LoveType {
        Male, FeMale, Lesibian, Gay, Bisexual, Transexual, Queer, Other
    }

    let a = LoveType::Gay;
    let res = match a {
        LoveType::Male => 1,
        LoveType::FeMale => 2,
        other => {
            println!("What fucking is this {:?}", other);
            666
        }
    };
    dbg!(res);
}
```

如果不想转移变量的所有权，则使用 `_` 进行替代。

对于 Option 类型的变量，使用以下 match 进行匹配：

```rust
#[test]
fn test_option2() {
    let a = Some(5);
    let x = match a {
        Some(i) => Some(i+5),
        None => Some(666)
    };

    dbg!(x);
}
```

对于 Option 类型的变量必须匹配 None 类型。

### 10. if let 简单控制流

```rust
let coin = Coin::Penny;
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```

## 三. 使用包、Crate和模块管理

Rust 的模块系统包含以下几个部分：

- **包**（*Packages*）： Cargo 的一个功能，它允许你构建、测试和分享 crate。
- **Crates** ：一个模块的树形结构，它形成了库或二进制项目。
- **模块**（*Modules*）和 **use**： 允许你控制作用域和路径的私有性。
- **路径**（*path*）：一个命名例如结构体、函数或模块等项的方式

### 1. 包和 crate

crate 是一个二进制项或者库。Crate Root 是一个源文件，Rust 编译器以它为起点，并且构成 crate 的根目录。

包中所包含的内容由几条规则来确立。保重最多包含一个库 crate(Library crate)，包中可以包含任意多个二进制 crate（至少一个）。

对于新创建的项目目录：

* `src/main.rs` 就是一个与包名同名的二进制根 crate。 
* `src/lib.rs` 是与包名相同的根库 crate
* `src/bin/*.rs` 是每个文件可以编译为一个二进制 crate。

如果同时包含上述两者，则有两个 crate，一个二级制 crate 和库 crate，名字都与包名相同，包名在 `Cargo.toml` 文件中的 name 指定。

### 2. 模块和路径

模块可以将 crate 中的代码进行分组，来提高可读性和重用性。模块还可以控制访问权限，决定公有(public)还是私有(private)。Rust 中默认所有项都是私有的，但是子模块可以使用父模块中的项。

```rust
// Restaurant
mod front {
    pub mod hosting {
        pub fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}

pub fn eat() {
    // absolute
    crate::front::hosting::add_to_waitlist();
    // relative
    front::hosting::add_to_waitlist();
}

```

模块中可以保留结构体、枚举、常量、trait以及函数等。

为了引用某一项，就需要使用路径的方式来对其进行访问，路径有两种方式：

* 绝对路径：从 crate 根开始，以 crate 名或者字面值 `crate` 开头
* 相对路径：从当前模块开始，以 `self`, `super` 或者当前模块的标识符开头

对于调用父类的方法，可以使用 `use` 或者 `super` 方式来进行调用：

```rust
pub fn eat() {
    // absolute
    crate::front::hosting::add_to_waitlist();
    // relative
    front::hosting::add_to_waitlist();
}

// 方式1
mod back {
    fn cook() {
        super::eat();
    }
}

// 方式2
mod back {
    use crate::eat;

    fn cook() {
        eat()
    }
}
```

`super` 关键字的作用相当于文件系统中的 `..` 来标识上一级。

### 3. use 关键字

上述调用的方式都比较冗长而且重复，不方便编写代码。

```rust
mod front {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

fn easy_eat() {
    // 方式1
    use front::hosting;
    // 方式2
    use self::front::hosting;
    hosting::add_to_waitlist();
}
```

如果不同子模块名称相同冲突，可以使用 `as` 关键字提供新的名称。

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
```

使用 `pub use` **重导出**名称

对于嵌套模块中的引用，为了防止引用链过长，可以将 `pub use` 联合进行使用，称为重导出(re-exporting)。

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;
```

如果需要引入很多定义于相同包或者模块的项时候，每次单独列出一行会占用很大空间：

```rust
// 之前
use std::cmp::Ordering;
use std::io;
// 之后
use std::{cmp::Ordering, io};
```

也可以将一个路径下**所有**项引入作用域：

```rust
use std::collections::*;
```

### 4. 拆分成多个文件模块

将模块分成不同的文件，方便代码阅读。将上述的 front 模块存放到 front.rs 中：

src/front.rs 文件中内容为：

```rust
// Restaurant
pub mod hosting {
    pub fn add_to_waitlist() {}

    fn seat_at_table() {}
}

mod serving {
    fn take_order() {}

    fn serve_order() {}

    fn take_payment() {}
}
```

src/lib.rs 中引用 front 内容：

```rust
mod front;	// 用于加载同名文件中的模块

use crate::front::hosting;

pub fn eat() {
    hosting::add_to_waitlist();
}
```

## 四. 常见集合

在本章将了解 Rust 中常用的集合：

- Sequences: [`Vec`](https://doc.rust-lang.org/std/vec/struct.Vec.html), [`VecDeque`](https://doc.rust-lang.org/std/collections/struct.VecDeque.html), [`LinkedList`](https://doc.rust-lang.org/std/collections/struct.LinkedList.html)
- Maps: [`HashMap`](https://doc.rust-lang.org/std/collections/hash_map/struct.HashMap.html), [`BTreeMap`](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html)
- Sets: [`HashSet`](https://doc.rust-lang.org/std/collections/hash_set/struct.HashSet.html), [`BTreeSet`](https://doc.rust-lang.org/std/collections/struct.BTreeSet.html)
- Misc: [`BinaryHeap`](https://doc.rust-lang.org/std/collections/struct.BinaryHeap.html)

### 1. Vector

vector 的基本使用如下：

```rust
#[cfg(test)]
mod test_vector {
    #[test]
    fn t1() {
        // 创建一个空的 vector
        let v1:Vec<i32> = Vec::new();
        // 使用宏进行创建不为空的 vector
        let mut v2 = vec![1,2,3];
        // 添加元素到末尾
        v2.push(4);
        // 删除末尾元素
        v2.pop();
        // 获取元素
        print!("{}\n", v2[0]);
        // get 方法获取 option
        if let Some(a) = v2.get(10) {
            println!("{}", a);
        } else {
            // None 值
            println!("The value is None");
        }
        // 遍历
        for i in &v2 {
            println!("{}.", i);
        }
        println!("=============");
        // 遍历修改
        for i in &mut v2 {
            *i += 2;
            println!("{}.", i);
        }
    }
}
```

建议使用 `get()` 方法来获取元素，修改元素的时候需要 `*` 就该引用的值。

vector 中的借用检查错误案例：

```rust
#[test]
fn t2() {
    let mut v2 = vec![1,2,3];
    let first = &v2[0];
    println!("{}", first);
    v2.push(1);
    println!("{}", first);
}
```

发生错误的原因：当获取不可变引用之后，对vector进行修改之后再次尝试使用这个不可变引用会发生错误。因为 vector 在增加新元素的时候，如果没有一次相邻元素存放的情况下，可能需要重新分配新内存并且将老元素拷贝到新的空间中。因此第一个元素的引用就指向了被释放的内存。

使用枚举存储多种类型的值：

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

### 2. 字符串

创建字符串：

```rust
#[test]
fn t1() {
    // 创建空的 String
    let mut s = String::new();
    // 创建非空 String
    let mut s = String::from("Hello world");
    let mut s1 = "Good Night".to_string();
    s1.push_str("_小狗");
    println!("{}", s1);
    // 拼接
    let s2 = format!("{}--{}", s, s1);  // 不会获取所有权
    let s3 = s + &s1;
    println!("{}", s2);
    println!("{}", s3);
}
```

需要注意函数是否会获取字符串的所有权。

对于 `+` 运算符，实际上是调用了 `fn add(self, s: &str) -> String` 函数，其中 self 没有添加引用，因此会转移所有权，s 是 slice 类型，并且使用的 Deref 强制转化(deref coercion)技术，将 `&String` 类型强转为 `&str` 类型。

Rust 中 String 类型不支持索引，String 类型其实是 `Vec<u8>` 的封装。String 类型的长度不是字符的数量，而是使用 UTF-8 所需要的字节数。如果偏要索引，则需要转为 `&str` 的 slice 类型进行索引操作。

```rust
#[test]
fn t2 () {
    let s1 = "你好,世界".to_string();
    println!("{}-- len:{}", &s1[0..3], s1.len());
}
// 输出
//你-- len:13
```

其中每个中文占3个字节，如果索引改为 `[0..1]` 就会报错无效的索引。

遍历字符串的方法：

```rust
// 遍历每个字符
for c in s1.chars() {
    println!("{}", c);
}
println!("==============");
// 遍历每个字节
for b in s1.bytes() {
    println!("{}", b);
}
// 输出
你
好
,
世
界
==============
228
189
160
229
165
189
44
228
184
150
231
149
140
```

### 3. HashMap

创建 Hashmap

```rust
#[test]
fn t1() {
    // 构建方式 1
    let mut score = HashMap::new();
    score.insert("a".to_string(), 1);
    score.insert("bs".to_string(), 2);
    // 构建方式 2
    let teams = vec!['a', 'b'];
    let scores = vec![10, 50];
    let maps: HashMap<_, _> = teams.into_iter().zip(scores).collect();
}
```

访问 map 中的值：

```rust
let mut score = HashMap::new();
score.insert("a".to_string(), 1);
score.insert("bs".to_string(), 2);
// 获取值
let key = "abc".to_string();
let v = score.get(&key);
if let Some(v1) = v {
    println!("Value is {}.", v1);
} else {
    println!("The key is not exists.");
}
// 遍历map
for (k, v) in &score {
    println!("{}--{}",k,v);
}
```

## 五. 错误处理

Rust 中的错误分为**可恢复的**（*recoverable*）和 **不可恢复的**（*unrecoverable*）错误。

大多数语言不区分这两种错误，采用异常同一处理。Rust 中有 `Result<T, E>` 类型用于处理可恢复的错误，`panic!` 宏用于处理不可恢复的错误。

### 1. panic

当遇到 panic 的时候，程序默认会**展开** (unwinding)，这意味着 Rust 会回溯栈并且清理它遇到每一个函数的数据。另一种选择是**终止** (abort)，这个不会清理就退出程序。

如果你需要项目最终的二进制文件越小越好，panic 会通过 `Cargo.toml` 的 `[profile]` 部分增减 `panic = 'abort'` 由此可以将展开切换为终止。在 release 中panic终止可以设置：

```toml
[profile.release]
panic = 'abort'
```

设置 panic：

```rust
panic!("Something panic!.");
```

可恢复的错误：

```rust
#[test]
fn t2() {
    let f = File::open("E:/Notes/rust/code/hello-rust/src/bin/add.rs");
    let f = match f {
        Ok(file) => file,
        Err(err) => panic!("Open file error: {}", err)
    };
}
```

匹配不同类型的错误：

```rust
#[test]
fn t3() {
    let f = File::open("add.rs");
    let f = match f {
        Ok(file) => file,
        Err(err) => match err.kind() {
            ErrorKind::NotFound => {
                panic!("File not find");
            },
            other => {
                panic!("Other error: {}", other)
            }
        }
    };
}
```

🔵panic 的简写：unwrap 和 expect

使用 match 有点冗长不好表达意图。

* `unwrap()` 如果 `Result` 返回的是 `OK`，`unwrap` 返回的是 `OK` 里的值。如果是 `Err` 则会自动调用 `panic!`

  ```rust
  #[test]
  fn t4() {
      let f = File::open("add.txt").unwrap();
      println!("Open success {:?}", f);
  }
  ```

* `expect()` 方法与 `unwrap()` 类似，它还可以允许自定义错误信息：

  ```rust
  #[test]
  fn t5() {
      let f = File::open("add.rs").expect("Failed OOOOOOOOOOPs");
      println!("Open success {:?}", f);
  }
  ```

🔵传播错误

```rust
fn main() {
    use std::fs::File;
    use std::io::{self, Read};

    fn read_username_from_file() -> Result<String, io::Error> {
        let f = File::open("hello.txt");

        let mut f = match f {
            Ok(file) => file,
            Err(e) => return Err(e),
        };

        let mut s = String::new();

        match f.read_to_string(&mut s) {
            Ok(_) => Ok(s),
            Err(e) => Err(e),
        }
    }
}
```

传播错误的简写形式： `?` 运算符

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

继续缩短：

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();
    File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```

### 2. 什么时候 panic

1. 实例、代码原型和测试非常适合 panic
2. 比编译器知道的更多时候
3. 错误指导原则
4. 自定义类型的有效性验证

## 六. 泛型、trait和生命周期

### 1. 泛型

函数泛型

```rust
fn largest<T>(list: &[T]) -> &T {
    if list.len() == 0 {
        panic!("Length error");
    }
    let ret = &list[0];
    ret
}
```

结构体泛型：

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

struct Point2<T, U> {
    x: T,
    y: U,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
    
    let both_integer = Point2 { x: 5.0, y: 10 };
    let both_float = Point2 { x: 1.0, y: 4.0 };
    let integer_and_float = Point2 { x: 5, y: 4.0 };
}
```

枚举中的泛型：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### 2. trait

trait 用于告诉 rust 编译器某个特定类型拥有可能与其他类型共享的功能。类似接口(interfaces)的功能。

```rust
pub trait Person {
    fn walk(&self);
    fn eat(&self, food: String);
    fn live() {
        println!("Living");
    }
}

pub struct Batman {
    name: String,
    age: i32,
}

impl Person for Batman {
    fn walk(&self) {
        println!("Batman is walking");
    }

    fn eat(&self, food: String) {
        println!("Batman({}) is eating {}", self.name, food);
    }
}

fn main() {
    let a = Batman {
        name: "Johny".to_string(),
        age: 22,
    };

    a.eat("Hamburger".to_string());
}
```

trait 中可以定义默认实现，子结构体可以选择实现或者不实现。

trait 也可以作为参数，可以传入众多子实现结构体。

```rust
fn call(p: &impl Person) {
    p.walk();
}
```

如果函数参数需要实现多个trait的时候，需要使用 `+` 将其拼接起来：

```rust
pub fn notify(item: &(impl Summary + Display)) {}
fn returns_summarizable() -> impl Summary {}
```

还可以：

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}
```

### 3. 生命周期和引用有效性

函数中泛型生命周期：

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}

fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

我们不需要 `longest` 函数获取参数的所有权，但是这段代码会报错：

```
help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
help: consider introducing a named lifetime parameter
  |
9 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |        ++++   ++         ++       ++
```

因为 rust 不知道将要返回的引用是指向 x 还是指向 y，并且我们也不知道。我们无法确定变量的声明周期，因此我们需要增加泛型生命周期参数来定义引用间的关系以便**借用检查器**可以进行分析。

🔵生命周期注解语法：

生命周期注解不改变任何引用的生命周期长短。当指定了生命周期后的函数也能接受任何生命周期的引用，声明周期描述了多个引用生命周期相互的关系。

生命周期参数名称必须以撇号（`'`）开头，其名称通常全是小写，类似于泛型其名称非常短。`'a` 是大多数人默认使用的名称。生命周期参数注解位于引用的 `&` 之后，并有一个空格来将引用类型与生命周期注解分隔开。

比如：

```rust
&i32        // 引用
&'a i32     // 带有显式生命周期的引用
&'a mut i32 // 带有显式生命周期的可变引用
```

用于标注 rust 多个引用的泛型生命周期是如何关联的。如果两个引用都被 `'a` 所修饰，就表示两个变量的泛型生命周期存在的一样久。

## 七. 测试

### 1. 如何编写测试

需要在模块上标注 `#[cfg(test)]` 和 `#[test]` 注解，可以使用宏函数 `assert*!` 等来进行判断。

```rust
#[derive(Debug)]
struct User {
    username: String,
    email: String, 
    age: i32,
}

impl User {
    fn getAge(&self) -> i32 {
        return self.age;
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    #[test]
    fn get_user() {
        let u = User {
            username: "Nick".to_string(),
            email: "abce@qq.com".to_string(),
            age: 12
        };
        dbg!(&u);
    }

    #[test]
    fn test_add () {
        let a = 4;
        let b = 6;
        assert_eq!(a+b, 11);
    }
}
```

使用 `assert!()` `assert_eq!()` 宏来进行判断：

```rust
assert_eq!(1, 1+1, "Not equal with {}", x);
assert!(result.contains("Carol"), "contain name, value was `{}`", result);
```

🔵使用 `should_panic` 检查 panic：

还应该检查代码是否按照期望处理错误。添加 `#[should_panic]` 来实现存在 panic 的时候会通过，没有 panic 的时候会失败。

```rust
#[test]
#[should_panic]
fn greater_than_100() {
    Guess::new(200);
}
```

也可以添加帮助信息：

```rust
#[test]
#[should_panic(expected = "Guess value must be less than or equal to 100")]
fn greater_than_100() {
    Guess::new(200);
}
```

🔵将 `Result<T, E>` 用于测试

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```

### 2. 控制测试运行

并行或者连续运行测试，当运行多个测试的时候，rust 会默认使用现存并行运行。

如果不希望并行运行测试，或者想要更加精确的控制线程的数量，可以设置如下：

```sh
cargo test -- --test-threads=1
```

🔵显示函数输出

默认情况下，使用 `cargo test` 命令 rust 会拦截打印到标准输出。

可以使用 `--show-output` 来显示输出。

```sh
cargo test -- --show-output
```

🔵指定名字来运行部分测试

```sh
cargo test add
```

这个会运行名字中包含 `add` 的测试。

🔵忽略部分测试

可以通过标注 `#[ignore]` 来进行忽略测试：

```rust
#[test]
#[ignore]
fn expensive_test() {
    // 需要运行一个小时的代码
}
```

使用 `cargo test` 会自动跳过这些测试。

如果希望只允许被忽略的测试，可以使用：

```sh
cargo test -- --ignored
```

### 3. 测试的组织结构

测试主要分为两类：单元测试和集成测试。单元测试更小而且更集中；集成测试相当于进行黑盒测试。

🔵单元测试

单元测试的目的就是在与其他部分隔离的情况下测试每一个单元的代码，以便于快速而且准确的判断某个单元的代码是否符合预期。

单元测试和要测试的代码共同存放在  src 目录下相同的文件中。规范就是在包含测试函数的 `tests` 模块，并且使用 `cfg(test)` 进行标注。

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

🔵集成测试

编写集成测试需要创建同 `src` 同级的 `tests` 目录

```rust
use adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

指定特定的集成测试：

```sh
cargo test --test integration_test
```

## 八. 函数式编程

- **闭包**（*Closures*），一个可以储存在变量里的类似函数的结构
- **迭代器**（*Iterators*），一种处理元素序列的方式

### 1. 闭包

函数 -> 闭包：

```rust
// 函数
fn simulated_expensive_calculation(intensity: u32) -> u32 {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    intensity
}
```

将其转为闭包：

```rust
let expensive_closure = |num| {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
```

一对 `|` 之间包裹的是闭包的参数，比如：`|param1, param2|`。

如果只有一行就不需要进行大括号。

```rust
fn main() {
    // 方式 1
    let f = |x: i32| x+1;
    dbg!(f(3));
    // 方式 2
    let f1 = |a: isize, b: isize| {
        let c = a + b;
        println!("{} + {} = {}", a, b, c);
        c
    };
    dbg!(f1(2,3));
    // 方式 3，参数类型和返回类型都指定
    let gmin = |a: isize, b: isize| -> f32 {(a-b) as f32};
    dbg!(gmin(1,3));
}
```

### 2. Fn trait 闭包

用于存放和调用闭包的结构体。这种结构体会在需要结果的时候执行闭包并且缓存结果。剩下的代码不必再复杂保存结果并且可以复用该值。这种称为**惰性求值**。

```rust
struct Cacher<T>
where
    T: Fn(u32) -> u32,
{
    calculation: T,
    value: Option<u32>,
}

impl<T> Cacher<T>
where
    T: Fn(u32) -> u32,
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            }
        }
    }
}
```

在此处 Cacher 中的 Calculation 类型被指定为输入为 `u32` 输出为 `u32` 的闭包泛型，其中的结构体实现举例的泛型闭包的使用方法。

其他的 `Fn` trait：

- `FnOnce` 消费从周围作用域捕获的变量，闭包周围的作用域被称为其 **环境**，*environment*。为了消费捕获到的变量，闭包必须获取其所有权并在定义闭包时将其移动进闭包。其名称的 `Once` 部分代表了闭包不能多次获取相同变量的所有权的事实，所以它只能被调用一次。
- `FnMut` 获取可变的借用值所以可以改变其环境
- `Fn` 从其环境获取不可变的借用值

### 3. 迭代器

迭代器模式允许你对一个序列进行某些处理。在 Rust 中迭代器是**惰性的**，这意味着在调用方法使用迭代器之前不会起任何效果。

迭代器都实现 `Iterator` 的 trait，定义类似于：

```rust
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    // 此处省略了方法的默认实现
}
```

获取迭代器：

```rust
// 获取迭代器(元素引用不可变)
let mut iter = l.iter();
// 获取迭代器(元素引用可变)
let mut iter_mut = l.iter_mut();
```

获取到的迭代器必须是可变的，由于需要改变序列位置的信息，因此必须要 `mut` 进行修饰。

默认 `iter()` 方法获取到的元素是不可变引用，如果想要获取可变引用就需要使用 `iter_mut()` 方法。

🔵消费迭代器：

```rust
let s: isize = iter.sum();
```

使用 `map` 方法：

```rust
let v1: Vec<i32> = vec![1, 2, 3];
let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
assert_eq!(v2, vec![2, 3, 4]);
```

❗注意这里必须要添加 `collect()` 方法，因为 rust 中迭代器是惰性的，不会自动生成结果，需要使用 `collect()` 方法后才会创建新的结果。

使用 `filter` 方法：

```rust
fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}
```

🔵循环 VS 迭代器

所有这些 Rust 能够提供的优化使得结果代码极为高效。现在知道这些了，请放心大胆的使用迭代器和闭包吧！他们使得代码看起来更高级，但并不为此引入运行时性能损失。

## 九. Cargo 和 Crates.io 

### 1. 发布配置

```sh
cargo build
cargo build --release
```

两者编译器使用不同的配置，对于 `Cargo.toml` 中未明确配置的时候，Cargo 会采取默认配置：

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

### 2. 将 Crate 发布到 Crates.io

编写有用的文档注释：文档注释通常使用三个斜杠 `///` 来支持 Markdown 注解来格式化文本。

```rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

可以使用 `Cargo doc --open` 生成文档。

还有另一种风格的文档注释：`//!` ，通常用于 Crate 根文件或者模块的跟文件来提供文档。

剩余发布见：[将 crate 发布到 Crates.io](https://kaisery.github.io/trpl-zh-cn/ch14-02-publishing-to-crates-io.html#创建-cratesio-账号)

### 3. Cargo 工作空间

随着项目开发的深入，库 crate 会持续增大，会拆分为多个库 crate。对于这种情况 Cargo 提供了一种**工作空间**(workspaces) 的功能，能够帮助管理多个相关协同开发的包。

工作空间是一系列共享同样的 Cargo.lock 和输出目录的包。

对于根目录需要指定一个 `Cargo.toml` 其中包含 crate 的成员：

```toml
[workspace]

members = [
    "adder",
    "add_one",
]
```

然后使用 `cargo new xxx` 来构建成员，最终使用 `cargo build` 命令最终生成的二进制文件也是在根目录下生成。

```sh
cargo new adder
cargo new add_one --lib
# 编译生成
cargo build
```

最终生成的目录为：

```
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

如果 crate `adder` 想要引用 `add_one` 库crate，需要在 `adder/Cargo.toml` 中进行配置。 

```toml
[dependencies]
add_one = { path = "../add_one" }
```

在 函数中使用：

```rust
use add_one;

fn main() {
    let num = 10;
    println!(
        "Hello, world! {} plus one is {}!",
        num,
        add_one::add_one(num)
    );
}
```

在工作空间中依赖外部包：

```toml
[dependencies]
rand = "0.8.3"
```

通过 `use rand;` 使用包，并且运行 `cargo build` 之后就可以引入并且编译了。

### 4. cargo install 和自定义扩展

这个命令用于在本地安装和使用二进制 crate。

使用 `cargo install` 会自动安装到目录 `$RUST_HOME/.cargo/bin`  目录下，

Cargo 的设计使得开发者可以通过新的子命令来对 Cargo 进行扩展，而无需修改 Cargo 本身。如果 `$PATH` 中有类似 `cargo-something` 的二进制文件，就可以通过 `cargo something` 来像 Cargo 子命令一样运行它。像这样的自定义命令也可以运行 `cargo --list` 来展示出来。能够通过 `cargo install` 向 Cargo 安装扩展并可以如内建 Cargo 工具那样运行他们是 Cargo 设计上的一个非常方便的优点！

## 十. 智能指针(未完结)

智能指针是一类数据结构，其类似指针但是也拥有元数据和功能。其中**引用计数**(reference counting) 就是一种智能指针的类型之一，其允许数据有多个所有者。

在 rust 中，普通引用和智能指针的区别是引用是一类只借用数据的指针，相反在大部分情况下，智能指针**拥有**他们指向的数据。

智能指针通常由结构体实现，相比于普通的结构体，智能指针实现了 `Deref` 和 `Drop` 的 trait。

* `Deref` 允许智能指针结构体实例像引用一样，既可以用于引用，也可以用于智能指针。
* `Drop` 允许智能指针离开作用域时候运行代码。

常见的智能指针有：

- `Box<T>`，用于在堆上分配值
- `Rc<T>`，一个引用计数类型，其数据可以有多个所有者
- `Ref<T>` 和 `RefMut<T>`，通过 `RefCell<T>` 访问。（ `RefCell<T>` 是一个在运行时而不是在编译时执行借用规则的类型）。

此外还会涉及**内部可变性**(interior mutability)模式，这是不可变类型暴露出改变其内部值的API。也会讨论**引用循环**(reference cycles)会如何泄露内存以及如何避免。

### 1. Box\<T\> 堆数据

box 类型运行将值存放在堆上而不是栈上。

Box 允许创建递归类型。Rust 在编译的时候需要知道类型占用多少空间，对于无法在编译时候知道的类型是**递归类型**(recursive type)，其值的一部分可以是相同类型的另一个值。

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
```

Box 只提供间接存储和堆分配，并没有任何其他特殊的功能，也没有性能损失。

Box 实现了 `Deref` trait 其能够允许重载**解引用运算符**(dereference operator) `*` 。通过这种方式可以实现将智能指针被当作常规引用来对待。

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);
    // let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

🔵函数以及方法中隐式 Deref 强制转换

Deref 强制转换是 rust 在函数或方法上的一种便利。比如将 `&String` 转换为 `&str`，因为 `String` 实现了 `Deref` trait 因此可以返回 `&str`

Rust 在发现类型和 trait 实现满足三种情况时会进行 Deref 强制转换：

- 当 `T: Deref<Target=U>` 时从 `&T` 到 `&U`。
- 当 `T: DerefMut<Target=U>` 时从 `&mut T` 到 `&mut U`。
- 当 `T: Deref<Target=U>` 时从 `&mut T` 到 `&U`。

🔵使用 Drop trait 清理代码：

Drop 允许值离开作用域的时候执行一些代码。指定在值离开作用域时候应该执行代码的方式就是实现 `Drop` trait，以下例子演示了什么时候堆调用 `drop` 函数，无需显式调用 `drop` 方法。

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from("my stuff"),
    };
    let d = CustomSmartPointer {
        data: String::from("other stuff"),
    };
    println!("CustomSmartPointers created.");
}
```

如果想要提前释放资源，不允许直接调用 drop 函数，而是使用标准库提供的 drop 函数。

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from("some data"),
    };
    println!("CustomSmartPointer created.");
    drop(c);
    println!("CustomSmartPointer dropped before the end of main.");
}
```

### 2. Rc\<T>引用计数智能指针

大部分情况下变量的所有权都是十分明确的，可以准确知道哪个变量拥有哪个值。然而有些情况下单个值可能会有多个所有者。rust 中有 `Rc<T>` 引用计数的类型来启用变量多所有权。引用计数意味着引用当前的值的数量来知晓这个值是否被使用。

> 注意，`Rc<T>` 只适用于单线程场景。

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("count after creating a = {}", Rc::strong_count(&a));
    let b = Cons(3, Rc::clone(&a));
    println!("count after creating b = {}", Rc::strong_count(&a));
    {
        let c = Cons(4, Rc::clone(&a));
        println!("count after creating c = {}", Rc::strong_count(&a));
    }
    println!("count after c goes out of scope = {}", Rc::strong_count(&a));
}

```

使用克隆会增加引用的数量，使用 `Rc::strong_count` 来获取当前的计数。

## 十一. 无畏并发

本章将要涉及到的内容：

- 如何创建线程来同时运行多段代码。
- **消息传递**（*Message passing*）并发，其中信道（channel）被用来在线程间传递消息。
- **共享状态**（*Shared state*）并发，其中多个线程可以访问同一片数据。
- `Sync` 和 `Send` trait，将 Rust 的并发保证扩展到用户定义的以及标准库提供的类型中。

其他：[Rust异步编程和tokio框架](https://rust-book.junmajinlong.com/ch100/00.html)

### 1. 线程

由编程语言调用操作系统 API 创建线程的模型有时被称为 1:1，一个 OS 线程对应一个语言线程。Rust 标准库只提供了 1:1 线程实现；有一些 crate 实现了其他有着不同取舍的线程模型。

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let t = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
    // join
    t.join();
}
```

注意：当主线程结束的时候，子线程也会结束。可以使用 `join()` 函数来等待线程的结束。

🔵线程与 `move` 闭包

对于以下一个例子：

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);	// 会报错
    });

    handle.join().unwrap();
}
```

rust 会推断如何捕获 `v` ，然而 `println!` 只需要 v 的引用，闭包尝试借用 v。rust 并不知道这个线程会执行多久，所以无法知晓 v 的引用是否会一直有效。假如在主线程中销毁 v 变量，那么子线程中可能会在运行途中报错。

因此使用 `move` 关键字，将变量所有权转移到子线程中。

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

### 2. 线程间通信 channel

rust 中借助 go 中消息传递的概念，通过通讯来共享内存。rust 中实现消息传递的并发工具也是 channel。

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

这里使用 `mpsc::channel` 函数创建一个新的信道；`mpsc` 是 多个生产者，单个消费者（multiple producer, single consumer）的缩写。tx 和 rx 分别表示发送者和接收者。

信道的接收端有两个有用的方法：`recv` 和 `try_recv`。

* `recv` 方法会阻塞主线程执行直到从信道中接收一个值。一旦发送了一个值，`recv` 会在一个 `Result<T, E>` 中返回它。当信道发送端关闭，`recv` 会返回一个错误表明不会再有新的值到来了。

* `try_recv` 不会阻塞，相反它立刻返回一个 `Result<T, E>`：`Ok` 值包含可用的信息，而 `Err` 值代表此时没有任何消息。如果线程在等待消息过程中还有其他工作时使用 `try_recv` 很有用：可以编写一个循环来频繁调用 `try_recv`，在有可用消息时进行处理，其余时候则处理一会其他工作直到再次检查。

发送多个消息：

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

这里使用 `for` 循环接受信息。

### 3. 资源共享

由于在多线程进行资源争抢的时候会对同一个变量进行操作，但是 `Rc<T>` 并不是线程安全的，因此应该使用 `Arc<T>` (Atomic Reference Counter) 原子引用计数器进行处理。

```rust
fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

### 4. sync 和 send trait

然而有两个并发概念是内嵌于语言中的：`std::marker` 中的 `Sync` 和 `Send` trait。

🔵通过 `Send` 运行线程之间转移所有权

几乎所有 rust 类型都是 `Send` 的除了 `Rc<T>`，因为如果克隆了 `Rc<T>` 的值并且尝试将所有权转移到另一个线程，这两个线程都可能同时更新引用计数。

🔵`sync` 运行多线程访问

`Sync` 标记 trait 表明实现了 `Sync` 的类型可以安全的在多个线程中拥有其值的引用。
