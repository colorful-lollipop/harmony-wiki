# 目录结构与模块职责

> **目的**: 描述 Sensor 子系统的目录结构和各模块的职责
> **适用范围**: /base/sensors/sensor（排除 test/ 目录）
> **关键结论**: Sensor 子系统采用分层架构，包含 interface、framework、service、utils 四个主要层
> **相关跳转**: [项目概览](00_Overview.md) | [架构说明](02_Architecture.md)

---

## 目录结构概览

```
/base/sensors/sensor/
├── frameworks/              # 框架层 - 客户端实现
│   ├── native/           # Native C++ 客户端
│   ├── js/napi/          # JavaScript N-API 绑定
│   ├── cj/                # Cangjie 语言 FFI 绑定
│   └── ets/taihe/         # ArkTS/ETS Taihe 绑定
├── interfaces/            # 接口层 - API 定义
│   ├── inner_api/         # 内部 API
│   └── kits/c/           # 公共 C API (NDK)
├── services/             # 服务层 - 服务端实现
│   ├── src/               # 服务源代码
│   ├── include/           # 服务头文件
│   └── hdi_connection/   # HDI 硬件连接
├── utils/                # 工具层 - 公共组件
│   ├── common/            # 公共工具
│   └── ipc/               # IPC 工具
├── sa_profile/           # System Ability 配置
├── vibration_convert/    # 音频转震动模块
├── figures/              # 架构图等资源
└── wiki/                # Wiki 文档（本目录）
```

> **证据**: `ls -la`, `find` 命令输出

---

## Interface 层 (`/interfaces/`)

### 职责

