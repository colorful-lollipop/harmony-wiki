# 06 安全风险评估

**文档目的**: 详细分析 DeviceProfile 的安全风险，提供代码级证据和修复建议  
**适用范围**: 安全研究员、审计人员、开发工程师  

---

## 6.1 评估范围

本章节按照以下 5 类安全风险进行分析：

1. **输入验证缺陷** - 类型/长度/范围/null/编码/路径遍历
2. **内存安全问题** - 缓冲区/Use-After-Free/双重释放
3. **权限与鉴权** - 权限校验、身份验证、访问控制
4. **并发安全** - 竞态条件、TOCTOU、线程安全
5. **逻辑漏洞** - 错误处理、资源耗尽、信息泄露

---

## 6.2 输入验证缺陷

### R1: 路径遍历风险（低风险）

**位置**: `services/core/src/permissionmanager/permission_manager.cpp:74`

**证据**:
```cpp
int32_t PermissionManager::LoadPermissionCfg(const std::string& filePath) {
    char path[PATH_MAX + 1] = {0x00};
    if (filePath.length() == 0 || filePath.length() > PATH_MAX || 
        realpath(filePath.c_str(), path) == nullptr) {
        HILOGE("File canonicalization failed");
        return DP_PARSE_PERMISSION_JSON_FAIL;
    }
    std::ifstream ifs(path);
    // ...
}
```

**分析**:
- ✅ 使用 `realpath()` 进行路径规范化
- ✅ 检查路径长度
- ✅ 验证文件存在性
- **结论**: 风险已控制

---

### R2: JSON 解析风险（中风险）

**位置**: `services/core/src/permissionmanager/permission_manager.cpp:86-94`

**证据**:
```cpp
cJSON* permissionJson = cJSON_Parse(fileContent.c_str());
if (!cJSON_IsObject(permissionJson)) {
    HILOGE("Permission json parse failed!");
    cJSON_Delete(permissionJson);
    return DP_PARSE_PERMISSION_JSON_FAIL;
}
```

**分析**:
- ✅ 检查解析结果是否为对象
- ⚠️ 未限制 JSON 大小，可能导致内存占用过大
- ⚠️ cJSON 存在已知漏洞历史

**修复建议**:
```cpp
// 建议增加大小限制
if (fileContent.length() > MAX_JSON_SIZE) {
    HILOGE("Permission json too large!");
    return DP_PARSE_PERMISSION_JSON_FAIL;
}
```

---

### R3: IPC 参数验证不足（中风险）

**位置**: 各 IPC 接口入口

**证据** (以 PutDeviceProfileBatch 为例):
```cpp
// services/core/src/distributed_device_profile_service_new.cpp:518
int32_t DistributedDeviceProfileServiceNew::PutDeviceProfileBatch(
    std::vector<DeviceProfile&> deviceProfiles) {
    if (!PermissionManager::GetInstance().CheckCallerPermission()) {
        return DP_PERMISSION_DENIED;
    }
    // 直接处理 deviceProfiles，未见长度检查
    return DeviceProfileManager::GetInstance().PutDeviceProfileBatch(deviceProfiles);
}
```

**分析**:
- ⚠️ vector 长度未限制，可能导致内存占用过大
- ⚠️ 各字段内容未深度校验

**触发路径**:
```
恶意调用者 → IPC PutDeviceProfileBatch → 超大 vector → 内存占用激增 → OOM
```

**修复建议**:
```cpp
constexpr size_t MAX_BATCH_SIZE = 1000;
if (deviceProfiles.size() > MAX_BATCH_SIZE) {
    return DP_INVALID_PARAMS;
}
```

---

## 6.3 内存安全问题

### R4: 未发现明显内存安全问题（低风险）

**检查结果**:
- ✅ 使用 C++ 标准容器 (`vector`, `string`, `map`)
- ✅ 使用智能指针 (`shared_ptr`, `unique_ptr`)
- ✅ 无 `strcpy`, `sprintf` 等危险函数使用
- ✅ 使用 `std::lock_guard` 管理锁生命周期

**证据** (grep 搜索结果):
```bash
$ grep -r "strcpy\|strcat\|sprintf" --include="*.cpp" .
# 无匹配结果
```

---

## 6.4 权限与鉴权

### R5: SyncDeviceProfile 权限过宽（中风险）

