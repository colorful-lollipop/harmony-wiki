# 构建配置

> GN 构建系统配置与编译产物说明

## 构建概述

**构建系统**: GN (Generate Ninja)  
**构建工具**: Ninja  
**目标平台**: Hi3861 (LiteOS-M)  
**模块类型**: NDK (Native Development Kit) 模块

## 根构建文件

**文件**: `BUILD.gn`  
**位置**: `/base/iothardware/peripheral/BUILD.gn`

**证据**:
```gn
# Copyright (c) 2020-2021 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");

import("//build/lite/ndk/ndk.gni")

group("iothardware") {
  deps = [
    "$ohos_board_adapter_dir/hals/iot_hardware/wifiiot_lite:hal_iothardware",
  ]
}

if (ohos_kernel_type == "liteos_m") {
  ndk_lib("iothardware_ndk") {
    deps = [
      "$ohos_board_adapter_dir/hals/iot_hardware/wifiiot_lite:hal_iothardware",
    ]
    head_files = [ "//base/iothardware/peripheral/interfaces/inner_api" ]
  }
}
```

## GN Targets 分析

### 1. group("iothardware")

| 属性 | 值 |
|------|-----|
| **类型** | group (聚合目标) |
| **目的** | 聚合所有 iothardware 相关依赖 |
| **条件** | 无条件 |

**依赖关系**:
```
iothardware
└── $ohos_board_adapter_dir/hals/iot_hardware/wifiiot_lite:hal_iothardware
```

### 2. ndk_lib("iothardware_ndk")

| 属性 | 值 |
|------|-----|
| **类型** | ndk_lib (NDK 库) |
| **目的** | 生成 Native Development Kit 供应用调用 |
| **条件** | `ohos_kernel_type == "liteos_m"` |

**依赖关系**:
```
iothardware_ndk
└── $ohos_board_adapter_dir/hals/iot_hardware/wifiiot_lite:hal_iothardware
```

**配置**:
| 配置项 | 值 |
|--------|-----|
| `head_files` | `//base/iothardware/peripheral/interfaces/inner_api` |
| `deps` | HAL 实现依赖 |

## 组件配置

**文件**: `bundle.json`

**证据**:
```json
{
    "name": "@ohos/iothardware_peripheral",
    "description": "Iot peripheral controller.",
    "version": "3.1",
    "license": "Apache License 2.0",
    "publishAs": "code-segment",
    "component": {
        "name": "peripheral",
        "subsystem": "iothardware",
        "adapted_system_type": [
            "mini"
        ],
        "build": {
            "sub_component": [
                "//base/iothardware/peripheral:iothardware"
            ],
            "inner_kits": [],
            "test": []
        }
    }
}
```

## 编译产物

### NDK 产物清单

| 产物类型 | 路径模式 | 说明 |
|----------|----------|------|
| **头文件** | `out/.../ndk/inner_api/` | 所有 .h 文件 |
| **静态库** | `out/.../libhal_iothardware.a` | HAL 实现库 |
| **动态库** | `out/.../libhal_iothardware.so` | HAL 共享库 |

### 安装路径

| 产物类型 | 目标安装路径 |
|----------|--------------|
| **头文件** | `//base/iothardware/peripheral/interfaces/inner_api/` |
| **NDK 产物** | `$OHOS_SDK/ndk/` |

### 产物依赖关系

```
用户应用
    │
    ├── #include "iot_gpio.h"
    ├── #include "iot_i2c.h"
    └── ...
    │
    ▼
编译时链接
    │
    ├── -liothardware_ndk (或源码集成)
    │
    ▼
运行时加载
    │
    ├── libhal_iothardware.so (动态库)
    └── (或静态链接入应用)
```

## 使用方式

### 1. GN 依赖方式

```gn
# 应用 BUILD.gn
deps += [
  "//base/iothardware/peripheral:iothardware_ndk"
]
```

### 2. 源码集成方式

```gn
# 集成完整源码
deps += [
  "//base/iothardware/peripheral:iothardware"
]
```

### 3. 头文件引用

```c
// 应用代码
#include "iot_gpio.h"
#include "iot_i2c.h"
#include "iot_uart.h"
// ... 其他头文件
```

## 构建条件

### 条件编译

| 条件 | 定义 | 说明 |
|------|------|------|
| `ohos_kernel_type == "liteos_m"` | 内核类型 | NDK 仅针对 LiteOS-M 构建 |

### 平台支持矩阵

| 目标平台 | 支持状态 | 说明 |
|----------|----------|------|
| **Hi3861 (LiteOS-M)** | ✅ 支持 | 主要目标平台 |
| Hi3516 (LiteOS-A) | ❌ 不支持 | NDK 未配置 |
| Hi3518 (Linux) | ❌ 不支持 | NDK 未配置 |

## 头文件清单

### 必需头文件

| 头文件 | 功能 | 必须包含 |
|--------|------|----------|
| `iot_errno.h` | 错误码定义 | ✅ 是 |
| `iot_gpio.h` | GPIO 接口 | 按需 |
| `iot_i2c.h` | I2C 接口 | 按需 |
| `iot_uart.h` | UART 接口 | 按需 |
| `iot_pwm.h` | PWM 接口 | 按需 |
| `iot_watchdog.h` | Watchdog 接口 | 按需 |
| `iot_flash.h` | Flash 接口 | 按需 |
| `reset.h` | 重置接口 | 按需 |
| `lowpower.h` | 低功耗接口 | 按需 |

## 构建命令

### 完整构建

```bash
# 设置环境
source build.sh

# 构建 NDK
hb build -f
```

### 增量构建

```bash
# 仅构建 iothardware 子系统
hb build //base/iothardware/peripheral
```

### NDK 产物验证

```bash
# 检查产物
ls -la out/.../ndk/inner_api/
```

## 相关文档

- [API 参考](01_API_Reference.md)
- [架构说明](03_Architecture.md)
- [安全评审](05_Security_Review.md)
