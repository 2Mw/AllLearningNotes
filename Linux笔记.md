# Linux Notes

[TOC]

## 权限与目录配置

### 用户及用户组

用户的相关信息记录在`/etc/passwd`内

个人密码记录在`/etc/shadow`文件内

Linux所有用户组的信息记录在`/etc/group`内

### 文件权限符

举例：
$$
\text{[d]\quad [rwx]\quad [rwx]\quad [rwx]}
$$
第一个字符表示文件类型：`d`表示目录，`-`表示文件，`l`表示链接，`b`为读写设备，`c`为串行设备（鼠标键盘）

`rwx`分别代表读 写 执行，第一组表示文件拥有者的权限，第二组表示该用户组的权限，第三组表示其他人的权限。

`drwxr-xr-x.		2	root	root	238	Sep 13 08:04	yum.repos.d`

第二栏表示有多少文件名链接到此节点

第三栏表示文件拥有者

第四栏表示拥有者所在用户组（一个账号可以加入多个组）

第五栏表示文件大小，单位Bytes

第六栏表示最近修改时间

第七栏文件名，以`.`开头的文件为隐藏文件

`[r-x]`可以进入此目录并且读取，但是不能写入

`[r--]`对于文件可以读；对于文件夹虽然有`r`但是无`x`，不可以进入此文件夹。

对于机密文件比如`/etc/shadow`文件夹修饰符是`[----------]`。

`[rws]`这里`s`是什么？

### 修改权限和文件属性

> 三个主要命令：`chgrp`, `chown`,`chmod`

如果想修改该文件夹以及文件夹下的所有文件，加上`-R`(大写)即可

1. 修改所属用户组

   `chgrp root a.txt`

   `change root -R folder`

2. 修改文件拥有者

   `chown jack a.txt`

   `chown nick:root a.txt`，也可以指定用户组，使用`:`隔开

3. 修改文件权限

   ```
   r: 4
   w: 2
   x: 1
   rwx = 4 + 2 + 1 = 7， 以此类推
   ```

   `chmod 744 a.txt`

   `chmod -R 755 folder`，修改其以及文件夹下的所有文件

### 目录配置

> Filesystem Hierarchy Standard，[FHS文档](https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.pdf)

这里介绍主要的几个目录

* `/bin`：放置常用的执行文件的目录，例如指令`cp mv ls bash`等
* `/boot`：放置linux内核文件的启动目录
* `/dev`：linux设备的目录
* `/etc`：当值Linux中几乎所有的配置文件
* `/media`：防止可删除的设备，软盘光盘DVD
* `/mnt`：挂载的额外设备，比如硬盘
* `/opt`：放置第三方辅助软件
* `/tmp`：放置临时文件的目录
* `/usr`：全称UNIX Software Resource，表示软件资源存放的目录，非`user`
  * `/usr/bin`：一般用户都能使用的命令放在这里，且不应该有子目录
  * `/usr/local`：系统管理员自身下载的软件
  * `/usr/share`：主要放置只读的数据文件，`man`为在线帮助文档，`doc`软件说明文档，`zoneinfo`时区文件
  * `/usr/include`：C/C++头文件和include文件
  * `/usr/src`：源代码放置的位置
* `/var`是系统运行后渐渐占用硬盘的目录
  * `/var/cache`：程序运行中的缓存
  * `/var/lib`：程序运行过程中数据文件放置的目录，比如mysql的数据文件`/var/lib/mysql`
  * `/var/log`：日志文件
  * `/var/run`：程序运行过程后，会将其PID放置到这个目录下

### 查看系统信息

```sh
uname -a
```

## 文件与目录

### 文件操作

文件夹操作命令：`pwd`查看当前目录，`mkdir` `rmdir`

查看目录`ls`:

* `-a`，列出全部文件
* `-d`，只列出目录
* `-h`，文件大小以KB，GB形式展现
* `-l`，列出详细信息`ll`
* `-R`，显示子目录文件
* `-r`按文件名排序 `-S`按文件容量排序 `-t`按时间排序

文件操作命令：复制`cp`、删除`rm`、移动或者重命名`mv`，操作目录需要加`-r`，`touch`

