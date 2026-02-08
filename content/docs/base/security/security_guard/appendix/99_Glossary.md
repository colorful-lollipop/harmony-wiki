# SecurityGuard 术语表

**文档版本**: 3.1.0  
**最后更新**: 2026-02-06

---

## A

### ACL (Access Control List)
访问控制列表。定义哪些用户或系统可以访问特定资源的机制。

**相关配置**: `sa_profile/*.cfg` 中的 permission 字段

**示例**:
```json
"permission": [
  "ohos.permission.COLLECT_SECURITY_EVENT"
]
```

### APL (Ability Privilege Level)
能力权限等级。OpenHarmony 中应用的权限等级。

**等级**:
| 等级 | 说明 |
|------|------|
| normal | 普通级别 |
| system_basic | 系统基础级别 |
| system_core | 系统核心级别 |

### API Support Error
API 不支持的错误码 (801)。

**错误码**: `JS_ERR_API_SUPPORT_ERROR`  
**触发场景**: 系统版本不支持该 API

---

## B

### Bundle Name
应用包名。唯一标识应用的字符串。

**格式**: `com.company.appname`

**代码位置**: `security_collector_manager_service.cpp:368`

### Business Error
业务错误。在 API 调用中返回的错误对象。

**结构**:
```typescript
interface BusinessError {
    code: number;      // 错误码
    message: string;    // 错误消息
}
```

---

## C

### Callback
回调函数。异步操作完成后执行的函数。

**使用场景**:
- `querySecurityEvent` 的 querier 参数
- `on`/`off` 订阅的事件回调

**示例**:
```javascript
securityGuard.querySecurityEvent(rules, {
    onQuery: (events) => { /* 处理事件 */ },
    onComplete: () => { /* 查询完成 */ },
    onError: (error) => { /* 处理错误 */ }
});
```

### CFI (Control Flow Integrity)
控制流完整性。安全编译选项，用于防止控制流劫持攻击。

**代码位置**: 各模块 `BUILD.gn` 的 sanitize 配置

### Content Max Length
事件内容最大长度。限制上报事件内容的字节数。

**值**: `CONTENT_MAX_LEN = 10240`

**代码位置**: `security_guard_api.h`

---

## D

### DataCollectManager
数据收集管理器。SA 3524，负责安全事件的数据收集和查询。

**SA ID**: 3524  
**进程**: security_guard  
**功能**: 事件上报、查询、订阅管理

**代码位置**: `services/data_collect/sa/include/data_collect_manager_service.h`

### dlopen
动态库加载函数。用于加载共享库。

**安全风险点**: `services/risk_classify/plugin_manager/src/detect_plugin_manager.cpp:60`

**防护措施**: 路径白名单验证

---

## E

### Event ID
事件唯一标识符。

**格式**: 10位或13位数字

**标准事件**:
| ID | 名称 |
|-----|------|
| 1011015000 | PASTEBOARD |
| 1011015001 | ACCOUNT |
| 1011015005 | FILE |
| 1011015006 | PROCESS |
| 1011015007 | NETWORK |

### Event Group
事件组。一组相关事件的集合。

**配置文件**: `security_guard_event_group.json`

**结构**:
```json
{
    "eventGroupName": "securityGroup",
    "eventList": ["0x10000100", "0x12003000"]
}
```

### Extra Max Length
额外参数最大长度。限制 param 等额外参数的字节数。

**值**: `EXTRA_MAX_LEN = 2000`

---

## F

### FFRT (Fast Function Runtime)
快速函数运行时。OpenHarmony 的异步任务调度框架。

**使用场景**:
- 异步任务提交
- 串行队列 (Serial Queue)
- 互斥锁 (Mutex)

**代码位置**: `services/data_collect/sa/acquire_data_subscribe_manager.h`

### Feature Flags
功能开关。控制功能是否启用的编译选项。

**定义文件**: `security_guard.gni`

**示例**:
```gn
security_guard_enable = true
security_guard_trim_model_analysis = false
```

