# 架构设计

## 系统架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                      OpenHarmony 系统架构                        │
├─────────────────────────────────────────────────────────────────┤
│  用户态                                                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                    用户应用层                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              ↓                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                    系统能力层                                 │ │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐        │ │
│  │  │ AbilityMS    │ │ BundleMS     │ │ DMSchedMgr  │ ...   │ │
│  │  └──────────────┘ └──────────────┘ └──────────────┘        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              ↑                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │              safwk_lite (foundation 进程)                    │ │
│  │                    [服务进程容器]                             │ │
│  │                                                              │ │
│  │   main() → OHOS_SystemInit() → SAMGR_Bootstrap()           │ │
│  │                            ↓                                │ │
│  │                    [无限循环: pause()]                      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              ↑                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │              Samgr Lite (系统能力管理框架)                    │ │
│  │   - 服务注册/发现                                             │ │
│  │   - IPC 路由                                                 │ │
│  │   - 生命周期管理                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              ↑                                   │
│  内核态                                                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │              LiteOS-A / Linux 内核                           │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## 进程模型

### 单进程多服务模式

`safwk_lite` 是一个**单进程多服务**的容器进程：

| 特性 | 描述 |
|------|------|
| 进程名 | `foundation` |
| 进程角色 | 服务进程容器 |
| 运行模式 | 无限循环（`pause()` 阻塞等待信号） |
| 服务加载 | 条件化编译（feature flags 控制） |

### 进程启动流程

```
1. init 进程启动 foundation 进程
         ↓
2. main() 函数开始执行
         ↓
3. 调用 OHOS_SystemInit()（可被覆盖的弱符号）
         ↓
4. SAMGR_Bootstrap() 初始化 Samgr 框架
         ↓
5. 各子服务（abilityms, bundlems 等）初始化
         ↓
6. 进入无限循环：pause() 等待信号
```

## Samgr 集成

### Samgr Bootstrap 流程

**代码证据**: `src/main.c:30-36`

```c
void __attribute__((weak)) OHOS_SystemInit(void)
{
    SAMGR_Bootstrap();
}
```

### Samgr 职责

| 功能 | 说明 |
|------|------|
| 服务注册 | 允许各子服务向 Samgr 注册能力 |
| 服务发现 | Consumer 通过 Samgr 查找 Provider |
| IPC 路由 | 处理进程间通信 |
| 生命周期 | 管理服务启动/停止 |

## 条件化编译 (Feature Flags)

### Feature 列表

| Feature | 默认值 | 控制服务 |
|---------|--------|---------|
| `safwk_lite_feature_enable_abilityms` | true | Ability Management Service |
| `safwk_lite_feature_enable_bundlems` | true | Bundle Management Service |
| `safwk_lite_feature_enable_dtbschedmgr` | true | Distributed Scheduler Manager |

### 条件编译逻辑

**代码证据**: `BUILD.gn:54-64`

```gn
if (safwk_lite_feature_enable_abilityms == true) {
  deps += [ "${aafwk_lite_path}/services/abilitymgr_lite:abilityms" ]
}
if (safwk_lite_feature_enable_bundlems == true) {
  deps += [ "${appexecfwk_lite_path}/services/bundlemgr_lite:bundlems" }
}
if (board_name != "hispark_aries") {
  if (safwk_lite_feature_enable_dtbschedmgr == true) {
    deps += [ "//foundation/ability/dmsfwk_lite:dtbschedmgr" ]
  }
}
```

**注意**: `dtbschedmgr` 在 `hispark_aries` 开发板上被禁用。

## 线程模型

### 主线程职责

| 阶段 | 操作 |
|------|------|
| 初始化 | 调用 `SAMGR_Bootstrap()` |
| 运行 | 进入 `pause()` 无限阻塞 |

### 说明

- **单线程模型**：使用 `pause()` 系统调用阻塞主线程
- **信号处理**：通过信号机制唤醒处理（如 SIGTERM）
- **子服务线程**：各子服务（abilityms等）可能有独立线程池
