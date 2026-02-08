# IMS 输入管理

## 概述

IMS (Input Manager Service) 是 window_manager_lite 的子组件，负责输入设备的监听和输入事件的分发。

## 核心组件

### 1. InputManagerService - 输入管理服务

**文件**: `services/ims/input_manager_service.h`

**职责**:
- 启动输入管理服务
- 维护事件队列
- 启动分发线程

```cpp
// input_manager_service.h:26-95
class InputManagerService {
public:
    static InputManagerService* GetInstance();
    static void Run();
    static void Stop();
    static void* Distribute(void* args);

    InputEventDistributer* GetDistributer();
    InputEventHub* GetHub();

private:
    static void ReadCallback(const RawEvent* event);
    static InputEventHub* hub_;
    static InputEventDistributer distributer_;
    static pthread_t distributerThread_;
    static std::queue<RawEvent> eventQueue_;
    static pthread_mutex_t lock_;
    static pthread_cond_t nonEmpty_;
    static pthread_cond_t nonFull_;
};
```

### 2. InputEventHub - 输入设备中心

**文件**: `services/ims/input_event_hub.h`

**职责**:
- 打开/关闭输入设备
- 注册读回调
- 事件封装

```cpp
// input_event_hub.h:29-82
class InputEventHub {
public:
    static InputEventHub* GetInstance();
    void SetUp();
    void TearDown();
    static int32_t RegisterReadCallback(const ReadCallback &callback);

private:
    static InputDevType GetDeviceType(uint32_t devIndex);
    static void EventCallback(const InputEventPackage **pkgs, uint32_t count, uint32_t devIndex);
    uint8_t ScanInputDevice();

    uint32_t mountDevIndex_[MAX_INPUT_DEVICE_NUM];
    uint32_t openDev_;
    RawEvent data_;
    static IInputInterface* inputInterface_;
    static InputEventCb callback_;
    static ReadCallback readCallback_;
};
```

### 3. InputEventDistributer - 事件分发器

**文件**: `services/ims/input_event_distributer.h`

**职责**:
- 将 RawEvent 分发给目标窗口
- 管理事件监听器列表

```cpp
// input_event_distributer.h:26-75
class InputEventDistributer {
public:
    void Distribute(const RawEvent* events, int32_t size);

    // 监听器管理
    void AddRawEventListener(RawEventListener* listener);
    void RemoveRawEventListener(RawEventListener* listener);

    class RawEventListener {
    public:
        virtual void OnRawEvent(const RawEvent& event) = 0;
    };

private:
    std::set<RawEventListener*> rawEventListeners_;
};
```

## 事件类型

### RawEvent 结构

```cpp
// 外部定义 (input_event_info.h)
struct RawEvent {
    uint32_t type;      // 事件类型
    uint32_t code;      // 事件代码
    int32_t value;      // 事件值
    int16_t x;          // X 坐标
    int16_t y;          // Y 坐标
    uint64_t time;      // 时间戳
};
```

### 输入设备类型

| 类型 | 说明 |
|------|------|
| `INDEV_TYPE_MOUSE` | 鼠标 |
| `INDEV_TYPE_TOUCH` | 触摸屏 |
| `INDEV_TYPE_KEY` | 键盘 |
| `INDEV_TYPE_BUTTON` | 按键 |

**证据**: `lite_wm.cpp:637-656`

## 事件分发流程

### 1. 设备事件读取

```
InputEventHub::SetUp()
    │
    ▼
打开输入设备 (HDI)
    │
    ▼
RegisterReadCallback(ReadCallback)
    │
    ▼
HDI 触发 EventCallback()
    │
    ▼
转换为 RawEvent
    │
    ▼
放入 eventQueue_
```

### 2. 事件分发线程

```cpp
// input_manager_service.cpp
void* InputManagerService::Distribute(void* args)
{
    while (distributerThreadCreated_) {
        pthread_mutex_lock(&lock_);
        while (eventQueue_.empty()) {
            pthread_cond_wait(&nonEmpty_, &lock_);
        }
        RawEvent* events = eventQueue_.front();
        eventQueue_.pop();
        pthread_mutex_unlock(&lock_);

        distributer_.Distribute(events, size);
    }
}
```

### 3. 分发给目标窗口

```
InputEventDistributer::Distribute()
    │
    ▼
遍历 rawEventListeners_
    │
    ▼
调用 LiteWM::OnRawEvent()
    │
    ▼
FindTargetWindow() // 找目标窗口
    │
    ├── 模态窗口 → 返回模态窗口
    ├── 鼠标/触摸 → 检查坐标是否在窗口内
    └── 按键 → 返回顶层窗口
    │
    ▼
SetEventData(window, event) // 设置事件数据
```

## 目标窗口选择算法

```cpp
// lite_wm.cpp:621-661
LiteWindow* LiteWM::FindTargetWindow(const RawEvent& event)
{
    // 1. 查找模态窗口
    auto node = winList_.Begin();
    while (node != winList_.End()) {
        if (node->data_->GetConfig().isModal) {
            return node->data_;  // 模态窗口优先
        }
        node = node->next_;
    }

    // 2. 根据事件类型分发
    switch (event.type) {
        case INDEV_TYPE_MOUSE:
        case INDEV_TYPE_TOUCH: {
            Point p = { event.x, event.y };
            auto win = winList_.Begin();
            while (win != winList_.End()) {
                if (win->data_->isShow_ &&
                    win->data_->GetConfig().rect.IsContains(p)) {
                    return win->data_;  // 坐标在窗口内
                }
                win = win->next_;
            }
            break;
        }
        case INDEV_TYPE_KEY:
        case INDEV_TYPE_BUTTON: {
            return winList_.Front();  // 按键发送给顶层窗口
        }
    }
    return nullptr;
}
```

## 线程安全

### 互斥锁保护

| 锁名 | 保护对象 |
|------|----------|
| `lock_` | eventQueue_ |
| `nonEmpty_` | 队列非空条件 |
| `nonFull_` | 队列非满条件 |

### 生产者/消费者模型

- **生产者**: HDI 回调 (`EventCallback`)
- **消费者**: Distribute 线程

## 客户端接口

### InputEventListenerProxy

**文件**: `frameworks/ims/input_event_listener_proxy.h`

**职责**: 客户端输入事件监听器代理

```cpp
class InputEventListenerProxy {
public:
    // 事件回调
    void OnInputEvent(const RawEvent& event);
};
```

### 调用关系

```
App (UI 框架)
    │
    ▼
InputEventListenerProxy::OnInputEvent()
    │
    ▼
LiteWM::OnRawEvent()
    │
    ▼
InputEventDistributer::Distribute()
```

## 与 WMS 的集成

### LiteWM 作为 RawEventListener

```cpp
// lite_wm.h:45
class LiteWM : public InputEventDistributer::RawEventListener {
public:
    void OnRawEvent(const RawEvent& event) override;
};
```

### 事件监听注册

```cpp
// lite_wm.cpp:108
InputManagerService::GetInstance()->GetDistributer()->AddRawEventListener(this);
```

## 错误处理

### 可能的错误场景

| 场景 | 处理方式 |
|------|----------|
| 输入设备打开失败 | 记录日志，继续运行 |
| 事件队列满 | 丢弃最旧事件 |
| 无目标窗口 | 丢弃事件 |
| HDI 回调失败 | 记录错误 |

### 错误码

IMS 层面没有定义专门的错误码，主要使用系统错误码。
