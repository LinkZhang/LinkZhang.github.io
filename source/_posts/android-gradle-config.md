---
title: Gradle常用配置
date: 2016-09-12 21:22
tags: Android
category: Android
---
Android Studio使用Gradle进行构建，不仅可以非常方便的管理依赖，还可以自定义一些实用的功能,例如多渠道打包，自动签名apk等。

<!-- more -->

## 多渠道打包

上线一款app后需要统计分析各个渠道的使用数据，这就需要对渠道进行标示，这里以友盟统计为例

- 在AndroidManifest中加入占位符
```xml
<meta-data
            android:name="UMENG_CHANNEL"
            android:value="${UMENG_CHANNEL_VALUE}"/>
```
- 在module的build.gradle中加入
```Java
android {

    defaultConfig {
        applicationId "com.linkzhang.gradlesample"
        minSdkVersion 15
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "example"]//默认渠道
        flavorDimensions "default"
      }

    //自动多渠道打包
    productFlavors {
       xiaomi {}
       _360 {}
       baidu {}
       wandoujia {}
       //...添加其它渠道
     }

    productFlavors.all {
        flavor -> flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]
     }
}
```

## 自动签名apk
使用命令行进行打包，需要读取签名配置并自动对apk进行签名
- 在module的根目录下新建signing.properties文件
```Java
STORE_FILE = keystore.jks
STORE_PASSWORD = 123456
KEY_ALIAS = example
KEY_PASSWORD = 123456
```
- 在module的build.gradle中创建
```Java
android {
      signingConfigs {
          debug {

          }

          release {
              storeFile
              storePassword
              keyAlias
              keyPassword
          }

      }
}
```
- 读取配置文件

```Java
android {
  signingConfigs {
        debug {

        }

        release {
            storeFile
            storePassword
            keyAlias
            keyPassword
        }

    }

    getSigningProperties()
}

//读取签名配置文件
def getSigningProperties(){

    def propFile = file('signing.properties')
    if (propFile.canRead()){
        Properties props = new Properties()
        props.load(new FileInputStream(propFile))
        if (props!=null && props.containsKey('STORE_FILE') && props.containsKey('STORE_PASSWORD') &&
                props.containsKey('KEY_ALIAS') && props.containsKey('KEY_PASSWORD')) {
            android.signingConfigs.release.storeFile = file(props['STORE_FILE'])
            android.signingConfigs.release.storePassword = props['STORE_PASSWORD']
            android.signingConfigs.release.keyAlias = props['KEY_ALIAS']
            android.signingConfigs.release.keyPassword = props['KEY_PASSWORD']
        } else {
            println 'signing.properties found but some entries are missing'
            android.buildTypes.release.signingConfig = null
        }
    }else {
        println 'signing.properties not found'
        android.buildTypes.release.signingConfig = null
    }
}
```
- 更改release设置

```Java
android {
  buildTypes {
        release {
            minifyEnabled true  //开启代码混淆
            zipAlignEnabled true
            shrinkResources true    // 移除无用的resource文件
            signingConfig signingConfigs.release
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
}

```


## 版本号自增
每次编译release版本时，版本号自动增加
- 在module的根目录下新建version.properties文件
```
VERSION_CODE=1
```

- 读取版本号

```Java
def getVersionCode() {
    def versionFile = file('version.properties')
    if (versionFile.canRead()){
        Properties versionProps = new Properties()
        versionProps.load(new FileInputStream(versionFile))
        def versionCode = versionProps['VERSION_CODE'].toInteger()
        def runTasks = gradle.startParameter.taskNames
        //仅在assembleRelease任务增加版本号
        if ('assembleRelease' in runTasks) {
            versionProps['VERSION_CODE'] = (++versionCode).toString()
            versionProps.store(versionFile.newWriter(), null)
        }
        return versionCode
    } else {
        throw new GradleException("Could not find version.properties!")
    }
}
```

- 修改defaultConfig

```Java
android {
    def currentVersionCode = getVersionCode()

    defaultConfig {
        applicationId "com.linkzhang.gradlesample"
        minSdkVersion 15
        targetSdkVersion 23
        versionCode currentVersionCode
        versionName "1.0"
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "example"]//默认渠道
    }
}

```

## 自定义apk名称
导出的apk以app名_版本号_打包时间_渠道名_release.apk格式命名

- 获取app名称和当前时间

```Java
// 获取当前系统时间
def static releaseTime() {
    return new Date().format("yyyy-MM-dd", TimeZone.getTimeZone("UTC"))
}

// 获取程序名称
def static getProductName(){
    return "gradlesample"
}
```

- 替换文件名

```Java
android {
    buildType {
        release {
            //修改生成的apk名字，格式为 app名_版本号_打包时间_渠道名_release.apk
            applicationVariants.all { variant ->
                variant.outputs.all { output ->
                     def channel = variant.getMergedFlavor().getManifestPlaceholders().get('UMENG_CHANNEL_VALUE');
                    if (variant.buildType.name == 'release')) {
                        outputFileName = getProductName() + "_v"+defaultConfig.versionName+"_"+releaseTime()+"_"+channel + "_"+variant.buildType.name+”.apk"                    
                    }
                }
            }
        }
    }
}
```

## 代码
[完整代码](https://github.com/LinkZhang/GradleSample)



## 不足
每次新建项目都要复制一份，准备写成Gradle插件发布到maven这样就能很方便的引用了



## 参考和感谢
- [Android Studio Gradle实践之多渠道自动化打包+版本号管理](http://unclechen.github.io/2015/10/22/Android%20Studio%20Gradle%E5%AE%9E%E8%B7%B5%E4%B9%8B%E5%A4%9A%E6%B8%A0%E9%81%93%E8%87%AA%E5%8A%A8%E5%8C%96%E6%89%93%E5%8C%85+%E7%89%88%E6%9C%AC%E5%8F%B7%E7%AE%A1%E7%90%86/)
- [使用 Xcode 和 Android Studio 管理 iOS 和 Android 项目版本](http://www.jianshu.com/p/e78cfc848d24)
- [Android Studio系列教程六--Gradle多渠道打包](http://stormzhang.com/devtools/2015/01/15/android-studio-tutorial6/)

