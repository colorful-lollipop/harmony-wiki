# GN Targets 与编译产物

## 目的
本文档说明 graphic_surface 的 GN 构建系统，包括 Targets 列表、类型、依赖关系和输出产物。

## 适用范围
面向需要：
- 理解构建系统的开发者
- 修改构建脚本的开发者
- 分析依赖关系的架构师
- 集成 graphic_surface 的产品集成者

## GN 构建系统概览

### 构建系统
graphic_surface 使用 Google Ninja (GN) 构建系统。

### 全局配置
**配置文件**：`graphic_surface_config.gni`

**关键配置**：
```gni
graphic_surface_root = "//foundation/graphic/graphic_surface"

# Feature Flags
graphic_2d_ext_delegator = false
graphic_surface_feature_tv_metadata_enable = false

# 跨平台定义
rosen_cross_platform = (mac, mingw, linux, android, ios)
if (rosen_cross_platform) {
  rs_common_define += [ "ROSEN_TRACE_DISABLE" ]
}
```

### 组件元数据
**配置文件**：`bundle.json`

**关键信息**：
```json
{
  "name": "@ohos/graphic_surface",
  "version": "4.1",
  "subsystem": "graphic",
  "adapted_system_type": [ "standard" ],
  "rom": "10000KB",
  "ram": "10000KB",
  "deps": {
    "components": [
      "ipc", "hilog", "hitrace", "samgr", ...
    ]
  }
}
```

## 核心模块 Targets

### 模块 1：Surface（核心库）

#### Target 信息

| 属性 | 值 |
|------|-----|
| Target 路径 | `//foundation/graphic/graphic_surface/surface:surface` |
| Target 类型 | `ohos_shared_library` |
| 输出文件 | `libsurface.so` |
| 符号链接 | `libnative_buffer.so`, `libnative_window.so` |

#### Source 文件（18 个）
```
src/buffer_client_producer.cpp
src/buffer_extra_data_impl.cpp
src/buffer_queue.cpp
src/buffer_queue_consumer.cpp
src/buffer_queue_producer.cpp
src/buffer_utils.cpp
src/consumer_surface.cpp
src/consumer_surface_delegator.cpp
src/delegator_adapter.cpp
src/metadata_helper.cpp
src/native_buffer.cpp
src/native_window.cpp
src/producer_surface.cpp
src/producer_surface_delegator.cpp
src/surface_buffer_impl.cpp
src/surface_delegate.cpp
src/surface_tunnel_handle.cpp
src/surface_utils.cpp
```

#### 依赖关系
```gn
# Internal deps
deps = [
  ":surface_static",
  "//foundation/graphic/graphic_surface/buffer_handle:buffer_handle",
  "//foundation/graphic/graphic_surface/sync_fence:sync_fence",
  "//foundation/graphic/graphic_surface/sandbox:sandbox_utils",
  "//foundation/graphic/graphic_surface/utils/frame_report:frame_report",
  "//foundation/graphic/graphic_surface/utils/hebc_white_list:hebc_white_list",
  "//foundation/graphic/graphic_surface/utils/rs_frame_report_ext:rs_frame_report_ext_surface",
]

# External deps
external_deps = [
  "c_utils:utils",
  "hilog:libhilog",
  "hitrace:hitrace_meter",
  "drivers_interface_display:v1.4",
  "drivers_interface_display:v1.0",
  "drivers_interface_display:v2.0",
  "drivers_interface_display:v2.1",
  "drivers_interface_display:v2.2",
  "eventhandler:libeventhandler",
  "init:libbegetutil",
  "ipc:ipc_single",
  "ipc:ipc_capi",
]
```

#### 包含目录
```
$graphic_surface_root/utils/frame_report/export
$graphic_surface_root/surface/include
$graphic_surface_root/interfaces/inner_api
$graphic_surface_root/interfaces/inner_api/surface
$graphic_surface_root/interfaces/inner_api/common
$graphic_surface_root/interfaces/inner_api/utils
$graphic_surface_root/sandbox
$graphic_surface_root/utils/rs_frame_report_ext/include
$graphic_surface_root/utils/trace
```

