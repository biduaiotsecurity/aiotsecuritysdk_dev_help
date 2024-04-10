# arpfirewall防火墙集成文档

# 目录
* [技术实现原理](#技术实现原理)
* [支持环境](#支持环境)
* [代码清单](#代码清单)
* [集成方式](#集成方式)
    * [android版本集成](#android版本集成)
        * [android4x与android5x版本集成方法](#android4x与android5x版本集成方法)
        * [android6x版本集成方法](#android6x版本集成方法)
        * [android7x版本集成方法](#android7x版本集成方法)
        * [android8x版本集成方法](#android8x版本集成方法)
        * [android9x版本集成方法](#android9x版本集成方法)
        * [android13x版本集成方法](#android13x版本集成方法)
        * [android14x版本集成方法](#android14x版本集成方法)
* [应用层启动方式](#应用层启动方式)
   * [android启动方式](#android启动方式)
   * [Linux启动方式](#Linux启动方式)
# 技术实现原理
## android技术原理
<div align=center><img src="https://github.com/baidutvsafe/baidutvsafe.github.io/blob/master/image/arpfirewall%E9%98%B2%E7%81%AB%E5%A2%99%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86.png"/></div>
<div align=center>arpfirewall技术实现原理图</div>

首先，通过System app反射调用android.os.SystemProperties发送启动命令到init进程；

其次，init进程通过对启动命令分析init.rc存在的service进而通过shell脚本启动arpfirewall；

最后，由arpfirewall通过对shell脚本参数分析进行开启、关闭以及查询arp防火墙状态，并且将结果回调到System app。

__备注：System app为系统应用。__

## Linux技术原理
   直接通过ELF文件的方式启动。

__备注：arpfirewall以root权限启动。__

# 支持环境
当前可以支持的环境：
* Android环境 [4.4 - 9.0以及13.0]
由于android4.3及以下版本由于内核SELinux模块缺失而不在此文档内容当中，需要适配android版本请联系我们。

* Linux环境
由于不用的Linux环境使用的交叉编译工具不一样，需要适配的Linux版本请提前告知交叉编译工具。

# 代码清单
* Android环境
    * arpfirewall_Android4.X版本对应Android4.X版本
    * arpfirewall_Android5.X版本对应Android5.X版本
    * arpfirewall_Android6.X版本对应Android6.X版本
    * arpfirewall_Android7.X版本对应Android7.X版本
    * arpfirewall_Android8.X版本对应Android8.X版本
    * arpfirewall_Android9.X版本对应Android9.X版本
    * arpfirewall_Android13.X版本对应Android13.X版本
* arpfirewallte文件夹用于集成SELinux模块
* arpfirewallbin文件夹用于集成init模块

* Linux环境
   * 单独存在一个ELF文件，可自行调用。

# 集成方式

必须先进行android版本集成的基础步骤后，再进行Androidx.x版本集成。 

## android版本集成
####  1 添加执行脚本exarpfirewall.sh和arpfirewall到/system/bin/目录
步骤一：在android源码目录下的/system/core/rootdir/下新建arpfirewall目录；

步骤二：将Android.mk、exarpfirewall.sh以及arpfirewall放到此目录system/core/rootdir/arpfirewall/下；

步骤三：在system/core/rootdir/arpfirewall/目录下，输入指令:"mm"进行模块编译，然后将此模块编译进入system.img中。

### Android4.x与Android5.x版本集成方法
#### 1 编写启动服务
打开/system/core/rootdir/init.rc文件，在文件末尾加入如下内容：
```diff
+ service startarpfirewall /system/bin/sh /system/bin/exarpfirewall.sh start
+     class core
+     disabled
+     oneshot 
    
+ service stoparpfirewall /system/bin/sh /system/bin/exarpfirewall.sh stop
+     class core
+     disabled
+     oneshot

+ service checkarpfirewall /system/bin/sh /system/bin/exarpfirewall.sh check
+     class core
+     disabled
+     oneshot
```

#### 2 编写启动服务
步骤一：将exarpfirewall.te和arpfirewall.te放到android源码目录下的/exrernal/sepolicy/下；

步骤二：打开android源码目录下的/exrernal/sepolicy/flie_contexts文件，在文件末尾加入如下内容：
```diff
+ /system/bin/exarpfirewall u:object_r:exarpfirewall_exec:s0

+ /system/bin/arpfirewall u:object_r:arpfirewall_exec:s0
```

步骤三：在/exrernal/sepolicy/目录下，输入指令:"mm"进行模块编译，然后将此模块编译进入boot.img中。

### Android6.X版本集成方法
#### 1 编写启动服务
打开/system/core/rootdir/init.rc文件，在文件末尾加入如下内容：
```diff
+ service startarpfirewall /system/bin/sh /system/bin/exarpfirewall.sh start
+     class core
+     disabled
+     oneshot
+     seclabel u:r:arpfirewall:s0
    
+ service stoparpfirewall /system/bin/sh /system/bin/exarpfirewall.sh stop
+     class core
+     disabled
+     oneshot
+     seclabel u:r:arpfirewall:s0

+ service checkarpfirewall /system/bin/sh /system/bin/exarpfirewall.sh check
+     class core
+     disabled
+     oneshot
+     seclabel u:r:arpfirewall:s0
```

### 2 编写selinux规则te文件
步骤一：将arpfirewall.te放到android源码目录下的/exrernal/sepolicy/下；

步骤二：打开android源码目录下的/exrernal/sepolicy/flie_contexts文件，在文件末尾加入如下内容：
```diff
+ /system/bin/arpfirewall u:object_r:arpfirewall_exec:s0
```

步骤三：打开android源码目录下的/exrernal/sepolicy/domain.te，修改如下内容：
```diff
neverallow {
    domain
    -appdomain
    -dumpstate
    -shell
    userdebug_or_eng(`-su')
    -system_server
    -zygote
+   -arpfirewall
} { file_type -system_file -exec_type }:file execute;

neverallow {
  domain
  -init # TODO: limit init to relabelfrom for files
  -zygote
  -installd
  -dex2oat
+ -arpfirewall
} dalvikcache_data_file:file no_w_file_perms;

neverallow {
  domain
  -init
  -installd
  -dex2oat
  -zygote
+ -arpfirewall
} dalvikcache_data_file:dir no_w_dir_perms;

neverallow { domain -init 
+ -system_app 
} default_prop:property_service set;
```

步骤四：在源码根目录下，输入指令:"makebootimage"进行模块编译，然后将此模块编译进入boot.img中。


### Android7.X版本集成方法
#### 1 编写启动服务
打开/system/core/rootdir/init.rc或者在被包含编译路径下的device/XXX/XXX/init.XXX.rc文件，在文件末尾加入如下内容：
```diff
+ service startarpfirewall /system/bin/sh /system/bin/exarpfirewall.sh start
+     class core
+     disabled
+     oneshot
+     seclabel u:r:arpfirewall:s0
    
+ service stoparpfirewall /system/bin/sh /system/bin/exarpfirewall.sh stop
+     class core
+     disabled
+     oneshot
+     seclabel u:r:arpfirewall:s0

+ service checkarpfirewall /system/bin/sh /system/bin/exarpfirewall.sh check
+     class core
+     disabled
+     oneshot
+     seclabel u:r:arpfirewall:s0
```

### 3 编写selinux规则te文件
步骤一：将arpfirewall.te放到android源码目录下的device/XXX/XXX/sepolicy下；

步骤二：打开android源码目录下的device/XXX/XXX/sepolicy/flie_contexts文件，在文件末尾加入如下内容：
```diff
+ /system/bin/arpfirewall u:object_r:arpfirewall_exec:s0
```

步骤三：打开android源码目录下的/system/sepolicy/domain.te，修改如下内容：
```diff
neverallow {
  domain
  -debuggerd
  -vold
  -dumpstate
  -system_server
+ -arpfirewall
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
+   -arpfirewall
} { file_type -system_file -exec_type -postinstall_file }:file execute;

neverallow {
  domain
  -init # TODO: limit init to relabelfrom for files
  -zygote
  -installd
  -postinstall_dexopt
  -cppreopts
  -dex2oat
  -otapreopt_slot
+ -arpfirewall
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
+ -arpfirewall
} dalvikcache_data_file:dir no_w_dir_perms;

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

### Android8.X版本集成方法
### 1 编写启动服务
打开/system/core/rootdir/init.rc文件，在文件末尾加入如下内容：
```diff
+ service startarpfirewall /system/bin/sh /system/bin/exarpfirewall.sh start
+    class core
+    disabled
+    oneshot
+    seclabel u:r:arpfirewall:s0
    
+ service stoparpfirewall /system/bin/sh /system/bin/exarpfirewall.sh stop
+    class core
+    disabled
+    oneshot
+    seclabel u:r:arpfirewall:s0

+ service checkarpfirewall /system/bin/sh /system/bin/exarpfirewall.sh check
+    class core
+    disabled
+    oneshot
+    seclabel u:r:arpfirewall:s0
```

### 2 编写selinux规则te文件
步骤一：将private/arpfirewall.te放到android源码目录下的/system/sepolicy/private/下；

步骤二：将public/arpfirewall.te放到android源码目录下的/system/sepolicy/public/下；

步骤三：将vendor/arpfirewall.te放到android源码目录下的/system/sepolicy/vendor/下；

步骤四：打开android源码目录下的/system/sepolicy/private/flie_contexts文件，在文件末尾加入如下内容：
```diff
+ /system/bin/arpfirewall u:object_r:arpfirewall_exec:s0
```

步骤五：打开android源码目录下的/system/sepolicy/private/domain.te，修改如下内容：
```diff
neverallow {
  domain
  -vold
  -dumpstate
  -storaged
  -system_server
+ -arpfirewall 
  userdebug_or_eng(`-perfprofd')
} self:capability sys_ptrace;
```

步骤六：打开android源码目录下的/system/sepolicy/private/app.te，修改如下内容：
```diff
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
neverallowxperm { domain -arpfirewall} domain:socket_class_set ioctl { 0 };

neverallow {
  domain
+ -arpfirewall 
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


### Android9.X版本集成方法
### 1 编写启动服务
打开/system/core/rootdir/init.rc文件，在文件末尾加入如下内容：
```diff
+ service startarpfirewall /system/bin/sh /system/bin/exarpfirewall.sh start
+    class core
+    disabled
+    oneshot
+    seclabel u:r:arpfirewall:s0
    
+ service stoparpfirewall /system/bin/sh /system/bin/exarpfirewall.sh stop
+    class core
+    disabled
+    oneshot
+    seclabel u:r:arpfirewall:s0

+ service checkarpfirewall /system/bin/sh /system/bin/exarpfirewall.sh check
+    class core
+    disabled
+    oneshot
+    seclabel u:r:arpfirewall:s0
```

### 2 编写selinux规则te文件
步骤一：将private/arpfirewall.te放到android源码目录下的/system/sepolicy/private/及/system/sepolicy/prebuilds/api/28.0/private/ 下(两个目录的放同样的文件)；

步骤二：将public/arpfirewall.te放到android源码目录下的/system/sepolicy/public/及/system/sepolicy/prebuilds/api/28.0/public/ 下(两个目录的放同样的文件)；

步骤三：打开android源码目录下的/system/sepolicy/private/flie_contexts与/system/sepolicy/prebuilts/api/28.0/private/file_contexts文件（两文件相同)，在文件末尾加入如下内容：

```diff
+ /system/bin/arpfirewall u:object_r:arpfirewall_exec:s0
```

步骤四：打开android源码目录下的/system/sepolicy/private/domain.te与/system/sepolicy/prebuilts/api/28.0/private/domain.te (两文件相同)，修改如下内容：

```diff
neverallow {
  domain
  -vold
  -dumpstate
  userdebug_or_eng(`-incidentd')
  -storaged
  -system_server
+ -arpfirewall
  userdebug_or_eng(`-perfprofd')
} self:global_capability_class_set sys_ptrace;
```

步骤五：打开android源码目录下的/system/sepolicy/public/domain.te与/system/sepolicy/prebuilts/api/28.0/public/domain.te (两文件相同)，修改如下内容：
```diff
neverallowxperm {domain 
+ -arpfirewall } 
domain:socket_class_set ioctl { 0 };

neverallowxperm {domain 
+ -arpfirewall } 
domain:socket_class_set ioctl { SIOCATMARK };

compatible_property_only(`
    neverallow { domain -init 
+    -system_app} 
    default_prop:property_service set;
    neverallow { domain -init } mmc_prop:property_service set;
    
    
full_treble_only(`
  neverallow {
    domain
    -coredomain
    -appdomain
+   -arpfirewall
    -binder_in_vendor_violators # TODO(b/35870313): Remove once all violations are gone
  } binder_device:chr_file rw_file_perms;
')

full_treble_only(`
  neverallow {
    domain
    -coredomain
    -appdomain # restrictions for vendor apps are declared lower down
    -binder_in_vendor_violators # TODO(b/35870313): Remove once all violations are gone
+   -arpfirewall
  } service_manager_type:service_manager find;
')

full_treble_only(`
  # Vendor apps are permited to use only stable public services. If they were to use arbitrary
  # services which can change any time framework/core is updated, breakage is likely.
  neverallow {
    appdomain
    -coredomain
  } {
    service_manager_type
    -app_api_service
    -ephemeral_app_api_service
    -audioserver_service # TODO(b/36783122) remove exemptions below once app_api_service is fixed
    -cameraserver_service
    -drmserver_service
    -keystore_service
    -mediadrmserver_service
    -mediaextractor_service
    -mediametrics_service
    -mediaserver_service
    -nfc_service
    -radio_service
    -virtual_touchpad_service
    -vr_hwc_service
    -vr_manager_service
+   -arpfirewall
  }:service_manager find;
')

full_treble_only(`
  neverallow {
    domain
    -coredomain
    -appdomain
    -binder_in_vendor_violators # TODO(b/35870313): Remove once all violations are gone
+   -arpfirewall
  } servicemanager:binder { call transfer };
')

full_treble_only(`
  neverallow_establish_socket_comms({
    domain
    -coredomain
    -netdomain
    -socket_between_core_and_vendor_violators
+   -arpfirewall
  }, netd);
')

full_treble_only(`
  neverallow {
    coredomain
    -appdomain # TODO(b/34980020) remove exemption for appdomain
    -data_between_core_and_vendor_violators
    -init
    -vold_prepare_subdirs
+   -arpfirewall
    } {
      data_file_type
      -core_data_file_type
      # TODO(b/72998741) Remove exemption. Further restricted in a subsequent
      # neverallow. Currently only getattr and search are allowed.
      -vendor_data_file
    }:dir *;

full_treble_only(`
  # vendor domains may only access files in /data/vendor, never core_data_file_types
  neverallow {
    domain
    -appdomain # TODO(b/34980020) remove exemption for appdomain
    -coredomain
    -data_between_core_and_vendor_violators # TODO(b/34980020) Remove once all violators have been cleaned up
    -vendor_init
+   -arpfirewall
  } {
    core_data_file_type
    # libc includes functions like mktime and localtime which attempt to access
    # files in /data/misc/zoneinfo/tzdata file. These functions are considered
    # vndk-stable and thus must be allowed for all processes.
    -zoneinfo_data_file
  }:file_class_set ~{ append getattr ioctl read write };

neverallow {vendor_init 
+ -arpfirewall} 
unencrypted_data_file:dir ~search;

full_treble_only(`
  # vendor domains may only access dirs in /data/vendor, never core_data_file_types
  neverallow {
    domain
    -appdomain # TODO(b/34980020) remove exemption for appdomain
    -coredomain
    -data_between_core_and_vendor_violators # TODO(b/34980020) Remove once all violators have been cleaned up
+   -arpfirewall
    } {
      system_data_file # default label for files on /data. Covered below
    }:dir ~{ getattr search };
')

full_treble_only(`
    # Do not allow vendor components to execute files from system
    # except for the ones whitelist here.
    neverallow {
        domain
        -coredomain
        -appdomain
        -vendor_executes_system_violators
        -vendor_init
+       -arpfirewall
    } {
        exec_type
        -vendor_file_type
        -crash_dump_exec
        -netutils_wrapper_exec
    }:file { entrypoint execute execute_no_trans };
')

neverallow {
  domain
  -system_server
  -system_app
  -init
  -installd # for relabelfrom and unlink, check for this in explicit neverallow
  -vold_prepare_subdirs # For unlink
+ -arpfirewall
  with_asan(`-asan_extract')
} system_data_file:file no_w_file_perms;

neverallow {
  domain
  -dnsmasq
  -dumpstate
  -init
  -installd
  -install_recovery
  -lmkd
  -netd
  -perfprofd
  -postinstall_dexopt
  -recovery
  -sdcardd
  -tee
  -ueventd
  -uncrypt
  -vendor_init
  -vold
  -vold_prepare_subdirs
  -zygote
+ -arpfirewall
} self:capability dac_override;

```

步骤六：打开android源码目录下的/system/sepolicy/public/app.te与 /system/sepolicyprebuilts/api/28.0/public/app.te(两文件相同)，修改如下内容：
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
-  system_app
  } {
  data_file_type
  -dalvikcache_data_file

```

步骤七：打开android源码目录下的/system/sepolicy/private/system_app.te与/system/sepolicy/prebuilts/api/28.0/private/system_app.te(两文件相同)，修改如下内容：

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

步骤八：打开android源码目录下的 /system/sepolicy/private/compat/26.0/26.0.ignore.cil和/system/sepolicy/prebuilts/api/28.0/private/compat/26.0/26.0.ignore.cil(两文件相同)，修改如下内容：
```diff 

	wpantund_exec
	wpantund_service
	wpantund_tmpfs

+	arpfirewall
+	arpfirewall_exec
+	arpfirewall_tmpfs
wm_trace_data_file))

```

步骤九：打开android源码目录下的 /system/sepolicy/private/compat/27.0/27.0.ignore.cil和/system/sepolicy/prebuilts/api/28.0/private/compat/27.0/27.0.ignore.cil(两文件相同)，修改如下内容：
```diff 

	wpantund
	wpantund_exec
	wpantund_service

+	arpfirewall
+	arpfirewall_exec
+	arpfirewall_tmpfs
	wpantund_tmpfs))

```

步骤十：打开android源码目录下的/system/sepolicy/tests/treble_sepolicy_tests.py，修改如下内容：
```diff 
def TestCoredomainViolations():
    global alldomains
    # verify that all domains launched from /system have the coredomain
    # attribute
    ret = ""
+   return ret   
```

步骤十一：在源码根目录下，输入指令:"m"进行编译得到所有相关文件。

### Android13.X版本集成方法
### 1 编写启动服务
打开/system/core/rootdir/init.rc文件，在文件末尾加入如下内容：
```diff
+ service startarpfirewall /system/bin/sh /system/bin/exarpfirewall.sh start
+    class core
+    disabled
+    oneshot
+    seclabel u:r:arpfirewall:s0
    
+ service stoparpfirewall /system/bin/sh /system/bin/exarpfirewall.sh stop
+    class core
+    disabled
+    oneshot
+    seclabel u:r:arpfirewall:s0

+ service checkarpfirewall /system/bin/sh /system/bin/exarpfirewall.sh check
+    class core
+    disabled
+    oneshot
+    seclabel u:r:arpfirewall:s0
```

### 2 编写selinux规则te文件
步骤一：将private/arpfirewall.te放到android源码目录下的/system/sepolicy/private/及/system/sepolicy/prebuilts/api/33.0/private/ 下(两个目录的放同样的文件)；

步骤二：将public/arpfirewall.te放到android源码目录下的/system/sepolicy/public/及/system/sepolicy/prebuilds/api/33.0/public/ 下(两个目录的放同样的文件)；

步骤三：打开android源码目录下的/system/sepolicy/private/flie_contexts与/system/sepolicy/prebuilts/api/33.0/private/file_contexts文件（两文件相同)，在文件末尾加入如下内容：

```diff
+ /system/bin/arpfirewall u:object_r:arpfirewall_exec:s0
```

步骤四：打开android源码目录下的/system/sepolicy/private/domain.te与/system/sepolicy/prebuilts/api/33.0/private/domain.te (两文件相同)，修改如下内容：
```diff
neverallow {
  domain
  -vold
  userdebug_or_eng(`-llkd')
  -dumpstate
+ -arpfirewall
  userdebug_or_eng(`-incidentd')
  userdebug_or_eng(`-profcollectd')
  userdebug_or_eng(`-simpleperf_boot')
  -storaged
  -system_server
} self:global_capability_class_set sys_ptrace;


- neverallow ~{dac_override_allowed} self:global_capability_class_set dac_override;
+ neverallow ~{dac_override_allowed arpfirewall} self:global_capability_class_set dac_override;
```

步骤五：打开android源码目录下的/system/sepolicy/public/domain.te与/system/sepolicy/prebuilts/api/33.0/public/domain.te (两文件相同)，修改如下内容：
```diff
- neverallowxperm *:{ dir notdevfile_class_set socket_class_set blk_file} ioctl { 0 };
+ neverallowxperm { -arpfirewall } *:{ dir notdevfile_class_set socket_class_set blk_file} ioctl { 0 };

- neverallowxperm { domain } domain:socket_class_set ioctl { SIOCATMARK };
+ neverallowxperm { domain -arpfirewall } domain:socket_class_set ioctl { SIOCATMARK };

full_treble_only(`
  # Vendor apps are permited to use only stable public services. If they were to use arbitrary
  # services which can change any time framework/core is updated, breakage is likely.
  #
  # Note, this same logic applies to untrusted apps, but neverallows for these are separate.
  neverallow {
    appdomain
    -coredomain
  } {
    service_manager_type

    -app_api_service
    -vendor_service # must be @VintfStability to be used by an app
    -ephemeral_app_api_service

    -apc_service
    -audioserver_service # TODO(b/36783122) remove exemptions below once app_api_service is fixed
    -cameraserver_service
    -drmserver_service
    -credstore_service
    -keystore_maintenance_service
    -keystore_service
    -legacykeystore_service
    -mediadrmserver_service
    -mediaextractor_service
    -mediametrics_service
    -mediaserver_service
    -nfc_service
    -radio_service
    -virtual_touchpad_service
    -vr_manager_service
+   -arpfirewall
    userdebug_or_eng(`-hal_face_service')
  }:service_manager find;
')

full_treble_only(`
  neverallow {
    coredomain
    -shell
    userdebug_or_eng(`-su')
+   -arpfirewall
    -ueventd # uevent is granted create for this device, but we still neverallow I/O below
  } vndbinder_device:chr_file rw_file_perms;
')

full_treble_only(`
  neverallow {
    coredomain
    -shell
+   -arpfirewall
    userdebug_or_eng(`-su')
  } vndservicemanager:binder *;
')

full_treble_only(`
  neverallow_establish_socket_comms({
    coredomain
    -init
    -adbd
+   -arpfirewall
  }, {
    domain
    -coredomain
    -socket_between_core_and_vendor_violators
  });
')

full_treble_only(`
  neverallow {
    coredomain
    -appdomain # TODO(b/34980020) remove exemption for appdomain
    -data_between_core_and_vendor_violators
    -init
    -vold_prepare_subdirs
+   -arpfirewall
    } {
      data_file_type
      -core_data_file_type
      -app_data_file_type
      # TODO(b/72998741) Remove exemption. Further restricted in a subsequent
      # neverallow. Currently only getattr and search are allowed.
      -vendor_data_file
    }:dir *;

')

  neverallow {
    domain
    -appdomain # TODO(b/34980020) remove exemption for appdomain
    -coredomain
    -data_between_core_and_vendor_violators # TODO(b/34980020) Remove once all violators have been cleaned up
    -vendor_init
+   -arpfirewall
  } {
    core_data_file_type
    # libc includes functions like mktime and localtime which attempt to access
    # files in /data/misc/zoneinfo/tzdata and /system/usr/share/zoneinfo/tzdata.
    # These functions are considered vndk-stable and thus must be allowed for
    # all processes.
    -zoneinfo_data_file
    with_native_coverage(`-method_trace_data_file')
  }:file_class_set ~{ append getattr ioctl read write map };
  
  neverallow {
    vendor_init
    -data_between_core_and_vendor_violators
+   -arpfirewall
  } {
    core_data_file_type
    -unencrypted_data_file
    -system_data_file
    -system_data_root_file
    -vendor_data_file
    -zoneinfo_data_file
    with_native_coverage(`-method_trace_data_file')
  }:dir *;
  
full_treble_only(`
  # vendor domains may only access dirs in /data/vendor, never core_data_file_types
  neverallow {
    domain
    -appdomain # TODO(b/34980020) remove exemption for appdomain
    -coredomain
    -data_between_core_and_vendor_violators # TODO(b/34980020) Remove once all violators have been cleaned up
+   -arpfirewall
    } {
      system_data_file # default label for files on /data. Covered below
    }:dir ~{ getattr search };
')

full_treble_only(`
    # Do not allow vendor components to execute files from system
    # except for the ones allowed here.
    neverallow {
        domain
        -coredomain
        -appdomain
        -vendor_executes_system_violators
        -vendor_init
+       -arpfirewall
    } {
        system_file_type
        -system_lib_file
        -system_linker_exec
        -crash_dump_exec
        -iorap_prefetcherd_exec
        -iorap_inode2filename_exec
        -netutils_wrapper_exec
        userdebug_or_eng(`-tcpdump_exec')
    }:file { entrypoint execute execute_no_trans };
')

    neverallow coredomain {
        file_type
+       -arpfirewall
        -system_file_type
        -postinstall_file
    }:file entrypoint;
    # Do not allow domains other than coredomain to access entrypoint
    # for anything but vendor_file_type and init_exec for vendor_init.
    neverallow { domain -coredomain + -arpfirewall} {
        file_type
        -vendor_file_type
        -init_exec
    }:file entrypoint;

full_treble_only(`
    neverallow {
      coredomain
      -shell
      -system_executes_vendor_violators
+     -arpfirewall
    } {
      vendor_file_type
      -same_process_hal_file
    }:file execute_no_trans;
')

full_treble_only(`
  # Do not allow vendor components access to /system files except for the
  # ones allowed here.
  neverallow {
    domain
    -appdomain
    -coredomain
    -vendor_executes_system_violators
    # vendor_init needs access to init_exec for domain transition. vendor_init
    # neverallows are covered in public/vendor_init.te
    -vendor_init
+   -arpfirewall
  } {
    system_file_type
    -crash_dump_exec
    -file_contexts_file
    -iorap_inode2filename_exec
    -netutils_wrapper_exec
    -property_contexts_file
    -system_event_log_tags_file
    -system_group_file
    -system_lib_file
    with_asan(`-system_asan_options_file')
    -system_linker_exec
    -system_linker_config_file
    -system_passwd_file
    -system_seccomp_policy_file
    -system_security_cacerts_file
    -system_zoneinfo_file
    -task_profiles_api_file
    -task_profiles_file
    userdebug_or_eng(`-tcpdump_exec')
  }:file *;
')

neverallow {
  domain
  -system_server
  -system_app
  -init
  -toolbox # TODO(b/141108496) We want to remove toolbox
  -installd # for relabelfrom and unlink, check for this in explicit neverallow
  -vold_prepare_subdirs # For unlink
+ -arpfirewall
  with_asan(`-asan_extract')
} system_data_file:file no_w_file_perms;
```

步骤六：打开android源码目录下的/system/sepolicy/public/app.te与 /system/sepolicyprebuilts/api/33.0/public/app.te(两文件相同)，修改如下内容：
```diff
neverallow appdomain drm_data_file:dir_file_class_set
    { create write setattr relabelfrom relabelto append unlink link rename };
-neverallow { appdomain -platform_app }
+neverallow { appdomain -platform_app -system_app}
    apk_data_file:dir_file_class_set
    { create write setattr relabelfrom relabelto append unlink link rename };
neverallow { appdomain -platform_app }
    apk_tmp_file:dir_file_class_set
```

步骤七：打开android源码目录下的/system/sepolicy/private/system_app.te与/system/sepolicy/prebuilts/api/33.0/private/system_app.te(两文件相同)，修改如下内容：
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

步骤八：打开android源码目录下的 /system/sepolicy/private/compat/32.0/32.0.ignore.cil和/system/sepolicy/prebuilts/api/32.0/private/compat/32.0/32.0.ignore.cil(两文件相同)，修改如下内容：
```diff 
    vendor_uuid_mapping_config_file
    vendor_vm_data_file
    vendor_vm_file
    virtual_device_service
    wallpaper_effects_generation_service
+   arpfirewall
+   arpfirewall_exec
+   arpfirewall_tmpfs
))
```

步骤九：打开android源码目录下的 /system/sepolicy/private/compat/31.0/31.0.ignore.cil，修改如下内容：
```diff 
    vendor_uuid_mapping_config_file
    vendor_vm_file
    vendor_vm_data_file
    virtual_device_service
+   arpfirewall
+   arpfirewall_exec
+   arpfirewall_tmpfs
  ))
```

步骤十：打开android源码目录下的/system/sepolicy/tests/treble_sepolicy_tests.py，修改如下内容：
```diff 
def TestCoredomainViolations():
    global alldomains
    # verify that all domains launched from /system have the coredomain
    # attribute
    ret = ""
+   return ret   
```

步骤十一：打开android源码目录下的/system/sepolicy/private/app.te与/system/sepolicy/prebuilts/api/33.0/private/app.te(两文件相同)，修改如下内容：
```diff 
neverallow {
  bluetooth
  isolated_app
  nfc
  radio
  shared_relro
  sdk_sandbox
+ -system_app
} {
  data_file_type
  -apex_art_data_file
  -dalvikcache_data_file
  -system_data_file # shared libs in apks
  -apk_data_file
}:file no_x_file_perms;
```

步骤十二：打开android源码目录下的/system/sepolicy/private/property.te与/system/sepolicy/prebuilts/api/33.0/private/property.te(两文件相同)，修改如下内容：
```diff 
system/sepolicy/private/property.te
neverallow {
  domain
  -init
+ -system_app
} default_prop:property_service set;

neverallow {
    domain
    -coredomain
    -appdomain
    -vendor_init
+   -arpfirewall
  } {
    core_property_type
    dalvik_config_prop
    extended_core_property_type
    exported3_system_prop
    systemsound_config_prop
    -debug_prop
    -logd_prop
    -nfc_prop
    -powerctl_prop
    -radio_prop
  }:file no_rw_file_perms;

- neverallow { coredomain -init -dumpstate } {
+ neverallow { coredomain -init -dumpstate -arpfirewall} {
  vendor_property_type
  vendor_internal_property_type
  -vendor_restricted_property_type
  -vendor_public_property_type
}:file no_rw_file_perms;

- neverallow { domain -coredomain } {
+ neverallow { domain -coredomain -arpfirewall} {
  system_property_type
  system_internal_property_type
  -system_restricted_property_type
  -system_public_property_type
}:file no_rw_file_perms;

```

步骤十三：打开android源码目录下的/system/sepolicy/microdroid/system/public/te_macros与/system/sepolicy/prebuilts/api/33.0/public/te_macros(两文件相同)，修改如下内容：
```diff 
define(`system_internal_prop', `
  define_prop($1, system, internal)
  treble_sysprop_neverallow(`
-    neverallow { domain -coredomain } $1:file no_rw_file_perms;
+    neverallow { domain -coredomain -arpfirewall} $1:file no_rw_file_perms;
  ')
')
```

步骤十四：在源码根目录下，输入指令:"m"进行编译得到所有相关文件。

### Android14.X版本集成方法
### 1 编写启动服务
打开/system/core/rootdir/init.rc文件，在文件末尾加入如下内容：
```diff
+ service startarpfirewall /system/bin/sh /system/bin/exarpfirewall.sh start
+    class core
+    disabled
+    oneshot
+    seclabel u:r:arpfirewall:s0
    
+ service stoparpfirewall /system/bin/sh /system/bin/exarpfirewall.sh stop
+    class core
+    disabled
+    oneshot
+    seclabel u:r:arpfirewall:s0

+ service checkarpfirewall /system/bin/sh /system/bin/exarpfirewall.sh check
+    class core
+    disabled
+    oneshot
+    seclabel u:r:arpfirewall:s0
```

### 2 编写selinux规则te文件
步骤一：将private/arpfirewall.te放到android源码目录下的/system/sepolicy/private/及/system/sepolicy/prebuilts/api/34.0/private/ 下(两个目录的放同样的文件);
步骤二：将public/arpfirewall.te放到android源码目录下的/system/sepolicy/public/及/system/sepolicy/prebuilds/api/34.0/public/ 下(两个目录的放同样的文件);
步骤三：打开android源码目录下的/system/sepolicy/private/flie_contexts与/system/sepolicy/prebuilts/api/34.0/private/file_contexts文件（两文件相同)，在文件末尾加入如下内容:
```diff
+ /system/bin/arpfirewall u:object_r:arpfirewall_exec:s0
```
步骤四：打开android源码目录下的/system/sepolicy/private/domain.te与/system/sepolicy/prebuilts/api/34.0/private/domain.te (两文件相同)，修改如下内容:
```diff
neverallow {
  domain
  -vold
  userdebug_or_eng(`-llkd')
  -dumpstate
+ -arpfirewall
  userdebug_or_eng(`-incidentd')
  userdebug_or_eng(`-profcollectd')
  userdebug_or_eng(`-simpleperf_boot')
  -storaged
  -system_server
} self:global_capability_class_set sys_ptrace;

- neverallow ~dac_override_allowed self:global_capability_class_set dac_override;
+ neverallow ~{dac_override_allowed arpfirewall} self:global_capability_class_set dac_override;
```

# 应用层启动方式
## android启动方式
* 若以app方式集成，app检测到arp劫持风险后将自动启动arpfirewall防火墙功能。
* 若以SDK方式集成，请查阅链接中的调用与回调方法：https://github.com/baidutvsafe/baidutvsafe.github.io/blob/master/index.md
    * 关键性接口与回调：
    ```
    public static boolean startArpFirewall()

    public static boolean stopArpFirewall()

    public static boolean checkArpFirewall()

    public void onArpFirewallResult(String behaviour, boolean suc)
    ```
## Linux启动方式
   * 启动
   arpfirewall -o start
   * 停止
   arpfirewall -o stop
   
