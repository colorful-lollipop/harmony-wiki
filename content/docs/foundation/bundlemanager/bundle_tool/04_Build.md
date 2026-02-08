# 构建系统 (Build)

> bundle_tool GN 构建配置、Targets 与编译产物

## 构建系统概述

### 构建工具

| 属性 | 值 |
|------|------|
| 构建系统 | GN (Generate Ninja) |
| 构建模板 | ohos.gni |
| 构建路径 | //foundation/bundlemanager/bundle_tool |

### 配置文件

| 文件 | 用途 |
|------|------|
| `BUILD.gn` (根) | 根目录构建入口 |
| `frameworks/BUILD.gn` | 框架构建配置 |
| `bundletool.gni` | GN 参数配置 |
| `bundle.json` | 组件配置 |

---

## BUILD.gn (根目录)

### 文件位置

`/Volumes/lexar/code/d/work/oh/foundation/bundlemanager/bundle_tool/BUILD.gn`

### 内容

```gn
# Copyright (c) 2022 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");

import("//build/ohos.gni")

group("bm") {
  deps = [ "frameworks:tools_bm" ]
}
```

### 目标

| target | 类型 | 输出 | 依赖 |
|--------|------|------|------|
| `bm` | group | - | `frameworks:tools_bm` |

**证据来源**: `BUILD.gn:16-18`

---

## bundletool.gni (参数配置)

### 文件位置

`/Volumes/lexar/code/d/work/oh/foundation/bundlemanager/bundle_tool/bundletool.gni`

### 内容

```gn
# Copyright (c) 2022-2023 Huawei Device Co., Ltd.

bundlemanager_path = "//foundation/bundlemanager"
bundle_framework_path = "${bundlemanager_path}/bundle_framework"

common_path = "${bundle_framework_path}/common"
kits_path = "${bundle_framework_path}/interfaces/kits"
inner_api_path = "${bundle_framework_path}/interfaces/inner_api"

bundletool_path = "${bundlemanager_path}/bundle_tool/frameworks"
bundletool_test_path = "${bundlemanager_path}/bundle_tool/test"

declare_args() {
  account_enable_bm = true
  overlay_install_bm = true
  quick_fix_bm = true
  distributed_bundle_framework_bm = true

  # 条件禁用逻辑...
}

bm_install_external_deps = [ "ffrt:libffrt" ]
```

### Feature Flags

| 开关 | 默认值 | 说明 |
|------|--------|------|
| `account_enable_bm` | true | 用户账号功能 |
| `overlay_install_bm` | true | Overlay 安装功能 |
| `quick_fix_bm` | true | 快速修复功能 |
| `distributed_bundle_framework_bm` | true | 分布式 Bundle 功能 |

**证据来源**: `bundletool.gni:14-46`

---

## frameworks/BUILD.gn

### 配置定义

#### tools_bm_config

```gn
config("tools_bm_config") {
  include_dirs = [
    "include",
    "include/bundle_tool_callback",
  ]

  defines = [
    "APP_LOG_TAG = \"BMSTool\"",
    "LOG_DOMAIN = 0xD001123",
  ]
}
```

**功能**: 公共编译配置

| 属性 | 值 |
|------|------|
| include_dirs | `include/`, `include/bundle_tool_callback/` |
| defines | APP_LOG_TAG, LOG_DOMAIN |

### Source Set 定义

#### tools_bm_source_set (主工具)

```gn
ohos_source_set("tools_bm_source_set") {
  branch_protector_ret = "pac_ret"

  sanitize = {
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    debug = false
    integer_overflow = true
    ubsan = true
  }

  sources = [
    "src/bundle_command.cpp",
    "src/bundle_command_common.cpp",
    "src/main.cpp",
    "src/quick_fix_command.cpp",
    "src/quick_fix_status_callback_host_impl.cpp",
    "src/shell_command.cpp",
    "src/status_receiver_impl.cpp",
  ]

  public_configs = [ ":tools_bm_config" ]

  cflags = [ "-fstack-protector-strong" ]
  cflags_cc = cflags

  if (target_cpu == "arm") {
    cflags += [ "-DBINDER_IPC_32BIT" ]
  }

  external_deps = [
    "ability_base:want",
    "ability_runtime:app_manager",
    "ability_runtime:quickfix_manager",
    "bundle_framework:appexecfwk_base",
    "bundle_framework:appexecfwk_core",
    "bundle_framework:bundle_tool_libs",
    "c_utils:utils",
    "common_event_service:cesfwk_innerkits",
    "hilog:libhilog",
    "init:libbegetutil",
    "ipc:ipc_core",
    "os_account:os_account_innerkits",
    "samgr:samgr_proxy",
  ]

  public_external_deps = [
    "bundle_framework:bundle_napi_common",
    "bundle_framework:libappexecfwk_common",
    "json:nlohmann_json_static",
  ]

  defines = []
  if (account_enable_bm) {
    external_deps += [ "os_account:os_account_innerkits" ]
    defines += [ "ACCOUNT_ENABLE" ]
  }

  if (overlay_install_bm) {
    defines += [ "BUNDLE_FRAMEWORK_OVERLAY_INSTALLATION" ]
  }

  subsystem_name = "bundlemanager"
  part_name = "bundle_tool"
}
```

