# 电视安全Sdk接入文档 

[TOC]


## 集成
```
repositories {
    flatDir {
        dirs 'libs'
    }
}

dependencies {
    compile(name: 'roosdk-release-vxxx', ext: 'aar')
    compile(name: 'libccb-release-vxxx', ext: 'aar')

	// 如果项目中已集成过gson，可以忽略
    compile 'com.google.code.gson:gson:2.8.0'
}
```

## 混淆配置
```
##---------------Begin: proguard configuration for Gson  ----------
# Gson uses generic type information stored in a class file when working with fields. Proguard
# removes such information by default, so configure it to keep all of it.
-keepattributes Signature

# For using GSON @Expose annotation
-keepattributes *Annotation*

# Gson specific classes
-keep class sun.misc.Unsafe { *; }
#-keep class com.google.gson.stream.** { *; }

# Application classes that will be serialized/deserialized over Gson
-keep class c.b.libccb.model.model.** { *; }

##---------------End: proguard configuration for Gson  ----------

## keep the service to receive all scan result, you can keep the callbackservice indirect which you created.
-keep public class * extends android.app.Service
```


## 使用

### 初始化与配置

```
@Override
    public void onCreate() {
        super.onCreate();
        
		// 必须在super之后
		// 参数1 ： ctx
		// 参数2 ： 回调服务所在的包名
		// 参数3 ： 回调服务名
        TVSafe.onApplicationCreate(getApplicationContext(), getApplicationContext().getPackageName(),
                MyResultService.class.getName());
    }

    @Override
    protected void attachBaseContext(Context base) {
	    // 必须在super之前
        TVSafe.attachBaseContext(base);
        super.attachBaseContext(base);
    }
```



### 调用方法。
这些方法都是静态的，你可以直接访问它们。如果接口返回失败，多数情况下是还没有加载好模块，稍等一会再尝试调用。
```
public class TVSafe {
    /**
     * 启动漏洞扫描（有界面）
     *
     * @return true 成功
     */
    public static boolean startVulnDetectorUi()

    /**
     * 扫描漏洞
     *
     * @param withPoc 是否包含poc检测，建议传false,带Poc的检测可能会导致设备重启
     * @return true 成功
     */
    public static boolean scanVuln(boolean withPoc) {

    /**
     * 启动病毒扫描模块（有界面）
     *
     * @return true 成功
     */
    public static boolean startVirusScannerUi()

    /**
     * 扫描病毒
     *
     * @param sysApp 是否包含系统app，建议传false。
     * @return true 成功
     */
    public static boolean scanVirus(boolean sysApp)

    /**
     * 静默启动病毒扫描模块，扫描指定文件(无界面）
     *
     * @param filePath 文件绝对路径
     * @return true 成功
     */
    public static boolean scanApkFile(String filePath)

    /**
     * 静默启动病毒扫描模块， 扫描指定路径(无界面）
     *
     * @param dir 文件夹路径
     * @return true 成功
     */
    public static boolean scanDir(String dir)


    /**
     * 启动网络扫描模块（有界面）
     *
     * @return true 成功
     */
    public static boolean startNetScannerUi()

    /**
     * 静默网络扫描模块(无界面）
     *
     * @return true 成功
     */
    public static boolean startNetScanner()

    /**
     * dns修复
     */
    public static boolean fixDns()

    /**
     * 通过url获取ip
     */
    public static boolean getIPsUrlByDomainUrl(String url)

    /**
     * 请求url风险信息
     */
    public static boolean queryUrlSafeLevel(String url)
}
```


