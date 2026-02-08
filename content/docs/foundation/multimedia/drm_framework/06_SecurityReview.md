# 06 安全风险评估

## 目的与适用范围

本文档面向**安全研究员**，深度分析 OpenHarmony DRM 框架的安全风险，包括输入验证、内存安全、权限控制、并发安全等方面，提供可利用性评估和修复建议。

## 评估方法

- **静态代码审计**: 分析 C/C++ 源码安全模式
- **攻击面分析**: 基于 05_AttackSurface.md 的输入点
- **信任边界检查**: 跨域数据流的安全性
- **最佳实践对比**: 对比 OpenHarmony 安全编码规范

## 风险分类汇总

| 风险类别 | 数量 | 最高等级 | 总体评估 |
|----------|------|----------|----------|
| 输入验证缺陷 | 3 | 中 | 防护完善 |
| 内存安全问题 | 2 | 低 | 使用安全函数 |
| 权限与鉴权 | 2 | 中 | 依赖IPC机制 |
| 并发安全 | 2 | 低 | 锁保护完善 |
| 逻辑漏洞 | 3 | 低 | 边界检查完整 |

---

## R1: 输入验证缺陷

### R1.1 字符串长度验证不完全 (中危)

**位置**: `frameworks/js/drm_napi/media_key_system_napi.cpp:144`

**证据**:
```cpp
NAPI_CHECK_ARGS(keySystemNameParam == nullptr || 
    !NapiParamUtils::GetValueString(env, keySystemNameParam, name), 
    DRM_INVALID_PARAM, DRM_ERR_INVALID_VAL);
```

**分析**:
- 检查了空指针和字符串获取是否成功
- 但未验证字符串的最大长度
- 超长字符串可能导致后续处理异常

**触发路径**:
```
JS createMediaKeySystem() → NAPI调用 → 超长name参数 → 服务处理
```

**影响评估**:
- 可利用性: 低（需要应用层调用）
- 影响: 拒绝服务（内存分配失败）
- 权限: 普通应用

**修复建议**:
```cpp
// 建议增加长度限制
if (name.length() > MAX_MEDIA_KEY_SYSTEM_NAME_LEN) {
    return DRM_ERR_INVALID_VAL;
}
```

### R1.2 配置名注入风险 (中危)

**位置**: `services/drm_service/server/src/mediakeysystem_service.cpp:157-173`

**证据**:
```cpp
int32_t MediaKeySystemService::SetConfigurationString(
    const std::string &configName, const std::string &value) {
    if (configName == "bundleName" || configName == "apiTargetVersion") {
        DRM_ERR_LOG("SetConfigurationString prohibited");
        return DRM_ERR_OPERATION_NOT_PERMITTED;
    }
    return hdiKeySystem_->SetConfigurationString(configName, value);
}
```

**分析**:
- 已禁止设置敏感配置项（bundleName, apiTargetVersion）
- 但其他配置项直接透传到 HDI 层
- 恶意配置可能影响插件行为

**触发路径**:
```
JS setConfigurationString() → IPC → Service → HDI → 插件
```

**影响评估**:
- 可利用性: 低（需要特定配置名）
- 影响: 取决于插件实现
- 权限: 普通应用

**修复建议**:
- 建立配置名白名单
- 对配置值进行格式校验

### R1.3 枚举值范围检查 (低危)

**位置**: `frameworks/c/drm_capi/native_mediakeysystem.cpp:79-83`

**证据**:
```cpp
if ((securityLevel <= CONTENT_PROTECTION_LEVEL_UNKNOWN) ||
    (securityLevel >= CONTENT_PROTECTION_LEVEL_MAX)) {
    DRM_ERR_LOG("ContentProtectionLevel is invalid");
    return false;
}
```

**分析**:
- 已正确检查枚举值范围
- 使用 <= 和 >= 确保边界安全
- 良好的安全实践

**评估**: ✅ 此风险不存在，代码已实现正确校验

---

## R2: 内存安全问题

### R2.1 malloc 返回值检查 (低危)

**位置**: `interfaces/kits/c/drm_capi/common/native_drm_object.h:111-130`

**证据**:
```cpp
dataInfo = (unsigned char *)malloc(data.size());
DRM_CHECK_AND_RETURN_LOG(dataInfo != nullptr, "malloc faild!");
errno_t ret = memcpy_s(dataInfo, data.size(), data.data(), data.size());
// ...
free(dataInfo);
dataInfo = nullptr;
```

**分析**:
- 检查了 malloc 返回值
- 使用 memcpy_s 安全函数
- 正确释放内存
- 但 malloc 失败后仅记录日志，未处理错误

**影响评估**:
- 可利用性: 低（内存紧张场景）
- 影响: 功能异常
- 权限: 任何应用

**修复建议**:
```cpp
dataInfo = (unsigned char *)malloc(data.size());
if (dataInfo == nullptr) {
    return DRM_ERR_NO_MEMORY;  // 返回错误码而非仅记录
}
```

