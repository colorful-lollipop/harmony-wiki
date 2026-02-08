# 目录结构与模块职责

## 目的

本文档介绍 Medical_Sensor 项目的目录结构、各模块职责和关键文件，帮助开发者快速定位代码位置。

---

## 适用范围

本文档适用于需要：
- 理解代码组织结构
- 定位特定功能的实现位置
- 了解模块间依赖关系

---

## 顶层目录结构

```
/base/sensors/medical_sensor/
├── frameworks/              # 框架代码（客户端）
│   └── native/medical_sensor/
├── interfaces/              # 对外接口
│   ├── native/            # Native C 接口
│   └── plugin/            # JS API (N-API)
├── sa_profile/            # 系统能力配置
├── services/              # 服务代码（服务端）
│   └── medical_sensor/
└── utils/                 # 公共工具
```

---

## 详细目录说明

### frameworks/native/medical_sensor/ - 客户端框架

**职责**：提供客户端与 Medical_Sensor 服务通信的框架层，包括 IPC Proxy、数据通道管理、事件处理。

**关键文件**：

| 文件 | 职责 |
|------|------|
| `include/i_medical_sensor_service.h` | 服务接口定义（IRemoteBroker） |
| `include/i_medical_sensor_client.h` | 客户端接口定义 |
| `include/medical_sensor_service_proxy.h` | Proxy 类头文件 |
| `include/medical_sensor_service_client.h` | 服务客户端封装（单例） |
| `include/medical_sensor_data_channel.h` | 数据通道类 |
| `src/medical_service_proxy.cpp` | Proxy 实现（客户端 IPC 调用） |
| `src/medical_service_client.cpp` | 服务客户端管理（生命周期管理） |

**核心类**：

- **MedicalSensorServiceProxy**：实现 IPC 客户端代理，与服务端通信
- **MedicalSensorServiceClient**：单例客户端，管理 Proxy 生命周期和重连

---

### interfaces/native/ - Native C 接口

**职责**：提供 Native C API（NDK），供 C/C++ 应用直接调用。

**目录结构**：

```
interfaces/native/
├── include/               # 公共头文件
│   ├── medical_native_type.h      # 传感器数据类型定义
│   └── medical_native_impl.h      # Native API 函数声明
├── src/                    # Native API 实现
│   └── medical_native_impl.cpp  # 订阅/取消订阅等实现
└── BUILD.gn               # 构建配置
```

**关键文件**：

| 文件 | 职责 |
|------|------|
| `include/medical_native_type.h` | 传感器数据类型（SensorEvent、MedicalSensorInfo） |
| `include/medical_native_impl.h` | Native API 函数声明 |
| `src/medical_native_impl.cpp` | 订阅、取消订阅、批处理等实现 |

**导出的 API**（`medical_native_impl.cpp`）：

| 函数名 | 说明 |
|--------|------|
| `GetAllSensors()` | 获取所有传感器列表 |
| `ActivateSensor()` | 启用传感器 |
| `DeactivateSensor()` | 禁用传感器 |
| `SetBatch()` | 设置批处理参数 |
| `SubscribeSensor()` | 订阅传感器数据 |
| `UnsubscribeSensor()` | 取消订阅传感器数据 |
| `SetMode()` | 设置传感器模式 |
| `SetOption()` | 设置传感器选项 |

---

### interfaces/plugin/ - JS API (N-API)

**职责**：提供 JS API，通过 N-API 将 JS 调用桥接到 Native 实现。

**目录结构**：

```
interfaces/plugin/
├── include/                    # N-API 头文件
│   ├── medical_js.h               # JS 模块头文件
│   └── medical_napi_utils.h       # N-API 工具函数
├── src/                        # N-API 实现
│   ├── medical_js.cpp             # 主 N-API 实现（注册、on/off/setOpt）
│   └── medical_napi_utils.cpp     # 异步回调、类型转换工具
└── BUILD.gn                    # 构建配置
```

**关键文件**：

