# 安全风险评审

## 评审范围

本文档对 `dataclassification` 模块进行安全风险分析，评审范围：

- **代码文件**：`frameworks/datatransmitmgr/`、`interfaces/inner_api/datatransmitmgr/`
- **构建文件**：`BUILD.gn`、`interfaces/inner_api/datatransmitmgr/BUILD.gn`
- **排除范围**：`test/` 目录测试代码不计入评审

## 威胁模型

### 外部输入

| 输入源 | 数据类型 | 信任级别 |
|-------|---------|---------|
| 调用方传入 UDID | `uint8_t[]` | 半信任 |
| 调用方传入回调函数 | 函数指针 | 半信任 |
| dlopen SDK 路径 | 硬编码字符串 | 信任 |

### 敏感操作

- 动态加载 SDK（`dlopen`）
- 函数指针调用（回调）
- 内存拷贝（`memcpy_s`）
- 链表操作

## 攻击面清单

| 攻击面 | 类型 | 风险等级 |
|-------|------|---------|
| API 参数校验 | 输入验证 | 中 |
| dlopen 路径注入 | 动态库加载 | 低 |
| 回调函数指针 | 回调安全 | 中 |
| 内存操作 | 内存安全 | 低 |
| 链表竞争条件 | 线程安全 | 中 |

## 安全加固措施（已实现）

### 编译时加固

| 加固项 | 配置 | 证据 |
|-------|------|------|
| FORTIFY_SOURCE | `-D_FORTIFY_SOURCE=2` | BUILD.gn:66 |
| 栈保护 | `-fstack-protector-strong` | BUILD.gn:70 |
| 控制流完整性 (CFI) | `cfi = true` | BUILD.gn:48 |
| 整数溢出检测 | `integer_overflow = true` | BUILD.gn:51 |
| 未定义行为检测 | `ubsan = true` | BUILD.gn:52 |
| 边界检查 | `boundary_sanitize = true` | BUILD.gn:53 |

> 证据：`interfaces/inner_api/datatransmitmgr/BUILD.gn:45-55`

### 运行时安全

| 安全措施 | 实现 | 证据 |
|---------|------|------|
| 参数校验 | NULL 检查 + 范围检查 | dev_slinfo_mgr.c:73-75 |
| 边界安全函数 | `memcpy_s`、`memset_s` | dev_slinfo_adpt.c:158-162 |
| 互斥锁保护 | `pthread_mutex` | dev_slinfo_list.c:22 |

---

## 风险分析与修复建议

### 风险 1：dlopen 路径注入

**风险等级**：低

**证据**：`dev_slinfo_adpt.c:45`

```c
g_deviceSecLevelHandle = dlopen("libdslm_sdk.z.so", RTLD_LAZY | RTLD_NODELETE);
```

**问题描述**：
- SDK 路径硬编码，无路径校验机制
- 攻击者可能通过修改 LD_LIBRARY_PATH 注入恶意库

**当前缓解**：
- 仅加载 `libdslm_sdk.z.so`，路径相对固定
- 使用 `RTLD_NODELETE` 防止库被卸载

**修复建议**（可选）：
```c
// 验证 SDK 签名或路径白名单
const char *sdk_path = "/system/lib64/libdslm_sdk.z.so";
if (access(sdk_path, F_OK) != 0) {
    return DEVSL_ERROR;
}
g_deviceSecLevelHandle = dlopen(sdk_path, RTLD_LAZY | RTLD_NODELETE);
```

---

### 风险 2：回调函数指针未校验

**风险等级**：中

**证据**：`dev_slinfo_adpt.c:220-231`

```c
// 直接调用回调，无校验
if (g_callbackList != NULL) {
    LookupCallback(g_callbackList, &queryParams, ret, levelInfo);
}
```

**问题描述**：
- 异步回调从链表取出后直接调用
- 若回调函数已被释放（UAF），可能导致崩溃或代码执行

