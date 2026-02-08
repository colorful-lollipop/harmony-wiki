# GN 构建

> utils_lite 的 GN 构建配置完整参考。

## Feature Flags

在 `BUILD.gn` 第 16-21 行定义：

```gn
declare_args() {
  utils_lite_feature_file = false        # 文件操作模块
  utils_lite_feature_kal_timer = false   # KAL 定时器模块
  utils_lite_feature_timer_task = false  # 定时器任务模块
  utils_lite_feature_js_builtin = false  # JS 内置 API 模块
}
```

**证据来源**：`BUILD.gn:16-21`

同时在 `bundle.json` 第 19-24 行注册：

```json
"features": [
    "utils_lite_feature_file",
    "utils_lite_feature_kal_timer",
    "utils_lite_feature_timer_task",
    "utils_lite_feature_js_builtin"
]
```

---

## Targets 清单

### 根级 Targets

| 名称 | 类型 | 输出 | 路径 | 行号 |
|------|------|------|------|------|
| `utils` | group | - | /BUILD.gn:23 | 23 |
| `native_api` | ndk_lib | .so (非 liteos_m) | /BUILD.gn:30 | 30 |
| `utils_lite` | group | - | /BUILD.gn:42 | 42 |

**utils group**：
```gn
group("utils") {
  deps = []
  if (ohos_kernel_type == "liteos_m") {
    deps += [ "file:file" ]
  }
}
```

**native_api ndk_lib**：
```gn
ndk_lib("native_api") {
  if (ohos_kernel_type != "liteos_m") {
    lib_extension = ".so"
  }
  deps = []
  head_files = [ "//commonlibrary/utils_lite/include/utils_config.h" ]
  if (ohos_kernel_type == "liteos_m") {
    deps += [ "file:native_file" ]
    head_files += [ "//commonlibrary/utils_lite/include/utils_file.h" ]
  }
}
```

**utils_lite group**：
```gn
group("utils_lite") {
  deps = []

  if (utils_lite_feature_file) {
    deps += [ "//commonlibrary/utils_lite/file:file" ]
  }

  if (utils_lite_feature_kal_timer) {
    deps += [ "//commonlibrary/utils_lite/kal/timer:kal_timer" ]
  }

  if (utils_lite_feature_timer_task) {
    deps += [ "//commonlibrary/utils_lite/timer_task:ace_kit_timer" ]
  }

  if (utils_lite_feature_js_builtin) {
    deps += [ "//commonlibrary/utils_lite/js/builtin:ace_utils_kits" ]
  }
}
```

---

### 模块级 Targets

#### file/ 模块

| 名称 | 类型 | 输出 | 路径 | 行号 |
|------|------|------|------|------|
| `native_file` | static_library | .a | /file/BUILD.gn:16 | 16 |
| `file` | lite_component | - | /file/BUILD.gn:32 | 32 |

**file/BUILD.gn**：
```gn
static_library("native_file") {
  sources = [ "src/file_impl_hal/file.c" ]
  include_dirs = [ "//utils/file/include" ]
  if (ohos_kernel_type == "liteos_m") {
    deps = [ "//utils/dfs/fs/spiffs:hmos_spiffs" ]
  } else {
    deps = [ "//third_party/bounds_checking_function:libsec_shared" ]
  }
}

lite_component("file") {
  deps = [ ":native_file" ]
}
```

#### hals/file/ 模块

| 名称 | 类型 | 输出 | 路径 | 行号 |
|------|------|------|------|------|
| `static_hal_file` | ohos_static_library | .a | /hals/file/BUILD.gn:16 | 16 |

**hals/file/BUILD.gn**：
```gn
ohos_static_library("static_hal_file") {
  sources = [ "hal_file.c" ]
  include_dirs = [ "//commonlibrary/utils_lite/hals/file" ]
}
```

#### timer_task/ 模块

| 名称 | 类型 | 输出 | 路径 | 行号 |
|------|------|------|------|------|
| `ace_kit_timer` | lite_library | .a/.so | /timer_task/BUILD.gn:16 | 16 |

**timer_task/BUILD.gn**：
```gn
lite_library("ace_kit_timer") {
  if (ohos_kernel_type == "liteos_m") {
    target_type = "static_library"
    sources = [ "src/nativeapi_timer_task.c" ]
    include_dirs = [ "include" ]
    deps = [ "//commonlibrary/utils_lite/kal/timer:kal_timer" ]
    public_deps = [ "//commonlibrary/utils_lite/kal/timer:kal_timer" ]
  } else {
    target_type = "shared_library"
    # ...
  }
}
```

