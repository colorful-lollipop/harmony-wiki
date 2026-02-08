# 构建系统

## 概述

Previewer 使用 **GN (Generate Ninja)** 构建系统。

## 配置文件

| 文件 | 职责 |
|------|------|
| `BUILD.gn` | 根构建入口，定义预览器可执行文件 |
| `bundle.json` | 组件配置，定义子组件和内部 Kit |
| `gn/config.gni` | 平台配置和工具链定义 |
| `*/BUILD.gn` | 各模块构建定义 |

---

## 根构建目标

### previewer_executable 模板

**文件**: `BUILD.gn:17-57`

定义了一个 GN 模板，用于创建预览器可执行文件。

### rich_previewer

**文件**: `BUILD.gn:79-96`

```gn
previewer_executable("rich_previewer") {
  part_name = "previewer"
  output_name = "Previewer"
  src = [ "RichPreviewer.cpp" ]
  includes = os_include_dirs
  includes += [
    "./mock/rich/",
    "./jsapp/rich/",
  ]
  deps = [
    "cli:cli_rich",
    "jsapp:jsapp_rich",
    "mock:mock_rich",
    "util:util_rich",
    "//third_party/libwebsockets:websockets_static",
  ]
}
```

### lite_previewer

**文件**: `BUILD.gn:98-126`

```gn
previewer_executable("lite_previewer") {
  part_name = "previewer"
  output_name = "Simulator"
  src = [ "ThinPreviewer.cpp" ]
  includes = [
    "//foundation/arkui/frameworks/base/utils/",
    "./mock/lite/",
    "./jsapp/lite/",
    "//foundation/arkui/ui_lite/interfaces/innerkits/",
    "//foundation/graphic/graphic_utils_lite/interfaces/kits/",
    "//foundation/graphic/graphic_utils_lite/interfaces/innerkits/",
    "//foundation/arkui/ui_lite/frameworks/dock/",
  ]
  deps = [
    "cli:cli_lite",
    "jsapp:lite",
    "mock:mock_lite",
    "util:util_lite",
    "//foundation/arkui/ace_engine_lite/frameworks/targets/simulator:ace_lite",
    "//third_party/libwebsockets:websockets_static",
  ]
}
```

---

## 模块构建目标

### cli/BUILD.gn

| 目标 | 类型 | 描述 |
|------|------|------|
| `cli_lite` | ohos_source_set | Lite 命令行处理 |
| `cli_rich` | ohos_source_set | Rich 命令行处理 |
| `cli_config` | config | 编译配置 (NOGDI, C++17) |

### jsapp/BUILD.gn

| 目标 | 类型 | 描述 |
|------|------|------|
| `jsapp_rich` | ohos_source_set | Rich JS 应用 |
| `jsapp_lite` | ohos_source_set | Lite JS 应用 |
| `jsapp_config` | config | 编译配置 |

### mock/BUILD.gn

| 目标 | 类型 | 描述 |
|------|------|------|
| `mock_rich` | ohos_source_set | Rich 交互模拟 |
| `mock_lite` | ohos_source_set | Lite 交互模拟 |

### util/BUILD.gn

| 目标 | 类型 | 描述 |
|------|------|------|
| `util_lite` | ohos_source_set | Lite 工具库 |
| `util_rich` | ohos_source_set | Rich 工具库 |
| `ide_util` | ohos_shared_library | **内部 Kit** |

### jsapp/rich/external/BUILD.gn

| 目标 | 类型 | 描述 |
|------|------|------|
| `ide_extension` | ohos_shared_library | **内部 Kit** |

---

## 平台配置

**文件**: `gn/config.gni`

```gn
platform = "${current_os}_${current_cpu}"
if (platform == "mac_arm64") {
  mac_buildtool = "//build/toolchain/mac:clang_arm64"
} else if (platform == "mac_x64") {
  mac_buildtool = "//build/toolchain/mac:clang_x64"
}
windows_buildtool = "//build/toolchain/mingw:mingw_x86_64"
linux_buildtool = "//build/toolchain/linux:clang_${host_cpu}"
```

### 支持的平台

| 平台 | 构建工具 | CPU |
|------|----------|-----|
| `mac_arm64` | clang_arm64 | Apple Silicon |
| `mac_x64` | clang_x64 | Intel Mac |
| `mingw_x86_64` | mingw_x86_64 | Windows |
| `linux_x64` | clang_x64 | Linux x64 |
| `linux_arm64` | clang_arm64 | Linux ARM |

---

## 组件配置

**文件**: `bundle.json:40-80`

### 子组件列表

```json
"sub_component": [
  "//ide/tools/previewer/cli:cli_lite",
  "//ide/tools/previewer/cli:cli_rich",
  "//ide/tools/previewer/jsapp:jsapp_lite",
  "//ide/tools/previewer/jsapp:jsapp_rich",
  "//ide/tools/previewer/mock:mock_lite",
  "//ide/tools/previewer/mock:mock_rich",
  "//ide/tools/previewer/util:util_lite",
  "//ide/tools/previewer/util:util_rich",
  "//ide/tools/previewer:rich_previewer",
  "//ide/tools/previewer:lite_previewer",
  "//ide/tools/previewer/jsapp/rich/external:ide_extension"
]
```

### 内部 Kit

```json
"inner_kits": [
  {
    "type": "so",
    "name": "//ide/tools/previewer/util:ide_util",
    "header": {
      "header_files": ["KeyboardHelper.h", "ClipboardHelper.h"],
      "header_base": "//ide/tools/previewer/util"
    }
  },
  {
    "type": "so",
    "name": "//ide/tools/previewer/jsapp/rich/external:ide_extension",
    "header": {
      "header_files": ["EventRunner.h", "EventHandler.h", "StageContext.h", "JsMockUtil.h"],
      "header_base": "//ide/tools/previewer/jsapp/rich/external"
    }
  }
]
```

---

## 相关文档

- 编译产物: [06_Artifacts.md](./06_Artifacts.md)
- 目录结构: [01_Directory_Structure.md](./01_Directory_Structure.md)
