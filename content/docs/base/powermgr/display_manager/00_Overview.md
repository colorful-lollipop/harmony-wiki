# 项目概览 - display_manager

> OpenHarmony 显示能效管理组件的总体概览

---

## 文档目的

本文档提供 display_manager 模块的高层次概览，帮助新人快速理解项目的：
- 核心功能与定位
- 运行环境与依赖
- 关键概念与术语

## 适用范围

- **模块路径**：`base/powermgr/display_manager`
- **适用对象**：OpenHarmony 应用开发者、系统集成员、模块维护者
- **前置知识**：熟悉 C++、N-API、System Ability 机制

---

## 核心能力

display_manager 模块提供显示屏的亮/灭、亮度调节等功能：

### 1. 屏幕开关控制

| 能力 | 说明 | 使用场景 |
|------|------|---------|
| **屏幕开/关** | 控制显示设备的电源状态（亮屏、灭屏） | 应用控制设备休眠、系统管理电源策略 |
| **显示状态查询** | 获取当前屏幕状态（ON/OFF/DIM 等） | 状态同步、电源管理决策 |

### 2. 亮度管理

| 能力 | 说明 | 使用场景 |
|------|------|---------|
| **设置亮度** | 设置屏幕亮度（0-255 等级） | 用户调节屏幕亮度、应用自适应亮度 |
| **获取亮度** | 获取当前/默认/最大/最小亮度 | 状态查询、UI 显示 |
| **渐变亮度** | 亮度平滑过渡动画 | 改善用户体验、避免突变闪烁 |
| **折扣亮度** | 暂时降低亮度比例 | 系统通知、降低功耗 |

### 3. 高级亮度特性

| 能力 | 说明 | 依赖条件 |
|------|------|---------|
| **自动亮度** | 基于环境光传感器自动调节亮度 | 传感器支持（`ENABLE_SENSOR_PART`） |
| **亮度提升** | 临时提升亮度（超时恢复） | 用户需要临时更高亮度 |
| **覆盖亮度** | 临时覆盖系统亮度（超时恢复） | 测试模式、特殊场景 |
| **游戏/相机模式** | 不同场景使用不同亮度曲线 | 场景识别、曲线配置 |

### 4. 事件与回调

| 能力 | 说明 | 使用场景 |
|------|------|---------|
| **电源状态变化监听** | 监听屏幕开关事件 | 应用同步状态、UI 更新 |
| **亮度变化监听** | 监听亮度变化事件 | 应用同步亮度、数据统计 |
| **数据变化监听** | 监听多种亮度数据变化 | APS 集成、UI 实时更新 |

---

## 运行环境

### 系统要求

| 项目 | 要求 | 证据 |
|------|------|------|
| **操作系统** | OpenHarmony | bundle.json: "adapted_system_type": [ "standard" ] |
| **子系统** | powermgr | bundle.json: "subsystem": "powermgr" |
| **系统能力** | SystemCapability.PowerManager.DisplayPowerManager | bundle.json: "syscap" |
| **目标平台** | standard | bundle.json: "adapted_system_type" |

### 资源限制

| 资源 | 限制值 | 证据 |
|------|--------|------|
| **ROM** | 1024KB | bundle.json: "rom": "1024KB" |
| **RAM** | 2048KB | bundle.json: "ram": "2048KB" |

---

## 核心组件

### 模块架构

```
display_manager/
├── state_manager/           # 显示状态管理（核心）
│   ├── frameworks/          # Framework 层
│   │   ├── napi/       # 传统 N-API（brightness 模块）
│   │   └── ets/taihe/  # 新版 ETS/ANI 绑定
│   ├── service/            # System Ability 服务
│   ├── interfaces/          # 内部 API
│   └── utils/              # 工具类
└── brightness_manager/      # 亮度管理（静态库）
```

### System Ability

