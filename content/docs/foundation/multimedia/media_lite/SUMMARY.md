# Media Lite Wiki - 全站导航

本文档提供 media_lite 项目的全面技术导航，帮助新人和开发者快速定位所需信息。

---

## 新人阅读顺序

建议按照以下顺序阅读，以最快速度理解项目：

1. **[00_Overview.md](00_Overview.md)** - 项目定位、边界、核心能力、运行环境
2. **[01_Directory_Structure.md](01_Directory_Structure.md)** - 目录结构与模块职责
3. **[02_Architecture.md](02_Architecture.md)** - 绶构说明、组件图、数据流、线程模型
4. **[03_JSI_Interfaces.md](03_JSI_Interfaces.md)** - 对外 JSI（JavaScript Interface）接口
5. **[04_Inner_API.md](04_Inner_API.md)** - 内部 API、模块接口、依赖方向
6. **[05_GN_Targets.md](05_GN_Targets.md)** - GN Targets、类型、依赖、产物
7. **[06_Build_Artifacts.md](06_Build_Artifacts.md)** - 编译产物、安装路径、运行时加载关系
8. **[07_Security_Audit.md](07_Security_Audit.md)** - 安全风险评审、攻击面、信任边界
9. **[08_FAQ.md](08_FAQ.md)** - 常见构建/运行/调试问题与定位路径

---

## 文档索引

| 文档 | 描述 | 适用人群 |
|------|------|---------|
| [00_Overview.md](00_Overview.md) | 项目概览、定位、核心能力 | 所有人 |
| [01_Directory_Structure.md](01_Directory_Structure.md) | 目录结构与模块职责 | 所有人 |
| [02_Architecture.md](02_Architecture.md) | 架构说明、组件图、数据流 | 架构师、资深开发者 |
| [03_JSI_Interfaces.md](03_JSI_Interfaces.md) | 对外 JSI 接口（API 清单表） | 应用开发者、JS 开发者 |
| [04_Inner_API.md](04_Inner_API.md) | 内部 API、模块接口、依赖方向 | 框架开发者、系统开发者 |
| [05_GN_Targets.md](05_GN_Targets.md) | GN Targets、类型、依赖、产物 | 构建工程师 |
| [06_Build_Artifacts.md](06_Build_Artifacts.md) | 编译产物、安装路径、运行时加载 | 构建工程师、系统集成人员 |
| [07_Security_Audit.md](07_Security_Audit.md) | 安全风险评审、攻击面、信任边界 | 安全工程师、架构师 |
| [08_FAQ.md](08_FAQ.md) | 常见问题与定位路径 | 所有人 |

---

## 关键概念

### 术语表

| 术语 | 说明 |
|------|------|
| **JSI** | JavaScript Interface，OpenHarmony 自定义 JS 绑定 API（非标准 Node.js N-API） |
| **SAMGR** | System Ability Manager Lite，服务管理和发现系统 |
| **IPC** | Inter-Process Communication，进程间通信 |
| **Passthrough 模式** | 直接链接实现库，减少 IPC 开销的模式 |
| **Binder 模式** | 通过 IPC/SAMGR 通信的标准模式 |
| **LiteOS** | OpenHarmony 小型系统内核 |

---

## 快速链接

### 按功能查找

- **播放相关**：[00_Overview.md](00_Overview.md) → [02_Architecture.md](02_Architecture.md) → [03_JSI_Interfaces.md](03_JSI_Interfaces.md)
- **录制相关**：[00_Overview.md](00_Overview.md) → [02_Architecture.md](02_Architecture.md) → [04_Inner_API.md](04_Inner_API.md)
- **构建相关**：[05_GN_Targets.md](05_GN_Targets.md) → [06_Build_Artifacts.md](06_Build_Artifacts.md)
- **安全相关**：[02_Architecture.md](02_Architecture.md) → [07_Security_Audit.md](07_Security_Audit.md)

### 按角色查找

- **应用开发者**：[03_JSI_Interfaces.md](03_JSI_Interfaces.md) → [08_FAQ.md](08_FAQ.md)
- **框架开发者**：[04_Inner_API.md](04_Inner_API.md) → [02_Architecture.md](02_Architecture.md)
- **构建工程师**：[05_GN_Targets.md](05_GN_Targets.md) → [01_Directory_Structure.md](01_Directory_Structure.md)
- **安全工程师**：[07_Security_Audit.md](07_Security_Audit.md) → [02_Architecture.md](02_Architecture.md)

---

## 附录

更多项目信息请参考：
- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [项目 README](../README_zh.md)
- [Wiki 工作笔记](./_work/NOTES.md)
- [项目评估](./_work/ASSESSMENT.md) - 项目类型、受众分析、文档策略
- [任务计划](./_work/PLAN.md) - Wiki 生成计划和进度
