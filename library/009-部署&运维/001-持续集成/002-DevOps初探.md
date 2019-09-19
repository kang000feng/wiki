# DevOps初探

## 背景

之前虽然对DevOps的概念和流程大概有了一些了解，但是最近准备和学弟们一起把DevOps流水线给搭起来。所以查找一些资料并且进行相关实践。

## 概念

（1）.Pipeline（流水线）：一个Pipeline可以包含多个stage，比如build，test，code check，deploy等。Commit或者Merge会触发一个Pipeline。

（2）Stage（阶段）：一个pipeline中的stage按顺序执行，一旦有一个失败则此pipeline失败。

（3）Jobs（任务）：一个stage有多个job，会并行执行，一旦有一个失败此stage失败。



## 理解

1. DevOps是一种文化：



## 构建模式的选择

当在微服务架构中实现CI（持续集成），CD（持续交付）时，需要考虑如何建立代码到CI流水线和最终线上环境的映射。

目前有三种选择[5]：

1. 所有服务的代码在同一个仓库中，一旦有Commit那么直接整体重新Build，执行所有服务对应的Test，并且将所有服务都重新Deploy。
2. 所有服务的代码在同一个仓库中，一旦有Commit，分析这次Commit所涉及的服务，这些服务分别进行Build、Test和Deploy。
3. 每个服务的代码分别放到不同的仓库，每个服务的仓库的Commit触发其对应的Build、Test和Deploy。

毫无疑问，第一种方案是最容易实现，但是代价很大的一种方案，是我们所无法接受的。每次提交所有服务都重新Build和Test，这需要花很长时间，占用很多资源。

第二种方案优点在于代码集中容易管理，每个服务单独的Pipeline也都能分开走，且代价相对第一种较小，但是这种方案导致代码提交时容易服务耦合，比如某次Commit修改多个服务，这种做法通常是不被建议的。

第三种方案优点在于不会出现一次Commit涉及多个服务，每个服务Pipeline也能分开走。但是缺点是代码不容易集中管理。

在真正企业实践中，肯定会选择第三种方案，因为不同服务往往由团队负责，天然就是适合分库管理的。但是trainticket作为开源项目，代码需要集中管理，而且第三种方案的优势相对来说不是很大，我们可以通过开发规范来限制这种情况的出现。所以，考虑到开源项目的限制，我们选择使用第二种构建模式。

## 相关工具

Draft：https://draft.sh/

Jenkins X：https://jenkins-x.io/

skaffold：http://storage.googleapis.com/skaffold/releases/v0.18.0/docs/index.html

工具介绍：https://www.cnblogs.com/xiaoxi-song/p/9635202.html

## Jenkins X部署[3]

安装jx命令行工具：

```shell
curl -L https://github.com/jenkins-x/jx/releases/download/v1.1.10/jx-linux-amd64.tar.gz | tar xzv
sudo mv jx /usr/local/bin
```

按照文档说明，安装Jenkins X之前需要k8s集群先开启rbac并且开启insecure docker，

### k8s开启rbac

rbac(role-based access controll) [4]:

```shell
# 查看apiserver的设置文件，- --authorization-mode=Node,RBAC有rbac字样代表开启了rbac
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

因为我的集群是使用kubeadm安装的，所以默认开启了rbac。

### 开启insecure docker registries

然后开启insecure docker registries：

```shell
# 先停止docker服务
service docker stop
# 执行如下命令
dockerd --insecure-registry=10.0.0.0/16
```

（此处执行失败，有何影响暂时未知，继续向下执行）

### 安装Helm[6]

如果k8s集群未安装Helm，则需按照[7]中的说明安装Helm，Helm分为客户端命令行工具以及服务端Tiller组成，此处我选择安装helm的2.14.3版本：

```shell
# 官网下载速度有些慢，建议耐心等待或寻找镜像下载
curl -O https://get.helm.sh/helm-v2.14.3-linux-amd64.tar.gz
tar -zxvf helm-v2.14.3-linux-amd64.tar.gz
cd linux-amd64/
cp helm /usr/local/bin/
```

接下来安装Helm的服务端tiller，需要使用service account分配合适的角色，这里直接分配了cluster-admin角色：

```shell
vi helm-rbac.yaml
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

```shell
kubectl create -f helm-rbac.yaml
```

使用helm部署tiller：

```shell
# 注意，这里制定了镜像仓库，如果不指定默认从gcr.io拉去镜像，可能遇到下面错误。
helm init --service-account tiller   --tiller-image registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.11.0 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```

