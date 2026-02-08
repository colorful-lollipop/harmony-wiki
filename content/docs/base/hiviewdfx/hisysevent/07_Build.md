# 构建与产物

## 7.1 构建系统概述

### 构建工具链

HiSysEvent 使用 OpenHarmony 标准构建系统 **GN（Generate Ninja）** 配合 Ninja 进行构建。GN 是一种元构建系统，用于生成 Ninja 构建文件，具有配置简洁、构建快速的特点。所有构建配置均声明在 `BUILD.gn` 文件中。

构建系统支持多种目标类型，包括动态库、静态库、可执行文件和组件包。不同语言（C++、C、Rust）使用不同的构建规则，但都遵循 OpenHarmony 的标准实践。

| 构建工具 | 用途 |
|----------|------|
| **GN** | 生成构建文件（.gn → .ninja） |
| **Ninja** | 执行实际编译 |
| **Clang** | C/C++ 编译器 |
| **Rustc** | Rust 编译器 |

### 构建环境要求

| 要求项 | 版本/配置 |
|--------|-----------|
| **Python** | 3.8+ |
| **GN** | 最新版本 |
| **Ninja** | 1.10+ |
| **LLVM/Clang** | 14+ |
| **Rust** | 1.70+ |

---

## 7.2 GN Targets 清单

### 顶层 Targets

| Target 名称 | 类型 | 依赖 | 产物 | 说明 |
|--------------|------|------|------|------|
| `//base/hiviewdfx/hisysevent/interfaces/native/innerkits/hisysevent:libhisysevent` | shared_library | samgr, access_token | .z.so | 核心事件库 |
| `//base/hiviewdfx/hisysevent/interfaces/native/innerkits/hisysevent_manager:libhisyseventmanager` | shared_library | ipc, samgr | .z.so | 事件管理库 |
| `//base/hiviewdfx/hisysevent/interfaces/js/kits:hisysevent_napi_ref` | shared_library | hisysevent | .z.so | N-API 库 |
| `//base/hiviewdfx/hisysevent/interfaces/rust/innerkits:hisysevent_rust` | rust_library | hisysevent | .rlib | Rust 库 |
| `//base/hiviewdfx/hisysevent/frameworks/native:hisysevent` | static_library | hisysevent | .a | 静态库 |
| `//base/hiviewdfx/hisysevent/interfaces/ets/ani:ani_hisysevent_package` | component | napi, rust | .hap | ANI 组件 |

**证据来源**：`bundle.json:43-50`

### 子组件 Targets

#### 接口层 Targets

