# 文档导航

本文档提供划词服务子系统 Wiki 的完整导航结构。

## 快速开始

### 新人必读

建议按照以下顺序阅读：

```
📖 1. [README](README.md) → 了解文档覆盖范围和使用指南
📖 2. [01_Project_Overview.md](01_Project_Overview.md) → 项目定位与核心能力
📖 3. [02_Architecture.md](02_Architecture.md) → 系统架构与设计
📖 4. [03_NAPI_Reference.md](03_NAPI_Reference.md) → JavaScript API 使用
```

## 完整目录

### 核心文档

| 章节 | 文档 | 描述 | 大小 |
|-----|------|------|------|
| 📘 | [README](README.md) | 文档概览、更新方式、贡献指南 | 5KB |
| 📗 | [01_Project_Overview.md](01_Project_Overview.md) | 项目定位、边界、核心能力、运行环境 | - |
| 📕 | [02_Architecture.md](02_Architecture.md) | 组件图、数据流、线程模型、关键时序 | 21KB |
| 📙 | [03_NAPI_Reference.md](03_NAPI_Reference.md) | N-API 清单、参数校验、错误码、调用链路 | 13KB |
| 📓 | [04_Build_System.md](04_Build_System.md) | GN Targets、编译产物、运行时加载关系 | 14KB |
| 📕 | [05_AttackSurface.md](05_AttackSurface.md) | 攻击面速查、信任边界、攻击向量 | - |
| 📕 | [05_Security_Review.md](05_Security_Review.md) | 深度安全分析、风险点、修复建议 | 23KB |
| 📘 | [06_Inner_API.md](06_Inner_API.md) | 内部 C++ API、头文件、使用指南 | 8KB |

### 工作文档

| 章节 | 文档 | 描述 |
|-----|------|------|
| 📋 | [_work/NOTES.md](_work/NOTES.md) | 代码证据收集、事实记录 |
| 📋 | [_work/PLAN.md](_work/PLAN.md) | 任务分解、进度追踪 |

## 按角色导航

### 应用开发者

如果您想开发划词扩展应用：

```
1. [README](README.md) → 了解文档结构
2. [01_Project_Overview.md](01_Project_Overview.md) → 理解划词能力
3. [03_NAPI_Reference.md](03_NAPI_Reference.md) → 学习 API 使用
4. 访问官方文档 → 查看使用示例
```

**关键 API**:
- `selectionManager.on()` - 订阅划词事件
- `selectionManager.getSelectionContent()` - 获取选中内容
- `selectionPanel.show()/hide()` - 面板管理

### 系统集成者

如果您需要将划词服务集成到系统：

```
1. [02_Architecture.md](02_Architecture.md) → 理解模块划分
2. [04_Build_System.md](04_Build_System.md) → 了解编译配置
3. [06_Inner_API.md](06_Inner_API.md) → 使用 Inner API
```

**关键配置**:
- SA ID: 8500
- 启动模式: 懒加载 (run-on-create: false)
- 进程名: selection_service

### 安全工程师

如果您需要评估划词服务的安全性：

```
1. [02_Architecture.md](02_Architecture.md) → 理解信任边界
2. [05_AttackSurface.md](05_AttackSurface.md) → 快速了解攻击面
3. [05_Security_Review.md](05_Security_Review.md) → 深度风险分析
```

**重点关注**:
- IPC 接口安全性
- 输入事件来源验证
- 剪贴板数据消毒
- 回调注册身份验证
- 攻击向量与利用路径

### 测试工程师

如果您需要编写测试用例：

```
1. [02_Architecture.md](02_Architecture.md) → 理解数据流
2. [03_NAPI_Reference.md](03_NAPI_Reference.md) → 了解 API 契约
3. [06_Inner_API.md](06_Inner_API.md) → 使用 Inner API 测试
```

## 文档链接索引

### 架构相关

- 系统分层: [02_Architecture.md](02_Architecture.md#1-系统架构总览)
- 模块依赖: [02_Architecture.md](02_Architecture.md#22-模块依赖关系)
- 线程模型: [02_Architecture.md](02_Architecture.md#4-线程模型)
- IPC 接口: [02_Architecture.md](02_Architecture.md#6-ipc-接口定义)

### API 相关

- SelectionManager: [03_NAPI_Reference.md](03_NAPI_Reference.md#2-selectionmanager-模块)
- SelectionPanel: [03_NAPI_Reference.md](03_NAPI_Reference.md#3-selectionpanel-模块)
- SelectionExtensionAbility: [03_NAPI_Reference.md](03_NAPI_Reference.md#4-selectionextensionability-模块)
- SelectionExtensionContext: [03_NAPI_Reference.md](03_NAPI_Reference.md#5-selectionextensioncontext-模块)
- 错误码: [03_NAPI_Reference.md](03_NAPI_Reference.md#6-错误码定义)

### 构建相关

- Targets 清单: [04_Build_System.md](04_Build_System.md#2-targets-清单)
- 编译产物: [04_Build_System.md](04_Build_System.md#4-编译产物清单)
- 运行时加载: [04_Build_System.md](04_Build_System.md#5-运行时加载关系)

### 安全相关

- 攻击面速查: [05_AttackSurface.md](05_AttackSurface.md)
- 威胁模型: [05_Security_Review.md](05_Security_Review.md#1-威胁模型概述)
- 风险清单: [05_Security_Review.md](05_Security_Review.md#31-风险清单)
- 攻击向量: [05_AttackSurface.md](05_AttackSurface.md#8-攻击向量汇总)
- 安全措施: [05_Security_Review.md](05_Security_Review.md#5-安全加固措施)

## 版本历史

| 版本 | 日期 | 更新内容 |
|-----|------|---------|
| 1.0 | 2025-02-06 | 初始版本，包含核心文档 |

## 外部链接

### 官方文档

- [划词服务子系统概述](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/basic-services/selectionInput/selection-services-intro-sys.md)
- [实现划词扩展能力](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/basic-services/selectionInput/selection-services-application-guide-sys.md)
- [SelectionExtensionAbility API](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-basic-services-kit/js-apis-selectionInput-selectionExtensionAbility-sys.md)
- [SelectionManager API](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-basic-services-kit/js-apis-selectionInput-selectionManager-sys.md)
- [SelectionPanel API](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-basic-services-kit/js-apis-selectionInput-selectionPanel-sys.md)

### 代码仓库

- [划词服务主仓库](https://gitee.com/openharmony-sig/systemabilitymgr_selectionfwk)

---

*本文档最后更新: 2025-02-06*
