---
layout:     post
title:      "基于SpringBoot的MyBlog项目总结"
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
整体使用Springboot进行后台快速开发，通过纯注解的方式使用SpringMVC来完成URL跳转和模型视图控制器的解耦，避免了配置文件的繁琐。同时利用Spring的AOP完成log功能，DI完成Service层中DAO层的注入，也没有使用配置文件。
对于ORM，选择Mybatis，通过xml配置文件的方式完成（为了以后扩展，注解方式不太好写复杂的sql语句）
整体使用REST，通过名词的URL+http动作来给后台传输数据，后台根据controller层的requestmapping注解选择合适的方法处理，在方法中调用Service层的服务得到ViewObject，通过setattributes传给视图层，并处理一些特殊情况和错误。
#### 技术栈
1. Springboot快速开发
2. Mybatis ORM框架减少DB代码
4. 前端CSS网上找的
5. 调试、部署
6. 大头：maven bug解决（最终发现是setting.xml设置问题，当初为了用aliyun的mirrors加快速度，从网上找的配置文件，结果出了问题。）

## 思考
这个项目花的最多的时间不是写后端代码，而是调试mvn，package老是不成功，通过各种命令分析，包括但不限于tree分析依赖关系，更换mvn仓库，mirror源。在mvn打包Springboot不成功之后尝试按照之前的web项目打war包，或者用docker配置。
最终在用docker配置时的错误信息使我发现是setting.xml的本地仓库配置有问题，这也解释了为什么我的IDEA的默认maven配置文件和仓库不匹配（之前不匹配我都以为是我修改了IDEA的默认自带的maven仓库导致的）。
问题的关键在于我的IDEA的pom配置文件和终端的mvn命令默认的配置文件不一致：这是IDEA的锅。![a](img/2019-05-01-myblog1.jpg)
settings文件一开始不是这个，这就导致了所有的问题。

这段经历使我收获了很多，面对bug，最重要的是要仔细读报错信息，如本次在用docker配置之前的`mvn clean package`报错“包找不到”，我去IDE看，明明包都在那里，pox.xml也没报错，lib里边也确实有文件。

最终解决是`vim ~/.m2/setting.xml`改对应repo即可。

其实之前出过几次这样的事，印象最深的是zsh的配置、mysql配置文件以及Spring boot启动事件。得出的经验教训都一样：仔细读报错信息，**分析错误原因（画重点**）上[google](www.google.com) [stackoverflow](https://stackoverflow.com)查找错误信息。

## 总结
从这件事上看，时间大概三成写代码，七成调bug。
显然效率很低，正确的做法一是加快bug解决速度，二是少写工程性强、“八股文”、无用的代码（比如配置文件，服务器配置，静态资源管理）等，而是多学习，加快知识吸收速度，等实际用到再操作。反正以后工作几十年，工作之后的问题可以找到大牛，现在自己google万一找不出实在影响心情。