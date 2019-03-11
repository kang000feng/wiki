# ServiceComb初体验
因为实验室工作原因知道了ServiceComb，需要去了解并且使用。今天先初步按照官方文档来安装并且使用一下。

## 入门
先暂且按照快速入门的步骤来，并且记录相关问题。  
### 环境准备
除了基本的git，jdk，maven，docker之外，还需要安装一下ServiceComb Java Chassis，emm，这是个啥？

```shell
git clone https://github.com/apache/servicecomb-java-chassis.git
 cd servicecomb-java-chassis
 mvn clean install -DskipTests
```
可能没有调整maven源的原因，安装这个要花很长时间，最好换成国内的源。  
更换maven源方法也很简单：
```shell
# 查看maven安装包路径
mvn -v
# 修改maven配置文件
vi 路径/conf/seeting.xml

# 在mirrors标签内添加

<mirror>  
    <id>nexus-aliyun</id>  
    <mirrorOf>central</mirrorOf>    
    <name>Nexus aliyun</name>  
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>  
</mirror>
```
不过还是安装失败了，看了一下觉得可能是jdk版本的问题，服务器安装了10.0.2版本的jdk，卸了换回1.8吧。  
换回1.8就好了 = =

### 运行Service Center
Service Center提供服务注册和服务发现的功能。这里只是使用它，所以直接用docker运行：
```shell
docker pull servicecomb/service-center
docker run -d -p 30100:30100 servicecomb/service-center:latest
```
这是去dockerhub拉去对应镜像并且运行在30100端口。

### 运行默认的示例
进入之前clone的servicecomb-java-chassis目录
```shell
cd ./samples/calculator
mvn spring-boot:run

cd ..
cd ./webapp
mvn spring-boot:run
```

## 参考内容
1. ServiceComb 官方文档 快速入门：http://servicecomb.apache.org/cn/docs/quick-start/
