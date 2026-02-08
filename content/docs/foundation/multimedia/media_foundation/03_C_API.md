# C API 接口文档

## 概述

HiStreamer 提供两层 Native API：

| 层级 | 路径 | 说明 |
|------|------|------|
| **C API** | `interface/kits/c/` | 标准 C 接口，用于 NDK 开发 |
| **Inner API** | `interface/inner_api/` | HiStreamer 内部模块间调用 |

> **注意**: 本仓库**不包含 N-API**（Node.js Addon）代码。N-API 绑定位于独立的 `player_framework` 仓库。

## C API 清单

### native_averrors.h - 错误码

**头文件路径**: `interface/kits/c/native_averrors.h:38-225`

**库文件**: `libnative_media_core.so`

| 错误码 | 值 | 说明 | 适用场景 |
|--------|-----|------|----------|
| `AV_ERR_OK` | 0 | 操作成功 | 全部 |
| `AV_ERR_NO_MEMORY` | 1 | 内存不足 | 内存分配失败 |
| `AV_ERR_OPERATE_NOT_PERMIT` | 2 | 操作不允许 | 状态错误 |
| `AV_ERR_INVALID_VAL` | 3 | 无效参数 | 参数校验失败 |
| `AV_ERR_IO` | 4 | IO 错误 | 文件/网络 IO |
| `AV_ERR_TIMEOUT` | 5 | 超时 | 网络超时 |
| `AV_ERR_UNKNOWN` | 6 | 未知错误 | 内部错误 |
| `AV_ERR_SERVICE_DIED` | 7 | 服务死亡 | 媒体服务异常 |
| `AV_ERR_INVALID_STATE` | 8 | 状态不支持 | 状态机错误 |
| `AV_ERR_UNSUPPORT` | 9 | 不支持接口 | 功能不支持 |
| `AV_ERR_INPUT_DATA_ERROR` | 10 | 输入数据错误 | 解码数据错误 |
| `AV_ERR_UNSUPPORTED_FORMAT` | 11 | 不支持格式 | 格式不支持 |
| `AV_ERR_HARDWARE_FAILED` | 12 | 硬件失败 | 硬件错误 |
| `AV_ERR_DRM_BASE` | 200 | DRM 错误基值 | DRM 相关 |
| `AV_ERR_DRM_DECRYPT_FAILED` | 201 | DRM 解密失败 | DRM 解密 |
| `AV_ERR_VIDEO_BASE` | 300 | 视频错误基值 | 视频相关 |

### native_avmemory.h - 内存管理（已废弃）

**头文件路径**: `interface/kits/c/native_avmemory.h:38-116`

**库文件**: `libnative_media_core.so`

> **废弃警告**: 此 API 已在 API 11 废弃，请使用 `OH_AVBuffer_*` 系列 API。

| API | 签名 | 说明 |
|-----|------|------|
| Create | `OH_AVMemory *OH_AVMemory_Create(int32_t size)` | 创建内存实例 |
| GetAddr | `uint8_t *OH_AVMemory_GetAddr(OH_AVMemory *mem)` | 获取虚拟地址 |
| GetSize | `int32_t OH_AVMemory_GetSize(OH_AVMemory *mem)` | 获取大小 |
| Destroy | `OH_AVErrCode OH_AVMemory_Destroy(OH_AVMemory *mem)` | 销毁内存 |

### native_avbuffer.h - 缓冲区管理

**头文件路径**: `interface/kits/c/native_avbuffer.h:37-184`

**库文件**: `libnative_media_core.so`

| API | 签名 | 说明 |
|-----|------|------|
| Create | `OH_AVBuffer *OH_AVBuffer_Create(int32_t capacity)` | 创建缓冲区 |
| Destroy | `OH_AVErrCode OH_AVBuffer_Destroy(OH_AVBuffer *buffer)` | 销毁缓冲区 |
| GetBufferAttr | `OH_AVErrCode OH_AVBuffer_GetBufferAttr(OH_AVBuffer *buffer, OH_AVCodecBufferAttr *attr)` | 获取属性 |
| SetBufferAttr | `OH_AVErrCode OH_AVBuffer_SetBufferAttr(OH_AVBuffer *buffer, const OH_AVCodecBufferAttr *attr)` | 设置属性 |
| GetParameter | `OH_AVFormat *OH_AVBuffer_GetParameter(OH_AVBuffer *buffer)` | 获取参数 |
| SetParameter | `OH_AVErrCode OH_AVBuffer_SetParameter(OH_AVBuffer *buffer, const OH_AVFormat *format)` | 设置参数 |
| GetAddr | `uint8_t *OH_AVBuffer_GetAddr(OH_AVBuffer *buffer)` | 获取地址 |
| GetCapacity | `int32_t OH_AVBuffer_GetCapacity(OH_AVBuffer *buffer)` | 获取容量 |
| GetNativeBuffer | `OH_NativeBuffer *OH_AVBuffer_GetNativeBuffer(OH_AVBuffer *buffer)` | 获取原生 Buffer |

