# 02. 目录结构

> 目的: 详细说明项目目录结构、模块划分和文件职责  
> 适用范围: 所有开发者

---

## 1. 顶层目录概览

```
applications/standard/screenlock/
├── AppScope/                    # 应用全局配置
│   ├── app.json5               # 应用配置 (bundleName=com.ohos.systemui)
│   └── resources/              # 全局资源 (图标、字符串)
│
├── common/                      # 公共模块 (HAR - Harmony Archive)
│   └── src/main/ets/default/   # 通用工具类源码
│
├── entry/                       # 入口模块 (Entry类型)
│   └── src/main/ets/           # Entry Ability源码
│
├── features/                    # 功能组件 (9个HAR)
│   ├── batterycomponent/       # 电池组件
│   ├── clockcomponent/         # 时钟组件
│   ├── datetimecomponent/      # 日期时间组件
│   ├── noticeitem/             # 通知项组件
│   ├── screenlock/             # 锁屏核心组件
│   ├── shortcutcomponent/      # 快捷操作组件
│   ├── signalcomponent/        # 信号组件
│   ├── wallpapercomponent/     # 壁纸组件
│   └── wificomponent/          # WiFi组件
│
├── product/                     # 产品形态模块
│   ├── pc/                     # PC/平板端
│   └── phone/                  # 手机端
│
├── infra/                       # 基础设施配置
├── hvigor/                      # Hvigor构建配置
├── signature/                   # 签名配置
├── build-profile.json5          # 构建配置文件
└── oh-package.json5             # 包管理配置
```

---

## 2. 模块详细说明

### 2.1 AppScope - 应用全局配置

**路径**: `/AppScope/`

| 文件/目录 | 说明 |
|-----------|------|
| `app.json5` | 应用级配置，定义bundleName、版本、API版本等 |
| `resources/` | 全局资源目录，包含应用图标、应用名称等 |

**关键配置** (`app.json5`):
```json
{
  "app": {
    "bundleName": "com.ohos.systemui",
    "versionCode": 1000000,
    "versionName": "1.0.0",
    "minAPIVersion": 8,
    "targetAPIVersion": 9
  }
}
```

---

### 2.2 Common模块 - 通用工具

**路径**: `/common/`
**类型**: HAR (Harmony Archive)

```
common/
├── src/main/
│   ├── ets/default/            # 源码目录
│   │   ├── abilitymanager/     # Ability管理
│   │   │   ├── abilityManager.ts
│   │   │   ├── bundleManager.ts
│   │   │   ├── featureAbilityManager.ts
│   │   │   └── notificationManager.ts
│   │   ├── commonEvent/        # 公共事件
│   │   │   └── CommonEventManager.ts
│   │   ├── event/              # 事件系统
│   │   │   ├── EventBus.ts     # 事件总线
│   │   │   ├── EventManager.ts # 事件管理器
│   │   │   └── EventUtil.ts
│   │   ├── CheckEmptyUtils.ts
│   │   ├── Constants.ts        # 常量定义
│   │   ├── DateTimeCommon.ts
│   │   ├── Decorators.ts       # 装饰器
│   │   ├── Log.ts              # 日志工具
│   │   ├── LunarCalendar.ts    # 农历
│   │   ├── ReadConfigUtil.ts
│   │   ├── ScreenLockCommon.ts
│   │   ├── ScreenLockManager.ts # 屏幕管理
│   │   ├── SingleInstanceHelper.ts
│   │   ├── StyleConfiguration.ts
│   │   ├── StyleManager.ts     # 样式管理
│   │   ├── SwitchUserManager.ts # 用户切换
│   │   ├── TimeManager.ts      # 时间管理
│   │   ├── Trace.ts            # 性能追踪
│   │   └── WindowManager.ts    # 窗口管理
│   └── resources/              # 资源文件
├── index.ts                    # 模块导出入口
└── oh-package.json5            # 模块包配置
```

**核心类职责**:

