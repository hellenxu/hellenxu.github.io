Hugo Advanced
=========

### Part One: How those three packages work together
How do those three packages -- plugin, annotation, and runtime, work together? It’s HugoPlugin.groovy which connects all together. There is a task ‘JavaCompile’, and within its doLast section, that’s the key point to connect runtime and annotation packages.

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

#### Part Two: What we get from Hugo?
1. Scenarios that can use aspectJ
Logger, data persistence, behaviour monitoring, data validation, caching.

2. How to make your own Hugo?
2.0 Preparation
Step One: add an android library module and configure build.gradle file of this new module.
```groovy
//build.gradle[timecost]
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'org.aspectj:aspectjtools:1.8.9'
    }
}

dependencies {
    //aj
    implementation 'org.aspectj:aspectjrt:1.8.9'
}
```

Step Two: configure JavaCompile task

```groovy
import org.aspectj.bridge.IMessage
import org.aspectj.bridge.MessageHandler
import org.aspectj.tools.ajc.Main

android.libraryVariants.all { variant ->
    JavaCompile javaCompile = variant.javaCompile
    javaCompile.doLast {
        String[] args = ["-showWeaveInfo",
                         "-1.5",
                         "-inpath", javaCompile.destinationDir.toString(),
                         "-aspectpath", javaCompile.classpath.asPath,
                         "-d", javaCompile.destinationDir.toString(),
                         "-classpath", javaCompile.classpath.asPath,
                         "-bootclasspath", project.android.bootClasspath.join(File.pathSeparator)]

        MessageHandler handler = new MessageHandler(true)
        new Main().run(args, handler)

        def log = project.logger
        for (IMessage message : handler.getMessages(null, true)) {
            switch (message.getKind()) {
                case IMessage.ABORT:
                case IMessage.ERROR:
                case IMessage.FAIL:
                    log.error message.message, message.thrown
                    break
                case IMessage.WARNING:
                case IMessage.INFO:
                    log.info message.message, message.thrown
                    break
                case IMessage.DEBUG:
                    log.debug message.message, message.thrown
                    break
            }
        }
    }
}
```

2.1 Annotation
AOP usually uses annotations to mark the entrance of point cuts, therefore we need an annotation class first.
Annotation class is quiet simple, you just need to define RetentionPolicy and Target. For example:
```java
@Retention(RetentionPolicy.CLASS)
@Target({ElementType.METHOD, ElementType.CONSTRUCTOR})
    public @interface DebugTimeCost {
}
```

2.2 Aspect
Aspect is quite simple, and it basically is those codes which would like to insert into Pointcut.
Then what is Pointcut? Pointcut is a mark which tells us where to insert codes.

```java
@Aspect
public class TimeCost {
    private static final String POINTCUT_METHOD = "execution(@ca.six.timecost.annotation.DebugTimeCost * *(..))";
    private static final String POINTCUT_CONSTRUCTOR = "execution(@ca.six.timecost.annotation.DebugTimeCost *.new(..))";

    @Pointcut(POINTCUT_METHOD)
    public void method(){}

    @Pointcut(POINTCUT_CONSTRUCTOR)
    public void constructor() {}

    @Around("method() || constructor()")
    public Object weaveInfo(ProceedingJoinPoint joinPoint) throws Throwable {
        MethodSignature methodSig = (MethodSignature) joinPoint.getSignature();
        String clzName = methodSig.getDeclaringType().getSimpleName();
        String methodName = methodSig.getMethod().getName();

        TimeCostHelper timeHelper = new TimeCostHelper();
        timeHelper.startTimer();
        Object result = joinPoint.proceed();
        timeHelper.stopTimer();
        Logger.d(clzName, Logger.buildLog(methodName, timeHelper.getElapsed()));

        return result;
    }

}
```
Like the sample above, we set two Pointcuts: one is methods with any arguments and any return value, annotated with @DebugTimeCost;
the other is constructors with any arguments, annotated with @DebugTimeCost.
These two pointcuts are execution type, which means it happens when annotated methods or constructors are executed.
In the example above, we start to log the time at the beginning of execution of the pointcuts; when methods are proceeded, we log the finishing time to calculate how long a method takes to execute all codes within it.

2.3 Helper methods
In this calculate method time cost example, we have two helpers: one is to log start time, stop time and elapsed time; the other is to format output.

#### Part Three: References
[DocGuide](https://www.eclipse.org/aspectj/doc/released/progguide/starting-aspectj.html)
[JoinPoints](https://www.eclipse.org/aspectj/doc/next/progguide/language-joinPoints.html)
[Advice](https://www.eclipse.org/aspectj/doc/next/adk15notebook/ataspectj-pcadvice.html)
