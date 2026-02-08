# libinput 原始库简介

> 了解 libinput 的原始功能、版本和在 OpenHarmony 中的定位

---

## 📚 库基本信息

### 基本信息

| 项目 | 内容 |
|------|------|
| **库名称** | libinput |
| **开发者** | freedesktop.org |
| **上游仓库** | https://gitlab.freedesktop.org/libinput/libinput |
| **GitHub 镜像** | https://github.com/wayland-project/libinput |
| **许可证** | MIT License |
| **上游版本** | 1.25.0 |
| **OH 集成版本** | 3.1 |
| **主要语言** | C |
| **构建系统** | Meson (上游), GN (OH) |

### 版本历史

libinput 从 2014 年开始开发，持续演进：

- **早期版本** (1.0-1.7)：基础输入事件处理
- **中期版本** (1.8-1.15)：添加手势识别、触控增强
- **近期版本** (1.16-1.25)：性能优化、新设备支持

**版本 1.25.0 (2024)**：
- 稳定的 API
- 广泛的设备支持
- 成熟的手势识别

---

## 🎯 核心功能

### 输入设备处理

libinput 为以下类型的输入设备提供统一的处理接口：

| 设备类型 | 说明 | 上游支持 |
|---------|------|----------|
| **键盘** | 标准键盘事件处理 | ✅ 原生支持 |
| **鼠标** | 指针移动、点击、滚轮 | ✅ 原生支持 |
| **触摸板** | 多点触控、手势识别、点击 | ✅ 原生支持 |
| **触摸屏** | 多点触控、直接映射 | ✅ 原生支持 |
| **平板** (Tablet) | 数字化板、画板 | ✅ 原生支持 (1.2+) |
| **平板板** (Tablet Pad) | 平板附加按钮、模式切换 | ✅ 原生支持 (1.3+) |
| **轨迹球** | Trackball 设备 | ✅ 原生支持 |
| **指点杆** | ThinkPad TrackPoint 等 | ✅ 原生支持 |
| **开关** | 合盖开关、平板模式开关 | ✅ 原生支持 (1.7+) |
| **游戏手柄** | Gamepad、Joystick | ⚠️ 检测但忽略 |
| **加速度计** | 陀螺仪等传感器 | ✅ 原生支持 |

### 事件处理能力

#### 指针事件
- **Motion**: 相对和绝对移动
- **Button**: 按钮按下和释放
- **Scroll**: 滚轮和滚动条
- **Axis**: 相对和绝对轴事件
- **Acceleration**: 指针加速配置

#### 键盘事件
- **Key**: 按键按下和释放
- **Modifiers**: 修饰键状态（Shift, Ctrl, Alt 等）
- **LED**: 键盘 LED 控制（Caps Lock, Num Lock 等）

#### 触摸事件
- **Down/Up/Motion/Cancel/Frame**: 基础触摸事件
- **压力**: 触摸压力值
- **Major/Minor**: 触摸椭圆信息

#### 平板事件
- **Proximity**: 工具接近检测
- **Pressure/Tilt/Rotation**: 笔角度和压力
- **Distance**: 笔距离

#### 手势事件
- **Swipe**: 滑动手势
- **Pinch**: 捏合手势
- **Hold**: 按住手势

#### 开关事件
- **Toggle**: 开关状态变化
- **类型**: Lid（合盖）、Tablet Mode（平板模式）

### 设备特性系统 (Quirks)

libinput 使用 quirks 系统处理特定硬件的怪癖：

**特性来源**：
- udev 属性
- 设备制造商和型号
- 内核输入能力

**支持的特性**：
- 设备模型识别
- 自定义按钮区域
- 禁用特定功能（如点击、滚动）
- 手势行为调整

---

## 🏗 库架构

### 分层结构

```
┌─────────────────────────────────────────────┐
│          用户应用                         │
│           ↓                              │
│    libinput 公共 API               ←───────┐
│    (libinput.h)                          │
│           ↓                              │
│  libinput 核心 (libinput.c)               │
│           ↓                              │
│  设备抽象层 (evdev.c, tablet.c, etc) │
│           ↓                              │
│  libevdev (内核事件包装)                 │
└─────────────────────────────────────────────┘
```

### 核心模块

| 模块 | 文件 | 功能 |
|------|------|------|
| **Core** | `libinput.c` | 库初始化、设备管理 |
| **Event Dispatch** | `evdev.c` | 事件调度和处理 |
| **Mouse** | `evdev-mouse.c` | 鼠标事件处理 |
| **Touchpad** | `evdev-mt-touchpad.c` | 触摸板和手势 |
| **Touchscreen** | `evdev-touch.c` | 触摸屏处理 |
| **Tablet** | `evdev-tablet.c` | 平板和笔 |
| **Keyboard** | `evdev-keyboard.c` | 键盘事件 |
| **Device quirks** | `quirks.c` | 设备特性管理 |
| **Timer** | `timer.c` | 内部计时器 |

---

## 🔧 构建和依赖

### 上游构建系统

