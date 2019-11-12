---
title: classpath和classpath*的区别
date: 2019-11-09 14:51:09
categories:
    - Java
tags:
    - Java
---


# 背景
现在我们在开发一套OA系统，用到了springMVC，我们的spring相关配置文件统一放在`src/main/resources/spring` 文件夹下面，web.xml进行如下配置：
```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring/application-*.xml</param-value>
</context-param>
```

<!-- more -->

# 问题
我们现在需要用到一个定时任务管理系统，是公司另一个部门开发的，通过maven dependence的方式引入，该jar包中也有spring相关的配置文件，路径为：`spring/application-task`。但是项目启动的时候，这个配置文件却没有加载进来。

# 解决
后面将web.xml的配置修改了一下，如下：
```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath*:spring/application-*.xml</param-value>
</context-param>
```

# 总结
我认为，classpath就是加载当前项目下的资源文件。
而classpath*就是加载包含jar包在内的所有依赖的资源文件。
后续看了源代码，继续进行深入分析，这里只是打一个标记，算是给遇到该问题的人一个出坑的思路。
