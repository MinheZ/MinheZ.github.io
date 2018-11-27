---
title: 将本地项目推送到github
date: 2018-11-20 15:30:52
tags: 使用教程
categories:
- github项目推送
---

## 写在前面

本篇文章将介绍怎么用git bash将自己本地项目推送至github仓库，仅此记录自学过程，如有不正确的地方，欢迎指正，探讨！

<!--more-->

## 1 软件安装
此部分为git客户端的安装，详细教程可以参考笔者另一篇文章[Create-your-blog-on-GitHub](https://minhez.github.io/2018/10/16/Create-your-blog-on-GitHub/)


## 2 推送教程
首先在你本地要上传的项目文件夹上按住shift+鼠标右键，点击git bash here，这就就能直接进入当前目录下操作，如下图
![](https://i.imgur.com/UOblh5u.png)

然后输入git init，使当前目录成为本地git仓库，完成此步之后，在当前目录下会有一个隐藏文件夹.git
![](https://i.imgur.com/lHoK96m.png)


先输入git pull -f --all 将远程的代码仓下拉至本地，再输入git add . 把本地项目里面的所有文件添加到刚生成的本地git仓库
![](https://i.imgur.com/B1BVYXQ.png)

输入 git commit -m "这里是注释，自己随便写点什么"
![](https://i.imgur.com/7JFc1F7.png)

输入git remote add origin https://github.com/MinheZ/SSM.git     后面链接是你远程github端的仓库链接，再进行这一步的时候可能会出现下图错误
![](https://i.imgur.com/8DS7GIn.png)

可以使用git push -u origin master -f将本地仓强制推上远程仓，但是要注意提前把github仓里面的文件下拉至本地备份，否则可能出现文件丢失。

	本文作者：MinheZ
	版权声明：转载请注明出处！
