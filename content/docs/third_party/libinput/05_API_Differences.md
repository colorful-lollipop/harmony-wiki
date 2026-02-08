# libinput API 差异

> OpenHarmony 对 libinput 的 API 扩展和新增接口

---

## 📚 文档说明

本文档详细列出了 OpenHarmony 对 libinput 进行的 API 修改和新增，包括：
- 新增的数据结构
- 新增的 API 函数
- 新增的枚举和常量
- OH 特有的使用示例

---

## 🆕 新增设备能力

### LIBINPUT_DEVICE_CAP_JOYSTICK

**定义**：
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

**用途**：检测设备是否支持游戏手柄/游戏摇杆

**使用示例**：
```c
struct libinput_device *device;
while ((device = libinput_get_device(libinput, i++))) {
    if (libinput_device_has_capability(device, LIBINPUT_DEVICE_CAP_JOYSTICK)) {
        printf("Device %d supports joystick\n", i);
    }
}
```

**OH 需求**：支持游戏手柄输入设备（如游戏手柄、摇杆）

---

## 🆕 新增 udev 标签

### EVDEV_UDEV_TAG_MSDP

**定义**：
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

**用途**：识别 MSDP（Multi-Source Data Processing）设备类型

**使用示例**：
```c
enum evdev_device_udev_tags tags = device->tags;

if (tags & EVDEV_UDEV_TAG_MSDP) {
    printf("Device is MSDP type\n");
}
```

**OH 需求**：支持多源数据处理设备（用于手部检测/触摸手部功能）

---

## 🆕 新增事件类型

### 触摸板专用事件

#### LIBINPUT_EVENT_TOUCHPAD_DOWN / UP / MOTION

**定义**：
```c
enum libinput_event_type {
    // ... 原有事件

    LIBINPUT_EVENT_TOUCHPAD_DOWN = 550,  // OH 新增
    LIBINPUT_EVENT_TOUCHPAD_UP,            // OH 新增
    LIBINPUT_EVENT_TOUCHPAD_MOTION,        // OH 新增

    LIBINPUT_EVENT_TOUCHPAD_ACTIVE = 580,  // OH 新增
};
```

**用途**：独立处理触摸板的下、上、移动事件，从通用的触摸事件中分离

**使用示例**：
```c
struct libinput_event *event;
while ((event = libinput_get_event(libinput))) {
    switch (libinput_event_get_type(event)) {
        case LIBINPUT_EVENT_TOUCHPAD_DOWN:
            printf("Touchpad down\n");
            break;

        case LIBINPUT_EVENT_TOUCHPAD_MOTION:
            struct libinput_event_touch *tp =
                libinput_event_get_touchpad_event(event);
            printf("Touchpad motion: %f, %f\n",
                   libinput_event_touch_get_x(tp),
                   libinput_event_touch_get_y(tp));
            break;

        case LIBINPUT_EVENT_TOUCHPAD_ACTIVE:
            printf("Touchpad is active\n");
            break;
    }
}
```

**新增 API**：
```c
// 获取触摸板事件
struct libinput_event_touch *
libinput_event_get_touchpad_event(struct libinput_event *event);
```

---

### Joystick 事件

#### LIBINPUT_EVENT_JOYSTICK_BUTTON

**定义**：
```c
enum libinput_event_type {
    LIBINPUT_EVENT_JOYSTICK_BUTTON = 450,  // OH 新增
};
```

**用途**：处理游戏手柄的按钮事件

**使用示例**：
```c
struct libinput_event *event;
while ((event = libinput_get_event(libinput))) {
    switch (libinput_event_get_type(event)) {
        case LIBINPUT_EVENT_JOYSTICK_BUTTON:
            struct libinput_event_joystick_button *joy_btn =
                libinput_event_get_joystick_button_event(event);
            enum libinput_button_state state =
                libinput_event_joystick_button_get_state(joy_btn);
            uint32_t button = libinput_event_joystick_button_get_button(joy_btn);
            printf("Joystick button %d: %s\n",
                   button,
                   state == LIBINPUT_BUTTON_STATE_PRESSED ? "pressed" : "released");
            break;
    }
}
```