### R2.2 new/delete 配对 (低危)

**位置**: `frameworks/c/drm_capi/native_mediakeysystem.cpp:156-167`

**证据**:
```cpp
struct MediaKeySystemObject *object = new (std::nothrow) MediaKeySystemObject(system);
DRM_CHECK_AND_RETURN_RET_LOG(object != nullptr, DRM_ERR_UNKNOWN, 
    "MediaKeySystemObject create failed!");

object->systemCallback_ = new (std::nothrow) MediaKeySystemCallbackCapi();
if (object->systemCallback_ == nullptr) {
    delete object;  // 正确释放
    return DRM_ERR_UNKNOWN;
}
```

**分析**:
- 使用 nothrow new
- 检查返回值
- 异常路径正确释放已分配内存
- 内存拷贝使用安全函数

**评估**: ✅ 代码实现良好，无明显风险

---

## R3: 权限与鉴权

### R3.1 缺少显式权限校验 (中危)

**位置**: 服务层多个入口

**证据**:
```cpp
// 代码中未发现以下调用:
// - VerifyAccessToken()
// - GetAccessTokenId()
// - 权限字符串检查

// 仅有PID检查:
pid_t pid = IPCSkeleton::GetCallingPid();
```

**分析**:
- 未发现 access_token 库的显式调用
- 仅依赖 IPC 的调用者 PID 进行基础识别
- 缺少细粒度的权限校验

**触发路径**:
```
任意应用 → IPC → Service → 敏感操作
```

**影响评估**:
- 可利用性: 中（任何有IPC能力的应用）
- 影响: 权限绕过
- 权限: 普通应用

**修复建议**:
```cpp
// 建议增加权限校验
if (!VerifyAccessToken(tokenId, "ohos.permission.ACCESS_DRM")) {
    return DRM_ERR_OPERATION_NOT_PERMITTED;
}
```

**注意**: DRM 服务配置中已声明权限，但代码中未显式校验：
```json
// services/etc/resident/drm_service.cfg
"permission" : [
    "ohos.permission.GET_SENSITIVE_PERMISSIONS",
    "ohos.permission.GET_BUNDLE_INFO_PRIVILEGED",
    "ohos.permission.MANAGE_SECURE_SETTINGS"
]
```

### R3.2 实例数量限制绕过 (低危)

**位置**: `services/drm_service/server/src/mediakeysystemfactory_service.cpp:256-259`

**证据**:
```cpp
if (currentMediaKeySystemNum_[name] >= MEDIA_KEY_SYSTEM_MAX_NUM) {
    return DRM_ERR_MAX_SYSTEM_NUM_REACHED;
}
```

**分析**:
- 已实施实例数量限制（最大64个）
- 但限制按 DRM 方案名(name)分别计算
- 攻击者可创建多种不同 name 的实例

**影响评估**:
- 可利用性: 低
- 影响: 资源耗尽
- 权限: 普通应用

**修复建议**:
```cpp
// 增加全局实例数限制
if (totalMediaKeySystemNum_ >= GLOBAL_MAX_NUM) {
    return DRM_ERR_MAX_SYSTEM_NUM_REACHED;
}
```

---

## R4: 并发安全

### R4.1 回调Map并发访问 (低危)

**位置**: `frameworks/js/drm_napi/key_session_callback_napi.cpp:27-39`

**证据**:
```cpp
void MediaKeySessionCallbackNapi::SetCallbackReference(
    const std::string eventType, std::shared_ptr<AutoRef> callbackPair) {
    std::lock_guard<std::mutex> lock(mutex_);  // 有锁保护
    callbackMap_[eventType] = callbackPair;
}
```

**分析**:
- 使用 std::mutex 保护 callbackMap_
- lock_guard 确保异常安全
- 良好的并发实践

**评估**: ✅ 代码实现良好，无明显风险

### R4.2 死亡监听竞争条件 (低危)

**位置**: `interfaces/inner_api/native/drm/media_key_system_factory_impl.cpp:47-89`

**证据**:
```cpp
const sptr<IMediaKeySystemFactoryService> GetServiceProxy() {
    auto samgr = SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();
    object = samgr->CheckSystemAbility(MEDIA_KEY_SYSTEM_SERVICE_ID);
    if (object == nullptr) {
        object = samgr->LoadSystemAbility(MEDIA_KEY_SYSTEM_SERVICE_ID, 30);
    }
    object->AddDeathRecipient(deathRecipient);  // 可能竞争
    tmpProxy->SetListenerObject(listenerObject);
}
```

**分析**:
- CheckSystemAbility 和 AddDeathRecipient 之间可能有竞争
- 服务可能在此期间死亡
- 但影响有限（下次调用会重新获取）

**影响评估**:
- 可利用性: 极低
- 影响: 短暂的功能异常
- 权限: 系统级

