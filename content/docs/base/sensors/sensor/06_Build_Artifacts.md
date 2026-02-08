# 编译产物与加载关系

> **目的**: 列出 Sensor 子系统的编译产物（.so/.a/.hap/可执行文件）、安装路径和运行时加载关系
> **适用范围**: /base/sensors/sensor（排除 test/ 目录）
> **关键结论**: Sensor 子系统生成 11 个主要产物，其中 SA 库作为 System Ability 3601 运行，N-API 库作为 JS 模块加载
> **相关跳转**: [GN 目标梳理](05_GN_Targets.md) | [常见问题](08_Troubleshooting.md)

---

## 编译产物清单

### 系统库产物

| 产物文件 | 来源 Target | 类型 | 安装路径 | 加载方式 | 说明 |
|----------|------------|------|----------|----------|------|
| `libsensor_service.z.so` | `libsensor_service` (SA) | shared library | system/lib | SA 注册 | Sensor 服务，SA ID 3601 |
| `libsensor_client.z.so` | `libsensor_client` | shared library | system/lib | 动态链接 | Native 客户端库 |
| `sensor_agent.z.so` | `sensor_interface_native` | shared library | system/lib | 动态链接 | 传感器代理 API |
| `libsensor_utils.z.so` | `libsensor_utils` | shared library | system/lib | 动态链接 | 公共工具库 |
| `libsensor_ipc.z.so` | `libsensor_ipc` | shared library | system/lib | 动态链接 | IPC 工具库 |
| `libsensor.z.so` | `libsensor` (NAPI) | shared library | system/lib/module/ | N-API 加载 | JS N-API 绑定 |
| `sensor.so` | `libsensor_ndk` (NDK) | shared library | system/lib | 动态链接 | NDK C API 库 |
| `ohsensor.so` | `ohsensor` | shared library | system/lib/ndk/ | 动态链接 | NDK C API (NDK 版本) |
| `cj_sensor_ffi.z.so` | `cj_sensor_ffi` | shared library | system/lib | 动态链接 | Cangjie FFI 库 |
| `sensor_taihe_native.z.so` | `sensor_taihe_native` | shared library | system/lib | 动态链接 | ArkTS Native 库 |
| `sensor_abc.abc` | `sensor_abc` (bytecode) | Ark bytecode | /system/framework/ | ArkTS 字节码 |

### 构建辅助产物

| 产物文件 | 类型 | 安装路径 | 说明 |
|----------|------|----------|------|
| `3601.json` | SA Profile | system/profile/ | SA 3601 配置文件 |

> **证据**: 各 BUILD.gn 文件中的 `output_name`, `relative_install_dir`, `device_path`

---

## 安装路径映射

### /system/lib/

以下库安装到系统库目录：

| 库文件 | 用途 |
|---------|------|
| `libsensor_service.z.so` | Sensor 服务 (System Ability 3601) |
| `libsensor_client.z.so` | Native 客户端库 |
| `sensor_agent.z.so` | 传感器代理 API |
| `libsensor_utils.z.so` | 公共工具 |
| `libsensor_ipc.z.so` | IPC 工具 |
| `cj_sensor_ffi.z.so` | Cangjie FFI 库 |
| `sensor_taihe_native.z.so` | ArkTS Native 库 |

> **证据**: BUILD.gn 文件中未指定 `install_path` 时的默认安装路径

### /system/lib/module/

| 库文件 | 用途 |
|---------|------|
| `libsensor.z.so` | JS N-API 模块 |

> **证据**: `frameworks/js/napi/BUILD.gn:49` - `relative_install_dir = "module"`

### /system/lib/ndk/

| 库文件 | 用途 |
|---------|------|
| `ohsensor.so` | NDK C API |

> **证据**: `frameworks/native/BUILD.gn:189` - `relative_install_dir = "ndk"`

### /system/lib/

| 库文件 | 用途 |
|---------|------|
| `sensor.so` | NDK C API |

> **证据**: `frameworks/native/BUILD.gn:106` - `output_name = "sensor"`

### /system/framework/

| 库文件 | 用途 |
|---------|------|
| `sensor_abc.abc` | ArkTS 字节码 |

