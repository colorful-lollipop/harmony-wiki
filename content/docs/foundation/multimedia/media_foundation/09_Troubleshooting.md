# 常见问题与定位

## 构建问题

### 问题 1：Feature Flag 配置错误

**现象**:
```
error: undefined variable 'media_foundation_enable_plugin_ffmpeg_adapter'
```

**原因**: Feature Flag 拼写错误或未在 config.gni 中定义。

**解决方案**:
```bash
# 1. 检查 config.gni 中的定义
grep "media_foundation_enable_plugin" config.gni

# 2. 确认系统类型
# Standard: media_foundation_enable_plugin_ffmpeg_adapter = true
# Small/Mini: 默认为 false
```

### 问题 2：依赖缺失

**现象**:
```
ninja: error: dependency '//foundation/multimedia/xxx' not found
```

**原因**: 依赖的子系统未编译。

**解决方案**:
```bash
# 1. 先编译依赖子系统
./build.sh --product-name <product> --build-target <dependency>

# 2. 或者在 args.gn 中添加依赖
# build/lite/product/<product>.json
```

### 问题 3：循环依赖

**现象**:
```
error: dependency cycle detected
```

**原因**: 模块间存在循环引用。

**解决方案**:
```bash
# 1. 检查 BUILD.gn 中的 deps
# 2. 重构代码，使用接口解耦
# 3. 合并相关模块
```

## 运行时问题

### 问题 1：插件加载失败

**现象**:
```
E/HiStreamer: [plugin_loader] dlopen failed: library "libFFmpegDemuxer.so" not found
```

**原因**: 插件库未安装或路径错误。

**解决方案**:
```bash
# 1. 检查插件库是否存在
ls -la system/lib/media/histreamer_plugins/

# 2. 确认 feature flag 已启用
# build/ohos/<product>/config.json
"media_foundation_enable_plugin_ffmpeg_adapter": true

# 3. 检查插件依赖
ldd system/lib/media/histreamer_plugins/libFFmpegDemuxer.so
```

### 问题 2：Pipeline Link 失败

**现象**:
```
E/PipelineCore: LinkFilters failed: mismatched capabilities
```

**原因**: 相邻 Filter 的能力不匹配。

**解决方案**:
```cpp
// 1. 检查 Filter 能力配置
CapabilityBuilder builder;
builder.SetMime(OH_MIME_AUDIO_AAC)
    .SetAudioSampleRate(44100);

// 2. 确认 Demuxer 输出与 Decoder 输入匹配
```

### 问题 3：内存不足

**现象**:
```
E/HiStreamer: CreateBuffer failed: no memory
```

**原因**: Buffer 分配失败。

**解决方案**:
```cpp
// 1. 检查 capacity 是否合理
int32_t capacity = 1024 * 1024;  // 1MB

// 2. 释放不需要的 Buffer
OH_AVBuffer_Destroy(buffer);
```

## 调试方法

### 日志查看

```bash
# 过滤 HiStreamer 日志
hilog | grep -i histreamer

# 查看详细日志
hilog | grep -E "HiStreamer|Pipeline|Filter"
```

### 调试宏

```cpp
// 启用调试日志
#define MEDIA_DEBUG 1

// 在代码中添加日志
MEDIA_LOGD("Buffer created: %{public}d", bufferId);
```

### 核心 Dump

```bash
# 触发 core dump
gdb ./media_app
(gdb) bt  # backtrace
```

## 性能问题

### 问题 1：Buffer 频繁分配

**现象**: 内存占用持续增长。

**解决方案**:
```cpp
// 1. 复用 Buffer
bufferPool = AVBufferPool::Create(10, 1024);
buffer = bufferPool->AllocateBuffer();

// 2. 限制最大 Buffer 数量
const int32_t MAX_BUFFER_COUNT = 16;
```

### 问题 2：Pipeline 卡顿

**现象**: 播放卡顿或音视频不同步。

**解决方案**:
```cpp
// 1. 检查 Pipeline 配置
pipeline->SetLatencyMode(LowLatencyMode);

// 2. 调整 Buffer 队列深度
demuxerFilter->SetMaxBufferCount(8);
```

## 常见错误码

| 错误码 | 含义 | 处理建议 |
|--------|------|----------|
| `AV_ERR_NO_MEMORY` | 内存不足 | 减少 Buffer 数量 |
| `AV_ERR_INVALID_VAL` | 参数错误 | 检查 API 参数 |
| `AV_ERR_IO` | IO 错误 | 检查文件路径 |
| `AV_ERR_SERVICE_DIED` | 服务死亡 | 重启媒体服务 |
| `AV_ERR_TIMEOUT` | 超时 | 增加超时时间 |
| `AV_ERR_UNSUPPORT` | 不支持 | 检查格式是否支持 |

## 故障排查流程

```
┌─────────────────────────────────────────────────────────────┐
│                    故障发生                                  │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│ Step 1: 查看日志                                            │
│ - hilog | grep -i "HiStreamer|ERROR"                       │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│ Step 2: 分类判断                                            │
│ - 构建问题? → 检查 BUILD.gn 和 feature flags                │
│ - 运行时问题? → 检查插件、Pipeline、Buffer                    │
│ - 性能问题? → 检查内存、CPU、延迟                            │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│ Step 3: 详细日志                                            │
│ - 启用 MEDIA_DEBUG                                          │
│ - 查看 Filter 状态机日志                                    │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│ Step 4: 核心分析                                            │
│ - gdb 调试                                                  │
│ - core dump 分析                                            │
└─────────────────────────────────────────────────────────────┘
```

## 性能指标

### 关键指标

| 指标 | 期望值 | 告警阈值 |
|------|--------|----------|
| Buffer 创建时间 | < 1ms | > 10ms |
| Pipeline 启动时间 | < 100ms | > 500ms |
| 内存占用 | < 50MB | > 100MB |
| 端到端延迟 | < 100ms | > 300ms |

### 监控方法

```cpp
// 开启性能监控
auto timer = std::make_shared<ScopedTimer>("PipelineStart");
pipeline->Start();
auto elapsed = timer->GetElapsed();

// 上报监控
MediaMonitor::ReportPerformance("pipeline_start", elapsed);
```
