# 06_SecurityReview - 安全风险评估

## 评估概述

基于代码审计，dmsfwk 组件存在以下安全风险：

| 风险类别 | 数量 | 最高等级 |
|---------|------|---------|
| 输入验证缺陷 | 4 | 高 |
| 内存安全 | 3 | 中 |
| 权限与鉴权 | 3 | 中 |
| 并发安全 | 2 | 中 |
| 逻辑漏洞 | 3 | 中 |

---

## 1. 输入验证缺陷

### R1: BundleName 格式验证缺失 [中危]

**位置**: `services/dtbcollabmgr/src/ability_connection_manager/ability_connection_manager.cpp:465`

**证据**:
```cpp
if (peerInfo.bundleName.empty()) {
    HILOGE("bundleName is empty!");
    return INVALID_PARAMETERS_ERR;
}
// 缺少格式校验：bundleName 应匹配 ^[a-zA-Z][a-zA-Z0-9._]*$
```

**触发路径**:
```
JS createAbilityConnectionSession() 
→ AbilityConnectionManager::ConnectSession()
→ 第465行 bundleName 检查
→ 传入 BundleManagerInternal 查询
```

**影响**: 
- 攻击者可传入包含特殊字符的 bundleName
- 可能导致日志注入或路径遍历（如果后续用于文件操作）

**修复建议**:
```cpp
// 添加格式验证
const std::regex bundleNameRegex("^[a-zA-Z][a-zA-Z0-9._]*$");
if (!std::regex_match(peerInfo.bundleName, bundleNameRegex)) {
    HILOGE("Invalid bundleName format");
    return INVALID_PARAMETERS_ERR;
}
```

---

### R2: JSON 解析缺乏深度限制 [低危]

**位置**: `services/dtbschedmgr/src/continue/dsched_continue_event.cpp:35-770`

**证据**:
```cpp
// 多处使用 cJSON_Parse 没有深度限制
int32_t DSchedContinueEvent::Unmarshal(...)
{
    cJSON *rootValue = cJSON_Parse(str.c_str());
    if (rootValue == nullptr) {
        HILOGE("Failed to parse JSON");
        return INVALID_PARAMETERS_ERR;
    }
    // 缺少递归深度检查
}
```

**触发路径**:
```
SoftBus 接收数据 
→ DSchedTransportSoftbusAdapter::OnBytes()
→ DSchedContinueManager::NotifyDataRecv()
→ DSchedContinueEvent::Unmarshal()
→ cJSON_Parse() 解析恶意嵌套 JSON
```

**影响**: 
- 攻击者可发送深度嵌套的 JSON 导致栈溢出
- DoS 攻击

**修复建议**:
```cpp
// 限制 JSON 解析深度
cJSON *rootValue = cJSON_ParseWithOpts(str.c_str(), nullptr, true);
// 或使用自定义解析器限制递归深度
```

---

### R3: Parcel 字符串长度检查不足 [中危]

**位置**: `services/dtbschedmgr/src/distributedWant/distributed_want_params.cpp:1129-1182`

**证据**:
```cpp
bool DistributedWantParams::ReadFromParcelString(Parcel& parcel, const std::string& key)
{
    // 读取字符串长度
    int32_t len = parcel.ReadInt32();
    if (len < 0) {
        return false;
    }
    // 读取字符串内容
    const char* str = reinterpret_cast<const char*>(parcel.ReadUnpadBuffer(len));
    // 风险：len 可能极大导致内存分配失败
}
```

**影响**: 
- 恶意构造的 Parcel 中 len 值过大
- 导致内存分配失败 (DoS)

**修复建议**:
```cpp
constexpr int32_t MAX_PARCEL_STRING_LEN = 10 * 1024 * 1024; // 10MB
if (len < 0 || len > MAX_PARCEL_STRING_LEN) {
    HILOGE("String length %d exceeds limit", len);
    return false;
}
```

---

### R4: SurfaceId 未经验证直接使用 [中危]

**位置**: `interfaces/kits/napi/ability_connection_manager/js_ability_connection_manager.cpp`

**证据**:
```cpp
// SurfaceId 从 JS 直接传入，用于设置窗口表面
// 缺乏对 SurfaceId 格式的验证
// TODO(证据不足): 需要进一步确认验证逻辑
```

**影响**: 
- 恶意 SurfaceId 可能导致 Surface 劫持
- 信息泄露或 UI 欺骗

**修复建议**:
- 验证 SurfaceId 格式和所有权
- 确保调用者有权操作该 Surface

