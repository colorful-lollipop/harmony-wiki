# 04_Security_Review - 安全风险评审

## 1. 评审概述

### 1.1 评审范围

本文档对 `screenlock_mgr` 子系统进行安全风险评审，覆盖以下代码范围:

| 目录 | 包含 | 排除 |
|------|------|------|
| `services/` | 核心服务实现 | 测试代码 |
| `frameworks/` | 接口层实现 | 测试代码 |
| `interfaces/` | API 接口定义 | - |
| `sa_profile/` | SA 配置文件 | - |

### 1.2 评审方法

- 代码静态分析
- 威胁建模
- 权限模型分析
- 输入验证检查

---

## 2. 威胁模型

### 2.1 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  应用进程 (不可信)                                          │ │
│  │  - 第三方 JS/ETS 应用                                       │ │
│  │  - 通过 N-API 调用进入                                      │ │
│  │  - 需要权限校验                                             │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│                   N-API 调用 (参数验证)                         │
│                              │                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  Native 进程边界 (可信)                                      │ │
│  │  - IPC 代理层                                               │ │
│  │  - 参数序列化                                               │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│                        Binder IPC                               │
│                              │                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  Foundation 进程 (可信)                                      │ │
│  │  - ScreenLockSystemAbility (SA)                             │ │
│  │  - 核心业务逻辑                                              │ │
│  │  - 状态管理                                                 │ │
│  │  - 事件发布                                                 │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 数据流

| 流向 | 数据 | 风险等级 |
|------|------|----------|
| 应用 → SA | API 参数 (userId, authState 等) | 高 |
| SA → 应用 | 状态查询结果 | 低 |
| SA → 外部 | 认证令牌、用户数据 | 高 |

---

## 3. 攻击面分析

### 3.1 攻击面清单

| 攻击面 | 类型 | 说明 |
|--------|------|------|
| N-API 接口 | 输入点 | JS 参数传入 |
| IPC 通信 | 输入点 | Binder 参数序列化 |
| 文件操作 | 输入/输出 | 偏好设置存储 |
| 事件订阅 | 输出点 | 系统事件发布 |
| 用户认证 | 信任边界 | 认证令牌验证 |

### 3.2 N-API 暴露的接口

| API | 输入参数 | 风险 |
|-----|----------|------|
| `lock()` | callback | 中 |
| `unlockScreen()` | callback | 中 |
| `setScreenLockDisabled(disable, userId)` | boolean, number | 高 |
| `setScreenLockAuthState(authState, userId, authToken)` | number, number, string | 高 |
| `sendScreenLockEvent(event, param)` | string, number | 高 |

---

## 4. 安全风险点

### 4.1 权限绕过风险

**风险 ID**: SEC-001

**描述**: 部分接口需要系统应用权限，但实现中可能存在绕过

**证据**:
```cpp
// test/unittest/screenlock_service_test.cpp
// 显示权限检查使用 AccessTokenKit
bool ret = ScreenLockSystemAbility::GetInstance()->CheckPermission("ohos.permission.ACCESS_SCREEN_LOCK_INNER");
```

**触发条件**:
1. 调用 `setScreenLockDisabled` 等敏感 API
2. 未正确检查调用者权限

**影响**:
- 权限提升攻击
- 未经授权修改锁屏状态

**修复建议**:
- 确保所有敏感 API 入口都有权限检查
- 使用 AccessTokenKit 进行权限验证
- 记录权限拒绝的审计日志

**当前状态**: ⚠️ 需要确认代码实现 (测试代码中存在引用，业务代码中需验证)

---

### 4.2 参数校验不完整

**风险 ID**: SEC-002

**描述**: 用户 ID 和认证状态参数范围验证不严格

**证据**:
```cpp
// interfaces/inner_api/include/screenlock_common.h
// 定义了特殊用户 ID 枚举
enum class SpecialUserId : int32_t {
    USER_ALL = -1,
    USER_CURRENT = -2,
    USER_UNDEFINED = -10000,
};
```

**触发条件**:
1. 传入异常的用户 ID 值
2. 认证状态值超出预期范围

**影响**:
- 整数溢出/下溢
- 状态管理混乱

**修复建议**:
- 在 N-API 层增加参数范围校验
- 使用白名单限制有效值
- 在服务层增加防御性检查

---

### 4.3 认证令牌处理

**风险 ID**: SEC-003

**描述**: `authToken` 参数在 IPC 传输和存储中的安全处理

**证据**:
```cpp
// interfaces/inner_api/include/screenlock_manager_interface.h:43
virtual int32_t SetScreenLockAuthState(int authState, int32_t userId, std::string &authToken) = 0;
```

**触发条件**:
1. 传入过长的 authToken 字符串
2. 认证令牌明文传输

**影响**:
- 缓冲区溢出
- 令牌泄露

**修复建议**:
- 对 authToken 长度限制 (MAX_VALUE_LEN = 4096)
- 考虑使用安全的令牌传输机制
- 日志中避免打印认证令牌

