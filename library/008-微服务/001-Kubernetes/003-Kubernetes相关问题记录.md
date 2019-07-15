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


#### 报错
过了两天，执行kubectl get pods命令发现失效，过了很长时间报错：
```
Error from server (Timeout): the server was unable to return a response in the time allotted, but may still be processing the request (get pods)
```
然后使用docker ps -a查看了一下，果然几个相关服务正在重启，但是重启失败，查看日志后发现重启失败的原因是端口占用，仔细看了一下发现：
k8s_POD开头的几个镜像仍在正在运行，重启的镜像是不包含POD标签的同名镜像。所以新镜像一直启动失败

#### 解决方法
找了一阵子资料发现没什么好办法，直接重新安装。

## 参考资料
1. Kubadm PKI certificates and requirements：https://kubernetes.io/docs/setup/best-practices/certificates/
2. 安装Kubeadm：https://kubernetes.io/zh/docs/setup/independent/install-kubeadm/
