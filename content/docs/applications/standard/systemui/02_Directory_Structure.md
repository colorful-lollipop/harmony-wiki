# SystemUI 目录结构

## 1. 顶层目录

```
systemui/
├── AppScope/                  # 应用级配置
│   └── resources/             # 应用资源文件
├── common/                     # [模块] 通用工具
├── entry/                      # [模块] 入口模块
│   ├── phone/                 # [HAP] 手机版
│   └── pc/                    # [HAP] PC 版
├── features/                   # [模块] 功能组件
├── figures/                    # 文档资源
├── hvigor/                     # hvigor 配置
├── infra/                      # 基础设施
├── product/                    # [模块] 产品模块
├── signature/                  # 签名文件
├── build-profile.json5         # 构建配置
├── gradle.properties           # Gradle 属性
├── hvigorfile.js              # hvigor 入口
├── LICENSE                     # Apache 2.0
├── OAT.xml                    # OpenHarmony 资产表
├── oh-package.json5
└── README.md
```

## 2. common 模块 (通用工具)

### 2.1 目录结构

```
common/src/main/ets/default/
├── abilitymanager/            # 能力管理
│   ├── abilityManager.ts     # 主管理类
│   ├── bundleManager.ts       # Bundle 管理
│   ├── featureAbilityManager.ts
│   └── notificationManager.ts
├── event/                     # 事件系统
│   ├── EventBus.ts            # 事件总线
│   ├── EventManager.ts        # 事件管理
│   └── EventUtil.ts           # 事件工具
├── commonEvent/               # 公共事件
├── heightcofigUtils/          # 高度配置
├── CheckEmptyUtils.ts         # 空值检查
├── CommonStyleManager.ts      # 样式管理
├── Constants.ts               # 常量定义
├── Decorators.ts              # 装饰器
├── InitSystemUi.ts            # 初始化入口
├── Log.ts                     # 日志工具
├── MultimodalInputManager.ts  # 多模输入
├── ReadConfigUtil.ts          # 配置读取
├── ResourceUtil.ts            # 资源工具
├── ScreenLockManager.ts       # 锁屏管理
├── SettingsUtil.ts            # 设置工具
├── SingleInstanceHelper.ts    # 单例辅助
├── StyleConfiguration.ts       # 样式配置
├── SwitchUserManager.ts       # 用户切换
├── SysFaultLogger.ts          # 故障日志
├── TimeManager.ts             # 时间管理
├── TintStateManager.ts        # 色调管理
├── Trace.ts                   # 追踪工具
└── WindowManager.ts           # 窗口管理
```

### 2.2 模块职责

| 目录/文件 | 职责 |
|-----------|------|
| `abilitymanager/` | Ability 上下文管理、Bundle 信息 |
| `event/` | EventBus 事件发布/订阅 |
| `Log.ts` | HiLog 日志封装，支持敏感信息过滤 |
| `WindowManager.ts` | 系统窗口创建和管理 |
| `TimeManager.ts` | 系统时间同步 |
| `ScreenLockManager.ts` | 锁屏状态管理 |
| `NotificationManager.ts` | 通知管理器 |

### 2.3 证据

- **模块类型**: HAR (`module.json5` 未声明 type 默认为 har)
- **证据文件**: `common/src/main/module.json5`
- **代码路径**: `common/src/main/ets/default/`

## 3. entry 模块 (入口模块)

### 3.1 phone 版本

```
entry/phone/src/main/
├── ets/
│   ├── Application/
│   │   └── AbilityStage.ts    # Ability 入口
│   ├── ServiceExtAbility/
│   │   └── ServiceExtAbility.ts
│   └── pages/
│       └── index.ets          # 主页面
├── module.json5               # 模块配置
└── resources/                # 资源文件
```

### 3.2 pc 版本

```
entry/pc/src/main/
├── ets/
│   ├── Application/
│   │   └── AbilityStage.ts
│   ├── ServiceExtAbility/
│   │   └── ServiceExtAbility.ts
│   └── pages/
│       └── index.ets
├── module.json5
└── resources/
```

### 3.3 关键配置

| 配置项 | phone 值 | pc 值 |
|--------|----------|-------|
| `module.name` | `phone_entry` | `pc_entry` |
| `module.type` | `entry` | `entry` |
| `mainElement` | `com.ohos.systemui.ServiceExtAbility` | 同左 |
| `extensionAbilities[0].type` | `service` | `service` |
| `requestPermissions` | 19 个权限 | 继承 |

**证据**: `entry/phone/src/main/module.json5:1-118`

### 3.4 ServiceExtAbility

```typescript
class ServiceExtAbility extends ServiceExtension {
  onCreate(want: Want): void {
    Log.showInfo(TAG, `onCreate, want: ${JSON.stringify(want)}`);
    initSystemUi(this.context);
    AbilityManager.setContext(ABILITY_NAME_ENTRY, this.context);
  }
}
```

**证据**: `entry/phone/src/main/ets/ServiceExtAbility/ServiceExtAbility.ts:23-28`

## 4. features 模块 (功能组件)

共 **21 个 HAR 模块**，按功能分类：

### 4.1 系统状态类

