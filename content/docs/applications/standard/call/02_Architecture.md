# 架构设计

## 1. 整体架构

### 1.1 分层架构

```
┌────────────────────────────────────────────────────────────────┐
│                        应用层 (FA-UI)                           │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    pages/                                │  │
│  │  ├── index.ets         (主通话页面)                      │  │
│  │  └── dialog/           (通话相关弹窗)                     │  │
│  └──────────────────────────────────────────────────────────┘  │
│                           │                                    │
│                    ViewModel 通信                              │
│                           │                                    │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    model/                                │  │
│  │  ├── CallManager.ts     (通话管理器)                     │  │
│  │  ├── CallServiceProxy.ts (通话服务代理)                   │  │
│  │  └── ...                 (其他数据模型)                   │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│                      ServiceAbility 层                           │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │               ServiceAbility.ts                          │  │
│  │  ├── onCreate()         (服务创建)                       │  │
│  │  ├── onConnect()        (连接 IPC)                       │  │
│  │  ├── onDisconnect()     (断开连接)                       │  │
│  │  └── Stub (IPC 服务端)                                   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                           │                                    │
│                      IPC 通信 (rpc)                            │
│                           │                                    │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │               TelephonyApi.ets                           │  │
│  │  └── 封装 @ohos.telephony.call API                       │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────────┐
│                      系统能力层 (SA)                             │
│  ┌──────────────────┐  ┌──────────────────┐                  │
│  │ Telephony SA     │  │ Notification SA  │                  │
│  │ (通话子系统)      │  │ (通知子系统)      │                  │
│  └──────────────────┘  └──────────────────┘                  │
└────────────────────────────────────────────────────────────────┘
```

### 1.2 模块职责

| 模块 | 类型 | 职责 |
|-----|------|-----|
| **entry** | entry | 主应用模块，包含通话核心功能 |
| **mobiledatasettings** | feature | 移动数据设置功能模块 |
| **common** | har | 公共组件库，可被其他模块复用 |

## 2. 组件详解

### 2.1 应用入口组件

```
entry/src/main/ets/
├── Application/
│   └── MyAbilityStage.ts        ← 应用生命周期入口
│       └── onCreate()            启用通知，创建全局状态
│
├── MainAbility/
│   └── MainAbility.ts            ← UI Ability
│       ├── onCreate()            保存 Want 和 AbilityContext
│       ├── onWindowStageCreate() 加载主页面
│       └── onDestroy()           清理资源
│
└── ServiceAbility/
    ├── ServiceAbility.ts         ← Service Ability (IPC)
    │   ├── onCreate()           初始化 CallManagerService
    │   ├── onConnect()          建立 IPC 连接
    │   └── onDisconnect()       断开连接
    │
    └── TelephonyApi.ets          ← Telephony API 封装
        ├── registerCallStateCallback()  注册通话状态监听
        ├── acceptCall()         接听电话
        ├── rejectCall()         拒绝电话
        └── hangUpCall()         挂断电话
```

### 2.2 数据模型组件

```
entry/src/main/ets/model/
├── CallManager.ts               ← 通话管理器 (单例)
│   ├── getInstance()           获取实例
│   ├── init()                  初始化
│   ├── update()               更新通话数据
│   └── ...
│
├── CallServiceProxy.ts          ← FA 侧通话服务代理
│   ├── dialCall()              拨打电话
│   ├── acceptCall()            接听
│   ├── rejectCall()            拒绝
│   ├── hangUpCall()            挂断
│   ├── holdCall()              保持
│   ├── unHoldCall()            取消保持
│   ├── switchCall()            切换通话
│   ├── registerCallStateCallback()  注册状态监听
│   └── publish()               发布公共事件
│
├── CallDataManager.ts           ← 通话数据管理
├── CallStateManager.ts          ← 通话状态管理
└── NotificationManager.ts       ← 通知管理
```

### 2.3 公共组件

```
entry/src/main/ets/common/
├── components/
│   ├── CallList.ets            通话列表组件
│   ├── ContactCard.ets          联系人卡片
│   ├── IncomingCom.ets          来电弹窗
│   ├── Keyboard.ets             拨号键盘
│   └── ...
│
├── constant/
│   ├── CallStateConst.ts       通话状态常量
│   ├── CallTypeConst.ts         通话类型常量
│   └── ...
│
├── utils/
│   ├── LogUtils.ts              日志工具
│   ├── CallUtils.ts             通话工具
│   └── ...
│
└── struct/
    ├── TypeUtils.ts             类型工具
    └── CallListStruct.ts        通话列表结构
```

