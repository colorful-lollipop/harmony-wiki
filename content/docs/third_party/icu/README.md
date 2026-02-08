# ICU (International Components for Unicode) - OpenHarmony 适配文档

## 简介

本文档详细记录了 ICU 库在 OpenHarmony 中的集成与适配情况。ICU 是一套成熟的 Unicode 支持、软件国际化和全球化 (i18n/g11n) 的 C/C++ 和 Java 库。

## 文档导航

| 文档 | 内容 |
|------|------|
| [01_Overview.md](./01_Overview.md) | ICU 库概览与在 OH 中的定位 |
| [02_Patches.md](./02_Patches.md) | OH 适配分析 (源码级修改) |
| [03_Build_Integration.md](./03_Build_Integration.md) | 构建系统适配说明 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用场景 |
| [05_API_Differences.md](./05_API_Differences.md) | 新增 API 与行为差异 |
| [06_Security.md](./06_Security.md) | 安全风险分析 |

## 快速信息

| 属性 | 内容 |
|------|------|
| 上游版本 | ICU 74.2 |
| OH 版本 | 3.1 |
| 许可证 | Unicode-DFS-2022 |
| 上游地址 | https://github.com/unicode-org/icu |
| Patch 数量 | 0 (采用源码级适配) |
| OH 特有代码 | `icu4c/source/ohos/` |

## 关键 OH 适配点

1. **数据文件路径适配** - 设置 ICU 数据目录为 `/system/usr/icu`
2. **中国农历支持** - 新增农历日历计算功能
3. **数据裁剪** - 支持精简版数据 (8MB vs 30MB)
4. **NDK 封装** - 提供 Native 应用接口
5. **Java 封装** - 完整的 ICU4J 封装

## 阅读建议

- **开发者**: 从 [01_Overview.md](./01_Overview.md) 开始，了解 ICU 在 OH 中的整体架构
- **系统工程师**: 重点阅读 [03_Build_Integration.md](./03_Build_Integration.md) 了解构建配置
- **应用开发者**: 查看 [04_Usage_in_OH.md](./04_Usage_in_OH.md) 了解如何使用 ICU
- **维护者**: 关注 [02_Patches.md](./02_Patches.md) 和 [06_Security.md](./06_Security.md)

## 相关链接

- [ASSESSMENT.md](./_work/ASSESSMENT.md) - 项目评估详细报告
- [NOTES.md](./_work/NOTES.md) - 分析过程记录
