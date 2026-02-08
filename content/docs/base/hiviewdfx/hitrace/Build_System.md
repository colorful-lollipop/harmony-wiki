# GN 构建系统

## 构建配置

### 根配置文件

**文件位置**: `/base/hiviewdfx/hitrace/hitrace.gni`

**路径变量定义**:
```gni
hitrace_path = "//base/hiviewdfx/hitrace"
hitrace_common_path = "$hitrace_path/common"
hitrace_cmd_path = "$hitrace_path/cmd"
hitrace_config_path = "$hitrace_path/config"
hitrace_frameworks_path = "$hitrace_path/frameworks"
hitrace_interfaces_path = "$hitrace_path/interfaces"
hitrace_utils_path = "$hitrace_path/utils"
hitrace_example_path = "$hitrace_path/example"
```

**功能开关** (`hitrace.gni:23-36`):
| 变量 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `hitrace_support_executable_file` | bool | true | 支持可执行文件 |
| `hitrace_snapshot_tracebuffer_size` | int | 0 | 快照追踪缓冲区大小 |
| `hiview_enable` | bool | auto-detect | HiView 集成 |
| `hitrace_snapshot_file_limit` | int | 0 | 快照文件限制 |
| `hitrace_record_file_limit` | int | 0 | 记录文件限制 |
| `hitrace_feature_enable_pgo` | bool | false | 启用 PGO 优化 |
| `hitrace_feature_pgo_path` | string | "" | PGO 路径 |
| `use_shared_libz` | bool | true | 使用共享 zlib |
| `hitrace_feature_support_usr_symlink` | bool | false | 支持 /usr/bin 符号链接 |

**证据来源**: `hitrace.gni:14-37`

---

## 构建目标清单

### 根目标 (BUILD.gn)

**文件位置**: `/base/hiviewdfx/hitrace/BUILD.gn`

```gn
group("hitrace_all_target") {
  deps = [
    "$hitrace_cmd_path:hitrace_target",
    "$hitrace_config_path:hitrace.cfg",
    "$hitrace_example_path:hitrace_example_target",
    "$hitrace_frameworks_path/hitrace_ndk:hitrace_ndk",
    "$hitrace_interfaces_path/cj/kits:hitrace_ffi",
    "$hitrace_interfaces_path/ets/ani:ani_hitracechain_package",
    "$hitrace_interfaces_path/ets/ani:ani_hitracemeter_package",
    "$hitrace_interfaces_path/js/kits:hitrace_napi",
    "$hitrace_interfaces_path/native/innerkits:hitrace_dump",
    "$hitrace_interfaces_path/native/innerkits:hitrace_meter",
    "$hitrace_interfaces_path/native/innerkits:libhitrace_option",
    "$hitrace_interfaces_path/native/innerkits:libhitracechain",
    "$hitrace_interfaces_path/rust/innerkits/hitrace_meter:hitrace_meter_rust",
    "$hitrace_interfaces_path/rust/innerkits/hitracechain:hitracechain_rust",
  ]
}
```

---

## Native 核心库

### 位置: `interfaces/native/innerkits/BUILD.gn`

#### libhitracechain (共享库)

| 属性 | 值 |
|------|-----|
| **目标类型** | `ohos_shared_library` |
| **输出文件** | `libhitracechain.so` |
| **源码** | 来自 `hitracechain_source` |
| **依赖** | `hilog:libhilog` |
| **安全编译** | `-fstack-protector-strong` |
| **安装路径** | `system`, `updater` |
| **API 标签** | `chipsetsdk_sp`, `platformsdk` |
| **版本脚本** | `libhitracechain.map` |

**证据来源**: `interfaces/native/innerkits/BUILD.gn:26-58`

#### hitrace_meter (共享库)

| 属性 | 值 |
|------|-----|
| **目标类型** | `ohos_shared_library` |
| **输出文件** | `hitrace_meter.so` |
| **源码** | 链接 `hitrace_inner`, `hitrace_etc`, `libhitracechain` |
| **依赖** | `bounds_checking_function:libsec_shared`, `hilog` |
| **API 标签** | `chipsetsdk_sp`, `platformsdk`, `sasdk` |
| **安装路径** | `system`, `updater` |

**证据来源**: `interfaces/native/innerkits/BUILD.gn:172-212`

#### hitrace_dump (共享库)

| 属性 | 值 |
|------|-----|
| **目标类型** | `ohos_shared_library` |
| **输出文件** | `hitrace_dump.so` |
| **源码** | `hitrace_dump.cpp`, `hitrace_util.cpp`, `dynamic_buffer.cpp` |
| **依赖** | `hitrace_meter`, `libhitrace_option`, `tracedump_executor`, `utils` |
| **优化选项** | `-Os`, `-Wl,--gc-sections` |

**证据来源**: `interfaces/native/innerkits/BUILD.gn:120-170`

#### libhitrace_option (共享库)

| 属性 | 值 |
|------|-----|
| **目标类型** | `ohos_shared_library` |
| **输出文件** | `libhitrace_option.so` |
| **源码** | `src/hitrace_option/hitrace_option.cpp` |
| **依赖** | `init:libbegetutil`, `hilog` |

