Gradle Basic
===========

0X00: General Configuration
------------


0X01: Parameters Configuration
-------------


0X02: NDK Configuration
-------------

0X03: Gradle Speeding Up
-------------
There are a few tips that can help us speed up gradle building.
1, Enable daemon
Using daemon will send initialization works, such as jvm setup, loading virtual machine and .class files, to an independent background process, which reduces building time.
```groovy
#gradle.properties
org.gradle.daemon=true
```
2, Enable parallel building
Parallel building is useful for projects that have more than two modules, which allows several modules to build at the same time.
```groovy
#gradle.properties
org.gradle.parallel=true
```
3, Use fixed version when introducing library dependencies.
```groovy
dependencies {
    compile 'com.android.support:appcompat-v7:23.3.0'
}
```
is better than
```groovy
dependencies {
    compile 'com.android.support:appcompat-v7:23.+'    
}
```
Gradle will pull the latest version of dependencies when using '+' to specify version ranges, in this case, we don't have to worry about updating issues about dependent libraries. However, it may cause potential problems like slowing down gradle build as Gradle will try to fetch the newest version from repositories. Also, you need to change your code base if there are changes of third party libraries, such as package name and API, to avoid gradle build failure.
4, Avoid heavy computation in build.gradle file
Try to avoid heavy computation, like I/O operations, in build.gradle file. 
```groovy
android{
    compileSdkVersion23
    buildToolsVersion"23.0.3"
 
    defaultConfig{
        applicationId"com.six.tipsproject"
        minSdkVersion15
        targetSdkVersion23
        versionCode gitVersion()
        versionName"1.0"
    }
}
def gitVersion(){
if(!System.getenv('CI_BUILD')){
    return1
}
def cmd='gitrev-listHEAD--first-parent--count'
    cmd.execute().text.trim().toInteger()
}
```
is better than
```groovy
def cmd='gitrev-listHEAD--first-parent--count'
defgitVersion=cmd.execute().text.trim().toInteger()
android{
    compileSdkVersion23
    buildToolsVersion"23.0.3"
 
    defaultConfig{
        applicationId"com.six.tipsproject"
        minSdkVersion15
        targetSdkVersion23
        versionCode gitVersion
        versionName"1.0"
    }
}
```
Since in the first example, it checks whether it's development environment or not, and then return different versionCode, which saves us a bit time for us.
