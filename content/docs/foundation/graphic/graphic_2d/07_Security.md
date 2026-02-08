# 安全风险评审

## 概述

本文档基于代码证据对 graphic_2d 进行安全风险分析，涵盖：
- 权限控制系统
- IPC 安全机制
- 信任边界
- 已识别的安全风险点

---

## 权限控制系统

### 权限字符串清单

graphic_2d 声明了以下敏感权限：

| 权限 | 用途 | 保护的操作 | 证据位置 |
|------|------|-----------|----------|
| `ohos.permission.CAPTURE_SCREEN` | 屏幕截图 | `TAKE_SURFACE_CAPTURE` | `rs_ipc_interface_code_access_verifier_base.cpp:22` |
| `ohos.permission.CAPTURE_SCREEN_ALL` | 全窗口捕获 | `TAKE_SURFACE_CAPTURE_WITH_ALL_WINDOWS` | 同上:25 |
| `ohos.permission.UPDATE_CONFIGURATION` | 配置更新 | `SET_REFRESH_RATE_MODE`, `SHOW_WATERMARK` | 同上:23 |
| `ohos.permission.GET_RUNNING_INFO` | 运行时信息 | `GET_MEMORY_GRAPHICS` | 同上:24 |

### 权限检查机制

权限检查在 IPC 接口代码访问验证器中实现：

```cpp
// rs_ipc_interface_code_access_verifier_base.cpp
bool RSIPCInterfaceCodeAccessVerifierBase::CheckPermission(
    CodeUnderlyingType code) const
{
    // 根据接口代码检查对应权限
    if (!hasPermission) {
        return false;  // 无权限拒绝
    }
    return true;
}
```

**证据来源**: `rs_ipc_interface_code_access_verifier_base.cpp:94-96`

---

## UID 白名单机制

### 硬编码的系统服务 UID

```cpp
static constexpr uint32_t ANCO_UID = 5557;              // ANCO 服务
static constexpr uint32_t FOUNDATION_UID = 5523;        // Foundation 服务
static constexpr uint32_t STYLUS_SERVICE_UID = 7555;    // 手写笔服务
static constexpr uint32_t EXFUSION_SERVICE_UID = 7015;  // 异显服务
static constexpr uint32_t TASK_MANAGER_SERVICE_UID = 7005; // 任务管理器
```

**证据来源**: `rs_ipc_interface_code_access_verifier_base.cpp:173-262`

### 进程名验证

部分服务需要 UID + 进程名双重验证：

```cpp
// Stylus Service 验证
static const std::string STYLUS_SERVICE_PROCESS_NAME = "stylus_service";
bool isStylusServiceProcessName = callingProcessName == STYLUS_SERVICE_PROCESS_NAME;

// 三重验证通过条件
isSystemCalling = isNativeCalling && isStylusServiceUid && isStylusServiceProcessName;
```

---

## Token 类型验证

系统验证三种 Token 类型：

```cpp
case Security::AccessToken::ATokenTypeEnum::TOKEN_HAP:     // 第三方应用
case Security::AccessToken::ATokenTypeEnum::TOKEN_NATIVE:  // Native 进程
case Security::AccessToken::ATokenTypeEnum::TOKEN_SHELL:   // Shell 进程
```

**证据来源**: `rs_ipc_interface_code_access_verifier_base.cpp`

---

## 黑名单/白名单安全模型

### 虚拟屏幕安全列表

| 列表类型 | 用途 | 访问控制 |
|---------|------|----------|
| `BlackList` | 排除在虚拟屏幕外的节点 | 系统调用修改 |
| `WhiteList` | 仅允许在安全显示的节点 | 系统调用修改 |
| `TypeBlackList` | 排除的节点类型 | 系统调用修改 |

**受保护的 IPC 接口**:
- `SET_VIRTUAL_SCREEN_BLACKLIST` - 系统调用
- `SET_VIRTUAL_SCREEN_TYPE_BLACKLIST` - 系统调用
- `ADD_VIRTUAL_SCREEN_WHITELIST` - 系统调用
- `REMOVE_VIRTUAL_SCREEN_BLACKLIST` - 系统调用

