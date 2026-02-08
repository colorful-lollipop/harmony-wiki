# 项目概览 (Overview)

> MemMgr 组件定位、边界与核心能力

## 1. 简介

### 1.1 组件定位

MemMgr (Memory Manager) 是 OpenHarmony **全局资源调度子系统**的核心组件，负责系统内存的智能管理与优化。

**定位**: 基于应用生命周期状态，动态管理进程回收优先级，通过内存回收、查杀等手段保障系统内存供给。

**证据**: `README_zh.md:18-21`
```
内存管理部件位于全局资源调度子系统中，基于应用的生命周期状态，
更新进程回收优先级列表，通过内存回收、查杀等手段管理系统内存，保障内存供给。
```

### 1.2 核心能力

| 能力 | 描述 | 模块 |
|------|------|------|
| 进程优先级管理 | 根据应用状态计算回收优先级 (-1000 ~ 400) | `reclaim_priority_manager/` |
| 内存回收策略 | 配置 kswapd/zswapd 参数，协调回收机制 | `reclaim_strategy_manager/` |
| 低内存查杀 | PSI 压力事件触发，按优先级杀进程 | `kill_strategy_manager/` |
| 事件驱动 | 监听应用/窗口/内存压力等多维事件 | `event/` |
| 配置管理 | XML 配置解析，动态调整回收参数 | `config/` |

### 1.3 版本信息

| 属性 | 值 | 证据 |
|------|-----|------|
| 组件名 | `memmgr` | `bundle.json:14` |
| 版本 | `3.1.0` | `bundle.json:3` |
| 子系统 | `resourceschedule` | `bundle.json:15` |
| ROM | `1000KB` | `bundle.json:17` |
| RAM | `4316KB` | `bundle.json:18` |

---

## 2. 运行环境

### 2.1 依赖子系统

MemMgr 依赖以下 OpenHarmony 子系统：

| 依赖组件 | 用途 | 证据 |
|----------|------|------|
| `ipc` | IPC 通信框架 | `BUILD.gn:122` |
| `safwk` | System Ability 框架 | `BUILD.gn:127` |
| `samgr` | 服务管理 | `BUILD.gn:128` |
| `ability_runtime` | 应用运行时 | `BUILD.gn:109-112` |
| `bundle_framework` | Bundle 框架 | `BUILD.gn:114-115` |
| `os_account` | 多账户管理 | `BUILD.gn:125` |
| `common_event_service` | 公共事件 | `BUILD.gn:117-118` |
| `access_token` | 权限管理 | `BUILD.gn:113,143` |
| `background_task_mgr` | 后台任务 (可选) | `BUILD.gn:131-133` |
| `libxml2` | XML 解析 | `BUILD.gn:124` |
| `json` | JSON 处理 | `bundle.json:37` |

**证据**: `bundle.json:19-38`, `services/memmgrservice/BUILD.gn:107-128`

### 2.2 System Ability 配置

MemMgr 以 SA 形式运行：

| 属性 | 值 |
|------|-----|
| SA ID | `1909` |
| 进程名 | `memmgrservice` |
| 库路径 | `libmemmgrservice.z.so` |
| 启动时机 | `run-on-create: true` (系统启动时) |
| 分布式 | `false` (仅本地) |
| Dump 级别 | `1` |

**证据**: `sa_profile/1909.json:1-13`

```json
{
    "process": "memmgrservice",
    "systemability": [{
        "name": 1909,
        "libpath": "libmemmgrservice.z.so",
        "run-on-create": true,
        "distributed": false,
        "dump_level": 1
    }]
}
```

---

## 3. 目录结构与模块职责

```
memmgr/
├── common/                          # [通用工具]
│   ├── include/
│   │   ├── kernel_interface.h      # Kernel 接口封装
│   │   ├── memmgr_log.h             # 日志封装 (hilog)
│   │   ├── memmgr_config_manager.h # 配置管理器
│   │   ├── single_instance.h        # 单例模板
│   │   └── xml_helper.h            # XML 解析工具
│   └── src/
│       ├── kernel_interface.cpp     # 与内核交互
│       ├── memmgr_config_manager.cpp# 配置加载
│       └── config/                   # 各类配置实现
│           ├── avail_buffer_config.h
│           ├── kill_config.h
│           ├── reclaim_config.h
│           └── ...
│
├── interface/innerkits/             # [Inner API - 对内 C++ 接口]
│   ├── include/
│   │   ├── mem_mgr_client.h         # 客户端单例
│   │   ├── mem_mgr_proxy.h          # IPC 代理
│   │   ├── i_mem_mgr.h              # 服务端接口
│   │   ├── mem_mgr_constant.h      # 常量定义
│   │   └── ...
│   └── src/
│       ├── mem_mgr_client.cpp
│       ├── mem_mgr_proxy.cpp
│       └── ...
│
├── services/memmgrservice/          # [SA 服务实现]
│   ├── include/
│   │   ├── mem_mgr_service.h        # SA 主类
│   │   ├── mem_mgr_stub.h           # IPC 存根
│   │   ├── event/                   # 事件中心
│   │   │   ├── mem_mgr_event_center.h
│   │   │   ├── app_state_observer.h
│   │   │   ├── window_visibility_observer.h
│   │   │   └── ...
│   │   ├── reclaim_priority_manager/# 回收优先级
│   │   │   ├── reclaim_priority_manager.h
│   │   │   ├── process_priority_info.h
│   │   │   └── ...
│   │   ├── reclaim_strategy_manager/# 回收策略
│   │   │   ├── reclaim_strategy_manager.h
│   │   │   ├── reclaim_param.h
│   │   │   └── ...
│   │   ├── kill_strategy_manager/   # 查杀策略
│   │   │   └── low_memory_killer.h
│   │   └── ...
│   └── src/
│       ├── mem_mgr_service.cpp
│       ├── mem_mgr_stub.cpp
│       ├── event/                    # 事件实现
│       ├── reclaim_priority_manager/ # 优先级实现
│       ├── reclaim_strategy_manager/# 策略实现
│       └── ...
│
├── profile/                         # 配置文件
│   ├── BUILD.gn
│   └── memmgr_config.xml           # 默认配置模板
│
├── sa_profile/                     # SA 配置
│   ├── BUILD.gn
│   └── 1909.json                  # SA ID 1909
│
├── figures/                        # 文档图片
│   └── zh-cn_image_fwk.png         # 架构图
│
├── bundle.json                     # 组件描述
├── memmgr.gni                      # GN 变量定义
└── README.md / README_zh.md        # 项目说明
```

