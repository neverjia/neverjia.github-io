---
title: "k8s的搭建与使用"
slug: "k8s的搭建与使用"
date: "2024-04-14"
tags: [k8s]
---


### 1.环境准备
- 准备3台机器
```
[root@kmaster ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.230.210 kmaster
192.168.230.211 knode1
192.168.230.212 knode2

#设置
scp  /etc/hosts knode1:/etc/
scp  /etc/hosts knode2:/etc/
```

修改主机名
```
hostnamectl set-hostname kmaster
hostnamectl set-hostname knode1
hostnamectl set-hostname knode2
```




- 系统初始化配置

配置静态IP
```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
[root@kmaster ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens160
TYPE=Ethernet
BOOTPROTO=none
NAME=ens160
DEVICE=ens160
ONBOOT=yes
IPADDR=192.168.230.210
NETMASK=255.255.255.0
GATEWAY=192.168.230.2
DNS1=192.168.230.2
[root@kmaster ~]#
```
重启网卡信息
```
[root@localhost network-scripts]# nmcli conn reload
[root@localhost network-scripts]# nmcli conn down ens33 && nmcli conn up ens33
Connection 'ens33' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/1)
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/2)

```


```
#确保三台机器，的跨节点的容器互相通信， Tigera Calico operator
主机名、节点ip、部署组件
# k8s kubeadm 一键自动化，安装k8s集群，安装所有运行需要的组件
kmaster  192.168.230.210  kubectl, kubelet, kube-proxy, docker
knode1   192.168.230.211 kubectl, kubelet, kube-proxy, docker
knode1   192.168.230.212 kubectl, kubelet, kube-proxy, docker
```

### 2.系统环境初始化
> 三台节点配置好主机名及IP地址即可，系统环境配置分别通过脚本完成

```
[root@kmaster ~]# sh Stream8-k8s-v1.27.0.sh 
[root@knode1 ~]# sh Stream8-k8s-v1.27.0.sh 
[root@knode2 ~]# sh Stream8-k8s-v1.27.0.sh 

```
脚本文件如下：
```
[root@kmaster ~]# cat Stream8-k8s-v1.27.0.sh 
#!/bin/bash
# CentOS stream 8 install kubenetes 1.27.0
# the number of available CPUs 1 is less than the required 2
# k8s 环境要求虚拟cpu数量至少2个
# 使用方法：在所有节点上执行该脚本，所有节点配置完成后，复制第11步语句，单独在master节点上进行集群初始化。
#1 rpm
echo '###00 Checking RPM###'
yum install -y yum-utils vim bash-completion net-tools wget
echo "00 configuration successful ^_^"
#Basic Information
echo '###01 Basic Information###'
hostname=`hostname`
hostip=$(ifconfig ens160 |grep -w "inet" |awk '{print $2}')
echo 'The Hostname is:'$hostname
echo 'The IPAddress is:'$hostip

#2 /etc/hosts
echo '###02 Checking File:/etc/hosts###'
hosts=$(cat /etc/hosts)
result01=$(echo $hosts |grep -w "${hostname}")
if [[ "$result01" != "" ]]
then
	echo "Configuration passed ^_^"
else
	echo "hostname and ip not set,configuring......"
	echo "$hostip $hostname" >> /etc/hosts
	echo "configuration successful ^_^"
fi
echo "02 configuration successful ^_^"

#3 firewall & selinux
echo '###03 Checking Firewall and SELinux###'
systemctl stop firewalld
systemctl disable firewalld
se01="SELINUX=disabled"
se02=$(cat /etc/selinux/config |grep -w "^SELINUX")
if [[ "$se01" == "$se02" ]]
then
	echo "Configuration passed ^_^"
else
	echo "SELinux Not Closed,configuring......"
	sed -i 's/^SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
	echo "configuration successful ^_^"
fi
echo "03 configuration successful ^_^"

#4 swap
echo '###04 Checking swap###'
swapoff -a
sed -i "s/^.*swap/#&/g" /etc/fstab
echo "04 configuration successful ^_^"

#5 docker-ce
echo '###05 Checking docker###'
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
echo 'list docker-ce versions'
yum list docker-ce --showduplicates | sort -r
yum install -y docker-ce
systemctl start docker 
systemctl enable docker
cat <<EOF > /etc/docker/daemon.json
{
  "registry-mirrors": ["https://cc2d8woc.mirror.aliyuncs.com"]
}
EOF
systemctl restart docker
echo "05 configuration successful ^_^"

#6 iptables
echo '###06 Checking iptables###'
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sysctl -p /etc/sysctl.d/k8s.conf
echo "06 configuration successful ^_^"

#7 cgroup(systemd/cgroupfs)
echo '###07 Checking cgroup###'
containerd config default > /etc/containerd/config.toml
sed -i "s#registry.k8s.io/pause#registry.aliyuncs.com/google_containers/pause#g" /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
systemctl restart containerd
echo "07 configuration successful ^_^"

#8 kubenetes.repo
echo '###08 Checking repo###'
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
echo "08 configuration successful ^_^"

#9 crictl
echo "Checking crictl"
cat <<EOF > /etc/crictl.yaml 
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 5
debug: false
EOF
echo "09 configuration successful ^_^"

#10 kube1.27.0
echo "Checking kube"
yum install -y kubelet-1.27.0 kubeadm-1.27.0 kubectl-1.27.0 --disableexcludes=kubernetes
systemctl enable --now kubelet
echo "10 configuration successful ^_^"
echo "Congratulations ! The basic configuration has been completed"

#11 Initialize the cluster
# 仅在master主机上做集群初始化
# kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version=v1.27.0 --pod-network-cidr=10.244.0.0/16

```

