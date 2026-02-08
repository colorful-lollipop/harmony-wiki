# CastEngine 框架 Wiki 导航

> 最后更新: 2026-02-06

## 新人快速入门（推荐阅读顺序）

如果您是刚接触 CastEngine 框架，建议按照以下顺序阅读文档：

### 1. 快速了解项目 (15 分钟)

**[00_Overview.md](00_Overview.md)**
- 项目定位和核心能力
- 运行环境和依赖
- 关键概念和术语
- 适用场景

**阅读后您将了解**: CastEngine 是什么，它能做什么，以及在什么场景下使用

### 2. 理解代码结构 (30 分钟)

**[01_Directory_Structure.md](01_Directory_Structure.md)**
- 完整目录树（不含测试）
- 各目录的职责和作用
- 模块划分和边界
- 代码组织方式

**阅读后您将了解**: 代码是如何组织的，每个目录放什么，如何定位需要的代码

### 3. 学习 JavaScript API 使用 (1 小时)

**[03_N-API.md](03_N-API.md)**
- 完整 N-API 清单表
- CastSessionManager API
- CastSession API
- StreamPlayer API
- MirrorPlayer API
- 参数校验和错误码

**阅读后您将能够**: 使用 JavaScript 调用 CastEngine 功能，创建投屏应用

### 4. 深入理解架构 (2 小时)

**[02_Architecture.md](02_Architecture.md)**
- 组件关系图
- 数据流向
- 线程模型
- 关键时序图
- IPC 通信机制

**阅读后您将理解**: CastEngine 内部如何工作，各组件如何协作

### 5. 内部开发参考 (按需)

**[04_Internal_API.md](04_Internal_API.md)**
- 内部 C++ 接口定义
- 模块间依赖关系
- 接口稳定性说明
- 扩展点和替换点

**适用人群**: 需要深入集成或扩展框架的开发者

**[05_GN_Targets.md](05_GN_Targets.md)**
- GN 构建系统详解
- 所有目标列表和依赖
- 编译产物映射
- 如何添加新目标

**适用人群**: 需要修改构建配置的开发者

### 6. 部署和运维 (按需)

**[06_Build_Artifacts.md](06_Build_Artifacts.md)**
- 编译产物清单
- 安装路径说明
- 运行时加载关系
- 部署注意事项

**适用人群**: 系统集成工程师、测试工程师

### 7. 安全和最佳实践 (必读)

**[07_Security_Review.md](07_Security_Review.md)**
- 攻击面清单
- 信任边界分析
- 已发现的安全风险
- 修复建议和最佳实践

**适用人群**: 所有开发、测试、安全人员

### 8. 问题排查 (按需)

**[08_QA.md](08_QA.md)**
- 常见构建问题
- 常见运行时问题
- 调试方法和工具
- 日志分析技巧

**适用人群**: 遇到问题时快速定位和解决

---

## 按主题查找

如果您有特定主题想了解，可以直接跳转：

### API 和接口

- **JavaScript API (N-API)**: [03_N-API.md](03_N-API.md)
- **内部 C++ API**: [04_Internal_API.md](04_Internal_API.md)

### 架构和设计

- **整体架构**: [02_Architecture.md](02_Architecture.md)
- **目录结构**: [01_Directory_Structure.md](01_Directory_Structure.md)
- **调用链**: [appendix/Callgraphs.md](appendix/Callgraphs.md)

### 构建和部署

- **GN 构建系统**: [05_GN_Targets.md](05_GN_Targets.md)
- **编译产物**: [06_Build_Artifacts.md](06_Build_Artifacts.md)
- **配置选项**: [appendix/Config_Flags.md](appendix/Config_Flags.md)

### 安全和质量

- **安全评审**: [07_Security_Review.md](07_Security_Review.md)
- **常见问题**: [08_QA.md](08_QA.md)

---

## 文档状态

| 文档 | 状态 | 完成度 | 最后更新 |
|------|------|---------|----------|
| README.md | ✅ | 100% | 2026-02-06 |
| SUMMARY.md | ✅ | 100% | 2026-02-06 |
| 00_Overview.md | ✅ | 100% | 2026-02-06 |
| 01_Directory_Structure.md | ✅ | 100% | 2026-02-06 |
| 02_Architecture.md | ✅ | 100% | 2026-02-06 |
| 03_N-API.md | ✅ | 100% | 2026-02-06 |
| 04_Internal_API.md | ✅ | 100% | 2026-02-06 |
| 05_GN_Targets.md | ✅ | 100% | 2026-02-06 |
| 06_Build_Artifacts.md | ✅ | 100% | 2026-02-06 |
| 07_Security_Review.md | ✅ | 100% | 2026-02-06 |
| 08_QA.md | ✅ | 100% | 2026-02-06 |

---

## 术语表

| 术语 | 英文 | 说明 |
|-----|------|------|
| 投屏 | Cast | 将本设备的音视频内容投射到远程设备 |
| 镜像 | Mirror | 实时屏幕同步投射 |
| 流播放 | Stream | 媒体文件播放 |
| 会话 | Session | 一次完整的投屏连接和交互过程 |
| N-API | Native API | OpenHarmony 的原生模块接口 |
| System Ability | SA | OpenHarmony 的系统服务能力 |
| IPC | Inter-Process Communication | 跨进程通信 |
| GN | Generate Ninja | OpenHarmony 使用的构建系统 |
| SoftBus | - | OpenHarmony 的分布式通信框架 |

---

**返回**: [README.md](README.md)
