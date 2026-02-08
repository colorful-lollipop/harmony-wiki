# bzip2 - OpenHarmony 集成文档

> **bzip2** 是一个高质量的无损数据压缩库，使用块排序算法实现高压缩率。

---

## 📋 快速概览

| 属性 | 信息 |
|------|------|
| **原始库名称** | bzip2 / libbzip2 |
| **上游版本** | 1.0.8 (2019-07-13) |
| **OH 组件名** | @ohos/bzip2 |
| **许可证** | bzip2 License (BSD-style) |
| **所属子系统** | thirdparty |
| **适配复杂度** | ⭐ 极低（无源码修改） |

---

## 🎯 OH 适配概述

**bzip2 在 OpenHarmony 中的集成几乎"零修改"：**

- ✅ **无 Patch**: 未应用任何上游补丁
- ✅ **无源码修改**: 完全使用原始源代码
- ✅ **无 OH 特定 API**: API 与上游 100% 兼容
- ✅ **构建简单**: 仅添加 GN 构建包装

**OpenHarmony 的调整：**
1. 添加 `BUILD.gn` 构建配置
2. 仅提供静态库（不包含命令行工具）
3. 精简源文件，仅包含库的核心实现

---

## 📚 文档导航

### 核心文档

| 文档 | 内容 | 推荐阅读 |
|------|------|----------|
| [SUMMARY.md](./SUMMARY.md) | 阅读路线建议 | 📌 首次阅读 |
| [01_Overview.md](./01_Overview.md) | 原始库简介 | ⭐ 必读 |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建适配 | ⭐ 必读 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用 | ⭐ 必读 |

### 详细文档

| 文档 | 内容 | 推荐阅读 |
|------|------|----------|
| [02_Patches.md](./02_Patches.md) | Patch 详细分析 | 参考 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异 | 参考 |
| [06_Security.md](./06_Security.md) | 安全风险分析 | 参考 |

### 工作文档

| 文档 | 内容 |
|------|------|
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | 项目评估结果（完整分析） |
| [_work/NOTES.md](./_work/NOTES.md) | 分析过程记录 |
| [_work/PLAN.md](./_work/PLAN.md) | 任务进度 |

---

## 🚀 快速开始

### 在 OH 中使用 bzip2

```gn
# 在你的 BUILD.gn 中添加依赖
deps = [ "//third_party/bzip2:libbz2" ]
```

```cpp
// C++ 代码示例
#include "bzlib.h"

// 压缩
char source[10000] = "...";
char dest[10000];
unsigned int destLen = sizeof(dest);
BZ2_bzBuffToBuffCompress(dest, &destLen, source, sizeof(source), 9, 0, 0);

// 解压
char dest2[10000];
unsigned int dest2Len = sizeof(dest2);
BZ2_bzBuffToBuffDecompress(dest2, &dest2Len, dest, destLen, 0, 0);
```

### 构建 bzip2 库

```bash
# 在 OH 根目录执行
./build.py --product-name <your_product> --build-target bzip2
```

---

## 🔑 关键信息

### 为什么选择 bzip2？

- **高压缩率**: 压缩率接近最佳统计压缩器（PPM 系列）
- **快速解压**: 解压速度是最佳技术的 6 倍
- **开源免费**: 无专利限制，BSD-style 许可证
- **稳定可靠**: 多年验证，被广泛使用

### OH 集成特点

| 特性 | 说明 |
|------|------|
| **维护成本** | 极低（无 OH 修改） |
| **升级难度** | 极低（无上游冲突） |
| **API 兼容性** | 100% 兼容上游 |
| **安全状态** | 使用最新稳定版 1.0.8 |

### 已知问题

- ⚠️ **公共头文件错误**: `bundle.json` 中错误地将私有头文件 `bzlib_private.h` 列为公共头文件
- ⚠️ **使用情况不明确**: 未发现直接依赖者，可能通过间接方式使用

---

## 📖 上游资源

- **官方主页**: https://sourceware.org/bzip2/
- **源码仓库**: https://sourceware.org/git/?p=bzip2.git
- **测试套件**: https://sourceware.org/git/bzip2-tests.git
- **维护者**: Julian Seward <jseward@acm.org>

---

## 🔗 相关资源

- **bundle.json**: OpenHarmony 组件配置
- **BUILD.gn**: GN 构建配置
- **bzlib.h**: 公共 API 头文件
- **LICENSE**: 许可证文件

---

**文档维护**: 本文档由 OpenHarmony 第三方库 Wiki 生成 Agent 自动生成并维护
**最后更新**: 2026-02-07
