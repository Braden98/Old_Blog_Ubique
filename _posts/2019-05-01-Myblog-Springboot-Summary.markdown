---
layout:     post
title:      "基于 SpringBoot 的 MyBlog 项目总结"
subtitle:   " \"Hello World, Hello Blog\""
date:       2019-04-29 12:00:00
author:     "Ubik"
header-img: "img/post-bg-2015.jpg"
catalog: true
comments: true
tags:
    - Summary
    - Project
---
## 概述
#### 时间 
两周左右
#### 思路
整体使用 Springboot 进行后台快速开发，通过纯注解的方式使用 SpringMVC 来完成 URL 跳转和模型视图控制器的解耦，避免了配置文件的繁琐。同时利用 Spring 的 AOP 完成 log 功能，DI 完成 Service 层中 DAO 层的注入，也没有使用配置文件。
对于 ORM，选择 Mybatis，通过 xml 配置文件的方式完成（为了以后扩展，注解方式不太好写复杂的sql语句）
整体使用 REST，通过名词的 URL+http 动作来给后台传输数据，后台根据 controller 层的 requestmapping 注解选择合适的方法处理，在方法中调用 Service 层的服务得到 ViewObject，通过 setattributes 传给视图层，并处理一些特殊情况和错误。
#### 技术栈
1. Springboot 快速开发
2. Mybatis ORM 框架减少DB代码
4. 前端 CSS 网上找的
5. 调试、部署
6. 大头：maven bug解决（最终发现是 setting.xml 设置问题，当初为了用 aliyun 的 mirrors 加快速度，从网上找的配置文件，结果出了问题。）

## 思考
这个项目花的最多的时间不是写后端代码，而是调试 mvn，package 老是不成功，通过各种命令分析，包括但不限于 tree 分析依赖关系，更换 mvn 仓库，mirror 源。在 mvn 打包 Springboot 不成功之后尝试按照之前的 web 项目打 war 包，或者用 docker 配置。
最终在用 docker 配置时的错误信息使我发现是 setting.xml 的本地仓库配置有问题，这也解释了为什么我的 IDEA 的默认 maven 配置文件和仓库不匹配（之前不匹配我都以为是我修改了IDEA的默认自带的maven仓库导致的）。
问题的关键在于我的IDEA的 pom 配置文件和终端的 mvn 命令默认的配置文件不一致：这是 IDEA的锅。

![配置方式]({{site.baseurl}}/img/2019-05-01-myblog1.jpg '配置方式')
<center>配置方式</center>
settings文件一开始不是这个，这就导致了所有的问题。

这段经历使我收获了很多，面对bug，最重要的是要仔细读报错信息，如本次在用docker配置之前的`mvn clean package`报错“包找不到”，我去IDE看，明明包都在那里，pox.xml也没报错，lib里边也确实有文件。

最终解决是`vim ~/.m2/setting.xml`改对应repo即可。

其实之前出过几次这样的事，印象最深的是zsh的配置、mysql配置文件以及Spring boot启动事件。得出的经验教训都一样：仔细读报错信息，**分析错误原因（画重点**）上[google](www.google.com) [stackoverflow](https://stackoverflow.com)查找错误信息。

## 总结
从这件事上看，时间大概三成写代码，七成调 bug。
显然效率很低，正确的做法一是加快bug解决速度，二是少写工程性强、“八股文”、无用的代码（比如配置文件，服务器配置，静态资源管理）等，而是多学习，加快知识吸收速度，等实际用到再操作。反正以后工作几十年，工作之后的问题可以找到大牛，现在自己google万一找不出实在影响心情。
## 后记
从暑假在 QingCloud 的实习经历来看，大部分时间都在读代码和思考如何改，剩下的大部分时间是对写的代码进行重构和改 Bug，至于之前总结提到的问题。一方面只需要功能符合要求，另一方面只需要跑过 CI。运维的问题不需要我来考虑（但也应该看看，哈哈）