# HiCollie GN Targets 构建文档

> GN Targets 列表、类型、依赖、产物映射

---

## 目的与适用范围

### 文档目的
本文档详细说明 HiCollie 的 GN 构建系统，包括所有 Targets 的类型、依赖关系、编译选项和输出产物。

### 适用场景
- 🔨 **构建修改** - 添加或修改构建目标
- 🔧 **配置调整** - 修改编译开关和依赖
- 📦 **集成部署** - 了解产物依赖和安装路径

---

## GN 文件清单

### 根构建入口

**文件**: `/bundle.json` (lines 50-57)

```json
"sub_component": [
    "//base/hiviewdfx/hicollie/interfaces/app:libapp_hicollie",
    "//base/hiviewdfx/hicollie/interfaces/native/innerkits:libhicollie",
    "//base/hiviewdfx/hicollie/frameworks/native/thread_sampler:libthread_sampler",
    "//base/hiviewdfx/hicollie/interfaces/rust:hicollie_rust",
    "//base/hiviewdfx/hicollie/interfaces/ndk:ohhicollie"
]
```

### 全局配置文件

| 文件 | 内容 | 用途 |
|-----|------|------|
| `hicollie.gni` | GN 变量定义 | 路径、编译开关 |
| `bundle.json` | 组件元数据 | 子系统、依赖、构建目标 |
| `hisysevent.yaml` | 系统事件定义 | SERVICE_TIMEOUT、IPC_FULL 等 |

**证据**: `hicollie.gni:14-24` - declare_args 定义

---

## 核心 Targets

### 1. Native 核心模块 (frameworks/native/)

#### 1.1 libhicollie_source

**文件**: `frameworks/native/BUILD.gn:26-83`

