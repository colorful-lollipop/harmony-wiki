# 08_故障排查指南

## 常见构建问题

### 1. 编译失败 - 找不到头文件

**错误信息**:
```
fatal error: 'native_avcodec_base.h' file not found
```

**排查步骤**:

1. 检查 `config.gni` 中的 `av_codec_root_dir` 是否正确配置
2. 确认 `include_dirs` 包含 `interfaces/kits/c` 目录
3. 验证头文件是否存在

**证据**: `interfaces/kits/c/BUILD.gn:17-18`
```gn
ohos_shared_headers("capi_packages") {
  include_dirs = [ "$av_codec_root_dir/interfaces/kits/c" ]
```

**解决方案**:
```bash
# 重新同步 gn 配置
hb set
hb build -f
```

---

### 2. 链接失败 - 找不到符号

**错误信息**:
```
undefined reference to 'OH_VideoDecoder_CreateByMime'
```

**排查步骤**:

1. 检查是否链接了正确的库 (`libnative_media_vdec.so`)
2. 确认 `external_deps` 配置正确

**证据**: `interfaces/kits/c/BUILD.gn:334-341`
```gn
external_deps = [
  "c_utils:utils",
  "graphic_surface:surface",
  "hilog:libhilog",
  "ipc:ipc_single",
  "media_foundation:media_foundation",
  "media_foundation:native_media_core",
]
```

**解决方案**:
```bash
# 确保链接了正确的库
hb build --gn-flags '--args av_codec_support_capi=true'
```

---

### 3. SA 启动失败

**错误信息**:
```
Failed to start service av_codec
```

**排查步骤**:

1. 检查 SA 配置文件是否正确
2. 验证 `AV_CODEC_SERVICE_ID` (3011) 是否冲突
3. 查看系统日志确认启动失败原因

**证据**: `services/services/sa_avcodec/server/avcodec_server.cpp:36`
```cpp
REGISTER_SYSTEM_ABILITY_BY_ID(AVCodecServer, AV_CODEC_SERVICE_ID, true)
```

**排查命令**:
```bash
# 查看 SA 状态
hisysevent -l | grep av_codec

# 查看服务日志
hilog | grep -E "av_codec|AVCodec"
```

---

## 运行时问题

### 1. 解码器创建失败

**现象**: `OH_VideoDecoder_CreateByMime()` 返回 `nullptr`

**可能原因**:

| 原因 | 检查方法 | 解决方案 |
|------|----------|----------|
| MIME 类型不支持 | 检查 `native_avcapability.h` | 使用支持的 MIME |
| 服务未启动 | 检查 SA 状态 | 重启媒体服务 |
| 权限不足 | 检查 `access_token` | 申请必要权限 |

**排查命令**:
```bash
# 查看支持的 MIME 类型
OH_AVCodec_GetCapability("video/avc", false);

# 检查能力列表
hdc shell
cat /sys/bus/media/devices/ | grep avc
```

---

### 2. 解码输出画面异常

**现象**: 花屏、卡顿或色彩异常

**可能原因**:

| 原因 | 检查方法 | 解决方案 |
|------|----------|----------|
| 尺寸配置错误 | 检查 `OH_MD_KEY_WIDTH/HEIGHT` | 匹配源视频尺寸 |
| 帧率配置错误 | 检查 `OH_MD_KEY_FRAME_RATE` | 设置正确帧率 |
| Surface 配置错误 | 检查 `SetSurface()` 参数 | 验证 Surface 格式 |

**代码示例**:
```cpp
// 正确配置
OH_AVFormat *format = OH_AVFormat_Create();
OH_AVFormat_SetIntValue(format, OH_MD_KEY_WIDTH, 1920);
OH_AVFormat_SetIntValue(format, OH_MD_KEY_HEIGHT, 1080);
OH_AVFormat_SetIntValue(format, OH_MD_KEY_FRAME_RATE, 30);
OH_AVFormat_SetIntValue(format, OH_MD_KEY_PIXEL_FORMAT, pixelFormat);
OH_VideoDecoder_Configure(codec, format);
```

---

### 3. 解封装 Seek 失败

**现象**: `OH_AVDemuxer_SeekToTime()` 返回错误

**可能原因**:

| 原因 | 检查方法 |
|------|----------|
| 时间戳越界 | 检查 `seekTime` 是否在媒体时长范围内 |
| 未选择轨道 | 确认已调用 `SelectTrackByID()` |
| 不支持 Seek | 检查媒体格式是否支持 Seek |

