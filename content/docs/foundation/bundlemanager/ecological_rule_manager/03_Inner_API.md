# 03_Inner_API - Inner API 接口规范

## 概述

**本模块不提供 N-API（JS API）**，仅对系统服务开放 Inner API。调用方必须为系统服务（System Service），且需通过权限校验。

## 接口清单

| # | 接口定义 | 调用方 | 说明 |
|---|----------|--------|------|
| 1 | `QueryStartExperience(const Want &want, const CallerInfo &callerInfo, ExperienceRule &rule): int32_t` | AbilityManagerService | 元服务打开时调用 |
| 2 | `QueryFreeInstallExperience(const Want &want, const CallerInfo &callerInfo, ExperienceRule &rule): int32_t` | BundleManagerService | 元服务免安装时调用 |
| 3 | `IsSupportPublishForm(const vector<Want> &wants, const CallerInfo &callerInfo, bool &bSupport): int32_t` | FormManagerService | 卡片加桌时调用 |
| 4 | `EvaluateResolveInfos(const Want &want, const CallerInfo &callerInfo, int32_t type, vector<AbilityInfo> &abilityInfos): int32_t` | AbilityManagerService | 过滤禁止提供者 |

## 错误码

```cpp
enum ErrCode {
    ERR_BASE = (-99),          // 基础错误码偏移
    ERR_FAILED = (-1),         // 通用失败
    ERR_PERMISSION_DENIED = (-2), // 权限校验失败
    ERR_OK = 0,               // 成功
};
```

**参考文件**: `interfaces/innerkits/include/ecological_rule_mgr_service_interface.h:50-55`

## 接口 1: QueryStartExperience

### 函数签名

```cpp
virtual int32_t QueryStartExperience(
    const Want &want,              // 目标 Want
    const CallerInfo &callerInfo, // 调用方信息
    ExperienceRule &rule          // 输出：体验规则
) = 0;
```

### 参数说明

| 参数 | 类型 | 说明 | 校验点 |
|------|------|------|--------|
| want | const Want & | 目标 Want | ReadParcelable 校验 |
| callerInfo | const CallerInfo & | 调用方信息 | ReadParcelable 校验 |
| rule | ExperienceRule & | 输出体验规则 | 调用方需初始化 |

### 返回值

| 值 | 说明 |
|---|------|
| ERR_OK (0) | 成功，rule 包含体验规则 |
| ERR_FAILED | 参数读取失败 |
| ERR_PERMISSION_DENIED | 调用方非系统应用 |

### 体验规则 (ExperienceRule)

| 字段 | 类型 | 说明 |
|------|------|------|
| isAllow | bool | 是否允许跳转 |
| resultCode | int32_t | 返回码（AMS 用） |
| replaceWant | sptr\<Want\> | 替代 Want |

**注意**: 返回 `replaceWant != nullptr` 时，应跳转至该替代 Want。

## 接口 2: QueryFreeInstallExperience

### 函数签名

```cpp
virtual int32_t QueryFreeInstallExperience(
    const Want &want,
    const CallerInfo &callerInfo,
    ExperienceRule &rule
) = 0;
```

### 参数说明

同 QueryStartExperience，应用于免安装场景。

### 返回值

| 值 | 说明 |
|---|------|
| ERR_OK (0) | 成功 |
| ERR_FAILED | 参数读取失败 |
| ERR_PERMISSION_DENIED | 调用方非系统应用 |

## 接口 3: IsSupportPublishForm

### 函数签名

```cpp
virtual int32_t IsSupportPublishForm(
    const std::vector<Want> &wants,   // Want 列表
    const CallerInfo &callerInfo,     // 调用方信息
    bool &bSupport                   // 输出：是否支持加桌
) = 0;
```

### 参数说明

| 参数 | 类型 | 说明 | 限制 |
|------|------|------|------|
| wants | const vector\<Want\> & | Want 列表 | 最多 15 个 |
| callerInfo | const CallerInfo & | 调用方信息 | - |
| bSupport | bool & | 输出结果 | - |

### 限制

- `MAX_WANT_SIZE`: 15（`services/manager/include/ecological_rule_mgr_service_stub.h:74`）
- 超过限制返回 `ERR_FAILED`

## 接口 4: EvaluateResolveInfos

### 函数签名

```cpp
virtual int32_t EvaluateResolveInfos(
    const Want &want,
    const CallerInfo &callerInfo,
    int32_t type,
    std::vector<AbilityInfo> &abilityInfos
) = 0;
```

### 参数说明

| 参数 | 类型 | 说明 | 限制 |
|------|------|------|------|
| want | const Want & | 目标 Want | - |
| callerInfo | const CallerInfo & | 调用方信息 | - |
| type | int32_t | 过滤类型 | - |
| abilityInfos | vector\<AbilityInfo\> & | 输入/输出能力列表 | 最多 1000 个 |

### 限制

- `MAX_ABILITY_INFO_SIZE`: 1000（`services/manager/include/ecological_rule_mgr_service_stub.h:75`）
- 超过限制返回 `ERR_FAILED`

## 调用方式

### 1. 获取 Client 实例

```cpp
#include "ecological_rule_mgr_service_client.h"

using namespace OHOS::EcologicalRuleMgrService;

auto client = EcologicalRuleMgrServiceClient::GetInstance();
```

### 2. 调用接口

```cpp
Want want;
CallerInfo callerInfo;
ExperienceRule rule;

// 设置 want 和 callerInfo ...

int32_t ret = client->QueryStartExperience(want, callerInfo, rule);
if (ret != ERR_OK) {
    // 处理错误
}
```

## 权限校验

### 调用前检查

1. **空 packageName 检查**: Client 端会对空包名做快速返回处理
2. **系统应用校验**: SA Stub 端 `VerifySystemApp()` 强制校验

### VerifySystemApp() 校验逻辑

```cpp
// 文件: services/manager/src/ecologic_rule_mgr_service_stub.cpp:233-257

bool EcologicalRuleMgrServiceStub::VerifySystemApp()
{
    // 1. 获取调用方 Token
    auto callerToken = IPCSkeleton::GetCallingTokenID();
    auto tokenType = AccessTokenKit::GetTokenTypeFlag(callerToken);
    
    // 2. Native/Shell Token 直接放行
    if (tokenType == TOKEN_NATIVE || tokenType == TOKEN_SHELL) {
        return true;
    }
    
    // 3. ROOT_UID (0) 直接放行
    auto callingUid = IPCSkeleton::GetCallingUid();
    if (callingUid == 0) {
        return true;
    }
    
    // 4. Foundation 进程 (UID=5523) 放行
    if (callingUid == ERMS_FOUNDATION_UID) {
        return true;
    }
    
    // 5. 其他情况检查 fullTokenID 是否为系统应用
    auto fullTokenId = IPCSkeleton::GetCallingFullTokenID();
    return TokenIdKit::IsSystemAppByFullTokenID(fullTokenId);
}
```

## 相关文档

- 架构设计: [02_Architecture.md](02_Architecture.md)
- 安全评审: [05_Security_Review.md](05_Security_Review.md)
