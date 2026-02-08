# API 参考

## 1. 系统 API 概述

本应用使用 OpenHarmony 系统提供的 telephony API 进行通话管理。

### 1.1 API 模块依赖

| 模块 | 包路径 | 用途 |
|-----|--------|-----|
| 通话 | `@ohos.telephony.call` | 语音/视频通话控制 |
| SIM | `@ohos.telephony.sim` | SIM 卡信息管理 |
| 无线 | `@ohos.telephony.radio` | 无线通信状态 |
| 短信 | `@ohos.telephony.sms` | 短信相关功能 |

## 2. Telephony Call API

### 2.1 API 清单

#### 2.1.1 拨号

```typescript
// 文件: entry/src/main/ets/model/CallServiceProxy.ets

/**
 * 拨打电话
 * @param phoneNumber  电话号码
 * @param accountId    账户 ID (默认 0)
 * @param videoState   视频状态 (0: 语音, 1: 视频)
 * @param dialScene    拨号场景 (默认 0)
 * @returns Promise<boolean>  是否成功发起
 */
dialCall(phoneNumber: string, accountId?: number, videoState?: number, dialScene?: number): Promise<boolean>
```

**使用示例**:
```typescript
callServiceProxy.dialCall('13800138000', 0, 0, 0)
  .then((success) => {
    console.log('拨号成功');
  })
  .catch((err) => {
    console.log('拨号失败:', err);
  });
```

#### 2.1.2 接听电话

```typescript
/**
 * 接听电话
 * @param callId 通话 ID
 * @returns Promise<void>
 */
acceptCall(callId: number): Promise<void>
```

#### 2.1.3 拒绝电话

```typescript
/**
 * 拒绝电话
 * @param callId      通话 ID
 * @param isSendSms   是否发送短信 (默认 false)
 * @param msg         拒接短信内容
 */
rejectCall(callId: number, isSendSms?: boolean, msg?: string): Promise<void>
```

#### 2.1.4 挂断电话

```typescript
/**
 * 挂断电话
 * @param callId 通话 ID
 * @returns Promise<boolean>
 */
hangUpCall(callId: number): Promise<boolean>
```

#### 2.1.5 保持通话

```typescript
/**
 * 保持通话
 * @param callId 通话 ID
 * @returns Promise<boolean>
 */
holdCall(callId: number): Promise<boolean>
```

#### 2.1.6 取消保持

```typescript
/**
 * 取消保持
 * @param callId 通话 ID
 * @returns Promise<boolean>
 */
unHoldCall(callId: number): Promise<boolean>
```

#### 2.1.7 切换通话

```typescript
/**
 * 切换通话 (在多个通话间切换)
 * @param callId 目标通话 ID
 * @returns Promise<boolean>
 */
switchCall(callId: number): Promise<boolean>
```

#### 2.1.8 静音控制

```typescript
/**
 * 静音
 */
setMuted(): Promise<void>

/**
 * 取消静音
 */
cancelMuted(): Promise<void>

/**
 * 静音响铃
 */
muteRinger(): void
```

#### 2.1.9 DTMF 信号

```typescript
/**
 * 发送 DTMF 按钮音
 * @param callId 通话 ID
 * @param str    DTMF 字符 (0-9, *, #)
 */
startDTMF(callId: number, str: string): Promise<void>

/**
 * 停止 DTMF
 */
stopDTMF(callId: number): Promise<void>
```

#### 2.1.10 会议通话

```typescript
/**
 * 合并为会议通话
 * @param callId 通话 ID
 */
combineConference(callId: number): Promise<void>
```

### 2.2 通话状态监听

```typescript
/**
 * 注册通话状态变化监听
 * @param callback 回调函数
 */
registerCallStateCallback(callback: (data: CallDetailsInfo) => void): void

/**
 * 取消注册
 */
unRegisterCallStateCallback(): void
```

**CallDetailsInfo 结构**:
```typescript
interface CallDetailsInfo {
  callState: number;      // 通话状态
  callId: number;         // 通话 ID
  callType: number;       // 通话类型
  accountId: number;      // 账户 ID
  accountNumber: string;  // 电话号码
  conferenceState: number;// 会议状态
  videoState: number;     // 视频状态
  startTime: number;      // 开始时间
  isECC: boolean;        // 是否紧急呼叫
}
```

### 2.3 通话状态常量

```typescript
// 文件: entry/src/main/ets/common/constant/CallStateConst.ts

const CallStateConst = {
  CALL_STATUS_ACTIVE: 0,      // 激活
  CALL_STATUS_HOLDING: 1,     // 保持
  CALL_STATUS_DIALING: 2,     // 拨号中
  CALL_STATUS_ALERTING: 3,   // 振铃中
  CALL_STATUS_INCOMING: 4,   // 来电
  CALL_STATUS_WAITING: 5,     // 等待
  CALL_STATUS_DISCONNECTED: 6,// 已断开
  CALL_STATUS_DISCONNECTING: 7// 断开中
};
```

## 3. Telephony SIM API

### 3.1 SIM 卡信息

