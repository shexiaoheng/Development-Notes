- 普通用法
tabIndicatorColor        下划线颜色
tabSelectedTextColor     tab选中时字体颜色
tabTextColor             tab 默认字体颜色
tabTextAppearance        tab的样式，可以自定义，比如定义字体尺寸  大小写等
```xml
<com.google.android.material.tabs.TabLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:tabIndicatorColor="@android:color/black"
    app:tabSelectedTextColor="@android:color/black"
    app:tabTextColor="@color/text_gray"
    app:tabTextAppearance="@android:style/TextAppearance.Widget.TabWidget" />
```

- 去除点击效果（背景波纹）：
```xml
<com.google.android.material.tabs.TabLayout
    app:tabRippleColor="@android:color/transparent"
    app:tabBackground="@android:color/transparent"/>
```

- 去除选中线（tab 下面的小横线）
```xml
<com.google.android.material.tabs.TabLayout
    app:tabIndicatorHeight="0dp" 
```
