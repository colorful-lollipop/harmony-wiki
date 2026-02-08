# 划词服务子系统 Wiki

## 文档概述

本文档集为 OpenHarmony 划词服务子系统 (selectionfwk) 的完整工程 Wiki，旨在帮助开发者快速理解项目架构、API 使用、构建方式和安全考量。

## 覆盖范围

### ✅ 已覆盖内容

| 文档 | 覆盖范围 | 详细程度 |
|-----|---------|---------|
| [概览](01_Project_Overview.md) | 项目定位、核心能力、约束条件 | 完整 |
| [架构说明](02_Architecture.md) | 系统分层、模块划分、数据流、线程模型 | 完整 |
| [N-API 参考](03_NAPI_Reference.md) | 所有 JS API、参数校验、错误码 | 完整 |
| [构建系统](04_Build_System.md) | GN Targets、编译产物、运行时加载 | 完整 |
| [攻击面分析](05_AttackSurface.md) | 攻击面速查、信任边界、攻击向量 | 完整 |
| [安全评审](05_Security_Review.md) | 威胁模型、深度风险分析、修复建议 | 完整 |
| [内部 API](06_Inner_API.md) | Inner C++ API、头文件、使用指南 | 完整 |

### ❌ 未覆盖内容

| 领域 | 未覆盖原因 |
|-----|-----------|
| 测试代码 | 遵循 Wiki 生成规范，不引用测试代码 |
| 第三方依赖内部实现 | 超出子系统分析范围 |
| MMI 驱动层实现 | 属于输入子系统 |
| 剪贴板服务实现 | 属于 pasteboard 子系统 |
| 窗口管理器实现 | 属于 window_manager 子系统 |
| 系统签名机制 | 属于系统级安全机制 |
| 性能测试数据 | 需要运行时测试 |

### ⚠️ 局限性声明

1. **静态分析限制**: 部分逻辑基于静态代码分析，建议运行时验证
2. **并发安全**: 并发场景安全性需要压力测试验证
3. **版本演进**: API 可能随版本更新发生变化，请以官方文档为准
4. **安全评估**: 建议进行专业安全审计

## 文档更新方式

### 建议更新时机

当代码发生以下变更时，应更新对应文档：

| 变更类型 | 应更新文档 |
|---------|-----------|
| 新增/删除/修改 N-API | 03_NAPI_Reference.md |
| 修改模块依赖 | 02_Architecture.md, 04_Build_System.md |
| 新增/修改 GN Target | 04_Build_System.md |
| 修改 SA 配置 | 02_Architecture.md, 04_Build_System.md |
| 修改数据结构 | 02_Architecture.md, 03_NAPI_Reference.md |
| 新增安全风险点 | 05_AttackSurface.md, 05_Security_Review.md |
| 新增 Inner API | 06_Inner_API.md |

### 更新流程

1. **修改源码**: 在对应源码文件中完成修改
2. **更新文档**: 找到相关文档章节，更新内容
3. **证据校验**: 确保关键结论有源码证据支持
4. **链接检查**: 验证 SUMMARY.md 中的链接存在
5. **术语统一**: 检查全文术语一致性

### 文档质量检查清单

更新后请确认：

- [ ] 所有关键结论有源码路径和行号证据
- [ ] API 清单与实际源码一致
- [ ] BUILD.gn 配置与实际文件一致
- [ ] Mermaid 图表能正确渲染
- [ ] 无测试代码引用
- [ ] 链接路径正确
- [ ] 术语使用一致

## 快速导航

### 新人阅读顺序

建议阅读顺序：

```
1. [概览] → 了解项目定位
2. [架构] → 理解系统设计
3. [N-API] → 学习 API 使用
4. [构建] → 了解编译流程
5. [安全] → 了解安全考量
```

### 按角色导航

| 角色 | 推荐文档 |
|-----|---------|
| 应用开发者 | 概览 + N-API + 使用指南 |
| 系统集成者 | 架构 + 构建 + Inner API |
| 安全工程师 | 架构 + 攻击面分析 + 安全评审 |
| 测试工程师 | 架构 + N-API + 构建 |

## 版本信息

| 项目 | 值 |
|-----|------|
| Wiki 版本 | 1.1 |
| 生成时间 | 2026-02-07 |
| 适用版本 | OpenHarmony 5.0+ |
| 维护团队 | selectionfwk 开发者 |

## 贡献指南

### 改进建议

欢迎通过以下方式贡献改进：

1. **Issue 反馈**: 在 OpenHarmony 仓库提交 Issue
2. **文档修正**: 直接提交 PR 修改 Wiki
3. **补充说明**: 添加使用示例或最佳实践

### 代码证据规范

所有关键结论必须包含：

```
1. 文件路径 (必须)
2. 行号 (建议)
3. 关键符号名 (必须)
4. 最小必要代码片段 (建议)
```

示例：
```cpp
// 证据: service/src/selection_service.cpp:56
const bool REGISTER_RESULT = SystemAbility::MakeAndRegisterAbility(
    SelectionService::GetInstance().GetRefPtr());
```

## 相关资源

### 官方文档
- [划词服务子系统概述](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/basic-services/selectionInput/selection-services-intro-sys.md)
- [实现划词扩展能力](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/basic-services/selectionInput/selection-services-application-guide-sys.md)
- [SelectionExtensionAbility API](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-basic-services-kit/js-apis-selectionInput-selectionExtensionAbility-sys.md)
- [SelectionManager API](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-basic-services-kit/js-apis-selectionInput-selectionManager-sys.md)

### 代码仓库
- [划词服务主仓库](https://gitee.com/openharmony-sig/systemabilitymgr_selectionfwk)

## 反馈与支持

如有文档问题，请：

1. 查看 [GitHub Issues](https://github.com/openharmony-sig/systemabilitymgr_selectionfwk/issues)
2. 提交新的 Issue 描述问题
3. 等待社区响应

---

*本文档基于代码证据自动生成，最后更新: 2026-02-07*
