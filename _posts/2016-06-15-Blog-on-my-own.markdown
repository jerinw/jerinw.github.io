---
layout:     post
title:      "Blog on my own"
subtitle:   "是时候自由发挥了"
date:       2016-06-15 14:00:00
author:     "Jerin"
header-img: "img/tag-bg.jpg"
catalog: true
tags:
    - 技术
---

# 前言

如今我的小屋已经砌成，在此分享一下经验，正文中部分内容参考了网上其他教程。

[跳过废话](#build) 

说起来也是今年刚开始写的博客，打算记录一下自己最近做的事，或是记录学习工作上遇到的问题，如何解决的。开始选择了新浪博客，但用着用着发现了很多不喜欢的地方，比如布局背景不能随意更换，文章字体行距不能选择，代码不能高亮......作为一个工科生怎么能被这些束缚住手脚呢，那么就自己动手搭一个自己的博客吧，想怎么改就怎么改，想着反正实验室有好多世界各地的服务器，搞个域名也不难。一开始查资料有人推荐Wordpress，后来又了解了Jekyll,Hexo,最后确定用Jeckyll。



为什么选择用Jekyll，我是这么觉得的：


首先不是每个人都想去花钱买空间买域名的，Jekyll生成的网页可以托管在Github上。其次Jekyll简洁明了，Wordpress功能如此强大，以至于大家都是在不断折腾Wordpress本身，而不是专注于写博客。然后Jekyll技术门槛相对较低，Markdown写起来也比较顺手。其实它也有一定缺点，比如Jekyll生成的都是静态网页，如果要加评论的模块比较麻烦，其次它是不依托数据库的，也就是说每次运行都要遍历所有文本文件，要是网站做得越大，生成时间也就越长。但是瑕不掩瑜，而且Jekyll是Proudly hosted by Github的（微笑）总结来说，用Jekyll就是：免费，简洁，够用。


<p id = "build"></p>
---

## 正文

本博客使用[Markdown](http://daringfireball.net/projects/markdown/) + [Jekyll](http://jekyll.bootcss.com/) + [GitHub Pages](https://pages.GitHub.com/) 的技术方案搭建。

其中 Jekyll 是一个博客形态的静态站点产生器，能够将.Markdown转换成可以发布的完整网站。使用 Jekyll 来进行本地网站的调试然后用Git进行 Blog 的发布。

[GitHub Pages](https://pages.github.com/) 是Github推出的展示自我和项目的网站。你的网站直接托管在自己的 GitHub 仓库上，默认使用GitHub的子域名，通过 Git 命令管理，不用自己捣腾主机，也没有流量和空间限制.是一个非常理想的展示的空间。

### 本地准备

接下来要介绍本地的准备，系统为Win7，文本编辑器用的Sublime text。

**1.安装 [Ruby](http://rubyinstaller.org/downloads/)**

注意64位和32位的区别，我装的是 Ruby 2.2.4(x64)

**注意安装时一定要勾选添加到环境变量！**

![](/img/2016-06-15/1.jpg)

**2.安装 [RubyGems](https://rubygems.org/pages/download)**

Windows下使用zip格式较为方便，将下载的文件解压到任意路径下。打开Windows的命令行窗口
(Win8以后可以按win+x+a)，输入命令：

```
$ cd {你的解压地址}
$ ruby setpu.rb
```

**3.安装 Jekyll**

在命令行输入：

```
$ gem install jekyll
$ gem install jekyll-paginate
```

**4.配置 Git 环境**

首先安装 [git for windows](https://git-for-windows.GitHub.io/)
我装的是Git-2.9.0-64-bit。全部默认即可,完成以后，后续配置需要用到GitHub账号，在后文中介绍。

### GitHub准备

**1.创建自己的 [GitHub](https://GitHub.com/) 账号**

**2.设置 git 账号**

在本地打开安装好的 Git Bash。

![](http://img.blog.csdn.net/20160326173741565)

在```shell```中执行以下命令，设置你的 git 用户名和邮箱：

```
$ git config --global user.name "{username}"          // 用你的用户名替换{username}
$ git config --global user.email "{name@site.com}"    // 用你的邮箱替换{name@site.com}
```

**3.配置SSH**

为了和GitHub的远程仓库进行传输，需要配置SSH。在```shell```中执行：

```
$ ssh-keygen -t rsa -C"{name@site.com}"    // 用你的邮箱替换{name@site.com}
```

这样在```C:\Users\username```下会生成一个```.ssh```文件夹。我的路径是在C:\Program Files\Java\jdk1.8.0_51\.ssh，反正在命令行下有提示，如下图。
![](/img/2016-06-15/2.jpg)

接下来使用浏览器登录你的GitHub账户，点击右上角的"Settings"

![](/img/2016-06-15/3.jpg)

点击“SSH and GPG Keys”，

![](/img/2016-06-15/4.jpg)

使用文本编辑器打开```.ssh```中的```id_rsa.pub```文件，将内容复制粘贴到Key中，
点“Add SSH Key”确定。

![](http://img.blog.csdn.net/20160326173829850)

配置好SSH之后，便可以在本地使用 git 访问自己的 GitHub 远程仓库了。

**4.创建自己的 GitHub Pages**

GitHub 自动将命名规则为```{yourusername}.github.io```的仓库识别为 GitHub Pages 项目。简单的
建站方法是挑选一个自己喜欢的[模板](https://github.com/jekyll/jekyll/wiki/sites), 将其Fork到
自己的空间。例如：

![](http://img.blog.csdn.net/20160326173933085)

然后在你的主页点开之前Fork的仓库，点击"Settings",将“Repository name”改为
 ```{yourusername}.github.io```，点击“Rename”。

![](http://img.blog.csdn.net/20160326173955662)

完成之后便可以通过```http://{yourusername}.github.io```来访问你Fork的网站啦。

**5.同步仓库**

为了方便网站的调试和 Blog 的编辑，我们需要将托管在 GitHub 上的仓库同步到本地计算机上。
再次打开Git Bash，输入以下命令切换到你想放置本地代码仓库的位置：

```
$ cd {本地路径}     // 比如：cd e:/workspace
```

clone（克隆）你自己的 GitHub Pages 远程仓库：

```
$ git clone https://github.com/{username}/{username}.github.io.git     
```

结果如下：

![](http://img.blog.csdn.net/20160326174041711)

这时所有远程仓库里的源码都拷贝到 ```e:/workspace/{username}.github.io``` 这个文件夹里来了。

### 正式使用

有了前面的准备，现在就可以正式的编辑网站和写 Blog 啦。

**1.Jekyll简单使用**

在终端中输入命令:

```
$ cd {local repository} // {local repository}替换成你的本地仓库的目录
$ jekyll serve
```

如果一切顺利，通过浏览器访问 http://localhost:4000/ 就已经可以看到自己的网站啦。

注意

```
$ jekyll serve
```

与

```
$ jekyll serve -watch
```

功能相似，开启服务后都可以查看网站效果，也能检测到修改。个人感觉差别在于被动刷新和主动
检测上。而Jekyll配置文件_config.yml的改动则需要重启服务才能生效。

所有的 Blog 放在仓库的 ```_POST``` 文件夹下，命名遵循 ```y-m-d-title.format``` 的格式。

对 Jekyll [基本用法](http://jekyll.bootcss.com/docs/usage/)和
[目录结构](http://jekyll.bootcss.com/docs/usage/)的更多介绍，需要的时候可以自己查看。


**2.使用 Git 更新 Blog**

无论是修改网站还是更新 Blog ，都可以通过 Git 命令来完成。打开 Git Bash 切换地址到本地仓库：

```
$ cd {your repository}
```

如果之前使用 Git 与其他远程仓库建立过连接，则需要断开旧的连接连接到我们的 GitHub Pages 仓库：

```
$ git remote rm origin
$ git remote add origin https://github.com/{yourusername}/{yourusername}.github.io.git
```

然后在 Bash 中输入一下命令将本地修改同步到GitHub上:

```
$ git add .
$ git commit -m "statement"   //此处statement填写此次提交修改的内容，作为日后查阅
$ git push origin master
```
完成后就能在你的主页上看到更新了。

## 后记


接下来打算要做的有：

1. 多写一些博客，不只是技术的，还有日常；

2. 添加一个评论系统；

3. 完善 Blog 的翻页支持；

4. 添加搜索功能；

5. 添加转发功能。