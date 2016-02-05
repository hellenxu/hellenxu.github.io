UnsatisfiedLinkError after updating apps
============
During the project I have been working on, clients who use our sdk report a frequent crash caused by unsuccessfully loading our .so files, such as: 
> java.lang.UnsatisfiedLinkError: Couldn’t load xxx from loader dalvik.system.PathClassLoader[DexPathList[[zip file “/data/app/com.xx-2.apk”], nativeLibraryDirectories=[/vendor/lib, /system/lib]]]: findLibrary returned null
> at java.lang.Runtime.loadLibrary(Runtime.java:358)
> at java.lang.System.loadLibrary(System.java:526)


> java.lang.UnsatisfiedLinkError: dolmen failed: “/mnt/asec/com.xx-1/lib/libxx.so” has bad ELF magic
> at java.lang.Runtime.loadLibrary(Runtime.java:364)
> at java.lang.System.loadLibrary(System.java:555)



