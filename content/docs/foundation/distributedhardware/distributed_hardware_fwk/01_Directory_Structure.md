# 目录结构与模块职责

本文档描述分布式硬件管理框架的目录结构和各模块职责。

> **适用范围**: 所有需要了解模块边界和代码位置的开发者

---

## 顶层目录结构

```
foundation/distributedhardware/distributed_hardware_fwk/
├── application/                    # [应用层] DHardware_UI 系统应用
├── av_transport/                  # [传输层] AV 音视频传输
│   ├── av_trans_control_center/   # 控制中心
│   ├── av_trans_engine/           # 传输引擎
│   │   ├── av_sender/            # 发送端
│   │   ├── av_receiver/          # 接收端
│   │   ├── filters/              # 滤镜模块
│   │   ├── framework/             # 框架
│   │   └── plugin/                # 插件
│   ├── common/                    # 公共组件
│   ├── interface/                 # 接口
│   └── av_trans_handler/          # 处理器
├── common/                        # [公共层] 公共接口
│   ├── log/                       # 日志
│   └── utils/                     # 工具类
├── interfaces/                    # [接口层]
│   ├── inner_kits/               # Inner Kit SDK
│   └── kits/napi/                 # N-API JavaScript 绑定
├── sa_profile/                    # [配置] SA 配置
├── services/                      # [服务层] 核心服务
│   └── distributedhardwarefwkservice/  # SA 服务实现
├── taihe/                         # [绑定] Taihe/ANI 现代化绑定
├── utils/                         # [工具] 工具库
├── figures/                       # 架构图
├── bundle.json                     # 模块配置
├── distributedhardwarefwk.gni    # GN 配置
├── hisysevent.yaml                 # HiSysEvent 配置
└── README_zh.md                    # 项目说明
```

---

## application/ - 应用层

### 目录结构

```
application/
├── AppScope/              # 应用作用域配置
│   └── app.json5          # 应用元数据
├── entry/                 # 模块入口
│   ├── src/main/
│   │   ├── module.json5   # 模块配置（含权限声明）
│   │   └── ...
│   └── ...
├── hvigor/                # 构建配置
├── signature/             # 签名文件
│   └── DHardware_UI.p7b   # 签名证书
├── BUILD.gn               # 构建配置
└── DHardware_UI.gni       # 签名配置
```

### 模块职责

| 组件 | 职责 |
|------|------|
| **DHardware_UI** | 系统应用，提供硬件管理用户界面 |
| **权限声明** | 声明 `ACCESS_DISTRIBUTED_HARDWARE` 等权限 |

**证据**: `bundle.json:159-164`
```json
"requestPermissions": [
  { "name": "ohos.permission.ACCESS_DISTRIBUTED_HARDWARE" },
  { "name": "ohos.permission.MANAGE_SECURE_SETTINGS" },
  ...
]
```

---

## av_transport/ - AV 传输层

### 目录结构

```
av_transport/
├── av_trans_control_center/     # 控制中心
│   ├── inner_kits/              # 内部接口
│   └── services/                # 控制服务
├── av_trans_engine/             # 传输引擎
│   ├── av_sender/              # 发送引擎
│   │   └── test/              # 测试
│   ├── av_receiver/            # 接收引擎
│   │   └── test/              # 测试
│   ├── filters/                # 滤镜
│   │   ├── av_trans_coder_filter/     # 编解码滤镜
│   │   ├── av_trans_input_filter/     # 输入滤镜
│   │   └── av_trans_output_filter/    # 输出滤镜
│   ├── framework/              # 框架
│   │   └── pipeline/           # 流水线
│   └── plugin/                 # 插件
│       ├── plugins/            # 插件实现
│       │   ├── av_trans_input/       # 输入插件
│       │   └── av_trans_output/      # 输出插件
│       └── test/               # 测试
├── common/                     # 公共组件
│   ├── include/               # 头文件
│   └── src/                   # 源文件
├── interface/                  # 接口
│   └── ...
└── av_trans_handler/          # 传输处理器
    └── histreamer_ability_querier/  # HiStreamer 能力查询
```

### 模块职责

| 模块 | 职责 |
|------|------|
| **av_sender** | 音视频数据发送引擎 |
| **av_receiver** | 音视频数据接收引擎 |
| **filters** | 输入/输出滤镜处理 |
| **framework** | 传输框架和流水线 |
| **control_center** | 会话控制和传输管理 |