**证据来源**:
- `rs_screen_thread_safe_property.cpp`
- `rs_special_layer_manager.cpp`

---

## N-API 层安全

### 系统应用验证

```cpp
// ui_effect_napi_utils.cpp
bool UIEffectNapiUtils::IsSystemApp() {
    uint64_t tokenId = OHOS::IPCSkeleton::GetCallingFullTokenID();
    return Security::AccessToken::AccessTokenKit::IsSystemAppByFullTokenID(tokenId);
}
```

### 受保护的 N-API 函数

以下滤镜效果函数受 `IsSystemApp()` 保护：

| 函数 | 文件 | 行号 |
|------|------|------|
| FrostedGlass 系列 (11 函数) | `filter_napi.cpp` | - |
| BorderLight 系列 | `filter_napi.cpp` | - |
| 背景色效果 | `filter_napi.cpp` | - |
| Harmonium 效果 | `filter_napi.cpp` | - |
| Mask 操作 | `mask_napi.cpp` | - |

### FormRenderService 例外

```cpp
bool UIEffectNapiUtils::IsFormRenderServiceCall() {
    static const std::string frsBundleName = "com.ohos.formrenderservice";
    return bundleName == frsBundleName;
}
```

**证据来源**: `ui_effect_napi_utils.cpp`

---

## 安全风险清单

### 风险 1: IPC 安全编译时禁用 ⚠️ 高风险

**位置**: `rs_ipc_interface_code_access_verifier_base.cpp:268-299`

```cpp
#ifndef ENABLE_IPC_SECURITY
// 当 IPC 安全被禁用时，所有安全检查返回 true！
bool IsSystemCalling(const std::string& /* callingCode */) { return true; }
bool CheckPermission(CodeUnderlyingType code) const { return true; }
#endif
```

**风险说明**: 
- 编译时宏可完全禁用所有 IPC 安全检查
- 生产版本必须确保启用 `ENABLE_IPC_SECURITY`
- 禁用后任何进程都可调用敏感接口

**建议**:
- 确保生产构建启用安全检查
- 添加运行时断言验证安全状态
- 考虑移除条件编译路径

---

### 风险 2: 硬编码 UID 常量 ⚠️ 中风险

**位置**: `rs_ipc_interface_code_access_verifier_base.cpp:173-262`

```cpp
static constexpr uint32_t ANCO_UID = 5557;
static constexpr uint32_t FOUNDATION_UID = 5523;
static constexpr uint32_t STYLUS_SERVICE_UID = 7555;
static constexpr uint32_t EXFUSION_SERVICE_UID = 7015;
static constexpr uint32_t TASK_MANAGER_SERVICE_UID = 7005;
```

**风险说明**:
- 无集中配置或验证机制
- UID 冲突可能导致权限绕过
- 系统 UID 变更需同步更新

**建议**:
- 使用配置系统管理 UID
- 添加运行时 UID 验证
- 文档化所有硬编码 UID

---

### 风险 3: Async Binder GetCallingPid 返回 0 ⚠️ 中风险

**位置**: `rs_client_to_render_connection_stub.cpp:363-365`

```cpp
// 注释说明:
// "Since GetCallingPid interface always returns 0 in asynchronous binder in Linux kernel system,
// The white list will be removed after GetCallingPid interface can return real PID."
```

**风险说明**:
- 异步 binder 调用无法可靠识别调用方 PID
- 白名单机制可能失效
- 潜在 PID 欺骗风险

**建议**:
- 短期: 记录警告日志
- 长期: 等待内核修复或使用替代方案

---

### 风险 4: Bundle Name 字符串比较 ⚠️ 低风险

**位置**: `ui_effect_napi_utils.cpp:269`

```cpp
static const std::string bundleName = GetBundleName();
return bundleName == frsBundleName;
```

**风险说明**:
- 字符串比较可被操纵
- GetBundleName() 失败返回空字符串
- 潜在绕过风险

**建议**:
- 添加额外的签名验证
- 使用常量比较优化
- 记录失败尝试

---

### 风险 5: 白名单大小限制仅日志警告 ⚠️ 低风险

**位置**: `rs_special_layer_manager.cpp:54-55`

