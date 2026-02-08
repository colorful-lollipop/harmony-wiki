# Audio Framework - 特性开关清单

## config.gni 特性开关

### 核心功能

| 开关 | 默认值 | 类型 | 描述 |
|------|--------|------|------|
| `audio_framework_feature_wired_audio` | true | bool | 有线音频支持 |
| `audio_framework_feature_usb_audio` | false | bool | USB 音频支持 |
| `audio_framework_feature_dtmf_tone` | true | bool | DTMF 音调支持 |
| `audio_framework_feature_double_pnp_detect` | false | bool | 双设备插拔检测 |
| `audio_framework_feature_distributed_audio` | true | bool | 分布式音频 |
| `audio_framework_feature_low_latency` | true | bool | 低延迟播放 |
| `audio_framework_feature_inner_capturer` | true | bool | 内置采集器 |
| `audio_framework_feature_opensl_es` | true | bool | OpenSL ES 兼容 |
| `audio_framework_feature_file_io` | true | bool | 文件 I/O 支持 |
| `audio_framework_feature_multi_bus` | false | bool | 多总线支持 |

### 高级功能

| 开关 | 默认值 | 类型 | 描述 |
|------|--------|------|------|
| `audio_framework_feature_audiosuite_support` | false | bool | Audio Suite 支持 |
| `audio_framework_feature_offline_effect` | true | bool | 离线音效 |
| `audio_framework_feature_hitrace_enable` | true | bool | HiTrace 追踪 |
| `audio_framework_feature_hiview_enable` | true | bool | HiView DFX |
| `audio_framework_feature_mutesink_enable` | true | bool | 静音 Sink 支持 |
| `audio_framework_feature_multi_alarm_level` | false | bool | 多闹钟级别 |

### 条件编译开关

| 开关 | 默认值 | 条件 | 描述 |
|------|--------|------|------|
| `audio_framework_feature_new_napi` | true | - | 新 N-API |
| `audio_framework_feature_new_engine_flag` | true | - | 新引擎标志 |
| `sonic_enable` | true | - | Sonic 音效 |
| `speex_enable` | false | - | Speex 压缩 |

### 依赖条件开关

| 开关 | 依赖 | 条件 |
|------|------|------|
| `audio_framework_feature_usb_audio` | `usb_usb_manager` | 如存在则启用 |
| `audio_framework_feature_distributed_audio` | `hdf_drivers_interface_distributed_audio` | 如存在则启用 |
| `audio_framework_feature_hitrace_enable` | `hiviewdfx_hitrace` | 如存在则启用 |
| `audio_framework_feature_input` | `multimodalinput_input` | 如存在则启用 |
| `audio_framework_feature_power_manager` | `powermgr_power_manager` | 如存在则启用 |
| `audio_framework_feature_device_manager` | `distributedhardware_device_manager` | 如存在则启用 |
| `audio_framework_feature_call_manager_enable` | `telephony_call_manager` | 如存在则启用 |
| `audio_telephony_core_service_enable` | `telephony_core_service` | 如存在则启用 |
| `audio_telephony_cellular_data_enable` | `telephony_cellular_data` | 如存在则启用 |

**证据来源**: `config.gni:14-123`

## bundle.json features

```json
"features": [
  "audio_framework_feature_wired_audio",
  "audio_framework_feature_usb_audio",
  "audio_framework_feature_double_pnp_detect",
  "audio_framework_feature_dtmf_tone",
  "audio_framework_feature_detect_soundbox",
  "audio_framework_feature_wireless_default_exclude",
  "audio_framework_feature_exclude_indirect_usb_input_device",
  "audio_framework_feature_opensl_es",
  "audio_framework_suport_svsession_manager",
  "audio_framework_feature_support_os_account",
  "audio_framework_feature_hitrace_enable",
  "audio_framework_feature_offline_effect",
  "audio_framework_feature_distributed_audio",
  "audio_framework_feature_file_io",
  "audio_framework_feature_inner_capturer",
  "audio_framework_feature_low_latency",
  "audio_framework_feature_device_manager",
  "audio_framework_feature_mutesink_enable",
  "audio_framework_feature_audiosuite_support",
  "audio_framework_feature_multi_alarm_level",
  "audio_framework_feature_multi_bus"
]
```

**证据来源**: `bundle.json:28-50`

## 使用示例

### 启用 USB 音频

```bash
# 在产品配置中添加
export audio_framework_feature_usb_audio = true
```

### 禁用 OpenSL ES

```bash
# 减少二进制大小
export audio_framework_feature_opensl_es = false
```

### 启用多闹钟级别

```bash
export audio_framework_feature_multi_alarm_level = true
```

## 注意事项

1. **条件开关**: 部分特性依赖其他系统组件，确保相关组件已启用
2. **二进制大小**: 禁用不需要的特性可以减小最终二进制大小
3. **功能兼容性**: 禁用某些特性可能影响其他功能
4. **调试**: `audio_framework_feature_hitrace_enable` 对性能分析很有帮助
