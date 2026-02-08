# 模块详解

## 1. 模块总览

### 1.1 模块列表

| 模块 | 类型 | 职责 | 依赖 |
|-----|------|-----|------|
| **entry** | entry | 通话主应用 | telephony APIs, common |
| **mobiledatasettings** | feature | 移动数据设置 | telephony APIs, common |
| **common** | har | 公共组件库 | 无 |

### 1.2 模块关系

```
┌─────────────────────────────────────────────────────────┐
│                      entry (entry)                       │
│  ┌─────────────────────────────────────────────────┐    │
│  │  主功能: 语音/视频通话、通话管理、来电处理        │    │
│  │  入口: MainAbility, ServiceAbility              │    │
│  │  依赖: @ohos.telephony.*, common               │    │
│  └─────────────────────────────────────────────────┘    │
│                          │                              │
│          ┌───────────────┴───────────────┐              │
│          ▼                               ▼              │
│  ┌─────────────────────┐   ┌─────────────────────┐      │
│  │ mobiledatasettings  │   │      common         │      │
│  │ (feature)           │   │      (har)          │      │
│  │ - APN 配置          │   │ - 公共组件          │      │
│  │ - 网络模式设置       │   │ - 常量定义          │      │
│  │ - SIM 卡信息        │   │ - 工具类            │      │
│  └─────────────────────┘   └─────────────────────┘      │
└─────────────────────────────────────────────────────────┘
```

## 2. entry 模块详解

### 2.1 模块结构

```
entry/src/main/
├── ets/
│   ├── Application/
│   │   └── MyAbilityStage.ts    # 应用生命周期
│   │
│   ├── MainAbility/
│   │   └── MainAbility.ts       # UI Ability
│   │
│   ├── ServiceAbility/
│   │   ├── ServiceAbility.ts    # Service Ability (IPC)
│   │   └── TelephonyApi.ets     # Telephony API 封装
│   │
│   ├── model/
│   │   ├── CallManager.ts       # 通话管理器 (核心)
│   │   ├── CallServiceProxy.ts  # 通话服务代理
│   │   ├── CallDataManager.ts   # 通话数据管理
│   │   ├── CallStateManager.ts  # 通话状态管理
│   │   └── NotificationManager.ts# 通知管理
│   │
│   ├── common/
│   │   ├── components/          # UI 组件
│   │   │   ├── CallList.ets     # 通话列表
│   │   │   ├── ContactCard.ets  # 联系人卡片
│   │   │   ├── Keyboard.ets     # 拨号键盘
│   │   │   ├── IncomingCom.ets  # 来电弹窗
│   │   │   └── ...
│   │   │
│   │   ├── constant/            # 常量
│   │   │   ├── CallStateConst.ts
│   │   │   ├── CallTypeConst.ts
│   │   │   └── ...
│   │   │
│   │   ├── utils/               # 工具
│   │   │   ├── LogUtils.ts
│   │   │   ├── CallUtils.ts
│   │   │   └── ...
│   │   │
│   │   └── struct/              # 数据结构
│   │       ├── TypeUtils.ts
│   │       └── CallListStruct.ts
│   │
│   └── pages/
│       └── index.ets             # 主页面
│
├── resources/                    # 资源文件
└── module.json                   # 模块配置
```

### 2.2 核心类说明

#### 2.2.1 CallManager

**文件**: `entry/src/main/ets/model/CallManager.ets`

**职责**: 通话管理器，单例模式，统一管理通话状态

**关键方法**:

```typescript
export default class CallManager {
  // 单例获取
  static getInstance(): CallManager

  // 初始化
  init(ctx: any): void

  // 更新通话数据 (核心方法)
  update(callData: DefaultCallData): Promise<void>

  // 设置服务连接状态
  setServiceConnected(isConnected: boolean): void

  // 计时器管理
  openTimer(): void
  clearTimer(): void

  // 更新通话时长列表
  updateCallTimeList(): void
}
```

**状态管理**:

```typescript
private callData: DefaultCallData = new DefaultCallData();
private callList: Array<CallListStruct> = [];
private timer: number = null;
private callTimeList = [];
private isServiceConnected: boolean = false;
```

#### 2.2.2 CallServiceProxy

**文件**: `entry/src/main/ets/model/CallServiceProxy.ets`

**职责**: FA 侧通话服务代理，封装 telephony.call API

**关键方法**:

