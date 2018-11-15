Hello Hugo
=========

### Part One: How to use

How to use Hugo is quite simple, and there are several steps you can follow:

Step One: add dependencies (by the date of Nov 13, 2018)

```groovy
//hugo
implementation 'com.jakewharton.hugo:hugo-plugin:1.2.1'
implementation 'com.jakewharton.hugo:hugo-annotations:1.2.1'
implementation 'com.jakewharton.hugo:hugo-runtime:1.2.1'

```

Step Two: use and outcomes

Sample:

```kotlin
class HelloHu: Activity() {
    private val TAG = "HUHU"

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val tv = TextView(this)
        tv.text = getString(R.string.txt_logcat)
        setContentView(tv)

        deLog(arrayOf("te", "er", "000", "wour", "[934-2"))
    }

    @DebugLog
    private fun deLog(args: Array<String>) {
        args.forEach {
            Log.i(TAG, it)
        }
    }
}
```

Output: TODO



### Part Two: How does it work
#### 2.1 Package structure

Hugo related dependencies include three parts: 1) plugin package; 2) annotation package; 3) runtime package

1. Plugin package:

![Plugin Package Structure](/imgs/20181114_hugo_plugin_structure.png)

The structure seems familiar, and it’s almost the same as what we did in custom plugin.
One of the most import parts will be the file — HugoPlugin.groovy. Let’s dive in the source code.
It’s quite straight forward, and only 77 lines. It implements gradle.api.Plugin:

```groovy
class HugoPlugin implements Plugin<Project>
```

Firstly, it will check whether this project has supported plugins. From the code, we can tell it requires android or android-library plugin.
```groovy
@Override void apply(Project project) {
  def hasApp = project.plugins.withType(AppPlugin)
  def hasLib = project.plugins.withType(LibraryPlugin)
  if (!hasApp && !hasLib) {
    throw new IllegalStateException("'android' or 'android-library' plugin required.")
  }
//...
}
```

Since Hugo is a plugin use to output debug info, then it will filter those variants that are non-debuggable.

```groovy
@Override void apply(Project project) {
//...
  final def log = project.logger
  final def variants
  if (hasApp) {
    variants = project.android.applicationVariants
  } else {
    variants = project.android.libraryVariants
  }

//...

  variants.all { variant ->
    if (!variant.buildType.isDebuggable()) {
      log.debug("Skipping non-debuggable build type '${variant.buildType.name}'.")
      return;
    }
//...
}

```

After that, an instance of MessageHandler attached to main thread will be created. We will look into Main and MessageHandler in package ‘org.aspectj’.

```groovy
MessageHandler handler = new MessageHandler(true);
new Main().run(args, handler);
for (IMessage message : handler.getMessages(null, true)) {
  switch (message.getKind()) {
    case IMessage.ABORT:
    case IMessage.ERROR:
    case IMessage.FAIL:
      log.error message.message, message.thrown
      break;
    case IMessage.WARNING:
      log.warn message.message, message.thrown
      break;
    case IMessage.INFO:
      log.info message.message, message.thrown
      break;
    case IMessage.DEBUG:
      log.debug message.message, message.thrown
      break;
  }
}
```

2. Annotation package:


3. Runtime package:

#### 2.2 How does it work


### Part Three: What we get from Hugo
#### 3.1 Build your own Hugo

#### 3.2 Scenario that can be applied with Hugo

#### Part Four: References



