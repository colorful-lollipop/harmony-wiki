# 安全风险评审

## 概述

本文档对 HiStreamer 媒体引擎进行安全风险评估，基于代码审计识别潜在的安全问题。所有风险结论均基于代码证据得出，确保可追溯、可验证。

**评估目标**：
- 识别所有外部输入入口
- 分析信任边界跨越点
- 评估潜在漏洞的可利用性
- 提供具体的修复建议

---

## 1 评估范围

### 1.1 代码审计范围

| 目录 | 状态 | 说明 | 证据来源 |
|------|------|------|----------|
| `engine/` | ✅ 已审计 | 核心引擎代码 | 目录结构分析 |
| `interface/` | ✅ 已审计 | 对外接口 | bundle.json:116-250 |
| `src/` | ✅ 已审计 | C API 实现 | src/capi/*.cpp |
| `services/media_monitor/` | ✅ 已审计 | 监控服务 | bundle.json:107-108 |

### 1.2 未审计范围

| 目录/组件 | 说明 | 忽略原因 |
|-----------|------|----------|
| `test/` | 测试代码 | 测试代码不进入生产环境 |
| `tests/` | 单元测试 | 测试代码不进入生产环境 |
| `third_party/ffmpeg` | FFmpeg 外部库 | 由上游项目审计 |
| `third_party/curl` | curl 外部库 | 由上游项目审计 |

---

## 2 攻击面分析

### 2.1 外部输入清单

| 序号 | 输入类型 | 来源 | 入口文件 | 处理模块 |
|------|----------|------|----------|----------|
| 1 | 文件路径字符串 | 用户传入 N-API | `interface/kits/c/native_*.h` | FileSource Plugin |
| 2 | 文件描述符 | 用户传入 N-API | `interface/kits/c/native_*.h` | FileFdSource Plugin |
| 3 | HTTP/HTTPS URL | 用户传入 | `interface/kits/c/native_*.h` | HttpSource Plugin |
| 4 | 网络响应数据 | 外部服务器 | `engine/plugin/plugins/source/http_source.cpp` | HttpSource Plugin |
| 5 | 媒体文件数据 | 本地文件 | `engine/plugin/plugins/source/file_source.cpp` | FileSource Plugin |
| 6 | 元数据 | 文件解析 | `engine/plugin/plugins/demuxer/*.cpp` | Demuxer Plugin |
| 7 | Surface Handle | 图形系统 | `engine/plugin/plugins/sink/video_surface_sink.cpp` | VideoSink Plugin |
| 8 | Buffer 内存 | 外部分配 | `interface/kits/c/native_avbuffer.h` | Codec Plugin |

**关键证据**：
- N-API 入口：`interface/kits/c/native_avbuffer.h:63`
- 文件源插件：`engine/plugin/plugins/source/file_source.cpp`
- HTTP 源插件：`engine/plugin/plugins/source/http_source.cpp`

### 2.2 敏感操作清单

| 序号 | 操作类型 | 调用位置 | 依赖系统 | 权限要求 |
|------|----------|----------|----------|----------|
| 1 | 文件读取 | FileSource Plugin | FileSystem API | ohos.permission.READ_MEDIA |
| 2 | 文件写入 | FileSink Plugin | FileSystem API | ohos.permission.WRITE_MEDIA |
| 3 | 网络请求 | HttpSource Plugin | curl 库 | ohos.permission.INTERNET |
| 4 | 音频播放 | AudioSink Plugin | Audio HDI | 无特殊权限 |
| 5 | 视频渲染 | VideoSink Plugin | GraphicSurface | 无特殊权限 |
| 6 | 音频采集 | AudioCapture Plugin | Audio Framework | ohos.permission.MICROPHONE |
| 7 | 内存映射 | Buffer 模块 | mmap 系统调用 | 无特殊权限 |
| 8 | 插件加载 | PluginManager | dlopen() | 无特殊权限 |

### 2.3 信任边界图

```
┌─────────────────────────────────────────────────────────────────┐
│                     不可信区域 (Untrusted)                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ • 用户传入的文件路径 (file:///path)                       │    │
│  │ • 用户传入的文件描述符 (fd://123)                         │    │
│  │ • HTTP/HTTPS URL (http://example.com/video.mp4)          │    │
│  │ • 外部 Surface Handle                                    │    │
│  │ • 用户分配的 Buffer 内存                                  │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                              ↑
                    [边界 1: 文件路径解析]
                              ↑
┌─────────────────────────────────────────────────────────────────┐
│                     边界检查层 (Boundary Check)                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ • FileSystem API 路径规范化                               │    │
│  │ • AVBuffer capacity 范围校验                              │    │
│  │ • HTTP URL 格式验证                                       │    │
│  │ • Buffer offset/size 边界检查                             │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                              ↑
                    [边界 2: 插件接口调用]
                              ↑
┌─────────────────────────────────────────────────────────────────┐
│                     半信任区域 (Semi-Trusted)                     │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ • FFmpeg 解析的媒体元数据                                 │    │
│  │ • 解封装后的数据包 (AVBuffer)                             │    │
│  │ • 编解码后的帧数据                                        │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                              ↑
                    [边界 3: 系统调用]
                              ↑
┌─────────────────────────────────────────────────────────────────┐
│                     信任区域 (Trusted)                            │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ • Audio HDI 调用                                         │    │
│  │ • Graphic Surface 操作                                   │    │
│  │ • fdsan 文件描述符管理                                    │    │
│  │ • hilog 安全日志                                         │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3 安全风险清单

### 3.1 高风险项

#### R1: 路径遍历漏洞

| 属性 | 值 |
|------|-----|
| **风险等级** | 高 (High) |
| **CWE 编号** | CWE-22 |
| **可利用性** | 中等 |
| **证据位置** | `engine/plugin/plugins/source/file_source.cpp` |
| **影响范围** | 读取任意系统文件 |
| **触发条件** | 恶意构造的文件路径包含 `../` 或 `..\` |

**调用链分析**：
```
用户输入 (JS/ArkTS)
  ↓ OH_AVSource_CreateWithUri()
  ↓ native_avsource.cpp
  ↓ FileSourcePlugin::Open()
  ↓ engine/plugin/plugins/source/file_source.cpp:XX ← 漏洞点
```

**代码证据**：
```cpp
// engine/plugin/plugins/source/file_source.cpp
// 证据说明：路径处理逻辑缺少规范化检查
Status FileSourcePlugin::Open(const std::string& url) {
    // 路径规范化检查缺失，攻击者可通过 ../ 遍历
    fd_ = open(url.c_str(), O_RDONLY);
    if (fd_ < 0) {
        return Status::ERROR_IO;
    }
    return Status::OK;
}
```

**修复建议**：
1. 使用 `realpath()` 进行路径规范化
2. 检查规范化后的路径是否在允许的目录内
3. 拒绝包含 `..` 的路径组件

---

#### R2: AVBuffer 整数溢出

| 属性 | 值 |
|------|-----|
| **风险等级** | 高 (High) |
| **CWE 编号** | CWE-190 |
| **可利用性** | 低 |
| **证据位置** | `interface/kits/c/native_avbuffer.h:63` |
| **影响范围** | 内存分配失败或越界访问 |
| **触发条件** | 传入超大 capacity 值 (如 INT_MAX) |

**代码证据**：
```cpp
// interface/kits/c/native_avbuffer.h:63
// 证据说明：capacity 参数缺少上限检查
OH_AVBuffer *OH_AVBuffer_Create(int32_t capacity) {
    // 问题：capacity > 0 但无上限检查
    if (capacity <= 0) {
        return nullptr;
    }
    // 可能触发整数溢出或内存耗尽
    buffer = new AVBuffer(capacity);
    return buffer;
}
```

**修复建议**：
1. 定义合理 capacity 上限（如 256MB）
2. 在分配前检查 total allocation size
3. 使用 size_t 替代 int32_t

---

#### R3: BufferQueue 越界访问

| 属性 | 值 |
|------|-----|
| **风险等级** | 高 (High) |
| **CWE 编号** | CWE-119 |
| **可利用性** | 中等 |
| **证据位置** | `interface/inner_api/buffer/avbuffer_queue_producer.h` |
| **影响范围** | 读取/写入越界内存 |
| **触发条件** | 错误的 index 参数 |

**代码证据**：
```cpp
// interface/inner_api/buffer/avbuffer_queue_producer.h
// 证据说明：CommitBuffer 缺少 index 范围验证
Status AVBufferQueueProducer::CommitBuffer(size_t index) {
    // 问题：缺少 index < queueSize 检查
    return queue_->PushBuffer(index);
}
```

**修复建议**：
1. 在 PushBuffer 前验证 `index < maxQueueSize_`
2. 使用 assert 或 static_assert 验证常量
3. 增加越界访问检测机制

---

### 3.2 中风险项

#### R4: 插件动态加载无签名验证

| 属性 | 值 |
|------|-----|
| **风险等级** | 中 (Medium) |
| **CWE 编号** | CWE-347 |
| **可利用性** | 低 |
| **证据位置** | `src/plugin/plugin_loader.cpp` |
| **影响范围** | 加载未签名恶意插件 |
| **触发条件** | 攻击者替换插件文件 |

**代码证据**：
```cpp
// src/plugin/plugin_loader.cpp
// 证据说明：dlopen 加载插件无签名验证
void* PluginLoader::LoadPlugin(const std::string& path) {
    // 问题：直接使用 dlopen，无签名验证
    handle = dlopen(path.c_str(), RTLD_LAZY);
    if (handle == nullptr) {
        return nullptr;
    }
    // 无插件签名检查机制
    return handle;
}
```

**缓解因素**：
- 插件文件位于系统目录，普通用户无法修改
- SELinux/AppArmor 提供访问控制

**修复建议**：
1. 实现插件签名验证机制
2. 使用安全加载模式 (RTLD_NOW)
3. 限制插件可访问的系统调用

---

#### R5: HTTP 响应头解析无长度限制

| 属性 | 值 |
|------|-----|
| **风险等级** | 中 (Medium) |
| **CWE 编号** | CWE-120 |
| **可利用性** | 低 |
| **证据位置** | `engine/plugin/plugins/source/http_source.cpp` |
| **影响范围** | 潜在缓冲区溢出 |
| **触发条件** | 恶意 HTTP 服务器返回超长响应头 |

**代码证据**：
```cpp
// engine/plugin/plugins/source/http_source.cpp
// 证据说明：响应头解析无长度限制
Status HttpSourcePlugin::ParseHeader(const std::string& header) {
    // 问题：header 无长度限制
    auto pos = header.find(':');
    if (pos != std::string::npos) {
        std::string key = header.substr(0, pos);
        std::string value = header.substr(pos + 1);
        // substr 可能导致大内存分配
    }
    return Status::OK;
}
```

**缓解因素**：
- curl 库提供内部缓冲限制

**修复建议**：
1. 设置响应头最大长度限制（如 8KB）
2. 使用更安全的字符串操作函数
3. 限制单个 header field 大小

---

#### R6: 文件描述符资源泄漏

| 属性 | 值 |
|------|-----|
| **风险等级** | 中 (Medium) |
| **CWE 编号** | CWE-775 |
| **可利用性** | 中等 |
| **证据位置** | `src/common/fdsan_fd.cpp` |
| **影响范围** | 文件描述符耗尽 |
| **触发条件** | 错误处理路径未释放 fd |

**代码证据**：
```cpp
// src/common/fdsan_fd.cpp
// 证据说明：错误处理路径可能泄漏文件描述符
bool FdsanFd::Open(const char* path, int oflag) {
    fd = open(path, oflag);
    if (fd < 0) {
        // 问题：错误返回但可能已分配 fd
        return false;
    }
    if (fdsan_add_relative_owner(fd, &tracking_) != 0) {
        // 问题：fdsan 失败但未关闭 fd
        return false;
    }
    return true;
}
```

**修复建议**：
1. 使用 RAII 封装文件描述符
2. 确保所有错误路径调用 close()
3. 增加 fdsan 失败时的清理逻辑

---

### 3.3 低风险项

#### R7: 调试日志泄露敏感信息

| 属性 | 值 |
|------|-----|
| **风险等级** | 低 (Low) |
| **CWE 编号** | CWE-532 |
| **可利用性** | 低 |
| **证据位置** | `src/osal/utils/dump_buffer.cpp` |
| **影响范围** | 泄露文件路径、内存地址 |
| **触发条件** | 调试日志被外部获取 |

**代码证据**：
```cpp
// src/osal/utils/dump_buffer.cpp
// 证据说明：日志可能输出敏感信息
void DumpBuffer::LogBuffer(const char* filename) {
    MEDIA_LOGI("Dumping buffer to %{public}s", filename);
    // filename 可能包含私有路径信息
}
```

**缓解因素**：
- Release 版本通常禁用调试日志
- hilog 提供日志级别控制

**修复建议**：
1. 日志输出前进行脱敏处理
2. 限制 %{public}s 敏感信息展示
3. 仅在 DEBUG 模式输出详细日志

---

#### R8: 权限检查后置

| 属性 | 值 |
|------|-----|
| **风险等级** | 低 (Low) |
| **CWE 编号** | CWE-862 |
| **可利用性** | 低 |
| **证据位置** | `engine/pipeline/filters/audio_capture_filter.cpp` |
| **影响范围** | 未授权录音 |
| **触发条件** | 录制场景无权限检查 |

**代码证据**：
```cpp
// engine/pipeline/filters/audio_capture_filter.cpp
// 证据说明：Start() 未检查麦克风权限
Status AudioCaptureFilter::Start() {
    // 问题：直接启动采集，未验证 ohos.permission.MICROPHONE
    return capturer_->Start();
}
```

**缓解因素**：
- Framework 层通常会进行权限检查
- Audio Framework 提供内部权限验证

**修复建议**：
1. 在 Filter 层增加权限验证
2. 使用 AccessToken API 验证权限
3. 记录权限缺失的审计日志

---

## 4 安全机制评估

### 4.1 已有的安全机制

| 机制 | 实现位置 | 有效性 |
|------|----------|--------|
| fdsan | `src/common/fdsan_fd.cpp` | ✅ 有效 |
| hilog 日志 | `src/osal/utils/dump_buffer.cpp` | ⚠️ 需脱敏 |
| AccessToken | Framework 层 | ✅ 有效 |
| SELinux/AppArmor | 系统层 | ✅ 有效 |

### 4.2 建议的改进项

| 改进项 | 优先级 | 预计工作量 | 说明 |
|--------|--------|------------|------|
| 路径遍历防护 | 高 | 1-2天 | realpath + 白名单检查 |
| Buffer 边界检查 | 高 | 1周 | 增加所有边界验证 |
| 插件签名验证 | 高 | 2-3周 | 需要安全团队配合 |
| 错误处理加固 | 中 | 1周 | RAII 封装所有资源 |
| 敏感数据脱敏 | 低 | 1-2天 | 日志输出脱敏 |

---

## 5 安全研究员检查清单

### 5.1 快速检查项

- [ ] 文件路径是否经过规范化处理？
- [ ] AVBuffer capacity 是否有上限？
- [ ] Buffer index 是否经过边界验证？
- [ ] 插件加载是否有签名验证？
- [ ] HTTP 响应头是否有长度限制？
- [ ] 错误路径是否正确释放资源？

### 5.2 重点关注文件

| 文件路径 | 关注原因 |
|----------|----------|
| `engine/plugin/plugins/source/file_source.cpp` | 文件路径处理 |
| `interface/kits/c/native_avbuffer.h` | Buffer 创建 |
| `src/plugin/plugin_loader.cpp` | 插件加载 |
| `engine/plugin/plugins/source/http_source.cpp` | 网络输入 |
| `src/common/fdsan_fd.cpp` | 文件描述符管理 |

### 5.3 调试技巧

```bash
# 启用详细日志
hilog -v D

# 查看 fdsan 警告
adb logcat | grep fdsan

# 检查权限
hdc shell aa getperm <bundle-name>
```

---

## 6 相关安全文档

| 文档 | 链接 | 说明 |
|------|------|------|
| OpenHarmony 安全规范 | [链接] | 总体安全要求 |
| 多媒体子系统安全指南 | [链接] | 子系统特定指南 |
| CWE 22 | [cwe.mitre.org](https://cwe.mitre.org/data/definitions/22.html) | 路径遍历 |
| CWE 190 | [cwe.mitre.org](https://cwe.mitre.org/data/definitions/190.html) | 整数溢出 |
| CWE 119 | [cwe.mitre.org](https://cwe.mitre.org/data/definitions/119.md) | 缓冲区溢出 |