### 回调
需要自己建立一个service，继承自`AbstractResultService `。
```
public class MyResultService extends AbstractResultService {
    @Override
    public void onAvpScanResult(List<AvpScanResult> list) {
        for (AvpScanResult scanResult : list) {
            Log.i("onAvpScanResult", scanResult.toString());
        }
    }

    @Override
    public void onVulnScanResult(List<VulnInfo> list) {
        for (VulnInfo vulnInfo : list) {
            Log.i("onVulnScanResult", vulnInfo.toString());
        }
    }

    @Override
    public void onNedScanResult(NedResult result) {
        Log.i("onNedScanResult", result.toString());
    }

    @Override
    public void onDnsFixed(boolean suc) {
        Log.i("onDnsFixed", suc ? "true" : "false");
    }

    @Override
    public void onGetIpResult(DnsResult dnsResult) {
        Log.i("onGetIpResult", dnsResult.toString());
    }

    @Override
    public void onGetUrlInfoResult(SafeUrlInfo safeUrlInfo) {
        Log.i("onGetUrlInfoResult", safeUrlInfo.toString());
    }
}
```

## 实验结果
### onAvpScanResult
```
public class AvpScanResult implements Serializable {
    /**
     * 病毒级别
     * 2, "低风险"
     * 3, "高风险"
     * 4, "恶意应用"
     */
    private int level;

    /**
     * 完整的风险列表
     */
    private List<AvpThreatInfo> threatInfos = new ArrayList();
    /**
     * 应用包名
     */
    private String pkgName;
    /**
     * app名字
     */
    private String label;
    /**
     * 扫描路径
     */
    private String path;
    /**
     * 是否是顽固木马
     */
    private boolean isStubbornVirus;
    /**
     * 是否是样本统计病毒
     */
    private boolean isSampleWanted;
}

public class AvpThreatInfo implements Serializable {
    /**
     * 风险名称
     */
    private String name;
    /**
     * 该条风险的安全级别
     */
    private int rating;
    /**
     * 风险类型
     * "Privacy", "隐私窃取"
     * "Payment", "恶意扣费"
     * "Remote", "远程控制"
     * "Spread", "恶意传播"
     * "Expense", "资源消耗"
     * "System", "系统破坏"
     * "Fraud", "诱骗欺诈"
     * "Rogue", "流氓行为"
     */
    private List<String> risks = new ArrayList();

    /**
     * 隐私泄露类型
     */
    private List<String> privacies = new ArrayList();
    /**
     * 广告类别
     */
    private List<String> styles = new ArrayList();
    /**
     * 广告行为描述
     */
    private List<String> actions = new ArrayList();
    /**
     * 行为分类
     */
    private List<String> behaviors = new ArrayList();
    /**
     * 风险描述
     */
    private String description;
}
```



```html
04-04 10:29:10.061 14352-14700/com.baidu.roosdkdemo I/onAvpScanResult: AvpScanResult{level=1, pkgName='com.tencent.mm', label='微信', isStubbornVirus='false', isSampleWanted='false', threatInfos=[], magicMd5='2bc2a45a1ffd921abd54e9b62d4ee8f5', path='/data/app/com.tencent.mm-1.apk', jsonResult='null'}
04-04 10:29:10.061 14352-14700/com.baidu.roosdkdemo I/onAvpScanResult: AvpScanResult{level=1, pkgName='com.baidu.hi', label='百度 Hi', isStubbornVirus='false', isSampleWanted='false', threatInfos=[], magicMd5='6266aacaa9d61c5818d00392ff7e3ac4', path='/data/app/com.baidu.hi-1.apk', jsonResult='null'}
04-04 10:29:10.061 14352-14700/com.baidu.roosdkdemo I/onAvpScanResult: AvpScanResult{level=1, pkgName='de.langerhans.wallet', label='Dogecoin Wallet', isStubbornVirus='false', isSampleWanted='false', threatInfos=[], magicMd5='83dcbe7a21a74aa9b2b54613d0573af1', path='/data/app/de.langerhans.wallet-1.apk', jsonResult='null'}

```

