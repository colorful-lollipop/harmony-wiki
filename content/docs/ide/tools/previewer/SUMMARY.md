# 文档导航

## 新人学习路线

**目标**: 在 30 分钟内理解项目架构、核心功能和开发流程

```
1. 概览 (5 分钟) → 2. 目录结构 (5 分钟) → 3. 架构设计 (10 分钟) 
   ↓
4. 通信协议 (5 分钟) → 5. 内部 API (5 分钟)
```

**学习路径**:
1. 📖 [概览](00_Overview.md) - 了解 Previewer 是什么、能做什么
2. 📁 [目录结构](01_Directory_Structure.md) - 熟悉代码组织
3. 🏗️ [架构设计](02_Architecture.md) - 理解组件交互和数据流
4. 🔌 [通信协议](03_Communication_Protocol.md) - 了解如何与 DevEco Studio 通信
5. 📚 [内部 API](04_Inner_API.md) - 学习可用的工具接口
6. 🔧 [构建系统](05_Build_System.md) - 了解如何编译项目
7. 🎯 [Mock 能力](08_Mock_Capabilities.md) - 学习如何扩展 Mock 功能

**进阶学习**:
8. ⚙️ [命令行参数](09_Command_Line_Params.md) - 深入了解所有参数
9. 🚀 [开发指南](10_Development_Guide.md) - 学习如何贡献代码

---

## 安全研究路线

**目标**: 快速识别攻击面、信任边界和潜在漏洞

```
1. 安全评审 (10 分钟) → 2. 通信协议 (10 分钟) → 3. 架构分析 (15 分钟)
   ↓
4. Mock 层审计 (10 分钟) → 5. 代码证据 (5 分钟)
```

**审计路径**:
1. 🛡️ [安全评审](07_Security_Review.md) - 识别所有安全风险
2. 🔌 [通信协议](03_Communication_Protocol.md) - 分析输入验证和认证机制
3. 🏗️ [架构设计](02_Architecture.md) - 识别信任边界和数据流
4. 🎭 [Mock 能力](08_Mock_Capabilities.md) - 审查模拟实现的安全性
5. 🔧 [构建系统](05_Build_System.md) - 检查编译安全和依赖

**深度审计**:
6. 📊 [代码证据](_work/NOTES.md) - 查看所有代码证据
7. 📋 [项目评估](_work/ASSESSMENT.md) - 了解安全需求分析
8. 📝 [工作计划](_work/PLAN.md) - 查看任务进度和待办

---

## 全站文档

### 快速入门
- [README](README.md) - 项目说明与使用指南
- [项目评估](_work/ASSESSMENT.md) - 项目画像与受众分析
- [工作计划](_work/PLAN.md) - Wiki 生成任务追踪

### 核心文档
- [概览](00_Overview.md) - 项目定位、核心能力、运行环境
- [目录结构](01_Directory_Structure.md) - 模块划分与职责说明
- [架构设计](02_Architecture.md) - 组件图、数据流、线程模型

### 技术细节
- [通信协议](03_Communication_Protocol.md) - 命名管道、WebSocket、JSON 格式
- [内部 API](04_Inner_API.md) - Inner Kit 与模块接口
- [构建系统](05_Build_System.md) - GN Targets 与编译配置
- [编译产物](06_Artifacts.md) - 二进制产物与安装路径

### 扩展文档
- [Mock 能力](08_Mock_Capabilities.md) - Mock 层设计与扩展指南
- [命令行参数](09_Command_Line_Params.md) - 完整参数列表与说明
- [开发指南](10_Development_Guide.md) - 构建、测试、调试指南

### 安全与运维
- [安全评审](07_Security_Review.md) - 攻击面分析与风险评估
- [常见问题](appendix/FAQ.md) - 构建与运行时问题

### 工作文档
- [代码证据](_work/NOTES.md) - 代码证据汇总（技术结论追溯）
- [项目评估](_work/ASSESSMENT.md) - 项目类型判定与文档策略

---

## 代码证据索引

