# Kubetnetes集群搭建
## 2019.10.14更新

此文档参考意义已不大，请严格按照下面文档进行安装：

https://kuboard.cn/install/install-k8s.html#%E9%85%8D%E7%BD%AE%E8%A6%81%E6%B1%82

## 前言

已经不知道搭了多少次了 = =，各种失败，希望这次是最后一次搭建吧。而且今天发现K8s官网文档有中文了，以后查阅起来就更加方便了。不过只有极少数页面有中文，大多数还是英文，希望能够官方能够尽快翻译吧。  
从官方文档可以看出，1.14相较于之前的版本有所不同，具体可以在安装之后再感受一下。

## 准备
目前我准备了两台服务器，分别作为master，node1.
都已经预先安装好了Docker.
## 系统配置
### 设置主机名称
```
vi /etc/hosts
```
修改为
```
10.141.212.138 master
10.141.211.176 node1
```
### 禁用防火墙

```
systemctl stop firewalld
systemctl disable firewalld
```

### 禁用SETLINUX

```
setenforce 0

vi /etc/selinux/config
SELINUX=disabled
```

```
vi /etc/sysctl.d/k8s.conf

net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

modprobe br_netfilter
sysctl -p /etc/sysctl.d/k8s.conf
```
## 安装kubeadm，kubalet，kubectl
首先按照参考内容[1]进行相关的准备和安装工作，下面是我执行的用于Contos的安装命令。  
官方地址下载会因为墙拉不到镜像，所以我们使用国内阿里云的源
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable --now kubelet

```

执行之后即可安装最新版本的kubelet，kubectl和kubeadm,但是如果最新版本有问题，如1.16.0版本就会在centos上出现cni启动失败的错误，那么可以指定想要安装kubeadm等工具的版本。如：指定安装1.15.3版本的工具。

```shell
yum install -y kubelet-1.15.3 kubeadm-1.15.3 kubectl-1.15.3  --disableexcludes=kubernetes
```



之后按照说明执行：
```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
为了保证服务器节点在资源紧张情况下更加稳定，修改各个节点docker的cgroup driver为systemd

```
vi /etc/docker/daemon.json

{
  "exec-opts": ["native.cgroupdriver=systemd"]
}

systemctl restart docker

docker info | grep Cgroup
```

关闭swap
```
swapoff -a
```
## 初始化节点并且加入集群
执行下面命令：
```
# 各节点开机启动kubelet
systemctl enable kubelet.service
export KUBECONFIG=/etc/kubernetes/admin.conf
```

新建一个文件，存放配置

```
vi kubeadm.yaml

apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.141.212.138
  bindPort: 6443
nodeRegistration:
  taints:
  - effect: PreferNoSchedule
    key: node-role.kubernetes.io/master
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.15.0
networking:
  podSubnet: 10.244.0.0/16

kubeadm init --config kubeadm.yaml --ignore-preflight-errors=Swap
```
解决了一堆报错之后终于成功运行：
```
kubeadm join 10.141.212.138:6443 --token qqp314.8t4e7jgqzy425hu4 \
    --discovery-token-ca-cert-hash sha256:9d4800e904e780810fe276d10785df132a9031ea1e3e995914ba3ea3f4018dcc
```

配置用户查看集群：
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
查看集群状态是否正常：
```
kubectl get cs
```

## 安装Pod Network
```
mkdir -p ~/k8s/
cd ~/k8s
curl -O https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f  kube-flannel.yml
```

## 子结点加入集群
```
kubeadm join 10.141.212.138:6443 --token qqp314.8t4e7jgqzy425hu4 \
    --discovery-token-ca-cert-hash sha256:9d4800e904e780810fe276d10785df132a9031ea1e3e995914ba3ea3f4018dcc
    
# 10.141.211.176 master

kubeadm join 10.141.211.176:6443 --token o6kx0h.dap3mno2y80ahf5d \
    --discovery-token-ca-cert-hash sha256:1979057fdfc041ac7cdeab525b158c76b4f0d37a47cb23095fdb4fffd832e00b
```

至此成功。





## 尝试高可用离线安装



## 参考内容
1. K8s官网安装教程：https://kubernetes.io/docs/setup/independent/install-kubeadm/
