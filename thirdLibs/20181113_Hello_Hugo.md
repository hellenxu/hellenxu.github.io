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
Basically, within the compile method, CompilerAdapter#execute() will be called. But how does it create an instance of CompilerAdapter? Let's move to CompilerAdapterFactory to find out how the instance of CompilerAdapter is created.
We found out within package of aj, there are two classes that implements CompilerAdapter interface：Ajc11CompilerAdapter and AjcCompilerAdapter. Then how CompilerAdapterFactory decide which type of instance is going to create? Are those two classes will be used?
```java
CompilerAdapter adapter =
    nestedAdapter != null ? nestedAdapter :
    CompilerAdapterFactory.getCompiler(compilerImpl, this, createCompilerClasspath());
```
From the code snippet above, we can see CompilerAdapterFactory#getCompiler() receives three arguments: 1) compilerImpl(it comes from Javac#getCompiler(), and usually returns "extJavac”); 2) this — is a ProjectComponent task; 3) compiler classpath (actually it returns the path of project)

```java
//Javac#createCompilerClasspath()
public Path createCompilerClasspath() {
    return facade.getImplementationClasspath(getProject());
}

//FacadeTaskHelper#getImplementationClasspath(getProject())
public Path getImplementationClasspath(Project project) {
    if (implementationClasspath == null) {
        implementationClasspath = new Path(project);
    }
    return implementationClasspath;
}
```

||ヽ(*￣▽￣*)ノミ|Ю then what will be the created compiler? Still remember the first argument compilerImpl, which is extJavac. So from the code snippet following, it will return an instance of JavacExternal.
```java
//CompilerAdapterFactory#getCompiler()
public static CompilerAdapter getCompiler(String compilerType, Task task,
                                          Path classpath)
    throws BuildException {
        ...
        if (compilerType.equalsIgnoreCase("extJavac")) {
            return new JavacExternal();
        }
        ...
        return resolveClassName(compilerType,
                                // Memory-Leak in line below
                            task.getProject().createClassLoader(classpath));
    }
```

What is JavacExternal? This class extends DefaultCompilerAdapter and DefaultCompilerAdapter implements CompilerAdapter. execute() is the most important method of JavacExternal.

Firstly, it creates an instance of Commandline, and passes the current javac executable to Commandline. Then call different methods based on the version of java.
- setupJavacCommandlineSwitches(): for java1.1 or java1.2;
- setupModernJavacCommandlineSwitches(): for java 1.3 and above.

```java
//JavacExternal#execute()
public boolean execute() throws BuildException {
    attributes.log("Using external javac compiler", Project.MSG_VERBOSE);

    Commandline cmd = new Commandline();
    cmd.setExecutable(getJavac().getJavacExecutable());
    if (!assumeJava11() && !assumeJava12()) {
        setupModernJavacCommandlineSwitches(cmd);
    } else {
        setupJavacCommandlineSwitches(cmd, true);
    }
    int firstFileName = assumeJava11() ? -1 : cmd.size();
    logAndAddFilesToCompile(cmd);
    //On VMS platform, we need to create a special java options file
    //containing the arguments and classpath for the javac command.
    //The special file is supported by the "-V" switch on the VMS JVM.
    if (Os.isFamily("openvms")) {
        return execOnVMS(cmd, firstFileName);
    }
    return
            executeExternalCompile(cmd.getCommandline(), firstFileName,
                    true)
            == 0;
}

```

Now, take a look at what is under the hood of setupModernJavacCommanlineSwitches(). In fact, this method will call setupJavacCommandlineSwitches(Commandline cmd, boolean useDebugLevel) directly.
Thus, let's see the source code of this method.
- 1.1: setupJavacCommandlineSwitches(Commandline cmd, boolean useDebugLevel);

- 1.2: set arguments of commandline;

1). Scenario One: Javac#getSource() != null and non-java1.3
As comments in this class, "support for -source 1.1 and -source 1.2 has been added with JDK 1.4.2 and isn’t present in 1.5.0 or 1.6.0 either."
when javac source is 1.5 or 1.6, attributes is null; then it will move to scenario two.

2). Javac#getSource() == null or java1.3

Now matter which situation, it will set the right argument value for the commandline by cmd.createArgument().setValue(s)

- 1.3: return the instance of Commandline.

```java
//DefaultCompilerAdapter#setupModernJavacCommandlineSwitches(Commandline cmd)
protected Commandline setupModernJavacCommandlineSwitches(Commandline cmd) {
    setupJavacCommandlineSwitches(cmd, true);
    if (attributes.getSource() != null && !assumeJava13()) {
        cmd.createArgument().setValue("-source");
        String source = attributes.getSource();
        if (source.equals("1.1") || source.equals("1.2")) {
            // support for -source 1.1 and -source 1.2 has been
            // added with JDK 1.4.2 - and isn't present in 1.5.0
            // or 1.6.0 either
            cmd.createArgument().setValue("1.3");
        } else {
            cmd.createArgument().setValue(source);
        }
    } else if ((assumeJava15() || assumeJava16())
               && attributes.getTarget() != null) {
        String t = attributes.getTarget();
        if (t.equals("1.1") || t.equals("1.2") || t.equals("1.3")
            || t.equals("1.4")) {
            String s = t;
            if (t.equals("1.1")) {
                // 1.5.0 doesn't support -source 1.1
                s = "1.2";
            }
            attributes.log("", Project.MSG_WARN);
            attributes.log("          WARNING", Project.MSG_WARN);
            attributes.log("", Project.MSG_WARN);
            attributes.log("The -source switch defaults to 1.5 in JDK 1.5 and 1.6.",
                           Project.MSG_WARN);
            attributes.log("If you specify -target " + t
                           + " you now must also specify -source " + s
                           + ".", Project.MSG_WARN);
            attributes.log("Ant will implicitly add -source " + s
                           + " for you.  Please change your build file.",
                           Project.MSG_WARN);
            cmd.createArgument().setValue("-source");
            cmd.createArgument().setValue(s);
        }
    }
    return cmd;
}
```


### Part Three: What we get from Hugo
#### 3.1 Build your own Hugo

#### 3.2 Scenario that can be applied with Hugo

#### Part Four: References



