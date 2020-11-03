---
title: 从Eclipse迁移到Android Studio，并打包指定文件到jar
date: 2016-08-18 21:54
tags: Android
category: Android
---

近期在维护一些老代码，是使用Eclipse开发的，我想用过Android Studio的人，再回去用Eclipse是无比难受的，所以趁着周末，赶紧把代码迁移到Android Studio。

<!-- more -->

# 迁移步骤
1. 打开Android Studio选择Import Project(Eclipse ADT, Gradle,etc.)
2. 选择Eclips项目AndroidManifest.xml文件所在的目录，单击确定
3. 选择保存转换后项目的路径
4. 后面，默认即可，如果项目依赖其它项目，会一同导入。
5. 稍等一会，项目就导入完毕

# 打包指定文件到jar中
经常需要把项目打包成jar,以供其它程序使用,Eclipse导出jar非常方便，Android Studio需要在module的build.gradle中进行配置
```
android{
......
}
dependencies {
......
}
//打包之前，删除以前的文件
task clearJar(type: Delete) {
    delete 'build/libs/test.jar'
}

//打包生成test.jar
task makeLibraryJar(type:org.gradle.api.tasks.bundling.Jar) {
    //指定生成的jar名
    baseName 'test'
    //从哪里打包class文件,可以指定文件和目录
    from('build/intermediates/classes/debug/com/example/test/')
    //打包到jar后的目录结构
    into('com/example/test/')
    //去掉不需要打包的目录和文件
    exclude 'META-INF/LICENSE.txt'
    exclude 'META-INF/NOTICE.txt'
}

makeJar.dependsOn(clearJar,build)
```
在自带命令行下运行
```
gradle makejar
```
即可生成需要的jar包

# 问题和解决方法
迁移完成之后，直接编译可能会遇到一些问题，以下是我遇到的问题和解决方案
### 配置Android NDK    
解决方法：在local.properties文件中加入
```
ndk.dir = XXXX
```
### Error:(12, 0) Error: NDK integration is deprecated in the current plugin.
解决方法：需要在gradle.properties文件中加入
```
android.useDeprecatedNdk=true
```
### Error:(2, 18) string: No such file or directory
解决方案：程序使用了C++标准库，需要在模块的build.gradle加上
```
stl "gnustl_static"
```
### Error:Duplicate files copied in APK META-INF/LICENSE.txt

解决方案：在module的build.gradle中加入
```
android {
    packagingOptions {
        exclude 'META-INF/LICENSE.txt'
        exclude 'META-INF/NOTICE.txt'
    }
}
```
### Error:(17, 28) 错误: 程序包org.apache.http.util不存在
解决方案：以前的项目使用了httpclient,而在Android 6.0已经把httpclient相关代码移除，如果需要使用，需要在module的build.gradle中加入
```
android {
    useLibrary 'org.apache.http.legacy'
}
```
# 参考
迁移过程参考一下文档：

  - [官方文档：Importing Projects to Android Studio](http://developer.android.com/sdk/installing/migrate.html)
  - [使用gradle打包指定包名和类的jar](http://www.alloyteam.com/2015/03/shi-yong-gradle-da-bao-zhi-ding-bao-ming-he-lei-di-jar/)