**证据来源**: `frameworks/BUILD.gn:29-94`

#### tools_test_bm_source_set (测试工具)

```gn
ohos_source_set("tools_test_bm_source_set") {
  # ... (类似配置，额外的测试依赖)
  sources += [ "src/quick_fix_status_callback_host_impl.cpp" ]

  if (quick_fix_bm) {
    defines += [ "BUNDLE_FRAMEWORK_QUICK_FIX" ]
  }

  if (account_enable_bm) {
    external_deps += [ "os_account:os_account_innerkits" ]
    defines += [ "ACCOUNT_ENABLE" ]
  }

  if (distributed_bundle_framework_bm) {
    external_deps += [ "distributed_bundle_framework:dbms_fwk" ]
    defines += [ "DISTRIBUTED_BUNDLE_FRAMEWORK" ]
  }
}
```

**证据来源**: `frameworks/BUILD.gn:107-186`

### 可执行文件定义

#### bm (主可执行文件)

```gn
ohos_executable("bm") {
  deps = [ ":tools_bm_source_set" ]

  external_deps = [ "hilog:libhilog" ]

  install_enable = true

  subsystem_name = "bundlemanager"
  part_name = "bundle_tool"
}
```

**属性**:
| 属性 | 值 |
|------|------|
| 类型 | ohos_executable |
| deps | tools_bm_source_set |
| install_enable | true |
| subsystem | bundlemanager |
| part | bundle_tool |

#### bundle_test_tool (测试工具)

```gn
ohos_executable("bundle_test_tool") {
  deps = [ ":tools_test_bm_source_set" ]

  install_enable = false

  external_deps = [ "hilog:libhilog" ]

  subsystem_name = "bundlemanager"
  part_name = "bundle_tool"
}
```

**属性**:
| 属性 | 值 |
|------|------|
| 类型 | ohos_executable |
| deps | tools_test_bm_source_set |
| install_enable | **false** |

#### group (工具组)

```gn
group("tools_bm") {
  deps = [
    ":bm",
    ":bundle_test_tool",
  ]
}
```

**证据来源**: `frameworks/BUILD.gn:96-204`

---

## Targets 汇总

### 完整列表

| # | target | 类型 | 开关 | 产物 | install |
|---|--------|------|------|------|---------|
| 1 | `tools_bm_config` | config | - | - | - |
| 2 | `tools_bm_source_set` | source_set | - | lib | - |
| 3 | `bm` | executable | - | bm | ✅ |
| 4 | `tools_test_bm_source_set` | source_set | quick_fix_bm, account_enable_bm, distributed_bm | lib | - |
| 5 | `bundle_test_tool` | executable | - | bundle_test_tool | ❌ |
| 6 | `tools_bm` | group | - | - | - |

### 产物清单

| 产物 | 源文件 | 预计路径 | 说明 |
|------|--------|----------|------|
| `bm` | main.cpp | 系统安装目录 | 主可执行文件 |
| `bundle_test_tool` | main_test_tool.cpp | out/... (不安装) | 测试工具 |

---

## 源文件列表

### tools_bm_source_set

| # | 源文件 | 大小 | 功能 |
|---|--------|------|------|
| 1 | `src/main.cpp` | 884B | 程序入口 |
| 2 | `src/bundle_command.cpp` | 128KB | 命令实现 |
| 3 | `src/bundle_command_common.cpp` | 27KB | 公共命令 |
| 4 | `src/shell_command.cpp` | 3KB | Shell 命令基类 |
| 5 | `src/quick_fix_command.cpp` | 6KB | 快速修复 |
| 6 | `src/status_receiver_impl.cpp` | 2KB | 状态接收 |
| 7 | `src/quick_fix_status_callback_host_impl.cpp` | 2KB | 回调实现 |

### tools_test_bm_source_set

