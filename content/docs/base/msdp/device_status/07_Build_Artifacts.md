# 编译产物说明

## 目的

本文档详细说明 `device_status` 模块的编译产物，包括输出文件、安装路径、运行时加载关系和运行时依赖。

---

## 产物分类

### 1. 共享库（Shared Libraries）

**说明**：`.z.so` 文件，运行时动态链接。

| 产物名称 | 输出路径 | 安装路径 | 加载方式 | 说明 |
|---------|---------|---------|---------|--------|
| `libdevicestatus_service.z.so` | `out/ohos-arm64/lib/` | `system/lib/` | SA 启动加载 | 主 System Ability 服务（SA 2902） |
| `libdevicestatus_client.z.so` | `out/ohos-arm64/lib/` | `system/lib/` | 动态链接 | 客户端库，供 Native 应用使用 |
| `libdevicestatus_util.z.so` | `out/ohos-arm64/lib/` | `system/lib/` | 动态链接 | 工具库，被服务端和客户端使用 |
| `libdevicestatus_ipc.z.so` | `out/ohos-arm64/lib/` | 动态链接 | IPC 通信库 |
| `devicestatus_algo.z.so` | `out/ohos-arm64/lib/` | 动态链接 | MSDP 算法库实现 |
| `libstationary.z.so` | `out/ohos-arm64/lib/module/` | 动态链接 | 静止状态 N-API 模块 |
| `libdevicestatus_napi.z.so` | `out/ohos-arm64/lib/module/` | 动态链接 | 设备状态 N-API 模块（v1） |
| `libdraginteraction.z.so` | `out/ohos-arm64/lib/module/` | 动态链接 | 拖拽交互 N-API 模块 |
| `intention_service.so` | `out/ohos-arm64/lib/` | 动态链接 | Intention 业务逻辑服务 |

### 2. 可执行文件（Executables）

| 产物名称 | 输出路径 | 安装路径 | 说明 |
|---------|---------|---------|---------|--------|
| `vdevadm` | `out/ohos-arm64/bin/` | `system/bin/` | 虚拟设备管理工具 |

### 3. System Ability 配置

| 产物名称 | 输出路径 | 安装路径 | 说明 |
|---------|---------|---------|---------|--------|
| `2902.json` | `system/profile/` | `system/profile/` | SA 2902 配置文件，系统启动时读取 |

### 4. Ark 字节码（Ark Bytecode）

| 产物名称 | 输出路径 | 安装路径 | 说明 |
|---------|---------|---------|---------|--------|
| `multimodalawareness_devicestatus.abc` | `system/framework/` | Ark 字节码 | 设备状态 ETS 模块编译产物 |
| `drag_abc.abc` | `system/framework/` | Ark 字节码 | 拖拽 ETS 模块编译产物 |
| `coordination_abc.abc` | `system/framework/` | Ark 字节码 | 协同 ETS 模块编译产物 |

---

## 安装路径

### 系统目录

```
/system/
├── bin/
│   └── vdevadm                    # 虚拟设备管理工具
├── lib/
│   ├── libdevicestatus_service.z.so  # 主服务库
│   ├── libdevicestatus_client.z.so  # 客户端库
│   ├── libdevicestatus_util.z.so      # 工具库
│   ├── libdevicestatus_ipc.z.so        # IPC 库
│   ├── libdevicestatus_algo.z.so       # 算法库
│   ├── libstationary.z.so             # N-API 模块
│   ├── libdevicestatus_napi.z.so      # N-API 模块
│   └── libdraginteraction.z.so        # N-API 模块
├── module/
│   ├── devicestatus/
│   │   ├── libstationary.z.so
│   │   ├── libdevicestatus_napi.z.so
│   │   └── libdraginteraction.z.so
└── framework/
    ├── multimodalawareness_devicestatus.abc
    ├── drag_abc.abc
    └── coordination_abc.abc
└── profile/
    └── 2902.json
```

---

## 运行时加载关系

### 1. 服务启动流程

```
System 启动
    ↓
[SA Manager]
    ↓
读取 SA Profile (2902.json)
    ↓
启动进程 "msdp"
    ↓
加载 libdevicestatus_service.z.so
    ↓
[DeviceStatusService::OnStart()]
    ↓
    初始化所有子系统
    ↓
    发布 IntentionService 到 SAMGR
```

### 2. N-API 模块加载

```
ArkTS/JS 应用
    ↓
导入 N-API 模块
    ↓
    import station from '@ohos.stationary'        // 静止状态（legacy）
    import deviceStatus from '@ohos.multimodalAwareness.deviceStatus'  // 设备状态 v1
    import motion from '@ohos.multimodalAwareness.motion'             // 运动感知
    import onScreen from '@ohos.multimodalAwareness.onScreen'         // 屏幕感知
    └── ...
    ↓
    N-API 模块注册到 ArkTS 运行时
```

### 3. Intention 插件加载

```
DeviceStatusService::Init()
    ↓
[PluginManager::LoadPlugin()]
    ↓
    加载各个插件
    │   ├── DragPlugin
    │   ├── CooperatePlugin
    │   ├── StationaryPlugin
    │   ├── BoomerangPlugin
    │   └── OnScreenPlugin
    └── 插件就绪
```

