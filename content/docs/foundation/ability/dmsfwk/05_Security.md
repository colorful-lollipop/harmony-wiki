# 安全风险评审

## 1. 威胁模型概述

### 1.1 系统边界

```
+------------------+     +------------------+     +------------------+
|   Local Device   |     |   Softbus        |     |  Remote Device   |
|                  |     |   (DSoftbus)     |     |                  |
| +--------------+ |     |                  |     | +--------------+ |
| | Application  | |     |                  |     | | Application  | |
| +--------------+ |     |                  |     | +--------------+ |
|       ^          |     |                  |     |       ^          |
|       | Trust    |     | Untrusted        |     |       | Trust    |
|       Boundary   |     | Network          |     |       Boundary   |
+------------------+     +------------------+     +------------------+
       Device Boundary              SA Boundary
```

### 1.2 信任边界

| 边界 | 信任级别 | 说明 |
|------|----------|------|
| 设备内应用 | 高 | 同设备应用，共享系统权限 |
| SA 服务 | 中 | 系统级服务，需权限校验 |
| 跨设备通信 | 低 | 网络传输，需加密和认证 |

## 2. 攻击面分析

### 2.1 N-API 输入点

| API | 输入类型 | 风险等级 |
|-----|----------|----------|
| `register` | ContinuationExtraParams | 中 |
| `connect` | ConnectOption | 高 |
| `sendMessage` | 消息内容 | 中 |
| `sendData` | 数据缓冲区 | 高 |
| `sendImage` | 图片数据 | 中 |
| `createStream` | StreamParams | 中 |
| `setSurfaceId` | Surface ID | 高 |

### 2.2 IPC 接口

| 接口 | 方法 | 风险等级 |
|------|------|----------|
| DistributedSchedService | StartAbility | 高 |
| DistributedSchedService | ConnectAbility | 高 |
| DistributedAbilityManager | Register | 中 |
| DistributedAbilityManager | UpdateConnectStatus | 低 |

### 2.3 文件操作

| 操作 | 路径 | 风险 |
|------|------|------|
| 配置文件读取 | `/etc/profile/` | 低 |
| 缓存文件 | `/data/` | 中 |

### 2.4 网络通信

| 协议 | 用途 | 加密 |
|------|------|------|
| Softbus | 设备发现和通信 | 已加密 |
| IPC (Binder) | 进程间通信 | 内核级安全 |

## 3. 权限控制

### 3.1 权限检查点

| 检查点 | 权限 | 说明 |
|--------|------|------|
| `VerifyAccessToken` | 系统能力权限 | 验证调用者权限 |
| `CheckPermission` | 分布式权限 | 检查分布式操作权限 |
| `BundleFlag` | 包信息标志 | 验证包状态 |
| `AppSignature` | 应用签名 | 验证应用签名 |

### 3.2 权限校验错误码

| 错误码 | 说明 |
|--------|------|
| 201 | `PERMISSION_DENIED` - 权限拒绝 |
| 202 | `ERR_NOT_SYSTEM_APP` - 非系统应用 |

**证据来源**：`interfaces/kits/napi/include/napi_error_code.h`, `services/dtbschedmgr/include/distributed_sched_permission.h`

## 4. 安全风险清单

### 4.1 高风险项

#### 风险 1: DeviceId 未校验

**证据**：`js_ability_connection_manager.cpp` - `JSToConnectOption` 函数

```cpp
// 未校验 deviceId 的有效性和来源
JSToConnectOption(const napi_env &env, const napi_value &jsValue, ConnectOption &option)
{
    // deviceId 直接从 JS 参数获取，未验证是否属于可信设备
    napi_get_value_string_utf8(env, jsValue, option.deviceId, ...);
}
```

**触发条件**：
- 恶意应用传入伪造的 deviceId
- 连接到未授权的远程设备

**影响**：
- 潜在的数据泄露
- 连接到恶意设备

**修复建议**：
1. 在连接前验证 deviceId 的合法性
2. 检查 deviceId 是否在可信设备列表中
3. 添加设备认证机制

#### 风险 2: 消息内容未校验

**证据**：`js_ability_connection_manager.cpp` - `SendMessage` 函数

```cpp
// 消息内容未进行长度限制和内容校验
SendMessage(napi_env env, napi_callback_info info)
{
    napi_get_value_string_utf8(env, jsValue, message, ...);
    // 无最大长度限制
}
```

**触发条件**：
- 发送超大消息导致资源耗尽
- 发送格式错误的消息

**影响**：
- 拒绝服务攻击
- 内存溢出

**修复建议**：
1. 添加消息长度限制
2. 对消息内容进行格式校验
3. 添加速率限制

#### 风险 3: SurfaceId 注入

**证据**：`js_ability_connection_manager.cpp` - `SetSurfaceId` 函数

```cpp
// SurfaceId 未校验来源合法性
SetSurfaceId(napi_env env, napi_callback_info info)
{
    napi_get_value_string_utf8(env, jsValue, surfaceId, ...);
    // 未验证 surfaceId 是否属于当前应用
}
```

**触发条件**：
- 恶意应用获取其他应用的 SurfaceId
- 设置伪造的 SurfaceId

**影响**：
- 跨应用数据访问
- 界面劫持

**修复建议**：
1. 验证 SurfaceId 是否属于调用者
2. 使用安全的 SurfaceId 生成机制
3. 添加权限检查

### 4.2 中风险项

#### 风险 4: 回调注册无身份验证

**证据**：`js_ability_connection_manager.h` - `AsyncConnectCallbackInfo`

```cpp
struct AsyncConnectCallbackInfo {
    napi_async_work asyncWork = nullptr;
    napi_deferred deferred = nullptr;
    napi_threadsafe_function tsfn = nullptr;
    int32_t sessionId;
    ConnectResult result;
    // 无调用者身份验证
};
```