---

## 2. 内存安全问题

### R5: 分片重组缓冲区溢出风险 [中危]

**位置**: `services/dtbschedmgr/src/softbus_adapter/transport/dsched_softbus_session.cpp:239-287`

**证据**:
```cpp
void DSchedSoftbusSession::AssembleFrag(...)
{
    if (!isWaiting_) {
        // 首次分片
        totalLen_ = headerPara.totalLen;
        recvBuf_ = std::make_shared<DSchedDataBuffer>(totalLen_);
        // 风险：如果 totalLen_ 被伪造为极大值
        // 可能导致内存分配失败或耗尽
    }
}
```

**触发路径**:
```
攻击者发送恶意分片包
→ OnBytesReceived()
→ PackRecvData()
→ AssembleFrag()
→ 分配 totalLen_ 大小缓冲区
```

**影响**: 
- DoS（内存耗尽）
- 可能的分片重叠攻击

**修复建议**:
```cpp
constexpr uint32_t MAX_TOTAL_LEN = 100 * 1024 * 1024; // 100MB
if (headerPara.totalLen > MAX_TOTAL_LEN) {
    HILOGE("Total length %u exceeds max", headerPara.totalLen);
    return;
}
```

---

### R6: cJSON 释放前返回导致内存泄漏 [低危]

**位置**: `services/dtbschedmgr/src/continue/dsched_continue_event.cpp`

**证据**:
```cpp
// 某些错误路径在 cJSON_Delete 前返回
cJSON *rootValue = cJSON_Parse(str.c_str());
if (condition) {
    return ERR_INVALID_DATA;  // 内存泄漏！
}
cJSON_Delete(rootValue);
```

**影响**: 
- 内存泄漏
- 长期运行后可能导致 OOM

**修复建议**:
```cpp
// 使用 RAII 或确保所有路径都释放
cJSON *rootValue = cJSON_Parse(str.c_str());
if (rootValue == nullptr) {
    return INVALID_PARAMETERS_ERR;
}
// 使用智能指针或 goto cleanup
cJSON_Delete(rootValue);
```

---

### R7: TLV 解析越界读取风险 [中危]

**位置**: `services/dtbschedmgr/src/softbus_adapter/transport/dsched_softbus_session.cpp:162-207`

**证据**:
```cpp
int32_t DSchedSoftbusSession::ReadTlvToHeader(...)
{
    while (index < totalLen) {
        tlvItem.type = buffer[index++];  // 可能越界
        tlvItem.length = buffer[index++]; // 可能越界
        // 基于 length 读取数据
        memcpy(tlvItem.value, &buffer[index], tlvItem.length);
        // 如果 length 被篡改，可能越界
    }
}
```

**影响**: 
- 信息泄露（读取越界数据）
- 可能导致服务崩溃

**修复建议**:
```cpp
if (index + 2 > totalLen) return ERROR;
tlvItem.type = buffer[index++];
tlvItem.length = buffer[index++];
if (index + tlvItem.length > totalLen) return ERROR;
memcpy(tlvItem.value, &buffer[index], tlvItem.length);
```

---

## 3. 权限与鉴权

### R8: IsFoundationCall 依赖进程名检查 [中危]

**位置**: `services/dtbschedmgr/src/distributed_sched_permission.cpp:618-628`

**证据**:
```cpp
bool DistributedSchedPermission::IsFoundationCall() const
{
    uint32_t accessToken = IPCSkeleton::GetCallingTokenID();
    AccessToken::NativeTokenInfo nativeTokenInfo;
    int32_t result = AccessToken::AccessTokenKit::GetNativeTokenInfo(accessToken, nativeTokenInfo);
    if (result == ERR_OK && nativeTokenInfo.processName == FOUNDATION_PROCESS_NAME) {
        return true;
    }
    return false;
}
```

**分析**:
- 仅检查进程名，不验证签名或证书
- 如果 foundation 进程被注入恶意代码，可绕过权限检查

**影响**: 
- 权限绕过

**修复建议**:
```cpp
// 增加额外的安全检查
bool DistributedSchedPermission::IsFoundationCall() const
{
    // 现有检查
    if (!CheckProcessName(FOUNDATION_PROCESS_NAME)) {
        return false;
    }
    // 增加 UID 检查
    uid_t callingUid = IPCSkeleton::GetCallingUid();
    if (callingUid != FOUNDATION_UID) {
        return false;
    }
    return true;
}
```

---