---

## 运行时依赖

### 服务端依赖

| 依赖 | 来源 | 说明 |
|--------|--------|------|--------|
| **SAMGR** | `safwk:system_ability_fwk` | SystemAbility 框架 |
| **BundleManager** | `bundle_framework:appexecfwk_core` | Bundle 管理 |
| **AccessToken** | `access_token:libaccesstoken_sdk` | 访问令牌 |
| **Eventhandler** | `eventhandler:libeventhandler` | 事件处理 |
| **MMI HDI** | `sensor:sensor_interface_native` | 传感器 HDI |
| **WindowManager** | `window_manager:libwm` | 窗口管理 |
| **Input** | `multimodalinput:libmmi-client` | 输入子系统 |
| **CommonEventService** | `common_event_service:cesfwk_innerkits` | 通用事件服务 |
| **HiSysEvent** | `hiviewdfx:hisysevent` | 系统事件上报 |

### N-API 模块依赖

| 模块 | 主要依赖 | 说明 |
|--------|---------|--------|
| **所有 N-API 模块** | `napi` | N-API 框架 | 提供 JS 运行时 |
| **客户端 Native** | `devicestatus_client` | 提供底层 IPC 通信 |

---

## 目标文件映射

### 产物到 GN Targets

| 产物 | GN Target | BUILD.gn 文件 |
|--------|-----------|----------|
| `libdevicestatus_service.z.so` | `devicestatus_service` | `services/BUILD.gn` |
| `libdevicestatus_client.z.so` | `devicestatus_client` | `interfaces/innerkits/BUILD.gn` |
| `libstationary.z.so` | `stationary` | `frameworks/js/napi/BUILD.gn` |
| `libdevicestatus_napi.z.so` | `devicestatus_napi` | `frameworks/js/napi/device_status/BUILD.gn` |
| `libdraginteraction.z.so` | `draginteraction` | `frameworks/js/napi/interaction/drag/BUILD.gn` |

---

## Rust 产物（条件）

**说明**：Rust 实现产物仅在 `device_status_rust_enabled = true` 时编译。

| 产物 | 类型 | 条件 |
|--------|--------|------|
| `libfusion_ipc_server_ffi.z.so` | shared_library | `device_status_rust_enabled` |
| `fusion_client_ffi` | rust | `device_status_rust_enabled` |
| `fusion_data_binding.z.so` | shared_library | `device_status_rust_enabled` |
| `fusion_drag_server_ffi` | rust | `device_status_rust_enabled` |

---

## 动态库依赖

### 模块间依赖关系

```
libdevicestatus_client.z.so
├── 依赖 libdevicestatus_util.z.so
└── 依赖 libdevicestatus_ipc.z.so

libdevicestatus_service.z.so
├── 依赖 libdevicestatus_util.z.so
├── 依赖 libdevicestatus_ipc.z.so
├── 依赖 intention_service.so
└── 依赖 drag_auth.so

intention_service.so
├── 依赖 intention_epoll.so
├── 依赖 intention_prototype.so
├── 依赖 intention_timer_manager.so
└── 依赖各种插件
```

---

## 资源文件

| 资源类型 | 文件路径 | 说明 |
|---------|---------|------|--------|
| **拖拽图标** | `etc/drag_icon/` | SVG、PNG 格式 | `libdevicestatus_service.z.so` 使用的图标资源 |

---

## 加载顺序

### 系统启动时的加载顺序

1. SAMGR 启动
2. 加载 SA Profile
3. 启动 "msdp" 进程
4. 加载 `libdevicestatus_service.z.so`
5. 执行 DeviceStatusService::OnStart()
6. 初始化各个子系统
7. 等待 IPC 连接

### N-API 模块加载

1. ArkTS/JS 应用启动
2. 应用导入 N-API 模块
3. N-API 模块调用 Native 接口
4. Native 接口通过 IPC 连接到服务

---

## 构建配置

### Feature Flags 的影响

| Feature | 影响的产物 | 代码路径 |
|---------|------------------|----------|
| `device_status_intention_framework` | 所有 Intention 插件 | `intention/` 目录 |
| `device_status_rust_enabled` | 所有 Rust 产物 | `rust/` 目录 |
| `device_status_interaction_coordination` | 协同相关代码 | `intention/cooperate/` 目录 |
| `device_status_drag_enable_animation` | 拖拽动画相关 | `services/interaction/drag/` 目录 |

---

## 相关跳转

- **[01_Overview](01_Overview.md)** - 项目概览
- **[02_Directory_Structure](02_Directory_Structure.md)** - 目录结构详解
- **[03_Architecture](03_Architecture.md)** - 架构设计
- **[04_N-API_Reference](04_N-API_Reference.md)** - JavaScript API 参考
- **[05_Inner_API](05_Inner_API.md)** - 内部 API
- **[06_GN_Targets](06_GN_Targets.md)** - GN 目标梳理

---

## 更新记录

- **初始版本**: 2026-02-06
- **代码版本**: HEAD commit of `/base/msdp/device_status`
