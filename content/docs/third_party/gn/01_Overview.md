# 01 - 原始库简介

## 1.1 库基本信息

| 属性 | 值 |
|-----|---|
| **名称** | GN (Generate Ninja) |
| **版本** | d823fd85da3fb83146f734377da454473b93a2b2 |
| **许可证** | BSD-3-Clause |
| **上游地址** | https://gn.googlesource.com/gn.git |
| **OH 组件名** | @ohos/gn |
| **OH 版本** | 3.1 |
| **OH 子系统** | thirdparty |

## 1.2 原始功能描述

GN 是一个**元构建系统（Meta-Build System）**，其核心功能是生成 [Ninja](https://ninja-build.org) 构建文件。

### 主要特性

| 特性 | 说明 |
|-----|------|
| **大规模项目支持** | 可高效处理数千个构建文件、数万个源文件 |
| **清晰语法** | 非专业人员也能轻松理解和编辑构建配置 |
| **多平台支持** | 干净地表达跨平台复杂构建变体 |
| **并行输出目录** | 同时维护 debug/release/不同平台的构建配置 |
| **正确性检查** | `gn check`、`testonly`、`assert_no_deps` 等工具 |
| **内置帮助** | 全面的命令行帮助系统 |

### 支持的语言
- C/C++
- Rust
- Objective-C
- Swift
- 其他语言可通过 "action" 规则集成

## 1.3 在 OpenHarmony 中的作用

### 核心定位
GN 是 **OpenHarmony 的标准构建系统**，所有 OH 组件都使用 GN 定义构建规则。

### OH 中的典型使用流程

```
开发者编写 BUILD.gn → 使用 gn 生成 Ninja 文件 → ninja 执行构建
```

### OH 构建系统的 GN 配置

在 OH 中，开发者通常在 BUILD.gn 中导入标准模板：

```gn
# 标准组件
import("//build/ohos.gni")

# 测试组件  
import("//build/test.gni")

# Lite 系统组件
import("//build/lite/config/component/lite_component.gni")
```

### GN 在 OH 中的部署位置

| 位置 | 说明 |
|-----|------|
| `//build/` | OH 构建系统核心，包含 .gni 模板文件 |
| `//prebuilts/build-tools/linux-x64/bin/gn` | prebuilt gn 二进制（典型位置） |
| `//developtools/integration_verification/tools/precise_build/gn` | 精确构建工具链中的 gn |

## 1.4 与上游的差异概览

| 方面 | 上游 GN | OH GN |
|-----|--------|-------|
| 交付方式 | 源码编译 | prebuilt 二进制 |
| 路径表示 | 支持 BUILD_DIR 占位符 | 使用实际路径（简化版） |
| 配置方式 | 项目自定义 .gn 文件 | 统一通过 //build/ohos.gni |
| Patch 数量 | 0 | 1 个路径处理简化 Patch |

## 1.5 相关文档

- [GN 快速入门](https://gn.googlesource.com/gn/+/main/docs/quick_start.md)
- [GN 常见问题](https://gn.googlesource.com/gn/+/main/docs/faq.md)
- [GN 参考手册](https://gn.googlesource.com/gn/+/main/docs/reference.md)
- [OpenHarmony 构建系统文档](../../../build/docs/)