#### 定义与配置
```gn
defines = [
  "SURFACE_LOG_TAG=\"Graphic\"",
  "SURFACE_ENABLE_FRAME_STATS",
]

if (graphic_surface_feature_tv_metadata_enable) {
  defines += [ "RS_ENABLE_TV_PQ_METADATA" ]
}

configs = [
  ":surface_config",
]

public_configs = [
  ":surface_public_config",
]
```

#### 子 Target
| Target | 类型 | 输出 |
|--------|------|------|
| `surface_static` | `ohos_static_library` | `libsurface_static.a` |
| `surface_headers` | `ohos_static_library` | 头文件 |

---

### 模块 2：BufferHandle（句柄管理）

#### Target 信息

| 属性 | 值 |
|------|-----|
| Target 路径 | `//foundation/graphic/graphic_surface/buffer_handle:buffer_handle` |
| Target 类型 | `ohos_shared_library` |
| 输出文件 | `libbuffer_handle.so` |

#### Source 文件（1 个）
```
src/buffer_handle.cpp
```

#### 依赖关系
```gn
external_deps = [
  "c_utils:utils",
  "hilog:libhilog",
  "ipc_single",
]
```

#### 包含目录
```
$graphic_surface_root/interfaces/inner_api/utils
```

#### 子 Target
| Target | 类型 | 输出 |
|--------|------|------|
| `buffer_handle_static` | `ohos_static_library` | `libbuffer_handle_static.a` |

---

### 模块 3：SyncFence（同步栅栏）

#### Target 信息

| 属性 | 值 |
|------|-----|
| Target 路径 | `//foundation/graphic/graphic_surface/sync_fence:sync_fence` |
| Target 类型 | `ohos_shared_library` |
| 输出文件 | `libsync_fence.so` |
| 符号链接 | `libnative_fence.so` |

#### Source 文件（5 个）
```
src/acquire_fence_manager.cpp
src/frame_sched.cpp
src/native_fence.cpp
src/sync_fence.cpp
src/sync_fence_tracker.cpp
```

#### 依赖关系
```gn
external_deps = [
  "c_utils:utils",
  "eventhandler:libeventhandler",
  "hilog:libhilog",
  "hisysevent:libhisysevent",
  "hitrace:hitrace_meter",
  "init:libbegetutil",
  "ipc_single",
]
```

#### 定义与配置
```gn
# Fence scheduling（仅非模拟器、非 SDK 构建）
if (!is_emulator && !build_ohos_sdk && current_os == "ohos") {
  defines = [ "FENCE_SCHED_ENABLE" ]
}
```

#### 子 Target
| Target | 类型 | 输出 |
|--------|------|------|
| `sync_fence_static` | `ohos_static_library` | `libsync_fence_static.a` |

---

### 模块 4：FrameReport（帧上报）

#### Target 信息

| 属性 | 值 |
|------|-----|
| Target 路径 | `//foundation/graphic/graphic_surface/utils/frame_report:frame_report` |
| Target 类型 | `ohos_static_library` |
| 输出文件 | `libframe_report.a` |

#### Source 文件（1 个）
```
src/frame_report.cpp
```

#### 依赖关系
```gn
external_deps = [
  "bounds_checking_function:libsec_shared",
  "hilog:libhilog",
  "init:libbegetutil",
  "init:libbeget_proxy",
]
```

#### 包含目录
```
export/
```

#### 定义与配置
```gn
# AI scheduling（依赖 HDF HwSched）
if (defined(global_parts_info.hdf_drivers_interface_hwsched)) {
  defines = [ "AI_SCHED_ENABLE" ]
}
```

---

### 模块 5：HEBCWhiteList（HEBC 白名单）

#### Target 信息

| 属性 | 值 |
|------|-----|
| Target 路径 | `//foundation/graphic/graphic_surface/utils/hebc_white_list:hebc_white_list` |
| Target 类型 | `ohos_static_library` |
| 输出文件 | `libhebc_white_list.a` |

#### Source 文件（1 个）
```
hebc_white_list.cpp
```

#### 依赖关系
```gn
external_deps = [
  "c_utils:utils",
  "config_policy:config_policy",
  "hilog:libhilog",
  "cJSON:json",
]
```

#### 包含目录
```
.
$graphic_surface_root/surface/include
```

---

### 模块 6：Sandbox（沙箱工具）

#### Target 信息

| 属性 | 值 |
|------|-----|
| Target 路径 | `//foundation/graphic/graphic_surface/sandbox:sandbox_utils` |
| Target 类型 | `ohos_static_library` |
| 输出文件 | `libsandbox_utils.a` |

