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
    compile(name: 'tvsafe-release-vxxx', ext: 'aar')
   

	// 如果项目中已集成过gson，可以忽略
    compile 'com.google.code.gson:gson:2.8.0'
    
    // 最少得集成一个v4包，如果已经集成过v4包，可以忽略
    compile 'com.android.support:support-v4:25.1.1'
}
```

## 混淆配置
SDK已经做好混淆规则。

## te规则
如果是系统app集成，则需要加一条te规则。
allow system_app system_app_data_file:file execute
如果关闭了selinux可以忽略。


## 使用

### 初始化与配置
这里面的回调是Application的。
```
@Override
    public void onCreate() {
        super.onCreate();
        
	// 必须在super之后
	// 参数1 ： ctx
	// 参数2 ： 回调服务所在的包名,就是最终app的包名。
	// 参数3 ： 回调服务名
        IoTSecurity.onApplicationCreate(getApplicationContext(), getApplicationContext().getPackageName(),
                MyResultService.class.getName());
    }

    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);
	IoTSecurity.attachBaseContext(this);
    }
```



### 调用方法
这些方法都是静态的，你可以直接访问它们。如果接口返回失败，多数情况下是还没有加载好模块，稍等一会再尝试调用。
```
public class IoTSecurity {
    /**
     * 启动垃圾扫描
     */
    public static void startScan(TrashScanCallback callback)

    /**
     * 启动深度垃圾扫描
     */
    public static void startDeepScan(TrashScanCallback callback)

    /**
     * 启动垃圾清理
     */
    public static void startClean(TrashCleanCallback callback)
}
    
```

### 方法参数说明
```
public interface TrashScanCallback {
    /**
     * 垃圾扫描开始
     */
    void onBegin();

    /**
     * 垃圾扫描进度回调
     *
     * @param progress 当前扫描进度，1-100
     */
    void onScanProgress(int progress);

    /**
     * 垃圾扫描回调
     *
     * @param item 扫描出的垃圾的描述
     */
    void onScanItem(String item);

    /**
     * 垃圾扫描结束
     */
    void onEnd();
}


public interface TrashCleanCallback {

    /**
     * 垃圾清理开始
     */
    void onBegin();

    /**
     * 垃圾清理进度
     * @param progress 当前的清理进度，1-100
     */
    void onCleanProgress(int progress);

    /**
     * 垃圾清理结束
     */
    void onEnd();
}

```

### 回调
需要自己建立一个service，继承自`SdkTrashCallbackService `。需要保证此服务和调用接口的代码运行在同一进程当中
```
public class MyResultService extends SdkTrashCallbackService {
     // no code needed
}
```

