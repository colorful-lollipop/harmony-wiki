# 00. 项目概览

> 目的: 帮助新人快速理解 ScreenLock 项目的定位、能力和运行环境  
> 适用范围: 所有开发者、架构师、安全审计人员

---

## 1. 项目定位

### 1.1 基本信息

| 属性 | 值 | 证据位置 |
|------|-----|----------|
| 应用名称 | ScreenLock（锁屏应用） | `README_zh.md:1` |
| BundleName | `com.ohos.systemui` | `AppScope/app.json5:3` |
| 应用类型 | 系统预置应用 | `README_zh.md:6` |
| Vendor | ohos | `AppScope/app.json5:4` |
| 版本号 | 1.0.0 | `AppScope/app.json5:5` |
| 版本代码 | 1000000 | `AppScope/app.json5:6` |

### 1.2 项目归属

- **代码仓**: `applications/standard/screenlock`
- **上级分类**: 系统应用 (SystemUI)
- **所属系统**: OpenHarmony 标准系统

### 1.3 一句话描述

> OpenHarmony 标准系统的预置锁屏应用，提供滑动/密码/图案等多种解锁方式，集成通知、时钟、电池等状态显示。

---

## 2. 核心能力

### 2.1 锁屏能力

| 能力 | 说明 | 实现位置 |
|------|------|----------|
| 滑动解锁 | 基础滑动解锁界面 | `product/phone/pages/slidescreenlock.ets` |
| 数字密码 | 数字键盘输入（0-9） | `product/phone/pages/digitalPassword.ets` |
| 混合密码 | 字母+数字混合输入 | `product/phone/pages/mixedPassword.ets` |
| 图案密码 | 手势图案解锁 | `product/phone/pages/customPassword.ets` |
| 人脸解锁 | TODO(需确认): 人脸解锁集成 | - |

### 2.2 信息展示

| 组件 | 功能 | 模块 |
|------|------|------|
| 时钟组件 | 显示当前时间 | `features/clockcomponent` |
| 日期时间组件 | 显示日期和星期 | `features/datetimecomponent` |
| 电池组件 | 显示电量状态 | `features/batterycomponent` |
| 信号组件 | 显示移动网络信号 | `features/signalcomponent` |
| WiFi组件 | 显示WiFi连接状态 | `features/wificomponent` |
| 通知服务 | 显示系统通知 | `features/notificationservice` |
| 快捷开关 | 快捷设置开关 | `features/shortcutcomponent` |
| 壁纸组件 | 锁屏壁纸显示 | `features/wallpapercomponent` |

### 2.3 系统交互

| 交互场景 | 系统服务 | 证据 |
|----------|----------|------|
| 锁屏状态管理 | @ohos.screenLock | `features/screenlock/model/screenLockModel.ts:17` |
| 屏幕事件监听 | @ohos.commonEvent | `common/ScreenLockManager.ts:16` |
| 窗口管理 | @ohos.window | `common/WindowManager.ts:17` |
| 显示信息获取 | @ohos.display | `phone/ServiceExtAbility.ts:18` |
| 通知管理 | @ohos.notification | `common/notificationManager.ts:17` |
| 账户管理 | @ohos.account.osAccount | `common/SwitchUserManager.ts:16` |

---

## 3. 运行环境

### 3.1 API版本要求

| 属性 | 值 | 证据 |
|------|-----|------|
| 最低API版本 | 8 | `AppScope/app.json5:10` |
| 目标API版本 | 9 | `AppScope/app.json5:11` |
| 编译SDK版本 | 23 | `build-profile.json5:7` |
| 兼容SDK版本 | 23 | `build-profile.json5:8` |

### 3.2 支持的设备类型

| 设备类型 | 模块 | 证据 |
|----------|------|------|
| Phone | phone, entry | `entry/module.json5:9`, `phone/module.json5:9` |
| Tablet | entry | `entry/module.json5:10` |
| PC | pc | `build-profile.json5:27` |

### 3.3 技术栈

| 技术 | 说明 | 证据 |
|------|------|------|
| 编程语言 | ArkTS (TypeScript超集) | 所有`.ets`文件 |
| UI框架 | ArkUI (声明式UI) | `@Entry`, `@Component`装饰器 |
| 应用模型 | Stage模型 | `module.json5`中的`extensionAbilities` |
| 原生扩展 | **无N-API** | 未找到`.cpp`/`.c`文件 |
| 构建工具 | Hvigor | `hvigorfile.js`, `build-profile.json5` |

