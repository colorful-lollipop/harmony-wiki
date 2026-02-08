# GN 构建配置

> Targets 列表、依赖关系与构建产物

## 1. 构建系统概述

ets_utils 使用 **GN (Generate Ninja)** 构建系统。

### 1.1 配置文件

| 文件 | 用途 |
|------|------|
| `bundle.json` | 组件配置 (子系统、子组件定义) |
| `ets_utils_config.gni` | 全局配置 (路径、开关) |
| `*/BUILD.gn` | 各模块构建配置 |

### 1.2 构建流程

```
.ts/.js 源文件
    ↓ (build_ts_js.py)
.js 中间文件
    ↓ (es2abc)
.abc 字节码
    ↓ (gen_obj)
.c/.o 目标文件
    ↓ (ohos_shared_library)
.so 动态库
```

## 2. bundle.json 配置

### 2.1 组件定义

```json
{
  "name": "@ohos/ets_utils",
  "component": {
    "name": "ets_utils",
    "subsystem": "commonlibrary",
    "syscap": ["SystemCapability.Utils.Lang"],
    "features": ["ets_utils_stacksize_low_enable"],
    "adapted_system_type": ["standard"],
    "rom": "1400KB",
    "ram": "~4096KB"
  }
}
```

### 2.2 子组件列表

```json
{
  "build": {
    "sub_component": [
      "//commonlibrary/ets_utils/base_sdk:base_sdk_ets",
      "//commonlibrary/ets_utils/base_sdk:base_transfer_ets",
      "//commonlibrary/ets_utils/js_api_module/uri:uri_packages",
      "//commonlibrary/ets_utils/js_api_module/url:url_packages",
      "//commonlibrary/ets_utils/js_api_module/convertxml:convertxml_packages",
      "//commonlibrary/ets_utils/js_api_module/xml:xml_packages",
      "//commonlibrary/ets_utils/js_api_module/buffer:buffer_packages",
      "//commonlibrary/ets_utils/js_api_module/fastbuffer:fastbuffer_packages",
      "//commonlibrary/ets_utils/js_concurrent_module/utils:utils_packages",
      "//commonlibrary/ets_utils/js_concurrent_module/taskpool:taskpool_packages",
      "//commonlibrary/ets_utils/js_concurrent_module/worker:worker_packages",
      "//commonlibrary/ets_utils/js_sys_module/console:console_packages",
      "//commonlibrary/ets_utils/js_sys_module/process:process_packages",
      "//commonlibrary/ets_utils/js_sys_module/dfx:dfx_packages",
      "//commonlibrary/ets_utils/js_sys_module/timer:timer_packages",
      "//commonlibrary/ets_utils/js_util_module/util:util_packages",
      "//commonlibrary/ets_utils/js_util_module/container:container_packages",
      "//commonlibrary/ets_utils/js_util_module/json:json_packages",
      "//commonlibrary/ets_utils/js_util_module/collections:collections_packages",
      "//commonlibrary/ets_utils/js_util_module/stream:stream_packages"
    ]
  }
}
```

### 2.3 内部接口 (Inner Kits)

```json
{
  "inner_kits": [
    {
      "name": "//commonlibrary/ets_utils/js_sys_module/timer:timer",
      "header": {
        "header_files": ["sys_timer.h"],
        "header_base": "//commonlibrary/ets_utils/js_sys_module/timer"
      }
    },
    {
      "name": "//commonlibrary/ets_utils/js_sys_module/console:console",
      "header": {
        "header_files": ["console.h", "log.h"],
        "header_base": "//commonlibrary/ets_utils/js_sys_module/console"
      }
    },
    {
      "name": "//commonlibrary/ets_utils/js_concurrent_module/worker:worker",
      "header": {
        "header_files": ["worker.h"],
        "header_base": "//commonlibrary/ets_utils/js_concurrent_module/worker"
      }
    }
  ]
}
```

## 3. BUILD.gn Targets 详解

### 3.1 js_api_module/url/BUILD.gn

```
Targets:
├── build_ts_js           # TS → JS 编译
├── gen_url_abc           # JS → ABC 字节码
├── url_js                # JS 目标文件
├── url_abc               # ABC 目标文件
├── url_static            # 静态库 (.a)
├── url                   # 动态库 (.so)
└── url_packages          # 包组
```

**关键配置**:
```gn
ohos_shared_library("url") {
  branch_protector_ret = "pac_ret"  # PAC 返回地址保护
  sanitize = {
    cfi = true                        # 控制流完整性
    cfi_cross_dso = true              # 跨 DSO CFI 检查
    debug = false
  }
  deps = [":url_static"]
  external_deps = ["hilog:libhilog"]
  subsystem_name = "commonlibrary"
  part_name = "ets_utils"
  relative_install_dir = "module"
}
```

