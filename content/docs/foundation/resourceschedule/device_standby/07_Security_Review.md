# 安全风险评审

## 概述

本文档对 Device Standby 部件进行安全风险评估，涵盖攻击面、信任边界及潜在风险点。

## 评估范围

| 类别 | 状态 |
|------|------|
| N-API 接口 | ✅ 已评估 |
| Inner Kits 接口 | ✅ 已评估 |
| IPC/System Ability | ✅ 已评估 |
| 插件机制 | ✅ 已评估 |
| 配置文件 | ✅ 已评估 |
| 测试代码 | ❌ 不在范围内 |

## 攻击面清单

### 1. N-API 接口

| 攻击面 | 说明 | 风险等级 |
|--------|------|----------|
| `requestExemptionResource` | 应用申请豁免资源 | 高 |
| `releaseExemptionResource` | 应用释放豁免资源 | 高 |
| `getExemptedApps` | 获取豁免应用列表 | 中 |
| `isDeviceInStandby` | 查询待机状态 | 低 |

### 2. Inner Kits 接口

| 攻击面 | 说明 | 风险等级 |
|--------|------|----------|
| `ApplyAllowResource` | 申请豁免资源 | 高 |
| `UnapplyAllowResource` | 释放豁免资源 | 高 |
| `SubscribeStandbyCallback` | 订阅状态变化 | 中 |
| `ReportDeviceStateChanged` | 上报设备状态 | 中 |

### 3. IPC 接口

| 攻击面 | 说明 | 风险等级 |
|--------|------|----------|
| SA ID 1914 | 系统能力调用 | 高 |
| `IStandbyService` | 服务接口 | 高 |

### 4. 配置文件

| 攻击面 | 说明 | 风险等级 |
|--------|------|----------|
| `sa_profile/1914.json` | SA 配置 | 低 |
| Feature Flags | 编译开关 | 低 |

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                        设备                                  │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐    │
│  │              resource_schedule_service              │    │
│  │  ┌─────────────────────────────────────────────┐  │    │
│  │  │           StandbyService (SA 1914)         │  │    │
│  │  │  ┌─────────────────────────────────────┐   │  │    │
│  │  │  │           Plugin Layer              │   │  │    │
│  │  │  └─────────────────────────────────────┘   │  │    │
│  │  └─────────────────────────────────────────────┘  │    │
│  └─────────────────────────────────────────────────────┘    │
│                          ↑ IPC                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              其他系统服务                            │    │
│  └─────────────────────────────────────────────────────┘    │
│                          ↑ IPC                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │           应用进程 (HAP/Native)                     │    │
│  │  ┌──────────────┐  ┌──────────────────────┐       │    │
│  │  │   N-API      │  │      Taihe           │       │    │
│  │  └──────────────┘  └──────────────────────┘       │    │
│  └─────────────────────────────────────────────────────┘    │
│                          ↑ IPC                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              用户态 (无特权)                        │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## 风险评估详情

### 风险 1：权限绑定绕过

| 项目 | 内容 |
|------|------|
| **风险 ID** | SEC-001 |
| **风险名称** | 权限校验绑定不严导致滥用豁免 |
| **证据** | `standby_service_impl.cpp:627-641` |
| **触发条件** | 非授权应用调用 `requestExemptionResource` |
| **影响** | 绕过待机管控，增加功耗 |
| **风险等级** | 中 |
| **修复建议** | 增加 UID 与权限的绑定校验 |

**证据代码**：

```cpp
// standby_service_impl.cpp:627-641
Security::AccessToken::AccessTokenID tokenId = OHOS::IPCSkeleton::GetCallingTokenID();
if (Security::AccessToken::AccessTokenKit::GetTokenType(tokenId)
    == Security::AccessToken::ATokenTypeEnum::TOKEN_HAP) {
    if (Security::AccessToken::AccessTokenKit::VerifyAccessToken(callerToken, STANDBY_EXEMPTION_PERMISSION)
        != Security::AccessToken::PermissionState::PERMISSION_GRANTED) {
        return ERR_PERMISSION_ERROR;
    }
}
```

### 风险 2：UID 伪造

