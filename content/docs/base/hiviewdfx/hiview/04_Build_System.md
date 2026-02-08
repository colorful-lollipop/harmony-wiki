# 构建系统

## GN 构建概述

Hiview 使用 **GN (Generate Ninja)** 作为构建系统，遵循 OpenHarmony 标准构建规范。

> 证据: `BUILD.gn`, `hiview.gni`, `build/hiview_var.gni`

### 构建配置入口

| 文件 | 用途 |
|------|------|
| `BUILD.gn` | 根构建文件，定义主目标和包 |
| `hiview.gni` | Feature flags 和路径定义 |
| `build/hiview_var.gni` | 构建变量和插件配置 |

---

## 主要 Targets

### 可执行文件

| Target | 类型 | 路径 | 依赖 | 说明 |
|--------|------|------|------|------|
| `hiview` | `ohos_executable` | `BUILD.gn` | hiviewbase, hiview_core, hiview_service, plugin_static_deps | 主可执行文件 |

### 共享库

| Target | 类型 | 路径 | 输出 | 说明 |
|--------|------|------|------|------|
| `hiviewbase` | `ohos_shared_library` | `base/BUILD.gn` | `libhiviewbase.so` | 基础库 |
| `libfaultlogger` | `ohos_shared_library` | `plugins/faultlogger/BUILD.gn` | `libfaultlogger.so` | 故障日志库 |
| `libucollection_client` | `ohos_shared_library` | `interfaces/inner_api/unified_collection/client/BUILD.gn` | `libucollectionclient.so` | 统一采集客户端 |
| `libtrace_manager` | `ohos_shared_library` | `framework/native/unified_collection/trace_manager/BUILD.gn` | `libtrace_manager.so` | Trace 管理库 |
| `adft` | `ohos_shared_library` | `plugins/plugin_build/BUILD.gn` | `libdft.so` | DFT 插件库 |
| `bdfr` | `ohos_shared_library` | `plugins/plugin_build/BUILD.gn` | `libdfr.so` | DFR 插件库 |
| `xperformance` | `ohos_shared_library` | `plugins/performance/BUILD.gn` | `libxperformance.so` | 性能监控库（条件编译） |
| `eventloggerso` | `ohos_shared_library` | `plugins/eventlogger/BUILD.gn` | `libeventloggerso.so` | 事件日志库（条件编译） |
| `libperfmonitor` | `ohos_shared_library` | `plugins/performance/xperf_service/services/BUILD.gn` | `libperfmonitor.so` | 性能监控服务 |

### Source Sets

| Target | 路径 | 说明 |
|--------|------|------|
| `hiview_base` | `base/BUILD.gn` | 基础模块 |
| `hiview_core` | `core/BUILD.gn` | 核心模块 |
| `hiview_service` | `service/BUILD.gn` | 服务模块 |
| `hiview_service_adapter` | `adapter/service/BUILD.gn` | 服务适配器 |
| `sysevent_source` | `plugins/sysevent_source/BUILD.gn` | 系统事件源 |
| `faultlogger` | `plugins/faultlogger/BUILD.gn` | 故障日志插件 |
| `eventlogger` | `plugins/eventlogger/BUILD.gn` | 事件日志插件 |
| `freeze_detector` | `plugins/freeze_detector/BUILD.gn` | 冻屏检测插件 |
| `unified_collector` | `plugins/unified_collector/BUILD.gn` | 统一采集器 |
| `usage_event_report` | `plugins/usage_event_report/BUILD.gn` | 使用事件上报 |

### Group Targets

| Target | 路径 | 说明 |
|--------|------|------|
| `hiview_package` | `BUILD.gn` | 主包，包含所有核心组件和插件 |
| `hiview_test_package` | `BUILD.gn` | 测试包 |

---

## 依赖关系

```
hiview (ohos_executable)
├── hiviewbase (ohos_shared_library)
│   ├── logger (ohos_source_set)
│   ├── event_publish:hiview_event_publish
│   ├── event_raw:hiview_event_raw_base
│   │   ├── encode
│   │   └── decode
│   ├── event_report:hiview_event_report
│   ├── event_store:event_store_source
│   ├── logstore:log_store
│   └── utility:hiview_utility
├── hiview_core (ohos_source_set)
│   ├── base:hiviewbase
│   ├── platform_config:hiviewplatform_config
│   └── param_update:hiview_param_update (条件)
└── hiview_service (ohos_source_set)
    ├── adapter/service:hiview_service_adapter
    ├── base:hiviewbase
    ├── core:hiview_core
    └── interfaces/inner_api/unified_collection/utility:libucollection_utility
```

---

## Feature Flags

> 证据: `hiview.gni` 和 `bundle.json:22-61`

### 可靠性相关

| Flag | 默认值 | 说明 |
|------|--------|------|
| `hiview_feature_bbox_userspace` | false | 用户态 BBox |
| `hiview_enable_leak_detector` | false | 内存泄漏检测 |
| `hiview_enable_crash_validator` | true | 崩溃验证器 |

