# 目录结构

## 目的

本文档说明 Resource Schedule Service 仓库的目录组织结构，帮助开发者快速定位代码。

## 适用范围

- 新加入的开发者
- 需要修改特定功能的开发者

---

## 顶层目录

```
resource_schedule_service/
├── ressched/                    # 资源调度主服务
│   ├── common/                  # 公共工具库
│   ├── interfaces/              # 接口层
│   ├── plugins/                 # 插件目录
│   ├── sched_controller/        # 调度控制器
│   ├── scene_recognize/         # 场景识别
│   ├── services/                # 服务端实现
│   ├── sa_profile/              # SA 配置
│   ├── profile/                 # 插件配置
│   └── etc/init/                # 初始化配置
├── ressched_executor/           # 资源调度执行器服务
│   ├── interfaces/
│   ├── services/
│   ├── plugins/
│   └── sa_profile/
├── figures/                     # 架构图
├── wiki/                        # 本文档
├── README.md                    # 项目说明
└── bundle.json                  # 组件配置
```

---

## ressched 详解

### common/ - 公共工具库

```
ressched/common/
├── include/                     # 头文件
│   ├── res_sched_file_util.h    # 文件操作工具
│   ├── res_sched_json_util.h    # JSON 处理
│   ├── res_sched_string_util.h  # 字符串工具
│   ├── res_sched_time_util.h    # 时间工具
│   ├── oobe_manager.h           # OOBE 管理
│   └── ...
└── src/                         # 实现
    ├── res_sched_file_util.cpp
    └── ...
```

**职责**: 提供文件、JSON、字符串、时间等通用工具函数。

---

### interfaces/ - 接口层

```
ressched/interfaces/
├── innerkits/                   # 内部客户端库
│   ├── ressched_client/         # RSS 客户端
│   │   ├── include/
│   │   │   ├── res_sched_client.h      # 客户端主类
│   │   │   ├── res_type.h              # 资源类型定义
│   │   │   ├── res_sched_errors.h      # 错误码
│   │   │   └── res_sched_ipc_interface_code.h  # IPC 接口码
│   │   ├── src/
│   │   │   └── res_sched_client.cpp
│   │   └── IResSchedService.idl        # IDL 接口定义
│   └── suspend_manager_base_client/    # 挂起管理客户端
│
└── kits/                        # 对外接口
    ├── js/napi/                 # JS/N-API 接口
    │   ├── systemload/          # 系统负载 API
    │   │   ├── include/
    │   │   ├── src/
    │   │   └── BUILD.gn
    │   └── background_process_manager/  # 后台进程管理 API
    │       ├── include/
    │       ├── src/
    │       └── BUILD.gn
    ├── ets/taihe/               # ArkTS/Taihe 接口
    │   ├── systemload/
    │   └── background_process_manager/
    └── c/                       # C 接口
        └── background_process_manager/
```

**关键文件**:
- `res_type.h` - 定义所有资源类型 (200+ 种)
- `res_sched_client.h` - C++ 客户端接口
- `res_sched_ipc_interface_code.h` - IPC 接口码定义

---

### plugins/ - 插件目录

```
ressched/plugins/
├── cgroup_sched_plugin/         # Cgroup 调度插件
│   ├── common/
│   ├── framework/
│   │   ├── process_group/       # 进程组管理
│   │   │   ├── include/
│   │   │   │   ├── sched_policy.h
│   │   │   │   ├── cgroup_action.h
│   │   │   │   └── cgroup_controller.h
│   │   │   └── src/
│   │   ├── sched_controller/    # 调度控制器
│   │   │   ├── include/
│   │   │   │   ├── sched_controller.h
│   │   │   │   ├── cgroup_adjuster.h
│   │   │   │   └── supervisor.h
│   │   │   └── src/
│   │   └── utils/
│   └── profiles/                # Cgroup 配置文件
│
├── socperf_plugin/              # SoC 性能插件
│   ├── include/
│   │   └── socperf_plugin.h
│   └── src/
│       └── socperf_plugin.cpp
│
├── frame_aware_plugin/          # 帧感知插件
│   ├── include/
│   │   ├── frame_aware_plugin.h
│   │   └── latency_control/
│   └── src/
│       ├── frame_aware_plugin.cpp
│       └── network_latency_controller.cpp
│
└── device_standby_plugin/       # 设备待机插件
    ├── include/
    └── src/
```

**插件说明**:

| 插件 | 功能 | 输出产物 |
|------|------|----------|
| cgroup_sched_plugin | 进程 Cgroup 分组调度 | libcgroup_sched.z.so, libprocess_group.z.so |
| socperf_plugin | CPU/GPU/DDR 频率调节 | libsocperf_plugin.z.so |
| frame_aware_plugin | 帧率优化、网络延迟控制 | libframe_aware_plugin.z.so |
| device_standby_plugin | 待机状态管理 | libdevice_standby_plugin.z.so |

---

### sched_controller/ - 调度控制器

```
ressched/sched_controller/
├── observer/                    # 状态观察者
│   ├── include/
│   │   ├── app_state_observer.h         # 应用状态监听
│   │   ├── window_state_observer.h      # 窗口状态监听
│   │   ├── audio_observer.h             # 音频事件监听
│   │   ├── camera_observer.h            # 相机事件监听
│   │   ├── sched_telephony_observer.h   # 电话状态监听
│   │   ├── mmi_observer.h               # 多模输入监听
│   │   ├── background_task_observer.h   # 后台任务监听
│   │   ├── av_session_state_listener.h  # AV 会话监听
│   │   └── observer_manager.h           # 观察者管理器
│   └── src/
│
└── common_event/                # 公共事件处理
    ├── include/
    │   └── event_controller.h
    └── src/
        └── event_controller.cpp
```

