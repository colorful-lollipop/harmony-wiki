# Inner API

本章节描述 window_manager_lite 的模块间接口（Inner API），这些接口用于进程内模块间的 C++ 调用。

**注意**: 本组件**不提供 N-API（JavaScript 接口）**，所有 API 均为 C++ 原生接口。

## 接口清单

### 1. IWindowsManager - 窗口管理器接口

**文件**: `interfaces/innerkits/iwindows_manager.h`

**职责**: 窗口管理器的抽象接口，提供窗口创建、销毁、截图等功能。

```cpp
// iwindows_manager.h:29-62
class IWindowsManager {
public:
    // 获取单例
    static IWindowsManager* GetInstance();

    // 初始化
    virtual int Init() = 0;

    // 窗口管理
    virtual IWindow* CreateWindow(const LiteWinConfig& config) = 0;
    virtual void RemoveWindow(IWindow* win) = 0;

    // 输入事件
    virtual void GetEventData(DeviceData* data) = 0;

    // 截图功能
    virtual void Screenshot() = 0;
    virtual void SetScreenshotListener(ScreenshotListener* listener) = 0;
};
```

#### 实现类

| 实现类 | 文件 | 用途 |
|--------|------|------|
| `LiteProxyWindowsManager` | `lite_proxy_windows_manager.h` | 客户端代理实现 |

### 2. IWindow - 窗口接口

**文件**: `interfaces/innerkits/iwindow.h`

**职责**: 单个窗口的抽象接口，提供窗口生命周期和几何操作。

```cpp
// iwindow.h:27-90
class IWindow {
public:
    // 生命周期
    virtual int Init() = 0;
    virtual void Destroy() = 0;

    // 显示控制
    virtual void Show() = 0;
    virtual void Hide() = 0;

    // 几何操作
    virtual void Resize(int16_t width, int16_t height) = 0;
    virtual void MoveTo(int16_t x, int16_t y) = 0;
    virtual void RaiseToTop() = 0;
    virtual void LowerToBottom() = 0;

    // 查询
    virtual ISurface* GetSurface() = 0;
    virtual int32_t GetWindowId() = 0;
    virtual void Update() = 0;
};
```

#### 实现类

| 实现类 | 文件 | 用途 |
|--------|------|------|
| `LiteProxyWindow` | `lite_proxy_window.h` | 客户端窗口代理 |

### 3. ISurface - Surface 接口

**文件**: `interfaces/innerkits/isurface.h`

**职责**: 图形缓冲区的抽象接口，用于帧缓冲操作。

```cpp
// isurface.h:27-39
class ISurface {
public:
    // 获取缓冲区
    virtual void Lock(void** buf, void** phyMem, uint32_t* strideLen) = 0;

    // 解锁缓冲区
    virtual void Unlock() = 0;
};
```

#### 实现类

| 实现类 | 文件 | 用途 |
|--------|------|------|
| `LiteProxySurface` | `lite_proxy_surface.h` | 客户端 Surface 代理 |

## 数据结构

### LiteWinConfig - 窗口配置

```cpp
// lite_wm_type.h:23-33
struct LiteWinConfig {
    enum CompositeMode {
        COPY,   // 覆盖模式
        BLEND   // 混合模式（支持透明度）
    };
    Rect rect;              // 窗口矩形
    uint8_t opacity;        // 透明度
    ImagePixelFormat pixelFormat;  // 像素格式
    CompositeMode compositeMode;   // 合成模式
    bool isModal;          // 是否模态窗口
};
```

### DeviceData - 输入事件数据

```cpp
// gfx_utils/input_event_info.h (外部定义)
struct DeviceData {
    Point point;           // 触摸/鼠标位置
    int32_t winId;         // 目标窗口 ID
    int32_t state;         // 事件状态
};
```

### LiteSurfaceData - Surface 数据

```cpp
// lite_wm_type.h:35-43
struct LiteSurfaceData {
    ImagePixelFormat pixelFormat;
    uint16_t width;
    uint16_t height;
    uint8_t* virAddr;      // 虚拟地址
    uint8_t* phyAddr;      // 物理地址
    uint32_t stride;
    uint8_t bytePerPixel;
};
```

## 接口详细说明

### CreateWindow - 创建窗口