| # | 源文件 | 功能 |
|---|--------|------|
| 1 | `src/main_test_tool.cpp` | 测试入口 |
| 2 | `src/bundle_test_tool.cpp` | 291KB | 测试工具 |
| 3 | `src/bundle_tool_callback_stub.cpp` | 回调存根 |

**证据来源**: `ls -la /Volumes/lexar/code/d/work/oh/foundation/bundlemanager/bundle_tool/frameworks/src/`

---

## 外部依赖

### external_deps

| 依赖 | 用途 |
|------|------|
| `ability_base:want` | Want 结构 |
| `ability_runtime:app_manager` | App 管理 |
| `ability_runtime:quickfix_manager` | 快速修复管理 |
| `bundle_framework:appexecfwk_base` | Bundle 基础 |
| `bundle_framework:appexecfwk_core` | Bundle 核心 |
| `bundle_framework:bundle_tool_libs` | Bundle 工具库 |
| `c_utils:utils` | C 工具库 |
| `common_event_service:cesfwk_innerkits` | 公共事件 |
| `hilog:libhilog` | 日志 |
| `init:libbegetutil` | 初始化工具 |
| `ipc:ipc_core` | IPC 核心 |
| `os_account:os_account_innerkits` | 账号 |
| `samgr:samgr_proxy` | SA 代理 |
| `ffrt:libffrt` | FFRT (来自 bm_install_external_deps) |

### public_external_deps

| 依赖 | 用途 |
|------|------|
| `bundle_framework:bundle_napi_common` | Bundle N-API 公共 |
| `bundle_framework:libappexecfwk_common` | AppExecFWK 公共 |
| `json:nlohmann_json_static` | JSON 库 |

---

## 条件编译

### ACCOUNT_ENABLE

```gn
if (account_enable_bm) {
  external_deps += [ "os_account:os_account_innerkits" ]
  defines += [ "ACCOUNT_ENABLE" ]
}
```

### BUNDLE_FRAMEWORK_OVERLAY_INSTALLATION

```gn
if (overlay_install_bm) {
  defines += [ "BUNDLE_FRAMEWORK_OVERLAY_INSTALLATION" ]
}
```

### BUNDLE_FRAMEWORK_QUICK_FIX

```gn
if (quick_fix_bm) {
  defines += [ "BUNDLE_FRAMEWORK_QUICK_FIX" ]
}
```

### DISTRIBUTED_BUNDLE_FRAMEWORK

```gn
if (distributed_bundle_framework_bm) {
  external_deps += [ "distributed_bundle_framework:dbms_fwk" ]
  defines += [ "DISTRIBUTED_BUNDLE_FRAMEWORK" ]
}
```

---

## 编译产物映射

### 推导关系

```
bm (可执行文件)
  ├── deps: tools_bm_source_set
  │     ├── sources: 7 个 cpp 文件
  │     ├── external_deps: 14 个依赖
  │     └── public_configs: tools_bm_config
  └── external_deps: hilog

bundle_test_tool (可执行文件)
  ├── deps: tools_test_bm_source_set
  │     ├── sources: 6 个 cpp 文件 + 测试文件
  │     ├── external_deps: 20+ 个依赖
  │     └── conditional: quick_fix_bm, account_enable_bm, distributed_bm
  └── external_deps: hilog
```

### 安装路径

| 产物 | 安装位置 |
|------|----------|
| `bm` | `/system/bin/bm` 或等价位置 |
| `bundle_test_tool` | 不安装 |

---

## 构建命令

### 完整构建

```bash
# 使用 hb (HarmonyOS Build)
hb set -p <platform> bundlemanager
hb build -f

# 或使用 GN + Ninja
gn gen out/<target>
ninja -C out/<target> bm
```

### 仅构建 bundle_tool

```bash
ninja -C out/<target> //foundation/bundlemanager/bundle_tool:bm
```

### 构建测试工具

```bash
ninja -C out/<target> //foundation/bundlemanager/bundle_tool:bundle_test_tool
```

---

## 安全编译选项

### Sanitizer 配置

```gn
sanitize = {
  boundary_sanitize = true    # 边界 sanitizer
  cfi = true                   # 控制流完整性
  cfi_cross_dso = true         # 跨 DSO CFI
  debug = false
  integer_overflow = true      # 整数溢出检测
  ubsan = true                 # 未定义行为检测
}
```

### 栈保护

```gn
cflags = [ "-fstack-protector-strong" ]
```

**证据来源**: `frameworks/BUILD.gn:32-39`, `53`

---

## 相关文档

- [01_Architecture.md](./01_Architecture.md) - 架构设计
- [03_Inner_API.md](./03_Inner_API.md) - 内部 API
- [05_Security.md](./05_Security.md) - 安全评审