**新增 API**：
```c
// 获取 Joystick 按钮事件
struct libinput_event_joystick_button *
libinput_event_get_joystick_button_event(struct libinput_event *event);

// 获取按钮状态
enum libinput_button_state
libinput_event_joystick_button_get_state(struct libinput_event_joystick_button *event);

// 获取按钮码
uint32_t
libinput_event_joystick_button_get_button(struct libinput_event_joystick_button *event);
```

---

#### LIBINPUT_EVENT_JOYSTICK_AXIS

**定义**：
```c
enum libinput_event_type {
    LIBINPUT_EVENT_JOYSTICK_AXIS,  // OH 新增
};
```

**用途**：处理游戏手柄的轴事件（摇杆、油门、刹车等）

**使用示例**：
```c
struct libinput_event *event;
while ((event = libinput_get_event(libinput))) {
    switch (libinput_event_get_type(event)) {
        case LIBINPUT_EVENT_JOYSTICK_AXIS:
            struct libinput_event_joystick_axis *joy_axis =
                libinput_event_get_joystick_axis_event(event);

            // 获取 X 轴信息
            struct libinput_event_joystick_axis_abs_info *x_info =
                libinput_event_joystick_axis_get_abs_info(joy_axis,
                    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_X);
            printf("Joystick X axis: %d (min: %d, max: %d)\n",
                   x_info->value, x_info->minimum, x_info->maximum);

            // 获取 Y 轴信息
            struct libinput_event_joystick_axis_abs_info *y_info =
                libinput_event_joystick_axis_get_abs_info(joy_axis,
                    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_Y);
            printf("Joystick Y axis: %d (min: %d, max: %d)\n",
                   y_info->value, y_info->minimum, y_info->maximum);
            break;
    }
}
```

**新增 API**：
```c
// 获取 Joystick 轴事件
struct libinput_event_joystick_axis *
libinput_event_get_joystick_axis_event(struct libinput_event *event);

// 获取轴信息
struct libinput_event_joystick_axis_abs_info *
libinput_event_joystick_axis_get_abs_info(struct libinput_event_joystick_axis *event,
                                     enum libinput_joystick_axis_source source);
```

---

### MSDP 事件

#### LIBINPUT_EVENT_MSDP

**定义**：
```c
enum libinput_event_type {
    LIBINPUT_EVENT_MSDP = 1000,  // OH 新增
};
```

**用途**：处理 MSDP（Multi-Source Data Processing）设备事件

**使用示例**：
```c
struct libinput_event *event;
while ((event = libinput_get_event(libinput))) {
    if (libinput_event_get_type(event) == LIBINPUT_EVENT_MSDP) {
        printf("MSDP event received\n");
        // 处理 MSDP 特有事件
        // 例如：手部检测数据、多源输入处理
    }
}
```

**OH 需求**：支持多源数据处理设备（用于高级手势识别）

---

## 🆕 新增开关类型

### LIBINPUT_SWITCH_PRIVACY

**定义**：
```c
enum libinput_switch {
    LIBINPUT_SWITCH_LID = 0,
    LIBINPUT_SWITCH_TABLET_MODE,
    LIBINPUT_SWITCH_PRIVACY,  // OH 新增
};
```

**用途**：表示隐私开关（摄像头、麦克风等硬件）的状态

**使用示例**：
```c
struct libinput_event *event;
while ((event = libinput_get_event(libinput))) {
    if (libinput_event_get_type(event) == LIBINPUT_EVENT_SWITCH_TOGGLE) {
        enum libinput_switch sw = libinput_event_switch_get_switch(event);
        enum libinput_switch_state state =
            libinput_event_switch_get_state(event);

        if (sw == LIBINPUT_SWITCH_PRIVACY) {
            if (state == LIBINPUT_SWITCH_STATE_ON) {
                printf("Privacy switch is ON\n");
                // 隐藏摄像头、禁用麦克风
            } else {
                printf("Privacy switch is OFF\n");
                // 启用摄像头、麦克风
            }
        }
    }
}
```

**新增输入开关码**：
```c
#define SW_SUPER_PRIVACY 0x11  /* set = super privacy open */
```

**OH 需求**：支持隐私硬件开关（如摄像头物理开关）

