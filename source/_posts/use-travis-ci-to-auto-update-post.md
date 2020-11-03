---
title: 使用Travis CI来自动化更新网站
date: 2018-06-30 11:03:34
tags: hexo 
category: hexo
---
换了电脑以后，由于没有备份，需要把网站重新搭建一遍，为了吸取这个教训，研究一下如何做到多设备同时修改网站并且利用Travis CI工具自动化部署网站。
<!-- more -->

## 利用Git分支来保存源码
平时写了一篇文章，只要用以下命令就更新好了
```shell
hexo d -g
```
实际上是hexo工具将你写好的Markdown文章按照模板自动生成html，再推送到github上，github上只保存了生成以后的html文件，并没有文章和网站配置源码。
所以，可以新建一个分支来保存源码，操作如下
在网站根目录，打开命令行，输入以下命令：
```
git checkout -b hexo
git add .
git commit -m "init website"
git push -u origin hexo
```
像node_moules/、public/文件夹是没必要提交上去的，可以在add之前，创建一个.gitignore文件进行忽略
```
package-lock.json
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```
好了，这样在其他设备，只需要把项目clone下来，切换到hexo分支，就可以重新部署网站了

## 利用Travis CI自动部署网站
完成上述步骤以后，日常更新文章的过程就是：
```
git add sources/_posts/new_post.md
git commit -m "update post"
git push -u origin hexo
hexo d -g
```
有没有更简洁的方法呢？可以通过提供持续集成（CI, Continuous Integration）服务的Travis CI，监听hexo分支提交的动态，为我们自动化部署网站。
1. 先在hexo分支里，创建.travis.yml，我的如下,需要填入email和用户名，token

    ```language: node_js
    node_js: stable
    # Travis-CI Caching
    cache:
      directories:
        - node_modules
    # S: Build Lifecycle
    install:
      - npm install
    
    before_script:
     # - npm install -g gulp
    
    
    script:
      - hexo g
    
    
    after_script:
      - cd ./public
      - git init
      - git config user.name "yourname"
      - git config user.email "youremail"
      - git add .
      - git commit -m "Update site"
      - git push --force --quiet "https://LinkZhang:${blog_coding_token}@${CODING_REF}" master:master
      - git push --force --quiet "https://${blog_token}@${GH_REF}" master:master
      
    # E: Build LifeCycle
    
    branches:
      only:
        - hexo
    env:
     global:
       - GH_REF: github.com/LinkZhang/LinkZhang.github.io.git
       - CODING_REF: git.coding.net/LinkZhang/LinkZhang.coding.me.git
    
    ```

2. 在github上个人设置页面生成[Personal access tokens](https://github.com/settings/tokens)
   如果需要同时部署到Coding,可以在Coding的账户设置里生成访问令牌
   ![Coding](http://linkzhang.com/images/coding_token.png)
3. 使用Github登录[Travis CI](https://www.travis-ci.org/)，在Travis CI平台上打开源码仓库同步开关,创建环境变量，将上述生成的token填入进去
    ![Travis](https://linkzhang.com/images/travis_settings.png) 
4. 以后，更新网站只需要把代码push到仓库，Travis CI监听到hexo有代码更新，就会在远程服务器自动初始化hexo环境，自动执行hexo命令，将生产的网页部署到github上。
5. 如果不需要在本地进行hexo操作，就可以不用安装hexo环境了 
    


