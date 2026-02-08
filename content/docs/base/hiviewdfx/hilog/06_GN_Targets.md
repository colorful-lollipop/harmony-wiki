# HiLog GN Targets 文档

> 生成时间: 2026-02-06
> 相关证据: `*.BUILD.gn`, `hilog.gni`

---

## 目的

本文档详细描述 HiLog 模块的 GN 构建系统，包括所有 targets、类型、依赖、输出产物。

## 适用范围

涵盖 `base/hiviewdfx/hilog/` 下的所有 BUILD.gn 文件（忽略测试目录）。

---

## 全局配置

### hilog.gni

**路径**: `/base/hiviewdfx/hilog/hilog.gni`

**定义内容**:

```gni
platforms = [
  "ohos",
  "windows",
  "mac",
  "linux",
  "android",
  "ios",
]

declare_args() {
  hilog_native_feature_ohcore = false
  hilog_feature_support_usr_symlink = false
}
```

**证据**: `hilog.gni:14-26`

### Feature Flags

| Flag | 默认值 | 描述 | 影响 |
|------|---------|------|--------|
| `hilog_native_feature_ohcore` | false | 控制 ohcore 原生功能 | 减小 ROM/RAM |
| `hilog_feature_support_usr_symlink` | false | 启用 /usr/bin/hilog 符号链接 | 工具可访问性 |

---

## Target Groups

### Group 1: 服务 Targets

#### hilogd

**文件**: `services/hilogd/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `hilogd` |
| **Target Type** | `ohos_executable` |
| **Output** | `hilogd` |
| **Install** | `/system/bin/hilogd` |

**Sources** (13 个文件):
```
cmd_executor.cpp
flow_control.cpp
kmsg_parser.cpp
log_buffer.cpp
log_collector.cpp
log_compress.cpp
log_domains.cpp
log_kmsg.cpp
log_persister.cpp
log_persister_rotator.cpp
log_stats.cpp
main.cpp
service_controller.cpp
```

**Configs**:
- `:hilogd_config` - include_dirs: ["include"]
- `//base/hiviewdfx/hilog/frameworks/libhilog:libhilog_config`

**Defines**:
- `__RECV_MSG_WITH_UCRED_`

**Deps**:
- `../../interfaces/native/innerkits:libhilog`
- `etc:hilogd_etc`

**External Deps**:
- `bounds_checking_function:libsec_shared`
- `ffrt:libffrt`
- `init:libbegetutil`
- `zlib:shared_libz`

**Install Images**: `system`, `updater`

**Evidence**: `services/hilogd/BUILD.gn:22-68`

#### hilogd_etc

**文件**: `services/hilogd/etc/BUILD.gn`

| 子 Target | Type | Source | Install Path |
|----------|------|--------|-------------|
| `hilogd.cfg` | ohos_prebuilt_etc | hilogd.cfg | `init/` |
| `hilog.para` | ohos_prebuilt_etc | hilog.para | `param/` |
| `hilog.para.dac` | ohos_prebuilt_etc | hilog.para.dac | `param/` |

---

#### hilog

**文件**: `services/hilogtool/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `hilog` |
| **Target Type** | `ohos_executable` |
| **Output** | `hilog` |
| **Install** | `/system/bin/hilog` |

**Sources** (2 个文件):
```
log_display.cpp
main.cpp
```

**Configs**:
- `:hilog_config` - include_dirs: ["include"]
- `//base/hiviewdfx/hilog/frameworks/libhilog:libhilog_config`

**Deps**:
- `../../interfaces/native/innerkits:libhilog`

**External Deps**:
- `bounds_checking_function:libsec_shared`
- `c_utils:utils`
- `zlib:shared_libz`

**Install Images**: `system`, `updater`

**Symlink** (if `hilog_feature_support_usr_symlink`):
- `../usr/bin/hilog`

**Evidence**: `services/hilogtool/BUILD.gn:23-60`

---

### Group 2: 框架 Targets

#### libhilog_source (Template)

