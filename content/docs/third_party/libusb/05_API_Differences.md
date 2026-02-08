# API/接口差异

本文档记录 libusb 在 OpenHarmony 适配过程中的 API 变更情况。

---

## 概述

经过对 libusb 1.0.28 OH 适配版本的全面分析，**未发现公开 API 的修改或扩展**。

所有 Patch 均为内部实现修改，不影响公共接口。

---

## API 变更清单

| 变更类型 | 状态 | 说明 |
|---------|------|------|
| 新增 API | ❌ 无 | 未添加新的公共 API |
| 删除 API | ❌ 无 | 未删除任何公共 API |
| 修改 API | ❌ 无 | 未修改任何公共 API 的签名或行为 |
| 废弃 API | ❌ 无 | 未标记任何 API 为废弃 |

---

## 内部接口变更

### Patch 影响范围

| Patch | 内部文件 | 影响类型 |
|-------|---------|---------|
| hide-log-dev-id.patch | `os/linux_usbfs.c` | 日志输出修改 |
| hide-log-dev-id.patch | `os/sunos_usb.c` | 日志输出修改 |
| hide-log-dev-id.patch | `os/windows_winusb.c` | 日志输出修改 |
| fix-init-fail.patch | `os/linux_usbfs.c` | 条件分支添加 |

**说明**: 以上变更仅影响内部实现细节，不涉及公共 API。

---

## 头文件变更

### 对外头文件

| 头文件 | 状态 | 变更说明 |
|-------|------|---------|
| `libusb.h` | ✅ 无变更 | 公共 API 头文件保持不变 |
| `libusbi.h` | ✅ 无变更 | 内部接口头文件保持不变 |

### OH 特定头文件目录

```
third_party/libusb/
├── libusb/          # 公共头文件目录
│   └── libusb/      # 头文件实际位置
│       ├── libusb.h
│       └── libusbi.h
├── linux/           # OHOS/Linux 平台目录（空）
├── darwin/          # macOS 平台目录（空）
└── windows/         # Windows 平台目录（空）
```

**说明**: `linux/`、`darwin/` 和 `windows/` 目录当前为空，预留用于未来可能的平台特定头文件。

---

## 宏定义变更

### 新增宏

| 宏名称 | 定义位置 | 值/含义 | 用途 |
|-------|---------|--------|------|
| `__OHOS__` | 编译器隐式定义 | - | 标识 OpenHarmony 系统 |

### 修改的宏

| 宏名称 | 原值 | 新值 | 说明 |
|-------|------|------|------|
| `__ANDROID__` | 可能定义 | 已取消定义 (`-U__ANDROID__`) | 禁用 Android 特定代码 |
| `USE_UDEV` | 可能定义 | 已取消定义 (`-UUSE_UDEV`) | 禁用 udev 支持 |
| `PLATFORM_POSIX` | - | 定义 (`-DPLATFORM_POSIX`) | 标识 POSIX 平台 |

---

## 编译配置差异

### 与上游默认配置对比

| 配置项 | 上游默认 | OH 适配版本 | 说明 |
|--------|---------|------------|------|
| `__ANDROID__` | 视平台 | 已禁用 | 避免 Android 特定路径 |
| `USE_UDEV` | 启用 | 已禁用 | OH 不使用 udev |
| 调试级别 | 默认 0 | 可配置 | 通过 `libusb_set_option` |
| 日志输出 | 包含设备路径 | 已隐藏 | 安全加固 |

---

## API 使用兼容性

### 完全兼容的上游 API

libusb 的所有标准 API 在 OH 版本中均保持完全兼容：

```c
// 这些 API 在 OH 版本中与上游完全一致
libusb_init()
libusb_exit()
libusb_get_device_list()
libusb_free_device_list()
libusb_open()
libusb_close()
libusb_claim_interface()
libusb_release_interface()
libusb_control_transfer()
libusb_bulk_transfer()
libusb_interrupt_transfer()
libusb_get_device_descriptor()
libusb_get_config_descriptor()
libusb_hotplug_register_callback()
libusb_handle_events()
libusb_error_name()
libusb_strerror()
```

### 使用示例

```c
// 此代码在 OpenHarmony 和其他平台上的行为完全一致
#include <libusb/libusb.h>

int main() {
    libusb_context *ctx = NULL;
    
    // 初始化 - 行为一致
    int r = libusb_init(&ctx);
    if (r < 0) {
        return r;
    }
    
    // 设置调试级别 - 行为一致
    libusb_set_option(ctx, LIBUSB_OPTION_LOG_LEVEL, 
                      LIBUSB_LOG_LEVEL_INFO);
    
    // 设备枚举 - 行为一致
    libusb_device **devs;
    ssize_t cnt = libusb_get_device_list(ctx, &devs);
    
    // 清理 - 行为一致
    libusb_free_device_list(devs, 1);
    libusb_exit(ctx);
    
    return 0;
}
```

---

## 注意事项

### 1. 内部结构访问

```c
// ❌ 不推荐：直接访问内部结构
#include <libusb/libusbi.h>
struct libusb_device_handle *handle;
// ...

// ✅ 推荐：仅使用公共 API
libusb_device_handle *handle;
libusb_open(device, &handle);
```

### 2. 平台检测

```c
// 使用标准方式检测平台
#ifdef __OHOS__
// OHOS 特定代码
#endif
```

### 3. 未来兼容性

为确保与上游版本的长期兼容性，建议：
- 仅使用公共 API
- 避免依赖内部实现细节
- 不使用未文档化的功能

---

## 结论

**libusb OH 适配版本与上游版本保持 100% 的 API 兼容性。**

所有适配修改均在内部实现层面完成，不影响公共接口的使用。

---

*文档版本: 1.0*
*最后更新: 2026-02-07*
