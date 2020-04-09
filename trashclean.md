# 垃圾清理Sdk接入文档 

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
   
    implementation  'com.squareup.okhttp3:okhttp:3.12.0'
    // 如果项目中已集成过gson，可以忽略
    implementation 'com.google.code.gson:gson:2.8.2'
    implementation 'androidx.legacy:legacy-support-v4:1.0.0'
    implementation 'org.greenrobot:eventbus:3.0.0'
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
     * @param item 扫描出的垃圾的描述，json格式的字符串，可以直接转换成TrashItem对象
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

### 垃圾描述
```
public class TrashType {

    /** 日志文件 */
    public static final int LOG_FILE = 1;

    /** 临时文件 */
    public static final int TEMP_FILE = 2;

    /** 空文件夹 */
    public static final int EMPTY_FOLDER = 4;

    /** APK安装包 */
    public static final int APK_FILE = 8;

    /** 大文件 */
    public static final int LARGE_FILE = 16;

    /** 缩略图 */
    public static final int THUMBNAIL = 32;

    /** 应用残留 */
    public static final int UNINSTALLED_APP = 64;

    /** 应用垃圾文件 */
    public static final int APP_TRASH_FILE = 128;

    /** 音乐文件 */
    public static final int MUSIC_FILE = 256;

    /** 视频文件 */
    public static final int VIDEO_FILE = 512;

    /** 图片文件 */
    public static final int IMAGE_FILE = 1024;

    /** 应用缓存 */
    public static final int APP_CACHE = 2048;

    /** 内存垃圾 */
    public static final int MEMORY_TRASH = 4096;

}

public class TrashItem {

    /**
     * 垃圾类型
     */
    public int trashType;

    
    /**
     * 应用包名，此字段仅对APK类型的垃圾有效：</br>
     * {@link TrashType#APP_CACHE},</br>
     * {@link TrashType#APK_FILE},</br>
     * {@link TrashType#UNINSTALLED_APP}
     */
    public String pkgName;

    /**
     * 应用名称，此字段仅对APK类型的垃圾有效：</br>
     * {@link TrashType#APP_CACHE},</br>
     * {@link TrashType#APK_FILE},</br>
     * {@link TrashType#UNINSTALLED_APP}
     */
    public String appName;

    /**
     * 文件路径（或目录），此字段仅对文件类型的垃圾有效：</br>
     * {@link TrashType#LOG_FILE},</br>
     * {@link TrashType#TEMP_FILE},</br>
     * {@link TrashType#EMPTY_FOLDER},</br>
     * {@link TrashType#APK_FILE},</br>
     * {@link TrashType#LARGE_FILE},</br>
     * {@link TrashType#THUMBNAIL},</br>
     * {@link TrashType#UNINSTALLED_APP}
     */
    public String filePath;

    /**
     * 垃圾大小，具体含义如下：</br>
     * 对于{@link TrashType#APP_CACHE}类型，该大小指缓存大小；</br>
     * 对于{@link TrashType#EMPTY_FOLDER}类型，此字段无效；</br>
     * 对于其它类型，均为所有垃圾文件总大小
     */
    public long size;

    public String desc;

    public TrashItem(JSONObject object) {
        this.trashType = object.optInt("trashtype");
        if (object.has("packagename")) {
            this.pkgName = object.optString("packagename");
        }
        if (object.has("appname")) {
            this.appName = object.optString("appname");
        }
        if (object.has("filepath")) {
            this.filePath = object.optString("filepath");
        }
        if (object.has("desc")) {
            this.desc = object.optString("desc");
        }
        this.size = object.optLong("size");
    }

}
```

### 注册回调服务
需要自己建立一个service，继承自`SdkTrashCallbackService`。需要保证此服务和调用接口的代码运行在同一进程当中
```
public class MyResultService extends SdkTrashCallbackService {
     // no code needed
}
```
