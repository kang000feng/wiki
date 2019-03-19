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


####
', 	           "", 			   '', 			   '

## 参考内容
1. Kieker官网： http://kieker-monitoring.net/download/