---

## 4. 关键概念

### 4.1 锁屏模式 (LockStyleMode)

```typescript
// features/screenlock/model/screenlockStyle
enum LockStyleMode {
  SlideScreenLock,   // 滑动解锁
  JournalScreenLock, // 日志锁屏
  CustomScreenLock   // 自定义锁屏（密码/图案）
}
```

### 4.2 窗口类型

| 窗口类型 | 值 | 用途 |
|----------|-----|------|
| TYPE_KEYGUARD | - | 锁屏窗口类型 |
| SystemUi_StatusBar | 2108 | 状态栏 |
| SystemUi_NavigationBar | 2112 | 导航栏 |
| SystemUi_DropdownPanel | 2109 | 下拉面板 |

### 4.3 核心事件

| 事件名称 | 常量定义 | 用途 |
|----------|----------|------|
| 屏幕状态变化 | `SCREEN_CHANGE_EVENT` | 监听屏幕开关 |
| 窗口显示隐藏 | `WINDOW_SHOW_HIDE_EVENT` | 窗口状态变更 |
| 窗口大小调整 | `WINDOW_RESIZE_EVENT` | 窗口尺寸变更 |

### 4.4 Ability角色

| Ability | 类型 | 职责 |
|---------|------|------|
| MainAbility | UIAbility | Entry模块入口，应用启动 |
| ServiceExtAbility | ServiceExtensionAbility | 锁屏服务核心，窗口管理 |
| AbilityStage | AbilityStage | Ability生命周期管理 |

---

## 5. 项目边界

### 5.1 职责范围内

- ✅ 锁屏界面渲染与交互
- ✅ 解锁方式（滑动/数字/混合/图案）
- ✅ 系统状态展示（时间/电量/信号等）
- ✅ 通知展示与管理
- ✅ 锁屏窗口生命周期管理
- ✅ 用户账户状态监听

### 5.2 职责范围外

- ❌ 系统级锁屏策略决策（由系统服务控制）
- ❌ 密码验证逻辑（委托给系统安全服务）
- ❌ 生物识别（人脸/指纹）底层实现
- ❌ 壁纸设置与管理（仅展示）
- ❌ 通知内容生成（仅展示系统通知）

### 5.3 与系统服务的边界

```
┌─────────────────────────────────────────────────────────────┐
│                     ScreenLock 应用                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  锁屏界面    │  │  解锁交互    │  │    状态展示组件      │ │
│  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘ │
└─────────┼────────────────┼────────────────────┼────────────┘
          │                │                    │
          ▼                ▼                    ▼
┌─────────────────────────────────────────────────────────────┐
│                      OpenHarmony 系统服务                    │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────────┐ │
│  │ screenLock│ │ window   │ │notification│ │ account.osAccount│ │
│  └──────────┘ └──────────┘ └──────────┘ └────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## 6. 快速开始

### 6.1 代码入口

| 场景 | 入口文件 | 说明 |
|------|----------|------|
| 应用启动 | `entry/src/main/ets/MainAbility/MainAbility.ts` | Entry Ability |
| 锁屏服务 | `product/phone/src/main/ets/ServiceExtAbility/ServiceExtAbility.ts` | 服务扩展 |
| 锁屏页面 | `product/phone/src/main/ets/pages/index.ets` | 主页面 |
| 滑动解锁 | `product/phone/src/main/ets/pages/slidescreenlock.ets` | 滑动解锁UI |

### 6.2 关键配置

| 配置文件 | 用途 |
|----------|------|
| `AppScope/app.json5` | 应用级配置（bundleName、版本） |
| `build-profile.json5` | 模块定义、SDK版本 |
| `phone/src/main/module.json5` | Phone模块权限、Ability声明 |
| `common/index.ts` | 公共模块导出 |

---

## 7. 相关链接

- [目录结构详情](02_Directory_Structure.md)
- [架构设计](03_Architecture.md)
- [安全风险分析](08_Security.md)
- [完整导航](SUMMARY.md)

---

*关键结论: 本项目是OpenHarmony标准系统的预置锁屏应用，纯ArkTS实现，无原生代码，使用21个系统权限与多个系统服务交互。*
