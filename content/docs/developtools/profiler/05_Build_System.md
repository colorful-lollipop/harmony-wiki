# 05_Build_System - GN 构建系统

## 1. 构建配置概览

### 1.1 根构建文件

| 文件 | 用途 |
|------|------|
| `device/BUILD.gn` | 设备侧主构建入口 |
| `build/config.gni` | 主配置（路径、特性开关） |
| `device/base/config.gni` | 测试配置 |
| `protos/protos.gni` | Protobuf 生成配置 |

### 1.2 构建目标分组

```gn
# device/BUILD.gn
group("hiprofiler_targets") {
  deps = [
    "cmds:hiprofiler_cmd",
    "plugins/api:hiprofiler_plugins",
    "services/profiler_service:hiprofilerd",
    # ... 更多插件
  ]
}

group("unittest") {      # 单元测试
  testonly = true
  deps = [ ... ]
}

group("fuzztest") {      # Fuzz 测试
  testonly = true
  deps = [ ... ]
}
```

---

## 2. 关键 Targets

### 2.1 可执行文件 (ohos_executable)

| Target | 产物 | 源码 | 依赖 |
|--------|------|------|------|
| `hiprofiler_cmd` | `hiprofiler_cmd` | `device/cmds/src/` | grpc, protobuf |
| `native_daemon` | `native_daemon` | `native_daemon/` | access_token, zlib, hook |
| `hiebpf` | `hiebpf` | `hiebpf/src/` | libbpf |
| `timestamps` | `timestamps` | `timestamps/` | - |
| `pagemap_parse` | `pagemap_parse` | `tools/pagemap_parse/` | - |

### 2.2 共享库 (ohos_shared_library)

| Target | 产物 | 用途 | 关键依赖 |
|--------|------|------|----------|
| `hiprofilerd` | `hiprofilerd.so` | Profiler 服务主进程 | grpc, abseil |
| `hiprofiler_plugins` | `hiprofiler_plugins.so` | 插件框架 | - |
| `libhidebug` | `libhidebug.so` | Native 调试 | hilog, init |
| `hidebug_native` | `hidebug_native.so` | Native 接口 | ability_runtime, ffrt |
| `ohhidebug` | `ohhidebug.so` | NDK 接口 | faultloggerd |
| `hidebug` | `hidebug.so` | N-API 模块 | napi, hidebug_native |

#### 插件共享库

| Target | 产物 | 类型 | 版本脚本 |
|--------|------|------|----------|
| `cpudataplugin` | `cpudataplugin.so` | 轮询 | `libcpu_plugin.map` |
| `memdataplugin` | `memdataplugin.so` | 轮询 | `libmemory_plugin.map` |
| `ftrace_plugin` | `ftrace_plugin.so` | 流式 | `libftrace_plugin.map` |
| `hilogplugin` | `hilogplugin.so` | 流式 | `libhilog_plugin.map` |
| `hiperfplugin` | `hiperfplugin.so` | 独立文件 | `libhiperf_plugin.map` |
| `native_hook` | `native_hook.so` | 轮询 | `libnative_hook.map` |
| `gpudataplugin` | `gpudataplugin.so` | 轮询 | `libgpu_plugin.map` |
| `diskiodataplugin` | `diskiodataplugin.so` | 轮询 | `libdiskio_plugin.map` |
| `networkplugin` | `networkplugin.so` | 轮询 | `libnetwork_plugin.map` |
| `processplugin` | `processplugin.so` | 轮询 | `libprocess_plugin.map` |
| `hiebpfplugin` | `hiebpfplugin.so` | 独立文件 | `libhiebpf_plugin.map` |
| `hidumpplugin` | `hidumpplugin.so` | 流式 | `libhidump_plugin.map` |
| `hisyseventplugin` | `hisyseventplugin.so` | 轮询 | `libhisysevent_plugin.map` |
| `bytraceplugin` | `bytraceplugin.so` | 流式 | - |
| `streamplugin` | `streamplugin.so` | 流式 | - |
| `xpowerplugin` | `xpowerplugin.so` | 轮询 | `libxpower_plugin.map` |
| `sampleplugin` | `sampleplugin.so` | 轮询 | - |
| `ffrt_profiler` | `libffrt_profiler.so` | 轮询 | - |
| `network_profiler` | `libnetwork_profiler.so` | 轮询 | - |