### 3.集群部署
#### 3.1初始化集群（仅在kmaster节点做）
```
[root@kmaster ~]# # kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version=v1.27.0 --pod-network-cidr=10.244.0.0/16
```

#### 3.2配置环境变量（仅kmaster节点做）
```
[root@kmaster ~]# mkdir -p $HOME/.kube
[root@kmaster ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@kmaster ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
[root@kmaster ~]# echo 'export KUBECONFIG=/etc/kubernetes/admin.conf' >> /etc/profile
[root@kmaster ~]# source /etc/profile

[root@kmaster ~]# kubectl get node
NAME      STATUS     ROLES           AGE     VERSION
kmaster   NotReady   control-plane   5m37s   v1.29.2
```

### 将工作节点加入集群
> 3.1初始化集群的操作后面的结果,f分别在knode1和knode2执行
```
#直接生成token，用一下命令
[root@kmaster ~]# kubeadm token create --print-join-command
kubeadm join 192.168.230.210:6443 --token 5n1nfv.jvy6ltevszz557au --discovery-token-ca-cert-hash sha256:7198d5a0615ff1cb18e1000143bf36ec77631a5d8a604128c39ac13270ec186e 
[root@kmaster ~]# 

kubeadm join 192.168.230.210:6443 --token 6vvi4j.ug2nlgqtl69s9hs6         --discovery-token-ca-cert-hash sha256:7c9f9019a4df5c7377e6f6841fed4de011061501dd18557097b99bbe8a119cfb
```

#### 安装 Tigera Calico operator 另一个文档tigera-operator-3-26-1.yaml
> 安装 calico 网络（仅master节点）
```
kubectl create -f tigera-operator-3-26-1.yaml
```

#### 配置 custom-resources.yaml 仅master做
```
[root@kmaster ~]# kubectl create -f custom-resources-3-26-1.yaml
```


