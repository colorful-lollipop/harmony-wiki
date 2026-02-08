# Appverify Wiki 文档

> 生成时间：2026-02-06
> 仓库：`/base/security/appverify`
> 子系统：security

## 文档覆盖范围

本文档系统性地描述 OpenHarmony **应用完整性校验 (appverify)** 模块的架构、实现、API 和安全特性。

### 已覆盖内容

- [x] 项目定位与核心能力
- [x] 目录结构与模块职责
- [x] 架构说明（数据流、验证流程）
- [x] 对外 C++ API（无 JS API）
- [x] 内部模块接口
- [x] GN 构建目标与编译产物
- [x] 安全风险评审
- [x] 常见问题与定位路径

### 未覆盖内容

- 具体算法实现细节（如 RSA-PSS 具体数学公式）
- 签名工具链（应用签名侧的 hap-sign-tool 等）
- 依赖的 OpenSSL 内部机制
- LiteOS-M 版本实现细节（重点关注标准版本）

## 文档更新方式

### 代码更新后的维护流程

1. **API 变更**：更新 `04_Public_API.md` 和相关头文件引用
2. **新增功能**：在相应章节补充，更新 `SUMMARY.md` 导航
3. **安全问题修复**：在 `08_Security_Review.md` 中标注修复状态
4. **构建配置变更**：更新 `06_GN_Targets.md` 和 `07_Build_Artifacts.md`

### 自动化建议

建议添加以下检查脚本：
- 头文件 API 声明与文档一致性检查
- BUILD.gn targets 列表与文档对比验证
- 配置文件格式与示例对比

## 新人阅读路线

按照以下顺序阅读可快速理解项目：

1. [00_Overview.md](00_Overview.md) - 项目概览（5 分钟）
2. [01_Project_Position.md](01_Project_Position.md) - 项目定位与边界（10 分钟）
3. [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构（10 分钟）
4. [03_Architecture.md](03_Architecture.md) - 架构与验证流程（20 分钟）
5. [04_Public_API.md](04_Public_API.md) - 对外 API 使用（15 分钟）
6. [08_Security_Review.md](08_Security_Review.md) - 安全机制理解（20 分钟）

**深入实现**（开发者/安全审计）：
- [05_Internal_API.md](05_Internal_API.md) - 内部模块接口
- [06_GN_Targets.md](06_GN_Targets.md) - 构建系统详解
- [07_Build_Artifacts.md](07_Build_Artifacts.md) - 编译产物说明
- [09_FAQ.md](09_FAQ.md) - 常见问题排查

## 快速索引

| 主题 | 文档位置 |
|------|----------|
| 如何调用 HapVerify | [04_Public_API.md](04_Public_API.md#api-使用示例) |
| 签名验证流程 | [03_Architecture.md](03_Architecture.md#签名验证流程) |
| 可信源配置 | [06_GN_Targets.md](06_GN_Targets.md#配置文件-预构建目标) |
| 企业应用验证 | [05_Internal_API.md](05_Internal_API.md#企业重签名管理-enterprisere-signmgr) |
| 安全风险清单 | [08_Security_Review.md](08_Security_Review.md#安全风险清单) |
| 编译失败排查 | [09_FAQ.md](09_FAQ.md#编译相关) |

## 参考资料

- [OpenHarmony 官方文档 - 应用签名](https://docs.openharmony.cn/)
- [bundle.json](../bundle.json) - 组件元数据
- [BUILD.gn](../BUILD.gn) - 根构建文件
- [README_zh.md](../README_zh.md) - 项目 README