**证据**: `bundle.json:70-73`
```json
"sub_component": [
  "//foundation/.../av_transport/av_trans_engine/av_receiver:distributed_av_receiver",
  "//foundation/.../av_transport/av_trans_engine/av_sender:distributed_av_sender",
  "//foundation/.../av_transport/framework:distributed_av_pipeline_fwk",
  ...
]
```

---

## common/ - 公共层

### 目录结构

```
common/
├── log/                    # 日志
│   └── include/
│       └── distributed_hardware_log.h  # 日志宏定义
└── utils/                  # 工具类
    └── include/
        ├── constants.h
        ├── device_type.h
        ├── distributed_hardware_errno.h
        ├── dhfwk_single_instance.h
        └── ...
```

### 模块职责

| 组件 | 职责 |
|------|------|
| **日志** | 提供 `DHLOGI`, `DHLOGE` 等日志宏 |
| **常量** | 定义错误码、设备类型等常量 |
| **单例** | 单例模式基类 |

---

## interfaces/ - 接口层

### 目录结构

```
interfaces/
├── inner_kits/               # Inner Kit SDK（供其他子系统使用）
│   ├── include/             # 头文件
│   │   ├── distributed_hardware_fwk_kit.h
│   │   ├── distributed_hardware_fwk_kit_paras.h
│   │   ├── idistributed_hardware_manager.h
│   │   ├── idistributed_hardware_sink.h
│   │   ├── idistributed_hardware_source.h
│   │   └── ipc/            # IPC 头文件
│   │       ├── distributed_hardware_proxy.h
│   │       ├── distributed_hardware_stub.h
│   │       └── ...
│   ├── src/ipc/            # IPC 实现
│   │   ├── distributed_hardware_proxy.cpp
│   │   ├── distributed_hardware_stub.cpp
│   │   └── ...
│   └── test/               # 测试
└── kits/napi/              # N-API JavaScript 绑定
    ├── include/
    │   └── native_distributedhardwarefwk_js.h
    └── src/
        └── native_distributedhardwarefwk_js.cpp
```

### 模块职责

| 模块 | 职责 | 调用方 |
|------|------|--------|
| **Inner Kit SDK** | 提供 C++ 接口供其他子系统调用 | 分布式相机、屏幕等 |
| **N-API** | 提供 JS 接口供应用调用 | 上层应用 |

**证据**: `bundle.json:82-93`
```json
"inner_kits": [
  {
    "type": "so",
    "name": "//foundation/.../interfaces/inner_kits:libdhfwk_sdk",
    "header": {
      "header_files": ["distributed_hardware_fwk_kit.h", ...],
      "header_base": "//foundation/.../interfaces/inner_kits/include"
    }
  }
]
```

---

## services/ - 服务层

### 目录结构

```
services/distributedhardwarefwkservice/
├── include/                     # 头文件
│   ├── distributed_hardware_service.h
│   ├── distributed_hardware_stub.h
│   ├── distributed_hardware_manager.h
│   ├── accessmanager/          # 硬件接入管理
│   ├── componentmanager/       # 部件管理
│   ├── componentloader/        # 部件加载
│   ├── resourcemanager/        # 资源管理
│   ├── localhardwaremanager/   # 本地硬件管理
│   ├── versionmanager/         # 版本管理
│   ├── task/                   # 任务调度
│   ├── transport/              # 传输
│   ├── ipc/                    # IPC
│   ├── publisher/              # 发布订阅
│   ├── hidumphelper/           # Dump 工具
│   ├── lowlatency/             # 低延迟
│   └── utils/                  # 工具
├── src/                        # 源文件
│   ├── distributed_hardware_service.cpp
│   ├── distributed_hardware_stub.cpp
│   ├── distributed_hardware_manager.cpp
│   ├── distributed_hardware_manager_factory.cpp
│   ├── accessmanager/
│   ├── componentmanager/
│   │   ├── component_manager.cpp
│   │   ├── component_enable.cpp
│   │   ├── component_disable.cpp
│   │   └── ...
│   ├── componentloader/
│   ├── resourcemanager/
│   ├── localhardwaremanager/
│   ├── versionmanager/
│   ├── task/
│   │   ├── task.cpp
│   │   ├── task_factory.cpp
│   │   ├── task_executor.cpp
│   │   ├── online_task.cpp
│   │   ├── offline_task.cpp
│   │   ├── enable_task.cpp
│   │   └── disable_task.cpp
│   ├── transport/
│   │   ├── dh_transport.cpp
│   │   └── dh_comm_tool.cpp
│   ├── ipc/
│   ├── publisher/
│   ├── hidumphelper/
│   ├── lowlatency/
│   └── utils/
└── test/                       # 测试
```