### 3.2 js_util_module/container/BUILD.gn

**容器库 Targets** (使用模板生成):
```
arraylist     → libarraylist.so
deque         → libdeque.so
queue         → libqueue.so
vector        → libvector.so
linkedlist    → liblinkedlist.so
list          → liblist.so
stack         → libstack.so
treemap       → libtreemap.so
treeset       → libtreeset.so
hashmap       → libhashmap.so
hashset       → libhashset.so
lightweightmap → liblightweightmap.so
lightweightset → liblightweightset.so
plainarray    → libplainarray.so
struct        → libstruct.so
```

**容器模板**:
```gn
template("container_lib") {
  ohos_source_set(name + "_static") {
    # 静态库
    include_dirs = ["..."]
    sources = [name + "/native_module_" + name + ".cpp"]
    deps = [...]
    configs = [":container_config"]
  }
  
  ohos_shared_library(name) {
    # 动态库
    deps = [":${name}_static"]
    relative_install_dir = "module/util"
  }
}
```

## 4. 依赖关系

### 4.1 外部依赖

| 依赖 | 用途 | 来源 |
|------|------|------|
| `hilog:libhilog` | 日志 | hiviewdfx |
| `napi:ace_napi` | N-API | arkui/napi |
| `bounds_checking_function:libsec_shared` | 安全检查 | c_utils |
| `icu:shared_icuuc` | 国际化 | third_party |
| `ffrt` | 快任务运行时 | - |
| `ipc` | 进程间通信 | - |

### 4.2 内部依赖

```
js_api_module
├── url → hilog, napi
├── xml → libxml2, hilog, napi
├── buffer → hilog, napi
└── convertxml → napi

js_util_module
├── container → hilog, napi
├── util → hilog, napi, icu
└── json → napi

js_sys_module
├── process → ipc, hilog, napi
├── timer → hilog, napi
└── console → hilog, napi

js_concurrent_module
├── worker → ffrt, hilog, napi
├── taskpool → ffrt, hilog, napi
└── utils → hilog, napi
```

## 5. 编译产物

### 5.1 产物列表

| 产物路径 | 类型 | 说明 |
|---------|------|------|
| `module/liburl.so` | .so | URL 模块 |
| `module/libxml.so` | .so | XML 模块 |
| `module/libbuffer.so` | .so | Buffer 模块 |
| `module/libconvertxml.so` | .so | ConvertXml 模块 |
| `module/libfastbuffer.so` | .so | FastBuffer 模块 |
| `module/liburi.so` | .so | URI 模块 |
| `module/util/*.so` | .so | 容器模块 (15个) |
| `module/libprocess.so` | .so | Process 模块 |
| `module/libtimer.so` | .so | Timer 模块 |
| `module/libconsole.so` | .so | Console 模块 |
| `module/libworker.so` | .so | Worker 模块 |
| `module/libtaskpool.so` | .so | Taskpool 模块 |
| `module/libutil.so` | .so | Util 模块 |

### 5.2 字节码产物

```
out/.../url.abc           # URL 字节码
out/.../buffer.abc        # Buffer 字节码
out/.../arraylist.abc      # ArrayList 字节码
...
```

## 6. 安全配置

### 6.1 CFI (Control Flow Integrity)

所有模块启用 CFI 检查：

```gn
sanitize = {
  cfi = true              # 启用 CFI
  cfi_cross_dso = true    # 跨动态库检查
  debug = false
}
```

### 6.2 PAC (Pointer Authentication)

```gn
branch_protector_ret = "pac_ret"  # 返回地址认证
```

## 7. 特性开关

### 7.1 全局开关

| 开关 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `ets_utils_stacksize_low_enable` | bool | false | 低功耗栈大小模式 |

### 7.2 条件编译

```gn
if (is_arkui_x) {
  # ArkUI X 平台特定配置
  deps += ["$plugins_root/libs/icu:icu_${target_os}"]
} else {
  # 标准系统配置
  external_deps = ["hilog:libhilog"]
}
```

## 相关文档

- [08_Compilation_Artifacts.md](./08_Compilation_Artifacts.md) - 编译产物详解
- [02_Architecture.md](./02_Architecture.md) - 架构设计
- [09_Security_Review.md](./09_Security_Review.md) - 安全分析

---

*文档版本: 1.0*
*最后更新: 2026-02-06*