```

[root@kmaster ~]# cat custom-resources-3-26-1.yaml 
# This section includes base Calico installation configuration.
# For more information, see: https://projectcalico.docs.tigera.io/master/reference/installation/api#operator.tigera.io/v1.Installation
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    # Note: The ipPools section cannot be modified post-install.
    ipPools:
    - blockSize: 26
      cidr: 10.244.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()

---

# This section configures the Calico API server.
# For more information, see: https://projectcalico.docs.tigera.io/master/reference/installation/api#operator.tigera.io/v1.APIServer
apiVersion: operator.tigera.io/v1
kind: APIServer
metadata:
  name: default
spec: {}
[root@kmaster ~]# 

```
#### 动态查看calico容器状态，待全部running后，集群状态变为正常
```
[root@kmaster ~]# watch kubectl get pods -n calico-system

Every 2.0s: kubectl get pods -n calico-system                                                                      kmaster: Sun Apr 14 22:06:17 2024

NAME                                      READY   STATUS    RESTARTS	   AGE
calico-kube-controllers-bf58f78b6-jbs8h   1/1     Running   3 (177m ago)   179d
calico-node-4l6pq                         1/1     Running   3 (173m ago)   179d
calico-node-r68jx                         1/1     Running   3 (177m ago)   179d
calico-node-t2h5v                         1/1     Running   3 (174m ago)   179d
calico-typha-777b6b9fc6-24s7r             1/1     Running   3 (174m ago)   179d
calico-typha-777b6b9fc6-sw7w8             1/1     Running   3 (173m ago)   179d
csi-node-driver-g7wf2                     2/2     Running   6 (173m ago)   179d
csi-node-driver-tx4km                     2/2     Running   6 (177m ago)   179d
csi-node-driver-x4f74                     2/2     Running   6 (174m ago)   179d
```


#### 获取 Kubernetes 集群中所有节点的信息。在 Kubernetes 中，节点是集群中的工作节点，用于运行应用程序和容器。
```
[root@kmaster ~]# kubectl get node
NAME      STATUS   ROLES           AGE    VERSION
kmaster   Ready    control-plane   179d   v1.27.0
knode1    Ready    <none>          179d   v1.27.0
knode2    Ready    <none>          179d   v1.27.0


```

#### 获取 Kubernetes 集群中所有命名空间（namespace）中的 Pod 的信息。
> 在 Kubernetes 中，Pod 是最小的调度单位，它可以包含一个或多个容器，并且共享网络命名空间和存储卷。Pod 在 Kubernetes 集群中运行着应用程序、微服务等工作负载。
```
[root@kmaster ~]#  kubectl get pod -A
NAMESPACE          NAME                                      READY   STATUS    RESTARTS       AGE
calico-apiserver   calico-apiserver-6c6fc7b5c4-2lfmd         1/1     Running   3 (178m ago)   179d
calico-apiserver   calico-apiserver-6c6fc7b5c4-9gwvh         1/1     Running   3 (178m ago)   179d
calico-system      calico-kube-controllers-bf58f78b6-jbs8h   1/1     Running   3 (3h2m ago)   179d
calico-system      calico-node-4l6pq                         1/1     Running   3 (178m ago)   179d
calico-system      calico-node-r68jx                         1/1     Running   3 (3h2m ago)   179d
calico-system      calico-node-t2h5v                         1/1     Running   3 (178m ago)   179d
calico-system      calico-typha-777b6b9fc6-24s7r             1/1     Running   3 (178m ago)   179d
calico-system      calico-typha-777b6b9fc6-sw7w8             1/1     Running   3 (178m ago)   179d
calico-system      csi-node-driver-g7wf2                     2/2     Running   6 (178m ago)   179d
calico-system      csi-node-driver-tx4km                     2/2     Running   6 (3h2m ago)   179d
calico-system      csi-node-driver-x4f74                     2/2     Running   6 (178m ago)   179d
kube-system        coredns-7bdc4cb885-pt56d                  1/1     Running   3 (3h2m ago)   179d
kube-system        coredns-7bdc4cb885-zkzhx                  1/1     Running   3 (3h2m ago)   179d
kube-system        etcd-kmaster                              1/1     Running   3 (3h2m ago)   179d
kube-system        kube-apiserver-kmaster                    1/1     Running   3 (3h2m ago)   179d
kube-system        kube-controller-manager-kmaster           1/1     Running   3 (3h2m ago)   179d
kube-system        kube-proxy-r6tqg                          1/1     Running   3 (178m ago)   179d
kube-system        kube-proxy-rzfs7                          1/1     Running   3 (178m ago)   179d
kube-system        kube-proxy-xp7lv                          1/1     Running   3 (3h2m ago)   179d
kube-system        kube-scheduler-kmaster                    1/1     Running   3 (3h2m ago)   179d
tigera-operator    tigera-operator-5f4668786-dp89z           1/1     Running   7 (178m ago)   179d
[root@kmaster ~]# 



```

