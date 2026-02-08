# 02_UserAuth_API - 认证框架 API

## 概述

本文档描述 Authentication Widget 与 [useriam_user_auth_framework](https://gitee.com/openharmony/useriam_user_auth_framework) 的 API 集成方式。

**证据**: `README.md` 行 5-8

## 核心 API

### userAuth 模块

**证据**: `Index.ets` 行 17, 174 和 `AuthUtils.ts` 行 16, 48

```typescript
import userAuth from '@ohos.userIAM.userAuth';
```

#### getUserAuthWidgetMgr

**证据**: `Index.ets` 行 174

| 属性 | 值 |
|------|-----|
| API 名称 | `getUserAuthWidgetMgr` |
| 参数 | version: number |
| 返回值 | `userAuth.UserAuthWidgetMgr` |
| 线程 | UI 线程 |

**使用示例**:
```typescript
userAuthWidgetMgr = userAuth.getUserAuthWidgetMgr(Constants.userAuthWidgetMgrVersion);
```

#### sendNotice

**证据**: `AuthUtils.ts` 行 48

| 属性 | 值 |
|------|-----|
| API 名称 | `sendNotice` |
| 参数 1 | noticeType: `NoticeType` |
| 参数 2 | jsonEventData: string |
| 返回值 | void |
| 线程 | UI 线程 |

**使用示例**:
```typescript
userAuth.sendNotice(userAuth.NoticeType.WIDGET_NOTICE, jsonEventData);
```

#### NoticeType 枚举

**证据**: `AuthUtils.ts` 行 48

| 枚举值 | 描述 |
|--------|------|
| `WIDGET_NOTICE` | Widget 通知 |

### PINAuth 模块

**证据**: `PasswordAuth.ets` 行 16, 34, 102

```typescript
import account_osAccount from '@ohos.account.osAccount';
```

#### PINAuth 类

**证据**: `PasswordAuth.ets` 行 102

| 方法 | 描述 |
|------|------|
| `constructor()` | 创建 PINAuth 实例 |
| `registerInputer(inputer)` | 注册密码输入器 |
| `unregisterInputer()` | 注销密码输入器 |

**使用示例**:
```typescript
pinAuthManager = new account_osAccount.PINAuth();
pinAuthManager.registerInputer({
  onGetData: (authSubType, callback) => {
    const uint8PW = FuncUtils.getUint8PW(pinData);
    callback.onSetData(authSubType, uint8PW);
  }
});
```

## 参数接口

### WantParams

**证据**: `Constants.ts` 行 172-181

认证请求参数，由 user_auth_framework 通过 Want 传递。

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `widgetContextId` | number | 是 | 认证实例上下文 ID |
| `type` | string[] | 是 | 认证类型列表 |
| `title` | string | 是 | 对话框标题 |
| `pinSubType` | string | 是 | PIN 子类型 |
| `navigationButtonText?` | string | 否 | 导航按钮文本 |
| `windowModeType` | string | 是 | 窗口模式 |
| `cmd` | CmdType[] | 是 | 认证指令 |
| `widgetContextIdStr?` | string | 否 | 字符串格式上下文 ID |

### WidgetCommand

**证据**: `Constants.ts` 行 183-187

认证指令，由 user_auth_framework 通过 command 回调传递。

| 字段 | 类型 | 描述 |
|------|------|------|
| `cmd` | Array\<CmdType\> | 认证指令列表 |
| `pinSubType` | string | PIN 子类型 |
| `skipLockedBiometricAuth` | boolean | 跳过已锁定生物特征认证 |

### CmdType

**证据**: `Constants.ts` 行 159-162

| 字段 | 类型 | 描述 |
|------|------|------|
| `event` | string | 事件类型 |
| `payload` | CmdData | 指令负载 |

### CmdData

**证据**: `Constants.ts` 行 149-157

| 字段 | 类型 | 描述 |
|------|------|------|
| `type` | string | 认证类型 (pin/face/fingerprint) |
| `remainAttempts` | number | 剩余尝试次数 |
| `lockoutDuration` | number | 锁定时长（毫秒） |
| `result` | number | 认证结果 |
| `sensorInfo?` | string | 传感器信息（指纹） |
| `tipType?` | number | 提示类型 |
| `tipInfo?` | Uint8Array | 提示信息 |

### FingerPosition

**证据**: `Constants.ts` 行 141-147

| 字段 | 类型 | 描述 |
|------|------|------|
| `sensorType` | string | 传感器类型 |
| `udSensorCenterXInThousandth?` | number | 屏下传感器 X 坐标（千分比） |
| `udSensorCenterYInThousandth?` | number | 屏下传感器 Y 坐标（千分比） |
| `udSensorRadiusInPx?` | number | 屏下传感器半径（像素） |
| `outOfScreenSensorType?` | string | 屏幕外传感器类型 |

**传感器类型枚举**: `Constants.ts` 行 206-208

| 类型值 | 描述 |
|--------|------|
| `UNDER_SCREEN_SENSOR` | 屏下传感器 |
| `BOTH_SENSOR` | 双传感器 |
| `SensorType1` | 传感器类型 1 |

## 事件常量

### noticeEvent 常量

**证据**: `Constants.ts` 行 46-52

| 常量名 | 常量值 | 描述 |
|--------|--------|------|
| `noticeEventCancel` | `'EVENT_AUTH_USER_CANCEL'` | 用户取消 |
| `noticeEventInvalidParam` | `'EVENT_AUTH_WIDGET_PARA_INVALID'` | 参数无效 |
| `noticeEventWidgetLoaded` | `'EVENT_AUTH_WIDGET_LOADED'` | Widget 已加载 |
| `noticeEventWidgetReleased` | `'EVENT_AUTH_WIDGET_RELEASED'` | Widget 已释放 |
| `noticeEventUserNavigation` | `'EVENT_AUTH_USER_NAVIGATION'` | 用户导航 |
| `noticeEventProcessTerminate` | `'EVENT_AUTH_PROCESS_TERMINATE'` | 进程终止 |
| `noticeEventAuthSendTip` | `'EVENT_AUTH_SEND_TIP'` | 发送提示 |

### noticeType 常量

**证据**: `Constants.ts` 行 42-44

| 常量名 | 常量值 | 描述 |
|--------|--------|------|
| `noticeTypePin` | `'pin'` | PIN 认证 |
| `noticeTypeFace` | `'face'` | 人脸认证 |
| `noticeTypeFinger` | `'fingerprint'` | 指纹认证 |

## 认证类型

### UserAuthType 枚举

**证据**: `Index.ets` 行 39 和 `Constants.ts` 行 42-44

| 枚举值 | 描述 |
|--------|------|
| `PIN` | PIN 认证 |
| `FINGERPRINT` | 指纹认证 |
| `FACE` | 人脸认证 |

### PIN 子类型

**证据**: `Constants.ts` 行 21-23

| 常量名 | 常量值 | 描述 |
|--------|--------|------|
| `pinSix` | `'PIN_SIX'` | 六位数字密码 |
| `pinNumber` | `'PIN_NUMBER'` | 任意长度数字密码 |
| `pinMixed` | `'PIN_MIXED'` | 混合密码 |

### DialogType 枚举

**证据**: `DialogType.ts` 分析

| 枚举值 | 描述 |
|--------|------|
| `ALL` | 显示所有认证类型 |
| `PIN_ONLY` | 仅显示 PIN |
| `FINGERPRINT_ONLY` | 仅显示指纹 |
| `FACE_ONLY` | 仅显示人脸 |

## 调用链

### 认证指令接收

```
user_auth_framework (IPC)
         │
         ▼ (command 回调)
userAuthWidgetMgr.on('command', callback)
         │
         ▼
Index.handleAuthStart()
         │
         ├── 解析 WidgetCommand
         ├── 更新 cmdData 状态
         └── 更新 PIN 子类型
         │
         ▼
UI 组件 (FaceAuth/FingerprintAuth/PasswordAuth)
```

### 事件通知发送

```
UI 组件 (用户操作)
         │
         ▼
AuthUtils.sendNotice(event, type, tipCode)
         │
         ├── 构建 eventData JSON
         └── 调用 userAuth.sendNotice()
         │
         ▼
user_auth_framework (IPC)
         │
         ▼
认证框架处理结果
```

## 相关文档

- [概览](00_Overview.md) - 项目定位与运行环境
- [架构说明](01_Architecture.md) - 组件图与数据流
- [组件 API](03_Components_API.md) - 内部组件接口
- [安全风险评审](20_Security_Review.md) - API 安全考虑
