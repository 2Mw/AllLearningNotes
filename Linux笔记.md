# Linux Notes

[TOC]

## 文件 目录 磁盘

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

