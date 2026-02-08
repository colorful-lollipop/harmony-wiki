# VPE 常见问题与调试指南

本文档汇总 VPE 视频处理引擎的常见构建、运行和调试问题，提供定位路径和解决方案。

---

## 1 构建问题

### 1.1 编译错误

#### 问题 1.1.1 缺少头文件

**错误信息**：
```
fatal error: 'xxx.h' file not found
```

**可能原因**：
- 头文件路径未正确配置
- 依赖模块未编译
- GN 配置 include_dirs 缺失

**解决方案**：

```bash
# 1. 检查依赖模块是否已编译
./build.sh --product-name {product_name} --build-target multimedia

# 2. 检查头文件是否存在
find /path/to/openharmony -name "xxx.h"

# 3. 检查 BUILD.gn 配置
grep -r "include_dirs" /path/to/video_processing_engine/framework/BUILD.gn
```

**证据位置**：`framework/BUILD.gn:62-88` ✅

#### 问题 1.1.2 链接错误

**错误信息**：
```
undefined reference to `xxx'
```

**可能原因**：
- 依赖库未链接
- 符号导出问题
- 静态库顺序问题

**解决方案**：

```bash
# 1. 检查依赖库
ldd libvideoprocessingengine.so

# 2. 检查符号导出
nm -D libvideoprocessingengine.so | grep "xxx"

# 3. 检查 BUILD.gn deps 配置
grep -A5 "deps" /path/to/video_processing_engine/framework/BUILD.gn
```

**证据位置**：`framework/BUILD.gn:206-243` ✅

### 1.2 配置问题

#### 问题 1.2.1 has_skia 变量未定义

**错误信息**：
```
error: 'has_skia' is not defined
```

**可能原因**：
- 全局 parts_info 未正确配置
- Skia 第三方库未启用

**解决方案**：

```bash
# 在 build.sh 中启用 Skia
./build.sh --product-name {product_name} \
    --build-target video_processing_engine \
    -Dglobal_parts_info.third_party_skia=true
```

**证据位置**：`config.gni:95-99` ✅

#### 问题 1.2.2 NDK 版本不匹配

**错误信息**：
```
error: invalid ndk version
```

**可能原因**：
- NDK 版本过低
- 系统能力声明不匹配

**解决方案**：

```bash
# 检查 NDK 版本要求
cat /path/to/video_processing_engine/interfaces/kits/c/video_processing/BUILD.gn

# 更新 build 配置
# 确保 ndk_version >= 12 (视频) / 13 (图像)
```

**证据位置**：`interfaces/kits/c/video_processing/BUILD.gn` ✅

---

## 2 运行问题

### 2.1 SA 服务启动失败

#### 问题 2.1.1 SA 未注册

**错误信息**：
```
Failed to get system ability: 66134
```

**可能原因**：
- SA 服务未正确注册
- SA 配置文件缺失
- 权限不足

**排查步骤**：

```bash
# 1. 检查 SA 配置
cat /system/profile/video_processing_service.json

# 2. 检查 SA 是否运行
hidumper -s 66134

# 3. 检查日志
hilog | grep -i "video_processing"

# 4. 检查 SELinux 权限
dmesg | grep -i "avc"
```

**证据位置**：`services/sa_profile/66134.json` ✅

#### 问题 2.1.2 SA 加载超时

**错误信息**：
```
LoadSystemAbility timeout
```

**可能原因**：
- SA 初始化时间过长
- 算法库加载阻塞
- 死锁

**解决方案**：

```bash
# 1. 检查 SA 初始化日志
hilog | grep -E "VideoProcessingServer|OnStart|OnStop"

# 2. 使用 strace 追踪
strace -f -p <sa_pid> -e trace=open,openat,read

# 3. 检查是否有死锁
kill -3 <sa_pid>  # 打印线程栈
```

### 2.2 API 调用失败

#### 问题 2.2.1 创建实例失败

**错误信息**：
```
IMAGE_PROCESSING_ERROR_CREATE_FAILED
```

**可能原因**：
- 内存不足
- 算法初始化失败
- 设备不支持

**排查步骤**：

```cpp
// 添加调试日志
ImageProcessing_ErrorCode ret = OH_ImageProcessing_Create(&processor, type);
if (ret != IMAGE_PROCESSING_SUCCESS) {
    VPE_LOGE("Create failed: %{public}d", ret);
    // 打印详细错误
}
```

**证据位置**：`image_processing.h:150` ✅

#### 问题 2.2.2 处理返回错误

**错误信息**：
```
VIDEO_PROCESSING_ERROR_PROCESS_FAILED
```

**可能原因**：
- 输入参数无效
- 算法执行失败
- SurfaceBuffer 格式不支持

**排查步骤**：

```cpp
// 1. 检查输入参数
if (input == nullptr || output == nullptr) {
    VPE_LOGE("Null input/output");
    return ERROR;
}

