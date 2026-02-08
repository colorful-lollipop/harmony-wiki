# 02_N-API 接口

## 模块信息

| 属性 | 值 |
|------|-----|
| JS 模块 | `@ohos.commonevent` |
| N-API 实现 | `interfaces/kits/napi/napi_common_event/` |
| 入口文件 | `interfaces/kits/napi/common_event/src/init.cpp` |
| 注册宏 | `napi_module_register(&_module)` |

## API 清单

### CommonEvent

| 方法 | 同步/异步 | 说明 |
|------|----------|------|
| `publish(event, callback)` | Async | 发布公共事件 |
| `publish(event, options, callback)` | Async | 带选项发布事件 |
| `createSubscriber(info, callback)` | Async | 创建订阅者 |
| `createSubscriber(info)` | Promise | 创建订阅者 (Promise) |
| `subscribe(subscriber, callback)` | Async | 订阅公共事件 |
| `unsubscribe(subscriber, callback?)` | Async | 取消订阅 |

### CommonEventSubscriber

| 方法 | 同步/异步 | 说明 |
|------|----------|------|
| `getCode()` | Promise | 获取事件代码 |
| `setCode(code)` | Promise | 设置事件代码 |
| `getData()` | Promise | 获取事件数据 |
| `setData(data)` | Promise | 设置事件数据 |
| `setCodeAndData(code, data)` | Promise | 设置代码和数据 |
| `isOrderedCommonEvent()` | Promise | 是否有序事件 |
| `abortCommonEvent()` | Promise | 终止有序事件 |
| `getAbortCommonEvent()` | Promise | 获取终止状态 |
| `clearAbortCommonEvent()` | Promise | 清除终止状态 |
| `getSubscribeInfo()` | Promise | 获取订阅信息 |
| `finishCommonEvent()` | Promise | 完成事件处理 |

## 详细 API 说明

### CommonEvent.publish

#### 原型

```typescript
// Callback 形式
function publish(event: string, callback: AsyncCallback<void>): void
function publish(event: string, options: CommonEventPublishData, callback: AsyncCallback<void>): void

// Promise 形式
function publish(event: string, options: CommonEventPublishData): Promise<void>
```

#### 参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| event | string | 是 | 公共事件名称 |
| options | CommonEventPublishData | 否 | 发布选项 |
| callback | AsyncCallback<void> | 是 (Callback 形式) | 回调函数 |

#### CommonEventPublishData

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| bundleName | string | 否 | 包名 |
| code | number | 否 | 结果代码 |
| data | string | 否 | 自定义数据 |
| subscriberPermissions | Array<string> | 否 | 订阅者权限 |
| isOrdered | boolean | 否 | 是否有序事件 |
| parameters | object | 否 | 附加参数 |

#### 绑定位置

| 位置 | 说明 |
|------|------|
| N-API 入口 | `napi_common_event.cpp` |
| 参数解析 | `common_event_parse.cpp` |
| Native 实现 | `CommonEventManager::PublishCommonEvent()` |

#### 使用示例

```javascript
import CommonEvent from '@ohos.commonevent'

// 简单发布
CommonEvent.publish("my_event", (err) => {
    if (err) {
        console.error(`Publish failed: ${err.code}`)
    } else {
        console.info('Publish success')
    }
})

// 带选项发布
const options = {
    bundleName: "com.example.app",
    code: 100,
    data: "test data",
    isOrdered: true
}
CommonEvent.publish("my_event", options, (err) => {
    // 处理结果
})
```

### CommonEvent.createSubscriber

#### 原型

```typescript
function createSubscriber(info: CommonEventSubscribeInfo, callback: AsyncCallback<CommonEventSubscriber>): void
function createSubscriber(info: CommonEventSubscribeInfo): Promise<CommonEventSubscriber>
```

#### 参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| info | CommonEventSubscribeInfo | 是 | 订阅信息 |
| callback | AsyncCallback<CommonEventSubscriber> | 是 (Callback 形式) | 回调函数 |

#### CommonEventSubscribeInfo

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| events | Array<string> | 是 | 订阅的事件列表 |
| publisherPermission | string | 否 | 发布者权限要求 |
| publisherDeviceId | number | 否 | 发布者设备ID |
| userId | number | 否 | 用户ID |
| priority | number | 否 | 优先级 (-100~1000) |