**触发条件**：
- 恶意应用注册回调
- 回调被劫持

**影响**：
- 回调信息泄露
- 状态篡改

**修复建议**：
1. 回调注册时验证调用者身份
2. 使用安全的消息队列传递回调结果

#### 风险 5: ContinuationExtraParams JSON 解析

**证据**：`js_continuation_manager.cpp` - `UnWrapContinuationExtraParams`

```cpp
// JSON 解析可能受到恶意输入影响
UnWrapContinuationExtraParams(const napi_env &env, const napi_value& options,
    std::shared_ptr<ContinuationExtraParams>& continuationExtraParams)
{
    nlohmann::json jsonObj;
    // 未限制 JSON 复杂度
}
```

**触发条件**：
- 发送超大或嵌套过深的 JSON
- JSON 中包含恶意数据

**影响**：
- 解析器崩溃
- 资源耗尽

**修复建议**：
1. 添加 JSON 大小和深度限制
2. 使用安全的 JSON 解析库
3. 捕获并处理解析异常

### 4.3 低风险项

#### 风险 6: 日志信息泄露

**证据**：`common/include/dtbschedmgr_log.h` - 日志宏定义

```cpp
// 日志可能输出敏感信息
DTBSCHED_LOGI("Connect to device: %{public}s", deviceId.c_str());
```

**影响**：
- 设备信息泄露
- 连接状态泄露

**修复建议**：
1. 对敏感信息使用 `%{private}s` 格式
2. 添加日志脱敏机制

#### 风险 7: 错误信息泄露

**证据**：`js_ability_connection_manager.cpp` - `ErrorMessageReturn`

```cpp
// 错误信息可能包含内部细节
ErrorMessageReturn(int32_t code) {
    // 返回详细错误信息
}
```

**影响**：
- 系统信息泄露
- 攻击者利用内部信息

**修复建议**：
1. 对外只返回通用错误码
2. 详细错误信息只在调试模式输出

## 5. 数据流安全

### 5.1 敏感数据流

```
+----------------+     +-------------+     +-----------------+
| Local Device   | --> | Softbus     | --> | Remote Device   |
| Application    |     | (Encrypted) |     | Application     |
+----------------+     +-------------+     +-----------------+
        |                      |                      |
        v                      v                      v
    [Sensitive Data]      [Encrypted Data]      [Sensitive Data]
```

### 5.2 加密要求

| 数据类型 | 加密方式 | 说明 |
|----------|----------|------|
| 消息内容 | Softbus 加密 | 传输层加密 |
| 设备认证 | 证书认证 | 设备间认证 |
| 用户数据 | 应用层加密 | 端到端加密 |

## 6. 安全最佳实践

### 6.1 输入验证

```cpp
// 示例：参数校验
int32_t CheckConnectOption(const ConnectOption &connectOption) {
    if (connectOption.deviceId.empty()) {
        return PARAMETER_CHECK_FAILED;
    }
    if (connectOption.deviceId.length() > MAX_DEVICE_ID_LENGTH) {
        return PARAMETER_CHECK_FAILED;
    }
    // 设备认证
    if (!VerifyDevicePermission(connectOption.deviceId)) {
        return PERMISSION_DENIED;
    }
    return 0;
}
```

### 6.2 权限检查

```cpp
// 示例：权限校验
bool IsSystemApp() {
    auto tokenId = IPCSkeleton::GetCallingTokenId();
    int32_t flag = AccessToken::CheckTokenTypeFlag(tokenId, 
        AccessToken::TokenTypeFlag::TOKEN_NATIVE);
    return flag == 0;
}
```

### 6.3 数据脱敏

```cpp
// 日志脱敏
DTBSCHED_LOGI("Connect to device: %{private}s", 
    GetMaskedDeviceId(deviceId).c_str());
```

## 7. 安全测试建议

### 7.1 测试场景

| 场景 | 测试内容 | 预期结果 |
|------|----------|----------|
| 权限绕过 | 未授权访问 | 返回 201 错误 |
| 参数注入 | 恶意参数 | 参数校验失败 |
| 消息洪泛 | 大量消息 | 速率限制生效 |
| 设备伪造 | 伪造设备连接 | 连接拒绝 |

### 7.2 工具建议

| 工具 | 用途 |
|------|------|
| Fuzz 测试 | 参数校验 |
| 渗透测试 | 安全漏洞 |
| 代码审计 | 安全代码检查 |

## 8. 安全更新建议

### 8.1 短期修复

1. 添加 deviceId 校验机制
2. 添加消息长度限制
3. 验证 SurfaceId 归属
4. 添加回调身份验证

### 8.2 中期改进

1. 完善 JSON 解析安全
2. 添加速率限制
3. 增强日志脱敏
4. 完善错误处理

### 8.3 长期规划

1. 引入设备认证体系
2. 实现端到端加密
3. 建立安全监控体系
4. 安全编码规范培训

## 9. 局限性声明

### 9.1 检查范围

本次安全评审覆盖以下代码：
- `interfaces/kits/napi/` - N-API 接口层
- `services/dtbschedmgr/` - 服务主逻辑
- `services/dtbabilitymgr/` - 能力管理服务
- `services/dtbcollabmgr/` - 协作管理服务
- `common/` - 公共模块

### 9.2 未覆盖范围

以下内容未纳入本次评审：
- `test/` - 测试代码
- `frameworks/native/distributed_extension/` - 框架层（部分）
- 第三方依赖库

### 9.3 评审方法

- 静态代码分析
- 模式匹配分析
- API 调用链路分析
- 威胁建模

**证据来源**：代码扫描结果和安全模式匹配分析
