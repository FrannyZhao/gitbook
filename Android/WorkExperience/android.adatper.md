[TOC]

# 安卓适配

## 屏幕适配

1. 用AndroidStudio插件ScreenMatch根据屏幕尺寸生成不同的dimen

https://www.jianshu.com/p/1302ad5a4b04

https://github.com/wildma/ScreenAdaptation/blob/master/app/src/main/res/values/dimens.xml

2. 用限制性布局ConstraintLayout

> implementation 'com.android.support.constraint:constraint-layout:1.1.3'

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
</android.support.constraint.ConstraintLayout>
```

* 图片用.9; 多用相对布局，尽量不用绝对布局；