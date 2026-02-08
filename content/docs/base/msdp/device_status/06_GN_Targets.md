# GN Targets 梳理

## 目的

本文档详细说明 `device_status` 模块的 GN 构建系统，包括关键 targets、依赖关系、编译产物和配置选项。

---

## 构建系统概览

### GN 配置文件

**主配置文件**: `device_status.gni`

**位置**: `/base/msdp/device_status/device_status.gni`

**主要配置项**：

| 配置项 | 默认值 | 说明 |
|--------|---------|------|
| `device_status_intention_framework` | true | 启用 Intention 框架（插件系统） |
| `device_status_rust_enabled` | false | 启用 Rust 实现替代方案 |
| `device_status_interaction_coordination` | false | 启用协同交互功能 |
| `device_status_drag_enable_monitor` | true | 启用拖拽监控 |
| `device_status_drag_enable_interceptor` | false | 启用拖拽拦截器 |
| `device_status_drag_enable_animation` | false | 启用拖拽动画 |
| `device_status_performance_check` | true | 启用性能检查 |
| `device_status_sensor_enable` | true | 启用传感器支持 |
| `device_status_memmgr_enable` | false | 启用内存管理 |

**条件编译宏**：

```cpp
// Intention 框架
#ifdef OHOS_BUILD_ENABLE_INTENTION_FRAMEWORK
// ... Intention 相关代码
#endif

// 协同功能
#ifdef OHOS_BUILD_ENABLE_COORDINATION
// ... 协同相关代码
#endif

// 拖拽功能
#ifdef OHOS_DRAG_ENABLE_MONITOR
// ... 拖拽监控相关代码
#endif
```

---

## 主要 Target 类型

### 1. ohos_shared_library

**说明**：共享动态链接库，输出 `.z.so` 文件。

**服务层 targets**：

| Target 名称 | 输出文件 | 说明 |
|------------|--------|------|
| `devicestatus_service` | `libdevicestatus_service.z.so` | 主 System Ability 服务 (SA 2902) |
| `intention_service` | `.so` | Intention 业务逻辑服务 |
| `devicestatus_static_service` | `.a` | 服务静态库变体 |
| `devicestatus_sa_profile` | `2902.json` | SA 配置文件 |
| `interaction_drag` | `.so` | 拖拽交互库 |
| `drag_auth` | `.so` | 拖拽鉴权库 |
| `intention_drag_server` | `.so` | 拖拽服务实现 |
| `intention_drag_client` | `.so` | 拖拽客户端实现 |
| `intention_cooperate_server` | `.so` | 协同服务实现 |
| `intention_cooperate_client` | `.so` | 协同客户端实现 |
| `intention_cooperate` | `.so` | 协同插件库 |
| `intention_socket_session_manager` | `.so` | Socket 会话管理 |
| `intention_socket_connection` | `.so` | Socket 连接库 |
| `intention_tunnel` | `.so` | IPC 隧道库 |
| `intention_prototype` | `.so` | Intention 基础原型库 |
| `intention_epoll` | `.so` | Epoll 事件循环库 |
| `intention_timer_manager` | `.so` | 定时器管理库 |
| `intention_plugin_manager` | `.so` | 插件管理器库 |
| `intention_device_manager` | `.so` | 设备管理库 |
| `intention_input_adapter` | `.so` | MMI 输入适配器 |
| `intention_dsoftbus_adapter` | `.so` | DSoftBus 适配器 |
| `intention_ddm_adapter` | `.so` | Distributed Device Manager 适配器 |
| `intention_common_event_adapter` | `.so` | Common Event Service 适配器 |

**客户端库 targets**：

| Target 名称 | 输出文件 | 说明 |
|------------|--------|------|
| `devicestatus_client` | `libdevicestatus_client.z.so` | 主客户端库 |
| `drag_data_util` | `.so` | 拖拽数据工具库 |

**工具库 targets**：

