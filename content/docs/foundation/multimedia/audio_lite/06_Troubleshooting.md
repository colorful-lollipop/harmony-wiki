# 06_Troubleshooting - 问题定位

本文档记录 audio_lite 常见构建、运行和调试问题及其解决方案。

## 6.1 构建问题

### 6.1.1 GN 构建失败

**问题现象**：

```
$ hb build audio_lite
ERROR at //foundation/multimedia/audio_lite/frameworks/BUILD.gn:20:5: Unable to load.
Could not find file: //build/lite/config/component/lite_component.gni
```

**原因分析**：

- GN 路径配置错误
- 构建环境未正确初始化

**解决方案**：

```bash
# 1. 检查 OpenHarmony 开发环境
source ~/.bashrc  # 或正确的环境配置文件

# 2. 验证 hb 工具
hb --version

# 3. 重新设置开发板
hb set

# 4. 清理并重新构建
hb build audio_lite --clean
```

---

### 6.1.2 头文件找不到

**问题现象**：

```
fatal error: 'audio_capturer_server.h' file not found
```

**原因分析**：

- `include_dirs` 配置不完整
- 路径引用错误

**解决方案**：

检查 `services/BUILD.gn` 中的 `include_dirs`：

```gn
include_dirs = [
    "//foundation/multimedia/audio_lite/services/server/include",  # 确保此行存在
    "//foundation/multimedia/audio_lite/interfaces/kits",
    # ... 其他路径
]
```

---

### 6.1.3 链接错误（找不到符号）

**问题现象**：

```
libaudio_capturer_lite.so: undefined reference to 'AudioCapturerServer::GetInstance()'
```

**原因分析**：

- 依赖的静态库未正确链接
- `deps` 配置遗漏

**解决方案**：

检查 `frameworks/BUILD.gn` 的 `deps`：

```gn
deps = [
    "//foundation/multimedia/audio_lite/services:audio_capturer_impl",  # 确保存在
    "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
]
```

---

### 6.1.4 编译开关相关错误

**问题现象**：

```
error: 'enable_media_passthrough_mode' is not defined
```

**原因分析**：

- `media_utils_lite/config.gni` 未正确引用
- 编译开关定义缺失

**解决方案**：

在 `frameworks/BUILD.gn` 顶部添加：

```gn
import("//foundation/multimedia/media_utils_lite/config.gni")
```

---

## 6.2 运行问题

### 6.2.1 构造函数抛出异常

**问题现象**：

```cpp
AudioCapturer capturer;
// 程序崩溃或异常
```

**原因分析**：

- IPC 连接失败
- Surface 创建失败
- Samgr 服务未就绪

**证据**：`frameworks/binder/audio_capturer_client.cpp:138-155`

```cpp
AudioCapturer::AudioCapturerClient::AudioCapturerClient()
{
    IUnknown *iUnknown = SAMGR_GetInstance()->GetDefaultFeatureApi(AUDIO_CAPTURER_SERVICE_NAME);
    if (iUnknown == nullptr) {
        MEDIA_ERR_LOG("iUnknown is nullptr");
        throw runtime_error("Ipc proxy GetDefaultFeatureApi failed.");
    }
    // ...
}
```

**解决方案**：

```cpp
try {
    AudioCapturer capturer;
    bool result = capturer.Start();
    if (!result) {
        // 处理启动失败
        MEDIA_ERR_LOG("Failed to start capturer");
    }
} catch (const std::exception& e) {
    MEDIA_ERR_LOG("AudioCapturer exception: %{public}s", e.what());
    // 异常处理
}
```

**调试方法**：

```bash
# 检查 Samgr 服务是否运行
hilog | grep -E "AudioCapServer|SAMGR"

# 查看服务注册日志
hilog | grep "AudioCapturerServiceReg"
```

---

### 6.2.2 Read() 返回错误码

**问题现象**：

```cpp
int32_t size = capturer.Read(buffer, sizeof(buffer), true);
if (size < 0) {
    // 错误处理
}
```

**返回值排查**：

| 返回值 | 定义位置 | 说明 |
|--------|----------|------|
| `ERR_INVALID_READ` | `audio_capturer_client.h` | 参数无效 |
| `ERR_ILLEGAL_STATE` | `media_errors.h` | 状态错误 |
| `ERR_SOURCE_NOT_SET` | `media_errors.h` | 音频源异常 |

**调试步骤**：

```cpp
int32_t AudioCapturer::Read(uint8_t *buffer, size_t userSize, bool isBlockingRead)
{
    if (buffer == nullptr || !userSize) {
        MEDIA_ERR_LOG("Invalid parameters: buffer=%p, userSize=%zu", buffer, userSize);
        return ERR_INVALID_READ;
    }
    // ...
}
```

---

### 6.2.3 状态机异常

**问题现象**：

```
// 调用 Start() 失败
capturer.Start();  // 返回 false
```

**状态检查**：

```cpp
State state = capturer.GetStatus();
switch (state) {
    case INITIALIZED:
        MEDIA_INFO_LOG("Not prepared");
        break;
    case PREPARED:
        MEDIA_INFO_LOG("Ready to start");
        break;
    case RECORDING:
        MEDIA_INFO_LOG("Already recording");
        break;
    case STOPPED:
        MEDIA_INFO_LOG("Stopped");
        break;
    case RELEASED:
        MEDIA_INFO_LOG("Already released");
        break;
}
```

**常见状态问题**：

| 当前状态 | 期望操作 | 错误原因 |
|----------|----------|----------|
| INITIALIZED | Read() | 未调用 SetCapturerInfo() |
| PREPARED | Read() | 未调用 Start() |
| RECORDING | SetCapturerInfo() | 录制中不能修改参数 |
| STOPPED | Read() | 需重新 Start() |
| RELEASED | 任何操作 | 需重新创建对象 |