#### kal/timer/ 模块

| 名称 | 类型 | 输出 | 路径 | 行号 |
|------|------|------|------|------|
| `kal_timer` | lite_library | .a/.so | /kal/timer/BUILD.gn:16 | 16 |

**kal/timer/BUILD.gn**：
```gn
lite_library("kal_timer") {
  if (ohos_kernel_type == "liteos_m") {
    target_type = "static_library"
    sources = [ "src/kal.c" ]
    include_dirs = [ "include" ]
  } else {
    target_type = "shared_library"
    # ...
  }
}
```

#### js/builtin/ 模块

| 名称 | 类型 | 输出 | 路径 | 行号 |
|------|------|------|------|------|
| `ace_utils_kits` | lite_component | - | /js/builtin/BUILD.gn:18 | 18 |
| `ace_kit_common` | lite_library | .a/.so | /js/builtin/common/BUILD.gn:18 | 18 |
| `ace_kit_file` | lite_library | .a/.so | /js/builtin/filekit/BUILD.gn:19 | 19 |
| `ace_kit_kvstore` | lite_library | .a/.so | /js/builtin/kvstorekit/BUILD.gn:19 | 19 |
| `ace_kit_deviceinfo` | lite_library | .a/.so | /js/builtin/deviceinfokit/BUILD.gn:18 | 18 |

**js/builtin/BUILD.gn**：
```gn
lite_component("ace_utils_kits") {
  deps = [
    ":ace_kit_common",
    ":ace_kit_file",
    ":ace_kit_kvstore",
    ":ace_kit_deviceinfo",
  ]
}
```

#### simulator/ 模块

| 名称 | 类型 | 输出 | 路径 | 行号 |
|------|------|------|------|------|
| `ace_kit_common_simulator` | ohos_static_library | .a | /js/builtin/simulator/BUILD.gn:20 | 20 |
| `ace_kit_deviceinfo_simulator` | ohos_static_library | .a | /js/builtin/simulator/BUILD.gn:37 | 37 |
| `ace_kit_file_simulator` | ohos_static_library | .a | /js/builtin/simulator/BUILD.gn:79 | 79 |
| `ace_kit_kvstore_simulator` | ohos_static_library | .a | /js/builtin/simulator/BUILD.gn:104 | 104 |

---

## 依赖关系

### 编译时依赖

```
timer_task:ace_kit_timer
    └── public_deps: kal/timer:kal_timer

js/builtin/filekit:ace_kit_file
    └── external_deps: bounds_checking_function:libsec_shared

js/builtin/kvstorekit:ace_kit_kvstore
    └── external_deps: bounds_checking_function:libsec_shared

js/builtin/deviceinfokit:ace_kit_deviceinfo
    └── external_deps: init:libbegetutil
```

### 条件依赖

```gn
# file/BUILD.gn
if (ohos_kernel_type == "liteos_m") {
  deps = [ "//utils/dfs/fs/spiffs:hmos_spiffs" ]
} else {
  deps = [ "//third_party/bounds_checking_function:libsec_shared" ]
}
```

---

## 关键配置

### 平台条件编译

```gn
if (ohos_kernel_type == "liteos_m") {
  target_type = "static_library"
} else {
  target_type = "shared_library"
}
```

### SDK 模拟器条件

```gn
# js/builtin/simulator/BUILD.gn
if (build_ohos_sdk) {
  # 仅构建 SDK 时包含模拟器库
}
```

### 编译器配置

```gn
# js/builtin/simulator/BUILD.gn
config("storage_config") {
  cflags = [
    "-D_INC_STDIO_S",
    "-D_INC_STDLIB_S",
    "-D_INC_MEMORY_S",
    "-D_INC_STRING_S",
    "-D_INC_WCHAR_S",
    "-D_SECTMP=//",
    "-D_STDIO_S_DEFINED",
    "-Wno-error",
  ]
}
```

### 可见性控制

```gn
# js/builtin/simulator/BUILD.gn
visibility = [
  ":*",
  "//ide/tools/previewer/mock/*",
]
```

---

## 子系统标识

所有 `ohos_*` 目标标记：

```gn
subsystem_name = "commonlibrary"
part_name = "utils_lite"
```

**证据来源**：`bundle.json:16-18`

---

## 相关跳转

- [概述](00_Overview.md) - 项目定位
- [目录结构](01_Directory_Structure.md) - 模块布局
- [编译产物](06_Build_Artifacts.md) - 产物清单
- [N-API 参考](03_NAPI_Reference.md) - JS 接口
- [故障排查](08_Troubleshooting.md) - 构建问题