## 3. 通信机制

### 3.1 IPC 通信 (ServiceAbility)

```typescript
// 文件: entry/src/main/ets/ServiceAbility/ServiceAbility.ts

// IPC 服务端 (ServiceAbility 中)
class Stub extends rpc.RemoteObject {
  onRemoteRequest(code, date, reply, option): boolean {
    // 处理远程请求
    return true;
  }
}

// 建立 IPC 连接
onConnect(want: Want): Stub {
  return new Stub('ServiceAbility');
}
```

### 3.2 CommonEvent 事件通信

```typescript
// 文件: entry/src/main/ets/model/CallServiceProxy.ets

// 发布通话事件
commonEvent.publish('callui.event.callEvent', {
  bundleName: 'com.ohos.callui',
  isOrdered: false,
  subscriberPermissions: ['ohos.permission.GET_TELEPHONY_STATE'],
  data: JSON.stringify(data)
}, callback);
```

### 3.3 通话状态监听

```typescript
// 注册通话状态变化监听
call.on('callDetailsChange', (data) => {
  // data.callState - 通话状态
  // data.callId    - 通话 ID
  // data.callType  - 通话类型
});
```

## 4. 线程模型

### 4.1 UI 线程

- ArkUI 组件渲染
- 用户交互响应
- 页面跳转

### 4.2 Service 线程

- 后台通话服务
- IPC 消息处理
- Telephony API 调用

### 4.3 回调线程

```typescript
// 异步回调在系统线程池执行
call.on('callDetailsChange', (data) => {
  // 需要切换到 UI 线程更新界面
  getGlobalObject().calluiAbilityContext?.runUIFunction(() => {
    // UI 更新操作
  });
});
```

## 5. 生命周期

### 5.1 应用生命周期

```
MyAbilityStage
├── onCreate()
│   └── 启用通知: notification.enableNotification()
│
└── onDestroy()

MainAbility
├── onCreate(want, launchParam)
│   └── globalThis.abilityWant = want
│   └── globalThis.calluiAbilityContext = context
│
├── onWindowStageCreate(windowStage)
│   └── windowStage.loadContent('pages/index')
│
├── onWindowStageDestroy()
├── onForeground()
├── onBackground()
│
└── onDestroy()
    └── CallManager.clearTimer()
```

### 5.2 Service 生命周期

```
ServiceAbility
├── onCreate()
│   └── CallManagerService.getInstance().init()
│
├── onConnect(want)
│   └── 返回 Stub 对象，建立 IPC 连接
│
├── onDisconnect()
│   └── CallManager.setServiceConnected(false)
│
├── onRequest(want, startId)
│
└── onDestroy()
    └── CallManagerService.removeRegisterListener()
```

## 6. 数据流

### 6.1 拨打电话数据流

```
用户点击拨号
    │
    ▼
┌─────────────────┐
│ pages/index     │  UI 层
│ DialCall()      │
└────────┬────────┘
         │ CallServiceProxy.dialCall()
         ▼
┌─────────────────┐
│ @ohos.telephony │  Telephony SA
│ .call.dial()    │
└────────┬────────┘
         │ 系统调用
         ▼
    电话系统
```

### 6.2 来电通知数据流

```
电话系统
    │
    │ 通话状态变化
    ▼
┌─────────────────┐
│ Telephony SA    │
└────────┬────────┘
         │ call.on('callDetailsChange')
         ▼
┌─────────────────┐
│ TelephonyApi    │  ServiceAbility
│ registerCallback│
└────────┬────────┘
         │ IPC 回调
         ▼
┌─────────────────┐
│ CallManager     │  FA 侧
│ update()        │
└────────┬────────┘
         │ 状态更新
         ▼
┌─────────────────┐
│ 页面刷新        │  UI 层
│ AppStorage      │
└─────────────────┘
```

## 7. 架构特点

### 7.1 优势

1. **分层清晰**: UI、业务逻辑、系统集成分离
2. **单例管理**: CallManager 全局单例，统一状态管理
3. **IPC 解耦**: ServiceAbility 处理底层通信
4. **事件驱动**: CommonEvent 支持组件间通信

### 7.2 可改进点

1. **状态管理**: 可考虑引入更轻量的状态管理方案
2. **依赖注入**: 可引入 DI 框架简化组件依赖
3. **模块化**: 部分功能可进一步拆分为独立 Feature

## 8. 相关文档

- [项目概述](01_Overview.md)
- [API 参考](03_API_Reference.md)
- [构建指南](04_Build_Guide.md)
- [安全评审](05_Security_Review.md)