---

### 6.2.4 Surface 相关问题

**问题现象**：

```
Surface::CreateSurface() failed
```

**原因分析**：

- 图形子系统未初始化
- 内存不足
- Surface 数量达到上限

**调试方法**：

```cpp
int32_t AudioCapturer::AudioCapturerClient::InitSurface()
{
    Surface *surface = Surface::CreateSurface();
    if (surface == nullptr) {
        MEDIA_ERR_LOG("CreateSurface failed");
        // 错误处理
        return -1;
    }
    
    surface->RegisterConsumerListener(*this);
    surface_.reset(surface);
    
    // 调试：检查 Surface 配置
    MEDIA_DEBUG_LOG("Surface created: width=%{public}d, height=%{public}d, queueSize=%{public}d",
                    surface->GetWidth(), surface->GetHeight(), surface->GetQueueSize());
    return 0;
}
```

---

## 6.3 调试方法

### 6.3.1 日志查看

```bash
# 查看 audio_lite 相关日志
hilog | grep -E "AudioCapturer|AUDIO|Media"

# 实时过滤关键日志
hilog | grep -E "ERROR|ERR|Failed"

# 保存日志到文件
hilog > audio_capturer.log
```

### 6.3.2 关键日志点

| 位置 | 日志级别 | 说明 |
|------|----------|------|
| `audio_capturer_samgr.cpp:106` | INFO | 服务注册成功 |
| `audio_capturer_client.cpp:171` | INFO | 客户端创建成功 |
| `audio_capturer_client.cpp:153` | ERROR | 连接服务端失败 |
| `audio_capturer_server.cpp:40` | ERROR | 连接数已满 |
| `audio_capturer_server.cpp:99` | ERROR | PID 验证失败 |

### 6.3.3 GDB 调试

```bash
# 启动调试
gdb ./your_app

# 设置断点
(gdb) break AudioCapturer::Read
(gdb) break AudioCapturerServer::Dispatch

# 运行程序
(gdb) run

# 查看调用栈
(gdb) backtrace

# 查看变量
(gdb) print buffer
(gdb) print userSize
```

### 6.3.4 内存调试

```bash
# 使用 AddressSanitizer 编译
gn gen out/asan --args="use_asan=true"
hb build audio_lite

# 检测内存泄漏
hilog | grep -E "LEAK|ASAN"
```

---

## 6.4 常见问题 FAQ

### Q1: 如何切换通信模式？

**答**：修改 `//foundation/multimedia/media_utils_lite/config.gni`：

```gn
# Passthrough 模式（直接调用）
enable_media_passthrough_mode = true

# Binder IPC 模式（跨进程）
enable_media_passthrough_mode = false
```

### Q2: 单客户端限制如何解除？

**答**：修改 `services/server/src/audio_capturer_server.cpp:40-47`：

```cpp
// 原代码（单客户端）
if (clientPid_ != -1) {
    MEDIA_WARN_LOG("Only support one client");
    WriteInt32(reply, MEDIA_IPC_FAILED);
    return;
}

// 修改为（多客户端，需自行管理 clientPid_ 映射）
std::map<pid_t, AudioCapturerImpl*> clientMap;
if (clientMap.size() >= MAX_CLIENTS) {
    MEDIA_WARN_LOG("Too many clients");
    WriteInt32(reply, MEDIA_IPC_FAILED);
    return;
}
```

### Q3: 如何添加新的 AudioSource 类型？

**答**：

1. 在 `services/impl/audio_source/include/audio_source.h` 中添加枚举：

```cpp
enum AudioSourceType : int32_t {
    AUDIO_MIC = 0,
    AUDIO_FILE = 1,  // 新增
    // ...
};
```

2. 在 `services/impl/audio_source/audio_source.cpp` 中实现：

```cpp
AudioSource::AudioSource(AudioSourceType type)
{
    switch (type) {
        case AUDIO_MIC:
            adapter_ = CreateMicAdapter();
            break;
        case AUDIO_FILE:
            adapter_ = CreateFileAdapter();
            break;
    }
}
```

### Q4: 如何配置音频参数？

**答**：

```cpp
AudioCapturerInfo info;
info.inputSource = AUDIO_MIC;
info.sampleRate = 48000;        // 采样率
info.channelCount = 2;          // 立体声
info.bitWidth = BIT_WIDTH_16;   // 16 位
info.audioFormat = AUDIO_AAC;   // AAC 编码

int32_t ret = capturer.SetCapturerInfo(info);
if (ret != SUCCESS) {
    MEDIA_ERR_LOG("SetCapturerInfo failed: %{public}d", ret);
}
```

### Q5: 如何获取时间戳？

**答**：

```cpp
Timestamp ts;
Timestamp::Timebase base = Timestamp::Timebase::MONOTONIC;

bool ok = capturer.GetAudioTime(ts, base);
if (ok) {
    MEDIA_INFO_LOG("Frame position: %{public}u", ts.framePosition);
    MEDIA_INFO_LOG("Timestamp: %{public}ld.%09ld", 
                   ts.time.tv_sec, ts.time.tv_nsec);
}
```

---

## 6.5 调试命令速查

```bash
# 构建并安装
hb build audio_lite
hdc install

# 查看系统服务
hidumper -s 3301  # Audio 服务

# 查看进程
ps -T | grep audio

# 查看线程
cat /proc/<pid>/task/*/stat

# 性能分析
simpleperf record -p <pid> --app=<package>
simpleperf report
```

---

**上一章**：[05_Security_Review](05_Security_Review.md) | **返回**：[SUMMARY](SUMMARY.md)
