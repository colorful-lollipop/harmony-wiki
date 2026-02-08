# FilePicker Wiki 导航

本文档目录提供了 FilePicker 应用的完整阅读路径。

## 新人阅读建议

如果你是第一次接触 FilePicker，建议按以下顺序阅读：

1. **快速了解**：从 [概览](00_Overview.md) 开始，了解项目定位和边界
2. **熟悉代码**：阅读 [目录结构](01_Directory_Structure.md)，理解代码组织
3. **深入架构**：查看 [架构说明](02_Architecture.md)，掌握核心设计
4. **接入使用**：参考 [对外 API](03_Public_API.md)，学习如何调用 FilePicker
5. **二次开发**：研究 [内部 API](04_Internal_API.md)，了解模块接口
6. **构建部署**：阅读 [构建系统](05_Build_System.md)，掌握编译流程
7. **安全合规**：检查 [权限机制](06_Permissions.md)，理解授权流程
8. **风险意识**：阅读 [安全评审](07_Security_Review.md)，了解潜在风险
9. **问题排查**：查阅 [常见问题](08_FAQ.md)，快速定位问题

---

## 文档目录

### 项目概览

- [00_Overview.md](00_Overview.md)
  - 项目定位与边界
  - 核心能力
  - 运行环境
  - 关键概念

### 目录结构

- [01_Directory_Structure.md](01_Directory_Structure.md)
  - 模块划分
  - 目录职责
  - 文件组织规范

### 架构设计

- [02_Architecture.md](02_Architecture.md)
  - 组件图
  - 数据流
  - 线程模型
  - 关键时序（Mermaid）

### 对外 API

- [03_Public_API.md](03_Public_API.md)
  - API 清单表
  - 参数说明
  - 错误码
  - 权限要求
  - 调用示例

### 内部 API

- [04_Internal_API.md](04_Internal_API.md)
  - 模块接口
  - 依赖方向
  - 稳定性说明
  - 可替换点

### 构建系统

- [05_Build_System.md](05_Build_System.md)
  - Hvigor 配置
  - 编译产物
  - 安装路径
  - 运行时加载关系

### 权限机制

- [06_Permissions.md](06_Permissions.md)
  - 权限声明
  - 鉴权流程
  - URI 授权机制
  - 安全检查点

### 安全评审

- [07_Security_Review.md](07_Security_Review.md)
  - 攻击面清单
  - 信任边界
  - 可被利用点
  - 修复建议

### 常见问题

- [08_FAQ.md](08_FAQ.md)
  - 构建问题
  - 运行问题
  - 调试技巧
  - 问题定位路径

### 附录

- [appendix/Callgraphs.md](appendix/Callgraphs.md)
  - 关键调用链
  - 入口→核心逻辑流程

- [appendix/Config_Flags.md](appendix/Config_Flags.md)
  - 关键宏定义
  - Feature flags

---

## 文档使用指南

### 符号说明

| 符号 | 含义 |
|-------|------|
| ✅ | 已完成，有完整代码证据 |
| ⚠️ | 有待确认，标注 TODO |
| 📁 | 文件路径 |
| 🔧 | 配置项 |
| 🔒 | 安全相关 |

### 代码证据格式

文档中的每个关键结论都附带了代码证据：

```
证据：[文件路径:行号]
示例代码：
```typescript
// 代码片段
```
```

### 术语表

| 术语 | 说明 |
|------|------|
| Ability | OpenHarmony 的能力单元，类似于 Android 的 Activity/Service |
| UIAbility | 带有用户界面的 Ability |
| UIExtensionAbility | UI 扩展能力，用于系统 Picker 等场景 |
| Want | OpenHarmony 中的组件间通信数据结构 |
| URI | 统一资源标识符，用于文件访问和授权 |
| HAP | HarmonyOS Ability Package，OpenHarmony 应用包格式 |
| UDMF | Unified Data Management Framework，统一数据管理框架 |

---

**最后更新**: 2026-02-05 23:20:03
