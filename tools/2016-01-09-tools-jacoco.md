Get test units coverage
======

Environment
------
> Espresso

> Android Studio

> Jacoco

How to do
-----
### Step1
In your build.gradle, enable testCoverage.
```groovy
android{
    compileSdkVersion 22
    buildToolsVersion "22.0.1"
    defaultConfig {
        applicationId 'com.six.practice'
        minSdkVersion 8
        targetSdkVersion 22
        versionCode 1
        VersionName '1.1'
        testInstrumentationRunner 'android.support.test.runner.AndroidJUnitRunner'
    }
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }

        debug {
            testCoverageEnabled true
        }
    }
    packagingOptions {
        exclude('LICENSE.txt')
    }
}
```
However, you will find out all the data of coverage statistics is zero by running gradle task **createDebugCoverageReport**.
It seems that android\' DVM is not 100% compatible with JVM, so we need to move to another step.

### Step2
Write your own AndroidJUnitRunner and remember to use it in the build.gradle file, which mainly is used to reset the path of *coverage.ec* as well as to call *dump* method using reflection.
```java
public class JUnitJacocoTestRunner extends AndroidJUnitRunner {
    static {
        final String path = "/data/data/" + BuildConfig.APPLICATION_ID + "/coverage.ec";
        System.setProperty("jacoco-agent.destfile", path);
    }

    @Override
    public void finish(int resultCode, Bundle results) {
        try {
            Class rt = Class.forName("org.jacoco.agent.rt.RT");
            Method getAgent = rt.getMethod("getAgent");
            Method dump = getAgent.getReturnType().getMethod("dump", boolean.class);
            Object agent = getAgent.invoke(null);
            dump.invoke(agent, false);
        }catch(ClassNotFoundException e){
            e.printStackTrace();
        }catch(Exception e){
            e.printStackTrace();
        }
    }
}
```
ps:
1. The method *getAgent* is one static method of RunTime Class
![](/imgs/20160109_1.png)

2. The method *dump* belongs to interface IAgent, which is used to collect data, I guess.
![](/imgs/20160109_2.png)

### Step3
In the build.gradle, add _jacoco_ plugin and your JacocoReport task.
```groovy
apply plugin: 'jacoco'
jacoco {
    toolVersion "0.7.4.201502262128"
}

task jacocoTestReport(type:JacocoReport, dependsOn: "connectedAndroidTest"){
    group = "Reporting"

    def fileFilter = ['**/R.class', '**/R&*.class', '**/Manifest*.*',
                    '**/BuildConfig.*', 'android/**/*.*']
    def debugTree = fileTree(dir: "${project.buildDir}/intermediates/classes/debug", excludes:fileFilter)
    def mainSrc = "${project.projectDir}/src/main/java"
    sourceDirectories = files([mainSrc])
    classDirectories = files([debugTree])
    addtionalSourceDirs = files(["${buildDir}/generated/source/buildConfig/debug",
                            "${buildDir}/generated/source/r/debug"])
    executionData = fileTree(dir:project.projectDir, includes: ['**/*.exec', '**/*.ec'])

    reports {
        xml.enabled = true
        xml.destination = "${buildDir}/jacocoTestReport.xml"
        csv.enabled = false
        html.enabled = true
        html.destination = "${buildDir}/reports/jacoco"
    }
}
```

Now, we can run *jacocoTestReport* to get our coverage data which is located in *build/reports/jacoco* directory.
