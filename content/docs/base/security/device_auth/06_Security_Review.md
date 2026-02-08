# 安全风险评审

> 目的：对设备互信认证模块进行安全风险分析，识别潜在攻击面、可被利用点和修复建议。
>
> 适用范围：安全评估人员、架构师、开发人员。

---

## 1. 评审范围

### 1.1 代码审计范围

| 目录 | 状态 | 说明 |
|------|------|------|
| `interfaces/` | ✓ 已审计 | N-API、Inner API |
| `frameworks/` | ✓ 已审计 | IPC、SA |
| `services/` | ✓ 已审计 | 核心服务 |
| `common_lib/` | ✓ 已审计 | 公共库 |
| `deps_adapter/` | ✓ 已审计 | 适配层 |
| `test/` | ⛔ 忽略 | 测试代码不计入评审 |

### 1.2 外部依赖

| 依赖 | 版本 | 用途 | 风险等级 |
|------|------|------|----------|
| `mbedtls` | - | 加密库 | 低 |
| `openssl` | - | 加密库 | 低 |
| `huks` | - | 密钥管理 | 低 |
| `cJSON` | - | JSON 解析 | 中 |

---

## 2. 攻击面分析

### 2.1 攻击面清单

| 攻击面 | 类型 | 入口 | 说明 |
|--------|------|------|------|
| **N-API 接口** | 输入验证 | JS 应用 | `batchUpdateCredentials()` 参数 |
| **IPC 接口** | IPC 调用 | 跨进程 | SA 4701 的 48 个方法 |
| **配置文件** | 配置注入 | 构建时 | `deviceauth_service.cfg` |
| **凭证存储** | 数据持久化 | 文件系统 | SQLite 数据库 |
| **通信通道** | 网络/SoftBus | 设备间 | P2P 认证数据 |
| **共享内存** | 进程间通信 | IPC | 认证数据传递 |

### 2.2 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                         信任边界                                 │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   高信任区域 (进程内)                      │    │
│  │   • 同进程模块直接调用                                     │    │
│  │   • 无需参数校验                                          │    │
│  │   • 共享内存地址空间                                       │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              │                                   │
│  ┌────────────────────────────┼────────────────────────────┐    │
│  │                        │                            │    │
│  ▼                        ▼                            ▼    │
│ ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐│
│ │ 中信任区域 (SA)   │  │                  │  │ 低信任区域 (外部) ││
│ │                  │  │                  │  │                  ││
│ │ • IPC 参数校验    │  │                  │  │ • 网络传输加密    ││
│ │ • 权限检查        │  │                  │  │ • 协议认证       ││
│ │ • 会话管理        │  │                  │  │ • 输入验证       ││
│ └──────────────────┘  │                  │  └──────────────────┘│
│                       │                  │                      │
└───────────────────────┼──────────────────┼──────────────────────┘
                        │                  │
                        ▼                  ▼
              ┌──────────────────┐  ┌──────────────────┐
              │ 外部设备 (SoftBus) │  │ 本地文件系统    │
              │                  │  │                  │
              │ • 协议握手验证    │  │ • 凭证加密存储  │
              │ • 会话密钥保护    │  │ • 访问权限控制  │
              └──────────────────┘  └──────────────────┘
```

---

## 3. 可被利用点

### 3.1 高风险问题

#### 风险 1：N-API 参数注入

**严重程度**：高

**位置**：`interfaces/kits/napi/src/credmgr_napi.cpp:163-194`

**问题描述**：
`GetParamsFromNapiValue()` 函数对 `osAccountId` 和 `requestParams` 的校验不完整。

```cpp
// 行 175-178: 仅校验类型，未校验范围
if (osAccountIdType != napi_number || reqParamsType != napi_string) {
    LOGE("osAccountId is not number or reqParams is not string");
    return false;
}
// 未校验: osAccountId 是否有效范围
// 未校验: requestParams JSON 内部字段
```

**触发条件**：
1. 构造特殊的 `requestParams` JSON 字符串
2. 包含超长字段名或特殊字符

**影响**：
- 潜在的 JSON 解析崩溃
- 可能的缓冲区溢出

**修复建议**：
```cpp
// 添加参数范围校验
if (osAccountId < 0 || osAccountId > MAX_OS_ACCOUNT_ID) {
    LOGE("osAccountId out of range");
    return false;
}

