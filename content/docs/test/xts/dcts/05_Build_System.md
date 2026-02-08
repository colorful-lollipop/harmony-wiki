# 构建系统

## 概述

DCTS 使用 OpenHarmony 的 **GN (Generate Ninja)** 构建系统，通过 `BUILD.gn` 和 `.gni` 文件定义构建配置。

## 主构建入口

### 根目录 BUILD.gn

**文件**: `/Volumes/lexar/code/d/work/oh/test/xts/dcts/BUILD.gn`

```gn
import("//hit/build/suite.gni")
import("test_packages.gni")

# 开源声明合并
merge_xts_notice("dcts_opensource_process") {
  target = "dcts"
  deps = selected_packages
  if (make_osp) {
    deps += [ "//build/ohos/packages:open_source_package" ]
  }
}

# 主测试套件
ohos_test_suite("xts_dcts") {
  deps = [ ":dcts_opensource_process" ]
}

# 设备类型特定测试套件
ohos_test_suite("dcts_ivi") {
  deps = selected_packages_ivi
}

ohos_test_suite("dcts_intellitv") {
  deps = selected_packages_intellitv
}

ohos_test_suite("dcts_wearable") {
  deps = selected_packages_wearable
}
```

### 组件配置

**文件**: `bundle.json`

```json
{
  "name": "@ohos/dcts",
  "component": {
    "name": "dcts",
    "subsystem": "xts",
    "adapted_system_type": ["standard"],
    "build": {
      "test": [ "//test/xts/dcts:xts_dcts" ]
    }
  },
  "deps": {
    "components": [
      "access_token",
      "c_utils",
      "dsoftbus",
      "hilog",
      "ipc",
      "samgr",
      "wifi"
    ]
  }
}
```

## GN Targets 清单

### 根级 Targets

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `xts_dcts` | ohos_test_suite | 测试套件 | 主 DCTS 测试套件 |
| `dcts_ivi` | ohos_test_suite | 测试套件 | 车载 IVI 系统测试 |
| `dcts_intellitv` | ohos_test_suite | 测试套件 | 智能电视测试 |
| `dcts_wearable` | ohos_test_suite | 测试套件 | 可穿戴设备测试 |

### 模块级 Targets

#### ability 模块

| Target | 类型 | 说明 |
|--------|------|------|
| `DctsDmsHapTest` | ohos_test | 主测试 HAP |
| `DctsDmsFaTest` | ohos_test | FA 模式测试 |
| `DctsDmsJsServer` | ohos_test | JS 服务端测试 |
| `DctsDmsFwkStagePermissionTest` | ohos_test | Stage 权限测试 |
| `DctsDmsFwkStageServer` | ohos_test | Stage 模式服务端 |
| `DctsDmsFwkStageTest` | ohos_test | Stage 模式测试 |
| `DctsDmsFwkStageTestServer` | ohos_test | Stage 测试服务端 |

#### communication 模块

| Target | 类型 | 说明 |
|--------|------|------|
| `DctsRpcJsTest` | ohos_test | RPC JS 测试 |
| `DctsRpcJsServer` | ohos_test | RPC 服务端测试 |
| `DctsRpcRequestJsTest` | ohos_test | RPC 请求测试 |
| `DctsRpcRequestJsServer` | ohos_test | RPC 请求服务端 |
| `DctsRpcEtsTest` | ohos_test | RPC + ETS 测试 |
| `DctsRpcEtsServer` | ohos_test | RPC + ETS 服务端 |
| `Softbustestserver` | ohos_test | 标准软总线服务端 |
| `DctsSoftBusSoketTransFuncTest` | ohos_test | Socket 传输测试 |
| `DctsSoftBusTransFileFunTest` | ohos_test | 文件传输测试 |
| `DctsSoftBusTransFunTest` | ohos_test | 消息传输测试 |
| `DctsSoftBusTransStreamFunTest` | ohos_test | 流传输测试 |
| `DctsSoftBusTransSessionFunTest` | ohos_test | 会话管理测试 |
| `DctsSoftBusTransReliabilityTest` | ohos_test | 传输可靠性测试 |

#### distributedhardware 模块

| Target | 类型 | 说明 |
|--------|------|------|
| `DctsSubDeviceJsTest` | ohos_test | 设备管理器 JS 测试 |
| `DctsDeviceManagerTestStatic` | ohos_test | 静态设备管理测试 |
| `DctsDeviceManagerAPITestStatic` | ohos_test | 静态 API 测试 |
| `DctsSubdisDeviceJsTest` | ohos_test | 分布式设备 JS 测试 |
| `DctsSubdisDeviceJsTestserver` | ohos_test | 分布式设备测试服务 |
| `DctsSubAudioTest` | ohos_test | 分布式音频测试 |
| `DctsSubdisCameraTest` | ohos_test | 分布式相机测试 |
| `DctsSubDistributedInputTest` | ohos_test | 分布式输入测试 |
| `DctsSubdisScreenTest` | ohos_test | 分布式屏幕测试 |

