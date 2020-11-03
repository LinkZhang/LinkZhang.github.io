---
title: Hello World
date: 2016-03-10 20:45:34
tags: hexo 
category: hexo
---
[Hexo](https://hexo.io/)是一个快速、简洁且高效的博客框架。Hexo使用Markdown解析文章，在几秒内，即可利用靓丽的主题生成静态网页。本文介绍如何在Mac上搭建Hexo，Linux用户也可以参考。网上关于Hexo的教程也比较多，但大都会让读者在一两个点上卡住。综合官网教程以及自己在安装过程中遇到的问题写下了本教程，一步一步来就可以无痛安装。

<!-- more -->

## 安装Hexo

### 安装Nodejs
Mac用户和Windows用户安装比较简单，直接[下载](https://link.jianshu.com/?t=http://nodejs.cn/download/)Node.js按照默认配置安装即可。

### 安装hexo

``` bash
$ npm install -g hexo-cli
```

## Quick Start
### 创建hexo工程

```
$ hexo init blog
```
创建一个文件夹blog（此处blog换成你自己想要的名字），使用Hexo命令初始化blog为hexo工程目录。

### 新建POST

```
$ cd blog
$ hexo new “HelloWorld”
```
进入初始化后的blog文件夹，创建名为HelloWorld的文件，此时会在/blog/sources/_post/目录下生成HelloWorld.md文件。

### 生成静态文件
```
$ hexo generate
```
使用Hexo引擎将Markdown格式的文件解析成可以使用浏览器查看的HTML文件，HTML文件存储在blog/public目录下。

### 本地预览

```
$ hexo server

```

打开命令行提示的地址，一般是[http://0.0.0.0:4000/](https://link.jianshu.com?t=http://0.0.0.0:4000/)，既可以看到我们的Hexo网站。如果提示找不到server命令则需要运行命令`$ npm install hexo-server --save`，Hexo3.0之后把server独立出来了，所以需要单独安装。

此时Hello world文章中没有任何内容。打开/blog/sources/_post/目录，使用编辑器打开其中的HelloWorld.md并在其中添加markdown格式的内容保存，然后重新运行以下命令：

```
$ hexo generate
$ hexo server

```

打开浏览器查看修改后的内容

### 部署
```
npm install hexo-deployer-git --save
```
在/blog/_config.yml中修改deploy属性(注意:之后有空格)

```
deploy:
  type: git
  repository: https://github.com/yourname/yourname.github.io.git
  branch: master
```
将Repository换成你申请的Git仓库地址

运行一下命令将Hexo上传到Github
```bash
hexo d -g
```

