# 项目定位与边界

> Settings 应用的详细定位、边界与核心能力

---

## 目的

本文档详细说明 Settings 应用的定位、项目边界、核心能力和关键概念。

## 适用范围

- 目标读者：开发者、架构师、系统工程师
- 项目：@ohos/settings (Settings 3.1)
- 版本：API 23 (OpenHarmony Standard)

---

## 项目定位

Settings 是 OpenHarmony 标准系统中的**系统应用**，是用户配置系统设置的主要入口。

### 角色定位

1. **设置服务提供者**：
   - 为其他应用提供设置管理 API
   - 支持三种绑定方式：N-API / ANI / CJ FFI

2. **系统 UI 提供者**：
   - 提供设置界面
   - 支持手机设备（phone）
   - 预留可穿戴设备（wearable）

3. **数据消费者**：
   - 通过 DataShare 访问设置数据
   - 通过 RDB 存储搜索数据

### 不属于的角色

❌ **不是**：
- 不是系统服务提供者（不实现 SA）
- 不是设备驱动（不直接访问硬件）
- 不是框架组件（依赖 OpenHarmony 框架）

---

## 核心能力

### 1. 设置管理

**API 概览**（详细见 [04_NAPI_API.md](04_NAPI_API.md)）：

| API 类别 | 主要功能 | 文档位置 |
|-----------|---------|-----------|
| 读取设置 | getValue / getValueSync | [04_NAPI_API.md](04_NAPI_API.md) |
| 写入设置 | setValue / setValueSync | [04_NAPI_API.md](04_NAPI_API.md) |
| 键观察者 | registerKeyObserver / unregisterKeyObserver | [04_NAPI_API.md](04_NAPI_API.md) |
| URI 管理 | getURI / getUriSync | [04_NAPI_API.md](04_NAPI_API.md) |

### 2. 系统控制

| 功能 | API | 说明 |
|--------|------|------|
| 飞行模式 | enableAirplaneMode() | 启用/禁用飞行模式 |
| 悬浮窗 | canShowFloating() | 查询是否可显示悬浮窗 |

### 3. 页面导航

| 功能 | API | 说明 |
|--------|------|------|
| 网络管理 | openNetworkManagerSettings() | 打开网络管理设置页面 |
| 输入法设置 | openInputMethodSettings() | 打开输入法设置页面 |
| 输入法详情 | openInputMethodDetail() | 打开输入法详情页面 |

### 4. 智能场景

| 功能 | API | 说明 |
|--------|------|------|
| 免打扰模式 | isDoNotDisturbEnabled() | 查询免打扰模式状态 |
| 通知权限 | isNotifyAllowedInDoNotDisturb() | 查询免打扰模式下通知权限 |

---

## 项目边界

### 职责范围

#### ✅ Settings 应用负责

1. **设置数据管理**：
   - 维护三张表：global, system, secure
   - 提供 DataShare 接口
   - 支持观察者模式

2. **API 提供**：
   - N-API：settings, intelligentscene
   - ANI：settings_ani, intelligentscene_ani
   - CJ FFI：cj_settings_ffi

3. **UI 实现**：
   - 主设置列表（product/phone）
   - 各功能页面（蓝牙、Wi-Fi、亮度、VPN 等）
   - 公共组件库（common/component）

4. **工具支持**：
   - 搜索功能（common/search）
   - 基础模型和工具（common/utils, common/settingsBase）

#### ❌ Settings 应用不负责

1. **系统服务提供**：
   - 不提供 SystemAbility
   - 不实现自定义 IPC 接口
   - 不实现 Binder 服务

2. **系统配置管理**：
   - 不直接修改系统配置文件（如 build.prop）
   - 不提供系统属性设置 API
   - 依赖其他系统服务（如 AbilityManager）

3. **硬件控制**：
   - 不直接控制硬件设备（蓝牙、Wi-Fi 芯片等）
   - 通过其他系统服务间接控制

4. **安全策略执行**：
   - 不执行权限检查（权限在应用层声明）
   - 不实现沙箱机制
   - 依赖系统权限框架

---

## 关键概念

### 1. 多语言 API 绑定

Settings 支持**三种 Native API 绑定方式**，满足不同场景需求：

| 绑定方式 | 语言 | 性能 | 使用场景 |
|-----------|------|--------|----------|
| N-API | C++ | 高性能 | 第三方应用 |
| ANI | C++ + ArkTS | 高性能 | ArkTS 应用 |
| CJ FFI | C++ | 中等性能 | C ABI 调用 |

