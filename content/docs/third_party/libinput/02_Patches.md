# libinput Patch 详细分析

> OpenHarmony 对 libinput 的完整定制分析

---

## 📚 文档说明

**Patch 文件**：`patch/diff_libinput_mmi/libinput/libinput_0000.diff`
- **大小**: 343,571 字节 (~336 KB)
- **行数**: 11,440 行
- **修改文件**: 37 个
- **修改类型**: 新增文件 + 修改文件

**分析依据**：
- Patch diff 代码分析
- 上游 libinput 1.25.0 对比
- BUILD.gn 配置和依赖关系

---

## 📋 Patch 清单

| # | Patch 分类 | 文件数量 | 主要内容 |
|---|----------|----------|---------|
| 1 | 新增源文件 | 7 | Joystick、Privacy Switch、hm_missing、libinput-util |
| 2 | 公共 API 修改 | 1 | export_include/libinput.h（大量新增） |
| 3 | 配置和头文件 | 4 | config.h、input-event-codes.h、evdev 头文件 |
| 4 | 触摸板增强 | 10 | mt-touchpad 相关文件 |
| 5 | 核心实现修改 | 8 | evdev.c、libinput.c、timer.c 等 |
| 6 | 工具和调试 | 2 | debug-events、record 工具 |

---

## 1️⃣ 新增源文件（7 个）

### 1.1 游戏手柄支持

| 文件 | 大小 | 用途 |
|------|------|------|
| `src/evdev-joystick.c` | 新增 | Joystick 事件处理实现 |
| `src/evdev-joystick.h` | 新增 | Joystick 事件处理声明 |

**新增内容**：
```c
// 调度器结构
struct joystick_dispatch {
    struct evdev_dispatch base;
    struct libinput_event_joystick_axis axis_event;
    struct libinput_event_joystick_button button_event;
    // 19 个轴信息结构
    struct libinput_event_joystick_axis_abs_info abs_throttle;
    struct libinput_event_joystick_axis_abs_info abs_hat0x;
    struct libinput_event_joystick_axis_abs_info abs_hat0y;
    // ... 其他轴
};

// API 函数
struct evdev_dispatch* evdev_joystick_create(struct evdev_device *device);
void joystick_process_event(struct evdev_device *device, struct input_event *event, uint64_t time);
```

**关键特性**：
- **19 种轴类型**：ABS_X, ABS_Y, ABS_Z, ABS_RX, ABS_RY, ABS_RZ, ABS_THROTTLE, ABS_RUDDER, ABS_WHEEL, ABS_GAS, ABS_BRAKE, 4 个 HAT 轴
- **按钮事件**：`LIBINPUT_EVENT_JOYSTICK_BUTTON` (450)
- **轴事件**：`LIBINPUT_EVENT_JOYSTICK_AXIS`
- **轴源枚举**：19 个轴源类型（见 export_include/libinput.h）

**OH 需求**：
- 支持游戏手柄输入设备（如游戏手柄、摇杆）
- 提供完整的轴和按钮事件
- 为游戏应用提供标准的输入 API

**升级建议**：
- **OH 特有功能**，上游明确不支持 joystick
- 升级上游版本时需重新适配所有 joystick 代码
- 可考虑将 joystick 支持作为可选功能推向上游（上游目前是检测后忽略）

---

### 1.2 隐私开关支持

| 文件 | 大小 | 用途 |
|------|------|------|
| `src/evdev-privacy-switch.c` | 新增 | 隐私开关设备处理 |
| `src/evdev-privacy-switch.h` | 新增 | 隐私开关设备声明 |

**新增内容**：
```c
// 隐私开关结构
struct privacy_switch {
    struct evdev_dispatch base;
    struct libinput_event_switch event;
    enum libinput_switch sw;
};

// API 函数
struct evdev_dispatch* evdev_privacy_switch_create(struct evdev_device *device);
void privacy_switch_process_event(struct evdev_device *device, struct input_event *event, uint64_t time);
```

**关键特性**：
- **开关类型**：`LIBINPUT_SWITCH_PRIVACY`
- **输入开关码**：`SW_SUPER_PRIVACY = 0x11`
- **事件类型**：`LIBINPUT_EVENT_SWITCH_TOGGLE`

