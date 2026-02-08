# 附录：配置标志 (Config Flags)

## Feature Flags 汇总

### 来自 windowmanager_aafwk.gni

| Flag | 默认值 | 说明 | 影响范围 |
|------|--------|------|----------|
| `window_manager_use_sceneboard` | true | 使用 Scene Board 架构 | 全局架构切换 |
| `device_status_enable` | true | 启用设备状态功能 | 设备状态检测 |
| `window_manager_feature_subscribe_motion` | false | 运动传感器订阅 | 手势识别 |
| `window_manager_feature_tp_enable` | false | TP (Touch Panel) 功能 | 触控面板 |
| `window_manager_fold_ability` | true | 折叠屏支持 | 折叠设备 |
| `window_manager_feature_screen_active_mode` | true | 屏幕主动模式 | 电源管理 |
| `window_manager_feature_screen_color_gamut` | true | 色域支持 | 显示效果 |
| `window_manager_feature_screen_hdr_format` | true | HDR 格式支持 | 显示效果 |
| `window_manager_feature_screen_color_space` | true | 色彩空间支持 | 显示效果 |
| `window_manager_feature_multi_screen` | true | 多屏幕支持 | 多显示器 |
| `window_manager_feature_multi_screen_frame_ctl` | true | 多屏幕帧控制 | 多显示器同步 |
| `window_manager_feature_cam_mode` | true | 相机模式 | 相机应用 |
| `window_manager_feature_multi_usr` | true | 多用户支持 | 多用户系统 |
| `window_manager_feature_screenless` | false | 无屏幕模式 | 特殊设备 |
| `window_manager_feature_asbng_path` | "" | 原子服务引擎路径 | 原子服务 |
| `window_manager_feature_support_dsoftbus` | true | 分布式软总线 | 分布式能力 |
| `window_manager_feature_support_dmsfwk` | true | 分布式管理框架 | 分布式能力 |

## Preprocessor Defines

### 全局定义

| Define | 条件 | 说明 |
|--------|------|------|
| `IS_RELEASE_VERSION` | `build_variant == "user"` | 发布版本标记 |
| `FRAME_TRACE_ENABLE` | 始终（当 frame_aware_sched 可用） | 帧跟踪启用 |
| `RESOURCE_SCHEDULE_SERVICE_ENABLE` | `resourceschedule_resource_schedule_service` | 资源调度服务 |
| `POWER_MANAGER_ENABLE` | `powermgr_power_manager` | 电源管理 |
| `IMF_ENABLE` | `inputmethod_imf` | 输入法框架 |
| `MEMMGR_WINDOW_ENABLE` | `resourceschedule_memmgr` | 内存管理窗口 |
| `SOC_PERF_ENABLE` | `resourceschedule_soc_perf` | SoC 性能 |
| `SENSOR_ENABLE` | `sensors_sensor` | 传感器 |

### Scene Board 专用

| Define | 条件 | 说明 |
|--------|------|------|
| `DEVICE_STATUS_ENABLE` | `device_status_enable` | 设备状态 |
| `WM_SUBSCRIBE_MOTION_ENABLE` | `window_manager_feature_subscribe_motion` | 运动订阅 |
| `TP_FEATURE_ENABLE` | `window_manager_feature_tp_enable` | TP 功能 |
| `FOLD_ABILITY_ENABLE` | `window_manager_fold_ability` | 折叠能力 |
| `WM_SCREEN_ACTIVE_MODE_ENABLE` | `window_manager_feature_screen_active_mode` | 主动模式 |
| `WM_SCREEN_COLOR_GAMUT_ENABLE` | `window_manager_feature_screen_color_gamut` | 色域 |
| `WM_SCREEN_HDR_FORMAT_ENABLE` | `window_manager_feature_screen_hdr_format` | HDR |
| `WM_SCREEN_COLOR_SPACE_ENABLE` | `window_manager_feature_screen_color_space` | 色彩空间 |
| `WM_MULTI_SCREEN_ENABLE` | `window_manager_feature_multi_screen` | 多屏幕 |
| `WM_MULTI_SCREEN_CTL_ABILITY_ENABLE` | `window_manager_feature_multi_screen_frame_ctl` | 帧控制 |
| `WM_CAM_MODE_ABILITY_ENABLE` | `window_manager_feature_cam_mode` | 相机模式 |
| `WM_MULTI_USR_ABILITY_ENABLE` | `window_manager_feature_multi_usr` | 多用户 |
| `WINDOW_MANAGER_FEATURE_SUPPORT_DMSFWK` | `window_manager_feature_support_dmsfwk` | 分布式框架 |
| `WINDOW_MANAGER_FEATURE_SUPPORT_DSOFTBUS` | `window_manager_feature_support_dsoftbus` | 软总线 |

### 传统 WMS 专用

| Define | 条件 | 说明 |
|--------|------|------|
| `SUPPORT_SCREEN` | 非 Scene Board | 屏幕支持 |
| `SUPPORT_GRAPHICS` | 非 Scene Board | 图形支持 |
| `CONFIG_USE_JEMALLOC_DFX_INTF` | `musl_use_jemalloc` | jemalloc DFX |

## 配置使用示例

### GN 文件中检查

```gn
# windowmanager_aafwk.gni
if (window_manager_use_sceneboard) {
  # Scene Board 配置
} else {
  # 传统配置
}

if (defined(global_parts_info.powermgr_power_manager)) {
  external_deps += [ "power_manager:powermgr_client" ]
  defines += [ "POWER_MANAGER_ENABLE" ]
}
```

### C++ 代码中检查

```cpp
// 检查 Scene Board
if (SceneBoardJudgement::IsSceneBoardEnabled()) {
    // Scene Board 代码路径
} else {
    // 传统代码路径
}

// 检查编译时定义
#ifdef POWER_MANAGER_ENABLE
    // 电源管理相关代码
#endif

// 检查特性开关
#ifdef WM_MULTI_SCREEN_ENABLE
    // 多屏幕支持代码
#endif
```

## 运行时配置

### 系统参数

```bash
# 查看 Window Manager 参数
dumpsys window parameters

# 设置参数（需要 root）
param set window_manager.use_sceneboard 1
```

### 配置文件

```
system/etc/
├── wms.para              # Window Manager 参数
├── wms.para.dac          # 权限配置
└── sceneboard.config     # Scene Board 配置
```

## 相关文档

- [GN Targets](../06_GN_Targets.md)
- [目录结构](../03_Directory_Structure.md)
