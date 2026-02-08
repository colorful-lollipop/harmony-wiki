# 安全风险评审

## 目的

本文档基于代码证据分析 thermal_manager 的安全风险，包括攻击面、信任边界和可被利用点。

## 适用范围

- OpenHarmony thermal_manager 模块
- 安全相关代码分析
- 风险评估与修复建议

## 相关文档

- [00_Overview.md](00_Overview.md) - 项目概览
- [01_Module_Boundaries.md](01_Module_Boundaries.md) - 模块边界与权限

---

## 威胁模型

```
外部输入（应用/配置）
    ↓
N-API 层 (thermal.so)
    ↓ (IPC)
Thermal Service (SA 3303)
    ↓ (权限检查)
核心功能（温度查询/策略执行/动作执行）
    ↓ (HDI)
Thermal Drivers (温度传感器/风扇控制）
```

### 攻击面

| 攻击面 | 描述 | 风险等级 |
|---|---|---|
| **N-API 输入** | JS 调用传参 | 中 |
| **IPC 通信** | Binder IPC 调用 | 中 |
| **配置文件** | XML 配置解析 | 中 |
| **文件操作** | sysfs 节点读写 | 低-中 |
| **HDI 通信** | 与驱动通信 | 中 |
| **动态库加载** | dlopen 加载能力库 | 低 |
| **权限绕过** | 权限检查不足 | 中 |

---

## 风险发现

### 1. 缺少输入长度校验

**风险类型**: 参数校验不足

**证据**: `frameworks/napi/thermal_manager_napi.cpp:204-224`

```cpp
// SubscribeThermalLevel 实现
napi_value ThermalManagerNapi::SubscribeThermalLevel(napi_env env, napi_callback_info info)
{
    size_t argc = MAX_ARGC;  // MAX_ARGC = 1
    napi_value argv[argc];
    NapiUtils::GetCallbackInfo(env, info, argc, argv);

    NapiErrors error;
    // 仅检查参数数量和类型，没有检查 callback 内容
    if (argc != MAX_ARGC || !NapiUtils::CheckValueType(env, argv[ARG_0], napi_function)) {
        return error.ThrowError(env, ThermalErrors::ERR_PARAM_INVALID);
    }
    // 直接保存回调引用，没有校验 callback 函数的完整性
    g_thermalLevelCallback->UpdateCallback(env, argv[ARG_0]);
    // ...
}
```

**可利用场景**:
1. 应用传入恶意构造的 callback 函数
2. Callback 包含异常逻辑或死循环
3. Callback 尝试访问受限资源

**影响**:
- 拒绝服务（DoS）
- 内存耗尽
- 潜在的代码执行（取决于 N-API 实现细节）

**修复建议**:
1. 对 callback 进行完整性校验
2. 限制 callback 注册数量
3. 实现 callback 执行的异常捕获

**代码示例**:
```cpp
// 建议添加校验
napi_valuetype callbackType;
napi_typeof(env, argv[ARG_0], &callbackType);
if (callbackType != napi_function) {
    return error.ThrowError(env, ThermalErrors::ERR_PARAM_INVALID);
}

// 限制最大回调数量
static const uint32_t MAX_CALLBACKS = 10;
if (callbackCount >= MAX_CALLBACKS) {
    return error.ThrowError(env, ThermalErrors::ERR_TOO_MANY_CALLBACKS);
}
```

---

### 2. 配置文件 XML 注入风险

**风险类型**: XML 解析注入

**证据**: `services/native/src/thermal_policy/thermal_srv_config_parser.cpp:68-98`

```cpp
bool ThermalSrvConfigParser::ParseXmlFile(const std::string& path)
{
    xmlDocPtr docParse;
    std::string result;
    if (DecryptConfig(path, result)) {
        docParse = xmlReadMemory(
            result.c_str(), int(result.size()), NULL, NULL, XML_PARSE_NOBLANKS);
    } else {
        docParse = xmlReadFile(path.c_str(), nullptr, XML_PARSE_NOBLANKS);
    }

    // 使用 libxml2 解析，但没有显式的 XXE 防护
    std::unique_ptr<xmlDoc, decltype(&xmlFreeDoc)> docPtr(docParse, xmlFreeDoc);
    // ... 解析配置
}
```