**OH 需求**：
- 支持摄像头、麦克风等隐私硬件的物理开关
- 提供标准化的隐私开关事件
- 允许上层应用监听隐私状态变化

**升级建议**：
- 合理的扩展，可以推向上游
- 上游已有 Switch 功能，仅需添加 PRIVACY 类型
- 升级风险低，与现有 Switch 功能兼容

---

### 1.3 OH 特有实现模块

| 文件 | 大小 | 用途 |
|------|------|------|
| `hm_src/hm_missing.c` | 新增 | OH 特有实现函数 |
| `hm_src/hm_missing.h` | 新增 | OH 特有函数声明 |
| `src/libinput-util.c` | 新增 | 工具函数实现 |

**hm_missing 模块**：
```c
#ifndef OHOS_LIBINPUT_HM_MISSING_H
#define OHOS_LIBINPUT_HM_MISSING_H

// 程序名称替换（OHOS 环境适配）
char *ohos_program_invocation_short_name() {
    return __progname;
}
#define program_invocation_short_name ohos_program_invocation_short_name()

// 其他 OH 特有实现
// ...
#endif // OHOS_LIBINPUT_HM_MISSING_H
```

**目的**：
- 提供上游 libinput 缺失的函数
- 适配 OHOS 环境
- 提供兼容性层

**OH 需求**：
- 适配 OHOS 的系统调用和库
- 提供 OH 特有的输入处理逻辑

**升级建议**：
- **OH 特有模块**，无法推向上游
- 升级上游版本时需保留此模块
- 可能需要更新以适配新的上游代码

---

## 2️⃣ 公共 API 修改

### 2.1 export_include/libinput.h

**修改类型**：重大新增和扩展

#### 新增设备能力

```c
enum libinput_device_capability {
    LIBINPUT_DEVICE_CAP_KEYBOARD = 1,
    LIBINPUT_DEVICE_CAP_POINTER = 2,
    LIBINPUT_DEVICE_CAP_TOUCH = 3,
    LIBINPUT_DEVICE_CAP_TABLET_TOOL = 4,
    LIBINPUT_DEVICE_CAP_TABLET_PAD = 5,
    LIBINPUT_DEVICE_CAP_GESTURE = 6,
    LIBINPUT_DEVICE_CAP_SWITCH = 6,
    LIBINPUT_DEVICE_CAP_JOYSTICK = 7,  // OH 新增
};
```

#### 新增 udev 标签

```c
enum evdev_device_udev_tags {
    EVDEV_UDEV_TAG_INPUT = 1 << 0,
    EVDEV_UDEV_TAG_KEYBOARD = 1 << 1,
    EVDEV_UDEV_TAG_MOUSE = 1 << 2,
    EVDEV_UDEV_TAG_TOUCHPAD = 1 << 3,
    EVDEV_UDEV_TAG_TOUCHSCREEN = 1 << 4,
    EVDEV_UDEV_TAG_TABLET = 1 << 5,
    EVDEV_UDEV_TAG_JOYSTICK = 1 << 6,
    EVDEV_UDEV_TAG_ACCELEROMETER = 1 << 7,
    EVDEV_UDEV_TAG_TABLET_PAD = 1 << 8,
    EVDEV_UDEV_TAG_POINTINGSTICK = 1 << 9,
    EVDEV_UDEV_TAG_TRACKBALL = 1 << 10,
    EVDEV_UDEV_TAG_SWITCH = 1 << 11,
    EVDEV_UDEV_TAG_MSDP = 1 << 12,  // OH 新增
};
```

#### 新增事件类型

