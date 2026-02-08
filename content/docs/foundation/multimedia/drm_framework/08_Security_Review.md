# 安全风险评审

> 本文档对 DRM Framework 进行安全风险分析，基于代码证据识别潜在攻击面和风险点。

## 评审范围

| 范围 | 说明 |
|------|------|
| **代码位置** | foundation/multimedia/drm_framework |
| **接口层** | N-API、C-API、SA IPC |
| **依赖** | HDI、IPC Framework、SAMgr |

## 威胁模型

### 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                      信任边界                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  DRM Service (进程 3012)                            │   │
│  │  - 敏感操作: 证书 Provision、密钥生成、解密          │   │
│  │  - 信任: IPC 调用者已通过 Token 校验                 │   │
│  └─────────────────────────────────────────────────────┘   │
│                            ↑ IPC                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  应用进程 (Client)                                   │   │
│  │  - 输入: 用户数据、网络响应                           │   │
│  │  - 信任: IPC Proxy、参数校验                          │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 数据流

| 数据流 | 方向 | 风险等级 |
|--------|------|----------|
| JS→Native | App→Framework | 中 |
| Framework→IPC | Framework→SA | 低 |
| SA→HDI | SA→Driver | 低 |
| HDI→Plugin | Driver→Vendor | 低 |
| 网络响应 | External→App | 高 |

## 攻击面识别

### 1. 输入校验

| 攻击面 | 位置 | 风险 |
|--------|------|------|
| DRM 方案名称 | `CreateMediaKeySystem()` | 中 |
| MimeType | `GenerateMediaKeyRequest()` | 中 |
| 初始化数据 | `initData` 参数 | 高 |
| 证书响应 | `ProcessKeySystemResponse()` | 高 |
| 许可证响应 | `ProcessMediaKeyResponse()` | 高 |

### 2. IPC 接口

| 攻击面 | 位置 | 风险 |
|--------|------|------|
| 接口 Token | IPC 校验 | 低 |
| PID/UID 获取 | `IPCSkeleton::GetCallingPid()` | 低 |
| BundleName 映射 | `GetClientBundleName()` | 中 |

### 3. 资源限制

| 资源 | 限制值 | 风险 |
|------|--------|------|
| MediaKeySystem 实例 | 64/插件 | 低 |
| MediaKeySession 实例 | 受限 | 低 |
| 内存缓冲区 | 配置限制 | 中 |

## 风险点分析

### 🔴 高风险 (需立即修复)

#### 风险 1: 证书响应缺少长度校验

**证据**: `native_mediakeysystem.h:290-291`

```c
Drm_ErrCode OH_MediaKeySystem_ProcessKeySystemResponse(MediaKeySystem *mediaKeySystem,
    uint8_t *response, int32_t responseLen);
```

**触发条件**:
```
应用调用 processKeySystemResponse(response) 时，
responseLen > 实际 response 缓冲区大小
```

**影响**: 内存越界读取，导致信息泄露

**修复建议**:
```c
// 增加响应数据完整性校验
if (responseLen > MAX_CERTIFICATE_RESPONSE_LEN) {
    return DRM_ERR_INVALID_VAL;
}
if (response == nullptr && responseLen > 0) {
    return DRM_ERR_INVALID_VAL;
}
```

**状态**: ⚠️ 需确认 responseLen 的上限定义

---

#### 风险 2: 离线密钥 ID 越界访问

**证据**: `native_drm_common.h:362-368`

```c
typedef struct DRM_OfflineMediakeyIdArray {
    uint32_t idsCount;
    int32_t idsLen[MAX_OFFLINE_MEDIA_KEY_ID_COUNT];  // MAX=512
    uint8_t ids[MAX_OFFLINE_MEDIA_KEY_ID_COUNT][MAX_OFFLINE_MEDIA_KEY_ID_LEN];  // MAX=64
} DRM_OfflineMediakeyIdArray;
```

**触发条件**:
```
应用调用 GetOfflineMediaKeyIds() 后，
使用超出范围的索引访问 ids[] 数组
```

**影响**: 内存越界读写，可能导致崩溃或代码执行

**修复建议**:
```c
// 返回前校验 idsCount 不超过数组定义
if (outParam->idsCount > MAX_OFFLINE_MEDIA_KEY_ID_COUNT) {
    outParam->idsCount = MAX_OFFLINE_MEDIA_KEY_ID_COUNT;
}
```

---

### 🟠 中风险 (建议修复)

#### 风险 3: 配置项名称缺少校验

**证据**: `mediakeysystem_service.cpp:157-173`

```cpp
int32_t MediaKeySystemService::SetConfigurationString(...) {
    DRM_CHECK_AND_RETURN_RET_LOG(configName != "bundleName", ...);
    DRM_CHECK_AND_RETURN_RET_LOG(configName != "apiTargetVersion", ...);
    // 其他 configName 未校验
}
```

**触发条件**:
```
应用调用 setConfigurationString() 设置非法的 configName
```