// 2. 检查格式支持
bool supported = OH_ImageProcessing_IsColorSpaceConversionSupported(
    &sourceInfo, &destInfo);
if (!supported) {
    VPE_LOGE("Format not supported");
    return UNSUPPORTED;
}

// 3. 检查 SurfaceBuffer 属性
VPE_LOGD("Input format: %{public}d", input->GetFormat());
VPE_LOGD("Output format: %{public}d", output->GetFormat());
```

**证据位置**：`video_processing.h:230-244` ✅

### 2.3 内存问题

#### 问题 2.3.1 内存泄漏

**排查步骤**：

```bash
# 使用 ASan 构建
./build.sh --product-name {product_name} \
    --asan \
    --build-target video_processing_engine

# 运行测试，观察 ASan 输出
hilog | grep -i "ASAN\|memory leak"
```

**证据位置**：`framework/BUILD.gn:150-156` ✅

#### 问题 2.3.2 内存越界

**排查步骤**：

```bash
# 使用 UBSan 构建
./build.sh --product-name {product_name} \
    --ubsan \
    --build-target video_processing_engine

# 检查 dmesg
dmesg | grep -i "ubsan\|out of bounds"
```

---

## 3 调试方法

### 3.1 日志调试

#### 3.1.1 启用详细日志

```cpp
// 设置日志级别
export VPE_LOG_LEVEL=DEBUG

// 或在代码中设置
VpeLog::SetLevel(LOG_LEVEL_DEBUG);
```

#### 3.1.2 关键日志位置

| 日志标签 | 说明 | 证据位置 |
|---------|------|---------|
| `VPE` | 核心日志 | `vpe_log.cpp` ✅ |
| `VideoProcessingServer` | SA 服务日志 | `video_processing_server.cpp` ✅ |
| `DetailEnhanceNapi` | NAPI 日志 | `detail_enhance_napi.cpp` ✅ |

**日志过滤**：

```bash
# 过滤 VPE 日志
hilog | grep -E "VPE|VideoProcessing|DetailEnhance"

# 按标签过滤
hilog -T default | grep "VPE"
```

### 3.2 追踪调试

#### 3.2.1 HITRACE 追踪

```cpp
// 在代码中添加追踪点
#include "hitrace_meter.h"

void ProcessImage() {
    HITRACE_METER_NAME(HITRACE_TAG_GRAPHIC_AGP, __func__);
    // 处理逻辑
}
```

**证据位置**：`detail_enhance_napi.cpp:93` ✅

#### 3.2.2 性能分析

```bash
# 使用 systrace
python3 systrace.py \
    -a com.example.app \
    -b 16384 \
    -o trace.html \
    gfx view wm am video_processing
```

### 3.3 GDB 调试

#### 3.3.1 附加调试

```bash
# 1. 获取进程 ID
pid = $(pidof videoprocessing_service)

# 2. 附加 GDB
gdb -p $pid

# 3. 设置断点
break VideoProcessingServer::Process
break DetailEnhancerImage::Process

# 4. 继续执行
continue
```

#### 3.3.2 调试 N-API

```bash
# 1. 设置 NAPI 调试
export ACE_NAPI_DEBUG=1

# 2. 运行应用
./bin/app

# 3. 查看 NAPI 调用栈
hilog | grep -i "napi"
```

### 3.4 内核调试

#### 3.4.1 Binder 调试

```bash
# 查看 Binder 事务
cat /sys/kernel/debug/binder/transactions

# 查看 Binder 状态
cat /sys/kernel/debug/binder/stats

# 清除 Binder 调试信息
echo clear > /sys/kernel/debug/binder/state
```

---

## 4 问题定位路径

### 4.1 问题类型与定位路径

| 问题类型 | 定位路径 | 关键文件 |
|---------|---------|---------|
| **SA 启动问题** | SA 配置 → 服务日志 → 初始化代码 | `video_processing_server.cpp` ✅ |
| **API 调用失败** | API 文档 → 错误码 → 框架代码 | `image_processing.h` ✅ |
| **算法执行错误** | 插件加载 → 算法初始化 → 具体实现 | `extension_manager.cpp` ✅ |
| **内存问题** | ASan/UBSan 输出 → 源码定位 | `framework/BUILD.gn` ✅ |
| **IPC 通信问题** | IDL 定义 → Proxy/Stub → Binder | `IVideoProcessingServiceManager.idl` ✅ |

### 4.2 关键调试文件

| 文件 | 用途 | 证据位置 |
|------|------|---------|
| `vpe_log.cpp` | 日志实现 | `framework/dfx/` ✅ |
| `vpe_trace.cpp` | 追踪实现 | `framework/dfx/` ✅ |
| `video_processing_server.cpp` | SA 服务端 | `services/src/` ✅ |
| `video_processing_client.cpp` | SA 客户端 | `services/src/` ✅ |
| `detail_enhance_napi.cpp` | NAPI 实现 | `framework/capi/` ✅ |

### 4.3 常用调试命令

```bash
# 查看进程
ps -A | grep video_processing