**触摸板专用事件**：
```c
enum libinput_event_type {
    // ... 原有事件

    // 指针事件增强
    LIBINPUT_EVENT_POINTER_TAP,
    LIBINPUT_EVENT_POINTER_MOTION_TOUCHPAD,
    LIBINPUT_EVENT_POINTER_BUTTON_TOUCHPAD,
    LIBINPUT_EVENT_POINTER_SCROLL_FINGER_BEGIN,
    LIBINPUT_EVENT_POINTER_SCROLL_FINGER_END,

    // Joystick 事件（OH 新增）
    LIBINPUT_EVENT_JOYSTICK_BUTTON = 450,
    LIBINPUT_EVENT_JOYSTICK_AXIS,

    // 触摸板事件（OH 新增）
    LIBINPUT_EVENT_TOUCHPAD_DOWN = 550,
    LIBINPUT_EVENT_TOUCHPAD_UP,
    LIBINPUT_EVENT_TOUCHPAD_MOTION,

    LIBINPUT_EVENT_TOUCHPAD_ACTIVE = 580,

    // MSDP 事件（OH 新增）
    LIBINPUT_EVENT_MSDP = 1000,
};
```

#### 新增开关类型

```c
enum libinput_switch {
    LIBINPUT_SWITCH_LID = 0,
    LIBINPUT_SWITCH_TABLET_MODE,
    LIBINPUT_SWITCH_PRIVACY,  // OH 新增
};
```

#### 新增数据结构

**槽位坐标**：
```c
#define MAX_SOLTED_COORDS_NUM 10

struct sloted_coords {
    int32_t is_active;
    float x;
    float y;
};

struct sloted_coords_info {
    struct sloted_coords coords[MAX_SOLTED_COORDS_NUM];
    unsigned int active_count;
};
```

**Joystick 轴信息**：
```c
struct libinput_event_joystick_axis_abs_info {
    int32_t code;
    int32_t value;
    int32_t minimum;
    int32_t maximum;
    int32_t fuzz;
    int32_t flat;
    int32_t resolution;
    float   standardValue;
};
```

#### 新增 API 函数

**触摸板事件**：
```c
struct libinput_event_touch *
libinput_event_get_touchpad_event(struct libinput_event *event);
```

**虚拟触摸板**：
```c
double
libinput_event_vtrackpad_get_dx_unaccelerated(struct libinput_event_pointer *event);
double
libinput_event_vtrackpad_get_dy_unaccelerated(struct libinput_event_pointer *event);
```

**按钮区域**：
```c
uint32_t
libinput_event_pointer_get_button_area(struct libinput_event_pointer *event);
```

**Joystick 事件**：
```c
struct libinput_event_joystick_button*
libinput_event_get_joystick_button_event(struct libinput_event *event);

struct libinput_event_joystick_axis*
libinput_event_get_joystick_axis_event(struct libinput_event *event);

struct libinput_event_joystick_axis_abs_info *
libinput_event_joystick_axis_get_abs_info(struct libinput_event_joystick_axis *event, enum libinput_joystick_axis_source source);
```

**槽位坐标**：
```c
struct sloted_coords_info *
libinput_event_get_solt_touches(struct libinput_event* event);
```

---

## 3️⃣ 配置和头文件修改

### 3.1 input-event-codes.h

**修改内容**：新增 OH 特有的输入事件码

```c
// 新增 KEY 码
#define KEY_MICMUTE                    251   // 麦克风静音
#define KEY_MOUSE_ASSISTANT           0x2e9  // 鼠标助手
#define KEY_MOUSE_INTELLIGENCE_SELECTION 0x2ea  // 鼠标智能选择
#define KEY_AOD_SINGLE_CLICK            0x2fd  // AOD 单击

// 新增 ABS 码
#define ABS_HAND_FEATURE                0x27   // 手势特性
#define ABS_MT_MOVEFLAG                 0x29   // 多触控移动标志
#define ABS_MT_TWIST                    0x2c   // 多触控旋转

// 新增 SW 码
#define SW_SUPER_PRIVACY                0x11   // 超级隐私开关
```

**目的**：
- 支持特定的 OH 硬件按键和功能
- 支持手部检测和手势识别
- 支持隐私开关硬件

**OH 需求**：
- 支持 OH 特有的输入设备功能
- 兼容 OH 硬件设计

**升级建议**：
- OH 特有事件码，无法推向上游
- 需确认这些是否是 Linux 内核标准事件码
- 升级时需确保事件码不冲突

---

### 3.2 quirks 路径修改

**修改内容**：quirks 配置文件路径