#### 绑定位置

| 位置 | 说明 |
|------|------|
| N-API 入口 | `napi_common_event.cpp` - `NapiCommonEventCreateSubscriber()` |
| Native 实现 | `CommonEventSubscriber` 类 |

#### 使用示例

```javascript
import CommonEvent from '@ohos.commonevent'

const subscribeInfo = {
    events: ["event1", "event2"],
    priority: 100
}

// Callback 形式
CommonEvent.createSubscriber(subscribeInfo, (err, subscriber) => {
    if (err) {
        console.error(`Create subscriber failed: ${err.code}`)
    } else {
        console.info('Subscriber created')
    }
})

// Promise 形式
const subscriber = await CommonEvent.createSubscriber(subscribeInfo)
```

### CommonEvent.subscribe

#### 原型

```typescript
function subscribe(subscriber: CommonEventSubscriber, callback: AsyncCallback<CommonEventData>): void
```

#### 参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| subscriber | CommonEventSubscriber | 是 | 订阅者对象 |
| callback | AsyncCallback<CommonEventData> | 是 | 事件回调 |

#### CommonEventData

| 字段 | 类型 | 说明 |
|------|------|------|
| event | string | 事件名称 |
| bundleName | string | 发布者包名 |
| code | number | 结果代码 |
| data | string | 自定义数据 |
| parameters | object | 附加参数 |

#### 绑定位置

| 位置 | 说明 |
|------|------|
| N-API 入口 | `napi_common_event.cpp` - `NapiCommonEventSubscribe()` |
| 回调处理 | `ThreadSafeCallback()` |
| Native 实现 | `CommonEventManager::SubscribeCommonEvent()` |

#### 使用示例

```javascript
import CommonEvent from '@ohos.commonevent'

function subscriberCallback(err, data) {
    if (err) {
        console.error(`Subscribe error: ${err.code}`)
        return
    }
    console.info(`Received event: ${data.event}`)
    console.info(`Bundle: ${data.bundleName}`)
    console.info(`Code: ${data.code}, Data: ${data.data}`)
}

CommonEvent.subscribe(subscriber, subscriberCallback)
```

### CommonEvent.unsubscribe

#### 原型

```typescript
function unsubscribe(subscriber: CommonEventSubscriber, callback?: AsyncCallback<void>): void
```

#### 参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| subscriber | CommonEventSubscriber | 是 | 订阅者对象 |
| callback | AsyncCallback<void> | 否 | 完成回调 |

#### 绑定位置

| 位置 | 说明 |
|------|------|
| N-API 入口 | `napi_common_event.cpp` - `NapiCommonEventUnsubscribe()` |
| Native 实现 | `CommonEventManager::UnSubscribeCommonEvent()` |

## 参数校验

### 事件名称校验

| 校验项 | 规则 |
|--------|------|
| 空值检查 | event 不能为空 |
| 长度限制 | 最大 64KB (`STR_DATA_MAX_SIZE`) |
| 编码 | UTF-8 |

### 订阅事件数校验

| 校验项 | 规则 |
|--------|------|
| 最大数量 | 由系统配置限制 |
| 重复检查 | 同一事件不重复订阅 |

### 权限校验

| 场景 | 校验项 |
|------|--------|
| publisherPermission | 发布者需具备指定权限 |
| 系统事件 | 部分事件需要特定权限 |

## 错误码

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| -1 | 通用错误 |
| 其他 | 见 `ces_inner_error_code.h` |

## 系统公共事件

CES 提供大量预定义系统公共事件，完整列表见 [README_zh.md](../README_zh.md#系统公共事件定义)。

| 类别 | 示例 |
|------|------|
| 启动完成 | `usual.event.BOOT_COMPLETED` |
| 电源事件 | `usual.event.BATTERY_CHANGED` |
| 用户事件 | `usual.event.USER_SWITCHED` |
| 网络事件 | `usual.event.wifi.SCAN_FINISHED` |

## 相关文档

- [概览](00_Overview.md)
- [架构](01_Architecture.md)
- [内部 API](03_Inner_API.md)