**可利用场景**:
1. 攻击者修改 `/vendor/etc/thermal_config/thermal_service_config.xml`
2. 注入恶意 XML 实体（XXE）
3. 导致任意文件读取（通过外部实体引用）

**影响**:
- 信息泄露（读取系统文件）
- 配置篡改
- 潜在的代码执行（XXE payload）

**修复建议**:
1. 禁用外部实体解析
2. 使用安全的 XML 解析选项
3. 限制配置文件来源（仅系统目录）

**代码示例**:
```cpp
// 建议添加 XXE 防护
docParse = xmlReadFile(path.c_str(), NULL, 
                   XML_PARSE_NOBLANKS | XML_PARSE_NOENT);
// 或者
docParse = xmlReadMemory(result.c_str(), int(result.size()), 
                     NULL, NULL, 
                     XML_PARSE_NOBLANKS | XML_PARSE_NOENT | XML_PARSE_NONET);
```

---

### 3. 文件操作缺少路径遍历保护

**风险类型**: 路径遍历

**证据**: `services/native/src/thermal_action/action/action_node.cpp:...` (多处)

```cpp
// 示例：使用用户输入直接拼接路径
ret = snprintf_s(buf, MAX_PATH, sizeof(buf) - 1, configDir.c_str(), iter.c_str());
// 没有 path 验证
```

**证据**: `services/native/src/thermal_action/thermal_action_manager.cpp:96-105`

```cpp
ret = snprintf_s(fileBuf, MAX_PATH, sizeof(fileBuf) - 1, configDir.c_str(), iter.c_str());
ret = snprintf_s(stateFileBuf, MAX_PATH, sizeof(stateFileBuf) - 1, 
                stateDir.c_str(), iter.c_str());
```

**可利用场景**:
1. 攻击者传入包含 `../` 的路径参数
2. 访问系统任意文件
3. 读取敏感配置信息

**影响**:
- 信息泄露
- 配置篡改
- 权限提升

**修复建议**:
1. 实现路径规范化和验证
2. 禁止路径遍历字符 (`..`, `./`)
3. 使用安全的路径拼接函数
4. 限制文件访问路径到安全目录

**代码示例**:
```cpp
// 添加路径验证
bool IsValidPath(const std::string& path) {
    if (path.find("..") != std::string::npos) {
        return false;
    }
    if (path.find("./") != std::string::npos) {
        return false;
    }
    return true;
}

// 使用路径规范化
std::string normalized = NormalizePath(inputPath);
if (!IsValidPath(normalized)) {
    return ERROR_INVALID_PATH;
}
```

---

### 4. 动态库加载安全风险

**风险类型**: 动态库加载

**证据**: `services/native/src/thermal_action/action/action_application_process.cpp:124`

```cpp
void* handler = dlopen("libthermal_ability.z.so", RTLD_LAZY | RTLD_NODELETE);
// 没有验证库路径或签名
```

**证据**: `services/native/src/thermal_action/action/action_popup.cpp:129`

```cpp
void *handler = dlopen("libpower_ability.z.so", RTLD_NOW | RTLD_NODELETE);
```

**可利用场景**:
1. 攻击者替换 `/system/lib64/libthermal_ability.z.so`
2. 加载恶意库到预期路径
3. 通过库劫持执行任意代码

**影响**:
- 代码执行
- 权限提升
- 系统妥协

**修复建议**:
1. 使用绝对路径加载库
2. 验证库签名（如果平台支持）
3. 限制动态库加载目录到系统路径
4. 实现库完整性校验