**当前缓解**：
- 链表操作受 `pthread_mutex` 保护
- 回调在链表移除后立即执行

**修复建议**（建议实现）：
```c
// 在调用前校验回调指针有效性
if (callback != NULL && IsValidFunctionPointer(callback)) {
    callback(queryParams, result, levelInfo);
}
```

---

### 风险 3：UDID 长度校验边界

**风险等级**：低

**证据**：`dev_slinfo_mgr.c:31`

```c
if ((queryParams->udidLen <= 0u) || (queryParams->udidLen > 64u)) {
    return DEVSL_ERR_BAD_PARAMETERS;
}
```

**问题描述**：
- `udidLen` 范围检查为 `(0, 64]`
- 长度为 0 被拒绝，但边界值 64 被允许

**当前缓解**：
- 使用 `memcpy_s` 进行安全拷贝，自动处理边界
- `MAX_UDID_LENGTH` 定义为 64

**评估**：当前实现安全，无实际风险

---

### 风险 4：链表竞争条件

**风险等级**：中

**证据**：`dev_slinfo_list.c:48-61`

```c
int32_t PushListNode(struct DATASLListParams *list, struct DATASLCallbackParams *callbackParams)
{
    (void)pthread_mutex_lock(&g_mutex);
    // ... 操作链表
    (void)pthread_mutex_unlock(&g_mutex);
}
```

**问题描述**：
- 多线程同时 `PushListNode` 可能导致链表不一致
- 但使用了 `pthread_mutex` 保护

**当前缓解**：
- 所有链表操作已加锁
- 初始化时检查互斥锁状态

**修复建议**（可选）：
```c
// 添加超时锁防止死锁
if (pthread_mutex_timedlock(&g_mutex, &timeout) != 0) {
    return DEVSL_ERROR;
}
```

---

### 风险 5：dlsym 符号查找失败处理

**风险等级**：低

**证据**：`dev_slinfo_adpt.c:66-94`

```c
RequestDeviceSecurityInfoFunction requestDeviceSecurityInfo = (RequestDeviceSecurityInfoFunction)dlsym(
    g_deviceSecLevelHandle, "RequestDeviceSecurityInfo");
if (requestDeviceSecurityInfo == NULL) {
    dlclose(g_deviceSecLevelHandle);
    g_deviceSecLevelHandle = NULL;
    return DEVSL_ERROR;
}
```

**问题描述**：
- SDK 符号查找失败会关闭句柄并返回错误
- 但未记录详细错误原因

**当前缓解**：
- 每个符号查找失败都有独立错误处理
- 使用 `dlerror()` 获取详细错误

**评估**：当前实现安全，无实际风险

---

## 安全总结

| 风险项 | 当前状态 | 建议 |
|-------|---------|------|
| dlopen 路径注入 | 已缓解 | 考虑路径白名单 |
| 回调函数指针 | 已缓解 | 增加指针有效性校验 |
| UDID 长度校验 | 安全 | 无需修改 |
| 链表竞争条件 | 已缓解 | 考虑添加超时锁 |
| dlsym 符号查找 | 安全 | 无需修改 |

## 安全最佳实践

### 调用方注意事项

1. **初始化前使用**：确保先调用 `DATASL_OnStart()` 再使用其他 API
2. **去初始化**：使用完成后调用 `DATASL_OnStop()` 释放资源
3. **UDID 校验**：传入前验证 UDID 长度在有效范围内
4. **回调生命周期**：确保回调函数在异步调用完成前保持有效

### 开发规范

1. **保持安全加固**：不要移除 CFI、UBSAN 等编译选项
2. **使用安全函数**：始终使用 `memcpy_s` 替代 `memcpy`
3. **错误处理**：所有外部调用必须检查返回值
4. **日志脱敏**：UDID 等敏感信息脱敏后记录

---

## 参考文档

- [架构说明](Architecture.md) - 模块架构
- [内部 API](Inner_API.md) - API 安全使用指南
- [构建配置](Build.md) - 安全加固选项