### onVulnScanResult
```
public class VulnInfo implements Serializable {
    /**
     * 风险等级
     * 5 :moderate
     * 3: high
     */
    int risk;
    /**
     * 漏洞名
     */
    String cve;
    /**
     * 漏洞类型，比如：远程代码执行漏洞
     */
    String type;
    /**
     * 漏洞公布日期
     */
    String date;
    /**
     * 漏洞简述
     */
    String desc;
    /**
     * 1 : 存在该漏洞
     * -1 ： 不存在该漏洞
     */
    int result;
}
```

```html
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2017-0421 seq : 1 risk : 5 type : 安卓框架中的信息泄漏漏洞 date : 2017-02-01 desc : 本地恶意应用程序可能利用此漏洞绕过系统限制获取其他应用的敏感信息 result : -1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2017-0389 seq : 2 risk : 3 type : 网络服务中的拒绝服务漏洞 date : 2017-01-03 desc : 攻击者可以利用此漏洞远程攻击系统核心网络服务 result : -1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-3897 seq : 3 risk : 5 type : WIFI模块中的信息泄漏漏洞 date : 2016-09-01 desc : 利用此漏洞可导致攻击者获取WIFI的敏感信息 result : -1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-3832 seq : 4 risk : 5 type : 安卓框架界面中的权限提升漏洞 date : 2016-08-01 desc : 利用此漏洞，攻击者可能使自身权限得以提升 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-2497 seq : 5 risk : 5 type : 安卓框架界面中的权限提升漏洞 date : 2016-08-05 desc : 利用此漏洞，攻击者可能使自身权限得以提升 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-2426 seq : 6 risk : 5 type : 安卓框架中的信息泄漏漏洞 date : 2016-04-02 desc : 利用此漏洞，可能导致攻击者获取系统敏感信息 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-2496 seq : 7 risk : 5 type : 安卓框架界面中的权限提升漏洞 date : 2016-06-01 desc : 利用此漏洞，攻击者可能使自身权限得以提升 result : -1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-2430 seq : 8 risk : 3 type : Debuggerd中的权限提升漏洞 date : 2016-02-25 desc : 可使本地恶意应用程序在Debuggerd进程内执行任意代码 result : -1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-2416 seq : 9 risk : 3 type : 未授权信息泄漏 date : 2016-02-26 desc : 导致未授权的攻击者可获取系统敏感信息 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-0830 seq : 10 risk : 3 type : 蓝牙组件中的远程代码执行漏洞 date : 2016-03-01 desc : 利用此漏洞，可能导致攻击者在蓝牙组件中执行任意代码 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-3862 seq : 11 risk : 5 type : 远程代码执行漏洞 date : 2016-09-01 desc : 系统在处理媒体文件和数据时，Mediaserver 中的远程代码执行漏洞可让攻击者使用特制文件破坏内存 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2017-0492 seq : 12 risk : 3 type : 提权漏洞 date : 2017-03-01 desc : 系统界面中的提取漏洞可让本地恶意应用创建覆盖整个屏幕的界面叠加层 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-3831 seq : 13 risk : 5 type : 拒绝服务 date : 2016-05-31 desc : 系统时钟内的拒绝服务漏洞可让远程攻击者破坏设备 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-6770 seq : 14 risk : 3 type : 提权漏洞 date : 2016-07-16 desc : Framework API 中的提权漏洞可让本地恶意应用访问超出其访问权限级别的系统功能 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2017-0493 seq : 15 risk : 3 type : 信息披露漏洞 date : 2017-05-01 desc : 文件级加密中的信息披露漏洞可让本地恶意攻击者绕过用于锁定屏幕的操作系统防护功能 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-3917 seq : 21 risk : 5 type : 提权漏洞 date : 2016-10-05 desc : 指纹登录中的提权漏洞可让恶意设备所有者在相应设备上使用其他用户帐号登录 result : -1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-3923 seq : 22 risk : 5 type : 提权漏洞 date : 2016-10-05 desc : 无障碍服务中的提权漏洞可让本地恶意应用在设备上生成异常触摸事件，从而可能导致应用在未经用户明确同意的情况下在接受权限对话框中执行接受操作 result : -1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2015-6764 seq : 16 risk : 5 type : 远程服务拒绝漏洞 date : 2015-12-01 desc : 在google chrome浏览器中，允许远端攻击者导致浏览器拒绝服务 result : -1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2015-6771 seq : 17 risk : 5 type : 远程服务拒绝漏洞 date : 2015-12-01 desc : 在google chrome浏览器中，允许远端攻击者导致浏览器拒绝服务 result : -1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-1688 seq : 20 risk : 3 type : 远程服务拒绝漏洞 date : 2016-06-01 desc : 在google chrome浏览器中，允许远端攻击者导致浏览器拒绝服务 result : -1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-1677 seq : 19 risk : 3 type : 远程代码执行漏洞 date : 2016-06-01 desc : 在google chrome浏览器中，允许远端攻击者获取敏感信息 result : -1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-1646 seq : 18 risk : 5 type : 远程服务拒绝漏洞 date : 2016-04-01 desc : 在google chrome浏览器中，允许远端攻击者导致浏览器拒绝服务 result : -1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-0826 seq : 23 risk : 5 type : 提权漏洞 date : 2016-03-01 desc : Mediaserver 中的提权漏洞可让本地恶意应用通过提权后的系统应用执行任意代码 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-3921 seq : 24 risk : 3 type : 提权漏洞 date : 2016-10-05 desc : Framework 监听器中的提权漏洞可让本地恶意应用通过特许进程执行任意代码 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2017-0423 seq : 25 risk : 3 type : 提权漏洞 date : 2017-02-05 desc : 蓝牙中的提权漏洞可让邻近区域内的攻击者管理对设备上文档的访问权限 result : -1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-3754 seq : 26 risk : 5 type : 拒绝服务漏洞 date : 2016-07-05 desc : Mediaserver 中的拒绝服务漏洞可让攻击者使用特制文件挂起或重启设备 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-3861 seq : 27 risk : 5 type : 远程代码执行漏洞 date : 2016-09-06 desc : LibUtils 中的远程代码执行漏洞可让攻击者使用特制文件通过特许进程执行任意代码 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-2495 seq : 29 risk : 5 type : 远程拒绝服务漏洞 date : 2016-06-01 desc : Mediaserver 中的远程拒绝服务漏洞可让攻击者使用特制文件让设备挂起或重启 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-3744 seq : 30 risk : 5 type : 远程代码执行漏洞 date : 2016-07-05 desc : 蓝牙中的远程代码执行漏洞可让邻近的攻击者在配对过程中执行任意代码 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-0838 seq : 33 risk : 5 type : 远程代码执行漏洞 date : 2016-04-02 desc : 攻击者可通过 mediaserver 中的漏洞破坏内存并通过 mediaserver 进程执行远程代码 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-2449 seq : 31 risk : 5 type : 提权漏洞 date : 2016-05-01 desc : Mediaserver 中的提权漏洞可让本地恶意应用通过提权后的系统应用执行任意代码 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-2476 seq : 32 risk : 5 type : 提权漏洞 date : 2016-06-01 desc : Mediaserver 中的提权漏洞可让本地恶意应用通过提权后的系统应用执行任意代码 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2016-3915 seq : 28 risk : 5 type : 提权漏洞 date : 2016-10-05 desc : 相机服务中的提权漏洞可让本地恶意应用通过特许进程执行任意代码 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2017-0782 seq : 34 risk : 5 type : 远程代码执行 date : 2017-09-05 desc : 可让邻近区域内的攻击者通过特许进程执行任意代码 result : 1
04-04 10:30:08.661 14352-14817/com.baidu.roosdkdemo I/onVulnScanResult: cve : CVE-2017-0783 seq : 35 risk : 5 type : 信息披露 date : 2017-09-05 desc : 完全的信息泄露导致所有系统文件暴露 result : 1

```


