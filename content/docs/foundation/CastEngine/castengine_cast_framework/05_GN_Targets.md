# GN 构建目标梳理

> 文档版本: 1.0.0
> 最后更新: 2026-02-06

## 目的

本文档详细说明 CastEngine 框架的 GN 构建系统，包括所有目标、依赖关系、编译产物和配置选项。

## GN 构建系统概览

### 构建入口

**主配置**: `BUILD.gn` (根目录)
**变量定义**: `cast_engine.gni`

**组件定义**: `bundle.json`

### 构建类型说明

| 类型 | 说明 | 示例 |
|-----|------|------|
| ohos_shared_library | 共享库 (.so) | libcast.z.so |
| ohos_static_library | 静态库 (.a) | cast_engine_common_sources |
| ohos_sa_profile | SA 配置文件 | 5526.json |
| ohos_prebuilt_etc | 预构建配置文件 | cast_engine_service.cfg |

---

## 主要 Targets

### 1. 根级 Targets

#### cast_engine_default_config

**类型**: config
**位置**: `BUILD.gn:14-30`

**作用**: 定义通用的编译选项和编译器标志

**内容**:
```gn
config("cast_engine_default_config") {
  cflags = [
    "-Wall", "-Wextra", "-Werror",
    "-Wno-shadow", "-Wno-unused-parameter",
    "-Wno-missing-field-initializers",
    "-FS", "-O2",
    "-D_FORTIFY_SOURCE=2",
    "-fvisibility=hidden",
    "-fvisibility-inlines-hidden"
  ]
  ldflags = [ "-Werror" ]
}
```

**安全特性**:
- 警告视为错误 (-Werror)
- FORTIFY_SOURCE 2 - 缓冲区溢出保护
- 符号隐藏 - 减少攻击面

---

### 2. 模块 Targets

#### cast_engine_common_sources

**类型**: ohos_static_library
**位置**: `common/BUILD.gn:16-45`

**输出**: libcast_engine_common.a（静态库）

**源文件**:
```
common/src/cast_engine_common_helper.cpp
common/src/cast_engine_dfx.cpp
```

**依赖**:
- 外部: access_token, c_utils, eventhandler, hilog, hisysevent, image, init, ipc, json
- 公共配置: cast_engine_default_config

**用途**: 提供跨模块共享的工具和辅助功能（日志、DFX 等）

#### cast_engine_service

**类型**: ohos_shared_library
**位置**: `service/BUILD.gn:23-72`

**输出**: libcast_engine_service.z.so（安装在 /system/lib64/）

**源文件**:
```
service/src/cast_service_listener_impl_proxy.cpp
service/src/cast_session_manager_service.cpp
service/src/cast_session_manager_service_stub.cpp
```

**内部依赖**:
- cast_client_inner
- cast_engine_common_sources
- cast_discovery (device_manager)
- cast_session (session)

**外部依赖**:
- ability_runtime, bundle_framework, c_utils, device_manager, dsoftbus
- graphic_surface, hilog, hisysevent, image_native, init, input
- ipc, openssl, os_account, player_framework, power_manager
- safwk, samgr

**用途**: System Ability 服务主库

#### cast_engine_client

**类型**: ohos_shared_library
**位置**: `interfaces/inner_api/BUILD.gn:20-49`

**输出**: libcast_engine_client.z.so（安装在 /system/lib64/）

**内部依赖**:
- cast_client_inner
- cast_engine_common_sources

**外部依赖**:
- c_utils, hilog, ipc, samgr, image_native

**用途**: 内部 C++ API 库

#### cast

**类型**: ohos_shared_library
**位置**: `interfaces/kits/js/BUILD.gn:22-67`

**输出**: libcast.z.so（安装在 /system/lib64/module/）