| 类名 | 文件路径 | 职责 |
|------|----------|------|
| EventBus | `event/EventBus.ts` | 类型安全的事件总线实现 |
| EventManager | `event/EventManager.ts` | 全局事件管理器 |
| WindowManager | `WindowManager.ts` | 系统窗口管理 |
| ScreenLockManager | `ScreenLockManager.ts` | 屏幕状态监听 |
| AbilityManager | `abilitymanager/abilityManager.ts` | Ability上下文管理 |
| Log | `Log.ts` | 日志工具 |
| TimeManager | `TimeManager.ts` | 时间管理 |
| SwitchUserManager | `SwitchUserManager.ts` | 用户切换管理 |

---

### 2.3 Entry模块 - 应用入口

**路径**: `/entry/`
**类型**: Entry

```
entry/
└── src/main/
    ├── ets/
    │   ├── Application/
    │   │   └── AbilityStage.ts    # AbilityStage
    │   ├── MainAbility/
    │   │   └── MainAbility.ts     # 主Ability
    │   └── pages/
    │       └── index.ets          # 入口页面
    ├── module.json5               # 模块配置
    └── resources/                 # 资源文件
```

**核心文件**:

| 文件 | 职责 |
|------|------|
| `MainAbility.ts` | Entry入口Ability，处理应用生命周期 |
| `AbilityStage.ts` | AbilityStage初始化 |
| `module.json5` | Entry模块配置，声明MainAbility |

---

### 2.4 Phone模块 - 手机端产品

**路径**: `/product/phone/`
**类型**: Feature

```
product/phone/
└── src/main/
    ├── ets/
    │   ├── Application/
    │   │   └── AbilityStage.ts           # Phone AbilityStage
    │   ├── ServiceExtAbility/
    │   │   └── ServiceExtAbility.ts      # 锁屏服务扩展
    │   ├── pages/                        # 锁屏页面
    │   │   ├── index.ets                 # 主页面
    │   │   ├── slidescreenlock.ets      # 滑动锁屏
    │   │   ├── digitalPassword.ets      # 数字密码
    │   │   ├── mixedPassword.ets        # 混合密码
    │   │   ├── customPassword.ets       # 图案密码
    │   │   ├── customscreenlock.ets     # 自定义锁屏
    │   │   └── journalscreenlock.ets    # 日志锁屏
    │   ├── vm/                           # ViewModel
    │   │   ├── indexViewModel.ts
    │   │   └── slideScreenLockViewModel.ts
    │   └── common/                       # Phone通用
    │       ├── StyleManager.ts
    │       └── constants.ts
    ├── module.json5                      # 模块配置
    └── resources/                        # 资源
```

**核心文件**:

| 文件 | 职责 |
|------|------|
| `ServiceExtAbility.ts` | 核心服务，创建锁屏窗口，初始化状态栏 |
| `pages/index.ets` | 锁屏主页面，根据模式显示不同UI |
| `pages/slidescreenlock.ets` | 滑动解锁界面 |
| `pages/digitalPassword.ets` | 数字密码界面 |
| `pages/mixedPassword.ets` | 混合密码界面 |
| `pages/customPassword.ets` | 图案密码界面 |

**模块配置** (`module.json5`):
- 声明21个系统权限
- 声明ServiceExtAbility (type: service)

---

### 2.5 PC模块 - PC/平板端产品

**路径**: `/product/pc/`
**类型**: Feature

结构与phone模块类似，适配PC/平板设备的大屏横屏场景。

---

### 2.6 Features组件 - 功能模块

#### 2.6.1 ScreenLock组件

**路径**: `/features/screenlock/`
**职责**: 锁屏核心功能