| 文件 | 职责 |
|------|------|
| `src/medical_js.cpp` | N-API 模块注册、JS 方法实现 |
| `src/medical_napi_utils.cpp` | N-API 工具函数（异步工作、类型转换） |
| `include/medical_napi_utils.h` | AsyncCallbackInfo 结构体定义 |

**导出的 JS 方法**：

| JS 方法 | 说明 | C++ 实现 |
|---------|------|-----------|
| `on(type, callback, options?)` | 订阅传感器数据 | `On()` |
| `off(type, callback?)` | 取消订阅 | `Off()` |
| `setOpt(type, option)` | 设置选项 | `SetOpt()` |

**证据**：
- 模块注册：`interfaces/plugin/src/medical_js.cpp:292-295`
- 方法导出：`interfaces/plugin/src/medical_js.cpp:271-275`

---

### services/medical_sensor/ - 服务端

**职责**：实现 Medical_Sensor 系统能力，处理客户端请求、管理传感器状态、与 HDI 通信。

**目录结构**：

```
services/medical_sensor/
├── include/                    # 服务头文件
│   ├── medical_sensor_service.h      # MedicalSensorService 主服务类
│   ├── medical_sensor_service_stub.h # IPC Stub 基类
│   ├── client_info.h               # 客户端信息管理
│   └── ...
├── src/                        # 服务实现
│   ├── medical_service.cpp          # 主服务实现
│   ├── medical_service_stub.cpp     # IPC Stub 分发
│   ├── medical_manager.cpp          # 传感器管理器
│   ├── client_info.cpp            # 客户端信息管理
│   └── ...
├── hdi_connection/             # HDI 适配层
│   ├── interface/                  # HDI 接口定义
│   ├── adapter/                   # HDI 适配器实现
│   └── hardware/                  # 硬件服务实现
└── BUILD.gn                    # 构建配置
```

**关键文件**：

| 文件 | 职责 |
|------|------|
| `src/medical_service.cpp` | SystemAbility 主实现（OnStart、OnStop） |
| `src/medical_service_stub.cpp` | IPC 请求分发 |
| `src/medical_manager.cpp` | 传感器状态管理 |
| `src/client_info.cpp` | 客户端信息存储（PID、UID、Token） |
| `hdi_connection/interface/include/sensor_hdi_connection.h` | HDI 适配层接口 |
| `hdi_connection/adapter/src/hdi_connection.cpp` | HDI 适配器实现 |

**核心类**：

- **MedicalSensorService**：SystemAbility 主服务类，继承 `SystemAbility` 和 `MedicalSensorServiceStub`
- **MedicalSensorServiceStub**：IPC Stub 基类，分发 IPC 请求
- **MedicalSensorManager**：传感器管理器，管理传感器状态和订阅关系
- **SensorHdiConnection**：HDI 适配层门面类

---

### sa_profile/ - 系统能力配置

**职责**：配置 System Ability 的元信息（进程名、动态库、启动参数）。

**关键文件**：

| 文件 | 职责 |
|------|------|
| `3605.xml` | Medical_Sensor 的 SA 配置 |
| `BUILD.gn` | 构建配置（安装 SA 配置） |

**SA 配置内容**（`3605.xml`）：

```xml
<info>
    <process>sensors</process>                    <!-- 服务进程名 -->
    <systemability>
        <name>3605</name>                       <!-- SA ID -->
        <libpath>libmedical_service.z.so</libpath>  <!-- 动态库 -->
        <run-on-create>true</run-on-create>          <!-- 创建即启动 -->
        <distributed>false</distributed>              <!-- 不分布式 -->
        <dump-level>1</dump-level>                <!-- Dump 级别 -->
    </systemability>
</info>
```

**证据**：
- SA 配置：`sa_profile/3605.xml:15-24`

---

### utils/ - 公共工具

**职责**：提供跨模块使用的公共工具，包括权限检查、日志、数据缓存等。

**目录结构**：

