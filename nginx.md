# Nginx笔记

[TOC]

[BV1ov41187bq](https://www.bilibili.com/video/BV1ov41187bq)  P32

## 初始nginx

### 准备

关闭防火墙：

```sh
systemctl stop firewalld	# 关闭运行的防火墙，相同重启仍会启用
systemctl disable firewalld	# 彻底关闭防火墙
systemctl status firewalld	# 参考状态
```

关闭selinux

```sh
sestatus		# 参考状态
vim /etc/selinux/config  # 将selinux改为disabled
```

安装GCC编译器

```sh
yum install -y gcc
gcc --version
```

安装PCRE（兼容正则表达式库）

```sh
yum install -y pcre pcre-devel	# 安装库及源码
rpm -qa pcre pcre-devel	# 参考是否安装成功
```

安装zlib

```sh
yum install -y zlib zlib-devel
rpm -qa zlib zlib-devel
```

安装openssl

```sh
yum install -y openssl openssl-devel
```

### 安装

方法：源码安装——简单安装和复杂安装；yum安装。

🔵简单安装

将压缩包解压到对应目录

```sh
./configure		# 根据相同环境生成makefile文件和C语言代码
make			# 编译
make install	# 安装
cd /usr/local/nginx/sbin	# 进入安装目录
./nginx -s stop	# 停止nginx
```

🔵yum安装

> 简单安装需要做很多前置准备，而yum安装不需要那么多的步骤

参考：[nginx: CentOS install](http://nginx.org/en/linux_packages.html#RHEL-CentOS)

```sh
sudo yum install yum-utils
```

创建文件`/etc/yum.repos.d/nginx.repo`

```ini
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

安装nginx：

```sh
sudo yum install nginx
```

yum安装和简易安装的差异：yum安装的configure argument参数很多

🔵复杂安装（了解`configure`程序的参数）

参考：[nginx编译安装之-./configure 参数详解](https://www.cnblogs.com/flashfish/p/11025961.html)

指令大致分为三类：

* PATH类，用于配置路径

* --with--xxx类，用于添加模块
* --without--xxx类，用于排除某些模块

`--prefix=PATH`，指定nginx的安装路径，默认`/usr/local/nginx`

`--sbin-path=PATH`，nginx二进制文件的目录，默认为`<prefix>/sbin/nginx`

`--modules-path=PATH`，动态模块的安装目录`<prefix>/modules`

`--conf-path=PATH`，配置文件nginx.conf的安装目录`<prefix>/conf/nginx.conf`

`--error-log-path=PATH`，错误日志文件路径`<prefix>/logs/error.log`

`--http-log-path=PATH`，访问日志文件路径`<prefix>/logs/access.log`

`--pid-path=PATH`，nginx启动后的ID文件路径`<prefix>/logs/nginx.pid`

`--lock-path=PATH`，nginx锁文件路径`<prefix>/logs/nginx.lock`

### nginx目录结构

```shell
[root@192 nginx]$ tree /usr/local/nginx/
/usr/local/nginx/
├── client_body_temp
├── conf
	  # cgi通用网关接口 fastcgi, scgi, uwsgi
│   ├── fastcgi.conf
│   ├── fastcgi.conf.default
│   ├── fastcgi_params
│   ├── fastcgi_params.default
│   ├── scgi_params
│   ├── scgi_params.default
│   ├── uwsgi_params
│   ├── uwsgi_params.default
	  # 编码转换相关配置文件
│   ├── koi-utf
│   ├── koi-win
│   ├── win-utf
	  # 处理HTML数据类型的文件
│   ├── mime.types
│   ├── mime.types.default
	  # nginx重要的配置文件
│   ├── nginx.conf
│   └── nginx.conf.default
├── fastcgi_temp
├── scgi_temp
└── uwsgi_temp
├── html
│   ├── 50x.html
│   └── index.html
├── logs
│   ├── access.log  # 访问日志
│   ├── error.log	# 错误日志
│   └── nginx.pid	# nginx进程号
└── sbin
    └── nginx		# nginx二进制文件
```

CGI(Common Gateway Interface)：通用网关接口，解决从客户端发送请求，服务器端调用CGI程序解决问题。

KOI文件：编码转换相关配置文件

### Nginx开启停止

1. nginx的服务信号控制

   查看nginx的master和worker进程`ps -ef | grep `

   <img src="E:\Notes\nginx\nginx.assets\image-20210914000625498.png" alt="image-20210914000625498" style="zoom: 67%;" />

   nginx的信号：

   | 信号       | 作用                                                 |
   | ---------- | ---------------------------------------------------- |
   | TERM / INT | 立即关闭服务                                         |
   | QUIT       | “优雅”关闭服务，不再处理新请求                       |
   | HUP        | 重新配置文件并且使服务对新配置生效                   |
   | USR1       | 重新打开日志文件，可以用来日志切割                   |
   | USR2       | 平滑升级到最新nginx                                  |
   | WINCH      | 所有子进程不再处理新连接，相当于给worker下达QUIT指令 |

   调用命令：`kill -TERM pid`

   **具体何平滑升级nginx：**

   当执行`kill -USR2 PID`之后，nginx升级之后会新产生nginx的master和worker进程，并且在对应的`logs`目录下生成`nginx.pid.oldbin`记录旧的nginx进程id，`nginx.pid`为新的nginx进程id，再进行命令`kill - QUIT pid`结束原先的旧nginx程序，让其停止处理新请求。

2. 使用命令行来控制nginx（较常用）

   ```sh
   [root@192 sbin]# ./nginx -?
   nginx version: nginx/1.20.1
   Usage: nginx [-?hvVtTq] [-s signal] [-p prefix]
                [-e filename] [-c filename] [-g directives]
   
   Options:
     -?,-h         : this help
     -v            : show version and exit
     -V            : show version and configure options then exit
     -t            : 检测nginx.conf 配置文件语法是否正确
     -T            : test configuration, dump it and exit
     -q            : suppress non-error messages during configuration testing
     -s signal     : send signal to a master process: stop, quit, reopen, reload
     -p prefix     : set prefix path (default: /usr/local/nginx/)
     -e filename   : set error log file (default: logs/error.log)
     -c filename   : 设置nginx的配置文件 (default: conf/nginx.conf)
     -g directives : set global directives out of configuration file
   ```

   `-V`参数：

   ```
   [root@192 sbin]# ./nginx -V
   nginx version: nginx/1.20.1
   built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
   configure arguments: --prefix=/usr/local/nginx
   ```

   `-s`参数：

   ```sh
   ./nginx -s stop # 类似TERM INT信号
   ./nginx -s quit # QUIT
   ./nginx -s reopen # 类似USR1
   ./nginx -s reload	# HUP，修改配置文件后重启
   ```


## Nginx.conf目录结构

内容：

> 配置文件默认三大块：全局块、events块、http块。http块有可以有多个server块，server块可以有

```conf

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

### 全局块指令

<h4>User指令</h4>

```sh
user user [group]
# 默认值 nobody， 用于指定worker进程的用户角色
user www
```

举例：

```sh
useradd www
./nginx -t			# 测试配置文件是否有语法问题
./nginx -s reload	# 重新加载配置文件并且重启
[root@localhost conf]# ps -ef | grep nginx
root       8705      1  0 00:57 ?        00:00:00 nginx: master process ../sbin/nginx
www        8706   8705  0 00:57 ?        00:00:00 nginx: worker process
root       8708   8492  0 00:57 pts/1    00:00:00 grep --color=auto nginx
```

使用user指令指定nginx可以访问的文件，可以更加细分用户类型，访问更加安全。

<h4>work process指令</h4>

> 用于指定是否开启工作进程

语法：`master_process on | off`，默认是`on`

语法：`worker_processes num/auto`，默认值是`1`。

`worker_processes`是用于配置Nginx生成的工作进程的数目，这个nginx服务器实现并发的关键所在，建议这个值与CPU的内核数一致。

<h4>其他指令</h4>

daemon: 设置nginx是否以守护进程的方式启动

语法：`daemon on | off`，默认`on`

pid指令：`pid logs/nginx.pid`

error_log指令：

语法：`error_log file [日志级别]`，也可以配置到全局块、http、server、location块

日志级别：`debug|info|notice|warn|error|crit|alert|emerg`，不建议设置成info级别以下。

include指令：

语法：`include nginx_b.conf`，可以引用其他的配置文件，更加灵活

### Events块指令

> 主要用于设置nginx服务器与用户的连接，对nginx服务器性能影响较大

<h4>accept_mutex指令</h4>

用来设置nginx网络连接的序列化

语法：`accept_mutex on | off`，默认`on`

主要解决的是“[惊群](https://zh.wikipedia.org/wiki/%E6%83%8A%E7%BE%A4%E9%97%AE%E9%A2%98)”问题，当许多进程等待请求，请求进入后所有worker进程被唤醒，但只有一个进程能获得CPU执行权，其他进程又得被阻塞，这造成了严重的系统上下文切换代价。

设置为on之后，每次请求只会一个一个激活。

<h4>multi_accept指令</h4>

用于设置是否允许工作进程同时接受多个网络连接

语法：`multi_accept on | off`，默认`off`

off时一个工作进程同时只能接受一个新的连接，**建议打开**，效率较高。

<h4>worker_connections指令</h4>

用于配置单个worker进程最大的连接数

语法：`worker_connections number`，默认`512`

这里的连接数不仅包括和前端用户建立的连接数目，而是包括所有可能的连接数。number数值**不可以**超过操作系统支持打开的最大句柄数目。

<h4>use指令</h4>

> 这也是nginx优化的重要部分，底层采用的是IO多路复用模型

用来设置nginx服务器选择使用哪种事件驱动来处理网络信息。

语法：`use method`，method可选值为：`select | epoll | poll | kqueue`，默认值由操作系统决定。

<h4>实例</h4>

```
events{
	accept_mutex on;
	multi_accept on;
	worker_connections 1024;
	use epoll;
}
```

进行测试：

```sh
./nginx -t
./nginx -s reload
```

### Http块的指令

<h4>MIME-TYPE</h4>

nginx支持显示不同资源的类型

默认指令：

```
include mime.types
default_type application/octet-stream
```

使用的范围可以是http, location, server

```
location /get_text {
            default_type application/json;
            return 200 "{'username': 'Tom'}";
            # default_type text/html; text/plain;
        }
```

<h4>自定义服务日志</h4>

nginx日志类别分为==access.log==和==error.log==

access.log用于记录用户访问的所有请求

指令分为：`access_log`和`log_format`

access_log指令：

语法：`access_log path [format[buffer=size]]`，path表示日志路径，format表示日志格式，buffer表示缓冲区大小。

可以在http，server，location块中使用

log_format指令：

语法：`log_format name '$http_user_agent'`

使用变量：

* `$remote_addr` 用户IP
* `$status`：请求状态码
* `$time_local`：请求时间

只能在http块中使用

<h4>其他指令</h4>

sendfile指令：

语法：`sendfile on | off` 默认`off`，可以提高nginx处理静态资源的性能

可以在http，server，location块中使用

keepalive_timeout指令：

语法：`keepalive_timeout time`默认`75s`，客户端向服务器端发送多个请求，每个请求都需要创建一次连接，较多请求的话就会导致处理效率较低，因此需要长连接保持连接提升效率，不需要重新创建新的连接，但是长连接时长也需要斟酌。

可以在http，server，location块中使用

keepalive_requests指令：

语法：`keepalive_requests number`，默认`100`,用来设置一个`keep-alive`连接使用的次数。

可以在http，server，location块中使用

### Server块

```
server{
	listen 80;
	server_name localhost;
	location / {	# 映射根目录
		root html;
		index index.html index.htm
	}
}
```

> 对于可以在http，server，location块中都可以使用的命令，那个会生效？
>
> > >>这个是按照“就近原则”处理的。在本块中有，就不会在父级中生效，互不交叉。
