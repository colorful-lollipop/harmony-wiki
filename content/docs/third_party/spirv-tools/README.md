# SPIRV-Tools Wiki

## 库概览

SPIRV-Tools 是 Khronos Group 提供的 SPIR-V（Standard Portable Intermediate Representation）处理工具集，在 OpenHarmony 中主要用于支持 **Vulkan/OpenGL 一致性测试（vk-gl-cts）**。

### 关键信息

| 项目 | 信息 |
|------|------|
| **OH 组件名称** | `@ohos/spirv-tools` |
| **OH 版本** | 3.2 |
| **上游版本** | vulkan-sdk-1.3.275.0 |
| **许可证** | Apache-2.0 / 3-Clause BSD License |
| **所属子系统** | thirdparty |
| **主要依赖者** | vk-gl-cts（Khronos 一致性测试套件） |

---

## OpenHarmony 适配特点

### 无 Patch 适配

SPIRV-Tools 在 OpenHarmony 中采用**原生适配**方式：

- ❌ 没有 `.patch` 文件
- ❌ 没有 `patches/` 目录
- ❌ 源代码中没有 `#ifdef OHOS` 条件编译
- ✅ 所有适配通过 `BUILD.gn` 构建系统完成

### 主要适配内容

1. **GN 构建系统集成**：完整的 `BUILD.gn` 配置
2. **deqp 目标命名**：使用 `deqp_` 前缀标识服务于 deqp 测试框架
3. **编译配置**：针对 OHOS 的编译器标志和宏定义
4. **与 vk-gl-cts 集成**：依赖 vk-gl-CTS 构建系统生成中间文件

---

## 文档导航

### 必读文档

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [README.md](./README.md) | 本文档，库概览和导航 | ⭐⭐⭐ |
| [02_Patches.md](./02_Patches.md) | Patch 分析（无 Patch 结论） | ⭐⭐⭐ |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 适配详解 | ⭐⭐⭐ |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系和使用场景 | ⭐⭐⭐ |

### 背景信息

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [01_Overview.md](./01_Overview.md) | 原始库功能介绍 | ⭐⭐ |
| [SUMMARY.md](./SUMMARY.md) | 阅读路线建议 | ⭐ |

### 工作文档

| 文档 | 说明 |
|------|------|
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | 项目评估结果（内部使用） |
| [_work/NOTES.md](./_work/NOTES.md) | 分析过程记录 |
| [_work/PLAN.md](./_work/PLAN.md) | 任务进度跟踪 |

---

## 快速开始

### 在 OH 中使用 SPIRV-Tools

```gn
# 在 BUILD.gn 中添加依赖
deps = [
  "//third_party/spirv-tools:libdeqp_spirvtools",
  "//third_party/spirv-tools/source/opt:libdeqp_spirvtools-opt",
  "//third_party/spirv-tools/source/val:libdeqp_spirvtools-val",
]
```

### 包含头文件

```cpp
#include "spirv-tools/libspirv.h"
#include "spirv-tools/libspirv.hpp"
#include "spirv-tools/optimizer.hpp"
```

---

## 关键概念

### SPIR-V 简介

SPIR-V（Standard Portable Intermediate Representation）是 Khronos 定义的一种二进制中间表示格式，用于：

- **Vulkan** 着色器
- **OpenGL** 着色器（通过 ARB_gl_spirv 扩展）
- **OpenCL** 内核

### SPIRV-Tools 组件

| 组件 | 功能 |
|------|------|
| **Assembler** | 将 SPIR-V 汇编文本转换为二进制 |
| **Disassembler** | 将 SPIR-V 二进制转换为汇编文本 |
| **Validator** | 根据 SPIR-V 规范验证模块 |
| **Optimizer** | 应用各种代码优化 pass |
| **Linker** | 合并多个 SPIR-V 模块 |
| **Reducer** | 简化测试用例 |
| **Fuzzer** | 生成模糊测试用例 |

---

## 版本信息

### 当前版本

- **上游版本**: vulkan-sdk-1.3.275.0
- **OH 版本**: 3.2
- **上游仓库**: https://github.com/KhronosGroup/SPIRV-Tools

### 版本说明

该库版本基于 Vulkan SDK 配套的 SPIRV-Tools 版本，确保与 Vulkan 规范的兼容性。

---

## 相关资源

### 内部资源

- [OpenHarmony third_party 目录](../)
- [vk-gl-cts 集成说明](../vk-gl-cts/)
- [SPIRV-Headers](../spirv-headers/)

### 外部资源

- [SPIRV-Tools 上游文档](https://github.com/KhronosGroup/SPIRV-Tools)
- [SPIR-V 规范](https://www.khronos.org/registry/spir-v/)
- [Khronos 官网](https://www.khronos.org/)

---

## 贡献者

**维护者**: zhangleiyu1@huawei.com

**组件所有者**: zhangleiyu1@huawei.com

---

*最后更新: 2026-02-07*
