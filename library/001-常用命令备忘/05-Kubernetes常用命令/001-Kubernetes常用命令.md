# Kubernetes常用命令

## Kubeadm 安装 Kubernetes
```
# 重置集群
kubeadm reset
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1
rm -rf /var/lib/cni/


```


## kubectl
```
# 查看集群状态
kubectl get pod -n kube-system

# 查看所有service,service后面的s可有可无
kubectl get service(s)

# 查看所有pod
kubectl get pods

# 查看所有service
kubectl get service


# 创建资源
kubectl create -f [xxx.yaml]
# -f 后可以指定文件夹，此文件夹下所有yaml文件都会会执行创建
kubectl create -f dir/


# 删除某个已经创建的服务
kubectl delete service [service name]
# 删除所有服务
kubectl delete services --all
# 删除所有pod
kubectl delete pods --all
# 删除所有deployment
kubectl delete deployments --all
# 删除所有PersistentVolumeClaim
kubectl delete pvc --all

```


## 安装helm
```
# 更换源
helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

```

## 参考资料
1. Kubernetes中文文档：http://docs.kubernetes.org.cn/490.html
