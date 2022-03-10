##### 1. Spring 两种占位符

> On the last point: since the default config files accept Spring style placeholders (`${…​}`) the Maven filtering is changed to use `@..@` placeholders (you can override that with a Maven property `resource.delimiter`).

###### 1.1 @*@

###### 1.2 ${*}

- 使用 @Value 注解注入属性时，只能使用 ${*} 占位符解析

##### 2. Spring Boot 核心配置文件 bootstrap & application 优先级

> boostrap 由父 ApplicationContext 加载，比 applicaton 优先加载，且 boostrap 里面的属性不能被覆盖



<h6>参考资料</h6>

1. [聊聊 SpringBoot 中的两种占位符：@*@ 和 ${*} - xiaoxi666 - 博客园](https://www.cnblogs.com/xiaoxi666/p/15676529.html)