| 主题 | 证据位置 |
|------|----------|
| **入口文件** | `RichPreviewer.cpp:99`, `ThinPreviewer.cpp:100` |
| **命令处理** | `cli/CommandLineInterface.cpp:73` |
| **WebSocket** | `util/WebSocketServer.h:25` |
| **命名管道** | `util/LocalSocket.h:22` |
| **Inner Kit** | `bundle.json:53-77` |
| **内部 Kit 头文件** | `util/KeyboardHelper.h:22`, `util/ClipboardHelper.h:22` |
|  | `jsapp/rich/external/EventRunner.h:25` |
|  | `jsapp/rich/external/EventHandler.h:22` |
|  | `jsapp/rich/external/StageContext.h:35` |
|  | `jsapp/rich/external/JsMockUtil.h:23` |
| **安全风险** | `automock/mock-generate/build.js:17-33` (CRITICAL) |
| **安全风险** | `automock/src/common/systemUtils.ts:23-31` (HIGH) |
| **安全风险** | `automock/src/main.ts:68-81` (HIGH) |
| **安全风险** | `automock/src/generate/generateContent.ts:708` (MEDIUM) |

---

## 模块索引

| 模块 | 路径 | 职责 | 文档 |
|------|------|------|------|
| **CLI** | `cli/` | 命令解析与处理 | [通信协议](03_Communication_Protocol.md) |
| **JSAPP** | `jsapp/` | 渲染引擎调用 | [架构设计](02_Architecture.md) |
| **MOCK** | `mock/` | 交互层模拟 | [Mock 能力](08_Mock_Capabilities.md) |
| **UTIL** | `util/` | 工具与基础设施 | [内部 API](04_Inner_API.md) |
| **GN** | `gn/` | 构建配置 | [构建系统](05_Build_System.md) |
| **AUTOMOCK** | `automock/` | 自动化 Mock 生成 | [安全评审](07_Security_Review.md) |

---

## 安全风险索引

| 风险 | 级别 | 模块 | 文档 |
|------|------|------|------|
| **命令注入** | CRITICAL | automock | [安全评审 R1](07_Security_Review.md#critical-命令注入风险-automock-模块) |
| **路径遍历** | HIGH | StageContext | [安全评审:HIGH:路径遍历漏洞](07_Security_Review.md#high-路径遍历漏洞) |
| **路径遍历** | HIGH | automock | [安全评审 R2](07_Security_Review.md#high-路径遍历风险---systemutils-automock-模块) |
| **Zip Slip** | HIGH | StageContext | [安全评审:HIGH:Zip-Slip-漏洞](07_Security_Review.md#high-zip-slip-漏洞) |
| **WebSocket SID 绕过** | HIGH | WebSocketServer | [安全评审:HIGH:WebSocket-SID-认证绕过](07_Security_Review.md#high-websocket-sid-认证绕过) |
| **命名管道无认证** | HIGH | CommandLineInterface | [安全评审:HIGH:命名管道无认证](07_Security_Review.md#high-命名管道无认证) |
| **动态代码执行** | MEDIUM | automock | [安全评审 R4](07_Security_Review.md#medium-动态代码执行风险---function-构造-automock-模块) |
| **代码注入** | MEDIUM | automock | [安全评审 R5](07_Security_Review.md#medium-代码注入风险---字符串模板-automock-模块) |
| **JSON 解析无限制** | MEDIUM | JsonReader | [安全评审:MEDIUM:JSON-解析无限制](07_Security_Review.md#medium-json-解析无限制) |
| **TOCTOU** | MEDIUM | FileSystem | [安全评审:MEDIUM:TOCTOU-竞态条件](07_Security_Review.md#medium-toctou-竞态条件) |

---

## 版本信息

- **当前版本**: 3.1
- **生成时间**: 2026-02-07
- **更新**: 补充 Mock 能力文档、更新安全风险评估
- **Agent**: Sisyphus

---

## 快速查找

**按主题查找**:
- 🎭 **Mock 功能** → [Mock 能力](08_Mock_Capabilities.md)
- 🔌 **通信机制** → [通信协议](03_Communication_Protocol.md)
- 🏗️ **架构设计** → [架构设计](02_Architecture.md)
- 🛡️ **安全风险** → [安全评审](07_Security_Review.md)
- 🔧 **构建编译** → [构建系统](05_Build_System.md)

**按角色查找**:
- 📚 **新人学习** → [新人学习路线](#新人学习路线)
- 🔍 **安全研究** → [安全研究路线](#安全研究路线)
- 🚀 **开发者** → [开发指南](10_Development_Guide.md)
- 📋 **文档维护** → [工作计划](_work/PLAN.md)

---

**建议**: 新人按"新人学习路线"顺序阅读，安全研究员按"安全研究路线"顺序阅读。
