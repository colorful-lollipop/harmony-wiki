# 项目概览

## 1. 项目定位与边界

### 1.1 项目定位

**resourceschedule_qos_manager** 是 OpenHarmony 系统中负责**并发任务调度权限管控**的核心服务部件。

| 属性 | 描述 |
|------|------|
| **所属子系统** | resourceschedule（资源调度） |
| **部件名称** | qos_manager |
| **版本** | 3.1 |
| **系统能力** | SystemCapability.Resourceschedule.QoS.Core |

### 1.2 核心职责

本部件服务于 **FFRT (Fair Future Runtime)** 并发编程框架，为特定线程提供调用底层 QoS (Quality of Service) 和 RTG (Runtime Governor) 接口的能力。

```
┌─────────────────────────────────────────────────────────────────┐
│                    qos_manager 核心职责                          │
├─────────────────────────────────────────────────────────────────┤
│  1. 接收帧感知调度 (frame_aware_sched) 发送的场景信息            │
│  2. 基于 uid 赋予系统服务/前台应用 RTG 操作权限                  │
│  3. 基于 uid 赋予线程 QoS 等级设置权限                           │
│  4. 将多级 QoS 配置下发到内核 (nice, uclamp 等参数)             │
│  5. 统一管理 RTG 分组的创建、销毁等操作                          │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 边界说明

| 边界 | 描述 | 代码位置 |
|------|------|----------|
| **输入边界** | 帧感知调度插件发送的场景信息 | `services/src/concurrent_task_service.cpp` |
| **输出边界** | 内核 syscalls (ioctl) | `services/src/qos_interface.cpp` |
| **控制边界** | RTG 分组、QoS 等级 | `qos/qos.cpp` |
| **依赖边界** | frame_aware_sched, RTG 驱动 | `bundle.json` external_deps |

### 1.4 不在范围内

| 功能 | 说明 | 归属 |
|------|------|------|
| **JS/ArkTS API** | 无 N-API 层，需在其他仓库查找 | arkui/ace_engine |
| **libtask_controller** | 动态库实现在外部仓库 | resource_schedule_service |
| **内核驱动** | sched_qos_ctrl, sched_rtg_ctrl | kernel 子系统 |
| **GEWU AI 推理** | libgewu_client.z.so 动态加载 | 独立仓库 |

---

## 2. 核心能力

### 2.1 RTG 权限管控与分组管理

#### 2.1.1 基于 uid 的 RTG 权限管控

接收帧感知调度发送的场景信息，为以下 uid 赋予 RTG 操作权限：

| uid 类型 | 权限 | 用途 |
|----------|------|------|
| 特权 uid | RTG 全权限 | 系统服务 |
| 前台 app uid | RTG 操作权限 | 前台应用线程 |
| 其他 uid | 受限权限 | 后台应用 |

**证据**: `bundle.json` → `external_deps` → `frame_aware_sched:rtg_interface`

#### 2.1.2 RTG 分组管理

统一管理 RTG 分组，支持：

- **创建分组**: 外部场景请求时创建
- **销毁分组**: 场景结束时销毁
- **分组查询**: 查询线程所属 RTG id

**内核节点**: `/proc/self/sched_rtg_ctrl`

**证据**: `services/src/qos_interface.cpp:57-83` → `EnableRtg()` 函数

### 2.2 多级 QoS 权限管控与信息下发

#### 2.2.1 QoS 等级定义

| 等级 | 枚举值 | 描述 | 典型场景 |
|------|--------|------|----------|
| QOS_BACKGROUND | 0 | 后台任务 | 日志同步、数据备份 |
| QOS_UTILITY | 1 | 实用工具 | 文件压缩、索引构建 |
| QOS_DEFAULT | 2 | 默认级别 | 普通任务 |
| QOS_USER_INITIATED | 3 | 用户主动发起 | UI 交互响应 |
| QOS_DEADLINE_REQUEST | 4 | 截止时间请求 | 音视频播放 |
| QOS_USER_INTERACTIVE | 5 | 用户交互 | 触摸响应、动画 |

**证据**: `interfaces/kits/c/qos.h:50-80` → `QoS_Level` 枚举

#### 2.2.2 基于 uid 的 QoS 权限管控

与 RTG 权限管控类似，根据 uid 授予 QoS 等级设置权限。

#### 2.2.3 多级 QoS 信息下发

将不同场景下的 QoS 等级参数下发到内核：

| 参数 | 作用 | 内核映射 |
|------|------|----------|
| nice | 进程nice值 | SCHED_OTHER |
| latencyNice | 延迟敏感度 | CFS 调度 |
| uclampMin | CPU 最小占用 | SCHED_CFS |
| uclampMax | CPU 最大占用 | SCHED_CFS |
| rtPriority | 实时优先级 | SCHED_FIFO/RR |

**证据**: `services/include/qos_interface.h:58-65` → `QosPolicyData` 结构体

---

## 3. 运行环境

### 3.1 系统要求

| 要求 | 规格 |
|------|------|
| **OpenHarmony 版本** | 5.0+ |
| **系统类型** | standard |
| **ROM 占用** | 2048 KB |
| **RAM 占用** | 10240 KB |

### 3.2 进程环境

| 属性 | 值 |
|------|-----|
| **进程名** | concurrent_task_service |
| **UID** | system |
| **GID** | system, shell |
| **SELinux 上下文** | u:r:concurrent_task_service:s0 |
| **启动时机** | post-fs-data 阶段 |
| **进程优先级** | -20 (高优先级) |

**证据**: `etc/init/concurrent_task_service.cfg`

### 3.3 依赖环境

#### 3.3.1 系统组件依赖

```json
// bundle.json deps.components
[
  "ability_base",           // 能力基础
  "ability_runtime",        // 能力运行时
  "access_token",           // 访问令牌（权限）
  "config_policy",          // 配置策略
  "c_utils",                // C 工具库
  "frame_aware_sched",      // 帧感知调度（核心依赖）
  "hilog",                  // 日志系统
  "hitrace",                // 性能跟踪
  "init",                   // 启动框架
  "ipc",                    // 进程间通信
  "libxml2",                // XML 解析
  "safwk",                  // SA 框架
  "samgr"                   // 服务管理
]
```

#### 3.3.2 第三方依赖

无第三方依赖，所有依赖均为 OpenHarmony 系统组件。

---

## 4. 关键概念

### 4.1 QoS (Quality of Service)

**定义**: 线程的服务质量等级，用于告知内核线程的优先级和响应需求。

**作用机制**:
```
应用设置 QoS 等级 → qos_manager 权限校验 → 内核调度策略调整 → 线程获得相应调度优先级
```

**代码路径**: `qos/qos.cpp` → `QosController` 类

### 4.2 RTG (Runtime Governor)

**定义**: 运行时Governor，用于管理线程组的运行时间配额。

**主要操作**:
- **EnableRtg**: 启用 RTG 调度
- **AddThreadToProcRtg**: 将线程加入进程 RTG 组
- **RemoveThreadFromProcRtg**: 从 RTG 组移除线程

**代码路径**: `services/src/qos_interface.cpp` → `EnableRtg()`, `TrivalOpenRtgNode()`

### 4.3 SA (System Ability)

**定义**: OpenHarmony 系统能力，本模块以 SA 形式运行。

| 属性 | 值 |
|------|-----|
| **SA ID** | 1912 |
| **是否分布式** | false |
| **启动模式** | run-on-create |
| **dump 级别** | 1 |

**配置文件**: `sa_profile/1912.json`

### 4.4 uid (User Identifier)

**定义**: 用户标识符，用于区分不同应用和系统服务。

| uid 类型 | 范围 | 用途 |
|----------|------|------|
| ROOT_UID | 0 | root 用户 |
| SYSTEM_UID | 1000 | 系统服务 |
| APP_UID | > 1000 | 应用 |

**证据**: `services/include/qos_interface.h:26-27`

### 4.5 FFRT (Fair Future Runtime)

**定义**: OpenHarmony 并发编程框架，qos_manager 为其提供底层支撑。

**协作关系**:
```
FFRT → qos_manager → frame_aware_sched → 内核
```

---

## 5. 目录结构概览

```
qos_manager/
├── common/                    # 通用工具
│   └── src/concurrent_task_utils.cpp
├── etc/
│   ├── init/                  # 启动配置
│   │   └── concurrent_task_service.cfg
│   ├── param/                 # 系统参数
│   │   ├── ffrt.para
│   │   └── ffrt.para.dac
│   └── sa_profile/            # SA 配置
│       └── 1912.json
├── frameworks/
│   ├── concurrent_task_client/# IPC 客户端
│   │   ├── idl/               # IDL 接口定义
│   │   ├── include/
│   │   └── src/
│   └── native/                # NDK 桥接
│       └── qos_ndk.cpp
├── include/
│   └── concurrent_task_log.h
├── interfaces/
│   ├── inner_api/             # C++ 内部 API
│   └── kits/                  # NDK Kit 接口
│       └── c/qos.h
├── qos/                       # QoS 核心库
│   └── qos.cpp
├── services/                  # 系统服务
│   ├── include/
│   └── src/
├── test/                      # 测试用例
├── wiki/                      # 本 Wiki 文档
├── bundle.json
└── BUILD.gn
```

---

## 6. 相关文档

| 文档 | 链接 |
|------|------|
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| NDK API | [02_NDK_API.md](./02_NDK_API.md) |
| 内部 API | [03_Inner_API.md](./03_Inner_API.md) |
| 构建配置 | [04_Build_Targets.md](./04_Build_Targets.md) |
| 安全评审 | [05_Security.md](./05_Security.md) |
| 调用链 | [appendix/Callgraphs.md](./appendix/Callgraphs.md) |
| 配置参数 | [appendix/Config_Flags.md](./appendix/Config_Flags.md) |