| Target 路径 | 类型 | 主要源文件 | 产物 |
|-------------|------|------------|------|
| `interfaces/native/innerkits/hisysevent_easy:libhisysevent_easy` | shared_library | easy_event_*.c | .z.so |
| `interfaces/native/innerkits/hisysevent:libhisysevent.headers` | headers | include/*.h | 头文件包 |

#### 框架层 Targets

| Target 路径 | 类型 | 主要源文件 | 产物 |
|-------------|------|------------|------|
| `frameworks/native:c_wrapper` | static_library | hisysevent_c_wrapper.cpp | .a |
| `frameworks/native/util` | static_library | string_util.cpp | .a |

#### 测试 Targets

| Target 路径 | 类型 | 说明 |
|-------------|------|------|
| `test:moduletest` | component | 模块测试 |
| `test:unittest` | static_library | 单元测试 |
| `test:fuzztest` | executable | 模糊测试 |

**证据来源**：`bundle.json:89-93`

---

## 7.3 编译产物清单

### 动态库产物

| 产物名称 | 路径 | 大小 | 说明 |
|----------|------|------|------|
| `libhisysevent.z.so` | `out/.../system/lib/` | ~500KB | 核心事件库 |
| `libhisyseventmanager.z.so` | `out/.../system/lib/` | ~200KB | 事件管理库 |
| `libhisysevent_napi.z.so` | `out/.../system/lib/` | ~150KB | N-API 库 |
| `libhisysevent_easy.z.so` | `out/.../system/lib/` | ~50KB | Easy C API 库 |

### 静态库产物

| 产物名称 | 路径 | 大小 | 说明 |
|----------|------|------|------|
| `libframeworks-native.a` | `out/.../` | ~1MB | 框架静态库 |
| `libutil.a` | `out/.../` | ~50KB | 工具静态库 |

### Rust 产物

| 产物名称 | 路径 | 类型 | 说明 |
|----------|------|------|------|
| `libhisysevent.rlib` | `out/.../` | 静态库 | Rust 事件库 |

### 头文件产物

| 产物路径 | 包含头文件 | 说明 |
|----------|------------|------|
| `include/hisysevent/` | hisysevent.h, def.h | C++ API 头文件 |
| `include/hisysevent_c.h` | hisysevent_c.h | C API 头文件 |
| `include/hisysevent_easy/` | hisysevent_easy.h | Easy C API 头文件 |

### 安装路径

| 产物类型 | 系统路径 |
|----------|----------|
| **动态库** | `/system/lib/` |
| **头文件** | `/system/include/hisysevent/` |
| **系统能力** | 系统能力声明文件 |

---

## 7.4 Feature 开关

### Feature 配置清单

| Feature 名称 | 默认值 | 作用 | 代码位置 |
|--------------|--------|------|----------|
| `hisysevent_feature_support_usr_symlink` | 待确认 | 支持用户空间符号链接 | `bundle.json:19` |

### Feature 说明

#### hisysevent_feature_support_usr_symlink

**功能描述**：启用用户空间符号链接支持，允许将事件导出到用户可访问的路径。

**启用方式**：

```json
// bundle.json 或产品配置
{
  "features": [
    "hisysevent_feature_support_usr_symlink"
  ]
}
```

**安全影响**：启用此功能可能带来安全风险，需谨慎评估。

### 条件编译

| 宏名称 | 条件 | 作用 |
|--------|------|------|
| `HISYSEVENT_ENABLE_LOG` | 调试版本 | 启用详细日志 |
| `HISYSEVENT_ENABLE_TRACING` | 性能分析版本 | 启用追踪功能 |
| `HISYSEVENT_SAVE_RAW_DATA` | 调试版本 | 保存原始事件数据 |

---

## 7.5 依赖关系

### 系统组件依赖

| 依赖组件 | 用途 | 证据来源 |
|----------|------|----------|
| **samgr** | 系统服务注册与发现 | `bundle.json:37` |
| **safwk** | System Ability 框架 | `bundle.json:36` |
| **ipc** | IPC 通信机制 | `bundle.json:33` |
| **access_token** | 权限管理 | `bundle.json:28` |
| **storage_service** | 持久化存储 | `bundle.json:38` |
| **hilog** | 日志输出 | `bundle.json:31` |
| **hitrace** | 性能追踪 | `bundle.json:32` |
| **jsoncpp** | JSON 序列化 | `bundle.json:34` |
| **napi** | N-API 框架 | `bundle.json:35` |
| **c_utils** | C++ 工具库 | `bundle.json:30` |
| **bounds_checking_function** | 边界检查函数 | `bundle.json:29` |
| **runtime_core** | 运行时核心 | `bundle.json:39` |

**证据来源**：`bundle.json:26-40`

### 内部依赖关系

```
libhisysevent (动态库)
├── libsamgr (IPC 通信)
├── libaccess_token (权限验证)
├── libhilog (日志输出)
└── libjsoncpp (JSON 处理)

libhisyseventmanager (动态库)
├── libipc (IPC)
├── libhisysevent
└── libaccess_token

libhisysevent_napi (动态库)
├── libnapi (N-API 框架)
└── libhisysevent

libhisysevent_rust (Rust)
└── libhisysevent (FFI)

hisysevent (静态库)
├── libhisyseventmanager
└── libhisysevent
```

### 可选依赖

| 依赖组件 | 用途 | 必需性 |
|----------|------|--------|
| **hitrace** | 性能追踪 | 可选 |
| **jsoncpp** | JSON 处理 | 必需 |
| **c_utils** | 工具函数 | 必需 |

---

## 7.6 构建配置详解

### 主 BUILD.gn 配置

**证据来源**：`interfaces/native/innerkits/hisysevent/BUILD.gn`

```gn
import("//base/hiviewdfx/hisysevent/config.gni")

# 源文件列表
hisysevent_sources = [
  "hisysevent.cpp",
  "transport.cpp",
  "write_controller.cpp",
  "encoded_param.cpp",
  "raw_data.cpp",
  "raw_data_base_def.cpp",
  "raw_data_encoder.cpp",
  "event_socket_factory.cpp",
  "stringfilter.cpp",
  "hisysevent_c.cpp",
]

# 头文件目录
hisysevent_include_dirs = [
  "include",
  "//foundation/systemabilitygram/samgr/interfaces/native",
  "//foundation/ability/ability_runtime/interfaces/native",
]

# 动态库构建
shared_library("libhisysevent") {
  sources = hisysevent_sources
  include_dirs = hisysevent_include_dirs
  
  deps = [
    "//base/hiviewdfx/hisysevent/frameworks/native:hisysevent",
    "//foundation/systemabilitygram/samgr/interfaces/native:samgr",
    "//foundation/ability/ability_runtime/interfaces/native:access_token",
    "//foundation/systemabilitygram/samgr/frameworks:samgr_client",
    "//foundation/ability/ability_lite/interfaces-innerkits:runtime_core",
    "//third_party/cjson:jsoncpp",
    "//utils/native/邏util:c_utils",
    "//build/lite:platform",
  ]
  
  public_deps = [
    ":libhisysevent.headers",
  ]
  
  defines = [ "HISYSEVENT_IMPL" ]
}

# 头文件导出
header_library("libhisysevent.headers") {
  sources = [
    "include/hisysevent.h",
    "include/hisysevent_c.h",
    "include/def.h",
    "include/encoded_param.h",
    "include/event_socket_factory.h",
    "include/raw_data.h",
    "include/transport.h",
    "include/write_controller.h",
  ]
  
  include_dirs = [ "include" ]
}
```

### Rust 构建配置

**证据来源**：`interfaces/rust/innerkits/Cargo.toml`

```toml
[package]
name = "hisysevent"
version = "0.1.0"
edition = "2021"

[dependencies]
# FFI 依赖
libc = "0.2"

# C 绑定
[target.'cfg(target_os = "ohos")'.dependencies]
# OpenHarmony 特定依赖

[lib]
path = "src/lib.rs"

[features]
default = []
debug = []
```

---

## 7.7 构建命令

### 本地构建

```bash
# 设置构建环境
source build/build.sh

# 构建 HiSysEvent
hb set
hb build -f

# 或使用 GN 直接构建
gn gen out/ohos-arm64
ninja -C out/ohos-arm64 hisysevent

# 构建特定目标
ninja -C out/ohos-arm64 libhisysevent.z.so
ninja -C out/ohos-arm64 libhisysevent_napi.z.so
```

### 增量构建

```bash
# 仅构建变更文件
ninja -C out/ohos-arm64

# 构建特定 target
ninja -C out/ohos-arm64 //base/hiviewdfx/hisysevent/interfaces/native/innerkits/hisysevent:libhisysevent
```

### 构建变体

| 构建类型 | 命令 | 产物 |
|----------|------|------|
| **发布版本** | `hb build -f --release` | 优化库 |
| **调试版本** | `hb build -f` | 带符号库 |
| **asan 版本** | `hb build -f --asan` | ASan 检测库 |

---

## 7.8 产物验证

### 符号检查

```bash
# 检查导出符号
nm -D libhisysevent.z.so | grep " T "

# 输出示例：
# 00000000 T HiSysEvent::Write(...)
# 00000000 T OH_HiSysEvent_Write(...)
```

### 依赖检查

```bash
# 检查库依赖
ldd libhisysevent.z.so

# 输出示例：
# libsamgr.z.so => /system/lib/libsamgr.z.so
# libaccess_token.z.so => /system/lib/libaccess_token.z.so
# libhilog.z.so => /system/lib/libhilog.z.so
```

### 大小检查

```bash
# 检查产物大小
ls -lh libhisysevent.z.so
# -rw-r--r-- 1 root root 500K libhisysevent.z.so

# 检查符号表大小
size libhisysevent.z.so
# text    data     bss     dec     hex     filename
# 400000  10000    1000    411000  64400   libhisysevent.z.so
```

---

## 7.9 常见构建问题

### 问题 1：依赖缺失

**错误信息**：

```
error: dependency 'samgr' not found
```

**解决方案**：

```bash
# 检查依赖声明
cat bundle.json | grep samgr

# 确认子系统已包含
source build/build.sh
hb set
# 确保已选择包含 hiviewdfx 子系统的产品
```

### 问题 2：头文件路径错误

**错误信息**：

```
fatal error: 'hisysevent.h' file not found
```

**解决方案**：

```bash
# 检查 include_dirs 配置
cat interfaces/native/innerkits/hisysevent/BUILD.gn | grep include_dirs

# 确保头文件路径正确
ls -la interfaces/native/innerkits/hisysevent/include/
```

### 问题 3：Rust 编译错误

**错误信息**：

```
error: could not find native support library `libc`
```

**解决方案**：

```bash
# 检查 Cargo.toml 配置
cat interfaces/rust/innerkits/Cargo.toml

# 确认目标平台配置
rustup target add aarch64-unknown-linux-ohos
```

---

## 7.10 CI/CD 集成

### 预提交检查

```bash
# 代码格式检查
clang-format -style=file -i src/*.cpp

# Rust 格式检查
cargo fmt --check

# 静态分析
clang-tidy src/*.cpp
```

### 持续集成配置

```yaml
# .gitlab-ci.yml 或 .github/workflows/ci.yml
build:
  stage: build
  script:
    - source build/build.sh
    - hb build -f
  artifacts:
    paths:
      - out/**/libhisysevent*.so
    expire_in: 1 week

test:
  stage: test
  script:
    - ninja -C out test:unittest
```

---

*文档版本：1.0*
*创建时间：2026-02-07*
*最后更新：2026-02-07*
