# 安全风险评估

## 目的

本文档基于代码证据对 ability_lite 进行安全风险评估，识别攻击面、信任边界和可被利用点。

## 适用范围

- 安全审计人员
- 需要了解安全风险的开发者

## 攻击面清单

### 1. N-API / JS API 攻击面

**位置**: `interfaces/kits/js/napi/js_aafwk.cpp`

| 攻击向量 | 风险等级 | 说明 |
|----------|----------|------|
| JS 参数解析 | 中 | Want 对象解析，字符串提取 |
| 内存分配 | 中 | malloc/free 使用 |
| 类型混淆 | 低 | napi_typeof 检查 |

### 2. IPC 攻击面

**位置**: `services/abilitymgr_lite/src/ability_mgr_feature.cpp`

| 攻击向量 | 风险等级 | 说明 |
|----------|----------|------|
| IPC 命令处理 | 高 | 所有 AMS 命令入口 |
| 身份验证 | 中 | GetCallingUid/GetCallingPid |
| 数据反序列化 | 中 | IpcIo 数据处理 |

### 3. 权限管理攻击面

**位置**: `services/abilitymgr_lite/src/app_record.cpp`

| 攻击向量 | 风险等级 | 说明 |
|----------|----------|------|
| 权限加载 | 高 | LoadPermissions 调用 |
| 权限查询 | 中 | QueryAppCapabilities |

### 4. 进程管理攻击面

**位置**: `services/abilitymgr_lite/src/client/app_spawn_client.cpp`

| 攻击向量 | 风险等级 | 说明 |
|----------|----------|------|
| 进程创建 | 高 | 与 AppSpawn 通信 |
| Bundle 信息查询 | 中 | 与 BundleMS 通信 |

### 5. 动态加载攻击面

**位置**: `frameworks/ability_lite/src/ability_thread.cpp`

| 攻击向量 | 风险等级 | 说明 |
|----------|----------|------|
| 库加载 | 高 | dlopen 调用 |
| 路径解析 | 中 | realpath 验证 |

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│  Trust Zone: System (高权限)                                │
│  - AMS Service (abilityms)                                  │
│  - AppSpawn 服务                                            │
│  - Permission Service                                       │
│                                                             │
│  权限: 创建进程、加载权限、管理应用生命周期                  │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ IPC (受控通道)
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Trust Zone: Application (低权限)                           │
│  - AbilityKit                                               │
│  - 应用代码                                                 │
│                                                             │
│  权限: 受限，通过 IPC 请求系统服务                           │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ N-API / JS 绑定
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Trust Zone: JavaScript (不受信任)                          │
│  - JS 应用代码                                              │
│                                                             │
│  权限: 完全受限，通过 N-API 调用 Native 功能                 │
└─────────────────────────────────────────────────────────────┘
```

## 可被利用点分析

### 风险 1: UID 验证不充分

**证据**: `services/abilitymgr_lite/src/ability_mgr_feature.cpp:149-152`

```cpp
pid_t uid = GetCallingUid();
if (uid < 0) {
    PRINTE("AbilityMgrFeature", "invalid uid argument");
    return EC_INVALID;
}
```

**问题分析**:
- 仅检查 UID 是否为负值
- 无进一步权限校验（如是否允许启动特定 Ability）
- 依赖外部 permission_lite 进行权限检查

**触发路径**:
```
JS/Native 调用 StartAbility
    -> IPC SendRequest(START_ABILITY)
        -> AbilityMgrFeature::StartAbilityInvoke()
            -> GetCallingUid() (仅检查 uid < 0)
                -> StartAbilityInner() (继续执行)
```

**影响**: 恶意应用可能绕过权限检查启动未授权的 Ability

**修复建议**:
```cpp
// 建议增加权限校验
pid_t uid = GetCallingUid();
if (uid < 0) {
    return EC_INVALID;
}
// 增加：检查 UID 是否有权限启动目标 Ability
if (!CheckPermission(uid, want, PERMISSION_START_ABILITY)) {
    return EC_PERMISSION_DENIED;
}
```

### 风险 2: Want 数据大小限制绕过

**证据**: `frameworks/want_lite/src/want.cpp:286-289`

```cpp
if ((io == nullptr) || (want == nullptr) || (want->dataLength > DATA_LENGTH)) {
    return false;
}
```

**问题分析**:
- DATA_LENGTH 限制为 2048 字节
- 检查仅在反序列化时进行
- 无内容验证，仅大小检查

**触发路径**:
```
构造超大 Want 数据
    -> IPC 传输
        -> DeserializeWant()
            -> 如果 dataLength <= 2048 则通过
                -> 后续处理可能溢出
