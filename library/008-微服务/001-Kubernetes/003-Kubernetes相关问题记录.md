# Kubernetes相关问题记录
## 安装
### 安装报错
#### 错误内容
安装Kubernetes，执行
```
kubeadm init kubeadm init --pod-network-cidr=10.244.0.0/16
```
报错：
```
invalid or incomplete external CA: failure loading certificate for API server: failed to load certificate: couldn't load the certificate file /etc/kubernetes/pki/apiserver.crt: open /etc/kubernetes/pki/apiserver.crt: no such file or directory
```
#### 解决方法
报错对应文件找不到，那么进入上面文件夹，查看里面的文件：
```
cd /etc/kubernetes/pki
ls
```

可以看到此文件夹下只有一个ca.crt文件，确实不存在上述文件，查阅官方文档[1]可知，ca文件是自动创建的，所以我尝试删除当前ca文件，然后重新init，不再报次错误。

但是接下来又报了这个错误：
```
error execution phase kubeconfig/kubelet: a kubeconfig file "/etc/kubernetes/kubelet.conf" exists already but has got the wrong CA cert
```

然后我生气了，直接执行  
```
rm -rf /etc/kubernetes
```
然后上面的错误没有了，然后出现了新的错误：
```
failed to run Kubelet: failed to create kubelet: misconfiguration: kubelet cgroup driver: "cgroupfs" is different from docker cgroup driver: "systemd"
```
这个可以看[2]里面说明，修改kubelet的cgroup为systemed
```
vi /etc/default/kubelet

KUBELET_EXTRA_ARGS=--cgroup-driver=systemed

systemctl daemon-reload
systemctl restart kubelet
```
再次init之前先执行
```
kubeadm reset
```
之后仍然报错：
```
The kubelet is not running
	- The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)
```
查看kubelet日志后发现问题在于：
```
node "master" not found
```
找了一阵资料发现是kubeadm的ip地址忘记改成自己的了，修改之后再次运行就没错了。

### 运行报错

#### 重启失败
过了两天，执行kubectl get pods命令发现失效，过了很长时间报错：
```
Error from server (Timeout): the server was unable to return a response in the time allotted, but may still be processing the request (get pods)
```
然后使用docker ps -a查看了一下，果然几个相关服务正在重启，但是重启失败，查看日志后发现重启失败的原因是端口占用，仔细看了一下发现：
k8s_POD开头的几个镜像仍在正在运行，重启的镜像是不包含POD标签的同名镜像。所以新镜像一直启动失败

#### 解决方法
找了一阵子资料发现没什么好办法，直接重新安装。

#### Master报错
使用kubectl命令发现报错：
```
The connection to the server 10.141.212.21:6443 was refused - did you specify the right host or port?
```
然后使用docker ps -a 看了一下，所有服务都启动失败了，查找资料后发现是因为swap没有永久禁用。

#### 解决方法
永久禁用swap：
```shell
cp -p /etc/fstab /etc/fstab.bak$(date '+%Y%m%d%H%M%S')

sed -i "s/\/dev\/mapper\/centos-swap/\#\/dev\/mapper\/centos-swap/g" /etc/fstab

mount -a

free -m
cat /proc/swaps
```
之后服务会自动重启。

#### hosts文件自动恢复
又发现了一个神奇的问题，hosts文件每过一会儿就自动回复原样，气死我了。  
没找到相关资料。

这个问题应该是挖矿病毒。。。垃圾挖矿病毒！！！



### kubeadm init拉取镜像失败

设置拉取dockerhub镜像：

```shell
kubeadm config print init-defaults ClusterConfiguration >kubeadm.conf
```

修改其中的

```
imageRepository: imdingtalk
```

这是dockerhub一个同步gcr.io镜像的仓库，截至1.16版本都有效。

或者直接拉取镜像然后改标签：

```
sudo docker pull imdingtalk/kube-controller-manager:v1.16.0
sudo docker pull imdingtalk/kube-apiserver:v1.16.0
sudo docker pull imdingtalk/kube-proxy:v1.16.0
sudo docker pull imdingtalk/kube-scheduler:v1.16.0
sudo docker pull imdingtalk/etcd:3.3.15-0
sudo docker pull imdingtalk/pause:3.1
sudo docker pull imdingtalk/coredns:1.6.2

sudo docker pull imdingtalk/kube-controller-manager:v1.15.3
sudo docker pull imdingtalk/kube-apiserver:v1.15.3
sudo docker pull imdingtalk/kube-proxy:v1.15.3
sudo docker pull imdingtalk/kube-scheduler:v1.15.3
sudo docker pull imdingtalk/etcd:3.3.10
sudo docker pull imdingtalk/pause:3.1
sudo docker pull imdingtalk/coredns:1.3.1
```



成功拉取镜像后还是改回gcr.io的标签吧。

```shell
sudo docker tag imdingtalk/kube-controller-manager:v1.16.0 k8s.gcr.io/kube-controller-manager:v1.16.0
sudo docker tag imdingtalk/kube-apiserver:v1.16.0 k8s.gcr.io/kube-apiserver:v1.16.0
sudo docker tag imdingtalk/kube-proxy:v1.16.0 k8s.gcr.io/kube-proxy:v1.16.0
sudo docker tag imdingtalk/kube-scheduler:v1.16.0 k8s.gcr.io/kube-scheduler:v1.16.0
sudo docker tag imdingtalk/etcd:3.3.15-0 k8s.gcr.io/etcd:3.3.15-0
sudo docker tag imdingtalk/pause:3.1 k8s.gcr.io/pause:3.1
sudo docker tag imdingtalk/coredns:1.6.2 k8s.gcr.io/coredns:1.6.2

sudo docker tag imdingtalk/kube-controller-manager:v1.15.3 k8s.gcr.io/kube-controller-manager:v1.15.3
sudo docker tag imdingtalk/kube-apiserver:v1.15.3 k8s.gcr.io/kube-apiserver:v1.15.3
sudo docker tag imdingtalk/kube-proxy:v1.15.3 k8s.gcr.io/kube-proxy:v1.15.3
sudo docker tag imdingtalk/kube-scheduler:v1.15.3 k8s.gcr.io/kube-scheduler:v1.15.3
sudo docker tag imdingtalk/etcd:3.3.10 k8s.gcr.io/etcd:3.3.10
sudo docker tag imdingtalk/pause:3.1 k8s.gcr.io/pause:3.1
sudo docker tag imdingtalk/coredns:1.3.1 k8s.gcr.io/coredns:1.3.1
```



#### 解决方法
莫名重启几次就。。还是那样，但是，集群正常了？？  
天知道为什么，不深究了。



### kubernetes 1.16 cni-flannel安装失败

困扰我一个晚上，更换机器反复验证后确认，此为1.16版本bug

见：https://github.com/kubernetes/kubernetes/issues/82997

明天回退。



## kubeadm init时报错

报错内容：

```
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get http://localhost:10248/healthz: dial tcp: lookup localhost on 202.120.224.6:53: no such host.
```

原因是host文件被修改之后不能解析localhost。。。。明明之前改了host不会有这个问题的，真让人生气。

在/etc/hosts文件中加上localhost之后就好了。。



## Kubelet启动失败

目前kubelet启动失败的原因是centos版本问题，centos7.2版本的kubelet已确认无法启动。

使用如下命令查看系统版本。

```shell
cat /etc/redhat-release
```



## 参考资料
1. Kubadm PKI certificates and requirements：https://kubernetes.io/docs/setup/best-practices/certificates/
2. 安装Kubeadm：https://kubernetes.io/zh/docs/setup/independent/install-kubeadm/
