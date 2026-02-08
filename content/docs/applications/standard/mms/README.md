# OpenHarmony MMS Wiki

## 文档概述

本文档是 OpenHarmony MMS（信息应用）的工程 Wiki，旨在帮助开发者快速理解项目架构、关键实现和安全注意事项。

### 覆盖范围

| 模块 | 覆盖状态 | 说明 |
|------|----------|------|
| 项目概览 | ✅ | 定位、功能、运行环境 |
| 架构设计 | ✅ | 组件图、数据流、线程模型 |
| 系统 API | ✅ | 电话、短信、数据存储等 |
| 安全评审 | ✅ | 攻击面、风险点、修复建议 |
| 构建产物 | ✅ | HAP 包、配置、签名 |
| 常见问题 | ✅ | 调试、定位路径 |

### 未覆盖范围

- 测试代码 (`test/` 目录下的内容)
- 详细 UI 设计规范
- 性能优化细节
- 运营商特定配置

### 更新方式

本文档基于代码分析自动生成。当代码变更时，建议：
1. 重新运行代码扫描
2. 更新相关章节
3. 验证证据链完整性

### 生成信息

- **生成时间**: 2026-02-05
- **代码版本**: 基于当前工作目录 HEAD
- **文档版本**: v1.0

### 相关资源

- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [电话服务子系统](https://gitee.com/openharmony/telephony_sms_mms)
- [联系人应用](https://gitee.com/openharmony/applications_contacts)

---

## 快速导航

- [项目概览](00_Overview.md) - 开始了解项目
- [架构设计](01_Architecture.md) - 技术架构详解
- [目录结构](02_DirectoryStructure.md) - 代码组织方式
- [系统API](03_SystemAPIs.md) - 外部接口清单
- [数据流](04_DataFlow.md) - 数据流转图
- [安全评审](05_SecurityReview.md) - 风险分析
- [构建产物](06_BuildArtifacts.md) - 编译输出
- [问题排查](07_Troubleshooting.md) - 调试指南

---

## 新人阅读路线

### 路线 1: 快速了解 (30 分钟)
1. [项目概览](00_Overview.md)
2. [目录结构](02_DirectoryStructure.md)
3. [架构设计](01_Architecture.md) 中的架构图

### 路线 2: 开发上手 (2 小时)
1. 路线 1 全部内容
2. [系统API](03_SystemAPIs.md)
3. [数据流](04_DataFlow.md)
4. 查看关键代码文件

### 路线 3: 安全审计 (3 小时)
1. 路线 1 全部内容
2. [安全评审](05_SecurityReview.md)
3. 查看调用链附录
4. 验证风险点

---

*本文档遵循 Apache 2.0 许可证*
