# Go Concurrency

[TOC]

多进程适合高密集计算型，多线程适合IO密集型如爬虫，文件操作。

[BV137411z7sp](https://www.bilibili.com/video/BV137411z7sp)

[BV1jK4y1N7ST IO复用](https://www.bilibili.com/video/BV1jK4y1N7ST)

## IO复用模型

> fd: file descriptor 文件描述符

### 流 IO操作 阻塞

流：可以进行IO操作的内核对象，比如文件、管道、套接字，流的入口(fd)

IO操作：对流的操作

阻塞等待：不能很好的处理多个IO请求，同一时间同一时刻只能处理一个阻塞监听。

多路IO复用：既能阻塞等待不占用资源，又能同一时刻监听多个IO请求。

<img src="https://i.loli.net/2021/07/16/Dv6bHQPhX5Mngu8.png" alt="image-20210716170919024" style="zoom:50%;" />

### IO复用

如何解决大量IO读写请求：

* 阻塞+多线程/多进程

  <img src="https://i.loli.net/2021/07/16/YCW7bfKA53mHX8t.png" alt="image-20210716171204591" style="zoom:50%;" />

  不可取，需要开辟很多资源

* 非阻塞+忙轮询

  会占用很多资源，相当于一直`while`循环。

* Select方法

  ```python
  while true:
  	select(stream[])	# 阻塞
  	
      for i in stream:
          read(i)
  ```

  会告诉你有事件发生，但不会告诉你时间具体信息。

* epoll方法

  ```python
  while True:
      stream[] = epoll_wait(epoll_fd)
      for i in strem[]:
          # Read..
  ```

  <img src="https://i.loli.net/2021/07/16/byqMzfexYod4aPu.png" alt="image-20210716171836848" style="zoom:50%;" />

什么是epoll：IO多路复用技术，只关心“活跃”的请求，能够处理大量的连接请求(**系统可以打开的最大文件数目**)。

```sh
Linux中可以打开的最大文件数码
[root ~]# cat /proc/sys/fs/file-max
183917
```

### 模型一 单线程accept(无IO复用)

<img src="https://i.loli.net/2021/07/16/aIu1Rf4Sh7YCnDt.png" alt="image-20210716174844814" style="zoom:50%;" />

当服务端处于读写状态的时候，对于其他客户端23的请求是无响应的，处理完毕之后才能接受限一个请求。

非并发模型，串行服务器，最大网络请求量为1，并发量为1。

### 模型二 单线程Accept+多线程读写业务(无IO复用)

<img src="https://i.loli.net/2021/07/16/p7b42NUq1leFaKg.png" alt="image-20210716175354278" style="zoom:50%;" />

client给server连接的时候，server会分配一个线程同该client进行读写操作，然后server会立马回到accept阻塞状态，可以继续处理下一个请求，继续分配线程与下一个客户端处理。

特点：支持了并发，一个客户端同一个线程进行处理，server处理的内聚性比较高。客户端数量增多，线程也增多。对于高并发情景，收到硬件的瓶颈；对于长连接，服务器需要保存心跳连接，占用连接和线程开销；适用于客户端数量不多的场景。

### 模型三 单线程多路IO复用

<img src="https://i.loli.net/2021/07/29/wlzrtOQSN5uen4i.png" alt="image-20210729205451699" style="zoom: 60%;" />

**分析：**

1. 主线程创建`ListenFd`之后，采用多路IO复用机制如(select, epoll)进行IO状态的阻塞监控。有Client1客户端的连接请求，IO复用机制检测后`ListenFd`出发读事件，则进行Accept建立连接，并将新生产的`ConnFd1`加入到监听IO的集合中。
2. Client1再次进行读写操作业务的时候，主线程中的多路IO复用会触发服务器端的读写事件业务。
3. 当服务器正在进行client1的读写业务的时候，其他客户端的连接请求和读写请求会阻塞，不能够及时响应。

特点：

* 使用单流程监听多个客户端的读写状态模型，不需要1：1的线程数量关系。
* 阻塞IO，可以极大利用CPU
* 可以监听多个客户端的请求业务，但是同一时间只能处理一个客户端的业务。
* 当有大量的客户端请求的时候，由于业务串行执行，会存在排队等待的现象，并发量还是为1。

### 模型四 单线程多路IO复用+多线程读写业务（工作池）

> 使用不是很多

<img src="E:\Notes\Go\GoConcurrency.assets\image-20210729212754812.png" alt="image-20210729212754812" style="zoom:50%;" />

跟模型三类似，只是减少了多路io复用的排队时间。将业务使用异步处理。

读写业务为1，处理事件业务为worker的数量。

### 模型五 单线程多路IO复用+多线程多路IO复用(线程池)

> 常用

<img src="https://i.loli.net/2021/07/29/xKm2QzgsRhCUGrl.png" alt="image-20210729222347312" style="zoom:50%;" />

**模型分析：**

1. 服务器在启动监听之前，需要开启固定数量N的线程，用线程池来进行管理
2. 主线程创建`ListenFd`之后，会采用多路IO复用机制(select, epoll)的状态进行阻塞监控。有客户端请求连接的时候，IO复用机制ListenFd出发读机制，进行Accept连接，并且将新生成的`connFd1`分发给线程池中的某个线程进行监听。
3. 线程池中的每个线程也启用多路IO复用，用来监听有主线程分发下来的socket套接字。
4. thread监听各自分发的，并且各自处理。

特点：

* 将模型三中业务处理分散到多个线程当中，可以监听的数量成倍增加；之前监控的数量取决于主线程的机制限制（select为1024，epoll与内存大小有关 约3-6W），建议N的大小与CPU的核心数量为1：1。
* 虽然监听的并发数量提升，但是最高的读写并行通道为N，并且多个处于同一个Thread的客户端仍会有延迟。

### 模型五 单线程多路IO复用+多线程多路IO复用(进程池)

<img src="E:\Notes\Go\GoConcurrency.assets\image-20210729224633583.png" alt="image-20210729224633583" style="zoom:50%;" />

main process只是监听ListenFd状态，⼀旦触发读事件(有新连接请求). 通过⼀些IPC(进程间通信：如信 号、共享内存、管道)等, 让各⾃⼦进程Process竞争Accept完成链接建⽴，并各⾃监听。

多进程内存资源空间占⽤稍微⼤⼀些，多进程模型安全稳定型较强，这也是因为各⾃进程互不⼲扰的特点导致。

### 模型六 单线程多路IO复用+多线程多路IO复用+多线程读写业务

>单线程多路IO复用用于分发connFd，多线程多路IO复用用于分发业务，最后一个多线程用于处理读写业务

<img src="E:\Notes\Go\GoConcurrency.assets\image-20210729225801845.png" alt="image-20210729225801845" style="zoom:50%;" />

在模型五基础上，除了能够保证同时响应最⾼的并发数，⼜能够解决读写并⾏通道的局限问题。同⼀时刻的读写并⾏通道，达到了最⼤化极限， ⼀个客户端可以对应⼀个单独的执⾏流程处理读写业务，读写并⾏通道与客户端的数量1：1关系。

**该模型过于理想化**，一味要求CPU核⼼数数量⾜够⼤。如果硬件CPU数量可数，那么该模型就造成⼤量的CPU切换的成本浪费。因为为了保证读写并⾏通道和客户 端是1：1的关系，就要保证server开辟的thread的数量与客户端⼀致。

## 进程互斥同步基础知识

[操作系统 P23-P27](https://www.bilibili.com/video/BV1YE411D7nH?p=23)

### 生产者消费者模型

有一组生产者进程和一组消费者进程，生产者每生产一个物品就会放到缓冲区，消费者每消费一个物品就会从缓冲区去除一个物品。生产者消费者共享同一个初始为空，大小为n的缓冲区。

当缓冲区不为空时消费者才可以取出一个产品否则会阻塞，当缓冲区不满的时候才可以生产者才可以生产一个产品到缓冲区否则会阻塞。

缓冲区为一种临界资源，需要互斥访问。（当有两个生产者同时进行生产的时候，两者所指指针指向同一个位置，如果不互斥访问，就会出现一个生产者将另一个生产者的产品覆盖的问题。）

分析：上述情况有两种同步关系，一种互斥关系。

* 同步一：buffer不满 -> 生产者才可生产
* 同步二：buffer不空 -> 消费者才可消费
* 互斥：buffer互斥访问

因此使用三个信号量进行处理：

* mutex=1表示互斥信号量，用于缓冲区互斥，设置为1。
* prod_num=0表示同步信号量，表示产品数量。
* bf_num=n表示同步信号量，表示空白缓冲区的数量。

程序如下：

```c
int mutex = 1;
int prod_num = 0;
int bf_num = n;

void productor(){
    // product
    P(bf_num);
    P(mutex);
    // send to buffer
    V(mutex);
    V(prod_num);
}

void consumer(){
    // consume
    P(prod_num);
    P(mutex);
    // get from buffer
    V(mutex);
    V(bf_num);
}
```

**❗注意**：

* 互斥信号量不可以放在同步信号量的外部，如果放在外部，初始阶段当消费者经过P(mutex)和P(prod_num)阻塞并且调度后，生产者也会执行P(mutex)也会阻塞，就会造成死锁deadlock。
* 与同步和互斥不相干的代码要放在外部，否则会增多上锁的时间，导致并发度下降。

golang使用管道实现：

```go
func consumer(ch chan int, id int) {
	for {
		fmt.Printf("consume %v: %v\n",id, <-ch)	//get from buffer
		time.Sleep(time.Second * 4)
	}
}

func product(ch chan int){
	for {
		num := rand.Intn(50)
		a := time.Now()		// product
		ch <- num			// send to buffer
		lps := time.Since(a).Seconds()
		fmt.Println("product:", num)
		if int(lps) >= 1{
			fmt.Printf("block time:%.0f seconds\n", lps)
		}
		//time.Sleep(time.Second * 2)
	}
}

func TestConsumerAndProduct(t *testing.T) {
	rand.Seed(time.Now().UnixNano())
	ch := make(chan int, 3)
	go consumer(ch, 0)
	go consumer(ch, 1)
	go product(ch)
	select {}
}
```

### 读者写者模型

有读者和写者两组并发进程，共享一个文件。当两个及以上的读进程访问的时候不会冲突，但如果写进程和其他进程（读、写进程）同时访问共享数据的时候就会导致数据不一致的问题，因此有以下要求：

* 允许多个读进程同时对文件进行读操作。
* 同一时间只允许一个写进程对文件进行写操作。
* 任意一个写进程完成之前都不允许其他读、写进程进行操作。
* 写者操作执行操作前必须让已有的读、写者进程全部退出。

**方法：**

1. 关系分析。找出哥哥进程并且分析他们之间的同步和互斥关系。

   两类进程：读、写进程。互斥关系：写—写进程，写—读进程。读—读进程之间不存在互斥问题。

2. 整理思路。根据流程确定P、V操作的大致顺序。

3. 设置信号量，并设置初值。（互斥信号量一般为1，同步信号量要看资源的数目）

   读者和写者之间的任何进程都要进行互斥，设置一个互斥信号量rw，读者和写者访问共享文件前后都要进行P、V操作。**但是这无法解决读者进程之间同时访问共享文件的问题**

   可以设置一个`count`变量来记录当前读者进程访问共享文件的数目，如果不为0就无需进程PV操作。

代码1：

```c
semaphore rw = 1;
int count = 0;
semaphore mutex = 1;

void writer(){
    while(1){
        P(rw);
        // 写文件
        V(rw)
    }
}

void reader(){
    while(1){
        P(mutex);
        if(count==0)P(rw);	// 第一个读文件负责加锁
        count++;
        V(mutex);
        // 读文件
        P(mutex);
        count--;
        if(count==0)V(rw);	// 最后一个读文件进程解锁
        V(mutex);
    }
}
```

* 为什么要对`count`变量进行互斥访问？当两个读进程同时运行的时候，会同时进入if语句，都会执行P操作，会导致两个读进程无法同时进行读操作。究其原因是因为无法"**一气呵成**"，访问需要原子性。因此需要一个互斥变量来保证执行的原子性。
* 所存在的问题：当有源源不断的读程序到来的时候，会导致写进程一直阻塞等待甚至”饿死“的情况。

改进2：

```c
semaphore rw = 1;		// 读写互斥信号量
int count = 0;			// 读进程的数量
semaphore mutex = 1;	// 原子操作信号量
semaphore w = 1;		// "写优先"信号量

void writer(){
    while(1){
        P(w);
        P(rw);
        // 写文件
        V(rw);
        V(w);
    }
}

void reader(){
    while(1){
        P(w);
        P(mutex);
        if(count==0)P(rw);	// 第一个读文件负责加锁
        count++;
        V(mutex);
        V(w);
        // 读文件
        P(mutex);
        count--;
        if(count==0)V(rw);	// 最后一个读文件进程解锁
        V(mutex);
    }
}
```

* 如果是先来先服务的算法，表面上是实现的”写优先“，其实是一种比较公平的原则，也叫”读写公平法“，写进程也不会出现一直阻塞到饿死的情况。
* **核心思想**：`count`计数器的设置；原子操作需要使用互斥信号量；需要防止饥饿的问题。

### 哲学家进餐问题



## 锁

读的多，冲突几率小，乐观锁。
写的多，冲突几率大，悲观锁。

### 乐观锁

>乐观锁（Optimistic Concurrency Control）多数用于数据争用不大、冲突较少的环境中，这种环境中，偶尔回滚事务的成本会低于读取数据时锁定数据的成本，因此可以获得比其他并发控制方法更高的**吞吐量**。

乐观锁假设多用户并发的事务在处理时不会彼此互相影响，各事务能够在不产生锁的情况下处理各自影响的那部分数据。在提交数据更新之前，每个事务会先检查在该事务读取数据后，有没有其他事务又修改了该数据。如果其他事务有更新的话，正在提交的事务会进行回滚。

### 悲观锁

> 悲观锁可以阻止一个事务以影响其他用户的方式来修改数据。如果一个事务执行的操作读某行数据应用了锁，那只有当这个事务把锁释放，其他事务才能够执行与该锁冲突的操作。

悲观锁主要用于数据争用激烈的环境，以及发生并发冲突时使用锁保护数据的成本要低于回滚事务的成本的环境中

## Go高阶训练

### goroutine

#### 不开启不知道何时结束的goroutine

例如：

```go
func main(){
    mux := http.NewServerMux()
    mux.HandleFunc("/", func(resp http.ResponseWriter, req *http.Request){
        fmt.Fprintln(resp, "Hello world")
    })
    go http.ListenAndServer(":8081", http.DefaultServerMux)	// 不知道什么时候结束
    http.ListenAndServer(":8080", mux)
}
```

* 不知道创建的goroutine什么时候结束
* 如何防止其结束

### memory model

