# xdns反DNS劫持方案集成文档——Linux版本

# 目录
* [支持环境](#支持环境)
* [代码清单](#代码清单)
* [启动方式](#启动方式)
   * [启动xdnsproxy](#启动xdnsproxy)
   * [停止xdnsproxy](#停止xdnsproxy)
* [Q&A](#Q&A)

# 支持环境
当前可支持的环境：
* Linux环境
   由于LinuxOS版本与CPU架构有所不同，需要适配其他的Linux版本提供交叉工具链并且请联系我们。

# 代码清单
* Linux环境
    * 二进制文件：xdnsproxy
    
# 启动方式
## 启动xdnsproxy
步骤一：root权限下，命令行输入"xdnsproxy"，启动xdnsproxy；
步骤二：修改系统配置文件/etc/resolv.conf，为如下内容：
   ```
   nameserver 127.0.0.1 # 新增DNS配置项
   nameserver X.X.X.X # 原DNS_1配置项目
   ```
步骤三：命令行行输入"ping xdnsproxy.keepalive.check", 显示如下内容即启动成功。
  ```
  ```
   
   
## 停止xdnsproxy
步骤一：root权限下，命令行输入"xdnsproxy --stop-proxy" 或者找到xdnsproxy 进程 ID， 假设为$PID，执行 kill -9 $PID 命令。
步骤二：修改系统配置文件/etc/resolv.conf，为如下内容：
   ```
   nameserver X.X.X.X # 原DNS_1配置项目
   nameserver X.X.X.X # 原DNS_2配置项目
   ```
   
# Q&A
   1、问：启动xdnsproxy返回false？  
   答：输入lsof -i:53 查看是否有进程占用53端口，如果有的话，麻烦输入“kill -9 占用进程的PID”。
