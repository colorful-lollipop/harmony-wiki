# 依赖关系与使用

本文档记录 libusb 在 OpenHarmony 系统中的依赖关系和使用场景，分析该库如何被其他模块调用以及典型使用方式。

---

## 直接依赖者分析

### 搜索结果

**搜索范围**: `oh/` 目录下所有 BUILD.gn 文件
**搜索关键词**: `third_party/libusb`
**搜索结果**: **0 个直接依赖者**

### 分析说明

libusb 在当前 OpenHarmony 版本中未被其他模块显式依赖，可能的原因包括：

1. **底层服务定位**: libusb 可能被更底层的系统服务封装，对外隐藏依赖关系
2. **可选组件**: 可能通过条件编译或动态加载方式使用
3. **芯片 SDK 内部**: `chipsetsdk_sp` API 标签表明该库可能用于芯片 SDK 内部实现
4. **新增依赖**: 依赖关系可能在后续版本中添加

---

## 典型使用场景

尽管未找到直接依赖者，基于 libusb 的功能特性，可以推断其在 OpenHarmony 中的典型使用场景：

### 1. USB 设备通信

```
应用层 ──► USB 服务框架 ──► libusb ──► USB 设备
```

**使用方式**:
```c
#include <libusb/libusb.h>

// 初始化
libusb_context *ctx = NULL;
libusb_init(&ctx);

// 枚举设备
libusb_device **devs;
ssize_t cnt = libusb_get_device_list(ctx, &devs);

// 打开并通信
libusb_device_handle *handle;
libusb_open(*devs, &handle);

// 传输数据
int transferred;
libusb_bulk_transfer(handle, 0x01, data, sizeof(data), &transferred, 0);

// 清理
libusb_close(handle);
libusb_free_device_list(devs, 1);
libusb_exit(ctx);
```

### 2. USB 设备热插拔监控

```c
static int LIBUSB_CALL hotplug_callback(libusb_context *ctx,
                                         libusb_device *device,
                                         libusb_hotplug_event event,
                                         void *user_data) {
    if (event == LIBUSB_HOTPLUG_EVENT_DEVICE_ARRIVED) {
        // 处理设备连接事件
        libusb_device_handle *handle;
        libusb_open(device, &handle);
        // ...
    } else if (event == LIBUSB_HOTPLUG_EVENT_DEVICE_LEFT) {
        // 处理设备断开事件
    }
    return 0;
}

// 注册热插拔回调
libusb_hotplug_callback_handle handle;
libusb_hotplug_register_callback(NULL,
    LIBUSB_HOTPLUG_EVENT_DEVICE_ARRIVED | LIBUSB_HOTPLUG_EVENT_DEVICE_LEFT,
    0, LIBUSB_HOTPLUG_MATCH_ANY, LIBUSB_HOTPLUG_MATCH_ANY,
    LIBUSB_HOTPLUG_MATCH_ANY, hotplug_callback, NULL, &handle);
```

### 3. USB 设备信息获取

```c
// 获取设备描述符
struct libusb_device_descriptor desc;
libusb_get_device_descriptor(device, &desc);

printf("Vendor ID:  0x%04x\n", desc.idVendor);
printf("Product ID: 0x%04x\n", desc.idProduct);

// 获取设备速度
enum libusb_speed speed = libusb_get_device_speed(device);
switch (speed) {
    case LIBUSB_SPEED_LOW:    printf("Speed: Low Speed (1.5 Mbps)\n"); break;
    case LIBUSB_SPEED_FULL:   printf("Speed: Full Speed (12 Mbps)\n"); break;
    case LIBUSB_SPEED_HIGH:   printf("Speed: High Speed (480 Mbps)\n"); break;
    case LIBUSB_SPEED_SUPER:   printf("Speed: Super Speed (5 Gbps)\n"); break;
}
```

---

## API 使用指南

### 核心 API 分类

| API 分类 | 主要函数 | 用途 |
|---------|---------|------|
| 初始化/退出 | `libusb_init()`, `libusb_exit()` | 库生命周期管理 |
| 设备枚举 | `libusb_get_device_list()`, `libusb_free_device_list()` | 发现 USB 设备 |
| 设备打开 | `libusb_open()`, `libusb_close()` | 打开/关闭设备 |
| 控制传输 | `libusb_control_transfer()` | 发送控制命令 |
| 批量传输 | `libusb_bulk_transfer()` | 批量数据传输 |
| 中断传输 | `libusb_interrupt_transfer()` | 中断方式数据传输 |
| 等时传输 | `libusb_alloc_transfer()`, `libusb_submit_transfer()` | 实时数据传输 |
| 热插拔 | `libusb_hotplug_register_callback()` | 设备事件监控 |
| 错误处理 | `libusb_error_name()`, `libusb_strerror()` | 错误信息获取 |

