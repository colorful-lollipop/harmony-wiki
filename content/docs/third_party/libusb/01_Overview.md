# 原始库简介

## libusb 概述

**libusb** 是一个跨平台的 C 语言库，旨在为用户提供空间（userspace）级别的 USB 设备访问能力。通过 libusb，开发者无需编写内核驱动即可实现与 USB 设备的数据交互，大大简化了 USB 应用开发流程。

该库设计为可移植的，支持 Linux、macOS、Windows、OpenBSD、NetBSD、Haiku 和 Solaris 等多种操作系统。

---

## 核心功能

### 1. 设备枚举与发现

libusb 提供了完整的 USB 设备枚举接口，开发者可以获取系统中所有 USB 设备的详细信息，包括：

- 设备描述符（Vendor ID、Product ID）
- 配置描述符
- 接口描述符
- 端点描述符
- 设备速度（Low-Speed、Full-Speed、High-Speed、SuperSpeed）

### 2. 设备打开与关闭

```c
// 打开设备
libusb_device_handle *handle = NULL;
int r = libusb_open(device, &handle);
if (r < 0) {
    fprintf(stderr, "Failed to open device: %s\n", libusb_error_name(r));
    return;
}

// 获取设备信息
uint8_t bus_number = libusb_get_bus_number(handle);
uint8_t device_address = libusb_get_device_address(handle);

// 关闭设备
libusb_close(handle);
```

### 3. 通信接口

libusb 提供了四种基本的 USB 通信方式：

#### 控制传输 (Control Transfer)

用于向设备发送控制命令，是最常用的通信方式：

```c
// 发送控制传输
uint8_t bmRequestType = LIBUSB_ENDPOINT_OUT | LIBUSB_REQUEST_TYPE_CLASS | LIBUSB_RECIPIENT_INTERFACE;
uint8_t bRequest = 0x09;
uint16_t wValue = 0x0200;
uint16_t wIndex = 0;
uint8_t data[64] = {0};

int r = libusb_control_transfer(handle, bmRequestType, bRequest, wValue, wIndex,
                                data, sizeof(data), timeout);
```

#### 批量传输 (Bulk Transfer)

用于大批量数据传输，适用于打印机、存储设备等：

```c
// 批量写
unsigned char data[4096];
int transferred;
int r = libusb_bulk_transfer(handle, 0x01, data, sizeof(data), &transferred, timeout);
```

#### 中断传输 (Interrupt Transfer)

用于小批量、低延迟的数据传输：

```c
// 中断读
unsigned char data[64];
int transferred;
int r = libusb_interrupt_transfer(handle, 0x81, data, sizeof(data), &transferred, timeout);
```

#### 等时传输 (Isochronous Transfer)

用于实时音视频等对时间敏感的应用：

```c
// 等时传输需要特殊处理
libusb_transfer *transfer = libusb_alloc_transfer(0);
libusb_fill_iso_transfer(transfer, handle, 0x81, data, packets, 
                          libusb_get_iso_packet_buffer_list, callback, user_data);
libusb_submit_transfer(transfer);
```

### 4. 热插拔支持

libusb 支持 USB 设备的热插拔事件通知：

```c
// 注册热插拔回调
libusb_hotplug_callback_handle handle;
int r = libusb_hotplug_register_callback(NULL, 
    LIBUSB_HOTPLUG_EVENT_DEVICE_ARRIVED | LIBUSB_HOTPLUG_EVENT_DEVICE_LEFT,
    LIBUSB_HOTPLUG_NO_FLAGS, LIBUSB_HOTPLUG_MATCH_ANY, 
    LIBUSB_HOTPLUG_MATCH_ANY, LIBUSB_HOTPLUG_MATCH_ANY,
    hotplug_callback, NULL, &handle);

// 事件循环
while (running) {
    libusb_handle_events_completed(ctx, NULL);
}

static int LIBUSB_CALL hotplug_callback(libusb_context *ctx, libusb_device *device,
                                        libusb_hotplug_event event, void *user_data) {
    if (event == LIBUSB_HOTPLUG_EVENT_DEVICE_ARRIVED) {
        // 设备已连接
    } else if (event == LIBUSB_HOTPLUG_EVENT_DEVICE_LEFT) {
        // 设备已断开
    }
    return 0;
}
```

