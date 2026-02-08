# Settings 应用概览

> 本文档：OpenHarmony Standard Settings 应用快速入门指南

---

## 目的

本文档提供 OpenHarmony Standard Settings 应用的概览，帮助新人快速理解项目定位、核心能力和运行环境。

## 适用范围

- 目标读者：OpenHarmony 应用开发者、系统开发者
- 版本范围：Settings 3.1 (Standard System)
- 时间范围：截至 2026-02-06

---

## 项目定位

Settings 是 OpenHarmony 标准系统中的**系统应用**，提供用户配置系统设置的入口。

### 核心功能

1. **设置管理**：
   - 读取系统配置（ getValue / getValueSync）
   - 修改系统配置（ setValue / setValueSync）
   - 键观察者（ registerKeyObserver / unregisterKeyObserver）

2. **网络设置**：
   - 打开网络管理设置页面
   - 打开输入法设置页面

3. **智能场景**：
   - 免打扰模式查询（ isDoNotDisturbEnabled）
   - 通知权限查询（ isNotifyAllowedInDoNotDisturb）

4. **系统控制**：
   - 启用/禁用飞行模式（ enableAirplaneMode）
   - 悬浮窗权限查询（ canShowFloating）

### 目标设备

- 标准系统（standard）
- 设备类型：默认、平板（default, tablet）
- 支持系统：OpenHarmony

---

## 运行环境

### 系统要求

- **OpenHarmony 版本**：API 23
- **编译 SDK**：23
- **兼容 SDK**：23
- **目标 SDK**：23
- **运行时**：OpenHarmony

### 系统能力（SystemCapability）

Settings 应用声明了以下系统能力：

```json
// bundle.json
{
  "syscap": [
    "SystemCapability.Applications.Settings.Core",
    "SystemCapability.Applications.IntelligentScene"
  ]
}
```

**说明**：
- `Settings.Core`：设置管理核心能力
- `IntelligentScene`：智能场景能力（免打扰模式等）

---

## 技术架构概览

### 多语言支持

Settings 应用支持三种 Native API 绑定方式：

1. **N-API**（Node-API）：
   - 模块：settings, intelligentscene
   - 语言：C++
   - JS ↔ C++ 绑定
   - 文件：napi/settings/, napi/intelligentscene/

2. **ANI**（Ark Native Interface）：
   - 模块：settings_ani, intelligentscene_ani
   - 语言：C++ + ArkTS
   - ArkTS ↔ C++ 绑定
   - 文件：ani/settings/, ani/intelligentscene/

3. **CJ FFI**（C-JSON FFI）：
   - 模块：cj_settings_ffi
   - 语言：C++
   - C ABI 绑定
   - 文件：cj/settings/

### 数据存储

Settings 使用以下数据存储机制：

1. **DataShare**（数据共享）：
   - URI：`dataability:///com.ohos.settingsdata.DataAbility`
   - 表名：global, system, secure
   - 字段：KEYWORD, VALUE

2. **RDB**（关系型数据库）：
   - 搜索功能使用
   - 搜索数据持久化

### 进程模型

Settings 应用采用**单进程架构**，不提供 SystemAbility 服务。

**关键点**：
- 作为客户端使用其他 SA（如 BundleManagerService）
- 不实现自定义 IPC 接口
- 数据访问通过 DataShare 框架

---

## 核心概念

### 1. 设置表（Settings Tables）

Settings 数据分为三个表：

| 表名 | 说明 | 访问权限 |
|--------|--------|-----------|
| global | 全局设置（所有应用共享） | 默认可读，部分可写需权限 |
| system | 系统设置（系统级） | 仅系统应用可写 |
| secure | 安全设置（敏感数据） | 需特殊权限 |

### 2. 观察者模式（Observer Pattern）

Settings 支持对设置键的变更监听：

- **注册**：`registerKeyObserver(name, domainName, observer)`
- **取消注册**：`unregisterKeyObserver(name, domainName)`
- **实现**：基于 DataAbilityObserverStub

### 3. 同步/异步模型

Settings API 支持多种调用模式：

| 模式 | API 类型 | 特点 |
|--------|-----------|------|
| 同步 | getValueSync, setValueSync | 阻塞调用，直接返回结果 |
| 异步（Callback） | getValue, setValue | 回调模式，避免阻塞 |
| 异步（Promise） | - | 未使用（预留） |

---

## 项目边界

### Settings 应用做什么

✅ **在范围内**：
- 提供设置管理 API（N-API/ANI/CJ FFI）
- 提供 UI 界面（product/phone）
- 读取/修改系统设置
- 监听设置变更
- 打开特定设置页面

❌ **不在范围内**：
- 不提供 SystemAbility 服务
- 不直接修改系统配置文件
- 不实现自定义 IPC 接口
- 不直接访问硬件设备

---

## 相关跳转

- **[01_Positioning.md](01_Positioning.md)** - 详细的项目定位与边界
- **[02_Directory_Structure.md](02_Directory_Structure.md)** - 完整的目录结构
- **[03_Architecture.md](03_Architecture.md)** - 架构设计与数据流
- **[04_NAPI_API.md](04_NAPI_API.md)** - 对外 API 详细文档

---

**最后更新**：2026-02-06 00:11:23
