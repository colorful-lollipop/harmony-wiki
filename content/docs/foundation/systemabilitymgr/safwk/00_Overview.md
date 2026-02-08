# SAFWK 项目概览

## 项目定位

**System Ability Framework (SAFWK)** 是 OpenHarmony 系统中负责管理系统能力 (System Ability, SA) 的核心框架。

### 核心职责

| 职责 | 描述 |
|------|------|
| **SA 生命周期管理** | 负责系统能力的加载、启动、停止、卸载 |
| **SA 注册与发现** | 提供 SA 注册机制和查询接口 |
| **IPC 通信基础** | 基于 IRemoteBroker 的进程间通信框架 |
| **按需启动** | 支持 SA 按需延迟启动，优化系统资源 |
| **权限控制** | 基于 AccessTokenKit 的权限验证机制 |

### 适用范围

- **目标系统**: OpenHarmony Standard (标准系统)
- **开发语言**: C++ (核心), Rust (绑定)
- **用户角色**: 系统能力开发者、框架维护者

## 目录结构

```
safwk/
├── etc/                          # 配置文件
│   └── profile/
│       ├── foundation.cfg        # Foundation 进程权限配置
│       ├── foundation_trust.json  # 可信 SA ID 列表
│       └── foundation_permission_desc.json  # 权限描述注册表
│
├── interfaces/                    # 对外 API
│   └── innerkits/safwk/
│       ├── system_ability.h      # SystemAbility 基类 (对外)
│       ├── system_ability_ondemand_reason.h  # 按需启动原因
│       ├── api_cache_manager.h   # API 缓存管理
│       ├── expire_lru_cache.h    # LRU 缓存工具
│       └── rust/                  # Rust 绑定
│           ├── src/
│           │   ├── ability.rs     # Rust SA trait
│           │   └── wrapper.rs     # FFI 包装器
│           └── examples/          # Rust 示例
│
├── services/                      # 核心服务实现
│   └── safwk/
│       ├── src/
│       │   ├── main.cpp          # 守护进程入口
│       │   ├── system_ability.cpp # SA 基类实现
│       │   ├── local_ability_manager.cpp  # LAM 实现
│       │   ├── local_ability_manager_stub.cpp  # IPC 接口
│       │   ├── system_ability_start.cpp  # 启动逻辑
│       │   ├── system_ability_ondemand_reason.cpp  # 按需原因
│       │   ├── api_cache_manager.cpp  # API 缓存
│       │   └── ffrt_handler.cpp  # FFRT 任务处理
│       └── include/
│           ├── system_ability.h   # SA 基类 (内部)
│           ├── local_ability_manager.h  # LAM 接口
│           └── safwk_log.h       # 日志定义
│
├── svc/                           # 服务控制工具
│   ├── src/
│   │   ├── main.cpp              # 工具入口
│   │   └── svc_control.cpp       # 控制命令实现
│   └── include/
│       └── svc_control.h         # 控制接口
│
├── wiki/                          # 文档 (本文档)
└── test/                          # 测试用例 (不计入文档范围)
```

## 关键概念

### System Ability (系统能力)

系统能力是 OpenHarmony 中提供系统级服务的组件，通常运行在独立进程中：

- **特征**: 跨进程访问、系统级服务、生命周期受框架管理
- **示例**: 媒体服务、账户服务、分布式调度服务
- **实现**: 继承 `SystemAbility` 基类

### LocalAbilityManager (本地能力管理器)

管理特定进程中所有本地 SA 的生命周期：

| 功能 | 描述 |
|------|------|
| SA 注册 | 接收 SA 注册请求 |
| 依赖管理 | 处理 SA 间依赖关系 |
| 按需启动 | 根据请求启动目标 SA |
| 状态监控 | 跟踪 SA 运行状态 |

### IPC 框架

基于 `IRemoteBroker/Proxy/Stub` 三元组的进程间通信：

```
┌─────────────┐         IPC          ┌─────────────┐
│   Client     │  ── MessageParcel ──→ │   Server    │
│ IRemoteProxy │                       │ IRemoteStub │
└─────────────┘                       └─────────────┘
       ↑                                    ↓
       │                            ┌─────────────┐
       │                            │  Service    │
       │                            │ IRemoteBroker│
       │                            └─────────────┘
```

## 运行环境

### 系统依赖

| 组件 | 版本要求 | 用途 |
|------|----------|------|
| OpenHarmony | 3.1+ | 基础系统 |
| samgr | - | 系统能力管理器 |
| ipc | - | 进程间通信 |
| access_token | - | 权限管理 |

### 进程模型

```
init 进程
    │
    ├── foundation 进程 (承载多个 SA)
    │       │
    │       ├── SAFWK 守护进程
    │       │       │
    │       │       ├── LocalAbilityManager
    │       │       ├── SystemAbility
    │       │       └── API Cache Manager
    │       │
    │       └── 其他 SA (如 dsoftbus, sensors)
    │
    └── 独立 SA 进程 (通过 sa_main 启动)
```

## 开发流程

### 开发 SA 的基本步骤

1. **定义 IPC 接口**: 继承 `IRemoteBroker`
2. **实现 Proxy/Stub**: 客户端代理和服务端存根
3. **继承 SystemAbility**: 实现业务逻辑
4. **注册 SA**: 使用 `REGISTER_SYSTEM_ABILITY_BY_ID` 宏
5. **配置 profile**: 创建 SA profile JSON 文件
6. **配置 .cfg**: 创建进程启动配置文件

### 代码示例

```cpp
// 1. 定义 IPC 接口
class IMyAbility : public IRemoteBroker {
public:
    virtual int32_t DoSomething(int32_t param) = 0;
    DECLARE_INTERFACE_DESCRIPTOR(u"OHOS.test.IMyAbility");
};

// 2. 实现 SA
class MyAbility : public SystemAbility {
public:
    MyAbility(int32_t saId, bool runOnCreate) 
        : SystemAbility(saId, runOnCreate) {}
    
    void OnStart() override {
        Publish(this);  // 必须调用
    }
    
    int32_t DoSomething(int32_t param) override {
        return param + 1;
    }
};

// 3. 注册 SA
REGISTER_SYSTEM_ABILITY_BY_ID(MyAbility, MY_ABILITY_ID, true);
```

## 相关文档

| 文档 | 链接 |
|------|------|
| 架构设计 | [01_Architecture.md](01_Architecture.md) |
| API 接口 | [02_APIs.md](02_APIs.md) |
| 构建系统 | [03_Build.md](03_Build.md) |
| 安全评审 | [04_Security.md](04_Security.md) |
| 常见问题 | [05_FAQ.md](05_FAQ.md) |
