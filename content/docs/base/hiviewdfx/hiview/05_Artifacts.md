# 编译产物

## 概述

本文档描述 Hiview 模块的编译产物清单、安装路径和运行时加载关系。

> 基于 `BUILD.gn` 构建配置和 OpenHarmony 标准产物路径

---

## 产物清单

### 可执行文件

| 产物名 | Target | 安装路径 | 说明 |
|--------|--------|----------|------|
| `hiview` | `ohos_executable` | `/system/bin/hiview` | Hiview 主服务 |

### 共享库 (.so)

| 产物名 | Target | 安装路径 | 说明 |
|--------|--------|----------|------|
| `libhiviewbase.so` | `hiviewbase` | `/system/lib64/` | 基础库 |
| `libfaultlogger.so` | `libfaultlogger` | `/system/lib64/` | 故障日志库 |
| `libucollectionclient.so` | `libucollection_client` | `/system/lib64/` | 统一采集客户端 |
| `libtrace_manager.so` | `libtrace_manager` | `/system/lib64/` | Trace 管理库 |
| `libdft.so` | `adft` | `/system/lib64/` | DFT 插件库 |
| `libdfr.so` | `bdfr` | `/system/lib64/` | DFR 插件库 |
| `libxperformance.so` | `xperformance` | `/system/lib64/` | 性能监控库 |
| `libeventloggerso.so` | `eventloggerso` | `/system/lib64/` | 事件日志库 |
| `libperfmonitor.so` | `libperfmonitor` | `/system/lib64/` | 性能监控服务 |

### N-API 模块 (.so)

| 产物名 | Target | 安装路径 | 说明 |
|--------|--------|----------|------|
| `libfaultlogger_napi.so` | `faultlogger_napi` | `@ohos/faultLogger/` | 故障日志 N-API |
| `libloglibrary_napi.so` | `loglibrary_napi` | `@ohos/logLibrary/` | 日志库 N-API |

### N-API Extension 模块 (.so)

| 产物名 | 安装路径 | 说明 |
|--------|----------|------|
| `libfaultlogextensionability_napi.so` | `@ability/fault_log_extension/` | FaultLog Extension Ability |
| `libfaultlogextensioncontext_napi.so` | `@ability/fault_log_extension/` | FaultLog Extension Context |

### 配置文件

| 产物名 | 源文件 | 安装路径 | 说明 |
|--------|--------|----------|------|
| `plugin_config` | `plugin_config` | `/system/etc/hiview/` | 插件配置 |
| `plugin_bundle.json` | `bundle_config/config/plugin_bundle.json` | `/data/hiview/bundle/` | 插件包配置 |
| `hiview.cfg` | `config/hiview.cfg` | `/system/etc/init/` | Hiview 启动配置 |
| `log_type.json` | `config/log_type.json` | `/data/hiview/` | 日志类型配置 |
| `hiview.para.dac` | `config/hiview.para.dac` | `/system parameter/` | DAC 配置 |
| `monitor_config` | `config/monitor.cfg` | `/system/etc/hiview/` | 监控配置 |

---

## 运行时加载关系

### 主服务依赖

```
hiview (executable)
    │
    ├── libhiviewbase.so
    │   ├── liblogger.so
    │   ├── libhiview_event_*.so
    │   └── libhiview_utility.so
    │
    ├── libfaultlogger.so
    │   └── (依赖系统 IPC 库)
    │
    └── libucollectionclient.so
        └── (依赖统一采集框架)
```

### 插件加载

```
hiview (主进程)
    │
    ├── plugin_config (配置文件)
    │
    ├── Load plugins via PluginBundle
    │   │
    │   ├── libdft.so (DFT 插件)
    │   │   ├── libsysevent_source.so
    │   │   ├── libunified_collector.so
    │   │   └── libevent_store.so
    │   │
    │   └── libdfr.so (DFR 插件)
    │       ├── libfaultlogger.so
    │       ├── libbbox_detectors.so
    │       └──istleak_detectors.so
```

### N-API 模块加载

```
JS Engine (ArkTS/JS)
    │
    ├── @ohos.faultLogger (Native Module)
    │   └── libfaultlogger_napi.so
    │       └── libfaultlogger.so
    │
    └── @ohos.logLibrary (Native Module)
        └── libloglibrary_napi.so
            └── libhiviewbase.so
```

---

## SA 注册与绑定

### System Ability ID 映射

| SA ID | 服务名 | 实现库 | 端口号 |
|-------|--------|--------|--------|
| `DFX_SYS_HIVIEW_ABILITY_ID` | HiviewServiceAbility | `hiview` (内置) | 1201 |
| `DFX_SYS_EVENT_SERVICE_ABILITY_ID` | SysEventServiceOhos | `libdft.so` | 1202 |
| `DFX_FAULT_LOGGER_ABILITY_ID` | FaultloggerServiceOhos | `libfaultlogger.so` | 1203 |
| `XPERF_SERVICE_SA_ID` | XperfServiceServer | `libperfmonitor.so` | 1401 |

---

## 数据存储路径

| 路径 | 用途 | 权限 |
|------|------|------|
| `/data/log/` | 日志文件存储 | system:rw |
| `/data/hiview/` | Hiview 数据目录 | system:rw |
| `/data/hiview/bundle/` | 插件配置文件 | system:rw |
| `/data/hiview/faultlog/` | 故障日志 | system:rw |
| `/data/hiview/trace/` | Trace 数据 | system:rw |

---

## 运行时配置

### 环境变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `HIVIEW_HOME` | Hiview 数据目录 | `/data/hiview/` |
| `HIVIEW_LOG_PATH` | 日志路径 | `/data/log/` |

### 启动参数

```bash
hiview [options]
    --help          # 显示帮助
    --version       # 显示版本
    --config <path> # 指定配置文件路径
    --daemon        # 后台运行模式
```
