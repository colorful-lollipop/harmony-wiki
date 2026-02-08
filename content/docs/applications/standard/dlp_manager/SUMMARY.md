# 全站导航

## 新人阅读路线

建议按以下顺序阅读Wiki，以建立完整的项目认知：

### 第一步：了解项目（5分钟）
1. [README](README.md) - 了解Wiki覆盖范围和更新方式
2. [项目概览](00_Overview.md) - 了解项目定位、核心能力和运行环境

### 第二步：理解架构（15分钟）
3. [目录结构](20_Directory_Structure.md) - 查看代码组织方式
4. [架构设计](10_Architecture.md) - 理解系统架构、数据流和线程模型

### 第三步：掌握接口（20分钟）
5. [对外接口](30_Public_API.md) - 学习如何调用DLP功能
6. [内部接口](40_Inner_API.md) - 了解模块间接口和依赖关系

### 第四步：开发实践（按需）
7. [构建系统](50_Build_System.md) - 了解如何编译和构建
8. [安全风险](60_Security_Analysis.md) - 了解安全风险和防护机制
9. [调试指南](70_Debugging.md) - 学习调试技巧和问题定位

### 附录（参考）
10. [调用链附录](appendix/Callgraphs.md) - 关键调用流程
11. [配置标志附录](appendix/Config_Flags.md) - 重要常量配置

---

## 文档列表

### 核心文档

| 文档 | 描述 | 重点内容 |
|------|------|---------|
| [README](README.md) | Wiki说明 | 覆盖范围、更新方式、生成时间 |
| [项目概览](00_Overview.md) | 项目定位与能力 | 核心功能、关键概念、运行环境 |
| [架构设计](10_Architecture.md) | 系统架构 | 组件图、数据流、线程模型、时序图 |
| [目录结构](20_Directory_Structure.md) | 代码组织 | 目录职责、关键文件、模块边界 |
| [对外接口](30_Public_API.md) | 外部调用接口 | Ability接口、参数、错误码 |
| [内部接口](40_Inner_API.md) | 内部模块接口 | RPC接口、Manager接口、工具类 |
| [构建系统](50_Build_System.md) | 编译构建 | GN配置、Targets、产物清单 |
| [安全风险](60_Security_Analysis.md) | 安全评审 | 攻击面、风险点、修复建议 |
| [调试指南](70_Debugging.md) | 调试定位 | 日志抓取、常见问题、定位路径 |

### 附录

| 文档 | 描述 |
|------|------|
| [调用链附录](appendix/Callgraphs.md) | 关键调用流程（入口→核心逻辑） |
| [配置标志附录](appendix/Config_Flags.md) | 关键宏、Feature Flags、配置项 |

---

## 主题索引

### 按主题查找

#### 想了解如何调用DLP功能？
- [对外接口](30_Public_API.md) - 查看startAbility参数和调用示例

#### 想了解DLP文件如何被打开？
- [架构设计](10_Architecture.md) - 查看打开DLP文件的时序图
- [调用链附录](appendix/Callgraphs.md) - 查看详细调用流程

#### 想了解权限检查机制？
- [安全风险](60_Security_Analysis.md) - 查看信任边界和权限检查点

#### 想了解构建配置？
- [构建系统](50_Build_System.md) - 查看GN配置和编译产物

#### 想调试问题？
- [调试指南](70_Debugging.md) - 查看日志抓取和问题定位

---

## 术语速查

| 术语 | 解释 |
|------|------|
| DLP | Data Leak Prevention，数据防泄漏 |
| Ability | OpenHarmony应用组件，类似Android Activity/Service |
| HAP | HarmonyOS Ability Package，应用安装包 |
| UIExtensionAbility | 支持UI扩展的Ability类型 |
| RPC | Remote Procedure Call，跨进程调用 |
| Sandbox | 沙箱，隔离的运行环境 |
| HUKs | Huawei Universal Keystore，密钥管理系统 |
| Domain Account | 域账号，企业统一身份认证账号 |

---

*最后更新: 2026-02-05*
