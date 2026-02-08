# optimized-routines @ OpenHarmony

本文档说明 OpenHarmony 对 `third_party/optimized-routines` 库的集成与适配。

---

## 快速概览

| 属性 | 值 |
|------|-----|
| **原始库** | Arm Optimized Routines |
| **上游版本** | v25.01 (2025年1月发布) |
| **OH 组件版本** | 3.1 |
| **许可证** | MIT OR Apache-2.0 WITH LLVM-exception |
| **上游地址** | https://github.com/ARM-software/optimized-routines |
| **子系统** | thirdparty |

### OH 适配特点

- ✅ **源码级集成**：直接编译源码集成到 musl libc，非预编译库链接
- ✅ **三系统支持**：mini (LiteOS-M)、small (UniProton)、standard (LiteOS-A)
- ✅ **架构全覆盖**：ARMv7-M / ARMv7-A / AArch64
- ✅ **特性支持**：SVE / MTE / PAC-RET / HWASAN
- ✅ **无 Patch 适配**：通过构建系统集成，保持与上游同步

### 库定位

optimized-routines 为 OpenHarmony 提供 ARM 架构优化的字符串和数学函数实现：

- **字符串函数**：`memcpy`、`memset`、`strcmp`、`strlen` 等常用操作
- **数学函数**：`sinf`、`cosf`、`expf`、`logf`、`powf` 等数学计算

这些优化实现通过 musl libc 暴露给整个系统，用于系统调用、用户空间库、图形计算、信号处理等场景。

---

## 文档导航

### 快速入门

- **[SUMMARY.md](SUMMARY.md)** - 阅读路线建议（根据角色选择）
- **[01_Overview.md](01_Overview.md)** - 原始库简介（3分钟快速了解）

### 核心文档

- **[02_Adaptations.md](02_Adaptations.md)** - OH 适配说明（无 Patch）
  - OH 特定文件清单
  - 源码级集成机制
  - 符号别名映射

- **[03_Build_Integration.md](03_Build_Integration.md)** - 构建集成详解
  - BUILD.gn 配置结构
  - ARMv7 / AArch64 配置
  - SVE / MTE 条件编译

- **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 使用情况和依赖
  - 依赖者清单
  - 集成方式
  - 使用场景

- **[06_Security.md](06_Security.md)** - 安全风险分析
  - PAC-RET / 栈保护 / HWASAN
  - 安全特性配置

### 工作文档

- **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - 项目评估结果
- **[_work/NOTES.md](_work/NOTES.md)** - 分析过程记录
- **[_work/PLAN.md](_work/PLAN.md)** - 文档计划

---

## 关键概念

### 1. 源码级集成

optimized-routines **不是**以独立静态库链接方式集成，而是直接编译源码集成到 musl libc：

```gn
# musl libc 的 BUILD.gn 中
import("//third_party/optimized-routines/optimized-routines.gni")
sources += OPTRT_STRING_ARM_SRC_FILES_FOR_ARMV7_M
```

**优势**：
- 编译器可以跨模块优化
- 避免符号冲突
- 更紧凑的最终二进制

### 2. 架构支持矩阵

| 架构 | 字符串函数 | 数学函数 | OH 使用 |
|------|-----------|----------|---------|
| ARMv7-M | ✅ 汇编 | ✅ C | LiteOS-M (IoT) |
| ARMv7-A | ✅ 汇编 | ✅ C | UniProton (RTOS) |
| AArch64 | ✅ 汇编 | ✅ C | LiteOS-A |
| AArch64 + SVE | ✅ 实验 | ✅ 向量 | 条件支持 |
| AArch64 + MTE | ✅ | ❌ | 条件支持 |

### 3. 符号别名

汇编代码中定义符号别名，与 musl libc 符号兼容：

```asm
.set __memcpy_aarch64, memcpy
.set __memset_aarch64, memset
```

---

## 常见问题

### Q: 为什么没有 Patch 文件？

A: OH 通过构建系统集成适配（BUILD.gn + optimized-routines.gni），无需修改源代码。这保持了与上游的同步能力。

### Q: 如何在模块中使用优化后的函数？

A: 无需特殊配置。optimized-routines 的函数通过 musl libc 自动可用，正常调用标准 C 库函数即可（如 `memcpy`、`sinf`）。

### Q: SVE / MTE 特性如何启用？

A: 通过 GN 变量 `ARM_FEATURE_SVE` 和 `ARM_FEATURE_MTE` 控制，在相应设备的配置中定义即可。

### Q: 如何升级上游版本？

A: 保留 OH 特定文件（BUILD.gn、optimized-routines.gni、bundle.json、OAT.xml），同步上游源码后验证构建通过即可。

---

## 技术联系

- **Owner**: zhaotianyu9@huawei.com
- **上游 Issue**: https://github.com/ARM-software/optimized-routines/issues

---

**文档版本**: 1.0
**最后更新**: 2026-02-07
