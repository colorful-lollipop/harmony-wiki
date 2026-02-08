# 系统架构

## 2.1 整体架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                      OpenHarmony 应用层                         │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                   ACELite JS 运行时                        │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │  device_attest (JS Interface)                       │  │  │
│  │  │  - getAttestStatus() [异步]                        │  │  │
│  │  │  - getAttestStatusSync() [同步]                    │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              ↓                                  │
┌─────────────────────────────────────────────────────────────────┤
│                      Native Layer                               │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                   Kit Layer (JS Binding)                │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │  kit_device_attest.so                              │  │  │
│  │  │  - native_device_attest.cpp                        │  │  │
│  │  │  - JSI API 实现                                    │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              ↓                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                 Framework Layer (SAMgr)                   │  │
│  │  ┌──────────────────────┐  ┌─────────────────────────┐   │  │
│  │  │ devattest_client.so │  │  devattest_server.so    │   │  │
│  │  │  - 客户端代理        │  │  - SA Feature 实现    │   │  │
│  │  └──────────────────────┘  └─────────────────────────┘   │  │
│  │                              ↓                             │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │           devattest_service (可执行文件)            │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              ↓                                  │
┌─────────────────────────────────────────────────────────────────┤
│                      Core Layer                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                   devattest_core                         │  │
│  │  ┌────────────┐ ┌────────────┐ ┌────────────────────┐  │  │
│  │  │ 认证逻辑    │ │ 网络模块    │ │ 安全模块 (mbedtls)│  │  │
│  │  │ attest/    │ │ network/    │ │ security/         │  │  │
│  │  └────────────┘ └────────────┘ └────────────────────┘  │  │
│  │  ┌────────────┐ ┌────────────┐ ┌────────────────────┐  │  │
│  │  │ 适配层      │ │ 工具函数   │ │ HAL 接口          │  │  │
│  │  │ adapter/   │ │ utils/     │ │ (token/manuKey)   │  │  │
│  │  └────────────┘ └────────────┘ └────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2.2 平台差异

### Mini System (LiteOS-M)

```
┌─────────────┐
│ devattest_sdk.a │  ← 静态库
└──────┬──────┘
       ↓
┌─────────────┐
│ devattest_core │  ← 核心库 (static_library)
└──────┬──────┘
       ↓
┌─────────────────────────────────┐
│  HAL Token 接口                  │
│  - HalReadToken()               │
│  - HalWriteToken()              │
│  - HalGetManufactureKey()       │
└─────────────────────────────────┘
```

**特点**: 无 SAMgr 支持，静态链接

### Small System (LiteOS-A / Linux)

```
┌─────────────┐
│ kit_device_attest.so │  ← JS 绑定
└──────┬──────┘
       ↓
┌─────────────┐     IPC     ┌─────────────────┐
│ devattest_client.so │ ←──→ │ devattest_service │
└──────┬──────┘              └────────┬────────┘
       ↓                            ↓
┌─────────────┐              ┌─────────────┐
│ devattest_core.so │        │ devattest_server.so │
└──────┬──────┘              └──────┬──────┘
       ↓                            ↓
       └──────────┬────────────────┘
                  ↓
          ┌─────────────────┐
          │  HAL Token 接口  │
          └─────────────────┘
```

**特点**: SAMgr 管理的 SA 架构，支持 IPC 通信

---

## 2.3 数据流

### 认证流程 (Small System)

```
1. JS 调用
   getAttestStatus() / getAttestStatusSync()
   ↓
2. Kit Layer (kit_device_attest.so)
   JSI → GetAttestStatusAsync/Sync()
   ↓
3. Framework Client (devattest_client.so)
   IPC Proxy → SendRequest()
   ↓
4. SAMgr Router
   Feature ID 路由
   ↓
5. Framework Server (devattest_server.so)
   Feature 处理 → 调用 Core
   ↓
6. Core Layer (devattest_core.so)
   - 获取 token/manuKey/productId
   - 生成挑战码
   - 网络请求认证
   - 验证响应
   ↓
7. 返回结果
   AttestResultInfo 结构
```

---

## 2.4 线程模型

### Small System

| 层级 | 线程 | 职责 |
|------|------|------|
| JS 线程 | 主线程 | JS 事件处理，JSI 调用 |
| Async Work | 线程池 | 异步任务执行 |
| Service 线程 | 主线程 | SA 消息处理 |

### Mini System

| 层级 | 线程 | 职责 |
|------|------|------|
| 主线程 | 调用上下文 | 同步认证请求 |

---

## 2.5 依赖方向

```
JS Interface (kit_device_attest)
    ↓ deps
Framework (devattest_client)
    ↓ deps
Core (devattest_core)
    ↓ deps
├── mbedtls (加密)
├── cJSON (序列化)
├── libsec (安全)
├── parameter (系统参数)
└── hal_token (设备凭证)
```

---

## 2.6 关键组件

### Framework Layer

| 组件 | 类型 | 职责 |
|------|------|------|
| attest_framework_client_proxy.c | Client | IPC 客户端代理 |
| attest_framework_server.c | Server | SA Feature 实现 |
| attest_framework_feature.c | Feature | Feature 注册 |
| attest_framework_service.c | Service | 服务主循环 |

### Core Layer

| 组件 | 路径 | 职责 |
|------|------|------|
| attest_service_*.c | attest/ | 认证服务逻辑 |
| attest_adapter_*.c | adapter/ | 平台适配 |
| attest_security_*.c | security/ | 加密操作 |
| attest_network_*.c | network/ | 网络通信 |
| attest_utils_*.c | utils/ | 工具函数 |

---

## 相关跳转

- [03_JSI_API](03_JSI_API.md) - JS 接口详细说明
- [04_InnerAPI](04_InnerAPI.md) - Inner API 说明
- [05_Build](05_Build.md) - 构建配置
