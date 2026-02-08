# RingtoneLibrary Wiki - 目录导航

## 新人阅读路线

建议按以下顺序阅读本文档，以快速理解项目：

1. **[00_Overview.md](./00_Overview.md)** - 项目概览与核心定位（5分钟）
2. **[01_Project_Positioning.md](./01_Project_Positioning.md)** - 项目边界、运行环境、关键概念（10分钟）
3. **[02_Directory_Structure.md](./02_Directory_Structure.md)** - 目录结构与模块职责（15分钟）
4. **[03_Architecture.md](./03_Architecture.md)** - 组件图、数据流、线程模型、关键时序（20分钟）
5. **[04_External_API.md](./04_External_API.md)** - 对外 API（DataShare、N-API、权限、错误码）（30分钟）
6. **[05_Internal_API.md](./05_Internal_API.md)** - 内部 API、依赖方向、接口稳定性（20分钟）
7. **[06_GN_Targets.md](./06_GN_Targets.md)** - GN 构建目标、编译配置、产物映射（15分钟）
8. **[07_Build_Artifacts.md](./07_Build_Artifacts.md)** - 编译产物、安装路径、运行时加载（10分钟）
9. **[08_Security_Review.md](./08_Security_Review.md)** - 安全风险评审、攻击面、可利用点（30分钟）
10. **[09_Common_Issues.md](./09_Common_Issues.md)** - 常见问题与定位路径（按需）

**总阅读时间**: 约 2.5 小时

---

## 完整文档列表

### 核心文档

| 文档 | 状态 | 更新时间 |
|--------|--------|---------|
| [00_Overview.md](./00_Overview.md) | TODO | - |
| [01_Project_Positioning.md](./01_Project_Positioning.md) | TODO | - |
| [02_Directory_Structure.md](./02_Directory_Structure.md) | TODO | - |
| [03_Architecture.md](./03_Architecture.md) | TODO | - |
| [04_External_API.md](./04_External_API.md) | TODO | - |
| [05_Internal_API.md](./05_Internal_API.md) | TODO | - |
| [06_GN_Targets.md](./06_GN_Targets.md) | TODO | - |
| [07_Build_Artifacts.md](./07_Build_Artifacts.md) | TODO | - |
| [08_Security_Review.md](./08_Security_Review.md) | TODO | - |
| [09_Common_Issues.md](./09_Common_Issues.md) | TODO | - |

### 附录文档（可选）

| 文档 | 状态 | 描述 |
|--------|--------|------|
| [appendix/Callgraphs.md](./appendix/Callgraphs.md) | TODO | 关键调用链（入口→核心逻辑） |
| [appendix/Config_Flags.md](./appendix/Config_Flags.md) | TODO | 关键宏/feature flags |

---

## 按角色查找文档

### 我是应用开发者

- 阅读 [04_External_API.md](./04_External_API.md) 了解如何使用 DataShare 接口访问铃音数据
- 阅读 [08_Security_Review.md](./08_Security_Review.md) 了解权限要求和安全最佳实践

### 我是系统开发者

- 阅读 [03_Architecture.md](./03_Architecture.md) 了解组件架构和数据流
- 阅读 [05_Internal_API.md](./05_Internal_API.md) 了解内部接口和依赖
- 阅读 [06_GN_Targets.md](./06_GN_Targets.md) 了解构建配置

### 我是安全审计员

- 阅读 [08_Security_Review.md](./08_Security_Review.md) 了解攻击面和已识别的风险点

### 我是维护者

- 阅读 [02_Directory_Structure.md](./02_Directory_Structure.md) 了解模块边界
- 阅读 [09_Common_Issues.md](./09_Common_Issues.md) 了解常见问题和调试方法
