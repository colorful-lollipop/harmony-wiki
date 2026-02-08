# 构建指南

## GN 构建系统

本模块使用 OpenHarmony 标准 GN (Generate Ninja) 构建系统。

## 根构建入口

**文件**：`//base/security/dataclassification/BUILD.gn`

```gn
import("//build/ohos.gni")

group("dataclassification_build_module") {
  if (os_level == "standard") {
    deps = [ "interfaces/inner_api/datatransmitmgr:data_transit_mgr" ]
  }
}
```

**说明**：
- 仅在 `os_level == "standard"` 时编译
- 依赖 `data_transit_mgr` target

> 证据：`BUILD.gn:14-19`

## 接口层构建配置

**文件**：`//base/security/dataclassification/interfaces/inner_api/datatransmitmgr/BUILD.gn`

### 配置定义

```gn
config("datatransmitmgr_config") {
  include_dirs = [ "include" ]
}
```

### 构建开关

```gn
declare_args() {
  dataclassification_feature_enabled = true
  if (defined(global_parts_info) &&
      !defined(global_parts_info.commonlibrary_c_utils)) {
    dataclassification_feature_enabled = false
  }
}
```

**说明**：
- `dataclassification_feature_enabled`：功能总开关
- 依赖 `c_utils` 组件，若未定义则自动关闭

> 证据：`BUILD.gn:23-29`

### 共享库 Target

```gn
ohos_shared_library("data_transit_mgr") {
  subsystem_name = "security"
  part_name = "dataclassification"

  public_configs = [ ":datatransmitmgr_config" ]

  include_dirs = [ "include" ]

  sources = [
    "../../../frameworks/datatransmitmgr/dev_slinfo_adpt.c",
    "../../../frameworks/datatransmitmgr/dev_slinfo_list.c",
    "../../../frameworks/datatransmitmgr/dev_slinfo_mgr.c",
  ]
  # ...
}
```

> 证据：`BUILD.gn:31-43`

## 产物配置

### 安全加固（仅 standard 版本）

```gn
if (os_level == "standard") {
  branch_protector_ret = "pac_ret"
  sanitize = {
    cfi = true                    # 控制流完整性
    cfi_cross_dso = true         # 跨 DSO CFI
    debug = false
    integer_overflow = true      # 整数溢出检测
    ubsan = true                 # 未定义行为检测
    boundary_sanitize = true     # 边界检查
  }
}
```

### 外部依赖

```gn
if (dataclassification_feature_enabled) {
  external_deps = [ "c_utils:utils" ]
}
external_deps += [
  "device_security_level:dslm_sdk",
  "hilog:libhilog",
]
```

### 编译选项

```gn
defines = [ "HILOG_ENABLE" ]

cflags = [
  "-D_FORTIFY_SOURCE=2",          # FORTIFY 增强
  "-DHILOG_ENABLE",
  "-Wall",
  "-fstack-protector-strong",     # 栈保护
]
```

> 证据：`BUILD.gn:45-72`

## Targets 汇总

| Target | 类型 | 输出产物 | 依赖 |
|-------|------|---------|------|
| `dataclassification_build_module` | group | N/A | 仅 standard 启用 |
| `data_transit_mgr` | ohos_shared_library | libdata_transit_mgr.z.so | dslm_sdk, hilog, c_utils |

## 产物清单

### 共享库

| 产物路径 | 类型 | 说明 |
|---------|------|------|
| `out/.../libs/libdata_transit_mgr.z.so` | 共享库 | 数据传输管控核心库 |

### 安装路径

最终安装到系统分区：
```
/system/lib64/libdata_transit_mgr.z.so
```

或
```
/system/lib/libdata_transit_mgr.z.so
```

（根据设备架构决定）

## 依赖关系

```
data_transit_mgr.so
├── libdslm_sdk.z.so (运行时 dlopen)
├── libhilog.z.so
├── libc_utils.z.so
└── libc.so
```

## 构建命令

### 全量编译

```bash
# 编译 dataclassification 模块
hb set
hb build -p //base/security/dataclassification

# 或通过子系统编译
hb build -p security
```

### 单模块编译

```bash
# 使用 GN 直接编译
python3 build.py --product {product} --build-type release \
  -p //base/security/dataclassification:dataclassification_build_module
```

## 配置开关

| 开关 | 类型 | 默认值 | 说明 |
|-----|------|-------|------|
| `dataclassification_feature_enabled` | boolean | true | 功能总开关 |
| `os_level == "standard"` | condition | - | 仅 standard 版本启用 |

详细配置见 [Config_Flags.md](appendix/Config_Flags.md)