```cpp
if (whiteListRootIds_.size() >= MAX_SPECIAL_LAYER_NUM) {
    RS_LOGE("whiteListRootIds_ exceeds size limit");  // 仅日志，未阻止插入
```

**风险说明**:
- 超过限制仅记录错误日志
- 调用方可能未处理失败
- DoS 风险

**建议**:
- 返回明确的错误码
- 文档化失败处理要求
- 考虑拒绝新条目而非静默忽略

---

## 边界检查机制

### 数据大小验证

```cpp
// 粒子发射器限制
if (size > PARTICLE_EMMITER_UPPER_LIMIT) { /* 拒绝 */ }

// 安全豁免列表限制
if (securityExemptionList.size() > MAX_SECURITY_EXEMPTION_LIST_NUMBER) { /* 拒绝 */ }

// 丢帧 PID 列表大小
if (pidList.size() > MAX_DROP_FRAME_PID_LIST_SIZE) { /* 拒绝 */ }

// Keyframe 大小限制
if (size > MAX_KEYFRAME_SIZE_NUMBER) { /* 拒绝 */ }

// 数据大小验证
if (size > MAX_DATA_SIZE || size < MIN_DATA_SIZE) { /* 拒绝 */ }
```

**证据来源**: `rs_marshalling_helper.cpp`, `rs_client_to_service_connection_proxy.cpp`

### 事务数据防 DoS

```cpp
// rs_unmarshal_thread.cpp - 超过限制则杀死应用
if (totalCount > TRANSACTION_DATA_KILL_COUNT) {
    appMgrClient->KillApplicationByUid(bundleName, uid);
}
```

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         信任边界                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  Render Service (服务端)                                              │   │
│  │  • IPC 接口访问控制                                                   │   │
│  │  • UID/Token 验证                                                    │   │
│  │  • 权限检查                                                          │   │
│  │  • 事务数据大小验证                                                   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                              │                                             │
│                          IPC 边界                                           │
│                              │                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  Render Service Client (客户端库)                                     │   │
│  │  • N-API 参数验证                                                    │   │
│  │  • 系统应用检查                                                      │   │
│  │  • 权限声明                                                         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                              │                                             │
│                          应用边界                                            │
│                              │                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  Application (应用层)                                                │   │
│  │  • JS/ArkTS 代码                                                     │   │
│  │  • 外部输入                                                          │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 攻击面分析

### 输入点

| 输入类型 | 处理位置 | 风险等级 |
|---------|----------|----------|
| **N-API 参数** | `interfaces/kits/napi/` | 中 |
| **IPC 消息** | `render_service_base/src/` | 高 |
| **Surface Buffer** | `rs_client_to_render_connection` | 中 |
| **配置文件** | `*.cfg`, `*.json` | 低 |
| **共享内存** | `rs_ashmem_helper.cpp` | 中 |

### 敏感操作

| 操作 | 权限要求 | 风险影响 |
|------|----------|----------|
| 屏幕截图 | `CAPTURE_SCREEN` | 隐私泄露 |
| 全窗口捕获 | `CAPTURE_SCREEN_ALL` | 隐私泄露 |
| 配置更新 | `UPDATE_CONFIGURATION` | DoS |
| 虚拟屏幕创建 | 系统调用 | 屏幕欺骗 |
| 帧率修改 | `UPDATE_CONFIGURATION` | 性能影响 |

---

## 安全建议总结

### 必须立即处理

1. **验证 ENABLE_IPC_SECURITY 在生产构建中启用**
2. **审计所有使用 `ENABLE_IPC_SECURITY` 的代码路径**
3. **为硬编码 UID 添加配置化方案**

### 建议优化

1. **完善白名单大小超限的错误处理**
2. **为 bundle name 比较添加额外验证**
3. **记录所有被拒绝的权限检查**

### 长期改进

1. **解决异步 binder GetCallingPid 返回 0 的问题**
2. **添加安全事件日志审计**
3. **考虑使用 MAC (Mandatory Access Control) 增强隔离**

---

## 相关文档

- [架构说明](03_Architecture.md) - 信任边界
- [N-API 接口](04_N-API.md) - API 安全
- [构建文档](06_Build.md) - 安全相关编译开关
