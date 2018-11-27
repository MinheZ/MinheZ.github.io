---
title: >-
  Intellij IDEA Spring aop 报错 BeanCreationException: Error creating bean with
  name
date: 2018-11-05 21:08:28
tags: Intellij IDEA Spring aop ERROR！
categories:
- Intellij IDEA Spring aop ERROR！
---

笔者习惯使用InteliJ IDEA开发工具写Java代码。但是在使用Spring aop或者是aspectj框架时可能遇到如下错误，此处仅提取跟代码相关的关键地方。

<!--more-->

``` bash
java.lang.IllegalStateException: Failed to load ApplicationContext

Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'customerDao' defined in class path resource [applicationContext2.xml]: BeanPostProcessor before instantiation of bean failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'org.springframework.aop.aspectj.AspectJPointcutAdvisor#0': Bean instantiation via constructor failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.aop.aspectj.AspectJPointcutAdvisor]: Constructor threw exception; nested exception is java.lang.NoClassDefFoundError: Could not initialize class org.aspectj.util.LangUtil

```
原因是Spring编程必须手动导入spring-aop.jar, aspectjrt.jar, aopalliance_1.0.jar, aspectjweaver-1.9.1.jar这4个jar包。

[下载地址](链接：https://pan.baidu.com/s/1JnK4eG__DpN-QbbyHile2A)  提取码：dexw 


	本文作者：MinheZ
	版权声明：转载请注明出处！
