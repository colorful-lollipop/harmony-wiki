# 安全风险评审

## 威胁模型概述

### 系统边界

```
┌─────────────────────────────────────────────────────────────────┐
│                      用户空间 (User Space)                         │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              Cangjie 应用进程                              │    │
│  │  ┌─────────────────────────────────────────────────┐      │    │
│  │  │       notification_cangjie_wrapper              │      │    │
│  │  │       (本模块 - FFI 包装层)                      │      │    │
│  │  └─────────────────────────────────────────────────┘      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              ↓ IPC                              │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │            common_event_service (C++ 服务)               │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              ↓ IPC                              │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │            CES (Common Event Service)                   │    │
│  │            - 事件存储与分发                              │    │
│  │            - 权限校验                                   │    │
│  │            - 订阅者管理                                 │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### 信任边界

| 边界 | 组件 | 信任级别 | 说明 |
|------|------|---------|------|
| 边界 1 | Cangjie 应用 | 中 | 应用代码可控，需验证输入 |
| 边界 2 | FFI 包装层 | 中 | 本模块，作为桥接层需校验参数 |
| 边界 3 | native_event_service | 高 | 系统服务，需信任但应防御 |
| 边界 4 | CES | 高 | 内核级服务，核心信任锚点 |

## 攻击面分析

### 输入向量

| 向量 | 类型 | 风险等级 | 代码位置 |
|------|------|---------|----------|
| `publish(event)` | 事件名称字符串 | 中 | `common_event_manager.cj:57` |
| `publish(data)` | 事件数据字符串 | 中 | `common_event_publish_data.cj:58` |
| `publish(parameters)` | HashMap 参数 | 高 | `common_event_publish_data.cj:91` |
| `subscribeInfo.events` | 订阅事件列表 | 中 | `common_event_subscribe_info.cj:80` |
| `subscribeInfo.publisherPermission` | 权限字符串 | 低 | `common_event_subscribe_info.cj:88` |
| `subscribeInfo.publisherBundleName` | 包名字符串 | 低 | `common_event_subscribe_info.cj:125` |
| `isSticky` | 布尔标记 | 高 | `common_event_publish_data.cj:83` |

## 可利用点分析

### 高风险项

#### 1. 粘性事件权限控制缺失

**证据**: `common_event_publish_data.cj:78-83`

```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.COMMONEVENT_STICKY",
    syscap: "SystemCapability.Notification.CommonEvent"
]
public var isSticky: Bool
```

**问题**: `@!APILevel` 注解中的 `permission` 字段仅作为**文档声明**，实际权限校验由底层 `common_event_service` 完成。Cangjie 层未阻止应用设置 `isSticky = true`。

**触发路径**:
```
应用调用 → CommonEventPublishData.isSticky = true → 
publish() → FFI: CJ_PublishEventWithData() → 
common_event_service (实际权限校验)
```

**影响**: 
- 若底层校验被绕过，可发送粘性事件
- 粘性事件持久化存储，可能导致信息泄露
- 拒绝服务: 频繁发布粘性事件消耗存储

**修复建议**:
- 在 Cangjie 层添加 `AccessToken` 权限预检查
- 使用 `AbilityAccessCtrl` API 验证调用者权限
- 记录权限校验失败日志

---

#### 2. 参数 HashMap 越界写入

**证据**: `common_event_publish_data.cj:155-157`

```cangjie
if (c.parameters.size != 0) {
    this.parameters = createCArrParam(c.parameters)
}
```

**问题**: `parameters` HashMap 大小未限制，可能传递超大数据结构导致:

1. 内存消耗过大 (DoS)
2. FFI 数据转换时的缓冲区溢出
3. 序列化/反序列化性能问题

**触发路径**:
```
应用构建 10000+ 参数 → HashMap 添加 → 
createCArrParam() → C 结构体分配 → 
FFI 传递 → 内存溢出/性能问题
```

**影响**:
- 内存耗尽导致系统不稳定
- 底层 C++ 处理时可能的缓冲区溢出
- 事件分发延迟

**修复建议**:
- 添加参数数量限制 (如最大 64 项)
- 添加单个参数值大小限制 (如最大 4KB)
- 使用 `try-catch` 捕获内存分配异常

---

#### 3. 字符串长度无校验

**证据**: `common_event_manager.cj:60`

```cangjie
cEvent = LibC.mallocCString(event).asResource()
```

**问题**: `event` 字符串和 `data` 字段长度未校验:

- 事件名称可能过长
- `data` 字段文档说最大 64KB，但 Cangjie 层未强制
- 未校验空字符串或特殊字符

**触发路径**:
```
publish("very_long_event_name_" + "x".repeat(100000)) → 
mallocCString 分配大内存 → 
FFI 传递 → 内存压力
```

**影响**:
- 内存耗尽
- 底层 C++ 可能截断导致数据丢失
- 特殊字符可能导致注入攻击

**修复建议**:
- 添加字符串长度限制 (事件名 < 256, data < 64KB)
- 校验字符串内容 (禁止控制字符)
- 使用安全的字符串拷贝函数

---

#### 4. FD 类型文件描述符传递

**证据**: `value_type.cj:68-73`

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Notification.CommonEvent"
]
FD(Int32)
```

**问题**: `CommonEventValueType.FD` 类型允许传递原始文件描述符整数:

- 未验证 FD 有效性
- FD 可能已关闭或属于其他进程
- FD 劫持攻击: 接收方可获得不应有的文件访问权限