### 头文件引用

```c
// 公共 API
#include <libusb/libusb.h>

// 内部接口（谨慎使用）
#include <libusb/libusbi.h>
```

---

## 依赖关系图

### 推断的依赖架构

```
┌─────────────────────────────────────────────────────────────────┐
│                     OpenHarmony 系统                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    应用层                               │   │
│  │  (直接调用 USB 服务的应用)                               │   │
│  └─────────────────────────────────────────────────────────┘   │
│                            │                                   │
│                            ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                 USB 服务框架                             │   │
│  │  - device_manager                                        │   │
│  │  - usb_service                                           │   │
│  │  - hdf_usb_driver                                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                            │                                   │
│                            ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    设备抽象层                           │   │
│  │                                                          │   │
│  │         ┌──────────────────────────────────┐            │   │
│  │         │         libusb                    │            │   │
│  │         │   (third_party/libusb)           │            │   │
│  │         │                                  │            │   │
│  │         │  - USB 设备枚举                  │            │   │
│  │         │  - 数据传输接口                  │            │   │
│  │         │  - 热插拔支持                    │            │   │
│  │         └──────────────────────────────────┘            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                            │                                   │
│                            ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                 操作系统抽象层                           │   │
│  │  - linux_usbfs (OHOS)                                   │   │
│  │  - Kernel USB Driver                                    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                            │                                   │
│                            ▼                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                      USB 硬件                           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 链接方式

### 动态链接

```gn
# BUILD.gn 依赖配置
deps = [
  "//third_party/libusb:libusb",
]
```

**说明**: 链接 `libusb_shared.so` 动态库，系统启动时加载。

### 静态链接

```gn
# BUILD.gn 依赖配置
deps = [
  "//third_party/libusb:libusb_source",
]
```

**说明**: 直接链接编译产物，适合对启动性能要求高的场景。

---

## 使用注意事项

### 1. 权限要求

```c
// USB 设备访问通常需要 root 权限或特定 SELinux 策略
// 在 OHOS 上，需要申请相应权限
```

### 2. 错误处理

```c
int r = libusb_init(&ctx);
if (r < 0) {
    fprintf(stderr, "libusb init failed: %s (%d)\n", 
            libusb_error_name(r), r);
    return r;
}

// 建议使用 libusb_set_option 设置调试级别
libusb_set_option(ctx, LIBUSB_OPTION_LOG_LEVEL, LIBUSB_LOG_LEVEL_INFO);
```

### 3. 线程安全

```c
// libusb 本身不是线程安全的
// 建议：
// 1. 单一线程管理 libusb_context
// 2. 使用同步机制保护设备句柄访问
// 3. 避免在回调中调用非线程安全的 API
```

### 4. 资源释放

```c
// 必须正确释放所有资源
libusb_device_handle *handle = NULL;
libusb_device **list = NULL;

// 1. 先释放设备列表
if (list) {
    libusb_free_device_list(list, 1);  // 参数 1 表示同时 unref 设备
}

// 2. 再关闭句柄
if (handle) {
    libusb_close(handle);
}

// 3. 最后退出库
if (ctx) {
    libusb_exit(ctx);
}
```

---

## 性能考虑

### 1. 批量传输优化

```c
// 大批量数据传输时，建议增大传输缓冲区
#define BULK_TRANSFER_SIZE (16 * 1024)  // 16KB
unsigned char buffer[BULK_TRANSFER_SIZE];
int transferred;

// 使用零拷贝方式提高性能
libusb_bulk_transfer(handle, endpoint, buffer, sizeof(buffer), 
                    &transferred, timeout);
```

### 2. 异步操作

```c
// 对于大量数据传输，使用异步 API
libusb_transfer *transfer = libusb_alloc_transfer(0);
libusb_fill_bulk_transfer(transfer, handle, endpoint, data, len,
                          callback, NULL, timeout);

int r = libusb_submit_transfer(transfer);
if (r < 0) {
    fprintf(stderr, "Failed to submit transfer: %s\n", 
            libusb_error_name(r));
}
```

---

## 相关文档

- [01_Overview.md](01_Overview.md) - libusb 功能介绍
- [02_Patches.md](02_Patches.md) - OH 适配 Patch
- [03_Build_Integration.md](03_Build_Integration.md) - 构建配置

---

*文档版本: 1.0*
*最后更新: 2026-02-07*
