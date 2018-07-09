# xdns接入文档

# 目录
* [技术实现原理](#技术实现原理)
* [Android5.X版本集成方法](#Android5.X版本集成方法)
* [Android6.X版本集成方法](#Android6.X版本集成方法)
* [Android7.X版本集成方法](#Android7.X版本集成方法)



## 技术实现原理

<div align=center><img src="https://github.com/baidutvsafe/baidutvsafe.github.io/blob/master/image/xdns%E7%9A%84%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86.png"/></div>
<div align=center>xdns技术实现原理图</div>

首先，通过System app反射调用android.os.SystemProperties发送启动命令到init进程；

其次，init进程通过对启动命令分析init.rc存在的service进而通过shell脚本启动xdns；

最后，由xdns通过对shell脚本参数分析进行开启、关闭以及查询xdnsproxy状态，并且将结果回调到System app。

备注：System app为系统应用。

## Android5.X版本集成方法
### 1 添加执行脚本exxdnsproxy.sh、xdns以及xdnsproxy到/system/bin/目录
步骤一：在android源码目录下的/system/core/rootdir/下新建xdns目录；

步骤二：将Android.mk、exxdnshproxy.sh、xdns以及xdnsproxy放到此目录system/core/rootdir/xdns/下；

步骤三：在system/core/rootdir/xdns/目录下，输入指令:"mm"进行模块编译，然后将此模块编译进入system.img中。

### 2 编写启动服务
打开/system/core/rootdir/init.rc文件，在文件末尾加入如下内容：
```
service startxdns /system/bin/sh /system/bin/exxdnsproxy.sh start
    class core
    disabled
    oneshot
    
 service stopxdns /system/bin/sh /system/bin/exxdnsproxy.sh stop
    class core
    disabled
    oneshot

 service checkxdns /system/bin/sh /system/bin/exxdnsproxy.sh check
    class core
    disabled
    oneshot
```

### 3 编写selinux规则te文件
步骤一：将exxdnsproxy.te、xdns.te以及xdnsproxy.te放到android源码目录下的/exrernal/sepolicy/下；

步骤二：打开android源码目录下的/exrernal/sepolicy/flie_contexts文件，在文件末尾加入如下内容：
```
/system/bin/exxdnsproxy u:object_r:exxdnsproxy_exec:s0

/system/bin/xdnsproxy u:object_r:xdnsproxy_exec:s0

/system/bin/xdns u:object_r:xdns_exec:s0
```

步骤三：打开android源码目录下的/exrernal/sepolicy/system_app.te，在文件末尾加入如下内容：
```
allow system_app ctl_default_prop:property_service{set};
```
步骤四：在/exrernal/sepolicy/目录下，输入指令:"mm"进行模块编译，然后将此模块编译进入boot.img中。

### 4 如何启动、关闭以及查询xdns
请查阅链接中的调用方法：https://github.com/baidutvsafe/baidutvsafe.github.io/blob/master/index.md

## Android6.X版本集成方法
### 1 添加执行脚本exxdnsproxy.sh、xdns以及xdnsproxy到/system/bin/目录
步骤一：在android源码目录下的/system/core/rootdir/下新建xdns目录；

步骤二：将Android.mk、exxdnshproxy.sh、xdns以及xdnsproxy放到此目录system/core/rootdir/xdns/下；

步骤三：在system/core/rootdir/xdns/目录下，输入指令:"mm"进行模块编译，然后将此模块编译进入system.img中。

### 2 编写启动服务
打开/system/core/rootdir/init.rc文件，在文件末尾加入如下内容：
```
service startxdns /system/bin/sh /system/bin/exxdnsproxy.sh start
    class core
    disabled
    oneshot
    seclabel u:r:xdns:s0
    
 service stopxdns /system/bin/sh /system/bin/exxdnsproxy.sh stop
    class core
    disabled
    oneshot
    seclabel u:r:xdns:s0

 service checkxdns /system/bin/sh /system/bin/exxdnsproxy.sh check
    class core
    disabled
    oneshot
    seclabel u:r:xdns:s0
```

### 3 编写selinux规则te文件
步骤一：将xdns.te以及xdnsproxy.te放到android源码目录下的/exrernal/sepolicy/下；

步骤二：打开android源码目录下的/exrernal/sepolicy/flie_contexts文件，在文件末尾加入如下内容：
```
/system/bin/xdnsproxy u:object_r:xdnsproxy_exec:s0

/system/bin/xdns u:object_r:xdns_exec:s0
```

步骤三：打开android源码目录下的/exrernal/sepolicy/domain.te，修改如下内容：
```
neverallow {
  domain
  -debuggerd
  -vold
  -dumpstate
  -system_server
  userdebug_or_eng(`-procrank')
  userdebug_or_eng(`-perfprofd')
  -xdns #增加此处
} self:capability sys_ptrace;

neverallow {
  domain
  -appdomain
  -dumpstate
  -shell
  userdebug_or_eng(`-su')
  -system_server
  -zygote
  -xdns #增加此处
} { file_type -system_file -exec_type }:file execute;

neverallow {
  domain
  -init # TODO: limit init to relabelfrom for files
  -zygote
  -installd
  -dex2oat
  -xdns #增加此处
} dalvikcache_data_file:file no_w_file_perms;

neverallow {
  domain
  -init
  -installd
  -dex2oat
  -zygote
  -xdns #增加此处
} dalvikcache_data_file:dir no_w_dir_perms;

neverallow {
  domain
  -system_server
  -system_app
  -init
  -xdns #增加此处
  -installd # for relabelfrom and unlink, check for this in explicit neverallow
} system_data_file:file no_w_file_perms;

neverallow { domain -init -system_app #增加此处} default_prop:property_service set;

```

步骤四：在源码根目录下，输入指令:"makebootimage"进行模块编译，然后将此模块编译进入boot.img中。

### 4 如何启动、关闭以及查询xdns
请查阅链接中的调用方法：https://github.com/baidutvsafe/baidutvsafe.github.io/blob/master/index.md

## Android7.X版本集成方法
### 1 添加执行脚本exxdnsproxy.sh、xdns以及xdnsproxy到/system/bin/目录
步骤一：在android源码目录下的/system/core/rootdir/下新建xdns目录；

步骤二：将Android.mk、exxdnshproxy.sh、xdns以及xdnsproxy放到此目录system/core/rootdir/xdns/下；

步骤三：在system/core/rootdir/xdns/目录下，输入指令:"mm"进行模块编译，然后将此模块编译进入system.img中。

### 2 编写启动服务
打开/system/core/rootdir/init.rc或者在被包含编译路径下的device/XXX/XXX/init.XXX.rc文件，在文件末尾加入如下内容：
```
service startxdns /system/bin/sh /system/bin/exxdnsproxy.sh start
    class core
    disabled
    oneshot
    seclabel u:r:xdns:s0
    
 service stopxdns /system/bin/sh /system/bin/exxdnsproxy.sh stop
    class core
    disabled
    oneshot
    seclabel u:r:xdns:s0

 service checkxdns /system/bin/sh /system/bin/exxdnsproxy.sh check
    class core
    disabled
    oneshot
    seclabel u:r:xdns:s0
```

### 3 编写selinux规则te文件
步骤一：将xdns.te以及xdnsproxy.te放到android源码目录下的device/XXX/XXX/sepolicy下；

步骤二：打开android源码目录下的device/XXX/XXX/sepolicy/flie_contexts文件，在文件末尾加入如下内容：
```
/system/bin/xdnsproxy u:object_r:xdnsproxy_exec:s0

/system/bin/xdns u:object_r:xdns_exec:s0
```

步骤三：打开android源码目录下的/system/sepolicy/domain.te，修改如下内容：
```
neverallow {
  domain
  -debuggerd
  -vold
  -dumpstate
  -system_server
  -xdns #增加此处
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
  -xdns #增加此处
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
  -xdns #增加此处
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
  -xdns #增加此处
} dalvikcache_data_file:dir no_w_dir_perms;

neverallow {
  domain
  -system_server
  -system_app
  -init
  -installd # for relabelfrom and unlink, check for this in explicit neverallow
  -xdns #增加此处
} system_data_file:file no_w_file_perms;

neverallow { domain -init -system_app #增加此处} default_prop:property_service set;
```

步骤四：在源码根目录下，输入指令:"makebootimage"进行模块编译，然后将此模块编译进入boot.img中。

### 4 如何启动、关闭以及查询xdns
请查阅链接中的调用方法：https://github.com/baidutvsafe/baidutvsafe.github.io/blob/master/index.md
