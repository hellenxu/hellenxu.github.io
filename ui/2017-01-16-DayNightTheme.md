How to use DayNight theme
======================
####1, Apply statically
Usually we can enable night mode within onCreate method of Application. For example:
```java
public class SixApplication extends Application {
  @Override
  public void onCreate() {
      super.onCreate();
      LeakCanary.install(this);
      AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_YES);
  }
}
```

Except from MODE_NIGHT_YES , there are other three options can be used.

| Type  | Constants |
| ------------- | ------------- |
| int  | MODE_NIGHT_AUTO: Mode which means to use night mode when it is determined that it is night or not.  |
| int  | MODE_NIGHT_FOLLOW_SYSTEM: Mode which uses the system's night mode setting to determine if it is night or not.  |
| int  | MODE_NIGHT_NO: Mode which means to not use night mode, and therefore prefer notnight qualified resources where available, regardless of the time.  |
| int  | MODE_NIGHT_YES: Mode which means to always use night mode, and therefore prefer night qualified resources where available, regardless of the time.  |

####2, Apply dynamically

* Add a theme which inherits from DayNight ;

```xml
<style name="AppTheme" parent="Theme.AppCompat.DayNight.NoActionBar">
	  <!-- Customize your theme here. -->
	<item name="colorPrimary">@color/colorPrimary</item>
	<item name="colorPrimaryDark">@color/colorPrimaryDark</item>
	<item name="colorAccent">@color/colorAccent</item>
	<item name="android:textColorPrimary">@color/textColorPrimary</item>
</style>
```

* Set different colors for day and night;
values-night/colors.xml -- for night theme

```xml
<resources>
	<colorname="colorPrimary">#c9fffc</color>
	<colorname="colorPrimaryDark">#b8ffea</color>
	<colorname="colorAccent">#f6ebfc</color>
	<colorname="textColorPrimary">#fff</color>
</resources>
```
values/colors.xml -- for day theme

```xml
<resources>
	<colorname="colorPrimary">#3F51B5</color>
	<colorname="colorPrimaryDark">#303F9F</color>
	<colorname="colorAccent">#FF4081</color>
	<colorname="textColorPrimary">#000</color>
</resources>
```

* Get the current state of Night mode;

```java
int mode = getResources().getConfiguration().uiMode & Configuration.UI_MODE_NIGHT_MASK;
```

* Add condition statements and set the right mode you want or do something you want.

```java
if(savedInstanceState == null) {
	  if (mode == Configuration.UI_MODE_NIGHT_YES) {
	      AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_NO);
	  } else if (mode == Configuration.UI_MODE_NIGHT_NO) {
	      AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_YES);
	  }
	  recreate();
}
```

####3, Tips
* DayNight theme does not apply to WebView, so you do need to do some adjustment;

* Don't use the MODE_NIGHT_AUTO as the default value since it needs the location permission. 
The calculation used to determine whether it is night or not makes use of the location APIs (if this app has the necessary permissions). This allows us to generate accurate sunrise and sunset times. If this app does not have permission to access the location APIs then we use hardcoded times which will be less accurate. 



