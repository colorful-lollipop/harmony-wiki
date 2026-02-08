# 目录结构与代码地图

> **目的**：快速定位核心代码位置，理解目录职责和模块组织
> **适用范围**：USB Manager v3.1.0
> **更新时间**：2026-02-07

---

## 顶层目录职责

### 目录结构总览

```
base/usb/usb_manager/
├── etc/                          # 配置文件
├── figures/                      # 文档图片
├── frameworks/                    # 框架层（UI 和 ArkTS 代码）
├── interfaces/                   # 接口层（N-API 和 IPC）
├── sa_profile/                  # System Ability 配置
├── services/                    # 服务层（C++ 实现）
├── test/                        # 测试用例（本文档不包含）
├── utils/                       # 工具库
├── wiki/                        # Wiki 文档
├── bundle.json                  # 组件配置
├── hisysevent.yaml             # HiSysEvent 配置
├── usbmgr.gni                  # GN 变量定义
└── README.md                   # 项目说明
```

**证据来源**：`ls -la`, `find` 命令扫描结果

---

## 详细目录结构（排除测试）

### 1. etc/ - 配置文件目录

**职责**：存储系统配置和参数文件

```
etc/
├── param/                       # 系统参数配置
│   └── [系统参数配置文件]
└── BUILD.gn                     # 配置文件构建规则
```

**关键文件**：
- `etc/BUILD.gn` - 配置文件构建规则，定义 `usb_etc_files` target
- `etc/param/` - 系统参数（具体文件需查看实际内容）

**证据**：`etc/BUILD.gn`

---

### 2. figures/ - 文档图片目录

**职责**：存储架构图、流程图等图片资源

```
figures/
├── usb-manager-architecture.png     # USB 服务架构图
└── usb-manager-architecture_zh.png  # USB 服务架构图（中文）
```

**用途**：README.md 和文档中使用

**证据**：`README.md:13` 引用了架构图

---

### 3. frameworks/ - 框架层

**职责**：提供 UI 界面和 ArkTS 绑定代码

```
frameworks/
├── dialog/                      # 对话框框架
│   └── dialog_ui/
│       └── usb_right_dialog/     # USB 权限对话框
│           └── [ArkTS UI 代码]
└── ets/                         # ArkTS 代码
    └── taihe/
        └── usb_manager/
            └── [ArkTS 绑定代码]
```

**子模块**：
- **dialog_ui/usb_right_dialog** - USB 权限请求对话框 UI
- **ets/taihe/usb_manager** - ArkTS 绑定代码

**产物**：
- `usb_right_dialog.hap` - 权限对话框 HAP 包

**证据**：`frameworks/ets/taihe/BUILD.gn`, `frameworks/dialog/dialog_ui/usb_right_dialog/BUILD.gn`

---

### 4. interfaces/ - 接口层（最核心）

**职责**：定义和实现 N-API 接口、IPC 接口、内部 C++ API

#### 4.1 innerkits/ - 内部 C++ API

**职责**：提供内部 C++ 接口，用于应用内直接调用 USB Manager 功能

```
interfaces/innerkits/
├── IUsbServer.idl                # IPC 接口定义（95 个方法）
├── UsbServerTypes.idl             # IPC 类型定义
├── native/
│   ├── include/                   # 内部头文件
│   │   ├── usb_srv_client.h       # 客户端 API 头文件
│   │   ├── usb_interface_type.h   # USB 接口类型定义
│   │   └── iusb_srv.h           # USB 服务接口头文件
│   └── src/                      # 内部实现
│       ├── usb_srv_client.cpp     # 客户端实现
│       ├── usb_device_pipe.cpp    # USB 设备管道实现
│       ├── usb_interface_type.cpp # USB 接口类型实现
│       ├── usb_request.cpp       # USB 请求实现
│       ├── usbd_bulk_callback.cpp       # Host 模式 Bulk 回调
│       ├── usbd_callback_server.cpp     # Host 模式回调服务器
│       └── usbd_callback_stub.cpp     # Host 模式回调存根
└── BUILD.gn                      # 内部库构建配置
```

