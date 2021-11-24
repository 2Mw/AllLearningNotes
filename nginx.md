# Nginx笔记

[TOC]

[BV1ov41187bq](https://www.bilibili.com/video/BV1ov41187bq)  P109

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

## nginx配置

### 实战

需求：

```
(1) 有如下访问:
http://192.168.200.133:8081/server1/location1
访问的是: index_sr1_location1.htm1
http://192.168.200.133:8081/server1/location2
访问的是: index_sr1_location2.htm1
http://192.168.200.133:8082/server2/location1
访问的是: index_sr2_locationl.htm1
http://192.168.200.133:8082/server2/location2
访问的是: index_sr2_location2.htm1
(2) 如果访问的资源不存在，返回自定义的404页面
(3) 将/server1和/server2的配置使用不同的配置文件分割
将文件放到/home/www/conf.d目录下，然后使用include进行合并
(4)为/server1和/server2各自创建一个访问日志文件
```

nginx.conf

```
user www;
worker_processes  2;
error_log  logs/error.log;
pid        logs/nginx.pid;
daemon on;
events {
    worker_connections  1024;
    accept_mutex on;
    multi_accept on;
    use epoll;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    log_format server1 '===> server 1';
    log_format server2 '===> server 2';

    include /home/www/conf.d/*conf;

    server {
        listen       80;
        server_name  localhost;

        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
}
```

server1.conf

```
server{
        listen 8081;
        server_name localhost;
        access_log /home/www/myweb/server1/logs/access.log server1;
        location /server1/location1{
                root /home/www/myweb;
                index index_sr1_location1.html;
        }

        location /server1/location2{
                root /home/www/myweb;
                index index_sr1_location2.html;
        }
        error_page 404 /404.html;
        location = /404.html{
                root /home/www/myweb;
                index 404.html;
        }

}
```

server2.conf大致类似，此处省略

### 配置系统服务

1. 在`/usr/lib/systemd/system/nginx.service`

   ```sh
   vim /usr/lib/systemd/system/nginx.service
   ```

2. 写入以下配置

   ```
   [Unit]
   Description=nginx web service
   Document=http://nginx.org/en/docs
   After=network.target
   
   [Service]
   Type=forking
   PIDFILE=/usr/local/nginx/logs/nginx.pid
   ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
   ExecStart=/usr/local/nginx/sbin/nginx
   ExecReload=/usr/local/nginx/sbin/nginx -s reload
   ExecStop=/usr/local/nginx/sbin/nginx -s stop
   PrivateTmp=true
   
   [Install]
   WantedBy=default.target
   ```

3. 给文件添加权限

   ```sh
   chmod 755 /usr/lib/systemd/system/nginx.service
   ```

4. 使用命令操作nginx

   ```sh
   systemctl start nginx
   systemctl stop nginx
   # 重启
   systemctl restart nginx
   # 重新加载配置文件
   systemctl reload nginx
   # 查看状态
   systemctl status nginx
   # 开机启动
   systemctl enable nginx
   ```


### 配置环境变量

1. 配置`/etc/profile`

2. 添加nginx的二进制目录：

   `export PATH=$PATH:/usr/local/nginx/sbin`

3. 生效：`source /etc/profile`

## Nginx静态资源部署

我们需要考虑以下几个问题：

1. 静态资源的配置指令
2. 静态资源的配置优化
3. 静态资源的压缩配置
4. 静态资源的缓存处理
5. 静态资源的访问控制，包括跨域和防盗链的问题

### 配置指令

<h4>listen指令</h4>

参考：[Listen指令](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen)

语法：`	listen address[:port] [default_server];`

这里的default_server，如果不设置，表示如果host的名称没有匹配到对于的地址，第一个server块就会成为default_server；

位置server块

```
listen 127.0.0.1:8000;
listen 127.0.0.1;
listen 8000;
listen *:8000;
listen localhost:8000;
```

<h4>server_name指令</h4>

语法：`server_name name ...`

位置：server块

有三种匹配方式：

* 精确匹配
* 正则匹配：`server_name ~^www\.(\w+)\.com$`
* 通配符匹配：`server_name *.jd.com www.baidu.*`，不可以`www.*.cn  www.aaa.c*`

可以指定对应的域名，实验的时候可以通过`hosts`文件进行实验。

匹配执行顺序：1. 精确匹配 2. 通配符（前 -> 后）  3. 正则表达式

<h4>location指令</h4>

语法：`location [ 无 |= | ~ | ~* | ^~ | @] uri {...}`

🔵不带符号的（以指定模式开始的）：

```
location /abc {
	
}
```

能匹配的`/abc`开头的，比如：`/abc, /abcd /abcaaa`等

🔵`=`开始的(精确匹配)

`location=/abc`只能匹配`/abc`路径

🔵正则表达式匹配

`~`区分大小写，`~*`不区分大小写

比如：

```
location ~^/abc$
```

🔵`^~`匹配

用于不包含正则表达式的url前，如果模式匹配，就不再向后搜索

`location ^~/abc`

<h4>设置请求资源的目录（root和alias）</h4>

root指令：

语法：`root path`

位置：location，server，http

alias指令：

语法：`alias path`

位置：location

root指令是指定请求的根目录，然后在这个根目录下根据`location`指定的路径查找文件

alias指令相当于是一个为长的目录起一个别名，方便快捷访问。

<h4>index指令</h4>

来设置网站的默认首页

语法：`index index.html`

位置：location， server， http

<h4>error_page指令</h4>

语法：`error_page code ... [=[response]] uri;`

位置：location， server， http

1. 指定其具体跳转的页面

   ```
   error_page 404 http://www.baidu.com
   ```

2. 指定重定向地址

   ```
   error_page 404 500 /50x.html
   location /50x.html{
   	root html;
   }
   ```

3. 使用location的`@`符号

   ```
   error_page 404 @toerror
   location @toerror{
   	default_type text/plain;
   	return 404 "Not Found!";
   }
   ```

4. `[=response]`的使用

   将原来的状态码改成另一个状态码，这里为`200`

   ```
   error_page 404 =200 /50x.html
   location /50x.html{
   	root html;
   }
   ```

### 配置优化

可以从三个方面进行优化：

```
sendfile on;
tcp_nopush on;
tcp_nodeplay on;
```

1. `sendfile` 开启高效的文件传输模式

   语法：`sendfile on | off`，默认`off`

   位置：location， server， http

   开启此命令之后，可以减少系统内核态的切换和内存拷贝，减少系统开销。

2. `tcp_nopush`，必须在`sendfile`开启之后才会生效，用于提高网络传输的效率

   语法：`tcp_nopush on | off` 默认`off`

   位置：location， server， http

   在tcp传输的过程中开辟一个缓冲区，等缓冲区存满的时候开始传输数据。和`tcp_nodelay`互斥。

3. `tcp_nodelay`，必须是在`keep-alive`开启后才会生效，用于提高网络传输的实时性

   语法：`tcp_nodelay on | off` 默认`on`

   位置：location， server， http

   当在数据传输的最后一个数据包的时候，nginx会忽略`tcp_nopush`这个参数，会直接将数据发送出去，因此建议这两个指令同时开启，提高网络的传输效率。

### 压缩配置

在nginx中可以使用gzip来对静态资源进行压缩，提高网络传输速度。有三个压缩模块

```
ngx_http_gzip_module模块
ngx_http_gzip_static_module模块
ngx_http_gunzip_module模块
```

后两个模块需要安装才能够使用

存在的问题：

```
1.gzip各模块支持的配置命令
2.gzip压缩功能的配置
3.gzip和sendfile冲突的解决
4.浏览器不支持gzip的解决方案
```

<h4>压缩配置指令</h4>

参考：[ngx_http_gzip_module](http://nginx.org/en/docs/http/ngx_http_gzip_module.html)

🔵gzip指令

语法：`gzip on | off ` 默认`off`

位置：location， server， http

🔵gzip_types指令

对于指定的`MIME-Types`的文件类型进行压缩

语法：`gzip_types mime-types` 默认：`text/html`

位置：location， server， http

```
gzip on;
gzip_types text/html application/javascript;
```

🔵gzip_comp_level指令

语法：`gzip_comp_level level;` 默认1，1表示压缩程度最低

🔵gzip_vary指令

语法：`gzip_vary on | off` 默认 off

即告诉对方是否使用的gzip压缩头：`Vary: Accept-Encoding`

🔵gzip_proxied指令

语法：`gzip proxied off | expired | no-cached | auth | any ...`

设置是否对服务器端返回的结果进行压缩

off作为反向代理服务器不会对数据进行压缩

🔵其他指令

`gzip_diasble "Mozila 5.0.*"`根据不同的客户端`User-Agent`来决定是否开始gzip压缩，兼容低版本的浏览器。

`gzip_min_length`针对传输数据大小来决定是否开启压缩，只要比某个值小，就不会进行压缩。

<h4>gzip和sendfile冲突的解决</h4>

`sendfile`命令可以减少静态资源在计算机中复制的次数，而`gzip`指令需要将静态资源先压缩，再进行发送就需要进行多次的复制和传输，与其产生了冲突。

因此使用`ngx_http_gzip_static_module`，再处理静态资源的时候预先压缩静态文件，

添加模块：

```sh
nginx -V
mv nginx nginx_old # 备份
cd /home/2mw/Downloads/nginx/ # 进入源代码目录
make clean
./configure --with-http_gzip_static_module	# 添加模块
make	# 重新编译
cp objs/nginx /usr/local/nginx/sbin/ # 将编译后的nginx重新复制到sbin目录下
make upgrade
```

在`nginx.conf`中设置`gzip_static on;`

这个模块会自动寻找同名文件下的`.gz`压缩的对应文件，因此需要对想要压缩的文件进行压缩。

```sh
gzip -9 jquery.js
```

### 缓存处理

如果服务端的网页没有发生变化，就不再请求服务器而是从缓存中读取数据，加快加载速度。

HTTP协议中和页面缓存相关的字段

| header        | 说明                              |
| :------------ | :-------------------------------- |
| Expires       | 缓存过期的日期和时间              |
| Cache-Control | 设置缓存相关的配置信息            |
| Last-Modified | 请求资源上传修改的时间            |
| Etag          | 请求变量的实体标签值，比如文件MD5 |

判断缓存流程，分为**强缓存**和**弱缓存**

![image-20211117215000300](https://i.loli.net/2021/11/17/pFXIy2DS5hGYJBQ.png)

<h4>expires指令</h4>

语法：`expires (time | epoch | echo | max | off)`，默认`off`

如果time设置为负数，则设置`Cache-Control:no-cache`不缓存，否则设置`Cache-Control: max-age=time`

设置max为10年，off为默认不缓存

举例：设置为10天

```
location ~ .*\.(png|js|img|jpg){
	expires 10d;
}
```

<h4>add_header指令</h4>

语法：`add_header name value [always]`

位置：http，server，location

```
location ~ .*\.(png|js|img|jpg){
	add_header Cache-Control no-cache;
}
```

### 跨域 防盗链

同源策略：协议、域名、端口全部相同即为同源。

添加两个头信息：

```
location =/get {
	add_header Access-Control-Allow-Origin: http://localhost;
	add_header Access-Control-Allow-Method: POST,PUT,GET,DELETE;
}
```

防盗链

根据相关头信息`Referer`来判定

<h4>valid_referers指令</h4>

语法：`valid_referers none|blocked|server_names|string...`

位置：server, location

none: 如果referer为空，允许访问

blocked：referer不为空，可能被防火墙或者代理伪装过，表示不带`http`或者`https`开始的网址。

server_names：表示指定的域名或者IP

string：表示匹配的正则表达式，需要以`~`开头

如果匹配到就会将`$invalid_referer`变量置为0，没有匹配到就会置为1.

```
location ~ .*\.(png|js|img|jpg){
	valid_referers none blocked www.baidu.com;
	if($invalid_referer){	# 未匹配到
		return 403;
	}
}
```

> 但是这种方式防不了真正的程序员，只需要伪造一个Referer请求头即可。

## Rewrite功能

Rewrite功能依赖于`PCRE`正则表达式库，主要用于URL的重写。

Rewrite相关命令：`set,if,break,return,rewrite,rewrite_log`

应用场景：域名跳转、域名镜像、独立域名、目录自动加`/`，合并目录，防盗链的实现

### 相关指令

<h4>set指令</h4>

该指令用来设置一个新的变量。

语法：`set $var value`

位置：server、location、if

```
server{
    listen 8080;
    server_name localhost;
    location /server{
        set $name TOMMY;
        set $age 20;
        default_type text/plain;
        return 200 $name=$age;
    }
}
```

注意不要与nginx内置的变量相覆盖。比如`$args $http_user_agent $host $document_uri $http_cookie $remote_addr $remote_port $remote_user $server_addr $request_method $request_uri`，可以配合`log_format`指令一起使用。

<h4>if指令</h4>

语法：`if (condtion) {}` 注意if后面要有一个空格` ' '`.

位置：server，location

> 注意`=`或者`!=`判断符前后**必须**要有空格。

```
location /testif{
    set $name 'Liasa';
    default_type text/plain;
    if ($name = 'Lisa'){
        return 200 'Yes, Lisa!';
    }
    return 404 'not lisa';
}
```

使用正则表达式：

`~`区分大小写，`~*`不区分大小写

```
if ($http_user_agent ~ Chrome){
    return 200 'Yes, Chrome!';
}
```

判断文件是否存在`-f`：

```
if (!-f $request_filename){
    return 200 'File not found!';
}
```

其他：`-d`目录是否存在 `-e`目录或者文件是否存在 `-X`判断文件是否可执行。

<h4>break指令</h4>

可以用于中断当前作用域种其他的配置，位于它前面的配置生效，后面的配置失效。**并且**中断当前URL的匹配，重定向到location目录下找对应的文件。

位置：server、location、if

<h4>return指令</h4>

语法：`return code [text | url]`或者`return url`

位置：server、location、if

<h4>rewrite指令</h4>

语法：`rewrite regex new_url [flag]`

regex为匹配的正则表达式，new_url匹配成功后用于替换被截取内容的字符串。如果字符串是以`http(s)://`开头的，则不会向下对URL进行其他处理，直接返回重写后的URL。

```
location /rewrite{
    default_type text/plain;
    rewrite ^/rewrite/url\w*$ https://www.qaqaqqa.asia;
    rewrite ^/rewrite/(demo)\w*$ /$1;
}
```

flag: 

* `last`：在其他的location块中寻找
* `break`：不去location块中寻找，而是去`/www/share/html`目录下寻找
* `redirect`：302并且重写URL返回，使用在不是以`http(s)://`开头的情况，会将页面的url改变。
* `permanent`：301使用在不是以`http(s)://`开头的情况，会将页面的url改变。

<h4>rewrite_log指令</h4>

语法：`rewrite_log on | off` 默认 off

开启需要将`error_log`级别设置为notice级别。

### 实际操作

<h4>域名跳转</h4>

```
location /rewrite{
    rewrite ^(.*) https://www.baidu.com$1;
}
```

这个可以将url后面的请求路径也可以带过去

<h4>独立域名</h4>

为每一个模块设置一个独立子域名。

```
server{
	listen 81;
	server_name a.jd.com;
	rewrite ^(.*) http://www.itcast.cn/a$1;
}
server{
	listen 82;
	server_name name.jd.com;
	rewrite ^(.*) http://www.itcast.cn/name$1;
}
```

<h4>目录自动加'/'</h4>

> 现在版本的nginx无需此操作

如果在地址栏中不加最后的`/`，nginx会自动返回301并且添加`/`来访问目录下的index。

```
server{
	listen 82;
	server_name localhost;
	location /get {
		if (-d $request_filename){
			rewrite ^(.*)([^/])$  http://$host:$server_port$1$2/ permanent;
		}
	}
}
```

<h4>合并目录</h4>

比如要访问`www.jd.com/11/22/33/44/55/a.html`

优化可以使用`alias`命令或者`rewrite`命令

```
server{
	listen 82;
	server_name localhost;
	location /get {
		rewrite ^/get-([0-9]+)-([0-9]+)-([0-9]+)-([0-9]+)-([0-9]+)-(\w+)\.html$ /server/$1/$2/$3/$4/$5/$6.html last;
	}
}
```

## 反向代理

都是由`nginx_http_proxy_module`解析。

### 配置语法

<h4>proxy_pass指令</h4>

用来被设置成被代理服务器，

语法：`proxy_pass URL`

位置：location

```
location / {
	# proxy_pass http://192.168.1.102/;
	proxy_pass http://192.168.1.102;
}

location /server {
	# proxy_pass http://192.168.1.102/;
	proxy_pass http://192.168.1.102;
}
```

在proxy_pass的URL中是否加`/`?

如果不加的话，nginx就会将`/server`拼接到代理的URL后面，如果加的话就不拼接。

<h4>proxy_set_header指令</h4>

这个指令更改nginx服务器接受到的客户端请求头信息，然后将新的请求头信息发给被代理服务器。可以用于将客户的真实地址发送给服务器端。

语法：`proxy_set_header field value`

默认值：`proxy_set_header Connection close`

位置：http，server，location

被代理服务器的配置（用来观察）：

```
server{
	listen 8080;
	server_name localhost;
	default_type text/plain;
	return 200 $http_username;
}
```

代理服务器：

```
server{
	listen 8080;
	server_name localhost;
	location /server{
		proxy_pass http://192.168.1.102:8080/;
		proxy_set_header username TOM;
	}
}
```

<h4>proxy_redirect指令</h4>

用来重置头信息中的`Redirect`和`Refresh`值。

语法：`proxy_redirect [ redirect replacement | default | off]` 默认default

用来防止暴露服务器的地址。

### SSL安全控制

默认nginx没有添加`nginx_http_ssl_module`模块，因此需要重新安装并且配置。

<h4>证书生成</h4>

方式一：阿里云腾讯云

方式二：自签证书

```
openssl genrsa -des3 -out server.key 1024
openssl req -new -key server.key -out server.csr
cp server.key server.key.org
openssl rsa -in server.key.org -out server.key
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

<h4>ssl指令</h4>

开启ssl

语法：`ssl on | off`默认off

也可以使用`listen 443 ssl`打开。

<h4>证书指令</h4>

* `ssl_certificate file` 指定ssl证书文件路径
* `ssl_certificate_key file` 指定密钥的路径
* `ssl_session_cache off | none |builtin | [shared:name:size] ` 配置ssl缓存，off禁用，builtin为内置ssl缓存，尽在一个工作进程中使用；shared所有工作进程都适用。
* `ssl_session_timeout 5m`：设置客户端能在缓存中反复使用的会话参数时间。
* `ssl_ciphers passwd`：指出允许的密码，为Openssl格式

实例配置：

```
server{
	listen 443 ssl;
	listen [::]:443 ssl; # ipv6
	
	ssl_certificate server.cert;
	ssl_certificate_key server.key;
	ssl_session_cache shared:SSL:1m;
	ssl_session_timeout 5m;
	
	ssl_ciphers HIGH:!aNULL:!MD5;
	ssl_prefer_server_ciphers on;
	
	location / {
		root html;
		index index.html;
	}
}
```

### http自动转https

```
server{
	listen 80;
	listen [::]:80;
	server_name www.aaa.com;
	location / {
		rewrite ^(.*) https://www.aaa.com$1;
	}
}
```

### 反向代理系统优化

<h4>proxy_buffering指令</h4>

用于开启或者关闭代理服务器的缓冲区。

语法：`proxy_buffering on | off` 默认 on

<h4>proxy_buffers指令</h4>

指定单个连接从代理服务器中读取的缓冲区的个数和大小。

语法：`proxy_buffers number size`默认 `8 4k | 8k`大小由操作系统配置决定

numer表示缓冲区的数目，size表示每个缓冲区的大小

## 负载均衡

较为过时的方法时手动选择与DNS轮询，手动选择比较原始，比如各个下载站各个下载链接。DNS轮询的方式可以实现简单的负载均衡，成本较低，不太可靠。

现在大多数为四层/七层负载均衡，这里四层七层指的是OSI网络模型中的传输层和应用层。

实现四层负载均衡的方式（传输层）主要基于IP+Port的方式负载均衡：

```
硬件：F5,Big-IP,RadWare等较为昂贵
软件：LVS，Nginx，Hayproxy
```

实现七层负载均衡的方式，主要基于虚拟URL和主机IP负载均衡：

```
软件：Nginx等
```

四层效率较高。

### 七层负载均衡

nginx要实现七层负载均衡需要使用到`proxy_pass`代理模块的配置。Nginx的负载均衡是在反向代理的基础上进行的，将用户的请求根据指定的算法分发到一组`Upstream虚拟服务池`中进行处理。

<h4>upstream指令</h4>

用于定义一组服务器，他们可以监听不同的端口服务器，也可以是同时监听TCP和Unix Socket服务器，并且服务器之间可以指定不同的权重，默认为1.

语法：`upstream name {...}`

位置：http

<h4>server指令</h4>

这个指令和之前学习的指令是不同的，可以使用域名、IP、端口或者是Unix Socket

语法：`server name [params]`

位置：🚩**upstream**

![image-20211123185857332](https://i.loli.net/2021/11/23/NLdxfyG5ChtJ41e.png)

由于没有过多的设备，这里为了演示方便，因此使用不同的端口来代表不同的服务器。

设置三个服务器：

```
server{
    listen 9001;
    server_name localhost;
    default_type text/plain;
    return 200 9001port;
}

server{
    listen 9002;
    server_name localhost;
    default_type text/plain;
    return 200 9002port;
}

server{
    listen 9003;
    server_name localhost;
    default_type text/plain;
    return 200 9003port;
}
```

设置负载均衡服务器：

需要在upstream中指定所有需要负载均衡的服务器，这里将服务名称设置为`backend`，然后在`proxy_pass`中指定`http://`+`服务名称`

```
upstream backend {
    server 192.168.230.138:9001;
    server 192.168.230.138:9002;
    server 192.168.230.138:9003;
}

server{
    listen 8000;
    server_name localhost;
    location / {
        proxy_pass http://backend;
    }
}
```

这种方式是以**轮询**的方式进行负载均衡的。

### 负载均衡状态

| 状态         | 描述                                  |
| ------------ | ------------------------------------- |
| down         | 表示当前的服务器不参与负载均衡        |
| backup       | 预留的备份服务器                      |
| max_fails    | 允许失败的请求次数                    |
| fail_timeout | 经过max_fails失败次数，服务暂停的时间 |
| max_conns    | 限制最大的接受连接数                  |

```
upstream backend {
    server 192.168.230.138:9001 down;
    server 192.168.230.138:9002 backup;
    server 192.168.230.138:9003 max_fails=3 fail_timeout=15;
}
```

down一般用于需要停机维护的服务器。

backup用于其他服务器都不能使用的时候，才启用这个服务器。

### 负载均衡策略

| 算法       | 说明             |
| ---------- | ---------------- |
| 轮询       | 默认方式，轮着来 |
| weight     | 权重的方式       |
| ip_hash    | 依据IP分配方式   |
| least_conn | 依据最少连接方式 |
| url_hash   | 依据URL分配方式  |
| fair       | 依据响应时间方式 |

* 轮询即为默认的策略，不需要做任何其他配置，默认的加权轮询`weight=1`。

* `weight`权重方式：

    ```
    upstream backend {
        ip_hash;
        server 192.168.230.138:9001;
        server 192.168.230.138:9002;
        server 192.168.230.138:9003;
    }
    ```

* `ip_hash` 可以用来解决各个服务器之间session不共享的问题，根据用户IP来计算得到对应的服务器。但是这个并不能进行很好的负载均衡，甚至可能会导致一些服务器过忙，而另一些过于空闲。

* `least_conn` 最少连接算法，将**当前请求数目最少**的服务器分配给用户。可以用在各个服务器处理时间长短不要的情况下

    ```
    upstream backend {
        least_conn;
        server 192.168.230.138:9001;
        server 192.168.230.138:9002;
        server 192.168.230.138:9003;
    }
    ```

    

* `url_hash`根据客户端url的hash结果，如果用户的url与上次相同，则将用户还是分配给上一次的服务器IP，如果不同则换一个服务器。需要配合**缓存命中**来使用，比如文件系统的使用。

    ```
    upstream backend {
        hash &request_uri;
        server 192.168.230.138:9001;
        server 192.168.230.138:9002;
        server 192.168.230.138:9003;
    }
    ```

* `fair`采用的不是内建负载均衡使用的轮换算法，而是可以根据页面大小、加载时长来进行智能负载均衡。但是nginx并不默认支持这种方式，需要添加第三方模块。

    下载：

    ```sh
    wget https://github.com/gnosek/nginx-upstream-fair
    unzip nginx-upstream-fair.zip
    mv nginx-upstream-fair fair
    ./configure --add-module=/root/fair	# 使用nginx源码中的configure，添加指定fair的目录
    # 在nginx源码中src/http/ngx_http_upstream.h文件中找到ngx_http_upstream_srv_conf_s
    # 在对应的结构体中port变量下添加一行
    # in_port_t default_port;
    # 否则会导致编译错误
    make
    cp objs/nginx /usr/local/bin/nginx/sbin/
    make upgrade	# 进行平滑升级
    ```

    添加配置：

    ```
    upstream backend {
        fair;
        server 192.168.230.138:9001;
        server 192.168.230.138:9002;
        server 192.168.230.138:9003;
    }
    ```

    