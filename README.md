简要概略国内Android厂商Push服务的相关信息


### 消息通知两种方式

 `透传消息` : 采用厂商服务，适用于应用不在手机后台情况下，具体方法通过厂商注册的brocastreceiver回调取得message; 点击通知开启自定义页面需要应用服务器
 传固定参数，客户端根据参数包装`intent`
 
 
 `通知消息` : 适用于应用在手机后台，可以是用自定义service保活应用，并通过回调获取消息



### Xiaomi Push Service 

 [官方文档](https://dev.mi.com/console/appservice/push.html)
 
 流程
 
* 在小米开发者站开通小米开发者账号。
* 创建应用，开发者账号审核通过后你就可以在开发者站创建你的应用。
* 开启应用的推送服务。
* 下载SDK、文档和DEMO。

[SDK 说明](https://dev.mi.com/console/doc/detail?pId=41)

代码

* 自定义广播接收器 (用于接收消息回调, 方法运行于非主线程， 注意混淆规则)

``` java
public class DemoMessageReceiver extends PushMessageReceiver {
    private String mRegId;
    private long mResultCode = -1;
    private String mReason;
    private String mCommand;
    private String mMessage;
    private String mTopic;
    private String mAlias;
    private String mUserAccount;
    private String mStartTime;
    private String mEndTime;
    @Override
    public void onReceivePassThroughMessage(Context context, MiPushMessage message) {
        mMessage = message.getContent();
        if(!TextUtils.isEmpty(message.getTopic())) {
            mTopic=message.getTopic();
        } else if(!TextUtils.isEmpty(message.getAlias())) {
            mAlias=message.getAlias();
        } else if(!TextUtils.isEmpty(message.getUserAccount())) {
            mUserAccount=message.getUserAccount();
        }
    }
    @Override
    public void onNotificationMessageClicked(Context context, MiPushMessage message) {
        mMessage = message.getContent();
        if(!TextUtils.isEmpty(message.getTopic())) {
            mTopic=message.getTopic();
        } else if(!TextUtils.isEmpty(message.getAlias())) {
            mAlias=message.getAlias();
        } else if(!TextUtils.isEmpty(message.getUserAccount())) {
            mUserAccount=message.getUserAccount();
        }
    }
    @Override
    public void onNotificationMessageArrived(Context context, MiPushMessage message) {
        mMessage = message.getContent();
        if(!TextUtils.isEmpty(message.getTopic())) {
            mTopic=message.getTopic();
        } else if(!TextUtils.isEmpty(message.getAlias())) {
            mAlias=message.getAlias();
        } else if(!TextUtils.isEmpty(message.getUserAccount())) {
            mUserAccount=message.getUserAccount();
        }
    }
    @Override
    public void onCommandResult(Context context, MiPushCommandMessage message) {
        String command = message.getCommand();
        List<String> arguments = message.getCommandArguments();
        String cmdArg1 = ((arguments != null && arguments.size() > 0) ? arguments.get(0) : null);
        String cmdArg2 = ((arguments != null && arguments.size() > 1) ? arguments.get(1) : null);
        if (MiPushClient.COMMAND_REGISTER.equals(command)) {
            if (message.getResultCode() == ErrorCode.SUCCESS) {
                mRegId = cmdArg1;
            }
        } else if (MiPushClient.COMMAND_SET_ALIAS.equals(command)) {
            if (message.getResultCode() == ErrorCode.SUCCESS) {
                mAlias = cmdArg1;
            }
        } else if (MiPushClient.COMMAND_UNSET_ALIAS.equals(command)) {
            if (message.getResultCode() == ErrorCode.SUCCESS) {
                mAlias = cmdArg1;
            }
        } else if (MiPushClient.COMMAND_SUBSCRIBE_TOPIC.equals(command)) {
            if (message.getResultCode() == ErrorCode.SUCCESS) {
                mTopic = cmdArg1;
            }
        } else if (MiPushClient.COMMAND_UNSUBSCRIBE_TOPIC.equals(command)) {
            if (message.getResultCode() == ErrorCode.SUCCESS) {
                mTopic = cmdArg1;
            }
        } else if (MiPushClient.COMMAND_SET_ACCEPT_TIME.equals(command)) {
            if (message.getResultCode() == ErrorCode.SUCCESS) {
                mStartTime = cmdArg1;
                mEndTime = cmdArg2;
            }
        } 
    }
    @Override
    public void onReceiveRegisterResult(Context context, MiPushCommandMessage message) {
        String command = message.getCommand();
        List<String> arguments = message.getCommandArguments();
        String cmdArg1 = ((arguments != null && arguments.size() > 0) ? arguments.get(0) : null);
        String cmdArg2 = ((arguments != null && arguments.size() > 1) ? arguments.get(1) : null);
        if (MiPushClient.COMMAND_REGISTER.equals(command)) {
            if (message.getResultCode() == ErrorCode.SUCCESS) {
                mRegId = cmdArg1; // 唯一识别应用Id
            }
        } 
   
```

* `regId` 属于唯一识别应用表示，每一个appId对应一个regId；获取regId后将Id上传至应用服务器
* 建议在application `onCreate()` 或者 activity `onCreate()` 进行 push token

### Huawei Push Service 


 [官方文档](https://developer.huawei.com/consumer/cn/doc/development/HMSCore-Guides/android-client-dev-0000001050042041)
 
 [流程](https://developer.huawei.com/consumer/cn/doc/development/HMSCore-Guides/android-dev-process-0000001050263396)
 
 注意事项 
 
* 开始前需要产生成签名文件，并提交sha1 key开发者平台 
* `token` 每个设备上的每个应用的Token都是唯一存在的，客户端调用HmsInstanceId类中的getToken方法向服务端请求应用的唯一标识

Token会在包括但不限于下述场景中发生变化

* App卸载重装。
* App调用注销Token方法。
* 用户恢复出厂设置。
* 清除应用数据。

最佳实践
* 应用不要固定判断Push Token长度，因为后续长度可变。
* 应用的Push Token要定期更新（建议应用每次启动的时候都获取Token，如果发现和上次取到的不同，则上报到自己的服务器）。
* 请勿使用Token跟踪标记用户。

代码

启动注册服务
``` java
private void getToken() {
    new Thread() {
        @Override
        public void run() {
            try {
                // read from agconnect-services.json
                String appId = AGConnectServicesConfig.fromContext(MainActivity.this).getString("client/app_id");
                String token = HmsInstanceId.getInstance(MainActivity.this).getToken(appId, "HCM");
                Log.i(TAG, "get token:" + token);
                if(!TextUtils.isEmpty(token)) {
                    sendRegTokenToServer(token);
                }
            } catch (ApiException e) {
                Log.e(TAG, "get token failed, " + e);
            }
        }
    }.start();
}
private void sendRegTokenToServer(String token) {
    Log.i(TAG, "sending token to server. token:" + token);
}

```

token发生变化时响应该回调
```
// This method callback must be completed in 10 seconds. Otherwise, you need to start a new Job for callback processing.
@Override
public void onNewToken(String token) {
    Log.i(TAG, "received refresh token:" + token);
    // send the token to your app server.
    if (!TextUtils.isEmpty(token)) {
        refreshedTokenToServer(token);
    }
}
private void refreshedTokenToServer(String token) {
    Log.i(TAG, "sending token to server. token:" + token);
}
```

透传消息回调

``` java
@Override
public void onMessageReceived(RemoteMessage message) {
    Log.i(TAG, "onMessageReceived is called");
    if (message == null) {
        Log.e(TAG, "Received message entity is null!");
        return;
    }
 
    Log.i(TAG, "getCollapseKey: " + message.getCollapseKey()
            + "\n getData: " + message.getData()
            + "\n getFrom: " + message.getFrom()
            + "\n getTo: " + message.getTo()
            + "\n getMessageId: " + message.getMessageId()
            + "\n getSendTime: " + message.getSentTime()
            + "\n getDataMap: " + message.getDataOfMap()
            + "\n getMessageType: " + message.getMessageType()
            + "\n getTtl: " + message.getTtl()
            + "\n getToken: " + message.getToken());
 
    RemoteMessage.Notification notification = message.getNotification();
    if (notification != null) {
        Log.i(TAG, "\n getImageUrl: " + notification.getImageUrl()
                + "\n getTitle: " + notification.getTitle()
                + "\n getTitleLocalizationKey: " + notification.getTitleLocalizationKey()
                + "\n getTitleLocalizationArgs: " + Arrays.toString(notification.getTitleLocalizationArgs())
                + "\n getBody: " + notification.getBody()
                + "\n getBodyLocalizationKey: " + notification.getBodyLocalizationKey()
                + "\n getBodyLocalizationArgs: " + Arrays.toString(notification.getBodyLocalizationArgs())
                + "\n getIcon: " + notification.getIcon()
                + "\n getSound: " + notification.getSound()
                + "\n getTag: " + notification.getTag()
                + "\n getColor: " + notification.getColor()
                + "\n getClickAction: " + notification.getClickAction()
                + "\n getChannelId: " + notification.getChannelId()
                + "\n getLink: " + notification.getLink()
                + "\n getNotifyId: " + notification.getNotifyId());
    }
 
    Boolean judgeWhetherIn10s = false;
    // If the messages are not processed in 10 seconds, the app needs to use WorkManager for processing.
    if (judgeWhetherIn10s) {
        startWorkManagerJob(message);
    } else {
        // Process message within 10s
        processWithin10s(message);
    }
}
private void startWorkManagerJob(RemoteMessage message) {
    Log.d(TAG, "Start new job processing.");
}
private void processWithin10s(RemoteMessage message) {
    Log.d(TAG, "Processing now.");
}

```

### 点击通知消息

关于通知消息，开发者可以自定义点击消息的动作，包括：打开App首页、打开特定URL、打开自定义App页面。其中打开App首页与自定义App页面需要开发者端云协同开发来完成

关于 `intent`, 必须为显示`intent`(赋予uri), 通过intent跳转自定义页面可通过`intent.data`或者`intent.extras`获取

该部分需要服务端传参定义intent属性,服务端具体可看 [服务端开发](https://developer.huawei.com/consumer/cn/doc/HMSCore-Guides-V5/android-server-dev-0000001050040110-V5)