**关键组件**：
- **IUsbServer.idl** - IPC 接口定义，95 个方法，分为 Host/Device/Port/Serial 四大模块
- **usb_srv_client.cpp** - IPC 客户端，封装 Binder 调用
- **usbd_callback_server.cpp** - Host 模式回调服务器，处理设备插拔通知

**产物**：
- `libusbsrv_client.z.so` - 内部客户端库
- `usb_server_stub` - IPC Stub 代码

**GN Targets**：
```gn
ohos_shared_library("usbsrv_client")      # 内部客户端库
ohos_source_set("usb_server_stub")       # IPC Stub 代码
```

**证据**：`interfaces/innerkits/BUILD.gn`

---

#### 4.2 kits/ - 外部 API（N-API）

**职责**：提供 JS API，通过 N-API 暴露 USB 功能到应用层

```
interfaces/kits/js/
└── napi/                           # N-API 接口
    ├── include/                      # N-API 头文件
    │   ├── usb_async_context.h        # 异步上下文结构
    │   ├── usb_param_map.h           # 参数映射表
    │   └── ...
    └── src/                         # N-API 实现
        ├── usb_middle.cpp            # USB 模块入口（动态加载）
        ├── usbmanager_middle.cpp     # USB Manager 模块入口
        ├── serial_middle.cpp         # Serial 模块入口
        ├── usb_info.cpp             # USB 功能实现（Host/Device/Port）
        ├── serial_info.cpp         # Serial 功能实现
        └── napi_util.cpp          # N-API 工具函数
    └── BUILD.gn                   # N-API 模块构建配置
```

**关键模块**：

##### 1. usb_middle.cpp - USB 模块入口

**职责**：动态加载 `libusbmanager.z.so`，提供延迟加载机制

**关键代码**：
```cpp
// usb_middle.cpp:17-24
static napi_module g_module = {
    .nm_version = 1,
    .nm_filename = "usb",
    .nm_register_func = nullptr,  // 动态加载
    .nm_modname = "usb",
};
```

**加载方式**：`dlopen("libusbmanager.z.so")` + `dlsym("UsbInit")`

**产物**：`libusb.z.so`

---

##### 2. usbmanager_middle.cpp - USB Manager 模块入口

**职责**：注册 USB Manager 模块，提供核心 JS API

**关键代码**：
```cpp
// usbmanager_middle.cpp:115-122
static napi_module g_moduleManager = {
    .nm_version = 1,
    .nm_filename = "usbManager",
    .nm_register_func = UsbInit,
    .nm_modname = "usbManager",
};
```

**产物**：`libusbmanager.z.so`

---

##### 3. serial_middle.cpp - Serial 模块入口

**职责**：注册 Serial 模块，提供串口 JS API

**关键代码**：
```cpp
// serial_middle.cpp:20-35
static napi_module g_module = {
    .nm_version = 1,
    .nm_filename = "serial",
    .nm_register_func = SerialInit,
    .nm_modname = "serial",
};
```

**产物**：`libserial.z.so`

---

##### 4. usb_info.cpp - USB 功能实现

**职责**：实现 Host/Device/Port 三大模块的所有 JS API

**主要函数**（Host 功能）：
- `CoreGetDevices` - 获取设备列表
- `CoreConnectDevice` - 连接设备
- `CoreHasRight` - 检查权限
- `CoreRequestRight` - 请求权限
- `PipeClaimInterface` - 声明接口
- `PipeReleaseInterface` - 释放接口
- `PipeBulkTransfer` - 批量传输
- `PipeControlTransfer` - 控制传输
- `PipeGetFileDescriptor` - 获取文件描述符

