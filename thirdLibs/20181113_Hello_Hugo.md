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
![Plugin Package Structure](/imgs/20181119_hugo_annotation_structure.png)

The annotation package is quite simple, and only includes one annotation class: DebugLog, which makes sense since this library is used to log info.
Now, let’s just take a look at how it looks.

```java
package hugo.weaving;

import java.lang.annotation.Retention;
import java.lang.annotation.Target;

import static java.lang.annotation.ElementType.CONSTRUCTOR;
import static java.lang.annotation.ElementType.METHOD;
import static java.lang.annotation.ElementType.TYPE;
import static java.lang.annotation.RetentionPolicy.CLASS;

@Target({TYPE, METHOD, CONSTRUCTOR}) @Retention(CLASS)
public @interface DebugLog {
}
```

Apparently, DebugLog is applied to type, method and constructor on the class level.


3. Runtime package:
![Plugin Package Structure](/imgs/20181119_hugo_runtime_structure.png)

Now, it’s the runtime package. Within this package, it has Hugo(the core class), Strings(Strings util), and BuildConfig(configuration info). Let’s crack them one by one.
3.1) Hugo.java
This class uses several annotations of aspectJ: @Aspect, @Pointcut, @Around, and the main one is method Hugo#logAndExecute(ProceedingJoinPoint joinPoint) marked with annotation @Around. Others are basically helper methods.
Well, let's dive into this method, logAndExecute().
```java
//Hugo#logAndExecute()
@Around("method() || constructor()")
public Object logAndExecute(ProceedingJoinPoint joinPoint) throws Throwable {
  enterMethod(joinPoint); //construct printable string for class, method, param name and values

  long startNanos = System.nanoTime();
  Object result = joinPoint.proceed();
  long stopNanos = System.nanoTime();
  long lengthMillis = TimeUnit.NANOSECONDS.toMillis(stopNanos - startNanos);

  exitMethod(joinPoint, result, lengthMillis); //construct printable string for return type

  return result;
}
```

Hugo#enterMethod
```java
private static void enterMethod(JoinPoint joinPoint) {
  CodeSignature codeSignature = (CodeSignature) joinPoint.getSignature();

  Class<?> cls = codeSignature.getDeclaringType();
  String methodName = codeSignature.getName();
  String[] parameterNames = codeSignature.getParameterNames();
  Object[] parameterValues = joinPoint.getArgs();

  StringBuilder builder = new StringBuilder("\u21E2 ");
  builder.append(methodName).append('(');
  for (int i = 0; i < parameterValues.length; i++) {
    if (i > 0) {
      builder.append(", ");
    }
    builder.append(parameterNames[i]).append('=');
    builder.append(Strings.toString(parameterValues[i]));//change params into printable strings
  }
  builder.append(')');

  if (Looper.myLooper() != Looper.getMainLooper()) {
    builder.append(" [Thread:\"").append(Thread.currentThread().getName()).append("\"]");
  }

  Log.v(asTag(cls), builder.toString()); //asTag(cls) is used to get the tag of an anonymous class
}
```

Basically, enterMethod is used to get all information that needed to print log: method name, parameter names and values.
A). Get code signature from JoinPoint
B). Get class, methodName, parameterNames, parameterValues
C). Thread name if it’s the current thread is not main thread

Hugo#exitMethod

```java
private static void exitMethod(JoinPoint joinPoint, Object result, long lengthMillis) {
  Signature signature = joinPoint.getSignature();

  Class<?> cls = signature.getDeclaringType();
  String methodName = signature.getName();
  boolean hasReturnType = signature instanceof MethodSignature
      && ((MethodSignature) signature).getReturnType() != void.class; //check whether this method has void return type

  StringBuilder builder = new StringBuilder("\u21E0 ")
      .append(methodName)
      .append(" [")
      .append(lengthMillis)
      .append("ms]");

  if (hasReturnType) {
    builder.append(" = ");
    builder.append(Strings.toString(result));
  }

  Log.v(asTag(cls), builder.toString());
}
```

Between enterMethod and exitMethod, it calculates time cost of ProceedingJoinPoint proceeding.

3.2) Strings.java
Strings is a util class that provides printable string.

3.3) BuildConfig.java
BuildConfig provides Hugo config information like: APPLICATION_ID, BUILD_TYPE,   FLAVOR, VERSION_CODE AND VERSION_NAME.


#### 2.2 How those three packages work together
It’s HugoPlugin.groovy which connects all together. There is a task ‘JavaCompile’, and within its doLast section, that’s the key point to connect runtime and annotation packages.

```groovy
JavaCompile javaCompile = variant.javaCompile
javaCompile.doLast {
  String[] args = [
      "-showWeaveInfo”,    //showWeaveInfo
      "-1.5”,        //java5
      "-inpath", javaCompile.destinationDir.toString(),
      "-aspectpath", javaCompile.classpath.asPath,
      "-d", javaCompile.destinationDir.toString(),
      "-classpath", javaCompile.classpath.asPath,
      "-bootclasspath", project.android.bootClasspath.join(File.pathSeparator)
  ]
  log.debug "ajc args: " + Arrays.toString(args)

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
}
```

Args are mostly from Options class from package ‘org.aspectj.weaver.loadtime’. For example, ‘-1.5’ means setting a boolean value of WeaverOption#java5.

Okay, let's go through the whole flow by walking through JavaCompile task. There is a class called Javac in package org.apache.tools.ant.taskdefs; and every time when we running javac to compile java file into classes file, Hugo is going to iterate all annotated type, method, and classes to output logs.

Firstly, Javac.

```java
//Source Code: Javac#compile()
protected void compile() {
    String compilerImpl = getCompiler();

    if (compileList.length > 0) {
    ...
        CompilerAdapter adapter =
            nestedAdapter != null ? nestedAdapter :
            CompilerAdapterFactory.getCompiler(compilerImpl, this,
                                               createCompilerClasspath());

        // now we need to populate the compiler adapter
        adapter.setJavac(this);

        // finally, lets execute the compiler!!
        if (adapter.execute()) {    //is used to parse
            // Success
            ...
        } else {
            // Fail path
            ...
        }
    }
}
```


### Part Three: What we get from Hugo
#### 3.1 Build your own Hugo

#### 3.2 Scenario that can be applied with Hugo

#### Part Four: References



