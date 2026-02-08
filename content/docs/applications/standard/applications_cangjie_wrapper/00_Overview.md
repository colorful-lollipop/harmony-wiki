# 项目概述

本文档提供 `applications_cangjie_wrapper` 项目的整体定位、核心能力、功能边界和关键概念说明。

---

## 项目定位

`applications_cangjie_wrapper` 是 OpenHarmony 应用程序框架的 **Cangjie 语言封装层**，为使用仓颉语言开发 OpenHarmony 应用的开发者提供系统设置访问能力。

### 在 OpenHarmony 架构中的位置

```
┌─────────────────────────────────────────────────────────────┐
│                    应用层 (Application)                      │
│              Cangjie 开发者编写的应用程序                     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│           kit.BasicServicesKit (公开 API 层)                │
│         本仓库: kit/BasicServicesKit/index.cj              │
│              统一的 Cangjie API 导出入口                     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│           ohos.settings (实现层)                            │
│      本仓库: ohos/settings/*.cj                            │
│    getValue() / DomainName / Date / Display 等实现          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              FFI 层 (Foreign Function Interface)            │
│           FfiSettingsGetValue() 外部函数声明                │
│     调用底层 settings 子系统的 C/C++ 实现                    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              OpenHarmony 系统服务层                          │
│              settings 子系统 (C++/Rust)                      │
│          实际访问 SettingsProvider 数据库                    │
└─────────────────────────────────────────────────────────────┘
```

### 设计目标

1. **语言桥接**：为 Cangjie 开发者提供访问 OpenHarmony 系统设置的标准方式
2. **API 一致性**：与 ArkTS 版 Settings API 保持概念对齐（功能子集）
3. **类型安全**：利用 Cangjie 的静态类型系统减少运行时错误
4. **最小依赖**：仅暴露必要的系统能力，保持轻量

---

## 核心能力

本项目目前提供以下三类系统设置查询能力：

### 1. 时间日期设置 (Date)

| 设置项 | 说明 | 返回值示例 |
|--------|------|------------|
| `DateFormat` | 日期显示格式 | `"mm/dd/yyyy"`, `"dd/mm/yyyy"`, `"yyyy/mm/dd"` |
| `TimeFormat` | 时间显示格式 | `"12"` (12小时制), `"24"` (24小时制) |
| `AutoGainTime` | 是否从 NITZ 自动获取时间 | `"true"` / `"false"` |
| `AutoGainTimeZone` | 是否从 NITZ 自动获取时区 | `"true"` / `"false"` |

**代码证据**: `ohos/settings/settings_common.cj:84-146`

### 2. 显示效果设置 (Display)

| 设置项 | 说明 | 返回值示例 |
|--------|------|------------|
| `FontScale` | 字体缩放因子 | `"1.0"`, `"1.25"` (float) |
| `ScreenBrightnessStatus` | 屏幕亮度值 | `"0"` ~ `"255"` |
| `AutoScreenBrightness` | 自动亮度调节 | `"0"` (关) / `"1"` (开) |
| `ScreenOffTimeout` | 屏幕超时时间(毫秒) | `"60000"` (1分钟) |

**代码证据**: `ohos/settings/settings_common.cj:152-225`

### 3. 域数据查询 (DomainName)

支持通过域名称查询设置项：

| 域名称 | 说明 | 内部字符串 |
|--------|------|------------|
| `DeviceShared` | 设备级共享键 | `"global"` |
| `UserProperty` | 用户级属性键 | `"system"` |
| `UserSecurity` | 用户安全级键 (内部) | `@!Hide` |

**代码证据**: `ohos/settings/settings_common.cj:26-75`

---

## 功能边界与限制

### 支持的操作

✅ **仅支持查询 (Read-Only)**
- 获取时间日期设置
- 获取显示效果设置
- 获取指定域的数据项

### 不支持的操作

❌ **暂不支持 (相比 ArkTS API)**

