# HiCollie Wiki 文档导航

> 新人快速上手指南与文档索引

---

## 推荐阅读顺序

### 📖 基础理解 (必读)
1. **[README.md](README.md)** - 文档概述、范围、维护说明
2. **[00_Overview.md](00_Overview.md)** - 项目定位、核心能力、运行环境、关键概念
3. **[01_Directory_Structure.md](01_Directory_Structure.md)** - 目录结构、模块职责划分

### 🏗️ 架构深入
4. **[02_Architecture.md](02_Architecture.md)** - 组件图、数据流、线程模型、关键时序

### 🔌 接口开发
根据你的开发需求选择：

| 开发类型 | 阅读文档 | 说明 |
|---------|----------|------|
| **C 语言调用** | [03_NDK_API.md](03_NDK_API.md) | NDK C API 完整清单、参数、调用链 |
| **C++ 调用** | [04_Internal_API.md](04_Internal_API.md) | Native C++ API、模块接口、依赖方向 |

### 🔨 构建与集成
5. **[05_GN_Targets.md](05_GN_Targets.md)** - GN 构建目标、依赖、编译开关
6. **[06_Build_Artifacts.md](06_Build_Artifacts.md)** - 编译产物、安装路径、运行时加载

### 🔒 安全审计
7. **[07_Security_Review.md](07_Security_Review.md)** - 安全风险评审、攻击面、威胁模型

### 🐛 问题定位
8. **[08_Troubleshooting.md](08_Troubleshooting.md)** - 常见问题与定位路径

---

## 文档速查表

| 文档 | 主要内容 | 适用场景 |
|-----|---------|---------|
| `00_Overview.md` | 项目定位、边界、核心能力、运行环境 | 快速了解 HiCollie |
| `01_Directory_Structure.md` | 目录结构、模块职责、关键文件位置 | 代码导航 |
| `02_Architecture.md` | 组件图、数据流、线程模型、时序 | 理解架构设计 |
| `03_NDK_API.md` | NDK C API 清单、参数、错误码、调用链 | C 语言开发 |
| `04_Internal_API.md` | 模块接口、依赖方向、稳定性标识 | C++ 模块开发 |
| `05_GN_Targets.md` | GN targets、类型、依赖、编译开关 | 构建系统修改 |
| `06_Build_Artifacts.md` | 编译产物、安装路径、运行时加载 | 集成与部署 |
| `07_Security_Review.md` | 攻击面、信任边界、可被利用点 | 安全审计 |
| `08_Troubleshooting.md` | 构建问题、运行问题、调试技巧 | 问题定位 |

---

## 关键概念速查

| 概念 | 说明 | 文档章节 |
|-----|------|---------|
| **Watchdog** | 线程监控器，检测线程是否及时处理事件 | 02_Architecture.md |
| **XCollie** | 超时检测框架，支持定时器和计数器 | 00_Overview.md |
| **IPC Full** | IPC 满监控，检测 Binder 缓冲区溢出 | 00_Overview.md |
| **Jank Detection** | 卡顿检测，监控主线程性能 | 00_Overview.md |
| **Thread Sampler** | 线程堆栈采样器，用于收集卡顿时的堆栈 | 02_Architecture.md |
| **FFRT** | Foundation Fault Response Task，故障响应任务框架 | 05_GN_Targets.md |
| **HiSysEvent** | 系统事件上报机制 | 06_Build_Artifacts.md |

---

## 按角色导航

### 🔵 应用开发者
- 阅读 `00_Overview.md` 了解 HiCollie 能做什么
- 查看 `03_NDK_API.md` 学习 NDK C API
- 参考 `08_Troubleshooting.md` 解决集成问题

### 🟢 系统服务开发者
- 阅读 `02_Architecture.md` 理解内部机制
- 查看 `04_Internal_API.md` 使用 Native C++ API
- 参考 `05_GN_Targets.md` 了解构建配置

### 🔴 安全审计员
- 从 `07_Security_Review.md` 开始
- 结合 `03_NDK_API.md` 理解攻击面
- 查阅源码验证安全机制

### 🟡 构建工程师
- 查看 `05_GN_Targets.md` 了解构建系统
- 参考 `06_Build_Artifacts.md` 理解产物依赖
- 阅读 `08_Troubleshooting.md` 解决构建问题

---

## 附录

### 可选文档
- `appendix/Callgraphs.md` - 关键调用链图
- `appendix/Config_Flags.md` - 关键宏和 feature flags

### 相关链接
- [OpenHarmony DFX 子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/DFX子系统.md)
- [HiCollie 源码](https://gitee.com/openharmony/hiviewdfx_hicollie)
