# blackbox_lite Wiki 导航

## 新人阅读路线

建议按以下顺序阅读，以快速建立对项目的完整理解：

```
1️⃣ 01_Overview    → 项目定位与核心概念
     ↓
2️⃣ 02_Architecture → 组件结构与数据流转
     ↓
3️⃣ 03_API_Reference → 接口与调用方式
     ↓
4️⃣ 04_Build_Configuration → 构建配置与产物
     ↓
5️⃣ 05_Security_Review → 安全风险（如需审计）
```

## 文档目录

### 快速入口

| 文档 | 说明 |
|------|------|
| [README](README.md) | Wiki 使用说明与贡献指南 |
| [SUMMARY](SUMMARY.md) | 本文档，全站导航 |

### 核心文档

| 文档 | 章节 |
|------|------|
| [01_Overview](01_Overview.md) | |
| ├─ 项目定位 | blackbox_lite 在 DFX 子系统中的角色 |
| ├─ 核心能力 | 故障信息获取、日志保存、死机重启 |
| ├─ 运行环境 | Mini 系统 (liteos_m/a)、资源占用 |
| └─ 关键概念 | WEAK 适配器模式、ErrorInfo 结构 |
| [02_Architecture](02_Architecture.md) | |
| ├─ 组件图 | 模块关系与依赖 |
| ├─ 数据流 | 故障信息流转路径 |
| ├─ 线程模型 | SaveErrorLog 线程与信号量 |
| └─ 关键时序 | 初始化与故障处理流程 |
| [03_API_Reference](03_API_Reference.md) | |
| ├─ 公共接口 | BBoxNotifyError、BBoxRegisterModuleOps |
| ├─ 适配器接口 | SystemModule* 系列（WEAK） |
| ├─ 事件类型 | 预定义故障事件常量 |
| └─ 调用链 | JS/调用入口 → 核心逻辑 → 适配层 |
| [04_Build_Configuration](04_Build_Configuration.md) | |
| ├─ GN 配置 | BUILD.gn 详解 |
| ├─ 依赖关系 | 组件与第三方依赖 |
| └─ 编译产物 | .a 静态库与安装路径 |
| [05_Security_Review](05_Security_Review.md) | |
| ├─ 威胁模型 | 攻击面与信任边界 |
| ├─ 风险点 | 5+ 条可利用点分析 |
| └─ 修复建议 | 安全加固方案 |
| [06_Troubleshooting](06_Troubleshooting.md) | |
| ├─ 构建问题 | GN 配置与依赖问题 |
| ├─ 运行问题 | 初始化与线程问题 |
| └─ 调试方法 | 日志与断点技巧 |

## 代码证据速查

| 证据类型 | 位置 |
|----------|------|
| 核心逻辑 | `blackbox_core.c` |
| 适配器实现 | `blackbox_adapter.c` |
| 事件上报 | `blackbox_detector.c` |
| 接口定义 | `interfaces/native/**/*.h` |
| 构建配置 | `BUILD.gn` |

## 术语表

| 术语 | 定义 |
|------|------|
| blackbox | 黑盒，指故障现场信息记录模块 |
| WEAK | GCC weak 符号，用于平台适配 |
| ModuleOps | 模块操作接口结构体 |
| ErrorInfo | 故障信息结构体 |
| SaveErrorLog | 日志保存工作线程 |
| CORE_INIT_PRI | 内核初始化优先级宏 |
