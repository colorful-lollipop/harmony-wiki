# GN 构建目标

## 目的

本文档详细描述 Resource Schedule Service 的 GN 构建配置，包括所有 targets、依赖关系和产物。

## 适用范围

- 构建系统开发者
- 需要修改编译配置的开发者

---

## 构建概述

### 构建入口

| 构建组 | 路径 | 说明 |
|--------|------|------|
| base_group | `//ressched/profile:*` | 基础配置文件 |
| fwk_group | `//ressched/interfaces/**:*` | 框架库和接口 |
| service_group | `//ressched/services:*` 等 | 服务和插件 |

### 全局配置

```
# 文件: bundle.json

build: {
    group_type: {
        base_group: [ ... ],
        fwk_group: [ ... ],
        service_group: [ ... ]
    }
}
```

---

## 核心服务 Targets

### resschedsvc - 主服务

**Target**: `//ressched/services:resschedsvc`

**类型**: `shared_library`

**主要源码**:
```
ressched/services/resschedservice/src/res_sched_service.cpp
ressched/services/resschedservice/src/res_sched_service_ability.cpp
ressched/services/resschedmgr/resschedfwk/src/plugin_mgr.cpp
ressched/services/resschedmgr/resschedfwk/src/res_sched_mgr.cpp
ressched/sched_controller/observer/src/observer_manager.cpp
ressched/scene_recognize/src/scene_recognizer_mgr.cpp
... (41个cpp文件)
```

**关键依赖**:
- `//ressched/interfaces/innerkits/ressched_client:ressched_client`
- `//ressched_executor/interfaces/innerkits/ressched_executor_client:resschedexe_client`
- `//ressched/common:ressched_common_utils`

**输出产物**: `libresschedsvc.z.so` (SA类型)

**安装路径**: `/system/lib64/libresschedsvc.z.so`

---

### resschedsvc_static - 静态库

**Target**: `//ressched/services:resschedsvc_static`

**类型**: `static_library`

**用途**: 单元测试使用

**输出产物**: `libresschedsvc_static.a`

---

### resschedexesvc - 执行器服务

**Target**: `//ressched_executor/services:resschedexesvc`

**类型**: `shared_library`

**主要源码**:
```
ressched_executor/services/resschedexeservice/src/res_sched_exe_service.cpp
ressched_executor/services/resschedexemgr/src/res_sched_exe_mgr.cpp
ressched_executor/services/resschedfwk/src/plugin_mgr.cpp
ressched_executor/services/resschedfwk/src/config_reader.cpp
```

**关键依赖**:
- `//ressched_executor/interfaces/innerkits/ressched_executor_client:resschedexe_client`
- `//ressched/plugins/cgroup_sched_plugin/framework/process_group:libprocess_group`
- `//ressched/common:ressched_common_utils`

**输出产物**: `libresschedexesvc.z.so` (SA类型)

---

## 客户端库 Targets

### ressched_client

**Target**: `//ressched/interfaces/innerkits/ressched_client:ressched_client`

**类型**: `shared_library`

**主要源码**:
```
ressched/interfaces/innerkits/ressched_client/src/res_sched_client.cpp
ressched/interfaces/innerkits/ressched_client/src/res_sa_init.cpp
ressched/interfaces/innerkits/ressched_client/src/kill_event_listener.cpp
ressched/interfaces/innerkits/ressched_client/IResSchedService.idl  (IDL生成)
```

**输出产物**: `libressched_client.z.so`

**inner_kits 导出**:
```json
{
    "header_base": "//ressched/interfaces/innerkits/ressched_client/include",
    "header_files": [
        "res_sa_init.h",
        "res_sched_client.h",
        "res_sched_errors.h",
        "res_type.h",
        "res_sched_ipc_interface_code.h"
    ]
}
```

---

### resschedexe_client

**Target**: `//ressched_executor/interfaces/innerkits/ressched_executor_client:resschedexe_client`

**类型**: `shared_library`

**输出产物**: `libresschedexeclient.z.so`

---

## 插件 Targets

### cgroup_sched_plugin

**Target**: `//ressched/plugins/cgroup_sched_plugin/framework:cgroup_sched`