> **证据**: `frameworks/ets/taihe/BUILD.gn:78` - `device_path = "/system/framework/`

### /system/profile/

| 文件 | 用途 |
|------|------|
| `3601.json` | Sensor Service SA 配置 |

> **证据**: `sa_profile/3601.json` + SA 注册机制

---

## 运行时加载关系

### JS 应用加载流程

```
ArkTS/JS 应用
    ↓
import sensor from '@ohos.sensor'
    ↓
加载 libsensor.z.so (N-API 模块)
    ↓
N-API 调用 sensor_agent.z.so
    ↓
加载 libsensor_client.z.so
    ↓
加载 libsensor_ipc.z.so, libsensor_utils.z.so
    ↓
通过 IPC 连接到 libsensor_service.z.so (SA 3601)
```

**加载顺序**:
1. **N-API 模块**: `libsensor.z.so` 被 JS 运行时加载
2. **Native 库链**: `sensor_agent.z.so` → `libsensor_client.z.so` → `libsensor_ipc.z.so`, `libsensor_utils.z.so`
3. **服务连接**: 客户端通过 IPC 连接到 SA 3601

> **证据**: `frameworks/js/napi/BUILD.gn:40`, `frameworks/native/BUILD.gn:77,143`

### Native 应用加载流程

```
Native C++ 应用
    ↓
链接 libsensor_client.z.so
    ↓
加载 libsensor_ipc.z.so, libsensor_utils.z.so
    ↓
通过 IPC 连接到 libsensor_service.z.so (SA 3601)
```

> **证据**: `frameworks/native/BUILD.gn:47-103`

### System Ability 启动流程

```
系统启动 (Init 进程)
    ↓
读取 /system/profile/3601.json
    ↓
启动 "sensors" 进程
    ↓
加载 libsensor_service.z.so
    ↓
SystemAbility::Publish(SENSOR_SERVICE_ABILITY_ID, 3601)
    ↓
SensorService::OnStart()
    ↓
连接到 HDF Sensor 驱动 (drivers_interface_sensor)
```

**启动参数** (from `3601.json`):
- SA ID: 3601
- Process: "sensors"
- Library: `libsensor_service.z.so`
- Run-on-create: true (立即启动)
- Min HDI proxy version: `libsensor_proxy_3.0.z.so`

> **证据**: `sa_profile/3601.json`, `services/src/sensor_service.cpp:51,345`

---

## 库依赖关系详解

### libsensor_service.z.so 依赖

**内部依赖**:
- 无内部依赖（服务端）

**外部依赖**:
- `libaccesstoken_sdk.z.so` - Access Token 权限管理
- `libtokenid_sdk.z.so` - Token ID 管理
- `libhilog.z.so` - 日志
- `libbegetutil.z.so`, `libbeget_proxy.z.so` - Init 进程
- `libsamgr_proxy.z.so` - SA 管理器
- `libipc_single.z.so` - IPC 框架
- `libsystem_ability_fwk.z.so` - SA 框架
- `memmgrclient.z.so` (条件) - 内存管理器
- `libhisysevent.z.so` (条件) - 系统事件
- `libhitrace_meter.z.so` (条件) - 性能追踪
- `libsensor_proxy_3.0.z.so` (条件) - HDI 传感器代理

> **证据**: `services/BUILD.gn:70-84`

### libsensor_client.z.so 依赖

**内部依赖**:
- `libsensor_service_stub.a` (静态链接)

**外部依赖**:
- `libhilog.z.so` - 日志
- `libeventhandler.z.so` - 事件处理器
- `libhicollie.z.so` - 崩溃监控
- `libsamgr_proxy.z.so` - SA 管理器
- `libipc_single.z.so` - IPC 框架
- `libhisysevent.z.so` (条件) - 系统事件
- `libhitrace_meter.z.so` (条件) - 性能追踪

> **证据**: `frameworks/native/BUILD.gn:77-90`

### libsensor.z.so (N-API) 依赖

**内部依赖**:
- `libsensor_agent.z.so` (sensor_interface_native)

**外部依赖**:
- `libace_napi.z.so` - N-API 框架
- `libappexecfwk_base.z.so`, `libappexecfwk_core.z.so` - 应用框架
- `libhilog.z.so` - 日志
- `libipc_single.z.so` - IPC 框架

