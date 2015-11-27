---
layout: post
title: 前端开发工具和环境搭建
category: web前端
tags:
keywords:
description:
---

> 本文提及的所有工具，就算不适用也能正常的完成前端开发工作，不过这些工具的使用可以让你做的更好、更有效、更专业

本文读者可以安装顺序来，先安装nodejs和ruby的运行环境，接着把常用的前端开发工具安装起来。最后我们安装和配置一下sublime常用的插件。如果读者发现一些更好的工具可以给我留言，我这里只是抛砖引玉。

本文重点在于环境安装，具体插件和包的使用可以请教度娘。

## node.js环境安装
---

node.js这个名字看起来像是一个js库的名字，实际上差的很远。简单的说，nodejs是一个c++写的一个解释器，用于把javascript转换成电脑或服务器上可执行的代码。一般用来开发服务端代码或者是作为开发计算机脚本。前端工程师用nodejs主要是因为nodejs环境可以给前端开发带来方便

安装

````
//mac 、linux
=
1：先安装一个 nvm（ https://github.com/creationix/nvm ） 当然也可以不装，不过装了的好处是便于nodejs版本切换
$ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.25.2/install.sh | bash

2：安装linux
$ nvm install 0.12

//查看nvm里面nodejs版本
$ nvm ls

//切换nodejs版本
$ nvm use 0.12


//windows请访问主页下载安装
http://nodejs.cn/

````

安装好了nodejs之后，就会自带一个npm的包管理工具，可以用它去安装一些其他的工具

npm常用命令：

````
//查看版本号
npm -v

//引导你创建一个package.json文件，包括名称、版本、作者这些信息等
npm init

//安装包 -g标示安装在全局 没有-g标示安装再本路径文件夹中
//安装bower示例
npm install -g bower

````

## ruby环境安装
---

[window用户可以参考这篇文章](http://jingyan.baidu.com/article/6c67b1d6e421e62787bb1ef8.html?pu=sz&st=2&bd_page_type=0&os=&rst=4)

下面说的是在mac和linux上搭建ruby环境的推荐方式 mac安装前要先装xcode

安装rvm

````
 $ curl -L https://get.rvm.io | bash -s stable
 $ source ~/.rvm/scripts/rvm
 $ rvm -v //打印版本号，检查安装是否正确

````

rvm操作

````
$ rvm list known //查找所有已知版本
$ rvm install xxx //安装xxx版本
$ rvm list //查看已安装版本
$ rvm remove xxx //移除xxx版本
$ rvm 2.0.0 --default //设置默认版本

//验证安装，打印版本号
ruby -v 

````

gem使用

gem是ruby环境的安装工具，类似npm，后文中用到npm都是用nodejs安装的，gem都是ruby安装的

````

//查看gem版本
gem -v

//修改下载source地址
$ gem source -r https://rubygems.org/
$ gem source -a https://ruby.taobao.org
$ gem sources -l  //验证成功

````

## 推荐前端工程师使用的工具
---

-	bower
-	gruntjs
-	gulp.js
-	handlebarsjs
-	compass
-	http-server 



###  [bower](http://bower.io/)
>	前端包依赖和管理工具

安装： npm install -g bower

用法：待补充，先把环境搭起来


###  [gruntjs](http://www.gruntjs.net/)
>	JavaScript 世界的构建工具

安装： npm install -g grunt-cli

用法：待补充，先把环境搭起来


###  [gulp.js](http://www.gulpjs.com.cn/)
>	用自动化构建工具增强你的工作流程!

安装： npm install --global gulp

用法：待补充，先把环境搭起来


###  [compass](http://compass-style.org/)
>	Compass是Sass的工具库（toolkit)

安装: gem install compass

用法：待补充，先把环境搭起来

推荐文章：http://www.ruanyifeng.com/blog/2012/11/compass.html


###  [http-server](https://github.com/indexzero/http-server)
>	一个轻量级的小型http服务

安装： npm install http-server 


## 前端工程师的IDE
---

- sublime text 3
- webStorm

先把这两个软件下载后安装起来.webStorm没什么好配的，功能和插件基本都很全了。sublime text推荐下载 版本3吧

webStorm是一个很好用的前端开发IDE，功能很全，而Sublime只是一个文本编辑器，他很轻，他可以安装许多扩展插件，就会有许多好用的功能，后面会说一说sublime推荐安装的插件。别和我说dreamweaver了，好像这东西只有在校大学生还在用。


##  sublime推荐插件
---

推荐插件：

-	package control 
-	Emmet 
-	Emmet liveStyle 
-	Pretty JSON 
-	Angluarjs



### package control 
>	sublime包安装功能扩展，有了他，再去用这个工具去安装其他的包就很方便了，所以必须先装

sublime 3安装方式：

ctrl + ~:打开命令管理器，输入命令后执行

````
import urllib.request,os,hashlib; h = '7183a2d3e96f11eeadd761d777e62404' + 'e330c659d4bb41d3bdf022e94cab3cd0'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
````


使用:

mac:command + shift  + p 输入 install ,选择install package, 然后查找自己想要安装的包

window:ctrl + alt + p 输入 install ,选择install package, 然后查找自己想要安装的包


安装好package就用它把之前推荐的插件都安装一下，具体插件的使用可以请教一下百度君


<div style="display:none">
	karma
 	Yeoman
 	###  [handlebarsjs](http://handlebarsjs.com/)

	### sea.js
</div>
 