**主要函数**（Device 功能）：
- `CoreSetCurrentFunctions` - 设置 USB 功能
- `CoreGetCurrentFunctions` - 获取当前 USB 功能
- `CoreUsbFunctionsFromString` - 字符串转功能掩码
- `CoreUsbFunctionsToString` - 功能掩码转字符串

**主要函数**（Port 功能）：
- `CoreGetPorts` - 获取端口列表
- `PortGetSupportedModes` - 获取支持模式
- `PortSetPortRole` - 设置端口角色

**主要函数**（Accessory 功能）：
- `DeviceGetAccessoryList` - 获取配件列表
- `DeviceOpenAccessory` - 打开配件
- `DeviceRequestAccessoryRight` - 请求配件权限

**代码行数**：约 2900 行

**证据**：`interfaces/kits/js/napi/src/usb_info.cpp`

---

##### 5. serial_info.cpp - Serial 功能实现

**职责**：实现 Serial 模块的所有 JS API

**主要函数**：
- `SerialGetPortListNapi` - 获取串口列表
- `SerialOpenNapi` - 打开串口
- `SerialCloseNapi` - 关闭串口
- `SerialReadNapi` - 读取数据（异步）
- `SerialReadSyncNapi` - 读取数据（同步）
- `SerialWriteNapi` - 写入数据（异步）
- `SerialWriteSyncNapi` - 写入数据（同步）
- `SerialGetAttributeNapi` - 获取属性
- `SerialSetAttributeNapi` - 设置属性
- `SerialHasRightNapi` - 检查串口权限
- `SerialRequestRightNapi` - 请求串口权限

**代码行数**：约 900 行

**证据**：`interfaces/kits/js/napi/src/serial_info.cpp`

---

##### 6. napi_util.cpp - N-API 工具函数

**职责**：提供 N-API 工具函数，简化 JS 和 C++ 交互

**主要函数**：
- 参数解析函数
- 错误处理函数
- 类型转换函数

**证据**：`interfaces/kits/js/napi/src/napi_util.cpp`

---

### 5. sa_profile/ - System Ability 配置

**职责**：定义 USB Service 的 System Ability 配置

```
sa_profile/
├── 4201.json                     # USB Service SA 配置
└── BUILD.gn                       # SA Profile 构建配置
```

**关键文件**：

#### 4201.json - USB Service SA 配置

**内容**：
```json
{
    "process": "usb_service",
    "systemability": [{
        "name": 4201,
        "libpath": "libusbservice.z.so",
        "run-on-create": false,
        "auto-restart": true,
        "distributed": false,
        "dump_level": 1
    }]
}
```

**配置说明**：
- `process`: 服务运行进程名
- `name`: SA ID（4201）
- `libpath`: 服务库路径
- `run-on-create`: false - 按需启动
- `auto-restart`: true - 自动重启
- `distributed`: false - 非分布式服务

**证据**：`sa_profile/4201.json`

---

### 6. services/ - 服务层（C++ 实现）

**职责**：实现 USB Service System Ability，处理业务逻辑和设备管理