| 属性 | 值 |
|------|-----|
| 函数原型 | `IWindow* CreateWindow(const LiteWinConfig& config)` |
| 调用方 | 客户端 |
| 实现类 | `LiteProxyWindowsManager` |
| 返回值 | 窗口指针，失败返回 `nullptr` |

**实现位置**: `lite_proxy_windows_manager.cpp:29-30`

```cpp
IWindow* LiteProxyWindowsManager::CreateWindow(const LiteWinConfig& config)
{
    return new LiteProxyWindow(LiteWMRequestor::GetInstance()->CreateWindow(config));
}
```

### RemoveWindow - 销毁窗口

| 属性 | 值 |
|------|-----|
| 函数原型 | `void RemoveWindow(IWindow* win)` |
| 参数 | `win` - 要销毁的窗口指针 |

### Show/Hide - 显示/隐藏窗口

| 属性 | 值 |
|------|-----|
| 函数原型 | `void Show()` / `void Hide()` |
| 调用链 | `LiteProxyWindow::Show()` → `lite_win_requestor_.Show()` → IPC → `LiteWMS::Show()` → `LiteWM::Show()` |

### Resize - 调整窗口大小

| 属性 | 值 |
|------|-----|
| 函数原型 | `void Resize(int16_t width, int16_t height)` |
| 参数 | `width` - 宽度<br>`height` - 高度 |

**服务端处理**: `lite_wms.cpp:166-176`

```cpp
void LiteWMS::Resize(IpcIo* req, IpcIo* reply)
{
    int32_t id;
    ReadInt32(req, &id);
    uint32_t width, height;
    ReadUint32(req, &width);
    ReadUint32(req, &height);
    LiteWM::GetInstance()->Resize(id, width, height);
}
```

### MoveTo - 移动窗口

| 属性 | 值 |
|------|-----|
| 函数原型 | `void MoveTo(int16_t x, int16_t y)` |
| 参数 | `x` - X 坐标<br>`y` - Y 坐标 |

### RaiseToTop/LowerToBottom - 窗口层级

| 属性 | 值 |
|------|-----|
| 函数原型 | `void RaiseToTop()` / `void LowerToBottom()` |
| 说明 | 将窗口置于栈顶或栈底 |

### GetSurface - 获取 Surface

| 属性 | 值 |
|------|-----|
| 函数原型 | `ISurface* GetSurface()` |
| 返回值 | Surface 指针，用于图形渲染 |

### GetWindowId - 获取窗口 ID

| 属性 | 值 |
|------|-----|
| 函数原型 | `int32_t GetWindowId()` |
| 返回值 | 窗口唯一标识符 |

### GetEventData - 获取输入事件数据

| 属性 | 值 |
|------|-----|
| 函数原型 | `void GetEventData(DeviceData* data)` |
| 参数 | `data` - 输出参数，接收事件数据 |

### Screenshot - 截图

| 属性 | 值 |
|------|-----|
| 函数原型 | `void Screenshot()` |
| **权限要求** | `ohos.permission.WRITE_MEDIA_IMAGES` |
| **实现位置**: `lite_wms.cpp:216-228` |

```cpp
void LiteWMS::Screenshot(IpcIo* req, IpcIo* reply)
{
    const char* writeMediaImagePermissionName = "ohos.permission.WRITE_MEDIA_IMAGES";
    pid_t uid = GetCallingUid();
    if (CheckPermission(uid, writeMediaImagePermissionName) != GRANTED) {
        GRAPHIC_LOGE("permission denied");
        WriteInt32(reply, LiteWMS_EUNKNOWN);
        return;
    }
    // 执行截图...
}
```

## 错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `LiteWMS_EOK` | 0 | 成功 |
| `LiteWMS_EUNKNOWN` | 1 | 未知错误 |
| `INVALID_WINDOW_ID` | -1 | 无效窗口 ID |

**证据**: `lite_wm_type.h:68-86`

## 接口稳定性

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `IWindowsManager` | 稳定 | 官方接口 |
| `IWindow` | 稳定 | 官方接口 |
| `ISurface` | 稳定 | 官方接口 |
| `LiteWMRequestor` | 内部 | 客户端内部使用 |
| `LiteWinRequestor` | 内部 | 客户端内部使用 |

**稳定性判定依据**: 接口定义在 `interfaces/innerkits/` 目录下，遵循 OpenHarmony 接口规范。
