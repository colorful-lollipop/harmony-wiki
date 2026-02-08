# sensors_miscdevice_lite 构建配置

## ⚠️ 重要声明

**本文档基于元数据和标准 OpenHarmony 构建模式推断**。

当前仓库**不包含源代码**，因此无实际 BUILD.gn 配置。实际构建配置请参考：
- **主实现仓库**: https://github.com/openharmony/sensors_miscdevice

---

## 4.1 构建系统概述

### 4.1.1 构建工具链

OpenHarmony 使用 **GN (Generate Ninja)** 作为构建系统：

| 工具 | 用途 |
|------|------|
| `gn` | 生成构建文件 |
| `ninja` | 执行构建 |
| `hb` (harmony build) | OpenHarmony 统一构建命令行 |

### 4.1.2 组件清单文件

**证据来源**: `bundle.json` (曾存在于 commit fed7b6f)

```json
{
    "name": "@ohos/sensors_miscdevice_lite",
    "component": {
        "name": "miscdevice_lite",
        "subsystem": "sensors",
        "syscap": ["SystemCapability.Sensors.MiscDevice_Lite"],
        "adapted_system_type": ["small"],
        "features": [],
        "build": {
            "sub_component": [],
            "inner_kits": [],
            "test": []
        }
    }
}
```

---

## 4.2 推断的构建结构

### 4.2.1 标准组件结构

基于 OpenHarmony 标准组件模式，推断的构建结构如下：

```
base/sensors/miscdevice_lite/
├── BUILD.gn                    # 根构建入口
├── interfaces/
│   ├── BUILD.gn
│   ├── native/
│   │   ├── BUILD.gn
│   │   └── *.cpp/.h
│   └── plugin/
│       ├── BUILD.gn
│       └── napi_*.cpp
├── frameworks/
│   └── native/
│       ├── BUILD.gn
│       └── *.cpp
├── services/
│   └── miscdevice_service/
│       ├── BUILD.gn
│       └── *.cpp
├── sa_profile/
│   └── BUILD.gn
│   └── *.xml
└── utils/
    ├── BUILD.gn
    └── *.cpp/.h
```

---

## 4.3 GN 构建配置示例

### 4.3.1 根 BUILD.gn（推断）

```gn
# Copyright (c) 2021 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import("//build/subsystem.gni")

sensors_miscdevice_lite_subsystem {
  deps = [
    "//base/sensors/miscdevice_lite/interfaces/native:miscddevice_native",
    "//base/sensors/miscdevice_lite/frameworks/native:miscdevice_framework",
    "//base/sensors/miscdevice_lite/services:miscdevice_service",
  ]
}
```

### 4.3.2 interfaces/plugin BUILD.gn（推断）

```gn
# N-API 插件构建配置
import("//build/napi.gni")

napi_module("miscdevice_napi") {
  sources = [
    "napi_vibrator.cpp",
    "napi_led.cpp",
  ]

  deps = [
    "//base/sensors/miscdevice_lite/frameworks/native:miscdevice_framework",
    "//foundation/ability/ability_runtime/interfaces/inner_api:ability_runtime_core",
    "//foundation/arkui/napi:ace_napi",
  ]

  external_deps = [
    "hdf_core:framework",
    "sensors:sensor_interface",
  ]

  cflags = [
    "-DWEBVIEW_SUPPORT_XCOMMANDSTATUS",
    "-DOHOS_LITE",
  ]
}
```

### 4.3.3 services BUILD.gn（推断）

```gn
# MiscDevice Service 构建配置
ohos_shared_library("miscdevice_service") {
  sources = [
    "miscdevice_service.cpp",
    "vibrator_manager.cpp",
    "led_manager.cpp",
  ]

  deps = [
    "//base/sensors/miscdevice_lite/utils:utils",
    "//foundation/systemabilitymgr/samgr/interfaces/inner_api:samgr_client",
    "//drivers/framework/core/common:dhwdfs",
  ]

  external_deps = [
    "hdf_core:framework",
    "sensors:sensor_interface",
  ]

  inner_kits = [
    "//base/sensors/miscdevice_lite/interfaces/native:miscddevice_inner_kit",
  ]
}
```

---

## 4.4 构建产物

### 4.4.1 预期产物

| 产物类型 | 产物路径（推断） | 说明 |
|----------|------------------|------|
| **N-API 模块** | `out/.../libs/libmiscdevice_napi.z.so` | JS 绑定库 |
| **Service 库** | `out/.../libs/libmiscdevice_service.z.so` | 服务实现 |
| **Framework 库** | `out/.../libs/libmiscdevice_framework.z.so` | 框架库 |
| **Utils 库** | `out/.../libs/libmiscdevice_utils.z.so` | 工具库 |

