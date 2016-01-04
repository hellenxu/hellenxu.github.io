Something about LeakCanary
==========================
LeakCanary is a convenient tool for Android and Java to detect memory leaks. Now we're going to talk about something about LeakCanary from three parts:<p>
1) How to use LeakCanary in an Android project;
2) How does it work;
3) Differences between MAT and LeakCanary.

Part One: how to use LeakCanary in an Android project

This part shorts for DIY (Dependencies && Install && onDestroy).

Step one: Dependencies means adding LeakCanary library in your build.gradle:
```groovy
dependencies {
	debugCompile 'com.square.leakcanary:leakcanary-android:1.3.1'
	releaseCompile 'com.square.leakcanary:leakcanary-android-no-op:1.3.1'
}

```

Step two: Install LeakCanary in your Application class, which will return a pre-configured RefWatcher as well as install an ActivityRefWatcher to detect whether an activity is leaking automatically after calling Activity.onDestroy.
```java
public class BaseApplication extends Application {
    private RefWatcher mRefWatcher;

    @Override
    public void onCreate(){
        super.onCreate();
        mRefWatcher = LeakCanary.install(this);
    }

    public static RefWatcher getRefWatcher(Context ctx){
        BaseApplication application = (BaseApplication)ctx.getApplicationContext();
        return application.mRefWatcher;
    }
}
```

Step three: Call RefWatcher.watch to watch for leaks.
```java
public class MainFragment extends Fragment {
    @Override
    public void onDestroy(){
        super.onDestroy();
        RefWatcher refWatcher = BaseApplication.getRefWatcher(this);
        refWatcher.watch(this);
    }
}
```

