# 分布式包管理服务 (DBMS) - Wiki 导航

本文档提供了分布式包管理服务 (DBMS) 的完整导航索引，帮助你快速定位所需信息。

---

## 快速导航

### 新人入门（推荐阅读顺序）

1. 📖 [README](./README.md) - Wiki 使用指南
2. 📋 [01_Overview](./01_Overview.md) - 项目定位与核心能力
3. 📁 [02_Directory_Structure](./02_Directory_Structure.md) - 目录结构与模块职责
4. 🏗️ [03_Architecture](./03_Architecture.md) - 架构说明与数据流
5. 🌐 [04_JS_API](./04_JS_API.md) - 对外 JS API 参考

### 深入开发

6. 🔌 [05_Inner_API](./05_Inner_API.md) - 内部 API 与接口稳定性
7. 🔨 [06_GN_Build](./06_GN_Build.md) - GN Targets 与编译产物
8. 🛡️ [07_Security_Analysis](./07_Security_Analysis.md) - 安全风险评审

### 附录参考

9. 📊 [appendix/Callgraphs](./appendix/Callgraphs.md) - 关键调用链分析
10. ⚙️ [appendix/Config_Flags](./appendix/Config_Flags.md) - 配置宏与 Feature Flags

---

## 详细文档目录

### 核心文档

| 文档 | 描述 | 关键内容 | 篇幅 |
|------|------|----------|--------|
| [README](./README.md) | Wiki 总览和使用指南 | 覆盖范围、更新方式 | 短 |
| [01_Overview](./01_Overview.md) | 项目概览 | 定位、能力、运行环境 | 中 |
| [02_Directory_Structure](./02_Directory_Structure.md) | 目录结构 | 模块职责、文件清单 | 中 |
| [03_Architecture](./03_Architecture.md) | 架构设计 | 组件图、数据流、线程模型 | 长 |
| [04_JS_API](./04_JS_API.md) | JS API 参考 | API 清单、参数、调用链 | 长 |
| [05_Inner_API](./05_Inner_API.md) | 内部 API | 模块接口、依赖关系 | 中 |
| [06_GN_Build](./06_GN_Build.md) | 构建系统 | Targets、产物、依赖 | 长 |
| [07_Security_Analysis](./07_Security_Analysis.md) | 安全评审 | 攻击面、风险点、修复建议 | 长 |
| [08_FAQ](./08_FAQ.md) | 常见问题 | 构建调试、问题定位 | 中 |

### 附录文档

| 文档 | 描述 | 用途 |
|------|------|------|
| [appendix/Callgraphs](./appendix/Callgraphs.md) | 调用链分析 | 理解代码执行路径 |
| [appendix/Config_Flags](./appendix/Config_Flags.md) | 配置说明 | 定制编译选项 |

---

## 按主题查找

### 想了解...

**...项目的定位和功能？**
→ 阅读 [01_Overview](./01_Overview.md)

**...代码组织结构？**
→ 阅读 [02_Directory_Structure](./02_Directory_Structure.md)

**...架构设计和组件关系？**
→ 阅读 [03_Architecture](./03_Architecture.md)

**...如何使用 JS API？**
→ 阅读 [04_JS_API](./04_JS_API.md)

**...内部接口和依赖？**
→ 阅读 [05_Inner_API](./05_Inner_API.md)

**...如何构建和编译？**
→ 阅读 [06_GN_Build](./06_GN_Build.md)

**...安全风险和防护？**
→ 阅读 [07_Security_Analysis](./07_Security_Analysis.md)

**...常见问题？**
→ 阅读 [08_FAQ](./08_FAQ.md)

**...代码执行流程？**
→ 阅读 [appendix/Callgraphs](./appendix/Callgraphs.md)

**...配置选项？**
→ 阅读 [appendix/Config_Flags](./appendix/Config_Flags.md)

---

## 证据引用规范

本文档遵循以下证据引用规范：

- **文件路径**: `interfaces/kits/js/distributedBundle/native_module.cpp`
- **文件+行号**: `services/dbms/src/distributed_bms.cpp:638-651`
- **符号引用**: `VerifyCallingPermission()`
- **未确认内容**: `TODO(需确认)`

---

## 文档维护

### 更新时机

建议在以下情况更新对应文档：

- **代码重构后**：更新架构和 API 文档
- **新增 API 后**：更新 JS API 参考
- **安全漏洞修复后**：更新安全评审文档
- **构建系统变更后**：更新 GN Build 文档

### 质量标准

所有文档遵循以下质量标准：

- ✅ 每个结论都有代码证据支持
- ✅ 路径和符号可追溯到实际代码
- ✅ 术语统一，避免混淆
- ✅ 链接完整，无死链
- ✅ 排除测试相关内容（除非作为验证依据）

---

**最后更新**: 2026-02-06