```
services/
├── native/                       # Native 实现
│   ├── include/                  # 服务头文件
│   │   ├── usb_service.h         # System Ability 主服务
│   │   ├── usb_host_manager.h    # Host 模式管理
│   │   ├── usb_device_manager.h  # Device 模式管理
│   │   ├── usb_port_manager.h    # Port 管理器
│   │   ├── usb_right_manager.h   # 权限管理
│   │   ├── serial_manager.h      # 串口管理
│   │   ├── usb_accessory_manager.h # USB 配件管理
│   │   ├── usb_descriptor_parser.h # 描述符解析
│   │   ├── usb_right_database.h  # 权限数据库
│   │   ├── usb_right_db_helper.h # 数据库辅助
│   │   └── ...
│   └── src/                     # 服务实现
│       ├── usb_service.cpp        # System Ability 主服务实现
│       ├── usb_host_manager.cpp   # Host 模式管理实现
│       ├── usb_device_manager.cpp # Device 模式管理实现
│       ├── usb_port_manager.cpp   # Port 管理器实现
│       ├── usb_right_manager.cpp  # 权限管理实现
│       ├── serial_manager.cpp     # 串口管理实现
│       ├── usb_accessory_manager.cpp # USB 配件管理实现
│       ├── usb_descriptor_parser.cpp # 描述符解析实现
│       ├── usb_right_database.cpp  # 权限数据库实现
│       ├── usb_right_db_helper.cpp # 数据库辅助实现
│       ├── usb_connection_notifier.cpp # 设备插拔通知
│       ├── usb_serial_reader.cpp   # 串口读取器
│       ├── usbd_bulk_callback_impl.cpp # Host 模式 Bulk 回调实现
│       ├── usbd_transfer_callback_impl.cpp # Host 模式传输回调实现
│       ├── usb_bulkcallback_impl.cpp # Bulk 回调实现（直通模式）
│       ├── usb_transfer_callback_impl.cpp # 传输回调实现（直通模式）
│       ├── usb_manager_subscriber.cpp # 管理器订阅者（直通模式）
│       ├── usb_service_subscriber.cpp # 服务订阅者
│       ├── usb_report_sys_event.cpp # HiSysEvent 上报
│       ├── usb_security_report.cpp  # 安全事件上报
│       ├── usb_settings_datashare.cpp # Data Share 设置
│       ├── usb_timer_wrapper.cpp  # 定时器封装
│       ├── usb_function_switch_window.cpp # 功能切换窗口
│       └── ...
├── zidl/                         # ZIDL 接口层
│   ├── include/
│   └── src/
├── usb_service.cfg                # init 配置
└── BUILD.gn                      # 服务构建配置
```

---

#### 6.1 服务核心模块

##### usb_service.cpp - System Ability 主服务

**职责**：USB Service 主入口，注册为 System Ability (SA ID: 4201)

**关键代码**：
```cpp
// usb_service.cpp:89
UsbService::UsbService() : SystemAbility(USB_SYSTEM_ABILITY_ID, true)

// usb_service.cpp:86-87
auto g_serviceInstance = DelayedSpSingleton<UsbService>::GetInstance();
const bool G_REGISTER_RESULT =
    SystemAbility::MakeAndRegisterAbility(DelayedSpSingleton<UsbService>::GetInstance().GetRefPtr());
```

**初始化流程**：
1. 创建管理器实例（UsbHostManager、UsbDeviceManager、UsbPortManager 等）
2. 注册 System Ability
3. 初始化 HAL 驱动
4. 注册设备插拔监听器

**代码行数**：约 2000+ 行

**证据**：`services/native/src/usb_service.cpp`

---

##### usb_host_manager.cpp - Host 模式管理

**职责**：管理 USB Host 模式的设备

**主要功能**：
- 设备枚举
- 设备打开/关闭
- 接口声明/释放
- 数据传输（Bulk、Control、Interrupt）
- 描述符解析

**主要方法**：
- `GetDevices()` - 获取设备列表
- `OpenDevice()` - 打开设备
- `ClaimInterface()` - 声明接口
- `BulkTransfer()` - 批量传输
- `ControlTransfer()` - 控制传输

**证据**：`services/native/src/usb_host_manager.cpp`

---

##### usb_device_manager.cpp - Device 模式管理

**职责**：管理 USB Device 模式的功能

**主要功能**：
- 功能切换（ACM/ECM/MTP/RNDIS/HDC）
- 功能查询
- 配件管理

**主要方法**：
- `SetCurrentFunctions()` - 设置当前功能
- `GetCurrentFunctions()` - 获取当前功能
- `OpenAccessory()` - 打开配件
- `RequestAccessoryRight()` - 请求配件权限

**证据**：`services/native/src/usb_device_manager.cpp`

---

