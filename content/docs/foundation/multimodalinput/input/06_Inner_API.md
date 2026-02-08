# 06_Inner_API - 内部模块接口

## 概述

本文档描述 multimodalinput_input 子系统的内部 API，供模块间调用参考。

---

## 6.1 Inner API 清单

### 6.1.1 InputManager (Inner API)

**头文件**: `interfaces/native/innerkits/proxy/include/input_manager.h`

| 方法 | 功能 | 稳定性 |
|------|------|--------|
| `GetInstance()` | 获取单例 | 稳定 |
| `GetDisplayBindInfo()` | 获取显示绑定信息 | 稳定 |
| `SetDisplayBind()` | 设置显示绑定 | 稳定 |
| `UpdateDisplayInfo()` | 更新显示信息 | 稳定 |
| `UpdateWindowInfo()` | 更新窗口信息 | 稳定 |
| `AddInputEventFilter()` | 添加事件过滤器 | 稳定 |
| `RemoveInputEventFilter()` | 移除事件过滤器 | 稳定 |
| `SubscribeKeyEvent()` | 订阅按键事件 | 稳定 |
| `UnsubscribeKeyEvent()` | 取消订阅按键 | 稳定 |
| `SetWindowInputEventConsumer()` | 设置窗口事件消费者 | 稳定 |

### 6.1.2 IInputEventConsumer

**头文件**: `interfaces/native/innerkits/event/include/i_input_event_consumer.h`

| 方法 | 功能 | 稳定性 |
|------|------|--------|
| `OnInputEvent()` | 输入事件回调 | 稳定 |
| `OnInputEventNull()` | 空事件回调 | 稳定 |

---

## 6.2 模块依赖关系

```
InputManager
    │
    ├──▶ DeviceManager (设备管理)
    │       │
    │       └──▶ DeviceConfig (设备配置)
    │               │
    │               └──▶ DeviceParser (设备解析)
    │
    ├──▶ EventHandler (事件处理)
    │       │
    │       ├──▶ EventNormalizeHandler
    │       ├──▶ EventFilterHandler
    │       ├──▶ EventInterceptorHandler
    │       ├──▶ EventDispatchHandler
    │       └──▶ EventMonitorHandler
    │
    └──▶ EventSubscriber (事件订阅)
            │
            ├──▶ KeySubscriberHandler
            └──▶ ShortKeyHandler
```

---

## 6.3 稳定性标注

### 6.3.1 稳定性等级

| 等级 | 标记 | 说明 |
|------|------|------|
| 稳定 | Stable | 可直接使用，API 不会变更 |
| 测试 | Testing | 已测试，但 API 可能变更 |
| 实验 | Experimental | 新功能，API 可能频繁变更 |
| 废弃 | Deprecated | 不推荐使用，将被移除 |

### 6.3.2 稳定性使用建议

```cpp
// 推荐: 使用稳定 API
auto inputManager = InputManager::GetInstance();
inputManager->SubscribeKeyEvent(keyOption, callback);

// 避免: 使用实验性 API
// experimentalFeature->SomeMethod();  // 不推荐
```

---

## 6.4 可替换点

### 6.4.1 事件归一化替换

| 替换点 | 当前实现 | 可替换为 |
|--------|---------|----------|
| 事件归一化 | EventNormalizeHandler | 自定义 Handler |

**替换方法**:
```cpp
// 在 event_handler_config.json 中配置
{
  "event_normalize_handler": {
    "impl": "custom_event_normalizer"
  }
}
```

### 6.4.2 设备解析替换

| 替换点 | 当前实现 | 可替换为 |
|--------|---------|----------|
| 设备配置解析 | DefaultDeviceParser | 自定义解析器 |

---

## 6.5 头文件索引

| 模块 | 头文件 | 稳定性 |
|------|--------|--------|
| 输入管理 | `input_manager.h` | Stable |
| 事件 | `input_event.h` | Stable |
| 按键事件 | `key_event.h` | Stable |
| 指针事件 | `pointer_event.h` | Stable |
| 事件观察者 | `mmi_event_observer.h` | Stable |
| 窗口信息 | `window_info.h` | Stable |
