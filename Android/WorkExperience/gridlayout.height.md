# 动态适配GridLayout的高度

参考： https://www.jianshu.com/p/441d60be7d8a

GridLayout的每一项宽高只能是：

wrap_content；

一个写死的xxdp值；

根据设置的column平分；

但是项目需要适配好一点的话，最好可以动态设置宽高。

```java
setLayoutManager(new GridLayoutManager(this, columeNum));
```

这样宽可以自动均分了；

高度是测量第1行中所有子View的高，取最大值作为该行的高，如果该行没有子View，行高设为0。

> GridLayout的子View不需要设置layout_width和layout_height属性，因为GridLayout会把所有的子View的这两个属性设置为WRAP_CONTENT，所以你设置了也没有用。

1. 高度跟宽一样

在子view的onMeasure里面把宽高设成一样

```xml
<com.m.launcher.CustomConstraintlayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/transparent">
    ...
</com.m.launcher.CustomConstraintlayout>
```



```java
package com.m.launcher;

import android.content.Context;
import android.util.AttributeSet;

import androidx.constraintlayout.widget.ConstraintLayout;

public class CustomConstraintlayout extends ConstraintLayout {
    public CustomConstraintlayout(Context context) {
        super(context);
    }

    public CustomConstraintlayout(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public CustomConstraintlayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }


    /**
     * set height the same size with height;
     * gridlayout计算子view最大高度，设置为这一行的高度；
     * 但是这里希望直接设置为跟宽度一样，而不用依赖子view的高度；
     * 这样子view可以用constraintlayout，根据父view的高度来布局
     * 所有的布局都不需要写dp值，只写比例值即可
     * @param widthMeasureSpec
     * @param heightMeasureSpec
     */
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, widthMeasureSpec);
        setMeasuredDimension(getMeasuredWidth(), getMeasuredWidth());
    }
}
```



2. 高度根据父view的高度均分

Adatper中设置子view的子view的高度

>为什么不设置子view, 因为上面说了，子view不管设置成什么，结果都是wrap_content

```xml
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <androidx.constraintlayout.widget.ConstraintLayout
        android:id="@+id/max_height_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent">
        ... </androidx.constraintlayout.widget.ConstraintLayout>
</androidx.constraintlayout.widget.ConstraintLayout>
```



```java
@NonNull
    @Override
    public XXXViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.XXX_item, null);
        View maxHeightView = view.findViewById(R.id.max_height_view);
        ConstraintLayout.LayoutParams layoutParams = (ConstraintLayout.LayoutParams) maxHeightView.getLayoutParams();
        layoutParams.height = mRecyclerView.getHeight() / 3;
        // 这里设置成父view的1/3
        maxHeightView.setLayoutParams(layoutParams);
        return new XXXViewHolder(view);
    }
```

