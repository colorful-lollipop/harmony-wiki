# 安全风险分析

## 目的

本文档基于代码证据对 Window Manager 子系统进行安全风险评估，包括攻击面、信任边界、可利用点和修复建议。

**分析范围**：
- 代码路径：`wm/`, `dm/`, `wmserver/`, `dmserver/`, `window_scene/`, `interfaces/`
- 排除测试代码：`test/` 目录
- 基于版本：当前代码仓库 (2025-02-06)

## 威胁模型

### 攻击面分类

| 攻击面 | 入口点 | 风险等级 |
|--------|--------|----------|
| N-API/JS API | 应用调用窗口 API | 高 |
| IPC 接口 | 跨进程通信 | 高 |
| 系统服务 | SA 接口 | 中 |
| 文件系统 | 配置文件、缓存 | 低 |
| 网络 | 分布式软总线 | 中 |

### 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│ Level 5: 第三方应用 (TOKEN_HAP)                              │
│  - 只能访问公共 API                                          │
│  - 需要显式权限声明                                          │
├─────────────────────────────────────────────────────────────┤
│ Level 4: 系统应用 (IsSystemAppByFullTokenID)                 │
│  - 可使用系统 API                                            │
│  - 受权限控制                                                │
├─────────────────────────────────────────────────────────────┤
│ Level 3: Shell (TOKEN_SHELL)                                 │
│  - ADB/hdcd 调试访问                                         │
│  - 受限系统访问                                              │
├─────────────────────────────────────────────────────────────┤
│ Level 2: System Ability (TOKEN_NATIVE)                       │
│  - 系统服务间通信                                            │
│  - 大部分 API 可访问                                         │
├─────────────────────────────────────────────────────────────┤
│ Level 1: Foundation (UID 5523)                               │
│  - 最高权限                                                  │
│  - 所有操作允许                                              │
└─────────────────────────────────────────────────────────────┘
```

## 安全机制概览

### 权限检查机制

**核心类**: `SessionPermission` (`window_scene/common/src/session_permission.cpp`)

```cpp
// 主要权限检查方法
bool VerifyCallingPermission(const std::string& permission);
bool VerifyPermissionByCallerToken(const std::string& permission, uint32_t tokenId);
bool VerifyPermissionByBundleName(const std::string& permission, const std::string& bundleName);
bool IsSystemCalling();
bool IsSACalling();
bool IsFoundationCall();
```

### 调用者身份验证

```cpp
// 获取调用者信息
uint32_t tokenId = IPCSkeleton::GetCallingTokenID();
int32_t uid = IPCSkeleton::GetCallingUid();
int32_t pid = IPCSkeleton::GetCallingPid();
int32_t realPid = IPCSkeleton::GetCallingRealPid();

// Token 类型检查
ATokenTypeEnum tokenType = AccessTokenKit::GetTokenTypeFlag(tokenId);
bool isSystemApp = TokenIdKit::IsSystemAppByFullTokenID(fullTokenId);
```

## 可利用点分析

### 风险 1: 截图权限绕过

**证据位置**: `screen_session_manager.cpp:2883`

```cpp
// screen_session_manager.cpp:13223
bool ScreenSessionManager::CheckScreenCapturePermission() {
    auto callerToken = IPCSkeleton::GetCallingTokenID();
    int32_t result = AccessTokenKit::VerifyAccessToken(callerToken, SCREEN_CAPTURE_PERMISSION);
    if (result != RET_SUCCESS) {
        // 检查自定义截图权限
        result = AccessTokenKit::VerifyAccessToken(callerToken, CUSTOM_SCREEN_CAPTURE_PERMISSION);
    }
    return result == RET_SUCCESS;
}
```

**风险描述**:
- 攻击者可尝试通过自定义截图权限绕过标准截图权限检查
- 若 `CUSTOM_SCREEN_CAPTURE_PERMISSION` 被授予给非可信应用，可能导致隐私泄露

**触发路径**:
```
JS: screenshot.save()
  → NAPI: ScreenshotModuleInit()
  → Native: save::MainFunc()
  → IPC: ScreenSessionManager::GetScreenSnapshot()
  → Permission Check: CheckScreenCapturePermission()