```
features/screenlock/
└── src/main/ets/com/ohos/
    ├── common/
    │   └── constants.ts            # 锁屏常量
    ├── model/                      # 数据模型
    │   ├── accountsModel.ts        # 账户模型
    │   ├── screenLockModel.ts      # 锁屏模型
    │   ├── screenLockService.ts    # 锁屏服务
    │   └── screenlockStyle.ts      # 锁屏样式
    ├── view/component/             # UI组件
    │   ├── accounts.ets            # 账户组件
    │   ├── customPSD.ets           # 图案密码UI
    │   ├── digitalPSD.ets          # 数字密码UI
    │   ├── lockIcon.ets            # 锁图标
    │   ├── mixedPSD.ets            # 混合密码UI
    │   ├── numkeyBoard.ets         # 数字键盘
    │   └── statusBar.ets           # 状态栏
    └── vm/                         # ViewModel
        ├── accountsViewModel.ts
        ├── baseViewModel.ts
        ├── customPSDViewModel.ts
        ├── digitalPSDViewModel.ts
        ├── mixedPSDViewModel.ts
        ├── StatusBarVM.ts
        └── lockIconViewModel.ts
```

**核心类**:

| 类名 | 文件 | 职责 |
|------|------|------|
| ScreenLockService | `screenLockService.ts` | 锁屏/解锁业务逻辑 |
| ScreenLockModel | `screenLockModel.ts` | 锁屏窗口模型 |
| AccountsModel | `accountsModel.ts` | 账户认证模型 |
| DigitalPSDViewModel | `digitalPSDViewModel.ts` | 数字密码逻辑 |
| MixedPSDViewModel | `mixedPSDViewModel.ts` | 混合密码逻辑 |
| CustomPSDViewModel | `customPSDViewModel.ts` | 图案密码逻辑 |

---

#### 2.6.2 NoticeItem组件

**路径**: `/features/noticeitem/`
**职责**: 通知显示与管理

```
features/noticeitem/
└── src/main/ets/com/ohos/noticeItem/
    ├── common/
    │   └── CommonUtil.ts
    ├── model/                      # 通知模型
    │   ├── NotificationDistributionManager.ts
    │   ├── NotificationManager.ts
    │   ├── NotificationService.ts
    │   ├── ParseDataUtil.ts
    │   └── rule/
    │       └── RuleController.ts
    ├── view/                       # 通知视图
    │   ├── NotificationListComponent.ets
    │   └── item/                   # 通知项组件
    │       ├── actionComponent.ets
    │       ├── bannerNotificationItem.ets
    │       ├── basicItem.ets
    │       ├── confirmDialog.ets
    │       ├── devicesDialog.ets
    │       ├── generalItem.ets
    │       ├── groupItem.ets
    │       ├── longItem.ets
    │       ├── multiItem.ets
    │       ├── notificationItem.ets
    │       ├── pictureItem.ets
    │       └── settingDialog.ets
    ├── viewmodel/
    │   └── ViewModel.ts
    └── vm/
        └── NotificationVM.ts
```

---

#### 2.6.3 其他Feature组件

| 组件 | 路径 | 核心文件 | 职责 |
|------|------|----------|------|
| **DateTime** | `features/datetimecomponent/` | `dateTimeViewModel.ts`, `dateTime.ets` | 日期时间显示 |
| **Wallpaper** | `features/wallpapercomponent/` | `wallpaperViewModel.ts`, `wallpaper.ets` | 壁纸显示 |
| **Shortcut** | `features/shortcutcomponent/` | `shortcutViewModel.ts`, `shortcut.ets` | 快捷操作 |
| **Battery** | `features/batterycomponent/` | `batteryModel.ts`, `batteryIcon.ets` | 电池状态 |
| **Signal** | `features/signalcomponent/` | `signalModel.ts`, `signalIcon.ets` | 信号状态 |
| **WiFi** | `features/wificomponent/` | `wifiModel.ts`, `wifiIcon.ets` | WiFi状态 |
| **Clock** | `features/clockcomponent/` | `clockIcon.ets` | 状态栏时间 |

---

## 3. 文件命名规范

### 3.1 TypeScript/ArkTS文件

