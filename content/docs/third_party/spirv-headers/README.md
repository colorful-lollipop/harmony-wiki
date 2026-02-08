# SPIRV-Headers OpenHarmony Wiki

## 库概览

SPIRV-Headers 是 Khronos Group 维护的官方 SPIR-V（Standard Portable Intermediate Representation）规范头文件库，为 OpenHarmony 图形栈提供 SPIR-V 指令集的标准定义。

| 属性 | 值 |
|------|-----|
| **版本** | vulkan-sdk-1.3.275.0 |
| **许可证** | Apache-2.0 |
| **上游地址** | https://github.com/KhronosGroup/SPIRV-Headers.git |
| **OH 维护者** | zhangleiyu1@huawei.com |
| **Patch 数量** | 0（无 Patch） |

## OH 适配概述

### 适配策略

OpenHarmony 对 SPIRV-Headers 采用**最小修改策略**：

1. **无 Patch**：作为纯头文件库，无需任何代码修改
2. **BUILD.gn 适配**：添加 GN 构建系统支持
3. **版本选择**：通过 sources 列表选择包含的头文件
4. **依赖管理**：与其他 SPIR-V 工具链协同

### 核心价值

- 为 Vulkan 实现提供标准 SPIR-V 规范
- 为 spirv-tools 提供指令定义和 JSON 语法
- 为图形一致性测试提供基础支持

## 文档导航

### 快速开始

| 文档 | 说明 |
|------|------|
| [01_Overview.md](01_Overview.md) | 原始库功能和 OH 定位介绍 |
| [SUMMARY.md](SUMMARY.md) | 完整阅读路线建议 |

### 核心内容

| 文档 | 说明 |
|------|------|
| [02_Patches.md](02_Patches.md) | Patch 分析（本库无 Patch） |
| [03_Build_Integration.md](03_Build_Integration.md) | BUILD.gn 构建适配详解 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系和使用场景 |
| [05_API_Differences.md](05_API_Differences.md) | API 差异（本库无差异） |
| [06_Security.md](06_Security.md) | 安全风险分析 |

### 工作文档

| 文档 | 说明 |
|------|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估报告 |
| [_work/NOTES.md](_work/NOTES.md) | 分析过程记录 |
| [_work/PLAN.md](_work/PLAN.md) | 任务进度计划 |

## 快速索引

### 主要依赖者

```
spirv-tools  →  SPIRV-Headers  ←  vk-gl-cts
                           ←  amber
```

### 使用场景

- **Vulkan 一致性测试**：vk-gl-cts 使用头文件进行测试
- **SPIR-V 工具链**：spirv-tools 使用语法定义构建工具
- **测试框架**：amber 使用头文件执行着色器测试

### 关键文件

| 文件 | 路径 |
|------|------|
| BUILD.gn | `//third_party/spirv-headers/BUILD.gn` |
| 核心头文件 | `include/spirv/unified1/spirv.h` |
| GLSL 扩展 | `include/spirv/unified1/GLSL.std.450.h` |

## 版本信息

- **当前版本**：vulkan-sdk-1.3.275.0
- **上游版本**：跟随 Khronos SPIRV-Headers 主分支
- **更新策略**：跟随 Vulkan SDK 版本同步更新

## 相关资源

- [SPIR-V Registry](https://www.khronos.org/registry/spir-v/)
- [SPIRV-Headers GitHub](https://github.com/KhronosGroup/SPIRV-Headers)
- [OpenHarmony 图形架构](../README.md)
- [spirv-tools Wiki](../spirv-tools/wiki/README.md)
