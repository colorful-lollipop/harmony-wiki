# libusb OpenHarmony Wiki

## 库概览

**libusb** 是一个提供 USB 设备通用访问能力的 C 语言库。在 OpenHarmony 系统中，该库为应用和系统服务提供了与 USB 设备进行通信的标准接口。

| 属性 | 值 |
|------|-----|
| **库名称** | libusb |
| **上游版本** | 1.0.28 |
| **OH 组件版本** | 3.1 |
| **许可证** | LGPL-2.1-only |
| **上游地址** | https://github.com/libusb/libusb.git |
| **所属子系统** | thirdparty |

---

## OpenHarmony 适配概述

libusb 在 OpenHarmony 中的适配工作主要集中在以下几个方面：

### 1. 平台兼容性问题修复

OpenHarmony 的内核版本检测机制与标准 Linux 存在差异。libusb 原始代码通过 `uname` 系统调用获取内核版本，但在 OHOS 系统上，系统名称可能不是标准的 "Linux"。适配 Patch 通过条件编译和默认版本返回，确保库能够正确初始化。

### 2. 安全加固

为防止敏感信息泄露，适配移除了日志输出中的设备路径信息。这些路径可能包含用户标识符、设备序列号等敏感数据，在生产环境中不宜暴露。

### 3. 构建系统集成

libusb 已完整集成到 OpenHarmony 的 GN 构建系统（`BUILD.gn`），支持 Linux/OHOS、macOS 和 Windows 三个平台的编译。构建配置针对 OHOS 环境进行了优化，禁用了 Android 和 udev 相关特性。

---

## 文档导航

### 必读文档

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [SUMMARY.md](SUMMARY.md) | 阅读路线建议 | ⭐⭐⭐ |
| [02_Patches.md](02_Patches.md) | Patch 详细分析 | ⭐⭐⭐ |
| [03_Build_Integration.md](03_Build_Integration.md) | 构建适配说明 | ⭐⭐ |

### 背景信息

| 文档 | 说明 | 优先级 |
|------|------|--------|
| [01_Overview.md](01_Overview.md) | 原始库功能介绍 | ⭐⭐ |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | OH 使用情况分析 | ⭐⭐ |
| [06_Security.md](06_Security.md) | 安全风险分析 | ⭐ |

### 工作文档

| 文档 | 说明 |
|------|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估结果 |
| [_work/NOTES.md](_work/NOTES.md) | 分析过程记录 |
| [_work/PLAN.md](_work/PLAN.md) | 任务进度跟踪 |

---

## 快速开始

### 包含头文件

```c
#include <libusb/libusb.h>
```

### 基础使用示例

```c
libusb_context *ctx = NULL;
int r;

// 初始化 libusb
r = libusb_init(&ctx);
if (r < 0) {
    fprintf(stderr, "Failed to initialize libusb: %s\n", libusb_error_name(r));
    return r;
}

// 获取设备列表
libusb_device **devs;
ssize_t cnt = libusb_get_device_list(ctx, &devs);
if (cnt < 0) {
    fprintf(stderr, "Failed to get device list\n");
    libusb_exit(ctx);
    return (int)cnt;
}

// ... 使用设备列表 ...

// 释放设备列表
libusb_free_device_list(devs, 1);

// 退出 libusb
libusb_exit(ctx);
```

---

## 版本信息

| 版本 | 日期 | 变更说明 |
|------|------|---------|
| 3.1 | 2026-02-07 | OH 适配版本 |
| 1.0.28 | 2024-XX-XX | 上游官方版本 |

---

## 相关资源

- [上游项目地址](https://github.com/libusb/libusb)
- [libusb API 文档](http://api.libusb.info)
- [libusb 官方主页](http://www.libusb.info)

---

*文档版本: 3.1*
*最后更新: 2026-02-07*
