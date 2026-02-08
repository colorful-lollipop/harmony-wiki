# 01 原始库简介

## 1.1 库基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | libevdev |
| **上游版本** | 1.13.1 |
| **OH 版本** | 3.1 |
| **许可证** | MIT License |
| **上游地址** | https://gitlab.freedesktop.org/libevdev/libevdev |
| **上游维护者** | Peter Hutterer <peter.hutterer@who-t.net> |

## 1.2 原始功能概述

libevdev 是 Linux 系统中处理 evdev (event device) 输入设备的**用户态包装库**。它的主要设计目标是：

### 核心功能

1. **evdev 设备包装**
   - 提供对 `/dev/input/event*` 设备的封装
   - 解析来自内核的原始 input 事件
   - 管理设备状态和事件队列

2. **类型安全接口**
   - 避免直接使用 ioctl 调用导致的错误
   - 提供清晰的数据结构和 API
   - 自动处理事件序列化和反序列化

3. **uinput 虚拟设备支持**
   - 允许用户空间程序创建虚拟输入设备
   - 支持注入模拟的输入事件
   - 常用于自动化测试和输入模拟

### 典型使用场景

```c
// 典型使用流程
struct libevdev *dev = NULL;
int fd = open("/dev/input/event0", O_RDONLY);

libevdev_init(&dev);
libevdev_set_fd(dev, fd);

// 读取事件
struct input_event ev;
while (libevdev_next_event(dev, LIBEVDEV_READ_FLAG_NORMAL, &ev) == 0) {
    // 处理事件
    if (ev.type == EV_KEY && ev.value == 1) {
        printf("Key pressed: %d\n", ev.code);
    }
}
```

## 1.3 架构设计

```
+------------------+     +------------------+     +------------------+
|   Linux Kernel   | --> |   evdev Driver   | --> |   /dev/input/*   |
+------------------+     +------------------+     +------------------+
                                                         |
                                                         v
+------------------+     +------------------+     +------------------+
|   Application    | <-- |  libevdev API    | <-- |   libevdev.so    |
+------------------+     +------------------+     +------------------+
                                |
                                v
                        +------------------+
                        |   uinput Device  |
                        +------------------+
```

## 1.4 OpenHarmony 中的定位

### 系统角色

在 OpenHarmony 生态系统中，libevdev 扮演着**输入基础设施**的关键角色：

```
OpenHarmony Input System
|
+-- multimodalinput/input
|   |-- libevdev (依赖)
|   |-- uinput 注入
|   |-- 触摸事件处理
|
+-- distributed_input
|   |-- libevdev (依赖)
|   |-- 分布式设备共享
|   |-- 输入事件同步
|
+-- libudev (间接依赖)
    |-- 设备枚举
    |-- 设备热插拔
```

### 与上游的差异

| 方面 | 上游版本 | OpenHarmony 版本 |
|------|---------|------------------|
| **核心功能** | 保持不变 | 保持不变 |
| **安全机制** | 无 | 添加 FDSAN 注解 |
| **Bug 修复** | 部分未修复 | 已修复 |
| **新键值** | 无 | 添加 OH 特有键值 |
| **总线类型** | 基础类型 | 添加 BUS_SDW |

### 为什么选择 libevdev

1. **成熟稳定**：上游维护超过 10 年，经过大规模验证
2. **标准化**：Linux 输入系统的事实标准
3. **类型安全**：避免常见的 C 语言错误
4. **高效**：零拷贝设计，最小化性能开销
5. **社区活跃**：定期更新和问题修复

## 1.5 关键数据结构

### libevdev 核心结构

```c
// 设备状态结构
struct libevdev {
    int fd;                          // 设备文件描述符
    const char *name;                 // 设备名称
    unsigned int id;                  // 设备 ID ( bustype, vendor, product, version )
    
    // 设备能力
    unsigned long evbit[NBITS(EV_MAX)];      // 支持的事件类型
    unsigned long keybit[NBITS(KEY_MAX)];    // 支持的按键
    unsigned long relbit[NBITS(REL_MAX)];   // 支持的相对坐标事件
    unsigned long absbit[NBITS(ABS_MAX)];    // 支持的绝对坐标事件
    
    // 多点触控支持
    int num_slots;                           // 触摸槽数量
    int *slots;                              // 触摸槽状态
};

// 事件结构
struct input_event {
    struct timeval time;    // 时间戳
    __u16 type;              // 事件类型 (EV_SYN, EV_KEY, EV_REL, EV_ABS, etc.)
    __u16 code;              // 事件代码 (键码、轴ID等)
    __s32 value;            // 事件值 (按下/释放、坐标值等)
};
```

## 1.6 官方资源

| 资源 | 链接 |
|------|-----|
| **上游代码** | https://gitlab.freedesktop.org/libevdev/libevdev |
| **API 文档** | http://www.freedesktop.org/software/libevdev/doc/latest/ |
| **邮件列表** | input-tools@lists.freedesktop.org |
| **问题跟踪** | https://gitlab.freedesktop.org/libevdev/libevdev/issues |

## 1.7 下一章

下一章将详细介绍 OpenHarmony 对 libevdev 的 **Patch 适配**，包括所有修改的技术细节和升级建议。

👉 **[02_Patches.md](./02_Patches.md)** →