**代码示例**:
```cpp
// 添加路径验证
std::string libraryPath = "/system/lib64/" + libraryName;
if (!IsSecurePath(libraryPath)) {
    THERMAL_HILOGE(COMP_SVC, "Invalid library path: %{public}s", libraryPath.c_str());
    return ERROR;
}

// 添加签名验证（如果支持）
if (!VerifyLibrarySignature(libraryPath)) {
    THERMAL_HILOGE(COMP_SVC, "Library signature verification failed");
    return ERROR;
}
```

---

### 5. 权限检查不充分

**风险类型**: 权限提升

**证据**: `services/native/src/thermal_service.cpp:528-537`

```cpp
int32_t ThermalService::GetThermalLevel(int32_t& level)
{
    ThermalXCollie thermalXCollie("ThermalService::GetThermalLevel", false);
    if (actionMgr_ == nullptr) {
        THERMAL_HILOGD(COMP_SVC, "actionMgr_ is nullptr");
        return ERR_FAIL;
    }
    level = static_cast<int32_t>(actionMgr_->GetThermalLevel());
    return ERR_OK;
    // 没有权限检查，任何应用都可以查询
}
```

**证据**: `services/native/src/thermal_service.cpp:469-488`

```cpp
int32_t ThermalService::SubscribeThermalLevelCallback(const sptr<IThermalLevelCallback>& callback)
{
    ThermalXCollie thermalXCollie("ThermalService::SubscribeThermalLevelCallback", false);
    // 没有权限检查
    auto uid = IPCSkeleton::GetCallingUid();
    THERMAL_HILOGI(COMP_SVC, "ScbLevelCb uid=%{public}d", uid);
    actionMgr_->SubscribeThermalLevelCallback(callback);
    return ERR_OK;
}
```

**对比**: 某些接口有权限检查，某些没有

| API | 权限检查 | 证据 |
|---|---|---|
| `SubscribeThermalTempCallback` | ✅ `Permission::IsSystem()` | thermal_service.cpp:446 |
| `UnSubscribeThermalTempCallback` | ✅ `Permission::IsSystem()` | thermal_service.cpp:459 |
| `SubscribeThermalActionCallback` | ✅ `Permission::IsSystem()` | thermal_service.cpp:503 |
| `UnSubscribeThermalActionCallback` | ✅ `Permission::IsSystem()` | thermal_service.cpp:517 |
| `SetScene` | ✅ `Permission::IsSystem()` | thermal_service.cpp:573 |
| `UpdateThermalState` | ✅ `Permission::IsSystem()` | thermal_service.cpp:584 |
| `GetThermalLevel` | ❌ 无 | thermal_service.cpp:528 |
| `SubscribeThermalLevelCallback` | ❌ 无 | thermal_service.cpp:481 |
| `UnSubscribeThermalLevelCallback` | ❌ 无 | thermal_service.cpp:490 |
| `Dump` | ✅ `Permission::IsSystem()` + `isBootCompleted_` | thermal_service.cpp:738 |

**可利用场景**:
1. 恶意应用订阅温度/动作回调
2. 获取热级别信息用于侧信道攻击
3. 正常应用被恶意回调信息干扰

**影响**:
- 信息泄露
- 隐私侵犯
- 业务逻辑干扰

**修复建议**:
1. 为所有接口添加统一的权限检查
2. 限制订阅者数量和权限级别
3. 实现调用者身份验证
4. 添加订阅审计日志

**代码示例**:
```cpp
// 建议添加权限检查
int32_t ThermalService::GetThermalLevel(int32_t& level)
{
    ThermalXCollie thermalXCollie("ThermalService::GetThermalLevel", false);
    
    // 添加权限检查
    if (!Permission::IsSystem()) {
        THERMAL_HILOGE(COMP_SVC, "Permission denied");
        return ERR_PERMISSION_DENIED;
    }
    
    if (actionMgr_ == nullptr) {
        THERMAL_HILOGD(COMP_SVC, "actionMgr_ is nullptr");
        return ERR_FAIL;
    }
    level = static_cast<int32_t>(actionMgr_->GetThermalLevel());
    return ERR_OK;
}
```