**职责**: 监听系统各类事件（应用、窗口、音频、相机等），转换为资源调度事件。

---

### scene_recognize/ - 场景识别

```
ressched/scene_recognize/
├── include/
│   ├── scene_recognizer_base.h                    # 场景识别基类
│   ├── scene_recognizer_mgr.h                     # 场景识别管理器
│   ├── slide_recognizer.h                         # 滑动识别
│   ├── app_startup_scene_rec.h                    # 应用启动场景识别
│   └── ...
└── src/
    ├── scene_recognizer_mgr.cpp
    ├── slide_recognizer.cpp
    └── ...
```

**职责**: 识别用户交互场景（滑动、点击、应用启动等），用于触发相应调度策略。

---

### services/ - 服务端实现

```
ressched/services/
├── resschedmgr/                 # 资源调度管理器
│   ├── pluginbase/              # 插件基础定义
│   │   └── include/
│   │       ├── plugin.h         # 插件接口定义
│   │       ├── res_data.h       # 资源数据结构
│   │       └── config_info.h    # 配置信息结构
│   └── resschedfwk/             # 调度框架
│       └── src/
│           ├── plugin_mgr.cpp   # 插件管理器
│           ├── res_sched_mgr.cpp # 资源调度管理器
│           └── ...
│
└── resschedservice/             # 资源调度服务
    ├── include/
    │   ├── res_sched_service.h           # 服务主类
    │   ├── res_sched_service_ability.h   # SA 能力类
    │   └── res_sched_service_stub.h      # IPC Stub
    └── src/
        ├── res_sched_service.cpp
        ├── res_sched_service_ability.cpp
        └── main.cpp              # 进程入口
```

**关键组件**:

| 组件 | 文件 | 职责 |
|------|------|------|
| ResSchedService | res_sched_service.h/cpp | IPC 服务实现，处理客户端请求 |
| ResSchedServiceAbility | res_sched_service_ability.h/cpp | SystemAbility 生命周期管理 |
| PluginMgr | plugin_mgr.cpp | 插件加载、事件分发 |
| ResSchedMgr | res_sched_mgr.cpp | 资源调度管理，场景识别协调 |

---

### sa_profile/ - SA 配置

```
ressched/sa_profile/
├── 1901.json                    # ressched 服务 SA 配置
└── BUILD.gn
```

**1901.json**:
```json
{
    "name": 1901,
    "path": "/system/lib64/libresschedsvc.z.so"
}
```

---

### profile/ - 插件配置

```
ressched/profile/
├── res_sched_config.xml         # 主配置文件
├── res_sched_plugin_switch.xml  # 插件开关配置
└── BUILD.gn
```

---

### etc/init/ - 初始化配置

```
ressched/etc/init/
├── resource_schedule_service.cfg    # 服务启动配置
└── BUILD.gn
```

**resource_schedule_service.cfg**:
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

## ressched_executor 详解

```
ressched_executor/
├── interfaces/
│   └── innerkits/ressched_executor_client/
│       ├── include/
│       │   ├── res_sched_exe_client.h
│       │   └── res_sched_exe_constants.h
│       ├── src/
│       │   └── res_sched_exe_client.cpp
│       └── IResSchedExeService.idl
│
├── services/
│   ├── resschedexemgr/
│   │   └── src/
│   │       ├── res_sched_exe_mgr.cpp
│   │       └── config_reader.cpp
│   └── resschedexeservice/
│       ├── include/
│       │   └── res_sched_exe_service.h
│       ├── src/
│       │   └── res_sched_exe_service.cpp
│       └── main.cpp
│
├── plugins/socperf_executor_plugin/
│   ├── include/
│   └── src/
│
└── sa_profile/
    └── 1918.json              # ressched_executor SA 配置
```

---

## 代码路径速查表

### 事件类型定义

```
ressched/interfaces/innerkits/ressched_client/include/res_type.h
```

### 错误码定义

```
ressched/interfaces/innerkits/ressched_client/include/res_sched_errors.h
```

### IPC 接口码

```
ressched/interfaces/innerkits/ressched_client/include/res_sched_ipc_interface_code.h
```

### 插件接口定义

```
ressched/services/resschedmgr/pluginbase/include/plugin.h
ressched/services/resschedmgr/pluginbase/include/res_data.h
```

### N-API 入口

```
ressched/interfaces/kits/js/napi/systemload/src/js_systemload_napi_init.cpp
ressched/interfaces/kits/js/napi/background_process_manager/src/background_process_manager_napi_init.cpp
```

### 服务启动入口

```
ressched/services/resschedservice/src/main.cpp
ressched_executor/services/resschedexeservice/src/main.cpp
```

### 配置文件

```
ressched/profile/res_sched_config.xml
ressched/profile/res_sched_plugin_switch.xml
ressched/plugins/cgroup_sched_plugin/profiles/cgroup_action_config.json
```

---

## 相关链接

- [概览](00_Overview.md) - 项目定位
- [架构设计](01_Architecture.md) - 组件图与数据流
- [GN 构建](05_GN_Targets.md) - 构建目标详情
