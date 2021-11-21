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