**源文件**:
```
interfaces/kits/js/src/napi_async_work.cpp
interfaces/kits/js/src/napi_callback.cpp
interfaces/kits/js/src/napi_cast_session.cpp
interfaces/kits/js/src/napi_cast_session_listener.cpp
interfaces/kits/js/src/napi_cast_session_manager.cpp
interfaces/kits/js/src/napi_cast_session_manager_listener.cpp
interfaces/kits/js/src/napi_castengine_enum.cpp
interfaces/kits/js/src/napi_castengine_utils.cpp
interfaces/kits/js/src/napi_mirror_player.cpp
interfaces/kits/js/src/napi_stream_player.cpp
interfaces/kits/js/src/napi_stream_player_listener.cpp
interfaces/kits/js/src/native_module.cpp
```

**内部依赖**:
- cast_engine_common_sources
- cast_engine_client

**外部依赖**:
- hilog, image, image_native, init, ipc, napi, samgr

**用途**: JavaScript N-API 模块

#### cast_client_inner

**类型**: ohos_static_library
**位置**: `client/BUILD.gn:22-45`

**输出**: 静态库

**源文件**:
```
client/src/cast_engine_service_load_callback.cpp
client/src/cast_service_listener_impl_stub.cpp
client/src/cast_session.cpp
client/src/cast_session_impl_proxy.cpp
client/src/cast_session_listener_impl_stub.cpp
client/src/cast_session_manager.cpp
client/src/cast_session_manager_adaptor.cpp
client/src/cast_session_manager_service_proxy.cpp
client/src/mirror_player.cpp
client/src/mirror_player_impl_proxy.cpp
client/src/stream_player.cpp
client/src/stream_player_impl_proxy.cpp
client/src/stream_player_listener_impl_stub.cpp
```

**依赖**: cast_engine_common_sources

**用途**: 客户端内部实现

### 3. 子模块 Targets

#### cast_discovery

**类型**: ohos_static_library
**位置**: `service/src/device_manager/BUILD.gn:23-63`

**源文件**:
```
service/src/device_manager/src/cast_device_data_manager.cpp
service/src/device_manager/src/connection_manager.cpp
service/src/device_manager/src/discovery_manager.cpp
```

**用途**: 设备发现和连接管理

#### cast_session

**类型**: ohos_static_library
**位置**: `service/src/session/BUILD.gn:15-52`

**源文件**:
```
service/src/session/src/cast_session_impl.cpp
service/src/session/src/cast_session_impl_stub.cpp
service/src/session/src/cast_session_listener_impl_proxy.cpp
service/src/session/src/cast_session_listeners.cpp
service/src/session/src/cast_session_state.cpp
```

**依赖**:
- cast_engine_common_sources
- cast_discovery
- cast_session_channel
- cast_session_mirror
- cast_session_rtsp
- cast_session_stream
- cast_session_utils

**用途**: 会话管理实现

#### cast_session_channel

**类型**: ohos_static_library
**位置**: `service/src/session/src/channel/BUILD.gn:16-50`

**源文件**:
```
service/src/session/src/channel/src/channel_manager.cpp
service/src/session/src/channel/src/softbus/softbus_connection.cpp
service/src/session/src/channel/src/softbus/softbus_wrapper.cpp
service/src/session/src/channel/src/tcp/tcp_connection.cpp
service/src/session/src/channel/src/tcp/tcp_socket.cpp
```

**用途**: 通道管理（SoftBus + TCP）

#### cast_session_rtsp

**类型**: ohos_static_library
**位置**: `service/src/session/src/rtsp/BUILD.gn:14-53`

**源文件**:
```
service/src/session/src/rtsp/src/rtsp_channel_manager.cpp
service/src/session/src/rtsp/src/rtsp_controller.cpp
service/src/session/src/rtsp/src/rtsp_package.cpp
service/src/session/src/rtsp/src/rtsp_param_info.cpp
service/src/session/src/rtsp/src/rtsp_parse.cpp
```

**用途**: RTSP 协议实现

#### cast_session_stream

**类型**: ohos_static_library
**位置**: `service/src/session/src/stream/BUILD.gn:14-74`

