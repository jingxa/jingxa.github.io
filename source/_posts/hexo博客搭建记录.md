---
title: hexo博客搭建记录
date: 2017-05-31 18:37:27
tags: hexo
categories: 
- 杂乱笔记
---
简单记录一下利用hexo+github搭建博客的过程
#### 主要思路
　　安装git和node的过程就不多提了，本次安装hexo博客主要采用建立GitHub pages托管，建立两个分支，一个用于提交源文件，一个用于提交hexo静态文件，如果博客需要换电脑，直接git clone下来即可
>hexo分支：博客源文件内容
>master分支:静态博客内容

#### 一 建立远程库
　　注意事项：
　　　　１.建立两个分支master 和hexo
　　　　２.将hexo分支设为默认分支，这样方便提交源文件
#### 二、下载远程库
```
git clone git@github.com:jingxa/jingxa.github.io.git
```
改成你的名字
###### 【特别注意】：先不要进行下面的步骤，先将仓库中【.git,.gitignore】复制出来，不然下面安装hexo会将这个给覆盖掉，后面无法提交
#### 三、开始安装hexo
##### 1、安装hexo
```
 npm install -g hexo-cli
```
##### 2、 初始化hexo
```
hexo init
```
##### 3、安装npm
 ```
 npm install
 ```
##### 4、 安装next主题 ：
```
git clone https://github.com/iissnan/hexo-theme-next themes/next
```
【注意：在*.jithub.io和themes/next都有一个_config.yml的文件，为了区分。我们使用一下称呼】
--*github.io/_config.yml:站点配置文件
--themes/next/_config.yml:主题配置文件
##### 5.配置站点文件
###### 【注意：每一项的填写，其:后面都要保留一个空格】
(1).修改网站相关信息
```
title: 你的题目
subtitle: 小题目
description: 描述
author: 作者
language: zh-Hans
timezone: Asia/Shanghai
```
(2).配置统一资源定位符（个人域名）
```
url: https://jingxa.github.io
```
对于root（根目录）、permalink（永久链接）、permalink_defaults（默认永久链接）等其他信息保持默认。
(3). 配置部署
```
deploy:
  type: git
  repo: ssh://git@github.com/jingxa/jingxa.github.io.git
  branch: master
```
这里我们使用ssh，不使用https,可以避免提交的时候需要输入密码验证
(4).主题修改
```
themes: next
```
##### 6.配置主题文件
这些大家自己百度
[【干货】2个小时教你hexo博客添加评论、打赏、RSS等功能](http://www.jianshu.com/p/5973c05d7100)
##### 7.提交
这一步大家讲源文件提交到github,托管源码，方便以后转移、
在提交之前，大家需要将【.gitignore】中添加一些配置，免得将一些不必要的东西提交到github上，我的配置如下：
```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```
将这个不必要的忽略掉
(1).增加文件 
```
git add .
```
提交所有的文件
(2).可以使用 
```
git status
```
检查提交哪些东西
(3).提交 
```
git commit -m "自己修改commit信息"
```
(4).重要的地方，我们需要将修改提交到hexo,分支
```
git push origin hexo
```
然后完成了源文件的提交，下一步就是提交hexo生成的静态文件，再次之前，需要安装依赖包
##### 8.安装npm install hexo-deployer-git --save
```
npm install hexo-deployer-git --save
```
其实这一步可以在前面npm install 之后安装
##### 9.静态博客生成
(1).我们可以写一篇文章
```
hexo new "你的题目"
```
我们可以在本地博客文件夹source->_post文件夹下看到我们新建的markdown文件。
保存后，我们进行本地发布：
(2).我自己测试，
```
　hexo s -p 8888
```
如果使用 
```
hexo server
```
 会使用默认端口4000，这个一直不能再本地显示，所以我们可以自己更改端口，就能正常访问
(3).打开浏览器，输入：
```
http://localhost:8888/
```
我们可以在浏览器端看到我们搭建好的博客和发布的文章：
##### 10.博客提交到github上
(1).这一步主要是清除缓存数据
```
hexo clean
```
(2). 生成
```
//hexo generate可以简写成
hexo g
```
(3). 发布
```
//hexo deploy可以简写成 
hexo d
```
然后访问你的默认网站。就可以看到你的成果了！

---
>版权声明：<br>
>本文作者：{{ author }}
>本文链接：{{ permalink }}
<hr>
除非注明，本博文章均为原创，转载请以链接形式标明本文地址。<br>