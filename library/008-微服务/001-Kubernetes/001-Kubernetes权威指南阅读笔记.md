# Kubernetes 权威指南

### 写在阅读之前
在去研究生所在的实验室之前，我是从未听说过Kubernetes这个东西的。所以现在相当于是完全从零开始学习这个东西，希望能够在研究生入学之前整整从头到尾把这本书过一遍吧。千里之行，始于足下，加油吧~

### 第一章 Kubernetes 是什么
Kubernetes是一个全新的基于容器技术的分布式架构领先方案。  
开始是对Kubernetes整体的介绍（以后简称Kubernates为k8s），很多名词还是比较陌生，开头这一段通读以后开始正式搭一个本地K8s集群环境，然后在本地简单跑一个项目来感受一下k8s的使用场景。  
https://www.kubernetes.org.cn/3787.html  
k8s中文社区  

Kubernates具有完备的集群管理能力，包括多层次的安全防护和准入机制、多租户应用制程能力、透明的服务注册和服务发现机制、内建智能负载均衡器、强大的故障发现和自我修复能力李、服务滚动升级和在线扩容能力、可扩展的资源自动调度机制以及多粒度的资源配额管理能力。  

Pod运行再Node中，一个Node中通常有数百个Pod，每个Pod中除了一个特殊的Pause容器外其余的为业务容器，其它容器共享 网络栈和Voulume挂载卷，只有提供服务的一组Pod才会被映射到Service，。

集群方面，K8s集群分为一个Master结点和一群工作节点，mater上运行着kube-apiserver,kube-contoller-manager和kube-scheduler，负责实现整个集群的资源管理，pod调度，弹性伸缩，安全控制，系统监控和纠错等功能。Node上运行着kubelet，kube-proxy等进程，负责Pod的创建，启动，监控，重启，销毁以及实现软件模式的负载均衡器。

服务扩容和升级则由Replication Controller来完成。

#### 环境准备


```
// 关闭系统防火墙
systemctl disable firewalld
systemctl stop firewalld

//安装etcd和kubernetes
yum install -y etcd kubernetes

修改Docker配置文件（/etc/sysconfig/docker）,把OPTION设置为：
OPTIONS='--setlinux-enabled=false --insecure-registry gcr.io'

修改Kubernetes apiserver配置文件为 /etc/kubernates/apiserver，删除--admission_control参数的ServiceAccount。

//启动所有的服务
systemctl start etcd
systemctl start docker
//执行此命令时如果运行出错注意8080端口是否被占用，如果被占用则修改/etc/kubernetes/apiserver中的端口号，并且取消该行注释。
systemctl start kube-apiserver
systemctl start kube-controller-manager
systemctl start kube-scheduler
systemctl start kubelet
systemctl start kube-proxy
```

至此，基本的单机集群就基本搭建完成了。

#### 启动MySQL服务

为MySQL服务创建一个RC文件：

```mysql-rc.yaml（去掉注释）
apiVersion: v1
# 声明类型
kind: ReplicationController
# RC名称（全局唯一）
metadata:
  name: mysql
spec:
  # 期待副本数量
  replicas: 1
  selector:
    # 符合目标的Pod拥有此标签
    app: mysql
  # 根据此模板创建pod副本
  template:
    metadata:
      # Pod副本拥有的标签，对应RC的selector
      labels:
        app: mysql
    spec:
      #Pod内容器的定义部分
      containers:
        # 容器的名称
      - name: mysql
        # 容器对应的Docker Image
        image: mysql:5.6
        ports:
          # 容器暴露的二端口号
        - containerPort: 3306
        # 注入到容器的环境变量
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "123456"  
```

其中必要的注释都已经加了，详细的内容写完之后应该有一定的认识和理解，这里就不一一详细写了。

之后执行如下命令：


```
kubectl create -f mysql-rc.yaml

```
创建成功

如果出现错误

```
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
```
则执行

```
export KUBECONFIG=/etc/kubernetes/admin.conf
```
接下来使用命令查看rc：

```
#查看RC状态
kubectl get rc

#查看pod
kubectl get pods

#查看镜像
docker ps -a | grep mysql

注意：如果是多机器的集群，那么master使用docker ps -a找不到mysql，因为它会被部署到某一个node上
```
然后创建一个与之关联的Kubernete Service，即MySQL的定义文件


```mysql-service.yaml
apiVersion: v1
kind: Service
metadata:
    name: mysql
spec:
    ports:
        - port: 3306
    selector:
        app: mysql
```
之后创建服务

```
kubectl create -f mysql-service.yaml

#查看服务
kubectl get svc
```

#### 启动Tomcat应用
同样是定义rc文件：

```myweb-rc.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myweb
spec:
  replicas: 5
  selector:
    app: myweb
  template:
    metadata:
      labels:
        app: myweb
    spec:
      containers:
      - name: myweb
        image: kubeguide/tomcat-app:v1
        ports:
        - containerPort: 8080
        env:
        - name: MYSQL_SERVICE_HOST
          value: "mysql"
        - name: MYSQL_SERVICE_PORT
          value: "3306"
```
执行的命令就不写了

之后再继续创建tomcat的service文件:

```tomcat-service.yaml
apiVersion: v1
kind: Service
metadata:
    name: myweb
spec:
    type: NodePort
    ports:
        - port: 8080
          nodePort:30001
    selector:
        app: myweb
```

其中的type: NodePort和nodePort:30001表明开启了NodePort的外网访问模式，在集群之外可以通过这个端口来进行访问.

很有可能遇到的错误是无法连接MySQL，错误原因时MySQL的版本问题，把前面的mysql image版本加上5.6就可以了。

#### Kubernetes基本概念和术语

##### Master
Master整个集群的控制节点，所有控制命令都在Master上执行。如果Master一旦宕机，那么所有的控制命令都将失效。

**Kubernetes API Server(kube-apiserver)**:提供了HTTP Rest的关键服务进程，是k8s集群所有资源的增删改查操作的唯一入口。  
**Kubernetes Controller Manager（kube-controller-manager）**:k8s所有资源对象的自动化控制中心。  
**Kubernetes Scheduler(kube-scheduler)**:负责资源调度的进程

##### Node
Node可以是一台物理机或者虚拟机，Master会给每一个Node分配一定的工作负载，当某个Node宕机时，其上的工作负载会被Master自动转移到其它节点。  
每个Node上运行着以下一组进程：  
**kubelet**：负责Pod对应的容器的创建、启停等。同时和Master合作完成集群的管理功能。  
**kube-proxy**：负责Kubernetes Service的通信与负载均衡机制的组件。  
**Docker Engine(docker)**:负责本机容器创建和管理。

#### Pod
Pod是K8s
