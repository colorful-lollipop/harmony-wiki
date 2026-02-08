# 配置标志说明 - display_manager

> 本文档说明 display_manager 模块的所有配置标志和编译选项

---

## 全局配置标志（displaymgr.gni）

### 特性开关

| 标志 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `display_manager_feature_brightnessext` | string | `""` | 亮度扩展功能名称，空字符串表示未启用 |
| `display_manager_feature_poweroff_strategy` | bool | `false` | 屏幕关闭策略功能开关 |

### 组件检测标志

| 标志 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `has_sensors_sensor_part` | bool | `true` | 传感器组件是否可用（自动检测） |
| `has_hiviewdfx_hisysevent_part` | bool | `true` | Hisysevent 组件是否可用（自动检测） |
| `has_dfx_hiview_part` | bool | `true` | HiView DFX 组件是否可用（自动检测） |

---

## 条件编译宏

### 功能宏

| 宏 | 定义条件 | 说明 | 影响 |
|------|----------|------|------|
| `ENABLE_SENSOR_PART` | `has_sensors_sensor_part == true` | 启用传感器支持 | 包含光传感器相关代码 |
| `ENABLE_SCREEN_POWER_OFF_STRATEGY` | `display_manager_feature_poweroff_strategy == true` | 启用关屏策略 | 包含关屏策略代码 |
| `OHOS_BUILD_ENABLE_BRIGHTNESS_WRAPPER` | `display_manager_feature_brightnessext != ""` | 启用亮度扩展包装 | 使用扩展实现替代默认实现 |

### 调试宏

| 宏 | 定义条件 | 说明 |
|------|----------|------|
| `HAS_HIVIEWDFX_HISYSEVENT_PART` | `has_hiviewdfx_hisysevent_part == true` | 启用 Hisysevent 日志上报 |
| `HAS_DFX_HIVIEW_PART` | `has_dfx_hiview_part == true` | 启用 HiView DFX 功能 |
| `FUZZ_COV_TEST` | `use_clang_coverage == true` | Fuzz 测试覆盖率模式 |
| `FUZZ_TEST` | `use_libfuzzer == true` | Fuzz 测试模式 |

---

## 使用示例

### 启用传感器支持

```gn
# 在组件配置中（默认自动检测）
has_sensors_sensor_part = true
```

影响：
- 添加 `sensor:sensor_interface_native` 依赖
- 定义 `ENABLE_SENSOR_PART` 宏
- 包含 `LightLuxManager` 相关代码

---

### 启用关屏策略

```gn
# 在 productdefine 或组件配置中
display_manager_feature_poweroff_strategy = true
```

影响：
- 定义 `ENABLE_SCREEN_POWER_OFF_STRATEGY` 宏
- 添加 `miscellaneous_display_power_strategy.cpp` 到编译
- 启用 `SetScreenPowerOffStrategy` 接口

---

### 启用亮度扩展

```gn
# 在组件配置中
display_manager_feature_brightnessext = "custom_wrapper"
```

影响：
- 定义 `OHOS_BUILD_ENABLE_BRIGHTNESS_WRAPPER` 宏
- `BrightnessManager` 使用 `mBrightnessManagerExt` 实现

---

## 配置优先级

```
productdefine（产品配置）
    ↓ 覆盖
global_parts_info（全局部件信息）
    ↓ 检测
displaymgr.gni（默认值）
```

---

## 相关链接

- **GN 构建**：[../06_GN_Targets.md](../06_GN_Targets.md)
- **项目概览**：[../00_Overview.md](../00_Overview.md)
