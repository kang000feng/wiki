# Kieker的学习与使用
## Kieker是什么
> Kieker is a Java-based application performance monitoring and dynamic software analysis framework.  

官方文档回答了这个问题。  
Kieker是一个基于Java实现的程序性能监控和动态软件分析的框架。

## 获取官方文档和示例
Kieker的文档非常齐全，所以建议直接阅读Kieker的官方文档学习如何使用Kieker。虽然是英文文档，但是阅读起来没什么难度不大，照着来就行。  
根据参考内容[1]可以下载到源代码以及官方文档，下面我将按照官方文档的章节进行学习和实验。

## Quick Start
根据2.1~2.4的内容，将示例的两个quick-start项目跑了起来，并且得到了运行结果。这里的监控是手动来做的，也就是说在代码内部新建对应的监控和分析实例来完成输出，文档中提到后面会有注解等其它方式来实现，这个等后面提到了再看。  
就从目前quick-start运行的效果来看，效果还一般，接下来第3章是每个模块的介绍。

## Kieker.Monitoring Component

### Monitoring Controller
MonitoringController可以构造和控制Kieker.Monitoring 实例。它提供了以下功能：
> 1. Creating MonitoringController Instances.
2. Logging monitoring records with the configured monitoring writer.
3. Retrieving the current time via the configured time source.
4. Scheduling and removing period samplers.
5. Controlling the monitoring state.
6. Activating and deactivating probes at runtime.

### Kieker.Monitoring Configuration
讲了Kirker.Monitoring 相关的配置，暂时不细看，后面真正用到或者遇到问题再来这里看。

### Monitoring Record
讲了Monitoring Record相关的存储和应用等，同理后面再看。

## Kieker.Analysis Component

## Kieker.TraceAnalysis Tool
这个是比较重要的一部分内容，需要仔细看。  

Kieker.TraceAnalysis实现了Kieker的特殊功能，允许监视，分析和可视化（分布式）方法执行的痕迹和相应的时序信息。

5.1章节描述如何监控Java应用程序。讲了如何利用Aspect来实现的探针，并且探针可以扩展。5.2介绍了可用于分析和可视化记录的跟踪数据的工具。5.3介绍Kieker.TraceAnalysis的例子和可视化输出。

### Monitoring Trace Information
这一部分主要讲的是怎么使用Kieker提供的基于AspectJ，Java Servlet API，Spring Framework和Apache CXF实现的监控探针。  

#### AspectJ-Based Instrumentation
> AspectJ [10] allows to weave code into the byte code of Java applications and libraries without requiring manual modifications of the source code.  

看到这句话觉得有戏啊，好像可以实现无侵入式的监控。

> When the probes with name element Annotation are used, methods to be monitored must be annotated by the Kieker annotation @OperationExecutionMonitoringProbe.

但是看到这句话就有点虚了，我不是很理解什么叫做“name element Annotation”，当用这个时就需要加注解，那么这个是做什么用的，不用可以么，先往后看吧。  

因为一些原因暂时没有继续往下看，先动手开始尝试使用。

## 使用

### 尝试

接下来尝试了使用AspectJ，Spring等方式的Demo，都跑了起来并且收集到了监控结果。  
剩下就是将Demo的成功演示复制到自己的项目上，目前这种方式几乎是没有侵入性的，如果能够这样实现的话就再好不过了。  

新建maven项目，使用Demo代码，导入相关依赖执行成功。  

新建Spring Boot项目，改动Demo代码导入缺少的几个依赖执行成功，排除Spring Boot依赖的问题。

SpringBoot，SpringMVC项目使用自己代码添加配置和依赖后运行没有效果，目前猜想是运行方式的差异问题或者是包目录的问题，明天继续看。

新建与目录，把示例几个类移动至对应文件夹下，运行能够获取到监控数据，可以排除目录结构的影响，那么导致一般Spring Boot项目和Spring MVC项目获取不到监控数据的原因很可能是因为Tomcat运行的原因，接下来再仔细查看一下。