```

**影响**: 高 - 可获取屏幕内容，泄露敏感信息

**修复建议**:
```cpp
// 建议增加多重校验
bool CheckScreenCapturePermission() {
    auto tokenId = IPCSkeleton::GetCallingTokenID();
    
    // 1. 检查标准权限
    if (VerifyAccessToken(tokenId, SCREEN_CAPTURE_PERMISSION) != RET_SUCCESS) {
        return false;
    }
    
    // 2. 增加系统应用校验
    if (!IsSystemAppByFullTokenID(IPCSkeleton::GetCallingFullTokenID())) {
        TLOGW("Screen capture rejected for non-system app");
        return false;
    }
    
    // 3. 记录权限使用
    AddPermissionUsedRecord(tokenId, SCREEN_CAPTURE_PERMISSION, 1, 0);
    return true;
}
```

---

### 风险 2: 窗口句柄重用攻击

**证据位置**: `scene_session_manager.cpp` (多处 PID 校验)

```cpp
// scene_session.cpp:9253-9258
int32_t callingPid = IPCSkeleton::GetCallingPid();
if (callingPid != -1 && callingPid != GetCallingPid()) {
    TLOGE(WmsLogTag::WMS_LIFE, "permission denied! persistentId:%{public}d, "
          "callingPid_:%{public}d, callingPid:%{public}d", 
          GetPersistentId(), GetCallingPid(), callingPid);
    return WMError::WM_ERROR_INVALID_OPERATION;
}
```

**风险描述**:
- 若 PID 校验逻辑存在缺陷，攻击者可能重用已释放窗口的 persistentId
- 通过伪造 IPC 调用，可能操作其他应用的窗口

**触发路径**:
```
App A: 创建窗口 → 获取 persistentId=100
App A: 销毁窗口 → persistentId 回收
App B: 尝试使用 persistentId=100 操作窗口
  → 若 PID 校验不完善，可能成功
```

**影响**: 高 - 可操控其他应用窗口，实施界面劫持

**修复建议**:
```cpp
// 增加 Token 绑定校验
bool ValidateWindowOwnership(int32_t persistentId, uint32_t callerToken) {
    auto session = GetSceneSession(persistentId);
    if (!session) return false;
    
    // 1. PID 校验
    if (session->GetCallingPid() != IPCSkeleton::GetCallingPid()) {
        return false;
    }
    
    // 2. Token 校验
    if (session->GetCallingTokenId() != callerToken) {
        TLOGE("Token mismatch for window %{public}d", persistentId);
        return false;
    }
    
    // 3. BundleName 校验
    if (!IsSameBundleNameAsCalling(session->GetBundleName())) {
        return false;
    }
    
    return true;
}
```

---

### 风险 3: 虚拟屏幕创建权限绕过

**证据位置**: `screen_session_manager.cpp:2872`

```cpp
// 检查虚拟屏幕访问权限
bool ScreenSessionManager::CheckVirtualScreenAccessPermission() {
    auto callerToken = IPCSkeleton::GetCallingTokenID();
    int32_t result = AccessTokenKit::VerifyAccessToken(callerToken, ACCESS_VIRTUAL_SCREEN_PERMISSION);
    return result == RET_SUCCESS;
}
```

**风险描述**:
- 虚拟屏幕可用于屏幕录制、投影等敏感操作
- 权限检查较为单一，可能通过权限混淆绕过

**触发路径**:
```
JS: display.createVirtualScreen()
  → DM: DisplayManager::CreateVirtualScreen()
  → IPC: ScreenSessionManager::CreateVirtualScreen()
  → Permission: CheckVirtualScreenAccessPermission()