**源文件**:
```
service/src/session/src/stream/src/cast_stream_manager_client.cpp
service/src/session/src/stream/src/cast_stream_manager_server.cpp
service/src/session/src/stream/src/i_cast_stream_manager.cpp
service/src/session/src/stream/src/local/src/cast_local_file_channel_client.cpp
service/src/session/src/stream/src/local/src/cast_local_file_channel_common.cpp
service/src/session/src/stream/src/local/src/cast_local_file_channel_server.cpp
service/src/session/src/stream/src/local/src/local_data_source.cpp
service/src/session/src/stream/src/player/src/cast_stream_player.cpp
service/src/session/src/stream/src/player/src/cast_stream_player_manager.cpp
service/src/session/src/stream/src/player/src/cast_stream_player_utils.cpp
service/src/session/src/stream/src/player/src/remote_player_controller.cpp
service/src/session/src/stream/src/player/src/stream_player_impl_stub.cpp
service/src/session/src/stream/src/player/src/stream_player_listener_impl_proxy.cpp
```

**用途**: 流播放器实现

#### cast_session_mirror

**类型**: ohos_static_library
**位置**: `service/src/session/src/mirror/BUILD.gn:11-39`

**源文件**:
```
service/src/session/src/mirror/src/mirror_player_impl.cpp
service/src/session/src/mirror/src/mirror_player_impl_stub.cpp
```

**用途**: 镜像播放器实现

#### cast_session_utils

**类型**: ohos_static_library
**位置**: `service/src/session/src/utils/BUILD.gn:16-56`

**源文件**:
```
service/src/session/src/utils/src/cast_timer.cpp
service/src/session/src/utils/src/encrypt_decrypt.cpp
service/src/session/src/utils/src/handler.cpp
service/src/session/src/utils/src/message.cpp
service/src/session/src/utils/src/permission.cpp
service/src/session/src/utils/src/state_machine.cpp
service/src/session/src/utils/src/utils.cpp
```

**用途**: 会话工具（权限、加密、状态机等）

### 4. 配置和资源 Targets

#### cast_engine_sa_profile

**类型**: ohos_sa_profile
**位置**: `sa_profile/BUILD.gn:16-19`

**输出**: 5526.json（安装在 /system/profile/）

**内容**:
```json
{
    "process": "cast_engine_service",
    "systemability": [{
        "name": 5526,
        "libpath": "libcast_engine_service.z.so",
        "run-on-create": false,
        "distributed": false,
        "dump_level": 1
    }]
}
```

#### cast_engine_service.cfg

**类型**: ohos_prebuilt_etc
**位置**: `etc/init/BUILD.gn:14-32`

**输出**: cast_engine_service.cfg（安装在 /etc/init/）

**用途**: 服务启动配置

---

## 依赖关系图

```
libcast.z.so (N-API)
    │ depends on
    ├─> libcast_engine_client.z.so
    │       │
    │       └─> libcast_engine_common_sources.a
    │
    └─> ace_napi.so (外部)
            └─> hilog.so (外部)

libcast_engine_service.z.so (SA)
    │ depends on
    ├─> libcast_engine_client.z.so
    │       │
    │       └─> libcast_engine_common_sources.a
    │
    ├─> cast_discovery
    │       └─> cast_session_utils
    │               └─> cast_engine_common_sources.a
    │
    ├─> cast_session
    │       ├─> cast_session_channel
    │       ├─> cast_session_mirror
    │       ├─> cast_session_rtsp
    │       ├─> cast_session_stream
    │       └─> cast_session_utils
    │                   └─> cast_engine_common_sources.a
    │
    └─> 多个外部依赖
```

---

## 编译产物

### 产物清单

| 产物 | 类型 | 目标 | 安装路径 | 大小（估算） |
|-----|------|------|----------|------------|
| libcast.z.so | 共享库 | cast | /system/lib64/module/ | ~500KB |
| libcast_engine_client.z.so | 共享库 | cast_engine_client | /system/lib64/ | ~200KB |
| libcast_engine_service.z.so | 共享库 | cast_engine_service | /system/lib64/ | ~1MB |
| 5526.json | SA 配置 | cast_engine_sa_profile | /system/profile/ | ~200B |
| cast_engine_service.cfg | 配置 | cast_engine_service.cfg | /etc/init/ | ~500B |

