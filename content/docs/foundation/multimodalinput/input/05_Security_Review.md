# 05_Security_Review - 安全风险评审

## 概述

本文档对 multimodalinput_input 子系统进行安全风险评审，识别潜在攻击面、信任边界和安全风险点。

## 代码证据

**核心安全文件**:
- `service/permission_helper/src/permission_helper.cpp` - 权限校验实现
- `util/common/include/error_multimodal.h` - 错误码定义
- `service/drag_security/src/drag_security_manager.cpp` - 拖拽安全

---

## 5.1 攻击面分析

### 5.1.1 输入攻击面

| 攻击面 | 说明 | 风险等级 |
|--------|------|----------|
| **N-API 接口** | injectEvent、setPointerStyle 等 | 高 |
| **IPC 通信** | 跨进程消息传递 | 中 |
| **设备文件** | /dev/input/* 设备节点 | 高 |
| **配置文件** | JSON/XML 配置文件解析 | 中 |
| **系统调用** | ioctl、read、write | 高 |
| **Socket 通信** | UDS 套接字通信 | 中 |

### 5.1.2 N-API 攻击面清单

| API | 攻击类型 | 风险等级 |
|-----|---------|----------|
| `injectEvent` | 事件注入伪造 | 高 |
| `injectKeyEvent` | 按键注入伪造 | 高 |
| `injectMouseEvent` | 鼠标注入伪造 | 高 |
| `injectTouchEvent` | 触摸注入伪造 | 高 |
| `injectJoystickEvent` | 手柄注入伪造 | 中 |
| `setPointerLocation` | 指针位置欺骗 | 中 |
| `setCustomCursor` | 光标劫持 | 中 |
| `transmitInfrared` | 红外信号伪造 | 中 |

---

## 5.2 信任边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            信任边界图                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────┐                           ┌─────────────────┐        │
│  │  硬件层          │                           │   内核层         │        │
│  │  (可信)         │                           │  (可信)         │        │
│  └────────┬────────┘                           └────────┬────────┘        │
│           │                                             │                  │
│           ▼                                             ▼                  │
│  ┌─────────────────┐    IPC/RPC    ┌─────────────────┐                   │
│  │  MMI Service    │◀──────────────▶│   客户端应用     │                   │
│  │  (半可信)       │                │  (半可信)       │                   │
│  └────────┬────────┘                └────────┬────────┘                   │
│           │                                   │                            │
│           ▼                                   ▼                            │
│  ┌─────────────────┐                 ┌─────────────────┐                   │
│  │  输入设备驱动    │                 │   JS/N-API     │                   │
│  │  (半可信)       │                 │  (不可信)      │                   │
│  └─────────────────┘                 └─────────────────┘                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.2.1 信任边界定义

| 区域 | 信任级别 | 说明 |
|------|----------|------|
| 硬件层 | 极高可信 | 物理设备输入 |
| 内核层 | 高可信 | 系统调用接口 |
| MMIService | 中可信 | 系统服务 |
| 客户端应用 | 低可信 | 用户空间进程 |
| N-API | 低可信 | 未经验证的输入 |

---

## 5.3 风险点识别

### 5.3.1 高风险项

#### R01: 事件注入权限绕过

**证据**: `permission_helper.cpp:67-69`

```cpp
// SHELL token 直接放行
if (isFromShell) {
    return true;  // 无权限检查直接放行
}
```

| 属性 | 说明 |
|------|------|
| **触发条件** | 调用方为 Shell 进程 (TOKEN_SHELL) |
| **影响** | Shell 权限可注入任意事件 |
| **风险等级** | 高 |
| **修复建议** | 对 Shell 注入也进行参数范围检查 |

**调用链**:
```
Shell 命令 → injectEvent API → PermissionHelper::CheckInjectPermission() → 绕过
```

---

#### R02: 非系统应用 API 调用

**证据**: `permission_helper.cpp:43-60`

```cpp
int32_t PermissionHelper::VerifySystemApp()
{
    AccessTokenID tokenId = IPCSkeleton::GetSelfTokenID();
    TokenType tokenType = PermissionHelper::GetTokenType(tokenId);
    if (tokenType != TOKEN_HAP) {
        return RET_OK;
    }
    // HAP 应用直接返回 ERROR_NOT_SYSAPI (202)
}
```

| 属性 | 说明 |
|------|------|
| **触发条件** | 非系统应用调用系统 API |
| **影响** | API 返回权限错误 |
| **风险等级** | 中 |
| **修复建议** | 正确配置权限声明 |

---

### 5.3.2 中风险项

#### R03: 配置解析安全

**证据**: `bundle_name_parser.cpp:55`

```cpp
// JSON 配置文件读取
nlohmann::json config;
std::ifstream configFile(configPath);
configFile >> config;  // JSON 解析
```

| 属性 | 说明 |
|------|------|
| **触发条件** | 配置文件被篡改 |
| **影响** | 可能的配置注入攻击 |
| **风险等级** | 中 |
| **修复建议** | 配置文件签名验证 |

---

#### R04: 路径遍历风险

**证据**: `util.cpp:324, 603`

```cpp
// realpath 规范化处理
char resolvedPath[PATH_MAX];
if (realpath(filePath.c_str(), resolvedPath) == nullptr) {
    return ERROR;  // 路径检查
}
```

| 属性 | 说明 |
|------|------|
| **触发条件** | 文件路径操作 |
| **影响** | 路径遍历攻击 |
| **风险等级** | 中 |
| **修复建议** | 使用 realpath 规范化 |

---

### 5.3.3 低风险项

#### R05: 拖拽安全签名

**证据**: `drag_security_manager.cpp:77-95`

```cpp
// 拖拽事件签名生成
uint64_t GenerateSignature(PointerEvent &event)
{
    // 签名算法用于防止事件篡改
}
```

| 属性 | 说明 |
|------|------|
| **功能** | 防止拖拽事件被中间人篡改 |
| **风险等级** | 低 |
| **评估** | 签名机制有效 |

---

## 5.4 权限矩阵

### 5.4.1 API 与权限对应

| API | 所需权限 | 检查位置 |
|-----|---------|---------|
| `injectEvent` | `ohos.permission.INJECT_INPUT_EVENT` | permission_helper.cpp:62 |
| `injectKeyEvent` | `ohos.permission.INJECT_INPUT_EVENT` | permission_helper.cpp:62 |
| `on('mouse')` | `ohos.permission.INPUT_MONITORING` | permission_helper.cpp:81 |
| `setShieldStatus` | `ohos.permission.INTERCEPT_INPUT_EVENT` | permission_helper.cpp:87 |
| `transmitInfrared` | `ohos.permission.MANAGE_INPUT_INFRARED_EMITTER` | permission_helper.cpp:93 |
| `setMousePrimaryButton` | `ohos.permission.MANAGE_MOUSE_CURSOR` | permission_helper.cpp:216 |
| `setInputDeviceEnabled` | `ohos.permission.INPUT_DEVICE_CONTROLLER` | permission_helper.cpp:228 |
| `setFunctionKeyEnabled` | `ohos.permission.INPUT_KEYBOARD_CONTROLLER` | permission_helper.cpp:234 |

### 5.4.2 权限检查流程

```
API 调用
    │
    ▼
┌─────────────────┐
│ 是否为系统应用?  │
└────────┬────────┘
         │ No
         ▼
┌─────────────────┐
│ 返回 ERROR_     │ → 拒绝访问
│ NOT_SYSAPI     │
└─────────────────┘
         │ Yes
         ▼
┌─────────────────┐
│ 获取 Token 类型  │
└────────┬────────┘
         │
         ├──▶ TOKEN_SHELL → 直接放行
         │
         ├──▶ TOKEN_NATIVE → 检查具体权限
         │
         └──▶ TOKEN_HAP → 检查具体权限
                   │
                   ▼
         ┌─────────────────┐
         │ 权限匹配检查     │
         └────────┬────────┘
                  │
                  ├──▶ 通过 → 执行 API
                  │
                  └──▶ 失败 → 返回权限错误
```

---

## 5.5 修复建议

### 5.5.1 高优先级修复

| ID | 问题 | 建议 | 难度 |
|----|------|------|------|
| R01 | Shell 权限直接放行 | 增加参数范围校验 | 中 |
| R02 | 权限声明缺失 | 完善 API 文档 | 低 |

### 5.5.2 中优先级修复

| ID | 问题 | 建议 | 难度 |
|----|------|------|------|
| R03 | 配置文件签名 | 增加 HMAC 验证 | 中 |
| R04 | 路径操作 | 严格路径白名单 | 低 |

### 5.5.3 低优先级修复

| ID | 问题 | 建议 | 难度 |
|----|------|------|------|
| R05 | 日志脱敏 | 敏感信息脱敏 | 低 |

---

## 5.6 安全检查清单

### 5.6.1 部署前检查

- [ ] 确认 `multimodalinput.cfg` 配置正确
- [ ] 验证 SA 配置文件权限
- [ ] 检查 N-API 模块签名

### 5.6.2 运行时监控

- [ ] 监控异常事件注入
- [ ] 监控权限检查失败日志
- [ ] 监控设备文件访问

### 5.6.3 安全加固建议

1. **最小权限原则**: 只授予应用必要权限
2. **输入验证**: 所有 N-API 参数严格校验
3. **沙箱隔离**: 限制应用访问设备节点
4. **日志审计**: 记录敏感操作

---

## 5.7 相关安全文档

- [OpenHarmony 应用签名指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/security/hapsigntool-guidelines.md)
- [AccessToken 权限管理](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/security/AccessToken/permissions-for-system-apps.md)
- [安全代码签名](https://github.com/openharmony/security_code_signature)

---

## 5.8 检查范围声明

### 5.8.1 已检查范围

- ✅ N-API 接口实现 (`frameworks/napi/`)
- ✅ 权限校验逻辑 (`service/permission_helper/`)
- ✅ IPC 通信机制 (`intention/ipc/`)
- ✅ 配置文件解析 (`service/*/parser/`)

### 5.8.2 未检查范围

- ❌ 第三方依赖库 (libinput, libevdev)
- ❌ 内核驱动层
- ❌ 硬件实现
- ❌ 模糊测试用例 (`test/fuzztest/`)

---

## 5.9 安全相关常量

```cpp
// 权限码定义 (permission_helper.cpp)
const std::string INJECT_PERMISSION_CODE = "ohos.permission.INJECT_INPUT_EVENT";
const std::string MONITOR_PERMISSION_CODE = "ohos.permission.INPUT_MONITORING";
const std::string INTERCEPT_PERMISSION_CODE = "ohos.permission.INTERCEPT_INPUT_EVENT";
const std::string INFRAREDEMITTER_PERMISSION_CODE = "ohos.permission.MANAGE_INPUT_INFRARED_EMITTER";

// 错误码定义
constexpr int32_t ERROR_NOT_SYSAPI { 202 };
constexpr int32_t COMMON_PERMISSION_CHECK_ERROR { 201 };
constexpr int32_t INPUT_PERMISSION_DENIED { 201 };
```
