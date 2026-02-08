# 导航索引

## 新人阅读路线

```
1. 先阅读 → 00_Overview.md    （理解项目定位）
2. 再阅读 → 01_Directory_Structure.md （熟悉目录结构）
3. 然后 → 02_Architecture.md   （理解整体架构）
4. 根据需求：
   - JS/ArkTS 开发者 → 03_NAPI.md
   - C++ 开发者 → 04_Inner_API.md
   - 构建/集成 → 05_GN_Build.md + 06_Build_Outputs.md
   - 安全审计 → 07_Security.md
   - 问题排查 → 08_Troubleshooting.md
```

## 文档目录

### 快速入门

| 文档 | 说明 |
|------|------|
| [README](README.md) | Wiki 使用说明 |
| [00_Overview](00_Overview.md) | 项目概览与核心能力 |

### 架构设计

| 文档 | 说明 |
|------|------|
| [01_Directory_Structure](01_Directory_Structure.md) | 目录结构与模块职责 |
| [02_Architecture](02_Architecture.md) | 组件图、数据流、线程模型 |

### API 接口

| 文档 | 说明 |
|------|------|
| [03_NAPI](03_NAPI.md) | JS API 接口文档 |
| [04_Inner_API](04_Inner_API.md) | Inner API 与 IPC 接口 |

### 构建部署

| 文档 | 说明 |
|------|------|
| [05_GN_Build](05_GN_Build.md) | GN 构建配置 |
| [06_Build_Outputs](06_Build_Outputs.md) | 编译产物与安装路径 |

### 安全与运维

| 文档 | 说明 |
|------|------|
| [07_Security](07_Security.md) | 安全风险评审 |
| [08_Troubleshooting](08_Troubleshooting.md) | 常见问题与故障排查 |

## 快速跳转

### N-API

- [导出函数列表](03_NAPI.md#api-清单) - 5 个 JS API
- [ConsumptionType 枚举](03_NAPI.md#consumptiontype-枚举) - 17 种耗电类型
- [错误码说明](03_NAPI.md#错误码)

### 构建

- [Targets 清单](05_GN_Build.md#n-api-层构建)
- [Feature Flags](05_GN_Build.md#feature-flags)
- [产物路径](06_Build_Outputs.md#产物清单)

### 安全

- [权限检查点](07_Security.md#权限检查)
- [攻击面分析](07_Security.md#攻击面分析)
- [风险清单](07_Security.md#可利用风险点)

### 运维

- [构建问题](08_Troubleshooting.md#构建问题)
- [运行时问题](08_Troubleshooting.md#运行时问题)
- [调试技巧](08_Troubleshooting.md#调试技巧)

## 代码证据索引

| 证据类型 | 位置 |
|----------|------|
| N-API 注册 | `frameworks/napi/src/battery_stats_module.cpp:155-170` |
| SA ID | `sa_profile/3304.json` |
| Service 类 | `services/native/include/battery_stats_service.h:33-41` |
| 实体类 | `services/native/include/entities/*.h` (15 个实体) |
| 错误码 | `interfaces/inner_api/include/battery_stats_errors.h` |
| 权限检查 | `services/native/src/battery_stats_service.cpp:195-351` |
| IPC 接口 | `services/IBatteryStats.idl` |
