# libabigail Wiki

> OpenHarmony 第三方库集成文档

## 库概览

| 属性 | 值 |
|------|-----|
| **库名称** | libabigail |
| **上游版本** | 2.8 |
| **许可证** | Apache 2.0 with LLVM exception |
| **上游地址** | https://sourceware.org/libabigail/ |
| **OH 组件名** | @ohos/libabigail |
| **所属子系统** | thirdparty |

## 库简介

**libabigail** 是一个用于**应用程序二进制接口 (ABI)** 分析的库和工具集。它能够：

- 从二进制文件（ELF 格式）中提取 ABI 信息
- 生成 ABI 特征文件（ABIXML 格式）
- 比较两个 ABI 特征文件的差异
- 检测 ABI 兼容性问题

## 在 OpenHarmony 中的作用

libabigail 在 OH 中主要用于 **SA (System Ability) 独立升级功能**的 ABI 兼容性检查：

1. 在编译时为动态库生成 ABI 特征文件
2. 对比当前版本与基线版本的 ABI 差异
3. 检测并报告不兼容的 ABI 变更
4. 防止破坏向后兼容性的升级

**重要特点**：
- 仅作为 **host 端工具** 使用
- 不随设备镜像发布
- 无代码 Patch，直接使用上游版本

## 文档导航

### 快速开始

| 文档 | 内容 | 推荐阅读顺序 |
|------|------|-------------|
| [01_Overview.md](01_Overview.md) | 库简介和 OH 定位 | ⭐ 第 1 步 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 在 OH 中的使用方式 | ⭐ 第 2 步 |

### 深度分析

| 文档 | 内容 | 适合读者 |
|------|------|----------|
| [02_Patches.md](02_Patches.md) | Patch 分析（无 Patch 的原因） | 维护人员 |
| [03_Build_Integration.md](03_Build_Integration.md) | BUILD.gn 构建适配详解 | 构建系统开发者 |
| [05_API_Differences.md](05_API_Differences.md) | API 差异说明 | 二次开发者 |
| [06_Security.md](06_Security.md) | 安全风险分析 | 安全工程师 |

### 工作文档

- [`_work/ASSESSMENT.md`](_work/ASSESSMENT.md) - 项目评估报告
- [`_work/NOTES.md`](_work/NOTES.md) - 分析过程记录
- [`_work/PLAN.md`](_work/PLAN.md) - 任务进度计划

## 关键结论

### Patch 情况

**本库无任何 Patch 文件**。

libabigail 是功能完整、接口稳定的成熟库，OH 仅使用其命令行工具而非库 API 集成，因此不需要代码层面的 Patch。

### 构建适配

OH 使用 **3 个 BUILD.gn 文件** 替代上游的 autotools 构建系统：

1. 根目录 `BUILD.gn` - 编译入口和全局配置
2. `src/BUILD.gn` - 静态库构建
3. `tools/BUILD.gn` - 工具（abidiff、abidw）构建

### 依赖关系

```
libabigail
  ├─ 依赖: elfutils (libdw_static)
  ├─ 依赖: libxml2 (系统库)
  ├─ 依赖: lzma (系统库)
  └─ 被依赖: ohos_module_package 模板 (间接)
```

## 贡献与反馈

如发现文档错误或有改进建议，请通过 OpenHarmony 社区渠道反馈。

---

*文档版本: 1.0 | 最后更新: 2026-02-07*