| 属性 | 值 |
|-----|-----|
| **Target 类型** | `ohos_source_set` |
| **Target 名称** | `libhicollie_source` |
| **Sources** | `handler_checker.cpp`, `ipc_full.cpp`, `watchdog.cpp`, `watchdog_inner.cpp`, `watchdog_task.cpp`, `xcollie.cpp`, `xcollie_ffrt_task.cpp`, `xcollie_utils.cpp` |
| **Configs** | `:hicollie_include` |
| **Include Dirs** | `.`, `${hicollie_part_path}/interfaces/native/innerkits/include/xcollie` |
| **External Deps** | `c_utils:utils`, `faultloggerd:libbacktrace_local`, `ffrt:libffrt`, `hilog:libhilog`, `hiview:libucollection_client`, `init:libbegetutil`, `ipc:ipc_core`, `storage_service:storage_manager_acl` |
| **Defines** | `HOOK_ENABLE`, `SUSPEND_CHECK_ENABLE`, `HICOLLIE_JANK_ENABLE`, `KICK_WATCHDOG_ENABLE`, `ASYNC_BINDER_SPACE_FULL`, `HISYSEVENT_ENABLE` (条件）|
| **Part/Subsystem** | `hicollie` / `hiviewdfx` |

**条件编译宏**:
```gn
if (use_musl && !is_asan) {
    defines += [ "HOOK_ENABLE" ]
}

if (hicollie_suspend_check_enable) {
    defines += [ "SUSPEND_CHECK_ENABLE" ]
}

if (hicollie_jank_detection_enable) {
    defines += [ "HICOLLIE_JANK_ENABLE" ]
}

if (hicollie_kick_watchdog_enable) {
    defines += [ "KICK_WATCHDOG_ENABLE" ]
}

if (hicollie_asyncbinderspacefull_enable) {
    defines += [ "ASYNC_BINDER_SPACE_FULL" ]
}
```

#### 1.2 libhicollie (Group)

**文件**: `frameworks/native/BUILD.gn:85-87`

| 属性 | 值 |
|-----|-----|
| **Target 类型** | `group` |
| **Target 名称** | `libhicollie` |
| **Deps** | `:libhicollie_source` |

---

### 2. Native InnerKits 接口 (interfaces/native/innerkits/)

#### 2.1 libhicollie

**文件**: `interfaces/native/innerkits/BUILD.gn:21-34`

| 属性 | 值 |
|-----|-----|
| **Target 类型** | `ohos_shared_library` |
| **Target 名称** | `libhicollie` |
| **Public Configs** | `:libhicollie_pub_config` (include_dirs: `include`) |
| **Deps** | `//base/hiviewdfx/hicollie/frameworks/native:libhicollie_source` |
| **External Deps** | `hilog:libhilog` |
| **InnerAPI Tags** | `chipsetsdk_sp`, `platformsdk` |
| **Version Script** | `libhicollie.map` |
| **输出文件** | `libhicollie.so` |

**符号导出控制**: `libhicollie.map` 控制对外暴露的符号

---

### 3. App 模块 (frameworks/app/)

#### 3.1 libapp_hicollie_source

**文件**: `frameworks/app/BUILD.gn:27-64`

| 属性 | 值 |
|-----|-----|
| **Target 类型** | `ohos_source_set` |
| **Target 名称** | `libapp_hicollie_source` |
| **Sources** | `src/app_watchdog.cpp`, `src/app_watchdog_inner.cpp`, `src/app_watchdog_utils.cpp` |
| **Configs** | `:app_hicollie_include` |
| **Include Dirs** | `.`, `include`, `${hicollie_part_path}/interfaces/app/include` |
| **External Deps** | `c_utils:utils`, `faultloggerd:dfx_signalhandler`, `faultloggerd:libbacktrace_local`, `ffrt:libffrt`, `hilog:libhilog`, `hiview:libucollection_client`, `init:libbegetutil`, `ipc:ipc_core`, `storage_service:storage_manager_acl` |
| **Defines** | `HICOLLIE_JANK_ENABLE`, `HISYSEVENT_ENABLE` |

#### 3.2 libapp_hicollie

**文件**: `frameworks/app/BUILD.gn:67-79`

| 属性 | 值 |
|-----|-----|
| **Target 类型** | `ohos_shared_library` |
| **Target 名称** | `libapp_hicollie` |
| **Public Configs** | `:libapp_hicollie_pub_config` (include_dirs: `include`) |
| **Deps** | `:libapp_hicollie_source` |
| **External Deps** | `hilog:libhilog` |
| **InnerAPI Tags** | `platformsdk` |
| **Version Script** | `libapp_hicollie.map` |
| **输出文件** | `libapp_hicollie.so` |

---

### 4. Thread Sampler 模块 (frameworks/native/thread_sampler/)

#### 4.1 libthread_sampler

**文件**: `frameworks/native/thread_sampler/BUILD.gn:23-49`

| 属性 | 值 |
|-----|-----|
| **Target 类型** | `ohos_shared_library` |
| **Target 名称** | `libthread_sampler` |
| **Sources** | `thread_sampler.cpp`, `thread_sampler_api.cpp`, `thread_sampler_utils.cpp` |
| **Public Configs** | `:thread_sampler_config` |
| **External Deps** | `c_utils:utils`, `faultloggerd:libasync_stack`, `faultloggerd:libstack_printer`, `faultloggerd:libunwinder`, `hilog:libhilog` |
| **InnerAPI Tags** | `chipsetsdk_sp_indirect`, `platformsdk_indirect` |
| **Version Script** | `libthread_sampler.map` |
| **输出文件** | `libthread_sampler.z.so` (注：.z 后缀表示压缩库）|

#### 4.2 libthread_sampler_static

**文件**: `frameworks/native/thread_sampler/BUILD.gn:51-71`

| 属性 | 值 |
|-----|-----|
| **Target 类型** | `ohos_static_library` |
| **Target 名称** | `libthread_sampler_static` |
| **Sources** | 与 libthread_sampler 相同 |
| **Public Configs** | `:thread_sampler_config` |
| **External Deps** | 与 libthread_sampler 相同 |
| **输出文件** | `libthread_sampler_static.a` |

---

### 5. NDK 接口模块 (interfaces/ndk/)

#### 5.1 ohhicollie

**文件**: `interfaces/ndk/BUILD.gn:16-49`

| 属性 | 值 |
|-----|-----|
| **Target 类型** | `ohos_shared_library` |
| **Target 名称** | `ohhicollie` |
| **Sources** | `hicollie.cpp` |
| **Include Dirs** | `include`, `../native/innerkits/include/xcollie`, `../../frameworks/native` |
| **Deps** | `../native/innerkits:libhicollie` |
| **External Deps** | `ability_runtime:app_manager`, `c_utils:utils`, `hilog:libhilog`, `ipc:ipc_core`, `samgr:samgr_proxy` |
| **Defines** | `SUPPORT_ASAN` (条件编译）|
| **InnerAPI Tags** | `ndk` |
| **Output Extension** | `so` |
| **Version Script** | `libohhicollie.map` |
| **输出文件** | `libohhicollie.so` |

**ASAN 支持宏**:
```gn
if (is_asan || asan_detector) {
    defines += [ "SUPPORT_ASAN" ]
}
```

---

### 6. Rust 接口模块 (interfaces/rust/)

#### 6.1 hicollie_rust

**文件**: `interfaces/rust/BUILD.gn:16-27`

| 属性 | 值 |
|-----|-----|
| **Target 类型** | `ohos_rust_shared_library` |
| **Target 名称** | `hicollie_rust` |
| **Sources** | `src/lib.rs` |
| **Rustflags** | `-Zstack-protector=all` |
| **Deps** | `../native/innerkits:libhicollie` |
| **Crate Name** | `hicollie_rust` |
| **Crate Type** | `dylib` |
| **输出文件** | `libhicollie_rust.dylib` (或平台对应格式）|

---

## 编译开关

### 全局配置 (hicollie.gni)

| 配置项 | 默认值 | 说明 | 使用的代码路径 |
|--------|---------|------|--------------|
| `hicollie_jank_detection_enable` | `true` | 启用卡顿检测 | `HICOLLIE_JANK_ENABLE` |
| `hicollie_suspend_check_enable` | `false` | 启用暂停检查 | `SUSPEND_CHECK_ENABLE` |
| `hicollie_kick_watchdog_enable` | `false` | 启用喂狗功能 | `KICK_WATCHDOG_ENABLE` |
| `hicollie_asyncbinderspacefull_enable` | `false` | 启用异步 Binder 空间满检测 | `ASYNC_BINDER_SPACE_FULL` |

**证据**: `hicollie.gni:18-23` - declare_args 定义

---

## Targets 依赖关系

### 依赖图

```
ohhicollie (NDK C API)
    │
    │ 静态链接
    ▼
libhicollie (Native C++ API)
    │
    │ 静态链接
    ▼
┌────────────────────────────────────────────┐
│ libhicollie_source (核心源码集合)           │
│ ───────────────────────────────────────── │
│ sources:                                   │
│   ├── handler_checker.cpp                 │
│   ├── ipc_full.cpp                        │
│   ├── watchdog.cpp                         │
│   ├── watchdog_inner.cpp                   │
│   ├── watchdog_task.cpp                    │
│   ├── xcollie.cpp                          │
│   ├── xcollie_ffrt_task.cpp                │
│   └── xcollie_utils.cpp                    │
└────────────────────────────────────────────┘
    │
    │ 外部依赖（动态链接）
    ▼
├── c_utils:utils              ──► 工具函数（字符串、安全函数）
├── hilog:libhilog            ──► 日志输出
├── ipc:ipc_core              ──► IPC 调用（Binder）
├── ffrt:libffrt              ──► FFRT 异步任务框架
├── faultloggerd:libbacktrace_local ──► 堆栈回溯
├── eventhandler:libeventhandler (条件) ──► EventHandler 支持
├── hiview:libucollection_client ──► 事件收集
└── init:libbegetutil         ──► 初始化工具

libapp_hicollie (应用层）
    │
    │ 静态链接
    ▼
libapp_hicollie_source
    │
    │ 外部依赖
    ▼
└── hilog:libhilog            ──► 日志输出
└── 其他依赖（同 libhicollie_source）

libthread_sampler (动态加载）
    │
    │ 外部依赖
    ▼
├── libasync_stack.z.so       ──► 异步栈处理
├── libstack_printer.so       ──► 栈打印
├── libunwinder.so            ──► 栈展开
└── hilog:libhilog            ──► 日志输出
```

### 依赖链文字说明

**层级 1: NDK 接口层 (`ohhicollie`)**
- 直接依赖: `libhicollie` (Native C++ API)
- 间接依赖: 所有 `libhicollie_source` 的外部依赖
- 构建时: 静态链接 `libhicollie.so`

**层级 2: Native 接口层 (`libhicollie`)**
- 直接依赖: `libhicollie_source` (核心源码)
- 间接依赖: DFX 子系统组件
- 特点: 这是主要的 API 暴露层

**层级 3: 核心源码层 (`libhicollie_source`)**
- 直接依赖: 外部组件（hilog, ipc, ffrt, faultloggerd）
- 特点: 包含所有核心实现逻辑

**层级 4: 动态加载层 (`libthread_sampler`)**
- 运行时: 通过 `dlopen()` 动态加载
- 延迟加载: 仅在需要采样时才加载
- 优势: 减少主进程的内存占用和启动时间

**证据**: 各 BUILD.gn 文件中的 deps 和 external_deps

---

## 构建产物

### 产物清单

| Target | 产物类型 | 输出文件名 | 安装路径 |
|--------|---------|------------|---------|
| `libhicollie` | 共享库 | `libhicollie.so` | `/system/lib64/` |
| `libapp_hicollie` | 共享库 | `libapp_hicollie.so` | `/system/lib64/` |
| `libthread_sampler` | 共享库（压缩）| `libthread_sampler.z.so` | `/system/lib64/` |
| `libthread_sampler_static` | 静态库 | `libthread_sampler_static.a` | 仅供内部链接 |
| `ohhicollie` | NDK 共享库 | `libohhicollie.so` | `/system/lib64/ndk/` |
| `hicollie_rust` | Rust 动态库 | `libhicollie_rust.dylib` | `/system/lib64/` |

### 运行时加载关系

```
应用进程
    ↓
libohhicollie.so (NDK C API)
    ↓
libhicollie.so (Native C++ API)
    ↓
├── libhicollie_source (核心代码）
│   ↓
│   ├── libhilog.so (日志）
│   ├── libipc_core.so (IPC）
│   ├── libffrt.so (FFRT）
│   ├── libeventhandler.so (条件）
│   └── libthread_sampler.z.so (动态加载）
└── libthread_sampler.z.so (运行时加载）
    ↓
├── libasync_stack.z.so (异步栈）
├── libstack_printer.so (栈打印）
└── libunwinder.so (栈展开）
```

**动态加载证据**: `watchdog_inner.cpp:366-396` - dlopen 调用

---

## 编译命令示例

### 构建特定目标

```bash
# 构建所有子组件
hb build

# 构建 Native C++ API
hb build --target //base/hiviewdfx/hicollie/interfaces/native/innerkits:libhicollie

# 构建 NDK C API
hb build --target //base/hiviewdfx/hicollie/interfaces/ndk:ohhicollie

# 构建 Thread Sampler
hb build --target //base/hiviewdfx/hicollie/frameworks/native/thread_sampler:libthread_sampler
```

### 启用特性开关

```bash
# 启用卡顿检测
hb build --define hicollie_jank_detection_enable=true

# 启用暂停检查
hb build --define hicollie_suspend_check_enable=true

# 启用异步 Binder 空间满检测
hb build --define hicollie_asyncbinderspacefull_enable=true
```

### 查看 target 依赖

```bash
# 查看完整依赖树
gn desc out/<target> //base/hiviewdfx/hicollie/interfaces/native/innerkits:libhicollie

# 查看输入文件
gn inputs out/<target> //base/hiviewdfx/hicollie/interfaces/native/innerkits:libhicollie
```

---

## 关键结论

### 构建特点
1. **多产物输出** - 支持 C、C++、Rust 多语言接口
2. **条件编译** - 通过 gi 支持特性开关
3. **动态库分离** - Thread Sampler 作为独立模块可动态加载
4. **符号控制** - 使用 version script 控制导出符号

### 依赖特点
1. **模块化** - 内部模块和接口层分离
2. **外部依赖** - 依赖多个 DFX 子系统组件
3. **分层依赖** - NDK → Native → Source → 外部库

### 扩展建议
1. **新增依赖** - 在相应 BUILD.gn 的 external_deps 中添加
2. **新增源文件** - 在 sources 中添加文件路径
3. **新增编译宏** - 在 hicollie.gni 中添加 declare_args
4. **控制符号导出** - 修改相应的 .map 文件

---

## 相关跳转

- [目录结构](01_Directory_Structure.md) - 找 BUILD.gn 位置
- [编译产物](06_Build_Artifacts.md) - 了解产物依赖
- [项目概览](00_Overview.md) - 了解依赖组件
