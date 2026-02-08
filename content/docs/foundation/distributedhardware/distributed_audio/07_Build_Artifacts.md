# 编译产物

## 产物清单

### 共享库 (.so)

| 文件名 | 来源目标 | 说明 | 安装路径 |
|--------|----------|------|----------|
| **libdistributed_audio_source.z.so** | distributed_audio_source | Source 服务库（SA 4805） | /system/lib/ |
| **libdistributed_audio_sink.z.so** | distributed_audio_sink | Sink 服务库（SA 4806） | /system/lib/ |
| **libdistributed_audio_handler.z.so** | distributed_audio_handler | 音频处理器 | /system/lib/ |
| **libdistributed_audio_utils.z.so** | distributed_audio_utils | 公共工具库 | /system/lib/ |
| **libdistributed_audio_source_sdk.z.so** | distributed_audio_source_sdk | Source SDK | /system/lib/ |
| **libdistributed_audio_sink_sdk.z.so** | distributed_audio_sink_sdk | Sink SDK | /system/lib/ |
| **libdistributed_audio_encode_transport.z.so** | distributed_audio_encode_transport | 编码传输 | /system/lib/ |
| **libdistributed_audio_decode_transport.z.so** | distributed_audio_decode_transport | 解码传输 | /system/lib/ |

### 配置文件

| 文件名 | 来源目标 | 说明 | 安装路径 |
|--------|----------|------|----------|
| **4805.json** | daudio_sa_profile | Source SA 配置 | /system/profile/ |
| **4806.json** | daudio_sa_profile | Sink SA 配置 | /system/profile/ |
| **daudio.cfg** | daudio.cfg | 进程配置（权限、SELinux） | /system/etc/init/ |

### 头文件

| 路径 | 来源 | 说明 |
|------|------|------|
| `//foundation/distributedhardware/distributed_audio/interfaces/inner_kits/native_cpp/audio_sink/include/idaudio_sink.h` | distributed_audio_sink_sdk | Sink 接口 |
| `//foundation/distributedhardware/distributed_audio/interfaces/inner_kits/native_cpp/audio_source/include/idaudio_source.h` | distributed_audio_source_sdk | Source 接口 |
| `//foundation/distributedhardware/distributed_audio/services/common/audiodata/include/audio_data.h` | distributed_audio_utils | 数据结构 |
| `//foundation/distributedhardware/distributed_audio/services/common/audioparam/audio_param.h` | distributed_audio_utils | 参数定义 |

## 产物映射关系

### BUILD.gn → 产物

```
audiohandler:distributed_audio_handler
└── libdistributed_audio_source.z.so (依赖)

services/audiomanager/servicesource:distributed_audio_source
├── libdistributed_audio_source.z.so
├── 依赖: libdistributed_audio_encode_transport.z.so
├── 依赖: libdistributed_audio_decode_transport.z.so
├── 依赖: libdistributed_audio_handler.z.so
└── 依赖: libdistributed_audio_utils.z.so

services/audiomanager/servicesink:distributed_audio_sink
├── libdistributed_audio_sink.z.so
├── 依赖: libdistributed_audio_encode_transport.z.so
├── 依赖: libdistributed_audio_decode_transport.z.so
├── 依赖: libdistributed_audio_sink_sdk.z.so
└── 依赖: libdistributed_audio_utils.z.so

services/audiotransport/senderengine:distributed_audio_encode_transport
└── libdistributed_audio_encode_transport.z.so

services/audiotransport/receiverengine:distributed_audio_decode_transport
└── libdistributed_audio_decode_transport.z.so

interfaces/inner_kits/native_cpp/audio_source:distributed_audio_source_sdk
└── libdistributed_audio_source_sdk.z.so

interfaces/inner_kits/native_cpp/audio_sink:distributed_audio_sink_sdk
└── libdistributed_audio_sink_sdk.z.so

services/common:distributed_audio_utils
└── libdistributed_audio_utils.z.so
```

## 运行时加载关系

### 进程: daudio

```
daudio (进程)
├── SA 4805 (libdistributed_audio_source.z.so)
│   ├── libdistributed_audio_encode_transport.z.so
│   ├── libdistributed_audio_decode_transport.z.so
│   ├── libdistributed_audio_handler.z.so
│   └── libdistributed_audio_utils.z.so
│
└── SA 4806 (libdistributed_audio_sink.z.so)
    ├── libdistributed_audio_encode_transport.z.so
    ├── libdistributed_audio_decode_transport.z.so
    ├── libdistributed_audio_sink_sdk.z.so
    └── libdistributed_audio_utils.z.so
```

