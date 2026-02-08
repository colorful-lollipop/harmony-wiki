# 01 - 原始库简介

本文档简要介绍 Arm Optimized Routines 原始库的功能和在 OpenHarmony 中的定位。

---

## 基本信息

| 属性 | 值 |
|------|-----|
| **名称** | Arm Optimized Routines |
| **开发者** | ARM Limited |
| **上游版本** | v25.01 (2025年1月季度发布) |
| **许可证** | MIT OR Apache-2.0 WITH LLVM-exception |
| **上游地址** | https://github.com/ARM-software/optimized-routines |
| **发布周期** | 季度发布（vYY.MM 格式） |

---

## 原始功能概述

optimized-routines 是 ARM 提供的高性能库函数实现，针对 ARM 架构处理器优化。

### 主要功能模块

#### 1. 数学函数库 (math/)

**标量数学函数**（C 实现）:
- 三角函数：`sin`、`cos`、`tan`、`asin`、`acos`、`atan` 等
- 指数/对数：`exp`、`log`、`log2`、`pow` 等
- 双曲函数：`sinh`、`cosh`、`tanh` 等
- 特殊函数：`erf`、`tgamma` 等

**向量数学函数**（AArch64 优化）:
- **AdvSIMD (NEON)**: 128 位向量指令优化，68+ 个函数
- **SVE**: 可伸缩向量扩展，66+ 个函数

**质量标准**:
- 双精度函数 ULP 误差 < 0.66
- 单精度函数 ULP 误差 < 1.0
- 符合 ISO C Annex F、POSIX、IEEE 754-2008 标准

#### 2. 字符串操作库 (string/)

**字符串函数**（ARM/AArch64 汇编优化）:
- 内存操作：`memcpy`、`memset`、`memcmp`、`memchr`
- 字符串处理：`strcpy`、`strcmp`、`strlen`、`strchr`、`strrchr`
- 其他：`stpcpy`、`strnlen`、`strncmp`

**架构特性支持**:
- **SVE**: `memcpy-sve.S`、`memset-sve.S` 等（实验性）
- **MTE**: `memchr-mte.S`、`strlen-mte.S` 等（内存标记扩展）
- **MOPS**: `memcpy-mops.S`、`memset-mops.S` 等（内存操作指令）

#### 3. 网络库 (networking/)

**校验和优化**:
- `chksum_simd.c` - SIMD 优化的校验和计算
- 支持 ARM 和 AArch64 架构

---

## 在 OpenHarmony 中的定位

### 集成位置

optimized-routines 集成到 **musl libc** 中，为 OpenHarmony 全系统提供优化的字符串和数学函数实现：

```
OpenHarmony 系统调用
        ↓
    musl libc
        ↓
optimized-routines  (优化实现)
```

### 系统覆盖

| OH 系统类型 | 代表内核 | 使用架构 | 优化函数 |
|------------|---------|---------|---------|
| **mini** | LiteOS-M | ARMv7-M | 字符串 + 数学 (C) |
| **small** | UniProton | ARMv7-A | 字符串 + 数学 (C) |
| **standard** | LiteOS-A | AArch64 | 字符串 + 数学 (C + 向量) |

### 应用场景

#### 字符串函数优化

用于系统调用的底层操作：
- `memcpy` / `memset` - 内存拷贝、清零
- `strcmp` / `strlen` - 字符串比较、计算长度
- 提升系统响应速度和整体性能

#### 数学函数优化

用于应用层的计算密集型场景：
- **图形计算**：`sinf` / `cosf` - 三角函数
- **信号处理**：`expf` / `logf` / `powf` - 指数/对数/幂
- **科学计算**：单精度数学计算
- 提升多媒体、游戏、AI 应用的性能

---

## 架构支持矩阵

| 架构 | 字符串函数 | 数学函数 | 向量化 | OH 使用 |
|------|-----------|----------|--------|---------|
| **ARMv7-M** | ✅ 汇编 | ✅ C | ❌ | LiteOS-M (IoT) |
| **ARMv7-A** | ✅ 汇编 | ✅ C | ❌ | UniProton (RTOS) |
| **AArch64** | ✅ 汇编 | ✅ C | ❌ | LiteOS-A |
| **AArch64 + SVE** | ✅ (实验) | ✅ 向量 | ✅ | 条件支持 |
| **AArch64 + MTE** | ✅ | ❌ | ❌ | 条件支持 |

**说明**:
- SVE (Scalable Vector Extension) - 可伸缩向量扩展，需要硬件支持
- MTE (Memory Tagging Extension) - 内存标记扩展，用于内存安全

---

## 编译器要求

| 特性 | 编译器版本要求 |
|------|--------------|
| **基础编译** | GCC / Clang (任意版本) |
| **SVE 支持** | GCC >= 10 **或** LLVM/Clang >= 5 |
| **ACLE (ARM C 语言扩展)** | 必须启用 |

**OpenHarmony 编译配置**:
- LLVM 构建时定义 `__IS_LLVM_BUILD` 宏
- 通过 GN 变量 `ARM_FEATURE_SVE` / `ARM_FEATURE_MTE` 控制特性启用

---

## 与其他优化库对比

| 特性 | optimized-routines | glibc | newlib |
|------|------------------|-------|--------|
| **优化目标** | ARM 架构 | 通用 | 嵌入式 |
| **汇编优化** | ✅ ARM/AArch64 | ✅ 多架构 | 部分 |
| **向量化** | ✅ SVE | ✅ AVX/NEON | ❌ |
| **许可证** | MIT/Apache-2.0 | LGPL | BSD |
| **适用于 OH** | ✅ 集成到 musl | ❌ | ❌ |

---

## 代码规模统计

| 类型 | 数量 | 主要分布 |
|------|------|---------|
| `.c` 文件 | 274 | math/ (通用实现) |
| `.S` 文件 | 47 | string/arm/, string/aarch64/ |
| `.h` 文件 | 61 | math/include/, string/include/ |

---

## 相关资源

### 官方资源
- **GitHub 仓库**: https://github.com/ARM-software/optimized-routines
- **发布页面**: https://github.com/ARM-software/optimized-routines/releases
- **Android AOSP**: platform/external/arm-optimized-routines

### 内部文档
- **[02_Adaptations.md](02_Adaptations.md)** - OH 适配说明
- **[03_Build_Integration.md](03_Build_Integration.md)** - 构建集成详解
- **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 使用情况和依赖

---

**文档版本**: 1.0
**最后更新**: 2026-02-07
