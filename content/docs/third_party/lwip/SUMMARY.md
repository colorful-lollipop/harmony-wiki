# lwIP Wiki 阅读路线

## 推荐阅读顺序

### 🔰 第一次接触 lwIP？

按以下顺序阅读：

1. **[README.md](README.md)** - 快速了解库概览和 OH 适配
2. **[01_Overview.md](01_Overview.md)** - 深入了解 lwIP 功能和定位
3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 了解谁在使用及使用方式

### 🔧 需要集成 lwIP 到项目？

1. **[03_Build_Integration.md](03_Build_Integration.md)** - 构建配置和 GN 集成
2. **[05_API_Differences.md](05_API_Differences.md)** - API 参考（特别是 OH 特有 API）
3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 查看示例和最佳实践

### 🔬 需要深入了解 OH 修改？

1. **[02_Patches.md](02_Patches.md)** - Patch 详细分析
2. **[01_Overview.md](01_Overview.md)** - 回顾 OH 特有功能概述
3. **[05_API_Differences.md](05_API_Differences.md)** - 查看新增 API

### 📊 需要评估升级风险？

1. **[02_Patches.md](02_Patches.md)** - 查看修改范围和回归风险
2. **[wiki/_work/ASSESSMENT.md](_work/ASSESSMENT.md)** - 查看项目评估报告

## 按角色阅读

### 👨‍💻 应用开发者

重点关注：
- [README.md](README.md) - 快速开始
- [03_Build_Integration.md](03_Build_Integration.md) - 构建集成
- [05_API_Differences.md](05_API_Differences.md) - API 参考（标准 API 部分）

### 🏗️ 系统开发者

重点关注：
- [01_Overview.md](01_Overview.md) - 系统定位
- [02_Patches.md](02_Patches.md) - OH 特有功能实现
- [03_Build_Integration.md](03_Build_Integration.md) - 内核集成

### 🔍 代码维护者

重点关注：
- [02_Patches.md](02_Patches.md) - 修改详情
- [wiki/_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 项目评估
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - 依赖关系

## 文档索引

| 文档 | 内容 | 适合读者 |
|------|------|----------|
| [README.md](README.md) | 库概览、快速开始 | 所有读者 |
| [01_Overview.md](01_Overview.md) | 原始库简介、OH 定位 | 新接触者 |
| [02_Patches.md](02_Patches.md) | Patch 分析、修改详情 | 维护者、系统开发者 |
| [03_Build_Integration.md](03_Build_Integration.md) | 构建配置、GN 集成 | 应用开发者 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系、使用场景 | 系统开发者 |
| [05_API_Differences.md](05_API_Differences.md) | API 参考、新增接口 | 应用开发者 |
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估、风险分析 | 维护者 |

## 常见问题

**Q: 为什么找不到 .patch 文件？**
A: OH 的 lwIP 采用条件编译方式管理修改，而非传统 Patch 文件。详见 [02_Patches.md](02_Patches.md)。

**Q: 如何启用低功耗模式？**
A: 在 `lwipopts.h` 中定义 `#define LWIP_LOWPOWER 1`。详见 [05_API_Differences.md](05_API_Differences.md)。

**Q: lwIP 在 OH 中主要被谁使用？**
A: 主要是 dsoftbus（分布式软总线）和内核（LiteOS）。详见 [04_Usage_in_OH.md](04_Usage_in_OH.md)。

**Q: 如何在自己的项目中使用 lwIP？**
A: 在 `BUILD.gn` 中添加 `deps = ["//third_party/lwip:liblwip"]`。详见 [03_Build_Integration.md](03_Build_Integration.md)。
