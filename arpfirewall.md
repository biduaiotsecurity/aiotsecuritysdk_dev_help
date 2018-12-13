# arpfirewall防火墙集成文档

# 目录
* [技术实现原理](#技术实现原理)
* [支持环境](#支持环境)
* [代码清单](#代码清单)
* [集成方式](#集成方式)
    * [android版本集成](#android版本集成)
        * [android4x与android5x版本集成方法](#android4x与android5x版本集成方法)
        * [android5x版本集成方法](#android5x版本集成方法)
        * [android6x版本集成方法](#android6x版本集成方法)
        * [android7x版本集成方法](#android7x版本集成方法)
        * [android8x版本集成方法](#android8x版本集成方法)
* [应用层启动方式](#应用层启动方式)
# 技术实现原理

<div align=center><img src="https://github.com/baidutvsafe/baidutvsafe.github.io/blob/master/image/arpfirewall%E9%98%B2%E7%81%AB%E5%A2%99%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86.png"/></div>
<div align=center>arpfirewall技术实现原理图</div>

首先，通过System app反射调用android.os.SystemProperties发送启动命令到init进程；

其次，init进程通过对启动命令分析init.rc存在的service进而通过shell脚本启动arpfirewall；

最后，由arpfirewall通过对shell脚本参数分析进行开启、关闭以及查询arp防火墙状态，并且将结果回调到System app。

__备注：System app为系统应用。__

# 支持环境
当前可以支持的环境：
* Android环境 [4.4 - 8.0]
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
* arpfirewallte文件夹用于集成SELinux模块
* arpfirewallbin文件夹用于集成init模块

* Linux环境
   * 单独存在一个ELF文件，可自行调用。

# 集成方式

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
+ /system/bin/xdnsproxy u:object_r:将arpfirewall_exec:s0
```