**解决方案**:
```cpp
// 1. 获取媒体时长
int64_t duration;
OH_AVFormat_GetLongValue(sourceFormat, OH_MD_KEY_DURATION, &duration);

// 2. 选择轨道
OH_AVDemuxer_SelectTrackByID(demuxer, trackId);

// 3. Seek 到合法时间
OH_AVDemuxer_SeekToTime(demuxer, std::min(seekTime, duration));
```

---

### 4. 封装写入失败

**现象**: `OH_AVMuxer_WriteSampleBuffer()` 返回错误

**可能原因**:

| 原因 | 检查方法 | 解决方案 |
|------|----------|----------|
| 文件描述符无效 | 检查 `fd` 是否已打开 | 重新打开文件 |
| 磁盘空间不足 | 检查 `df` | 清理磁盘 |
| 轨道未添加 | 检查 `AddTrack()` 返回值 | 添加轨道后再写入 |

**排查命令**:
```bash
# 检查磁盘空间
df -h /path/to/output

# 检查文件权限
ls -la /path/to/output
```

---

## 日志定位

### 关键日志标签

| 标签 | 用途 |
|------|------|
| `AVCodec` | 编解码器日志 |
| `Demuxer` | 解封装日志 |
| `Muxer` | 封装日志 |
| `CodecServer` | 服务日志 |

### 日志查看命令

```bash
# 查看所有 AVCodec 日志
hilog | grep -i "avcodec\|AVCodec"

# 查看解封装日志
hilog | grep -i "demuxer\|Demuxer"

# 查看错误日志
hilog | grep -i "error\|Error" | grep av_codec
```

### DFX 日志

**证据**: `services/dfx/` 目录包含 DFX 实现

```bash
# 查看 DFX 统计信息
hdc shell
hidumper -s avcodec

# 查看系统事件
hisysevent -e | grep av_codec
```

---

## 性能问题

### 1. 解码卡顿

**排查步骤**:

1. 检查是否为硬件解码器
2. 验证 Surface 配置是否正确
3. 检查是否有丢帧

**优化建议**:

```cpp
// 使用硬件加速
OH_AVCodec *codec = OH_VideoDecoder_CreateByMime("video/avc");

// 正确配置输出 Surface
OHNativeWindow *window = nullptr;
OH_VideoDecoder_SetSurface(codec, window);

// 使用 Buffer 模式提高效率
OH_AVBuffer *buffer = OH_AVBuffer_Create(capacity);
```

---

### 2. 内存占用过高

**排查步骤**:

1. 检查 Buffer 释放是否及时
2. 确认未创建过多 Codec 实例
3. 使用内存分析工具

**优化建议**:

```cpp
// 及时释放资源
OH_VideoDecoder_Stop(codec);
OH_VideoDecoder_Release(codec);
OH_AVFormat_Destroy(format);

// 限制 Buffer 数量
OH_AVFormat_SetIntValue(format, OH_MD_KEY_MAX_INPUT_BUFFER_COUNT, 16);
```

---

## 调试工具

### 1. xcollie 监控

**用途**: 监控编解码操作耗时

**证据**: `config.gni:27`
```gni
av_codec_support_xcollie = true
```

**使用**:
```cpp
// 开启监控
XCollieHolder holder("VideoDecode", 5000);

// 超时会自动打印日志
```

---

### 2. Bitstream Dump

**用途**: 转储码流用于分析

**证据**: `config.gni:28`
```gni
av_codec_support_bitstream_dump = true
```

**配置**:
```bash
# 开启 bitstream dump
hdc shell
setprop debug.avcodec.dump.enable 1
```

---

## 常见错误码速查

| 错误码 | 含义 | 排查方向 |
|--------|------|----------|
| `AV_ERR_OPERATE_NOT_PERMIT` | 操作不允许 | 权限检查 |
| `AV_ERR_NO_MEMORY` | 内存不足 | 资源释放 |
| `AV_ERR_DATA_INVALID` | 数据无效 | 输入参数校验 |
| `AV_ERR_IO` | IO 错误 | 文件/网络检查 |
| `AV_ERR_TIMEOUT` | 超时 | 性能分析 |
| `AV_ERR_SERVICE_DIED` | 服务已终止 | SA 重启 |

**证据**: `interfaces/kits/c/native_avcodec_base.h` 错误码定义

---

**相关文档**: [对外 C-API](04_C_API.md) | [GN 构建系统](06_Build_System.md) | [安全风险评审](07_Security_Review.md)
