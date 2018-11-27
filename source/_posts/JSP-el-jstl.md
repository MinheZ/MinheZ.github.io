---
title: JSP&el&jstl
date: 2018-11-01 08:54:21
tags:JSP&el&jstl
---

##jsp(Java Server Page)
Java服务器页面：运行在服务器端，本质上就是一个Servlet。
	
作用：将内容的生成信息和展示分离。

## 1.JSP脚本：
``` bash
	<%....%> Java代码片段
	<%=...%> Java输出表达式，相当于out.print();
	<%!...%> 声明成员变量 
```
## 2.JSP的指令

### 2.1作用
声明JSP页面的一些属性和动作
### 2.2格式
<%@指令名称 属性=“值” 属性=“值”%>
### 2.3JSP指令的分类

#### 2.3.1page指令
重要属性:3个
	contentType:设置响应流的编码,及通知浏览器用什么编码打开.设置文件的mimetype
	pageEncoding:设置页面的编码
	import:导入所需要的包
contentType和pageEncoding联系:
	若两者都出现的时候,各自使用各自的编码
	若只出现一者,两个都使用出现的这个编码
	若两者都不出现,使用服务器默认的编码 tomcat7使用的 iso-8859-1
了解属性:
	language:当前jsp页面里面可以嵌套的语言
	buffer:设置jsp页面的流的缓冲区的大小
	autoFlush:是否自动刷新
	extends:声明当前jsp的页面继承于那个类.必须继承的是httpservlet 及其子类
	session:设置jsp页面是否可以使用session内置对象
	isELIgnored:是否忽略el表达式
	errorPage:当前jsp页面出现异常的时候要跳转到的jsp页面
	isErrorPage:当前jsp页面是否是一个错误页面
	若值为true,可以使用jsp页面的一个内置对象 exception

#### 2.3.2include指令
静态包含，就是将其他页面或者Servlet的内容包含进来，一起进行编译运行，生成一个Java文件。（重复页面的抽取，提高开发效率）
	格式:
		<%@include file="相对路径或者是内部路径" %>
	例如:
		<%@include file="/jsp/include/i1.jsp" %>
路径:
	相对路径:./或者什么都不写 当前路径
				上一级路径  ../
	绝对路径:带协议和主机的绝对路径
			不带协议和主机的绝对路径
			/项目名/资源				
	内部路径:
		不带协议和主机的绝对路径去掉项目名
		请求转发 静态包含 动态包含


