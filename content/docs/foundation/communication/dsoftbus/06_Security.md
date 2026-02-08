# DSoftBus 安全风险评审

## 评审范围

本评审覆盖 DSoftBus 核心代码，不包含测试代码。

| 范围 | 状态 | 说明 |
|------|------|------|
| N-API 接口层 | ✅ 已评审 | `sdk/napi/` + `br_proxy/` |
| 核心模块 | ✅ 已评审 | `core/*/` |
| SDK 层 | ✅ 已评审 | `sdk/*/` |
| 测试代码 | ❌ 已排除 | 依规范不评审 |

---

## 攻击面分析

### 外部输入点

| 输入类型 | 来源 | 处理模块 |
|---------|------|---------|
| **JS API 参数** | 应用层 N-API 调用 | `sdk/napi/` |
| **设备发现数据** | BLE/CoAP/NFC/USB 广播 | `core/discovery/` |
| **连接地址** | 用户指定设备地址 | `core/connection/` |
| **传输数据** | 对端设备发送数据 | `core/transmission/` |
| **认证数据** | HiChain 认证协议 | `core/authentication/` |

### 信任边界

```
┌─────────────────────────────────────────────────┐
│                   应用层                         │
│  ┌───────────────────────────────────────────┐  │
│  │         N-API 接口 (信任边界)               │  │
│  └───────────────────────────────────────────┘  │
│                      ↓                          │
│  ┌───────────────────────────────────────────┐  │
│  │         softbus_client SDK                │  │
│  └───────────────────────────────────────────┘  │
│                      ↓                          │
│  ┌───────────────────────────────────────────┐  │
│  │         IPC 通信 (进程边界)                │  │
│  └───────────────────────────────────────────┘  │
│                      ↓                          │
│  ┌───────────────────────────────────────────┐  │
│  │         softbus_server                    │  │
│  │  ┌─────────────────────────────────────┐ │  │
│  │  │       核心模块 (认证/发现/连接/传输)   │ │  │
│  │  └─────────────────────────────────────┘ │  │
│  └───────────────────────────────────────────┘  │
│                      ↓                          │
│  ┌───────────────────────────────────────────┐  │
│  │         底层协议栈 (BT/WiFi/IPC)          │  │
│  └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

---

## 已识别风险

### 风险 1: N-API 参数校验不足

**严重程度**: 中

**证据**: `sdk/napi/link_enhance/src/napi_link_enhance_utils.cpp`

```cpp
// 行 54: CONN_CHECK_AND_RETURN_RET_LOGE
CONN_CHECK_AND_RETURN_RET_LOGE(name != nullptr, SOFTBUS_INVALID_PARAM, COMM_SDK, "name is nullptr");
```

**问题**: 部分 N-API 函数缺少完整的参数边界检查。

**触发条件**: 传入异常参数可能导致未定义行为。

**修复建议**: 
- 对所有 N-API 入口添加完整的参数校验
- 对字符串参数添加长度限制
- 对数组参数添加边界检查

---

### 风险 2: JS 对象生命周期管理

**严重程度**: 中

**证据**: `sdk/napi/link_enhance/src/napi_link_enhance_connection.cpp`

```cpp
// 行 46-47: 静态列表存储连接对象
static std::vector<NapiLinkEnhanceConnection *> connectionList_;
static std::mutex connectionListMutex_;
```

**问题**: 静态容器存储 JS 对象指针，缺少引用计数管理。

**触发条件**:
1. JS 对象被 GC 回收
2. 后续回调访问已释放对象

**修复建议**:
- 使用 `napi_reference` 跟踪 JS 对象生命周期
- 添加对象有效性检查

---

### 风险 3: 异步回调竞态条件

**严重程度**: 低

**证据**: `sdk/napi/link_enhance/src/napi_link_enhance_module.cpp`

```cpp
// 行 54-80: 异步 lambda 回调
auto func = [enhanceServer, serverName, deviceId, inHandle]() {
    // ...
    napi_new_instance(enhanceServer->env_, constructor, argc, argv, &argvOut[ARGS_SIZE_ZERO]);
};
return DoInJsMainThread(enhanceServer->env_, std::move(func));
```

**问题**: 异步回调中访问已释放的 `enhanceServer` 指针。

**触发条件**: 服务器对象在回调执行前被销毁。

**修复建议**:
- 添加对象存在性检查
- 使用智能指针管理生命周期

---

### 风险 4: 内存拷贝安全

**严重程度**: 低

**证据**: `sdk/napi/link_enhance/src/napi_link_enhance_module.cpp`

```cpp
// 行 167-169: memcpy_s
auto outData = std::shared_ptr<uint8_t>(new uint8_t[len], std::default_delete<uint8_t[]>());
if (outData == nullptr || memcpy_s(outData.get(), len, data, len) != EOK) {
    return;
}
```

**问题**: `memcpy_s` 返回值检查不够严格。

**修复建议**:
- 添加更详细的错误处理日志
- 考虑使用更安全的内存管理方案

---

### 风险 5: 错误码转换泄露内部信息

**严重程度**: 低

**证据**: `sdk/napi/link_enhance/src/napi_link_enhance_module.cpp`

```cpp
// 行 108-110: 错误码转换
int32_t napiReason = reason;
if (napiReason != 0) {
    napiReason = ConvertToJsErrcode(reason);
}
```

**问题**: 原始错误码可能泄露内部实现细节。

**修复建议**:
- 统一错误码转换逻辑
- 避免暴露敏感错误信息

---

## 安全机制评估

### 已实现的安全机制

| 机制 | 位置 | 说明 |
|-----|------|------|
| **权限校验** | `core/common/security/` | Access token 检查 |
| **设备认证** | `core/authentication/` | HiChain 认证协议 |
| **会话加密** | `core/authentication/` | 传输加密 |
| **签名校验** | `bundle.json` | 跨设备绑定前校验 |

### 未实现/可增强的安全机制

| 机制 | 当前状态 | 建议 |
|-----|---------|------|
| 输入数据过滤 | 部分实现 | 增强边界检查 |
| 速率限制 | 无 | 添加防 DDoS 机制 |
| 日志脱敏 | 无 | 敏感数据脱敏 |

---

## 权限要求

### 必选权限

| 权限 | 用途 |
|-----|------|
| `ohos.permission.DISTRIBUTED_DATASYNC` | 分布式数据同步 |
| `ohos.permission.DISTRIBUTED_SOFTBUS_CENTER` | 软总线中心管理 |

> 代码证据: `README.md` 第 58 行

### 潜在风险

- 无权限调用 N-API 可能返回错误码，但缺少权限校验日志
- 建议：增强权限拒绝的审计日志

---

## 风险等级说明

| 等级 | 描述 | 处理建议 |
|-----|------|---------|
| **高** | 可直接利用的漏洞 | 立即修复 |
| **中** | 可能被利用 | 计划修复 |
| **低** | 影响有限 | 条件允许时修复 |
| **信息** | 观察项 | 持续监控 |

---

## 安全建议总结

1. **高优先级**: 增强 N-API 参数校验，避免注入攻击
2. **高优先级**: 完善 JS 对象生命周期管理，防止 UAF
3. **中优先级**: 添加速率限制，防范资源耗尽
4. **中优先级**: 统一错误码转换，避免信息泄露
5. **低优先级**: 敏感日志脱敏

---

**相关文档**

- [项目概览](./01_Overview.md)
- [架构说明](./02_Architecture.md)
- [N-API 接口](./03_NAPI.md)
- [编译配置](./05_Build.md)
