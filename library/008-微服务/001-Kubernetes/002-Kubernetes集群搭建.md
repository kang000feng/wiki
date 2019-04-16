# Kubetnetes集群搭建
## 前言
已经不知道搭了多少次了 = =，各种失败，希望这次是最后一次搭建吧。而且今天发现K8s官网文档有中文了，以后查阅起来就更加方便了。不过只有极少数页面有中文，大多数还是英文，希望能够官方能够尽快翻译吧。  
从官方文档可以看出，1.14相较于之前的版本有所不同，具体可以在安装之后再感受一下。

## 准备
目前我准备了四台服务器，分别作为master，node1，node2和node3.  
都已经预先安装好了Docker，版本分别是：18.06,18.06,17.03,17.03  
Docker版本可能会对后面的安装有所影响，但是这里暂且不修改，看看后面安装是否会出错。

## 安装kubeadm，kubalet，kubectl
首先按照参考内容[1]进行相关的准备和安装工作，下面是我执行的用于Contos的安装命令。
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable --now kubelet
```  

执行安装kubeadm，kubectl，kubelet命令之后，会发现因为被墙所以找不到镜像，但是它会自动寻找其它镜像，耐心等待之后会完成安装，但是安装出来的版本可能很是1.13或者更旧。这个时候重新执行一下上面的安装命令就会更新到1.14，天知道为什么（= =）。  
之后按照说明执行：
```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
cgroup什么的先不配吧，搞不懂是什么。。
## 初始化节点并且加入集群
执行下面命令：
```
export KUBECONFIG=/etc/kubernetes/admin.conf
```

## 参考内容
1. K8s官网安装教程：https://kubernetes.io/docs/setup/independent/install-kubeadm/
