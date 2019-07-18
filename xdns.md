# xdns反DNS劫持方案集成文档

# 目录
* [技术实现原理](#技术实现原理)
* [支持环境](#支持环境)
* [代码清单](#代码清单)
* [集成方式](#集成方式)
    * [android4x版本集成方法](#android4x版本集成方法)
    * [android5x版本集成方法](#android5x版本集成方法)
    * [android6x版本集成方法](#android6x版本集成方法)
    * [android7x版本集成方法](#android7x版本集成方法)
    * [android8x版本集成方法](#android8x版本集成方法)
    * [android9x版本集成方法](#android9x版本集成方法)
* [应用层启动方式](#应用层启动方式)
# 技术实现原理

<div align=center><img src="https://github.com/baidutvsafe/baidutvsafe.github.io/blob/master/image/xdns%E7%9A%84%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86.png"/></div>
<div align=center>xdns技术实现原理图</div>

首先，通过System app反射调用android.os.SystemProperties发送启动命令到init进程；

其次，init进程通过对启动命令分析init.rc存在的service进而通过shell脚本启动xdns；

最后，由xdns通过对shell脚本参数分析进行开启、关闭以及查询xdnsproxy状态，并且将结果回调到System app。

__备注：System app为系统应用。__

# 支持环境
当前可以支持的环境：
* Android环境 [4.4 - 9.0]
由于android4.3及以下版本由于内核SELinux模块缺失而不在此文档内容当中，需要适配android版本请联系我们。

# 代码清单
* Android环境
    * xdns_Android4.X版本对应Android4.X版本
    * xdns_Android5.X版本对应Android5.X版本
    * xdns_Android6.X版本对应Android6.X版本
    * xdns_Android7.X版本对应Android7.X版本
    * xdns_Android8.X版本对应Android8.X版本
    * xdns_Android9.X版本对应Android9.X版本
* xdnste文件夹用于集成SELinux模块
* xdnsbin文件夹用于集成init模块

# 集成方式

## Android4.X版本集成方法
### 1 添加执行脚本exxdnsproxy.sh、xdns以及xdnsproxy到/system/bin/目录
步骤一：在android源码目录下的/system/core/rootdir/下新建xdns目录；

步骤二：将Android.mk、exxdnshproxy.sh、xdns以及xdnsproxy放到此目录system/core/rootdir/xdns/下；

步骤三：在system/core/rootdir/xdns/目录下，输入指令:"mm"进行模块编译，然后将此模块编译进入system.img中。

### 2 编写启动服务
打开/system/core/rootdir/init.rc文件，在文件末尾加入如下内容：
```diff
+ service startxdns /system/bin/sh /system/bin/exxdnsproxy.sh start
+     class core
+     disabled
+     oneshot 
    
+ service stopxdns /system/bin/sh /system/bin/exxdnsproxy.sh stop
+     class core
+     disabled
+     oneshot

+ service checkxdns /system/bin/sh /system/bin/exxdnsproxy.sh check
+     class core
+     disabled
+     oneshot
```

### 3 编写selinux规则te文件
步骤一：将exxdnsproxy.te、xdns.te以及xdnsproxy.te放到android源码目录下的/exrernal/sepolicy/下；

步骤二：打开android源码目录下的/exrernal/sepolicy/flie_contexts文件，在文件末尾加入如下内容：
```diff
+ /system/bin/exxdnsproxy u:object_r:exxdnsproxy_exec:s0

+ /system/bin/xdnsproxy u:object_r:xdnsproxy_exec:s0

+ /system/bin/xdns u:object_r:xdns_exec:s0
```

步骤三：在/exrernal/sepolicy/目录下，输入指令:"mm"进行模块编译，然后将此模块编译进入boot.img中。

## Android5.X版本集成方法
### 1 添加执行脚本exxdnsproxy.sh、xdns以及xdnsproxy到/system/bin/目录
步骤一：在android源码目录下的/system/core/rootdir/下新建xdns目录；

步骤二：将Android.mk、exxdnshproxy.sh、xdns以及xdnsproxy放到此目录system/core/rootdir/xdns/下；

步骤三：在system/core/rootdir/xdns/目录下，输入指令:"mm"进行模块编译，然后将此模块编译进入system.img中。

### 2 编写启动服务
打开/system/core/rootdir/init.rc文件，在文件末尾加入如下内容：
```diff
+ service startxdns /system/bin/sh /system/bin/exxdnsproxy.sh start
+     class core
+     disabled
+     oneshot
    
+ service stopxdns /system/bin/sh /system/bin/exxdnsproxy.sh stop
+     class core
+     disabled
+     oneshot

+ service checkxdns /system/bin/sh /system/bin/exxdnsproxy.sh check
+     class core
+     disabled
+     oneshot
```

### 3 编写selinux规则te文件
步骤一：将exxdnsproxy.te、xdns.te以及xdnsproxy.te放到android源码目录下的/exrernal/sepolicy/下；

步骤二：打开android源码目录下的/exrernal/sepolicy/flie_contexts文件，在文件末尾加入如下内容：
```diff
+ /system/bin/exxdnsproxy u:object_r:exxdnsproxy_exec:s0

+ /system/bin/xdnsproxy u:object_r:xdnsproxy_exec:s0

+ /system/bin/xdns u:object_r:xdns_exec:s0
```

步骤三：打开android源码目录下的/exrernal/sepolicy/system_app.te，在文件末尾加入如下内容：
```diff
+ allow system_app ctl_default_prop:property_service{set};
```
步骤四：在/exrernal/sepolicy/目录下，输入指令:"mm"进行模块编译，然后将此模块编译进入boot.img中。

## Android6.X版本集成方法
### 1 添加执行脚本exxdnsproxy.sh、xdns以及xdnsproxy到/system/bin/目录
步骤一：在android源码目录下的/system/core/rootdir/下新建xdns目录；

步骤二：将Android.mk、exxdnshproxy.sh、xdns以及xdnsproxy放到此目录system/core/rootdir/xdns/下；

步骤三：在system/core/rootdir/xdns/目录下，输入指令:"mm"进行模块编译，然后将此模块编译进入system.img中。

### 2 编写启动服务
打开/system/core/rootdir/init.rc文件，在文件末尾加入如下内容：
```diff
+ service startxdns /system/bin/sh /system/bin/exxdnsproxy.sh start
+     class core
+     disabled
+     oneshot
+     seclabel u:r:xdns:s0
    
+ service stopxdns /system/bin/sh /system/bin/exxdnsproxy.sh stop
+     class core
+     disabled
+     oneshot
+     seclabel u:r:xdns:s0

+ service checkxdns /system/bin/sh /system/bin/exxdnsproxy.sh check
+     class core
+     disabled
+     oneshot
+     seclabel u:r:xdns:s0
```

### 3 编写selinux规则te文件
步骤一：将xdns.te以及xdnsproxy.te放到android源码目录下的/exrernal/sepolicy/下；

步骤二：打开android源码目录下的/exrernal/sepolicy/flie_contexts文件，在文件末尾加入如下内容：
```diff
+ /system/bin/xdnsproxy u:object_r:xdnsproxy_exec:s0

+ /system/bin/xdns u:object_r:xdns_exec:s0
```

步骤三：打开android源码目录下的/exrernal/sepolicy/domain.te，修改如下内容：
```diff
neverallow {
  domain
  -debuggerd
  -vold
  -dumpstate
  -system_server
  userdebug_or_eng(`-procrank')
  userdebug_or_eng(`-perfprofd')
+ -xdns 
} self:capability sys_ptrace;

neverallow {
  domain
  -appdomain
  -dumpstate
  -shell
  userdebug_or_eng(`-su')
  -system_server
  -zygote
+ -xdns 
} { file_type -system_file -exec_type }:file execute;

neverallow {
  domain
  -init # TODO: limit init to relabelfrom for files
  -zygote
  -installd
  -dex2oat
+ -xdns 
} dalvikcache_data_file:file no_w_file_perms;

neverallow {
  domain
  -init
  -installd
  -dex2oat
  -zygote
+ -xdns 
} dalvikcache_data_file:dir no_w_dir_perms;

neverallow {
  domain
  -system_server
  -system_app
  -init
+ -xdns 
  -installd # for relabelfrom and unlink, check for this in explicit neverallow
} system_data_file:file no_w_file_perms;

neverallow { domain -init 
+ -system_app 
} default_prop:property_service set;

```

步骤四：在源码根目录下，输入指令:"makebootimage"进行模块编译，然后将此模块编译进入boot.img中。

## Android7.X版本集成方法
### 1 添加执行脚本exxdnsproxy.sh、xdns以及xdnsproxy到/system/bin/目录
步骤一：在android源码目录下的/system/core/rootdir/下新建xdns目录；

步骤二：将Android.mk、exxdnshproxy.sh、xdns以及xdnsproxy放到此目录system/core/rootdir/xdns/下；

步骤三：在system/core/rootdir/xdns/目录下，输入指令:"mm"进行模块编译，然后将此模块编译进入system.img中。

### 2 编写启动服务
打开/system/core/rootdir/init.rc或者在被包含编译路径下的device/XXX/XXX/init.XXX.rc文件，在文件末尾加入如下内容：
```diff
+ service startxdns /system/bin/sh /system/bin/exxdnsproxy.sh start
+     class core
+     disabled
+     oneshot
+     seclabel u:r:xdns:s0
    
+ service stopxdns /system/bin/sh /system/bin/exxdnsproxy.sh stop
+     class core
+     disabled
+     oneshot
+     seclabel u:r:xdns:s0

+ service checkxdns /system/bin/sh /system/bin/exxdnsproxy.sh check
+     class core
+     disabled
+     oneshot
+     seclabel u:r:xdns:s0
```

### 3 编写selinux规则te文件
步骤一：将xdns.te以及xdnsproxy.te放到android源码目录下的device/XXX/XXX/sepolicy下；

步骤二：打开android源码目录下的device/XXX/XXX/sepolicy/flie_contexts文件，在文件末尾加入如下内容：
```diff
+ /system/bin/xdnsproxy u:object_r:xdnsproxy_exec:s0

+ /system/bin/xdns u:object_r:xdns_exec:s0
```

步骤三：打开android源码目录下的/system/sepolicy/domain.te，修改如下内容：
```diff
neverallow {
  domain
  -debuggerd
  -vold
  -dumpstate
  -system_server
+  -xdns 
  userdebug_or_eng(`-perfprofd')
} self:capability sys_ptrace;

neverallow {
  domain
  -appdomain
  -autoplay_app
  -dumpstate
  -shell
  userdebug_or_eng(`-su')
  -system_server
  -zygote
+  -xdns 
} { file_type -system_file -exec_type -postinstall_file }:file execute

neverallow {
  domain
  -init # TODO: limit init to relabelfrom for files
  -zygote
  -installd
  -postinstall_dexopt
  -cppreopts
  -dex2oat
  -otapreopt_slot
+ -xdns 
} dalvikcache_data_file:file no_w_file_perms;

neverallow {
  domain
  -init
  -installd
  -postinstall_dexopt
  -cppreopts
  -dex2oat
  -zygote
  -otapreopt_slot
+ -xdns 
} dalvikcache_data_file:dir no_w_dir_perms;

neverallow {
  domain
  -system_server
  -system_app
  -init
  -installd # for relabelfrom and unlink, check for this in explicit neverallow
+ -xdns 
} system_data_file:file no_w_file_perms;

neverallow { domain -init 
+ -system_app 
} default_prop:property_service set;
```

步骤四：打开android源码目录下的/system/sepolicy/app.te，修改如下内容：
```diff
# Blacklist app domains not allowed to execute from /data
neverallow {
  bluetooth
  isolated_app
  nfc
  radio
  shared_relro
- system_app 
} {
  data_file_type
  -dalvikcache_data_file
  -system_data_file # shared libs in apks
  -apk_data_file
}:file no_x_file_perms;
```

步骤五：在源码根目录下，输入指令:"makebootimage"进行模块编译，然后将此模块编译进入boot.img中。

## Android8.X版本集成方法
### 1 添加执行脚本exxdnsproxy.sh、xdns以及xdnsproxy到/system/bin/目录
步骤一：在android源码目录下的/system/core/rootdir/下新建xdns目录；

步骤二：将Android.mk、exxdnshproxy.sh、xdns以及xdnsproxy放到此目录system/core/rootdir/xdns/下；

步骤三：在system/core/rootdir/xdns/目录下，输入指令:"mm"进行模块编译，然后将此模块编译进入system.img中。

### 2 编写启动服务
打开/system/core/rootdir/init.rc文件，在文件末尾加入如下内容：
```diff
+ service startxdns /system/bin/sh /system/bin/exxdnsproxy.sh start
+    class core
+    disabled
+    oneshot
+    seclabel u:r:xdns:s0
    
+ service stopxdns /system/bin/sh /system/bin/exxdnsproxy.sh stop
+    class core
+    disabled
+    oneshot
+    seclabel u:r:xdns:s0

+ service checkxdns /system/bin/sh /system/bin/exxdnsproxy.sh check
+    class core
+    disabled
+    oneshot
+    seclabel u:r:xdns:s0
```

### 3 编写selinux规则te文件
步骤一：将private/xdns.te以及private/xdnsproxy.te放到android源码目录下的/system/sepolicy/private/下；

步骤二：将public/xdns.te放到android源码目录下的/system/sepolicy/public/下；

步骤三：将vendor/xdns.te放到android源码目录下的/system/sepolicy/vendor/下；

步骤四：打开android源码目录下的/system/sepolicy/private/flie_contexts文件，在文件末尾加入如下内容：
```diff
+ /system/bin/xdnsproxy u:object_r:xdnsproxy_exec:s0

+ /system/bin/xdns u:object_r:xdns_exec:s0
```

步骤五：打开android源码目录下的/system/sepolicy/private/domain.te，修改如下内容：
```diff
# with other UIDs to these whitelisted domains.
neverallow {
  domain
  -vold
  -dumpstate
  -storaged
  -system_server
  userdebug_or_eng(`-perfprofd')
+ -xdns 
} self:capability sys_ptrace;
```

步骤六：打开android源码目录下的/system/sepolicy/private/app.te，修改如下内容：
```diff
# Blacklist app domains not allowed to execute from /data
neverallow {
  bluetooth
  isolated_app
  nfc
  radio
  shared_relro
+ -system_app 
} {
  data_file_type
  -dalvikcache_data_file
  -system_data_file # shared libs in apks
  -apk_data_file
}:file no_x_file_perms;

neverallow { appdomain -platform_app 
+ -system_app }
    apk_data_file:dir_file_class_set
    { create write setattr relabelfrom relabelto append unlink link rename };
```

步骤七：打开android源码目录下的/system/sepolicy/public/domain.te，修改如下内容：
```diff 
neverallow {
  domain
  -adbd
  -dumpstate
  -hal_drm
  -init
  -mediadrmserver
  -recovery
  -shell
  -system_server
+ -xdns 
} serialno_prop:file r_file_perms;

neverallow {
  domain
+ -xdns 
  -system_server
  -system_app
  -init
  -installd # for relabelfrom and unlink, check for this in explicit neverallow
  with_asan(`-asan_extract')
} system_data_file:file no_w_file_perms;

neverallow { domain -init 
+ -system_app 
} default_prop:property_service set;
```

步骤八：在源码根目录下，输入指令:"make -j4"进行模块编译，然后将此模块编译进入boot.img/system.img/vendor.img中。

## Android9.X版本集成方法

### 1 添加执行脚本exxdnsproxy.sh、xdns以及xdnsproxy到/system/bin/目录

步骤一：在android源码目录下的/system/core/rootdir/下新建xdns目录；

步骤二：将Android.mk、exxdnshproxy.sh、xdns以及xdnsproxy放到此目录system/core/rootdir/xdns/下；

步骤三：在system/core/rootdir/xdns/目录下，输入指令:"mm"进行模块编译，然后将此模块编译进入system.img中。

### 2 编写启动服务

打开/system/core/rootdir/init.rc文件，在文件末尾加入如下内容：

```diff
+ service startxdns /system/bin/sh /system/bin/exxdnsproxy.sh start
+    class core
+    disabled
+    oneshot
+    seclabel u:r:xdns:s0
    
+ service stopxdns /system/bin/sh /system/bin/exxdnsproxy.sh stop
+    class core
+    disabled
+    oneshot
+    seclabel u:r:xdns:s0

+ service checkxdns /system/bin/sh /system/bin/exxdnsproxy.sh check
+    class core
+    disabled
+    oneshot
+    seclabel u:r:xdns:s0
```



### 3 编写selinux规则te文件

步骤一：将private/xdns.te以及private/xdnsproxy.te放到android源码目录下的/system/sepolicy/private/及/system/sepolicy/prebuilds/api/28.0/private下(两个目录的放同样的文件)；

步骤二：将public/xdns.te以及public/xdnsproxy.te放到android源码目录下的/system/sepolicy/public/及/system/sepolicy/prebuilds/api/28.0/private下(两个目录的放同样的文件)；

步骤三：将vendor/xdns.te放到android源码目录下的/system/sepolicy/vendor/下；

步骤四：打开android源码目录下的/system/sepolicy/private/flie_contexts与/system/sepolicy/prebuilts/api/28.0/private/file_contexts文件（两文件相同)，在文件末尾加入如下内容：

```diff

+ /system/bin/xdnsproxy u:object_r:xdns_exec:s0
+ /system/bin/xdns u:object_r:xdns_exec:s0

```

步骤五：打开android源码目录下的/system/sepolicy/private/domain.te与/system/sepolicy/prebuilts/api/28.0/private/domain.te (两文件相同)，修改如下内容：

```diff

   userdebug_or_eng(`-incidentd')
   -storaged
   -system_server
+ -xdns
   userdebug_or_eng(`-perfprofd')

 } self:global_capability_class_set sys_ptrace;





     -init
     -ueventd
     -vold
+    -xdns
} sysfs:file no_rw_file_perms;

```

步骤六：打开android源码目录下的/system/sepolicy/public/domain.te与/system/sepolicy/prebuilts/api/28.0/public/domain.te (两文件相同)，修改如下内容：

```diff

-neverallow * { file_type -exec_type -postinstall_file }:file entrypoint;
+neverallow { domain -xdns } { file_type -exec_type -postinstall_file }:file entrypoint;





 neverallow { domain -init } property_data_file:dir no_w_dir_perms;
 neverallow { domain -init } property_data_file:file { no_w_file_perms no_x_file_perms };
-neverallow { domain -init } property_type:file { no_w_file_perms no_x_file_perms };
+neverallow { domain -init -xdns} property_type:file { no_w_file_perms no_x_file_perms };
 neverallow { domain -init } properties_device:file { no_w_file_perms no_x_file_perms };
 neverallow { domain -init } properties_serial:file { no_w_file_perms no_x_file_perms };



-neverallow { domain -init -vendor_init } default_prop:property_service set;
+neverallow { domain -init -vendor_init -system_app -xdns} default_prop:property_service set;
 neverallow { domain -init -vendor_init } mmc_prop:property_service set;



   -shell
   -system_server
   -vendor_init

+ -xdns
} serialno_prop:file r_file_perms;



 -appdomain
         -vendor_executes_system_violators
         -vendor_init

+ -xdns

+ -xdnsproxy
} {
    exec_type
    -vendor_file_type





   -init
   -installd # for relabelfrom and unlink, check for this in explicit neverallow
   -vold_prepare_subdirs # For unlink

+  -xdns
with_asan(`-asan_extract')
 } system_data_file:file no_w_file_perms;





   -vold
   -vold_prepare_subdirs
   -zygote

+  -xdns
} self:capability dac_override;
 neverallow { domain -traced_probes } self:capability dac_read_search;


```


步骤七：打开android源码目录下的/system/sepolicy/public/app.te与 /system/sepolicyprebuilts/api/28.0/public/app.te(两文件相同)，修改如下内容：

```diff

neverallow appdomain drm_data_file:dir_file_class_set
     { create write setattr relabelfrom relabelto append unlink link rename };
-neverallow { appdomain -platform_app }
+neverallow { appdomain -platform_app -system_app}
     apk_data_file:dir_file_class_set
     { create write setattr relabelfrom relabelto append unlink link rename };
 neverallow { appdomain -platform_app }



   nfc
   radio
   shared_relro

- system_app
  } {
  data_file_type
  -dalvikcache_data_file

```



步骤八：打开android源码目录下的/system/sepolicy/private/system_app.te与/system/sepolicy/prebuilts/api/28.0/private/system_app.te(两文件相同)，修改如下内容：

```diff 

net_domain(system_app)
 binder_service(system_app)

+allow system_app system_app_data_file:file execute;
+allow system_app apk_data_file:dir write;
+allow system_app sysfs_net:file { read getattr open };
+allow system_app apk_data_file:dir { remove_name add_name };
+allow system_app apk_data_file:file { write create unlink };
+allow system_app selinuxfs:file read;
+allow system_app sysfs_net:dir search;
+allow system_app default_prop:property_service set;



```





步骤九：打开android源码目录下的 /system/sepolicy/private/compat/26.0/26.0.ignore.cil和/system/sepolicy/prebuilts/api/28.0/private/compat/26.0/26.0.ignore.cil(两文件相同)，修改如下内容：

```diff 

	wpantund_exec
	wpantund_service
	wpantund_tmpfs

+	xdns
+	xdns_exec
+	xdns_tmpfs
+	xdnsproxy
+	xdnsproxy_exec
+	xdnsproxy_tmpfs
wm_trace_data_file))

```



步骤十：打开android源码目录下的 /system/sepolicy/private/compat/27.0/27.0.ignore.cil和/system/sepolicy/prebuilts/api/28.0/private/compat/27.0/27.0.ignore.cil(两文件相同)，修改如下内容：

```diff 

​	wpantund
​	wpantund_exec
​	wpantund_service

+	xdns
+	xdns_exec
+	xdns_tmpfs
+	xdnsproxy
+	xdnsproxy_exec
+	xdnsproxy_tmpfs
	wpantund_tmpfs))

```

步骤十一：打开android源码目录下的/system/sepolicy/tests/treble_sepolicy_tests.py，修改如下内容：

```diff 

ret += "\"coredomain\" attribute because they are executed off of "
         ret += "/system:\n"
         ret += " ".join(str(x) for x in sorted(violators)) + "\n"

+ return ret

#verify that all domains launched form /vendor do not have the coredomain
#attribute
violators = []

```



步骤十二：在源码根目录下，输入指令:"m"进行编译得到所有相关文件。


# 应用层启动方式
* 若以app方式集成，app检测到dns风险后将自动启动xdns反dns劫持功能。
* 若以SDK方式集成，请查阅链接中的调用与回调方法：https://github.com/baidutvsafe/baidutvsafe.github.io/blob/master/index.md
    * 关键性接口与回调：
    ```
    public static boolean fixDns()

    public static boolean stopDns()

    public static boolean checkDnsStatus()

    public void onDnsBehaviourResult(String behaviour, boolean suc)
    ```
