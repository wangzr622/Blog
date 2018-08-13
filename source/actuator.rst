Spring Boot2 监控自定义实现
=========================================

1.actuator 简介
-----------------------------------------

actuator 是spring boot提供的一个监控应用的starter插件。提供了jmx 和 http两种访问方式。

在spring boot 2.x 中， actuator有了较大的改动.

首先pom依赖变为
::

	<dependency>
   		<groupId>org.springframework.boot</groupId>
   		<artifactId>spring-boot-starter-actuator</artifactId>
	</dependency>

在2.x版本中，http访问方式，spring团队默认只开放health和info断点。其中health也是只能看到应用的状态。

需要手动放开所有的端点，但是这会导致安全问题。不建议生产上这么做。

actuator开放的所有端点
::
	
	{
		_links: {
			self: {
				href: "http://localhost:8080/actuator",
				templated: false
			},
			auditevents: {
				href: "http://localhost:8080/actuator/auditevents",
				templated: false
			},
			beans: {
				href: "http://localhost:8080/actuator/beans",
				templated: false
			},
			health: {
				href: "http://localhost:8080/actuator/health",
				templated: false
			},
			conditions: {
				href: "http://localhost:8080/actuator/conditions",
				templated: false
			},
			configprops: {
				href: "http://localhost:8080/actuator/configprops",
				templated: false
			},
			env: {
				href: "http://localhost:8080/actuator/env",
				templated: false
			},
			env-toMatch: {
				href: "http://localhost:8080/actuator/env/{toMatch}",
				templated: true
			},
			info: {
				href: "http://localhost:8080/actuator/info",
				templated: false
			},
			loggers: {
				href: "http://localhost:8080/actuator/loggers",
				templated: false
			},
			loggers-name: {
				href: "http://localhost:8080/actuator/loggers/{name}",
				templated: true
			},
			heapdump: {
				href: "http://localhost:8080/actuator/heapdump",
				templated: false
			},
			threaddump: {
				href: "http://localhost:8080/actuator/threaddump",
				templated: false
			},
			metrics-requiredMetricName: {
				href: "http://localhost:8080/actuator/metrics/{requiredMetricName}",
				templated: true
			},
			metrics: {
				href: "http://localhost:8080/actuator/metrics",
				templated: false
			},
			scheduledtasks: {
				href: "http://localhost:8080/actuator/scheduledtasks",
				templated: false
			},
			httptrace: {
				href: "http://localhost:8080/actuator/httptrace",
				templated: false
			},
			mappings: {
				href: "http://localhost:8080/actuator/mappings",
				templated: false
			},
			jolokia: {
				href: "http://localhost:8080/actuator/jolokia",
				templated: false
			}
		}
	}

各个端点功能简介：

================  =====================================================================================
 路径               描述 

================  =====================================================================================
/auditevents      审计事件
/auditevents	  查看自动配置的使用情况 提供了一份自动配置报告，记录哪些自动配置条件通过了，哪些没通过	
/configprops      查看配置属性，包括默认配置 描述配置属性(包含默认值)如何注入Bean
/beans	          查看bean及其关系列表 描述应用程序上下文里全部的Bean，以及它们的关系
/dump	 	      打印线程栈,获取线程活动的快照	
/env	          查看所有环境变量	
/env/{name}	      根据名称获取特定的环境属性值	
/health	          查看应用健康指标
/heapdump	      打印堆信息
/info	          查看应用信息,这些信息由info打头的属性提供	
/loggers	 	 	
/loggers/{name}	 
/loggers/{name}	 	 	
/mappings	 	  查看所有url映射,以及它们和控制器(包含Actuator端点)的映射关系
/metrics	      度量	报告各种应用程序度量信息，比如内存用量和HTTP请求计数
/metrics/{name}	  度量	报告指定名称的应用程序度量值	
/shutdown	 	  关闭应用
/trace	 	      查看基本追踪信息,提供基本的HTTP请求跟踪信息(时间戳、HTTP头等)	

================  =====================================================================================


2.设计思路
-----------------------------------------

直接使用http方式访问，会返回一个Json串，对用户不是很友好，而且直接将http方式暴露给外部用户，对应用安全性来说不是很好的方案。

所以考虑重新设计自己的监控插件。包含以下功能：

-采集系统各种信息，包括cpu、内存、磁盘等基础信息
-监控界面展示
-供外部使用的数据

采集信息
>>>>>>>>>

通过阅读actuator源码，找到端点的接入层，直接在代码层面调用接入层。

actuator的接入层就是加'@Endpoint' 注解的类，当然也可以直接点用个方法中的源码。

这样便可以不使用http方式直接获取监控信息

缓存解耦
>>>>>>>>>

采集到信息以后，面临的主要问题就是使用这些信息。

信息的消费放主要是前端界面和其他应用。所以很有必要设计一个缓存来进行解耦操作。

缓存的主要作用就是将采集到的信息存储下来，然后展示给前端界面 以及 供第三方调用。

