# 文档导航

本文档为分布式硬件管理框架（Distributed Hardware Framework）的完整 Wiki 导航。

---

## 快速开始

### 📚 双路线阅读指南

本 Wiki 提供 **新人学习** 和 **安全研究** 两条阅读路线,请根据您的角色选择:

---

### 🎓 新人学习路线 (推荐)

**目标**: 快速理解项目定位、架构设计和代码组织

**阅读顺序**:

```
📖 [index.md](index.md) - 项目概览
   │
   ├── 📁 [01_Directory_Structure.md](01_Directory_Structure.md) - 目录结构
   │      │
   │      └── 📁 [02_Architecture.md](02_Architecture.md) - 系统架构
   │             │
   │             ├── 📁 [03_NAPI_Reference.md](03_NAPI_Reference.md) - N-API 接口
   │             │      │
   │             │      └── 📁 [04_Inner_API.md](04_Inner_API.md) - 内部 API
   │             │
   │             └── 📁 [05_GN_Build.md](05_GN_Build.md) - GN 构建
   │                    │
   │                    └── 📁 [06_Build_Artifacts.md](06_Build_Artifacts.md) - 编译产物
```

**时间投入**:
- 5 分钟: 读完 `index.md` 理解项目定位
- 15 分钟: 浏览 `01_Directory_Structure.md` 了解代码组织
- 30 分钟: 研读 `02_Architecture.md` 掌握核心架构
- 60 分钟: 按需查阅 API 和构建文档

---

### 🔒 安全研究路线 (推荐)

**目标**: 快速识别攻击面、定位风险点和审计关键代码

**阅读顺序**:

```
📖 [07_Security_Review.md](07_Security_Review.md) - 安全评审概览
   │
   ├── 🔍 [攻击面分析](07_Security_Review.md#攻击面分析)
   │      ├── N-API 接口
   │      ├── IPC 接口
   │      └── 跨设备传输
   │
   ├── 🛡️ [信任边界](07_Security_Review.md#信任边界)
   │      ├── 不可信区域 ↔ 可信区域
   │      └── 边界验证点
   │
   ├── ⚠️ [风险点清单](07_Security_Review.md#安全风险评估)
   │      ├── R1: 动态库加载 (高危)
   │      ├── R2: JSON 解析 (中危)
   │      ├── R3: IPC 权限 (中危)
   │      ├── R4: Native SA 绕过 (中危)
   │      └── R5: 整数溢出 (低危)
   │
   └── 📋 [已实施的安全措施](07_Security_Review.md#已实施的安全加固措施)
```

**关键代码定位**:
| 风险点 | 文件路径 | 行号 |
|--------|----------|------|
| 动态库加载 | `component_loader.cpp` | 314 |
| IPC 权限检查 | `distributed_hardware_stub.cpp` | 805-833 |
| JSON 解析 | `capability_info.cpp` | 126-252 |
| 输入验证 | `dh_utils_tool.cpp` | 306-374 |
| 内存分配 | `dh_transport.cpp` | 98-114 |

---

---

## 文档列表

### 核心文档

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [README.md](README.md) | Wiki 使用指南与覆盖范围 | 必读 |
| [index.md](index.md) | 项目概览与核心能力 | 必读 |
| [01_Directory_Structure.md](01_Directory_Structure.md) | 目录结构与模块职责 | 必读 |
| [02_Architecture.md](02_Architecture.md) | 系统架构与数据流 | 必读 |
| [03_NAPI_Reference.md](03_NAPI_Reference.md) | N-API JavaScript 接口 | 重要 |
| [04_Inner_API.md](04_Inner_API.md) | 内部 Inner API 接口 | 重要 |
| [05_GN_Build.md](05_GN_Build.md) | GN 构建配置 | 重要 |
| [06_Build_Artifacts.md](06_Build_Artifacts.md) | 编译产物与加载关系 | 重要 |
| [07_Security_Review.md](07_Security_Review.md) | 安全风险评审 | 必读 |

### 附录文档

| 文档 | 说明 | 状态 |
|------|------|------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链图谱 | 可选 |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 关键配置开关 | 可选 |

---

## 按主题索引

### N-API / JavaScript 接口

- [N-API 概览](03_NAPI_Reference.md#概述)
- [API 清单表](03_NAPI_Reference.md#api-清单表)
- [错误码说明](03_NAPI_Reference.md#错误码)
- [使用示例](03_NAPI_Reference.md#使用示例)

### 内部模块接口

- [Inner Kit SDK](04_Inner_API.md#inner-kit-sdk)
- [IPC 接口](04_Inner_API.md#ipc-接口)
- [模块依赖关系](04_Inner_API.md#模块依赖)

### 构建与编译

- [GN Targets 清单](05_GN_Build.md#targets-清单)
- [依赖配置](05_GN_Build.md#依赖配置)
- [编译产物路径](06_Build_Artifacts.md#产物清单)
- [运行时加载](06_Build_Artifacts.md#运行时加载关系)

### 安全相关

- [攻击面分析](07_Security_Review.md#攻击面分析)
- [信任边界](07_Security_Review.md#信任边界)
- [风险点清单](07_Security_Review.md#风险点清单)
- [修复建议](07_Security_Review.md#修复建议)

---

## 模块快速链接

### 核心服务

| 模块 | 路径 | 说明 |
|------|------|------|
| DistributedHardwareService | `services/distributedhardwarefwkservice/` | 核心 SA 服务 |
| AccessManager | `services/.../accessmanager/` | 硬件接入管理 |
| ResourceManager | `services/.../resourcemanager/` | 资源管理 |
| ComponentManager | `services/.../componentmanager/` | 部件管理 |

### 接口层

| 模块 | 路径 | 说明 |
|------|------|------|
| N-API | `interfaces/kits/napi/` | JS 接口 |
| Inner Kit SDK | `interfaces/inner_kits/` | 内部 SDK |
| Taihe/ANI | `taihe/` | 现代化绑定 |

### AV 传输

| 模块 | 路径 | 说明 |
|------|------|------|
| AV Sender | `av_transport/av_trans_engine/av_sender/` | 发送引擎 |
| AV Receiver | `av_transport/av_trans_engine/av_receiver/` | 接收引擎 |
| Pipeline | `av_transport/framework/` | 传输框架 |

---

## 版本信息

- **当前版本**: 4.0
- **最后更新**: 2025-02-06
- **OpenHarmony 分支**: master

---

## 贡献者

[查看 Git 历史](../../commits/main)

---

## 许可证

[Apache License 2.0](../../LICENSE)