### onNedScanResult
```
public class NedResult implements Serializable {
    /**
     * true : 网络在线
     */
    public boolean isNetOnline;
    /**
     * true : 不是虚假wifi
     */
    public boolean isNotFakeWifi;
    /**
     * true : wifi无线已加密
     */
    public boolean isWifiEncryption;
    /**
     * true : dns检测安全
     */
    public boolean isNotFakeDNS;
    /**
     * true : arp本地检测安全
     */
    public boolean isWifiArpSafe;
    /**
     * true : ssl 攻击检测安全
     */
    public boolean isWifiSSLSafe;
    /**
     * wifi强度
     */
    public int wifiIntensity;
    /**
     * 重定向Url
     */
    public String wifiRedirectURL;
}
```

```html
04-04 10:31:02.481 14352-15027/com.baidu.roosdkdemo I/onNedScanResult: {"state":{"checktime":0,"code":128},"datas":{"isNotFakeWifi":false,"isWifiArpSafe":false,"isWifiEncryption":false,"isNetOnline":false,"isNotFakeDNS":false,"isWifiSSLSafe":false}}
```

### onDnsFixed
```html
04-08 13:52:58.848 19851-20155/? I/onDnsFixed: true
```

### onGetIpResult
```
public class DnsResult implements Serializable {
    /**
     * 返回代码
     * 1 ： 解析域名成功
     * -1 ：解析域名失败
     * -2 ： 当前无网络
     * -3 ： 服务器出错
     * -4 ： 未知错误
     */
    public int code;
    /**
     * 错误描述
     */
    public String err = "";
    /**
     * 安全dns解析后的ip列表
     */
    public List<String> ips = new ArrayList<>();
}
```

