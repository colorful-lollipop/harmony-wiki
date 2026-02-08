# 阅读指南

本文档旨在帮助开发者快速理解 qrcodegen 库在 OpenHarmony 中的集成与适配。以下是针对不同场景的推荐阅读顺序。

## 场景一：仅使用 QRCode 组件

如果你是 ArkUI 开发者，只需要使用 QRCode 组件生成二维码：

**推荐阅读**：[04_在 OH 中的使用](04_Usage_in_OH.md)

了解：
- QRCode 组件如何调用本库
- 二维码生成的基本流程
- 错误处理方式

**跳过**：`02_适配分析`、`05_API 差异`（技术细节）

## 场景二：修改 QRCode 组件

如果你需要修改 AceEngine 中的 QRCode 相关代码：

**推荐阅读顺序**：
1. [01_概述](01_Overview.md) - 了解库的基本功能
2. [04_在 OH 中的使用](04_Usage_in_OH.md) - 了解组件如何使用本库
3. [02_适配分析](02_Patches.md) - 理解 OH 特有的代码分支
4. [03_构建适配](03_Build_Integration.md) - 了解构建配置

## 场景三：升级上游版本

如果你需要将 qrcodegen 升级到上游新版本：

**推荐阅读顺序**：
1. [02_适配分析](02_Patches.md) - **必读**，了解所有 OH 适配点
2. [05_API 差异](05_API_Differences.md) - 了解 API 变化影响
3. [06_安全风险](06_Security.md) - 检查安全更新

**关键检查清单**：
- [ ] `ACE_ENGINE_QRCODE_ABLE` 条件编译块是否适用
- [ ] 异常处理改动的兼容性
- [ ] API 签名变化

## 场景四：贡献代码到上游

如果你希望将 OH 适配推向上游：

**推荐阅读**：
1. [02_适配分析](02_Patches.md) - 整理可上游化的改动
2. [05_API 差异](05_API_Differences.md) - 分析 API 设计

## 文档结构速查

| 文档 | 目标读者 | 重要性 |
|------|----------|--------|
| README.md | 所有用户 | ⭐ 入口文件 |
| SUMMARY.md | 所有用户 | ⭐ 阅读导航 |
| 01_概述.md | 全体 | ⭐⭐ 基础认知 |
| 02_适配分析.md | 适配者/维护者 | ⭐⭐⭐ **核心文档** |
| 03_构建适配.md | 构建系统开发者 | ⭐⭐ |
| 04_在 OH 中的使用.md | 组件开发者 | ⭐⭐ |
| 05_API 差异.md | API 使用者 | ⭐ |
| 06_安全风险.md | 安全审查者 | ⭐ |
| _work/ASSESSMENT.md | 维护者 | ⭐⭐ 参考资料 |

## 关键概念

### ACE_ENGINE_QRCODE_ABLE

这是本库 OH 适配的核心宏定义，用于区分：
- **定义时**：使用 OH 定制代码（无异常、`flag` 错误处理）
- **未定义时**：使用上游原始代码（异常机制）

### 构建目标

| 目标名称 | 用途 |
|----------|------|
| `qrcodegen_static` | 通用静态库，供非 AceEngine 使用 |
| `ace_engine_qrcode` | AceEngine 专用静态库，定义 `ACE_ENGINE_QRCODE_ABLE` |

## 常见问题

**Q: 为什么没有 `.patch` 文件？**

A: 本库的 OH 适配采用**条件编译**方式，适配代码直接嵌入源文件。这种方式便于维护，无需单独管理 patch 文件，但升级上游时需重新检查适配兼容性。

**Q: 遇到二维码生成失败怎么办？**

A: 使用 `qrCode.getFlag()` 检查生成状态，返回 `false` 表示生成失败。常见原因：输入数据过长、版本超出范围等。

**Q: 如何调试二维码生成问题？**

A: 参考 [QRCode 知识库](foundation/arkui/ace_engine/docs/pattern/qrcode/QRCode_Knowledge_Base.md) 中的调试指南。
