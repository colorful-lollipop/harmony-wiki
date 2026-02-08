# 错误码速查

## 错误码总览

| 错误码 | 名称 | 来源 | 严重程度 |
|--------|------|------|----------|
| 801 | CAPABILITY_NOT_SUPPORTED | 系统 | 中 |
| 1500003 | ERR_CES_TOO_FREQUENT | CES | 中 |
| 1500007 | ERR_CES_SEND_FAIL | CES | 高 |
| 1500008 | ERR_CES_UNINITIALIZED | CES | 高 |
| 1500009 | ERR_SYSTEM_PARAM | 系统 | 中 |
| 1500010 | ERR_TOO_MANY_SUBSCRIBERS | CES | 低 |

## 详细说明

### 801 - CAPABILITY_NOT_SUPPORTED

**含义**: 能力不支持

**触发场景**:
- 设备不支持 Common Event Service
- API Level 低于 22

**调用方**: `subscribe()`, `unsubscribe()`

**处理建议**:
```cangjie
try {
    CommonEventManager.subscribe(subscriber, callback)
} catch (e: BusinessException) {
    if (e.code == 801) {
        // 降级处理: 禁用公共事件功能
        println("Common Event not supported on this device")
    }
}
```

---

### 1500003 - ERR_CES_TOO_FREQUENT

**含义**: 公共事件发送频率过高

**触发场景**:
- 在短时间内发布过多事件
- 违反系统频率限制策略

**调用方**: `publish()`

**处理建议**:
```cangjie
try {
    CommonEventManager.publish(event)
} catch (e: BusinessException) {
    if (e.code == 1500003) {
        // 添加退避策略
        Thread.sleep(1000)  // 等待 1 秒后重试
        retryublish(event)
    }
}
```

**预防措施**:
- 实现事件发送节流 (throttle)
- 批量合并相似事件
- 使用标记避免重复发送

---

### 1500007 - ERR_CES_SEND_FAIL

**含义**: 向公共事件服务发送消息失败

**触发场景**:
- CES 服务无响应
- IPC 通信失败
- 系统资源不足

**调用方**: `publish()`, `subscribe()`, `unsubscribe()`

**处理建议**:
```cangjie
let maxRetries = 3
var retries = 0
var success = false

while (!success && retries < maxRetries) {
    try {
        CommonEventManager.publish(event)
        success = true
    } catch (e: BusinessException) {
        if (e.code == 1500007 && retries < maxRetries - 1) {
            retries++
            Thread.sleep(500)  // 重试前等待
        } else if (e.code == 1500007) {
            // 重试次数耗尽
            throw e
        }
    }
}
```

---

### 1500008 - ERR_CES_UNINITIALIZED

**含义**: 公共事件服务未完成初始化

**触发场景**:
- 系统启动早期调用
- CES 服务异常重启

**调用方**: `publish()`, `createSubscriber()`, `subscribe()`, `unsubscribe()`

**处理建议**:
```cangjie
try {
    CommonEventManager.createSubscriber(info)
} catch (e: BusinessException) {
    if (e.code == 1500008) {
        // 等待 CES 初始化
        Thread.sleep(2000)
        // 重试或提示用户
    }
}
```

**预防措施**:
- 应用启动时检测 CES 可用性
- 监听 `FOUNDATION_READY` 事件后再初始化

---

### 1500009 - ERR_SYSTEM_PARAM

**含义**: 获取系统参数失败

**触发场景**:
- 系统参数服务不可用
- 参数解析错误
- 内存分配失败

**调用方**: `createSubscriber()`, `publish()` (内部数据转换)

**处理建议**:
```cangjie
try {
    let subscriber = CommonEventManager.createSubscriber(info)
} catch (e: BusinessException) {
    if (e.code == 1500009) {
        // 清理部分创建的订阅
        // 降级到轮询模式
    }
}
```

---

### 1500010 - ERR_TOO_MANY_SUBSCRIBERS

**含义**: 订阅者数量超过系统规格

**触发场景**:
- 应用注册过多订阅
- 系统资源耗尽

**调用方**: `subscribe()`

**处理建议**:
```cangjie
try {
    CommonEventManager.subscribe(subscriber, callback)
} catch (e: BusinessException) {
    if (e.code == 1500010) {
        // 合并订阅: 减少订阅数量
        // 移除优先级低的订阅
    }
}
```

**预防措施**:
- 限制单个应用的订阅数量
- 使用单个订阅接收多个事件

---

## 异常处理模板

### 完整错误处理

```cangjie
import ohos.business_exception.BusinessException

func safePublish(event: String, data: CommonEventPublishData): Bool {
    try {
        CommonEventManager.publish(event, options: data)
        return true
    } catch (e: BusinessException) {
        match (e.code) {
            801 => println("Capability not supported")
            1500003 => {
                println("Rate limited, backing off")
                Thread.sleep(1000)
            }
            1500007 => println("CES not available")
            1500008 => {
                println("CES not initialized")
                Thread.sleep(2000)
            }
            1500009 => println("System parameter error")
            else => println("Unknown error: ${e.code}")
        }
        return false
    }
}
```

### 异步订阅错误处理

```cangjie
let callback = AsyncCallback<CommonEventData> { err, data ->
    if let e = err {
        match (e.code) {
            801 => {
                println("Capability lost, unsubscribing")
                CommonEventManager.unsubscribe(subscriber)
            }
            else => println("Callback error: ${e.code}")
        }
    } else if let event = data {
        processEvent(event)
    }
}

CommonEventManager.subscribe(subscriber, callback)
```

---

## 错误码来源验证

| 错误码 | 定义位置 | 验证方式 |
|--------|----------|----------|
| 801 | ArkTS 标准 | 系统定义 |
| 1500003-1500010 | common_event_service | CES 返回码 |

**代码证据**:

- `common_event_manager.cj:46-50` - 异常注释定义
- `common_event_manager_errors.cj` - 错误码常量定义 (待确认)