**影响**: 潜在配置注入，可能导致意外行为

**修复建议**:
```c
// 增加白名单校验
static const char* ALLOWED_CONFIG_NAMES[] = {
    "vendor", "version", "description", 
    "algorithms", "deviceUniqueId", /* ... */
};

if (!IsInAllowList(configName)) {
    return DRM_ERR_INVALID_VAL;
}
```

---

#### 风险 4: BundleName 获取可能失败

**证据**: `drm_dfx_utils.cpp:28-58`

```cpp
std::string GetClientBundleName(int32_t uid) {
    auto samgr = SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();
    sptr<IRemoteObject> object = samgr->GetSystemAbility(BUNDLE_MGR_SERVICE_SYS_ABILITY_ID);
    // samgr 或 bms 可能为 nullptr
    auto result = bms->GetNameForUid(uid, bundleName);
}
```

**触发条件**:
```
Bundle Manager 服务不可用时调用
```

**影响**: 审计日志无法记录调用者身份，降低可追溯性

**修复建议**:
```c
if (samgr == nullptr || bms == nullptr) {
    DRM_WARN_LOG("BundleManager unavailable, uid=%{public}d", uid);
    return "unknown";
}
```

---

#### 风险 5: 事件回调线程资源泄漏

**证据**: `media_key_system_impl.h:111`

```cpp
std::thread eventQueueThread;  // 未检查线程是否成功创建
```

**触发条件**:
```
频繁创建/销毁 MediaKeySystem 对象，
eventQueueThread 构造失败导致资源泄漏
```

**影响**: 资源耗尽，DoS 攻击

**修复建议**:
```c
// 检查线程启动结果
if (eventQueueThread.joinable()) {
    eventQueueThread.join();
}
// 或使用线程池替代
```

---

### 🟢 低风险 (可接受)

#### 风险 6: PID 复用导致混淆

**证据**: `mediakeysystemfactory_service.cpp:280-282`

```cpp
int32_t pid = IPCSkeleton::GetCallingPid();
mediaKeySystemForPid_[pid].insert(mediaKeySystemService);
```

**触发条件**:
```
进程退出后 PID 被新进程复用，
旧 MediaKeySystem 记录未及时清理
```

**影响**: 资源泄漏、潜在的跨进程信息泄露

**缓解措施**: 
- 已通过 DeathRecipient 机制监听进程死亡
- 会话数量有上限限制

---

#### 风险 7: 日志泄露敏感信息

**证据**: 多处 `DRM_DEBUG_LOG`, `DRM_INFO_LOG` 

**触发条件**:
```
调试日志未关闭，敏感信息输出到日志
```

**影响**: 密钥材料、许可证信息泄露

**缓解措施**:
- 敏感数据使用 `%{public}d` 格式化符
- 关键信息仅在 DEBUG 级别输出

---

## 权限与能力

### 声明的系统能力

| 能力 | 说明 |
|------|------|
| SystemCapability.Multimedia.Drm.Core | 使用 DRM 功能 |

### 服务权限配置

**文件**: `services/etc/resident/drm_service.cfg`

| 权限 | 说明 |
|------|------|
| ohos.permission.GET_SENSITIVE_PERMISSIONS | 获取敏感权限 |
| ohos.permission.GET_BUNDLE_INFO_PRIVILEGED | 获取包信息 |
| ohos.permission.MANAGE_SECURE_SETTINGS | 管理安全设置 |

### 缺失的权限校验

| 接口 | 当前校验 | 建议 |
|------|----------|------|
| CreateMediaKeySystem | ListenerObject 检查 | 增加 AccessToken 校验 |
| ProcessKeySystemResponse | 参数校验 | 增加签名校验 |
| DecryptMediaData | 无显式校验 | 增加权限声明检查 |

## 安全建议

### 1. 输入验证加强

```c
// 所有公开 API 增加参数校验
#define VALIDATE_INPUT(param, ret) \
    if ((param) == nullptr) { \
        DRM_ERR_LOG("Invalid input: " #param); \
        return DRM_ERR_INVALID_VAL; \
    }
```

### 2. 敏感数据保护

```c
// 内存中的密钥材料使用加密存储
// 日志中敏感数据脱敏
// 密钥缓冲区使用后立即清零
```

### 3. 资源限制

```c
// 增加速率限制
RateLimiter rateLimiter(MAX_REQUESTS_PER_SECOND);

// 增加并发限制
Semaphore semaphore(MAX_CONCURRENT_SESSIONS);
```

### 4. 安全审计

```c
// 所有敏感操作记录审计日志
HiSysEventWrite(..., DRM_SECURITY_EVENT, ...);
```

## 相关文档

- [JS N-API 参考](03_NAPI_Reference.md) - API 接口
- [C API 参考](04_CAPI_Reference.md) - Native API
- [架构设计](05_Architecture.md) - 调用关系
- [GN Targets](06_GN_Targets.md) - 构建配置