定义供上层应用和系统组件使用的 API 接口，包括：
- 内部 API（供系统组件使用）
- 公共 C API (NDK，供应用开发者使用）

### 目录结构

```
interfaces/
├── inner_api/           # 内部 API (供系统组件使用)
│   ├── sensor_agent.h              # 主传感器代理接口
│   └── sensor_agent_type.h         # 传感器类型定义
│
└── kits/c/              # 公共 C API (NDK)
    ├── oh_sensor.h               # C API 函数声明
    └── oh_sensor_type.h          # C API 类型定义
```

### 关键文件说明

#### `interfaces/inner_api/sensor_agent.h`

**目的**: 定义 Native 客户端的核心 API 接口

**主要内容**:
- `SensorAgent` 类 - 传感器代理接口
- 订阅/取消订阅方法
- 数据回调机制

**关键函数** (待补充详细签名):

```cpp
class SensorAgent {
public:
    // 订阅传感器
    int32_t SubscribeSensor(uint32_t sensorTypeId, const SensorUserCallback &callback);

    // 取消订阅
    int32_t UnsubscribeSensor(uint32_t sensorTypeId);
};
```

> **证据**: `interfaces/inner_api/sensor_agent.h`

#### `interfaces/inner_api/sensor_agent_type.h`

**目的**: 定义传感器相关的数据结构和枚举

**主要内容**:
- 传感器类型枚举 (SENSOR_TYPE_ID_*)
- 传感器数据结构 (SensorData)
- 传感器信息结构 (SensorInfo)

**关键类型**:

| 类型 | 说明 |
|------|------|
| `SensorUserCallback` | 用户回调接口 |
| `SensorData` | 传感器数据事件 |
| `SensorInfo` | 传感器信息 |

> **证据**: `interfaces/inner_api/sensor_agent_type.h`

#### `interfaces/kits/c/oh_sensor.h`

**目的**: 定义 NDK C API

**主要内容**:
- 公开的 C 函数接口
- 与 Native 实现的绑定

> **证据**: `interfaces/kits/c/oh_sensor.h`

---

## Framework 层 (`/frameworks/`)

### 职责

实现传感器客户端框架，包括：
- Native C++ 客户端实现
- JavaScript N-API 绑定
- Cangjie (CJ) 语言 FFI 绑定
- ArkTS/ETS Taihe 绑定

### 目录结构

```
frameworks/
├── native/              # Native C++ 客户端
│   ├── include/           # 头文件
│   └── src/               # 实现文件
│
├── js/napi/             # JavaScript N-API 绑定
│   ├── include/           # 头文件
│   └── src/               # 实现文件
│
├── cj/                  # Cangjie FFI 绑定
│   ├── include/           # 头文件
│   └── src/               # 实现文件
│
└── ets/taihe/           # ArkTS/ETS Taihe 绑定
    ├── idl/               # IDL 定义
    └── author/src/         # 实现文件
```

### Native 模块 (`frameworks/native/`)

#### 职责

实现 Native C++ 客户端，包括：
- IPC 客户端
- 传感器代理
- 数据通道管理
- 事件处理

#### 关键文件

| 文件 | 职责 |
|------|------|
| `sensor_agent.cpp` | 传感器代理实现 |
| `sensor_agent_proxy.cpp` | Agent 代理实现 |
| `sensor_client_stub.cpp` | 客户端 Stub 实现 |
| `sensor_service_client.cpp` | SensorService 客户端 |
| `sensor_data_channel.cpp` | 数据通道管理 |
| `sensor_event_handler.cpp` | 事件处理器 |
| `geomagnetic_field.cpp` | 地磁场计算算法 |
| `sensor_algorithm.cpp` | 传感器算法 |

**头文件**:
- `sensor_agent_proxy.h` - 传感器代理
- `sensor_client_proxy.h` - 客户端代理
- `sensor_service_client.h` - 服务客户端
- `i_sensor_client.h` - 客户端接口定义

> **证据**: `frameworks/native/src/`, `frameworks/native/include/`

#### IDL 接口

**文件**: `frameworks/native/ISensorService.idl`

**目的**: 定义 Sensor 服务的 IPC 接口

> **证据**: `frameworks/native/ISensorService.idl`

### N-API 模块 (`frameworks/js/napi/`)

#### 职责

实现 JavaScript (N-API) 绑定，将 JS API 映射到 Native 实现。

#### 关键文件

| 文件 | 职责 |
|------|------|
| `sensor_js.cpp` | 主 JS API 实现，导出 46 个方法 |
| `sensor_napi_utils.cpp` | N-API 工具函数 |
| `sensor_napi_error.cpp` | 错误处理 |
| `sensor_system_js.cpp` | 系统传感器实现 |

> **证据**: `frameworks/js/napi/src/`, `frameworks/js/napi/include/`

### Taihe 模块 (`frameworks/ets/taihe/`)

#### 职责

实现 ArkTS/ETS (Taihe) 语言绑定，新一代 TypeScript API。

#### 关键文件

| 文件 | 职责 |
|------|------|
| `ohos.sensor.taihe` | IDL 定义 |
| `ohos.sensor.impl.cpp` | Taihe 实现 |
| `ani_constructor.cpp` | ANI 构造器 |

> **证据**: `frameworks/ets/taihe/`

### CJ 模块 (`frameworks/cj/`)

#### 职责

实现 Cangjie (CJ) 语言 FFI 绑定。

#### 关键文件

| 文件 | 职责 |
|------|------|
| `cj_sensor_ffi.cpp` | FFI 实现 |
| `cj_sensor_impl.cpp` | 核心实现 |

> **证据**: `frameworks/cj/src/`, `frameworks/cj/include/`

---

## Service 层 (`/services/`)

### 职责

实现 Sensor 服务端，包括：
- 传感器数据管理
- 客户端连接管理
- HDI 硬件连接
- 权限验证
- 数据处理和分发

### 目录结构

```
services/
├── src/                    # 服务源代码
├── include/                # 服务头文件
└── hdi_connection/          # HDI 硬件连接
    ├── interface/           # HDI 接口定义
    ├── adapter/            # HDI 适配器
    └── hardware/           # 硬件服务实现（工程构建）
```

### 关键模块

| 模块 | 职责 |
|------|------|
| `SensorService` | 主服务类，继承自 SystemAbility |
| `SensorManager` | 传感器管理器 |
| `SensorDataManager` | 数据管理器 |
| `SensorDataProcesser` | 数据处理器 |
| `ClientInfo` | 客户端信息管理 |
| `SensorObserver` | 观察者模式实现 |
| `SensorPowerPolicy` | 电源策略管理 |
| `StreamServer` | 流服务器 |
| `FifoCacheData` | FIFO 缓存 |

> **证据**: `services/src/`, `services/include/`

### HDI 连接 (`services/hdi_connection/`)

#### 职责

连接到 HDF 传感器驱动接口，实现硬件抽象层。

#### 关键文件

| 文件 | 职责 |
|------|------|
| `sensor_hdi_connection.cpp` | HDI 连接实现 |
| `hdi_connection.cpp` | HDI 适配器 |
| `compatible_connection.cpp` | 兼容连接 |
| `sensor_event_callback.cpp` | 传感器事件回调 |
| `sensor_plug_callback.cpp` | 传感器插拔回调 |

> **证据**: `services/hdi_connection/`

---

## Utils 层 (`/utils/`)

### 职责

提供跨模块使用的公共组件，包括：
- 公共工具函数
- IPC 通信工具
- 权限工具
- 日志工具

### 目录结构

```
utils/
├── common/              # 公共工具
│   ├── include/           # 头文件
│   └── src/               # 实现文件
│
└── ipc/                 # IPC 工具
    ├── include/           # 头文件
    └── src/               # 实现文件
```

### Common 模块 (`utils/common/`)

#### 关键组件

| 组件 | 职责 |
|------|------|
| `Sensor` | 基础传感器类 |
| `PermissionUtil` | 权限验证工具 |
| `SensorBasicDataChannel` | 数据通道基类 |
| `SensorDataEvent` | 数据事件结构 |
| `SensorLog` | 日志工具 |
| `SensorXcollie` | 崩溃监控 |
| `ActiveInfo` | 活动信息 |

> **证据**: `utils/common/src/`, `utils/common/include/`

### IPC 模块 (`utils/ipc/`)

#### 关键组件

| 组件 | 职责 |
|------|------|
| `StreamSocket` | Socket 流 |
| `StreamSession` | 会话管理 |
| `StreamBuffer` | 缓冲区管理 |
| `CircleStreamBuffer` | 循环缓冲区 |
| `NetPacket` | 网络数据包 |

> **证据**: `utils/ipc/src/`, `utils/ipc/include/`

---

## SA Profile (`/sa_profile/`)

### 职责

配置 Sensor System Ability 的注册和启动参数。

### 关键文件

| 文件 | 说明 |
|------|------|
| `3601.json` | SA 配置文件 (SA ID: 3601) |
| `BUILD.gn` | SA profile 构建配置 |

### SA 配置 (3601.json)

```json
{
    "process": "sensors",
    "systemability": [
        {
            "name": 3601,
            "libpath": "libsensor_service.z.so",
            "run-on-create": true,
            "distributed": false,
            "dump_level": 1,
            "min_hdi_proxy_version": ["libsensor_proxy_3.0.z.so"]
        }
    ]
}
```

> **证据**: `sa_profile/3601.json`, `sa_profile/BUILD.gn`

---

## 音频转震动模块 (`/vibration_convert/`)

### 职责

提供音频转震动功能，将音频文件转换为震动模式。

### 目录结构

```
vibration_convert/
├── core/
│   ├── native/               # 核心 Native 实现
│   ├── algorithm/            # 转换算法
│   │   ├── conversion/       # FFT/Filter/MFCC
│   │   ├── intensity_processor/  # 强度处理
│   │   └── peak_finder/      # 峰值检测
│   └── utils/               # 工具函数
│
└── interfaces/js/              # JS API
    ├── include/               # 头文件
    └── src/                   # 实现文件
```

### 关键组件

| 组件 | 职责 |
|------|------|
| `VibrationConvertCore` | 核心转换逻辑 |
| `ConversionFilter` | 滤波器 |
| `ConversionFFT` | FFT 变换 |
| `ConversionMFCC` | MFCC 特征提取 |
| `IntensityProcessor` | 强度处理 |
| `PeakFinder` | 峰值检测 |

> **证据**: `vibration_convert/core/native/src/`, `vibration_convert/core/algorithm/`

---

## 模块依赖关系

### 接口层依赖

- **无外部依赖** - 接口层只定义 API，不依赖其他模块

### Framework 层依赖

- **Interface 层** - 使用 `inner_api` 和 `kits/c` 定义
- **Utils 层** - 使用 `libsensor_utils` 和 `libsensor_ipc`
- **N-API** - 使用 `napi` 库
- **HDF** - 使用 `drivers_interface_sensor`

### Service 层依赖

- **Interface 层** - 使用 `inner_api` 定义
- **Utils 层** - 使用 `libsensor_utils` 和 `libsensor_ipc`
- **HDF** - 使用 `drivers_interface_sensor`
- **IPC** - 使用 `ipc_single`
- **SAMGR** - 使用 `samgr_proxy`
- **Access Token** - 使用 `libaccesstoken_sdk`

### Utils 层依赖

- **无跨层依赖** - Utils 层是基础工具库，被其他层依赖

> **证据**: 各 BUILD.gn 文件中的 `deps` 和 `external_deps`

---

## 目录排除说明

### 不包含的目录

- **test/****: 所有测试代码，包括单元测试、模糊测试
- **.git/**: Git 版本控制目录
- **wiki/_work/**: Wiki 工作文档（非最终文档）

> **证据**: 文件系统结构

---

## 相关跳转

- [项目概览](00_Overview.md) - 了解整体架构
- [对外 N-API 参考](03_NAPI_Reference.md) - 学习 JS API
- [内部模块接口](04_Internal_API.md) - 查看内部 API
- [架构说明](02_Architecture.md) - 理解组件交互和数据流
