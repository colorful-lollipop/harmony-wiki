# PrintSpooler Wiki - 全站导航

> 本文档提供 PrintSpooler 项目的完整导航，适合新人按顺序阅读。

---

## 新人推荐阅读顺序

1. **快速入门**（10 分钟）
   - [项目概览](00_Overview.md) - 了解项目定位、核心能力、运行环境
   - [项目定位与边界](01_Project_Scope.md) - 理解项目职责与约束

2. **架构理解**（30 分钟）
   - [目录结构](02_Directory_Structure.md) - 熟悉代码组织
   - [架构设计](03_Architecture.md) - 理解模块关系、数据流、线程模型

3. **接口参考**（45 分钟）
   - [对外 API](04_External_API.md) - OpenHarmony 系统 JS API 使用
   - [内部 API](05_Internal_API.md) - 模块间接口与依赖

4. **构建与部署**（20 分钟）
   - [构建配置](06_GN_Build.md) - Hvigor 构建系统
   - [编译产物](07_Build_Artifacts.md) - HAP/HAR 文件说明

5. **安全评审**（30 分钟）
   - [安全风险评审](08_Security_Review.md) - 攻击面分析与安全建议

6. **问题排查**（按需）
   - [常见问题](09_FAQ.md) - 构建、运行、调试问题

7. **参考资料**（按需）
   - [附录：关键调用链](appendix/Callgraphs.md) - 重要流程梳理
   - [附录：关键配置](appendix/Config_Flags.md) - 配置参数速查

---

## 文档目录

### 核心文档

- [首页/概览](00_Overview.md)
  - 目的：提供项目的快速概览
  - 适用范围：所有开发者
  - 关键结论：项目定位、核心能力、技术栈

- [项目定位与边界](01_Project_Scope.md)
  - 目的：定义项目职责与限制
  - 适用范围：架构设计和功能开发
  - 关键结论：支持的功能范围、约束条件

- [目录结构与模块职责](02_Directory_Structure.md)
  - 目的：说明代码组织结构
  - 适用范围：代码导航和模块定位
  - 关键结论：模块划分、文件组织

- [架构说明](03_Architecture.md)
  - 目的：描述系统架构设计
  - 适用范围：架构理解和技术选型
  - 关键结论：组件关系、数据流、线程模型、时序图

### API 文档

- [对外 API](04_External_API.md)
  - 目的：列出使用的 OpenHarmony 系统 JS API
  - 适用范围：API 使用和集成
  - 关键结论：API 清单、参数说明、调用模式

- [内部 API](05_Internal_API.md)
  - 目的：描述模块间接口
  - 适用范围：模块开发和维护
  - 关键结论：接口定义、依赖关系、稳定性标注

### 构建与部署

- [构建配置](06_GN_Build.md)
  - 目的：说明 Hvigor 构建配置
  - 适用范围：构建配置修改
  - 关键结论：模块列表、配置参数、依赖关系

- [编译产物](07_Build_Artifacts.md)
  - 目的：列出编译输出文件
  - 适用范围：部署和打包
  - 关键结论：产物清单、安装路径、加载关系

### 安全与质量

- [安全风险评审](08_Security_Review.md)
  - 目的：识别安全风险和提供修复建议
  - 适用范围：安全审计和漏洞修复
  - 关键结论：攻击面清单、可被利用点、修复建议

### 支持文档

- [常见问题](09_FAQ.md)
  - 目的：解答常见问题
  - 适用范围：问题排查和知识积累
  - 关键结论：问题分类、解决方案、定位路径

### 附录

- [关键调用链](appendix/Callgraphs.md)
  - 目的：梳理重要业务流程
  - 适用范围：流程理解和问题定位
  - 关键结论：调用链图、关键节点

- [关键配置](appendix/Config_Flags.md)
  - 目的：汇总配置参数
  - 适用范围：配置调优和功能开关
  - 关键结论：参数列表、默认值、作用范围

---

## 文档规范

### 证据要求

本 Wiki 遵循以下规范：

1. **所有结论必有代码证据**：包含文件路径、行号（如适用）、符号名
2. **无测试引用**：不引用测试相关内容作为业务证据
3. **术语一致性**：统一使用 OpenHarmony 官方术语
4. **链接完整性**：所有内部链接指向有效文件

### 文档结构

每个文档遵循以下结构：

- **目的** - 文档的目标读者和使用场景
- **适用范围** - 文档适用的阶段或角色
- **关键结论** - 最重要的发现和建议
- **详细内容** - 完整的技术描述
- **相关跳转** - 相关文档链接

---

## 快速查找

### 按主题查找

- **想了解项目概况？** → [00_Overview.md](00_Overview.md)
- **想快速上手代码？** → [02_Directory_Structure.md](02_Directory_Structure.md)
- **想调用 print API？** → [04_External_API.md](04_External_API.md)
- **想修改模块接口？** → [05_Internal_API.md](05_Internal_API.md)
- **遇到编译问题？** → [09_FAQ.md](09_FAQ.md)
- **关心安全？** → [08_Security_Review.md](08_Security_Review.md)

### 按角色查找

- **新人开发** → 按"新人推荐阅读顺序"阅读前 3 篇
- **架构师** → 阅读 [03_Architecture.md](03_Architecture.md) 和 [05_Internal_API.md](05_Internal_API.md)
- **测试工程师** → 阅读 [04_External_API.md](04_External_API.md) 和 [09_FAQ.md](09_FAQ.md)
- **安全工程师** → 阅读 [08_Security_Review.md](08_Security_Review.md)
- **DevOps 工程师** → 阅读 [06_GN_Build.md](06_GN_Build.md) 和 [07_Build_Artifacts.md](07_Build_Artifacts.md)

---

## 文档更新日志

- 2026-02-05: 初始版本创建
