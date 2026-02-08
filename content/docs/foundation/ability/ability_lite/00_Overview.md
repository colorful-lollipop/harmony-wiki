# 项目概览

## 目的

本文档介绍 OpenHarmony ability_lite 组件的项目定位、核心能力、运行环境和关键概念。

## 适用范围

- **目标读者**: OpenHarmony 应用开发者、系统开发者、架构师
- **适用版本**: 3.1
- **适用系统类型**: mini (LiteOS-M)、small (Linux-based)

## 项目定位

ability_lite 是 OpenHarmony 轻量级系统的 **Ability 管理框架**，为 mini 和 small 系统提供应用生命周期管理、Ability 调度和进程管理能力。

### 核心定位

| 维度 | 说明 |
|------|------|
| **系统层级** | 基础平台服务 (foundation/ability) |
| **功能定位** | Ability 生命周期管理、进程调度、应用栈管理 |
| **目标设备** | 轻量级 IoT 设备（IP Camera、智能手表等） |
| **架构模式** | C/S 架构，AMS 作为系统服务运行在 foundation 进程 |

### 与标准系统对比

| 特性 | ability_lite (轻量) | aafwk (标准) |
|------|---------------------|--------------|
| 适用系统 | mini/small | standard |
| IPC 机制 | SAMGR Lite | Binder |
| Ability 类型 | Page/Service | Page/Service/Extension |
| 多任务支持 | 可选 (_MINI_MULTI_TASKS_) | 完整支持 |
| 分布式能力 | 基础支持 | 完整支持 |

## 核心能力

### 1. Ability 生命周期管理

```
UNINITIALIZED → INITIAL → INACTIVE → ACTIVE → BACKGROUND → STOP
```

- **状态转换**: 系统通过 AMS 协调 Ability 状态转换
- **生命周期回调**: 应用通过 `Ability` 类虚函数接收事件
- **任务调度**: 使用任务队列处理生命周期事件

### 2. Ability 类型支持

#### Page Ability
- 提供用户界面（UI）
- 支持 AbilitySlice 多页面管理
- 依赖 `ui_lite` 和 `surface_lite`

#### Service Ability
- 后台运行，无 UI
- 支持跨应用连接（Connect/Disconnect）
- 用于后台任务处理

### 3. 进程管理

- **AppSpawn**: 通过 appspawn 服务创建应用进程
- **进程生命周期**: 与应用生命周期绑定
- **权限加载**: 应用启动时加载权限配置

### 4. 应用栈管理

- **Mission Stack**: 管理 Page Ability 的显示顺序
- **Ability Stack**: 管理同一应用内 Ability 层级
- **任务记录**: 维护 Ability 启动历史

## 运行环境

### 硬件要求

| 系统类型 | 最小内存 | 存储 |
|----------|----------|------|
| mini | 256KB RAM | 1MB Flash |
| small | 4MB RAM | 16MB Flash |

### 软件依赖

#### 系统组件
- `samgr_lite`: 系统服务管理框架
- `bundle_framework_lite`: 包管理服务
- `hilog_lite`: 日志服务
- `kv_store`: 键值存储
- `ipc`: 进程间通信

#### 可选组件
- `ui_lite`: UI 框架（Page Ability 需要）
- `surface_lite`: 图形 surface（窗口需要）
- `window_manager_lite`: 窗口管理

### 进程模型

```
┌─────────────────────────────────────────┐
│         Foundation Process              │
│  ┌─────────────────────────────────┐    │
│  │    Ability Manager Service      │    │
│  │         (AMS)                   │    │
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │    Service Manager (SAMGR)      │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
                    │ IPC
                    ▼
┌─────────────────────────────────────────┐
│         Application Process             │
│  ┌─────────────────────────────────┐    │
│  │    AbilityKit (Client)          │    │
│  │    - Ability Thread             │    │
│  │    - Ability Scheduler          │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

## 关键概念

### Ability

Ability 是 OpenHarmony 应用的基本组成单元，一个应用可以包含多个 Ability。

**代码位置**: `interfaces/kits/ability_lite/ability.h:75`

```cpp
class Ability : public AbilityContext {
public:
    virtual void OnStart(const Want &want);
    virtual void OnActive(const Want &want);
    virtual void OnInactive();
    virtual void OnBackground();
    virtual void OnStop();
    virtual const SvcIdentity *OnConnect(const Want &want);
    virtual void OnDisconnect(const Want &want);
};
```

### Want

Want 是 Ability 间传递的意图信息载体，包含目标 Ability 的标识和传递的数据。

**代码位置**: `interfaces/kits/want_lite/want.h:57`

```cpp
typedef struct {
    ElementName *element;     // Ability 标识（deviceId/bundleName/abilityName）
    SvcIdentity *sid;         // 回调服务标识
    void *data;               // 携带数据
    uint16_t dataLength;      // 数据长度
    const char *appPath;      // 应用路径
    uint32_t mission;         // Mission ID
} Want;
```

### AbilitySlice

AbilitySlice 是 Page Ability 的页面片段，一个 Page Ability 可包含多个 Slice。

**适用条件**: 定义 `ability_lite_enable_ohos_appexecfwk_feature_ability = true`

### AMS (Ability Manager Service)

AMS 是系统服务，负责：
- Ability 生命周期调度
- 应用进程管理
- Ability 栈管理
- Service Ability 连接管理

**服务注册**: `services/abilitymgr_lite/src/ability_mgr_service.cpp:46`

```cpp
BOOL result = sm->RegisterService(AbilityMgrService::GetInstance());
```

### AbilityLoader

AbilityLoader 用于注册和加载 Ability 类。

**代码位置**: `interfaces/kits/ability_lite/ability_loader.h`

```cpp
#define REGISTER_AA(className) \
    AbilityLoader::GetInstance().RegisterAbility(#className, new className())
```

## 架构图

### 整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        Application Layer                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │   JS App    │  │ Native App  │  │    aa Tool (CLI)        │  │
│  └──────┬──────┘  └──────┬──────┘  └───────────┬─────────────┘  │
└─────────┼────────────────┼─────────────────────┼────────────────┘
          │                │                     │
          ▼                ▼                     ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Framework Layer                          │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │   N-API (aafwk) │  │   AbilityKit    │  │ AbilityManager  │  │
│  │   js_aafwk.cpp  │  │   ability.cpp   │  │   Client        │  │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘  │
└───────────┼────────────────────┼────────────────────┼───────────┘
            │                    │                    │
            └────────────────────┼────────────────────┘
                                 │ IPC (IpcIo)
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Service Layer                            │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │           Ability Manager Service (AMS)                 │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │    │
│  │  │   Feature   │  │   Handler   │  │  Task Manager   │  │    │
│  │  │   Invoke    │  │   Process   │  │   (Worker)      │  │    │
│  │  └─────────────┘  └─────────────┘  └─────────────────┘  │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

## 相关链接

- [目录结构](01_Directory_Structure.md)
- [架构说明](02_Architecture.md)
- [对外 Native API](03_Native_API.md)
- [Ability 框架官方文档](https://gitee.com/openharmony/docs/blob/master/en/readme/ability.md)
