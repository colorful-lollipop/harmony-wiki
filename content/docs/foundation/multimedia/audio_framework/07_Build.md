# Audio Framework - 构建系统

> 本文档描述 OpenHarmony 音频框架的 GN 构建系统和编译产物。

---

## 1. 构建系统概述

### 1.1 构建工具

- **构建系统**: GN (Generate Ninja)
- **编译器**: Clang/LLVM
- **链接器**: lld
- **输出**: `.so` 共享库、配置文件、测试可执行文件

### 1.2 顶层构建文件

| 文件 | 用途 |
|------|------|
| `config.gni` | Feature 开关配置 |
| `bundle.json` | 组件配置和构建分组 |
| `bluetooth_part.gni` | 蓝牙组件开关 |
| `multimedia_aafwk.gni` | AAFWK 路径配置 |
| `accessibility.gni` | 无障碍配置 |
| `window_manager.gni` | 窗口管理器配置 |
| `sensor.gni` | 传感器配置 |
| `ressche_part.gni` | 资源调度配置 |
| `appgallery.gni` | 应用市场配置 |
| `audio_framework_test_sources.gni` | 测试源文件配置 |

---

## 2. Feature 开关

### 2.1 核心 Feature 开关

**配置位置**: `config.gni:14-45`

| Feature 名称 | 默认值 | 说明 |
|-------------|--------|------|
| `audio_framework_feature_wired_audio` | true | 有线音频支持 |
| `audio_framework_feature_usb_audio` | false | USB 音频支持 |
| `audio_framework_feature_double_pnp_detect` | false | 双 PnP 检测 |
| `audio_framework_feature_dtmf_tone` | true | DTMF tone 支持 |
| `audio_framework_feature_detect_soundbox` | false | 音箱检测 |
| `audio_framework_feature_wireless_default_exclude` | false | 无线默认排除 |
| `audio_framework_feature_exclude_indirect_usb_input_device` | false | 排除间接 USB 输入设备 |
| `audio_framework_feature_opensl_es` | true | OpenSL ES 支持 |
| `audio_framework_feature_support_os_account` | true | OS 账户支持 |
| `audio_framework_feature_distributed_audio` | true | 分布式音频 |
| `audio_framework_feature_hitrace_enable` | true | HiTrace 启用 |
| `audio_framework_feature_offline_effect` | true | 离线音效 |
| `audio_framework_feature_file_io` | true | 文件 IO |
| `audio_framework_feature_inner_capturer` | true | 内部采集器 |
| `audio_framework_feature_low_latency` | true | 低延迟支持 |
| `audio_framework_feature_new_engine_flag` | true | 新引擎标志 |
| `audio_framework_feature_device_manager` | true | 设备管理器 |
| `audio_framework_feature_power_manager` | true | 电源管理器 |
| `audio_framework_feature_mutesink_enable` | true | MuteSink 启用 |
| `audio_framework_feature_audiosuite_support` | false | Audio Suite 支持 |
| `audio_framework_feature_multi_alarm_level` | false | 多级闹钟 |
| `audio_framework_feature_multi_bus` | false | 多总线 |
| `sonic_enable` | true | Sonic 库启用 |
| `speex_enable` | false | Speex 库启用 |

### 2.2 条件编译宏

**代码中使用示例**:
```cpp
// frameworks/js/napi/common/napi_audio_entry.cpp:42-44
#ifdef FEATURE_DTMF_TONE
    NapiTonePlayer::Init(env, exports);
#endif
```

**主要宏定义**:
- `FEATURE_DTMF_TONE` - DTMF tone 功能
- `FEATURE_HIVIEW_ENABLE` - HiView 日志
- `SUPPORT_LOW_LATENCY` - 低延迟支持
- `HAS_FEATURE_INNERCAPTURER` - 内部采集器

---

## 3. 构建目标

### 3.1 Framework Group

**定义位置**: `bundle.json:119-127`

```json
"fwk_group": [
  "//foundation/multimedia/audio_framework/frameworks/js/napi:audio",
  "//foundation/multimedia/audio_framework/frameworks/cj:cj_multimedia_audio_ffi",
  "//foundation/multimedia/audio_framework/frameworks/native/ohaudio:ohaudio",
  "//foundation/multimedia/audio_framework/frameworks/native/ohaudiosuite:ohaudiosuite",
  "//foundation/multimedia/audio_framework/frameworks/native/opensles:opensles",
  "//foundation/multimedia/audio_framework/frameworks/taihe:audio_framework_taihe",
  "//foundation/multimedia/audio_framework/services/audio_service:audio_sasdk"
]
```

### 3.2 Service Group

**定义位置**: `bundle.json:128-135`

```json
"service_group": [
  "//foundation/multimedia/audio_framework/sa_profile:audio_service_sa_profile",
  "//foundation/multimedia/audio_framework/services/audio_service:audio_service_packages",
  "//foundation/multimedia/audio_framework/services/audio_policy:audio_policy_packages",
  "//foundation/multimedia/audio_framework/services/audio_suite:audio_suite_packages",
  "//foundation/multimedia/audio_framework/frameworks/native/pulseaudio/modules:pa_extend_modules",
  "//foundation/multimedia/audio_framework/frameworks/native/audioclock:audio_clock"
]
```

### 3.3 主要共享库目标