##### usb_port_manager.cpp - Port 管理器

**职责**：管理 USB-C 端口角色配置

**主要功能**：
- 端口枚举
- 角色设置（Source/Sink, Host/Device）
- 模式查询

**主要方法**：
- `GetPorts()` - 获取端口列表
- `SetPortRole()` - 设置端口角色
- `GetSupportedModes()` - 获取支持模式

**证据**：`services/native/src/usb_port_manager.cpp`

---

##### usb_right_manager.cpp - 权限管理

**职责**：管理 USB 设备访问权限

**主要功能**：
- 权限请求
- 权限检查
- 权限添加
- 权限移除
- 权限数据库操作

**主要方法**：
- `HasRight()` - 检查权限
- `RequestRight()` - 请求权限
- `AddRight()` - 添加权限
- `RemoveRight()` - 移除权限

**权限流程**：
1. 应用请求权限
2. 弹出权限对话框（UI）
3. 用户授权/拒绝
4. 存储权限到数据库
5. 后续访问直接查询数据库

**证据**：`services/native/src/usb_right_manager.cpp`

---

##### serial_manager.cpp - 串口管理

**职责**：管理 USB 串口设备

**主要功能**：
- 串口枚举
- 串口打开/关闭
- 串口读写
- 串口属性配置（波特率、数据位等）
- 串口权限管理

**主要方法**：
- `SerialOpen()` - 打开串口
- `SerialClose()` - 关闭串口
- `SerialRead()` - 读取数据
- `SerialWrite()` - 写入数据
- `SerialSetAttribute()` - 设置属性
- `SerialGetAttribute()` - 获取属性

**证据**：`services/native/src/serial_manager.cpp`

---

##### usb_descriptor_parser.cpp - 描述符解析

**职责**：解析 USB 设备描述符

**主要功能**：
- 设备描述符解析
- 配置描述符解析
- 接口描述符解析
- 端点描述符解析

**主要方法**：
- `ParseDeviceDescriptor()` - 解析设备描述符
- `ParseConfigurationDescriptor()` - 解析配置描述符
- `ParseInterfaceDescriptor()` - 解析接口描述符

**风险**：描述符解析可能存在缓冲区溢出风险（需注意）

**证据**：`services/native/src/usb_descriptor_parser.cpp`

---

##### usb_right_database.cpp - 权限数据库

**职责**：存储和管理 USB 权限数据

**主要功能**：
- 数据库初始化
- 权限插入/查询/删除
- 数据库升级

**数据库类型**：RDB (Relational Database)

**主要方法**：
- `InitDatabase()` - 初始化数据库
- `InsertRight()` - 插入权限
- `QueryRight()` - 查询权限
- `DeleteRight()` - 删除权限

**证据**：`services/native/src/usb_right_database.cpp`

---

### 7. utils/ - 工具库

**职责**：提供通用工具函数和定义

```
utils/
├── native/
│   ├── include/                   # 工具头文件
│   │   ├── usb_common.h           # USB 通用定义
│   │   ├── usb_errors.h           # USB 错误码定义
│   │   ├── usb_napi_errors.h     # N-API 错误码定义
│   │   ├── usb_request.h          # USB 请求定义
│   │   └── ...
│   └── src/                      # 工具实现
│       ├── struct_parcel.cpp     # 结构体序列化
│       ├── usb_napi_errors.cpp   # N-API 错误处理
│       ├── usb_settings_datashare.cpp # Data Share 设置
│       └── ...
└── BUILD.gn                       # 工具库构建配置
```

**关键文件**：

##### usb_common.h - USB 通用定义

**职责**：定义 USB 通用常量和类型

**主要定义**：
- USB 功能类型（ACM、ECM、MTP、RNDIS、HDC）
- USB 传输类型（BULK、CONTROL、INTERRUPT、ISOCHRONOUS）
- USB 端点方向（IN、OUT）
- USB 端口角色（HOST、DEVICE、SOURCE、SINK）

