# Brotli 第三方库集成文档

## 库概览

| 项目 | 内容 |
|-----|------|
| **库名称** | Brotli |
| **版本** | v1.1.0 |
| **许可证** | MIT License |
| **上游地址** | [google/brotli](http://github.com/google/brotli) |
| **OH 组件** | @ohos/brotli |

Brotli 是一种通用无损压缩算法，结合了 LZ77 算法变体、Huffman 编码和二阶上下文建模，在压缩比上与目前最好的通用压缩方法相当，同时保持了与 deflate 相似的压缩速度。

## OpenHarmony 适配概述

Brotli 在 OpenHarmony 中的集成相对简单，主要特点如下：

### ✅ 无需代码 Patch

- Brotli 是纯 C 实现的跨平台压缩库
- 源代码本身无需任何 OH 特定修改
- OH 的配置修改通过 `productdefine_common` 项目的 PR 863 集中管理

### ✅ 已适配 GN 构建系统

- 完整的 `BUILD.gn` 构建配置
- 启用了 ARM PAC（指针认证）安全特性
- 共享库构建配置完整

### ✅ 安全版本

- 集成的 v1.1.0 版本已包含 CVE-2020-8927 安全修复

## 核心使用场景

1. **HTTP 压缩传输**：curl 使用 Brotli 进行 HTTP 请求/响应的压缩
2. **字体压缩**：WOFF2 等字体格式内部使用 Brotli
3. **图像格式**：JPEG XL 等现代图像格式的压缩后端

## 文档导航

### 必读文档

| 文档 | 说明 | 优先级 |
|-----|------|-------|
| [SUMMARY.md](SUMMARY.md) | 阅读路线建议 | ⭐⭐⭐ |
| [01_Overview.md](01_Overview.md) | 原始库功能介绍 | ⭐⭐ |
| [02_Patches.md](02_Patches.md) | Patch 策略说明 | ⭐⭐⭐ |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系和使用场景 | ⭐⭐⭐ |

### 技术参考

| 文档 | 说明 | 优先级 |
|-----|------|-------|
| [03_Build_Integration.md](03_Build_Integration.md) | GN 构建配置详解 | ⭐⭐ |
| [06_Security.md](06_Security.md) | 安全风险分析 | ⭐⭐ |

## 快速开始

### 在 OH 模块中依赖 Brotli

```gn
deps += [ "//third_party/brotli:brotli_shared" ]
```

### 头文件引用

```c
#include <brotli/decode.h>
#include <brotli/encode.h>
```

## 相关资源

- **上游仓库**：http://github.com/google/brotli
- **上游文档**：http://brotli.org
- **OH 配置 PR**：https://gitee.com/openharmony/productdefine_common/pulls/863
- **bundle.json**：`//third_party/brotli/bundle.json`
- **BUILD.gn**：`//third_party/brotli/BUILD.gn`

## 维护信息

| 项目 | 内容 |
|-----|------|
| **版本负责人** | heqianmo@huawei.com |
| **上次更新** | 2023-08-28 |
| **CVE 状态** | 无已知未修复漏洞 |
| **Patch 策略** | 通过 productdefine_common 管理 |