### 性能相关

| Flag | 默认值 | 说明 |
|------|--------|------|
| `hiview_enable_performance_monitor` | false | 性能监控 |
| `hiview_enable_xperf_perfmonitor` | true | XPerf 性能监控 |

### 事件日志相关

| Flag | 默认值 | 说明 |
|------|--------|------|
| `hiview_freeze_collect_enable` | true | 冻屏采集 |
| `hiview_eventlogger_window_manager_enable` | true | 窗口管理捕获 |
| `hiview_eventlogger_stacktrace_catcher_enable` | true | 堆栈捕获 |
| `hiview_eventlogger_binder_catcher_enable` | true | Binder 捕获 |
| `hiview_eventlogger_dmesg_catcher_enable` | true | Dmesg 捕获 |
| `hiview_eventlogger_hilog_catcher_enable` | true | Hilog 捕获 |
| `hiview_eventlogger_hitrace_catcher_enable` | true | Hitrace 捕获 |
| `hiview_eventlogger_usage_catcher_enable` | true | 使用信息捕获 |
| `hiview_eventlogger_scb_catcher_enable` | true | SCB 捕获 |
| `hiview_eventlogger_other_catcher_enable` | true | 其他捕获 |
| `hiview_eventlogger_kernel_catcher_enable` | false | 内核捕获 |

### 统一采集器相关

| Flag | 默认值 | 说明 |
|------|--------|------|
| `hiview_unified_collector_PC_app_state_collect_enable` | false | PC 应用状态采集 |
| `hiview_unified_collector_perf_enable` | true | 性能采集 |
| `hiview_unified_collector_ebpf_enable` | true | eBPF 采集 |
| `hiview_unified_collector_network_enable` | true | 网络采集 |
| `hiview_unified_collector_graphic_enable` | true | 图形采集 |
| `hiview_unified_collector_gpu_enable` | true | GPU 采集 |
| `hiview_unified_collector_cpu_enable` | true | CPU 采集 |
| `hiview_unified_collector_mem_profiler_enable` | true | 内存分析器 |
| `hiview_unified_collector_io_enable` | true | IO 采集 |
| `hiview_unified_collector_thermal_enable` | true | 热信息采集 |
| `hiview_unified_collector_memory_enable` | true | 内存采集 |
| `hiview_unified_collector_hilog_enable` | true | Hilog 采集 |
| `hiview_unified_collector_wm_enable` | true | 窗口管理采集 |
| `hiview_unified_collector_process_enable` | true | 进程采集 |
| `hiview_unified_collector_trace_enable` | true | Trace 采集 |

### 其他特性

| Flag | 默认值 | 说明 |
|------|--------|------|
| `hiview_appevent_publish_enable` | true | 应用事件发布 |
| `hiview_param_update_enable` | true | 参数更新 |
| `hiview_sysevent_store_enable` | true | 系统事件存储 |
| `hiview_privacy_enable` | true | 隐私控制 |
| `hiview_usage_stat_enable` | true | 使用统计 |
| `hiview_usage_fold_stat_enable` | true | 折叠屏使用统计 |

---

## 构建变量

> 证据: `build/hiview_var.gni`

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `plugin_so` | false | 插件是否编译为 SO |
| `plugin_target_platform` | "hisi" | 目标平台 |
| `plugin_target_ram` | "2G" | 目标 RAM |
| `plugin_target_rom` | "32G" | 目标 ROM |
| `build_with_config` | false | 是否使用外部配置构建 |
| `config_path` | "" | 配置文件路径 |

---

## 预配置文件

| Target | 源文件 | 安装路径 | 说明 |
|--------|--------|----------|------|
| `plugin_config` | `plugin_config` | `hiview/` | 插件配置 |
| `plugin_bundle.json` | `bundle_config/config/plugin_bundle.json` | `hiview/bundle/` | 插件包配置 |
| `hiview.cfg` | `config/hiview.cfg` | `init/` | Hiview 启动配置 |
| `log_type.json` | `config/log_type.json` | `hiview/` | 日志类型配置 |
| `adft_plugin_config` | `adft_plugin_config` | `hiview/` | ADFT 插件配置 |
| `bdfr_plugin_config` | `bdfr_plugin_config` | `hiview/` | BDFR 插件配置 |
| `hiview.para.dac` | `config/hiview.para.dac` | `param/` | 参数 DAC 配置 |
| `monitor_config` | `config/monitor.cfg` | `hiview/` | 监控配置 |

---

## 编译命令

### 全量编译

```bash
# 生成构建文件
hb set
hb build

# 或使用 GN + Ninja
gn gen out/default
ninja -C out/default hiview_package
```

### 增量编译

```bash
ninja -C out/default hiview
```

### 编译单个模块

```bash
# 编译故障日志插件
ninja -C out/default libfaultlogger

# 编译 N-API 模块
ninja -C out/default faultlogger_napi
ninja -C out/default loglibrary_napi
```

---

## 构建产物

详见 [编译产物文档](05_Artifacts.md)