| 项目 | 内容 |
|------|------|
| **风险 ID** | SEC-002 |
| **风险名称** | ResourceRequest 中 UID 未校验 |
| **证据** | `standby_service_impl.cpp:759` |
| **触发条件** | 应用传入伪造的 UID |
| **影响** | 冒充其他应用申请豁免 |
| **风险等级** | 高 |
| **修复建议** | 对比请求 UID 与实际调用者 UID |

**证据代码**：

```cpp
// standby_service_impl.cpp:759
if (!CheckAllowTypeInfo(resourceRequest.GetAllowType()) || resourceRequest.GetUid() < 0) {
    STANDBYSERVICE_LOGE("resourceRequest param is invalid");
    return ERR_RESOURCE_TYPES_INVALID;
}
// 注意：只检查了 uid >= 0，没有校验 UID 是否与调用者匹配
```

### 风险 3：时长溢出

| 项目 | 内容 |
|------|------|
| **风险 ID** | SEC-003 |
| **风险名称** | duration 参数可能导致整数溢出 |
| **证据** | `standby_service_impl.cpp:763-765` |
| **触发条件** | 传入超大 duration 值 |
| **影响** | 内存溢出或计算错误 |
| **风险等级** | 中 |
| **修复建议** | 增加 duration 最大值校验 |

**证据代码**：

```cpp
// standby_service_impl.cpp:763-765
if (resourceRequest.GetDuration() < 0) {
    STANDBYSERVICE_LOGE("duration param is invalid");
    return ERR_DURATION_INVALID;
}
// 只检查了 < 0，没有检查最大值
```

### 风险 4：状态订阅者劫持

| 项目 | 内容 |
|------|------|
| **风险 ID** | SEC-004 |
| **风险名称** | SubscribeStandbyCallback 缺少权限校验 |
| **证据** | `services/core/include/standby_service_impl.h:111` |
| **触发条件** | 任意应用订阅待机状态 |
| **影响** | 敏感状态信息泄露 |
| **风险等级** | 中 |
| **修复建议** | 增加 DUMP 权限校验 |

### 风险 5：配置注入

| 项目 | 内容 |
|------|------|
| **风险 ID** | SEC-005 |
| **风险名称** | 配置文件路径遍历 |
| **证据** | `utils/policy/src/standby_config_manager.cpp` |
| **触发条件** | 恶意配置文件路径 |
| **影响** | 读取任意文件 |
| **风险等级** | 低 |
| **修复建议** | 校验配置文件路径 |

## 已有的安全机制

### 1. 权限校验

- **敏感权限**：`ohos.permission.DEVICE_STANDBY_EXEMPTION`
- **实现方式**：`AccessTokenKit::VerifyAccessToken()`

### 2. 参数校验

| 校验项 | 位置 |
|--------|------|
| UID 范围 | `standby_service_impl.cpp:759` |
| Duration 范围 | `standby_service_impl.cpp:763` |
| ResourceType 有效值 | `standby_service_impl.cpp:759` |

### 3. 进程隔离

- **SA 独立进程**：运行在 `resource_schedule_service`
- **IPC 通信**：通过 IPCSkeleton 校验调用者

### 4. CFI 保护

- **编译选项**：`sanitize: { cfi = true, cfi_cross_dso = true }`

### 5. 分支保护

- **编译选项**：`branch_protector_ret = "pac_ret"`

## 安全建议优先级

| 优先级 | 风险 ID | 建议 |
|--------|---------|------|
| P0 | SEC-002 | 增加 UID 绑定校验 |
| P1 | SEC-001 | 完善权限校验逻辑 |
| P1 | SEC-003 | 增加 duration 上限校验 |
| P2 | SEC-004 | 增加订阅权限校验 |
| P3 | SEC-005 | 配置文件路径校验 |

## 检查局限性

1. **未覆盖范围**：测试代码 (`test/`, `*_test.cpp`)
2. **依赖组件**：未深入分析 `access_token`, `ipc` 等依赖的安全性
3. **运行时**：未进行动态安全测试
4. **配置**：未分析 Feature Flags 的安全影响

## 参考资料

- [OpenHarmony 安全规范](https://gitee.com/openharmony/security)
- [访问控制框架](https://gitee.com/openharmony/docs/blob/master/zh-cn/security/AccessToken开发指导.md)
