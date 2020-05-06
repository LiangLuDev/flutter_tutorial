原生功能需要使用插件实现，例如：蓝牙、WiFi、Camera等，需要调用相应平台的原生代码实现。

![image](https://static001.infoq.cn/resource/image/94/7f/94580b00c44b98e1bfe557daadb29c7f.png)


### 创建插件

Android Studio ：` New - Moudle - Flutter Plugin `

命令行： `flutter create -t plugin flutter_plugin_demo`


### 通信方式
- MethodChannel：方法调用（Native <-> Flutter）
- BasicMessageChannel：发送消息，传递字符串和半结构化的信息（Native <-> Flutter）
- EventChannel：原生发送消息，Flutter接收，数据流（event streams）（Native -> Flutter）


### MethodChannel（方法调用）

#### Android调用Flutter

##### Android 代码

```
// 初始化MethodChannel
MethodChannel channel = MethodChannel(registrar.messenger(),'flutterplugin');
//调用Flutter方法，1.调用flutter方法名。2.传入参数，基本数据类型 & list & map。3.非必须，回调
channel.invokeMethod("flutter_method", "flutter method test", new Result() {
      @Override
      public void success(Object o) {
        
      }

      @Override
      public void error(String s, String s1, Object o) {

      }

      @Override
      public void notImplemented() {

      }
    });
```
##### Flutter 代码

```
// 初始化MethodChannel
static const MethodChannel _channel = const MethodChannel('flutterplugin');
// 根据方法名，处理Android调用的方法
_channel.setMethodCallHandler((MethodCall call) async{
      switch (call.method) {
        case 'flutter_method':
          print('method : ${call.arguments}');
          return 'return返回的数据在Android的回调中接收';
          break;
      }
    });
```

#### Flutter调用Android

##### Android 代码

```
//在onMethodCall中监听Flutter调用什么名字的方法（此处getPlatformVersion），通过result返回方法的执行结果。
channel.setMethodCallHandler(new MethodChannel.MethodCallHandler(){
     @Override
     public void onMethodCall(MethodCall methodCall, Result result) {
        if (methodCall.method.equals("getPlatformVersion")) {
          result.success("Android " + android.os.Build.VERSION.RELEASE);
        }
     }
});
```
##### Flutter 代码

```
// 调用Android的方法，接收返回数据，方法是异步的
static Future<String> get platformVersion async {
    final String version = await _channel.invokeMethod('getPlatformVersion');
    return version;
  }
```

### BasicMessageChannel（互相发送消息）

#### Android调用Flutter

##### Android 代码

```
// 初始化BasicMessageChannel
BasicMessageChannel messageChannel = new BasicMessageChannel<>(registrar.messenger(),"fluttermsg", StandardMessageCodec.INSTANCE);
//调用发送消息的方法
messageChannel.send("fluttermsg : hi");
```
##### Flutter 代码

```
// 初始化MethodChannel
static const BasicMessageChannel _messageChannel = const BasicMessageChannel('fluttermsg',StandardMessageCodec());
// 根据方法名，处理Android调用的方法
_messageChannel.setMessageHandler((handler)async{
    print('message : ${handler.toString()}');
});
```

#### Flutter调用Android

##### Android 代码

```
//实现onMessage方法，处理收到Flutter发送来的消息
messageChannel.setMessageHandler(new BasicMessageChannel.MessageHandler(){
      @Override
      public void onMessage(Object o, BasicMessageChannel.Reply reply) {
        Log.d("处理接收到Flutter发送的消息",o.toString());
        //reply数据到Flutter
        reply.reply(o.toString()+"to flutter");
      }
    });
```
##### Flutter 代码

```
// 发送消息给Native并返回数据
Future<dynamic> _sendMessageToNative(String msg) async{
    return await _messageChannel.send(msg);
  }
```

### EventChannel（原生发送消息，Flutter接收）

##### Android代码
```
//初始化EventChannel
EventChannel eventChannel = new EventChannel(registrar.messenger(),"flutterevent");
//监听充电广播，实时反馈充电状态给Flutter
eventChannel.setStreamHandler(new EventChannel.StreamHandler(){
      @Override
      public void onListen(Object o, EventChannel.EventSink eventSink) {
        BroadcastReceiver chargingBroadcastReceiver = createChargingBroadcastReceiver(eventSink);
        registrar.context().registerReceiver(chargingBroadcastReceiver,new IntentFilter(Intent.ACTION_BATTERY_CHANGED));
      }
      @Override
      public void onCancel(Object o) {
      }
    });
    
private static BroadcastReceiver createChargingBroadcastReceiver(final EventChannel.EventSink eventSink) {
    return new BroadcastReceiver() {
      @Override
      public void onReceive(Context context, Intent intent) {
        int status = intent.getIntExtra(BatteryManager.EXTRA_STATUS, -1);
        if (status == BatteryManager.BATTERY_STATUS_UNKNOWN) {
          eventSink.error("UNAVALIABLE", "charging status is unavailable", null);
        } else {
          boolean isCharging = status == BatteryManager.BATTERY_STATUS_CHARGING;
          eventSink.success(isCharging ? "charging" : "disCharging");
        }
      }
    };
  }
```

##### Flutter代码
```
static const EventChannel _eventChannel = const EventChannel('flutterevent');
//监听Android发送的消息
 _eventChannel.receiveBroadcastStream().listen((onData){
      //处理Android发送过来的消息
      print(onData);
 }).onError((error){
      print(error);
 });
```