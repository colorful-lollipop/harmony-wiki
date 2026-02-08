# GN - OpenHarmony Wiki

> **GN (Generate Ninja)** 是一个元构建系统，为 Ninja 生成构建文件。
>
> 本文档专注于 GN 在 **OpenHarmony** 中的集成、适配和使用方式。

---

## 库概览

| 属性 | 值 |
|-----|---|
| **上游名称** | GN (Generate Ninja) |
| **上游版本** | d823fd85da3fb83146f734377da454473b93a2b2 |
| **上游许可证** | BSD-3-Clause |
| **上游地址** | https://gn.googlesource.com/gn.git |
| **OH 组件名** | @ohos/gn |
| **OH 版本** | 3.1 |
| **OH 子系统** | thirdparty |
| **适配系统** | mini, small, standard |

## 在 OpenHarmony 中的作用

GN 是 **OpenHarmony 的标准构建系统**，所有 OH 组件使用 GN 定义构建规则。

```
开发者编写 BUILD.gn → GN 生成 Ninja 文件 → Ninja 执行构建
```

## OH 适配概述

### Patch 情况

| Patch 数量 | 主要修改 |
|-----------|---------|
| 1 个 | 简化构建目录路径表示，移除 BUILD_DIR 占位符 |

### 关键差异

| 方面 | 上游 GN | OH GN |
|-----|--------|-------|
| 交付方式 | 源码编译 | Prebuilt 二进制 |
| 路径表示 | 支持 BUILD_DIR 占位符 | 使用实际路径 |
| 配置方式 | 项目自定义 | 统一通过 //build/ohos.gni |

## 文档导航

### 核心文档

| 文档 | 内容 |
|-----|------|
| [01_Overview.md](./01_Overview.md) | 原始库简介、OH 中的定位和作用 |
| [02_Patches.md](./02_Patches.md) | **核心文档** - Patch 详细分析、修改目的、回归风险 |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建适配、prebuilt 集成 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系、使用场景、典型示例 |
| [05_API_Differences.md](./05_API_Differences.md) | API/接口差异说明（GN 作为工具，API 等价） |
| [06_Security.md](./06_Security.md) | 安全风险分析、CVE 状态、供应链安全 |

### 工作文档

- [`_work/ASSESSMENT.md`](./_work/ASSESSMENT.md) - 项目评估报告、信息收集结果
- [`_work/NOTES.md`](./_work/NOTES.md) - 分析过程记录
- [`_work/PLAN.md`](./_work/PLAN.md) - 任务进度

## 快速参考

### GN 在 OH 中的位置

```
# Prebuilt GN 二进制（典型位置）
prebuilts/build-tools/linux-x64/bin/gn
developtools/integration_verification/tools/precise_build/gn

# GN 源码
third_party/gn/
```

### 常用 GN 命令

```bash
# 生成构建文件
gn gen out

# 检查 BUILD.gn 语法
gn check //path/to/component

# 查看目标详情
gn desc out //path/to:target

# 查看依赖树
gn desc out //path/to:target --tree
```

### 典型 BUILD.gn 结构

```gn
import("//build/ohos.gni")

ohos_shared_library("my_component") {
  sources = [ "src/foo.cpp" ]
  external_deps = [ "hilog:libhilog" ]
}
```

## 维护信息

- **维护者**: huhao51@huawei.com
- **文档创建**: 2025-02-08
- **文档版本**: 1.0

## 相关链接

- [GN 官方文档](https://gn.googlesource.com/gn/+/main/docs/)
- [GN 快速入门](https://gn.googlesource.com/gn/+/main/docs/quick_start.md)
- [GN 参考手册](https://gn.googlesource.com/gn/+/main/docs/reference.md)
- [OpenHarmony 构建系统文档](../../../build/docs/)
