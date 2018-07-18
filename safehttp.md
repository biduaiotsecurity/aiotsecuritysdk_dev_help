# SafeHttp接入文档 
## 说明

## SDK功能
SDK的功能包含SSL防护（MesaLink）与DNS防护（HttpDNS）

SDK中的DNS防护功能包含两种，一种只能覆盖HTTPS连接的DNS请求，另外一种能够覆盖所有HTTP连接的DNS请求

SDK的接入方式有两种，一种是集成到APP当中，可以解决接入APP的SSL安全和DNS安全；另一种方式是集成到Android Framework当中，可以解决整个Android系统，以及所有APP的SSL安全和DNS安全。

接入方式与防护功能可以任意搭配

PS：目前还不能解决Native库和WebView的SSL安全。

## SDK文件内容
不同接入方式的SDK文件内容稍有不同

SDK for APP：

总共一个文件

safehttp-release.aar

SDK for Framework：

总共两个文件夹

jniLibs文件夹：分别包含armabi-v7a与arm64-v8a两个架构的so库，对应分别存放与arm与arm64子文件夹内

libs文件夹：包含一个jar包safehttp.jar

## 集成方法
### 1 集成到APP当中
#### 1.1 添加aar文件到APP工程
将safehttp-release.aar拷贝到需要引入的module的libs目录下

#### 1.2 添加依赖
```
repositories {
    flatDir {
        dirs 'libs'
    }
}

dependencies {
    compile(name: 'safehttp-release', ext: 'aar')
}
```

#### 1.3 混淆
```
-keep class com.baidu.safehttp.** { *; }
-keep class * implements android.os.IInterface {*;}
```

#### 1.4 使用
```
Application：
@Override
public void onCreate() {
    super.onCreate();
    SafeHttp.init(this);
}
```

### 2 集成到Android Framework当中
#### 2.1 拷贝文件
在Android4.4.4及以下系统，找到libcore文件夹
在Android5.0及以上系统，找到external/conscrypt文件夹
将libs与jniLibs文件夹拷贝到其中
#### 2.2 修改编译脚本
在Andriod4.4.4及以下系统，修改libcore/JavaLibrary.mk
在Android5.0及以上系统，修改external/conscrypt/Android.mk

```
core_cflags := -Wall -Wextra -Werror
core_cppflags := -std=gnu++11

# add these lines
include $(CLEAR_VARS)
LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES := safehttp:libs/safehttp.jar
LOCAL_MODULE_TAGS := optional
include $(BUILD_MULTI_PREBUILT)

include $(CLEAR_VARS)
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE := libmesalink-jni
LOCAL_MODULE_SUFFIX := .so
LOCAL_MODULE_CLASS := SHARED_LIBRARIES
LOCAL_SRC_FILES_arm := jniLibs/arm/libmesalink-jni.so
###### 如果是64位系统，添加以下几行 #####
#LOCAL_SRC_FILES_arm64 := jniLibs/arm64/libmesalink-jni.so
#LOCAL_MODULE_TARGET_ARCHS := arm arm64
#LOCAL_MULTILIB := both
##### end #####
include $(BUILD_PREBUILT)
# add end

# Build for the target (device).
#

# Create the conscrypt library
include $(CLEAR_VARS)
LOCAL_SRC_FILES := $(call all-java-files-under,src/main/java)
LOCAL_SRC_FILES += $(call all-java-files-under,src/platform/java)
LOCAL_JAVA_LIBRARIES := core-libart
# add this line
LOCAL_STATIC_JAVA_LIBRARIES := safehttp
# add end
LOCAL_NO_STANDARD_LIBRARIES := true
LOCAL_JAVACFLAGS := $(local_javac_flags)
LOCAL_JARJAR_RULES := $(LOCAL_PATH)/jarjar-rules.txt
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE := conscrypt
LOCAL_REQUIRED_MODULES := libjavacrypto
LOCAL_ADDITIONAL_DEPENDENCIES := $(LOCAL_PATH)/Android.mk
include $(BUILD_JAVA_LIBRARY)
```

#### 2.3 添加编译项目
修改build/target/product/core.mk，PRODUCT_PACKAGES的最后一行在添加以下内容：
```
PRODUCT_PACKAGES += \
    BasicDreamd \
    ......
    MmsService \
# add this line, and a '\' at the end of the previous line
    libmesalink-jni
```

#### 2,4 添加模块白名单
修改build/core/tasks/check_boot_jars/package_whitelist.txt，在最后添加以下内容：
```
com\.baidu\..*
```

#### 2.5 修改te规则
在Android7.1及以下系统，修改文件system/sepolicy/system_server.te
在Android8.0及以上系统，修改文件system/sepolicy/private/system_server.te
将
```
neverallow system_server self:process execmem;
```
修改为
```
allow system_server self:process execmem;
```
如果没有找到这个文件或者文件中没有这一行内容，请忽略这一项

#### 2.6 添加入口（仅在集成DNS for HTTP时需要）
修改类文件frameworks/base/core/java/android/app/Application.java
```
// add import
import com.baidu.safehttp.SafeHttp;

// modify this method
public void onCreate() {
    SafeHttp.init(this);
}
```

#### 2.5 编译
编译成功之后请检查out/target/product/generic/system/lib文件夹下有没有SDK的so库

如果没有，单独编译一遍conscrypt模块，然后make snod即可
