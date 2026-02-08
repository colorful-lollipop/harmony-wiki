# 关键调用链

## 发布事件调用链

### publish()

```mermaid
sequenceDiagram
    participant App as Cangjie 应用
    participant CEM as CommonEventManager
    participant Trans as 数据转换层
    participant FFI as FFI 边界
    participant CES as CommonEventService

    App->>CEM: publish("event", options)
    CEM->>Trans: CCommonEventPublishData(options)
    Trans->>Trans: mallocCString() 字符串转换
    Trans->>Trans: createCArrParam() 参数转换
    CEM->>FFI: CJ_PublishEventWithData(event, UNDEFINED_USER, options)
    FFI->>CES: IPC call
    CES->>CES: 权限校验
    CES->>CES: 事件持久化
    CES-->>FFI: 返回 retCode
    FFI-->>CEM: 返回 retCode
    alt retCode != 0
        CEM->>CEM: throw BusinessException
    end
```

**代码路径**:

1. `common_event_manager.cj:57-67`
2. `common_event_publish_data.cj:141-163` (数据转换)
3. `common_event_manager_ffi.cj:26` (FFI 声明)
4. → `common_event_service:cj_common_event_manager_ffi` (C++ 实现)

---

## 创建订阅者调用链

### createSubscriber()

```mermaid
sequenceDiagram
    participant App as Cangjie 应用
    participant CEM as CommonEventManager
    participant Trans as 数据转换层
    participant FFI as FFI 边界
    participant CES as CommonEventService

    App->>CEM: createSubscriber(subscribeInfo)
    CEM->>Trans: CSubscribeInfo(subscribeInfo)
    Trans->>Trans: mallocCString() 字符串转换
    Trans->>Trans: cjArr2CArr() 数组转换
    CEM->>FFI: FfiCommonEventManagerCreateSubscriber(info, &errorCode)
    FFI->>CES: IPC call
    CES->>CES: 创建订阅者句柄
    CES-->>FFI: 返回 subscriberId
    FFI-->>CEM: 返回 subscriberId
    CEM->>Trans: info.free() 释放内存
    alt errorCode == INVALID_CODE
        CEM->>CEM: throw BusinessException
    else
        CEM-->>App: CommonEventSubscriber(id)
    end
```

**代码路径**:

1. `common_event_manager.cj:82-92`
2. `common_event_subscribe_info.cj:36-54` (数据转换)
3. `common_event_manager_ffi.cj:38` (FFI 声明)
4. → `common_event_service:cj_common_event_manager_ffi` (C++ 实现)

---

## 订阅事件调用链

### subscribe()

```mermaid
sequenceDiagram
    participant App as Cangjie 应用
    participant CEM as CommonEventManager
    participant Callback as AsyncCallback
    participant Wrapper as Callback Wrapper
    participant FFI as FFI 边界
    participant CES as CommonEventService

    App->>CEM: subscribe(subscriber, callback)
    CEM->>Wrapper: 创建 wrapper Closure
    Wrapper->>Wrapper: Callback1Param<CCommonEventData>
    CEM->>FFI: CJ_Subscribe(subscriberId, callbackId)
    FFI->>CES: IPC call - 注册订阅
    CES->>CES: 保存回调引用
    CES-->>FFI: 返回 retCode
    FFI-->>CEM: 返回 retCode
    
    Note over CES,App: 事件触发时
    CES->>CES: 事件匹配
    CES->>Wrapper: 触发回调 (CCommonEventData)
    Wrapper->>Wrapper: 数据转换 CommonEventData
    Wrapper->>Callback: callback(err, data)
    
    alt retCode != 0
        CEM->>CEM: throw BusinessException
    end
```

**代码路径**:

1. `common_event_manager.cj:109-118`
2. `common_event_data.cj:77-91` (数据接收)
3. `common_event_manager_ffi.cj:34` (FFI 声明)
4. → `common_event_service:cj_common_event_manager_ffi` (C++ 实现)

**线程注意**: callback 在主线程执行

---

## 取消订阅调用链