---

### 6. 多线程竞态条件

**风险类型**: 竞态条件

**证据**: `services/native/include/thermal_observer/thermal_observer.h:101-106`

```cpp
class ThermalObserver {
    // 多个互斥锁，但存在死锁风险
    std::mutex mutexActionCallback_;
    std::mutex mutexTempCallback_;
    std::mutex mutexActionMap_;
    std::mutex mutexCallbackInfo_;
    // ...
};
```

**证据**: `frameworks/napi/thermal_manager_napi.cpp:43-44`

```cpp
void ThermalLevelCallback::UpdateCallback(napi_env env, napi_value jsCallback)
{
    std::lock_guard lock(mutex_);
    // 持有回调引用期间，如果 ReleaseCallback 被并发调用可能导致问题
    if (napi_ok != napi_create_reference(env, jsCallback, 1, &callbackRef_)) {
        callbackRef_ = nullptr;
    }
    env_ = env;
}
```

**可利用场景**:
1. 同时调用 Subscribe 和 Unsubscribe
2. 回调执行期间释放回调引用
3. 多线程同时访问共享资源

**影响**:
- 崩溃
- Use-after-free
- 数据不一致

**修复建议**:
1. 使用 RAII 模式管理资源
2. 实现引用计数
3. 使用 std::shared_ptr 管理共享资源
4. 添加线程安全的状态机

**代码示例**:
```cpp
// 使用 std::shared_ptr
class ThermalLevelCallback : public ThermalLevelCallbackStub {
public:
    void UpdateCallback(napi_env env, napi_value jsCallback);
    void ReleaseCallback();
    
private:
    std::shared_ptr<napi_ref> callbackRef_;
    std::shared_ptr<napi_env> env_;
    std::mutex mutex_;
};
```

---

### 7. 信息泄露风险（日志）

**风险类型**: 信息泄露

**证据**: `services/native/src/thermal_service.cpp:448, 463, 484`

```cpp
// 日志中记录 UID
auto uid = IPCSkeleton::GetCallingUid();
THERMAL_HILOGI(COMP_SVC, "ScbTempCb uid=%{public}d", uid);
```

**证据**: `services/native/src/thermal_service.cpp:463`

```cpp
// 日志中记录 PID
auto pid = IPCSkeleton::GetCallingPid();
auto uid = IPCSkeleton::GetCallingUid();
THERMAL_HILOGI(COMP_SVC, "ScbActionCb pid=%{public}d,uid=%{public}d", pid, uid);
```

**可利用场景**:
1. 日志访问泄露用户身份
2. 通过日志侧信道推断用户行为
3. 恶意应用分析其他应用的 UID

**影响**:
- 隐私泄露
- 用户跟踪
- 社会工程攻击

**修复建议**:
1. 移除不必要的 UID/PID 日志
2. 对敏感日志添加访问控制
3. 实现日志脱敏
4. 使用权限标记日志可见性

**代码示例**:
```cpp
// 建议移除或保护敏感日志
#ifdef THERMAL_DEBUG_LOGGING
auto uid = IPCSkeleton::GetCallingUid();
THERMAL_HILOGI(COMP_SVC, "ScbTempCb uid=%{public}d", uid);
#endif
```

---

## 信任边界

### 用户空间 vs 内核空间

| 边界 | 说明 | 保护机制 |
|---|---|---|
| 用户空间 → 内核 | N-API/Service/应用 | HDI 框架权限验证 |
| 内核空间 → 用户 | HDI 驱动回调 | 无直接用户访问 |

### 进程边界

| 进程 | 信任级别 | 通信方式 |
|---|---|---|
| 应用进程 | 低信任 | N-API 调用 |
| powermgr (SA) | 中信任 | IPC Binder |
| 内核驱动 | 高信任 | HDI 回调 |

### 权限边界

