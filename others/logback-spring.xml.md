##### #### 1. 标签元素

##### 1.1 root

- 只有level属性，是logger的上级

- 可包含至少0个<appender-ref>元素，标识指定appender添加到root

##### 1.2 logger

- 指定受此约束的某一个包或者具体的某一个类

- 可包含至少0个元素，标识指定appender添加到root

##### 1.3 appender

- 负责写日志的组件，有两个必选属性 name 和 class

- name 指定 appender 名称，class 指定 appender 的全限定名

###### 1.3.1 ConsoleAppender

> 把日志输出到控制台

- <encoder>：对日志进行格式化

- <target>：字符串System.out(默认)或者System.err

###### 1.3.2 FileAppender

> 把日志添加到文件

- <file>：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值

- <append>：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true

- <encoder>：对记录事件进行格式化。（具体参数稍后讲解 ）

- <prudent>：如果是 true，日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是 false

###### 1.3.3 RollingFileAppender

> 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件

- <file>：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值

- <append>：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true

- <rollingPolicy>:当发生滚动时，决定RollingFileAppender的行为，涉及文件移动和重命名
  
  - TimeBasedRollingPolicy：根据时间
  
  - SizeBasedTriggeringPolicy：根据文件大小
  
  - FixedWindowRollingPolicy：根据固定窗口算法

#### 1.4 property

> 用来定义变量值，它有两个属性name和value，通过定义的值会被插入到logger上下文中，可以使“${}”来使用变量

#### 2. 其他

##### 2.1 输出源优先级

    若子节点（logger）存在输出源，则直接输出，反之，若配置addtivity属性，则采用root的输出源

##### 2.2 日志级别优先级

> 只输出比自身等级还高的日志

    OFF > FATAL > ERROR > WARN > INFO > DEBUG > ALL

##### 2.3 SpringBoot 添加 Slf4j 日志依赖

> spring-boot-starter-web依赖，用于自动导入日志框架的依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

#### 3. 配置样例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
    小技巧: 在根pom里面设置统一存放路径，统一管理方便维护
    <properties>
        <log-path>/Users/meisui</log-path>
    </properties>
    1. 其他模块加日志输出，直接copy本文件放在resources 目录即可
    2. 注意修改 <property name="${log-path}/log.path" value=""/> 的value模块
-->
<configuration debug="false" scan="false">
	<property name="log.path" value="logs/${project.artifactId}"/>
	<!-- 彩色日志格式 -->
	<property name="CONSOLE_LOG_PATTERN"
			  value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
	<!-- 彩色日志依赖的渲染类 -->
	<conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
	<conversionRule conversionWord="wex"
					converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
	<conversionRule conversionWord="wEx"
					converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>
	<!-- Console log output -->
	<appender name="console" class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
			<pattern>${CONSOLE_LOG_PATTERN}</pattern>
		</encoder>
	</appender>

	<!-- Log file debug output -->
	<appender name="debug" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${log.path}/debug.log</file>
		<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
			<fileNamePattern>${log.path}/%d{yyyy-MM, aux}/debug.%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
			<maxFileSize>50MB</maxFileSize>
			<maxHistory>30</maxHistory>
		</rollingPolicy>
		<encoder>
			<pattern>%date [%thread] %-5level [%logger{50}] %file:%line - %msg%n</pattern>
		</encoder>
	</appender>

	<!-- Log file error output -->
	<appender name="error" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<file>${log.path}/error.log</file>
		<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
			<fileNamePattern>${log.path}/%d{yyyy-MM}/error.%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
			<maxFileSize>50MB</maxFileSize>
			<maxHistory>30</maxHistory>
		</rollingPolicy>
		<encoder>
			<pattern>%date [%thread] %-5level [%logger{50}] %file:%line - %msg%n</pattern>
		</encoder>
		<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
			<level>ERROR</level>
		</filter>
	</appender>

	<logger name="org.activiti.engine.impl.db" level="DEBUG">
		<appender-ref ref="debug"/>
	</logger>

	<appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
		<!--可以访问的logstash日志收集端口-->
		<destination>meisui-elk:4587</destination>
		<encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder"/>
	</appender>

	<!--nacos 心跳 INFO 屏蔽-->
	<logger name="com.alibaba.nacos" level="OFF">
		<appender-ref ref="error"/>
	</logger>
	<!-- Level: FATAL 0  ERROR 3  WARN 4  INFO 6  DEBUG 7 -->
	<root level="INFO">
		<appender-ref ref="console"/>
		<appender-ref ref="debug"/>
		<appender-ref ref="LOGSTASH"/>
	</root>
</configuration>

```

<h6>参考资料</h6>

1. [logback配置文件---logback.xml详解 - 马非白即黑 - 博客园](https://www.cnblogs.com/gavincoder/p/10091757.html)
