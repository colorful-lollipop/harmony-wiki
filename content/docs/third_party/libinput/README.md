# libinput - OpenHarmony 输入处理库

> libinput 为 OpenHarmony 多模态输入子系统提供完整的输入设备处理栈

---

## 📋 目录

- [库概览](#库概览)
- [OH 适配概述](#oh-适配概述)
- [文档导航](#文档导航)
- [快速开始](#快速开始)

---

## 库概览

### 基本信息

| 项目 | 内容 |
|------|------|
| **库名称** | libinput |
| **上游版本** | 1.25.0 |
| **OH 组件版本** | 3.1 |
| **许可证** | MIT License |
| **上游地址** | https://gitlab.freedesktop.org/libinput/libinput |
| **GitHub 镜像** | https://github.com/wayland-project/libinput |
| **官方文档** | https://wayland.freedesktop.org/libinput/doc/latest/ |

### 功能描述

libinput 是一个为显示服务器和其他应用程序提供完整输入设备处理栈的库。它提供了设备检测、事件处理和抽象功能，以最小化用户需要提供的自定义输入代码量。

**核心功能**：
- 设备检测和分类
- 输入事件处理和抽象
- 触摸坐标缩放
- 从触摸板生成相对指针事件
- 指针加速配置
- 手势识别（滑动、捏合、按住）
- 设备特性适配（quirks 系统）

### 在 OpenHarmony 中的作用

libinput 是 OpenHarmony **多模态输入子系统 (MMI)** 的核心组件，提供底层的输入事件处理能力。

**主要职责**：
- 处理各种输入设备（鼠标、触摸板、触摸屏、手写笔、游戏手柄等）
- 将内核输入事件转换为统一的 libinput 事件
- 提供设备抽象和事件处理能力
- 支持事件标准化和规范化

**集成方式**：
```
硬件输入 → 内核 evdev → libinput → MMI 适配层 → OH 应用层
```

---

## OH 适配概述

OpenHarmony 对 libinput 进行了显著的定制和扩展，以满足多模态输入子系统的需求。

### 主要定制内容

| 功能领域 | 上游状态 | OH 实现 | 说明 |
|---------|-----------|---------|------|
| **游戏手柄支持** | ⚠️ 检测但忽略 | ✅ 完整支持 | 新增 Joystick 事件和 API |
| **MSDP 设备** | ❌ 不支持 | ✅ 完整支持 | 多源数据处理设备 |
| **隐私开关** | ⚠️ 仅有 lid/tablet_mode | ✅ 扩展支持 | 新增 PRIVACY 开关类型 |
| **触摸板事件** | ⚠️ 通用 POINTER 事件 | ✅ 专用事件 | 独立的 TOUCHPAD 事件 |
| **触控增强** | ✅ 基础支持 | ✅ 增强 | 新增压力、方向、工具类型 |
| **日志系统** | ✅ 原生日志 | ✅ hilog 集成 | 集成 OH 日志系统 |
| **构建系统** | ✅ Meson | ✅ GN | 使用 GN 构建系统 |

### OH 特有功能

#### 1. Joystick (游戏手柄) 支持
- **新增设备能力**：`LIBINPUT_DEVICE_CAP_JOYSTICK`
- **新增事件类型**：
  - `LIBINPUT_EVENT_JOYSTICK_BUTTON` (450)
  - `LIBINPUT_EVENT_JOYSTICK_AXIS`
- **支持 19 种轴类型**：ABS_X, ABS_Y, ABS_Z, ABS_RX, ABS_RY, ABS_RZ, ABS_THROTTLE, ABS_RUDDER, ABS_WHEEL, ABS_GAS, ABS_BRAKE, 4 个 HAT 轴
- **新增源文件**：`evdev-joystick.c/h`

#### 2. MSDP 设备支持
- **设备标签**：`EVDEV_UDEV_TAG_MSDP = 1 << 12`
- **事件类型**：`LIBINPUT_EVENT_MSDP = 1000`
- **检测方式**：通过 `ABS_HAND_FEATURE` (0x27) 轴检测
- **用途**：多源数据处理设备（用于手部检测/触摸手部功能）

#### 3. Privacy Switch (隐私开关) 支持
- **新增开关类型**：`LIBINPUT_SWITCH_PRIVACY`
- **输入开关码**：`SW_SUPER_PRIVACY = 0x11`
- **新增源文件**：`evdev-privacy-switch.c/h`
- **用途**：摄像头/麦克风等隐私硬件的物理开关控制

#### 4. Touchpad (触摸板) 事件独立化
- **新增触摸板专用事件**：
  - `LIBINPUT_EVENT_TOUCHPAD_DOWN` (550)
  - `LIBINPUT_EVENT_TOUCHPAD_UP`
  - `LIBINPUT_EVENT_TOUCHPAD_MOTION`
  - `LIBINPUT_EVENT_TOUCHPAD_ACTIVE` (580)
- **新增触摸板指针事件**：
  - `LIBINPUT_EVENT_POINTER_TAP`
  - `LIBINPUT_EVENT_POINTER_MOTION_TOUCHPAD`
  - `LIBINPUT_EVENT_POINTER_BUTTON_TOUCHPAD`
  - `LIBINPUT_EVENT_POINTER_SCROLL_FINGER_BEGIN/END`
- **目的**：将触摸板事件从通用指针事件中分离，提供更精细的控制

#### 5. 增强的 Touch 属性
- **新增触控属性**：
  - `libinput_event_touch_get_orientation()` - 触摸方向
  - `libinput_event_touch_get_tool_type()` - 工具类型（手指、笔等）
  - `libinput_event_touch_get_tool_x/y()` - 工具中心坐标
  - `libinput_event_touch_get_tool_width/height()` - 工具尺寸
  - Touch pressure support - 触摸压力

### 构建系统适配

**上游构建**：Meson (`meson.build`)
**OH 构建**：GN (`BUILD.gn`)

**OH 特性**：
- 使用 `apply_patch.sh` 在构建时应用 11440 行的 patch
- 生成到 `$root_out_dir/diff_libinput_mmi`
- 添加安全编译选项：CFI、PAC_RET
- 集成 hilog 日志系统
- 添加笔设备特性开关：`libinput_feature_pen` → `OHOS_BUILD_ENABLE_PEN`

---

## 文档导航

### 核心文档

| 文档 | 内容 | 阅读时长 |
|------|------|-----------|
| [01_Overview.md](01_Overview.md) | 原始库简介、在 OH 中的作用 | 5 分钟 |
| [02_Patches.md](02_Patches.md) | **Patch 详细分析**（核心文档） | 30 分钟 |
| [03_Build_Integration.md](03_Build_Integration.md) | OH 构建适配、编译选项 | 15 分钟 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系、使用场景 | 20 分钟 |
| [05_API_Differences.md](05_API_Differences.md) | API 差异、新增接口 | 25 分钟 |

### 工作文档

| 文档 | 内容 |
|------|------|
| [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 项目评估结果、Patch 清单 |
| [_work/NOTES.md](_work/NOTES.md) | 分析过程记录、技术笔记 |
| [_work/PLAN.md](_work/PLAN.md) | 任务进度、工作计划 |

---

## 快速开始

### 了解 OH 定制内容

1. **从 Patch 分析开始**：
   - 阅读 [02_Patches.md](02_Patches.md) 了解所有 OH 定制
   - 重点关注 Joystick、MSDP、Touchpad 事件增强

2. **理解构建系统**：
   - 阅读 [03_Build_Integration.md](03_Build_Integration.md) 了解 OH 如何构建 libinput
   - 了解 patch 应用机制和安全编译选项

3. **查看使用情况**：
   - 阅读 [04_Usage_in_OH.md](04_Usage_in_OH.md) 了解哪些模块使用 libinput
   - 查看依赖关系图理解架构

### 使用 OH 特有功能

#### 使用 Joystick 支持

```c
#include <libinput.h>

// 检查设备是否支持 Joystick
if (libinput_device_has_capability(device, LIBINPUT_DEVICE_CAP_JOYSTICK)) {
    printf("Device supports joystick\n");
}

// 处理 Joystick 事件
switch (libinput_event_get_type(event)) {
    case LIBINPUT_EVENT_JOYSTICK_BUTTON:
        // 处理按钮事件
        break;
    case LIBINPUT_EVENT_JOYSTICK_AXIS:
        // 处理轴事件
        break;
}
```

#### 使用 Touchpad 专用事件

```c
// 检查触摸板激活状态
switch (libinput_event_get_type(event)) {
    case LIBINPUT_EVENT_TOUCHPAD_ACTIVE:
        printf("Touchpad is active\n");
        break;
    case LIBINPUT_EVENT_TOUCHPAD_DOWN:
        printf("Touchpad down: %f, %f\n", x, y);
        break;
}
```

#### 使用隐私开关

```c
// 监听隐私开关
if (libinput_event_get_type(event) == LIBINPUT_EVENT_SWITCH_TOGGLE) {
    enum libinput_switch sw = libinput_event_switch_get_switch(event);
    enum libinput_switch_state state = libinput_event_switch_get_state(event);
    
    if (sw == LIBINPUT_SWITCH_PRIVACY) {
        if (state == LIBINPUT_SWITCH_STATE_ON) {
            printf("Privacy is ON\n");
        } else {
            printf("Privacy is OFF\n");
        }
    }
}
```

---

## 依赖关系

### 外部依赖

| 依赖 | 版本 | 用途 |
|------|--------|------|
| **libevdev** | - | 输入事件库（evdev 事件处理） |
| **mtdev** | 1.1.6 | 多点触控库 |
| **hilog** | - | OH 日志系统 |
| **input:mmi_libudev** | - | udev 设备管理 |

### OH 内部依赖者

**主要依赖者**：
- `foundation/multimodalinput/input/service` (MMI 核心服务，50+ 依赖)
- `foundation/multimodalinput/input/service/mouse_event_normalize`
- `foundation/multimodalinput/input/service/joystick`
- `foundation/multimodalinput/input/service/touch_event_normalize`

**总计**：338 个 BUILD.gn 文件引用 libinput，280 个文件依赖 `libinput-third-mmi`

详细依赖关系见 [04_Usage_in_OH.md](04_Usage_in_OH.md)

---

## 编译和构建

### 构建 libinput

```bash
# 在 OH 根目录执行
./build.sh --product-name <product> --ccache

# 输出：
# $root_out_dir/diff_libinput_mmi/
#   ├── src/
#   ├── include/
#   ├── export_include/
#   └── hm_src/
```

### 启用笔设备支持

```bash
# 在 GN args 中添加
libinput_feature_pen = true

# 这会定义 OHOS_BUILD_ENABLE_PEN 宏
```

### 生成的产物

**共享库**：
- `libinput-third-mmi.so` - 主要输入处理库

**工具程序**：
- `libinput-debug-mmi` - 事件调试工具
- `libinput-list-mmi` - 列出设备工具
- `libinput-tablet-mmi` - 平板调试工具
- `libinput-record-mmi` - 事件记录工具
- `libinput-analyze-mmi` - 分析工具
- `libinput-measure-mmi` - 测量工具
- `libinput-quirks-mmi` - 设备特性工具

**配置文件**：
- `/sys_prod/etc/libinput/quirks/` - 设备特性配置文件（31 个）

---

## 相关资源

### 官方资源
- [libinput 官方文档](https://wayland.freedesktop.org/libinput/doc/latest/)
- [libinput GitLab](https://gitlab.freedesktop.org/libinput/libinput)
- [libinput API 文档](https://wayland.freedesktop.org/libinput/doc/latest/development.html)

### OpenHarmony 资源
- [OpenHarmony 输入子系统](https://gitee.com/openharmony/multimodalinput_input)
- [OH 开发者文档](https://docs.openharmony.cn/)

---

**文档版本**: 1.0
**最后更新**: 2026-02-08