```c
// 原路径
#define LIBINPUT_QUIRKS_DIR "/etc/libinput/quirks"
#define LIBINPUT_QUIRKS_OVERRIDE_FILE "/etc/libinput/quirks/local-overrides.quirks"
#define LIBINPUT_QUIRKS_SRCDIR "/etc/libinput/quirks"

// OH 路径
#define LIBINPUT_QUIRKS_DIR "/sys_prod/etc/libinput/quirks"
#define LIBINPUT_QUIRKS_OVERRIDE_FILE "/sys_prod/etc/libinput/quirks/local-overrides.quirks"
#define LIBINPUT_QUIRKS_SRCDIR "/sys_prod/etc/libinput/quirks"
```

**目的**：
- 适配 OH 的生产系统目录结构
- quirks 文件部署到 `/sys_prod/etc`

**OH 需求**：
- 符合 OH 的系统目录规范
- 支持生产环境的配置文件部署

**升级建议**：
- **构建系统适配**，不影响上游代码
- 上游版本可正常使用，仅需修改构建配置
- 风险低

---

## 4️⃣ 触摸板增强（10 个文件）

### 4.1 修改的文件

| 文件 | 修改类型 |
|------|---------|
| `src/evdev-mt-touchpad.c` | 增强触摸板核心 |
| `src/evdev-mt-touchpad.h` | 添加新结构和声明 |
| `src/evdev-mt-touchpad-buttons.c` | 按钮区域处理 |
| `src/evdev-mt-touchpad-gestures.c` | 手势识别增强 |
| `src/evdev-mt-touchpad-tap.c` | 点击检测增强 |
| `src/evdev-mt-touchpad-thumb.c` | 拇指检测 |

### 4.2 关键增强

#### 触摸板事件独立化

**新增事件类型**：
- `LIBINPUT_EVENT_TOUCHPAD_DOWN` (550)
- `LIBINPUT_EVENT_TOUCHPAD_UP`
- `LIBINPUT_EVENT_TOUCHPAD_MOTION`
- `LIBINPUT_EVENT_TOUCHPAD_ACTIVE` (580)

**新增 API**：
```c
struct libinput_event_touch *
libinput_event_get_touchpad_event(struct libinput_event *event);
```

**目的**：
- 将触摸板事件从通用的 POINTER 事件中分离
- 提供更精细的触摸板控制
- 区分触摸板和普通鼠标事件

**OH 需求**：
- 支持独立的触摸板事件流
- 为 MMI 提供更好的触摸板抽象

**升级建议**：
- API 增强可以推向上游
- 上游可能考虑类似的分离（已有部分工作）
- 升级风险中等，需重新测试触摸板功能

#### 触摸板增强属性

**新增特性**：
- 槽位坐标信息 (`struct sloted_coords_info`)
- 触摸板频率信息
- 按钮区域配置
- 手势超时配置
- 手指移动距离阈值

**新增 quirks 属性**（约 15+ 个）：
```c
AttrDwtPointerUnlockTimeThreshold
AttrTouchpadAxisSpeedGain
AttrTouchpadFrequency
AttrTouchpadHoldAndMotionThreshold
AttrTouchpadGestureSwitchTimeout
AttrTouchpadMinMoveDistanceThreshold
AttrTouchpadFourFingerDistanceScale
AttrTouchpadFingerMovedThreshold
QUIRK_MODEL_HUAWEI_FREETOUCH_TOUCHPAD
// ... 更多
```

**目的**：
- 支持华为/ OH 特有的触摸板硬件
- 提供更灵活的手势识别配置
- 优化触摸板性能和精度

**OH 需求**：
- 适配 OH 设备的触摸板特性
- 支持复杂的手势场景

**升级建议**：
- 部分属性可以推向上游（通用的手势参数）
- 特定硬件的 quirks 为 OH 特有
- 升级时需保留 OH 特性

---

## 5️⃣ 核心实现修改（8 个文件）

### 5.1 修改的文件

| 文件 | 修改类型 |
|------|---------|
| `src/evdev.c` | 重大修改：MSDP、Joystick、调度增强 |
| `src/libinput.c` | API 实现：新增事件类型处理 |
| `src/evdev.h` | 结构体：新增 dispatch 类型 |
| `src/libinput-private.h` | 内部 API：MSDP、Joystick 声明 |
| `src/libinput-util.h` | 工具函数声明 |
| `src/quirks.c` | quirks 属性实现 |
| `src/quirks.h` | quirks 属性声明 |
| `src/timer.c` | 定时器实现 |