```typescript
export default class CallServiceProxy {
  // 通话控制
  dialCall(phoneNumber, accountId?, videoState?, dialScene?): Promise<boolean>
  acceptCall(callId): void
  rejectCall(callId, isSendSms?, msg?): Promise<void>
  hangUpCall(callId): Promise<boolean>
  holdCall(callId): Promise<boolean>
  unHoldCall(callId): Promise<boolean>
  switchCall(callId): Promise<boolean>

  // 状态监听
  registerCallStateCallback(callback: Function): void
  unRegisterCallStateCallback(): void
  registerCallEventCallback(): void
  unRegisterCallEventCallback(): void

  // DTMF
  startDTMF(callId?, str?): void
  stopDTMF(callId?): void

  // 会议
  combineConference(callId): void

  // 事件发布
  publish(data): void
}
```

#### 2.2.3 ServiceAbility

**文件**: `entry/src/main/ets/ServiceAbility/ServiceAbility.ts`

**职责**: 后台服务，处理 IPC 通信

**生命周期**:

```typescript
export default class ServiceAbility extends ServiceExtension {
  // 创建时初始化
  onCreate(want): void {
    this.callManagerService = CallManagerService.getInstance();
    this.callManagerService.init(this.context);
  }

  // 连接时返回 IPC Stub
  onConnect(want): Stub {
    return new Stub('ServiceAbility');
  }

  // 断开连接
  onDisconnect(): void {
    CallManager.getInstance().setServiceConnected(false);
  }

  // 请求处理
  onRequest(want, startId): void {}

  // 销毁
  onDestroy(): void {
    this.callManagerService.removeRegisterListener();
  }
}
```

#### 2.2.4 TelephonyApi

**文件**: `entry/src/main/ets/ServiceAbility/TelephonyApi.ets`

**职责**: Service 侧 telephony.call API 封装

**关键方法**:

```typescript
export default class TelephonyApi {
  // 通话状态监听
  registerCallStateCallback(callBack): void
  unRegisterCallStateCallback(): void

  // 通话控制
  acceptCall(callId): void
  rejectCall(callId, isSendSms?, msg?): void
  hangUpCall(callId): Promise<boolean>
}
```

### 2.3 页面结构

```
entry/src/main/ets/pages/
└── index.ets                      # 主页面
    │
    ├── @Component                # 主组件
    │   ├── build()              # 构建 UI
    │   ├── onShow()             # 显示回调
    │   ├── onHide()             # 隐藏回调
    │   └── aboutToDisappear()   # 销毁前清理
    │
    ├── @State                   # 状态变量
    │   ├── callList: []         # 通话列表
    │   ├── callState: number    # 通话状态
    │   └── ...
    │
    └── @Builder                 # 构建器
        ├── dialPad()            # 拨号键盘
        ├── callListView()       # 通话列表
        └── contactCard()        # 联系人卡片
```

## 3. mobiledatasettings 模块详解

### 3.1 模块结构

```
mobiledatasettings/src/main/
├── ets/
│   ├── Application/
│   │   └── MyAbilityStage.ts    # 应用生命周期
│   │
│   ├── MainAbility/
│   │   └── MainAbility.ts       # UI Ability
│   │
│   ├── pages/
│   │   ├── index.ets            # 主页面
│   │   ├── apnList.ets          # APN 列表
│   │   ├── apnDetail.ets        # APN 详情
│   │   └── networkStand.ets      # 网络模式
│   │
│   ├── model/
│   │   ├── mobileDataStatus.ets  # 移动数据状态
│   │   ├── apnDataStorage.ets    # APN 数据存储
│   │   └── registerSimStateApi.ets# SIM 状态注册
│   │
│   └── common/
│       ├── model/               # API 封装
│       │   ├── setPreferredNetworkApi.ets
│       │   ├── getSimStateApi.ets
│       │   └── ...
│       │
│       ├── constant/            # 常量
│       │   ├── apnData.ets
│       │   ├── apnItemInfo.ets
│       │   └── ...
│       │
│       └── components/          # 组件
│           ├── apnComponent.ets
│           ├── networkStandItem.ets
│           └── ...
│
└── resources/
```

### 3.2 页面功能

#### 3.2.1 index.ets - 移动数据主页面

```typescript
// 功能: 显示移动数据开关和设置入口
// 依赖: @ohos.telephony.radio, @ohos.telephony.sim
```

#### 3.2.2 apnList.ets - APN 列表

