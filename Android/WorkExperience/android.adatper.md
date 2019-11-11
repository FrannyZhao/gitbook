[TOC]

# 安卓适配

## 屏幕适配

#### 定义多种分辨率的dimens.xml（不推荐）

用AndroidStudio插件ScreenMatch根据屏幕尺寸生成不同的dimen

https://www.jianshu.com/p/1302ad5a4b04

https://github.com/wildma/ScreenAdaptation/blob/master/app/src/main/res/values/dimens.xml



#### 用限制性布局ConstraintLayout

> implementation 'androidx.constraintlayout:constraintlayout:1.1.3'


```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
	<Button
        android:id="@+id/btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintHorizontal_bias="0.5"
        app:layout_constraintVertical_bias="0.55"
        android:text="Button"/>
	<android.support.constraint.ConstraintLayout
		android:layout_width="0dp"
		android:layout_height="0dp"
		app:layout_constraintWidth_default="percent"
		app:layout_constraintHeight_default="percent"
		app:layout_constraintWidth_percent="0.25"
		app:layout_constraintHeight_percent="0.52"
		app:layout_constraintLeft_toLeftOf="parent"
		app:layout_constraintRight_toRightOf="parent"
		app:layout_constraintTop_toTopOf="parent"
		app:layout_constraintBottom_toBottomOf="parent"
		app:layout_constraintHorizontal_bias="0.5"
		app:layout_constraintVertical_bias="0.55" />
    <ImageView
                android:layout_width="0dp"
                android:layout_height="0dp"
                app:layout_constraintHeight_default="percent"
                app:layout_constraintHeight_percent="0.15"
                app:layout_constraintDimensionRatio="1:1"
                app:layout_constraintLeft_toLeftOf="parent"
                app:layout_constraintRight_toRightOf="parent"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintBottom_toBottomOf="parent"
                app:layout_constraintHorizontal_bias="0.5"
                app:layout_constraintVertical_bias="0.7"
                android:src="@drawable/testImg" />
</android.support.constraint.ConstraintLayout>
```



#### 用自适应TextView
https://blog.csdn.net/Virgil_K2017/article/details/88725298

> implementation 'androidx.appcompat:appcompat:1.1.0'


8.0以上：

```xml
<TextView
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        app:layout_constraintWidth_default="percent"
        app:layout_constraintWidth_percent="0.3"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintHorizontal_bias="0.1"
        app:layout_constraintVertical_bias="0.7"
        android:text="012345678901234567890"
        android:gravity="center"
        android:maxLines="1"
        android:ellipsize="end"
        android:textSize="25dp"
        android:autoSizeTextType="uniform"
        android:autoSizeMaxTextSize="25dp"
        android:autoSizeMinTextSize="20dp"
        android:autoSizeStepGranularity="1dp" />
```

8.0以下：

```
implementation 'com.android.support:appcompat-v7:28.0.0'
```

```xml
<android.support.v7.widget.AppCompatTextView
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        app:layout_constraintWidth_default="percent"
        app:layout_constraintWidth_percent="0.3"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintHorizontal_bias="0.7"
        app:layout_constraintVertical_bias="0.7"
        android:text="012345678901234567890"
        android:gravity="center"
        android:maxLines="1"
        android:ellipsize="end"
        android:textSize="25dp"
        app:autoSizeTextType="uniform"
        app:autoSizeMaxTextSize="25dp"
        app:autoSizeMinTextSize="20dp"
        app:autoSizeStepGranularity="1dp"/>
```

androidx的话用`androidx.appcompat.widget.AppCompatTextView`



#### 其他

* 图片用.9或者svg
* 多用相对布局，尽量不用绝对布局
* 少用dp值，多用百分比



## 安卓版本适配

通过gradle flavor选择不同的source set



