# GN 构建系统

## 构建配置

### 根配置文件

```gni
# distributedaudio.gni
distributedaudio_path = "//foundation/distributedhardware/distributed_audio"
distributedaudio_ext_path = "//foundation/distributedhardware/distributed_audio_ext"

# 特性开关
declare_args() {
  distributed_audio_extension_sa = false    # 扩展 SA 支持
  device_security_level_control = true      # 设备安全级别控制
  distributed_audio_shared_buffer = false   # 共享缓冲区支持
  distributed_audio_same_account = false    # 同账号模式
  daudio_os_account = true                  # OS 账号支持
}

# 编译标志
build_flags = [ "-Werror" ]
```

## 构建目标列表

### 核心共享库

| 目标名 | 类型 | 路径 | 输出文件 | 说明 |
|--------|------|------|----------|------|
| **distributed_audio_handler** | shared_library | audiohandler/ | libdistributed_audio_handler.z.so | 音频处理器 |
| **distributed_audio_utils** | shared_library | services/common/ | libdistributed_audio_utils.z.so | 公共工具 |
| **distributed_audio_source** | shared_library | services/audiomanager/servicesource/ | libdistributed_audio_source.z.so | Source 服务（SA 4805） |
| **distributed_audio_sink** | shared_library | services/audiomanager/servicesink/ | libdistributed_audio_sink.z.so | Sink 服务（SA 4806） |
| **distributed_audio_encode_transport** | shared_library | services/audiotransport/senderengine/ | libdistributed_audio_encode_transport.z.so | 编码传输 |
| **distributed_audio_decode_transport** | shared_library | services/audiotransport/receiverengine/ | libdistributed_audio_decode_transport.z.so | 解码传输 |
| **distributed_audio_source_sdk** | shared_library | interfaces/inner_kits/native_cpp/audio_source/ | libdistributed_audio_source_sdk.z.so | Source SDK |
| **distributed_audio_sink_sdk** | shared_library | interfaces/inner_kits/native_cpp/audio_sink/ | libdistributed_audio_sink_sdk.z.so | Sink SDK |

### SA 配置目标

| 目标名 | 类型 | 路径 | 输出 | 说明 |
|--------|------|------|------|------|
| **daudio_sa_profile** | ohos_sa_profile | sa_profile/ | SA 配置文件 | 4805.json, 4806.json |
| **daudio.cfg** | ohos_prebuilt_etc | sa_profile/ | init/daudio.cfg | 进程配置 |

### 测试目标

| 目标名 | 类型 | 路径 | 说明 |
|--------|------|------|------|
| **audio_distributed_test** | executable | services/test_example/ | 示例测试程序 |

## 目标详细定义

### 1. distributed_audio_source（Source 服务）

