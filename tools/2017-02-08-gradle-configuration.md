Gradle Basic
===========

0X00: General Configuration
------------
1, Configure signature information
Signature information is private, and it is not a good idea to store these information in build.gradle file as well as push them to version control system, like git, svn. Instead, it would be better to save them in a local configuration file, such as gradle.properties, local.properties or a custom configuration file that may need to write gradle task to read.
Let's take saving signature information in local.properties for example.
```groovy
#local.properties
RELEASE_KEY_PASSWORD=android
RELEASE_KEY_ALIAS=androidreleasekey
RELEASE_STORE_PASSWORD=android
RELEASE_STORE_FILE=E\:\\signature\\testing\\release.jks
DEBUG_KEY_PASSWORD=android
DEBUG_KEY_ALIAS=androiddebugkey
DEBUG_STORE_PASSWORD=android
DEBUG_STORE_FILE=E\:\\signature\\testing\\debug.keystore
```
If your signature info is not saved in gradle.properties, then you need to write a task to read them, and reading information task is as below:
```groovy
task checkSignInfo {
    File propFile = project.rootProject.file('local.properties');
    if(propFile.exists() && propFile.canRead()){
        println("loading signing configuration...")
        def Properties properties = new Properties()
        properties.load(new FileInputStream(propFile))

        if(properties.containsKey('RELEASE_STORE_FILE')
                && properties.containsKey('RELEASE_STORE_PASSWORD')
                && properties.containsKey('RELEASE_KEY_ALIAS')
                && properties.containsKey('RELEASE_KEY_PASSWORD')){
            android.signingConfigs.release.storeFile = file(properties['RELEASE_STORE_FILE'])
            android.signingConfigs.release.storePassword = properties['RELEASE_STORE_PASSWORD']
            android.signingConfigs.release.keyAlias = properties['RELEASE_KEY_ALIAS']
            android.signingConfigs.release.keyPassword = properties['RELEASE_KEY_PASSWORD']
        } else {
            android.buildTypes.release.signingConfig = null
            println('signing config found but some entries are missing.')
        }
    } else {
        android.buildTypes.release.signingConfig = null
        println('signing config not found')
    }
}
assembleRelease.dependsOn checkSignInfo
assembleAndroidTest.dependsOn checkSignInfo
```
2, Set Maven repository address
Name and credentials are optional. Sample is as follows:
```groovy
#build.gradle
allprojects {
   repositories {
       maven {
           url 'url'
           name 'maven name'
           credentials {
               username = 'username'
               password = 'password'
           }
       }
   }
}
```

3, Split up build.gradle
If there are too many tasks in build.gradle, you can separate them based on functions or other matching criteria, and then apply in build.gradle file just like below example:
```groovy
#build.gradle
apply from: "config.gradle"
```
4, Define global variables and use them
Have you encountered this situation? There are more than two modules in your project and you get a task to update configuration info for all modules. Then you might need to modify build.gradle file of each module. If there is one module left that is not changed or uses different version of build tools. In this case, there might be version conflict errors when building gradle. Defining global variables for all projects and use them in submodules will address those issues and save you a second because you just need to change info in configuration file instead of build.gradle in each module.
+ Step one: define global variables
```groovy
#config.gradle[save global info]
ext {
    android = [compileSdkVersion: 23,
               buildToolsVersion: "23.0.3",
               minSdkVersion: 16,
               targetSdkVersion: 23,
               versionCode: gitVersion(),
               versionName: "1.0"]

    dependencies = ["appcompat-v7": 'com.android.support:appcompat-v7:23.3.0',
                    "junit": 'junit:junit:4.12']
}
```
+ Step two: apply in build.gradle that located in root project
```groovy
#build.gradle
apply from: "config.gradle"
```
+ Step three: use global variables in submodules
```groovy
android {
    compileSdkVersion rootProject.ext.android.compileSdkVersion
    buildToolsVersion rootProject.ext.android.buildToolsVersion
}
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    compile rootProject.ext.dependencies["appcompat-v7"]
    //Testing dependencies
    testCompile rootProject.ext.dependencies["junit"]
}
```
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
Since in the first example, it checks whether it's development environment or not, and then return different versionCode, which saves us a bit time.
