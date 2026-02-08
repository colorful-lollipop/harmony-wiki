# 03 目录结构与代码地图

**文档目的**: 帮助读者快速定位代码文件，建立项目空间认知  
**适用范围**: 新人学习者、代码审查者  

---

## 3.1 顶层目录结构

```
foundation/deviceprofile/device_info_manager/
│
├── bundle.json                    # 组件配置清单 [核心]
├── deviceprofile.gni              # GN 构建变量定义
├── hisysevent.yaml                # 埋点事件配置
├── OAT.xml                        # 开源合规检查配置
├── LICENSE                        # Apache 2.0 许可证
│
├── sa_profile/                    # SA 声明配置
├── permission/                    # 接口权限配置
├── etc/                           # 配置文件和初始化脚本
├── figures/                       # 架构图资源
├── radar/                         # 性能监控埋点
│
├── common/                        # 公共模块 [核心]
├── interfaces/                    # IPC 接口定义 [核心]
└── services/                      # 服务实现 [核心]
```

---

## 3.2 目录详细导航

### 3.2.1 sa_profile/ - SA 配置

| 文件 | 说明 | 关键配置 |
|------|------|----------|
| `6001.json` | SA 6001 配置 | 进程名、库路径、启动条件 |
| `BUILD.gn` | 构建配置 | - |

**证据**: `sa_profile/6001.json`

### 3.2.2 permission/ - 权限配置

| 文件 | 说明 | 关键配置 |
|------|------|----------|
| `permission.json` | 接口权限映射 | 接口名 → 允许调用者列表 |
| `BUILD.gn` | 构建配置 | - |

**证据**: `permission/permission.json`

### 3.2.3 common/ - 公共模块

```
common/
├── BUILD.gn
├── include/
│   ├── constants/                 # 常量定义
│   │   ├── distributed_device_profile_constants.h
│   │   └── distributed_device_profile_errors.h      [错误码定义]
│   │
│   ├── interfaces/                # 数据结构接口
│   │   ├── i_distributed_device_profile.h          [IPC接口定义]
│   │   ├── dp_ipc_interface_code.h                 [IPC命令码]
│   │   ├── device_profile.h
│   │   ├── service_profile.h
│   │   ├── characteristic_profile.h
│   │   ├── access_control_profile.h
│   │   ├── trust_device_profile.h
│   │   ├── service_info_profile_new.h
│   │   ├── local_service_info.h
│   │   ├── dp_subscribe_info.h
│   │   ├── dp_sync_options.h
│   │   └── ... (更多Profile定义)
│   │
│   └── utils/                     # 工具类
│       ├── distributed_device_profile_log.h
│       ├── profile_utils.h
│       ├── ipc_utils.h
│       └── single_instance.h
│
└── src/
    └── utils/
        └── profile_utils.cpp
```

**核心文件**:
- `common/include/constants/distributed_device_profile_errors.h:21` - 错误码定义
- `common/include/interfaces/i_distributed_device_profile.h` - IPC 接口定义
- `common/include/interfaces/dp_ipc_interface_code.h` - IPC 命令码

### 3.2.4 interfaces/innerkits/core/ - IPC 客户端

```
interfaces/innerkits/core/
├── BUILD.gn                       # SDK 构建配置
├── include/
│   ├── distributed_device_profile_client.h:46     [客户端入口]
│   ├── distributed_device_profile_proxy.h         [IPC代理]
│   └── i_distributed_device_profile.h            [接口定义]
│
└── src/
    ├── distributed_device_profile_client.cpp
    └── distributed_device_profile_proxy.cpp
```

**核心文件**:
- `interfaces/innerkits/core/include/distributed_device_profile_client.h:46` - 客户端单例
- `interfaces/innerkits/core/include/distributed_device_profile_proxy.h` - IPC 代理

### 3.2.5 services/core/ - 服务实现

