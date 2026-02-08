# 安全风险评审

## 评审概述

本章节对 State Registry 模块进行安全风险评估，基于威胁建模方法论，分析攻击面、信任边界和潜在的安全漏洞。

**评审范围**：
- `frameworks/native/`：Native 框架代码
- `frameworks/js/napi/`：JS N-API 绑定代码
- `services/`：SA 服务实现
- `interfaces/`：API 接口定义

**评审方法**：代码审计 + 架构分析

## 攻击面分析

### 输入向量清单

| 输入源 | 类型 | 路由 | 风险等级 |
|--------|------|------|----------|
| JS API 参数 | 用户空间 | N-API → SA | 中 |
| IPC 调用参数 | 跨进程 | Binder | 中 |
| 配置文件 | 系统存储 | SA Profile | 低 |
| 网络状态数据 | 内核/Modem | core_service | 低 |
| 信号强度数据 | 内核/Modem | core_service | 低 |
| SIM 卡状态 | 硬件 | Modem | 低 |

### 暴露接口

| 接口位置 | 接口类型 | 调用者 | 鉴权要求 |
|----------|----------|--------|----------|
| @ohos.telephony.observer.on | JS API | 三方应用 | 按事件类型 |
| @ohos.telephony.observer.off | JS API | 三方应用 | 无 |
| ITelephonyStateRegistry | IPC | 系统组件 | 系统权限 |

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    第三方应用                           │    │
│  │  - 输入验证：✅ 需要                                     │    │
│  │  - 权限检查：✅ 需要                                     │    │
│  │  - 降级策略：拒绝访问                                    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              │                                  │
│                            N-API                                 │
│                              │                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   State Registry SA                     │    │
│  │  - 输入验证：✅ 需要                                     │    │
│  │  - 权限检查：✅ 需要                                     │    │
│  │  - 隔离性：进程隔离                                      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              │                                  │
│                         IPC/Binder                              │
│                              │                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   Core Service SA                        │    │
│  │  - 输入验证：✅ 已实现                                   │    │
│  │  - 权限检查：✅ 已实现                                   │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              │                                  │
│                          Modem                                   │
└─────────────────────────────────────────────────────────────────┘
```

## 风险分析与修复建议

### 风险 1：N-API 参数校验不完整 ⚠️

**证据位置**：`frameworks/js/napi/observer/napi_observer_utils.cpp`

**问题描述**：
在 N-API 参数解析过程中，对部分输入字段的长度校验不严格，可能导致缓冲区溢出或拒绝服务。

**触发条件**：
```typescript
// 构造超长参数
observer.on('callStateChange', { slotId: 0 }, (err, data) => {});
// 其中 phoneNumber 字段可能超过预期长度
```

**潜在影响**：
- 拒绝服务（DoS）
- 内存损坏（低风险，因有边界检查）

**风险等级**：中

**修复建议**：
```cpp
// 在 napi_observer_utils.cpp 中添加严格的长度校验
static constexpr size_t MAX_PHONE_NUMBER_LEN = 32;
static constexpr size_t MAX_SLOT_ID = 2;