### R9: 权限缓存可能导致权限提升 [低危]

**位置**: 多处使用 VerifyAccessToken

**证据**:
```cpp
// AccessTokenKit 可能缓存权限检查结果
// 如果权限被动态撤销，缓存可能导致过期权限通过检查
// TODO(证据不足): 需要确认 AccessTokenKit 实现
```

**影响**: 
- TOCTOU 问题
- 权限提升

**修复建议**:
- 考虑在关键操作前重新验证权限
- 或确保 AccessTokenKit 实时查询

---

### R10: 跨设备权限粒度较粗 [中危]

**位置**: `services/dtbschedmgr/src/distributed_sched_permission.cpp:210-220`

**证据**:
```cpp
int32_t DistributedSchedPermission::CheckCustomPermission(...)
{
    // 使用 AllocLocalTokenID 进行跨设备权限映射
    // 但权限粒度较粗，缺乏细粒度控制
}
```

**影响**: 
- 一旦设备被信任，该设备上的所有应用都可调用
- 缺乏应用级别的细粒度控制

**修复建议**:
- 实现应用级别的跨设备权限控制
- 引入设备+应用双重验证

---

## 4. 并发安全

### R11: Session 映射并发访问 [中危]

**位置**: `services/dtbschedmgr/src/softbus_adapter/transport/dsched_transport_softbus_adapter.cpp`

**证据**:
```cpp
// sessions_ 映射可能被多个线程并发访问
std::map<int32_t, std::shared_ptr<DSchedSoftbusSession>> sessions_;
// TODO(证据不足): 需要确认互斥锁保护情况
```

**影响**: 
- 竞态条件
- 数据不一致或崩溃

**修复建议**:
```cpp
std::mutex sessionsMutex_;
// 所有访问加锁
std::lock_guard<std::mutex> lock(sessionsMutex_);
sessions_[socketId] = session;
```

---

### R12: Handler 回调竞态 [低危]

**位置**: `services/dtbschedmgr/src/mission/notification/dms_continue_send_manager.cpp:168-184`

**证据**:
```cpp
// screenLockedHandler_ 可能在使用时被释放
if (screenLockedHandler_ != nullptr) {
    // 检查和使用之间可能有竞态
    screenLockedHandler_->RemoveTask(taskName);
}
```

**修复建议**:
```cpp
auto handler = screenLockedHandler_;
if (handler != nullptr) {
    handler->RemoveTask(taskName);
}
```

---

## 5. 逻辑漏洞

### R13: 设备离线检测延迟 [中危]

**位置**: `services/dtbschedmgr/src/dtbschedmgr_device_info_storage.cpp:191-310`

**证据**:
```cpp
// 设备离线后，缓存可能未及时清理
// 攻击者可利用窗口期向离线设备发送敏感数据
```

**影响**: 
- 数据发送到已离线/不可信设备

**修复建议**:
- 每次发送前实时检查设备状态
- 缩短缓存有效期

---

### R14: 错误码信息泄露 [低危]

**位置**: 多处 HILOGE 打印错误信息

**证据**:
```cpp
HILOGE("Failed to connect to device %s, error: %d", deviceId, errCode);
// 详细错误信息可能泄露内部状态
```

**影响**: 
- 信息泄露，辅助攻击者

**修复建议**:
- 对敏感信息脱敏
- 区分内部日志和对外错误码

---

### R15: 超时配置不当 [低危]

**位置**: `services/dtbschedmgr/src/distributed_sched_service.cpp:1361`

**证据**:
```cpp
constexpr int64_t CHECK_REMOTE_INSTALL_ABILITY = 40000; // 40秒
// 超时时间过长，DoS 攻击窗口大
```

**修复建议**:
- 根据网络状况动态调整超时
- 实现更细粒度的超时控制

---

## 修复优先级

| 优先级 | 风险项 | 修复难度 |
|-------|-------|---------|
| **P0** | R5 (缓冲区溢出) | 低 |
| **P0** | R7 (越界读取) | 低 |
| **P1** | R1 (BundleName验证) | 低 |
| **P1** | R3 (Parcel长度) | 低 |
| **P1** | R8 (权限检查) | 中 |
| **P2** | R11 (并发安全) | 中 |
| **P2** | R2 (JSON深度) | 低 |
| **P3** | 其他 | - |

---

## 相关链接

- 上一章: [05_AttackSurface.md](05_AttackSurface.md) - 攻击面分析
- 下一章: [07_Build.md](07_Build.md) - 构建配置
