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
暂无
