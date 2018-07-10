# baidutvsafe.github.io


风险apk样本在githubfengxian文件夹里，可以下载。
也可以去百度云链接下载，https://pan.baidu.com/s/1NN_Afvji_BVfmSBRhh1wHQ 密码: fuy9

#Q&A
##UnsatisfiedLinkError导致网络安全扫描不可用。
一般这种问题就是so平台混用导致的。
07-10 16:38:05.837 W/System.err(14028): java.lang.UnsatisfiedLinkError: dalvik.system.PathClassLoader[DexPathList[[zip file "/data/app/com.coocaa.tvmanager-1/base.apk"],nativeLibraryDirectories=[/data/app/com.coocaa.tvmanager-1/lib/arm, /vendor/lib, /system/lib]]] couldn't find "libtvshield.so"
07-10 16:38:05.838 W/System.err(14028):         at java.lang.Runtime.loadLibrary(Runtime.java:366)
07-10 16:38:05.838 W/System.err(14028):         at java.lang.System.loadLibrary(System.java:988)
07-10 16:38:05.838 W/System.err(14028):         at com.baidu.tvshield.jni.Asc.<clinit>(Asc.java:10)
07-10 16:38:05.838 W/System.err(14028):         at com.baidu.tvshield.b.h.a(EncryptConnUtil.java:48)
07-10 16:38:05.838 W/System.err(14028):         at com.baidu.tvshield.ac.U.run(U.java:3030)
  出了这个问题，一般看看自己编译出的apk是什么架构，为了兼容性，我们目前只提供armabi架构的so。
  如果你们是armabiv7a,把我们的aar自己改一下，把两个so丢进你们的v7a就OK了。