```

**影响**: 可能导致缓冲区溢出或拒绝服务

**修复建议**:
```cpp
// 增加内容深度验证
if (want->dataLength > 0) {
    // 验证数据格式合法性
    if (!ValidateWantData(want->data, want->dataLength)) {
        return false;
    }
}
```

### 风险 3: N-API 内存泄漏

**证据**: `interfaces/kits/js/napi/js_aafwk.cpp:136-155`

```cpp
static bool GetCharPointerArgument(napi_env env, napi_value value, char* result)
{
    // ...
    result = (char *) malloc((bufLength + 1) * sizeof(char));
    if (result != nullptr) {
        status = napi_get_value_string_utf8(env, value, result, bufLength + 1, &bufLength);
        if (status == napi_ok) {
            ret = true;
        }
        // 注意：如果 status != napi_ok，result 未释放
    }
    return ret;
}
```

**问题分析**:
- malloc 分配的内存如果在 napi_get_value_string_utf8 失败时未释放
- 可能导致内存泄漏

**触发路径**:
```
JS 调用 startAbility(want)
    -> JSAafwkStartAbility()
        -> GetWantFromNapiValue()
            -> GetCharPointerArgument() (失败时泄漏)
```

**影响**: 长时间运行可能导致内存耗尽

**修复建议**:
```cpp
static bool GetCharPointerArgument(napi_env env, napi_value value, char* &result)
{
    result = nullptr;
    // ...
    result = (char *) malloc((bufLength + 1) * sizeof(char));
    if (result != nullptr) {
        status = napi_get_value_string_utf8(env, value, result, bufLength + 1, &bufLength);
        if (status != napi_ok) {
            free(result);  // 释放内存
            result = nullptr;
            return false;
        }
        return true;
    }
    return false;
}
```

### 风险 4: Token 溢出可能导致冲突

**证据**: `services/abilitymgr_lite/src/slite/ability_record_manager.cpp:717`

```cpp
uint16_t AbilityRecordManager::GenerateToken()
{
    uint16_t token = INVALID_TOKEN;
    for (int i = 0; i < MAX_TOKEN_TRY_TIMES; i++) {
        token = nextToken_++;
        // ...
    }
    return token;
}
```

**问题分析**:
- Token 为 uint16_t，范围 0-65535
- 长时间运行可能溢出回绕
- 可能导致 Token 冲突

**触发路径**:
```
大量 Ability 启动/停止
    -> nextToken_ 递增
        -> 超过 65535 后回绕
            -> 可能生成重复 Token
```

**影响**: Token 冲突可能导致权限混淆

**修复建议**:
```cpp
// 增加 Token 冲突检测
uint16_t AbilityRecordManager::GenerateToken()
{
    for (int i = 0; i < MAX_TOKEN_TRY_TIMES; i++) {
        uint16_t token = nextToken_++;
        // 检查是否已存在
        if (FindAbilityRecordByToken(token) == nullptr) {
            return token;
        }
    }
    return INVALID_TOKEN;
}
```

### 风险 5: 动态库加载路径验证

**证据**: `frameworks/ability_lite/src/ability_thread.cpp:174-185`

```cpp
if (modulePath.size() > PATH_MAX) {
    continue;
}
char realPath[PATH_MAX + 1] = { 0 };
if (realpath(modulePath.c_str(), realPath) == nullptr) {
    HILOG_ERROR(...);
    continue;
}
void *handle = dlopen(static_cast_cast<char *>(realPath), RTLD_NOW | RTLD_GLOBAL);
```

**问题分析**:
- 使用 realpath 验证路径
- 但未验证路径是否在允许目录内
- 如果 modulePath 被篡改，可能加载恶意库

**触发路径**:
```
构造恶意 modulePath
    -> AbilityThread::PerformAppInit()
        -> realpath() 解析
            -> dlopen() 加载
```

**影响**: 可能加载并执行恶意代码

**修复建议**:
```cpp
// 增加路径白名单检查
if (!IsPathInAllowedList(realPath)) {
    HILOG_ERROR("Path %s not in allowed list", realPath);
    continue;
}
void *handle = dlopen(realPath, RTLD_NOW | RTLD_GLOBAL);
```

## 安全建议总结

| 优先级 | 建议 | 涉及文件 |
|--------|------|----------|
| 高 | 增加 UID 权限校验 | `ability_mgr_feature.cpp` |
| 高 | 修复 N-API 内存泄漏 | `js_aafwk.cpp` |
| 中 | 增加 Want 数据内容验证 | `want.cpp` |
| 中 | 增加 Token 冲突检测 | `ability_record_manager.cpp` |
| 中 | 增加库路径白名单 | `ability_thread.cpp` |
| 低 | 增加 IPC 数据签名验证 | `ability_mgr_feature.cpp` |

## 检查范围与局限性

### 已检查范围
- N-API 绑定层
- IPC 命令处理入口
- 权限管理调用
- 内存分配操作
- 动态库加载

### 未检查范围（依赖外部组件）
- permission_lite 的具体实现
- IPC 底层传输安全
- Bundle Manager 的验证逻辑
- AppSpawn 的进程创建安全

### 局限性说明
1. 本评估仅基于 ability_lite 仓库代码
2. 完整安全评估需要结合依赖组件分析
3. 部分风险需要运行时测试验证

## 相关链接

- [架构说明](02_Architecture.md)
- [内部 API](05_Inner_API.md)
- [目录结构](01_Directory_Structure.md)
