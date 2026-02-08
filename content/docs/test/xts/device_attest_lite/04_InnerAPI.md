# Inner API 接口

## 4.1 Inner API 清单

### C/C++ 接口

| 函数名 | 类型 | 描述 | 行号 |
|--------|------|------|------|
| `StartDevAttestTask()` | 异步 | 启动设备认证任务 | `devattest_interface.h:31` |
| `GetAttestStatus(AttestResultInfo*)` | 同步 | 获取认证结果 | `devattest_interface.h:38` |

**证据**: `interfaces/innerkits/devattest_interface.h:31, 38`

---

## 4.2 StartDevAttestTask

### 函数签名

```cpp
// 文件: interfaces/innerkits/devattest_interface.h:27-31

/**
 * @Desc Asynchronous interface, start device_attest task.
 * @Para Void.
 * @Return Returning 0 represents success, while returning other represents failure.
 */
int32_t StartDevAttestTask(void);
```

### 调用链

```
StartDevAttestTask()
    ↓
attest_entry.c:EntryStartDevAttestTask()
    ↓
ProcAttest() [attest_service.c]
    ↓
AttestStartup()
    ├── ResetAttestDevice()     // 重置认证
    ├── AuthAttestDevice()      // 认证设备
    └── ActiveToken()           // 激活 Token
```

### 实现位置

| 文件 | 行号 | 职责 |
|------|------|------|
| `services/core/attest_entry.c` | 28-42 | 入口点 |
| `services/core/attest_service.c` | 297-516 | 主流程 |

---

## 4.3 GetAttestStatus

### 函数签名

```cpp
// 文件: interfaces/innerkits/devattest_interface.h:33-38

/**
 * @Desc Synchronous interface, get the result of device_attest. And it consume about 10ms.
 * @Para Pointer to the structure of result of device_attest.
 * @Return Returning 0 represents success, while returning other represents failure.
 */
int32_t GetAttestStatus(AttestResultInfo* attestResultInfo);
```

### 参数说明

| 参数 | 类型 | 说明 |
|------|------|------|
| attestResultInfo | AttestResultInfo* | 输出参数，接收认证结果 |

### 返回值

| 返回值 | 说明 |
|--------|------|
| 0 | 成功 |
| 非0 | 失败 |

### AttestResultInfo 结构

```cpp
// 文件: interfaces/innerkits/attest_result_info.h:37-43

#define SOFTWARE_RESULT_DETAIL_SIZE   5
#define MAX_ATTEST_RESULT_SIZE       (SOFTWARE_RESULT_DETAIL_SIZE + 2)

typedef struct {
    int32_t authResult;                                   // 认证结果
    int32_t softwareResult;                              // 软件结果
    int32_t softwareResultDetail[SOFTWARE_RESULT_DETAIL_SIZE]; // 详细结果
    int32_t ticketLength;                                 // Ticket 长度
    char* ticket;                                         // Ticket 内容
} AttestResultInfo;
```

### 返回值字段说明

| 字段 | 描述 |
|------|------|
| authResult | 认证结果状态码 |
| softwareResult | 软件验证结果 |
| softwareResultDetail | 详细结果数组 (版本ID、补丁级别、RootHash、PCID、保留) |
| ticket | 认证票据 |

---

## 4.4 模块依赖方向

```
                    JS Interface (kit_device_attest)
                            ↓
                    Framework (devattest_client)
                            ↓
                    ┌─────────────────────────────────────────┐
                    │              Core Layer                 │
                    │  ┌─────────┐  ┌─────────┐  ┌─────────┐  │
                    │  │  attest │  │ network │  │security │  │
                    │  └─────────┘  └─────────┘  └─────────┘  │
                    └─────────────────────────────────────────┘
                            ↓
            ┌──────────────┴──────────────┐
            │                              │
     hal_token (Token)              parameter (系统参数)
```

### Core 层内部依赖

| 模块 | 路径 | 职责 | 依赖 |
|------|------|------|------|
| attest | `services/core/attest/` | 认证业务逻辑 | network, security |
| network | `services/core/network/` | 网络通信 (CoAP/TLS) | security |
| security | `services/core/security/` | 加密操作 | mbedtls |
| adapter | `services/core/adapter/` | 平台适配 | OEM 实现 |
| utils | `services/core/utils/` | 工具函数 | - |

---

## 4.5 稳定性标注

### 稳定接口 (Stable)

| 接口 | 路径 | 稳定性依据 |
|------|------|-----------|
| `StartDevAttestTask()` | `interfaces/innerkits/` | Inner API，头文件位于 innerkits |
| `GetAttestStatus()` | `interfaces/innerkits/` | Inner API，头文件位于 innerkits |
| `AttestResultInfo` | `interfaces/innerkits/` | 公共数据结构 |

### 内部接口 (Internal)

| 接口 | 路径 | 稳定性依据 |
|------|------|-----------|
| `ProcAttest()` | `services/core/attest/` | 仅限 core 内部使用 |
| `AttestStartup()` | `services/core/attest/` | 仅限 core 内部使用 |
| `EncryptAesCbc()` | `services/core/security/` | adapter 层以下实现 |

### 不稳定接口 (Unstable)

| 接口 | 路径 | 警告 |
|------|------|------|
| HAL 接口 | `services/core/adapter/` | OEM 实现可能变化 |

---

## 4.6 资源生命周期

### 初始化顺序

```
1. HAL 层初始化 (hal_token, parameter)
2. Core 层初始化 (attest_entry.c)
3. Framework 层注册 (SAMGR)
4. Kit 层初始化 (JS Module)
```

### 内存管理

| 类型 | 分配方式 | 释放时机 |
|------|----------|----------|
| AttestResultInfo.ticket | malloc() | 调用者负责 free |
| g_authResultCode | static 全局 | 进程退出 |
| g_mtxAttest | static 全局 | 进程退出 |

**证据**: `native_device_attest.cpp:155-156`

```cpp
free(attestResultInfo.ticket);
attestResultInfo.ticket = NULL;
```

---

## 4.7 可替换点

| 替换点 | 接口 | 路径 |
|--------|------|------|
| Token 存储 | `HalReadToken()` / `HalWriteToken()` | `adapter/attest_adapter_oem.c` |
| 设备信息 | `AttestGetDeviceInfo()` | `adapter/attest_adapter.c` |
| 系统参数 | `GetProductInfo()` | `adapter/attest_adapter.c` |
| 网络实现 | `D2CConnect()` | `network/attest_network.c` |

---

## 相关跳转

- [03_JSI_API](03_JSI_API.md) - JS 接口说明
- [02_Architecture](02_Architecture.md) - 架构概览
- [05_Build](05_Build.md) - 构建配置
