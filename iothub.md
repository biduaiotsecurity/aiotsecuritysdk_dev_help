# IoT hub SDK接入文档
## 前置条件
1、请先注册 登录百度云

2、进入物接入IoT Hub https://cloud.baidu.com/product/iot.html

3、创建项目

4、设置策略、身份、用户等信息

## IoT hub SDK 接入方法

```
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
```

```
/**
* 获取设备id
* @return 设备id
*/
public static String generateClientId()
```

```
/**
* 设置连接保持时间
* @param interval 保持时间
*/
public static void setKeepaliveInterval(int interval)
```

```
/**
* 设置是否自动重连
* @param flag 是否重连
*/
public static void setAutoReconnect(boolean flag)
```

```
/**
* 设置重连时间间隔
* @param interval 时间间隔
*/
public static void setReconnectBaseInterval(int interval)
```

```
/**
* 设置离线后，缓存数据长度
* @param len 数据长度
*/
public static void setOfflineBufferLen(int len)
```

```
/**
* 是否清除会话
* @param flag 是否清除
*/
public static void setCleanSession(boolean flag) 
```

```
/**
* 设置LastWill消息
* @param topic LastWill消息topic
* @param msg LastWill消息内容
* @param isRetain 设置是否保留
*/
public static void setLastWill(String topic, String msg, boolean isRetain)
```

```
/**
* 设置消息接受者
* @param messageProcessor 消息接受者
*/
public static void setMessageProcessor(RawMessageProcessor messageProcessor)
```

```
/**
* 设置连接事件接受者
* @param notifier 连接事件接受者
*/
public static void setMqttClientEventNotifier(MqttClientEventNotifier notifier) throws Exception
```

```
/**
* mqtt连接
*/
public static Future<Void> connect()
```

```
/**
* 当前连接状态
* @return 连接状态
*/
public static boolean isConnected()
```

```
/**
* 发布消息
* @param rawMessage 发布的消息
*/
public static Future<Void> publishMessage(final byte[] rawMessage)
```

```
/**
* 断开连接
*/
public static void close()
```









