---
title: Create your blog on GitHub
date: 2018-10-16 10:09:02
tags: 博客环境搭建
categories:
- Hexo博客环境搭建

---
声明：本文将介绍如何在github上面发表自己的博客。笔者在windows10 64位系统下操作，其它系统仅供参考。

<!--more-->

## 1 搭建环境

首先需要下载2个软件

``` bash
node.Js
git
```
下载地址：[node.Js](https://nodejs.org/en/)  [git](https://git-scm.com/downloads)
More info: [Writing](https://hexo.io/docs/writing.html)

具体的下载，安装就不用多说了，基本上下载完默认安装即可，安装的路径最好先记住。Git 安装的时候会弹出下面的窗口，我们选择第二个即可。这样我们在Windows的命令窗口也可以进行Git操作了。

![](https://i.imgur.com/hN62NQV.png)

两个都安装完成之后，打开git bash分别输入node-v、npm -v和git--version，这3个命令能看到我们刚才安装软件的版本，如下图，即可进入下一个步骤。
![](https://i.imgur.com/2PsqpdE.png)


## 2 配置GitHub

### 2.1 注册GitHub
  在[GitHub](https://github.com/)	官网注册一个账号。

### 2.2 SSH授权
### 2.2.1 生成SSH Key

``` bash
打开Git Bash，输入ssh-keygen -t rsa
```
按3下回车（此处跳过密码设置），由于我已经设置过了，这里选择不覆盖。

![](https://i.imgur.com/bnSVXCu.png)
接着就会在C:\Users\Administrator.ssh目录下生成到id_rsa和id_rsa.pub两个文件，id_rsa是密钥，id_rsa.pub是公钥，接下来需要将id_rsa.pub的内容添加到GitHub上，这样本地的id_rsa密钥才能跟GitHub上的id_rsa.pub公钥进行配对，才能够授权成功。
#### 2.2.2 在GitHub上添加SSH Key
在个人主页右上角上点击倒三角，进入Setting
![](https://i.imgur.com/57zAVCm.png)
点击new SSH key，再把刚才的公钥复制粘贴进去。SSHKey添加成功之后，可以输入ssh-T git@github.com进行测试，出现以下图片样式则证明成功。
![](https://i.imgur.com/Mbj4th3.png)

## 创建GitHub仓库
这一波比较关键！！！下图是新建一个repositories（仓库），此处注意项目的名称一定是：你的名字+.github.io，例如笔者的是MinheZ.github.io。
![](https://i.imgur.com/TznCa3E.png)

## 4 设置本地博客的配置
### 4.1 安装Hexo
在一个合适的地方创建一个文件夹，然后再文件夹空白处按住Shift+鼠标右键，在此处打开Git bash命令行窗口
``` bash
在命令行输入 npm install -g hexo
```
注意：笔者在此过程中遇到过 E404报错，详细内容忘记截图=。=，原因是npm在国内被墙，使用镜像服务器就可以了，此处仅为读者提供一种思路，并不一定是有效方法。
``` bash
npm install -g hexo --registry=https://registry.npm.taobao.org
```

然后
``` bash
 输入 npm install hexo --save
```
可以输入
``` bash
hexo -v 来检查hexo是否安装成功
```
### 4.2 初始化Hexo
在命令窗口中继续输入
``` bash
hexo init，等待下载好之后输入hexo s
```
此时打开浏览器在地址栏输入 http://localhost:4000/ 就能看到自己搭建的本地博客
![](https://i.imgur.com/YRfd6CJ.png)

博客文章放在文件夹下面sorcerer/_posts文件夹，再进入看到一篇初始化hello-world.md。如果需要新建文章的话，在命令窗口输入
``` bash
hexo new 'filename'
```

### 4.3 发布博客
复制我们Github项目地址，打开新建文件夹下面生成的_config.yml文件，在最下方修改
![](https://i.imgur.com/TuGgb66.png)
注意：repo自己加上就好。冒号后面追加一个空格！接下来回到命令窗口，输入
``` bash
npm install hexo-deployer-git --save
```
安装好Git上传插件之后，输入 hexo g，然后输入 hexo d就可以将我们的博客上传到我们的GitHub了，而且以后更新文章就只需要用这两个命令就可以了。这样别人也可以通过 https://yourname.github.io 来访问我们的博客了（开头一定要用https，yourname是你的github的名字）。

## 5 主题太丑怎么办
命令行输入
``` bash
git clone https://github.com/iissnan/hexo-theme-next themes/next
```
等待下载完成之后，打开配置文件_config_yml，找到theme将next替换原来默认的landscape。然后在命令窗口下
``` bash
输入 hexo clean, hexo g, hexo s, 本地确认样子之后再hexo d部署至GitHub
```
![](https://i.imgur.com/k56iJO6.png)

至此，博客环境就搭建完成，谢谢观看！


	本文作者：MinheZ
	版权声明：转载请注明出处！