**文件**: `frameworks/libhilog/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `libhilog_source_<platform>` |
| **Target Type** | `ohos_source_set` (template) |

**Common Sources**:
```
hilog.cpp
hilog_printf.cpp
log_print.cpp (utils)
log_utils.cpp (utils)
```

**Platform-Specific Sources**:
| Platform | 额外 Sources | Defines |
|----------|---------------|--------|
| ohos/android/ios | properties.cpp, log_ioctl.cpp, all socket sources | `__OHOS__`, `__RECV_MSG_WITH_UCRED_`, `HILOG_USE_MUSL` |
| windows | Base only | `__WINDOWS__`, `-std=c++17` |
| mac | Base only | `__MAC__`, `-std=c++17`, `-Wno-deprecated-declarations` |
| linux | Base only | `__LINUX__`, `-std=c++17` |

---

#### libhilog

**文件**: `interfaces/native/innerkits/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `libhilog` |
| **Target Type** | `ohos_shared_library` |
| **Output** | `libhilog.so` |
| **Install** | `/system/lib/` |

**Platform Variants**:
| Variant | Condition | Type |
|---------|-----------|------|
| `libhilog` | is_mingw/is_mac/is_linux/is_ohos | shared_library |
| `libhilog_<platform>` | windows/mac/linux/ios/android | template-generated |

**Deps** (Platform Dependent):
| Platform | Source Dependency |
|----------|-------------------|
| ohos | `../../../frameworks/libhilog:libhilog_source_ohos` |
| mingw | `../../../frameworks/libhilog:libhilog_source_windows` |
| mac | `../../../frameworks/libhilog:libhilog_source_mac` |
| linux | `../../../frameworks/libhilog:libhilog_source_linux` |

**Configs** (ohos):
- `:libhilog_pub_config` - include_dirs: ["include"]

**Version Script**: `libhilog.map`

**Inner API Tags**: `chipsetsdk`, `platformsdk`, `sasdk`

**Install Images**: `system`, `updater`

**Install Enable**: `!hilog_native_feature_ohcore`

**Branch Protector**: `pac_ret`

**Sanitize**: CFI enabled, cfi_cross_dso enabled

**Evidence**: `interfaces/native/innerkits/BUILD.gn:32-68`

#### libhilog_base

**文件**: `interfaces/native/innerkits/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `libhilog_base` |
| **Target Type** | `ohos_static_library` |

**Sources**:
```
$libhilog_base_root/hilog_base.c
$vsnprintf_root/vsnprintf_s_p.c
```

**Include Dirs**:
```
include
../../../frameworks/libhilog/include
../../../frameworks/libhilog/vsnprintf/include
```

**Defines**:
- `__RECV_MSG_WITH_UCRED_`
- `HILOG_PROHIBIT_ALLOCATION`

**External Deps**:
- `bounds_checking_function:libsec_static`

**Evidence**: `interfaces/native/innerkits/BUILD.gn:169-190`

#### libhilog_base_for_musl

**文件**: `interfaces/native/innerkits/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `libhilog_base_for_musl` |
| **Target Type** | `ohos_static_library` |

**Special Flags**:
- ldflags: `-nostdlib`
- remove_configs: musl_inherited_configs_for_musl
- configs: `//build/config/components/musl:soft_musl_config`

---

#### libhilog_snapshot

**文件**: `interfaces/native/innerkits/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `libhilog_snapshot` |
| **Target Type** | `ohos_static_library` |

**Sources**:
```
$libhilog_snapshot_root/hilog_snapshot.c
```

**Include Dirs**:
```
include
../../../frameworks/libhilog/include
```

**Branch Protector**: `pac_ret`

**Sanitize**: CFI enabled, cfi_cross_dso enabled

**Evidence**: `interfaces/native/innerkits/BUILD.gn:200-218`

#### libhilog_host

**文件**: `interfaces/native/innerkits/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `libhilog_host` |
| **Target Type** | `group` |

**Deps**:
- `:libhilog($host_toolchain)`

---

### Group 3: NDK Target

#### hilog_ndk

**文件**: `frameworks/hilog_ndk/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `hilog_ndk` |
| **Target Type** | `ohos_shared_library` |
| **Output** | `libhilog_ndk.so` |
| **Install** | `/system/lib/` |

**Sources**:
```
hilog_ndk.c
```

**Deps**:
- `../../interfaces/native/innerkits:libhilog`

**External Deps**:
- `bounds_checking_function:libsec_shared`

**Install Images**: `system_base_dir`, `updater`

**Inner API Tags**: `ndk`

**Branch Protector**: `pac_ret`

**Sanitize**: CFI enabled, cfi_cross_dso enabled

**Evidence**: `frameworks/hilog_ndk/BUILD.gn:19-35`

---

### Group 4: JS N-API Targets

#### hilog_napi (Group)

**文件**: `interfaces/js/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `hilog_napi` |
| **Target Type** | `group` |
| **Condition** | `support_jsapi` |

