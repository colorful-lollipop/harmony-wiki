# 攻击面分析

**本文档识别和分析所有潜在攻击入口，供安全研究员快速审计。**

---

## 1. 系统边界与信任模型

### 1.1 边界划分

```
┌─────────────────────────────────────────────────────────────────────┐
│                        用户空间 (不可信)                              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              Cangjie 应用代码 (任意用户编写)                    │   │
│  │  ┌─────────────────────────────────────────────────────┐     │   │
│  │  │       notification_cangjie_wrapper (本模块)          │     │   │
│  │  │       ┌───────────────────────────────────────┐     │     │   │
│  │  │       │    FFI 边界 (Cangjie ↔ C++)          │     │     │   │
│  │  │       └───────────────────────────────────────┘     │     │   │
│  │  └─────────────────────────────────────────────────────┘     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              ↓ IPC                                 │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │            common_event_service (C++ 服务层)                  │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              ↓ IPC                                 │
└─────────────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────────────┐
│                        系统服务空间 (可信)                            │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │            CES (Common Event Service 系统服务)                │   │
│  │            - 权限实际校验点                                   │   │
│  │            - 事件存储与分发                                   │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2 信任级别

| 边界 | 组件 | 信任级别 | 攻击可能性 |
|------|------|----------|-----------|
| 边界 1 | Cangjie 应用 | 不可信 | 高 - 任意代码执行 |
| 边界 2 | FFI 包装层 | 半可信 | 中 - 需绕过参数校验 |
| 边界 3 | common_event_service | 可信 | 低 - 需利用 IPC 漏洞 |
| 边界 4 | CES | 核心可信 | 极低 - 系统内核级 |

---

## 2. 外部输入清单

### 2.1 公共 API 输入

| 输入点 | 类型 | 位置 | 风险等级 |
|--------|------|------|----------|
| `publish(event: String)` | 事件名称 | `common_event_manager.cj:57` | 🔴 高 |
| `publish(options.data)` | 事件数据 | `common_event_publish_data.cj:58` | 🔴 高 |
| `publish(options.parameters)` | HashMap 参数 | `common_event_publish_data.cj:91` | 🔴 高 |
| `publish(options.bundleName)` | 包名字符串 | `common_event_publish_data.cj:42` | 🟡 中 |
| `publish(options.subscriberPermissions)` | 权限数组 | `common_event_publish_data.cj:66` | 🟡 中 |
| `publish(options.isSticky)` | 布尔标记 | `common_event_publish_data.cj:83` | 🔴 高 |
| `createSubscriber(events)` | 事件列表 | `common_event_subscribe_info.cj:80` | 🟡 中 |
| `createSubscriber(publisherPermission)` | 权限字符串 | `common_event_subscribe_info.cj:88` | 🟢 低 |
| `createSubscriber(publisherBundleName)` | 包名字符串 | `common_event_subscribe_info.cj:125` | 🟢 低 |
| `createSubscriber(userId)` | 用户 ID | `common_event_subscribe_info.cj:106` | 🟡 中 |
| `createSubscriber(priority)` | 优先级 | `common_event_subscribe_info.cj:117` | 🟡 中 |
| `subscribe(callback)` | 用户回调 | `common_event_manager.cj:109` | 🟡 中 |

### 2.2 FFI 回调输入

| 输入点 | 类型 | 位置 | 风险等级 |
|--------|------|------|----------|
| `CCommonEventData` | 事件数据结构 | `common_event_data.cj:77` | 🟡 中 |
| `CParameters.valueType` | 类型标记 | `parameters.cj:44` | 🔴 高 |
| `CParameters.value` | 类型擦除指针 | `parameters.cj:46` | 🔴 高 |

---

## 3. 敏感操作清单

### 3.1 特权操作

| 操作 | 权限要求 | 位置 | 风险 |
|------|----------|------|------|
| 发布粘性事件 | `ohos.permission.COMMONEVENT_STICKY` | `common_event_publish_data.cj:83` | 仅注解校验 |
| 指定 userId | 系统权限 | `common_event_subscribe_info.cj:106` | 透传到底层 |
| 设置优先级 | 无 | `common_event_subscribe_info.cj:117` | 静默截断 |

### 3.2 资源操作

| 操作 | 资源类型 | 位置 | 风险 |
|------|----------|------|------|
| 字符串分配 | 堆内存 | `LibC.mallocCString()` | 无上限分配 |
| HashMap 转换 | 堆内存 | `createCArrParam()` | 大小无限制 |
| FFI 句柄创建 | 句柄表 | `FfiCommonEventManagerCreateSubscriber()` | 句柄泄漏 |
| 回调注册 | 回调表 | `CJ_Subscribe()` | 回调悬挂 |

### 3.3 跨边界操作

| 操作 | 源边界 | 目标边界 | 位置 |
|------|--------|----------|------|
| FFI 函数调用 | Cangjie | C++ | `*_ffi.cj` |
| 结构体序列化 | Cangjie 对象 | C 结构体 | `@C struct` |
| 回调触发 | C++ | Cangjie | `Callback1Param` |
| 数据反序列化 | C 结构体 | Cangjie 对象 | `init(c: CType)` |

---

## 4. 攻击向量分析

### 4.1 输入验证绕过

#### 向量 1: 超长字符串 DoS

**目标**: `publish(event)` 和 `publish(data)`

**攻击路径**:
```
攻击者构造 100MB 字符串 → publish() → mallocCString() → 
内存分配失败或系统卡顿
```

**证据**: `common_event_manager.cj:60`
```cangjie
cEvent = LibC.mallocCString(event).asResource()  // 无长度检查
```

**影响**: 内存耗尽，服务拒绝

**修复建议**: 添加长度限制（事件名 < 256, data < 64KB）

---

#### 向量 2: HashMap 参数轰炸

**目标**: `publish(options.parameters)`

**攻击路径**:
```
攻击者构造 10000+ 参数的 HashMap → createCArrParam() → 
内存耗尽或序列化超时
```

**证据**: `value_type/parameters.cj:132-148`
```cangjie
unsafe protected func createCArrParam(parameters: HashMap<String, CommonEventValueType>): CArrParameters {
    let cp = safeMalloc<CParameters>(count: parameters.size)  // 无大小限制！
```

**影响**: 内存 DoS，系统不稳定

**修复建议**: 限制参数数量（如最大 64 项）

---

#### 向量 3: 类型混淆攻击

**目标**: `CParameters.valueType` 反序列化

**攻击路径**:
```
恶意 native 层返回错误 valueType → 
CPointer 错误类型转换 → 内存访问违规或信息泄露
```

**证据**: `value_type/parameters.cj:44-58`
```cangjie
match {
    case c.valueType == INT_TYPE => Int32Value(CPointer<Int32>(c.value).read())
    case c.valueType == STRING_TYPE => StringValue(CString(CPointer<UInt8>(c.value)).toString())
    // ... 无 default 错误处理，直接回退到 ArrayFD
    case _ => ArrayFD(c.toArr<Int32>())  // 危险回退！
}
```

**影响**: 类型混淆，可能的信息泄露或崩溃

**修复建议**: 严格校验 valueType，非法值抛出异常

---

### 4.2 权限绕过

#### 向量 4: 粘性事件权限绕过

**目标**: `isSticky` 标记

**攻击路径**:
```
应用设置 isSticky = true → 本层仅文档校验 → 
传递到 common_event_service → 若底层校验有漏洞则发送成功
```

**证据**: `common_event_publish_data.cj:78-83`
```cangjie
@!APILevel[
    permission: "ohos.permission.COMMONEVENT_STICKY",  // 仅文档声明
]
public var isSticky: Bool
```

**影响**: 未授权发送粘性事件，信息泄露或 DoS

**修复建议**: 本层添加 AccessToken 预校验

---

#### 向量 5: FD 类型特权提升

**目标**: `CommonEventValueType.FD`

**攻击路径**:
```
应用获取敏感文件 FD → publish(parameters: { "fd": FD(42) }) → 
订阅者接收 FD → 越权访问文件
```

**证据**: `value_type/value_type.cj:68-73`
```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Notification.CommonEvent"
]
FD(Int32)  // 无任何权限限制！
```

**影响**: 文件描述符劫持，权限提升，信息泄露

**修复建议**: 禁止普通应用使用 FD 类型

---

### 4.3 资源耗尽

#### 向量 6: 订阅者泄漏

**目标**: `createSubscriber()` 和 `subscribe()`

**攻击路径**:
```
循环创建订阅者但不 unsubscribe → 
FFI 句柄表耗尽 → 1500010 错误（订阅者超限）
```

**证据**: `common_event_manager_errors.cj:33`
```cangjie
const ERROR_SUBSCRIBER_EXCEED_SPECIFICATION: Int32 = 1500010
```

**影响**: 订阅者 DoS

**修复建议**: 应用层资源管理，系统层限制单应用订阅数

---

## 5. 信任边界跨越点

### 5.1 Cangjie → C++ 边界

| 位置 | 机制 | 风险 |
|------|------|------|
| `common_event_manager_ffi.cj` | foreign 函数声明 | 参数类型不匹配 |
| `@C struct` | 内存布局 | 结构体字段偏移错误 |
| `mallocCString` | 字符串分配 | 内存泄漏 |

### 5.2 C++ → Cangjie 边界

| 位置 | 机制 | 风险 |
|------|------|------|
| `Callback1Param` | 回调注册 | 回调悬挂 |
| `CCommonEventData` | 数据反序列化 | 类型混淆 |
| `releaseFFIData` | 资源释放 | 双重释放 |

---

## 6. 攻击面汇总图

```
                    ┌──────────────────────────────────────┐
                    │         攻击者控制区域                │
                    │  ┌────────────────────────────────┐  │
                    │  │  恶意 Cangjie 应用              │  │
                    │  │  - 超长字符串输入               │  │
                    │  │  - 超大参数列表                 │  │
                    │  │  - 伪造 FD                      │  │
                    │  │  - 越界 priority                │  │
                    │  └────────────────────────────────┘  │
                    └──────────────────────────────────────┘
                                    ↓
                    ┌──────────────────────────────────────┐
                    │         FFI 包装层 (本模块)           │
                    │  ┌────────────────────────────────┐  │
                    │  │  ⚠️ 弱校验点：                  │  │
                    │  │  - 字符串长度无限制             │  │
                    │  │  - HashMap 大小无限制           │  │
                    │  │  - isSticky 仅文档校验          │  │
                    │  │  - FD 类型无限制                │  │
                    │  │  - priority 静默截断            │  │
                    │  └────────────────────────────────┘  │
                    └──────────────────────────────────────┘
                                    ↓
                    ┌──────────────────────────────────────┐
                    │         系统服务层                   │
                    │  ┌────────────────────────────────┐  │
                    │  │  common_event_service          │  │
                    │  │  - 最终权限校验                 │  │
                    │  │  - 事件存储/分发                │  │
                    │  └────────────────────────────────┘  │
                    └──────────────────────────────────────┘