---

## 🆕 新增数据结构

### 槽位坐标信息

#### struct sloted_coords

**定义**：
```c
#define MAX_SOLTED_COORDS_NUM 10

struct sloted_coords {
    int32_t is_active;  // 槽位是否激活
    float x;           // X 坐标
    float y;           // Y 坐标
};
```

**用途**：表示单个触摸槽位的坐标信息

---

#### struct sloted_coords_info

**定义**：
```c
struct sloted_coords_info {
    struct sloted_coords coords[MAX_SOLTED_COORDS_NUM];  // 10 个槽位
    unsigned int active_count;                          // 激活的槽位数量
};
```

**用途**：管理多点触控的多个槽位坐标信息（最多 10 个槽位）

**使用示例**：
```c
struct libinput_event *event;
while ((event = libinput_get_event(libinput))) {
    // 从手势事件中获取槽位坐标
    struct sloted_coords_info *coords =
        libinput_event_get_solt_touches(event);

    printf("Active touches: %d\n", coords->active_count);
    for (unsigned int i = 0; i < coords->active_count; i++) {
        printf("Touch %d: (%.1f, %.1f)\n",
               i, coords->coords[i].x, coords->coords[i].y);
    }
}
```

**新增 API**：
```c
// 获取槽位坐标信息
struct sloted_coords_info *
libinput_event_get_solt_touches(struct libinput_event *event);
```

**OH 需求**：支持复杂的多点触控场景和手势识别

---

### Joystick 轴信息

#### struct libinput_event_joystick_axis_abs_info

**定义**：
```c
struct libinput_event_joystick_axis_abs_info {
    int32_t code;       // 轴码
    int32_t value;      // 当前值
    int32_t minimum;    // 最小值
    int32_t maximum;    // 最大值
    int32_t fuzz;       // 模糊度
    int32_t flat;       // 平区大小
    int32_t resolution;  // 分辨率
    float   standardValue;  // 标准化值
};
```

**用途**：表示 Joystick 轴的详细信息（位置、范围、分辨率等）

**使用示例**：
```c
struct libinput_event *event;
while ((event = libinput_get_event(libinput))) {
    if (libinput_event_get_type(event) == LIBINPUT_EVENT_JOYSTICK_AXIS) {
        struct libinput_event_joystick_axis *joy_axis =
            libinput_event_get_joystick_axis_event(event);

        // 获取 X 轴信息
        struct libinput_event_joystick_axis_abs_info *x_info =
            libinput_event_joystick_axis_get_abs_info(joy_axis,
                LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_X);

        printf("X axis: %d [range: %d~%d, fuzz: %d, res: %d]\n",
               x_info->value, x_info->minimum, x_info->maximum,
               x_info->fuzz, x_info->resolution);
    }
}
```

---

## 🆕 新增枚举

### libinput_joystick_axis_source

**定义**：
```c
enum libinput_joystick_axis_source {
    LIBINPUT_JOYSTICK_AXIS_SOURCE_UNKNOWN = 0,

    // 主轴
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_X = 1 << 0,
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_Y = 1 << 1,
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_Z = 1 << 2,

    // 旋转轴
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_RX = 1 << 3,
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_RY = 1 << 4,
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_RZ = 1 << 5,

    // 油门/方向舵
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_THROTTLE = 1 << 6,
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_RUDDER = 1 << 7,

    // 滚轮
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_WHEEL = 1 << 8,

    // 刹车/加速
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_GAS = 1 << 9,
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_BRAKE = 1 << 10,

    // 方向帽
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_HAT0X = 1 << 11,
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_HAT0Y = 1 << 12,
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_HAT1X = 1 << 13,
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_HAT1Y = 1 << 14,
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_HAT2X = 1 << 15,
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_HAT2Y = 1 << 16,
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_HAT3X = 1 << 17,
    LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_HAT3Y = 1 << 18,
};
```

**用途**：识别 Joystick 的轴类型（19 种不同的轴）