| 功能 | 说明 | 未来支持可能性 |
|------|------|----------------|
| 设置时间日期 | 修改系统时间格式 | 需系统权限，受限 |
| 设置显示效果 | 修改屏幕亮度等 | 需系统权限，受限 |
| 注册数据观察者 | 监听设置变化 | TODO(待确认) |
| 打开网络设置页 | 跳转系统设置界面 | 依赖 Ability 框架 |
| 飞行模式控制 | 开启/关闭飞行模式 | 需系统权限，受限 |
| 悬浮窗检查 | 检查应用悬浮窗权限 | 依赖 AMS |

**证据来源**: `README.md:58-65`

---

## 运行环境

### 系统要求

| 项目 | 要求 |
|------|------|
| OpenHarmony 版本 | 3.2+ (API Level 22+) |
| 系统类型 | Standard (仅标准设备) |
| Syscap | `SystemCapability.Applications.Settings.Core` |

**代码证据**: `bundle.json:17-19`
```json
"adapted_system_type": [
    "standard"
]
```

### 资源占用

根据 `bundle.json:20-21`：

| 指标 | 数值 |
|------|------|
| ROM | ~100 KB |
| RAM | ~104 KB |

---

## 关键概念

### UIAbilityContext

应用能力上下文，是访问系统设置的入口凭证。

```cangjie
import ohos.app.ability.ui_ability.{UIAbilityContext, getStageContext}

// 从 Ability 获取上下文
let context: UIAbilityContext = /* Ability 提供的上下文 */
let stageContext = getStageContext(context)
```

**验证逻辑**: `ohos/settings/settings.cj:55-58`
```cangjie
let stageContext = getStageContext(context)
if (stageContext.isNull()) {
    throw BusinessException(14800000, "Parameter error.")
}
```

### Settings Data 键名

系统设置数据库使用字符串键存储各项设置：

```
settings.date.date_format          → 日期格式
settings.date.time_format          → 时间格式
settings.display.font_scale        → 字体缩放
settings.display.screen_brightness_status  → 屏幕亮度
settings.display.auto_screen_brightness    → 自动亮度
settings.display.screen_off_timeout        → 屏幕超时
```

**代码证据**: `ohos/settings/settings_common.cj:137-144`, `217-223`

### FFI 调用链

Cangjie 层通过 FFI 调用底层 C/C++ 实现：

```
getValue() [Cangjie]
    │
    ├──> FfiSettingsGetValue() [FFI 声明]
    │       └──> settings::cj_settings_ffi [C++]
    │               └──> SettingsProvider [系统服务]
    │                       └──> 数据库查询
```

**FFI 声明**: `ohos/settings/settings_ffi.cj:22-25`

---

## 版本信息

| 项目 | 值 |
|------|-----|
| 包名 | `@ohos/applications_cangjie_wrapper` |
| 版本 | 6.1 |
| API Level | 22+ (`since: "22"`) |
| License | Apache License 2.0 |

**代码证据**: `bundle.json:2-5`

---

## 依赖关系

### 运行时依赖

| 组件 | 用途 | 来源 |
|------|------|------|
| settings | 基础设置功能 | OpenHarmony 系统 |
| cangjie_ark_interop | API 注解、异常类 | ohos-sig |
| ability_cangjie_wrapper | UIAbilityContext | ohos-sig |
| hiviewdfx_cangjie_wrapper | 日志 (Hilog) | ohos-sig |

**代码证据**: `bundle.json:22-29`

### 可选依赖 (kit 层聚合)

`kit/BasicServicesKit/index.cj` 还聚合了其他子系统的接口：

```cangjie
public import ohos.device_info.*      // 设备信息
public import ohos.request.*          // 下载请求
public import ohos.common_event_manager.*  // 公共事件
public import ohos.system_date_time.* // 系统日期时间
public import ohos.battery_info.*     // 电池信息
```

**注意**: 这些依赖项的具体实现位于其他仓库，本文档仅覆盖 `applications_cangjie_wrapper` 本身的实现。

---

## 下一步阅读

- **[架构说明](./10_Architecture.md)** - 深入理解组件设计和数据流
- **[API 参考](./20_API_Reference.md)** - 查看完整 API 文档和使用示例
- **[构建系统](./30_GN_Build.md)** - 了解 GN 构建配置和产物