#### Source 文件（1 个）
```
sandbox_utils.cpp
```

#### 包含目录
```
.
```

---

### 模块 7：RSFrameReportExt（RS 帧上报扩展）

#### Target 信息

| 属性 | 值 |
|------|-----|
| Target 路径 | `//foundation/graphic/graphic_surface/utils/rs_frame_report_ext:rs_frame_report_ext_surface` |
| Target 类型 | `ohos_source_set` |
| 输出文件 | 目标文件（仅编译，无独立输出） |

#### Source 文件（1 个）
```
src/rs_frame_report_ext.cpp
```

#### 依赖关系
```gn
external_deps = [
  "hilog:libhilog",
]
```

#### 包含目录
```
include/
```

---

### 模块 8：TestHeader（测试支持）

#### Target 信息

| 属性 | 值 |
|------|-----|
| Target 路径 | `//foundation/graphic/graphic_surface/test_header:test_header` |
| Target 类型 | `ohos_static_library` |
| 输出文件 | 头文件 |

#### Source 文件
```
export/test_header.h
```

#### 依赖关系
```gn
external_deps = [
  "hilog:libhilog",
]
```

## Target 依赖图

### 依赖关系图
```
surface (so)
  ├─ surface_static (a)
  ├─ buffer_handle (so)
  │   └─ buffer_handle_static (a)
  ├─ sync_fence (so)
  │   └─ sync_fence_static (a)
  ├─ sandbox_utils (a)
  ├─ frame_report (a)
  ├─ hebc_white_list (a)
  └─ rs_frame_report_ext_surface (source_set)

External deps (c_utils, hilog, ipc, eventhandler, hitrace, etc.)
```

### Target 层级

```
应用层（外部）
  │
  ├─ surface (so) ◄─── 主要对外产物
  ├─ sync_fence (so)
  └─ buffer_handle (so)

模块内部（静态库）
  │
  ├─ surface_static (a)
  ├─ sync_fence_static (a)
  ├─ buffer_handle_static (a)
  ├─ sandbox_utils (a)
  ├─ frame_report (a)
  ├─ hebc_white_list (a)
  └─ rs_frame_report_ext (source_set)

基础依赖（外部）
  │
  ├─ c_utils (a)
  ├─ hilog (so)
  ├─ ipc (so)
  ├─ eventhandler (so)
  ├─ hitrace (so)
  └─ ... 其他
```

## Feature Flags

### 启用的 Feature Flags

| Flag | 默认值 | 条件 | 效果 |
|------|---------|--------|------|
| `ROSEN_TRACE_DISABLE` | 条件编译 | `rosen_cross_platform` 为真 | 禁用追踪 |
| `RS_ENABLE_TV_PQ_METADATA` | `false` | `graphic_surface_feature_tv_metadata_enable` = true | 启用 TV PQ 元数据 |
| `AI_SCHED_ENABLE` | 条件编译 | HDF HwSched 存在 | 启用 AI 调度 |
| `FENCE_SCHED_ENABLE` | 条件编译 | 非模拟器、非 SDK 构建、OS = "ohos" | 启用 Fence 调度 |

### 扩展委托模式
| Flag | 默认值 | 用途 |
|------|---------|------|
| `graphic_2d_ext_delegator` | `false` | 启用 2D 扩展委托 |
| `graphic_2d_ext_delegator_gni` | `""` | 委托模式 GNI 配置路径 |

## 公共外部依赖

### 核心依赖
| 组件 | 用途 | 使用位置 |
|--------|------|---------|
| `c_utils:utils` | C 工具库 | 所有模块 |
| `hilog:libhilog` | 日志记录 | 所有模块 |
| `ipc:ipc_single` | IPC 单线程 | buffer_handle, sync_fence |
| `ipc:ipc_capi` | IPC C API | surface |
| `eventhandler:libeventhandler` | 事件处理 | sync_fence, surface |
| `hitrace:hitrace_meter` | 性能追踪 | sync_fence, surface |