```
services/core/
├── BUILD.gn                       # 服务构建配置 [安全编译选项]
│
├── include/
│   ├── distributed_device_profile_service_new.h:42    [SA主入口]
│   ├── distributed_device_profile_stub_new.h          [IPC Stub]
│   │
│   ├── deviceprofilemanager/
│   │   └── device_profile_manager.h                   [Profile管理]
│   ├── trustprofilemanager/
│   │   └── trust_profile_manager.h                    [可信设备管理]
│   ├── subscribeprofilemanager/
│   │   └── subscribe_profile_manager.h                [订阅管理]
│   ├── profiledatamanager/
│   │   └── profile_data_manager.h                     [数据持久化]
│   ├── contentsensormanager/
│   │   └── content_sensor_manager.h                   [内容采集]
│   ├── permissionmanager/
│   │   └── permission_manager.h                       [权限管理]
│   └── multiusermanager/
│       └── multi_user_manager.h                       [多用户支持]
│
└── src/
    ├── distributed_device_profile_service_new.cpp     [服务实现]
    ├── distributed_device_profile_stub_new.cpp        [IPC分发]
    │
    ├── deviceprofilemanager/
    │   └── device_profile_manager.cpp
    ├── trustprofilemanager/
    │   └── trust_profile_manager.cpp
    ├── subscribeprofilemanager/
    │   └── subscribe_profile_manager.cpp
    ├── profiledatamanager/
    │   ├── profile_data_manager.cpp
    │   ├── kvadapter/                                 [KV Store适配]
    │   └── rdbadapter/                                [RDB适配]
    ├── contentsensormanager/
    │   └── content_sensor_manager.cpp
    ├── permissionmanager/
    │   └── permission_manager.cpp:218                 [权限检查实现]
    └── multiusermanager/
        └── multi_user_manager.cpp
```

**核心文件**:
- `services/core/include/distributed_device_profile_service_new.h:42` - SA 主类
- `services/core/src/permissionmanager/permission_manager.cpp:218` - 权限检查
- `services/core/src/distributed_device_profile_stub_new.cpp` - IPC 请求分发

---

## 3.3 代码导航图

### 3.3.1 功能 → 代码位置映射

| 功能 | 入口位置 | 实现位置 |
|------|----------|----------|
| **获取 Profile** | `distributed_device_profile_client.h:57` | `services/core/src/distributed_device_profile_service_new.cpp:728` |
| **插入 Profile** | `distributed_device_profile_client.h:56` | `services/core/src/distributed_device_profile_service_new.cpp:518` |
| **同步 Profile** | `distributed_device_profile_client.h:62` | `services/core/src/distributed_device_profile_service_new.cpp:1258` |
| **订阅变更** | `distributed_device_profile_client.h:64` | `services/core/src/distributed_device_profile_service_new.cpp:876` |
| **权限检查** | `permission_manager.h` | `services/core/src/permissionmanager/permission_manager.cpp:218` |
| **数据查询** | `profile_data_manager.h` | `services/core/src/profiledatamanager/` |

### 3.3.2 IPC 调用链

```
调用者
  ↓ (调用)
interfaces/innerkits/core/include/distributed_device_profile_client.h:46
  ↓ (SendRequest)
interfaces/innerkits/core/include/distributed_device_profile_proxy.h
  ↓ (Binder IPC)
services/core/include/distributed_device_profile_stub_new.h
  ↓ (OnRemoteRequest)
services/core/include/distributed_device_profile_service_new.h:42
  ↓ (权限检查)
services/core/include/permissionmanager/permission_manager.h
  ↓ (数据操作)
services/core/include/profiledatamanager/profile_data_manager.h
```

### 3.3.3 数据结构定义