| Target 名称 | 输出文件 | 说明 |
|------------|--------|------|
| `devicestatus_util` | `.so` | 公共工具库 |
| `devicestatus_ipc` | `.so` | IPC 工具库 |
| `json_parser` | `.so` | JSON 解析库 |
| `devicestatus_algo` | `.so` | 算法库实现 |
| `custom_config_parser` | `.so` | 配置解析库 |

**N-API 模块 targets**：

| Target 名称 | 输出文件 | 说明 |
|------------|--------|------|
| `stationary` | `libstationary.z.so` | 静止状态 N-API |
| `devicestatus_napi` | `libdevicestatus_napi.z.so` | 设备状态 N-API (v1) |
| `draginteraction` | `libdraginteraction.z.so` | 拖拽交互 N-API |
| `inputdevicecooperate` | `.so` | 输入设备协同 N-API |

### 2. ohos_source_set

**说明**：可重用的源代码集合，不生成独立的输出文件。

**示例**：`devicestatus_util` - 公共工具源集合

---

### 3. ohos_sa_profile

**说明**：System Ability 配置文件。

| Target 名称 | 输出文件 | 说明 |
|------------|--------|------|
| `devicestatus_sa_profile` | `2902.json` | SA 2902 配置 |

**SA 配置内容**（2902.json）：

```json
{
  "process": "msdp",
  "systemability": [{
    "name": 2902,
    "libpath": "libdevicestatus_service.z.so",
    "run-on-create": true,
    "distributed": false,
    "dump_level": 1
  }]
}
```

**配置说明**：
- `process`: 运行进程名 "msdp"
- `name`: SA ID 2902
- `libpath`: 服务库路径
- `run-on-create`: 系统启动时自动启动
- `distributed`: 非分布式服务
- `dump_level`: dump 支持级别

---

### 4. ETS/ANI Targets

**说明**：ArkUI Next 绑定生成的库和字节码。

**主要 targets**：

| Target 名称 | 输出文件 | 说明 |
|------------|--------|------|
| `multimodalawareness_devicestatus_ani` | `.so` + `.abc` | 设备状态 ETS 绑定 |
| `DragInteraction` | `.so` | 拖拽交互 ETS 绑定 |

**输出文件**：
- `.so` - 共享库
- `.abc` - Ark 字节码文件

---

### 5. Rust Targets（条件编译）

**说明**：Rust 实现的 targets，仅在 `device_status_rust_enabled = true` 时编译。

**主要 targets**：

| Target 名称 | 输出文件 | 类型 |
|------------|--------|------|
| `fusion_services_binding` | `.so` | shared_library - Rust 服务绑定 |
| `fusion_client_ffi` | (rust) | Rust 客户端 FFI |
| `fusion_data_binding` | `.so` | shared_library - Rust 数据绑定 |
| `fusion_ipc_server_ffi` | (rust) | Rust IPC 服务端 FFI |
| `fusion_drag_server_ffi` | (rust) | Rust 拖拽服务 FFI |
| `fusion_scheduler_test` | (test) | Rust 调度器测试 |

---

## 依赖关系图

### 服务层依赖

```
devicestatus_service
├── intention_service (条件)
│   ├── intention_plugin_manager
│   ├── intention_epoll
│   ├── intention_prototype
│   ├── intention_timer_manager
│   ├── intention_device_manager
│   └── 各种插件 (drag, cooperate, etc.)
├── interaction_drag
│   ├── drag_auth
│   └── 拖拽动画相关 (条件)
├── devicestatus_ipc
├── devicestatus_util
├── json_parser
├── custom_config_parser
└── drag_auth
```

### 客户端依赖

```
devicestatus_client
├── devicestatus_util
└── devicestatus_ipc
```

### Intention 框架依赖

```
intention_service
├── intention_plugin_manager
├── intention_epoll
├── intention_prototype
├── intention_timer_manager
├── intention_device_manager
└── 各种适配器 (input, dsoftbus, ddm)
```

---

## 编译宏定义

### 全局宏（来自 device_status.gni）

