# DevOps初探

## 背景

之前虽然对DevOps的概念和流程大概有了一些了解，但是最近准备和学弟们一起把DevOps流水线给搭起来。所以查找一些资料并且进行相关实践。

## 概念

（1）.Pipeline（流水线）：一个Pipeline可以包含多个stage，比如build，test，code check，deploy等。Commit或者Merge会触发一个Pipeline。

（2）Stage（阶段）：一个pipeline中的stage按顺序执行，一旦有一个失败则此pipeline失败。

（3）Jobs（任务）：一个stage有多个job，会并行执行，一旦有一个失败此stage失败。



## 理解

1. DevOps是一种文化：
2.

## 相关工具

Draft：https://draft.sh/

Jenkins X：https://jenkins-x.io/

skaffold：http://storage.googleapis.com/skaffold/releases/v0.18.0/docs/index.html



## 参考资料

1. 博客：https://blog.csdn.net/chengzi_comm/article/details/78778284
2. Jenkins x系列实践：https://www.cnblogs.com/xiaoqi/p/jenkins-x-part1.html
3. Jenkins x：https://feisky.gitbooks.io/kubernetes/apps/jenkinsx.html
