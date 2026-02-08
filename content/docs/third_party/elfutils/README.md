# elfutils OpenHarmony 文档

elfutils 在 OpenHarmony 中的集成与适配文档。

---

## 文档导航

### 快速开始
- [项目评估](./_work/ASSESSMENT.md) - Phase 0 收集的完整评估信息

### 核心文档
- [01_Overview.md](./01_Overview.md) - 库概览与 OH 中的定位
- [02_Patches.md](./02_Patches.md) - Patch 详细分析（当前无独立 Patch）
- [03_Build_Integration.md](./03_Build_Integration.md) - OH 构建系统适配
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系与使用方式

### 扩展文档
- [05_API_Differences.md](./05_API_Differences.md) - API 差异（暂无）
- [06_Security.md](./06_Security.md) - 安全风险分析

---

## 库信息

**库名称**: elfutils
**OH 版本**: 4.0
**上游版本**: 0.193
**许可证**: LGPL V3.0, GPL V2.0, GPL V3.0
**子系统**: thirdparty
**维护者**: zhanghaibo0@huawei.com

---

## OH 适配概述

elfutils 在 OpenHarmony 中的适配主要通过 **构建系统迁移** 实现，从 autotools 迁移到 GN，同时：
- 仅编译静态库（不编译命令行工具）
- 移除 debuginfod 守护进程
- 添加预生成的 config.h
- 少量源代码 Bugfix

**主要用途**: 为 libabigail 提供 DWARF 调试信息解析能力，用于 ABI 兼容性检查。

---

## 阅读建议

1. **首次了解**: 从 [01_Overview.md](./01_Overview.md) 开始
2. **开发者**: 阅读 [03_Build_Integration.md](./03_Build_Integration.md) 了解构建细节
3. **集成者**: 查看 [04_Usage_in_OH.md](./04_Usage_in_OH.md) 了解依赖关系
4. **深度分析**: 参考 [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) 获取完整评估信息

---

## 上游链接

- 官方网站: https://sourceware.org/elfutils/
- 源码仓库: git://sourceware.org/git/elfutils.git
- Bug 跟踪: https://sourceware.org/bugzilla/
- 邮件列表: elfutils-devel@sourceware.org

---

## 更新日志

- **2026-02-08**: 初始版本创建