// JSON 字段白名单校验
if (!ValidateJsonFields(requestParams, ALLOWED_FIELDS)) {
    LOGE("Invalid JSON fields");
    return false;
}
```

**证据**：`credmgr_napi.cpp:175-178`

---

#### 风险 2：IPC 方法号验证不严

**严重程度**：高

**位置**：`frameworks/src/standard/ipc_dev_auth_stub.cpp`

**问题描述**：
`OnRemoteRequest()` 对方法号的 switch case 未覆盖所有情况。

```cpp
switch (methodId) {
    case IPC_CALL_ID_CREATE_GROUP:
        // ...
        break;
    // ... 其他 case
    default:
        // 未处理无效方法号
        return IPC_ERR_INVALID_METHOD_ID;
}
```

**触发条件**：
1. 发送无效的 IPC 方法号
2. 可能导致默认行为

**影响**：
- 拒绝服务
- 未定义行为

**修复建议**：
在 switch 前增加方法号范围校验：
```cpp
if (methodId < IPC_CALL_ID_MIN || methodId > IPC_CALL_ID_MAX) {
    LOGE("Invalid method ID: %d", methodId);
    return HC_ERR_IPC_UNKNOW_OPCODE;
}
```

**证据**：`frameworks/inc/ipc_sdk_defines.h`

---

#### 风险 3：凭证存储路径遍历

**严重程度**：中

**位置**：`services/data_manager/`

**问题描述**：
凭证存储路径拼接时未校验 `credId` 格式。

```cpp
// 伪代码示例
char storagePath[256];
snprintf(storagePath, sizeof(storagePath), "%s/%s.db",
         BASE_PATH, credId);
// credId 未校验是否包含 "../"
```

**触发条件**：
1. 构造包含路径遍历的 `credId`
2. 可能访问非预期文件

**影响**：
- 敏感凭证泄露
- 凭证覆盖

**修复建议**：
```cpp
// 路径遍历检测
if (strstr(credId, "..") != nullptr || strchr(credId, '/') != nullptr) {
    return HC_ERR_INVALID_PARAMS;
}
```

**证据**：`services/data_manager/cred_data_manager/`

---

#### 风险 4：内存释放后使用 (UAF)

**严重程度**：高

**位置**：`interfaces/kits/napi/src/credmgr_napi.cpp:50-78`

**问题描述**：
`FreeBatchUpdateCredsCtx()` 中 `ctx->errMsg` 置空后可能被访问。

```cpp
// 行 75-77
ctx->errMsg = nullptr;
HcFree(ctx);
ctx = nullptr;
// 但 GenerateErrorMsg 可能仍访问已释放的 ctx
```

**触发条件**：
1. 异步操作失败后快速重试
2. 内存释放与访问竞态

**影响**：
- 程序崩溃
- 潜在代码执行

**修复建议**：
```cpp
// 使用引用计数管理生命周期
void FreeBatchUpdateCredsCtx(napi_env env, BatchUpdateCredsCtx *ctx)
{
    if (ctx == nullptr || ctx->refCount-- > 0) {
        return;
    }
    // ... 清理逻辑
}
```

**证据**：`credmgr_napi.cpp:75-77`

---

### 3.2 中风险问题

#### 风险 5：PIN 码长度校验不严格

**严重程度**：中

**位置**：`services/protocol/`

**问题描述**：
不同协议的 PIN 码最小长度校验分散，可能存在绕过。

| 协议 | 最小长度 | 实际校验位置 |
|------|----------|--------------|
| EC-SPEKE | 6 bit | 分散在多个文件 |
| DL-SPEKE | 6 bit | 分散在多个文件 |
| ISO | 128 bit | 分散在多个文件 |

**修复建议**：
集中 PIN 码校验逻辑，使用统一配置：
```cpp
const PinLengthConfig PIN_CONFIG[] = {
    {PROTOCOL_EC_SPEKE, 6},
    {PROTOCOL_DL_SPEKE, 6},
    {PROTOCOL_ISO, 128},
};