| 宏 | 默认值 | 启用条件 | 说明 |
|------|---------|---------|------|
| `OHOS_BUILD_ENABLE_INTENTION_FRAMEWORK` | true | - | 启用 Intention 框架 |
| `OHOS_BUILD_ENABLE_RUST_IMPL` | false | `device_status_rust_enabled` | 启用 Rust 实现 |
| `OHOS_BUILD_ENABLE_COORDINATION` | false | `device_status_interaction_coordination` | 启用协同功能 |
| `ENABLE_PERFORMANCE_CHECK` | true | - | 启用性能检查 |
| `OHOS_DRAG_ENABLE_MONITOR` | true | - | 启用拖拽监控 |
| `DEVICE_STATUS_SENSOR_ENABLE` | true | `device_status_sensor_enable` | 启用传感器支持 |

### 条件编译示例

```cpp
// 拖拽监控代码
#ifdef OHOS_DRAG_ENABLE_MONITOR
// 拖拽监控相关实现
#endif

// 协同代码
#ifdef OHOS_BUILD_ENABLE_COORDINATION
// 协同相关实现
#endif
```

---

## 编译产物

### 主要输出文件

| 类型 | 路径 | 文件类型 | 说明 |
|------|---------|-----------|--------|
| **共享库** | `system/lib/` | `.z.so` | 动态链接库 |
| **可执行文件** | `system/bin/` | 二进制可执行文件 |
| **SA 配置** | `system/profile/` | JSON 配置文件 |
| **Ark 字节码** | `system/framework/` | `.abc` | Ark 字节码 |
| **资源文件** | `system/etc/device_status/` | 静态资源（图标等） |

### 关键产物

| 产物 | 安装路径 | 说明 |
|--------|-----------|------|
| `libdevicestatus_service.z.so` | `system/lib/` | 主服务库，SA 2902 |
| `libdevicestatus_client.z.so` | `system/lib/` | 客户端库 |
| `libstationary.z.so` | `system/lib/module/` | 静止状态 N-API 模块 |
| `libdraginteraction.z.so` | `system/lib/module/devicestatus/` | 拖拽交互 N-API 模块 |

---

## 构建命令

### 完整编译命令

```bash
# 生成编译配置
gn gen out --root=target_os --default-targets=//base/msdp/device_status:devicestatus_service

# 编译服务
ninja -C out/ohos-arm64 //base/msdp/device_status:devicestatus_service

# 安装到系统
hdc install libdevicestatus_service.z.so
```

### 特定功能编译

```bash
# 仅编译 Intention 框架
gn gen out --root=target_os --args="device_status_intention_framework=true device_status_rust_enabled=false"

# 编译拖拽功能（不启用动画）
gn gen out --root=target_os --args="device_status_drag_enable_animation=false"
```

---

## 配置选项说明

### Intention 框架特性

| 特性 | 启用后影响 | 相关代码 |
|--------|------------|----------|
| **插件系统** | 完整启用 | 所有插件（Drag、Cooperate、Stationary、Boomerang、OnScreen） |
| **任务调度** | 异步和同步任务支持 | TaskScheduler |
| **定时器** | 定时任务管理 | TimerManager |
| **Socket 通信** | 高性能数据传输 | SocketSessionManager |

### Rust 实现

| 特性 | 状态 | 说明 |
|--------|-------|------|
| **默认禁用** | `device_status_rust_enabled = false` | 所有 Rust targets 不参与编译 |
| **实验性** | 替代 C++ 实现 | 用于性能优化和内存安全验证 |
| **条件编译** | 需显式启用才能编译 | 需要修改 `device_status.gni` |

---

## 相关跳转

- **[01_Overview](01_Overview.md)** - 项目概览
- **[02_Directory_Structure](02_Directory_Structure.md)** - 目录结构详解
- **[03_Architecture](03_Architecture.md)** - 架构设计
- **[04_N-API_Reference](04_N-API_Reference.md)** - JavaScript API 参考
- **[05_Inner_API](05_Inner_API.md)** - 内部 API
- **[07_Build_Artifacts](07_Build_Artifacts.md)** - 编译产物说明

---

## 更新记录

- **初始版本**: 2026-02-06
- **代码版本**: HEAD commit of `/base/msdp/device_status`
