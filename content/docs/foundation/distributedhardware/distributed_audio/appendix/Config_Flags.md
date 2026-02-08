# 附录：配置标志

## GN 配置选项

### 根配置 (distributedaudio.gni)

```gni
# 路径配置
distributedaudio_path = "//foundation/distributedhardware/distributed_audio"
distributedaudio_ext_path = "//foundation/distributedhardware/distributed_audio_ext"

# 编译标志
build_flags = [ "-Werror" ]

# 特性开关
declare_args() {
  # 扩展 SA 支持
  distributed_audio_extension_sa = false
  
  # 设备安全级别控制
  device_security_level_control = true
  
  # 共享缓冲区支持
  distributed_audio_shared_buffer = false
  
  # 同账号模式
  distributed_audio_same_account = false
  
  # OS 账号支持（自动检测）
  daudio_os_account = true
}
```

## 特性标志详解

### 1. distributed_audio_extension_sa

**默认值**: `false`

**功能**: 启用扩展 SA 功能，包括回声消除

**影响**:
- 添加回声消除管理器
- 加载 `libdaudio_aec_effect_processor.z.so`

**编译定义**:
```cpp
ECHO_CANNEL_ENABLE
```

**相关代码**:
```cpp
// services/audiomanager/servicesource/BUILD.gn
if (distributed_audio_extension_sa) {
  sources += [ "${services_path}/audiomanager/managersource/src/daudio_echo_cannel_manager.cpp" ]
  cflags += [ "-DECHO_CANNEL_ENABLE" ]
}
```

### 2. device_security_level_control

**默认值**: `true`

**功能**: 启用设备安全级别控制（DSLM）

**影响**:
- 设备间通信前验证安全级别
- 需要 security_device_security_level 组件

**编译定义**:
```cpp
DEVICE_SECURITY_LEVEL_ENABLE
```

**相关代码**:
```gni
// distributedaudio.gni
if (defined(global_parts_info) &&
    !defined(global_parts_info.security_device_security_level)) {
  device_security_level_control = false
}
```

### 3. distributed_audio_shared_buffer

**默认值**: `false`

**功能**: 启用共享缓冲区支持（MMAP 模式）

**影响**:
- 使用共享内存传输音频数据
- 降低数据拷贝开销

**编译定义**:
```cpp
AUDIO_SUPPORT_SHARED_BUFFER
```

**相关代码**:
```cpp
// services/audiomanager/servicesource/BUILD.gn
if (distributed_audio_shared_buffer) {
  cflags += [ "-DAUDIO_SUPPORT_SHARED_BUFFER" ]
}
```

### 4. distributed_audio_same_account

**默认值**: `false`

**功能**: 仅允许同账号设备间使用分布式音频

**影响**:
- 设备配对时验证账号
- 增强安全性

**编译定义**:
```cpp
DAUDIO_OPEN_SAME_ACCOUNT
```

### 5. daudio_os_account

**默认值**: `true`（自动检测）

**功能**: 集成 OS 账号系统

**影响**:
- 链接 OS 账号库
- 编译定义 `OS_ACCOUNT_PART`

**依赖**:
```gni
if (daudio_os_account) {
  external_deps += [
    "os_account:libaccountkits",
    "os_account:os_account_innerkits",
  ]
  defines += [ "OS_ACCOUNT_PART" ]
}
```

## 调试配置

### 调试转储（Root Build）

```gni
if (build_variant == "root") {
  defines += [
    "DUMP_DSPEAKERDEV_FILE",
    "DUMP_DMICDEV_FILE",
  ]
}
```

**功能**: 启用音频数据转储到文件，用于调试

**输出**: `/data/local/tmp/daudio_dump_*.pcm`

### 日志级别

```cpp
// common/include/daudio_log.h
#define LOG_DOMAIN 0xD004130
#define LOG_TAG "DAudio"
```

**日志标签**:
- `DAudio` - 通用日志
- `DAudioSource` - Source 端日志
- `DAudioSink` - Sink 端日志
- `DAudioTrans` - 传输层日志

## 编译配置示例

### 标准配置

```gni
# 默认配置，无需额外参数
gn gen out/target
ninja -C out/target
```

### 启用扩展功能

```gni
# 启用回声消除
gn gen out/target --args='distributed_audio_extension_sa=true'

# 启用共享缓冲区
gn gen out/target --args='distributed_audio_shared_buffer=true'

# 启用同账号模式
gn gen out/target --args='distributed_audio_same_account=true'
```

### 调试配置

```gni
# 启用调试日志
gn gen out/debug --args='is_debug=true daudio_log_level=3'

# 启用 AddressSanitizer
gn gen out/asan --args='is_asan=true'
```

### 组合配置

```gni
# 完整调试配置
gn gen out/debug --args='
  is_debug=true
  distributed_audio_extension_sa=true
  distributed_audio_shared_buffer=true
  daudio_log_level=3
'
```

## 配置文件

### SA 配置 (4805.json)

```json
{
    "process": "daudio",
    "systemability": [
        {
            "name": 4805,
            "libpath": "libdistributed_audio_source.z.so",
            "run-on-create": false,
            "distributed": true,
            "dump_level": 1,
            "min_hdi_proxy_version": [
                "libdaudio_proxy_1.0.z.so",
                "libdaudioext_proxy_2.1.z.so"
            ]
        }
    ]
}
```

### 进程配置 (daudio.cfg)

```ini
[service]
name = daudio
path = /system/bin/sa_main
uid = daudio
gid = daudio
group = system shell daudio
secon = u:r:daudio:s0

# 权限列表
privileges = [
    "ohos.permission.MICROPHONE",
    "ohos.permission.DISTRIBUTED_DATASYNC",
    "ohos.permission.ACCESS_SERVICE_DM",
    "ohos.permission.ACCESS_DISTRIBUTED_HARDWARE",
    "ohos.permission.CAPTURE_VOICE_DOWNLINK_AUDIO"
]
```

### Bundle 配置 (bundle.json)

```json
{
    "name": "@ohos/distributed_audio",
    "version": "4.0",
    "component": {
        "name": "distributed_audio",
        "subsystem": "distributedhardware",
        "features": [
            "distributed_audio_extension_sa",
            "distributed_audio_same_account"
        ],
        "rom": "2000KB",
        "ram": "6MB"
    }
}
```

## 性能调优配置

### 缓冲区配置

```cpp
// 抖动缓冲区大小
constexpr int32_t JITTER_BUFFER_SIZE = 12;  // 最大队列长度

// 环形缓冲区大小
constexpr size_t RING_BUFFER_SIZE = 8192;   // 8KB
```

### 线程配置

```cpp
// 看门狗线程间隔
constexpr int32_t WATCHDOG_INTERVAL_MS = 3000;

// 音频帧大小
constexpr int32_t FRAME_SIZE = 1920;  // 48kHz, 20ms, 2ch
```

---

*文档生成时间: 2025-02-06*
