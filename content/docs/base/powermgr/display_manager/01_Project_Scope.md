# 项目定位与边界 - display_manager

> 本文档说明 display_manager 模块的项目定位、边界与关键概念

---

## 文档目的

本文档说明：
- 模块在 OpenHarmony 系统中的定位
- 模块边界与职责范围
- 与其他模块的交互关系
- 关键概念定义

## 适用范围

- **适用对象**：OpenHarmony 系统集成成员、display_manager 模块开发者
- **前置知识**：熟悉 OpenHarmony System Ability 机制、IPC 通信

---

## 项目定位

### 模块定义

display_manager 是 OpenHarmony **powermgr（电源管理）子系统的核心组件**，负责显示设备的电源与亮度管理。

**证据**：`bundle.json:15-24`
```json
{
  "name": "@ohos/display_manager",
  "subsystem": "powermgr",
  "description": "显示能效管理，包括屏幕亮灭、亮度调节等。"
}
```

### 在系统中的位置

```
base/powermgr/
├── power_manager/            # 电源管理子系统根目录
│   ├── display_manager/    # 显示管理模块（本文档）
│   ├── power_manager/       # 电源管理服务
│   ├── battery_manager/     # 电池管理
│   ├── thermal_manager/    # 热管理
│   └── battery_statistics/  # 电池统计
```

---

## 核心能力

### 主要功能

| 功能类别 | 具体能力 | API 层面 |
|---------|----------|---------|
| **显示电源管理** | 屏幕开/关、状态查询、休眠控制 | JS N-API + IPC |
| **亮度管理** | 设置/获取亮度、自动亮度、渐变动画 | JS N-API + IPC |
| **场景适配** | 游戏/相机模式亮度曲线 | 内部逻辑 |
| **高级功能** | 亮度提升、覆盖模式、折扣亮度 | JS N-API + IPC |

### 非核心能力

| 功能 | 说明 | 证据 |
|------|------|------|
| **配置解析** | JSON 配置文件解析（亮度曲线、光阈值等） | config_parser 相关类 |
| **数据监听** | 亮度变化、光强度变化事件分发 | 回调接口 |
| **渐变动画** | 平滑亮度过渡动画 | GradualAnimator |
| **光传感器集成** | 环境光采样与过滤 | LightLuxManager（可选） |

---

## 模块边界

### 职责边界

| 模块 | 负责 | 不负责 |
|------|------|------|
| **display_manager** | 显示电源控制、亮度计算与设置、动画执行 | 显示内容渲染、触摸输入处理 |
| **power_manager** | 提供电源状态接口、系统电源策略 | 具体的显示驱动适配 |
| **window_manager** | 窗口合成、Surface 管理 | 亮度数值计算 |

### 数据边界

| 数据类型 | 来源 | 目的 |
|---------|------|------|
| **显示状态** | window_manager | display_manager | 来自 Window Manager，用于控制 |
| **亮度值** | 用户输入、自动算法 | display_manager | 设置到显示器 |
| **光传感器数据** | 传感器 HAL | display_manager | 用于自动亮度计算 |
| **配置数据** | JSON 文件、系统设置 | display_manager | 亮度曲线、阈值 |

### 接口边界

| 接口 | 提供者 | 使用者 |
|------|------|------|
| **JS N-API** | display_manager | 应用开发者 |
| **System Ability（IPC）** | display_manager | 系统服务、其他模块 |
| **内部 C++ API** | display_manager | brightness_manager 等内部模块 |

---

## 关键概念

### System Ability（系统能力）

display_manager 注册为 OpenHarmony System Ability，提供跨进程的显示管理服务。

| 属性 | 值 | 证据 |
|------|------|------|
| **SA ID** | 3308 | `state_manager/sa_profile/3308.json:5` |
| **SA 名称** | Display Power Manager Service | 配置文件 |
| **进程名** | powermgr | `3308.json:2` |
| **服务库** | libdisplaymgrservice.z.so | `3308.json:6` |

