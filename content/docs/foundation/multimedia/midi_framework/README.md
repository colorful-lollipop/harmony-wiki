# OpenHarmony MIDI Framework Wiki

> **文档版本**: 1.0
> **最后更新**: 2025-02-06
> **项目版本**: 6.1

## 文档覆盖范围

本 Wiki 全面文档化 OpenHarmony Multimedia MIDI Framework 项目，包括：

### 包含内容

| 文档 | 描述 |
|------|------|
| [项目概览](01_Overview.md) | 项目定位、核心能力、运行环境 |
| [架构说明](02_Architecture.md) | 组件图、数据流、线程模型、时序图 |
| [NDK C API 参考](03_NDK_API.md) | 完整 API 清单、参数说明、错误码 |
| [模块详解](04_Module_Details.md) | 框架层、服务层、内部 API |
| [构建系统](05_Build_System.md) | GN Targets、编译产物、依赖关系 |
| [安全评审](06_Security.md) | 攻击面分析、风险识别、修复建议 |
| [附录](appendix/) | 调用链、配置参数等补充信息 |

### 不包含内容

- 测试相关内容（test/ 目录）
- 已废弃或未使用的 API
- 第三方依赖库的内部实现

## 文档更新方式

### 何时更新文档

当发生以下变更时，应同步更新 Wiki：

1. **新增 API** - 在 `03_NDK_API.md` 添加新接口文档
2. **修改现有 API** - 更新对应接口的参数、返回值、错误码说明
3. **架构变更** - 更新架构图、模块职责、调用链
4. **新增模块** - 在 `04_Module_Details.md` 添加模块说明
5. **构建配置变更** - 更新 `05_Build_System.md` 的 targets 列表
6. **安全相关变更** - 更新 `06_Security.md` 的风险分析

### 更新步骤

```bash
# 1. 克隆或拉取最新 Wiki 仓库
git pull origin main

# 2. 编辑相关文档
#    - 修改对应 .md 文件
#    - 添加代码证据引用（文件路径 + 行号）

# 3. 验证文档质量
#    - 检查链接有效性
#    - 确认术语统一
#    - 验证代码证据引用

# 4. 提交变更
git add wiki/
git commit -m "docs: 更新 MIDI Framework Wiki [变更说明]"
git push
```

### 文档质量检查清单

在提交前，请确认：

- [ ] 所有关键结论有代码证据支持（路径 + 符号）
- [ ] API 文档包含完整的参数、返回值、错误码说明
- [ ] 架构图和时序图为最新版本
- [ ] 链接到其他文档的引用正确
- [ ] 术语使用一致
- [ ] 无测试相关内容引用

## 快速导航

### 新人阅读顺序

1. **[项目概览](01_Overview.md)** - 理解项目定位和核心能力
2. **[NDK C API 参考](03_NDK_API.md)** - 了解如何使用 API
3. **[架构说明](02_Architecture.md)** - 理解内部工作机制
4. **[构建系统](05_Build_System.md)** - 了解如何编译项目

### 开发者参考

| 需求 | 文档 |
|------|------|
| 使用 MIDI API | [NDK C API 参考](03_NDK_API.md) |
| 添加新 API | [模块详解 - NDK 封装层](04_Module_Details.md#ndk-封装层) |
| 修改服务层 | [模块详解 - 服务层](04_Module_Details.md#服务层) |
| 调试问题 | [架构说明 - 线程模型](02_Architecture.md#线程模型) |
| 安全问题 | [安全评审](06_Security.md) |
| 编译项目 | [构建系统](05_Build_System.md) |

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2025-02-06 | 初始版本 |

## 贡献者

感谢以下贡献者完善本 Wiki：

- 文档生成工具

---

*本 Wiki 由 OpenHarmony MIDI Framework 团队维护*