**使用示例**：
```c
// 获取特定轴的信息
struct libinput_event_joystick_axis_abs_info *x_info =
    libinput_event_joystick_axis_get_abs_info(event,
        LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_X);

struct libinput_event_joystick_axis_abs_info *gas_info =
    libinput_event_joystick_axis_get_abs_info(event,
        LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_GAS);

struct libinput_event_joystick_axis_abs_info *hat0x_info =
    libinput_event_joystick_axis_get_abs_info(event,
        LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_HAT0X);
```

---

## 🆕 新增输入事件码

### KEY 码

```c
#define KEY_MICMUTE                    251   // 麦克风静音
#define KEY_MOUSE_ASSISTANT           0x2e9  // 鼠标助手
#define KEY_MOUSE_INTELLIGENCE_SELECTION 0x2ea  // 鼠标智能选择
#define KEY_AOD_SINGLE_CLICK            0x2fd  // AOD 单击
```

**用途**：支持 OH 特有的按键和功能

---

### ABS 码

```c
#define ABS_HAND_FEATURE                0x27   // 手势特性（MSDP 检测）
#define ABS_MT_MOVEFLAG                 0x29   // 多触控移动标志
#define ABS_MT_TWIST                    0x2c   // 多触控旋转
```

**用途**：
- `ABS_HAND_FEATURE`：MSDP 设备的手部检测轴
- `ABS_MT_MOVEFLAG`：多点触控的移动标志
- `ABS_MT_TWIST`：触摸点的旋转角度

---

### SW 码

```c
#define SW_SUPER_PRIVACY                0x11   // 超级隐私开关
```

**用途**：隐私硬件开关的状态

---

## 🆕 新增 API 函数

### 触摸板相关

#### libinput_event_get_touchpad_event

**原型**：
```c
struct libinput_event_touch *
libinput_event_get_touchpad_event(struct libinput_event *event);
```

**用途**：获取触摸板专用的事件

**参数**：
- `event`：libinput 事件指针

**返回值**：
- 成功：返回触摸板事件指针
- 失败/不匹配：返回 NULL

**使用示例**：
```c
struct libinput_event *event = libinput_get_event(libinput);

if (libinput_event_get_type(event) == LIBINPUT_EVENT_TOUCHPAD_DOWN ||
    libinput_event_get_type(event) == LIBINPUT_EVENT_TOUCHPAD_MOTION) {

    struct libinput_event_touch *tp_event =
        libinput_event_get_touchpad_event(event);

    float x = libinput_event_touch_get_x(tp_event);
    float y = libinput_event_touch_get_y(tp_event);

    printf("Touchpad event: (%.1f, %.1f)\n", x, y);
}
```

---

### 虚拟触摸板相关

#### libinput_event_vtrackpad_get_dx_unaccelerated

**原型**：
```c
double
libinput_event_vtrackpad_get_dx_unaccelerated(struct libinput_event_pointer *event);
```

**用途**：获取虚拟触摸板的未加速 X 偏移

**参数**：
- `event`：指针事件指针

**返回值**：
- 未加速的 X 偏移值

---

#### libinput_event_vtrackpad_get_dy_unaccelerated

**原型**：
```c
double
libinput_event_vtrackpad_get_dy_unaccelerated(struct libinput_event_pointer *event);
```

**用途**：获取虚拟触摸板的未加速 Y 偏移

**参数**：
- `event`：指针事件指针

**返回值**：
- 未加速的 Y 偏移值

---

### 按针区域相关

#### libinput_event_pointer_get_button_area

**原型**：
```c
uint32_t
libinput_event_pointer_get_button_area(struct libinput_event_pointer *event);
```

**用途**：获取按钮的区域信息

**参数**：
- `event`：指针事件指针

**返回值**：
- 按钮区域的标识符

**OH 需求**：支持触摸板的按钮区域配置（如触摸板底部点击区域）

---

### Joystick 相关

#### libinput_event_get_joystick_button_event

**原型**：
```c
struct libinput_event_joystick_button *
libinput_event_get_joystick_button_event(struct libinput_event *event);
```

**用途**：获取 Joystick 按钮事件

**参数**：
- `event`：libinput 事件指针

**返回值**：
- Joystick 按钮事件指针
- 失败/不匹配：返回 NULL

---

#### libinput_event_get_joystick_axis_event

