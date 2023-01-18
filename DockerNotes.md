# DockerNotes

[TOC]

[BV1og4y1q7M4](https://www.bilibili.com/video/BV1og4y1q7M4)

## 基本概念

镜像(Image)：相当于模板，提供模板创建容器。Tomcat镜像->Run->tomcat容器

容器(container)：提供镜像来创建，可以理解为简单的linux系统

仓库(repository)：存放镜像的地方，分为公有和私有仓库。

Docker是C/S架构的系统，需要使用Client去连接Docker服务。

## Docker安装

首先设置安装仓库：

```sh
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

更新yum软件包索引：

```sh
yum makecache fast
```

安装`docker-ce`社区版

```sh
sudo yum install docker-ce docker-ce-cli containerd.io
```

启动docker服务：

```sh
sudo systemctl start docker
```

判断是否安装成功：

```sh
docker version
```

运行镜像：

```sh
docker run hello-world
```

查看镜像：

```sh
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d1165f221234   4 months ago   13.3kB
```

> Windows下安装docker如果运行出现错误使用`netsh winsock reset`命令进行尝试。以及docker的C盘迁移[LINK](https://www.cnblogs.com/xhznl/p/13184398.html)

## Windows使用docker

[安装 WSL | Microsoft Docs](https://docs.microsoft.com/zh-cn/windows/wsl/install)

## WSL迁移

1. 首先关闭docker

2. 关闭所有发行版：
   `wsl --shutdown`
   
3. 查看所有镜像

   `wsl -l -v`

4. 将docker-desktop-data导出到D:\SoftwareData\wsl\docker-desktop-data\docker-desktop-data.tar（注意，原有的docker images不会一起导出）
   `wsl --export docker-desktop-data D:\SoftwareData\wsl\docker-desktop-data\docker-desktop-data.tar`

5. 注销docker-desktop-data：
   `wsl --unregister docker-desktop-data`

6. 重新导入docker-desktop-data到要存放的文件夹：D:\SoftwareData\wsl\docker-desktop-data\：
   `wsl --import docker-desktop-data D:\SoftwareData\wsl\docker-desktop-data\ D:\SoftwareData\wsl\docker-desktop-data\docker-desktop-data.tar --version 2`

样例CMD：

```sh
wsl --shutdown
wsl -l -v
wsl --export docker-desktop-data .\docker-desktop-data.tar
wsl --unregister docker-desktop-data
wsl --import docker-desktop-data  .\   .\docker-desktop-data.tar --version 2
echo 'migrate docker-desktop-data successfully'

wsl --export docker-desktop .\docker-desktop.tar
wsl --unregister docker-desktop
wsl --import docker-desktop  .\   .\docker-desktop.tar --version 2

echo 'Over'
pause
```

## 在 WSL 上安装docker遇到的问题

1. 由于 docker 在 k8s 1.24 之后不再是默认运行时，需要安装 cri-dockerd

   ```sh
   wget https://ghproxy.com/https://github.com/Mirantis/cri-dockerd/releases/download/v0.2.5/cri-dockerd_0.2.5.3-0.ubuntu-jammy_amd64.deb
   dpkg -i cri-dockerd_0.2.5.3-0.ubuntu-jammy_amd64.deb
   sed -i -e 's#ExecStart=.*#ExecStart=/usr/bin/cri-dockerd --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.8#g' /usr/lib/systemd/system/cri-docker.service
   systemctl daemon-reload
   systemctl enable cri-docker
   ```

2. 遇到 docker 的守护进程 dockerd 因为 iptable 无法启动

   ```sh
   sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
   sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
   ```

## 配置阿里云镜像加速

登陆阿里云控制台=》容器镜像服务=》镜像加速器

```sh
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xei795wg.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

> Windows版直接图标Settings的json里加入网址即可。

## Docker命令

### 镜像命令

* `docker images [-a -q]`：查看镜像

* `docker search mysql`：搜索某个镜像

  `docker search mysql --filter=STARS>3000`

* `docker pull pkg`：下载镜像

* `docker rmi [IMAGE ID, IMAGE ID]`：删除镜像

### 容器命令

> 有了镜像才能创建容器，这里使用CentOS镜像

下载CentOS：

```sh
docker pull centos
```

启动容器命令：

```sh
docker run [--args] image
docker run --name <container_name> -e <env_var> -d  -p <host_port:con_port> <image>:<tag>
# 参数说明
--name="name"	 容器名字 tomcat01 tomcat02 用来区分容器
-d				后台方式启动，相当于nohup
-it				使用交互式运行，进入容器查看内容
-p				指定 [8080:8080](主机端口:容器端口)，映射到容器端口
-P				随机指定端口(大写)
-e				设置环境变量
```

举例：`docker run --name psql_a -e POSTGRES_USER=root -e POSTGRES_PASSWORD=**** -p 5433:5432 -d postgres:13`

启动CentOS：

```sh
docker run -it centos /bin/bash
# =============
[root@7c52f9205593 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
# 镜像CentOS很多命令并不完善
```

列出所有运行的容器：

```sh
docker ps
docker ps -a 		# 将运行过的程序也列出来
docker ps -n=2	# 显示最近创建的n个容器
docker ps -a -q	# 只显示容器的ID
```

退出容器：

```sh
exit			# 停止容器并且退出
Ctrl + p + q	 # 不停止容器并且退出
```

删除容器：

```sh
docker rm 容器ID	
docker rm -f [Con ID] # 强制删除运行中的容器
```

启动重启停止杀死容器：

```sh
docker start [Con ID]
docker restart [Con ID]
docker stop [Con ID]
docker kill [Con ID]
```

**后台启动容器：**

```sh
docker run -d centos
# 如果使用docker ps -a会发现centos已经停止使用了
```

> 这是常见的问题：docker容器在后台使用，必须有一个前台应用，否则程序会自动停止。

### 其他命令

查看日志：

```sh
docker logs
```

查看容器内部的进程信息：

```sh
docker top [ID]
# ======================
C:\Users\admin>docker top b11b4ccb7004
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                1218                1197                0                   11:01               ?                   00:00:00            /bin/bash
```

查看镜像元数据：

```sh
docker inspect conID
```

<details>
C:\Users\admin>docker inspect b11b4ccb7004
[
    {
        "Id": "b11b4ccb70048f362bce11daae6476f67b5202f5ec219b1ac069d92393c4504f",
        "Created": "2021-07-06T11:01:54.1616653Z",
        "Path": "/bin/bash",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 1218,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-07-06T11:01:55.8334204Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55",
        "ResolvConfPath": "/var/lib/docker/containers/b11b4ccb70048f362bce11daae6476f67b5202f5ec219b1ac069d92393c4504f/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/b11b4ccb70048f362bce11daae6476f67b5202f5ec219b1ac069d92393c4504f/hostname",
        "HostsPath": "/var/lib/docker/containers/b11b4ccb70048f362bce11daae6476f67b5202f5ec219b1ac069d92393c4504f/hosts",
        "LogPath": "/var/lib/docker/containers/b11b4ccb70048f362bce11daae6476f67b5202f5ec219b1ac069d92393c4504f/b11b4ccb70048f362bce11daae6476f67b5202f5ec219b1ac069d92393c4504f-json.log",
        "Name": "/sweet_pascal",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                48,
                158
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/779837a8025e8ce5888d8b808b9c8db6577aee296c9349c3e9c55b776d6425d4-init/diff:/var/lib/docker/overlay2/db857974f796bbafa74cbbfe00b2a7bb47606900c6098271289e05f7b734c0ec/diff",
                "MergedDir": "/var/lib/docker/overlay2/779837a8025e8ce5888d8b808b9c8db6577aee296c9349c3e9c55b776d6425d4/merged",
                "UpperDir": "/var/lib/docker/overlay2/779837a8025e8ce5888d8b808b9c8db6577aee296c9349c3e9c55b776d6425d4/diff",
                "WorkDir": "/var/lib/docker/overlay2/779837a8025e8ce5888d8b808b9c8db6577aee296c9349c3e9c55b776d6425d4/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "b11b4ccb7004",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20201204",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "f66dbf3b3d86673bc3b120802359789a364e5083239056d11014f3d6e9519e7c",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/f66dbf3b3d86",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "565bec3d893f50bb6fbe9fb4da05192bbb350bba83ec497ffc0f47ffdf23161f",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "05463d6ac4bd4b8f733c29a8a4f1375f5090e2da1e2ed64c74f49f97bdb41844",
                    "EndpointID": "565bec3d893f50bb6fbe9fb4da05192bbb350bba83ec497ffc0f47ffdf23161f",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]    
</details>

进入当前正在运行的容器：

```sh
# 方式一 相当于重新打开一个终端
docker exec -it [Con ID] /bin/bash
# 方式二 进入容器正在运行的终端
docker attach [Con ID] 
```

从容器内拷贝到主机上

```
docker cp [ConId]:[Con Path] [Host Path]
```

查看docker容器CPU的情况：

```sh
docker stats
```

## Docker实例

### 安装nginx

```sh
docker search nginx
docker pull nginx
docker run -d --name nginx01 -p 3344:80 nginx
docker exec -it nginx01 /bin/bash
docker stop nginx01
```

### 安装tomcat

```sh
docker run -it --rm tomcat:9.0		# --rm 表示测试容器，用完即删容器（不删除镜像）
docker run -d --name tomcat01 -p 3345:8080 tomcat:9.0
docker exec -it tomcat01
```

### 部署es+kibana

```sh
# es 暴露端口很多并且十分耗内存
# es 的数据一般要放置到安全目录！ 需要挂载
# --net somenetwork

# docker network create somenetwork
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:tag
```

### 保存自己的镜像

> 相当于快照

```sh
docker commit -a="Author" -m="Some msg" <conId> <NewImgId>:<TAG>
```

## 容器数据卷

> 一种数据共享的技术，docker产生的数据会存储到本地。将容器上的目录挂载到本机上+容器间数据共享。

### 挂载：

方式一：

```sh
docker run --name <name> -v <hostpath>:<conpath> -p <hport>:<cport> <img>
```

MySQL的数据存放在其`data`目录下

### 具名挂载和匿名挂载

```sh
# 匿名挂载 ro->readonly rw->readwrite
docker run --name <name> -v <conpath>:<ro | rw> -p <hport>:<cport> <img>

# 具名挂载  此处name不是目录
docker run --name <name> -v <name>:<conpath> -p <hport>:<cport> <img>

# 查看所有卷的情况
docker volume ls

# 查看某个卷的所在目录：
docker volume inspect 090a2c068e303678c2384df0e5980a4be517c83c71e6e5a82d790f7baf1de531
```



```json
[
    {
        "CreatedAt": "2021-07-22T10:03:50Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/090a2c068e303678c2384df0e5980a4be517c83c71e6e5a82d790f7baf1de531/_data",
        "Name": "090a2c068e303678c2384df0e5980a4be517c83c71e6e5a82d790f7baf1de531",
        "Options": null,
        "Scope": "local"
    }
]
```

### 初步构建dockerfile

>Dockerfile就是用来构建docker镜像的文件

```dockerfile
FROM centos						# 从那个镜像而来

VOLUME ["/home"]		# 设置挂载的目录，只写了容器内的目录为匿名挂载

CMD echo "======END======"		# 输出

CMD /bin/bash					# 进入命令
```

根据dockerfile生成镜像：

```sh
docker build -f <dockerfile> -t <2mw/centos:tag> .
```

### 数据卷容器

> 多个容器共享数据

```sh
docker run --it --name centos02 --volumes-from docker01 <imgID>
# 使用 --volumes-from 来继承同步父容器挂载的内容
```

删除父容器之后，其他容器的数据不会消失。只要有一个容器挂载，数据就不会删除。

## Dockerfile

### 初识dockerfile

* 指令使用大写字母
* 每个指令都会创建一层镜像并且提交

<img src="https://i.loli.net/2021/08/02/b1eiuSYHXIEZMBP.jpg" alt="Dockerfile编写指南" style="zoom: 80%;" />

```sh
FROM			# 基础镜像
MAINTAINER		# 维护者
RUN				# 镜像构建的时候需要运行的命令
ADD				# 添加内容，比如centos镜像中再添加一个tomcat的镜像
WORKDIR			# 进入exec模式的时候的工作目录
VOLUME			# 设置挂载的容器卷
EXPOSE			# 暴露的端口
RUN
CMD				# 指定此容器启动时需要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT		 # 指定此容器启动时需要运行的命令，可以追加命令
ONBUILD			 # 当构建一个被继承的 dockerfile的时候，就会运行ONBUILD指令
COPY			# 类似ADD，将文件拷贝到镜像中
ENV				# 构建的时候设置的环境变量
```

**CMD与ENTRYPOINT的区别：**

cmd：

```sh
# dockerfile 文件
FROM centos
CMD ["ls", "-a"]

# run dockerfile，会列出所有目录
docker run <imgId>

# 如果想要追加命令，如 ls -a -l的形式
docker run <imgId> -l  		# 会出错，使用-l替换了 ls -a
docker run <imgId> ls -al  	# 更正形式
```

entrypoint

```sh
# dockerfile 文件
FROM centos
ENTRYPOINT ["ls", "-a"]

# build后run dockerfile，会列出所有目录
docker run <imgId>

# 如果想要追加命令，如 ls -a -l的形式
docker run <imgId> -l  		# 相当于 ls -al
```

<h4>举例：</h4>

```dockerfile
# 构建一个自己的centos
FROM centos
MAINTAINER 2mw<appose@gmail.com>

ENV MYPATH /usr/local	# 设置环境变量
WORKDIR $MYPATH

RUN yum -y install vim
RUN yun -y install net-tools

EXPOSE 80		# 暴露端口

CMD echo $MYPATH
CMD echo "====END===="
CMD /bin/bash
```

### 实战：构建tomcat镜像

需要准备tomcat+jdk的安装包

```dockerfile
FROM centos
MAINTAINER 2mw<appose@gmail.com>

COPY readme.txt /usr/local/readme.txt  			# 将主机上的readme.txt拷贝到镜像中
ADD	jdk-8ull-linux-x64.tar.gz	/usr/local/		# Add命令会自动解压
ADD apache-tomcat-x64.tar.gz	/usr/local/

RUN yum -y install vim

ENV MYPATH /usr/local
WORKDIR $MYPATH

ENV JAVA_HOME /usr/local/jdk1.8.0_11
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat
ENV CATALINA_BASH /usr/local/apache-tomcat
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

EXPOSE 8080

CMD /usr/local/apache-tomcat/bin/startup.sh && tail -F /usr/local/apache-tomcat/bin/logs/catalina.out
```

构建：

```sh
# 如果文件名为Dockerfile，使用docker build的时候就不需要指定文件名
docker build -t <name> .
```

### 发布自己的镜像

> dockerhub

1. 登录：

   ```sh
   docker login -u <account>
   ```

2. Push

   ```sh
   # 加一个tag（可选）
   docker tag <oldimg> <newimg>:<tag>
   # push
   docker push <name/img>:<tag>
   ```

> 阿里云

1. 创建命名空间
2. 创建容器镜像

## Docker网络

### docker0地址

![image-20210803094916078](https://i.loli.net/2021/08/03/Bs5Z82JKuCqSxoa.png)

查看内部容器ip地址：

```sh
docker exec -it nginx01 ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: sit0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/sit 0.0.0.0 brd 0.0.0.0
7: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

查看Docker网络信息：

```sh
docker network ls
docker network inspect 9eb8509e1d11
```

```json
{
    "Name": "bridge",
    "Id": "9eb8509e1d11e0ff5a52cf9cf5b758cbf1f3017a4aaecd0332e3a98385b71eb8",
    "Created": "2021-08-03T01:50:17.5758794Z",
    "Scope": "local",
    "Driver": "bridge",
    "EnableIPv6": false,
    "IPAM": {
        "Driver": "default",
        "Options": null,
        "Config": [
            {
                "Subnet": "172.17.0.0/16",
                "Gateway": "172.17.0.1"
            }
        ]
    },
    "Internal": false,
    "Attachable": false,
    "Ingress": false,
    "ConfigFrom": {
        "Network": ""
    },
    "ConfigOnly": false,
    "Containers": {
        "07386948c83e25b6ec5a984781f02c70e9bbfcc102afb5fbeda7fa0300364ce2": {
            "Name": "nginx02",
            "EndpointID": "c7ba5f239d7502b348cfb1077f1b0bb05b9c99cc56e8ba7a2095593093cbb6f2",
            "MacAddress": "02:42:ac:11:00:03",
            "IPv4Address": "172.17.0.3/16",
            "IPv6Address": ""
        },
        "10b13848801668474ab7e708a2a29bb675c121b48b095ec8b47b5e004e845ea9": {
            "Name": "nginx01",
            "EndpointID": "76fbe2e051c604f1233c912428a007a205f8574dfd9146de29ac02ac8e7c08bf",
            "MacAddress": "02:42:ac:11:00:02",
            "IPv4Address": "172.17.0.2/16",
            "IPv6Address": ""
        }
    },
    "Options": {
        "com.docker.network.bridge.default_bridge": "true",
        "com.docker.network.bridge.enable_icc": "true",
        "com.docker.network.bridge.enable_ip_masquerade": "true",
        "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
        "com.docker.network.bridge.name": "docker0",
        "com.docker.network.driver.mtu": "1500"
    },
    "Labels": {}
}
```

### --link

> 不推荐使用

实现两个主机ping互通

```sh
docker run -d -P --name nginx01 nginx
docker run -d -P --name nginx02 --link nginx01 nginx
docker exec -it nginx02 ping nginx01  # 可以ping通
#docker exec -it nginx01 ping nginx02  # 不可以ping通，因为没有配置
```

原理是将nginx01的ip放到其`/etc/hosts`文件中

### 自定义网络

如果直接启动一个镜像会带一个默认参数：

```sh
docker run -d -P --name nginx01 --net bridge nginx  # 默认带 --net bridge
```

创建网络：

```sh
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 <net-name>
# --driver bridge 为指定默认桥接模式，可以忽略
```

* 自定义网络可以使用`--link`后的效果。

将不同子网的主机之间相互连通：

```sh
docker network connect <net-name> <con-name>
```

相当于将此容器加入到这个网络中

## 启动挂载样例

### nginx

配置所在目录：`/etc/nginx/`

日志目录：`/var/log/nginx/`

网页所在目录：`/usr/share/nginx/html`

```sh
docker run --name nginx01 -d -p 80:80 -v E:\Notes\docker\vhost\nginx01\html:/usr/share/nginx/html nginx
```

使用自己的nginx.conf

```sh
docker run --name nginx01 -d -p 80:80 -v E:\Notes\docker\vhost\nginx01\html:/usr/share/nginx/html -v E:\Notes\docker\vhost\nginx01\conf\nginx.conf:/etc/nginx/nginx.conf nginx
```

### MySQL

挂载数据到本地：

```sh
docker run --name mysql01 -d -p 3306:3306 -v E:\Notes\docker\vhost\mysql01\data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=password mysql
```

### Redis

redis持久化的目录在`/data`下，`--save`表示每60秒如果有一次写操作就进行持久化，并且加载配置文件。

```sh
docker run --name redis01 -d -p 6379:6379 -v E:\Notes\docker\vhost\redis01\data:/data -v E:\Notes\docker\vhost\redis01\conf\redis.conf:/etc/redis/redis.conf redis redis-server /etc/redis/redis.conf --save 60 1
```

### PostgreSQL

```sh
docker run --name psql_a -e POSTGRES_USER=root -e POSTGRES_PASSWORD=**** -p 5433:5432 -d postgres
```