> **证据**: `frameworks/js/napi/BUILD.gn:40-48`

### libsensor_utils.z.so 依赖

**外部依赖**:
- `libhilog.z.so` - 日志
- `libcesfwk_innerkits.z.so` - 公共事件服务

> **证据**: `services/BUILD.gn:64`, `utils/common/BUILD.gn` (推断)

### libsensor_ipc.z.so 依赖

**外部依赖**:
- `libhilog.z.so` - 日志
- `libsamgr_proxy.z.so` - SA 管理器

> **证据**: `services/BUILD.gn:67`, `utils/ipc/BUILD.gn` (推断)

---

## 产物大小估算

从 `bundle.json` 中获取的ROM 和 RAM 占用：

| 资源类型 | 占用量 | 说明 |
|----------|--------|------|
| ROM | 2048 KB (2 MB) | 传感器子系统的镜像大小 |
| RAM | ~4096 KB (4 MB) | 传感器服务的运行时内存占用 |

> **证据**: `bundle.json:15-16`

**说明**: 这些值是估算值，实际占用取决于：
- 启用的传感器数量
- 采样频率配置
- 订阅的应用数量
- 数据缓存大小

---

## HDI 依赖

**HDF 传感器驱动接口** (条件依赖):

| HDI 库 | 版本 | 用途 |
|---------|------|------|
| `libsensor_proxy_3.0.z.so` | 3.0 | 主传感器驱动代理 |
| `libsensor_convert_proxy_1.0.z.so` | 1.0 | 传感器数据转换代理 |

**加载路径**: 当 HDF 传感器驱动可用时加载
**条件**: `hdf_drivers_interface_sensor` feature flag

> **证据**: `services/BUILD.gn:131-134`

---

## 产物加载时机

### 启动时加载 (Boot-time)

以下库在系统启动时加载：

1. **System Ability**: `libsensor_service.z.so` (SA 3601)
   - 通过 `sensors` 进程启动
   - 配置文件: `3601.json`

2. **SA 配置**: `3601.json`
   - 安装到 `/system/profile/`

> **证据**: `sa_profile/3601.json`, SystemAbility 机制

### 按需加载 (On-demand)

以下库在被应用使用时加载：

1. **JS N-API 模块**: `libsensor.z.so`
   - 当应用执行 `import sensor from '@ohos.sensor'` 时加载
   - 安装路径: `/system/lib/module/libsensor.z.so`

2. **NDK C API 库**: `sensor.so`, `ohsensor.so`
   - 当 Native 应用链接时加载
   - 安装路径: `/system/lib/sensor.so`, `/system/lib/ndk/ohsensor.so`

3. **Native 客户端库**: `libsensor_client.z.so`, `sensor_agent.z.so`
   - 当应用使用 SensorAgent 时加载
   - 安装路径: `/system/lib/`

> **证据**: `frameworks/js/napi/BUILD.gn`, `frameworks/native/BUILD.gn`

---

## 加载失败处理

### 常见加载失败原因

| 失败原因 | 可能原因 | 定位方法 |
|----------|----------|----------|
| SA 未启动 | 进程崩溃、配置错误 | `hilog | grep "SensorService"`
| HDI 驱动未找到 | 驱动未加载、HDF 服务异常 | `hilog | grep "HDI"`
| 依赖库缺失 | 构建配置错误、安装不完整 | `ldd` 检查依赖 |
| 权限配置错误 | bundle.json 配置问题 | 检查 SA 配置 |
| 内存不足 | 设备内存限制 | `free` 命令查看 |

### 日志关键字

| 关键字 | 说明 | 日志域 |
|--------|------|--------|
| "sensorJs" | JS N-API 模块 | 0xD002700 |
| "ISensorServiceIdl" | IDL 接口 | 0xD002700 |
| "SensorService" | Sensor 服务 | 0xD002700 (默认) |

> **证据**: `frameworks/js/napi/BUILD.gn:25-26`, `frameworks/native/BUILD.gn:21`

---

## 相关跳转

- [GN 目标梳理](05_GN_Targets.md) - 查看构建目标和依赖
- [常见问题](08_Troubleshooting.md) - 查看加载失败处理
