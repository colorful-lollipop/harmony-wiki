# 关键调用链

## 1. 拨打流程

### 1.1 完整调用链

```
用户点击拨号 (pages/index.ets)
        │
        ▼
┌─────────────────────────┐
│ CallServiceProxy.dialCall() │
│ 文件: CallServiceProxy.ts  │
│ 行号: 54-61               │
└───────────┬─────────────┘
            │ call.dial(phoneNumber, options)
            ▼
    @ohos.telephony.call.dial()
            │
            ▼
    Telephony Framework (系统)
            │
            ▼
    电话子系统 (调制解调器)
```

### 1.2 核心代码

```typescript
// 文件: entry/src/main/ets/model/CallServiceProxy.ets

/**
 * 拨打电话
 * @param phoneNumber 电话号码
 * @param accountId 账户 ID
 * @param videoState 视频状态
 * @param dialScene 拨号场景
 */
public dialCall(phoneNumber, accountId = 0, videoState = 0, dialScene = 0) {
  LogUtils.i(TAG, 'dialCall phoneNumber :');
  return call.dial(phoneNumber, {
    accountId,
    videoState,
    dialScene
  });
}
```

## 2. 来电处理流程

### 2.1 完整调用链

```
电话系统 (Telephony Framework)
        │
        │ 通话状态变化
        ▼
@ohos.telephony.call.on('callDetailsChange')
        │
        ▼
TelephonyApi.registerCallStateCallback() [ServiceAbility]
        │
        │ IPC 回调
        ▼
ServiceAbility.Stub.onRemoteRequest()
        │
        │ 数据传递
        ▼
CallManager.update(callData)
        │
        │ 状态更新
        ▼
AppStorage.SetOrCreate('CallState', data)
        │
        │ UI 响应
        ▼
页面刷新 (pages/index.ets)
```

### 2.2 核心代码

```typescript
// 文件: entry/src/main/ets/ServiceAbility/TelephonyApi.ets

public registerCallStateCallback(callBack) {
  try {
    call.on('callDetailsChange', (data) => {
      if (!data) {
        LogUtils.i(TAG, 'call.on registerCallStateCallback')
        return;
      }
      LogUtils.i(TAG, 'call.on registerCallStateCallback callState: ' + JSON.stringify(data.callState))
      callBack(data);  // 回调传递数据
    });
  } catch (err) {
    LogUtils.i(TAG, 'call.on registerCallStateCallback catch:' + JSON.stringify(err));
  }
}

// 文件: entry/src/main/ets/model/CallManager.ets

async update(callData) {
  LogUtils.i(TAG, 'update calldata:')
  this.callData = callData;
  this.mCallDataManager.update(callData);

  // 更新 AppStorage，触发 UI 更新
  AppStorage.SetOrCreate('AccountNumber', formattedNumber);
  LogUtils.i(TAG, 'update :');
}
```

## 3. 接听/挂断流程

### 3.1 接听电话

```
用户点击接听 (页面按钮)
        │
        ▼
CallServiceProxy.acceptCall(callId)
        │
        │ call.answerCall(callId)
        ▼
@ohos.telephony.call.answerCall()
        │
        ▼
电话系统处理
```

### 3.2 挂断电话

```
用户点击挂断 (页面按钮)
        │
        ▼
CallServiceProxy.hangUpCall(callId)
        │
        │ call.hangUpCall(callId)
        ▼
@ohos.telephony.call.hangUpCall()
        │
        ▼
Promise 返回结果
        │
        ▼
更新通话状态
```

### 3.3 核心代码

```typescript
// 文件: entry/src/main/ets/model/CallServiceProxy.ets

/**
 * 接听电话
 */
public acceptCall = function (callId) {
  call.answerCall(callId).then((res) => {
    LogUtils.i(TAG, prefixLog + 'call.answerCall : %s' + JSON.stringify(callId))
  }).catch((err) => {
    LogUtils.i(TAG, prefixLog + 'call.answerCall catch : %s' + JSON.stringify(err))
  });
};

/**
 * 挂断电话
 */
public hangUpCall = (callId) => new Promise((resolve, reject) => {
  call.hangUpCall(callId).then((res) => {
    resolve(res);
    LogUtils.i(TAG, prefixLog + 'then:hangUpCall : %s' + JSON.stringify(callId))
  }).catch((err) => {
    reject(err);
    LogUtils.i(TAG, prefixLog + 'catch:hangUpCall : %s' + JSON.stringify(err))
  });
});
```

