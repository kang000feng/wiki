# Jenkins CI with Docker Compose
持续集成也算当前一个非常热门的技术，工业届有现成的工具，学术界也有不少相关的论文。之前因为课程原因读了几篇相关的论文，对持续集成的概念和作用也有了相关的了解，所以这里就不再说基础概念，直接说记录一下使用Docker Compose启动Jenkins的经验。

## 准备
安装好Docker和Docker-Compose

## 运行
新建文件夹，在文件夹下新建docker-compose-jenkins.yml文件
```yml
version: '3'
services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    restart: always
    ports:
      - 7777:8080
      - 7778:50000
    volumes:
      - /var/jenkins_home:/var/jenkins_home
```
这是使用docker-compose直接定义的方式运行的，也可以新建一个文件夹，在文件夹中写Dockerfile，在文件中拉取相关的镜像，这里暂且不研究。  
其中image使用的是官方提供的进项，port则是将jenkins运行的端口映射到物理机，volumes则是将容器中的相关配置放到物理机中以免丢失。  
不过这样直接运行会报错，因为默认进程对"/var/jenkins_home"文件夹是没有读写权限的。所以需要执行：
```shell
sudo chown -R 1000:1000 /var/jenkins_home/
```
之后在文件夹执行
```shell
docker-compose -f docker-compose-jenkins.yml up
```
就可以了，会出现如下内容：  
![](assets/009/20190129-e8858f37.png)  
其中圈起来那串字符需要留意一下。

## 配置
之后访问"http://[IP address]:7777"即可看到如下界面：  
![](assets/009/20190129-3337e0f8.png)  
在输入栏输入刚刚前面圈起来那串字符，即可进入选择界面：  
![](assets/009/20190129-e15c6190.png)  
因为我尚且是新手，不知道哪些插件是必要的哪些是不需要的，所以就选择第一个继续。等待一段时间，插件安装结束后就会出现设定管理员账号的界面：  
![](assets/009/20190129-57f8cc6e.png)  
输入账号密码设定之后，jenkins URL也可以根据自己的需要进行设定，我是选择了默认。

## 创建
打开创建新的任务，发现没有Maven选项，说明需要安装maven插件：  
![](assets/009/20190129-db5bc299.png)  

![](assets/009/20190129-a5fcb5bd.png)  
依次选择Jekins，系统管理，插件管理，之后选择Available，在右上角filter中输入 ：“maven integration”，选择对应项并且安装：  
![](assets/009/20190129-bcfa1ecd.png)  
再次回到新建任务，就会发现已经有了Maven项目的选项：  
![](assets/009/20190129-30283cd7.png)  

## 结语
到目前为止，我们成功地把Jenkins使用Docker-Compose启动并且运行了起来，之后我简单尝试了一下部署。选项比较多，不过可以看出来Jenkins还是非常强大的，之后我会先把WithMe这个项目给整理一下，先整理出来一套正常情况下部署和运行的流程，之后研究一下怎么使用Jenkins来自动化的完成这个过程。  
好好学习，天天向上，加油。

Jenkins 容器内执行宿主机docker

```
docker run -it -d  \
 --restart=always -u root \
 -v /usr/bin/docker:/usr/bin/docker \
 -v /usr/bin/docker-compose:/usr/bin/docker-compose \
 -v /var/run/docker.sock:/var/run/docker.sock \
 -v /usr/lib64/libltdl.so.7:/usr/lib/x86_64-linux-gnu/libltdl.so.7 jenkins/jenkins:lts
```





## 相关网址
- Jenkins 官网： https://jenkins.io/
- Docker Hub： https://hub.docker.com/_/jenkins
- 参考博客：
    1. 使用dockerc-ompose安装Jekins：https://www.jianshu.com/p/0fefc7e5cd0a
    2. Jenkins安装maven插件L：https://blog.csdn.net/qq_32218457/article/details/80775049
- 参考流程： https://github.com/bz51/SpringBoot-Dubbo-Docker-Jenkins