**Deps**:
- `//base/hiviewdfx/hilog/interfaces/js/kits/napi:libhilognapi`

**Evidence**: `interfaces/js/BUILD.gn:16-21`

#### libhilognapi

**文件**: `interfaces/js/kits/napi/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `libhilognapi` |
| **Target Type** | `ohos_shared_library` |
| **Output** | `libhilog_napi.so` |
| **Install** | `/system/lib/module/` |

**Source Set**: `:libhilognapi_src`

**Sources** (6 个文件):
```
src/common/napi/n_class.cpp
src/common/napi/n_func_arg.cpp
src/common/napi/n_val.cpp
src/hilog/module.cpp
src/hilog/src/hilog_napi.cpp
src/hilog/src/hilog_napi_base.cpp
```

**Include Dirs**:
```
include
//base/hiviewdfx/hilog/frameworks/libhilog/param/include
//base/hiviewdfx/hilog/frameworks/libhilog/include
//base/hiviewdfx/hilog/interfaces/js/kits/napi/src/common/napi
//base/hiviewdfx/hilog/interfaces/js/kits/napi/src/hilog/include/context
```

**Platform Deps**:
| Platform | libhilog Dependency |
|----------|---------------------|
| mingw | `libhilog_windows` |
| mac | `libhilog_mac` |
| linux | `libhilog_linux` |
| ohos | `libhilog` (innerkits) |

**Platform Defines**: `__WINDOWS__`, `__MAC__`, `__LINUX__`

**Sanitize**: CFI enabled, cfi_cross_dso enabled

**Relative Install Dir**: `module/`

**Evidence**: `interfaces/js/kits/napi/BUILD.gn:17-82`

---

### Group 5: Rust Target

#### hilog_rust

**文件**: `interfaces/rust/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `hilog_rust` |
| **Target Type** | `ohos_rust_shared_library` |
| **Crate Name** | `hilog_rust` |
| **Crate Type** | `dylib` |

**Sources**:
```
src/lib.rs
src/macros.rs
```

**Deps**:
- `../../interfaces/native/innerkits:libhilog`

**Rustflags**:
- `-Zstack-protector=all`

**Evidence**: `interfaces/rust/BUILD.gn:14-23`

---

### Group 6: ETS/ANI Targets

#### hilog_ani (Group)

**文件**: `interfaces/ets/ani/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `ani_hilog_package` |
| **Target Type** | `group` |
| **Condition** | `support_jsapi` |

**Deps**:
- `hilog:hilog_ani`
- `hilog:hilog_etc`

**Evidence**: `interfaces/ets/ani/BUILD.gn:13-21`

#### hilog_ani

**文件**: `interfaces/ets/ani/hilog/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `hilog_ani` |
| **Target Type** | `ohos_shared_library` |
| **Output** | `libhilog_ani.so` |

**Sources** (3 个文件):
```
src/ani_util.cpp
src/hilog_ani.cpp
src/hilog_ani_base.cpp
```

**Include Dirs**:
```
include
```

**External Deps**:
- `bounds_checking_function:libsec_shared`
- `runtime_core:ani`
- `runtime_core:libarkruntime`

**Platform Deps**:
| Platform | libhilog Dependency |
|----------|---------------------|
| mingw | `libhilog_windows` |
| mac | `libhilog_mac` |
| linux | `libhilog_linux` |
| default | `libhilog` (innerkits) |

**Evidence**: `interfaces/ets/ani/hilog/BUILD.gn:19-65`

#### hilog (Static ABC)

**文件**: `interfaces/ets/ani/hilog/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `hilog` |
| **Target Type** | `generate_static_abc` |
| **Output** | `hilog.abc` |

**Sources**:
```
./ets/@ohos.hilog.ets
```

**Device Destination**: `/system/framework/hilog.abc`

**Is Boot ABC**: `True`

**Evidence**: `interfaces/ets/ani/hilog/BUILD.gn:56-62`

#### hilog_etc

**文件**: `interfaces/ets/ani/hilog/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `hilog_etc` |
| **Target Type** | `ohos_prebuilt_etc` |
| **Source** | `$target_out_dir/hilog.abc` |
| **Module Install Dir** | `framework` |