**触发路径**:
```
应用获取文件 FD = 42 → 
publish(event, parameters: {"fd": FD(42)}) → 
订阅者接收 FD = 42 → 
读写任意文件描述符
```

**影响**:
- 权限提升: 访问其他进程的文件描述符
- 信息泄露: 读取敏感文件内容
- 拒绝服务: 关闭关键系统文件描述符

**修复建议**:
- 禁止普通应用使用 FD 类型
- 验证 FD 属于当前进程
- 使用 `dup()` 创建副本后再传递
- 仅允许特定系统应用使用

---

#### 5. 订阅者优先级越界

**证据**: `common_event_subscribe_info.cj:160-170`

```cangjie
func checkPriority(priority: Int32): Int32 {
    let maxValue = 1000i32
    let minValue = -100i32
    if (priority > maxValue) { maxValue }
    else if (priority < minValue) { minValue }
    // 越界值被静默截断，无错误反馈
}
```

**问题**: 优先级超出范围时静默截断，不抛出异常:

- 应用无法感知配置错误
- 可能导致事件分发优先级混乱
- 隐藏的 Bug 难以调试

**触发路径**:
```
subscribeInfo.priority = 10000 → 
checkPriority 截断为 1000 → 
事件按错误优先级分发
```

**影响**:
- 优先级逻辑失效
- 应用行为不符合预期
- 难以排查的问题

**修复建议**:
- 抛出 `BusinessException` 替代静默截断
- 或返回 Result 类型让应用处理
- 添加优先级边界校验日志

---

### 中风险项

#### 6. 用户 ID 未验证

**证据**: `common_event_subscribe_info.cj:38`

```cangjie
userId = info.userId  // 无校验
```

**问题**: `userId` 直接传递到 FFI，未验证是否为有效用户:

- 可能查询不存在的用户 ID
- 可能导致资源泄漏 (用户不存在但创建了订阅)

**修复建议**:
- 校验 userId 有效性
- 使用 `getOsAccountLocalId()` 验证

---

#### 7. Callback 未校验

**证据**: `common_event_manager.cj:113`

```cangjie
callback(None, commonData)  // 未捕获 callback 异常
```

**问题**: 用户提供的 callback 异常可能导致订阅状态不一致:

- callback 崩溃不清理 FFI 资源
- 可能导致内存泄漏
- 订阅状态不确定

**修复建议**:
- 在 FFI 层捕获 callback 异常
- 确保 subscriber 引用计数正确管理
- 添加超时机制防止 callback 永久阻塞

---

### 低风险项

#### 8. C 字符串内存泄漏风险

**证据**: `common_event_subscribe_info.cj:46-52`

```cangjie
try {
    publisherPermission = LibC.mallocCString(info.publisherPermission)
    publisherDeviceId = LibC.mallocCString(info.publisherDeviceId)
    publisherBundleName = LibC.mallocCString(info.publisherBundleName)
} catch(e: Exception) {
    free()  // 正确释放
    throw BusinessException(1500009, "Error obtaining system parameters.")
}
```

**状态**: ✅ **已正确处理** - 使用 try-catch 确保异常时释放内存

---

#### 9. 空指针传递

**证据**: `common_event_manager.cj:88`

```cangjie
if (errorCode == INVALID_CODE) { ... }
```

**状态**: ✅ **已处理** - 检查 `INVALID_CODE` 并抛出异常

---

## 安全修复优先级

| 优先级 | 问题 | 建议措施 |
|--------|------|----------|
| P0 | FD 类型越权 | 禁止普通应用使用 FD 类型 |
| P0 | 粘性事件权限 | 添加 AccessToken 预校验 |
| P1 | 参数 HashMap 越界 | 添加大小限制 |
| P1 | 字符串长度无校验 | 添加长度限制 |
| P2 | 优先级越界静默 | 改为抛出异常 |
| P2 | userId 未验证 | 添加有效性校验 |
| P2 | callback 异常未捕获 | 添加 try-catch |
| P3 | 文档不完整 | 完善安全注意事项 |

## 安全使用建议

### 应用开发者

1. **权限申请**: 使用 `isSticky` 前确保已申请 `ohos.permission.COMMONEVENT_STICKY`
2. **参数验证**: 发布前验证所有输入参数
3. **回调保护**: 为 subscribe callback 添加异常处理
4. **避免 FD**: 不要在 parameters 中传递文件描述符

### 系统集成

1. **模块隔离**: 确保 FFI 层与 native 层权限边界清晰
2. **输入过滤**: 在 common_event_service 入口增加全面校验
3. **资源限制**: 限制单个进程的订阅数量
4. **审计日志**: 记录敏感操作 (粘性事件发布、权限错误)

## 检查范围说明

本评审基于以下代码范围:

- `ohos/common_event_manager/` - 核心管理器 (6.1KB)
- `ohos/common_event_data/` - 事件数据 (1.7KB)
- `ohos/common_event_publish_data/` - 发布数据 (3.0KB)
- `ohos/common_event_subscribe_info/` - 订阅信息 (3.8KB)
- `ohos/common_event_subscriber/` - 订阅者 (0.5KB)
- `ohos/value_type/` - 值类型 (2.1KB)

**未包含范围**:
- `test/` - 测试代码
- `mock/` - Mock 实现
- `common_event_service` - 底层 C++ 服务
- `cangjie_ark_interop` - Cangjie 运行时