**证据来源**: `interfaces/native/innerkits/BUILD.gn:214-248`

---

## 框架层

### 位置: `frameworks/native/BUILD.gn`

#### hitracechain_source (源集合)

| 属性 | 值 |
|------|-----|
| **目标类型** | `ohos_source_set` |
| **源码** | `hitracechain.cpp`, `hitracechainc.c`, `hitraceid.cpp`, `hitrace_meter*.cpp` |
| **包含目录** | `common`, `interfaces/innerkits/include`, `frameworks/include`, `utils` |

**证据来源**: `frameworks/native/BUILD.gn:17-51`

---

## N-API 接口层

### 位置: `interfaces/js/kits/napi/BUILD.gn`

#### hitracechain_napi

| 属性 | 值 |
|------|-----|
| **目标类型** | `ohos_shared_library` |
| **输出路径** | `module/hitracechain_napi.so` |
| **源码** | `napi_hitrace_init.cpp`, `napi_hitrace_js.cpp`, `napi_hitrace_util.cpp` |
| **依赖** | `libhitracechain`, `ace_napi` (napi), `hilog` |

**证据来源**: `interfaces/js/kits/napi/BUILD.gn:17-45`

#### hitracemeter_napi

| 属性 | 值 |
|------|-----|
| **目标类型** | `ohos_shared_library` |
| **输出路径** | `module/hitracemeter_napi.so` |
| **源码** | `hitracemeter/napi_hitrace_meter.cpp` |
| **依赖** | `hitrace_meter`, `ace_napi`, `hilog` |

**证据来源**: `interfaces/js/kits/napi/BUILD.gn:47-69`

---

## 编译产物清单

### 产物映射表

| Target | 输出文件 | 安装路径 | 用途 |
|--------|----------|----------|------|
| `libhitracechain` | `libhitracechain.so` | `system/lib` | 调用链核心 |
| `hitrace_meter` | `hitrace_meter.so` | `system/lib` | 性能追踪 |
| `hitrace_dump` | `hitrace_dump.so` | `system/lib` | 追踪转储 |
| `libhitrace_option` | `libhitrace_option.so` | `system/lib` | 配置选项 |
| `hitrace_ndk` | `libhitrace_ndk.so` | `system/lib` | NDK 接口 |
| `hitracechain_napi` | `hitracechain_napi.so` | `system/lib/module` | JS 调用链 |
| `hitracemeter_napi` | `hitracemeter_napi.so` | `system/lib/module` | JS 性能追踪 |
| `bytrace` | `bytrace.so` | `system/lib/module` | JS 遗留接口 |
| `hitrace` | `hitrace` | `system/bin` | 命令行工具 |
| `hitrace_tags` | `hitrace_utils.json` | `system/etc` | 追踪标签配置 |

---

## 依赖关系图

```
hitrace_all_target
│
├── hitrace (可执行文件)
│   ├── hitrace_cmd.cpp
│   ├── hitrace_tags (配置)
│   └── utils (工具库)
│
├── hitrace_ndk
│   ├── hitrace_meter
│   └── libhitracechain
│
├── hitrace_napi
│   ├── hitracechain_napi
│   │   └── libhitracechain
│   └── hitracemeter_napi
│       └── hitrace_meter
│
├── hitrace_dump
│   ├── hitrace_meter
│   ├── libhitrace_option
│   ├── tracedump_executor
│   └── trace_factory
│
└── libhitracechain (核心库)
    └── hitracechain_source
        ├── hitrace_meter
        └── utils
```

---

## 常见编译问题

### 问题 1: 缺少 hilog 依赖

**错误现象**: 链接失败，提示找不到 `HilogSymbol`

**解决方案**: 确保在 `external_deps` 中添加 hilog 依赖

```gn
ohos_shared_library("your_target") {
  external_deps = [ "hilog:libhilog" ]
}
```

### 问题 2: N-API 编译失败

**错误现象**: 找不到 `napi.h` 或相关符号

**解决方案**: 添加 napi 依赖

```gn
ohos_shared_library("your_napi_target") {
  external_deps = [ "napi:ace_napi" ]
}
```

### 问题 3: PGO 优化编译失败

**错误现象**: 提示 `profile-use` 文件不存在

**解决方案**: 确保 PGO 路径正确或禁用 PGO

```gn
hitrace_feature_enable_pgo = false  # 或
hitrace_feature_pgo_path = "/path/to/profdata"
```

---

## 构建命令

### 完整构建

```bash
# 生成构建文件
gn gen out/default --args="target_os=\"ohos\" target_cpu=\"arm64\""

# 构建所有目标
ninja -C out/default hitrace_all_target
```

### 单独构建某个模块

```bash
# 构建 N-API 模块
ninja -C out/default //base/hiviewdfx/hitrace/interfaces/js/kits/napi:hitracechain_napi

# 构建命令行工具
ninja -C out/default //base/hiviewdfx/hitrace/cmd:hitrace
```

### 查看所有目标

```bash
ninja -C out/default -t targets | grep hitrace
```