| Target | 路径 | 产物 | 安装路径 |
|--------|------|------|----------|
| `audio` | `frameworks/js/napi` | audio.z.so | module/multimedia |
| `ohaudio` | `frameworks/native/ohaudio` | libohaudio.so | system/lib |
| `ohaudiosuite` | `frameworks/native/ohaudiosuite` | libohaudiosuite.so | system/lib |
| `opensles` | `frameworks/native/opensles` | libOpenSLES.so | system/lib |
| `audio_renderer` | `frameworks/native/audiorenderer` | libaudio_renderer.z.so | system/lib |
| `audio_capturer` | `frameworks/native/audiocapturer` | libaudio_capturer.z.so | system/lib |
| `audio_policy_service` | `services/audio_policy` | libaudio_policy_service.z.so | system/lib |
| `audio_common` | `services/audio_service` | libaudio_common.z.so | system/lib |

---

## 4. 编译产物

### 4.1 共享库 (.so)

**安装路径**: `/system/lib` 或 `/system/lib64`

| 产物名称 | 说明 |
|----------|------|
| `libohaudio.so` | OHAudio C API |
| `libohaudiosuite.so` | Audio Suite C API |
| `libOpenSLES.so` | OpenSL ES 兼容层 |
| `audio.z.so` | NAPI JS 接口 |
| `libaudio_renderer.z.so` | 渲染器库 |
| `libaudio_capturer.z.so` | 采集器库 |
| `libaudio_policy_service.z.so` | 策略服务库 |
| `libaudio_common.z.so` | 服务公共库 |
| `libhdiadapter_new.z.so` | HDI 适配器 |
| `libaudio_effect.z.so` | 音效处理库 |

### 4.2 配置文件

**安装路径**: `/system/etc/audio`, `/system/etc/pulse`

| 产物名称 | 路径 | 说明 |
|----------|------|------|
| `audio_effect_config.xml` | `etc/audio/` | 音效配置 |
| `audio_volume_config.xml` | `etc/audio/` | 音量配置 |
| `audio_strategy_router.xml` | `etc/audio/` | 路由策略 |
| `audio_interrupt_policy_config.xml` | `etc/audio/` | 中断策略 |
| `audio_device_privacy.xml` | `etc/audio/` | 设备隐私配置 |
| `daemon.conf` | `etc/pulse/` | PulseAudio 配置 |
| `audio_config.para` | `etc/param/` | 参数配置 |

### 4.3 测试可执行文件

| Target | 路径 | 说明 |
|--------|------|------|
| `audio_renderer_test` | `frameworks/native/audiorenderer` | 渲染器测试 |
| `audio_capturer_test` | `frameworks/native/audiocapturer` | 采集器测试 |
| `oh_audio_renderer_test` | `frameworks/native/ohaudio` | OHAudio 渲染测试 |
| `oh_audio_capturer_test` | `frameworks/native/ohaudio` | OHAudio 采集测试 |
| `audio_opensles_player_test` | `frameworks/native/opensles` | OpenSL ES 播放测试 |
| `audio_policy_test` | `services/audio_policy` | 策略测试 |

---

## 5. 编译配置

### 5.1 安全编译选项

**CFI (Control Flow Integrity)**:
```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  cfi_vcall_icall_only = true
  integer_overflow = true
  ubsan = true
  boundary_sanitize = true
  debug = false
}
```

**分支保护** (ARM64):
```gn
branch_protector_ret = "pac_ret"
```

### 5.2 版本脚本

部分库使用版本脚本控制符号导出:
```gn
version_script = "../../../audio_framework.versionscript"
```

---

## 6. 依赖关系

### 6.1 外部依赖

**核心依赖**:
- `c_utils:utils` - C 工具库
- `hilog:libhilog` - 日志系统
- `ipc:ipc_single/ipc_core` - IPC 框架
- `samgr:samgr_proxy` - 系统 Ability 管理器

**服务依赖**:
- `ability_runtime` - Ability 运行时
- `bundle_framework` - 包管理框架
- `power_manager` - 电源管理
- `device_manager` - 设备管理
- `bluetooth` - 蓝牙服务
- `hiviewdfx_hitrace/hiview` - 性能分析

**多媒体依赖**:
- `media_foundation` - 媒体基础
- `pulseaudio` - PulseAudio 音频服务器
- `opensles` - OpenSL ES

### 6.2 模块依赖图

```
audio_policy_packages
    ├── audio_policy_service
    ├── audio_foundation
    ├── audio_manager_client
    └── 配置文件

audio_service_packages
    ├── audio_common
    ├── audio_service
    └── audio_server_init

ohaudio
    ├── audio_client
    ├── audio_engine_manager
    ├── audio_policy_manager
    ├── audio_renderer
    └── audio_capturer
```

---

## 7. 构建命令

### 7.1 完整构建

```bash
# 生成构建配置
gn gen out --args='...'

# 构建整个音频框架
ninja -C out audio_framework
```

### 7.2 特定目标构建

```bash
# 构建 N-API 接口
ninja -C out //foundation/multimedia/audio_framework/frameworks/js/napi:audio

# 构建 OHAudio C API
ninja -C out //foundation/multimedia/audio_framework/frameworks/native/ohaudio:ohaudio

# 构建策略服务
ninja -C out //foundation/multimedia/audio_framework/services/audio_policy:audio_policy_service

# 构建测试
ninja -C out //foundation/multimedia/audio_framework/test:audio_unit_test
```

### 7.3 查询命令

```bash
# 查询目标信息
gn desc out //foundation/multimedia/audio_framework/frameworks/js/napi:audio

# 查询依赖关系
gn deps out //foundation/multimedia/audio_framework/services/audio_policy:audio_policy_service
```

---

## 8. 构建变体

### 8.1 调试构建

```gn
is_debug = true
audio_framework_enable_unittest_debug = true
```

### 8.2 模拟器构建

```gn
IS_EMULATOR = true
```

### 8.3 Root 版本构建

```gn
AUDIO_BUILD_VARIANT_ROOT = true
```

---

*最后更新: 2026-02-07*