查看一下tiller是否成功运行：

```shell
kubectl get pods -n kube-system
```

一般来讲会正常运行：

```shell
tiller-deploy-75f5747884-qvhqf   1/1     Running   0          19s
```



如果执行init命令时不指定镜像地址，或者想要使用的版本镜像仓库不存在，可能会出现如下情况：

```shell
tiller-deploy-8557598fbc-8ntlb   0/1     ImagePullBackOff   0          10m
```

仔细查看一下原因：

```shell
kubectl describe pods tiller-deploy-8557598fbc-8ntlb -n kube-system
Name:           tiller-deploy-8557598fbc-8ntlb
Namespace:      kube-system
Priority:       0
Node:           node-1/10.141.211.176
Start Time:     Wed, 18 Sep 2019 13:17:44 -0400
Labels:         app=helm
                name=tiller
                pod-template-hash=8557598fbc
Annotations:    <none>
Status:         Pending
IP:             10.244.2.63
Controlled By:  ReplicaSet/tiller-deploy-8557598fbc
Containers:
  tiller:
    Container ID:   
    Image:          gcr.io/kubernetes-helm/tiller:v2.14.3
    Image ID:       
    Ports:          44134/TCP, 44135/TCP
    Host Ports:     0/TCP, 0/TCP
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Liveness:       http-get http://:44135/liveness delay=1s timeout=1s period=10s #success=1 #failure=3
    Readiness:      http-get http://:44135/readiness delay=1s timeout=1s period=10s #success=1 #failure=3
    Environment:
      TILLER_NAMESPACE:    kube-system
      TILLER_HISTORY_MAX:  0
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from tiller-token-xzz2s (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  tiller-token-xzz2s:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  tiller-token-xzz2s
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason          Age                             From               Message
  ----     ------          ----                            ----               -------
  Normal   Scheduled       13m                             default-scheduler  Successfully assigned kube-system/tiller-deploy-8557598fbc-8ntlb to node-1
  Normal   SandboxChanged  <invalid>                       kubelet, node-1    Pod sandbox changed, it will be killed and re-created.
  Warning  Failed          <invalid> (x3 over <invalid>)   kubelet, node-1    Failed to pull image "gcr.io/kubernetes-helm/tiller:v2.14.3": rpc error: code = Unknown desc = Error response from daemon: Get https://gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
  Warning  Failed          <invalid> (x3 over <invalid>)   kubelet, node-1    Error: ErrImagePull
  Warning  Failed          <invalid> (x7 over <invalid>)   kubelet, node-1    Error: ImagePullBackOff
  Normal   Pulling         <invalid> (x4 over <invalid>)   kubelet, node-1    Pulling image "gcr.io/kubernetes-helm/tiller:v2.14.3"
  Normal   BackOff         <invalid> (x37 over <invalid>)  kubelet, node-1    Back-off pulling image "gcr.io/kubernetes-helm/tiller:v2.14.3"
```

发现错误主要原因为无法成功拉取镜像，参考了[8]中的解决方案：通过DockerHub找到想要的tiller版本，拉到本地后，再把镜像标签换回“gcr.io"。

我在DockerHub找了一个比较合适的镜像：

```shell
docker pull imdingtalk/tiller:v2.14.3
```

接下来更改标签：

```shell
docker tag docker.io/imdingtalk/tiller:v2.14.3 gcr.io/kubernetes-helm/tiller:v2.14.3
```

然后重新进行初始化：

```shell
# 删除tiller
kubectl get all -n kube-system -l app=helm -o name|xargs kubectl delete -n kube-system

helm init --service-account tiller --skip-refresh
```

### 安装jenkins X

```shell
jx install --provider=kubernetes
```



## 参考资料

1. 博客：https://blog.csdn.net/chengzi_comm/article/details/78778284
2. Jenkins x系列实践：https://www.cnblogs.com/xiaoqi/p/jenkins-x-part1.html
3. Jenkins x文档：https://feisky.gitbooks.io/kubernetes/apps/jenkinsx.html
4. k8s rbac : https://kubernetes.io/docs/reference/access-authn-authz/rbac/
5. 微服务设计 Sam Newnab著 崔力强等译 中国工信出版社
6. Helm官网：https://helm.sh/
7. Helm安装说明：https://helm.sh/docs/using_helm/#installing-helm
8. kubernetes1.13.1集群安装包管理工具helm：https://blog.51cto.com/jerrymin/2346902
9. Helm包管理工具介绍：https://www.cnblogs.com/xzkzzz/p/10445807.html
