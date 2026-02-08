# ArkWeb Cangjie Wrapper 工程文档

## 文档概述

本文档是 `arkweb_cangjie_wrapper` 项目的完整工程 Wiki，旨在帮助开发者快速理解项目架构、使用方法、API 接口、构建配置以及安全注意事项。

## 覆盖范围

### 已覆盖内容

| 类别 | 内容 |
|------|------|
| 项目概述 | 项目定位、功能特性、运行环境、版本信息 |
| 目录结构 | 模块划分、文件组织、职责边界 |
| 架构设计 | 组件关系、数据流、FFI 调用模式 |
| API 参考 | WebviewController、WebCookieManager、BackForwardList 完整接口 |
| 构建配置 | GN 编译目标、依赖关系、平台适配 |
| 安全评审 | 攻击面分析、风险点识别、修复建议 |
| 常见问题 | 构建、运行、调试问题定位 |

### 未覆盖内容

| 类别 | 说明 |
|------|------|
| 测试代码 | 测试目录内容不在文档范围内 |
| 外部实现 | webview native 实现细节（位于独立仓库） |
| 完整 API 矩阵 | 仅当前版本开放的功能（Beta） |

## 使用指南

### 新人阅读顺序（推荐）

建议按照以下顺序阅读，以逐步深入理解项目：

1. `README.md` — 本文档，了解文档结构和使用方法
2. `00_Overview.md` — 项目整体定位和核心能力
3. `01_Directory_Structure.md` — 目录结构和模块职责
4. `02_Architecture.md` — 架构设计和组件关系
5. `03_N_API_Reference.md` — 对外 API 接口详解
6. `05_GN_Build.md` — 构建配置和编译产物
7. `07_Security_Review.md` — 安全注意事项

### 导航

使用 `SUMMARY.md` 获取完整导航链接，或直接通过目录跳转到目标章节。

## 文档更新

### 更新时机

当以下变更发生时，应同步更新本文档：

- 新增、删除或修改 API 接口
- 调整构建配置或编译目标
- 发现并修复安全漏洞
- 重大架构变更
- 新增模块或依赖

### 更新方式

1. 定位变更对应的章节
2. 找到相关证据（代码路径、符号定义）
3. 更新对应的小节内容
4. 保持术语一致性和链接有效性
5. 验证所有引用链接存在且正确

## 版本信息

| 项目 | 值 |
|------|-----|
| 项目版本 | 6.1 |
| API Level | 22+ |
| 系统能力 | SystemCapability.Web.Webview.Core |
| ROM 占用 | 约 300KB |
| RAM 占用 | 约 216KB |
| 文档生成时间 | 2025-02-06 |

## 反馈与贡献

如发现文档错误或遗漏，请通过以下方式反馈：

- 在代码仓库提交 Issue
- 完善对应章节后提交 Pull Request
- 遵守项目贡献流程（参考 `README.md`）

## 相关链接

- 项目仓库：[arkweb_cangjie_wrapper](https://gitee.com/openharmony/web_webview)
- API 参考：[ArkWeb Cangjie API](https://gitee.com/openharmony-sig/arkcompiler_cangjie_ark_interop)
- 开发指南：[ArkWeb 开发指南](https://gitee.com/openharmony-sig/arkcompiler_cangjie_ark_interop)
- OpenHarmony 官网：https://www.openharmony.cn/
