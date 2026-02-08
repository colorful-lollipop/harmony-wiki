# Work Scheduler Wiki

> 本文档由 OpenHarmony Work Scheduler 模块自动生成
> **生成时间**: 2025-02-06
> **版本**: work_scheduler 3.1
> **覆盖范围**: 完整模块文档，包括架构、API、编译和安全分析

---

## 文档导航

本文档采用渐进式阅读方式，建议按以下顺序阅读：

1. **新人入门**
   - [00_Overview.md](00_Overview.md) - 模块概览和核心概念 ✅
   - [01_Positioning.md](01_Positioning.md) - 项目定位和边界 ✅
   - [02_Directory.md](02_Directory.md) - 目录结构与模块职责 ✅

2. **架构理解**
   - [03_Architecture.md](03_Architecture.md) - 架构说明和数据流 ✅
   - [04_External_API.md](04_External_API.md) - 对外 N-API 接口 ✅
   - [05_Internal_API.md](05_Internal_API.md) - 内部 API 和依赖 ✅

3. **构建与部署**
   - [06_GN_Targets.md](06_GN_Targets.md) - GN 构建配置 ✅
   - [07_Build_Artifacts.md](07_Build_Artifacts.md) - 编译产物和安装 ✅

4. **进阶主题**
   - [08_Security.md](08_Security.md) - 安全风险评审 ✅
   - [09_FAQ.md](09_FAQ.md) - 常见问题与定位 ✅

5. **附录**
   - [appendix/Callgraphs.md](appendix/Callgraphs.md) - 关键调用链 ✅
   - [appendix/Config_Flags.md](appendix/Config_Flags.md) - 配置标志 ✅

---

## 文档维护

### 如何更新文档

1. **代码变更时更新**:
   - 修改了 N-API 接口 → 更新 `04_External_API.md`
   - 修改了架构 → 更新 `03_Architecture.md`
   - 新增/删除 BUILD.gn target → 更新 `06_GN_Targets.md`

2. **文档版本**:
   - 每次代码变更后更新 `生成时间`
   - 在文档末尾添加 `更新记录` 章节

3. **证据要求**:
   - 所有结论必须有代码证据（文件路径:行号）
   - 不确定的内容标注 `TODO(需确认)`

### 覆盖范围

| 模块 | 状态 |
|------|------|
| 目录结构 | ✅ 完整 |
| N-API 接口 | ✅ 完整 |
| 内部架构 | ✅ 完整 |
| GN Targets | ✅ 完整 |
| 编译产物 | ✅ 完整 |
| 安全评审 | ✅ 完整 |

### 未覆盖范围

- 测试相关代码（已按规范忽略）
- 动态生成的代码（IDL 生成的 Proxy/Stub 文件）
- 第三方依赖模块的内部实现

---

## 快速索引

### 按主题索引

| 主题 | 文档 | 关键内容 |
|------|--------|---------|
| **模块概述** | 00_Overview.md | 核心功能、运行环境、依赖 |
| **项目定位** | 01_Positioning.md | 边界、Syscap、ROM/RAM 占用 |
| **目录结构** | 02_Directory.md | 各层职责、文件组织 |
| **架构设计** | 03_Architecture.md | 组件图、数据流、线程模型 |
| **对外 API** | 04_External_API.md | N-API 清单、参数校验、错误码 |
| **内部 API** | 05_Internal_API.md | 模块接口、依赖方向、稳定性 |
| **构建系统** | 06_GN_Targets.md | Targets 列表、依赖、开关 |
| **编译产物** | 07_Build_Artifacts.md | 库文件、安装路径、运行时加载 |
| **安全分析** | 08_Security.md | 攻击面、可利用点、修复建议 |
| **常见问题** | 09_FAQ.md | 构建、运行、调试问题 |

### 按关键词索引

- **N-API**: `startWork`, `stopWork`, `getWorkStatus`, `obtainAllWorks`, `stopAndClearWorks`, `isLastWorkTimeOut`
- **ExtensionAbility**: `WorkSchedulerExtensionAbility`, `WorkSchedulerExtensionContext`
- **System Ability**: SA 1904, `IWorkSchedService`, `IWorkScheduler`
- **条件监听**: 网络类型、电池状态、充电类型、存储状态、屏幕状态、定时器
- **策略过滤器**: CPU、内存、功耗、温度、功耗模式
- **编译产物**: `libworkschedservice.z.so`, `libworkscheduler.so`, `libcj_work_scheduler_ffi.so`

---

## 技术栈概览

### 编程语言
- **C/C++**: 服务端和框架层核心实现
- **TypeScript/JavaScript**: N-API 接口定义
- **ArkTS/ETS**: Taihe 框架和 Extension 实现
- **Cangjie**: FFI 接口绑定

### 核心依赖
- **OpenHarmony 框架**: Ability Runtime, Bundle Framework, IPC, NAPI, Event Handler
- **系统服务**: Device Usage Statistics, Device Standby, Background Task Manager, Battery Manager, Thermal Manager, Power Manager
- **权限系统**: Access Token, Token ID

### 构建系统
- **GN (Generate Ninja)**: 主构建工具
- **IDL 编译器**: 生成 IPC 接口代码
- **Taihe 编译器**: 生成 ArkTS 原生接口代码
