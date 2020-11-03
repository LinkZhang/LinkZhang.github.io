---
title: Next主题配置
date: 2016-03-11 19:43:34
tags: hexo 
category: hexo
---
在[上篇文章](https://linkzhang.com/2016/03/10/hello-world/),使用hexo搭建一个个人网站，这篇文章记录一下Next主题配置
<!-- more -->

## 启用主题
在站点文件夹根目录，打开命令行
```shell
git clone https://github.com/iissnan/hexo-theme-next themes/next
```
打开站点配置文件config.yml, 找到 `theme `字段，并将其值更改为 `next`
```
theme: next
```
这样我们的网站的主题就替换成了Next

## 主题配置
进入`themes/next`文件夹,打开主题配置文件config.yml, 按照[官方文档](http://theme-next.iissnan.com/getting-started.html)进行配置


## 主题优化
### 添加RSS订阅
在站点文件夹目录，打开命令行
```shell
 npm install --save hexo-generator-feed
```
打开站点配置文件config.yml,添加
```
plugins: hexo-generate-feed
```
在主题配置文件中的rss选项中加入
```
rss: /atom.xml
```

### 修改文章底部的那个带#号的标签
修改模板/themes/next/layout/_macro/post.swig，搜索 rel="tag">#，将 # 换成<i class="fa fa-tag"></i>

### 在每篇文章末尾统一添加“本文结束”标记
在路径 \themes\next\layout\_macro 中新建 post-end-tag.swig 文件,并添加以下内容：
```html
<div>
    {% if not is_index %}
        <div style="text-align:center;color: #ccc;font-size:14px;">-------------本文结束<i class="fa fa-paw"></i>感谢您的阅读-------------</div>
    {% endif %}
</div>
```
接着打开\themes\next\layout\_macro\post.swig文件，在post-body 之后， post-footer 之前添加如下画红色部分代码（post-footer之前两个DIV）：
![post_end_tag](https://linkzhang.com/images/post_end_tag.png)
```
<div>
  {% if not is_index %}
    {% include 'post-end-tag.swig' %}
  {% endif %}
</div>
```

然后打开主题配置文件（_config.yml),在末尾添加
```
post_end_tag:
  enabled: true
```
### 网站底部字数统计
切换到根目录下，然后运行如下代码
```
npm install hexo-wordcount --save
```
在主题配置文件中配置
```
post_wordcount:
  item_text: true
  wordcount: true
  min2read: true
```

## 参考文章
- [hexo的next主题个性化教程:打造炫酷网站](https://www.jianshu.com/p/f054333ac9e6)