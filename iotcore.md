# IoT Core SDK Android(Security) 接入文档
## 前置条件
1、请先注册 登录百度云

2、进入物接入IoT Core https://cloud.baidu.com/product/iot.html

3、创建项目

4、设置策略、身份、用户等信息

## IoT Core SDK Android(Security) 接入方法

### SDK初始化
在Application中接入
```
@Override
public void onCreate() {
    super.onCreate();
    IoTSecurity.onApplicationCreate(this, getPackageName(),
            MyIotcorResultService.class.getName());
    // SDK初始化，初始化结果会在指定的IntentService中回调（本例中是MyIotcorResultService）
}

@Override
protected void attachBaseContext(Context base) {
    super.attachBaseContext(base);
    IoTSecurity.attachBaseContext(this);
}
```
### 建立回调服务
创建IntentService，继承自DefaultIotCoreResultService
```
public class MyIotcorResultService extends DefaultIotCoreResultService {
    @Override
    public void onInitialized(boolean suc) {
        // 接收到这个回调后，代表iotcore SDK准备就绪，可以使用iotcore相关功能了。
    }

    @Override
    public void onDeviceRegistered(int rc, UserDeviceInfo deviceInfo, String errMsg) {
        // 接收到这个回调后，代表设备自注册成功，可以开始连接、订阅、发布消息了。
    }
}
```

### 设备动态注册与订阅

1. 调用IoTSecurity.autoRegisterDevice开始自注册，回调结果会在IoTSecurity.onApplicationCreate指定的服务中回调（本例中在MyIotcorResultService的onInitialized中回调）。
2. 收到注册成功后，调用IoTSecurity.setCallback(this)注册mqtt回调。
3. IoTSecurity.connect 连接云物联
4. IoTSecurity.subscribe 开始订阅
5. IoTSecurity.publish 发布消息

## IoT Core SDK Android(Security) 接口列表
```
/**
* 设备预配（动态注册）
*
* @param deviceKey 如果填写null或空字符串，那么默认采集mac作为deviceKey
*                  最终注册结果会在intentService里通知，@see DefaultIotCoreResultService
*/
public static void autoRegisterDevice(String deviceKey, String taskId, String taskSecret)

/**
* 缓存devicekey & deviceSecret，连接之前必须调用
*/
public static boolean setIoTCoreKey(String deviceKey, String deviceSecret)
    
/**
* 初始化接口
* @param broker 服务器地址，可以在物接入项目中可以看到，推荐使用tcp方式
* @param clientId 设备id，可以通过generateClientId获取，或者自定义
* @param userName 连接用户名，物接入控制台创建用户后可以看到
* @param password 连接密码，物接入控制台创建用户后可以看到
* @param publishTopic 发布的topic
* @param subscribeTopic 订阅的topic
* @throws Exception 初始化异常时，会抛出异常
*/
public static void init(String broker,
                     String clientId, String userName,
                     String password, String publishTopic,
                     String subscribeTopic) throws Exception

/**
* 获取设备id
* @return 设备id
*/
public static String generateClientId()



/**
* 设置连接保持时间
* @param interval 保持时间
*/
public static void setKeepaliveInterval(int interval)



/**
* 设置是否自动重连
* @param flag 是否重连
*/
public static void setAutoReconnect(boolean flag)



/**
* 设置重连时间间隔
* @param interval 时间间隔
*/
public static void setReconnectBaseInterval(int interval)



/**
* 设置离线后，缓存数据长度
* @param len 数据长度
*/
public static void setOfflineBufferLen(int len)


/**
* 是否清除会话
* @param flag 是否清除
*/
public static void setCleanSession(boolean flag) 



/**
* 设置LastWill消息
* @param topic LastWill消息topic
* @param msg LastWill消息内容
* @param isRetain 设置是否保留
*/
public static void setLastWill(String topic, String msg, boolean isRetain)



/**
* 设置消息接受者
* @param messageProcessor 消息接受者
*/
public static void setMessageProcessor(RawMessageProcessor messageProcessor)



/**
* 设置连接事件接受者
* @param notifier 连接事件接受者
*/
public static void setMqttClientEventNotifier(MqttClientEventNotifier notifier) throws Exception



/**
* mqtt连接
*/
public static Future<Void> connect()


/**
* 当前连接状态
* @return 连接状态
*/
public static boolean isConnected()



/**
* 发布消息
* @param rawMessage 发布的消息
*/
public static Future<Void> publishMessage(final byte[] rawMessage)



/**
* 断开连接
*/
public static void close()
```