#### 查看镜像
> 用于查看容器运行时（Container Runtime Interface，CRI）中镜像的命令。CRI 是 Kubernetes 使用的标准接口，用于与容器运行时（如 Docker、containerd、CRI-O 等）进行通信。crictl 是一个用于与 CRI 兼容容器运行时进行交互的命令行工具。
```
[root@kmaster ~]# crictl images
IMAGE                                                             TAG                 IMAGE ID            SIZE
docker.io/calico/cni                                              v3.26.1             9dee260ef7f59       93.4MB
docker.io/calico/csi                                              v3.26.1             677ad13d73108       8.91MB
docker.io/calico/kube-controllers                                 v3.26.1             1919f2787fa70       32.8MB
docker.io/calico/node-driver-registrar                            v3.26.1             c623084712495       11MB
docker.io/calico/node                                             v3.26.1             8065b798a4d67       86.6MB
docker.io/calico/pod2daemon-flexvol                               v3.26.1             092a973bb20ee       7.29MB
registry.aliyuncs.com/google_containers/coredns                   v1.10.1             ead0a4a53df89       16.2MB
registry.aliyuncs.com/google_containers/etcd                      3.5.7-0             86b6af7dd652c       102MB
registry.aliyuncs.com/google_containers/kube-apiserver            v1.27.0             6f707f569b572       33.4MB
registry.aliyuncs.com/google_containers/kube-controller-manager   v1.27.0             95fe52ed44570       31MB
registry.aliyuncs.com/google_containers/kube-proxy                v1.27.0             5f82fc39fa816       23.9MB
registry.aliyuncs.com/google_containers/kube-scheduler            v1.27.0             f73f1b39c3fe8       18.2MB
registry.aliyuncs.com/google_containers/pause                     3.6                 6270bb605e12e       302kB
registry.aliyuncs.com/google_containers/pause                     3.9                 e6f1816883972       322kB
[root@kmaster ~]# 


```

tips:
```

Tigera Calico 和 Flannel 都是 Kubernetes 集群中常见的网络解决方案，用于实现 Pod 之间的通信和网络连接。它们之间的区别主要体现在以下几个方面：

网络模型：

Tigera Calico：Calico 使用了一种基于 BGP (Border Gateway Protocol) 的网络模型，每个节点都有一个独立的路由器，它们之间通过 BGP 协议进行路由信息的交换。这种模型可以实现灵活的、高性能的网络互联，适用于大规模的、跨多个数据中心的 Kubernetes 集群。
Flannel：Flannel 使用了一种 Overlay 网络模型，它通过在节点之间创建虚拟网络（如 VXLAN、GRE 等）来实现 Pod 的通信。Flannel 提供了简单、易于部署的解决方案，适用于小型或中型规模的 Kubernetes 集群。
路由方式：

Tigera Calico：Calico 使用基于 IP 路由的方式来转发数据包，每个节点都会为 Pod 分配一个独立的 IP 地址，并通过路由表来进行数据包的转发。
Flannel：Flannel 使用 Overlay 网络来传输数据包，它将 Pod 的 IP 包装在 Overlay 网络的数据包中，然后通过底层网络进行传输。
网络策略：

Tigera Calico：Calico 提供了丰富的网络策略（Network Policies）功能，允许管理员定义和控制 Pod 之间的流量规则，实现细粒度的网络隔离和安全性。
Flannel：Flannel 本身并不提供网络策略功能，但可以与其他网络策略实现（如 Calico 的网络策略）配合使用，以实现网络隔离和安全性。
总的来说，Tigera Calico 适用于大规模、高性能、需要复杂网络策略的 Kubernetes 集群，而 Flannel 更适合于小型或中型规模的集群，以及对网络性能要求不那么高、不需要复杂网络策略的场景。


```


```

https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart
```


### 问题
> 镜像原因
> calico/cni镜像一直下载不成功
```
#将 image: docker.io/calico/cni:v3.25.0 更改为其他镜像源
kubectl edit daemonset calico-node -n kube-system
image: quay.io/calico/cni:v3.25.0



```