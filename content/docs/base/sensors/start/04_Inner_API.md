# 内部 API

**适用范围**: 本文档适用于所有需要了解内部 API 和模块接口的人员
**目的**: 说明内部 API、模块接口、依赖方向、稳定性
**关键结论**: **本组件不提供任何内部 API（无源代码）**

---

## 内部 API 概述

### 组件特点

⚠️ **重要提示**: `sensors_start` 是一个**纯配置型组件**

**不包含**:
- ❌ 无 C/C++ 源代码
- ❌ 无头文件 (`.h`/`.hpp`)
- ❌ 无内部函数定义
- ❌ 无模块接口

**职责**:
- ✅ 仅提供 INIT 系统启动配置文件
- ✅ 服务实现由外部组件提供（`sensors_sensor`、`sensors_miscdevice`、`msdp`）

### 内部 API 证据

通过仓库文件列表可以确认:

| 内部 API 相关元素 | 搜索结果 | 证据 |
|-----------------|----------|------|
| `.cpp` 源代码 | 未找到 | 仓库无任何 C++ 源代码 |
| `.c` 源代码 | 未找到 | 仓库无任何 C 源代码 |
| `.h`/`.hpp` 头文件 | 未找到 | 仓库无任何头文件 |
| 内部函数定义 | 未找到 | 仓库无源代码 |
| 类定义 | 未找到 | 仓库无源代码 |

---

## 与外部组件的接口

### 组件依赖关系

```
sensors_start (本仓库)
    ↓ 提供
    ├─ /etc/init/sensors.rc
    └─ /etc/init/msdp.rc
    ↓ 使用
    ├─ /system/bin/init (INIT 进程)
    ├─ /system/bin/sa_main (SA 进程管理器)
    ├─ /system/profile/sensors.json (SA 配置，外部)
    └─ /system/profile/msdp.json (SA 配置，外部)
```

### 接口类型

#### 1. 配置文件接口 (输出)

**输出**: INIT 系统启动配置文件

| 文件 | 接收方 | 格式 | 说明 |
|------|--------|------|------|
| /etc/init/sensors.rc | INIT 进程 | JSON | sensors 服务启动配置 |
| /etc/init/msdp.rc | INIT 进程 | JSON | msdp 服务启动配置 |

**证据**: [06_Build_Artifacts.md](./06_Build_Artifacts.md)

#### 2. 进程启动接口 (输入)

**输入**: INIT 系统提供的进程启动能力

| 依赖 | 用途 | 说明 |
|------|------|------|
| /system/bin/init | 读取配置并启动服务 | INIT 系统进程 |
| /system/bin/sa_main | 启动 SA 进程 | SA 进程管理器 |

**证据**: [02_Architecture.md](./02_Architecture.md)

#### 3. SA 配置文件接口 (引用)

**引用**: SA 配置文件（由外部组件生成）

| 文件 | 生成方 | 说明 |
|------|--------|------|
| /system/profile/sensors.json | sensors_sensor 组件 | SA 3601/3602 配置 |
| /system/profile/msdp.json | msdp 组件 | msdp SA 配置 |

**说明**: 这些文件不在本仓库，由外部组件生成并安装

---

## 依赖方向

### 构建时依赖

```
sensors_start (本仓库)
    ├─ 依赖 GN 构建系统 (//build/ohos.gni)
    ├─ 依赖 ohos_prebuilt_etc GN 模板
    └─ 无其他构建时依赖
```

### 运行时依赖

```
sensors_start (配置文件)
    ├─ 被 INIT 进程使用
    │   └─ 启动 sensors 进程
    │       └─ 依赖 sensors_sensor / sensors_miscdevice
    │
    └─ 启动 msdp 进程
        └─ 依赖 msdp 组件
```

**依赖方向**: sensors_start → INIT → 服务实现（外部组件）

### 反向依赖

**本组件不直接被其他组件依赖**

`sensors_sensor` 和 `sensors_miscdevice` 间接依赖本组件:
- 两个组件共享 `sensors` 进程
- 需要本组件提供的统一启动配置
- 避免重复启动进程

**证据**: [README.md:24](../README.md:24)

---

## 接口稳定性

### 配置文件接口稳定性

| 配置文件 | 稳定性 | 兼容性 | 说明 |
|----------|--------|--------|------|
| /etc/init/sensors.rc | 稳定 | 向后兼容 | INIT 系统标准格式 |
| /etc/init/msdp.rc | 稳定 | 向后兼容 | INIT 系统标准格式 |

**说明**: INIT 配置文件格式遵循 OpenHarmony 标准，稳定性较高

### SA 配置文件引用稳定性

| 配置文件 | 稳定性 | 维护方 | 说明 |
|----------|--------|--------|------|
| /system/profile/sensors.json | 由外部组件维护 | sensors_sensor | 格式可能变化 |
| /system/profile/msdp.json | 由外部组件维护 | msdp 组件 | 格式可能变化 |

**说明**: SA 配置文件格式由外部组件决定，可能变化

---

## 可替换点

### 可替换的组件

**sensors_start** 本身不可替换（它是 sensors 子系统的一部分），但其功能可以被其他配置实现替代。

### 配置文件可替换性

| 配置文件 | 可替换性 | 替换方式 | 风险 |
|----------|----------|----------|------|
| sensors.cfg | 可替换 | 修改配置文件 | 低（需确保服务正常启动） |
| msdp.cfg | 可替换 | 修改配置文件 | 低（需确保服务正常启动） |

**注意**: 修改配置文件需要重新编译系统镜像

### GN Target 可替换性

**不可替换**: GN targets (`sensors.rc`, `msdp.rc`) 是 OpenHarmony 构建系统的一部分，不可替换

---

## 安全考虑

### 配置文件安全

| 安全考虑 | 状态 | 说明 |
|----------|------|------|
| 配置文件篡改保护 | 需系统完整性保护 | /etc/init/ 目录应受保护 |
| 配置文件签名 | TODO | 建议添加数字签名 |
| 配置文件权限 | 系统决定 | 通常为 644 (rw-r--r--) |

### 权限最小化

**sensors 服务**: ✅ 权限较少，符合最小权限原则

**msdp 服务**: ⚠️ 权限过多，建议审计

**详细分析**: [07_Security_Audit.md](./07_Security_Audit.md)

---

## 相关跳转

- [项目概览](./00_Overview.md) - 组件定位
- [目录结构](./01_Directory_Structure.md) - 文件组织
- [架构说明](./02_Architecture.md) - 依赖关系
- [N-API 文档](./03_NAPI_API.md) - N-API 说明（无 N-API）
- [安全评审](./07_Security_Audit.md) - 权限和安全分析

---

**最后更新**: 2026-02-06