**技术对比**：
- **N-API**：标准 Node-API，跨语言支持好
- **ANI**：Ark Native Interface，ArkTS 原生支持
- **CJ FFI**：C-JSON FFI，兼容性最好

### 2. 数据访问分层

Settings 使用**分层数据访问**架构：

```
┌─────────────────────────────────────────┐
│  应用层（ArkTS/C++）            │
├─────────────────────────────────────────┤
│  API 层（N-API/ANI/CJ FFI）    │
├─────────────────────────────────────────┤
│  数据层（DataShare）              │
├─────────────────────────────────────────┤
│  存储层（DataAbility）          │
└─────────────────────────────────────────┘
```

**说明**：
- 应用层通过 API 访问数据
- API 层调用 DataShare 框架
- DataShare 通过 IPC 访问存储
- 存储层由系统 DataAbility 提供

### 3. 单进程模型

Settings 采用**单进程架构**：

```
┌──────────────────────────────────────────┐
│  Settings 应用进程                │
│                                 │
│  ┌────────────────────────────┐     │
│  │ API 层（N-API/ANI/   │     │
│  │  CJ FFI）              │     │
│  └────────────────────────────┘     │
│                                 │
│  ┌────────────────────────────┐     │
│  │ UI 层（ArkTS）        │     │
│  └────────────────────────────┘     │
│                                 │
│  ┌────────────────────────────┐     │
│  │ 数据层（DataShare）    │     │
│  └────────────────────────────┘     │
│                                 │
│  ┌────────────────────────────┐     │
│  │ 搜索层（RDB）        │     │
│  └────────────────────────────┘     │
└──────────────────────────────────────────┘
```

**关键点**：
- 所有组件运行在同一进程
- 无 IPC 开销（除 DataShare 系统调用）
- 观察者通过 DataAbilityObserverStub 实现

---

## 运行环境

### 支持的设备类型

| 设备类型 | 状态 | 说明 |
|----------|--------|------|
| default | ✅ 激活 | 默认设备（手机/平板） |
| tablet | ✅ 激活 | 平板设备 |
| wearable | ⚠️ 已注释 | 可穿戴设备（预留） |

### 系统要求

- **OpenHarmony 版本**：API 23+
- **子系统**：applications
- **系统能力**：Settings.Core, IntelligentScene
- **运行时**：OpenHarmony

---

## 依赖关系

### 外部依赖（来自 bundle.json）

| 依赖组件 | 用途 |
|-----------|--------|
| ability_base | 基础能力支持 |
| ability_runtime | 能力运行时 |
| ace_engine | ACE 引擎 |
| c_utils | C 工具库 |
| data_share | 数据共享框架 |
| hilog | 日志系统 |
| relational_store | 关系型数据库 |
| os_account | 账户管理 |
| napi | N-API 支持 |
| ipc | IPC 框架 |
| runtime_core | 运行时核心 |
| hisysevent | 系统事件 |
| samgr | 系统能力管理器 |
| distributed_notification_service | 分布式通知 |
| access_token | 访问令牌 |

### 使用的系统服务

Settings 作为**客户端**使用以下系统服务：

1. **BundleManagerService**（通过 SA ID）：
   - 用途：获取当前应用 Bundle 信息
   - 文件：native/settings/src/napi_bundle_util.cpp:34

2. **DataAbility Service**（通过 DataShare）：
   - URI：`dataability:///com.ohos.settingsdata.DataAbility`
   - 用途：访问设置数据
   - 表名：global, system, secure

---

## 设计约束

### API 兼容性

Settings API 遵循以下设计约束：

1. **参数验证**：
   - 必须提供键名（name）
   - 可选：域（domainName）、默认值（defaultValue）
   - 同步/异步分开设计

2. **错误处理**：
   - 权限错误码：201
   - 内部错误码：35200001
   - IPC 错误码：1600002, 1600003

3. **数据安全**：
   - 敏感数据使用 `DEFAULT_ANONYMOUS = "******"` 掩码
   - 日志使用匿名化处理
   - 遵循数据访问边界

---

## 相关跳转

- **[00_Overview.md](00_Overview.md)** - 项目概览
- **[02_Directory_Structure.md](02_Directory_Structure.md)** - 目录结构与模块职责
- **[03_Architecture.md](03_Architecture.md)** - 架构说明
- **[04_NAPI_API.md](04_NAPI_API.md)** - 对外 API 详细文档

---

**最后更新**：2026-02-06 00:11:23
