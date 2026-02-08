# device_attest Wiki

本文档是 OpenHarmony `device_attest` 模块的工程 Wiki，旨在帮助开发者快速理解项目架构、接口和安全特性。

## 生成信息

- **生成日期**: 2025-02-06
- **代码版本**: 基于仓库 `/test/xts/device_attest`
- **文档语言**: 中文
- **目标读者**: OpenHarmony 开发者、安全审计人员、系统集成商

## 覆盖范围

### 已覆盖内容
- 项目定位与核心能力
- 目录结构与模块职责
- 架构说明（组件图、数据流、线程模型）
- 对外 N-API 接口完整清单
- 内部 IPC 接口与客户端 SDK
- GN 构建目标与编译产物
- 安全风险分析与建议
- 常见问题与定位路径

### 未覆盖内容
- 测试代码细节（`test/` 目录）
- OEM 适配层的具体实现细节
- 云端服务器的交互协议细节

## 文档结构

```
wiki/
├── README.md           # 本文档 - Wiki 说明
├── SUMMARY.md          # 全站导航与阅读路线
├── index.md            # 首页概览
├── 01_Overview.md      # 项目定位与关键概念
├── 02_Architecture.md  # 架构说明
├── 03_NAPI.md          # N-API 接口文档
├── 04_Inner_API.md     # 内部 API 文档
├── 05_GN_Targets.md    # GN 构建目标与产物
├── 06_Security.md      # 安全风险评审
├── 07_Troubleshooting.md # 常见问题
└── appendix/
    └── Callgraphs.md   # 关键调用链
```

## 阅读建议

### 新人入门路线
1. [首页概览](index.md) - 快速了解项目
2. [项目定位](01_Overview.md) - 理解业务背景
3. [架构说明](02_Architecture.md) - 掌握整体架构
4. [N-API接口](03_NAPI.md) - 了解对外接口

### 开发者深入路线
1. [内部API](04_Inner_API.md) - 理解模块间接口
2. [GN构建](05_GN_Targets.md) - 掌握构建系统
3. [安全评审](06_Security.md) - 理解安全设计
4. [调用链附录](appendix/Callgraphs.md) - 代码追踪

### 安全审计路线
1. [项目定位](01_Overview.md) - 理解信任边界
2. [架构说明](02_Architecture.md) - 理解数据流
3. [安全评审](06_Security.md) - 全面安全分析
4. [N-API接口](03_NAPI.md) - 检查对外攻击面

## 更新维护

### 随代码更新文档
当代码发生以下变更时，需要同步更新本文档：
- N-API 接口增删改（更新 `03_NAPI.md`）
- 架构调整（更新 `02_Architecture.md`）
- 新增/删除 GN target（更新 `05_GN_Targets.md`）
- 安全相关代码变更（更新 `06_Security.md`）

### 文档生成工具
本文档基于代码直接生成，关键证据包括：
- 文件路径（如 `services/core/attest/attest_service.c`）
- 符号名称（函数、类、宏）
- 代码行号（必要时标注）

## 术语表

| 术语 | 说明 |
|------|------|
| device_attest | 设备认证模块，负责设备认证状态管理 |
| XTS | OpenHarmony 兼容性测试套件 (eXtended Test Suite) |
| SA | System Ability，系统 Ability |
| N-API | Native API，JS 与 C/C++ 的桥接接口 |
| Inner API | 内部模块间通信接口 |
| manuKey | 厂商密钥，从兼容性平台获取 |
| productId | 产品标识符 |
| token | 设备凭证，每台设备唯一 |

## 参考链接

- [OpenHarmony 兼容性平台](https://compatibility.openharmony.cn/)
- [README_zh.md](../README_zh.md) - 原始项目文档
- [bundle.json](../bundle.json) - 组件配置

---

*本文档由工程 Agent 自动生成，如有疑问请以代码为准。*