仔细想来这确实应该是容器的关系：普通的Java类知道直接调用，但是容器内部却是使用了一系列初始化和监听操作（具体的操作我也不太清楚），那从这个角度出发来看，Spring方式的监控就显得有些鸡肋了：现在使用Spring的项目基本都使用了Tomcat，Jetty或者类似的容器。或许看到后面会发现有别的用处吧，现在应该看的是使用Servlet的方式来进行监控的方法。或许到后面两者可以结合一下也说不定。

现在想来使用AspectJ方式应该也是仅限于普通的Java应用吧，使用了容器的应该也是监控不了的。

### JavaEE Servlet Container Example
这个例子使用的容器是Jetty，并且使用了基于JavaEE Servlet API，Spring和AspectJ的探针，233刚刚的猜测都被这句话打脸了。原来基于容器的应用三种方式都能够使用，接下来具体运行尝试一下吧。这三种探针用于监控执行，条用路径和会话数据（execution,trace and session data）。

> The example is prepared to use two alternative types of Kieker probes: either the Kieker Spring interceptor (default) or the Kieker AspectJ aspects. Both alternatives additionally use Kieker’s Servlet filter.

从这句话的来看，貌似是Spring or AspectJ + Servlrt的方式，后面尝试并且确认一下。

尝试之后发现确实是这个样子，使用Spring方式方式添加aop配置后，还要在web.xml里面配置一下filter，配置好之后就可以拿到监控数据。但是Spring Boot项目可能是因为配置有些问题，尚且没有拿到运行数据。

Spring Boot项目尝试引入xml配置，可以获取到监控数据了。

下一步应该尝试的是使用AspectJ的方式看是否能够获取到监控数据。

