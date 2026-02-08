# 元能力仓颉封装（ability_cangjie_wrapper）Wiki

## 概述

本文档为 OpenHarmony `ability_cangjie_wrapper` 子系统的工程 Wiki，提供完整的架构说明、API 文档、构建配置和安全分析。

**项目定位**：基于 OpenHarmony Ability 子系统的仓颉（Cangjie）语言封装，提供 UIAbility、应用上下文、组件管理器等核心能力的仓颉 API。

**关键信息**：
- 版本：6.1
- 许可证：Apache License 2.0
- 适配系统：standard（仅标准设备）
- ROM：1500KB | RAM：1536KB
- API 起始版本：22
- 系统能力：`SystemCapability.Ability.AbilityRuntime.Core`

## 覆盖范围

### 本 Wiki 包含

1. **项目概览与架构**：模块职责、依赖关系、线程模型
2. **N-API 导出清单**：所有仓颉 API 的完整清单（含参数、返回值、错误码）
3. **GN 构建配置**：targets 列表、依赖关系、编译产物
4. **安全风险评审**：攻击面识别、信任边界、漏洞分析与修复建议
5. **常见问题**：构建、运行、调试问题定位

### 本 Wiki 不包含

- 测试代码相关内容（test/ 目录）
- 未在本仓库实现的 API（依赖其他子系统实现的功能）
- 运行时错误调试（请参考官方文档）

## 文档更新方式

当代码发生变更时，需同步更新相关 Wiki 章节：

| 变更类型 | 更新章节 | 更新优先级 |
|---------|---------|-----------|
| 新增/删除模块 | 模块职责、API 清单、SUMMARY | P0 |
| API 签名变更 | API 清单表、调用链 | P0 |
| 构建配置变更 | GN Targets、编译产物 | P1 |
| 安全相关变更 | 安全风险评审 | P0 |

### 更新检查清单

- [ ] API 清单表与源码一致
- [ ] 路径引用已更新
- [ ] 调用链图示准确
- [ ] 安全分析覆盖新增攻击面
- [ ] SUMMARY.md 链接验证通过

## 相关链接

### 内部文档

- [README.md](../README.md) - 项目英文说明
- [README_zh.md](../README_zh.md) - 项目中文说明
- [SUMMARY.md](SUMMARY.md) - 全站导航
- [架构说明](02_Architecture.md) - 组件图与数据流
- [API 文档](03_APIs.md) - N-API 完整清单
- [构建配置](04_Build.md) - GN targets 与产物
- [安全评审](05_Security.md) - 风险分析与修复建议

### 外部资源

- [仓颉元能力 API 文档](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/tree/master/doc/API_Reference/source_zh_cn/apis/AbilityKit)
- [程序框架服务开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_zh_cn/application-models/cj-abilitykit-overview.md)
- [Ability Runtime 源码](https://gitcode.com/openharmony/ability_ability_runtime)
- [Access Token 源码](https://gitcode.com/openharmony/security_access_token)

## 生成信息

- **生成时间**：2026-02-06
- **源码版本**：当前仓库 HEAD
- **生成工具**：OpenHarmony 工程 Wiki 生成 Agent