### 模块职责映射表

| 模块路径 | 职责 | 关键类 |
|----------|------|--------|
| `event/` | 事件监听与分发 | `MemMgrEventCenter`, `*Observer` |
| `reclaim_priority_manager/` | 进程优先级计算 | `ReclaimPriorityManager` |
| `reclaim_strategy_manager/` | 内存回收参数调整 | `ReclaimStrategyManager` |
| `kill_strategy_manager/` | 低内存查杀执行 | `LowMemoryKiller` |
| `config/` | XML 配置解析 | `MemmgrConfigManager` |
| `kernel_interface.cpp` | 内核命令下发 | `KernelInterface` |

---

## 4. 关键概念

### 4.1 进程回收优先级 (Reclaim Priority)

MemMgr 定义了进程的回收/查杀优先级，数值越**低**越容易被回收：

| 优先级 | 描述 | 证据 |
|--------|------|------|
| `-1000` | 系统进程 (白名单) | `README_zh.md:78` |
| `-800` | 常驻进程 (可拉起) | `README_zh.md:79` |
| `0` | 前台应用 | `README_zh.md:80` |
| `100` | 后台短时任务 / 关联 extension | `README_zh.md:81` |
| `200` | 后台感知应用 (导航/播放) | `README_zh.md:82` |
| `260` | 连接分布式设备的后台应用 | `README_zh.md:83` |
| `400` | 普通后台应用 | `README_zh.md:84` |

**证据**: `README_zh.md:76-84`

### 4.2 内存水线 (Memory Waterline)

查杀策略基于内存水线触发：

| 水线 (MB) | 可杀最小优先级 | 证据 |
|-----------|----------------|------|
| 500 | 400 | `README_zh.md:108` |
| 400 | 300 | `README_zh.md:109` |
| 300 | 200 | `README_zh.md:110` |
| 200 | 100 | `README_zh.md:111` |
| 100 | 0 | `README_zh.md:112` |

### 4.3 事件驱动模型

MemMgr 通过事件中心监听系统事件，触发优先级更新：

```
外部事件 → 事件中心 → 回收优先级管理 → 回收/查杀策略 → Kernel
```

**监听的事件类型**:

| 事件源 | 监听器 | 描述 |
|--------|--------|------|
| 应用状态 | `AppStateObserver` | 应用前后台切换 |
| 窗口可见性 | `WindowVisibilityObserver` | 窗口显隐变化 |
| 内存压力 | `MemoryPressureObserver` | PSI 压力事件 |
| kswapd 状态 | `KswapdObserver` | 内核回收状态 |
| 后台任务 | `BgTaskObserver` | 后台任务状态 |
| 扩展连接 | `ExtensionConnectionObserver` | 分布式连接 |
| 公共事件 | `CommonEventObserver` | 系统公共事件 |
| 账户变化 | `AccountObserver` | 用户账户切换 |

**证据**: `services/memmgrservice/include/event/` 全部头文件

---

## 5. 配置说明

### 5.1 运行时配置路径

```
/etc/memmgr/memmgr_config.xml
```

**证据**: `README_zh.md:124`

### 5.2 配置项分类

| 配置项 | 用途 | 头文件 |
|--------|------|--------|
| `availbufferSize` | 可用缓冲区阈值 | `avail_buffer_config.h` |
| `ZswapdParam` | zswapd 回收参数 | `reclaim_config.h` |
| `killConfig` | 查杀级别配置 | `kill_config.h` |
| `nandlife` | 磁盘寿命管控 | `nand_life_config.h` |
| `purgeablemem` | 可清理内存 (条件) | `purgeablemem_config.h` |

**证据**: `common/include/config/` 全部头文件

---

## 6. 相关跳转

- **架构详解**: [01_Architecture.md](./01_Architecture.md)
- **Inner API**: [02_Inner_API.md](./02_Inner_API.md)
- **构建配置**: [03_Build.md](./03_Build.md)
- **安全评审**: [04_Security.md](./04_Security.md)
- **导航总览**: [SUMMARY.md](./SUMMARY.md)

---

*文档版本: 3.1.0 | 最后更新: 2026-02-06*