### unsubscribe()

```mermaid
sequenceDiagram
    participant App as Cangjie 应用
    participant CEM as CommonEventManager
    participant FFI as FFI 边界
    participant CES as CommonEventService

    App->>CEM: unsubscribe(subscriber)
    CEM->>FFI: CJ_Unsubscribe(subscriberId)
    FFI->>CES: IPC call - 取消订阅
    CES->>CES: 移除回调引用
    CES->>CES: 释放订阅者资源
    CES-->>FFI: 返回 retCode
    FFI-->>CEM: 返回 retCode
    CEM->>CEM: 析构 CommonEventSubscriber
    CEM->>CEM: releaseFFIData(subscriberId)
    
    alt retCode != 0
        CEM->>CEM: throw BusinessException
    end
```

**代码路径**:

1. `common_event_manager.cj:134-137`
2. `common_event_subscriber.cj:35-37` (资源释放)
3. `common_event_manager_ffi.cj:36` (FFI 声明)
4. → `common_event_service:cj_common_event_manager_ffi` (C++ 实现)

---

## 订阅者属性操作调用链

### CommonEventSubscriber getter/setter

```mermaid
sequenceDiagram
    participant Sub as CommonEventSubscriber
    participant FFI as FFI 边界
    participant CES as CommonEventService

    Sub->>FFI: CJ_GetCode(id)
    FFI->>CES: IPC call
    CES-->>FFI: 返回 code
    FFI-->>Sub: RetDataI32
    
    Sub->>FFI: CJ_SetCode(id, code)
    FFI->>CES: IPC call
    CES-->>FFI: 返回 retCode
    
    Sub->>FFI: CJ_GetData(id)
    FFI->>CES: IPC call
    CES-->>FFI: 返回 data 指针
    FFI-->>Sub: RetDataCString
```

**代码路径**:

1. `common_event_subscriber_ffi.cj:22-45` (所有 FFI 声明)
2. → `common_event_service:cj_common_event_manager_ffi` (C++ 实现)

---

## 完整调用层次

```
Cangjie 应用
    │
    ├── CommonEventManager
    │       ├── publish()
    │       ├── createSubscriber()
    │       ├── subscribe()
    │       └── unsubscribe()
    │
    ├── CommonEventPublishData
    │       └── @C struct CCommonEventPublishData
    │
    ├── CommonEventSubscribeInfo
    │       └── @C struct CSubscribeInfo
    │
    ├── CommonEventData
    │       └── @C struct CCommonEventData
    │
    └── CommonEventSubscriber
            └── RemoteDataLite
                    │
                    └── FFI 数据句柄 (Int64 id)

FFI 边界 (foreign { func CJ_* })
    │
    └── common_event_service:cj_common_event_manager_ffi
            │
            ├── IPCSkeleton
            │       │
            │       └── SendRequest(CommonEventService)
            │
            ├── CES Permission Check
            │
            └── Event Store & Dispatch
```

---

## 资源生命周期

```
┌─────────────────────────────────────────────────────────────────┐
│                     publish() 资源生命周期                        │
├─────────────────────────────────────────────────────────────────┤
│  mallocCString(event)  ──→  CString  ──→  asResource()  ──→  │
│                          自动释放                                     │
├─────────────────────────────────────────────────────────────────┤
│  mallocCString(data)  ──→  CString  ──→  asResource()  ──→  │
│                          自动释放                                     │
├─────────────────────────────────────────────────────────────────┤
│  CCommonEventPublishData  ──→  CTypeResource  ──→  自动释放    │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                  createSubscriber() 资源生命周期                  │
├─────────────────────────────────────────────────────────────────┤
│  CSubscribeInfo  ──→  mallocCString()  ──→  FFI 调用  ──→  │
│                                      free() 释放                 │
├─────────────────────────────────────────────────────────────────┤
│  subscriberId  ──→  CommonEventSubscriber  ──→  析构  ──→     │
│                              releaseFFIData()                    │
└─────────────────────────────────────────────────────────────────┘
```
