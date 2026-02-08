# libpng OpenHarmony Wiki

## 库概述

libpng 是 PNG（Portable Network Graphics）格式的官方参考实现库，提供完整的 PNG 图像编解码功能。在 OpenHarmony 系统中，libpng 作为图像处理的基础组件，被用于 PNG 图像的加载、解析和显示。

**上游版本**: 1.6.44  
**许可证**: libpng License  
**依赖**: zlib

---

## OpenHarmony 适配概述

libpng 在 OpenHarmony 中经过了以下主要适配：

### 1. 构建系统适配
- 采用 GN 构建系统，通过 `BUILD.gn` 文件集成
- 使用 `libpng_action` 实现源码解压和 Patch 应用
- 支持 lite 和 standard 两种构建模式

### 2. 性能优化
- **ARM NEON 指令优化**: 扩展了滤波器函数，支持向量化计算
- **多行解码优化**: `PNG_MULTY_LINE_ENABLE` 宏启用批量行处理
- **符号隐藏**: ARM 平台启用 `-fvisibility=hidden` 减少动态链接开销

### 3. 安全修复
- 应用了 12 个 CVE 安全修复 Patch
- 覆盖整数溢出、堆缓冲区溢出等多种漏洞类型

---

## 文档导航

| 文档 | 内容说明 |
|------|----------|
| [SUMMARY.md](SUMMARY.md) | 阅读路线建议 |
| [01_Overview.md](01_Overview.md) | 原始库功能简介 |
| [02_Patches.md](02_Patches.md) | **核心** - Patch 详细分析 |
| [03_Build_Integration.md](03_Build_Integration.md) | 构建适配说明 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | OH 使用场景和依赖关系 |
| [05_API_Differences.md](05_API_Differences.md) | API 差异分析 |
| [06_Security.md](06_Security.md) | 安全风险分析 |

---

## 快速开始

### 在 OH 模块中引用 libpng

**动态链接方式**:
```gn
deps += [ "//third_party/libpng:libpng" ]
```

**静态链接方式**:
```gn
deps += [ "//third_party/libpng:libpng_static" ]
```

### 头文件引用

```c
#include "png.h"
#include "pngconf.h"
```

---

## 版本信息

| 组件 | 版本 |
|------|------|
| 上游 libpng | 1.6.44 |
| OH 组件版本 | 3.1 |
| 最后更新 | 2025年 |

---

## 相关资源

- [上游官网](http://www.libpng.org/pub/png/libpng.html)
- [PNG 格式规范](http://www.w3.org/TR/PNG/)
- OpenHarmony Image Framework
