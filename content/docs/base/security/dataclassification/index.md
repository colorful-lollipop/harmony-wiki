# 项目概览 - Data Classification

## 模块定位

**dataclassification** 是 OpenHarmony 安全子系统的核心模块之一，负责为分布式服务提供**跨设备数据传输的管控策略**。

## 核心能力

### 数据风险等级管控

模块根据设备安全等级，映射并返回允许传输的数据风险等级：

| 设备安全等级 | 支持的数据风险等级 |
|-------------|------------------|
| SL5 | S0 ~ S4 |
| SL4 | S0 ~ S4 |
| SL3 | S0 ~ S3 |
| SL2 | S0 ~ S2 |
| SL1 | S0 ~ S1 |

> 证据来源：`dev_slinfo_adpt.c:280-305`

### 分布式安全校验

在分布式服务发起跨设备数据传输前，调用本模块校验：

1. 获取目标设备的安全等级
2. 映射为允许传输的最高数据风险等级
3. 返回校验结果给分布式服务
4. 分布式服务根据结果决定是否放行数据传输

## 关键特征

- **仅 Inner API**：不提供对外 JS/N-API，仅供系统内部调用
- **同步/异步双模式**：支持同步查询和异步回调两种方式
- **动态加载 SDK**：通过 `dlopen` 动态加载设备安全等级 SDK
- **线程安全**：使用 `pthread_mutex` 保护异步回调链表

## 依赖关系

### 上游依赖（调用方）
- 分布式服务（Distributed Service）

### 下游依赖（被调用方）
- `device_security_level` - 设备安全等级管理模块
- `access_token` - 权限管理
- `hilog` - 日志框架
- `c_utils` - 基础工具库

> 证据来源：`bundle.json:26-36`、`interfaces/inner_api/datatransmitmgr/BUILD.gn:57-62`

## 产物清单

| 产物 | 类型 | 说明 |
|-----|------|------|
| `libdata_transit_mgr.z.so` | 共享库 | 数据传输管控核心库 |
| `libdata_transit_mgr.a` | 静态库 | 链接产物（可选） |

## 相关文档

- [架构说明](Architecture.md) - 详细架构设计
- [内部 API](Inner_API.md) - 接口使用指南
- [安全评审](Security.md) - 安全风险分析
