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

