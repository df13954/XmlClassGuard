# XmlClassGuard 简介

`XmlClassGuard`是一个可混淆任意类的Gradle插件，Android 4大组件、自定义View、任意类，只要你想，都可以将其混淆

# 有什么用？

- 增加aab、apk反编译的难度

- 极大降低aab包查重率，避免上架`Google Play`因查重率过高，导致下架或封号问题

# 原理

`XmlClassGuard`不同于`AndResGuard(apk资源混淆)、AadResGuard(aab资源混淆)`侵入打包流程的方案，`XmlClassGuard`
需要在打包前执行`xmlClassGuard`任务，该任务会检索`AndroidManifest.xml`及`navigation、layout`
文件夹下的xml，找出xml文件中引用的类，如4大组件及自定义View等，更改其`包名+类名`，并将更改后的内容同步到其他文件中，此过程暂时不可逆，所以，在执行`xmlClassGuard`
任务前，请务必做好代码备份

# 警告警告⚠️

**任务执行是不可逆的，执行前，务必做好代码备份，否则代码将很难还原**

# 上手

1、在`build.gradle(root project)`中配置

```gradle
buildscript {
    repositories {
        maven { url 'https://jitpack.io' }
    }
    dependencies {
        classpath "com.github.liujingxing:xml-class-guard-plugin:1.0.0"
    }
}
```
2、在 `build.gradle(application)` 中配置

```gradle
apply plugin: "xml-class-guard"

//以下均为非必须
xmlClassGuard {
    //用于增量混淆的 mapping 文件
    mappingFile = file("xml-class-mapping.txt")
    //更改manifest文件的package属性，即包名
    packageChange = ["com.ljx.example": "ab.cd"]
    //移动目录
    moveDir = ["com.ljx.example": "ef.gh"]
}
```

此时就可以在`Gradle`栏中，找到以下3个任务

![guard.jpg](https://github.com/liujingxing/xml-class-guard-plugin/blob/master/image/guard.jpg)

# 任务介绍

`XmlClassGuard`插件共有3个任务，分别是`moveDir`、`packageChange`及`xmlClassGuard`，下面将一一介绍

## 1、moveDir

`moveDir`是一个移动目录的任务，它支持同时移动任意个目录，它会将原目录下的所有文件(包括子目录)移动到另外一个文件夹下，并将移动的结果，同步到其他文件中，配置如下：

```gradle
xmlClassGuard {
    //移动目录
    moveDir = ["com.ljx.example": "ef.gh",
               "com.ljx.example.test": "ff.gg"]
}
```

上面代码中`moveDir`是一个Map对象，其中key代表要移动的目录，value代表目标目录； 上面任务会把`com.ljx.example`目录下的所有文件，移动到`ef.gh`
目录下，将`com.ljx.example.test`目录下的所有文件移动到`ff.gg`目录下

## 2、packageChange

`packageChange`是一个更改`manifest`文件里`package`属性的任务，也就是更改app包名的任务(不会更改applicationId)
，改完后，会将更改结果，同步到其他文件中，配置如下：

```gradle
xmlClassGuard {
    //更改manifest文件的package属性，即包名
    packageChange = ["com.ljx.example": "ab.cd"]
}
```

以上`packageChange`是一个Map对象，key为原始package属性，value为要更改的package属性，原始package属性不匹配，将更改失败

## 3、xmlClassGuard

`xmlClassGuard`是一个混淆类的任务，该任务会检索`AndroidManifest.xml`及`navigation、layout`
文件夹下的xml文件，找出xml文件中引用到的类，如4大组件及自定义View等，更改其`包名+类名`，并将更改的结果，同步到其他文件中，最后会将混淆映射写出到mapping文件中，配置如下：

```gradle
xmlClassGuard {
    //用于增量混淆的 mapping 文件
    mappingFile = file("xml-class-mapping.txt")
}
```

上面配置的`mappingFile`可以是一个不存在的文件，混淆结束后，会将混淆映射写出到该文件中，如下：

```xml
dir mapping:
	com.ljx.example -> e
	com.ljx.example.activity -> dh

class mapping:
	com.ljx.example.AppHolder -> e.B
	com.ljx.example.activity.MainActivity -> dh.C
```

`dir mapping`是混淆的目录列表，`class mapping`
是具体类的混淆列表，以上内容，也可以手动写入，下次混淆时，便会根据此配置进行增量混淆，如果你需要混淆指定的类`com.ljx.example.test.Test`，便可以在`dir mapping`
下写入
`com.ljx.example.test -> h`,此时再次执行`xmlClassGuard`任务，便会将`com.ljx.example.test`目录下的所有类(不包含子目录下的类)
移动到`h`文件夹中，并将所有类名混淆，再次混淆的后mapping文件如下：

```xml
dir mapping:
	com.ljx.example -> e
	com.ljx.example.activity -> dh
    com.ljx.example.test -> h

class mapping:
	com.ljx.example.AppHolder -> e.B
	com.ljx.example.activity.MainActivity -> dh.C
    com.ljx.example.test.Test -> h.D
```

**注：手动输入时，需要注意，混淆后的目录仅支持小写字母，类名仅支持大写字母，位数不限**




 













