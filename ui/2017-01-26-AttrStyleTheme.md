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