### 5.2 evdev.c 关键修改

#### MSDP 设备支持

```c
// MSDP 设备检测
static inline bool is_touchhand(const struct libevdev *evdev)
{
    return (libevdev_has_event_type(evdev, EV_ABS) &&
            libevdev_has_event_code(evdev, EV_ABS, ABS_HAND_FEATURE));
}

// MSDP 事件处理
if (udev_tags & EVDEV_UDEV_TAG_MSDP) {
    device->seat_caps |= EVDEV_DEVICE_MSDP;
    // ...
}

if (device->seat_caps & EVDEV_DEVICE_MSDP) {
    fallback_flush_msdp_motion(dispatch, time);
    touch_notify_msdp(device, time);
}
```

#### Joystick 设备支持

```c
// Joystick 设备能力
if (udev_tags & EVDEV_UDEV_TAG_JOYSTICK) {
    device->tags |= EVDEV_TAG_JOYSTICK;
}

// Joystick 调度器创建
case DISPATCH_JOYSTICK:
    dispatch = evdev_joystick_create(device);
    break;
```

#### Privacy Switch 支持

```c
// Privacy Switch 检测
if (udev_tags & EVDEV_UDEV_TAG_SWITCH &&
    libevdev_has_event_code(evdev, EV_SW, SW_SUPER_PRIVACY)) {
    device->tags |= EVDEV_TAG_PRIVACY_SWITCH;
}

// Privacy Switch 调度器创建
case DISPATCH_PRIVACY_SWITCH:
    pSwitchDispatch->ev_switch.sw = LIBINPUT_SWITCH_PRIVACY;
    break;
```

**OH 需求**：
- 支持 MSDP、Joystick、Privacy Switch 设备
- 集成新的调度器类型
- 扩展设备能力标签

**升级建议**：
- **核心修改**，升级上游版本时需重新适配
- 新调度器类型可能与上游冲突
- 需要仔细测试所有新增设备类型

---

## 6️⃣ 工具和调试修改（2 个文件）

### 6.1 修改的文件

| 文件 | 修改内容 |
|------|---------|
| `tools/libinput-debug-events.c` | 新增 MSDP、Joystick 事件调试输出 |
| `tools/libinput-record.c` | 新增事件记录支持 |

### 6.2 关键修改

#### 调试输出增强

```c
// MSDP 事件调试
case LIBINPUT_EVENT_MSDP:
    printf("MSDP event\n");
    break;

// Joystick 事件调试
case LIBINPUT_EVENT_JOYSTICK_BUTTON:
    printf("Joystick button\n");
    break;
case LIBINPUT_EVENT_JOYSTICK_AXIS:
    printf("Joystick axis\n");
    break;
```

**目的**：
- 支持调试新增的设备类型
- 提供完整的事件诊断能力

**OH 需求**：
- 调试工具支持所有 OH 设备类型
- 提供详细的日志输出

**升级建议**：
- 调试工具的兼容性修改
- 升级上游版本时需同步调试工具
- 风险低，不影响核心功能

---

## 📊 Patch 分类总结

### 按功能分类

| 功能 | 修改文件 | 上游兼容 | OH 需求 |
|------|----------|-----------|---------|
| **Joystick 支持** | 2 (新文件) + 2 (修改) | ❌ 上游不支持 | ✅ 游戏手柄 |
| **MSDP 支持** | 3 (修改) | ❌ 上游不支持 | ✅ 多源数据设备 |
| **Privacy Switch** | 2 (新文件) + 2 (修改) | ⚠️ 部分支持 | ✅ 隐私硬件 |
| **Touchpad 增强** | 6 (修改) | ⚠️ 部分支持 | ✅ 精细化控制 |
| **事件码扩展** | 2 (修改) | ❌ 无 | ✅ OH 硬件 |
| **配置适配** | 2 (修改) | ⚠️ 路径差异 | ✅ OH 系统适配 |
| **工具增强** | 2 (修改) | ⚠️ 部分支持 | ✅ 完整调试 |