| 组件 | 路径 | 职责 |
|------|------|------|
| `batterycomponent` | `features/batterycomponent/` | 电池电量显示 |
| `brightnesscomponent` | `features/brightnesscomponent/` | 屏幕亮度 |
| `clockcomponent` | `features/clockcomponent/` | 系统时钟 |
| `signalcomponent` | `features/signalcomponent/` | SIM 卡信号 |
| `volumecomponent` | `features/volumecomponent/` | 音量状态 |
| `volumepanelcomponent` | `features/volumepanelcomponent/` | 音量面板 |

### 4.2 网络连接类

| 组件 | 路径 | 职责 |
|------|------|------|
| `airplanecomponent` | `features/airplanecomponent/ | 飞行模式 |
| `bluetoothcomponent` | `features/bluetoothcomponent/` | 蓝牙状态 |
| `locationcomponent` | `features/locationcomponent/` | 位置服务 |
| `nfccomponent` | `features/nfccomponent/` | NFC 状态 |
| `wificomponent` | `features/wificomponent/` | WiFi 状态 |

### 4.3 通知相关类

| 组件 | 路径 | 职责 |
|------|------|------|
| `capsulecomponent` | `features/capsulecomponent/` | 通知胶囊 UI |
| `controlcentercomponent` | `features/controlcentercomponent/` | 控制中心 |
| `managementcomponent` | `features/managementcomponent/` | 通知管理 |
| `navigationservice` | `features/navigationservice/` | 导航服务 |
| `noticeitem` | `features/noticeitem/` | 通知项组件 |
| `notificationservice` | `features/notificationservice/` | 通知服务 |

### 4.4 其他

| 组件 | 路径 | 职责 |
|------|------|------|
| `autorotatecomponent` | `features/autorotatecomponent/` | 自动旋转 |
| `ringmodecomponent` | `features/ringmodecomponent/` | 铃声模式 |
| `statusbarcomponent` | `features/statusbarcomponent/` | 状态栏组件 |

### 4.5 标准结构

```
features/[component]/src/main/
├── ets/
│   └── [component-specific structure]
├── module.json5
└── resources/
```

**证据**: `features/batterycomponent/src/main/module.json5:1-10`

```json5
{
  "module": {
    "name": "batterycomponent",
    "type": "har",
    "deviceTypes": ["default"],
    "uiSyntax": "ets"
  }
}
```

## 5. product 模块 (产品模块)

共 **9 个 feature 模块**，按设备和产品组合：

### 5.1 default 产品

| 模块 | 路径 | 职责 |
|------|------|------|
| `default_navigationBar` | `product/default/navigationBar/` | 导航栏 |
| `default_notificationmanagement` | `product/default/notificationmanagement/` | 通知管理 |
| `default_volumepanel` | `product/default/volumepanel/` | 音量面板 |
| `default_dialog` | `product/default/dialog/` | 系统对话框 |

### 5.2 phone 产品

| 模块 | 路径 | 职责 |
|------|------|------|
| `phone_statusbar` | `product/phone/statusbar/` | 状态栏 |
| `phone_dropdownpanel` | `product/phone/dropdownpanel/` | 下拉面板 |

### 5.3 pc 产品

| 模块 | 路径|------|------ | 职责 |
|------|
| `pc_statusbar` | `product/pc/statusbar/` | 状态栏 |
| `pc_controlpanel` | `product/pc/controlpanel/` | 控制面板 |
| `pc_notificationpanel` | `product/pc/notificationpanel/` | 通知面板 |

### 5.4 标准结构

```
product/[device]/[module]/src/main/
├── ets/
│   ├── Application/
│   │   └── AbilityStage.ts
│   ├── ServiceExtAbility/
│   │   └── ServiceExtAbility.ts
│   ├── pages/
│   │   └── [pages.ets]
│   └── [other dirs]
├── module.json5
└── resources/
```

**证据**: `product/phone/statusbar/src/main/module.json5:1-33`

## 6. 目录职责总览

| 目录 | 类型 | 职责 | 可被引用 |
|------|------|------|----------|
| `common/` | HAR | 通用工具 | ✅ features + product |
| `entry/` | HAP | 入口模块 | ❌ (独立安装) |
| `features/*/` | HAR | 功能组件 | ✅ product |
| `product/*/` | feature | 产品模块 | ✅ entry |

## 7. 排除的目录

以下目录**不包含**在 Wiki 文档范围内：

| 目录 | 原因 |
|------|------|
| `test/` | 测试代码 |
| `unittest/` | 单元测试 |
| `*_test.*` | 测试文件 |
| `fuzz/` | 模糊测试 |

## 8. 资源目录结构

```
[module]/src/main/resources/
├── base/
│   ├── element/           # 字符串、颜色等
│   ├── media/             # 图片资源
│   └── profile/           # 配置文件
└── default_02/           # 默认设备资源
```

## 9. 相关文档

- [项目概览](01_Overview.md) - 项目定位和核心能力
- [架构设计](03_Architecture.md) - 系统架构图
- [内部 API](04_API_Inner.md) - 模块接口
- [构建指南](05_Build.md) - 编译配置