### 库依赖树

```
libdistributed_audio_source.z.so
├── libdistributed_audio_encode_transport.z.so
│   └── libdistributed_audio_utils.z.so
├── libdistributed_audio_decode_transport.z.so
│   └── libdistributed_audio_utils.z.so
├── libdistributed_audio_handler.z.so
│   └── libdistributed_audio_utils.z.so
└── libdistributed_audio_utils.z.so
    ├── libhilog.z.so
    ├── libhisysevent.z.so
    ├── libhitrace_meter.z.so
    ├── libsoftbus_client.z.so
    └── libcjson.z.so

libdistributed_audio_sink.z.so
├── libdistributed_audio_encode_transport.z.so
├── libdistributed_audio_decode_transport.z.so
├── libdistributed_audio_sink_sdk.z.so
│   └── libdistributed_audio_utils.z.so
└── libdistributed_audio_utils.z.so
```

## 运行时库加载

### 系统启动

```
1. init 进程读取 /system/etc/init/daudio.cfg
2. 启动 daudio 进程
3. 加载 SA 配置文件 4805.json, 4806.json
4. 按需加载 libdistributed_audio_source.z.so 或 libdistributed_audio_sink.z.so
```

### 动态库加载

```cpp
// 动态加载 AV 引擎库（dlopen）
// services/audiomanager/managersource/src/daudio_source_manager.cpp:514
void *handler = dlopen("libdistributed_av_sender.z.so", RTLD_LAZY | RTLD_NODELETE);
void *handler = dlopen("libdistributed_av_receiver.z.so", RTLD_LAZY | RTLD_NODELETE);

// 动态加载回声消除库（如果启用）
// services/audiomanager/managersource/src/daudio_echo_cannel_manager.cpp:329
void *handler = dlopen("libdaudio_aec_effect_processor.z.so", RTLD_LAZY | RTLD_NODELETE);
```

## 产物大小

### ROM 占用

| 组件 | 估算大小 |
|------|----------|
| libdistributed_audio_source.z.so | ~500KB |
| libdistributed_audio_sink.z.so | ~500KB |
| libdistributed_audio_handler.z.so | ~100KB |
| libdistributed_audio_utils.z.so | ~200KB |
| libdistributed_audio_encode_transport.z.so | ~200KB |
| libdistributed_audio_decode_transport.z.so | ~200KB |
| libdistributed_audio_source_sdk.z.so | ~100KB |
| libdistributed_audio_sink_sdk.z.so | ~100KB |
| **总计** | **~2000KB** |

### RAM 占用

| 组件 | 估算大小 |
|------|----------|
| 代码段 | ~2MB |
| 数据段（静态） | ~1MB |
| 堆（运行时） | ~2-3MB |
| 栈（线程） | ~1MB |
| **总计** | **~6MB** |

## 安装路径

### 系统分区

```
/system/
├── lib/
│   ├── libdistributed_audio_source.z.so
│   ├── libdistributed_audio_sink.z.so
│   ├── libdistributed_audio_handler.z.so
│   ├── libdistributed_audio_utils.z.so
│   ├── libdistributed_audio_source_sdk.z.so
│   ├── libdistributed_audio_sink_sdk.z.so
│   ├── libdistributed_audio_encode_transport.z.so
│   └── libdistributed_audio_decode_transport.z.so
│
├── profile/
│   ├── 4805.json
│   └── 4806.json
│
└── etc/init/
    └── daudio.cfg
```

### 配置文件示例

#### 4805.json

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

#### daudio.cfg

```ini
[service]
name = daudio
path = /system/bin/sa_main
uid = daudio
gid = daudio
group = system shell daudio
secon = u:r:daudio:s0
privileges = [
    "ohos.permission.MICROPHONE",
    "ohos.permission.DISTRIBUTED_DATASYNC",
    "ohos.permission.ACCESS_SERVICE_DM",
    "ohos.permission.ACCESS_DISTRIBUTED_HARDWARE",
    "ohos.permission.CAPTURE_VOICE_DOWNLINK_AUDIO"
]
```

## 调试符号

### 符号文件

```
/out/target/symbols/system/lib/
├── libdistributed_audio_source.z.so
├── libdistributed_audio_sink.z.so
└── ...
```

### 调试版本

```bash
# 编译调试版本
gn gen out/debug --args='is_debug=true'
ninja -C out/debug distributed_audio_source distributed_audio_sink
```

---

*文档生成时间: 2025-02-06*