- **SA ID**：3308
- **服务名**：Display Manager Service
- **配置文件**：`state_manager/sa_profile/3308.json`
- **详细文档**：参见 [03_Architecture.md](03_Architecture.md#system-ability-注册)

---

## 关键概念

### DisplayState（显示状态）

显示设备的电源状态枚举（证据：`display_power_info.h:24-33`）：

| 值 | 名称 | 说明 |
|------|------|------|
| 0 | DISPLAY_OFF | 屏幕关闭 |
| 1 | DISPLAY_DIM | 屏幕变暗（渐变中） |
| 2 | DISPLAY_ON | 屏幕开启 |
| 3 | DISPLAY_SUSPEND | 屏幕挂起 |
| 4 | DISPLAY_DELAY_OFF | 延迟关闭 |
| 5 | DISPLAY_DOZE | Doze 模式 |
| 6 | DISPLAY_DOZE_SUSPEND | Doze 挂起 |

### BrightnessMode（亮度模式）

亮度调节模式（证据：`brightness_base.h:45-49`）：

| 模式 | 说明 |
|------|------|
| CAMERA_MODE | 相机模式（不同亮度曲线） |
| GAME_MODE | 游戏模式（不同亮度曲线） |
| DEFAULT_MODE | 默认模式 |

### DisplayDataChangeListenerType（数据监听类型）

亮度数据变化监听类型（证据：`DisplayPowerMgrIdlTypes.idl:18-23`）：

| 类型 | 说明 |
|------|------|
| 0 | LIGHT_OR_BRIGHTNESS_FOR_APS | 通知 APS 光或亮度变化 |
| 1 | STABLE_LUX | 通知稳定环境光变化 |
| 2 | FORCE_EXIT_OVERRIDDEN_MODE | 通知强制退出覆盖状态 |
| 3 | BRIGHTNESS_FOR_UI | 通知所有亮度更新（包含渐变） |
| 4 | BRIGHTNESS_TARGET | 仅通知最终目标亮度变化 |

---

## 依赖关系

### 外部依赖（来自 bundle.json）

| 组件 | 用途 | 说明 |
|------|------|------|
| ability_base, ability_runtime | 能力框架 | Ability 注册、生命周期管理 |
| cJSON | JSON 解析 | 配置文件解析 |
| c_utils | 工具库 | 基础工具函数 |
| eventhandler | 事件处理 | 事件循环、定时器 |
| ffrt | Fibre FRT | 异步任务调度 |
| graphic_2d | 图形框架 | 2D 图形渲染 |
| hicollie, hiview, hilog | 日志与监控 | 日志系统、性能监控 |
| ipc | IPC 框架 | 进程间通信 |
| napi | N-API | Native API 框架 |
| power_manager | 电源管理 | 电源服务接口、电源管理客户端 |
| safwk, samgr | 系统能力框架 | System Ability 管理 |
| sensor | 传感器 | 光传感器接口（可选） |
| window_manager | 窗口管理 | 屏幕控制接口 |

### 内部依赖

| 模块 | 依赖 | 说明 |
|------|------|------|
| state_manager | 依赖 brightness_manager | 亮度管理作为静态库被服务链接 |
| frameworks/napi | 依赖 interfaces/inner_api | 通过 libdisplaymgr.so 调用服务 |

---

## 版本与许可证

| 项目 | 值 | 证据 |
|------|------|------|
| **版本** | 3.1 | bundle.json: "version": "3.1" |
| **许可证** | Apache License 2.0 | LICENSE 文件 |

---

## 相关链接

- **架构文档**：[03_Architecture.md](03_Architecture.md)
- **目录结构**：[02_Directory_Structure.md](02_Directory_Structure.md)
- **N-API 文档**：[04_NAPI_Interface.md](04_NAPI_Interface.md)
- **内部 API**：[05_Internal_API.md](05_Internal_API.md)
- **构建系统**：[06_GN_Targets.md](06_GN_Targets.md)
- **安全分析**：[08_Security_Analysis.md](08_Security_Analysis.md)

---

## 快速开始

### 查看对外 API

如果需要使用 JS API 调用显示管理功能，请参阅：
- [04_NAPI_Interface.md](04_NAPI_Interface.md) - 完整的 N-API 清单

### 了解架构实现

如果需要了解内部实现细节，请参阅：
- [03_Architecture.md](03_Architecture.md) - 完整架构说明

### 构建与定制

如果需要修改构建配置或添加新功能，请参阅：
- [06_GN_Targets.md](06_GN_Targets.md) - GN 构建说明
- [07_Build_Artifacts.md](07_Build_Artifacts.md) - 编译产物说明
- [appendix/Config_Flags.md](appendix/Config_Flags.md) - 配置标志

### 安全审查

如果需要了解安全风险或进行安全审查，请参阅：
- [08_Security_Analysis.md](08_Security_Analysis.md) - 安全风险评审

---

## 常见问题

参见 [09_Troubleshooting.md](09_Troubleshooting.md) 获取常见问题的解决方案。

---

## 文档更新记录

- **2026-02-06**：初始版本 v1.0，基于代码扫描生成