---

## R5: 逻辑漏洞

### R5.1 网络监听无限重试 (低危)

**位置**: `services/utils/drm_net_observer.cpp:54-78`

**证据**:
```cpp
int32_t retryCount = 0;
do {
    if (self->stopRequested_) {
        return;
    }
    ret = NetConnClient::GetInstance().RegisterNetConnCallback(specifier, self, 0);
    // ...
    sleep(RETRY_INTERVAL_S);
} while (retryCount < RETRY_MAX_TIMES);
```

**分析**:
- 有最大重试次数限制（RETRY_MAX_TIMES）
- sleep 可能被信号中断
- 重试间隔合理（RETRY_INTERVAL_S=3s）

**评估**: ✅ 实现合理，风险可控

### R5.2 配置文件解析错误处理 (低危)

**位置**: `services/utils/drm_api_operation.cpp:39-58`

**证据**:
```cpp
bool ConfigParser::LoadConfigurationFile(const std::string &configFile) {
    std::ifstream file(configFile);
    if (!file.is_open()) {
        perror("Unable to open api operation config file!");
        return false;
    }
    // 解析配置...
    return true;
}
```

**分析**:
- 文件打开失败有错误处理
- 但解析过程中的错误可能未完全处理
- 配置项格式错误可能导致未定义行为

**影响评估**:
- 可利用性: 低（需要文件篡改权限）
- 影响: 服务行为异常
- 权限: root

**修复建议**:
- 增加配置项格式校验
- 使用严格的解析器（如JSON Schema）

### R5.3 统计信息泄露 (低危)

**位置**: `services/drm_service/server/src/mediakeysystem_service.cpp:314`

**证据**:
```cpp
int32_t MediaKeySystemService::GetStatistics(std::vector<MetircKeyValue> &metrics) {
    return hdiKeySystem_->GetStatistics(metrics);
}
```

**分析**:
- 统计信息直接返回给调用者
- 可能包含敏感信息（调用次数、解密次数等）
- 但未验证调用者身份

**影响评估**:
- 可利用性: 低
- 影响: 信息泄露
- 权限: 普通应用

---

## 安全加固建议

### 立即修复（高优先级）

1. **增加权限校验**
   - 在敏感操作前调用 VerifyAccessToken()
   - 定义 DRM 专用权限

2. **完善输入验证**
   - 增加字符串最大长度限制
   - 建立配置名白名单

### 建议修复（中优先级）

3. **内存错误处理**
   - malloc/new 失败后返回错误码
   - 不要仅记录日志

4. **全局资源限制**
   - 增加全局 KeySystem 数量限制
   - 防止资源耗尽攻击

5. **配置安全**
   - 配置文件使用签名验证
   - 解析时增加格式校验

### 防御增强（低优先级）

6. **日志安全**
   - 避免在日志中打印敏感数据
   - 密钥ID、证书数据等应脱敏

7. **回调安全**
   - 验证回调函数指针有效性
   - 防止函数指针劫持

## 合规性检查

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 安全函数使用 | ✅ | 使用 memcpy_s, memset_s, strcpy_s |
| Sanitize配置 | ✅ | 开启 integer_overflow, ubsan, boundary_sanitize |
| Stack保护 | ✅ | stack_protector_ret = true |
| CFI | ✅ | 开启控制流完整性保护 |
| 空指针检查 | ✅ | 所有入口有检查 |
| 边界检查 | ✅ | 数组、枚举有范围检查 |
| 权限校验 | ⚠️ | 依赖IPC，缺少显式AccessToken检查 |
| 加密传输 | ⚠️ | HTTP由应用层控制，框架不强制HTTPS |

## 总结

### 总体安全评估

| 维度 | 评分 | 说明 |
|------|------|------|
| 代码质量 | 良好 | 使用安全函数，检查完善 |
| 架构安全 | 良好 | 分层清晰，信任边界明确 |
| 权限控制 | 一般 | 依赖IPC，缺少细粒度权限 |
| 风险等级 | 中低 | 无高危漏洞，主要是改进项 |

### 关键发现

1. **未发现高危漏洞**: 代码整体安全实践良好
2. **权限控制可加强**: 建议增加显式权限校验
3. **内存安全良好**: 使用安全函数，检查完善
4. **并发安全良好**: 锁使用正确，无竞争条件

### 后续建议

1. 定期进行安全审计
2. 建立安全测试用例
3. 关注依赖组件安全更新
4. 加强代码审查安全培训

## 相关链接

- [05_AttackSurface.md](05_AttackSurface.md) - 攻击面分析
- [02_Architecture.md](02_Architecture.md) - 架构设计
- [_work/NOTES.md](_work/NOTES.md) - 代码证据汇总
- [SUMMARY.md](SUMMARY.md) - 文档导航

---

*本文档基于静态代码分析生成，评估结果仅供参考，实际安全性需结合动态测试验证。*