```

**影响**: 中 - 可创建虚拟屏幕，可能导致信息泄露

**修复建议**:
- 增加应用签名白名单
- 限制虚拟屏幕数量
- 增加用户授权确认

---

### 风险 4: IPC 接口整数溢出

**证据位置**: 多处 IPC 参数解析

```cpp
// 典型模式（窗口大小参数）
uint32_t width = data.ReadUint32();
uint32_t height = data.ReadUint32();
// 直接用于内存分配，可能溢出
```

**风险描述**:
- IPC 消息解析时，若对整数参数范围校验不足
- 可能导致整数溢出，进而造成缓冲区溢出

**影响**: 中 - 可能导致服务崩溃或内存损坏

**修复建议**:
```cpp
// 增加范围校验
bool ValidateWindowSize(uint32_t width, uint32_t height) {
    if (width == 0 || height == 0) {
        TLOGE("Invalid window size: %{public}u x %{public}u", width, height);
        return false;
    }
    if (width > MAX_WINDOW_SIZE || height > MAX_WINDOW_SIZE) {
        TLOGE("Window size too large: %{public}u x %{public}u", width, height);
        return false;
    }
    // 防止乘法溢出
    if (width > UINT32_MAX / height) {
        TLOGE("Window size overflow");
        return false;
    }
    return true;
}
```

---

### 风险 5: 悬浮窗权限滥用

**证据位置**: `scene_session_manager.cpp:4825`

```cpp
// 悬浮窗创建权限检查
bool SceneSessionManager::CheckFloatWindowPermission(const WindowType& type) {
    if (type == WindowType::WINDOW_TYPE_FLOAT) {
        return VerifyCallingPermission("ohos.permission.SYSTEM_FLOAT_WINDOW");
    }
    return true;
}
```

**风险描述**:
- 悬浮窗可覆盖其他应用界面
- 可能被用于点击劫持、钓鱼攻击
- 权限检查仅基于权限字符串，缺乏额外校验

**触发路径**:
```
App: 申请 SYSTEM_FLOAT_WINDOW 权限
  → 创建悬浮窗覆盖银行应用
  → 伪造输入框窃取密码
```

**影响**: 中 - 可导致界面劫持、钓鱼攻击

**修复建议**:
```cpp
// 增加系统应用校验 + 用户确认
bool CheckFloatWindowPermission(const WindowType& type) {
    if (type != WindowType::WINDOW_TYPE_FLOAT) {
        return true;
    }
    
    // 1. 权限检查
    if (!VerifyCallingPermission("ohos.permission.SYSTEM_FLOAT_WINDOW")) {
        return false;
    }
    
    // 2. 系统应用或签名白名单
    if (!IsSystemCalling() && !IsInFloatWindowWhitelist()) {
        TLOGW("Non-system app trying to create float window");
        return false;
    }
    
    // 3. 限制悬浮窗大小和位置
    if (!ValidateFloatWindowBounds(option)) {
        return false;
    }
    
    return true;
}
```

---

## 安全建议汇总

### 1. 强化权限校验

- 对敏感操作增加多重校验（权限 + Token + PID）
- 定期检查权限授予情况，移除不必要的权限
- 增加权限使用审计日志

### 2. 加固 IPC 接口

- 所有 IPC 参数增加严格范围校验
- 使用安全的序列化/反序列化方式
- 增加 IPC 调用频率限制（防 DoS）

### 3. 窗口安全增强

- 窗口句柄增加 Token 绑定
- 定期回收未使用的窗口 ID
- 增加窗口操作审计日志

### 4. 隐私保护

- 截图、录屏操作增加用户确认
- 敏感窗口类型禁止截图
- 虚拟屏幕创建增加额外限制

### 5. 代码安全

- 启用完整的安全编译选项（CFI、ASAN 等）
- 定期进行模糊测试 (Fuzzing)
- 建立安全漏洞响应机制

## 检查局限性说明

本次安全分析基于静态代码审查，存在以下局限：

1. **未覆盖运行时漏洞**：如竞争条件、内存泄漏等
2. **未分析依赖库**：第三方库的安全风险未纳入评估
3. **未进行动态测试**：未通过 Fuzzing 或渗透测试验证
4. **代码覆盖不全**：仅分析了主要代码路径，边缘情况可能遗漏

**建议后续行动**：
- 进行动态安全测试
- 引入自动化安全扫描工具
- 建立定期安全审计机制

## 相关文档

- [N-API 参考](04_NAPI_Reference.md)
- [架构说明](02_Architecture.md)
- [项目概览](01_Overview.md)
