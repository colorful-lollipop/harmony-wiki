# CMSIS - OpenHarmony Wiki

## 概述

本 Wiki 文档记录 **CMSIS (Cortex Microcontroller Software Interface Standard)** 在 OpenHarmony 中的集成与适配情况。

CMSIS 是 ARM 官方提供的硬件抽象层标准，为基于 Arm Cortex 处理器的微控制器提供统一的软件接口。

---

## 文档导航

| 文档 | 说明 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 原始库简介与 OH 定位 |
| [02_Patches.md](./02_Patches.md) | Patch 分析（本库无 Patch） |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建系统适配 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用场景 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异分析 |
| [06_Security.md](./06_Security.md) | 安全风险分析 |

---

## 关键信息速览

### 库基本信息

| 属性 | 值 |
|------|-----|
| **上游库名** | CMSIS_6 |
| **上游版本** | 6.0.0 |
| **上游许可证** | Apache License V2.0 |
| **OH 组件名** | @ohos/cmsis |
| **OH 组件版本** | 3.1 |
| **所属子系统** | thirdparty |
| **适配系统类型** | mini (轻量级系统) |

### 核心特点

- **零 Patch**：本库无需任何 Patch，保持上游源码纯净
- **头文件库**：纯头文件实现，无编译产物
- **外部适配**：OH 定制化工作全部在外部目录完成
- **标准兼容**：使 LiteOS-M 兼容 CMSIS-RTOS2 标准

### 目录结构

```
CMSIS/
├── Core/Include/          # CMSIS-Core 头文件（硬件抽象）
│   ├── core_cm*.h         # Cortex-M 系列内核支持
│   ├── core_ca.h          # Cortex-A 系列内核支持
│   ├── m-profile/         # M-profile 编译器抽象
│   └── a-profile/         # A-profile 编译器抽象
└── RTOS2/Include/         # CMSIS-RTOS2 头文件（OS API）
    ├── cmsis_os2.h        # RTOS2 API 定义
    └── os_tick.h          # OS Tick 定义
```

---

## OH 适配核心

CMSIS 本身只是一个接口标准定义，真正的 OpenHarmony 适配工作在以下目录完成：

| 适配层路径 | 功能说明 |
|-----------|----------|
| `kernel/liteos_m/kal/cmsis/` | **核心适配层**：CMSIS-RTOS2 → LiteOS-M 适配 |
| `device/soc/*/kal/cmsis/` | SoC 特定适配（海思芯片等） |

### 适配层架构

```
应用程序 (CMSIS-RTOS2 API)
         ↓
    cmsis_os2.h (第三方库接口定义)
         ↓
   cmsis_liteos2.c (OH 适配实现)
         ↓
    LiteOS-M 内核 API
         ↓
    硬件抽象层
```

---

## 主要依赖者

| 模块 | 用途 |
|------|------|
| kernel/liteos_m/kal/cmsis | 核心适配实现 |
| test/xts/acts/kernel_lite/kernelcmsis_hal | CMSIS 功能测试 |
| device/soc/hisilicon/*/kal/cmsis | 海思 SoC 适配 |
| device/qemu/arm_mps3_an547/liteos_m/board | QEMU 模拟器支持 |

---

## 维护建议

### 升级上游版本

1. **风险等级**：中
2. **需验证内容**：
   - `kal/cmsis` 适配层 API 兼容性
   - XTS 测试套件通过率
   - 各 SoC 平台编译验证

3. **无需关注**：
   - Patch 重新适配（无 Patch）
   - BUILD.gn 修改（仅包含路径）

---

*文档版本: 1.0*  
*最后更新: 2025-02-08*