### native_avbuffer_info.h - 缓冲区属性

**头文件路径**: `interface/kits/c/native_avbuffer_info.h:37-94`

**库文件**: `libnative_media_core.so`

#### OH_AVCodecBufferFlags

| 标志 | 值 | 说明 |
|------|-----|------|
| `AVCODEC_BUFFER_FLAGS_NONE` | 0 | 无标志 |
| `AVCODEC_BUFFER_FLAGS_EOS` | 1<<0 | 帧结束 |
| `AVCODEC_BUFFER_FLAGS_SYNC_FRAME` | 1<<1 | 关键帧 |
| `AVCODEC_BUFFER_FLAGS_INCOMPLETE_FRAME` | 1<<2 | 不完整帧 |
| `AVCODEC_BUFFER_FLAGS_CODEC_DATA` | 1<<3 | 编解码数据 |
| `AVCODEC_BUFFER_FLAGS_DISCARD` | 1<<4 | 可丢弃帧 |
| `AVCODEC_BUFFER_FLAGS_DISPOSABLE` | 1<<5 | 非参考帧 |

#### OH_AVCodecBufferAttr

```c
typedef struct OH_AVCodecBufferAttr {
    int64_t pts;       // 时间戳 (微秒)
    int32_t size;      // 数据大小 (字节)
    int32_t offset;    // 起始偏移
    uint32_t flags;    // 标志组合
} OH_AVCodecBufferAttr;
```

### native_avformat.h - 格式参数

**头文件路径**: `interface/kits/c/native_avformat.h:38-476`

**库文件**: `libnative_media_core.so`

#### 创建/销毁

| API | 签名 | 说明 |
|-----|------|------|
| Create | `OH_AVFormat *OH_AVFormat_Create(void)` | 创建空 Format |
| CreateAudioFormat | `OH_AVFormat *OH_AVFormat_CreateAudioFormat(const char *mimeType, int32_t sampleRate, int32_t channelCount)` | 创建音频 Format |
| CreateVideoFormat | `OH_AVFormat *OH_AVFormat_CreateVideoFormat(const char *mimeType, int32_t width, int32_t height)` | 创建视频 Format |
| Destroy | `void OH_AVFormat_Destroy(OH_AVFormat *format)` | 销毁 Format |
| Copy | `bool OH_AVFormat_Copy(OH_AVFormat *to, const OH_AVFormat *from)` | 复制 Format |

#### Set 方法

| API | 签名 | 说明 |
|-----|------|------|
| SetIntValue | `bool OH_AVFormat_SetIntValue(OH_AVFormat *format, const char *key, int32_t value)` | 设置 Int |
| SetLongValue | `bool OH_AVFormat_SetLongValue(OH_AVFormat *format, const char *key, int64_t value)` | 设置 Long |
| SetFloatValue | `bool OH_AVFormat_SetFloatValue(OH_AVFormat *format, const char *key, float value)` | 设置 Float |
| SetDoubleValue | `bool OH_AVFormat_SetDoubleValue(OH_AVFormat *format, const char *key, double value)` | 设置 Double |
| SetStringValue | `bool OH_AVFormat_SetStringValue(OH_AVFormat *format, const char *key, const char *value)` | 设置 String |
| SetBuffer | `bool OH_AVFormat_SetBuffer(OH_AVFormat *format, const char *key, const uint8_t *addr, size_t size)` | 设置 Buffer |
| SetUintValue | `bool OH_AVFormat_SetUintValue(OH_AVFormat *format, const char *key, uint32_t value)` | 设置 Uint |
| SetIntBuffer | `bool OH_AVFormat_SetIntBuffer(OH_AVFormat *format, const char *key, const int32_t *addr, size_t size)` | 设置 Int 数组 |

#### Get 方法