**证据**：`utils/native/include/usb_common.h`

---

##### usb_errors.h - USB 错误码定义

**职责**：定义 USB 错误码

**主要错误码**：
- 设备未找到
- 权限被拒绝
- 参数错误
- 设备忙
- 传输失败

**证据**：`utils/native/include/usb_errors.h`

---

##### usb_napi_errors.h - N-API 错误码定义

**职责**：定义 N-API 返回给 JS 层的错误码

**主要错误码**：
- USB_ERROR_INVALID_PARAM
- USB_ERROR_BUSY
- USB_ERROR_NOT_SUPPORTED
- USB_ERROR_PERMISSION_DENIED
- ...

**证据**：`utils/native/include/usb_napi_errors.h`

---

## 核心文件定位

### 入口文件

| 文件 | 路径 | 作用 |
|------|------|------|
| N-API 入口 | `interfaces/kits/js/napi/src/usb_middle.cpp` | USB 模块入口（动态加载） |
| N-API 入口 | `interfaces/kits/js/napi/src/usbmanager_middle.cpp` | USB Manager 模块入口 |
| N-API 入口 | `interfaces/kits/js/napi/src/serial_middle.cpp` | Serial 模块入口 |
| Service 入口 | `services/native/src/usb_service.cpp` | System Ability 主服务入口 |

---

### 配置文件

| 文件 | 路径 | 作用 |
|------|------|------|
| 组件配置 | `bundle.json` | 组件元信息、依赖、Feature Flags |
| SA 配置 | `sa_profile/4201.json` | System Ability 配置（进程名、SA ID） |
| Init 配置 | `services/usb_service.cfg` | 服务启动参数 |
| GN 变量 | `usbmgr.gni` | GN 构建变量定义 |
| HiSysEvent | `hisysevent.yaml` | 安全事件配置 |

---

### 构建文件

| 文件 | 路径 | Target |
|------|------|--------|
| N-API 构建 | `interfaces/kits/js/napi/BUILD.gn` | usb, usbmanager, serial |
| 内部库构建 | `interfaces/innerkits/BUILD.gn` | usbsrv_client, usb_server_stub |
| 服务构建 | `services/BUILD.gn` | usbservice |
| 工具库构建 | `utils/BUILD.gn` | 通用工具库 |
| SA 配置构建 | `sa_profile/BUILD.gn` | sa_profile |

---

## 代码导航图

### 功能模块 → 文件映射

| 功能模块 | 实现文件 | 职责 |
|---------|---------|------|
| **USB Host 设备管理** | `services/native/src/usb_host_manager.cpp` | 设备枚举、打开、关闭、接口操作 |
| **USB Device 功能管理** | `services/native/src/usb_device_manager.cpp` | 功能切换、配件管理 |
| **USB Port 角色管理** | `services/native/src/usb_port_manager.cpp` | 端口枚举、角色设置 |
| **USB 权限管理** | `services/native/src/usb_right_manager.cpp` | 权限请求、检查、存储 |
| **Serial 串口管理** | `services/native/src/serial_manager.cpp` | 串口打开、关闭、读写、属性配置 |
| **USB 描述符解析** | `services/native/src/usb_descriptor_parser.cpp` | 描述符解析 |
| **USB 权限数据库** | `services/native/src/usb_right_database.cpp` | 权限数据存储和查询 |

---

### JS API → C++ 实现映射

| JS API 模块 | C++ 实现文件 | 主要函数 |
|------------|------------|---------|
| **@ohos.usb** | `interfaces/kits/js/napi/src/usb_info.cpp` | Host/Device/Port 功能实现 |
| **@ohos.usbManager** | `interfaces/kits/js/napi/src/usb_info.cpp` | USB Manager 功能实现 |
| **@ohos.usbManager.serial** | `interfaces/kits/js/napi/src/serial_info.cpp` | Serial 功能实现 |