```

---

## 7. 快速审计检查清单

### 7.1 新人审计（5 分钟）

- [ ] 检查 `isSticky` 是否有运行时权限校验
- [ ] 检查字符串输入是否有长度限制
- [ ] 检查 HashMap 参数是否有大小限制
- [ ] 检查 FD 类型是否有使用限制

### 7.2 深度审计（30 分钟）

- [ ] 审计所有 `foreign` 函数参数类型
- [ ] 检查所有 `@C struct` 内存布局
- [ ] 验证异常路径资源释放
- [ ] 检查回调异常处理
- [ ] 审计类型转换边界

### 7.3 完整审计（2 小时）

- [ ] 结合底层 common_event_service 审计
- [ ] 审计 IPC 通信安全
- [ ] 模糊测试所有输入点
- [ ] 审计权限模型一致性

---

## 8. 安全边界声明

### 8.1 本层安全职责

**负责**:
- ✅ 参数类型正确性
- ✅ 内存分配异常处理
- ✅ 错误码映射

**不负责**（依赖底层）:
- ❌ 权限实际校验
- ❌ 字符串内容过滤
- ❌ 用户 ID 有效性
- ❌ 速率限制

### 8.2 风险接受项

| 风险 | 原因 | 缓解措施 |
|------|------|----------|
| 权限绕过 | 本层仅桥接 | 依赖底层 common_event_service 校验 |
| 字符串注入 | 事件名透明传输 | 底层 CES 负责事件路由安全 |
| DoS | 资源限制在系统层 | 系统层限制单应用资源 |

---

**相关文档**:
- [安全风险评估 →](05_Security.md)
- [内部实现细节 →](08_Internals.md)
- [API 参考 →](03_API_Reference.md)
