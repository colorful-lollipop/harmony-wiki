# FFRT 编译构建

## 构建系统

FFRT 支持两种构建系统：

| 构建系统 | 适用场景 | 主要文件 |
|----------|----------|----------|
| **GN** | OpenHarmony 标准构建 | `BUILD.gn`, `ffrt.gni` |
| **CMake** | Linux 独立开发 | `CMakeLists.txt` |

## GN 构建

### 构建入口

**主构建文件**: `BUILD.gn`

```gn
import("//build/ohos.gni")
import("ffrt.gni")

ohos_shared_library("libffrt") {
    # 源码文件列表
    sources = [
        "src/core/entity.cpp",
        "src/core/task.cpp",
        # ... 更多文件
    ]

    # 公共配置
    public_configs = [ ":ffrt_config" ]
    
    # 内部配置
    configs = [ ":ffrt_inner_config" ]

    # 外部依赖
    external_deps = [
        "bounds_checking_function:libsec_shared",
        "c_utils:utils",
        "hilog:libhilog",
        "hisysevent:libhisysevent",
        "faultloggerd:libbacktrace_local",
        # ... 更多依赖
    ]

    # 输出配置
    output_extension = "so"
    part_name = "ffrt"
    subsystem_name = "resourceschedule"
}
```

### 配置参数

**GN 配置模板**: `ffrt.gni`

```gn
declare_args() {
    # 功能开关
    ffrt_support_enable = true
    ffrt_async_stack_enable = true
    ffrt_task_local_enable = false
    
    # 内存配置
    ffrt_allocator_mmap_size = "8 * 1024 * 1024"  # 8MB
    ffrt_stack_size = "1 << 20"                     # 1MB
}
```

### 构建宏

| 宏 | 说明 | 默认值 |
|-----|------|--------|
| `FFRT_LOG_LEVEL` | 日志级别 (0-3) | 3 (DEBUG) |
| `FFRT_BBOX_ENABLE` | 黑匣子功能 | 开启 |
| `FFRT_OH_EVENT_RECORD` | 事件录制 | 开启 |
| `FFRT_OH_TRACE_ENABLE` | 追踪功能 | 开启 |
| `FFRT_ALLOCATOR_MMAP_SIZE` | 分配器大小 | 8MB |
| `FFRT_STACK_SIZE` | 协程栈大小 | 1MB |

### 构建命令

```bash
# 编译 FFRT (64位)
./build.sh --product-name rk3568 --target-cpu arm64 --ccache --build-target ffrt

# 编译 FFRT (32位)
./build.sh --product-name rk3568 --ccache --build-target ffrt
```

### 子组件 Targets

| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `libffrt` | ohos_shared_library | `libffrt.z.so` | 主共享库 |
| `ffrt_ndk` | group | - | NDK 导出组 |
| `whitelist_cfg` | ohos_prebuilt_etc | `ffrt_whitelist.conf` | 白名单配置 |

## CMake 构建

### 构建入口

**CMake 文件**: `CMakeLists.txt`

### 构建命令

```bash
# 创建构建目录
mkdir -p build
cd build

# 配置
cmake .. -DFFRT_EXAMPLE=ON -DFFRT_TEST_ENABLE=OFF

# 编译
cmake --build . -j

# 运行示例
./examples/ffrt_submit
```

### 构建选项

| 选项 | 说明 | 默认值 |
|------|------|--------|
| `FFRT_EXAMPLE` | 编译示例 | OFF |
| `FFRT_BENCHMARKS` | 编译基准测试 | OFF |
| `FFRT_TEST_ENABLE` | 编译单元测试 | OFF |
| `FFRT_CLANG_COMPILE` | 使用 Clang 编译 | OFF |
| `FFRT_SANITIZE` | 开启 Sanitizer | OFF |
| `FFRT_BBOX_ENABLE` | 开启黑匣子 | OFF |

## 源码结构

### BUILD.gn 源码分组

| 分组 | 源码文件 | 职责 |
|------|----------|------|
| **core** | entity.cpp, task.cpp, loop_api.cpp | 核心任务管理 |
| **sched** | scheduler.cpp, task_scheduler.cpp | 调度器实现 |
| **eu** | co_routine.cpp, cpu_worker.cpp | 执行单元 |
| **tm** | cpu_task.cpp, io_task.cpp | 任务类型 |
| **queue** | concurrent_queue.cpp, serial_queue.cpp | 队列管理 |
| **sync** | mutex.cpp, condition_variable.cpp | 同步原语 |
| **dm** | dependence_manager.cpp | 依赖管理 |
| **dfx** | log, trace, bbox, watchdog | 维测功能 |
| **util** | cpu_boost.cpp, init.cpp | 工具函数 |
| **ipc** | ipc.cpp | IPC 通信 |

## 依赖配置

### bundle.json 依赖

```json
"deps": {
    "components": [
        "bounds_checking_function",
        "c_utils",
        "hilog",
        "hisysesevent",
        "faultloggerd",
        "napi"
    ]
}
```

### 外部依赖

| 依赖 | 用途 | 配置键 |
|------|------|--------|
| bounds_checking_function | 安全函数 | `libsec_shared` |
| c_utils | C 工具库 | `utils` |
| hilog | 日志系统 | `libhilog` |
| hisysevent | 系统事件 | `libhisysevent` |
| faultloggerd | 故障日志 | `libfaultloggerd` |

## 编译产物

### Linux CMake 产物

```
build/
├── src/
│   └── libffrt.so          # 主共享库
├── examples/
│   └── ffrt_submit         # 示例程序
└── test/
    └── ...
```

### OpenHarmony GN 产物

```
out/{product}/
├── system/
│   └── lib/
│       └── libffrt.z.so    # 主共享库
├── updater/
│   └── lib/
│       └── libffrt.z.so    # 更新器用库
└── etc/
    └── ffrt/
        └── ffrt_whitelist.conf  # 白名单
```

## 编译配置示例

### 调试配置

```bash
# GN 构建开启调试
FFRT_LOG_LEVEL=3 FFRT_BBOX_ENABLE=true ./build.sh --product-name rk3568 --build-target ffrt
```

### 发布配置

```gn
# ffrt.gni
ffrt_log_level = 0  # 关闭日志
# ffrt_release_defines = ["FFRT_RELEASE"]
```

### ASAN 检测

```gn
# 开启 Address Sanitizer
is_asan = true
```

## 常见构建问题

### 问题 1: 缺少依赖

**错误信息**:
```
error: bounds_checking_function not found
```

**解决方案**:
```bash
# 下载第三方依赖到正确位置
├── third_party
    └── bounds_checking_function
```

### 问题 2: CMake 版本过低

**错误信息**:
```
CMake Error: CMake version 3.10 required
```

**解决方案**:
```bash
# 安装新版本 CMake
wget https://github.com/Kitware/CMake/releases/download/v3.25.1/cmake-3.25.1-linux-x86_64.sh
sudo sh cmake-3.25.1-linux-x86_64.sh --skip-license --prefix=/usr/local
```

### 问题 3: 编译内存不足

**解决方案**:
```bash
# 减少并行度
cmake --build . -j 2

# 或增加 swap
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

## 相关文档

- [概览](01_Overview.md) - 项目介绍
- [API 参考](03_API_Reference.md) - 接口说明
- [安全风险](05_Security.md) - 安全配置
- [故障排查](06_Troubleshooting.md) - 问题定位
- [BUILD.md](../BUILD.md) - 官方构建指南
