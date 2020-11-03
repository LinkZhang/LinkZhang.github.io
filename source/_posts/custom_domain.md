---
title: Github Pages自定义域名
date: 2016-6-8 21:45:34
tags: hexo 
category: hexo
---

作为一个程序员，有个属于自己的个人网站，瞬间就高大上了许多，今天把自己英文名的域名`linkzhang.com`买了下来，一次性买了2年，赶紧学习一下如何为github pages绑定自定义域名吧！

<!-- more -->

##  域名购买
我是直接在[godaddy](https://sg.godaddy.com/zh/)上买的，因为可以直接用支付宝支付，还可以搜godaddy的优惠码，一大堆，找一个填进这里，填完之后，费用会少上不少。

## 将独立域名与GitHub Pages的空间绑定
在hexo/source目录下面，新建一个名为CNAME的文本文件，里面写入你要绑定的域名，比如`linkzhang.com`
再通过
```
hexo d -g
```
部署到网站上

## DNS设置
用DNSpod，快，免费，稳定。

注册[DNSpod](http://cnfeat.com/2014/05/10/2014-05-11-how-to-build-a-blog/www.dnspod.cn)，添加域名，并添加2条A记录。
其中A的两条记录指向的ip地址是github Pages的提供的ip
可以通过ping命令获取

![dnspod](http://linkzhang.com/images/dnspod.png)


## 修改NameServer

更改godaddy的Nameservers为DNSpod的NameServers。

![dns](http://linkzhang.com/images/dns.png)

如有不详看可以看[DNSpod提供的官方帮助](https://support.dnspod.cn/Kb/showarticle/tsid/42/)

## 大功告成
大概等10分钟左右，就可以直接通过个人域名访问了！