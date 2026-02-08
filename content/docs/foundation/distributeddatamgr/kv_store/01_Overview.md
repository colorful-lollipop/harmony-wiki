# 项目概述

## 简介

**KV数据库（KV store）** 是 OpenHarmony 分布式数据管理子系统的核心组件，为设备应用提供键值对（Key-Value）数据管理能力。

### 核心特性

- **分布式同步**：支持跨设备数据同步
- **多模式存储**：
  - 单版本 KV 存储（`SINGLE_VERSION`）
  - 多版本 KV 存储（`MULTI_VERSION`）
  - 设备协同模式（`DEVICE_COLLABORATION`）
- **加密支持**：基于 SQLite Codec 的数据加密
- **查询能力**：支持谓词查询和结果集遍历

### 定位

> 依托当前公共基础库提供的 KV 存储能力开发，为设备应用提供键值对数据管理能力。在有进程的平台上，KV 存储作为基础库加载在应用进程，以保障不被其他进程访问。

## 目录结构

```
kv_store/
├── frameworks/           # 框架层代码
│   ├── common/          # 公共工具类
│   ├── innerkitsimpl/   # 部件间接口实现
│   │   ├── distributeddatafwk/  # 分布式数据框架
│   │   ├── distributeddatasvc/  # 分布式数据服务
│   │   └── kvdb/        # KV 数据库实现
│   ├── jskitsimpl/      # JS API 实现
│   │   ├── distributeddata/    # 分布式数据 JS 绑定
│   │   └── distributedkvstore/ # KV 存储 JS 绑定
│   ├── libs/            # 库实现
│   │   └── distributeddb/      # 分布式数据库核心库
│   ├── native/          # Native 接口
│   └── ets/            # ArkTS/ETS 接口
├── interfaces/         # 接口声明
│   ├── inner_api/      # 内部 Native API
│   ├── innerkits/      # 部件间接口
│   ├── jskits/         # JS 接口声明
│   └── cj/             # CJ (C-JavaScript) 接口
├── databaseutils/      # 数据库工具类
├── kvstoremock/        # Mock 组件
└── test/               # 测试用例
```

### 关键模块职责

| 模块 | 路径 | 职责 |
|-----|------|------|
| distributeddb | `frameworks/libs/distributeddb/` | 分布式数据库核心实现 |
| jskitsimpl | `frameworks/jskitsimpl/` | N-API 绑定实现 |
| innerkits | `interfaces/innerkits/` | 部件间接口声明 |
| distributeddatafwk | `frameworks/innerkitsimpl/distributeddatafwk/` | 数据管理框架 |

## 运行环境

### 系统要求

- **OpenHarmony 版本**：5.0+
- **适配系统类型**：standard（标准系统）
- **内存要求**：RAM 15,360KB

### 依赖组件

```
SystemCapability.DistributedDataManager.KVStore.Core
SystemCapability.DistributedDataManager.KVStore.DistributedKVStore
```

### 第三方依赖

| 依赖 | 用途 |
|-----|------|
| SQLite | 嵌入式数据库 |
| OpenSSL | 加密支持 |
| libuv | 事件循环 |
| bounds_checking_function | 安全字符串函数 |

## 关键概念

### StoreId

数据库实例标识符，用于区分不同的 KV 存储实例。

### Options

创建 KV 存储时的配置选项，包括：
- `kvStoreType`：存储类型（DEVICE_COLLABORATION / SINGLE_VERSION / MULTI_VERSION）
- `securityLevel`：安全级别（NO_LEVEL / S0 / S1 / S2 / S3 / S4）
- `encrypt`：是否加密
- `schema`：数据 Schema 定义

### SecurityLevel

安全级别枚举：

| 级别 | 值 | 描述 |
|-----|---|------|
| NO_LEVEL | 0 | 无安全级别 |
| S0 | 1 | 最低安全 |
| S1 | 2 | 低安全 |
| S2 | 3 | 中等安全 |
| S3 | 5 | 高安全 |
| S4 | 6 | 最高安全 |

### SyncMode

同步模式：

| 模式 | 值 | 描述 |
|-----|---|------|
| PULL_ONLY | 0 | 仅拉取 |
| PUSH_ONLY | 1 | 仅推送 |
| PUSH_PULL | 2 | 推送+拉取 |

## 约束与限制

> 以下信息摘自 `README_zh.md`

- **配置可修改**：KV 大小及可存储条目数在平台可承受内可修改配置
- **平台兼容**：针对不同平台（LiteOS-M、LiteOS-A 等）表现接口语义不变
- **数据库裁剪**：由于平台能力差异，数据库能力需要做相应裁剪
- **单例限制**：对于指定路径仅支持创建数据库单例

## 相关文档

- [代码地图](03_CodeMap.md) - 快速定位代码文件
- [N-API 接口参考](04_NAPI_Reference.md) - JS API详细文档
- [架构设计](02_Architecture.md) - 模块架构与数据流
- [GN 构建指南](06_Build_GN.md) - 构建配置与产物
- [攻击面分析](07_AttackSurface.md) - 安全入口点分析
