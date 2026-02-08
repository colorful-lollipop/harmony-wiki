# startup_cangjie_wrapper 项目概览

## 目的

本文档提供 startup_cangjie_wrapper（启动恢复仓颉封装）项目的整体概览，帮助新开发者快速理解项目定位、核心能力和基本架构。

## 适用范围

- 使用仓颉（Cangjie）语言开发 OpenHarmony 应用的开发者
- 参与设备信息查询服务开发的工程师
- 需要了解 OpenHarmony 设备信息获取机制的技术人员

---

## 项目简介

startup_cangjie_wrapper 是 OpenHarmony 系统中一个面向仓颉语言的设备信息查询服务封装组件。它为仓颉开发者提供了统一的 API 来获取设备的硬件信息、软件版本信息、标识信息等。

### 关键特性

- **设备信息查询**: 提供 30+ 个设备信息属性
- **仓颉原生支持**: 使用仓颉语言开发，为仓颉应用提供原生 API
- **系统信息获取**: 基于 init 组件的 SA 服务实现
- **权限控制**: 敏感信息（UDID、序列号）需要特殊权限
- **API Level 22**: 支持最新的 OpenHarmony API

---

## 核心能力

### 1. 设备类型信息

获取设备的基本类型、品牌、制造商、型号等信息。

**关键 API**:
- `deviceType` - 设备类型（phone/tablet/tv/wearable 等）
- `brand` - 设备品牌
- `manufacture` - 设备制造商
- `productModel` - 产品型号
- `hardwareModel` - 硬件型号
- `softwareModel` - 软件型号

### 2. 系统版本信息

获取 OpenHarmony 系统的版本信息。

**关键 API**:
- `osFullName` - OS 版本完整字符串
- `majorVersion` - 主版本号（M）
- `seniorVersion` - 次版本号（S）
- `featureVersion` - 特性版本号（F）
- `buildVersion` - 构建版本号（B）
- `sdkApiVersion` - SDK API 版本号
- `displayVersion` - 显示版本号
- `osReleaseType` - OS 发布类型（Release/Beta/Canary）

### 3. 构建信息

获取系统构建的相关信息。

**关键 API**:
- `buildTime` - 构建时间
- `buildUser` - 构建用户
- `buildHost` - 构建主机
- `buildType` - 构建类型
- `buildRootHash` - 版本 hash
- `incrementalVersion` - 增量版本
- `securityPatchTag` - 安全补丁级别

### 4. 设备标识信息

获取设备的唯一标识符（需要特殊权限）。

**关键 API**:
- `udid` - 设备 UDID（需要 `ohos.permission.sec.ACCESS_UDID`）
- `serial` - 设备序列号（需要 `ohos.permission.sec.ACCESS_UDID`）
- `ODID` - 开发者级非永久设备标识

### 5. Distribution OS 信息

获取独立软件供应商（ISV）分发的 OS 信息。

**关键 API**:
- `distributionOSName` - 分发 OS 名称
- `distributionOSVersion` - 分发 OS 版本
- `distributionOSApiVersion` - 分发 OS API 版本
- `distributionOSApiName` - 分发 OS API 名称
- `distributionOSReleaseType` - 分发 OS 发布类型

---

## 技术栈

### 编程语言
- **仓颉（Cangjie）**: 主要实现语言
- **C/C++**: init 组件的 FFI 层（外部依赖）

### 构建系统
- **GN**: 构建配置工具
- **CJC**: 仓颉编译器

### 依赖组件
- **init 组件**: 提供设备信息 SA 服务（`cj_device_info_ffi`）
- **cangjie_ark_interop**: 提供注解和异常类（`ohos.labels`）

---

## 运行环境

### 支持的系统
- **OpenHarmony standard**（标准系统）
- 仅支持标准设备，不支持轻量级设备

### 系统要求
- OpenHarmony API Level 22+
- SystemCapability.Startup.SystemInfo

### 资源占用
- ROM: 150KB
- RAM: 116KB

---

## 项目特点

### 与 ArkTS 的差异

与 ArkTS 版本的设备信息 API 相比，仓颉版本暂不支持以下功能：