**原型**：
```c
struct libinput_event_joystick_axis *
libinput_event_get_joystick_axis_event(struct libinput_event *event);
```

**用途**：获取 Joystick 轴事件

**参数**：
- `event`：libinput 事件指针

**返回值**：
- Joystick 轴事件指针
- 失败/不匹配：返回 NULL

---

#### libinput_event_joystick_button_get_state

**原型**：
```c
enum libinput_button_state
libinput_event_joystick_button_get_state(struct libinput_event_joystick_button *event);
```

**用途**：获取 Joystick 按钮状态（按下或释放）

**参数**：
- `event`：Joystick 按钮事件指针

**返回值**：
- `LIBINPUT_BUTTON_STATE_PRESSED`：按钮按下
- `LIBINPUT_BUTTON_STATE_RELEASED`：按钮释放

---

#### libinput_event_joystick_button_get_button

**原型**：
```c
uint32_t
libinput_event_joystick_button_get_button(struct libinput_event_joystick_button *event);
```

**用途**：获取 Joystick 按钮码

**参数**：
- `event`：Joystick 按钮事件指针

**返回值**：
- 按钮码（如 BTN_A, BTN_B, BTN_X 等）

---

#### libinput_event_joystick_axis_get_abs_info

**原型**：
```c
struct libinput_event_joystick_axis_abs_info *
libinput_event_joystick_axis_get_abs_info(struct libinput_event_joystick_axis *event,
                                     enum libinput_joystick_axis_source source);
```

**用途**：获取 Joystick 轴的绝对信息

**参数**：
- `event`：Joystick 轴事件指针
- `source`：轴源类型（见 `enum libinput_joystick_axis_source`）

**返回值**：
- 轴信息结构指针，包含：
  - `code`：轴码
  - `value`：当前值
  - `minimum`：最小值
  - `maximum`：最大值
  - `fuzz`：模糊度
  - `flat`：平区大小
  - `resolution`：分辨率
  - `standardValue`：标准化值

---

### 槽位坐标相关

#### libinput_event_get_solt_touches

**原型**：
```c
struct sloted_coords_info *
libinput_event_get_solt_touches(struct libinput_event *event);
```

**用途**：获取多点触控的槽位坐标信息

**参数**：
- `event`：libinput 事件指针

**返回值**：
- 槽位坐标信息结构指针，包含：
  - `coords`：10 个槽位的坐标数组
  - `active_count`：激活的槽位数量

**OH 需求**：支持复杂的多点触控手势和场景

---

## 📊 API 总结

### 按功能分类

| 功能 | 新增 API 数量 | 主要用途 |
|------|--------------|---------|
| **触摸板专用事件** | 1 | 独立的 TOUCHPAD 事件处理 |
| **虚拟触摸板** | 2 | 虚拟触摸板未加速偏移 |
| **按钮区域** | 1 | 按钮区域配置 |
| **Joystick 按钮** | 3 | Joystick 按钮事件和状态 |
| **Joystick 轴** | 2 | Joystick 轴事件和信息 |
| **槽位坐标** | 1 | 多点触控槽位信息 |

### 按兼容性分类

| API 类别 | 与上游关系 | 升级建议 |
|----------|------------|---------|
| **完全 OH 特有** | 上游不存在 | 保留 OH 定制 |
| **上游扩展** | 上游有部分支持 | 可合并/增强 |
| **合理扩展** | 符合 libinput 设计 | 可推向上流 |

---

## 🔧 开发示例

### 示例 1：完整的 Joystick 处理程序