| 权限级别 | 接口 | 说明 |
|---|---|---|
| 无权限 | GetThermalLevel, Subscribe*LevelCallback | 可被所有应用调用 |
| 系统权限 | Subscribe*TempCallback, Subscribe*ActionCallback, SetScene, UpdateThermalState | 需要系统应用 |

---

## 安全加固建议

### 1. 统一权限检查

**建议**: 为所有敏感 API 添加权限检查

```cpp
// 建议添加的权限宏
#define CHECK_SYSTEM_PERMISSION() \
    if (!Permission::IsSystem()) { \
        THERMAL_HILOGE(COMP_SVC, "Permission denied"); \
        return ERR_PERMISSION_DENIED; \
    }
```

### 2. 输入验证加强

| 参数类型 | 当前状态 | 建议改进 |
|---|---|---|
| Callback | 仅类型检查 | 完整性校验 + 数量限制 |
| 场景字符串 | 无长度限制 | 最大长度 + 格式验证 |
| 温度值 | 无范围检查 | 有效范围验证 |
| 文件路径 | 无遍历保护 | 路径规范化 + 白名单 |

### 3. XML 安全配置

**建议**:
1. 禁用外部实体：`XML_PARSE_NOENT`
2. 限制配置文件来源：仅 `/system/` 和 `/vendor/etc/`
3. 添加配置文件签名验证
4. 实现配置缓存和哈希校验

### 4. 动态库安全

**建议**:
1. 使用绝对路径加载库
2. 验证库存在性和权限
3. 实现库完整性检查（哈希/签名）
4. 限制加载来源到系统目录

### 5. 日志安全

**建议**:
1. 移除 UID/PID 日志
2. 对敏感操作添加权限检查
3. 实现日志访问控制
4. 使用脱敏记录敏感数据

---

## 检查范围与局限性

### 已检查范围

✅ **已检查**:
- N-API 参数校验（`frameworks/napi/`）
- 权限检查（`services/native/src/thermal_service.cpp`）
- 配置文件解析（`services/native/src/thermal_policy/thermal_srv_config_parser.cpp`）
- 文件操作（`services/native/src/thermal_action/`）
- 动态库加载（`action_application_process.cpp`, `action_popup.cpp`）
- 多线程锁（`services/native/include/`）
- 日志记录（所有源文件）

### 未检查范围

⚠️ **未深入检查**:
- ETS/CJ API 安全（时间有限）
- Thermal Protector 完整实现
- 具体动作实现的漏洞（每个动作单独审计）
- HDI 驱动层面的漏洞（驱动代码不在此仓库）

### 局限性

1. **基于静态分析**：本评审基于代码静态分析，未进行动态测试
2. **依赖编译器保护**：CFI 和 PACRET 已启用，可缓解某些漏洞
3. **SELinux 上下文**：未深入检查 SELinux 策略配置

---

## 风险等级总结

| 风险 | 等级 | 优先级 |
|---|---|---|
| 缺少输入长度校验 | 中 | 高 |
| XML 注入 | 中 | 高 |
| 路径遍历 | 中 | 高 |
| 动态库加载 | 低 | 中 |
| 权限检查不充分 | 中 | 中 |
| 多线程竞态 | 低 | 中 |
| 信息泄露（日志） | 低 | 低 |

---

## 结论

Thermal Manager 的安全状况总体良好，但存在以下需要改进的方面：

**优点**:
- ✅ 启用了 CFI 和 PACRET 编译保护
- ✅ 使用了安全的字符串函数（`snprintf_s`）
- ✅ 大部分接口有权限检查
- ✅ 使用互斥锁保护共享资源

**需要改进**:
- ⚠️ 统一权限检查，避免不一致
- ⚠️ 加强输入验证，特别是 callback 完整性
- ⚠️ 添加 XML 解析的 XXE 防护
- ⚠️ 加强文件路径的安全检查
- ⚠️ 限制动态库加载来源
- ⚠️ 改进日志实践，避免信息泄露

**总体评估**: 中等风险，建议按优先级逐步修复。