```html
04-10 11:12:31.018 1251-1781/com.baidu.roosdkdemo I/onGetIpResult: code : 1,err : 解析域名成功,ip : 119.75.213.61,ip : 119.75.216.20
```

### onGetUrlInfoResult
```
public class SafeUrlInfo implements Serializable {
    /**
     * url
     */
    public String url;
    /**
     * 安全等级
     * 1 ：LEVEL_TOP_SAFE
     * 2 ：LEVEL_SAFE
     * 3 ：LEVEL_NORMAL
     * 4 ：LEVEL_LOW_RISK
     * 5 ：LEVEL_HIGH_RISK
     * 6 ：LEVEL_MALICIOUS
     */
    public int safeLevel;
    /**
     * 描述
     */
    public String desc;
```

```html
04-10 10:53:25.678 31600-382/? I/onGetUrlInfoResult: url : www.baidu.com,safeLevel : 2,desc : null
```



# dns修复
这个模块比较特殊，有个限制条件，必须在root权限下执行。
因此给出两种起调方案。
## 直接以接口形式
如果集成了roosdk的宿主，可以获得root权限，直接调用`RooSdk.fixDns(getApplicationContext());`，此方法在内部会执行`su -c ./xxxxx`
## 可执行文件
如果集成了roosdk的宿主没法获取root权限，可以把这两个可执行文件(xdns、xdnsproxy，在sdk的assets文件夹里，解压它就可以了)单独拿出来，在适当需要修复dns的时候，比如onNedScanResult回调dns异常时。需要做两件工作。

*	把xdns 与 xdnsproxy放同一目录，给他们两个都chmod 上可执行权限
*	执行xdns，参数加上回调的service，比如：`./xdns -p com.baidu.roosdkdemo -c com.baidu.roosdkdemo.MyResultService`
	*	-p : 包名
	*	-c : 服务的类名
## 最后
选择后者（以可执行文件集成）的话，我们会给到一个不含那两个文件的sdk，这样可以缩小sdk的体积，两个可执行文件可以单独给到。具体方案上的选择可以再沟通。