| 类型 | 命名规范 | 示例 |
|------|----------|------|
| Ability类 | PascalCase + Ability | `MainAbility.ts`, `ServiceExtAbility.ts` |
| Manager类 | PascalCase + Manager | `WindowManager.ts`, `EventManager.ts` |
| Model类 | PascalCase + Model | `screenLockModel.ts`, `accountsModel.ts` |
| ViewModel类 | PascalCase + ViewModel | `indexViewModel.ts`, `digitalPSDViewModel.ts` |
| 工具类 | camelCase | `Log.ts`, `EventBus.ts` |
| 常量 | UPPER_SNAKE_CASE | `SCREEN_CHANGE_EVENT` |

### 3.2 ArkUI页面文件

| 类型 | 命名规范 | 示例 |
|------|----------|------|
| 页面 | camelCase + .ets | `index.ets`, `slidescreenlock.ets` |
| 组件 | PascalCase + .ets | `digitalPSD.ets`, `statusBar.ets` |

---

## 4. 关键文件索引

### 4.1 入口与配置

| 文件路径 | 说明 |
|----------|------|
| `AppScope/app.json5` | 应用配置 |
| `build-profile.json5` | 构建配置 |
| `oh-package.json5` | 包管理配置 |
| `entry/src/main/module.json5` | Entry模块配置 |
| `product/phone/src/main/module.json5` | Phone模块配置（权限声明） |

### 4.2 核心服务

| 文件路径 | 说明 |
|----------|------|
| `product/phone/src/main/ets/ServiceExtAbility/ServiceExtAbility.ts` | 锁屏服务核心 |
| `features/screenlock/src/main/ets/com/ohos/model/screenLockService.ts` | 锁屏业务逻辑 |
| `features/screenlock/src/main/ets/com/ohos/model/screenLockModel.ts` | 锁屏窗口模型 |
| `features/screenlock/src/main/ets/com/ohos/model/accountsModel.ts` | 账户认证模型 |

### 4.3 事件与窗口

| 文件路径 | 说明 |
|----------|------|
| `common/src/main/ets/default/event/EventBus.ts` | 事件总线 |
| `common/src/main/ets/default/event/EventManager.ts` | 事件管理器 |
| `common/src/main/ets/default/WindowManager.ts` | 窗口管理 |
| `common/src/main/ets/default/ScreenLockManager.ts` | 屏幕状态管理 |

### 4.4 ViewModel

| 文件路径 | 说明 |
|----------|------|
| `product/phone/src/main/ets/vm/indexViewModel.ts` | 主页ViewModel |
| `features/screenlock/vm/digitalPSDViewModel.ts` | 数字密码VM |
| `features/screenlock/vm/mixedPSDViewModel.ts` | 混合密码VM |
| `features/screenlock/vm/customPSDViewModel.ts` | 图案密码VM |

---

## 5. 模块依赖图

```
┌─────────────────────────────────────────────────────────────────┐
│                          Entry模块                               │
│                    (应用入口，MainAbility)                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Product模块                               │
│              ┌───────────────────────────────┐                  │
│              │    phone / pc 模块             │                  │
│              │  (ServiceExtAbility + Pages)   │                  │
│              └───────────────────────────────┘                  │
└─────────────────────────────────────────────────────────────────┘
         │                         │                         │
         ▼                         ▼                         ▼
┌─────────────────┐  ┌─────────────────────┐  ┌─────────────────┐
│  Features模块    │  │     Features模块     │  │  Features模块    │
│  (screenlock)   │  │    (noticeitem)      │  │ (datetime, ...)│
│                 │  │                     │  │                 │
│ 锁屏核心功能     │  │ 通知显示与管理       │  │ 状态展示组件     │
└─────────────────┘  └─────────────────────┘  └─────────────────┘
         │                         │                         │
         └─────────────────────────┼─────────────────────────┘
                                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                         Common模块                               │
│     (EventBus, WindowManager, Log, AbilityManager, ...)         │
└─────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                       OpenHarmony系统服务                        │
│  @ohos.screenLock, @ohos.window, @ohos.account.osAccount, ...   │
└─────────────────────────────────────────────────────────────────┘
```

---

*关键结论: 项目采用模块化架构，13个模块职责清晰，通过Common模块复用通用能力，通过系统服务API与OpenHarmony深度集成。*