**构建工具**：Meson (现代的构建系统，支持跨平台）

**关键文件**：
- `meson.build` - 主构建配置（34,339 行）
- `meson_options.txt` - 编译选项

**编译选项**：
- 设备支持选择
- 后端选择（libudev, libevdev）
- 特性开关（调试、测试工具）
- 文档生成

### 上游依赖

| 依赖 | 版本 | 用途 |
|------|--------|------|
| **libevdev** | - | 包装内核输入事件 |
| **libudev** | - | 设备热插拔和发现 |
| **mtdev** | 1.1.6 | 多点触控协议 |
| **systemd** | - | 日志系统（可选） |

---

## 📊 API 设计

### 核心对象

#### libinput
主上下文对象，管理所有输入设备。

```c
struct libinput *libinput = libinput_udev_create_context(udev_interface);
libinput_dispatch(libinput);
libinput_unref(libinput);
```

#### libinput_device
表示单个输入设备。

```c
struct libinput_device *device;
while ((device = libinput_get_device(libinput, i++)) {
    // 处理设备
}
```

#### libinput_event
输入事件基类，所有事件类型的共同父类。

```c
struct libinput_event *event = libinput_get_event(libinput);
enum libinput_event_type type = libinput_event_get_type(event);
```

### 事件处理流程

```
1. libinput_ref(libinput)
   ↓
2. libinput_get_device(libinput)
   ↓
3. libinput_dispatch(libinput)
   ↓
4. libinput_get_event(libinput)
   ↓
5. switch (event type) → process event
   ↓
6. libinput_event_unref(event)
   ↓
7. libinput_unref(libinput)
```

---

## 🌐 官方资源

### 文档

- **API 文档**: https://wayland.freedesktop.org/libinput/doc/latest/development.html
- **功能文档**: https://wayland.freedesktop.org/libinput/doc/latest/features.html
- **FAQ**: https://wayland.freedesktop.org/libinput/doc/latest/faqs.html
- **Bug 报告**: https://gitlab.freedesktop.org/libinput/libinput/issues

### 工具

上游提供以下调试工具：

| 工具 | 功能 |
|------|------|
| **libinput-debug-events** | 实时显示输入事件 |
| **libinput-list-devices** | 列出所有输入设备 |
| **libinput-record** | 记录输入事件到文件 |
| **libinput-analyze** | 分析事件记录文件 |
| **libinput-measure** | 测量输入延迟 |
| **libinput-quirks** | 管理设备 quirks |
| **libinput-debug-gui** | GTK GUI 显示触控和指针 |

---

## ⚖️ 设计原则

### 1. 设备抽象
- 统一不同输入设备的 API
- 应用程序不需要关心设备类型差异

### 2. 事件标准化
- 统一的事件格式和语义
- 一致的坐标系统和单位

### 3. 手势识别
- 内置手势识别（滑动、捏合、按住）
- 可配置的手势参数

### 4. 设备特性处理
- quirks 系统处理设备怪癖
- 无需应用程序特殊处理

### 5. 高级输入处理
- 指针加速
- 触摸板点击检测
- 平板压力处理

---

## 📈 发展历程

### 早期阶段 (1.0 - 1.5)
- 基础输入事件处理
- 设备检测和分类
- 指针加速

### 发展阶段 (1.6 - 1.15)
- 手势识别（1.6+）
- 平板支持（1.2+）
- 平板板支持（1.3+）
- 触摸板增强（1.7+）

### 成熟阶段 (1.16 - 1.25)
- 性能优化
- 新设备类型支持
- API 稳定化
- 丰富文档和工具

---

## 🔍 vs 其他输入库

| 特性 | libinput | evdev | tslib |
|------|-----------|--------|--------|
| **设备抽象** | ✅ 完整 | ❌ 无 | ❌ 无 |
| **手势识别** | ✅ 内置 | ❌ 无 | ❌ 无 |
| **设备 quirks** | ✅ 丰富 | ❌ 无 | ⚠️ 有限 |
| **跨平台** | ✅ | ❌ Linux only | ❌ Linux only |
| **高级处理** | ✅ | ❌ | ❌ 无 |
| **API 设计** | ✅ 面向对象 | ⚠️ C 风格 | ⚠️ C 风格 |

**libinput 的优势**：
- 完整的设备抽象
- 内置手势识别
- 丰富的设备 quirks
- 面向对象的 API 设计
- 成熟的文档和工具

---

## 📝 在 OpenHarmony 中的定位

### 核心作用

在 OpenHarmony 中，libinput 是**多模态输入子系统 (MMI)** 的核心组件：

**主要职责**：
1. **输入设备处理**：将内核输入事件转换为统一格式
2. **设备抽象**：为上层提供统一的设备接口
3. **事件标准化**：规范不同设备的事件格式
4. **手势识别**：处理复杂的触摸手势

### 架构位置

```
OH 输入栈:
硬件层 → 内核 → libinput → MMI 适配层 → OH 应用框架
                ↑
            第三方库
```

### 价值

- **统一性**：为不同输入设备提供统一接口
- **可维护性**：成熟的、经过充分测试的库
- **可扩展性**：通过 quirks 系统支持新设备
- **性能**：优化的输入处理管道

---

## 🔗 相关文档

- [02_Patches.md](02_Patches.md) - OH Patch 详细分析
- [03_Build_Integration.md](03_Build_Integration.md) - OH 构建适配
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 中的使用情况
- [05_API_Differences.md](05_API_Differences.md) - API 差异

---

**文档版本**: 1.0
**最后更新**: 2026-02-08
