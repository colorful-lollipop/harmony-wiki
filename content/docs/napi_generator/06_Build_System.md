# 构建系统

> GN Targets、编译产物与依赖关系

## 构建系统概览

### 1. 主构建配置

| 用途 | 文件路径 | 说明 |
|------|----------|------|
| **工具构建** | `src/cli/*/package.json` | npm 包配置 |
| **OpenHarmony 构建** | `src/cli/*/BUILD.gn` | GN 构建脚本 |
| **示例构建** | `examples/*/BUILD.gn` | 示例项目构建 |

### 2. GN 构建文件位置

**证据**: 探索发现当前仓库主要使用 npm 作为工具构建系统，GN 构建文件较少。

```
src/cli/
├── cmake2gn/          # TODO: BUILD.gn (待验证)
├── dts2cpp/           # TODO: BUILD.gn (待验证)
├── dts2ets/           # TODO: BUILD.gn (待验证)
├── h2dts/             # TODO: BUILD.gn (待验证)
├── h2dtscpp/          # TODO: BUILD.gn (待验证)
├── h2hdf/             # TODO: BUILD.gn (待验证)
└── h2sa/              # TODO: BUILD.gn (待验证)
```

### 3. 构建产物生成

**dts2cpp** 工具会根据输入生成 BUILD.gn 文件：

```
输出目录/
├── BUILD.gn                  # 自动生成
├── {module}_middle.h
├── {module}_middle.cpp
├── {module}.h
├── {module}.cpp
└── tool_utility.h/cpp
```

---

## dts2cpp 生成的 BUILD.gn

### 1. 模板结构

```gn
# {module}_middle.h 的 BUILD.gn 模板
# 证据: src/cli/dts2cpp/src/gen/extend/build_gn.js

import("//build/ohos.gni")  # OpenHarmony 构建导入

# 静态库: 工具辅助代码
static_library("tool_utility") {
  sources = [
    "tool_utility.cpp",
  ]
  
  include_dirs = [
    "//third_party/node/src/libplatform",
    "//third_party/node/src",
    "//foundation/ability/ability_runtime/interfaces/inner_api/native",
    "//foundation/graphic/graphic_utils/interfaces/native",
  ]
  
  deps = []
  public_deps = []
  configs = []
  defines = []
}

# 共享库: N-API 模块
shared_library("entry") {
  sources = [
    "{module}_middle.cpp",
    "{module}.cpp",
  ]
  
  include_dirs = [
    "//third_party/node/src/libplatform",
    "//third_party/node/src",
    "//foundation/ability/ability_runtime/interfaces/inner_api/native",
    "//foundation/graphic/graphic_utils/interfaces/native",
  ]
  
  deps = [
    ":tool_utility",
  ]
  
  public_deps = [
    "//foundation/ability/ability_runtime/interfaces/inner_api/native:native_interface",
  ]
  
  # N-API 模块配置
  config = {
    defines = [ "NAPI_EXPORT" ]
  }
}
```

### 2. 依赖关系

```
entry (shared_library)
    │
    ├── deps:
    │   └── :tool_utility (static_library)
    │       │
    │       └── include_dirs:
    │           ├── third_party/node/src/libplatform
    │           ├── third_party/node/src
    │           └── foundation/ability/ability_runtime/interfaces/inner_api/native
    │
    └── public_deps:
        └── foundation/ability/ability_runtime/interfaces/inner_api/native:native_interface
```

### 3. 关键配置项

| 配置项 | 类型 | 说明 |
|--------|------|------|
| `sources` | list | 源文件列表 |
| `include_dirs` | list | 头文件搜索路径 |
| `deps` | list | 内部依赖 |
| `public_deps` | list | 公共依赖 (可传递) |
| `configs` | list | 构建配置 |
| `defines` | list | 预处理器宏 |

---

## h2sa 生成的 BUILD.gn

### 1. SA 配置文件结构

```
{serviceName}service/
├── sa_profile/
│   ├── BUILD.gn
│   └── {serviceId}.xml
├── BUILD.gn
└── bundle.json
```

### 2. sa_profile/BUILD.gn

```gn
# SA 组件构建配置
# 证据: src/cli/h2sa/src/gen/file_template.js

ohos_component("myservice_sa") {
  component_type = "service"
  
  sources = [
    "../../src/myservice_service.cpp",
    "../../src/myservice_service_proxy.cpp",
    "../../src/myservice_service_stub.cpp",
    "../../src/i_myservice_service.cpp",
  ]
  
  header_deps = [
    "//interfaces/inner_api/native",
    "//foundation/systemabilityMgr/safwk/interfaces/inner_api/safwk",
    "//foundation/systemabilityMgr/samgr/interfaces/inner_api/samgr",
  ]
  
  install_images = [ "//system" ]
  
  subsystem_name = "mysubsystem"
  component_name = "myservice_sa"
}
```

