# 阅读路线建议

本文档为不同角色的读者提供推荐的阅读顺序。

---

## 开发者（使用 libpng）

如果你需要在 OpenHarmony 应用或模块中使用 libpng：

**推荐阅读顺序**:
1. [README.md](README.md) - 快速了解 libpng 在 OH 中的定位
2. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 查看使用示例和依赖配置
3. [01_Overview.md](01_Overview.md) - 了解 libpng 的基本 API

**快速参考**:
```gn
# BUILD.gn 中添加依赖
deps += [ "//third_party/libpng:libpng" ]
```

```c
// 基础使用示例
#include "png.h"
png_image image;
memset(&image, 0, sizeof(image));
image.version = PNG_IMAGE_VERSION;
png_image_begin_read_from_file(&image, "test.png");
```

---

## 系统集成工程师

如果你负责维护 libpng 或需要适配新平台：

**推荐阅读顺序**:
1. [03_Build_Integration.md](03_Build_Integration.md) - 深入理解构建配置
2. [02_Patches.md](02_Patches.md) - 了解所有 OH 定制 Patch
3. [06_Security.md](06_Security.md) - 关注安全修复和升级策略

**关键配置点**:
- ARM NEON 优化开关: `PNG_ARM_NEON`
- 多行解码优化: `PNG_MULTY_LINE_ENABLE`
- 符号隐藏: `-fvisibility=hidden`

---

## 安全工程师

如果你负责安全审计或漏洞响应：

**推荐阅读顺序**:
1. [06_Security.md](06_Security.md) - 安全修复总览
2. [02_Patches.md](02_Patches.md) - CVE Patch 详细分析
3. [03_Build_Integration.md](03_Build_Integration.md) - 构建配置检查

**关注重点**:
- CVE 修复状态和遗漏风险
- Patch 升级策略
- 整数溢出防护措施

---

## 架构师

如果你需要评估 libpng 在 OH 系统中的位置和依赖关系：

**推荐阅读顺序**:
1. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 依赖关系图
2. [03_Build_Integration.md](03_Build_Integration.md) - 构建集成方式
3. [README.md](README.md) - 系统定位

**核心结论**:
- libpng 是 image_framework 的核心依赖
- vk-gl-cts 测试套件大量使用 libpng
- 建议保持静态链接以减少运行时依赖

---

## Patch 升级指南

如果你需要将 OH 的 libpng 升级到上游新版本：

**推荐阅读顺序**:
1. [02_Patches.md](02_Patches.md) - 分析现有 Patch 目的
2. [06_Security.md](06_Security.md) - 识别必须保留的安全修复
3. [03_Build_Integration.md](03_Build_Integration.md) - 检查构建配置兼容性

**升级检查清单**:
- [ ] 新版本是否已包含 OH 的 CVE 修复
- [ ] libpng_optimize.patch 是否需要移植
- [ ] ARM NEON 优化代码是否兼容
- [ ] BUILD.gn 配置是否需要更新

---

## 文档索引

| 章节 | 内容 | 目标读者 |
|------|------|----------|
| README.md | 快速入门和导航 | 所有人 |
| SUMMARY.md | 阅读路线建议 | 所有人 |
| 01_Overview.md | 原始库功能介绍 | 全体开发者 |
| 02_Patches.md | **OH Patch 详细分析** | 集成工程师 |
| 03_Build_Integration.md | 构建配置说明 | 集成工程师 |
| 04_Usage_in_OH.md | OH 使用场景 | 全体开发者 |
| 05_API_Differences.md | API 差异 | 全体开发者 |
| 06_Security.md | 安全风险分析 | 安全工程师 |
| _work/ASSESSMENT.md | 项目评估原始数据 | 文档维护者 |