```gn
# services/audiomanager/servicesource/BUILD.gn
ohos_shared_library("distributed_audio_source") {
  branch_protector_ret = "pac_ret"
  sanitize = {
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    integer_overflow = true
    ubsan = true
  }
  stack_protector_ret = true

  include_dirs = [
    "include",
    "${audio_client_path}/micclient/include",
    "${audio_client_path}/spkclient/include",
    "${audio_control_path}/controlsource/include",
    "${audio_hdi_proxy_path}/include",
    "${audio_processor_path}/interface",
    "${audio_transport_path}/audioctrltransport/include",
    "${audio_transport_path}/interface",
    "${audio_transport_path}/receiverengine/include",
    "${audio_transport_path}/senderengine/include",
    # ... 更多 include
  ]

  sources = [
    "${audio_control_path}/controlsource/src/daudio_source_dev_ctrl_manager.cpp",
    "${audio_hdi_proxy_path}/src/daudio_hdi_handler.cpp",
    "${audio_hdi_proxy_path}/src/daudio_manager_callback.cpp",
    "${audio_transport_path}/audioctrltransport/src/daudio_source_ctrl_trans.cpp",
    "${common_path}/dfx_utils/src/daudio_hidumper.cpp",
    "${interfaces_path}/inner_kits/native_cpp/audio_sink/src/daudio_sink_proxy.cpp",
    "${interfaces_path}/inner_kits/native_cpp/audio_source/src/daudio_source_proxy.cpp",
    "${services_path}/audiomanager/managersource/src/daudio_source_dev.cpp",
    "${services_path}/audiomanager/managersource/src/daudio_source_manager.cpp",
    "${services_path}/audiomanager/managersource/src/daudio_source_mgr_callback.cpp",
    "${services_path}/audiomanager/managersource/src/dmic_dev.cpp",
    "${services_path}/audiomanager/managersource/src/dspeaker_dev.cpp",
    "src/daudio_ipc_callback_proxy.cpp",
    "src/daudio_source_service.cpp",
    "src/daudio_source_stub.cpp",
  ]

  deps = [
    "${audio_transport_path}/receiverengine:distributed_audio_decode_transport",
    "${audio_transport_path}/senderengine:distributed_audio_encode_transport",
    "${distributedaudio_path}/audiohandler:distributed_audio_handler",
    "${services_path}/common:distributed_audio_utils",
  ]

  external_deps = [
    "access_token:libaccesstoken_sdk",
    "access_token:libtokenid_sdk",
    "access_token:libtokensetproc_shared",
    "audio_framework:audio_capturer",
    "audio_framework:audio_client",
    "audio_framework:audio_renderer",
    "cJSON:cjson",
    "c_utils:utils",
    "device_manager:devicemanagersdk",
    "distributed_hardware_fwk:distributed_av_receiver",
    "distributed_hardware_fwk:distributed_av_sender",
    "distributed_hardware_fwk:distributedhardwareutils",
    "distributed_hardware_fwk:libdhfwk_sdk",
    "drivers_interface_distributed_audio:libdaudio_proxy_1.0",
    "drivers_interface_distributed_audio:libdaudioext_proxy_2.1",
    "dsoftbus:softbus_client",
    "eventhandler:libeventhandler",
    "hdf_core:libhdf_utils",
    "hdf_core:libhdi",
    "hicollie:libhicollie",
    "hilog:libhilog",
    "hisysevent:libhisysevent",
    "hitrace:hitrace_meter",
    "ipc:ipc_core",
    "ipc:ipc_single",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
  ]

  if (daudio_os_account) {
    external_deps += [
      "os_account:libaccountkits",
      "os_account:os_account_innerkits",
    ]
  }

  defines = [
    "HI_LOG_ENABLE",
    "LOG_DOMAIN=0xD004130",
  ]

  if (distributed_audio_extension_sa) {
    defines += [ "ECHO_CANNEL_ENABLE" ]
  }
  if (distributed_audio_shared_buffer) {
    defines += [ "AUDIO_SUPPORT_SHARED_BUFFER" ]
  }

  ldflags = [
    "-fpie",
    "-Wl,-z,relro",
    "-Wl,-z,now",
  ]
}
```

### 2. distributed_audio_utils（公共工具）

```gn
# services/common/BUILD.gn
ohos_shared_library("distributed_audio_utils") {
  branch_protector_ret = "pac_ret"
  sanitize = {
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    integer_overflow = true
    ubsan = true
  }
  stack_protector_ret = true

  configs = [ ":daudio_common_private_config" ]
  public_configs = [ ":daudio_common_pub_config" ]

  sources = [
    "${common_path}/dfx_utils/src/daudio_hisysevent.cpp",
    "${common_path}/dfx_utils/src/daudio_hitrace.cpp",
    "${common_path}/dfx_utils/src/daudio_radar.cpp",
    "${common_path}/src/daudio_latency_test.cpp",
    "${common_path}/src/daudio_ringbuffer.cpp",
    "${common_path}/src/daudio_util.cpp",
    "audiodata/src/audio_data.cpp",
  ]

  external_deps = [
    "cJSON:cjson",
    "c_utils:utils",
    "distributed_hardware_fwk:distributedhardwareutils",
    "dsoftbus:softbus_client",
    "hilog:libhilog",
    "hisysevent:libhisysevent",
    "hitrace:hitrace_meter",
    "init:libbegetutil",
  ]
}
```

## 依赖关系图