# 查看线程
ps -T -p <pid>

# 查看内存使用
cat /proc/<pid>/status

# 查看打开文件
lsof -p <pid>

# 查看网络连接
netstat -an | grep <pid>

# 查看系统调用
strace -p <pid> -f -e trace=open,read,write
```

---

## 5 性能问题

### 5.1 性能分析

#### 5.1.1 CPU 使用率

```bash
# 使用 top 分析 CPU 使用
top -H -p <pid>

# 使用 perf 分析热点
perf top -p <pid>
```

#### 5.1.2 内存使用

```bash
# 查看内存分布
cat /proc/<pid>/smaps

# 使用 massif 堆分析
valgrind --tool=massif ./app
```

### 5.2 常见性能问题

| 问题 | 可能原因 | 解决方案 |
|------|---------|---------|
| **初始化慢** | 算法库加载 | 延迟初始化 |
| **处理延迟高** | 同步阻塞 | 异步处理 |
| **内存占用高** | 缓存过大 | 限制缓存大小 |
| **CPU 占用高** | 算法复杂度高 | 优化算法或降级 |

---

## 6 日志分析

### 6.1 日志级别

| 级别 | 说明 | 使用场景 |
|------|------|---------|
| **ERROR** | 错误 | API 调用失败 |
| **WARN** | 警告 | 可恢复的错误 |
| **INFO** | 信息 | 关键状态变化 |
| **DEBUG** | 调试 | 详细执行流程 |
| **TRACE** | 追踪 | 性能追踪点 |

### 6.2 日志格式

```bash
# VPE 日志格式
# [时间] [级别] [标签] [文件:行号] 消息

# 示例
02-06 10:30:15.123  12345 12345 E/VPE     [video_processing_server.cpp:100] Create failed: 0x1
02-06 10:30:15.456  12345 12345 W/VPE     [detail_enhance_napi.cpp:250] Retry processing
```

---

## 7 常见错误码速查

### 7.1 图像处理错误码

| 错误码 | 定义 | 解决方法 |
|--------|------|---------|
| `-1` | IMAGE_PROCESSING_ERROR | 检查日志 |
| `-2` | INVALID_INSTANCE | 重新创建实例 |
| `-3` | INVALID_PARAMETER | 检查参数有效性 |
| `-4` | INVALID_VALUE | 检查值范围 |
| `-5` | UNSUPPORTED_PROCESSING | 检查设备支持 |
| `-6` | PROCESS_FAILED | 检查输入数据 |
| `-7` | NO_MEMORY | 释放内存后重试 |
| `-8` | INITIALIZE_FAILED | 检查系统资源 |
| `-9` | OPERATION_NOT_PERMITTED | 检查调用顺序 |

### 7.2 视频处理错误码

| 错误码 | 定义 | 解决方法 |
|--------|------|---------|
| `-1` | VIDEO_PROCESSING_ERROR | 检查日志 |
| `-2` | INVALID_INSTANCE | 重新创建实例 |
| `-3` | INVALID_PARAMETER | 检查参数有效性 |
| `-4` | INVALID_VALUE | 检查值范围 |
| `-5` | UNSUPPORTED_PROCESSING | 检查设备支持 |
| `-6` | PROCESS_FAILED | 检查 Surface 状态 |
| `-7` | NO_MEMORY | 释放内存后重试 |
| `-8` | INITIALIZE_FAILED | 检查系统资源 |
| `-9` | OPERATION_NOT_PERMITTED | 检查是否已启动/停止 |

---

## 8 相关文档链接

| 文档 | 说明 |
|------|------|
| [Architecture.md](./Architecture.md) | 架构设计 |
| [NAPI_Reference.md](./NAPI_Reference.md) | API 参考 |
| [Build_System.md](./Build_System.md) | 构建系统 |
| [Artifacts.md](./Artifacts.md) | 编译产物 |
| [Security_Review.md](./Security_Review.md) | 安全评审 |

---

## 9 更新日志

| 版本 | 日期 | 变更内容 |
|------|------|---------|
| 1.0 | 2026-02-06 | 初始版本，包含常见问题与调试指南 |
