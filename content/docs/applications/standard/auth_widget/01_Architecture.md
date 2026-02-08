# 01_Architecture - 架构说明

## 整体架构

Authentication Widget 采用 **UI Extension + ArkTS/ArkUI** 架构，与 user_auth_framework 通过 IPC 进行通信。

**证据**: `README.md` 行 7-9

```
Authentication Widget ←→ user_auth_framework
         ↓
    UI Extension (ArkTS/ArkUI)
         ↓
    System Dialog
```

## 组件图

```
┌─────────────────────────────────────────────────────────────────┐
│                    user_auth_framework                           │
│  (认证请求发起、认证结果处理、凭据管理)                            │
└─────────────────────────────┬───────────────────────────────────┘
                              │ IPC (Want + Parameters)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                  UserAuthExtensionAbility                        │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ onCreate → onForeground → onSessionCreate → ...           │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────┬───────────────────────────────────┘
                              │ AppStorage + UIExtension
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        pages/Index                               │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ @Entry @Component struct Index                              │  │
│  │ ├── authType: UserAuthType[]                              │  │
│  │ ├── dialogType: DialogType                                 │  │
│  │ ├── windowModeType: string                                 │  │
│  │ └── cmdData: CmdType[]                                     │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────┬───────────────────────────────────┘
                    ┌────────┴────────┐
                    ▼                 ▼
     ┌──────────────────────┐  ┌──────────────────────┐
     │   DIALOG_BOX 模式    │  │   全屏模式           │
     │  ┌────────────────┐  │  │  ┌────────────────┐  │
     │  │ FaceAuth      │  │  │  │ CustomPassword │  │
     │  │ FingerprintAuth│  │  │  └────────────────┘  │
     │  │ PasswordAuth  │  │  │                     │
     │  └────────────────┘  │  │                     │
     └──────────────────────┘  └──────────────────────┘
```

## 数据流

### 认证流程数据流

```
1. user_auth_framework 发起认证请求
   ↓
2. 系统创建 UIExtension 会话，携带 Want 参数
   ↓
3. UserAuthAbility.onSessionCreate 解析参数
   ├── wantParams → AppStorage
   └── session → AppStorage
   ↓
4. Index.aboutToAppear 获取参数
   ├── 获取 wantParams
   ├── 获取 userAuthWidgetMgr
   └── 调用 handleAuthStart()
   ↓
5. 注册 command 回调，接收认证指令
   ↓
6. 根据认证类型显示对应 UI 组件
   ↓
7. 用户交互 → AuthUtils.sendNotice
   ↓
8. user_auth_framework 处理结果
```

**证据**: `Index.ets` 行 48-90, 171-196

### 关键数据接口

#### WantParams（输入参数）

**证据**: `Constants.ts` 行 172-181

```typescript
interface WantParams {
  widgetContextId: number;           // 认证实例上下文 ID
  type: string[];                     // 认证类型列表
  title: string;                      // 对话框标题
  pinSubType: string;                 // PIN 子类型
  navigationButtonText?: string;      // 导航按钮文本（可选）
  windowModeType: string;             // 窗口模式
  cmd: CmdType[];                     // 认证指令
  widgetContextIdStr?: string;        // 字符串格式上下文 ID
}
```

#### CmdData（指令负载）

**证据**: `Constants.ts` 行 149-157

```typescript
interface CmdData {
  type: string;                       // 认证类型 (pin/face/fingerprint)
  remainAttempts: number;              // 剩余尝试次数
  lockoutDuration: number;            // 锁定时长（毫秒）
  result: number;                    // 认证结果
  sensorInfo?: string;                // 传感器信息
  tipType?: number;                   // 提示类型
  tipInfo?: Uint8Array;              // 提示信息
}
```

## 线程模型

### UI 线程

**证据**: `UserAuthAbility.ts` 行 48-52

```
主线程 (ArkTS UI Thread)
├── onCreate → onForeground → onBackground → onDestroy
├── onSessionCreate (加载 UI 页面)
└── onSessionDestroy (终止会话)
```