---

## G

### GetModelResult
获取模型结果。调用安全模型进行风险检测。

**返回值类型**: `Promise<SecurityModelResult>`

**模型列表**:
| 模型名 | ID |
|--------|-----|
| SecurityGuard_JailbreakCheck | 3001000000 |
| SecurityGuard_IntegrityCheck | 3001000001 |
| SecurityGuard_SimulatorCheck | 3001000002 |

---

## H

### HiLog
OpenHarmony 日志系统。

**日志域**: `0xD002F07`

**日志标签**:
| 标签 | 服务 |
|------|------|
| SG_Service | SecurityGuard 服务层 |
| S_COLLCTOR | SecurityCollector 采集器层 |

**查看命令**:
```bash
hilog -t SG_Service -l error
```

### Hisysevent
OpenHarmony 系统事件上报服务。

**配置文件**: `hisysevent.yaml`

**使用示例**:
```cpp
SGLOGI("Risk analysis completed");
```

---

## I

### IDL (Interface Definition Language)
接口定义语言。用于定义 IPC 接口的元语言。

**生成文件**: `services/*/idl/*.idl`

**示例**:
```idl
interface DataCollectManagerIdl {
    void RequestDataSubmit([in] long eventId, [in] String content);
}
```

### Inner API
内部 API。平台 SDK 级别的接口，需要特定标签才能使用。

**标签**: `platformsdk`, `sasdk`

**代码位置**:
- `interfaces/inner_api/collect/include/`
- `interfaces/inner_api/classify/include/`
- `interfaces/inner_api/collector/include/`

### IPC (Inter-Process Communication)
进程间通信。

**使用框架**: OpenHarmony Binder IPC

**核心组件**:
- Proxy (客户端代理)
- Stub (服务端存根)
- MessageParcel (消息序列化)

---

## J

### JS_ERR_BAD_PARAM
参数错误。JS 层错误码 401。

**触发条件**:
- 参数类型不匹配
- 参数值超出范围
- 缺少必需参数

### JSON Err
JSON 解析错误。内部错误码 7。

**触发场景**: 解析用户输入的 JSON 内容失败

---

## M

### Model ID
安全模型唯一标识符。

**命名规则**: `SecurityGuard_<FunctionName>`

**支持的模型**:
| ID | 模型名 | 功能 |
|-----|--------|------|
| 3001000000 | JailbreakCheck | 越狱检测 |
| 3001000001 | IntegrityCheck | 完整性检测 |
| 3001000002 | SimulatorCheck | 模拟器检测 |
| 3001000009 | RiskFactorCheck | 风险因子检测 |
| 3001000011 | WifiCheck | WiFi 风险检测 |

### Module Name
JS 模块名称。

**值**: `security.securityGuard`

**注册位置**: `security_guard_napi.cpp:1374`

---

## N

### N-API
Node.js API。OpenHarmony 提供的 JavaScript/TypeScript 接口层。

**文档位置**: `frameworks/js/napi/`

**注册宏**: `DECLARE_NAPI_FUNCTION`

### NO_PERMISSION
权限不足。内部错误码 2，JS 层映射为 201。

**解决方案**:
1. 在 `module.json5` 中声明权限
2. 动态申请权限
3. 检查 APL 等级

---

## O

### OEM Property
厂商配置。设备制造商自定义的配置项。

**配置文件**:
- `oem_property/hos/security_guard_event.json`
- `oem_property/hos/security_guard_model.cfg`

---

## P

### Parcel
消息序列化包。IPC 通信中用于序列化和反序列化的数据结构。

**常用方法**:
| 方法 | 功能 |
|------|------|
| WriteInt64() | 写入64位整数 |
| WriteString() | 写入字符串 |
| ReadRemoteObject() | 读取远程对象 |

### Promise
Promise。JavaScript 异步编程的标准化 API。

**使用 API**:
- `getModelResult()`
- `updatePolicyFile()`

### Process Name
进程名称。

