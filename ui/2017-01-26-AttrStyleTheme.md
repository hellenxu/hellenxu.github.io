Attr vs. Style vs. Theme
======================
### Basics about Attr, Style and Theme
Attr: Attribute, is the smallest unit of style.

Style: is a collection of attributes; usually in views.

Theme: also a collection of attributes, but applied to Activity and Application.

### Attr
There are several formats that attr can be used.

| format  | description |
| ------------- | ------------- |
| color  | e.g. #fff  |
| reference  | refer to other resource id, which means this attribute can use a reference as its value  |
| boolean  | boolean value  |
| dimension  | dimension value, could be match_parent or specific value, e.g. 30dip  |
| float  | float value  |
| fraction  | fraction value  |
| integer  | integer value  |
| string  | string value  |
| enum  | enum value  |
| flag  | accept more than one value, and each value is separated by ‘|’  |

How to customize attribute?

```xml
<declare-styleable name="ColorLoadingBar">
  <attr name="textSize" format="dimension" />
  <attr name="loadingColor" format="color" />
  <attr name="pauseColor" format="color" />
</declare-styleable>
```

How to use custom attribute?
1. declare name space of res-auto
2. set custom attribute for one particular view

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:six="http://schemas.android.com/apk/res-auto"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:orientation="vertical">
  <com.six.tipsproject.view.ColorfulLoadingView
      android:id="@+id/loading"
      six:loadingColor="#ff0999"
      android:layout_margin="20dp"
      android:layout_width="match_parent"
      android:layout_height="wrap_content" />
</LinearLayout>
```

How to get value of custom attribute?

```java
private void initValue(Context context, AttributeSet attrs) {
  final TypedArray ta = context.obtainStyledAttributes(attrs, R.styleable.ColorLoadingBar);
  loadingColor = ta.getColor(R.styleable.ColorLoadingBar_loadingColor, DEFAULT_LOADING_COLOR);
  pauseColor = ta.getColor(R.styleable.ColorLoadingBar_pauseColor, DEFAULT_PAUSE_COLOR);
  textSize = ta.getDimensionPixelSize(R.styleable.ColorLoadingBar_textSize, DEFAULT_TEXT_SIZE);
  ta.recycle();
}
```

### Style

Custom style could be inherited from one parent or no parent, just a simple collection of attributes.

How to customize style?

```xml
<style name="LoadingDialog" parent="@android:style/Theme.Dialog">
  <item name="android:windowBackground">@android:color/transparent</item>
  <item name="android:windowIsFloating">true</item>
  <item name="android:windowNoTitle">true</item>
  <item name="android:windowContentOverlay">@null</item>
</style>
```

How to use custom style?

@style/name_of_custom_style

How to get values of attributes of one specific style?

Theme.obtainStyledAttributes()


### Theme

When using a custom theme, we need to pay attention to declare name of its parent. Otherwise, systems will throw an IllegalStateException.

```java
1-30 22:09:54.706 2488-2488/com.six.tipsproject E/AndroidRuntime: FATAL EXCEPTION: main
                                                                   Process: com.six.tipsproject, PID: 2488
                                                                   java.lang.RuntimeException: Unable to start activity ComponentInfo{com.six.tipsproject/com.six.tipsproject.drawer.DrawerActivity}: java.lang.IllegalStateException: You need to use a Theme.AppCompat theme (or descendant) with this activity.
```

### Inheritance of Style and Theme

Custom style and theme can be defined by inheriting from one particular parent. There are two ways of inheritance:

* declare parent by using parent attribute

We can inherit styles or themes that are built into the platform through using parent attribute.

```xml
<style name="LoadingDialog" parent="@android:style/Theme.Dialog">
  <item name="android:windowBackground">@android:color/transparent</item>
  <item name="android:windowIsFloating">true</item>
  <item name="android:windowNoTitle">true</item>
  <item name="android:windowContentOverlay">@null</item>
</style>
```

* prefix the name of parent

If we want to inherit from styles or themes that defined by ourselves, we can just prefix the name of the style or theme that we want to inherit to, separated by a period.

```xml
<style name="LoadingDialog.Red">
  <item name="android:windowBackground">@android:color/holo_red_light</item>
</style>
```