### 2.3 静态库 (ohos_static_library)

| Target | 产物 | 用途 |
|--------|------|------|
| `shared_memory_lite` | `shared_memory_lite.a` | 无 protobuf 共享内存 |
| `libhidebug_init` | `libhidebug_init.a` | Init 阶段 hidebug |

### 2.4 源码集 (ohos_source_set)

| Target | 用途 |
|--------|------|
| `hiprofiler_base` | 基础框架（事件轮询、调度） |
| `profiler_service` | Profiler 服务实现 |
| `hiprofiler_plugin_service` | 插件管理服务 |
| `plugins_sources` | 插件 API |
| `shared_memory_source` | 共享内存管理 |
| `proto_encoder_source` | Protobuf 编码器 |
| `native_hook_source` | Hook 框架 |
| `ipc` | Unix Socket IPC |
| `hiebpf_source_common` | eBPF 通用代码 |

---

## 3. 编译产物

### 3.1 系统库 (system/)

```
/system/lib/
├── libhiprofilerd.so                    # Profiler 服务
├── libhiprofiler_plugins.so             # 插件框架
├── libhidebug.so                        # N-API 模块
├── libhidebug_native.so                 # Native 接口
├── libohhidebug.so                      # NDK 接口
├── libshared_memory.so                  # 共享内存
├── libcpudataplugin.so                  # CPU 插件
├── libmemdataplugin.so                  # 内存插件
├── libftrace_plugin.so                  # Ftrace 插件
├── libhilog_plugin.so                   # Hilog 插件
├── libhiperf_plugin.so                  # HiPerf 插件
├── libnative_hook.so                    # Native Hook
├── libgpu_plugin.so                     # GPU 插件
├── libdiskio_plugin.so                  # 磁盘 I/O 插件
├── libnetwork_plugin.so                 # 网络插件
├── libprocess_plugin.so                 # 进程插件
├── libhiebpf_plugin.so                  # eBPF 插件
├── libhidump_plugin.so                  # HiDump 插件
├── libhisysevent_plugin.so              # HiSysEvent 插件
├── libstream_plugin.so                  # 流式插件
├── libsample_plugin.so                  # 采样插件
├── libxpower_plugin.so                  # 功耗插件
├── libffrt_profiler.so                  # FFRT 插件
├── libnetwork_profiler.so               # 网络性能插件
├── libbytrace_plugin.so                 # Bytrace 插件
└── libnative_daemon_client.so           # Native Daemon 客户端
```

### 3.2 系统可执行文件 (system/bin/)

```
/system/bin/
├── hiprofiler_cmd                       # 命令行工具
├── native_daemon                        # Native 守护进程
└── native_daemon_client                 # 客户端工具
```

### 3.3 开发者工具 (vendor/)

```
/vendor/bin/
├── hiebpf                              # eBPF 工具
├── timestamps                          # 时间戳工具
└── pagemap_parse                       # pagemap 解析
```

### 3.4 NDK 库 (ndk/)

```
/ndk/lib/
└── libohhidebug.so                     # NDK hidebug
```

---

## 4. 关键依赖

### 4.1 第三方库

| 库 | 用途 | 配置 |
|------|------|------|
| `grpc` | RPC 框架 | `grpc:grpc`, `grpc:grpcxx` |
| `protobuf` | 序列化 | `protobuf:protobuf_lite`, `protobuf:protobuf` |
| `openssl` | 加密 | `openssl:libcrypto_shared` |
| `abseil-cpp` | C++ 基础库 | `abseil-cpp:absl_sync`, `abseil-cpp:absl_cord` |
| `libbpf` | eBPF 支持 | `libbpf:libbpf` |
| `zlib` | 压缩 | `zlib:libz` |
| `cJSON` | JSON 解析 | `cJSON:cjson` |
| `libunwind` | 栈展开 | `faultloggerd:libunwinder` |

