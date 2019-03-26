# 使用Kieker监控项目

## 前言
因实验室工作需要，了解到了Kieker这个工具，在经过几天的摸索、尝试以及阅读文档后，今天正式使用Kieker来监控一个开源项目，以获取项目中所以函数的调用路径。

## 实验对象
准备监控的对象是Jessite，它是一个开源的企业信息化开发平台，在Github上拥有不少star，是我们目前选择的试验系统。  
Github地址：https://github.com/thinkgem/jeesite
目前我已经将其fork至我的Github，并且新建分支kieker，对其进行监控配置。监控方式我选用Servlet + Spring的方式。

## 运行项目
按照说明文档，首先新建数据库jessite，并且修改jeesite.properties文件夹中数据库地址与密码。  
之后执行db目录下的jeesite_mysql.sql文件中的sql语句。  
虽然报了不少外检检查的错误，不过大概业务逻辑应该是没有问题的，下面开始部署监控。  
这是一个SpringMVC的项目，配置起来不是很麻烦。

## 配置监控
### 添加依赖
这是个maven项目，因此在pom文件中添加：
```xml

<dependency>
			<groupId>org.aspectj</groupId>
			<artifactId>aspectjweaver</artifactId>
			<version>1.9.2</version>
			<scope>runtime</scope>
		</dependency>

		<dependency>
			<groupId>net.kieker-monitoring</groupId>
			<artifactId>kieker</artifactId>
			<version>1.13</version>
			<scope>runtime</scope>
		</dependency>

```
当然，Kieker监控依赖的jar包不止当前这些，但是其它的依赖项目中已经包含，所以只需要添加上述两个依赖即可。全部依赖请参考Kieker官方文档。

### 配置filter
在web.xml文件最后添加servlet过滤。
```xml
<!-- 配置Kieker监听 -->
<filter>
  <filter-name>
    sessionAndTraceRegistrationFilter
  </filter-name>
  <filter-class>
    kieker.monitoring.probe.servlet.SessionAndTraceRegistrationFilter
  </filter-class>
  <init-param>
    <param-name>logFilterExecution</param-name>
    <param-value>true</param-value>
  </init-param>
</filter>

<filter-mapping>
  <filter-name>sessionAndTraceRegistrationFilter</filter-name>
  <url-pattern>/∗</url-pattern>
</filter-mapping>
```

### 配置aop
这个项目spring.xml文件有很多个，在spring-mvc.xml中添加：
```xml
<bean id="opEMII" class="kieker.monitoring.probe.spring.executions.OperationExecutionMethodInvocationInterceptor" />

<aop:aspectj-autoproxy />

<!--注意！如果service目录变了，需要改这里的内容，否则会报错-->
<!--proxy-target-class 为flase，被代理类是基于jdk的动态代理，如果为true，被代理类基于cglib动态代理-->
<aop:config proxy-target-class="true">
	<!-- 只监控 com.thinkgem.jeesite下面所有public方法-->
	<aop:advisor advice-ref="opEMII" pointcut="execution(public * com.thinkgem.jeesite..*.*(..))"/>
	<!-- 监控所有public方法 -->
	<!--<aop:advisor advice-ref="opEMII" pointcut="execution(public * *(..))"/>-->
</aop:config>
```
如果aop标签报错，则在xml文件头部的bean后面加上，具体添加的位置看一下应该就明白：
```xml

xmlns:aop="http://www.springframework.org/schema/aop"
<!-- 若此标签已有，则合并 -->
xsi:schemaLocation="
		http://www.springframework.org/schema/aop
		http://www.springframework.org/schema/aop/spring-aop.xsd"
```


### 设置JVM参数

最后在配置中添加JVM参数
```
-Dkieker.monitoring.writer.filesystem.AsciiFileWriter.customStoragePath=yourpath
```
设置这个参数的目的是为了指定输出的监控数据的路径，后面的yourpath应该修改为存储输出监控数据文件的目录的绝对路径，因为项目在容器中运行，所以相对路径是无效的。

## 参考内容：
1. Kieker User Guide 1.13： https://github.com/kieker-monitoring/kieker/releases/download/1.13/kieker-1.13-userguide.pdf
2. https://alanli7991.github.io/2016/10/21/%E5%88%87%E9%9D%A2%E7%BC%96%E7%A8%8B%E4%B8%89AspectJ%E4%B8%8EShiro%E4%B8%8D%E5%85%BC%E5%AE%B9%E5%92%8CSpring%E4%BA%8C%E6%AC%A1%E4%BB%A3%E7%90%86%E9%94%99%E8%AF%AF%E5%88%86%E6%9E%90/