| API | 签名 | 说明 |
|-----|------|------|
| GetIntValue | `bool OH_AVFormat_GetIntValue(OH_AVFormat *format, const char *key, int32_t *out)` | 获取 Int |
| GetLongValue | `bool OH_AVFormat_GetLongValue(OH_AVFormat *format, const char *key, int64_t *out)` | 获取 Long |
| GetFloatValue | `bool OH_AVFormat_GetFloatValue(OH_AVFormat *format, const char *key, float *out)` | 获取 Float |
| GetDoubleValue | `bool OH_AVFormat_GetDoubleValue(OH_AVFormat *format, const char *key, double *out)` | 获取 Double |
| GetStringValue | `bool OH_AVFormat_GetStringValue(OH_AVFormat *format, const char *key, const char **out)` | 获取 String |
| GetBuffer | `bool OH_AVFormat_GetBuffer(OH_AVFormat *format, const char *key, uint8_t **addr, size_t *size)` | 获取 Buffer |
| GetUintValue | `bool OH_AVFormat_GetUintValue(OH_AVFormat *format, const char *key, uint32_t *out)` | 获取 Uint |
| GetIntBuffer | `bool OH_AVFormat_GetIntBuffer(OH_AVFormat *format, const char *key, int32_t **addr, size_t *size)` | 获取 Int 数组 |

#### 辅助方法

| API | 签名 | 说明 |
|-----|------|------|
| DumpInfo | `const char *OH_AVFormat_DumpInfo(OH_AVFormat *format)` | 打印信息 |
| GetKeyCount | `uint32_t OH_AVFormat_GetKeyCount(OH_AVFormat *format)` | 获取键数量 |
| GetKey | `bool OH_AVFormat_GetKey(OH_AVFormat *format, uint32_t index, const char **key)` | 获取键名 |

### native_audio_channel_layout.h - 音频通道布局

**头文件路径**: `interface/kits/c/native_audio_channel_layout.h`

**库文件**: `libnative_media_core.so`

## 错误码与返回值

### 通用返回值规则

| 返回类型 | 成功值 | 失败处理 |
|----------|--------|----------|
| `OH_AVErrCode` | `AV_ERR_OK` | 检查错误码 |
| `OH_AVMemory*` | 非 NULL | 释放资源 |
| `OH_AVBuffer*` | 非 NULL | 释放资源 |
| `OH_AVFormat*` | 非 NULL | 销毁 Format |
| `bool` | `true` | 检查返回值 |

### 常见错误处理

```c
// 错误码检查
OH_AVBuffer *buffer = OH_AVBuffer_Create(capacity);
if (buffer == NULL) {
    // 内存不足或参数错误
}

// 获取参数时的错误
bool ret = OH_AVFormat_GetIntValue(format, KEY_BIT_RATE, &value);
if (!ret) {
    // 参数不存在或类型不匹配
}
```

## 参数校验规则

### 通用校验

| 参数 | 校验规则 | 错误码 |
|------|----------|--------|
| NULL 指针 | 必须检查 | `AV_ERR_INVALID_VAL` |
| size <= 0 | 范围校验 | `AV_ERR_INVALID_VAL` |
| 内存不足 | 分配失败 | `AV_ERR_NO_MEMORY` |
| 系统错误 | 内部错误 | `AV_ERR_UNKNOWN` |

### Format Key 校验

| 键类型 | 校验规则 | 错误码 |
|--------|----------|--------|
| key = NULL | 必填检查 | `AV_ERR_INVALID_VAL` |
| 类型不匹配 | 类型校验 | `AV_ERR_INVALID_VAL` |
| 键不存在 | 查找失败 | `false` 返回 |

## 使用示例

### 创建音频 Format

```c
OH_AVFormat *audioFmt = OH_AVFormat_CreateAudioFormat(
    OH_MIME_AUDIO_AAC,  // MIME 类型
    44100,              // 采样率
    2                   // 通道数
);

// 设置参数
OH_AVFormat_SetIntValue(audioFmt, OH_MD_KEY_BIT_RATE, 128000);
OH_AVFormat_SetIntValue(audioFmt, OH_MD_KEY_AAC_PROFILE, 2);

// 使用完成后销毁
OH_AVFormat_Destroy(audioFmt);
```

### 创建 Buffer

```c
OH_AVBuffer *buffer = OH_AVBuffer_Create(1024);
if (buffer != NULL) {
    uint8_t *addr = OH_AVBuffer_GetAddr(buffer);
    int32_t capacity = OH_AVBuffer_GetCapacity(buffer);
    // 使用 buffer...
    OH_AVBuffer_Destroy(buffer);
}
```

## 相关文件

| 文件 | 路径 | 说明 |
|------|------|------|
| native_averrors.h | `interface/kits/c/` | 错误码定义 |
| native_avmemory.h | `interface/kits/c/` | 内存 API（废弃） |
| native_avbuffer.h | `interface/kits/c/` | 缓冲区 API |
| native_avformat.h | `interface/kits/c/` | 格式参数 API |
| native_avbuffer_info.h | `interface/kits/c/` | 缓冲区属性 |
| native_media_core.so | `out/` | 库文件 |
