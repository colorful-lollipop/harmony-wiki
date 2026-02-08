# 03_Components_API - 组件 API

## 概述

本文档描述 Authentication Widget 的内部组件接口，包括 UI 组件和工具类。

## 组件概览

```
ets/
├── extensionability/
│   └── UserAuthAbility.ts     # 扩展能力入口
├── common/
│   ├── components/
│   │   ├── PassWord.ets       # 密码输入组件
│   │   ├── SixPassword.ets    # 六位密码组件
│   │   ├── NumkeyBoard.ets    # 数字键盘组件
│   │   └── FullScreen.ets     # 全屏组件
│   ├── utils/
│   │   ├── LogUtils.ts       # 日志工具
│   │   ├── AuthUtils.ts      # 认证工具
│   │   ├── FuncUtils.ts      # 函数工具
│   │   ├── TimeUtils.ts      # 时间工具
│   │   └── WindowPrivacyUtils.ts # 窗口隐私工具
│   ├── vm/
│   │   └── Constants.ts      # 常量定义
│   └── module/
│       └── DialogType.ts    # 对话框类型
└── pages/
    ├── Index.ets             # 主页面
    └── components/
        ├── FaceAuth.ets      # 人脸认证
        ├── FingerprintAuth.ets # 指纹认证
        ├── PasswordAuth.ets  # 密码认证
        └── CustomPassword.ets # 自定义密码
```

## UI 组件接口

### UserAuthAbility

**证据**: `UserAuthAbility.ts` 行 29-71

| 方法 | 参数 | 返回值 | 描述 |
|------|------|--------|------|
| `onCreate()` | void | void | 扩展能力创建 |
| `onForeground()` | void | void | 切换到前台 |
| `onBackground()` | void | void | 切换到后台 |
| `onDestroy()` | void | Promise\<void\> | 销毁 |
| `onSessionCreate(want, session)` | Want, UIExtensionContentSession | void | 会话创建 |
| `onSessionDestroy(session)` | UIExtensionContentSession | void | 会话销毁 |

**生命周期**: `onCreate` → `onForeground` → `onSessionCreate` → `onSessionDestroy` → `onBackground` → `onDestroy`

### Index

**证据**: `Index.ets` 行 36-320

| 属性 | 类型 | 描述 |
|------|------|------|
| `@State authType` | UserAuthType[] | 认证类型 |
| `@State type` | string[] | 认证类型字符串 |
| `@State dialogType` | DialogType | 对话框类型 |
| `@State windowModeType` | string | 窗口模式 |
| `@State cmdData` | CmdType[] | 认证指令 |
| `@State pinSubType` | string | PIN 子类型 |
| `@State isLandscape` | boolean | 横屏模式 |

| 方法 | 描述 |
|------|------|
| `aboutToAppear()` | 组件即将显示 |
| `aboutToDisappear()` | 组件即将销毁 |
| `onScreenChange()` | 屏幕变化回调 |
| `getParams(result)` | 获取认证参数 |
| `handleAuthStart()` | 处理认证开始 |
| `handleLocked()` | 处理锁定状态 |

### PassWord

**证据**: `PassWord.ets` 行 21-50+

| 属性 | 类型 | 描述 |
|------|------|------|
| `@Link pinSubType` | string | PIN 子类型 |
| `@Link textValue` | string | 输入密码 |
| `@Link inputValue` | string | 提示信息 |
| `@Link isEdit` | boolean | 是否可编辑 |

| 方法 | 描述 |
|------|------|
| `clearPassword()` | 清空密码 |
| `aboutToDisappear()` | 清理敏感数据 |

### PasswordAuth

**证据**: `PasswordAuth.ets` 行 38-262

| 属性 | 类型 | 描述 |
|------|------|------|
| `@Link pinSubType` | string | PIN 子类型 |
| `@Link textValue` | string | 密码输入 |
| `@Link cmdData` | CmdType[] | 认证指令 |

| 方法 | 描述 |
|------|------|
| `aboutToAppear()` | 注册 PIN 输入器 |
| `aboutToDisappear()` | 注销输入器 |
| `handleCancel()` | 处理取消 |
| `countTime(freezingTime)` | 锁定倒计时 |

## 工具类接口

### AuthUtils

**证据**: `AuthUtils.ts` 行 24-54

| 方法 | 参数 | 返回值 | 描述 |
|------|------|--------|------|
| `getInstance()` | void | AuthUtils | 获取单例 |
| `sendNotice(cmd, type, tipCode?)` | string, string[], TipCode | void | 发送认证通知 |

### LogUtils

**证据**: `LogUtils.ts` 行 37-107

| 方法 | 参数 | 描述 |
|------|------|------|
| `debug(tag, format)` | string, string | Debug 日志 |
| `info(tag, format)` | string, string | Info 日志 |
| `warn(tag, format)` | string, string | Warning 日志 |
| `error(tag, format)` | string, string | Error 日志 |
| `fatal(tag, format)` | string, string | Fatal 日志 |

### Constants

**证据**: `Constants.ts` 行 17-208

| 分类 | 常量 | 值 | 描述 |
|------|------|-----|------|
| PIN 类型 | `pinSix` | `'PIN_SIX'` | 六位密码 |
| | `pinNumber` | `'PIN_NUMBER'` | 数字密码 |
| | `pinMixed` | `'PIN_MIXED'` | 混合密码 |
| 认证类型 | `noticeTypePin` | `'pin'` | PIN |
| | `noticeTypeFace` | `'face'` | 人脸 |
| | `noticeTypeFinger` | `'fingerprint'` | 指纹 |
| 事件 | `noticeEventCancel` | `'EVENT_AUTH_USER_CANCEL'` | 取消 |
| | `noticeEventWidgetLoaded` | `'EVENT_AUTH_WIDGET_LOADED'` | 已加载 |
| | `noticeEventWidgetReleased` | `'EVENT_AUTH_WIDGET_RELEASED'` | 已释放 |

## 接口定义

### WantParams

**证据**: `Constants.ts` 行 172-181

```typescript
interface WantParams {
  widgetContextId: number;           // 认证上下文 ID
  type: string[];                     // 认证类型列表
  title: string;                      // 标题
  pinSubType: string;                 // PIN 子类型
  navigationButtonText?: string;      // 导航按钮文本
  windowModeType: string;             // 窗口模式
  cmd: CmdType[];                    // 认证指令
  widgetContextIdStr?: string;        // 字符串格式 ID
}
```

### CmdType / CmdData

**证据**: `Constants.ts` 行 149-162

```typescript
interface CmdData {
  type: string;                       // 认证类型
  remainAttempts: number;              // 剩余次数
  lockoutDuration: number;            // 锁定时长
  result: number;                    // 认证结果
  sensorInfo?: string;                // 传感器信息
  tipType?: number;                   // 提示类型
  tipInfo?: Uint8Array;              // 提示信息
}

interface CmdType {
  event: string;                     // 事件
  payload: CmdData;                   // 数据负载
}
```

## 相关文档

- [概览](00_Overview.md) - 项目定位
- [架构说明](01_Architecture.md) - 组件关系图
- [认证框架 API](02_UserAuth_API.md) - 外部 API
- [GN 构建配置](10_GN_Build.md) - 构建配置
