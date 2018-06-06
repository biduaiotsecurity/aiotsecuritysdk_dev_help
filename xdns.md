# xdns接入文档

## 技术实现原理

<div align=center><img src="https://github.com/baidutvsafe/baidutvsafe.github.io/blob/master/image/xdns%E7%9A%84%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86.png"/></div>
<div align=center>xdns技术实现原理图</div>

首先，通过System app反射调用android.os.SystemProperties发送启动命令到init进程；

其次，init进程通过对启动命令分析init.rc存在的service进而通过shell脚本启动xdns；

最后，由xdns通过对shell脚本参数分析进行开启、关闭以及查询xdnsproxy状态，并且将结果回调到System app。

备注：System app为系统应用。

## 集成方法
### 1 添加执行脚本exxdnsproxy.sh、xdns以及xdnsproxy到/system/bin/目录
步骤一：在android源码目录下的/system/core/rootdir/下新建xdns目录；

步骤二：将Android.mk、exxdnshproxy.sh、xdns以及xdnsproxy放到此目录system/core/rootdir/xdns/下；

步骤三：在system/core/rootdir/xdns/目录下，输入指令:"mm"进行模块编译，然后将此模块编译进入system.img中。

### 2 编写启动服务
打开/system/core/rootdir/init.rc文件，在文件末尾加入如下内容：
```
service startxdns /system/bin/sh /system/bin/exxdnsproxy.sh com.baidu.rootv com.baidu.rootv.MyResultService start
    class core
    disabled
    oneshot
    
 service stopxdns /system/bin/sh /system/bin/exxdnsproxy.sh com.baidu.rootv com.baidu.rootv.MyResultService stop
    class core
    disabled
    oneshot

 service checkxdns /system/bin/sh /system/bin/exxdnsproxy.sh com.baidu.rootv com.baidu.rootv.MyResultService check
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
步骤四：在/exrernal/sepolicy/目录下，输入指令:"mm"进行模块编译，然后将此模块编译进入system.img中。

### 4 如何启动、关闭以及查询xdns
请查阅链接中的调用方法：https://github.com/baidutvsafe/baidutvsafe.github.io/blob/master/index.md
