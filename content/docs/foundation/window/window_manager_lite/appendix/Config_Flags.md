# 附录：配置项说明

## 编译期配置

### 像素格式配置

#### LAYER_PF_ARGB1555

| 配置项 | 值 |
|--------|-----|
| 宏定义 | `LAYER_PF_ARGB1555` |
| 类型定义 | `LayerColorType = uint16_t` |
| 每像素 | 2 字节 |
| 用途 | 15位色深 + 1位Alpha |

#### LAYER_PF_ARGB8888

| 配置项 | 值 |
|--------|-----|
| 宏定义 | `LAYER_PF_ARGB8888` |
| 类型定义 | `LayerColorType = uint32_t` |
| 每像素 | 4 字节 |
| 用途 | 32位真彩色 + Alpha |

**证据**: `lite_wm_type.h:78-82`

```cpp
#ifdef LAYER_PF_ARGB1555
    typedef uint16_t LayerColorType;
#elif defined LAYER_PF_ARGB8888
    typedef uint32_t LayerColorType;
#endif
```

### 窗口数量限制

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `MAX_WINDOW_NUMBLE` | 32 | 最大窗口数量 |
| `WINDOW_ID_FULL_STORAGE` | 0xFFFFFFFF | ID 位图全满标记 |

**证据**: `lite_wm.cpp:95-96`

```cpp
const uint8_t MAX_WINDOW_NUMBLE = 32;
const uint32_t WINDOW_ID_FULL_STORAGE = 0xFFFFFFFF;
```

### 鼠标光标配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `CURSOR_WIDTH` | 24 | 光标宽度 |
| `CURSOR_HEIGHT` | 25 | 光标高度 |
| `CURSOR_MAP[]` | 硬编码位图 | 光标像素数据 |

**证据**: `lite_wm.cpp:25-93`

### 更新区域配置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `MAX_UPDATE_SIZE` | 8 | 最大更新区域数量 |

**证据**: `lite_wm.h:32`

```cpp
constexpr uint8_t MAX_UPDATE_SIZE = 8;
struct UpdateRegions {
    int num;
    Rect updates[MAX_UPDATE_SIZE];
    Rect bound;
};
```

## 运行期配置

### 窗口配置 (LiteWinConfig)

```cpp
struct LiteWinConfig {
    Rect rect;                    // 窗口位置和大小
    uint8_t opacity;              // 透明度 (0-255)
    ImagePixelFormat pixelFormat; // 像素格式
    CompositeMode compositeMode;   // 合成模式 (COPY/BLEND)
    bool isModal;                 // 是否模态窗口
};
```

### 任务配置 (SAMGR)

| 配置项 | 值 | 说明 |
|--------|-----|------|
| LEVEL | HIGH | 任务优先级级别 |
| PRIORITY | PRI_BELOW_NORMAL | 低于普通优先级 |
| STACK_SIZE | 0x800 | 栈大小 (2KB) |
| QUEUE_SIZE | 20 | 消息队列深度 |
| TASK_POLICY | SHARED_TASK | 共享任务池 |

**证据**: `samgr_wms.cpp:62-67`

```cpp
static TaskConfig GetTaskConfig(Service* service)
{
    (void)service;
    TaskConfig config = {LEVEL_HIGH, PRI_BELOW_NORMAL, 0x800, 20, SHARED_TASK};
    return config;
}
```

### 层配置 (LiteSurfaceData)

```cpp
struct LiteSurfaceData {
    ImagePixelFormat pixelFormat;  // 像素格式
    uint16_t width;                // 宽度
    uint16_t height;              // 高度
    uint8_t* virAddr;             // 虚拟地址
    uint8_t* phyAddr;             // 物理地址
    uint32_t stride;              // 行步幅
    uint8_t bytePerPixel;         // 每像素字节数
};
```

### 层信息 (LiteLayerInfo)

```cpp
struct LiteLayerInfo {
    ImagePixelFormat pixelFormat;
    uint16_t width;
    uint16_t height;
};
```

## 错误码定义

### WMS 错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `LiteWMS_EOK` | 0 | 成功 |
| `LiteWMS_EUNKNOWN` | 1 | 未知错误 |
| `INVALID_WINDOW_ID` | -1 | 无效窗口 ID |

**证据**: `lite_wm_type.h:68-86`

### IPC 错误码

WMS 使用标准 IPC 错误码：

| 错误码 | 说明 |
|--------|------|
| `EC_SUCCESS` | 成功 |
| `EC_FAILURE` | 失败 |

**证据**: `samgr_wms.cpp:72`

```cpp
static int Invoke(IServerProxy* iProxy, int funcId, void* origin, IpcIo* req, IpcIo* reply)
{
    LiteWMS::WMSRequestHandle(funcId, origin, req, reply);
    return EC_SUCCESS;
}
```

## 功能开关

### 鼠标光标支持

| 功能 | 默认值 | 控制方式 |
|------|--------|----------|
| 鼠标光标 | 关闭 | 首次鼠标事件触发 |

**证据**: `lite_wm.cpp:677-685`

```cpp
if (firstTime) {
    if (event.type == InputDevType::INDEV_TYPE_MOUSE) {
        cursorInfo_.enableCursor = true;
        cursorInfo_.needRedraw = true;
    } else {
        cursorInfo_.enableCursor = false;
    }
    firstTime = false;
}
```

### 截图功能

| 功能 | 权限要求 | 开关方式 |
|------|----------|----------|
| 截图 | `ohos.permission.WRITE_MEDIA_IMAGES` | 权限控制 |

### 模态窗口

| 功能 | 说明 |
|------|------|
| 模态窗口 | 优先级最高，拦截所有输入事件 |

**证据**: `lite_wm.cpp:630-631`

```cpp
if (node->data_->GetConfig().isModal) {
    return node->data_;
}
```

## 旋转配置

### 层旋转类型

| 配置 | 说明 |
|------|------|
| `LAYER_ROTATE_90` | 90度旋转 |

**证据**: `lite_wm.cpp:671-675`

```cpp
if (GetLayerRotateType() == LAYER_ROTATE_90) {
    int16_t tmp = layerData_->height - event.x;
    event.x = event.y;
    event.y = tmp;
}
```

## 服务名称定义

| 服务 | 名称 | 用途 |
|------|------|------|
| WMS | `"WMS"` | 窗口管理服务 |
| IMS | `"IMS"` | 输入管理服务 |

**证据**: `lite_wm_type.h:84`

```cpp
const char SERVICE_NAME[] = "WMS";
```

## 宏定义汇总

| 宏 | 值 | 用途 |
|------|-----|------|
| `SERVICE_NAME` | `"WMS"` | WMS 服务名 |
| `INVALID_WINDOW_ID` | -1 | 无效窗口 ID |
| `INVALID_PID` | -1 | 无效进程 ID |
| `MAX_UPDATE_SIZE` | 8 | 最大更新区域数 |
| `MAX_WINDOW_NUMBLE` | 32 | 最大窗口数 |
| `CURSOR_WIDTH` | 24 | 光标宽度 |
| `CURSOR_HEIGHT` | 25 | 光标高度 |
