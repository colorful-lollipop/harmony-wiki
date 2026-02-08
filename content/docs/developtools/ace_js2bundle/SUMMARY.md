# SUMMARY - ace_js2bundle Wiki 导航

## 📚 双路线导航

本 Wiki 提供两条阅读路线，满足不同受众需求：
- **新人学习路线**：快速理解项目，上手开发
- **安全研究路线**：识别攻击面，评估安全风险

---

## 🎓 新人学习路线

### 第一步：了解项目（10分钟）
1. [index.md](./index.md) - 项目首页与概览
2. [01_Overview.md](./01_Overview.md) - 项目定位与核心能力

### 第二步：理解结构（20分钟）
3. [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构
4. [03_Architecture.md](./03_Architecture.md) - 架构与数据流

### 第三步：深入细节（按需阅读）
5. [05_Internal_API.md](./05_Internal_API.md) - 内部 API 与模块接口
6. [06_Build_System.md](./06_Build_System.md) - GN 构建系统
7. [07_Security.md](./07_Security.md) - 安全风险评估

### 第四步：实战参考
8. [08_Troubleshooting.md](./08_Troubleshooting.md) - 常见问题

---

## 🔒 安全研究路线

### 第一阶段：快速评估（15分钟）
**目标**：理解项目定位，识别主要风险

1. [01_Overview.md](./01_Overview.md) - 项目定位与边界
   - 重点关注：运行域、对外暴露面、项目边界
   
2. [07_Security.md](./07_Security.md) - 安全风险评估
   - 重点关注：攻击面分析、风险清单、信任边界

### 第二阶段：攻击面分析（30分钟）
**目标**：识别所有外部输入入口和敏感操作

3. [03_Architecture.md](./03_Architecture.md) - 架构与数据流
   - 重点关注：数据流图、输入层、外部输入清单
   
4. [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构
   - 重点关注：核心文件定位、外部接口文件
   
5. [04_External_API.md](./04_External_API.md) - 对外接口
   - 重点关注：CLI 入口、环境变量、配置文件

### 第三阶段：深度审计（60分钟）
**目标**：详细分析风险点，评估可利用性

6. [07_Security.md](./07_Security.md) - 安全风险详情
   - 重点关注：R1-R8 风险详细分析、触发路径、修复建议
   
7. [05_Internal_API.md](./05_Internal_API.md) - 内部 API
   - 重点关注：敏感操作函数、文件系统操作、子进程调用
   
8. [appendix/Callgraphs.md](./appendix/Callgraphs.md) - 关键调用链
   - 重点关注：输入到漏洞点的完整调用链

### 第四阶段：验证与利用（按需）
**目标**：验证风险，开发 PoC

9. [06_Build_System.md](./06_Build_System.md) - 构建系统
   - 重点关注：Feature 开关、配置选项、构建流程
   
10. [appendix/Config_Flags.md](./appendix/Config_Flags.md) - 配置与 Flags
    - 重点关注：影响安全行为的配置项

---

## 📋 快速索引

### 按主题

| 主题 | 文档 |
|------|------|
| 项目定位 | [01_Overview.md](./01_Overview.md) |
| 目录结构 | [02_Directory_Structure.md](./02_Directory_Structure.md) |
| 架构设计 | [03_Architecture.md](./03_Architecture.md) |
| 对外接口 | [04_External_API.md](./04_External_API.md) |
| 内部 API | [05_Internal_API.md](./05_Internal_API.md) |
| 构建系统 | [06_Build_System.md](./06_Build_System.md) |
| 安全风险 | [07_Security.md](./07_Security.md) |
| 问题排查 | [08_Troubleshooting.md](./08_Troubleshooting.md) |

### 按角色

| 角色 | 推荐路线 |
|------|----------|
| 🆕 我是新人开发者 | → 新人学习路线 |
| 🔧 我要了解构建系统 | → [06_Build_System.md](./06_Build_System.md) |
| 🐛 我要排查问题 | → [08_Troubleshooting.md](./08_Troubleshooting.md) |
| 🔒 我要评估安全风险 | → 安全研究路线 |
| 🔧 我要扩展功能 | → [03_Architecture.md](./03_Architecture.md) → [05_Internal_API.md](./05_Internal_API.md) |
| 📝 我要查阅配置 | → [appendix/Config_Flags.md](./appendix/Config_Flags.md) |

---

## 📖 完整文档索引

---

## 完整文档索引

### 核心文档

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [README.md](./README.md) | Wiki 说明 | 生成信息、更新方式 |
| [index.md](./index.md) | 项目首页 | 一句话描述、快速开始、关键特性 |
| [01_Overview.md](./01_Overview.md) | 项目概览 | 定位、边界、核心能力、运行环境 |
| [02_Directory_Structure.md](./02_Directory_Structure.md) | 目录结构 | 模块职责、文件组织 |
| [03_Architecture.md](./03_Architecture.md) | 架构说明 | 组件图、数据流、插件链、时序 |
| [04_External_API.md](./04_External_API.md) | 对外 API | N-API 接口清单（本项目无） |
| [05_Internal_API.md](./05_Internal_API.md) | 内部 API | 模块接口、依赖方向、稳定性 |
| [06_Build_System.md](./06_Build_System.md) | GN 构建 | Targets、依赖、产物、开关 |
| [07_Security.md](./07_Security.md) | 安全评审 | 攻击面、风险点、修复建议 |
| [08_Troubleshooting.md](./08_Troubleshooting.md) | 常见问题 | 构建/运行/调试问题 |

### 附录

| 文档 | 描述 |
|------|------|
| [appendix/Callgraphs.md](./appendix/Callgraphs.md) | 关键调用链 |
| [appendix/Config_Flags.md](./appendix/Config_Flags.md) | 关键配置与 Feature Flags |

---

## 按角色导航

### 我是新开发者
→ 按【新人阅读路线】顺序阅读

### 我要了解构建系统
→ [06_Build_System.md](./06_Build_System.md)

### 我要排查问题
→ [08_Troubleshooting.md](./08_Troubleshooting.md)

### 我要评估安全风险
→ [07_Security.md](./07_Security.md)

### 我要扩展功能
→ [03_Architecture.md](./03_Architecture.md) → [05_Internal_API.md](./05_Internal_API.md)

---

## 术语速查

| 术语 | 说明 |
|------|------|
| ace-loader | 核心 webpack loader |
| HML | HarmonyOS Markup Language（类 HTML） |
| ABC | Ark Bytecode（Ark 字节码） |
| 富设备 (Rich) | 手机/平板等高性能设备 |
| 瘦设备 (Lite) | 手表/IoT 等低性能设备 |
| 卡片 (Card/Form) | 服务卡片/表单 |