### 工作线程模式

- **认证指令处理**: `userAuthWidgetMgr.on('command')` 回调在 UI 线程执行
- **PIN 输入**: `pinAuthManager.registerInputer()` 在 UI 线程注册
- **事件通知**: `AuthUtils.sendNotice()` 在 UI 线程调用

**证据**: `PasswordAuth.ets` 行 104-110

```typescript
pinAuthManager.registerInputer({
  onGetData: (authSubType, callback) => {
    const uint8PW = FuncUtils.getUint8PW(pinData);
    callback.onSetData(authSubType, uint8PW);
  }
});
```

## 关键时序

### 时序图 1：组件初始化

```
UserAuthExtensionAbility    Index          user_auth_framework
        │                    │                     │
        │ onCreate           │                     │
        │───────────────────>│                     │
        │                    │                     │
        │ onSessionCreate    │                     │
        │ (want + session)   │                     │
        │───────────────────>│                     │
        │                    │ getUserAuthWidgetMgr│
        │                    │────────────────────>│
        │                    │                     │
        │                    │ on('command')       │
        │                    │ <───────────────────│
        │                    │                     │
        │                    │ loadContent('pages/Index')
        │                    │                     │
        │                    │ handleAuthStart()   │
        │                    │────────────────────>│
        │                    │                     │
```

### 时序图 2：认证事件流程

```
Index/PasswordAuth    AuthUtils       user_auth_framework
        │                  │                   │
        │ sendNotice()     │                   │
        │─────────────────>│                   │
        │                  │ sendNotice()      │
        │                  │──────────────────>│
        │                  │                   │
        │                  │                   │ 认证结果
        │                  │                   │────────> user_auth_framework
        │                  │                   │
        │                  │ command 回调      │
        │                  │ <─────────────────│
        │                  │                   │
        │ UI 更新          │                   │
        │<─────────────────│                   │
```

**证据**: `AuthUtils.ts` 行 35-54

```typescript
sendNotice(cmd: string, type: Array<string>, tipCode: TipCode = TipCode.NORMAL): void {
  const eventData = {
    widgetContextId: this.widgetContextId,
    event: cmd,
    version: Constants.noticeVersion,
    payload: { type: type, tipCode: tipCode }
  };
  userAuth.sendNotice(userAuth.NoticeType.WIDGET_NOTICE, jsonEventData);
}
```

## 组件职责

| 组件 | 文件 | 职责 |
|------|------|------|
| UserAuthAbility | `extensionability/UserAuthAbility.ts` | 扩展能力生命周期管理、会话管理 |
| Index | `pages/Index.ets` | 主页面路由、认证类型分发 |
| FaceAuth | `pages/components/FaceAuth.ets` | 人脸认证 UI |
| FingerprintAuth | `pages/components/FingerprintAuth.ets` | 指纹认证 UI |
| PasswordAuth | `pages/components/PasswordAuth.ets` | 对话框模式密码 UI |
| CustomPassword | `pages/components/CustomPassword.ets` | 全屏密码 UI |
| PassWord | `common/components/PassWord.ets` | 任意长度密码输入 |
| SixPassword | `common/components/SixPassword.ets` | 六位固定密码输入 |
| NumkeyBoard | `common/components/NumkeyBoard.ets` | 数字键盘组件 |
| AuthUtils | `common/utils/AuthUtils.ts` | 认证事件通知 |
| LogUtils | `common/utils/LogUtils.ts` | 日志输出 |
| FuncUtils | `common/utils/FuncUtils.ts` | 工具函数 |
| Constants | `common/vm/Constants.ts` | 常量定义 |

## 相关文档

- [项目概览](00_Overview.md) - 项目定位与核心能力
- [认证框架 API](02_UserAuth_API.md) - 外部接口详细说明
- [安全风险评审](20_Security_Review.md) - 架构层面的安全考虑
