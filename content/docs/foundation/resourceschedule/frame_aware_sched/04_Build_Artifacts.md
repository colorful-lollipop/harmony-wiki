# 构建产物与 GN Targets

## GN 构建入口

- **根 BUILD.gn**: `BUILD.gn`
- **模块配置**: `frameaware.gni`
- **构建入口**: `interfaces/innerkits/frameintf/BUILD.gn`

## Targets 清单

### 1. frame_trace_intf

| 属性 | 值 |
|------|-----|
| **类型** | `ohos_shared_library` |
| **sources** | `frame_trace.cpp` |
| **cflags** | `-fstack-protector-strong` |
| **subsystem** | `resourceschedule` |
| **part** | `frame_aware_sched` |
| **innerapi_tags** | `"platformsdk_indirect"` |
| **产物** | `libframe_trace_intf.z.so` |

### 2. frame_ui_intf

| 属性 | 值 |
|------|-----|
| **类型** | `ohos_shared_library` |
| **sources** | `frame_ui_intf.cpp`, `rtg_interface.cpp` + frame_aware_collector 源码 |
| **cflags** | `-Wno-shift-negative-value`, `-fstack-protector-strong` |
| **external_deps** | `c_utils:utils`, `hilog:libhilog`, `hitrace:hitrace_meter` |
| **subsystem** | `resourceschedule` |
| **part** | `frame_aware_sched` |
| **innerapi_tags** | `"platformsdk"` |
| **产物** | `libframe_ui_intf.z.so` |

**sources 详细**:
```
frame_ui_intf.cpp
rtg_interface.cpp
../../frameworks/core/frame_aware_collector/src/frame_msg_mgr.cpp
../../frameworks/core/frame_aware_collector/src/frame_window_mgr.cpp
../../frameworks/core/frame_aware_collector/src/rme_core_sched.cpp
../../frameworks/core/frame_aware_collector/src/rme_scene_sched.cpp
```

### 3. frame_msg_intf

| 属性 | 值 |
|------|-----|
| **类型** | `ohos_shared_library` |
| **sources** | `frame_msg_intf.cpp`, `rtg_interface.cpp` + frame_aware_policy 源码 + qos_common 源码 |
| **cflags** | `-Wno-shift-negative-value`, `-fstack-protector-strong` |
| **external_deps** | `c_utils:utils`, `eventhandler:libeventhandler`, `ffrt:libffrt`, `hilog:libhilog`, `hitrace:hitrace_meter`, `libxml2:libxml2` |
| **subsystem** | `resourceschedule` |
| **part** | `frame_aware_sched` |
| **产物** | `libframe_msg_intf.z.so` |

**sources 详细**:
```
frame_msg_intf.cpp
rtg_interface.cpp
../../frameworks/core/frame_aware_policy/src/app_info.cpp
../../frameworks/core/frame_aware_policy/src/intellisense_server.cpp
../../frameworks/core/frame_aware_policy/src/para_config.cpp
../../qos_manager/src/qos_common.cpp
```

### 4. rtg_interface

| 属性 | 值 |
|------|-----|
| **类型** | `ohos_shared_library` |
| **sources** | `rtg_interface.cpp` |
| **cflags** | `-Wno-shift-negative-value`, `-fstack-protector-strong` |
| **external_deps** | `bounds_checking_function:libsec_shared`, `hilog:libhilog` |
| **subsystem** | `resourceschedule` |
| **part** | `frame_aware_sched` |
| **install_enable** | `true` |
| **产物** | `librtg_interface.z.so` |

### 5. frame_aware_sched_config

| 属性 | 值 |
|------|-----|
| **类型** | profile |
| **sources** | `hwrme.xml` |
| **subsystem** | `resourceschedule` |
| **part** | `frame_aware_sched` |
| **产物** | `frame_aware_sched_config` (安装到 `/etc/` 或配置目录) |

## 产物清单

| 产物名 | 类型 | 用途 |
|--------|------|------|
| `libframe_trace_intf.z.so` | 共享库 | 帧追踪接口 |
| `libframe_ui_intf.z.so` | 共享库 | UI 帧信息接口 |
| `libframe_msg_intf.z.so` | 共享库 | 帧消息接口 |
| `librtg_interface.z.so` | 共享库 | RTG 控制接口 |
| `frame_aware_sched_config` | 配置文件 | RME 智能感知配置 |

## 运行时加载关系

```
应用进程
    ├── libframe_ui_intf.z.so ──> 加载 ──> frame_aware_collector 模块
    └── libframe_trace_intf.z.so ──> 加载 ──> 帧追踪模块

系统服务
    ├── libframe_msg_intf.z.so ──> 加载 ──> frame_aware_policy 模块
    │       ├── libffrt.z.so (FFRT 任务队列)
    │       ├── libxml2.so (XML 解析)
    │       └── libhilog.so (日志)
    └── librtg_interface.z.so ──> 加载 ──> RTG 控制接口
            ├── libsec_shared.so (安全函数)
            └── /proc/sched_rtg_ctrl (内核节点)
```

## 配置产物安装

`frame_aware_sched_config` (hwrme.xml) 安装路径：
- 默认安装到系统配置目录，如 `/etc/` 或 `/system/etc/`

## 依赖关系图

```mermaid
graph TD
    subgraph "libframe_ui_intf.z.so"
        A1[frame_ui_intf.cpp]
        A2[frame_aware_collector]
        A3[rtg_interface.cpp]
    end

    subgraph "libframe_msg_intf.z.so"
        B1[frame_msg_intf.cpp]
        B2[frame_aware_policy]
        B3[qos_manager]
        B4[rtg_interface.cpp]
    end

    A3 -->|ioctl| K1[/proc/sched_rtg_ctrl]
    B4 -->|ioctl| K1
    B3 -->|ioctl| K2[/dev/basic_auth_ctrl]

    A2 -->|depends| C[c_utils]
    A2 -->|depends| D[hilog]
    B2 -->|depends| E[ffrt]
    B2 -->|depends| F[libxml2]
