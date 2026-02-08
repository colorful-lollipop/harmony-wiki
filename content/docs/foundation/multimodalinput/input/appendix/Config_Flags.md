# appendix/Config_Flags - 配置开关

## 概述

本文档描述 multimodalinput_input 子系统的编译配置开关和产品适配配置。

---

## 8.1 Feature Flags

### 8.1.1 功能开关清单

| Flag | 默认值 | 用途 | 所在文件 |
|------|--------|------|---------|
| `input_feature_touchscreen` | true | 触摸屏支持 | multimodalinput_mini.gni |
| `input_feature_mouse` | true | 鼠标支持 | multimodalinput_mini.gni |
| `input_feature_keyboard` | true | 键盘支持 | multimodalinput_mini.gni |
| `input_feature_touchpad` | true | 触摸板支持 | multimodalinput_mini.gni |
| `input_feature_joystick` | false | 游戏手柄支持 | multimodalinput_mini.gni |
| `input_feature_pointer_drawing` | false | 指针绘制 | multimodalinput_mini.gni |
| `input_feature_virtual_keyboard` | false | 虚拟键盘 | multimodalinput_mini.gni |
| `input_feature_knuckle` | false | 指关节手势 | multimodalinput_mini.gni |
| `input_feature_crown` | false | 表冠支持 | multimodalinput_mini.gni |
| `input_feature_short_key` | true | 快捷键 | multimodalinput_mini.gni |
| `input_feature_monitor` | true | 事件监控 | multimodalinput_mini.gni |
| `input_feature_interceptor` | true | 事件拦截 | multimodalinput_mini.gni |
| `input_feature_fingerprint` | false | 指纹识别 | multimodalinput_mini.gni |

---

## 8.2 编译配置

### 8.2.1 GCC/Clang 配置

```gn
# BUILD.gn
config("coverage_flags") {
  if (input_feature_coverage) {
    cflags = [ "--coverage" ]
    ldflags = [ "--coverage" ]
  }
}

# CFI 控制流完整性
sanitize = {
  cfi = true
  cfi_cross_dso = true
}
```

### 8.2.2 优化配置

```gn
# PGO 优化
if (input_feature_enable_pgo && input_feature_product != "default") {
  cflags += [
    "-fprofile-use=" + rebase_path("${input_feature_pgo_path}/libmmi-server.profdata"),
    "-Oz",
  ]
}
```

---

## 8.3 产品配置示例

### 8.3.1 rk3568 产品配置

```gn
# product/rk3568/multimodalinput.gni
input_feature_product = "rk3568"
input_feature_mouse = true
input_feature_keyboard = true
input_feature_touchpad = true
input_feature_joystick = true
input_feature_pointer_drawing = true
```

### 8.3.2 hi3516dv300 产品配置

```gn
# product/hi3516dv300/multimodalinput.gni
input_feature_product = "hi3516dv300"
input_feature_mouse = false
input_feature_keyboard = false
input_feature_touchpad = false
input_feature_joystick = false
input_feature_pointer_drawing = false
```

---

## 8.4 配置文件

### 8.4.1 服务配置

**文件**: `multimodalinput.cfg`

```json
{
  "input": {
    "device": {
      "touchscreen": {
        "enabled": true
      },
      "mouse": {
        "enabled": true
      }
    },
    "event": {
      "queue_size": 1024,
      "timeout_ms": 100
    }
  }
}
```

### 8.4.2 SA 配置文件

**文件**: `sa_profile/3101.json`

```json
{
  "process": "multimodalinput",
  "systemability": [
    {
      "name": 3101,
      "libpath": "libmmi-server.z.so",
      "run-on-create": true,
      "distributed": false,
      "dump_level": 1
    }
  ]
}
```

---

## 8.5 设备特定配置

### 8.5.1 触摸屏配置

**文件**: `etc/mmi_touchscreen_config.json`

```json
{
  "touchscreen": {
    "default": {
      "poll_time": 8,
      "max_fingers": 10
    }
  }
}
```

### 8.5.2 鼠标配置

**文件**: `etc/mmi_mouse_config.json`

```json
{
  "mouse": {
    "default": {
      "scroll_speed": 3,
      "double_click_time": 500
    }
  }
}
```