尝试使用AspectJ方式时，发现按照文档中的命令行编译并且执行就能够正常运行，但是当自己在IDE里面编译运行时，就出现了错误，和在SpringMVC中尝试使用AspectJ方式出现的的错误一模一样。
```
三月 25, 2019 10:02:26 上午 kieker.monitoring.core.controller.WriterController newQueue
警告: An exception occurred
java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at kieker.monitoring.core.controller.WriterController.newQueue(WriterController.java:190)
	at kieker.monitoring.core.controller.WriterController.<init>(WriterController.java:92)
	at kieker.monitoring.core.controller.MonitoringController.<init>(MonitoringController.java:63)
	at kieker.monitoring.core.controller.MonitoringController.createInstance(MonitoringController.java:82)
	at kieker.monitoring.core.controller.MonitoringController$LazyHolder.<clinit>(MonitoringController.java:362)
	at kieker.monitoring.core.controller.MonitoringController.getInstance(MonitoringController.java:355)
	at kieker.monitoring.probe.aspectj.flow.operationExecution.AbstractAspect.<clinit>(AbstractAspect.java:42)
	at com.intellij.rt.execution.application.AppMainV2$Agent.premain(AppMainV2.java:153)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at sun.instrument.InstrumentationImpl.loadClassAndStartAgent(InstrumentationImpl.java:386)
	at sun.instrument.InstrumentationImpl.loadClassAndCallPremain(InstrumentationImpl.java:401)
Caused by: org.aspectj.lang.NoAspectBoundException: kieker.monitoring.probe.aspectj.flow.operationExecution.FullInstrumentationNoGetterAndSetter
	at kieker.monitoring.probe.aspectj.flow.operationExecution.FullInstrumentationNoGetterAndSetter.aspectOf(FullInstrumentationNoGetterAndSetter.java:1)
	at org.jctools.util.Pow2.roundToPowerOfTwo(Pow2.java:1)
	at org.jctools.queues.ConcurrentCircularArrayQueue.<init>(ConcurrentCircularArrayQueue.java:43)
	at org.jctools.queues.MpscArrayQueueL1Pad.<init>(MpscArrayQueue.java:28)
	at org.jctools.queues.MpscArrayQueueProducerIndexField.<init>(MpscArrayQueue.java:54)
	at org.jctools.queues.MpscArrayQueueMidPad.<init>(MpscArrayQueue.java:76)
	at org.jctools.queues.MpscArrayQueueProducerLimitField.<init>(MpscArrayQueue.java:103)
	at org.jctools.queues.MpscArrayQueueL2Pad.<init>(MpscArrayQueue.java:125)
	at org.jctools.queues.MpscArrayQueueConsumerIndexField.<init>(MpscArrayQueue.java:151)
	at org.jctools.queues.MpscArrayQueueL3Pad.<init>(MpscArrayQueue.java:178)
	at org.jctools.queues.MpscArrayQueue.<init>(MpscArrayQueue.java:199)
	... 18 more

Exception in thread "main" java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at sun.instrument.InstrumentationImpl.loadClassAndStartAgent(InstrumentationImpl.java:386)
	at sun.instrument.InstrumentationImpl.loadClassAndCallPremain(InstrumentationImpl.java:401)
Caused by: java.lang.ExceptionInInitializerError
	at kieker.monitoring.core.controller.MonitoringController.getInstance(MonitoringController.java:355)
	at kieker.monitoring.probe.aspectj.flow.operationExecution.AbstractAspect.<clinit>(AbstractAspect.java:42)
	at com.intellij.rt.execution.application.AppMainV2$Agent.premain(AppMainV2.java:153)
	... 6 more
Caused by: java.lang.IllegalStateException: java.lang.reflect.InvocationTargetException
	at kieker.monitoring.core.controller.WriterController.newQueue(WriterController.java:202)
	at kieker.monitoring.core.controller.WriterController.<init>(WriterController.java:92)
	at kieker.monitoring.core.controller.MonitoringController.<init>(MonitoringController.java:63)
	at kieker.monitoring.core.controller.MonitoringController.createInstance(MonitoringController.java:82)
	at kieker.monitoring.core.controller.MonitoringController$LazyHolder.<clinit>(MonitoringController.java:362)
	... 9 more
Caused by: java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at kieker.monitoring.core.controller.WriterController.newQueue(WriterController.java:190)
	... 13 more
Caused by: org.aspectj.lang.NoAspectBoundException: kieker.monitoring.probe.aspectj.flow.operationExecution.FullInstrumentationNoGetterAndSetter
	at kieker.monitoring.probe.aspectj.flow.operationExecution.FullInstrumentationNoGetterAndSetter.aspectOf(FullInstrumentationNoGetterAndSetter.java:1)
	at org.jctools.util.Pow2.roundToPowerOfTwo(Pow2.java:1)
	at org.jctools.queues.ConcurrentCircularArrayQueue.<init>(ConcurrentCircularArrayQueue.java:43)
	at org.jctools.queues.MpscArrayQueueL1Pad.<init>(MpscArrayQueue.java:28)
	at org.jctools.queues.MpscArrayQueueProducerIndexField.<init>(MpscArrayQueue.java:54)
	at org.jctools.queues.MpscArrayQueueMidPad.<init>(MpscArrayQueue.java:76)
	at org.jctools.queues.MpscArrayQueueProducerLimitField.<init>(MpscArrayQueue.java:103)
	at org.jctools.queues.MpscArrayQueueL2Pad.<init>(MpscArrayQueue.java:125)
	at org.jctools.queues.MpscArrayQueueConsumerIndexField.<init>(MpscArrayQueue.java:151)
	at org.jctools.queues.MpscArrayQueueL3Pad.<init>(MpscArrayQueue.java:178)
	at org.jctools.queues.MpscArrayQueue.<init>(MpscArrayQueue.java:199)
FATAL ERROR in native method: processing of -javaagent failed
```
如此看来，只要解决了这个问题那么其他问题就能够迎刃而解了。