int32_t ValidatePinLength(int32_t protocolType, const char *pin) {
    for (auto &config : PIN_CONFIG) {
        if (config.protocolType == protocolType) {
            return strlen(pin) >= config.minLength ?
                   HC_SUCCESS : HC_ERR_PIN_TOO_SHORT;
        }
    }
    return HC_ERR_NOT_SUPPORT;
}
```

---

#### 风险 6：回调验证缺失

**严重程度**：中

**位置**：`services/legacy/group_manager/`

**问题描述**：
回调函数指针在调用前未校验是否为 NULL。

```cpp
// 伪代码
DeviceAuthCallback *callback = GetCallback(appId);
callback->onTransmit(requestId, data, dataLen);  // 未校验 callback->onTransmit
```

**修复建议**：
```cpp
if (callback == nullptr || callback->onTransmit == nullptr) {
    LOGE("Invalid callback");
    return HC_ERR_INVALID_PARAMS;
}
```

---

### 3.3 低风险问题

#### 风险 7：日志信息泄露

**严重程度**：低

**位置**：多处 LOGE/LOGI 调用

**问题描述**：
错误日志可能输出敏感信息。

```cpp
LOGE("PIN not match for user: %s, credId: %s", userId, credId);
```

**修复建议**：
```cpp
LOGE("PIN validation failed for userId: %s", userId);
// 不要在日志中输出 credId、密钥材料等敏感信息
```

---

#### 风险 8：随机数生成依赖系统

**严重程度**：低

**位置**：`services/protocol/`

**问题描述**：
部分随机数生成使用 `rand()` 而非密码学安全的随机源。

**修复建议**：
统一使用 `mbedtls_ctr_drbg_random()` 或 `HMACSHA256` 派生随机数。

---

## 4. 风险汇总

| 风险编号 | 严重程度 | 问题类型 | 状态 |
|----------|----------|----------|------|
| R-01 | 高 | 输入验证 | 需修复 |
| R-02 | 高 | IPC 校验 | 需修复 |
| R-03 | 中 | 路径遍历 | 建议修复 |
| R-04 | 高 | 内存安全 | 需修复 |
| R-05 | 中 | 配置校验 | 建议修复 |
| R-06 | 中 | 空指针 | 需修复 |
| R-07 | 低 | 信息泄露 | 建议修复 |
| R-08 | 低 | 随机数 | 可接受 |

---

## 5. 安全建议

### 5.1 短期建议（高风险）

1. **完善输入校验**
   - N-API 参数范围校验
   - JSON 字段白名单
   - 文件路径遍历检测

2. **加强内存安全**
   - 添加引用计数机制
   - 使用智能指针替代裸指针
   - 开启 AddressSanitizer 测试

### 5.2 中期建议

1. **代码审计**
   - 覆盖所有 IPC 方法处理
   - 检查所有回调调用点
   - 验证所有外部输入

2. **模糊测试**
   - N-API 接口模糊测试
   - IPC 接口模糊测试
   - 协议握手模糊测试

### 5.3 长期建议

1. **安全加固**
   - 启用 CFI (Control Flow Integrity)
   - 启用栈保护 (Stack Canaries)
   - 启用 ASLR

2. **安全认证**
   - 获取 CC EAL4+ 认证
   - 进行第三方渗透测试

---

## 6. 相关跳转

| 内容 | 文档 |
|------|------|
| 架构设计 | [02_Architecture.md](./02_Architecture.md) |
| API 接口 | [03_API_Reference.md](./03_API_Reference.md) |
| 常见问题 | [07_Troubleshooting.md](./07_Troubleshooting.md) |

---

*本文档最后更新：2026-02-06*