### 模块职责

| 模块 | 职责 |
|------|------|
| **DistributedHardwareService** | 核心 SA 服务 (SA ID: 4801) |
| **AccessManager** | 设备上下线监听和响应 |
| **ResourceManager** | 硬件资源存储和管理 |
| **ComponentManager** | 分布式硬件部件生命周期管理 |
| **ComponentLoader** | 部件驱动动态加载 |
| **LocalHardwareManager** | 本地硬件感知 |
| **VersionManager** | 版本兼容性管理 |
| **Task** | 任务调度和执行 |

---

## sa_profile/ - SA 配置

### 目录结构

```
sa_profile/
├── close_source/              # 闭源配置
│   └── dhardware.cfg
├── dhardware.cfg              # 默认配置
├── dhfwk_sa_profile.gni
└── BUILD.gn
```

### SA 配置内容

**证据**: `sa_profile/dhardware.cfg`
```json
{
  "services": [{
    "name": "dhardware",
    "path": ["/system/bin/sa_main", "/system/profile/dhardware.json"],
    "uid": "dhardware",
    "gid": ["dhardware", "input"],
    "ondemand": true,
    "apl": "system_basic",
    "permission": [
      "ohos.permission.DISTRIBUTED_DATASYNC",
      "ohos.permission.CAMERA",
      "ohos.permission.ACCESS_SERVICE_DM",
      "ohos.permission.ACCESS_DISTRIBUTED_HARDWARE",
      ...
    ]
  }]
}
```

---

## taihe/ - Taihe 绑定

### 目录结构

```
taihe/
├── idl/                      # IDL 定义
│   └── ohos.distributedHardware.hardwareManager.idl
├── src/                      # 实现
│   └── ohos.distributedHardware.hardwareManager.impl.cpp
└── BUILD.gn
```

### 模块职责

| 组件 | 职责 |
|------|------|
| **Taihe** | ANI (ArkNative Interface) 现代化绑定 |
| **ABC** | 静态字节码输出 |

---

## utils/ - 工具库

### 目录结构

```
utils/
├── include/                  # 头文件
│   ├── anonymous_string.h
│   ├── dh_utils_hisysevent.h
│   ├── dh_utils_hitrace.h
│   ├── dh_utils_tool.h
│   ├── histreamer_ability_parser.h
│   ├── histreamer_query_tool.h
│   └── ...
└── src/                      # 源文件
    └── ...
```

### 模块职责

| 组件 | 职责 |
|------|------|
| **工具类** | 提供匿名化、日志、追踪等工具函数 |
| **HiStreamer** | HiStreamer 能力解析和查询 |

---

## 关键依赖关系

```
                    ┌─────────────────────────────────────┐
                    │     DHardware_UI (应用层)            │
                    └─────────────────┬───────────────────┘
                                      │
                    ┌─────────────────▼───────────────────┐
                    │        hardwaremanager (N-API)        │
                    └─────────────────┬───────────────────┘
                                      │
                    ┌─────────────────▼───────────────────┐
                    │     libdhfwk_sdk (Inner Kit)          │
                    └─────────────────┬───────────────────┘
                                      │
                    ┌─────────────────▼───────────────────┐
                    │  DistributedHardwareService (SA)     │
                    │         (SA ID: 4801)                │
                    └─────────────────────────────────────┘
                                      │
         ┌────────────────────────────┼────────────────────────────┐
         │                            │                            │
         ▼                            ▼                            ▼
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│   DeviceManager │      │   KV Store      │      │   SoftBus       │
└─────────────────┘      └─────────────────┘      └─────────────────┘
```

---

## 后续文档

- 系统架构详解 → [02_Architecture.md](02_Architecture.md)
- N-API 接口参考 → [03_NAPI_Reference.md](03_NAPI_Reference.md)
- GN 构建配置 → [05_GN_Build.md](05_GN_Build.md)