```typescript
// 功能: 显示 APN 列表，支持添加/编辑/删除
// 依赖: @ohos.telephony.sim, preferences
```

#### 3.2.3 networkStand.ets - 网络模式设置

```typescript
// 功能: 设置首选网络模式 (5G/4G/3G/2G/自动)
// 依赖: @ohos.telephony.radio
```

### 3.3 关键 API

#### 3.3.1 网络模式控制

```typescript
// 文件: mobiledatasettings/src/main/ets/common/model/getPreferredNetworkModeApi.ets

export default class GetPreferredNetworkModeApi {
  getNetworkMode(): NetworkMode {
    radio.getPreferredNetworkMode(slotId, (err, mode) => {
      return mode;
    });
  }
}
```

#### 3.3.2 SIM 状态

```typescript
// 文件: mobiledatasettings/src/main/ets/common/model/getSimStateApi.ets

export default class GetSimStateApi {
  getSimState(slotId: number): SimState {
    return sim.getSimState(slotId, (err, state) => {
      return state;
    });
  }
}
```

## 4. common 模块详解

### 4.1 模块结构

```
common/src/main/
├── ets/
│   └── components/
│       └── MainPage.ets         # 公共页面组件
│
└── module.json5                  # 模块配置
```

### 4.2 导出内容

| 类型 | 导出项 | 用途 |
|-----|-------|------|
| 组件 | MainPage | 公共页面组件 |
| 资源 | - | (HAR 不包含资源) |

### 4.3 使用方式

```typescript
// 在 entry 或 mobiledatasettings 中使用
import { MainPage } from '@ohos/common';
```

## 5. 依赖关系

### 5.1 模块间依赖

```
entry (entry)
├── imports: common               # HAR 依赖
│
├── usesSystemApi:
│   ├── @ohos.telephony.call    # 通话 API
│   ├── @ohos.telephony.sim     # SIM API
│   ├── @ohos.telephony.radio   # 无线 API
│   ├── @ohos.telephony.sms     # 短信 API
│   ├── @ohos.app.ability       # Ability 框架
│   ├── @ohos.rpc               # IPC
│   ├── @ohos.commonEvent       # 公共事件
│   └── @ohos.notification      # 通知

mobiledatasettings (feature)
├── imports: common              # HAR 依赖
│
└── usesSystemApi:
    ├── @ohos.telephony.radio   # 无线 API
    ├── @ohos.telephony.sim     # SIM API
    └── @ohos.data.preferences  # 偏好设置

common (har)
└── 无外部依赖
```

### 5.2 循环依赖检查

**检查结果**: ✅ 无循环依赖

- `entry` → `common` (单向)
- `mobiledatasettings` → `common` (单向)

## 6. 稳定性评估

### 6.1 内部 API

| API | 稳定性 | 说明 |
|-----|-------|------|
| `CallManager.*` | 稳定 | 内部单例，接口稳定 |
| `CallServiceProxy.*` | 稳定 | 对外接口，长期维护 |
| `ServiceAbility.*` | 稳定 | 系统框架接口 |
| `TelephonyApi.*` | 稳定 | Service 侧接口 |

### 6.2 系统 API

| API | 稳定性 | 来源 |
|-----|-------|------|
| `@ohos.telephony.call` | 系统 API | OpenHarmony SDK |
| `@ohos.telephony.sim` | 系统 API | OpenHarmony SDK |
| `@ohos.rpc` | 系统 API | OpenHarmony SDK |

## 7. 扩展点

### 7.1 可替换组件

| 组件 | 替换方式 | 影响范围 |
|-----|---------|---------|
| 通话列表 UI | 替换 CallList 组件 | UI 层 |
| 联系人显示 | 替换 ContactCard 组件 | UI 层 |
| 拨号键盘 | 替换 Keyboard 组件 | UI 层 |

### 7.2 扩展接口

```typescript
// 扩展通话管理器
interface CallManagerExtension {
  // 通话前验证
  beforeDial(phoneNumber: string): boolean;
  // 通话后处理
  afterCallEnd(callData: DefaultCallData): void;
  // 自定义状态处理
  handleCallState(state: number): void;
}

// 使用扩展
CallManager.getInstance().registerExtension(extension);
```

## 8. 相关文档

- [项目概述](01_Overview.md)
- [架构设计](02_Architecture.md)
- [API 参考](03_API_Reference.md)
- [构建指南](04_Build_Guide.md)
- [安全评审](05_Security_Review.md)