### 3. SA 配置 XML

```xml
<!-- {serviceId}.xml -->
<info>
    <!-- 进程名 -->
    <process> Myservice.sa</process>
    <!-- SA 名称 -->
    <name> myservice</name>
    <!-- SA ID -->
    <libpath> libmyservice.z.so</libpath>
    <!-- 启动模式: true=开机自启, false=按需启动 -->
    <run-on-create> true</run-on-create>
    <distributed> false</distributed>
    <dump_level> 1</dump_level>
</info>
```

### 4. 服务启动配置

```json
// etc/myservice_service.cfg
{
    "services": [{
        "name": "myservice",
        "path": ["/system/bin/sa_main", "12"],
        "permission": {
            "permission": [
                "ohos.permission.XXX"
            ]
        }
    }]
}
```

---

## h2dtscpp 生成的构建配置

### 1. CMakeLists.txt (Linux 测试)

```cmake
# CMakeLists.txt (由 h2dtscpp 生成)
cmake_minimum_required(VERSION 3.10)
project(napi_test)

set(CMAKE_CXX_STANDARD 17)

add_library(entry SHARED
    napi_init.cpp
    test1.cpp
    test2.cpp
)

# 链接 OpenHarmony N-API
target_include_directories(entry PRIVATE
    ${OHOS_NAPI_INCLUDE_DIR}
)

target_link_libraries(entry PRIVATE
    ${OHOS_NAPI_LIBRARY}
)
```

---

## 编译产物路径

### 1. 工具编译产物

| 工具 | 产物 | 路径 |
|------|------|------|
| dts2cpp | Node.js CLI | `src/cli/dts2cpp/src/gen/*.js` |
| h2sa | Node.js CLI | `src/cli/h2sa/src/gen/*.js` |
| h2dtscpp | Node.js CLI | `src/cli/h2dtscpp/src/src/*.js` |

### 2. 生成代码编译产物

| 产物类型 | 路径 | 说明 |
|----------|------|------|
| **静态库** | `out/libs/libtool_utility.a` | 工具辅助代码 |
| **共享库** | `out/default/libs/libentry.so` | N-API 模块 |
| **SA 库** | `out/system/lib/lib{myservice}.z.so` | 系统服务 |

### 3. 运行时加载关系

```
JS/ArkTS 应用
    │
    │ import from 'libentry.so'
    │ (dlopen 加载)
    ▼
┌─────────────────────────────────────────┐
│  libentry.so (N-API 模块)               │
│  - 符号: NAPI_MODULE                    │
│  - 入口: init()                         │
└─────────────────────────────────────────┘
    │
    │ 链接
    ▼
┌─────────────────────────────────────────┐
│  libtool_utility.a (静态链接)           │
│  - XNapiTool 工具类                     │
│  - 类型转换辅助函数                      │
└─────────────────────────────────────────┘
    │
    │ N-API 调用
    ▼
┌─────────────────────────────────────────┐
│  libnapi.so (OpenHarmony 运行时)        │
│  - napi_create_*                        │
│  - napi_get_value_*                     │
│  - napi_call_function                   │
└─────────────────────────────────────────┘
```

---

## 依赖关系详解

### 1. 外部依赖

| 依赖项 | 来源 | 用途 |
|--------|------|------|
| **node** | `//third_party/node` | N-API 运行时 |
| **arkui_napi** | `//foundation/ability/ability_runtime` | N-API 头文件 |
| **safwk** | `//foundation/systemabilityMgr/safwk` | SA Framework |
| **samgr** | `//foundation/systemabilityMgr/samgr` | SA Manager |
| **hdf** | `//drivers/hdf_core` | 硬件驱动框架 |

### 2. 内部依赖

```
src/cli/*/
    │
    ├── tools/          # 公共工具
    │   ├── common.js
    │   ├── re.js
    │   └── NapiLog.js
    │
    ├── gen/           # 生成器
    │   ├── main.js
    │   ├── analyze.js
    │   └── generate.js
    │
    └── docs/          # 文档
```

---

## 构建命令

### 1. 构建 N-API 模块

```bash
# 使用 hb (HarmonyOS Build)
hb set
hb build

# 或使用 gn + ninja
gn gen out/default
ninja -C out/default entry
```

### 2. 构建 SA 服务

```bash
# 构建 SA 组件
cd sa_profile
hb build myservice_sa

# 安装到系统镜像
hdc shell
mount -o rw,remount /
cp libmyservice.z.so /system/lib/
```

---

## 相关章节

- 目录结构: [02_Directory_Structure.md](02_Directory_Structure.md)
- 架构说明: [03_Architecture.md](03_Architecture.md)
- API 参考: [04_NAPI_Reference.md](04_NAPI_Reference.md)

---

[返回 SUMMARY.md](SUMMARY.md)