### 4.2 OpenHarmony 系统组件

| 组件 | 用途 |
|------|------|
| `hilog` | 日志系统 |
| `hisysevent` | 系统事件 |
| `ipc` | IPC 框架 |
| `samgr` | 服务管理 |
| `safwk` | 系统能力框架 |
| `bundle_framework` | 包框架 |
| `access_token` | 访问令牌 |
| `ability_runtime` | 能力运行时 |
| `ffrt` | FFRT 调度 |
| `hitrace` | 追踪 |
| `faultloggerd` | 故障日志 |

---

## 5. 特性开关

### 5.1 编译开关

| 开关 | 默认值 | 用途 |
|------|--------|------|
| `HAVE_HILOG` | 开启 | 启用 hilog 日志 |
| `HOOK_ENABLE` | 开启 | 启用 Hook 功能 |
| `NO_PROTOBUF` | 关闭 | 禁用 protobuf (lite 模式) |
| `LITE_PROTO` | 关闭 | 使用 lite protobuf |
| `HIDEBUG_IN_INIT` | 关闭 | Init 阶段编译 |
| `ENABLE_HAP_EXTRACTOR` | 开启 | 启用 HAP 提取 |
| `PERFORMANCE_DEBUG` | 关闭 | 性能调试模式 |

### 5.2 条件编译

```gn
# 根设备构建
if (is_linux && use_libunwind) {
  defines += [ "HAVE_LIBUNWINDER=1" ]
}

# Native Hook (musl + 非 ASan)
if (is_musl && !enable_asan) {
  deps += [ ":native_hook" ]
}

# eBPF (arm64 + root)
if (arm_arch == "arm64" && ro.enable_root_security) {
  deps += [ ":ebpf_targets" ]
}
```

---

## 6. 构建示例

### 6.1 全量构建

```bash
# 构建所有 profiler 组件
hb set -p
hb build -f

# 或使用 GN 直接构建
gn gen out/default
ninja -C out/default hiprofiler_targets
```

### 6.2 模块构建

```bash
# 仅构建 hidebug N-API
ninja -C out/default hidebug

# 仅构建 hiprofiler_cmd
ninja -C out/default hiprofiler_cmd

# 仅构建插件
ninja -C out/default cpudataplugin memdataplugin
```

### 6.3 测试构建

```bash
# 单元测试
ninja -C out/default unittest

# Fuzz 测试
ninja -C out/default fuzztest
```

---

## 7. 常见问题

### Q1: 编译提示找不到 protobuf

**解决**: 确保 `protobuf` 子系统已同步

```bash
# 检查依赖
hb env | grep protobuf

# 同步依赖
hb sync -p
```

### Q2: native_hook 编译失败

**原因**: ASan 与 Hook 不兼容

**解决**: 使用非 ASan 构建

```bash
# 禁用 ASan
gn gen out/default --args="enable_asan=false"
ninja -C out/default native_hook
```

### Q3: 插件版本脚本错误

**解决**: 检查 `version_script` 文件路径

```gn
ohos_shared_library("myplugin") {
  output_name = "myplugin"
  sources = [ ... ]
  version_script = "//path/to/libmyplugin.map"  # 使用绝对路径
}
```

---

## 8. 相关跳转

| 主题 | 链接 |
|------|------|
| 项目概览 | [00_Overview.md](./00_Overview.md) |
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| 插件系统 | [02_Plugin_System.md](./02_Plugin_System.md) |
| 安全评审 | [06_Security_Review.md](./06_Security_Review.md) |
| 故障排查 | [07_Troubleshooting.md](./07_Troubleshooting.md) |

---

*最后更新: 2026-02-06*