### 显示相关依赖
| 组件 | 版本 | 用途 |
|--------|------|------|
| `drivers_interface_display:v1.0` | v1.0 | 显示驱动接口（兼容） |
| `drivers_interface_display:v1.1` | v1.1 | 显示驱动接口 |
| `drivers_interface_display:v2.0` | v2.0 | 显示驱动接口 |
| `drivers_interface_display:v2.1` | v2.1 | 显示驱动接口 |
| `drivers_interface_display:v2.2` | v2.2 | 显示驱动接口 |
| `drivers_interface_display:v1.4` | v1.4 | 显示驱动接口 |

### 系统服务依赖
| 组件 | 用途 |
|--------|------|
| `samgr` | 系统能力管理器 |
| `init:libbegetutil` | 初始化工具 |
| `init:libbeget_proxy` | 初始化代理 |
| `hisysevent:libhisysevent` | 系统事件 |

### 安全相关依赖
| 组件 | 用途 |
|--------|------|
| `access_token` | 访问令牌（声明但未直接使用） |
| `selinux_adapter` | SELinux 策略 |
| `bounds_checking_function` | 边界检查（libsec_shared） |

### 其他依赖
| 组件 | 用途 |
|--------|------|
| `config_policy:config_policy` | 配置策略 |
| `cJSON:json` | JSON 解析 |
| `hicollie` | 故障诊断 |

## 构建配置

### Sanitizers
所有共享库和 hebc_white_list 启用以下 sanitizers：

```gn
# surface/BUILD.gn, buffer_handle/BUILD.gn, hebc_white_list/BUILD.gn
boundary_sanitize = true
integer_overflow = true
ubsan = true
```

**用途**：
- `boundary_sanitize` - 数组边界检查
- `integer_overflow` - 整数溢出检查
- `ubsan` - 未定义行为检查

### Inner API Tags
部分 Target 声明 `inner_api_tags`：

| Target | Tags |
|--------|-------|
| `surface` | `chipsetsdk_sp`, `platformsdk` |
| `buffer_handle` | `chipsetsdk_sp`, `platformsdk` |
| `sync_fence` | `platformsdk_indirect`, `ndk`, `llndk` |

**用途**：
- `chipsetsdk_sp` - 芯片套件 SDK
- `platformsdk` - 平台 SDK
- `ndk` - Native 开发者 Kit
- `llndk` - 低级 Native 开发者 Kit

## 编译命令

### 标准编译
```bash
# 编译整个 graphic_surface
hb build graphic_surface

# 编译特定模块
hb build //foundation/graphic/graphic_surface/surface:surface
hb build //foundation/graphic/graphic_surface/sync_fence:sync_fence
hb build //foundation/graphic/graphic_surface/buffer_handle:buffer_handle
```

### 带 Feature Flags 编译
```bash
# 启用 TV PQ metadata
hb build graphic_surface --build-option graphic_surface_feature_tv_metadata_enable=true

# 启用 2D 扩展委托
hb build graphic_surface --build-option graphic_2d_ext_delegator=true
```

### 调试编译
```bash
# 带 Sanitizers 编译
hb build graphic_surface --gn-args='use_asan=true'

# 带符号表编译
hb build graphic_surface --ccache
```

### 测试编译
```bash
# 编译所有测试
hb build graphic_surface --test

# 编译单元测试
hb build //foundation/graphic/graphic_surface/surface/test:unittest

# 编译 Fuzz 测试
hb build //foundation/graphic/graphic_surface/surface/test:fuzztest
```

## 产物映射

### Target 到产物映射

| Target | 主产物 | 符号链接 | 类型 |
|--------|---------|---------|------|
| `surface` | `libsurface.so` | `libnative_buffer.so`, `libnative_window.so` | 共享库 |
| `surface_static` | `libsurface_static.a` | - | 静态库 |
| `sync_fence` | `libsync_fence.so` | `libnative_fence.so` | 共享库 |
| `sync_fence_static` | `libsync_fence_static.a` | - | 静态库 |
| `buffer_handle` | `libbuffer_handle.so` | - | 共享库 |
| `buffer_handle_static` | `libbuffer_handle_static.a` | - | 静态库 |
| `frame_report` | `libframe_report.a` | - | 静态库 |
| `hebc_white_list` | `libhebc_white_list.a` | - | 静态库 |
| `sandbox_utils` | `libsandbox_utils.a` | - | 静态库 |

### 典型安装路径

OpenHarmony 标准系统的典型库安装路径：