### 4.4.2 安装路径（推断）

| 产物 | 目标路径 |
|------|----------|
| N-API 模块 | `/system/lib/module/@ohos/miscdevice_lite.z.so` |
| Service 库 | `/system/lib/libmiscdevice_service.z.so` |
| HAP 依赖 | `/system/etc/abilities/.../miscdevice_service` |

---

## 4.5 构建命令

### 4.5.1 全量构建

```bash
# 使用 hb 工具
hb set
hb build -f

# 或使用 gn + ninja
gn gen out/sensors_miscdevice_lite
ninja -C out/sensors_miscdevice_lite
```

### 4.5.2 增量构建

```bash
# 增量构建 miscdevice_lite
ninja -C out/sensors_miscdevice_lite //base/sensors/miscdevice_lite/...
```

### 4.5.3 单独构建模块

```bash
# 构建 N-API 模块
ninja -C out/sensors_miscdevice_lite //base/sensors/miscdevice_lite/interfaces/plugin:miscdevice_napi

# 构建 Service
ninja -C out/sensors_miscdevice_lite //base/sensors/miscdevice_lite/services:miscdevice_service
```

---

## 4.6 构建配置选项

### 4.6.1 编译宏（推断）

| 宏定义 | 用途 | 默认值 |
|--------|------|--------|
| `OHOS_LITE` | 标识 Lite 系统 | 未定义 |
| `WEARABLE_PRODUCT` | 可穿戴产品 | 未定义 |
| `IOT_PRODUCT` | IoT 产品 | 未定义 |

### 4.6.2 特性开关（推断）

```gn
# 可选特性配置
enable_feature_vibrator = true
enable_feature_led = true
enable_feature_haptic_feedback = true
```

---

## 4.7 依赖配置

### 4.7.1 内部依赖

| 依赖项 | 类型 | 用途 |
|--------|------|------|
| `interfaces/native` | inner_kit | Native API 接口 |
| `frameworks/native` | deps | 框架实现 |
| `utils` | deps | 公共工具 |

### 4.7.2 外部依赖

| 依赖项 | 来源 | 用途 |
|--------|------|------|
| `hdf_core` | 系统组件 | 驱动框架 |
| `sensors` | 同子系统 | 传感器接口 |
| `ability_runtime` | 系统组件 | 运行时支持 |
| `ace_napi` | 系统组件 | N-API 框架 |

---

## 4.8 构建验证

### 4.8.1 验证步骤

```bash
# 1. 检查构建产物
ls -la out/.../libs/*.so | grep miscdevice

# 2. 检查模块加载
hdc shell "ls /system/lib/module/"

# 3. 检查服务注册
hdc shell "bm dump -a" | grep miscdevice
```

### 4.8.2 构建产物完整性检查

| 检查项 | 预期结果 |
|--------|----------|
| N-API 模块存在 | `libmiscdevice_napi.z.so` |
| Service 库存在 | `libmiscdevice_service.z.so` |
| 符号导出 | `nm -D libmiscdevice_napi.z.so` |
| 依赖检查 | `ldd libmiscdevice_service.z.so` |

---

## 4.9 常见构建问题

### 4.9.1 问题排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| Ninja 找不到 | gn 未配置 | 配置 PATH 环境变量 |
| 依赖缺失 | 子模块未初始化 | `git submodule update --init` |
| 编译错误 | 语法问题 | 检查源文件语法 |
| 链接失败 | 依赖库路径 | 检查 external_deps |

### 4.9.2 调试命令

```bash
# 查看详细构建日志
ninja -C out/... -v > build.log 2>&1

# 检查依赖树
gn deps //base/sensors/miscdevice_lite/...

# 生成 compile_commands.json
gn gen out/... --export-compile-commands
```

---

## 4.10 相关文档

### 官方资源
- [OpenHarmony 构建系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/quick-start/编译构建.md)
- [GN 官方文档](https://gn.googlesource.com/gn/+/HEAD/README.md)
- [sensors_miscdevice GitHub](https://github.com/openharmony/sensors_miscdevice)

### 本地文档
- [02_Architecture.md](./02_Architecture.md) - 架构说明
- [03_API.md](./03_API.md) - API 文档
- [06_Troubleshooting.md](./06_Troubleshooting.md) - 常见问题