```typescript
// 文件: mobiledatasettings/src/main/ets/pages/apnList.ets
import sim from '@ohos.telephony.sim';

// 获取 SIM 卡状态
sim.getSimState(slotId: number, callback: AsyncCallback<SimState>): void

// 获取 SIM 卡电话号码
sim.getSimTelephoneNumber(slotId: number, callback: AsyncCallback<string>): void

// 获取 SIM 卡 ICCID
sim.getSimIccId(slotId: number, callback: AsyncCallback<string>): void

// 获取运营商信息
sim.getOperatorName(slotId: number, callback: AsyncCallback<string>): void
```

## 4. Telephony Radio API

### 4.1 网络信息

```typescript
// 文件: mobiledatasettings/src/main/ets/pages/networkStand.ets
import radio from '@ohos.telephony.radio';

// 获取首选网络模式
radio.getPreferredNetworkMode(slotId: number, callback: AsyncCallback<NetworkMode>): void

// 设置首选网络模式
radio.setPreferredNetworkMode(slotId: number, networkMode: NetworkMode, callback: AsyncCallback<boolean>): void

// 获取网络注册信息
radio.getNetworkState(slotId: number, callback: AsyncCallback<NetworkState>): void
```

## 5. 应用内部 API

### 5.1 CallManager

```typescript
// 文件: entry/src/main/ets/model/CallManager.ets

export default class CallManager {
  // 单例获取
  static getInstance(): CallManager

  // 初始化
  init(ctx: any): void

  // 更新通话数据
  update(callData: DefaultCallData): Promise<void>

  // 设置服务连接状态
  setServiceConnected(isConnected: boolean): void

  // 打开通话计时器
  openTimer(): void

  // 清除计时器
  clearTimer(): void
}
```

### 5.2 CallServiceProxy

```typescript
// 文件: entry/src/main/ets/model/CallServiceProxy.ets

export default class CallServiceProxy {
  // 单例获取
  static getInstance(): CallServiceProxy

  // 拨打电话
  dialCall(phoneNumber: string, accountId?: number, videoState?: number, dialScene?: number): Promise<boolean>

  // 接听/拒绝/挂断
  acceptCall(callId: number): void
  rejectCall(callId: number, isSendSms?: boolean, msg?: string): void
  hangUpCall(callId: number): Promise<boolean>

  // 通话控制
  holdCall(callId: number): Promise<boolean>
  unHoldCall(callId: number): Promise<boolean>
  switchCall(callId: number): Promise<boolean>

  // 状态监听
  registerCallStateCallback(callback: Function): void
  unRegisterCallStateCallback(): void

  // 发布事件
  publish(data: Object): void
}
```

### 5.3 NotificationManager

```typescript
// 文件: entry/src/main/ets/model/NotificationManager.ets

export default class NotificationManager {
  // 发布来电通知
  publishIncomingCallNotification(callData: DefaultCallData): void

  // 取消通知
  cancelNotification(tag: string): void
}
```

## 6. 权限要求

### 6.1 API 权限映射

| API | 必需权限 | 敏感度 |
|-----|---------|-------|
| dialCall | `ohos.permission.PLACE_CALL` | 高 |
| answerCall | `ohos.permission.ANSWER_CALL` | 高 |
| rejectCall | `ohos.permission.REJECT_CALL` | 高 |
| hangUpCall | `ohos.permission.HANG_UP_CALL` | 高 |
| getCallState | `ohos.permission.GET_TELEPHONY_STATE` | 中 |
| registerCallback | `ohos.permission.GET_TELEPHONY_STATE` | 中 |

### 6.2 权限声明

```json
// entry/src/main/module.json
{
  "requestPermissions": [
    { "name": "ohos.permission.PLACE_CALL" },
    { "name": "ohos.permission.ANSWER_CALL" },
    { "name": "ohos.permission.GET_TELEPHONY_STATE" },
    { "name": "ohos.permission.SET_TELEPHONY_STATE" },
    { "name": "ohos.permission.READ_CONTACTS" },
    { "name": "ohos.permission.SEND_MESSAGES" }
  ]
}
```

## 7. 错误处理

### 7.1 常见错误码

| 错误码 | 含义 | 处理建议 |
|-------|------|---------|
| 401 | 参数错误 | 检查参数类型和范围 |
| 405 | 权限不足 | 检查权限声明和用户授权 |
| 463 | 通话忙 | 提示用户稍后重试 |
| 801 | 操作不支持 | 设备不支持该功能 |
| 8300001 | 内部错误 | 重试或重启应用 |

### 7.2 错误处理示例

```typescript
call.dialCall(phoneNumber).then((res) => {
  console.log('拨号成功');
}).catch((err) => {
  if (err.code === 401) {
    console.error('参数错误');
  } else if (err.code === 405) {
    console.error('权限不足');
  } else if (err.code === 463) {
    console.error('通话忙');
  } else {
    console.error('拨号失败:', err);
  }
});
```

## 8. 相关文档

- [项目概述](01_Overview.md)
- [架构设计](02_Architecture.md)
- [构建指南](04_Build_Guide.md)
- [安全评审](05_Security_Review.md)
