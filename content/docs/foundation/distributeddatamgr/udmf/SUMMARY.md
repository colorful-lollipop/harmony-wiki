# 文档导航

本文档为 UDMF Wiki 的全局导航索引，列出了所有可用文档及其阅读顺序建议。

## 阅读路线图

新人建议按以下顺序阅读，以建立对 UDMF 的完整认知：

```
附录
├── Appendix/Callgraphs.md     ← 关键调用链图示
```

## 文档列表

### 第一部分：架构与概念

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [00_Overview.md](./00_Overview.md) | 项目概述、核心能力、技术特点 | 必读 |
| [01_Directory_Structure.md](./01_Directory_Structure.md) | 目录结构、模块职责 | 必读 |
| [02_Data_Types.md](./02_Data_Types.md) | UTD 类型体系、类型层次关系 | 推荐 |

### 第二部分：接口规范

| 文档 | 描述 | 适用场景 |
|------|------|----------|
| [10_NAPI_Reference.md](./10_NAPI_Reference.md) | N-API 模块注册、导出方法、异步回调 | JS/ArkTS 应用开发 |
| [11_NDK_Reference.md](./11_NDK_Reference.md) | NDK C API、数据操作函数、错误码 | Native 应用开发 |
| [12_InnerKit_Reference.md](./12_InnerKit_Reference.md) | C++ 接口、数据结构定义、系统组件 | 系统组件开发 |

### 第三部分：实现原理

| 文档 | 描述 | 主题 |
|------|------|------|
| [20_Service_Layer.md](./20_Service_Layer.md) | UdmfService、IPC 代理、服务生命周期 | IPC 通信 |
| [21_Client_Layer.md](./21_Client_Layer.md) | UdmfClient、UtdClient、单例管理 | 客户端架构 |
| [22_Core_Data_Structures.md](./22_Core_Data_Structures.md) | UnifiedData、UnifiedRecord、TLV 序列化 | 核心数据结构 |

### 第四部分：工程配置

| 文档 | 描述 | 内容 |
|------|------|------|
| [30_GN_Build_Targets.md](./30_GN_Build_Targets.md) | BUILD.gn 目标清单、依赖关系 | 构建系统 |
| [31_Build_Artifacts.md](./31_Build_Artifacts.md) | .so/.a 文件、安装路径、加载关系 | 编译产物 |

### 第五部分：安全评估

| 文档 | 描述 | 目标读者 |
|------|------|----------|
| [40_Security_Analysis.md](./40_Security_Analysis.md) | 攻击面分析、信任边界、数据流 | 安全架构师 |
| [41_Security_Risks.md](./41_Security_Risks.md) | 风险点列表、修复建议 | 开发者 |

## 附录

| 文档 | 描述 |
|------|------|
| [Appendix/Callgraphs.md](./Appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑） |

> **说明**：特性开关与编译宏已在「GN 构建目标」文档中覆盖。

## 术语表

本 Wiki 使用的主要术语：

| 术语 | 含义 |
|------|------|
| UDMF | Unified Data Management Framework，统一数据管理框架 |
| UTD | Uniform Type Descriptor，统一类型描述符 |
| UDS | Unified Data Structure，统一数据结构 |
| N-API | Node.js API，OpenHarmony 的 JS/Native 接口层 |
| NDK | Native Development Kit，Native 应用开发套件 |
| InnerKit | 系统内部组件接口 |
| Intention | 数据使用意图（drag、pasteboard、dataHub 等） |
| ShareOption | 共享选项（InApp、CrossApp） |

## 版本信息

- 文档版本：1.0.0
- 生成时间：2026-02-06
- 源码版本：对应 OpenHarmony UDMF 主线版本