#### filemanagement 模块

| Target | 类型 | 说明 |
|--------|------|------|
| `DctsFileioClientTest` | ohos_test | 文件 IO 客户端测试 |
| `DctsFileioServer` | ohos_test | 文件 IO 服务端测试 |

#### multimedia 模块

| Target | 类型 | 说明 |
|--------|------|------|
| `DctsAVSessionClientTest` | ohos_test | AV 会话客户端测试 |
| `DctsAVSessionServerTest` | ohos_test | AV 会话服务端测试 |

## 构建配置模式

### 标准测试模块 BUILD.gn 模式

```gn
import("//build/ohos_var.gni")

# 测试目标定义
ohos_test("ModuleName") {
  testonly = true
  
  # 源文件
  sources = [
    "src/**/*.cpp",
    "src/**/*.js",
    "src/**/*.ts",
  ]
  
  # 头文件目录
  include_dirs = [
    "src",
    "$_relative_path_to_framework",
  ]
  
  # 依赖
  deps = [
    "//foundation/ability/ability_runtime:ability",
    "//foundation/communication/ipc:ipc_core",
    "//third_party/cJSON:cjson",
  ]
  
  # 外部依赖
  external_deps = [
    "hilog:libhilog",
    "rpc:rpc_proxy",
  ]
  
  # 配置
  configs = []
  
  # C/C++ 标志
  cflags = []
  cxxflags = []
  
  # 条件编译
  if (is_standard_system) {
    # 标准系统特定配置
  }
}
```

### 测试套件 BUILD.gn 模式

```gn
import("//build/ohos_var.gni")

group("module_name") {
  testonly = true
  if (is_standard_system) {
    deps = [
      "submodule1:Target1",
      "submodule2:Target2",
    ]
  }
}
```

## 编译产物

### 产物类型

| 产物类型 | 说明 | 格式 |
|----------|------|------|
| **测试 HAP** | 可安装的测试应用包 | `.hap` |
| **可执行文件** | 命令行测试工具 | `.bin` |
| **静态库** | 链接到系统的测试代码 | `.a` |

### 输出路径

```
out/xts/dcts/
├── suites/
│   ├── dcts/
│   │   ├── DctsDmsHapTest.hap
│   │   ├── DctsRpcJsTest.hap
│   │   └── ...
│   ├── dcts_ivi/
│   ├── dcts_intellitv/
│   └── dcts_wearable/
└── libs/
    └── libdcts_*.a
```

## 构建命令

### 全量构建

```bash
# 使用 hb 工具构建
hb set -p <product_name>
hb build -f -t test

# 或直接使用 gn + ninja
gn gen out/xts/dcts --args="is_standard_system=true"
ninja -C out/xts/dcts xts_dcts
```

### 单模块构建

```bash
ninja -C out/xts/dcts DctsDmsHapTest
ninja -C out/xts/dcts DctsRpcJsTest
```

### 构建特定产品

```bash
# IVI 产品
hb set -p <ivi_product_name>
hb build -f -t test

# 智能电视
hb set -p <tv_product_name>
hb build -f -t test

# 可穿戴设备
hb set -p <wearable_product_name>
hb build -f -t test
```

## 构建配置变量

### 关键 GN 变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `is_standard_system` | 是否为标准系统 | `true` |
| `is_small_system` | 是否为小型系统 | `false` |
| `is_mini_system` | 是否为微型系统 | `false` |
| `board_name` | 开发板名称 | `depends on product` |

### 条件编译示例

```gn
ohos_test("ConditionalTest") {
  if (board_name == "liteos_m") {
    # 微型系统特定配置
    sources += [ "liteos_m_specific.c" ]
  } else if (board_name == "liteos_a") {
    # 小型系统特定配置
    sources += [ "liteos_a_specific.cpp" ]
  } else {
    # 标准系统特定配置
    sources += [ "standard_specific.cpp" ]
  }
}
```

## 依赖关系

### 模块依赖

```
ability  →  distributeddatamgr  →  filemanagement
    ↓              ↓                   ↓
communication  ←  distributedhardware  ←  multimedia
```

### 外部依赖

- **OpenHarmony 框架**: ability_runtime, distributedDatamgr 等
- **系统服务**: dsoftbus, ipc, samgr
- **工具库**: hilog (日志), c_utils (C 工具)
- **第三方库**: gtest (测试框架)

## 相关文档

- [项目概览](01_Overview.md)
- [目录结构](02_Directory_Structure.md)
- [架构设计](03_Architecture.md)
- [模块详解](04_Modules.md)
- [安全评审](06_Security_Review.md)