```
distributed_audio_source
├── distributed_audio_encode_transport
│   └── distributed_audio_utils
├── distributed_audio_decode_transport
│   └── distributed_audio_utils
├── distributed_audio_handler
│   └── distributed_audio_utils
└── distributed_audio_utils

distributed_audio_sink
├── distributed_audio_encode_transport
│   └── distributed_audio_utils
├── distributed_audio_decode_transport
│   └── distributed_audio_utils
├── distributed_audio_sink_sdk
│   └── distributed_audio_utils
└── distributed_audio_utils
```

## 外部依赖

### 核心依赖组件

```
audio_framework          # 音频框架
├── audio_capturer       # 音频采集
├── audio_client         # 音频客户端
└── audio_renderer       # 音频渲染

access_token             # 访问令牌
├── libaccesstoken_sdk   # SDK
├── libtokenid_sdk       # Token ID
└── libtokensetproc_shared

distributed_hardware_fwk # 分布式硬件框架
├── distributed_av_receiver  # AV 接收
├── distributed_av_sender    # AV 发送
├── distributedhardwareutils
└── libdhfwk_sdk

dsoftbus                 # 软总线
└── softbus_client

hdf_core                 # HDF 核心
├── libhdf_utils
└── libhdi

ipc                      # IPC
├── ipc_core
└── ipc_single

其他:
├── cJSON:cjson
├── c_utils:utils
├── device_manager:devicemanagersdk
├── drivers_interface_distributed_audio
├── eventhandler:libeventhandler
├── hicollie:libhicollie
├── hilog:libhilog
├── hisysevent:libhisysevent
├── hitrace:hitrace_meter
├── safwk:system_ability_fwk
└── samgr:samgr_proxy
```

## 编译特性开关

| 特性 | GN 变量 | 编译定义 | 效果 |
|------|---------|----------|------|
| 扩展 SA | `distributed_audio_extension_sa` | `ECHO_CANNEL_ENABLE` | 启用回声消除 |
| 共享缓冲区 | `distributed_audio_shared_buffer` | `AUDIO_SUPPORT_SHARED_BUFFER` | 启用共享缓冲区 |
| 同账号 | `distributed_audio_same_account` | `DAUDIO_OPEN_SAME_ACCOUNT` | 同账号模式 |
| OS 账号 | `daudio_os_account` | `OS_ACCOUNT_PART` | OS 账号集成 |
| 安全级别 | `device_security_level_control` | `DEVICE_SECURITY_LEVEL_ENABLE` | DSLM 安全 |
| Root 构建 | `build_variant=="root"` | `DUMP_DSPEAKERDEV_FILE`, `DUMP_DMICDEV_FILE` | 调试转储 |

## 构建命令

### 构建所有目标

```bash
# 编译整个组件
ninja -C out/target distributed_audio_source distributed_audio_sink

# 或使用 GN 构建
 gn gen out/target
ninja -C out/target //foundation/distributedhardware/distributed_audio/services/audiomanager/servicesource:distributed_audio_source
```

### 构建特定目标

```bash
# Source 服务
ninja -C out/target distributed_audio_source

# Sink 服务
ninja -C out/target distributed_audio_sink

# SDK
ninja -C out/target distributed_audio_source_sdk distributed_audio_sink_sdk

# 工具库
ninja -C out/target distributed_audio_utils

# 传输库
ninja -C out/target distributed_audio_encode_transport distributed_audio_decode_transport
```

## 安全编译选项

```gn
# 所有目标启用的安全选项
branch_protector_ret = "pac_ret"    # 分支保护
stack_protector_ret = true           # 栈保护

sanitize = {
  boundary_sanitize = true           # 边界检查
  cfi = true                         # 控制流完整性
  cfi_cross_dso = true               # 跨 DSO CFI
  integer_overflow = true            # 整数溢出检查
  ubsan = true                       # 未定义行为检查
}

ldflags = [
  "-fpie",                           # 位置独立可执行
  "-Wl,-z,relro",                    # 重定位只读
  "-Wl,-z,now",                      # 立即绑定
]
```

---

*文档生成时间: 2025-02-06*
