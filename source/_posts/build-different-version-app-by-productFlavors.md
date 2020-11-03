---
title: 利用productFlavors创建不同版本的App
date: 2016-10-22 12:06
tags: Android
category: Android
---
最近运营人员需要创建一个"壳版"应用进行渠道推广，即将当前的App更换名称、包名、图标、第三方服务等，成为一个新的应用。

<!-- more -->

## 方案
1. 比较简单的方案就是将代码copy一份，缺点也比较明显，以后维护起来比较麻烦，每次更新代码， 都要把代码复制一次
2. 通过gradle的productFlavors可以创建多个不同版本的App，维护起来也比较方便

考虑到实际情况选择方案二，主要涉及包名的更换，资源的更换，AndroidManifest.xml的合并，第三方服务的配置

## 具体步骤
### 更换包名
   更换包名之前需要搞清楚packageName和applicationId的区别.
   简而言之：packageName作为R资源，四大组件的路径； applicationId作为应用唯一标识， 具体可以参考官方文档[ApplicationId versus PackageName](http://tools.android.com/tech-docs/new-build-system/applicationid-vs-packagename)(需要翻墙)

   在app的build.gradle文件中加入
```python
  android {
     productFlavors {   
           demo1 {       
                 applicationId "com.demo1.android"    
           }    
           demo2 {        
                  applicationId "com.demo2.android"    
           }
     }
 }
 ```

### 资源更换
需要更换的资源有应用名称和启动图标
在src目录下建立demo1和demo2两个文件夹，如图所示：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/690205-dbe088c87fa81540.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其src/main存放的是公共文件，src/demo1、src/demo2中分别存放各自独有的文件
gradle在打包apk的时候回将相应的文件、资源进行合并，例如打包demo1时将src/main和src/demo1进行合并打包。
合并规则如下
- java文件直接合并，存在相同路径的同名文件会造成冲突，例如src/main/java/com/demo1/android/MainActivity.java 就会和src/demo1/java/com/demo1/android/MainActivity.java 冲突

- 资源的内容进行合并，同名文件的资源内容进行合并，同属性名会造成冲突
例如scr/main/res/values/strings.xml 和src/demo/res/valuse/strings.xml的内容进行合并成一个strings.xml文件，如果2个文件中都包含同一属性，例如

```xml
<string name="app_name">android</string>
```
就会造成冲突 

1. 更换应用名称
  在src/demo1/res/values/strings.xml中添加
  ```xml
  <string name="app_name">demo1</string>
  ```
  
  在src/demo2/res/values/strings.xml中添加
  ```xml
  <string name="app_name">demo2</string>
  ```
  
  删除src/main/res/values/strings.xml中的app_name属性
2. 更换启动图标
  和上面的操作类似，在相应的文件夹中放入启动图标，删除main中的启动图标。不过似乎启动图标只有放在drawable文件夹才能生效，在mipmap文件夹中不生效
  
### 集成第三方服务
第三方服务例如友盟统计，推送，分享，支付都需要申请Appkey，而Appkey是与ApplicationId绑定的，所以需要重新申请一份。
Appkey是直接写在AndroidManifest.xml文件中，所以需要创建相应的AndroidManifest.xml文件
AndroidManifest.xml合并规则是：将每个元素及其子元素的节点和属性进行合并，如果遇到相同属性会造成冲突，例如
```xml
<activity
    android:name=”.MainActivity”
    android:theme=”@theme1”/>
```
和
```xml
<activity
    android:name=”.MainActivity”
    android:screenOrientation="portrait"/>
```
合并成
```xml
<activity
    android:name=”.MainActivity”
    android:theme=”@theme1”
    android:screenOrientation="portrait"/>
```
如果2个文件同时存在`android:theme`就会造成冲突

所以第三方服务的Appkey在相应的AndroidManifest.xml配置就行，不要在main中的AndroidManifest.xml中进行配置

### 错误
按照以上步骤将项目改造之后，本以为大功告成，没想到仅仅是开始
- 错误一：微信分享无法回调
 使用sharesdk进行分享功能，根据文档微信分享必须在包名下创建
 wxapi/WXEntryActivity才能回调.
 原因:猜测分享代码中是根据
 ```
 getPackageName()+"wxapi/WXEntryActivity"
 ```
 进行回调的，由于更改的ApplicationId，得到的结果为com.demo2.android与实际的路径是不一致的，所以无法进行回调
 解决方案：创建src/main/java/com/demo2/android/wxapi/WXEntryActivity.java才能进行回调，AndroidMenifest.xml配置如下
 demo1
```xml
 <activity
            android:name="com.demo1.android.wxapi.WXEntryActivity"
            android:configChanges="keyboardHidden|orientation|screenSize"
            android:exported="true"
            android:screenOrientation="portrait"
            android:theme="@android:style/Theme.Translucent.NoTitleBar"/>
 ```
 
 demo2
```xml
<activity
            android:name="com.demo2.android.wxapi.WXEntryActivity"
            android:configChanges="keyboardHidden|orientation|screenSize"
            android:exported="true"
            android:screenOrientation="portrait"
            android:theme="@android:style/Theme.Translucent.NoTitleBar"/>
```

- 错误二：友盟反馈点击崩溃
原因：启动反馈的Activity时需要查找资源，反馈SDK使用`Class.forName(getPackageName()+”.R”)`来获取R类的，由于ApplicationId和实际的包名不一致,所以无法获取到R类
解决方案：
```java
com.umeng.fb.util.Res.setPackageName(R.class.getPackage().getName());//增加这行
FeedbackAgent agent = new FeedbackAgent(getActivity());
agent.startFeedbackActivity2(); 
```

- 错误三：极光推送能收到推送但点击无法唤起app
原因：极光sdk无法找到自定义的receiver
解决方案：在AndroidManifest.xml中注册四大组件时，包名全部使用applicationId
```xml
<receiver    
    android:name="com.demo1.android.receiver.JpushReceiver"    //实际路径
    android:enabled="true">    
    <intent-filter>        
        <action android:name="cn.jpush.android.intent.REGISTRATION"/>        
        <action android:name="cn.jpush.android.intent.MESSAGE_RECEIVED"/>        
        <action android:name="cn.jpush.android.intent.NOTIFICATION_RECEIVED"/>        
        <action android:name="cn.jpush.android.intent.NOTIFICATION_OPENED"/> 
         <category android:name="${applicationId}"/>    //这里要使用ApplicationId
    </intent-filter>
</receiver>
```

## 参考
[如何使用Gradle构建不同版本的app?](https://www.zhihu.com/question/22842123?sort=created)
