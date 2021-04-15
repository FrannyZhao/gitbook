[toc]

# Android 11 (R) Client bindService 失败

## 问题

| targetSDKVersion in build.gradle | device android version | client bindService result |
| -------------------------------- | ---------------------- | ------------------------- |
| 30                               | 11(API level 30)       | **Fail**                  |
| 30                               | < 11(API level 30)     | Success                   |
| < 30                             | 11(API level 30)       | Success                   |

## 原因

从 Android 11 (Api level 30) 开始, 引入了 Package visibility 的行为变更. 简单来说就是查阅其他 App 的信息的行为开始受到约束, 具体可以查看官方文档：[Android 11 中的软件包可见性](https://developer.android.com/training/basics/intents/package-visibility).

## 解决

在 targetSDKVersion 30 的配置下, 需要在 Client的AndroidMainfest.xml 中增加 <queries> 标签, 描述想要查询的远端 App 信息 (例如包名等):

1. 包名：

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.franny.test.aidlclientone">
    <queries>
        <package android:name="com.franny.test.aidlserver" />
    </queries>
    <application
        ...>
            ...
    </application>
</manifest>
```

2. intent filter, 如果不知道具体包名，可以用intent-filter

```xml
<manifest package="com.example.game">
    <queries>
        <intent>
            <action android:name="android.intent.action.SEND" />
            <data android:mimeType="image/jpeg" />
        </intent>
    </queries>
    ...
</manifest>
```

3. provider, 如果不知道包名，但是知道需要一个content provider

```xml
<manifest package="com.example.suite.enterprise">
    <queries>
        <provider android:authorities="com.example.settings.files" />
    </queries>
    ...
</manifest>
```

## 扩展

如果你的应用确实需要访问所有的应用，那么可以申请QUERY_ALL_PACKAGES权限





