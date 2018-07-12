# baidutvsafe.github.io


风险apk样本在githubfengxian文件夹里，可以下载。
也可以去百度云链接下载，https://pan.baidu.com/s/1NN_Afvji_BVfmSBRhh1wHQ 密码: fuy9

# Q&A
## UnsatisfiedLinkError导致网络安全扫描不可用。
一般这种问题就是so平台混用导致的。
07-10 16:38:05.837 W/System.err(14028): java.lang.UnsatisfiedLinkError: dalvik.system.PathClassLoader[DexPathList[[zip file "/data/app/com.coocaa.tvmanager-1/base.apk"],nativeLibraryDirectories=[/data/app/com.coocaa.tvmanager-1/lib/arm, /vendor/lib, /system/lib]]] couldn't find "libtvshield.so"
07-10 16:38:05.838 W/System.err(14028):         at java.lang.Runtime.loadLibrary(Runtime.java:366)
07-10 16:38:05.838 W/System.err(14028):         at java.lang.System.loadLibrary(System.java:988)
07-10 16:38:05.838 W/System.err(14028):         at com.baidu.tvshield.jni.Asc.<clinit>(Asc.java:10)
07-10 16:38:05.838 W/System.err(14028):         at com.baidu.tvshield.b.h.a(EncryptConnUtil.java:48)
07-10 16:38:05.838 W/System.err(14028):         at com.baidu.tvshield.ac.U.run(U.java:3030)
  出了这个问题，一般看看自己编译出的apk是什么架构，为了兼容性，我们目前只提供armabi架构的so。
  如果你们是armabiv7a,把我们的aar自己改一下，把两个so丢进你们的v7a就OK了。
  
  ## java.lang.NoClassDefFoundError: c/b/libccb/util/JsonParser
  这个Log之前，前面一般jvm会再打印一条Rejecting re-init on previously-failed class Lc/b/libccb/util/JsonParser; v=0x0
  从虚拟机源码来看，现场不在这里。
  源码位置：
  android-4.4.0_r1.0/xref/art/runtime/class_linker.cc
  
`static void ThrowEarlierClassFailure(mirror::Class* c)
      SHARED_LOCKS_REQUIRED(Locks::mutator_lock_) {
    // The class failed to initialize on a previous attempt, so we want to throw
    // a NoClassDefFoundError (v2 2.17.5).  The exception to this rule is if we
    // failed in verification, in which case v2 5.4.1 says we need to re-throw
    // the previous error.
    if (!Runtime::Current()->IsCompiler()) {  
        // Give info if this occurs at runtime.
        LOG(INFO) << "Rejecting re-init on previously-failed class " << PrettyClass(c);
    }
    
    CHECK(c->IsErroneous()) << PrettyClass(c) << " " << c->GetStatus();
    Thread* self = Thread::Current();
    ThrowLocation throw_location = self->GetCurrentLocationForThrow();
    if (c->GetVerifyErrorClass() != NULL) {
        // TODO: change the verifier to store an _instance_, with a useful detail message?
        ClassHelper ve_ch(c->GetVerifyErrorClass());
        self->ThrowNewException(throw_location, ve_ch.GetDescriptor(), PrettyDescriptor(c).c_str());
    } else {
        self->ThrowNewException(throw_location, "Ljava/lang/NoClassDefFoundError;",
                          PrettyDescriptor(c).c_str());
    }
}`
  
  再结合之前的log来看：
  
  `47:50.200 4545-4597/com.coocaa.tvmanager:p1 W/dalvikvm: VFY: unable to resolve static method 8532: Lcom/google/gson/internal/$Gson$Types;.e (Ljava/lang/reflect/Type;)Ljava/lang/Class;
07-11 10:47:50.200 4545-4597/com.coocaa.tvmanager:p1 D/dalvikvm: VFY: replacing opcode 0x71 at 0x003c
07-11 10:47:50.200 4545-4597/com.coocaa.tvmanager:p1 I/dalvikvm: Could not find method com.google.gson.internal.$Gson$Types.f, referenced from method com.google.gson.b.a.toString
07-11 10:47:50.200 4545-4597/com.coocaa.tvmanager:p1 W/dalvikvm: VFY: unable to resolve static method 8533: Lcom/google/gson/internal/$Gson$Types;.f (Ljava/lang/reflect/Type;)Ljava/lang/String;
07-11 10:47:50.200 4545-4597/com.coocaa.tvmanager:p1 D/dalvikvm: VFY: replacing opcode 0x71 at 0x0002
07-11 10:47:50.225 4545-4597/com.coocaa.tvmanager:p1 W/dalvikvm: Exception Ljava/lang/NoSuchMethodError; thrown while initializing Lcom/google/gson/e;
    Exception Ljava/lang/NoSuchMethodError; thrown while initializing Lc/b/libccb/util/JsonParser;
07-11 10:47:50.225 4545-4597/com.coocaa.tvmanager:p1 I/avpsdk: [thread : pool-3-thread-1 : 325] [EnterpriseEditionAvpScanEngine] Enterprise startScan exception : com.google.gson.internal.$Gson$Types.d
07-11 10:47:50.225 4545-4597/com.coocaa.tvmanager:p1 W/System.err: java.lang.NoSuchMethodError: com.google.gson.internal.$Gson$Types.d
07-11 10:47:50.230 4545-4597/com.coocaa.tvmanager:p1 W/System.err:     at com.google.gson.b.a.getSuperclassTypeParameter(TypeToken.java:87)`
  
  很明显一定是你们是目标系统集成了gson的某个版本，有可能版本不匹配或者是没有混淆，在双亲委派时候，优先使用系统类中已经加载过的gson，里面细节对不上，再在这个加载好的类里去找某个类，也就classnofound了。最后解决办法就是我们的gson不混淆。这样可以规避问题，不过如果gson版本差异过大，那只有最后的解决方案：就是我们的gson源码拉下来，改个包名。
