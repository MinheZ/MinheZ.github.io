---
title: Setup your Tomcat server on IDEA
date: 2018-10-26 08:54:12
tags: Tomcat Service
categories:
- Tomcat Service
---

## 如何在InteliJ上配置Tomcat进行web开发

### 1.写在前面
最近开始接触Java当中的web开发，由于习惯了InteliJ的“智能操作”，于是自己花了一整天研究如何在InteliJ上面部署Tomcat的servlet项目（忽视笔者太菜），在此记录搭建的操作流程，希望帮到跟我一样刚入门的人，少走弯路~


<!--more-->

### 2.正式开始，新建工程
首先，新建一个工程，如下图

![](https://i.imgur.com/taXNm27.png)

选择Java下面的JavaEE，勾选Web Application，如下图，下面的版本默认勾选create web.xml，接着点next

![](https://i.imgur.com/j1zryaV.png)

然后是项目名称，这里我写TomcatTest，路径可以自由选择，点击Finish

![](https://i.imgur.com/5SbRb8f.png)

接下来点击File-Project Structure进入项目结构

![](https://i.imgur.com/wOyB2cE.png)

Project栏保持默认设置就行，如图

![](https://i.imgur.com/8KerabY.png)

接下来module栏，选中Source，在WEB-INF上右键，新建文件夹，分别命名为classes和lib，分别用来存放项目编译后输出的class文件和外部jar包，如下图

![](https://i.imgur.com/nI513vK.png)

然后点击Paths，在compiler output下面按如下图勾选，选择刚才新建classes文件夹的路径存放项目编译后输出的class文件

![](https://i.imgur.com/FAS3QVh.png)

在Dependencies点击右边“+”号，选择JARs or directories,选择之前新建的lib文件夹目录

![](https://i.imgur.com/daZmLRi.png)

再点击 Jar Directories

![](https://i.imgur.com/9TA8QLu.png)

继续点OK就行，以后第三方的Jar包就存放在此（例如dbutils等工具类的开发包）。接下来在Libraries点击“+”，如下图点击，添加的目录为你的Apache-Tomcat安装文件路径下的lib

![](https://i.imgur.com/UI0gUaA.png)

![](https://i.imgur.com/BIa010j.png)

再点击Artifacts，如下图，Output Directories可以定义其它的web项目部署路径。例如我这里的F:\Server\apache-tomcat-9.0.12\webapps\TomcatTest。注意：应该新建一个文件夹，名称就按照工程名就行。如果下面出现感叹号，点击fix新建一个lib文件夹就行，这些问题都不大~点击Apply，OK就行了。

![](https://i.imgur.com/kAP4P19.png)

### 3.Tomcat Configuration设置

接下来进行[Tomcat](https://tomcat.apache.org/download-90.cgi) Configuration设置，按如下图标注的顺序依次点击，服务器点击local就行

![](https://i.imgur.com/mo293tJ.png)

接下来设置，如图，Name我直接设置的工程名字。Application server点击右边的Configture

![](https://i.imgur.com/rrw407d.png)

Tomcat Home直接打开文件夹，定位到你Tomcat在硬盘中存放的位置就行，然后点击OK~，这个你只要设置一次就行了，后面的工程都会自动生成。

![](https://i.imgur.com/upUHQB1.png)

下面的Tomcat Server Setting保持默认设置就可以了。（想修改也行，HTTP port可以设置为1024之内的80，或者65536之内的其它端口号，只要不与其它程序冲突就行，建议默认设置）。

接下来设置部署
![](https://i.imgur.com/ciSCqZj.png)

这里有一点很奇怪，Application context我设置为工程名就没问题，设置为其它的，例如/hello，在IDEA后续部署servlet就会出现404报错，不思其解~ Apply---OK就完成了。

![](https://i.imgur.com/djvFTS2.png)

打开web文件夹下的index.jsp，修改标题和内容（内容随意==），如下图，

![](https://i.imgur.com/4RUvBfa.png)

再点击右上角Run，就能部署成功。

![](https://i.imgur.com/iywlvTr.png)

### 4.部署servlet到Tomcat服务器

在src下新建如下类，名字自定义

![](https://i.imgur.com/lOruDoN.png)

这里测试的是servlet下，服务器是否能接收到网页的请求（request），以及服务器对网页请求作出的响应（response），测试的程序清单如下：

``` bash
// HelloServlet
package minhe.HelloServlet;

import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response){
        System.out.println("收到");
    }
}

```

``` bash
// RequestServlet
package minhe.RequestServlet;

import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class RequestServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("text/html;charset=utf-8");
        String value = request.getParameter("username");
        System.out.println(value);
        response.getWriter().print("数据" + value);
    }
}
```
重新配置index.jsp文件，如下

``` bash
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>MyFirstWeb</title>
  </head>
  <body>
  <a href="http://localhost:8080/TomcatTest/request?username=tom">b_请求参数</a>
  </body>
</html>
```
服务器配置文件web.xml如下：

``` bash
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>HelloServlet</servlet-name>
        <servlet-class>minhe.HelloServlet.HelloServlet</servlet-class>
    </servlet>
    <servlet>
        <servlet-name>RequestServlet</servlet-name>
        <servlet-class>minhe.RequestServlet.RequestServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>HelloServlet</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>RequestServlet</servlet-name>
        <url-pattern>/request</url-pattern>
    </servlet-mapping>
</web-app>
```
注意：上述代码不能出错。
运行结果如下图，点击网页中的超链接，将会跳转到“数据：tom”页面，同时服务器会接收到Tom字样，证明请求和应答正常。

![](https://i.imgur.com/7PY3Khw.png)

![](https://i.imgur.com/UM6LsXE.png)

至此服务器配置完成。Enjoying~



	本文作者：MinheZ
	版权声明：转载请注明出处！