**SecurityGuard 进程**:
| 进程 | 服务 |
|------|------|
| security_guard | DataCollectManager, RiskAnalysisManager |
| security_collector | SecurityCollectorManager |

---

## Q

### QuerySecurityEvent
查询安全事件。异步回调模式 API。

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| rulers | SecurityEventRuler[] | 查询条件 |
| querier | QuerierCallback | 回调对象 |

### QuerierCallback
查询回调接口。

**结构**:
```typescript
interface QuerierCallback {
    onQuery: (events: SecurityEvent[]) => void;
    onComplete: () => void;
    onError: (error: BusinessError) => void;
}
```

---

## R

### ReportSecurityEvent
上报安全事件。同步 API。

**权限**: `ohos.permission.COLLECT_SECURITY_EVENT`

**参数**:
| 字段 | 类型 | 说明 |
|------|------|------|
| eventId | number | 事件 ID |
| version | string | 版本号 (< 50 字符) |
| content | string | 事件内容 (< 10240 字符) |

---

## S

### SA (System Ability)
系统能力。后台运行的基础能力服务。

**SecurityGuard SA**:
| ID | 名称 | 进程 |
|-----|------|------|
| 3523 | RiskAnalysisManager | security_guard |
| 3524 | DataCollectManager | security_guard |
| 3525 | SecurityCollectorManager | security_collector |

### SA Profile
SA 配置文件。定义 SA 的加载参数和权限。

**配置文件**:
- `sa_profile/3523.json`
- `sa_profile/3524.json`
- `sa_profile/3525.json`

### SecurityCollector
安全采集器。收集特定类型安全事件的组件。

**配置文件**: `security_audit.cfg`

### SecurityEvent
安全事件。系统安全相关的事件数据。

**结构**:
```typescript
interface SecurityEvent {
    eventId: number;
    version: string;
    content: string;
    timestamp: string;
}
```

### SGLOGD/SGLOGI/SGLOGW/SGLOGE
SecurityGuard 日志宏。

**级别**:
| 宏 | 级别 | 使用场景 |
|-----|------|----------|
| SGLOGD | DEBUG | 调试信息 |
| SGLOGI | INFO | 关键流程 |
| SGLOGW | WARN | 警告信息 |
| SGLOGE | ERROR | 错误信息 |

### Subscribe
订阅安全事件。

**事件类型**: `securityEventOccur`

**代码位置**: `security_guard_napi.cpp:1254`

---

## T

### Time Max Length
时间字符串最大长度。

**值**: `TIME_MAX_LEN = 15`

**格式**: `YYYYMMDDHHMMSS` (14位数字)

### Token Bucket
令牌桶。限流算法的一种实现。

**代码位置**: `data_collect_manager_service.cpp`

**参数**:
| 参数 | 说明 |
|------|------|
| TOKEN_BUCKET_MAX_SIZE | 桶最大容量 |
| TOKEN_BUCKET_STEP_SIZE | 每次添加数量 |
| TOKEN_BUCKET_INTERVAL_TIME | 添加间隔 |

### TOCTOU
Time-of-check to time-of-use。检查-使用时间差漏洞。

**风险场景**: 文件大小检查与读取之间的时间差

**防护措施**: 使用文件锁定

---

## U

### Unsubscribe
取消订阅。停止接收安全事件通知。

**代码位置**: `security_guard_napi.cpp:1346`

---

## V

### Version Max Length
版本字符串最大长度。

**值**: `VERSION_MAX_LEN = 50`

---

## W

### WiFi Risk Detection
WiFi 风险检测。模型 ID 3001000011。

**功能**: 检测设备连接的 WiFi 是否安全

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [API 参考](../../02_NAPI_Reference.md) | 完整 API 文档 |
| [架构详解](../../03_Architecture.md) | 系统架构 |
| [调试指南](../../08_Debugging.md) | 日志与排错 |
| [错误码速查](./99_ErrorCodeQuickRef.md) | 错误码索引 |