### 运行时加载关系

```
应用启动
    │
    ├── 1. Node.js 加载 libcast.z.so
    │
    ├── 2. libcast.z.so 依赖 libcast_engine_client.z.so
    │
    ├── 3. libcast_engine_client.z.so 通过 Samgr 获取 SA
    │
    └── 4. libcast_engine_service.z.so 被 Samgr 启动
         │
         ▼
    ┌──────────────────────────┐
    │ System Ability 启动    │
    │ (SA 5526)             │
    └──────────────────────────┘
```

### 安装目录映射

| 文件类型 | 安装目录 | 说明 |
|---------|----------|------|
| N-API 模块 | /system/lib64/module/ | N-API 模块专用目录 |
| 客户端/服务库 | /system/lib64/ | 标准库目录 |
| SA 配置 | /system/profile/ | SA 配置目录 |
| 初始化配置 | /etc/init/ | init 配置目录 |

---

## 编译配置

### 默认编译标志

从 `cast_engine_default_config` 继承到所有 targets：

```gn
cflags = [
    "-Wall",              # 所有警告
    "-Wextra",            # 额外警告
    "-Werror",           # 警告=错误
    "-Wno-shadow",        # 忽略变量阴影警告
    "-Wno-unused-parameter",  # 忽略未使用参数警告
    "-Wno-missing-field-initializers",  # 忽略缺少字段初始化器
    "-FS",               # 函数级链接（优化）
    "-O2",               # 优化级别 2
    "-D_FORTIFY_SOURCE=2",  # 源代码强化
    "-fvisibility=hidden",        # 隐藏所有符号
    "-fvisibility-inlines-hidden"
]
```

### 安全特性

所有主要库启用了以下安全特性：

```gn
sanitize = {
    cfi = true              # 控制流完整性
    cfi_cross_dso = true    # 跨 DSO CFI
    debug = false             # 发布版本不启用调试
}
branch_protector_ret = "pac_ret"  # 返回地址保护
```

### 特定模块配置

#### channel 模块

```gn
# service/src/session/src/channel/BUILD.gn:27-31
cflags = [
    "-DPDT_MIRACAST",         # 定义 Miracast 支持
    "-DFILLP_SERVER_SUPPORT",    # 定义服务器支持
    "-DFILLP_LINUX"            # 定义 Linux 支持
]
```

---

## 如何添加新 Target

### 步骤 1: 创建 BUILD.gn

在目标模块目录创建或修改 BUILD.gn 文件：

```gn
import("//foundation/CastEngine/castengine_cast_framework/cast_engine.gni")

# 定义目标
ohos_shared_library("my_new_target") {
  sources = [ "my_source.cpp" ]

  configs = [
    ":my_config",
    "${cast_engine_root}:cast_engine_default_config",
  ]

  deps = [
    "${cast_engine_common}:cast_engine_common_sources",
  ]

  external_deps = [
    "hilog:libhilog",
    "ipc:ipc_core",
  ]

  subsystem_name = "castplus"
  part_name = "cast_engine"
}
```

### 步骤 2: 添加到 bundle.json

如果需要作为独立构建产物，添加到 `bundle.json`:

```json
"build": {
    "sub_component": [
        "//foundation/CastEngine/castengine_cast_framework/mypath:my_new_target"
    ]
}
```

### 步骤 3: 更新依赖

确保依赖关系正确，避免循环依赖。

---

## 构建命令

### 标准构建

```bash
hb build cast
```

### 清理构建

```bash
hb clean
```

### 特定 Target 构建

```bash
hb build //foundation/CastEngine/castengine_cast_framework/interfaces/kits/js:cast
```

---

## 相关文档

- [编译产物](06_Build_Artifacts.md) - 产物安装和运行时加载
- [安全评审](07_Security_Review.md) - 编译器安全标志
- [目录结构](01_Directory_Structure.md) - 模块组织

---

**返回**: [SUMMARY.md](SUMMARY.md)
