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



## 流程调研

现在DevOps有很多实现方式，常见的工具链如下图所示：

![DevOps工具链](http://www.jamesbowman.me/post/cdlandscape/ContinuousDeliveryToolLandscape-fullsize.jpeg)

其中我所知的比较常用的工具链是：

代码仓库：Github/GitLab

依赖管理：Maven/Gradle/Npm

容器、镜像管理：Docker/DockerHub

CI服务器：Jenkins/Travis

部署/运行：k8s

在查找了一些资料之后，我们发现了基于Jenkins的Jenkins X项目，它是一个基于Jenkins和k8s的CI/CD平台，主要解决微服务架构下云原生应用的持续集成和继续交付的问题，它使用Jenkin、Helm、Draft、GitOps以及Github等工具链够早了一个从集群安装、环境管理、持续集成、持续部署一直到应用发布等支持整个流程的平台。[3]

从它的介绍中可以看出，它和我们所熟知的工具链的不同之处在于，它是完全基于k8s实现的，并且把集群管理也整合到了流程之中，并且能够与Git相整合，也引出了几种全新的工具用以实现整个流程。

因为我们的工作主要是探索性的实践，本身没有太多创造性，为了让这次探索更加有意义，我们更应该尝试一些比较新的实践，总结出来才会更加有意义。而且，我们本身也是基于k8s的微服务应用，这套工具也非常契合我们的需求。

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
vi /lib/systemd/system/docker.service
```

找到ExecStart这一行，在末尾添加 ：--insecure-registry=10.0.0.0/16

```shell
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --insecure-registry=10.0.0.0/16
```

之后重启Docker服务：

```
systemctl daemon-reload
service docker restart
```

### 安装Helm2.x[6]

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

使用helm2.0也可以选择不安装tiller，执行

```shell
helm init --client-only
```

然后直接执行后面命令。

#### 安装tiller（可选）

使用helm部署tiller：

```shell
# 注意，这里制定了镜像仓库，如果不指定默认从gcr.io拉去镜像，可能遇到下面错误。
helm init --service-account tiller   --tiller-image registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.11.0 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```

#### 注意

> kubernetes1.16 版本在安装helm 2.14.0或之前版本时，将一直报错：
>
> Error: error installing: the server could not find the requested resource
>
> 具体原因可见Github helm仓库的issue：
>
> https://github.com/helm/helm/issues/6374
>
> 该issue中提供了临时的解决方法，在helm 2.15.0版本发布之前，可以先按照此issue中方法来解决。
>
> 执行以下命令来进行初始化：
>
> ```shell
> helm init --service-account tiller --tiller-image registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.14.3 --output yaml | sed 's@apiVersion: extensions/v1beta1@apiVersion: apps/v1@' | sed 's@  replicas: 1@  replicas: 1\n  selector: {"matchLabels": {"app": "helm", "name": "tiller"}}@' | kubectl apply -f -
> ```

查看一下tiller是否成功运行：

```shell
kubectl get pods -n kube-system
```

一般来讲会正常运行：

```shell
tiller-deploy-75f5747884-qvhqf   1/1     Running   0          19s
```

如果helm仓库无法拉取镜像，可以采用如下命令来更新仓库地址：

```shell
helm init --client-only --stable-repo-url https://aliacs-app-catalog.oss-cn-hangzhou.aliyuncs.com/charts/
helm repo add incubator https://aliacs-app-catalog.oss-cn-hangzhou.aliyuncs.com/charts-incubator/
helm repo update
```

#### 可能存在的问题

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

### 安装Helm3.x

helm 2.x在k8s1.16版本上会有一些问题，因此这里准备尝试安装helm3.0。

同样先下载helm安装包：

```shell
# 如果安装了旧版本的helm,执行下面命令来删除tiller和helm工具
helm reset -f
rm -rf /usr/local/bin/helm

# 可以下载，但速度较慢，建议耐心等待
wget https://get.helm.sh/helm-v3.0.0-beta.3-linux-amd64.tar.gz
tar -zxvf helm-v3.0.0-beta.3-linux-amd64.tar.gz
mv ./linux-amd64/helm /usr/local/bin/

# 查看helm版本
helm version
```



### 安装Draft

按照官网文档[10]来安装Draft：

```shell
# 这个速度依然很慢，但是因为文件不大，倒是可以等会儿
wget https://azuredraft.blob.core.windows.net/draft/draft-v0.16.0-linux-amd64.tar.gz
# 解压
tar -xzvf draft-v0.16.0-linux-amd64.tar.gz
# 命令行工具移至系统路径
sudo mv linux-amd64/draft /usr/local/bin/draft
```

如果未安装git，需先安装git

```shell
sudo yum install git -y
```

然后配置draft

```shell
draft init
```

### 安装jenkins X

```shell
# 如果安装的helm2
jx install --provider=kubernetes --domain icedsoulk8s.cn --on-premise
# 如果安装的helm3
jx install --provider=kubernetes --helm3 --domain icedsoulk8s.cn --on-premise
```

这个需要等待一段时间。

在这个过程中，可以设置DNS域名，设置Github账号，添加Github token。

在遇到选择

```shell
> Serverless Jenkins X Pipelines with Tekon
  Static Jenkins Server and Jenkinsfiles
```

关于它们的不同可以见[14]：

从中可以了解到Serverless Jenkins X Pipelines是Jenkins X的主推方式，以后也有研究的必要。

但是目前，我们的主要目的还是先把完整的CI/CD流程快速搭建起来，后续再考虑做这些更加激进的探索，所以目前暂时采用Static Jenkins Server的方式，写出来的Jenkinfile也更容易推广和识别。



后续可能会报错：

```
no matches for kind "Deployment" in version "extensions/v1beta1"
```

这是kubernetes1.16的锅，Deployment已经从extensions/v1beta1中移除，加入到了正式库apps/v1，按照[15]即可临时解决。



中间可能会出现镜像拉不下来的问题，全部手动拉一下：

三个node上执行：

```
docker pull googlecontainer/exposecontroller:2.3.111
docker pull xinglongjian/builder-jx:2.0.851-206
docker pull googlecontainer/heapster:v1.5.2
docker pull googlecontainer/addon-resizer:1.7
docker pull xinglongjian/jenkinsx:0.0.80
docker pull xinglongjian/nexus:0.1.7
docker pull xinglongjian/jx:2.0.645

docker tag googlecontainer/exposecontroller:2.3.111 gcr.io/jenkinsxio/exposecontroller:2.3.111
docker tag xinglongjian/builder-jx:2.0.851-206 gcr.io/jenkinsxio/builder-jx:2.0.842-199
docker tag googlecontainer/heapster:v1.5.2 k8s.gcr.io/heapster:v1.5.2
docker tag xinglongjian/jenkinsx:0.0.80 gcr.io/jenkinsxio/jenkinsx:0.0.80
docker tag xinglongjian/nexus:0.1.7 gcr.io/jenkinsxio/nexus:0.1.7
docker tag xinglongjian/jx:2.0.645 gcr.io/jenkinsxio/jx:2.0.645
docker tag googlecontainer/addon-resizer:1.7 k8s.gcr.io/addon-resizer:1.7


```



镜像拉取下来之后，可能还会遇到几个pod pending的状态：

```shell
[root@master ~]# kubectl get pods
NAME                                           READY   STATUS    RESTARTS   AGE
jenkins-5744dd7454-n8p8j                       0/1     Pending   0          20s
jenkins-x-chartmuseum-774f8b95b-6r89x          0/1     Pending   0          20s
jenkins-x-controllerrole-99b87c4b8-grfsl       1/1     Running   0          20s
jenkins-x-controllerteam-5447bb4db8-8v6r7      1/1     Running   0          20s
jenkins-x-controllerworkflow-f79544447-6jchn   1/1     Running   0          20s
jenkins-x-docker-registry-dbf6889cd-pxkl9      0/1     Pending   0          20s
jenkins-x-heapster-78fd4ccdc9-psjkc            2/2     Running   0          20s
jenkins-x-nexus-b87fb7f7b-s4m4l                0/1     Pending   0          20s
```

查看了一下，实际pending的原因是因为pvc处于pending状态，没有对应的pv：

```shell
[root@master ~]# kubectl get pvc
NAME                        STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
jenkins                     Pending                                                     2d17h
jenkins-x-chartmuseum       Pending                                                     2d17h
jenkins-x-docker-registry   Pending                                                     2d17h
jenkins-x-nexus             Pending                                                     2d17h
```

然后在jx的github仓库的issue中可以找到有人有类似的提问[16]：

在此issue的讨论中，确认了问题是缺少pv导致的，根据此issue中讨论的解决方法，确认在本地安装时需要创建StorgeClass来确保PVC可以绑定PV，具体可见[17]，操作过程请参考[18]



注意：

如果加了--on-premise，不会出现以下情况。

如果安装过程中出现：

```shell
ingress init failed: service jxing-nginx-ingress-controller never became ready
```

则需要按照下面说明自行安装Ingress Controller。

如果没有出现错误，则可以跳过下面安装Ingress的过程，因为在jx安装过程中会自动安装Ingress。

### 安装Ingress Controller

jx本身也是使用Helm安装的Ingress，因此按照[11]中方式来排查错误。

最终发现问题是没有指定ingress-controller的EXTERNAL-IP

```shell
# 先查看上一步的残留
helm list
# 删除之前的安装不成功的chart
helm delete --purge jxing
# 执行安装，此处选用externalIP方式，IP可以更改为想要使用的IP
helm install stable/nginx-ingress --name jxing --set "rbac.create=true,controller.service.externalIPs[0]=10.141.212.21" --namespace kube-system
```

这里我注意到jenkins X之所以自动安装失败是因为jxing-nginx-ingress-controller这个service一直处于pending状态，根据[12]中高票答案的说法，因为我们没有指定域名或者NodePort，所以参考[11]中的做法以及[12]官方文档的介绍，使用如上命令，自己指定externalIP，建立nginx-ingress。

成功之后，重新执行：

```shell
jx install --provider=kubernetes --domain icedsoulk8s.cn --on-premise
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
10. Draft quickstart guide：https://github.com/Azure/draft/blob/master/docs/quickstart.md
11. 利用Helm部署Ingress：https://blog.csdn.net/bbwangj/article/details/82863042
12. StackOverFlow Q:kubernetes service external ip pending ：https://stackoverflow.com/questions/44110876/kubernetes-service-external-ip-pending
13. Helm Hub nginx-ingress: https://hub.helm.sh/charts/stable/nginx-ingress
14. Jenkinx X piplines engine：https://jenkins-x.io/news/jenkins-x-next-gen-pipeline-engine/
15. Jenkinx issue:support for kubernetes1.16：https://github.com/jenkins-x/jx/issues/5675
16. Github-jx-issue:https://github.com/jenkins-x/jx/issues/1317
17. Kubernetes StrogeClass:https://kubernetes.io/docs/concepts/storage/storage-classes/
18. PV,PVC,NFS:https://wiki.icedsoul.cn/?file=008-%E5%BE%AE%E6%9C%8D%E5%8A%A1/001-Kubernetes/004-Kubernetes%E7%9B%B8%E5%85%B3%E7%9F%A5%E8%AF%86/001-PV%20PVC

flux-GitOps：https://github.com/fluxcd/flux