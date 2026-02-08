# 项目概述

> 了解 utils_lite 的项目定位、核心能力与运行环境。

## 项目定位

**utils_lite** 是 OpenHarmony 的基础组件库，为子系统。该仓库存储提供基础能力支撑轻量级的基础组件，供 OpenHarmony 子系统和上层应用使用。

**证据来源**：`README.md:10`

## 核心能力

utils_lite 提供以下核心能力：

| 模块 | 能力 | 平台支持 |
|------|------|----------|
| 文件操作 | 统一文件操作接口 | LiteOS Cortex-M (Hi3861) |
| 定时器 | 统一定时器操作接口 | LiteOS Cortex-M / Cortex-A |
| JavaScript API | 设备信息查询、数据存储 | LiteOS Cortex-A |
| 内存管理 | 内存池管理 | 全平台 |

**证据来源**：`README.md:12-48`

### 文件操作能力

提供跨平台的文件系统操作 API，支持：

- 文件打开、关闭、读取、写入
- 文件删除、复制、移动
- 文件大小查询、位置调整

**SPIFFS 平台限制**（LiteOS Cortex-M）：
- 不支持多级目录
- 文件名最大 32 字节（含结束符）
- 最多同时打开 32 个文件

**证据来源**：`include/utils_file.h:25-31`

### 定时器能力

提供统一风格的定时器操作接口：

- 单次定时器（One-shot）
- 周期性定时器（Repeating）
- 定时器创建、启动、修改、停止、删除
- 运行状态查询

**证据来源**：`kal/timer/include/kal.h:27-44`

### JavaScript API 能力

为 LiteOS Cortex-A 平台提供 JS 接口：

- **KV 存储**：get、set、delete、clear
- **文件操作**：move、copy、delete、list、get、readText、writeText、access、mkdir、rmdir
- **设备信息**：获取 40+ 项设备属性

**证据来源**：`js/builtin/` 各模块

## 运行环境

### 支持平台

| 平台 | 内核 | 说明 |
|------|------|------|
| LiteOS Cortex-M | liteos_m | Hi3861 平台，仅文件操作和定时器 |
| LiteOS Cortex-A | liteos_a | Hi3516/Hi3518 平台，支持 JS API |

**证据来源**：`bundle.json:25-28`

```
"adapted_system_type": [
    "mini",    # LiteOS Cortex-M
    "small"    # LiteOS Cortex-A
]
```

### 系统依赖

根据 `bundle.json:29-43`，utils_lite 依赖以下组件：

```
- ability_lite     # 能力框架
- init             # 系统初始化
- graphic_utils_lite # 图形工具
- resource_management_lite # 资源管理
- ui_lite          # UI 框架
- ace_engine_lite  # ACE 引擎
- bounds_checking_function # 边界检查
- musl             # C 标准库
```

## 关键概念

### JSI (JavaScript Interface)

utils_lite 使用 **JSI** 框架实现 JavaScript API，而非标准 N-API。JSI 是 OpenHarmony ACELite 框架的轻量级 JavaScript 绑定机制。

**注册模式**：
```cpp
JSI::SetModuleAPI(exports, "methodName", HandlerFunc);
```

**证据来源**：`js/builtin/kvstorekit/src/nativeapi_kv.cpp:255-261`

### KAL (Kernel Abstraction Layer)

**内核抽象层**，封装底层内核定时器接口，提供统一的跨平台定时器实现。基于 POSIX 实时定时器实现。

**证据来源**：`kal/timer/include/kal.h`

### HAL (Hardware Abstraction Layer)

**硬件抽象层**，封装底层文件系统操作，屏蔽硬件差异。

**证据来源**：`hals/file/hal_file.h`

## 组件标识

| 标识 | 值 |
|------|------|
| 子系统 | `commonlibrary` |
| 组件名 | `utils_lite` |
| 包名 | `@ohos/utils_lite` |
| 版本 | `4.0.2` |

**证据来源**：`bundle.json:2-17`

## 相关跳转

- [目录结构](01_Directory_Structure.md) - 了解各模块布局
- [架构说明](02_Architecture.md) - 理解组件关系
- [N-API 参考](03_NAPI_Reference.md) - JS API 详情
- [内部 API](04_Inner_API.md) - C/C++ 接口
- [GN 构建](05_GN_Build.md) - 构建配置