获取文件名和目录名称：

```sh
echo $(pwd)/a.txt
> /opt/rh/a.txt
basename $(pwd)/a.txt
> a.txt
dirname $(pwd)/a.txt
> /opt/rh
```

### 读取文件

* `cat`查看文件内容，`tac`是其倒序输出
* `nl` 显示行号输出内容
* `more`一页一页显示内容，`less`类似但是可以向前翻页
* `head` `tail`可以看前（后）几行，`-n`指定行数
* `od`使用二进制的方式读取文件内容

### 配置文件隐藏属性

查看文件权限`umask [-S]`

配置文件隐藏属性`chattr [+-=][ASacdistu]`：参考[chattr](https://en.wikipedia.org/wiki/Chattr)较为常用的有：

* `a`添加后文件只能增加数据，不能输出不能修改旧数据，可以用于日志
* `i`让文件不能删除、改名，无法写入和修改

查看文件隐藏属性`lsattr`

### 文件特殊权限SUID SGID SBIT

16章完后来补

### 文件查找

脚本文件的查找：`which ls`

普通文件的查找：

`whereis`：只在特定目录下查找文件

`locate`：`-i`忽略大小写 `-l` 输出几行 `-S`查看数据库信息，配合`updatedb`使用

`find`：`find [PATH] [-ctime | -atime | -mtime]`

## Linux文件压缩

### gzip

```sh
# 压缩
gzip -v a.txt
# 解压缩
gzip -d a.txt.gz
```

* `-d`，解压缩
* `-t`，检查数据一致性
* `-v`，输出压缩比
* `-#`，#表示压缩等级数字，1压缩比最差，9压缩比最高

对于压缩文件中的文本文件可以使用`zcat a.txt.gz`来查看。

### bzip2

> 压缩比要比gzip更好

```sh
bzip2 -vk a.txt  # 压缩
bzip2 -d a.txt.bz2	# 解压缩
```

其他参数与`gzip`一致

* `-k`，保留源文件压缩

类似`zcat`，bzip2压缩方式也有对应的`bzcat/bzmore/bzless/bzgrep`

### xz

> 压缩比要比bzip2还要好

```sh
xz -vk a.txt  # 压缩
xz -d a.txt.bz2	# 解压缩
```

参数同bzip2

### 打包 tar

```sh
# 压缩
tar -zcvf a.tar.gz filename
tar [-z|-j|-J] [cv] [-f 待建立的包名] 待压缩的文件或者文件名
# 解压缩
tar -zxvf file.tar.gz [-C 目录]
# 查看压缩文件名称
tar -ztvf file.tar.gz
```

* 压缩方式：`-z`表示gzip  `-j`表示bzip2，`-J`表示xz
* 操作类型：`-c`表示compress，`-x`表示解压缩，`-t`表示查看文件名
* `-v`表示输出详细信息，`-f`表示操作的文件名
* `-C` 表示解压缩到特定的目录

## Linux shell

### 基本介绍

不同的用户可能使用不同的shell

```sh
[root@localhost ~]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
2mw:x:1000:1000:2mw:/home/2mw:/bin/bash
nginx:x:987:981:nginx user:/var/cache/nginx:/sbin/nologin
www:x:1001:1001::/home/www:/bin/bash
```

对于root用户使用的是`/bin/bash`，对于nginx使用的是`/sbin/nologin`

查看历史记录：`history`，存储文件`~/.bash_history`（记录的是本次登录以前执行的命令）

* `-n`输出最近n条历史记录
* `-c`清楚所有历史记录

设置别名命令：`alias`比如`alias lm='ls -al'`，取消别名使用`unalias`

查询命令类型指令：`type ls`，会有alias，file、builtin三种类型

换行的命令使用：`\[enter]`

### Shell变量

> 注意变量赋值中间不能有空格

对于变量使用`$`为前缀，在双引号内的保留原有变量属性，在单引号内仅为一般字符。

```sh
[root@localhost ~]> name=jack
[root@localhost ~]> echo "my name is $name"
my name is jack
[root@localhost ~]> echo 'my name is $name'
my name is $name
```

取消变量：`unset name`

获取命令的值，使用==`==字符进行包含

或者使用`$()`

```sh
[root@localhost ~]> version=`uname -r`
[root@localhost ~]> echo $version
3.10.0-1160.el7.x86_64
[root@localhost ~]> version2=$(uname -a)
[root@localhost ~]> echo $version2
Linux localhost.localdomain 3.10.0-1160.el7.x86_64 1 SMP Mon Oct 19 16:18:59 UTC 2020 ...
```

### 环境变量

可以使用`env`或者`set`来查看系统的环境信息。

输出其他信息：

```sh
# 输出本次使用的shell
[root@localhost ~]> echo $SHELL
/bin/bash
# 输出上次命令运行的返回结果（0表示正常）
[root@localhost ~]> echo $?
0
# 输出本次shell的PID
[root@localhost ~]> echo $$
6305
```

可以使用`export`将自定义变量转为环境变量

> 在linux中，子进程仅会继承父进程的环境变量，不会继承自定义变量。

语法：`export var`

如果var为空，则将所有的环境变量输出

```sh
[root@localhost ~]> myname=`uname -a`haha
[root@localhost ~]> echo $myname
Linux localhost.localdomain 3.10.0-1160.el7.x86_64 1 SMP Mon Oct 19 16:18:59 UTC 2020 x86_64 x86_64 x86_64 GNU/Linuxhaha
# 进入子进程（未export）
[root@localhost ~]> bash
[root@localhost ~]> echo $myname

[root@localhost ~]> exit
exit
[root@localhost ~]> export myname
# 重新进入子进程
[root@localhost ~]> bash
[root@localhost ~]> echo $myname
Linux localhost.localdomain 3.10.0-1160.el7.x86_64 1 SMP Mon Oct 19 16:18:59 UTC 2020 x86_64 x86_64 x86_64 GNU/Linuxhaha
# 查看所有环境变量（看最后一行）
[root@localhost ~]> export
declare -x DISPLAY="localhost:13.0"
declare -x HISTCONTROL="ignoredups"
declare -x HISTSIZE="1000"
declare -x HOME="/root"
declare -x HOSTNAME="localhost.localdomain"
declare -x LANG="en_US.UTF-8"
declare -x LESSOPEN="||/usr/bin/lesspipe.sh %s"
declare -x LOGNAME="root"
declare -x MAIL="/var/spool/mail/root"
declare -x OLDPWD
declare -x PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin"
declare -x PWD="/root"
declare -x SHELL="/bin/bash"
declare -x SHLVL="2"
declare -x SSH_CLIENT="192.168.230.1 5591 22"
declare -x SSH_CONNECTION="192.168.230.1 5591 192.168.230.134 22"
declare -x SSH_TTY="/dev/pts/3"
declare -x TERM="xterm"
declare -x USER="root"
declare -x XDG_DATA_DIRS="/root/.local/share/flatpak/exports/share:/var/lib/flatpak/exports/share:/usr/local/share:/usr/share"
declare -x XDG_RUNTIME_DIR="/run/user/0"
declare -x XDG_SESSION_ID="33"
declare -x myname="Linux localhost.localdomain 3.10.0-1160.el7.x86_64 #1 SMP Mon Oct 19 16:18:59 UTC 2020 x86_64 x86_64 x86_64 GNU/Linuxhaha"
```

### 变量读取与声明

`read`用于读取用户的键盘输入。

* `-p`表示提示文字
* `-t`表示等候用户的秒数

`declare`用于声明变量的类型

* `-a`将变量定义为数组array类型
* `-i`将变量定义为int数字类型
* `-x`将变量定义为环境变量，类似`export`
* `-r`将变量定义为只读类型

```sh
[root@localhost ~]> declare -i sum=100+30+50
[root@localhost ~]> echo $sum
180
[root@localhost ~]> declare -x sum
[root@localhost ~]> export | grep sum
declare -ix sum="180"

# 数组
[root@localhost ~]> arr[1]=ok
[root@localhost ~]> arr[0]=good
[root@localhost ~]> echo $arr
good
[root@localhost ~]> echo $arr[1]
good[1]
[root@localhost ~]> echo ${arr[1]}
ok
```

### 设置开机显示信息

登陆中显示信息修改`/etc/issue`

登录成功后显示信息修改：`/etc/motd`

### 重定向数据流

标准输入：`<`，`<<`

标准输出：覆盖`>`，追加`>>`

标准错误输出(`stderr`)：`2>`，`2>>`

```sh
find /home -name .bashrc > right.txt 2> error.txt
```

🔵管道命令

`|`，上一个命令的结果是下一个命令的输入

```sh
ls -al | less
last | grep 2mm
```

🔵排序命令

`sort`, `wc`, `uniq`

🔵双向重定向

`tee`，既将数据输出到屏幕上，也将数据输出到文件中

```sh
last | tee [-a] last.txt
# -a 表示append
```

🔵处理命令

* `tr`删除或者替换字符串
* `split`如果文件过大，将大文件划分为多个小文件
  * `-b`表示划分成文件的大小
  * `-l`按照行数来划分

### 脚本编写

<h4>默认变量$0, $1..</h4>

```sh
./start.sh	lihua	24	eat
$0			$1		$2	$3
```

其他变量：

* `$#`表示参数的个数
* `$@`表示`$1 $2 ...`

复杂判断流程：

> 注意判断条件需要由`[ ]`包含，并且需要空格隔开，`==, !=`之间也需要空格隔开。

```sh
read -p "Input you name:" name
if [ "${name}" == "me" ]; then
        echo "Yes it is me"
elif [ "${name}" == "you" ]; then
        echo "No thats you"
else
        echo "Other people"
fi
```

举例：

```sh
tim=$(date "+%Y%m%d-%H%M%S")
redisPort=6379
mysqlPort=3306
tmpFile=port_file.tmp
netstat -tunvp > $tmpFile
content=`grep ":3306" $tmpFile | grep "docker"`
if [ "${content}" != "" ]; then
	echo "Mysql has opened."
else
	docker start mysql01
	echo "Starting mysql from docker."
fi

content=`grep ":6379" $tmpFile | grep "redis"`
if [ "${content}" != "" ]; then
	echo "Redis has opened."
else
	/usr/local/bin/redis-server /usr/local/bin/redis.conf
	echo "Starting redis."
fi
nohup java -jar blogBackend.jar > ./log/${tim}".out" 2>&1 &
echo "Start Server, log file is in: ./log/${tim}.out"
rm $tmpFile
```

Case语句：

```sh
read -p "Input your name" name
case $name in
        "jack" )
        echo "yes, Jack"
        ;;

        "john" )
        echo "yes john"
        ;;

        * )
        echo "Other Girl..."
esac
```

函数功能：

函数的定义和传参

```sh
function s(){
        echo function name is ${0};
        echo you name is $1.
}

read -p "Name: " nn
s $nn
```

循环：

```sh
while [ condition ]
do
	# ...
done

until [ condition ]
do
	# ...
done
```

迭代：

```sh
for var in a b c
do
	echo $var
done
# a b c
for var in $( seq 1 5 ) # 从1循环到5
do 
	# ...
done

for (( i=1; i <= 10; i=1+1 ))
do
	s=$(( ${s} + ${i} ))
done
```

<h4>脚本检验</h4>

用法：`sh [-nx] a.sh`

* `-n` 不执行脚本，只检验语法问题
* `-x` 将使用到的脚本内容输出到屏幕
* `-v` 执行前，先将脚本内容输出到屏幕上

## 用户管理

### 用户和用户组信息

`id`查看每个用户有自己唯一的UID，所在的组为GID

```sh
[root@localhost ~]# id
uid=0(root) gid=0(root) groups=0(root)
[root@localhost ~]# id nginx
uid=987(nginx) gid=981(nginx) groups=981(nginx)
```

查看`/etc/passwd`用户信息：

```sh
> head -n 4 /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
```

分号之间分割的是：账号名称：密码：UID：GID：用户信息栏：home目录：shell

这里的密码迁移到了`/etc/shadow`文件中了。

```sh
> head -n 6 /etc/shadow
root:$1$jAtUS7QW$ztQIqapSdbgJCKnS/eJrs/::0:99999:7:::
bin:*:18353:0:99999:7:::
daemon:*:18353:0:99999:7:::
adm:*:18353:0:99999:7:::
lp:*:18353:0:99999:7:::
2mw:$1$jAtUS7QW$ztQIqapSdbgJCKnS/eJrs/:18882:0:99999:7:::
```

分号之间分割的是：

* 账号名称
* 密码
* 最近密码修改日期
* 密码不可被修改的天数
* 密码需要重新修改的天数
* 密码需要修改期限前警告天数
* 密码过期后宽限的天数
* 账号失效日期
* 保留

root密码忘记了怎么办：可以使用Live CD方式启动系统，并且修改`/etc/shadow`文件中的密码。

查看linux中密码加密机制：`authconfig --test | grep hashing`

查看用户组的信息文件`/etc/group`

```sh
> head -n 4 /etc/group
root:x:0:
bin:x:1:
daemon:x:2:
sys:x:3:
adm:x:4:
```

分别是组名：用户组密码：GID：用户组支持的账号名称

切换用户组：`newgrp root`

### 账号管理

修改密码：`sudo passwd jack`，不知道用户名默认为修改使用者账号密码。

```sh
-l # 表示Lock，是密码失效，在/etc/shadow对应密码项前加一个!
-u # 表示unlock，解锁密码
-n # 表示多久不可以修改密码的天数
-x # 表示多久内必须修改密码的天数
-w # 表示密码过期前警告天数
-i # 接日期，表示密码失效的日期
-S # 查看账户详细信息
```

添加用户：`useradd`

```
useradd jack [-g 初始group] [-G 次要用户组] [-c 说明] [-s 使用的shell] [-d home目录] [-e YYYY-MM-DD失效日期]
```

显示密码详细参数：`chage`

```sh
-l # 列出详细账户参数
-E # 修改账号失效日期 YYYY-MM-DD
-I # 修改密码失效天数
-M # 密码多久需要进行修改
-W # 密码过期前警告日期
```

修改用户权限`usermod`：

```sh
-L # 暂时锁定用户
-U # 解锁用户
# 其他指令于useradd类似，比如-g 修改初始用户组，-G修改次要用户组
```

修改自己启动的shell `chsh`指令

```sh
chsh -l	# 查看可以用的shell
chsh -s /bin/bash # 修改自己的shell
```

删除用户：`userdel`

```sh
userdel [-r] jack
# -r 表示连同home目录一起删除
```

<h4>组管理</h4>

添加组：`groupadd student`

修改组：`groupmod -n newname name`

删除组：`groupdel groupname`

用户组管理员：`gpasswd`

```sh
gpasswd group	# 不加参数表示设置组管理员密码
gpasswd -A user1 user2 groupname	# 设置组管理员
gpasswd -M user1 groupname			# 将用户添加进这个组
gpasswd -r gname	# 删除密码
```

查看当前登录的用户：`who`或者`w`

查看登录记录：`last`或者`lastlog`

终端之间发送消息：

`write root`向root用户发送消息

`mesg [y|n]` 选择是否开启接收消息，y表示yes

## 计划任务

### at命令

`at`命令用于只执行一次的任务，不过需要`atd`服务的开启，有些linux服务器可能并未开启。

开启atd服务：

```sh
systemctl restart atd
systemctl enable atd
systemctl status atd
```

使用at命令的限制：

允许使用at命令的配置在`/etc/at.allow`文件中，禁止使用at命令的配置在`/etc/at.deny`文件中。

使用语法：

```sh
at [-mldv] TIME
at -c taskId
-m # 表示发送email
-l # 表示列出所有使用者的计划任务
-d # 取消一个计划任务
-v # 使用较为明显的时间格式来显示任务列表
-c # 列出该项任务的实际内容，后面接任务ID
Time：
	HH:MM
	HH:MM YYYY-MM-DD
	HH:MM[am|pm] [Month] [Date]
	HH:MM[am|pm] + number [minutes|hours|days|weeks]
	now + 5 minutes
```

删除任务：

```
at -d id
atrm id
```

`batch`命令：在CPU任务负载小于0.8的时候才执行工作任务，命令类似at。

### crontab命令

同样限制使用的配置文件在`/etc/cron.allow`和`/etc/cron.deny`

```sh
crontab [-u username] [-ler]
-e # 编辑定时任务
-l # 查看定时任务
-r # 删除定时任务
```

特殊字符：

* `*` 代表任意时间
* `,` 代表“或” `0 3,6 * * *` 代表3：00和6：00执行命令
* `-` 代表区间。`20 8-12 * * *` 代表 8，9，10，11，12时的20分会执行任务
* `/n` 代表时间间隔 `*/5 * * * *` 表示每间隔5分钟执行一次

### timeout命令

```sh
$ timeout [OPTION] DURATION COMMAND [ARG]...
 s : 秒 (默认)
 m : 分钟
 h : 小时
 d : 天
 # 长选项必须使用的参数对于短选项时也是必需使用的。
 -k, 命令在初始信号发出后再经过所指定持续时间后仍在运行，则对其发送 KILL 信号
 -s : --signal=信号，超时后发送的信号。信号类似"HUP"的信号名或是信号数。查看"kill -l"以获得信号列表
 --help        显示此帮助信息并退出
 --version        显示版本信息并退出
 # 如果不添加任何单位，默认是秒。如果DURATION为0，则关联的超时是禁用的。
 # 如果程序超时则退出状态数为124，否则返回程序退出状态。
 # 如果没有指定信号则默认为TERM 信号。TERM 信号在进程没有捕获此信号时杀死进程。
 # 对于另一些进程可能需要使用KILL (9)信号，当然此信号不能被捕获。
```

比如：

```sh
timeout 10 top 					# 10秒后结束top命令
timeout 5m ping www.baidu.com	# 5分钟后停止ping命令
timeout -k 10s 1m ./gcc			# 1分钟后还在运行，则再过10秒后结束
```

### sleep命令

延迟

```sh
sleep 1		# 延迟一秒
sleep 5m
sleep 3h
sleep 1d
```

## 任务管理

### 后台任务

开启后台任务 `&`

```
./redis-server &
```

在vim编辑器中切换后台：`ctrl+z`键

查看所有后台: `jobs`

将应用从后台切换上来：

```sh
[root@localhost bin]# jobs
[2]+  Stopped                 vim cron.log  (wd: ~)
[3]-  Running                 redis-server &
[root@localhost bin]# fg %2
```

`kill`命令：可以杀死进程，也可以向进程传递信号

### 查看进程

```sh
ps -l # 只查看和自己bash相关的进程
ps aux # 查看系统中所有进程的状态
ps -ef # 查看每一个进程详细信息
```

动态查看进程：`top`

查看进程树：`pstree`

查看内存使用情况：`free`

查看系统启动时间和任务负载：`uptime`

查看网络情况：`netstat -tunlp`	tu表示tcp和udp，n时服务以端口显示 p显示进程。

## 系统服务

系统服务配置文件目录：`/usr/lib/systemd/system`

### systemctl查看状态

```bash
# systemctl [cmd] [service]
systemctl status atd
# start stop restart reload enable disable status
systemctl # 查看所有的服务
systemctl list-dependencies [unit] [--reverse]
# --reverse 表示反向追踪谁使用这个unit
```

### 配置文件详解

service文件：

```
[Unit]
Description=简易说明
Documentation=文档说明
After=在哪个Daemon启动后才运行（不强制）
Requires=在哪个Daemon启动后才运行（强制）
Conflicts=冲突性检查
[Service]
Type=启动方式会影响ExecStart.simple由ExecStart启动，forking由于程序运行过久因此关闭，oneshot一次性的
EnvironmentFile=环境变量配置文件（一般在/etc/sysconfig/sshd）
ExecStart=启动命令或脚本
ExecStop=关闭服务的命令 systemctl stop命令
ExecRelod=systemctl reload相关命令
Restart=
RestartSec=被关闭后需要多长时间重启(毫秒)
[Install]
```