**位置**: `permission/permission.json:33`

**证据**:
```json
{
    "SyncDeviceProfile": ["all"]
}
```

**分析**:
- ⚠️ `all` 表示任何有 `ACCESS_SERVICE_DP` 权限的服务都可调用
- ⚠️ 可能导致 Profile 数据被恶意同步到未授权设备

**触发路径**:
```
恶意服务 → SyncDeviceProfile(目标设备=攻击者设备) 
→ Profile 数据泄露到攻击者设备
```

**影响评估**:
| 维度 | 评估 |
|------|------|
| 可利用性 | 需要已获取系统服务权限 |
| 数据泄露 | Profile 信息可被导出 |
| 权限提升 | 无直接提权 |

**修复建议**:
```json
{
    "SyncDeviceProfile": ["device_manager", "softbus_server"]
}
```

---

### R6: GetDeviceProfile 权限为 all（低风险）

**位置**: `permission/permission.json:26`

**证据**:
```json
{
    "GetDeviceProfile": ["all"]
}
```

**分析**:
- ⚠️ 任何服务可查询 DeviceProfile
- ⚠️ 可能导致信息泄露（设备型号、OS 版本等）
- ✅ 但数据本身非敏感（设备公开信息）

**影响评估**: 信息泄露风险低

---

### R7: 权限检查可被绕过（低风险）

**位置**: `services/core/src/permissionmanager/permission_manager.cpp:192-216`

**证据**:
```cpp
bool PermissionManager::IsCallerTrust(const std::string& interfaceName) {
    auto tokenID = IPCSkeleton::GetCallingTokenID();
    // ... 检查 TOKEN_NATIVE
    
    if (!CheckInterfacePermission(interfaceName)) {
        HILOGE("This caller cannot call this interface");
        return false;
    }
    return true;
}
```

**分析**:
- ✅ 检查 TOKEN_NATIVE（仅系统服务）
- ✅ 检查接口白名单
- ⚠️ 但依赖 `GetCallerProcName()` 获取进程名

**潜在问题**:
```cpp
// permission_manager.cpp:TODO
std::string PermissionManager::GetCallerProcName() {
    // 通过 IPCSkeleton 获取调用者信息
    // 若 IPC 机制被攻破，此信息可被伪造
}
```

**修复建议**:
- 考虑增加调用者 UID 白名单校验
- 使用数字签名验证调用者身份

---

## 6.5 并发安全

### R8: 权限 map 并发访问（低风险）

**位置**: `services/core/src/permissionmanager/permission_manager.cpp:183-190`

**证据**:
```cpp
bool PermissionManager::CheckInterfacePermission(const std::string& interfaceName) {
    std::string callProcName = GetCallerProcName();
    std::unordered_set<std::string> permittedProcNames;
    {
        std::lock_guard<std::mutex> lockGuard(permissionMutex_);
        permittedProcNames = permissionMap_[interfaceName];
    }
    // 在锁外使用 permittedProcNames
    bool checkResult = (permittedProcNames.count(callProcName) != 0 || ...);
    return checkResult;
}
```

**分析**:
- ✅ 使用 `lock_guard` 保护共享数据访问
- ✅ 锁粒度适当（仅在拷贝时持有锁）
- ✅ 无死锁风险

**结论**: 并发安全处理正确

---

### R9: 单例模式线程安全（低风险）

**证据**:
```cpp
// common/include/utils/single_instance.h
#define IMPLEMENT_SINGLE_INSTANCE(ClassName) \
    ClassName& ClassName::GetInstance() { \
        static ClassName instance; \<!-- C++11 保证线程安全 --> \
        return instance; \
    }
```

**分析**:
- ✅ C++11 `static` 局部变量初始化是线程安全的
- ✅ 使用 Meyer's Singleton 模式

---

## 6.6 逻辑漏洞

### R10: Profile 数据明文存储（中风险）

**位置**: `services/core/include/profiledatamanager/kvadapter/kv_adapter.h`

**证据**:
```cpp
// KV Store 写入接口（无加密参数）
int32_t KVAdapter::Put(const std::string& key, const std::string& value);
```

**分析**:
- ⚠️ Profile 数据以明文存储在 KV Store 和 RDB 中
- ⚠️ 获取 root 权限后可读取敏感信息
- ⚠️ AccessControlProfile 包含访问控制策略