---

### IPC 接口 → 实现映射

| IPC 方法 | 服务端实现 | 客户端代理 |
|---------|----------|-----------|
| `IUsbServer::GetDevices` | `services/native/src/usb_service.cpp` | `interfaces/innerkits/native/src/usb_srv_client.cpp` |
| `IUsbServer::OpenDevice` | `services/native/src/usb_host_manager.cpp` | `interfaces/innerkits/native/src/usb_srv_client.cpp` |
| `IUsbServer::RequestRight` | `services/native/src/usb_right_manager.cpp` | `interfaces/innerkits/native/src/usb_srv_client.cpp` |
| `IUsbServer::SerialOpen` | `services/native/src/serial_manager.cpp` | `interfaces/innerkits/native/src/usb_srv_client.cpp` |

---

### 调用链示例

#### 设备枚举调用链

```
JS: usb.getDevices()
  ↓
N-API: usb_info.cpp::CoreGetDevices()
  ↓
UsbSrvClient: usb_srv_client.cpp::GetDevices()
  ↓
IPC: IUsbServer::GetDevices() (Binder)
  ↓
UsbService: usb_service.cpp::GetDevices()
  ↓
UsbHostManager: usb_host_manager.cpp::GetDevices()
  ↓
HAL: drivers_interface_usb::IUsbClient::GetDevices()
  ↓
USB Driver
```

#### 权限请求调用链

```
JS: usb.requestRight(deviceName)
  ↓
N-API: usb_info.cpp::CoreRequestRight()
  ↓
UsbSrvClient: usb_srv_client.cpp::RequestRight()
  ↓
IPC: IUsbServer::RequestRight() (Binder)
  ↓
UsbService: usb_service.cpp::RequestRight()
  ↓
UsbRightManager: usb_right_manager.cpp::RequestRight()
  ↓
UI: 弹出权限对话框 (frameworks/dialog/dialog_ui/usb_right_dialog)
  ↓
UsbRightDatabase: usb_right_database.cpp::InsertRight()
```

---

## 快速查找指南

### 查找 JS API 实现

**步骤**：
1. 确定 API 所属模块（Host/Device/Port/Serial）
2. 到对应文件查找：
   - Host/Device/Port → `interfaces/kits/js/napi/src/usb_info.cpp`
   - Serial → `interfaces/kits/js/napi/src/serial_info.cpp`

---

### 查找 IPC 方法实现

**步骤**：
1. 查看 `interfaces/innerkits/IUsbServer.idl` 确认方法签名
2. 到服务端实现查找：
   - Host 方法 → `services/native/src/usb_host_manager.cpp`
   - Device 方法 → `services/native/src/usb_device_manager.cpp`
   - Port 方法 → `services/native/src/usb_port_manager.cpp`
   - Serial 方法 → `services/native/src/serial_manager.cpp`
   - 通用方法 → `services/native/src/usb_service.cpp`

---

### 查找错误码定义

**步骤**：
1. N-API 错误码 → `utils/native/include/usb_napi_errors.h`
2. 内部错误码 → `utils/native/include/usb_errors.h`

---

### 查找权限管理代码

**步骤**：
1. 权限检查逻辑 → `services/native/src/usb_right_manager.cpp`
2. 权限数据库操作 → `services/native/src/usb_right_database.cpp`
3. 权限对话框 UI → `frameworks/dialog/dialog_ui/usb_right_dialog`

---

## 相关文档

- [项目概览](01_Overview.md) - 项目定位和核心功能
- [架构与数据流](02_Architecture.md) - 详细的架构设计
- [对外接口文档](04_Interface.md) - 完整的 API 文档
- [攻击面分析](05_AttackSurface.md) - 外部输入点和安全风险

---

**更新时间**：2026-02-07
**证据来源**：`find` 命令扫描、`README.md`、各文件 BUILD.gn