| 数据结构 | 头文件位置 |
|----------|------------|
| DeviceProfile | `common/include/interfaces/device_profile.h` |
| ServiceProfile | `common/include/interfaces/service_profile.h` |
| CharacteristicProfile | `common/include/interfaces/characteristic_profile.h` |
| AccessControlProfile | `common/include/interfaces/access_control_profile.h` |
| TrustDeviceProfile | `common/include/interfaces/trust_device_profile.h` |
| ServiceInfoProfile | `common/include/interfaces/service_info_profile_new.h` |
| LocalServiceInfo | `common/include/interfaces/local_service_info.h` |
| SubscribeInfo | `common/include/interfaces/dp_subscribe_info.h` |
| SyncOptions | `common/include/interfaces/dp_sync_options.h` |

---

## 3.4 关键代码文件清单

### 必看文件 (新人)

| 优先级 | 文件 | 说明 |
|--------|------|------|
| ⭐⭐⭐ | `interfaces/innerkits/core/include/distributed_device_profile_client.h` | 客户端 API |
| ⭐⭐⭐ | `services/core/include/distributed_device_profile_service_new.h` | 服务入口 |
| ⭐⭐⭐ | `common/include/interfaces/dp_ipc_interface_code.h` | IPC 命令码 |
| ⭐⭐ | `services/core/include/permissionmanager/permission_manager.h` | 权限管理 |
| ⭐⭐ | `permission/permission.json` | 权限配置 |
| ⭐⭐ | `common/include/constants/distributed_device_profile_errors.h` | 错误码 |
| ⭐ | `common/include/interfaces/*.h` | 数据结构定义 |

### 必看文件 (安全研究员)

| 优先级 | 文件 | 说明 |
|--------|------|------|
| ⭐⭐⭐ | `services/core/src/distributed_device_profile_service_new.cpp` | IPC 处理入口 |
| ⭐⭐⭐ | `services/core/src/permissionmanager/permission_manager.cpp` | 权限检查实现 |
| ⭐⭐⭐ | `permission/permission.json` | 接口权限映射 |
| ⭐⭐ | `services/core/src/distributed_device_profile_stub_new.cpp` | IPC 请求分发 |
| ⭐⭐ | `common/include/interfaces/dp_ipc_interface_code.h` | IPC 命令码 |
| ⭐ | `services/core/include/profiledatamanager/*.h` | 数据持久化 |
| ⭐ | `sa_profile/6001.json` | SA 配置 |

---

## 3.5 配置文件清单

| 配置文件 | 用途 | 影响范围 |
|----------|------|----------|
| `bundle.json` | 组件清单、依赖、构建目标 | 整体项目 |
| `deviceprofile.gni` | GN 变量定义 | 构建系统 |
| `sa_profile/6001.json` | SA 启动配置 | 服务生命周期 |
| `permission/permission.json` | 接口权限映射 | 访问控制 |
| `hisysevent.yaml` | 埋点事件定义 | 监控日志 |
| `etc/profile/BUILD.gn` | Profile 配置文件 | 初始数据 |

---

## 3.6 快速定位指南

### 如何找到...

| 想找的内容 | 方法 |
|------------|------|
| **IPC 接口定义** | 搜索 `i_distributed_device_profile.h` |
| **客户端 API** | 查看 `distributed_device_profile_client.h:50-114` |
| **错误码定义** | 查看 `distributed_device_profile_errors.h:21-219` |
| **权限检查逻辑** | 查看 `permission_manager.cpp:218-243` |
| **IPC 命令码** | 查看 `dp_ipc_interface_code.h` |
| **SA 配置** | 查看 `sa_profile/6001.json` |
| **构建目标** | 查看 `bundle.json:55-68` 和各目录 `BUILD.gn` |

---

## 3.7 相关章节

| 目标 | 推荐阅读 |
|------|----------|
| 架构设计 | [02_Architecture.md](02_Architecture.md) |
| API 详情 | [04_Interface.md](04_Interface.md) |
| 构建配置 | [07_Build.md](07_Build.md) |
| 安全分析 | [05_AttackSurface.md](05_AttackSurface.md) |