### 5. 错误处理

libusb 提供了详细的错误代码和错误名称映射：

```c
int r = libusb_init(&ctx);
if (r < 0) {
    fprintf(stderr, "Error: %s [%d]\n", libusb_error_name(r), r);
}

// 释放字符串描述符
char *error_str = libusb_strerror(r);
```

---

## 架构设计

### 后端架构

libusb 采用后端（backend）架构设计，每个操作系统平台有独立的实现后端：

| 平台 | 后端文件 | 说明 |
|------|---------|------|
| Linux | `linux_usbfs.c` | 使用 USB FS 接口 |
| macOS | `darwin_usb.c` | 使用 IOKit 框架 |
| Windows | `windows_winusb.c` | 使用 WinUSB 驱动 |
| OpenBSD | `openbsd_usb.c` | 使用 OpenBSD USB 接口 |
| NetBSD | `netbsd_usb.c` | 使用 NetBSD USB 接口 |
| Haiku | `haiku_usb.cpp` | Haiku 平台 C++ 实现 |
| SunOS | `sunos_usb.c` | SunOS 平台实现 |

### 线程模型

libusb 使用跨平台的线程和事件处理机制：

- **POSIX 平台**: 使用 `events_posix.c` 和 `threads_posix.c`
- **Windows**: 使用 `events_windows.c` 和 `threads_windows.c`

### 数据结构

核心数据结构包括：

```c
// USB 设备描述符
struct libusb_device_descriptor {
    uint8_t bLength;
    uint8_t bDescriptorType;
    uint16_t bcdUSB;
    uint8_t bDeviceClass;
    uint8_t bDeviceSubClass;
    uint8_t bDeviceProtocol;
    uint8_t bMaxPacketSize0;
    uint16_t idVendor;
    uint16_t idProduct;
    uint16_t bcdDevice;
    uint8_t iManufacturer;
    uint8_t iProduct;
    uint8_t iSerialNumber;
    uint8_t bNumConfigurations;
};

// USB 传输请求
struct libusb_transfer {
    libusb_device_handle *dev_handle;
    uint8_t flags;
    unsigned char *buffer;
    int length;
    int actual_length;
    libusb_transfer_cb_fn callback;
    void *user_data;
    // ...
};
```

---

## OpenHarmony 中的定位

在 OpenHarmony 生态系统中，libusb 扮演着以下角色：

### 1. USB 设备通信基础库

libusb 为 OpenHarmony 提供了标准的 USB 设备访问接口，使得上层应用和系统服务能够：

- 枚举系统中的 USB 设备
- 与 USB 设备进行数据交换
- 获取设备信息进行设备识别

### 2. 系统服务支撑

libusb 被设计为 **chipsetsdk_sp**（芯片 SDK 内部 API），意味着它主要用于：

- 硬件抽象层（HAL）
- 设备驱动框架
- 系统级 USB 服务

### 3. 跨平台抽象

通过 libusb 的后端架构，OpenHarmony 能够在不同硬件平台上提供一致的 USB 访问接口，屏蔽底层平台差异。

---

## 版本历史

| 版本 | 发布日期 | 主要变更 |
|------|---------|---------|
| 1.0.28 | 2024-XX-XX | 当前上游稳定版本 |
| 1.0.27 | 2023-XX-XX | 上一稳定版本 |
| 1.0.26 | 2022-XX-XX | 早期稳定版本 |

OpenHarmony 组件版本 3.1 对应上游 libusb 1.0.28。

---

## 相关资源

- **上游项目**: https://github.com/libusb/libusb
- **官方 API 文档**: http://api.libusb.info
- **项目主页**: http://www.libusb.info
- **邮件列表**: mailing-list@libusb.info

---

*文档版本: 1.0*
*最后更新: 2026-02-07*