尝试了一下，这个问题产生的根本原因还是因为编译结果的路不对，aop.xml以及kieker.monitoring.properties无法找到，尝试着修改编译结果的路径就可以了。
aop.xml以及kieker.monitoring.properties必须在class编译结果同级的META-INF文件夹下面，位置不对，文件夹名字不对都会出现错误。
那么尝试之着在SpringMVC中使用AspectJ的方式来进行监控。这里我就有个小问题了，Spring监控的方式还好说，简单配置后就能使用，但是现在使用AspectJ的方式，既要配置，同时还需要在JVM参数里面指定javaagent，而且貌似这个配置方式是可以简化的，有没有比较方便点的方法呢，再仔细看看文档吧。

spring项目还是无法使用，仍然报无法找到配置文件的错误。不过因为工作催着的原因，所以不深究下去了。暂且使用spring 方式来进行监控。

### Trace Analysis and Visualization
这一部分讲的是如何利用Kieker提供的工具对监控的数据进行分析和可视化。  
使用Kieker可视化工具的前提是安装两个可视化工具：  

1. Graphviz：A graph visualization software.
2. GNU PlotUtils：A set of tools for generating 2D plot graphics.
3. ps2pdf： The ps2pdf tool is used to convert ps files to pdf files.

下载安装这三个工具,Ubuntu系统可以使用下面命令进行安装，其它发行版或者Windows系统请见参考内容[2][3]:
```shell
sudo apt-get update
sudo apt-get install graphviz
sudo apt-get install plotutils
sudo apt-get install ghostscript
```

之后执行命令：
```
chmod 777 ./bin/trace-analysis.sh

 ./bin/trace-analysis.sh -inputdirs /code/micoservice/jeesite/trace-data/kieker-20190326-061219-103288581501467-UTC--KIEKER-SINGLETON -outputdir /code/micoservice/jeesite/trace-visualization/ -plot-Deployment-Sequence-Diagrams -plot-Call-Trees –short-labels

sudo apt-get install texlive-extra-utils  

 chmod 777 ./bin/dotPic-fileConverter.sh
 ./bin/dotPic-fileConverter.sh /code/micoservice/jeesite/trace-visualization/ pdf png

```


## 杂记

Spring容器和MVC容器可以设置各自的context package以及proxy方式，其中父容器spring无法访问子容器spring mvc的内容，子容器可以访问父容器的内容，在context:component-scan标签内添加use-default-filters="false"即可过过滤掉父容器中扫描过内容。

productuin_ssm项目已经可以确认spring容器中使用jdk作为代理，springmvc子容器中使用cglib作为代理，controller中所以private方法改为public方法即可正常监控到所有数据。

jessite项目问题也已经确认，因为dao层mybatis接口动态生成的class是final类型的，所以不能使用cglib动态代理（因为cglib动态代理的原理是继承生成一个子类，所以不能代理fianl class。）但是service层jessite没有使用接口，直接在controller层autowrie实现类，所以使用jdk动态代理会报autowried期望类型与实际不符的错误，也就是说dao层和service层前者必须使用jdk代理，后者必须使用cglib代理，但是两者都在父容器中扫描配置的，只能使用一种代理方式，所以目前可以考虑的解决方法为：  
1. 为jeesite所有service添加对应接口，然后在controller层autowrie接口而不是实现类，这样spring父容器中就可以使用jdk代理来完成监控。
2. 将dao层扫描配置或者service层配置放到springmvc子容器中，spring父容器和springmvc子容器可以采取不同的代理方式，这样可以实现监控。  
方式1需要改动大量代码，改动量虽然大但是改动方式简单明了，结果可以预测。  
方式2需要大幅改动配置文件，改动量虽然相对较小，但是改动之后难以预料会对系统产生什么影响。

## 参考内容
1. Kieker官网： http://kieker-monitoring.net/download/
2. Graphviz： http://www.graphviz.org/
3. GNU PlotUtils：http://www.gnu.org/software/plotutils/  http://gnuwin32.sourceforge.net/packages/plotutils.htm