```
utils/
├── include/                    # 公共头文件
│   ├── permission_util.h        # 权限检查工具
│   ├── medical_sensor.h        # MedicalSensor 类定义
│   ├── medical_errors.h        # 错误码定义
│   ├── medical_log_domain.h     # 日志域定义
│   └── ...
└── src/                        # 工具实现
    ├── permission_util.cpp      # 权限检查实现
    ├── dmd_report.cpp          # DMD（设备管理调试）上报
    ├── report_data_cache.cpp   # 数据缓存
    └── ...
```

**关键文件**：

| 文件 | 职责 |
|------|------|
| `src/permission_util.cpp` | 权限检查（AccessTokenKit 集成） |
| `include/permission_util.h` | 权限检查工具类声明 |
| `include/medical_sensor.h` | MedicalSensor 类（Parcelable） |
| `include/medical_errors.h` | 错误码定义 |

**关键工具类**：

- **PermissionUtil**：权限检查工具，使用 `AccessTokenKit` 验证权限
- **MedicalSensor**：传感器信息类，用于 IPC 传输
- **DmdReport**：设备管理调试上报工具

---

## 模块依赖关系

```
┌─────────────────────────────────────────────────────────────────┐
│                    应用层 (JS/C++)                        │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│              interfaces/plugin/ (N-API)                      │
│                   (medical.so)                             │
│                                                             │
│  ┌──────────────────────┴───────────────────────────────┐   │
│  │      interfaces/native/ (Native API)                    │   │
│  │           (medical_agent.so)                            │   │
│  └──────────────────────┬───────────────────────────────┘   │
│                         ▼                                     │
│  ┌──────────────────────────────────────────────────────┐     │
│  │  frameworks/native/ (Client Framework)              │     │
│  │         (libmedical_native.so)                         │     │
│  │                                                       │     │
│  └──────────────────────┬───────────────────────────────┘     │
│                         ▼                                      │
│  ┌──────────────────────────────────────────────────────┐     │
│  │    services/medical_sensor/ (Service)               │     │
│  │          (libmedical_service.so)                         │     │
│  │                                                        │     │
│  │  ┌──────────────────────┴──────────────────────────┐   │     │
│  │  │     hdi_connection/ (HDI Adapter)          │   │     │
│  │  │                                             │   │     │
│  │  └──────────────────────┬──────────────────────────┘   │     │
│  │                         ▼                             │     │
│  │              传感器驱动 (HDI)                         │     │
│  └──────────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────────┘
```

---

## 模块职责边界

| 模块 | 职责 | 不负责 |
|------|------|--------|
| **interfaces/plugin/** | JS API 桥接、参数校验 | 传感器数据处理 |
| **interfaces/native/** | Native API、数据通道管理 | 权限检查 |
| **frameworks/native/** | IPC 代理、客户端生命周期 | 传感器状态管理 |
| **services/medical_sensor/** | 系统能力、传感器管理、权限检查 | 硬件驱动实现 |
| **utils/** | 公共工具（权限、日志、数据缓存） | 模块间通信 |

---

## 关键文件索引

| 功能 | 文件路径 |
|------|----------|
| N-API 注册 | `interfaces/plugin/src/medical_js.cpp:292-295` |
| IPC 接口定义 | `frameworks/native/medical_sensor/include/i_medical_sensor_service.h` |
| Service Ability | `services/medical_sensor/src/medical_service.cpp:51` |
| 权限检查 | `utils/src/permission_util.cpp:40-49` |
| HDI 适配 | `services/medical_sensor/hdi_connection/interface/include/sensor_hdi_connection.h` |
| SA 配置 | `sa_profile/3605.xml` |

---

## 相关跳转

- [架构说明](02_Architecture.md) - 详细的架构分析和时序图
- [N-API 参考](03_N-API_Reference.md) - 完整的 JS API 文档
- [内部 API](04_Internal_API.md) - 模块接口和依赖关系
- [GN Targets](05_GN_Targets.md) - 构建目标和依赖图
