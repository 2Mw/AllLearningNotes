# Kubernetes 笔记

[TOC]

2022/11/29

[BV1Qv41167ck](https://www.bilibili.com/video/BV1Qv41167ck) P10

## 0x0 介绍

kubernetes 集群主要是由**控制节点**(master)和**工作节点**(node)构成，每个节点上都会安装不同的组件。

![image-20221130122232509](kubernetes.assets/image-20221130122232509.png)

* master：集群控制，负责集群决策
  * kuberctl: 控制程序
  * apiserver：资源操作的唯一入口，用于接收用户输入的命令，提供认证、授权、API注册和发现的机制。
  * scheduler：负责集群资源的调度，按照预定的调度策略将 pod 调度到相应的 node 节点上。
  * ControllerManager：负责维护集群的状态，比如程序部署安排、故障检测、自动扩展等
  * etcd：负责存储集群中各种资源的信息
* node：数据控制，负责提供运行环境
  * kubelet：负责维护容器的生命周期，控制docker 来创建、更新、销毁容器
  * kubeProxy：负责集群内部的服务发现和负载均衡
  * Docker：负责节点上的容器的各种操作。

## 0x1. 安装

### 0. 准备

首先需要配置一主两从的环境，准备主机配置 hosts

```
10.0.0.3	master01
10.0.0.4	node01
10.0.0.5	node02
```

关闭防火墙，dnsmasq, swap 分区：

```sh
systemctl disable --now firewalld
systemctl disable --now dnsmasq
swapoff -a && sysctl -w vm.swappiness=0
```

注释swap代码：

```sh
vim /etc/fstab
#/swap.img      none    swap    sw      0       0
```

安装 ntpdate 来同步时间:

```sh
apt install ntpdate
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
echo 'Asia/Shanghai' > /etc/timezone
ntpdate time2.aliyun.com
```

将同步任务加到开机自启动：

```sh
crontab -e
# 输入以下指令
*/5 * * * * ntpdate time2.aliyun.com
```

配置 limit：

```sh
# 用于配置 shell 可支配的资源数目
ulimit -SHn 65535
```

禁用 iptables，因为 k8s 和 docker 在运行的时候会产生大量的 iptables 规则，为了不让系统规则和他们混淆，直接关闭系统的规则

```sh
systemctl stop iptables
```

修改 linux 内核参数，添加网桥过滤和地址转发功能：

```sh
# vim /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sysctl -p
```

运行以下指令：

```sh
# 重新加载配置
sysctl -p
# 加载网桥过滤模块
modprobe br_netfilter
```

配置 ipvs 功能，因为 k8s 中 service 由两种代理模型，一种是基于 iptables 的，另一种是基于 ipvs 的。后者性能更高：

```sh
apt install ipset ipvsadm

# 写入脚本
cat << EOF > /etc/sysconfig/modules/ipvs.modules
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack	# 内核4.18之后用这行
modprobe -- nf_conntrack_ipv4
EOF

# 添加执行权限
chmod +x /etc/sysconfig/modules/ipvs.modules
# 执行脚本
/bin/bash /etc/sysconfig/modules/ipvs.modules
# 查看是否配置完成
lsmod | grep -e ip_vs -e nf_conn
```

### 1. 安装 docker

参考：[Install Docker on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)，[国内镜像安装](https://mirrors.bfsu.edu.cn/help/docker-ce/)

```sh
apt install ca-certificates curl gnupg lsb-release
mkdir -p /etc/apt/keyrings
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

apt-get update
apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

systemctl enable docker
```

由于 docker 在 k8s 1.24 之后不再是默认运行时，需要安装 cri-dockerd

```sh
wget https://ghproxy.com/https://github.com/Mirantis/cri-dockerd/releases/download/v0.2.5/cri-dockerd_0.2.5.3-0.ubuntu-jammy_amd64.deb
dpkg -i cri-dockerd_0.2.5.3-0.ubuntu-jammy_amd64.deb
sed -i -e 's#ExecStart=.*#ExecStart=/usr/bin/cri-dockerd --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.8#g' /usr/lib/systemd/system/cri-docker.service
systemctl daemon-reload
systemctl enable cri-docker
```

### 2. 安装 k8s 组件

[kubernetes镜像_阿里巴巴](https://developer.aliyun.com/mirror/kubernetes?spm=a2c6h.13651102.0.0.b3761b11TtN9Mq)

```sh
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl

systemctl enable kubelet
```

配置 kubelet：

```sh
vim /etc/sysconfig/kubelet

KUBELET_CGROUP_ARGS="--cgroup-driver=systemd"
KUBE_PROXY_MODE="ipvs"
```

### 3. 创建集群

在 master 节点上运行：

```sh
kubeadm init \
--kubernetes-version=v1.25.4 \
--pod-network-cidr=10.244.0.0/16 \
--service-cidr=10.96.0.0/12 \
--apiserver-advertise-address=192.168.233.100
```

注意：版本设置成自己的版本，apiserver 设置成master的ip地址。

如果出现错误，执行以下指令后重新运行。

```sh
rm /etc/containerd/config.toml 
systemctl restart containerd
```

拉取所需镜像：

```sh
$ kubeadm config images list

registry.k8s.io/kube-apiserver:v1.25.4
registry.k8s.io/kube-controller-manager:v1.25.4
registry.k8s.io/kube-scheduler:v1.25.4
registry.k8s.io/kube-proxy:v1.25.4
registry.k8s.io/pause:3.8
registry.k8s.io/etcd:3.5.5-0
registry.k8s.io/coredns/coredns:v1.9.3
```

定义镜像，然后从阿里云拉取（注意里面的版本和上面对应）：

```sh
images=(
    kube-apiserver:v1.25.4
    kube-controller-manager:v1.25.4
    kube-scheduler:v1.25.4
    kube-proxy:v1.25.4
    pause:3.8
    etcd:3.5.5-0
    coredns:v1.9.3
)

for n in ${images[@]}; do
	docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$n
	docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$n	k8s.gcr.io/$n
	docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$n
done
```

---

🔵k8s 1.25的集群初始化方法：

```sh
kubeadm config images pull --cri-socket=unix:///var/run/cri-dockerd.sock \
             --image-repository registry.aliyuncs.com/google_containers

kubeadm init --image-repository registry.aliyuncs.com/google_containers --apiserver-advertise-address=192.168.233.100 --service-cidr=10.96.0.0/12 --pod-network-cidr=10.244.0.0/16 --cri-socket /var/run/cri-dockerd.sock
```

在 master 节点产生 token 让 node 节点加入：

```sh
> kubeadm token create --print-join-command
kubeadm join 192.168.233.100:6443 --token m22sdu.lb5gye6e7owj48xf --discovery-token-ca-cert-hash sha256:cc96de7b2f1a6f15e4699422cd4c4bd54a8995f308b3372178eb514cf86c44c9
```

在 node 节点上加入集群：

```sh
kubeadm join 192.168.233.100:6443 --token m22sdu.lb5gye6e7owj48xf --discovery-token-ca-cert-hash sha256:cc96de7b2f1a6f15e4699422cd4c4bd54a8995f308b3372178eb514cf86c44c9 --cri-socket /var/run/cri-dockerd.sock
```

在 master 节点上查看节点信息：

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

> kubectl get nodes
NAME     STATUS     ROLES           AGE     VERSION
master   NotReady   control-plane   11m     v1.25.4
node-1   NotReady   <none>          9m19s   v1.25.4
node-2   NotReady   <none>          8m56s   v1.25.4
```

可以看到各个及其的状态还是 NotReady 的状态，还需要安装网络插件。kubernetes 支持多种网络插件，比如 flannel，calico，canal 等，任选一种即可。

> 操作只需要在 master 节点上运行即可，插件使用的是 DaemonSet 控制器，会在每个节点上都运行

现在下载并且使用 flannel 配置文件：

```sh
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml
```

由于使用的网址是 docker.io 在墙内可能会安装不成功，将 flannel 换位 jmgao1983/flannel：

```yml
- name: install-cni
#image: flannelcni/flannel:v0.20.2 for ppc64le and mips64le (dockerhub limitations may apply)
#image: docker.io/rancher/mirrored-flannelcni-flannel:v0.20.2
image: jmgao1983/flannel:v0.20.2
command:
- cp
args:
- -f
- /etc/kube-flannel/cni-conf.json
- /etc/cni/net.d/10-flannel.conflist
volumeMounts:
- name: cni
mountPath: /etc/cni/net.d
- name: flannel-cfg
mountPath: /etc/kube-flannel/
containers:
- name: kube-flannel
#image: flannelcni/flannel:v0.20.2 for ppc64le and mips64le (dockerhub limitations may apply)
#image: docker.io/rancher/mirrored-flannelcni-flannel:v0.20.2
image: jmgao1983/flannel:v0.20.2
```

如果未下载成功则直接使用 docker pull 下来：

```sh
docker pull jmgao1983/flannel:v0.20.2
```

查看状态：

```sh
kubectl describe pod -n kube-flannel
kubectl get pods -n kube-system
```

如果是在不行就使用科学上网的方法（添加代理）：

```sh
# 代理设置成自己的代理端口
export https_proxy=http://192.168.233.1:9999
export http_proxy=http://192.168.233.1:9999
export all_proxy=socks5://192.168.233.1:9999
```

### 4. 服务部署

测试部署 nginx

```sh
# 部署 nginx
kubectl create deployment nginx --image=nginx
# 暴露端口
kubectl expose deployment nginx --port=80 --type=NodePort
```

查看部署情况：

```sh
root@master:~# kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
nginx-76d6c9b8c-6bw72   1/1     Running   0          5m11s
root@master:~# kubectl get service
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        76m
nginx        NodePort    10.102.23.201   <none>        80:30237/TCP   2m32s
```

暴露端口在 `30237` ，访问 master ip + 端口即可。

## 0x3 资源管理