**详细文档**：参见 [03_Architecture.md](03_Architecture.md#system-ability-注册)

### IPC 通信架构

display_manager 使用 ZIDL（Zero-copy IDL）定义 IPC 接口。

| 组件 | 角色 | 证据 |
|------|------|------|
| **服务端（Stub）** | DisplayPowerMgrService | `display_power_mgr_service.h:44` |
| **客户端（Proxy）** | DisplayPowerMgrClient | `display_power_mgr_client.h:31` |
| **IDL 定义** | IDisplayPowerMgr.idl | `state_manager/service/IDisplayPowerMgr.idl` |
| **生成代码** | Proxy/Stub 通过 IDL 工具生成 | GN 构建配置 |

**详细文档**：参见 [03_Architecture.md](03_Architecture.md#zidl-通信架构)

### 单例模式

display_manager 使用两种单例模式：

| 单例类型 | 类 | 证据 |
|---------|------|------|
| **服务端单例** | DisplayPowerMgrService | `display_power_mgr_service.h:170` |
| **客户端延迟单例** | DisplayPowerMgrClient | `display_power_mgr_client.h:31` |

**证据**：
```cpp
// 服务端：friend DelayedSpSingleton<DisplayPowerMgrService>;
// 客户端：DECLARE_DELAYED_REF_SINGLETON(DisplayPowerMgrClient);
```

### 亮度计算

display_manager 包含复杂的亮度计算逻辑，用于根据环境光和场景自动调整亮度。

| 组件 | 职责 | 证据 |
|------|------|------|
| **CalculationManager** | 亮度曲线插值计算 | `calculation_manager.h:27` |
| **LightLuxManager** | 光传感器数据采集与滤波 | `light_lux_manager.h:27` |
| **BrightnessService** | 亮度策略协调 | `brightness_service.h:51` |

---

## 依赖关系

### 向上依赖

| 外部模块 | 用途 | 接口 |
|---------|------|------|
| **safwk/samgr** | System Ability 框架 | 注册/查询 SA |
| **power_manager** | 电源管理基础服务 | RunningLock、电源状态 |
| **window_manager** | 窗口管理服务 | 屏幕控制接口 |
| **sensor** | 光传感器 HAL | 环境光数据（可选） |

### 被依赖模块

| 依赖模块 | 用途 | 接口 |
|---------|------|------|
| **其他子系统服务** | 使用显示管理功能 | System Ability 3308 |

### 内部模块依赖

```
displaymgrservice (服务)
├── 依赖: brightness_manager (静态库)
├── 依赖: displaymgr_proxy (source_set)
└── 调用: ScreenController, GradualAnimator

brightness_manager (静态库)
├── 依赖: 无（自包含）
└── 包含: CalculationManager, LightLuxManager

libdisplaymgr.so (内部 API)
├── 依赖: displaymgr_proxy
└── 提供: DisplayPowerMgrClient 给应用层
```

---

## 运行时特性

### 条件编译

| 特性 | 编译标志 | 默认值 | 证据 |
|------|----------|--------|------|
| **传感器支持** | `ENABLE_SENSOR_PART` | 自动检测 | `displaymgr.gni:18-24` |
| **屏幕关闭策略** | `ENABLE_SCREEN_POWER_OFF_STRATEGY` | false | `displaymgr.gni:26,44-46` |
| **亮度扩展包装** | `OHOS_BUILD_ENABLE_BRIGHTNESS_WRAPPER` | 空（未启用） | `displaymgr.gni:25,85-87` |

### 配置驱动

display_manager 支持通过配置文件驱动行为：

| 配置类型 | 位置 | 用途 |
|---------|------|------|
| **亮度曲线** | JSON 文件（亮度-光强映射） | 场景自适应 |
| **光阈值** | JSON 文件（亮/暗切换点） | 防抖动 |
| **参数配置** | display.para | 系统参数 |

---

## 安全边界

### 权限模型

display_manager 采用**系统应用专用**的安全模型（证据：`display_power_mgr_service.cpp:211` 等 24 处检查）：

| 操作类型 | 权限要求 | 证据 |
|---------|----------|------|
| **常规亮度操作** | 系统应用 | `Permission::IsSystem()` |
| **高级亮度操作** | 系统应用 | `Permission::IsSystem()` |
| **JSON 命令** | 系统应用 | `Permission::IsSystem()` |
| **测试模式** | 系统应用 | `Permission::IsSystem()` |

**详细文档**：参见 [08_Security_Analysis.md](08_Security_Analysis.md)

### IPC 安全

| 安全机制 | 说明 | 证据 |
|---------|------|------|
| **调用者识别** | GetCallingPid/Uid | `display_power_mgr_service.cpp:797` |
| **身份重置** | ResetCallingIdentity | `screen_action.cpp:38-40` |
| **死亡通知** | Death Recipient | `display_power_mgr_service.h:135-142` |

---

## 限制与约束

### 功能限制

| 限制 | 说明 | 理由 |
|------|------|------|
| **仅系统应用访问** | 非 system 应用无法调用 | 安全策略 |
| **传感器可选** | 部分设备不支持光传感器 | 硬件差异 |
| **多显示器限制** | 当前实现主要针对主显示器 | 部分多屏设备 |
| **亮度范围** | 1-255（硬件限制） | 硬件约束 |

### 性能约束

| 约束 | 值 | 说明 |
|------|------|------|
| **最大亮度** | 255 | 8 位表示 |
| **渐变时长** | 可配置 | 平滑度与性能权衡 |
| **动画更新率** | 60Hz 可配置 | 视觉效果 vs. CPU 占用 |

---

## 相关链接

- **架构文档**：[03_Architecture.md](03_Architecture.md) - 详细架构说明
- **N-API 文档**：[04_NAPI_Interface.md](04_NAPI_Interface.md) - JS API 清单
- **内部 API**：[05_Internal_API.md](05_Internal_API.md) - C++ API 说明
- **GN 构建**：[06_GN_Targets.md](06_GN_Targets.md) - 构建系统

---

## 文档更新记录

- **2026-02-06**：初始版本 v1.0
