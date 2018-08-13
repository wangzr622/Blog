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
+--------------+-----------------------------------------------------------------------------+
|  路径         | 描述                                                                        | 
+==============+=============================================================================+
| /auditevents | 审计事件                                                                     |  
+--------------+-----------------------------------------------------------------------------+
| /autoconfig  | 查看自动配置的使用情况提供了一份自动配置报告，记录哪些自动配置条件通过了，哪些没通过    |  
+--------------+-----------------------------------------------------------------------------+
| /configprops | 查看配置属性，包括默认配置 描述配置属性(包含默认值)如何注入Bean                     |  
+--------------+-----------------------------------------------------------------------------+
| /beans	   | 查看bean及其关系列表 描述应用程序上下文里全部的Bean，以及它们的关系                 |  
+--------------+-----------------------------------------------------------------------------+
2.设计思路
-----------------------------------------