### 按修改性质分类

| 修改性质 | 数量 | 升级难度 |
|---------|------|----------|
| **新增文件** | 7 | 🔶 高（需保留） |
| **修改文件** | 30 | 🔶 高（需重新适配） |
| **新增 API** | ~20 | 🔶 中（API 扩展） |
| **新增数据结构** | ~10 | 🔶 中（结构扩展） |
| **新增枚举/常量** | ~40 | 🔶 中（常量添加） |

---

## 🚀 升级建议

### 升级上游版本时的注意事项

#### 1. 保留 OH 特有功能

| 功能 | 保留策略 |
|------|---------|
| **Joystick 支持** | 重新移植所有 joystick 代码，保留 `evdev-joystick.c/h` |
| **MSDP 支持** | 重新移植 MSDP 检测和事件处理 |
| **Privacy Switch** | 检查上游是否已添加，如未添加则保留 |
| **hm_missing 模块** | 始终保留，更新以适配新版本 |
| **OH 事件码** | 确认上游是否支持，如不支持则保留 |
| **quirks 路径** | 修改构建配置，使用 OH 路径 |

#### 2. API 扩展优先级

| API | 推向上流优先级 |
|------|--------------|
| **Privacy Switch** | ✅ 高（合理扩展） |
| **Touchpad 专用事件** | ✅ 中（API 增强） |
| **Joystick 支持** | 🔶 中（上游故意不支持） |
| **MSDP 设备** | ❌ 低（OH 特有） |
| **增强的 Touch 属性** | ✅ 高（通用价值） |

#### 3. 冲突检查清单

升级前需检查：

- [ ] 新增的调度器类型（`DISPATCH_JOYSTICK`, `DISPATCH_PRIVACY_SWITCH`）是否与上游冲突
- [ ] 新增的事件类型 ID 是否冲突（Joystick: 450, MSDP: 1000）
- [ ] 新增的 udev 标签（`EVDEV_UDEV_TAG_MSDP`）是否冲突
- [ ] quirks 属性是否与上游现有属性冲突
- [ ] hm_missing 模块是否需要更新
- [ ] 日志宏（`HAVE_LIBINPUT_LOG_ENABLE`）是否仍需保留

#### 4. 回归测试要点

升级后必须测试：

- [ ] Joystick 设备的按钮和轴事件
- [ ] MSDP 设备的事件处理
- [ ] Privacy Switch 的状态变化
- [ ] 触摸板的专用事件（DOWN/UP/MOTION/ACTIVE）
- [ ] 所有原有设备类型（键盘、鼠标、触摸屏、平板）
- [ ] 手势识别（滑动、捏合、按住）
- [ ] 性能（延迟、CPU 使用率）

---

## 📝 Patch 价值评估

### OH 特定价值

| 定制 | 业务价值 | 技术复杂度 |
|------|----------|------------|
| **Joystick 支持** | ⭐⭐⭐⭐⭐ 游戏应用 | ⭐⭐⭐ 中等 |
| **MSDP 设备** | ⭐⭐⭐⭐ 手势识别 | ⭐⭐⭐⭐⭐ 高 |
| **Privacy Switch** | ⭐⭐⭐⭐ 隐私保护 | ⭐⭐ 低 |
| **Touchpad 增强** | ⭐⭐⭐⭐⭐ 输入精度 | ⭐⭐⭐ 中等 |
| **事件码扩展** | ⭐⭐ OH 硬件兼容 | ⭐ 低 |

### 推向上流潜力

| 功能 | 推向上流可能性 | 理由 |
|------|--------------|------|
| **Privacy Switch** | ✅ 高 | 合理的扩展，符合通用需求 |
| **增强的 Touch 属性** | ✅ 高 | pressure、orientation 等有通用价值 |
| **Touchpad 专用事件** | ✅ 中 | 需讨论是否对上游有价值 |
| **Joystick 支持** | 🔶 中 | 上游目前是检测后忽略，但可能作为可选功能提供 |
| **MSDP 设备** | ❌ 低 | OH 特有设备类型，通用性待评估 |

---

**文档版本**: 1.0
**最后更新**: 2026-02-08
