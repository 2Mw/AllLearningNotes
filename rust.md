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

如果想在上述函数与所有权中还能继续使用字符串，应该使用引用。使用引用不改变所有权而且运行使用。

```rust
fn main() {
    let s = String::from("Hello");
    use_string(&s);
    println!("{}", s);
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

## 三. 测试

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

