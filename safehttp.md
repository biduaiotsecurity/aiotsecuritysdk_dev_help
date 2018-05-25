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
总共两个文件
一个so库：libmesalink-jni.so
一个jar包：safehttp.jar

## 集成方法
### 集成到APP当中
#### 1 添加aar文件到APP工程
将mesalink-release.aar拷贝到需要引入的module的libs目录下
