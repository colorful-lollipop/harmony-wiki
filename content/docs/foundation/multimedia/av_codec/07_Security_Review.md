# 07_安全风险评审

## 威胁模型

### 信任边界

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         不可信区域 (Untrusted)                          │
│  - 应用层 (第三方应用)                                                   │
│  - 网络输入 (远程媒体资源)                                               │
│  - 文件系统输入 (本地媒体文件)                                           │
├─────────────────────────────────────────────────────────────────────────┤
│                          信任边界 (Trust Boundary)                      │
└─────────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          可信区域 (Trusted)                             │
│  - av_codec 服务进程 (SA 3011)                                         │
│  - 框架层 (frameworks/native/capi/)                                    │
│  - 引擎层 (services/engine/*/)                                         │
│  - 硬件编解码器 (HDI)                                                  │
└─────────────────────────────────────────────────────────────────────────┘
```

### 数据流

```
不可信输入                              可信处理
     │                                      │
     ▼                                      ▼
┌──────────┐    IPC 调用    ┌──────────────────────┐
│  应用进程  │ ─────────────▶│   av_codec_service   │
│  (C-API)  │               │   (SA 3011)          │
└──────────┘               └──────────────────────┘
     │                              │
     │                              ▼
     │                       ┌──────────────┐
     │                       │  解码/编码    │
     │                       │  引擎处理     │
     │                       └──────────────┘
     │                              │
     ▼                              ▼
┌──────────┐                 ┌──────────────┐
│ 渲染/存储 │                 │  硬件加速    │
└──────────┘                 │  (HDI)      │
                              └──────────────┘
```

## 攻击面清单

| 攻击面 | 类型 | 说明 |
|--------|------|------|
| **N-API/C-API** | 跨进程调用 | 应用通过 IPC 调用服务，参数需校验 |
| **文件输入** | 解封装 | 解析媒体文件格式，可能存在解析漏洞 |
| **网络输入** | HTTP Source | 从网络加载媒体资源 |
| **Surface** | 图形 | 视频编解码器的输入/输出缓冲区 |
| **IPC** | SA 通信 | 30+ IPC 方法调用 |
| **DRM** | 解密 | 受保护内容的解密处理 |

## 可被利用点分析

### 1. 解封装器整数溢出

**风险等级**: 高

**证据**: `services/media_engine/modules/demuxer/media_demuxer.cpp` (4,878 行)

**问题描述**: 解封装器在解析媒体文件头时，对输入的尺寸参数缺乏边界检查，可能导致整数溢出。

**触发路径**:
```
恶意构造的 MP4/TS 文件
  │
  └──▶ OH_AVDemuxer_CreateWithSource()
       │
       └──▶ MediaDemuxer::ParseHeader()
            │
            └──▶ 整数溢出导致缓冲区分配过小
                 │
                 └──▶ 堆溢出 / 拒绝服务
```

**影响**:
- 拒绝服务 (DoS)
- 潜在的代码执行

**修复建议**:
```cpp
// 在 media_demuxer.cpp 中添加
if (headerSize > MAX_HEADER_SIZE || headerSize == 0) {
    return AV_ERR_DATA_INVALID;
}
size_t bufferSize = SafeMul(headerSize, 2);  // 使用安全乘法
```

---

### 2. 视频解码器 Surface 指针校验缺失

**风险等级**: 高

**证据**: `native_avcodec_videodecoder.h:147`
```cpp
OH_AVErrCode OH_VideoDecoder_SetSurface(OH_AVCodec *codec, OHNativeWindow *window);
```

**问题描述**: `SetSurface()` 接受的 `OHNativeWindow*` 指针未校验有效性。

**触发路径**:
```
恶意应用传递伪造的 OHNativeWindow*
  │
  └──▶ OH_VideoDecoder_SetSurface(codec, maliciousWindow)
       │
       └──▶ IPC 传递到服务侧
            │
            └──▶ 解引用野指针导致崩溃或代码执行
```

**影响**:
- 进程崩溃
- 潜在的提权

**修复建议**:
```cpp
OH_AVErrCode OH_VideoDecoder_SetSurface(OH_AVCodec *codec, OHNativeWindow *window) {
    if (codec == nullptr || window == nullptr) {
        return AV_ERR_INVALID_VAL;
    }
    // IPC 调用前验证远程对象有效性
    return IPC_SetSurface(codec->GetRemoteObject(), window);
}
```

---

### 3. 内存缓冲区长度校验缺失

**风险等级**: 中

**证据**: `native_avdemuxer.h:182`
```cpp
OH_AVErrCode OH_AVDemuxer_ReadSampleBuffer(OH_AVDemuxer *demuxer,
    uint32_t trackId, OH_AVBuffer *buffer);
```

**问题描述**: `buffer` 参数的长度字段未被充分校验。

**触发路径**:
```
应用创建小尺寸 buffer
  │
  └──▶ OH_AVDemuxer_ReadSampleBuffer() 提交大数据
       │
       └──▶ 数据截断或越界写入
```

**影响**:
- 数据损坏
- 信息泄露

**修复建议**:
```cpp
if (buffer->GetSize() < expectedSampleSize) {
    return AV_ERR_DATA_INVALID;
}
```

---

### 4. 路径遍历漏洞

**风险等级**: 中

**证据**: `native_avmuxer.h:58`
```cpp
OH_AVMuxer *OH_AVMuxer_Create(int32_t fd);
```

**问题描述**: 文件描述符 `fd` 可能指向任意文件位置。

**触发路径**:
```
应用打开恶意路径的文件
  │
  └──▶ OH_AVMuxer_Create(fd)
       │
       └──▶ 写入输出文件
            │
            └──▶ 覆盖系统文件或敏感数据
```

**影响**:
- 任意文件写入
- 权限提升

**修复建议**:
```cpp
// 使用 SELinux 或权限检查
if (!CheckFilePermission(fd, O_WRONLY)) {
    return AV_ERR_OPERATE_NOT_PERMIT;
}
```

---

### 5. IPC 参数校验不完整

**风险等级**: 中

**证据**: `services/services/sa_avcodec/ipc/av_codec_service_ipc_interface_code.h`

**问题描述**: 30+ IPC 方法中，部分方法对传入参数的范围检查不严格。

**触发路径**:
```
恶意应用发送超范围参数
  │
  └──▶ CodecServiceStub::OnRemoteRequest(code, data, reply, option)
       │
       └──▶ 未校验参数直接使用
            │
            └──▶ 数组越界 / 整数溢出
```

**影响**:
- 服务崩溃
- 潜在代码执行

**修复建议**:
```cpp
// 在 codec_service_stub.cpp 中添加
int32_t CodecServiceStub::Configure(MessageParcel &data) {
    int32_t codecId = data.ReadInt32();
    if (codecId < 0 || codecId >= MAX_CODEC_COUNT) {
        return AV_ERR_INVALID_VAL;
    }
    // ...
}
```

---

## 已有的安全机制

### 1. 编译时安全

| 机制 | 配置 | 证据 |
|------|------|------|
| 栈保护 | `-fstack-protector-all` | `interfaces/kits/c/BUILD.gn:58` |
| FORTIFY_SOURCE | `-D_FORTIFY_SOURCE=2` | `interfaces/kits/c/BUILD.gn:61` |
| CFI | `cfi = true` | `config.gni:65` |
| ASan/UBSan | `ubsan = true`, `boundary_sanitize = true` | `config.gni:64,67` |

### 2. 运行时安全

| 机制 | 用途 |
|------|------|
| IPC 权限校验 | `IPCSkeleton::GetCallingUid()` |
| SELinux | 进程隔离 |
| PAC/RET | 控制流保护 |

### 3. DRM 支持

| 机制 | 状态 |
|------|------|
| DRM 解密 | `av_codec_support_drm = false` (默认关闭) |

**证据**: `config.gni:28`
```gni
av_codec_support_drm = false
```

## 安全检查范围

### 已覆盖检查

- ✅ C-API 参数空值检查
- ✅ IPC 描述符校验
- ✅ 编译时 sanitizers
- ✅ 服务 SA 隔离

### 未完全覆盖检查

- ⚠️ 解封装器边界检查
- ⚠️ Surface 对象验证
- ⚠️ 文件路径验证

## 安全最佳实践建议

1. **所有外部输入必须校验**: 文件尺寸、MIME 类型、编码参数
2. **使用安全的字符串操作**: 避免 `strcpy`/`sprintf`，使用 `snprintf`/`strncpy`
3. **最小权限原则**: 只申请必要的系统能力
4. **依赖项安全**: 定期更新 FFmpeg 等第三方库

---

**相关文档**: [架构设计](03_Architecture.md) | [GN 构建系统](06_Build_System.md) | [故障排查指南](08_Troubleshooting.md)