**类型**: `shared_library`

**主要源码**:
```
ressched/plugins/cgroup_sched_plugin/framework/sched_controller/src/*.cpp
ressched/plugins/cgroup_sched_plugin/framework/utils/src/*.cpp
```

**输出产物**: `libcgroup_sched.z.so`

---

### libprocess_group

**Target**: `//ressched/plugins/cgroup_sched_plugin/framework/process_group:libprocess_group`

**类型**: `shared_library`

**主要源码**:
```
ressched/plugins/cgroup_sched_plugin/framework/process_group/src/sched_policy.cpp
ressched/plugins/cgroup_sched_plugin/framework/process_group/src/cgroup_action.cpp
ressched/plugins/cgroup_sched_plugin/framework/process_group/src/cgroup_controller.cpp
ressched/plugins/cgroup_sched_plugin/framework/process_group/src/cgroup_map.cpp
ressched/plugins/cgroup_sched_plugin/framework/process_group/src/process_group_util.cpp
```

**输出产物**: `libprocess_group.z.so`

---

### socperf_plugin

**Target**: `//ressched/plugins/socperf_plugin:socperf_plugin`

**类型**: `shared_library`

**主要源码**:
```
ressched/plugins/socperf_plugin/src/socperf_plugin.cpp
```

**输出产物**: `libsocperf_plugin.z.so`

**条件编译**:
```gn
if (ressched_socperf_enable) {
    deps += [ "//foundation/resourceschedule/soc_perf:socperf_client" ]
}
```

---

### frame_aware_plugin

**Target**: `//ressched/plugins/frame_aware_plugin:frame_aware_plugin`

**类型**: `shared_library`

**主要源码**:
```
ressched/plugins/frame_aware_plugin/src/frame_aware_plugin.cpp
ressched/plugins/frame_aware_plugin/src/network_latency_controller.cpp
```

**输出产物**: `libframe_aware_plugin.z.so`

---

### device_standby_plugin

**Target**: `//ressched/plugins/device_standby_plugin:device_standby_plugin`

**类型**: `shared_library`

**主要源码**:
```
ressched/plugins/device_standby_plugin/src/device_standby_plugin.cpp
```

**输出产物**: `libdevice_standby_plugin.z.so`

---

### socperf_executor_plugin

**Target**: `//ressched_executor/plugins/socperf_executor_plugin:socperf_executor_plugin`

**类型**: `shared_library`

**输出产物**: `libsocperf_executor_plugin.z.so`

---

## N-API / JS 接口 Targets

### systemload (JS)

**Target**: `//ressched/interfaces/kits/js/napi/systemload:systemload`

**类型**: `shared_library`

**主要源码**:
```
ressched/interfaces/kits/js/napi/systemload/src/js_systemload.cpp
ressched/interfaces/kits/js/napi/systemload/src/js_systemload_listener.cpp
ressched/interfaces/kits/js/napi/systemload/src/js_systemload_napi_init.cpp
```

**输出产物**: `systemload.so`

**安装路径**: `/system/lib64/module/resourceschedule/`

---

### backgroundprocessmanager_napi

**Target**: `//ressched/interfaces/kits/js/napi/background_process_manager:backgroundprocessmanager_napi`

**类型**: `shared_library`

**主要源码**:
```
ressched/interfaces/kits/js/napi/background_process_manager/src/background_process_manager_napi_init.cpp
```

**输出产物**: `libbackgroundprocessmanager_napi.z.so`

---

### background_process_manager (C)

**Target**: `//ressched/interfaces/kits/c/background_process_manager:background_process_manager`

**类型**: `shared_library`

**主要源码**:
```
ressched/interfaces/kits/c/background_process_manager/src/background_process_manager.cpp
```

**输出产物**: `libbackground_process_manager.z.so`

**安装路径**: `/system/lib64/ndk/`

---

## 配置文件 Targets

### ressched_sa_profile

**Target**: `//ressched/sa_profile:ressched_sa_profile`

**类型**: `sa_profile`

**源文件**: `ressched/sa_profile/1901.json`

