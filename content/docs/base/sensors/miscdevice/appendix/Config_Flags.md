# 配置开关与 Feature Flags

## 概述

miscdevice 子系统支持多种编译时配置开关（Feature Flags），用于控制功能启用/禁用、条件编译和行为定制。

## GN 配置入口

**根配置文件**: `miscdevice.gni`

```gn
import("//build/ohos.gni")

declare_args() {
  miscdevice_feature_vibrator_custom = true
  miscdevice_feature_hiviewdfx_hisysevent = true
  miscdevice_feature_vibrator_input_method_enable = true
  miscdevice_feature_crown_vibrator_enable = false
  miscdevice_feature_do_not_disturb_enable = false
}
```

## Feature Flags 详解

### 1. miscdevice_feature_vibrator_custom

| 属性 | 值 |
|------|-----|
| 默认值 | `true` |
| 宏定义 | `OHOS_BUILD_ENABLE_VIBRATOR_CUSTOM` |
| 优先级 | **最高** (条件编译入口) |

**功能**: 启用自定义振动功能

**影响范围**:
```
启用时:
├── IDL 选择: all/IMiscdeviceService.idl (完整接口)
├── 包含文件: compatible_connection.cpp
├── 功能: 支持自定义振动模式
└── SA 接口: 完整的 PlayVibratorCustom API

禁用时:
├── IDL 选择: part/IMiscdeviceService.idl (部分接口)
├── 排除文件: compatible_connection.cpp
├── 功能: 仅支持预设振动
└── SA 接口: 移除自定义振动相关 API
```

**代码证据**:
```gn
# BUILD.gn
if (miscdevice_feature_vibrator_custom) {
  sources += [ "hdi_connection/adapter/src/compatible_connection.cpp" ]
}

# vibrator/BUILD.gn
if (miscdevice_feature_vibrator_custom) {
  sources = [ "all/IMiscdeviceService.idl" ]
} else {
  sources = [ "part/IMiscdeviceService.idl" ]
}
```

**使用场景**:
- 标准设备: 保持 `true`
- 轻量设备: 可设为 `false` (减少代码体积)

---

### 2. miscdevice_feature_vibrator_input_method_enable

| 属性 | 值 |
|------|-----|
| 默认值 | `true` |
| 宏定义 | `OHOS_BUILD_ENABLE_VIBRATOR_INPUT_METHOD` |
| 互斥项 | `OHOS_BUILD_ENABLE_VIBRATOR_PRESET_INFO` |

**功能**: 启用输入法振动支持

**影响范围**:
```
启用时 (OHOS_BUILD_ENABLE_VIBRATOR_INPUT_METHOD):
├── 振动优先级: 输入法优先级模式
├── API: 支持输入反馈专用 API
└── 依赖: os_account 模块

禁用时 (OHOS_BUILD_ENABLE_VIBRATOR_PRESET_INFO):
├── 振动优先级: 预设信息模式
├── API: 预设效果列表
└── 依赖: 无
```

**代码证据**:
```gn
# services/miscdevice_service/BUILD.gn
if (miscdevice_feature_vibrator_input_method_enable) {
  external_deps += [ "os_account:os_account_innerkits" ]
}
```

**条件宏逻辑**:
```cpp
// miscdevice.gni
if (miscdevice_feature_vibrator_input_method_enable) {
  defines += [ "OHOS_BUILD_ENABLE_VIBRATOR_INPUT_METHOD" ]
} else {
  defines += [ "OHOS_BUILD_ENABLE_VIBRATOR_PRESET_INFO" ]
}
```

---

### 3. miscdevice_feature_crown_vibrator_enable

| 属性 | 值 |
|------|-----|
| 默认值 | `false` |
| 宏定义 | `OHOS_BUILD_ENABLE_VIBRATOR_CROWN` |

**功能**: 启用表冠(Crown)旋转振动反馈

**影响范围**:
```
启用时:
├── 功能: 支持表冠旋转振动
├── API: 额外的表冠振动模式
└── 依赖: 可能需要额外的 HDI 支持

禁用时:
├── 排除: 表冠相关代码
└── 代码体积: 减少
```

**使用建议**: 智能手表设备启用

---

### 4. miscdevice_feature_do_not_disturb_enable

| 属性 | 值 |
|------|-----|
| 默认值 | `false` |
| 宏定义 | `OHOS_BUILD_ENABLE_DO_NOT_DISTURB` |

**功能**: 启用勿扰模式支持

**影响范围**:
```
启用时:
├── 功能: 支持勿扰时间段配置
├── API: 勿扰模式控制
└── 行为: 在勿扰期间忽略振动请求

禁用时:
├── 排除: 勿扰相关逻辑
└── 始终允许振动
```

---

### 5. miscdevice_feature_hiviewdfx_hisysevent

| 属性 | 值 |
|------|-----|
| 默认值 | `true` |
| 宏定义 | `HIVIEWDFX_HISYSEVENT_ENABLE` |

**功能**: 启用 HiSysEvent 事件上报

