# OpenHarmony Contacts 应用 Wiki

## 文档概述

本文档为 OpenHarmony Contacts（联系人）应用的工程 Wiki，旨在帮助开发者快速理解项目结构、架构设计、API 使用方式及安全风险。

## 项目定位

**Contacts** 是 OpenHarmony 标准系统中预置的系统应用，提供联系人管理、通话记录、拨号盘等核心功能。

- **技术栈**: ArkTS + ArkUI
- **系统类型**: OpenHarmony 标准系统 (standard)
- **子系统**: applications
- **包名**: @ohos/contacts
- **版本**: 3.0
- **许可证**: Apache 2.0

## 文档覆盖范围

### ✅ 已覆盖

| 模块 | 状态 | 说明 |
|------|------|------|
| 项目概览 | ✅ | 定位、功能、技术栈 |
| 架构说明 | ✅ | MVP + 领域驱动设计 |
| 模块结构 | ✅ | 各模块职责划分 |
| 构建系统 | ✅ | hvigor 构建配置 |
| 安全评审 | ✅ | 权限、风险分析 |
| API 参考 | ✅ | Native API 调用模式 |

### ⚠️ 不适用

- **N-API 文档**: 本项目为纯 ArkTS 应用，不提供 C/C++ 原生接口
- **GN 构建**: 使用 hvigor 而非 GN 构建系统

## 文档更新方式

本文档基于代码自动生成，代码更新后需重新运行文档生成脚本。

### 手动更新

```bash
# 文档生成位置
wiki/
├── README.md           # 本文档
├── SUMMARY.md          # 全站导航
├── 00_Overview.md     # 项目概览
├── 01_Architecture.md  # 架构说明
├── 02_Module_Structure.md  # 模块结构
├── 03_API_Reference.md    # API 参考
├── 04_Build_System.md     # 构建系统
├── 05_Security_Review.md  # 安全评审
├── 06_Troubleshooting.md  # 问题排查
└── _work/              # 生成工作区
```

## 相关仓库

- [applications_mms](https://gitee.com/openharmony/applications_mms) - 短信应用
- [applications_contactsdata](https://gitee.com/openharmony/applications_contactsdata) - 联系人数据提供方
- [applications_call](https://gitee.com/openharmony/applications_call) - 通话应用

## 快速开始

1. **新人阅读顺序**: 建议按 `SUMMARY.md` 导航顺序阅读
2. **开发者参考**: 直接跳转至对应模块的 API 参考
3. **安全评审**: 部署前请阅读安全评审章节

## 问题反馈

如发现文档错误或遗漏，请提交 Issue 或 PR。

---

**文档生成时间**: 2026-02-05