**内容**:
```json
{
    "name": 1901,
    "path": "/system/lib64/libresschedsvc.z.so"
}
```

---

### resschedexe_sa_profile

**Target**: `//ressched_executor/sa_profile:resschedexe_sa_profile`

**类型**: `sa_profile`

**源文件**: `ressched_executor/sa_profile/1918.json`

**内容**:
```json
{
    "name": 1918,
    "path": "/system/lib64/libresschedexesvc.z.so"
}
```

---

### 插件配置

**Target**: `//ressched/profile:ressched_plugin_config`

**类型**: `prebuilt_etc`

**源文件**: `ressched/profile/res_sched_config.xml`

**安装路径**: `/system/etc/ressched/`

---

### Init 配置

**Target**: `//ressched/etc/init:resource_schedule_service.cfg`

**类型**: `prebuilt_etc`

**源文件**: `ressched/etc/init/resource_schedule_service.cfg`

**安装路径**: `/system/etc/init/`

**内容**:
```json
{
    "services": [{
        "name": "resource_schedule_service",
        "path": ["/system/bin/sa_main", "/system/lib64/libresschedsvc.z.so"],
        "uid": "system",
        "gid": "system",
        "secon": "u:r:resource_schedule_service:s0"
    }]
}
```

---

## Feature Flags

| Flag | 说明 | 影响目标 |
|------|------|----------|
| `resource_schedule_service_with_ffrt_enable` | FFRT 任务调度 | 所有主要目标 |
| `resource_schedule_service_with_ext_res_enable` | 扩展资源 | resschedsvc |
| `resource_schedule_service_socperf_executor_enable` | SocPerf 执行器 | resschedexesvc |
| `resource_schedule_service_with_app_nap_enable` | AppNap 功能 | resschedsvc |
| `ressched_socperf_enable` | SocPerf 插件 | socperf_plugin |
| `ressched_frame_aware_enable` | 帧感知插件 | frame_aware_plugin |
| `rss_device_standby_enable` | 设备待机插件 | device_standby_plugin |

---

## 依赖关系图

```
resschedsvc (libresschedsvc.z.so)
├── ressched_client (libressched_client.z.so)
│   ├── res_sched_service_stub
│   └── suspend_manager_base_client
├── resschedexe_client (libresschedexeclient.z.so)
├── ressched_common_utils (libressched_common_utils.z.so)
└── suspend_manager_base_service

resschedexesvc (libresschedexesvc.z.so)
├── resschedexe_client
├── res_sched_exe_service_stub
├── libprocess_group (libprocess_group.z.so)
└── ressched_common_utils

Plugins:
├── cgroup_sched → resschedsvc, libprocess_group
├── socperf_plugin → resschedsvc
├── frame_aware_plugin → resschedsvc
└── device_standby_plugin → resschedsvc

NAPI:
├── systemload → ressched_client
├── backgroundprocessmanager_napi → background_process_manager → ressched_client
```

---

## 代码证据

### resschedsvc BUILD.gn 片段

```gn
# 文件: ressched/services/BUILD.gn

ohos_shared_library("resschedsvc") {
    sources = [
        "resschedservice/src/res_sched_service.cpp",
        "resschedservice/src/res_sched_service_ability.cpp",
        "resschedmgr/resschedfwk/src/plugin_mgr.cpp",
        "resschedmgr/resschedfwk/src/res_sched_mgr.cpp",
        # ... 更多源文件
    ]
    
    deps = [
        "//ressched/interfaces/innerkits/ressched_client:ressched_client",
        "//ressched_executor/interfaces/innerkits/ressched_executor_client:resschedexe_client",
        "//ressched/common:ressched_common_utils",
    ]
    
    if (resource_schedule_service_with_ffrt_enable) {
        deps += [ "//foundation/commonlibrary/c_utils:utils" ]
    }
    
    subsystem_name = "resourceschedule"
    part_name = "resource_schedule_service"
}
```

---

## 相关链接

- [编译产物](06_Build_Artifacts.md) - 完整产物清单
- [目录结构](02_Directory_Structure.md) - 代码组织
- [概览](00_Overview.md) - 项目定位