**Evidence**: `interfaces/ets/ani/hilog/BUILD.gn:64-71`

---

### Group 7: 平台 Targets

#### libhilog_platform_source (Template)

**文件**: `platform/BUILD.gn`

| 属性 | 值 |
|------|------|------|
| **Target Name** | `libhilog_platform_source_<platform>` |
| **Target Type** | `ohos_source_set` (template) |

**Sources**:
```
hilog.cpp
hilog_printf.cpp
hilog_utils.cpp
interface/native/log.cpp
```

**Defines**:
- `DFX_PLATFORM_LOG_TAG="Ace"`

**Platform Defines**:
| Platform | Defines |
|----------|--------|
| ANDROID_PLATFORM | `ANDROID_PLATFORM` (if target_os == "android") |
| IOS_PLATFORM | `IOS_PLATFORM` (if target_os == "ios") |

**Evidence**: `platform/BUILD.gn:1-34`

---

## Target 依赖关系

### 依赖图

```
hilogd (executable)
├─→ libhilog (shared)
├─→ hilogd_etc (group)
│   ├─→ hilogd.cfg (prebuilt)
│   ├─→ hilog.para (prebuilt)
│   └─→ hilog.para.dac (prebuilt)
├─→ libsec_shared (external)
├─→ libffrt (external)
├─→ libbegetutil (external)
└─→ shared_libz (external)

hilog (executable)
├─→ libhilog (shared)
├─→ libsec_shared (external)
├─→ utils (external)
└─→ shared_libz (external)

libhilog (shared)
└─→ libhilog_source_<platform> (source set)

libhilognapi (shared)
├─→ libhilognapi_src (source set)
└─→ libhilog (shared, platform dependent)

hilog_ndk (shared)
└─→ libhilog (shared)

hilog_rust (shared)
└─→ libhilog (shared)

hilog_ani (shared)
├─→ libhilog (shared, platform dependent)
├─→ libsec_shared (external)
├─→ ani (external)
└─→ libarkruntime (external)

libhilog_base (static)
├─→ libsec_static (external)
└─→ vsnprintf_s_p.c (source)
```

---

## 条件编译选项

### Platform Defines

| Define | Condition | 效果 |
|--------|-----------|--------|
| `__OHOS__` | `platform == "ohos"` | OpenHarmony 平台构建 |
| `__WINDOWS__` | `is_mingw` | Windows 平台构建 |
| `__MAC__` | `is_mac` | macOS 平台构建 |
| `__LINUX__` | `is_linux` | Linux 平台构建 |
| `__RECV_MSG_WITH_UCRED_` | Non-desktop platforms | 使用用户凭证在 socket 消息 |
| `HILOG_USE_MUSL` | `use_musl` | 使用 musl libc |
| `HILOG_PROHIBIT_ALLOCATION` | libhilog_base | 禁止动态分配 |
| `ANDROID_PLATFORM` | target_os == "android"` | Android 平台构建 |
| `IOS_PLATFORM` | target_os == "ios"` | iOS 平台构建 |

---

## 编译选项汇总

### 安全特性

| 选项 | Targets | 说明 |
|------|---------|------|
| **Branch Protection: pac_ret** | 所有 executable, shared_library targets | 返回地址保护 |
| **CFI (Control Flow Integrity)** | 所有 executable, shared_library targets | 控制流完整性 |
| **CFI Cross-DSO** | 所有 executable, shared_library targets | 跨库 CFI 检查 |
| **Sanitize: debug = false** | 所有 executable, shared_library targets | 调试信息禁用 |

---

## 相关跳转链接

- [编译产物](07_Build_Artifacts.md)
- [目录结构](02_Directory_Structure.md)

---

## 证据索引

| 主题 | 文件路径 | 行号/符号 |
|------|---------|----------|
| 全局配置 | hilog.gni:14-26 | platforms, feature flags |
| hilogd target | services/hilogd/BUILD.gn:22-68 | 全文 |
| hilog target | services/hilogtool/BUILD.gn:23-60 | 全文 |
| libhilog target | interfaces/native/innerkits/BUILD.gn:32-68 | 全文 |
| N-API targets | interfaces/js/kits/napi/BUILD.gn, interfaces/js/BUILD.gn | 全文 |
| Rust target | interfaces/rust/BUILD.gn:14-23 | 全文 |
| ETS targets | interfaces/ets/ani/BUILD.gn:13-71 | 全文 |