bool ValidatePhoneNumber(const std::string& number) {
    if (number.length() > MAX_PHONE_NUMBER_LEN) {
        TELEPHONY_LOGE("Phone number too long: %{public}zu", number.length());
        return false;
    }
    return true;
}
```

**当前缓解措施**：
- C++ 编译启用 `-D_FORTIFY_SOURCE=2`
-  sanitizer 启用 CFI 检查

### 风险 2：权限声明不完整 ⚠️

**证据位置**：`bundle.json` 中 `component.syscap` 声明

**问题描述**：
部分敏感事件的权限要求在 JS API 文档中有说明，但 `module.json5` 中的权限声明可能不完整，导致开发者遗漏权限申请。

**风险事件类型**：
- `callStateChange` - 需要 `ohos.permission.READ_CALL_LOG`
- `cellInfoChange` - 需要 `ohos.permission.LOCATION`

**潜在影响**：
- 应用未声明权限时调用失败
- 用户对权限用途不清晰

**风险等级**：低

**修复建议**：
```json5
// module.json5 中完善权限声明
"requestPermissions": [
  {
    "name": "ohos.permission.READ_CALL_LOG",
    "reason": "需要读取通话状态以通知应用",
    "usedScene": {
      "abilities": ["EntryAbility"],
      "when": "inuse"
    }
  },
  {
    "name": "ohos.permission.LOCATION",
    "reason": "需要获取小区位置信息",
    "usedScene": {
      "abilities": ["EntryAbility"],
      "when": "inuse"
    }
  }
]
```

### 风险 3：IPC 回调对象生命周期管理

**证据位置**：`services/src/telephony_state_registry_stub.cpp`

**问题描述**：
IPC 回调对象（RemoteObject）的生命周期管理存在潜在竞态条件，如果客户端在回调触发前已销毁，可能导致悬空指针。

**触发场景**：
```cpp
// 客户端
let callback = observer.on('callStateChange', handler);
// ... 应用退出
observer.off('callStateChange');
```

**潜在影响**：
- 空指针解引用
- 进程崩溃

**风险等级**：低（IPC 框架有保护机制）

**当前缓解措施**：
- IPC 框架自动管理引用计数
- 回调注册时进行有效性检查

**修复建议**：
```cpp
// 在 RegisterObserver 中添加客户端验证
int32_t TelephonyStateRegistryStub::RegisterObserver(...) {
    // 验证 IPC 调用者身份
    auto callingUid = IPCSkeleton::GetCallingUid();
    if (!IsValidTelephonyCaller(callingUid)) {
        TELEPHONY_LOGE("Invalid caller uid: %{public}d", callingUid);
        return ERR_PERMISSION_DENIED;
    }
    
    // 验证回调对象有效性
    if (callback == nullptr) {
        TELEPHONY_LOGE("Invalid callback object");
        return ERR_NULL_OBJECT;
    }
    
    return registry_.AddObserver(observerId, type, callback);
}
```

### 风险 4：日志信息泄露

**证据位置**：`BUILD.gn` 中 `cflags_cc` 配置

**问题描述**：
调试日志可能泄露敏感信息（如电话号码、IMSI），在生产环境中应避免详细日志输出。

**当前配置**：
```gn
cflags_cc = [
    "-O2",
    "-D_FORTIFY_SOURCE=2",
]
```

**潜在影响**：
- 隐私数据泄露
- 安全事件信息暴露

**风险等级**：低（生产版本日志级别可控）

**缓解措施**：
- 日志标签：`TELEPHONY_LOG_TAG = "StateRegistry"`
- 日志域：`LOG_DOMAIN = 0xD001F07`

### 风险 5：竞态条件 - 观察者并发注销

**证据位置**：`services/src/telephony_state_registry_record.cpp`

**问题描述**：
在多线程环境下，观察者的并发注销可能导致链表遍历时的竞态条件。

**触发场景**：
```typescript
// 多个线程同时调用 off()
thread1: observer.off('callStateChange')
thread2: observer.off('callStateChange')
```

**潜在影响**：
- 数据竞争
- 内存损坏

**风险等级**：低（内部已加锁）

**当前缓解措施**：
- 服务层使用互斥锁保护观察者列表

## 缓解措施总结

| 风险 | 严重性 | 已缓解 | 缓解措施 |
|------|--------|--------|----------|
| N-API 参数校验 | 中 | ✅ | CFI + FORTIFY |
| 权限声明 | 低 | ✅ | 文档说明 |
| IPC 生命周期 | 低 | ✅ | 引用计数 |
| 日志泄露 | 低 | ✅ | 日志分级 |
| 竞态条件 | 低 | ✅ | 互斥锁 |

## 安全加固建议

1. **输入验证加强**：在 N-API 层添加所有字符串字段的长度校验
2. **敏感数据脱敏**：日志输出时对电话号码、IMSI 等敏感字段进行脱敏处理
3. **完整性校验**：对 SA Profile 配置文件进行签名校验
4. **最小权限原则**：严格限制 IPC 调用者的权限范围

## 相关文档

- [JS API](03_JS_API.md)
- [Native API](04_Native_API.md)
- [架构设计](02_Architecture.md)
- [故障排查](08_Troubleshooting.md)
