# 项目概览

## 1.1 项目定位

### 模块定位

`device_attest_lite` 是 OpenHarmony XTS 子系统中的设备认证模块，**核心职责**是：

1. **设备认证**: 验证设备的合法性和软件完整性
2. **生态统计**: 通过云端数据统计 OpenHarmony 生态设备数量

### 适用系统

| 系统类型 | 内核 | 最低要求 |
|---------|------|---------|
| Mini System | LiteOS-M | ROM: 3072KB, RAM: ~64KB |
| Small System | LiteOS-A / Linux | ROM: 3072KB, RAM: ~64KB |

**证据**: `bundle.json:22-27`

### 系统能力

```
SystemCapability.XTS.DeviceAttest.Lite
```

**证据**: `bundle.json:20`

---

## 1.2 核心能力

### 主要功能

| 功能 | 描述 | 支持平台 |
|------|------|---------|
| 启动认证任务 | `StartDevAttestTask()` | Small |
| 获取认证状态 | `GetAttestStatus()` | All |
| JS 接口 | `getAttestStatus`, `getAttestStatusSync` | Small |
| Token 管理 | 设备凭证读写 | All |

### 外部依赖

| 依赖库 | 版本 | 用途 |
|--------|------|------|
| mbedtls | 2.16.11 | TLS 协议实现 |
| cJSON | 1.7.15 | JSON 解析 |
| libsec | 1.1.10 | 安全函数 |
| parameter | OpenHarmony 1.0+ | 系统参数获取 |

**证据**: `README.md:48-56`

---

## 1.3 关键概念

### 术语表

| 术语 | 英文 | 描述 |
|------|------|------|
| 合作伙伴 | partners | 申请 OpenHarmony 兼容性评估的企业 |
| 制造商密钥 | manuKey | 从兼容性平台获取，用于加密保护产品数据 |
| 产品标识 | productId | 平台分配的产品唯一标识 |
| 产品密钥 | productKey | 平台分配的产品密钥 |
| 设备凭证 | token | 每个设备唯一的凭证，存储在安全分区 |

**证据**: `README.md:60-70`

### 认证结果

| 字段 | 类型 | 描述 |
|------|------|------|
| authResult | int32_t | 认证结果状态码 |
| softwareResult | int32_t | 软件验证结果 |
| softwareResultDetail | int32_t[5] | 详细结果数组 |
| ticket | char* | 认证票据字符串 |

**证据**: `interfaces/innerkits/attest_result_info.h:37-43`

---

## 1.4 目录结构

```
device_attest_lite/
├── build/                   # 构建配置
│   └── devattestconfig.gni  # 特性开关配置
├── figures/                 # 图片资源
├── framework/               # 系统能力服务框架
│   ├── mini/               # Mini 系统服务框架
│   └── small/              # Small 系统服务框架
│       ├── include/        # 框架头文件
│       ├── src/
│       │   ├── client/     # 客户端实现
│       │   └── service/    # 服务端实现
├── interfaces/             # 外部接口
│   ├── innerkits/          # Inner API
│   │   ├── devattest_interface.h
│   │   └── attest_result_info.h
│   └── kit/js/             # JS Kit 接口
│       ├── include/
│       │   └── native_device_attest.h
│       └── src/
│           └── native_device_attest.cpp
├── services/               # 服务主体与业务逻辑
│   └── core/               # 业务逻辑代码
│       ├── mini/           # Mini 系统实现
│       ├── small/          # Small 系统实现
│       ├── common/         # 公共代码
│       ├── adapter/        # 适配层
│       ├── attest/        # 认证逻辑
│       ├── security/       # 安全模块
│       ├── network/       # 网络模块
│       └── utils/          # 工具函数
└── test/                   # 测试 (不作为业务证据)
```

---

## 1.5 运行环境

### 硬件要求

| 资源 | Mini System | Small System |
|------|-------------|--------------|
| ROM | ≥3072KB | ≥3072KB |
| RAM | ~64KB | ~64KB |

### 软件依赖

| 组件 | 用途 |
|------|------|
| samgr_lite | 系统能力管理 |
| ipc | 进程间通信 |
| hilog_lite | 日志输出 |
| ace_engine_lite | JS 运行时 |
| init | 系统初始化 |
| syscap_codec | 能力编解码 |

**证据**: `bundle.json:29-36`

---

## 1.6 相关跳转

- [02_Architecture](02_Architecture.md) - 详细架构说明
- [03_JSI_API](03_JSI_API.md) - JS 接口文档
- [04_InnerAPI](04_InnerAPI.md) - Inner API 文档