```
/usr/lib/                    # 系统库
  ├── libsurface.so
  ├── libnative_buffer.so ◄───> libsurface.so
  ├── libnative_window.so ◄───> libsurface.so
  ├── libsync_fence.so
  ├── libnative_fence.so ◄───> libsync_fence.so
  └── libbuffer_handle.so

/usr/lib64/                  # 64 位系统库
  └── (同上）
```

### 符号链接作用

| 符号链接 | 目标 | 用途 |
|---------|------|------|
| `libnative_buffer.so` | `libsurface.so` | NDK 兼容性（Native Buffer C API） |
| `libnative_window.so` | `libsurface.so` | NDK 兼容性（Native Window C API） |
| `libnative_fence.so` | `libsync_fence.so` | NDK 兼容性（Native Fence C API） |

## 构建流程

### 完整构建流程
```
1. GN 配置解析
   ├─ graphic_surface_config.gni（全局配置）
   ├─ bundle.json（组件元数据）
   └─ 各模块 BUILD.gn

2. 依赖解析
   ├─ 内部依赖（deps, public_deps）
   ├─ 外部依赖（external_deps）
   └─ 传递依赖（自动解析）

3. Source 编译
   ├─ C++ 编译（clang）
   ├─ 依赖头文件生成
   └─ 目标文件生成

4. 链接
   ├─ 静态库链接（.a）
   ├─ 共享库链接（.so）
   └─ 符号表生成

5. 后处理
   ├─ 符号链接创建
   ├─ Strip 符号（release）
   └─ 安装到目标目录
```

### 关键构建步骤

#### Surface 模块构建
```
surface/BUILD.gn
  ├─ 编译 src/*.cpp（18 个文件）
  ├─ 链接 surface_static
  │   ├─ buffer_handle_static
  │   ├─ sync_fence_static
  │   ├─ sandbox_utils
  │   ├─ frame_report
  │   ├─ hebc_white_list
  │   └─ rs_frame_report_ext_surface
  ├─ 链接 surface (so)
  │   ├─ surface_static
  │   ├─ 外部依赖（c_utils, hilog, ipc, ...）
  │   └─ 链接 libsurface.so
  └─ 创建符号链接
      ├─ libnative_buffer.so -> libsurface.so
      └─ libnative_window.so -> libsurface.so
```

## 依赖管理

### 添加新依赖

**示例**：添加新库 `mylib`

修改 `surface/BUILD.gn`：
```gn
ohos_shared_library("surface") {
  # ... 现有配置 ...

  # 添加外部依赖
  external_deps = [
    # ... 现有依赖 ...
    "mylib:mylib",  # 新增依赖
  ]
}
```

修改 `bundle.json`：
```json
{
  "component": {
    "deps": {
      "components": [
        # ... 现有组件 ...
        "mylib",  # 新增组件依赖
      ]
    }
  }
}
```

### 依赖版本管理

**显示驱动多版本支持**：
```gn
external_deps = [
  "drivers_interface_display:v1.4",  # 支持最新
  "drivers_interface_display:v1.0",  # 向后兼容
  "drivers_interface_display:v2.0",
  # ...
]
```

## 常见构建问题

### 问题 1：找不到头文件

**症状**：
```
error: surface.h: No such file or directory
```

**解决方案**：
```gn
# 检查 include_dirs 配置
include_dirs = [
  "$graphic_surface_root/interfaces/inner_api/surface",
  # ...
]
```

### 问题 2：链接错误

**症状**：
```
undefined reference to 'BufferQueue::RequestBuffer'
```

**解决方案**：
```gn
# 检查 deps 是否正确
deps = [
  ":surface_static",  # 必须依赖静态库
]
```

### 问题 3：Feature Flag 不生效

**症状**：
宏未定义，相关代码被跳过

**解决方案**：
```gn
# 检查 defines 是否正确设置
if (graphic_surface_feature_tv_metadata_enable) {
  defines += [ "RS_ENABLE_TV_PQ_METADATA" ]
}

# 编译时传递参数
hb build graphic_surface --build-option graphic_surface_feature_tv_metadata_enable=true
```

## 相关跳转
- [目录结构与模块职责](01_Directory_Structure.md) - BUILD.gn 文件位置
- [编译产物](06_Build_Artifacts.md) - 构建输出清单
- [附录 - 配置宏与 Feature Flags](appendix/Config_Flags.md) - 编译开关详细说明
