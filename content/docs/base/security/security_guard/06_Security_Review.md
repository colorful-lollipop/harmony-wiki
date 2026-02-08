# SecurityGuard 安全风险评审

**文档版本**：3.1.0  
**最后更新**：2026-02-06  
**维护者**：SecurityGuard Team

---

## 1. 概述

本文档对 SecurityGuard 项目进行全面的安全风险评审，包括攻击面分析、信任边界划分、可利用点识别以及修复建议。所有风险点均可追溯到代码证据。

**评审范围**：
- N-API 接口层
- Inner API SDK 层
- SA 服务层
- 数据存储层

**证据来源**：`frameworks/js/napi/*.cpp`、`services/*/*.cpp`、`interfaces/inner_api/*.h`

---

## 2. 攻击面分析

### 2.1 外部输入点清单

| 攻击面 | 输入类型 | 入口函数 | 位置 |
|--------|----------|----------|------|
| JS API 参数 | 用户可控参数 | NapiGetModelResult | frameworks/js/napi/security_guard_napi.cpp:537 |
| JS API 参数 | 用户可控参数 | NapiQuerySecurityEvent | frameworks/js/napi/security_guard_napi.cpp:894 |
| JS API 参数 | 用户可控参数 | NapiReportSecurityInfo | frameworks/js/napi/security_guard_napi.cpp:318 |
| 策略文件 | 文件描述符 | NapiUpdatePolicyFile | frameworks/js/napi/security_guard_napi.cpp:633 |
| 订阅回调 | IPC 回调 | Subscribe | frameworks/js/napi/security_guard_napi.cpp:1254 |
| IPC 请求 | 序列化数据 | DataCollectManagerStub | services/data_collect/sa/*.cpp |
| IPC 请求 | 序列化数据 | SecurityCollectorManagerStub | services/security_collector/*.cpp |

### 2.2 信任边界划分

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            不可信区域                                        │
│                                                                              │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐               │
│  │   JS 应用层     │  │   外部进程      │  │   文件系统      │               │
│  │  (用户输入)     │  │  (IPC 客户端)   │  │  (用户文件)     │               │
│  └────────┬───────┘  └────────┬───────┘  └────────┬───────┘               │
│           │                   │                   │                          │
└───────────┼───────────────────┼───────────────────┼──────────────────────────┘
            │                   │                   │
            ▼                   ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           信任边界                                            │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ N-API 层                                                            │  │
│  │  • 参数解析与验证                                                     │  │
│  │  • 类型转换                                                          │  │
│  │  • 权限预处理                                                        │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                   │                                          │
│                                   ▼                                          │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ Inner API 层                                                         │  │
│  │  • 业务参数校验                                                      │  │
│  │  • 输入净化                                                          │  │
│  │  • IPC 调用封装                                                      │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                   │                                          │
│                                   ▼                                          │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ SA 服务层                                                            │  │
│  │  • 权限最终校验                                                      │  │
│  │  • 数据格式验证                                                      │  │
│  │  • 业务逻辑执行                                                      │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                   │                                          │
│                                   ▼                                          │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ 数据存储层                                                           │  │
│  │  • SQL 预处理                                                        │  │
│  │  • 文件路径验证                                                      │  │
│  │  • 数据持久化                                                        │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└───────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. 安全机制分析

### 3.1 权限控制机制

**证据来源**：`services/security_collector/src/security_collector_manager_service.cpp:393-402`

#### 3.1.1 权限校验实现

```cpp
int32_t SecurityCollectorManagerService::HasPermission(const std::string &permission)
{
    AccessToken::AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    int code = AccessToken::AccessTokenKit::VerifyAccessToken(callerToken, permission);
    if (code != AccessToken::PermissionState::PERMISSION_GRANTED) {
        return NO_PERMISSION;
    }
    return SUCCESS;
}
```

#### 3.1.2 权限校验调用点

| 方法 | 权限检查 | 行号 | 完整性 |
|------|----------|------|--------|
| Subscribe | COLLECT_SECURITY_EVENT / QUERY_SECURITY_EVENT | 115 | ✅ |
| Unsubscribe | COLLECT_SECURITY_EVENT / QUERY_SECURITY_EVENT | 149 | ✅ |
| CollectorStart | COLLECT_SECURITY_EVENT | 175 | ✅ |
| CollectorStop | COLLECT_SECURITY_EVENT | 226 | ✅ |
| QuerySecurityEvent | QUERY_SECURITY_EVENT | 351-356 | ✅ |
| AddFilter | QUERY_SECURITY_EVENT | 407 | ✅ |
| RemoveFilter | QUERY_SECURITY_EVENT | 422 | ✅ |

### 3.2 输入验证机制

**证据来源**：`services/data_collect/sa/data_format.cpp:30-44`

#### 3.2.1 内容安全检查

```cpp
bool DataFormat::CheckRiskContent(std::string content)
{
    auto size = content.size();
    if (size > MAX_CONTENT_SIZE) {  // MAX_CONTENT_SIZE = 10240
        return false;
    }
    nlohmann::json jsonObj = nlohmann::json::parse(content, nullptr, false);
    if (jsonObj.is_discarded()) {
        return false;
    }
    return true;
}
```

#### 3.2.2 参数长度限制

**证据来源**：`frameworks/js/napi/security_guard_napi.cpp:47-59`

| 参数 | 最大长度 | 位置 |
|------|----------|------|
| version | 50 | VERSION_MAX_LEN |
| extra (content/param) | 2000 | EXTRA_MAX_LEN |
| fileName | 64 | FILE_NAME_MAX_LEN |
| modelName | 64 | MODEL_NAME_MAX_LEN |
| param | 900 | PARAM_MAX_LEN |
| timeString | 15 | TIME_MAX_LEN |

### 3.3 路径安全机制

**证据来源**：`frameworks/common/utils/src/file_util.cpp:26`

```cpp
// 路径规范化检查
std::string FileUtil::PathToRealPath(const std::string &path)
{
    char realPath[PATH_MAX] = {0};
    if (realpath(path.c_str(), realPath) == nullptr) {
        return "";
    }
    return std::string(realPath);
}
```

#### 3.3.1 库加载路径限制

**证据来源**：`services/collector_manager/src/lib_loader.cpp:34-48`

```cpp
void *LibLoader::Load(const std::string &path)
{
    // 路径规范化
    std::string realPath = PathToRealPath(path);
    if (realPath.empty()) {
        return nullptr;
    }
    
    // 路径前缀检查 - 限制在 /system/lib 下
    if (realPath.rfind("/system/lib/", 0) != 0) {
        SGLOGE("Invalid library path: %{public}s", realPath.c_str());
        return nullptr;
    }
    
    // 安全加载模式
    void *handle = dlopen(realPath.c_str(), RTLD_LAZY);
    return handle;
}
```

### 3.4 SQL 注入防护

**证据来源**：`services/data_collect/store/src/database_helper.cpp:289-299`

```cpp
std::string DatabaseHelper::FilterSpecialChars(const std::string &input)
{
    std::string filtered;
    for (auto c : input) {
        // 只允许字母、数字、下划线、百分号
        if (isalnum(c) || c == '_' || c == '%') {
            filtered += c;
        }
    }
    return filtered;
}
```

### 3.5 IPC 接口令牌验证

**证据来源**：`services/security_collector/src/security_collector_manager_stub.cpp`

```cpp
ErrCode SecurityCollectorManagerStub::Subscribe(
    const MessageParcel &data, const MessageParcel &reply)
{
    // 接口令牌验证
    std::u16string token = data.ReadInterfaceToken();
    if (!ifaceTokenCheck(token)) {
        SGLOGE("Interface token check failed");
        return ERR_INVALID_VALUE;
    }
    
    // 参数验证
    std::unique_ptr<SecurityCollectorSubscribeInfo> info(
        data.ReadParcelable<SecurityCollectorSubscribeInfo>());
    if (info == nullptr) {
        return ERR_INVALID_VALUE;
    }
    
    auto callback = data.ReadRemoteObject();
    if (callback == nullptr) {
        return ERR_INVALID_VALUE;
    }
    
    // 调用服务实现
    int32_t ret = Subscribe(*info, callback);
    reply.WriteInt32(ret);
    return ERR_OK;
}
```

---

## 4. 可利用点分析

### 4.1 高风险点

#### 风险点 1：动态库加载路径验证不完整

| 属性 | 值 |
|------|-----|
| 风险 ID | SG-SEC-001 |
| 风险等级 | 高 |
| 触发条件 | 攻击者控制采集器库路径 |
| 影响 | 任意代码执行 |

**证据来源**：`services/risk_classify/plugin_manager/src/detect_plugin_manager.cpp:60`

```cpp
void *handle = dlopen(path.c_str(), RTLD_LAZY);
if (handle == nullptr) {
    SGLOGE("Failed to load plugin: %{public}s", dlerror());
    return nullptr;
}
```

**问题**：虽然有 `PathToRealPath` 检查，但插件目录 `/system/lib64/` 的访问控制依赖系统权限。

**修复建议**：
1. 增加插件签名验证机制
2. 实现插件完整性校验
3. 使用代码签名框架验证插件合法性

---

#### 风险点 2：JSON 解析异常处理

| 属性 | 值 |
|------|-----|
| 风险 ID | SG-SEC-002 |
| 风险等级 | 中 |
| 触发条件 | 恶意构造的 JSON 内容 |
| 影响 | 拒绝服务 |

**证据来源**：`services/data_collect/sa/data_format.cpp:35-40`

```cpp
nlohmann::json jsonObj = nlohmann::json::parse(content, nullptr, false);
if (jsonObj.is_discarded()) {
    return false;  // 只返回 false，没有日志记录
}
```

**问题**：解析失败时仅返回 false，没有足够的日志记录，可能导致问题难以追踪。

**修复建议**：
1. 增加详细的错误日志
2. 实现解析异常的专门处理
3. 添加解析超时保护

---

#### 风险点 3：文件流检查绕过风险

| 属性 | 值 |
|------|-----|
| 风险 ID | SG-SEC-003 |
| 风险等级 | 中 |
| 触发条件 | 竞争条件导致文件状态变化 |
| 影响 | 文件读取异常 |

**证据来源**：`services/collector_manager/src/data_collection.cpp:340-357`

```cpp
ErrorCode DataCollection::CheckFileStream(std::ifstream &stream)
{
    if (!stream.is_open()) {
        return STREAM_ERROR;
    }
    stream.seekg(0, std::ios::end);
    std::ios::pos_type fileSize = stream.tellg();
    if (fileSize == 0 || fileSize > MAX_FILE_SIZE) {
        return STREAM_ERROR;
    }
    return SUCCESS;
}
```

**问题**：TOCTOU（Time-of-check to time-of-use）竞争条件。

**修复建议**：
1. 使用文件描述符代替文件流
2. 在检查和读取之间增加文件锁定
3. 使用 fstat() 获取精确文件大小

---

### 4.2 中风险点

#### 风险点 4：Token Bucket 限流绕过

| 属性 | 值 |
|------|-----|
| 风险 ID | SG-SEC-004 |
| 风险等级 | 低 |
| 触发条件 | 大量并发请求 |
| 影响 | 服务拒绝 |

**证据来源**：`services/data_collect/sa/data_collect_manager_service.cpp`

```cpp
auto tokenBucketTask = [this]() {
    while (true) {
        if (tokenBucket_.load() < TOKEN_BUCKET_MAX_SIZE) {
            tokenBucket_.fetch_add(TOKEN_BUCKET_STEP_SIZE);
        }
        ffrt::this_task::sleep_for(std::chrono::milliseconds(TOKEN_BUCKET_INTERVAL_TIME));
    }
};
```

**问题**：单进程限流，无法防止分布式攻击。

**修复建议**：
1. 考虑全局限流方案
2. 增加请求特征识别
3. 实现更细粒度的限流策略

---

#### 风险点 5：字符串处理潜在问题

| 属性 | 值 |
|------|-----|
| 风险 ID | SG-SEC-005 |
| 风险等级 | 低 |
| 触发条件 | 特殊字符序列 |
| 行为异常 |

**证据来源**：多处使用 `strerror()` 的代码

```cpp
SGLOGE("Failed to load library: %{public}s", dlerror());
```

**问题**：`strerror()` 不是线程安全的。

**修复建议**：
1. 使用 `strerror_r()` 替代
2. 或使用 C++ 异常处理机制

---

### 4.3 低风险点

#### 风险点 6：错误信息泄露

| 属性 | 值 |
|------|-----|
| 风险 ID | SG-SEC-006 |
| 风险等级 | 低 |
| 触发条件 | 错误情况 |
| 内部路径泄露 | |

**证据来源**：`services/security_collector/src/security_collector_manager_stub.cpp`

```cpp
SGLOGE("Failed to read subscribe info");
```

**问题**：错误日志可能泄露内部路径信息。

**修复建议**：
1. 统一错误信息格式
2. 敏感信息脱敏处理
3. 实施分级日志策略

---

## 5. 安全审计事件

### 5.1 Hisysevent 定义

**证据来源**：`hisysevent.yaml`

| 事件名称 | 类型 | 级别 | 描述 |
|---------|------|------|------|
| OBTAIN_DATA | STATISTIC | CRITICAL | 获取详细数据 |
| RISK_ANALYSIS | STATISTIC | CRITICAL | 获取设备风险状态 |
| SG_EVENT_SUBSCRIBE | STATISTIC | CRITICAL | 安全事件订阅 |
| SG_EVENT_UNSUBSCRIBE | STATISTIC | CRITICAL | 安全事件取消订阅 |
| SC_EVENT_SUBSCRIBE | STATISTIC | CRITICAL | 采集器事件订阅 |
| SC_EVENT_UNSUBSCRIBE | STATISTIC | CRITICAL | 采集器事件取消订阅 |
| SG_UPDATE_CONFIG | STATISTIC | CRITICAL | 配置更新事件 |
| SG_EVENT_SET_MUTE | STATISTIC | CRITICAL | 设置事件静音 |
| SG_EVENT_SET_UNMUTE | STATISTIC | CRITICAL | 设置事件取消静音 |

### 5.2 审计日志字段

| 事件 | 关键字段 |
|------|----------|
| OBTAIN_DATA | CALLER_PID, CALL_TIME, EVENT_SIZE |
| RISK_ANALYSIS | CALLER_PID, CALL_TIME, EVENT_INFO, RISK_STATUS |
| SG_EVENT_SUBSCRIBE | CALLER_PID, CALL_TIME, EVENT_ID, SUB_RET |
| SG_UPDATE_CONFIG | CONFIG_PATH, CALL_TIME, RET |

---

## 6. 修复建议汇总

### 6.1 高优先级修复

| 风险 ID | 修复建议 | 预期效果 |
|---------|----------|----------|
| SG-SEC-001 | 实现插件签名验证机制 | 防止恶意插件加载 |
| SG-SEC-002 | 增加 JSON 解析异常日志 | 提高可追溯性 |

### 6.2 中优先级修复

| 风险 ID | 修复建议 | 预期效果 |
|---------|----------|----------|
| SG-SEC-003 | 使用文件描述符代替文件流 | 防止 TOCTOU 竞争 |
| SG-SEC-004 | 考虑全局限流方案 | 增强抗攻击能力 |

### 6.3 低优先级修复

| 风险 ID | 修复建议 | 预期效果 |
|---------|----------|----------|
| SG-SEC-005 | 使用线程安全字符串函数 | 提升代码健壮性 |
| SG-SEC-006 | 实施敏感信息脱敏 | 防止信息泄露 |

---

## 7. 安全最佳实践

### 7.1 开发规范

1. **参数验证**：所有外部输入必须经过验证
2. **最小权限**：使用最小必要权限运行服务
3. **防御性编程**：假设所有输入都是恶意的
4. **日志审计**：关键操作必须记录审计日志

### 7.2 代码审查清单

- [ ] 所有 API 入口都有参数验证
- [ ] 所有文件操作都有路径检查
- [ ] 所有数据库操作都使用预处理语句
- [ ] 所有 IPC 调用都有权限检查
- [ ] 所有敏感操作都有审计日志

### 7.3 测试建议

1. **模糊测试**：使用提供的 fuzz test 进行边界测试
2. **权限测试**：验证权限校验的正确性
3. **压力测试**：验证限流机制的有效性
4. **安全扫描**：定期进行静态安全扫描

---

## 8. 相关文档

| 文档 | 说明 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 项目概览 |
| [02_NAPI_Reference.md](./02_NAPI_Reference.md) | JS API 参考 |
| [03_Architecture.md](./03_Architecture.md) | 架构详解 |
| [04_Build.md](./04_Build.md) | 构建配置 |
| [05_Services.md](./05_Services.md) | SA 服务详解 |