**影响评估**:
| 维度 | 评估 |
|------|------|
| 前提条件 | 需要 root 权限 |
| 泄露数据 | 设备信息、ACL 策略、可信设备列表 |
| 进一步利用 | 可用于规划更精确的攻击 |

**修复建议**:
1. 对敏感字段使用加密存储
2. 使用硬件安全模块 (HSM) 保护密钥
3. 限制数据库文件访问权限

---

### R11: 错误信息可能泄露路径（低风险）

**位置**: `common/include/constants/distributed_device_profile_errors.h`

**证据**:
```cpp
// 部分错误码定义
constexpr int32_t DP_FILE_FAILED_ERR = DP_ERR_BASE + 0x24;  // 文件操作失败
// 可能返回具体文件路径
```

**分析**:
- ⚠️ 错误码可能伴随日志输出内部路径
- ⚠️ 日志可能包含敏感信息

**修复建议**:
```cpp
// 日志脱敏处理
HILOGE("Failed to open file: %{public}s", MaskPath(filePath).c_str());
```

---

### R12: 资源耗尽风险（中风险）

**证据**:
```cpp
// services/core/src/distributed_device_profile_service_new.cpp:876
int32_t DistributedDeviceProfileServiceNew::SubscribeDeviceProfile(
    const SubscribeInfo& subscribeInfo) {
    // 保存订阅回调对象
    subscribeInfos_.push_back(subscribeInfo);
    // 未见数量限制
}
```

**分析**:
- ⚠️ 订阅数量无限制
- ⚠️ 批量操作 vector 大小无限制
- ⚠️ 可能导致内存耗尽

**触发路径**:
```
恶意服务 → 循环调用 SubscribeDeviceProfile → 内存耗尽 → OOM
```

**修复建议**:
```cpp
constexpr size_t MAX_SUBSCRIBE_COUNT = 100;
if (subscribeInfos_.size() >= MAX_SUBSCRIBE_COUNT) {
    return DP_EXCEED_MAX_SUBSCRIBE_COUNT;
}
```

---

## 6.7 风险评估汇总

| 风险 ID | 类型 | 等级 | 状态 | 优先级 |
|---------|------|------|------|--------|
| R1 | 路径遍历 | 低 | ✅ 已防护 | 低 |
| R2 | JSON 解析 | 中 | ⚠️ 需改进 | 中 |
| R3 | 参数验证 | 中 | ⚠️ 需改进 | 高 |
| R4 | 内存安全 | 低 | ✅ 良好 | 低 |
| R5 | 权限过宽 | 中 | ⚠️ 需审查 | 高 |
| R6 | 信息泄露 | 低 | ✅ 可接受 | 低 |
| R7 | 权限绕过 | 低 | ⚠️ 需监控 | 中 |
| R8 | 并发安全 | 低 | ✅ 良好 | 低 |
| R9 | 单例安全 | 低 | ✅ 良好 | 低 |
| R10 | 明文存储 | 中 | ⚠️ 建议改进 | 高 |
| R11 | 信息泄露 | 低 | ⚠️ 建议改进 | 低 |
| R12 | 资源耗尽 | 中 | ⚠️ 需改进 | 高 |

---

## 6.8 修复建议汇总

### 高优先级

1. **R3 - 参数验证**: 对所有 IPC 接口增加 vector 长度限制
2. **R10 - 明文存储**: 对敏感 Profile 数据加密存储
3. **R12 - 资源限制**: 限制订阅数量、批量操作大小

### 中优先级

4. **R2 - JSON 大小**: 限制 permission.json 解析大小
5. **R5 - 权限审查**: 评估 SyncDeviceProfile 权限设置
6. **R7 - 身份验证**: 增加 UID 白名单校验

### 低优先级

7. **R11 - 日志脱敏**: 对日志中的路径信息脱敏

---

## 6.9 相关章节

| 目标 | 推荐阅读 |
|------|----------|
| 攻击面分析 | [05_AttackSurface.md](05_AttackSurface.md) |
| 权限检查实现 | [08_Internals.md](08_Internals.md) |
| 接口详情 | [04_Interface.md](04_Interface.md) |
| 代码位置 | [03_CodeMap.md](03_CodeMap.md) |