---

### 4.4 事件注入风险

**风险 ID**: SEC-004

**描述**: `sendScreenLockEvent` API 可能被恶意调用注入事件

**证据**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:77
DECLARE_NAPI_FUNCTION("sendScreenLockEvent", OHOS::ScreenLock::NAPI_ScreenLockSendEvent)
```

**触发条件**:
1. 应用调用 `sendScreenLockEvent`
2. 传入非预期的事件类型或参数

**影响**:
- 系统状态异常
- 事件处理逻辑混乱

**修复建议**:
- 白名单限制允许的事件类型
- 验证事件参数的合法性
- 仅允许系统应用调用

---

### 4.5 竞态条件

**风险 ID**: SEC-005

**描述**: 锁屏状态查询和修改之间可能存在竞态

**证据**:
```cpp
// services/src/screenlock_system_ability.cpp
// 使用互斥锁保护
std::mutex ScreenLockSystemAbility::instanceLock_;
std::mutex ScreenLockSystemAbility::queueLock_;
```

**触发条件**:
1. 并发调用锁屏相关 API
2. 状态查询和修改同时进行

**影响**:
- 状态不一致
- 潜在的崩溃

**修复建议**:
- 使用原子操作保护关键状态
- 考虑使用读写锁优化并发性能
- 增加并发测试覆盖

---

### 4.6 信息泄露

**风险 ID**: SEC-006

**描述**: 错误信息可能泄露系统内部状态

**证据**:
```cpp
// frameworks/js/napi/src/napi_screenlock_ability.cpp:38-44
constexpr const char *PERMISSION_VALIDATION_FAILED = "Permission verification failed.";
constexpr const char *NON_SYSTEM_APP = "Permission verification failed, application which is not a system application uses system API.";
```

**触发条件**:
1. API 调用失败
2. 返回详细错误信息

**影响**:
- 系统信息泄露
- 攻击面扩大

**修复建议**:
- 对外返回通用错误码
- 内部日志记录详细信息
- 避免在错误消息中暴露敏感信息

---

### 4.7 缓冲区安全

**风险 ID**: SEC-007

**描述**: IPC 参数序列化/反序列化可能存在缓冲区问题

**证据**:
```cpp
// interfaces/inner_api/include/screenlock_common.h:145
constexpr std::int32_t MAX_VALUE_LEN = 4096;
```

**触发条件**:
1. IPC 调用传输大数据
2. 字符串操作超出边界

**影响**:
- 缓冲区溢出
- 远程代码执行

**修复建议**:
- 严格限制传输数据大小
- 使用安全字符串函数
- 启用 CFI 保护 (已启用)

---

## 5. 已启用的安全措施

### 5.1 编译时保护

| 保护措施 | 状态 | 说明 |
|----------|------|------|
| CFI | ✅ 启用 | 控制流完整性 |
| PAC_RET | ✅ 启用 | 指针认证 |
| 符号隐藏 | ✅ 启用 | -fvisibility=hidden |
| 代码裁剪 | ⏸ 可选 | screenlock_mgr_so_crop |

### 5.2 运行时保护

| 保护措施 | 状态 | 说明 |
|----------|------|------|
| 权限检查 | ✅ 存在 | AccessTokenKit |
| 输入校验 | ⚠️ 部分 | 需要增强 |
| 日志脱敏 | ⚠️ 部分 | 需确认 |

---

## 6. 安全建议优先级

| 优先级 | 风险 ID | 建议 | 难度 |
|--------|---------|------|------|
| P0 | SEC-001 | 权限检查审计 | 中 |
| P1 | SEC-002 | 参数校验增强 | 低 |
| P1 | SEC-004 | 事件白名单 | 中 |
| P2 | SEC-003 | 令牌安全 | 中 |
| P2 | SEC-005 | 竞态测试 | 低 |
| P3 | SEC-006 | 错误信息脱敏 | 低 |
| P3 | SEC-007 | 边界检查 | 低 |

---

## 7. 测试建议

### 7.1 安全测试用例

| 用例 | 说明 |
|------|------|
| 权限绕过测试 | 非系统应用调用敏感 API |
| 参数越界测试 | 传入异常值 |
| 并发测试 | 多线程并发调用 |
| 模糊测试 | 随机参数生成 |

### 7.2 工具建议

| 工具 | 用途 |
|------|------|
| libfuzzer | 模糊测试 |
| AddressSanitizer | 内存安全 |
| ThreadSanitizer | 竞态检测 |

---

## 8. 相关文档

| 文档 | 说明 |
|------|------|
| [00_Overview](00_Overview.md) | 项目概览 |
| [01_NAPI_Reference](01_NAPI_Reference.md) | 接口参考 |
| [02_Architecture](02_Architecture.md) | 系统架构 |
| [03_GN_Build](03_GN_Build.md) | 构建系统 |