| 功能 | ArkTS | 仓颉 | 说明 |
|------|-------|------|------|
| 认证型号别名 | ✓ | ✗ | `productModelAlias` |
| 硬盘序列号 | ✓ | ✗ | `diskSN` |
| 设备能力等级 | ✓ | ✗ | `performanceClass` |

**证据**: `README_zh.md:51-55`

### 模拟实现支持

项目提供了一套模拟实现（`mock/ohos.device_info.cj`），用于：
- Windows/Mac 开发环境
- 编译测试
- 不依赖真实设备信息

---

## 快速开始

### 基本使用

```cangjie
package main

import ohos.device_info

func main() {
    // 获取设备类型
    let deviceType = DeviceInfo.deviceType
    println("Device Type: ${deviceType}")

    // 获取制造商
    let manufacture = DeviceInfo.manufacture
    println("Manufacture: ${manufacture}")

    // 获取 OS 版本
    let osFullName = DeviceInfo.osFullName
    println("OS Full Name: ${osFullName}")

    // 获取 API 版本
    let sdkApiVersion = DeviceInfo.sdkApiVersion
    println("SDK API Version: ${sdkApiVersion}")
}
```

### 权限声明

如果需要访问 UDID 或序列号，需要在 module.json5 中声明权限：

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.sec.ACCESS_UDID",
        "reason": "需要获取设备唯一标识",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      }
    ]
  }
}
```

**注意**: `ohos.permission.sec.ACCESS_UDID` 权限只能由系统应用和企业定制应用申请。

---

## 项目架构概览

```
┌─────────────────────────────────────────┐
│          仓颉应用（用户代码）            │
└──────────────┬──────────────────────────┘
               │
               ↓ ohos.device_info API
┌─────────────────────────────────────────┐
│    DeviceInfo 静态属性（同步调用）        │
│  - deviceType, manufacture, brand...    │
└──────────────┬──────────────────────────┘
               │
               ↓ unsafe FFI 调用
┌─────────────────────────────────────────┐
│    FFI 层（34 个 foreign func）        │
│  - FfiOHOSDeviceInfoDeviceType()       │
│  - FfiOHOSDeviceInfoManufacture()       │
│  ...                                   │
└──────────────┬──────────────────────────┘
               │
               ↓ init 组件 SA 服务
┌─────────────────────────────────────────┐
│    init 组件设备信息 SA 服务             │
│  (cj_device_info_ffi)                  │
└─────────────────────────────────────────┘
```

**证据**:
- 架构图: `figures/startup_cangjie_wrapper_architecture_zh.png`
- API 实现: `ohos/device_info/device_info.cj:103-709`
- FFI 声明: `ohos/device_info/device_info.cj:22-94`

---

## 关键结论

1. **纯仓颉实现**: 所有公开 API 使用仓颉语言实现
2. **同步调用**: 所有 API 均为同步属性访问，无异步模式
3. **无状态设计**: DeviceInfo 类为静态类，无实例化
4. **FFI 桥接**: 通过 FFI 调用 init 组件的 SA 服务
5. **权限控制**: 敏感信息需要 `ohos.permission.sec.ACCESS_UDID` 权限
6. **API Level 22**: 所有 API 标注为 API Level 22
7. **Beta 特性**: 当前版本为 beta 特性

---

## 相关跳转链接

- [01_Project_Positioning.md](01_Project_Positioning.md) - 项目定位和边界
- [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构详解
- [03_Architecture.md](03_Architecture.md) - 详细架构说明
- [04_Cangjie_API.md](04_Cangjie_API.md) - API 清单和调用链
- [仓颉 API 官方文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_zh_cn/apis/BasicServicesKit/cj-apis-device_info.md)

---

## 参考资料

- [OpenHarmony 官方文档](https://docs.openharmony.cn)
- [仓颉语言规范](https://developer.openharmony.cn/cn/doc/cangjie-quickstart)
- [OpenHarmony 设备信息 API 规范](https://docs.openharmony.cn/application-dev/reference/apis/js-apis-device-info.md)

---

*最后更新: 2026-02-06*
