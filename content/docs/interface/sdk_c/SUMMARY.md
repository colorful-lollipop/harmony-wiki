# OpenHarmony SDK C Wiki 导航

> **新人阅读路线** - 按此顺序阅读，快速掌握项目全貌

---

## 📖 推荐阅读顺序

### 第一站：项目概览（5分钟）
1. **[README.md](./README.md)** - Wiki 简介与覆盖范围
2. **[00_Overview.md](./00_Overview.md)** - 项目定位、边界、核心能力

### 第二站：结构与架构（15分钟）
3. **[01_Directory_Structure.md](./01_Directory_Structure.md)** - 目录结构与模块职责
4. **[02_Architecture.md](./02_Architecture.md)** - 组件图、数据流、线程模型

### 第三站：API 文档（按需查阅）
5. **[03_NAPI_Reference.md](./03_NAPI_Reference.md)** - 对外 N-API 接口文档
6. **[04_Internal_API.md](./04_Internal_API.md)** - 内部 API 与模块接口

### 第四站：构建与产物（10分钟）
7. **[05_GN_Build.md](./05_GN_Build.md)** - GN 构建目标与编译配置
8. **[06_Build_Artifacts.md](./06_Build_Artifacts.md)** - 编译产物与安装路径

### 第五站：安全与问题（按需查阅）
9. **[07_Security_Analysis.md](./07_Security_Analysis.md)** - 安全风险评审
10. **[08_Common_Issues.md](./08_Common_Issues.md)** - 常见问题与定位

---

## 📚 文档索引

| 文档 | 内容摘要 | 目标读者 |
|------|----------|----------|
| [README.md](./README.md) | Wiki 简介、覆盖范围、更新方式 | 所有读者 |
| [00_Overview.md](./00_Overview.md) | 项目定位、核心能力、运行环境、关键概念 | 新接触项目者 |
| [01_Directory_Structure.md](./01_Directory_Structure.md) | 完整目录树、30+ 模块职责 | 需要了解模块划分者 |
| [02_Architecture.md](./02_Architecture.md) | 组件关系图、数据流、线程模型、关键时序 | 架构设计者 |
| [03_NAPI_Reference.md](./03_NAPI_Reference.md) | N-API 清单表、参数、返回值、错误码、权限 | N-API 开发者 |
| [04_Internal_API.md](./04_Internal_API.md) | 模块接口、依赖方向、稳定性标注 | 内部开发者 |
| [05_GN_Build.md](./05_GN_Build.md) | GN Targets 列表、类型、依赖、配置 | 构建维护者 |
| [06_Build_Artifacts.md](./06_Build_Artifacts.md) | 产物清单、安装路径、运行时加载关系 | 发布工程师 |
| [07_Security_Analysis.md](./07_Security_Analysis.md) | 攻击面、信任边界、可被利用点、修复建议 | 安全工程师 |
| [08_Common_Issues.md](./08_Common_Issues.md) | 构建/运行/调试问题与定位路径 | 问题排查者 |

### 附录
| 文档 | 内容摘要 |
|------|----------|
| [appendix/Callgraphs.md](./appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑） |
| [appendix/Config_Flags.md](./appendix/Config_Flags.md) | 关键宏/feature flags |

---

## 🔍 快速定位

### 按模块查找

| 功能域 | 对应 Wiki 章节 |
|--------|---------------|
| **ArkUI / UI 开发** | [03_NAPI_Reference.md#arkui](./03_NAPI_Reference.md#arkui) |
| **多媒体（音视频）** | [03_NAPI_Reference.md#multimedia](./03_NAPI_Reference.md#multimedia) |
| **图形（OpenGL/绘制）** | [03_NAPI_Reference.md#graphic](./03_NAPI_Reference.md#graphic) |
| **网络（HTTP/WebSocket）** | [03_NAPI_Reference.md#network](./03_NAPI_Reference.md#network) |
| **安全（密钥/证书）** | [03_NAPI_Reference.md#security](./03_NAPI_Reference.md#security) |
| **存储（RDB/Preferences）** | [03_NAPI_Reference.md#distributeddatamgr](./03_NAPI_Reference.md#distributeddatamgr) |
| **日志/调试** | [03_NAPI_Reference.md#hiviewdfx](./03_NAPI_Reference.md#hiviewdfx) |

### 按问题类型查找

| 问题类型 | 对应文档 |
|----------|----------|
| 如何添加新的 C API？ | [05_GN_Build.md#添加新模块](./05_GN_Build.md#添加新模块) |
| 编译失败怎么处理？ | [08_Common_Issues.md#构建问题](./08_Common_Issues.md#构建问题) |
| 运行时库加载失败？ | [08_Common_Issues.md#运行问题](./08_Common_Issues.md#运行问题) |
| API 安全风险分析 | [07_Security_Analysis.md](./07_Security_Analysis.md) |
| 头文件找不到？ | [01_Directory_Structure.md#头文件组织](./01_Directory_Structure.md#头文件组织) |

---

## 📊 关键数据速查

### 项目规模
- **模块数**: 30+ 个主要模块
- **公开头文件**: 394+ 个（排除 third_party）
- **NDK API 定义**: 110+ 个 .ndk.json 文件
- **GN 构建目标**: 287+ 个 target（ndk_targets.gni）

### 核心目录
```
interface_sdk_c/
├── arkui/              # ArkUI 框架（最丰富 API）
├── multimedia/         # 多媒体（最大模块）
├── graphic/            # 图形（OpenGL/Vulkan）
├── security/           # 安全
├── network/            # 网络
├── hiviewdfx/          # 日志/调试
├── docs/               # 官方文档
└── wiki/               # 本 Wiki
```

---

## 🔄 版本信息

| 项目 | 版本/时间 |
|------|----------|
| Wiki 生成时间 | 2025-02-06 |
| 文档版本 | 1.0 |
| 代码仓库 | interface_sdk_c |

---

## 📝 贡献指南

发现 Wiki 内容有误或需要补充？

1. **确认问题**: 检查对应的代码文件确认问题
2. **提供证据**: 提供代码证据（文件路径 + 行号）
3. **提交反馈**: 创建 Issue 描述问题

---

**开始阅读**: 建议从 [00_Overview.md](./00_Overview.md) 开始 →