```c
#include <stdio.h>
#include <libinput.h>

int main(int argc, char **argv) {
    // 创建 libinput 上下文
    struct libinput *li = libinput_path_create_context(NULL);

    // 获取设备
    struct libinput_event *event;
    struct libinput_device *device;

    while ((event = libinput_get_event(li))) {
        device = libinput_event_get_device(event);

        // 检查设备能力
        if (libinput_device_has_capability(device,
                                         LIBINPUT_DEVICE_CAP_JOYSTICK)) {
            printf("Joystick device: %s\n",
                   libinput_device_get_name(device));

            // 处理 Joystick 事件
            switch (libinput_event_get_type(event)) {
                case LIBINPUT_EVENT_JOYSTICK_BUTTON:
                    {
                        struct libinput_event_joystick_button *joy_btn =
                            libinput_event_get_joystick_button_event(event);

                        uint32_t button =
                            libinput_event_joystick_button_get_button(joy_btn);
                        enum libinput_button_state state =
                            libinput_event_joystick_button_get_state(joy_btn);

                        printf("Button %d: %s\n", button,
                               state == LIBINPUT_BUTTON_STATE_PRESSED ?
                                   "pressed" : "released");
                    }
                    break;

                case LIBINPUT_EVENT_JOYSTICK_AXIS:
                    {
                        struct libinput_event_joystick_axis *joy_axis =
                            libinput_event_get_joystick_axis_event(event);

                        // 获取 X 轴
                        struct libinput_event_joystick_axis_abs_info *x_info =
                            libinput_event_joystick_axis_get_abs_info(joy_axis,
                                LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_X);

                        // 获取 Y 轴
                        struct libinput_event_joystick_axis_abs_info *y_info =
                            libinput_event_joystick_axis_get_abs_info(joy_axis,
                                LIBINPUT_JOYSTICK_AXIS_SOURCE_ABS_Y);

                        printf("X: %d, Y: %d\n", x_info->value, y_info->value);
                    }
                    break;
            }
        }
    }

    libinput_unref(li);
    return 0;
}
```

---

### 示例 2：触摸板专用事件处理

```c
#include <stdio.h>
#include <libinput.h>

void handle_touchpad_event(struct libinput_event *event) {
    // 获取触摸板事件
    struct libinput_event_touch *tp =
        libinput_event_get_touchpad_event(event);

    if (!tp) {
        return;
    }

    // 获取坐标
    float x = libinput_event_touch_get_x(tp);
    float y = libinput_event_touch_get_y(tp);

    // 处理不同类型的触摸板事件
    switch (libinput_event_get_type(event)) {
        case LIBINPUT_EVENT_TOUCHPAD_DOWN:
            printf("Touchpad DOWN at (%.1f, %.1f)\n", x, y);
            break;

        case LIBINPUT_EVENT_TOUCHPAD_MOTION:
            printf("Touchpad MOTION to (%.1f, %.1f)\n", x, y);
            break;

        case LIBINPUT_EVENT_TOUCHPAD_UP:
            printf("Touchpad UP at (%.1f, %.1f)\n", x, y);
            break;

        case LIBINPUT_EVENT_TOUCHPAD_ACTIVE:
            printf("Touchpad ACTIVE\n");
            break;
    }
}

int main(int argc, char **argv) {
    struct libinput *li = libinput_path_create_context(NULL);
    struct libinput_event *event;

    while ((event = libinput_get_event(li))) {
        handle_touchpad_event(event);
    }

    libinput_unref(li);
    return 0;
}
```

---

### 示例 3：隐私开关监听

```c
#include <stdio.h>
#include <libinput.h>

void handle_privacy_switch(struct libinput_event *event) {
    if (libinput_event_get_type(event) != LIBINPUT_EVENT_SWITCH_TOGGLE) {
        return;
    }

    enum libinput_switch sw = libinput_event_switch_get_switch(event);
    enum libinput_switch_state state =
        libinput_event_switch_get_state(event);

    if (sw == LIBINPUT_SWITCH_PRIVACY) {
        if (state == LIBINPUT_SWITCH_STATE_ON) {
            printf("✓ Privacy is ON\n");
            printf("  → Camera disabled, Microphone disabled\n");
        } else {
            printf("✗ Privacy is OFF\n");
            printf("  → Camera enabled, Microphone enabled\n");
        }
    }
}

int main(int argc, char **argv) {
    struct libinput *li = libinput_path_create_context(NULL);
    struct libinput_event *event;

    while ((event = libinput_get_event(li))) {
        handle_privacy_switch(event);
    }

    libinput_unref(li);
    return 0;
}
```

---

## 📚 相关文档

- [02_Patches.md](02_Patches.md) - Patch 详细分析
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 中的使用情况

---

**文档版本**: 1.0
**最后更新**: 2026-02-08
