# sensors_miscdevice_lite 架构说明

## 2.1 架构概览

### 2.1.1 系统架构图

基于 OpenHarmony 标准组件架构和小器件元数据，推断的架构如下：

```
┌─────────────────────────────────────────────────────────────────────┐
│                         用户应用层                                    │
│              (调用 @ohos/sensors_miscdevice_lite API)                 │
├─────────────────────────────────────────────────────────────────────┤
│                         N-API 层                                      │
│         (JavaScript 接口绑定，位于 interfaces/plugin)                   │
├─────────────────────────────────────────────────────────────────────┤
│                      Native Framework 层                              │
│              (C++ 实现，位于 frameworks/native)                        │
├─────────────────────────────────────────────────────────────────────┤
│                      Service 层                                        │
│           (MiscDevice Service，位于 services/)                        │
│                    ↓ IPC (可能)                                        │
├─────────────────────────────────────────────────────────────────────┤
│                         HDI 层                                        │
│              (Hardware Device Interface)                               │
├─────────────────────────────────────────────────────────────────────┤
│                      HDF 框架                                          │
│            (Hardware Driver Foundation)                               │
├─────────────────────────────────────────────────────────────────────┤
│                      Driver 层                                         │
│            (马达驱动、LED驱动，OSAL + Platform)                         │
├─────────────────────────────────────────────────────────────────────┤
│                       硬件层                                          │
│                    (I2C/SPI/GPIO)                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.1.2 层级职责

| 层级 | 职责 | 证据状态 |
|------|------|----------|
| **应用层** | 业务调用，提供 JS/ArkTS 接口调用 | ✅ 文档存在 |
| **N-API 层** | JS 到 C++ 的绑定 | ❌ 无代码（见 sensors_miscdevice） |
| **Native Framework** | 客户端-服务端连接管理 | ❌ 无代码 |
| **Service 层** | 业务逻辑实现、硬件控制 | ❌ 无代码 |
| **HDI 层** | 稳定硬件接口定义 | ❌ 无代码 |
| **HDF 框架** | 驱动框架、OSAL 抽象 | ❌ 无代码 |
| **Driver 层** | 具体马达/LED 驱动 | ❌ 无代码 |
| **硬件层** | 物理马达、LED 设备 | 硬件相关 |

---

## 2.2 组件模块

### 2.2.1 模块清单（推断）

| 模块 | 路径（推断） | 职责 |
|------|-------------|------|
| **N-API 插件** | `interfaces/plugin/` | JS API 注册与绑定 |
| **Native 接口** | `interfaces/native/` | Native API 定义 |
| **Framework** | `frameworks/native/` | 客户端框架 |
| **服务实现** | `services/miscdevice_service/` | 核心业务逻辑 |
| **SA 配置** | `sa_profile/` | System Ability 配置 |
| **工具库** | `utils/` | 公共组件 |

**证据来源**: librarian agent 返回的 `sensors_miscdevice` 仓库结构

---

## 2.3 核心能力流程

### 2.3.1 振动控制流程（推断）

```
用户应用 (ArkTS)
    ↓
vibrator.startVibration(effect, attribute)
    ↓
N-API (interfaces/plugin/)
    ↓
Native Framework (frameworks/native/)
    ↓
MiscDevice Service (IPC)
    ↓
HDI Interface
    ↓
HDF Driver (马达驱动)
    ↓
Hardware (振动马达)
```

### 2.3.2 LED 控制流程（推断）

```
用户应用 (ArkTS)
    ↓
LED Control API
    ↓
N-API (interfaces/plugin/)
    ↓
Native Framework (frameworks/native/)
    ↓
MiscDevice Service (IPC)
    ↓
HDI Interface
    ↓
HDF Driver (LED驱动)
    ↓
Hardware (LED 灯)
```

---

## 2.4 跨系统差异

### 2.4.1 Lite vs Standard 系统

| 特性 | Lite (本仓库) | Standard (sensors_miscdevice) |
|------|---------------|------------------------------|
| **代码位置** | 本仓库（空） | https://github.com/openharmony/sensors_miscdevice |
| **N-API** | 无 | 有完整实现 |
| **IPC 机制** | 可能简化 | 完整 SAMgr |
| **HDF** | 轻量版 | 完整版 |
| **目标设备** | MCU 类设备 | 应用处理器 |

### 2.4.2 适配策略

```
┌────────────────────────────────────────────┐
│         sensors_miscdevice_lite             │
│         (轻量系统适配层)                     │
├────────────────────────────────────────────┤
│    sensors_miscdevice (标准系统实现)         │
│    复制/适配关键代码到 Lite 环境            │
└────────────────────────────────────────────┘
```

---

## 2.5 依赖关系

### 2.5.1 外部依赖（推断）

| 依赖项 | 类型 | 用途 |
|--------|------|------|
| **sensors_sensor_lite** | 同子系统 | 传感器公共接口 |
| **IPC 框架** | 系统组件 | 进程间通信 |
| **HDF** | 系统组件 | 驱动框架 |
| **OSAL** | 系统组件 | 操作系统抽象 |

**证据来源**: `bundle.json` 元数据显示无显式依赖：
```json
{
    "deps": {
        "components": [],
        "third_party": []
    }
}
```

---

## 2.6 线程模型（推断）

### 2.6.1 振动控制线程

```
主线程 (JS 调用)
    ↓
Native 线程 (参数校验)
    ↓
Service 线程 (IPC 调用)
    ↓
驱动线程 (HDF 异步)
```

### 2.6.2 同步/异步模式

| API | 模式 | 说明 |
|-----|------|------|
| `startVibration()` | 异步 | 启动振动后立即返回 |
| `stopVibration()` | 同步 | 停止振动后返回 |
| `isSupportEffect()` | 异步/回调 | 查询效果支持状态 |

---

## 2.7 稳定性评估

### 2.7.1 接口稳定性

| 接口类型 | 稳定性 | 说明 |
|----------|--------|------|
| **JS API** | 稳定 | 基于 N-API，版本兼容 |
| **Native API** | 稳定 | HDI 接口，版本管理 |
| **HDF 接口** | 稳定 | 内核接口，长期支持 |

### 2.7.2 架构约束

- **多内核部署**: 支持 LiteOS、Linux
- **OSAL 抽象**: 跨平台兼容
- **HDI 接口**: 稳定的硬件抽象层
- **服务化架构**: 驱动作为系统服务暴露

---

## 2.8 相关文档

### 官方资源
- [sensors_miscdevice GitHub](https://github.com/openharmony/sensors_miscdevice)
- [Vibrator 开发指南](https://gitee.com/openharmony/docs/blob/master/en/application-dev/device/sensor/vibrator-guidelines.md)
- [泛 Sensor 子系统文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/泛Sensor子系统.md)

### 本地文档
- [01_Overview.md](./01_Overview.md) - 项目概览
- [03_API.md](./03_API.md) - API 文档
- [05_Security.md](./05_Security.md) - 安全评审