## 4. ServiceAbility 生命周期

### 4.1 初始化流程

```
应用启动
        │
        ▼
MyAbilityStage.onCreate()
        │
        │ 启用通知
        ▼
notification.enableNotification()
        │
        ▼
MainAbility.onCreate()
        │
        │ 加载页面
        ▼
ServiceAbility.onCreate()
        │
        │ 初始化 CallManagerService
        ▼
CallManagerService.getInstance().init()
```

### 4.2 连接流程

```
外部组件连接 Service
        │
        ▼
IPC Framework
        │
        ▼
ServiceAbility.onConnect(want)
        │
        │ 返回 Stub
        ▼
Stub.onRemoteRequest(code, data, reply, option)
```

### 4.3 核心代码

```typescript
// 文件: entry/src/main/ets/ServiceAbility/ServiceAbility.ts

onCreate(want): void {
  LogUtils.i(TAG, 'onCreate callUI service');
  this.callManagerService = CallManagerService.getInstance();
  this.callManagerService.init(this.context);
}

onConnect(want: Want): Stub {
  LogUtils.i(TAG, 'onConnect callUI service');
  let callData: DefaultCallData = new DefaultCallData();
  callData.accountNumber = want.parameters?.accountNumber;
  callData.videoState = want.parameters?.videoState;
  // ... 初始化通话数据
  this.callManagerService.getCallData(callData);
  CallManager.getInstance().setServiceConnected(true);
  return new Stub('ServiceAbility');
}

onDisconnect(): void {
  LogUtils.i(TAG, 'onDisconnect callUI service');
  CallManager.getInstance().setServiceConnected(false);
  this.callManagerService.onDisconnected();
}

onDestroy(): void {
  LogUtils.i(TAG, 'onDestroy callUI service');
  this.callManagerService.removeRegisterListener();
}
```

## 5. 事件发布流程

### 5.1 CommonEvent 发布

```
通话事件发生
        │
        ▼
CallServiceProxy.publish(data)
        │
        │ commonEvent.publish()
        ▼
@ohos.commonEvent.publish()
        │
        ▼
事件分发到订阅者
```

### 5.2 核心代码

```typescript
// 文件: entry/src/main/ets/model/CallServiceProxy.ets

public publish(data) {
  LogUtils.i(TAG, prefixLog + 'callui.event.callEvent publish')
  commonEvent.publish('callui.event.callEvent', {
    bundleName: 'com.ohos.callui',
    isOrdered: false,
    subscriberPermissions: ['ohos.permission.GET_TELEPHONY_STATE'],
    data: JSON.stringify(data)
  }, (res) => {
    LogUtils.i(TAG, prefixLog + 'callui.event.callEvent success')
  });
  LogUtils.i(TAG, prefixLog + 'callui.event.callEvent publish end')
}
```

## 6. 通知流程

### 6.1 来电通知

```
来电事件
        │
        ▼
NotificationManager.publishIncomingCallNotification()
        │
        │ notification.publish()
        ▼
@ohos.notification.publish()
        │
        ▼
系统通知栏显示
```

## 7. 数据流图

### 7.1 通话数据流

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  电话系统   │────▶│ Telephony   │────▶│ CallManager │
│             │     │   API       │     │             │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                                               ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  UI 页面    │◀────│ AppStorage  │◀────│CallDataManager│
│             │     │             │     │             │
└─────────────┘     └─────────────┘     └─────────────┘
```

### 7.2 状态同步流

```
┌─────────────────────────────┐
│     CallManager (单例)       │
│  - callData: 当前通话数据    │
│  - callList: 通话列表       │
│  - timer: 计时器            │
└─────────────┬───────────────┘
              │
    ┌─────────┼─────────┐
    ▼         ▼         ▼
┌───────┐ ┌───────┐ ┌───────┐
│ Call- │ │ Notif- │ │ 页面  │
│ Data- │ │ ication│ │ UI    │
│Manager│ │ Manager│ │       │
└───────┘ └───────┘ └───────┘
```

## 8. 相关文档

- [架构设计](02_Architecture.md)
- [API 参考](03_API_Reference.md)
- [模块详解](06_Module_Details.md)
