# 00_Overview - 项目概览

## 项目定位

OpenHarmony Authentication Widget（用户认证组件）是一个**系统级 UI 扩展模块**，为设备提供统一的用户身份认证交互界面。

**证据**: `README.md` 行 5-8

> The Authentication Widget works with the User Authentication Framework (user_auth_framework) to provide a user authentication interaction interface when the user_auth_framework processes user authentication requests.

### 核心职责

1. **UI 展示**: 提供认证对话框（人脸、指纹、PIN 密码）
2. **模式切换**: 支持多种认证模式切换
3. **取消功能**: 提供取消认证按钮
4. **自定义支持**: 支持自定义显示信息（标题、提示等）

## 核心能力

### 支持的认证类型

**证据**: `Constants.ts` 行 42-44 和 `Index.ets` 行 39

| 认证类型 | 常量值 | 描述 |
|----------|--------|------|
| PIN | `UserAuthType.PIN` | 数字密码认证 |
| FINGERPRINT | `UserAuthType.FINGERPRINT` | 指纹认证 |
| FACE | `UserAuthType.FACE` | 人脸认证 |

### PIN 子类型

**证据**: `Constants.ts` 行 21-23

| 子类型 | 常量值 | 描述 |
|--------|--------|------|
| 纯数字 | `pinNumber` | 任意长度数字密码 |
| 六位数字 | `pinSix` | 固定六位数字密码 |
| 混合密码 | `pinMixed` | 数字+字母密码 |

### 窗口模式

**证据**: `module.json` 行 20-25 和 `Index.ets` 行 284-316

| 模式 | 窗口模式类型 | 描述 |
|------|-------------|------|
| 对话框 | `DIALOG_BOX` | 模态对话框，显示认证类型对应 UI |
| 全屏 | 其他值 | 全屏 PIN 密码输入界面 |

## 运行环境

### 系统要求

**证据**: `bundle.json` 行 16-18

```json
"adapted_system_type": ["standard"]
```

- **系统类型**: OpenHarmony Standard（标准版）
- **最低 API 版本**: 10
- **目标 API 版本**: 10
- **API 发布类型**: Beta5

### 硬件要求

**证据**: `module.json` 行 7-10

```json
"deviceTypes": ["default", "tablet"]
```

- **默认设备**: 支持标准设备
- **平板设备**: 支持平板设备

### 资源占用

**证据**: `bundle.json` 行 19-20

- **ROM**: 860KB
- **RAM**: 0KB（运行时按需分配）

## 关键概念

### UI Extension 机制

Authentication Widget 使用 **UI Extension** 技术实现系统级对话框：

**证据**: `UserAuthAbility.ts` 行 29, 48-52

```typescript
export default class UserAuthAbility extends UserAuthExtensionAbility {
  onSessionCreate(want: Want, session: UIExtensionContentSession): void {
    AppStorage.setOrCreate('wantParams', want?.parameters?.useriamCmdData);
    AppStorage.setOrCreate('session', session);
    (session as UIExtensionContentSession)?.loadContent('pages/Index');
  }
}
```

### Extension Ability 类型

**证据**: `module.json` 行 20

```json
"type": "sysDialog/userAuth"
```

- **类型**: 系统对话框 (sysDialog)
- **子类型**: 用户认证 (userAuth)
- **元数据**: commonDialog

### AppStorage 数据共享

**证据**: `UserAuthAbility.ts` 行 32, 50-51 和 `Index.ets` 行 26

组件间通过 `AppStorage` 进行数据共享：

| 存储键 | 类型 | 描述 |
|--------|------|------|
| `wantParams` | WantParams | 认证参数 |
| `session` | UIExtensionContentSession | UI 会话 |
| `context` | ExtensionContext | 扩展上下文 |
| `widgetContextId` | string | 认证实例上下文 ID |

## 模块依赖

### 内部模块

**证据**: 代码文件结构分析

```
ets/
├── extensionability/     # 扩展能力入口
├── common/              # 公共组件与工具
└── pages/              # 认证页面
```

### 系统 API 依赖

**证据**: `Index.ets` 和 `AuthUtils.ts` 导入分析

| 模块 | API | 用途 |
|------|-----|------|
| `@ohos.userIAM.userAuth` | getUserAuthWidgetMgr, sendNotice | 认证框架主接口 |
| `@ohos.app.ability.UserAuthExtensionAbility` | 生命周期回调 | 扩展能力基类 |
| `@ohos.app.ability.UIExtensionContentSession` | loadContent, terminateSelf | UI 会话管理 |
| `@ohos.account.osAccount.PINAuth` | registerInputer | PIN 认证输入器 |
| `@ohos.screen` | getAllScreens | 屏幕信息 |
| `@ohos.hilog` | 日志输出 | 调试日志 |

## 相关文档

- [架构说明](01_Architecture.md) - 详细组件图与数据流
- [认证框架 API](02_UserAuth_API.md) - 与 user_auth_framework 集成
- [GN 构建配置](10_GN_Build.md) - 构建 targets 与编译产物
- [安全风险评审](20_Security_Review.md) - 安全分析与建议