**影响范围**:
```
启用时:
├── 上报: VIBRATOR_PERMISSIONS_EXCEPTION
├── 上报: LIGHT_PERMISSIONS_EXCEPTION
├── 日志: 完整的审计日志
└── 依赖: hisysevent 模块

禁用时:
├── 排除: HiSysEvent 相关代码
└── 减少: 系统日志开销
```

**代码证据**:
```gn
# services/miscdevice_service/BUILD.gn
if (miscdevice_feature_hiviewdfx_hisysevent) {
  external_deps += [ "hisysevent:libhisysevent" ]
}
```

---

### 6. miscdevice_feature_hiviewdfx_hitrace

| 属性 | 值 |
|------|-----|
| 默认值 | `true` |
| 宏定义 | `HIVIEWDFX_HITRACE_ENABLE` |

**功能**: 启用 HiTrace 性能追踪

**影响范围**:
```
启用时:
├── 追踪: 完整的调用链追踪
├── 性能: 性能分析支持
└── 依赖: hitrace 模块

禁用时:
├── 排除: HiTrace 相关代码
└── 减少: 追踪开销
```

---

## HDI 相关配置

### miscdevice_feature_hdf_drivers_interface_vibrator

| 属性 | 值 |
|------|-----|
| 默认值 | `true` (可被覆盖) |
| 宏定义 | `HDF_DRIVERS_INTERFACE_VIBRATOR` |

**功能**: 启用 HDF 振动器驱动接口

**代码证据**:
```gn
# miscdevice.gni
if (!defined(global_parts_info) ||
    defined(global_parts_info.hdf_drivers_interface_vibrator)) {
  miscdevice_feature_hdf_drivers_interface_vibrator = true
  miscdevice_default_defines += [ "HDF_DRIVERS_INTERFACE_VIBRATOR" ]
}
```

### hdf_drivers_interface_light

| 属性 | 值 |
|------|-----|
| 默认值 | `true` (可被覆盖) |
| 宏定义 | `HDF_DRIVERS_INTERFACE_LIGHT` |

**功能**: 启用 HDF 灯光驱动接口

---

## 构建变体配置

### 工程模式 (root)

```gn
if (build_variant == "root") {
  defines += [ "BUILD_VARIANT_ENG" ]
  miscdevice_build_eng = true
}
```

**启用时**:
- 包含调试符号
- 启用 `compatible_connection.cpp` (更完整的兼容性代码)
- 可能的额外日志

### 用户模式

```gn
build_variant != "root" -> miscdevice_build_eng = false
```

---

## 内存管理配置

### miscdevice_memmgr_enable

| 属性 | 值 |
|------|-----|
| 默认值 | `true` (可被覆盖) |
| 宏定义 | `MEMMGR_ENABLE` |

**功能**: 启用系统内存管理

**代码证据**:
```gn
if (miscdevice_memmgr_enable) {
  defines += [ "MEMMGR_ENABLE" ]
  external_deps += [ "memmgr:memmgrclient" ]
}
```

---

## NDK 配置

### libvibrator_ndk

| 属性 | 值 |
|------|-----|
| 描述文件 | `libvibrator.json` |
| 最小兼容版本 | 6 |

**NDK 头文件**:
- `interfaces/inner_api/vibrator/vibrator_agent.h`
- `interfaces/inner_api/vibrator/vibrator_agent_type.h`

---

## 配置组合示例

### 标准设备配置

```gn
miscdevice_feature_vibrator_custom = true
miscdevice_feature_vibrator_input_method_enable = true
miscdevice_feature_crown_vibrator_enable = false
miscdevice_feature_do_not_disturb_enable = false
miscdevice_feature_hiviewdfx_hisysevent = true
miscdevice_feature_hiviewdfx_hitrace = true
```

### 轻量设备配置

```gn
miscdevice_feature_vibrator_custom = false  # 减少代码体积
miscdevice_feature_vibrator_input_method_enable = false  # 无需输入法
miscdevice_feature_crown_vibrator_enable = false
miscdevice_feature_do_not_disturb_enable = false
miscdevice_feature_hiviewdfx_hisysevent = false  # 减少日志
miscdevice_feature_hiviewdfx_hitrace = false
```

### 智能手表配置

```gn
miscdevice_feature_vibrator_custom = true
miscdevice_feature_vibrator_input_method_enable = false
miscdevice_feature_crown_vibrator_enable = true  # 表冠支持
miscdevice_feature_do_not_disturb_enable = true
miscdevice_feature_hiviewdfx_hisysevent = true
miscdevice_feature_hiviewdfx_hitrace = true
```

---

## 配置验证

### 检查当前配置

```bash
# 在构建输出中查看
python3 build.py --build-var miscdevice_feature_vibrator_custom
```

### 配置冲突检测

| 冲突场景 | 解决方法 |
|---------|----------|
| `input_method` 与 `preset_info` 互斥 | 确保只有一个为 true |
| `hisysevent` 禁用但使用 HiSysEvent API | 添加条件编译保护 |

---

## 遗留配置

### DEPRECATED Flags

| Flag | 状态 | 替代方案 |
|------|------|----------|
| (无) | - | 当前无废弃配置 |
