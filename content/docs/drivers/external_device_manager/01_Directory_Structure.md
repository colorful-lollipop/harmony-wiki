# 目录结构与模块职责

本文档描述扩展外部设备管理模块的目录结构、各目录的职责划分以及关键文件的用途说明。理解目录结构对于阅读代码、定位问题和扩展功能都具有重要意义。

## 顶层目录结构

```
external_device_manager/
├── frameworks/                    # 框架层：N-API 绑定和 DDK 实现
├── interfaces/                   # 接口层：对外 API 定义
├── sa_profile/                   # SA 配置文件
├── services/                     # 服务层：系统服务实现
├── utils/                        # 工具层：通用工具和类型定义
├── test/                         # 测试代码（本文档不涉及）
├── wiki/                         # 文档目录
├── bundle.json                   # 组件配置文件
├── extdevmgr.gni                 # GN 构建变量定义
├── hisysevent.yaml               # 系统事件配置
└── README_zh.md                  # 项目说明文档
```

各顶层目录的功能定位如下。`frameworks/` 目录包含面向应用开发者的框架代码，包括 N-API 绑定层（提供 JS API 接口）和 DDK 层（提供 C API 接口）。`interfaces/` 目录包含对外接口的头文件定义，包括 Innerkits（系统内部接口）和 DDK（驱动开发接口）。`sa_profile/` 目录包含 System Ability 的配置文件，定义了 SA ID、进程名、启动参数等。`services/` 目录包含系统服务的核心实现代码，以 native 库形式运行在 `hdf_ext_devmgr` 进程中。`utils/` 目录包含跨模块复用的工具代码，包括错误码定义、单例模式基类等。

## frameworks 目录详解

### frameworks/js/napi/ 目录

该目录包含 N-API 绑定层代码，负责将 C++ 服务接口暴露给 JavaScript 应用调用。这是应用开发者直接接触的接口层。

| 目录/文件 | 类型 | 职责 |
|----------|------|------|
| `device_manager/` | 目录 | 设备管理器 N-API 实现，提供设备查询、绑定、解绑定等 JS API |
| `device_manager/device_manager_middle.cpp` | 文件 | 主 N-API 模块入口，注册 `driver.deviceManager` 模块 |
| `device_manager/device_manager_middle.h` | 文件 | 异步操作数据结构和回调定义 |
| `driver_extension_ability/` | 目录 | 驱动扩展 Ability 的 N-API 绑定 |
| `driver_extension_ability/driver_extension_ability_module.cpp` | 文件 | 注册 `app.ability.DriverExtensionAbility` 模块 |
| `driver_extension_ability/driver_extension_ability.js` | 文件 | JS 侧的能力包装器 |
| `driver_extension_context/` | 目录 | 驱动扩展上下文的 N-API 绑定 |
| `driver_extension_context/driver_extension_context_module.cpp` | 文件 | 注册 `application.DriverExtensionContext` 模块 |
| `driver_extension_context/driver_extension_context.js` | 文件 | JS 侧的上下文包装器 |

关键实现文件说明如下。`device_manager_middle.cpp` 是 N-API 层的核心文件，模块注册在第 816 行，通过 `napi_module_register(&g_moduleManager)` 完成。导出的 JS API 包括 `queryDevices`（查询设备）、`bindDevice`（绑定设备）、`unbindDevice`（解绑设备）、`bindDriverWithDeviceId`（按设备ID绑定驱动）、`unbindDriverWithDeviceId`（按设备ID解绑驱动）、`queryDeviceInfo`（查询设备信息）、`queryDriverInfo`（查询驱动信息），以及 `BusType` 枚举。

### frameworks/ddk/ 目录

该目录包含 Driver Development Kit（DDK）实现，为驱动开发者提供访问硬件的 C API 接口。

| 目录/文件 | 类型 | 职责 |
|----------|------|------|
| `base/` | 目录 | 基础 DDK，提供共享内存等通用功能 |
| `base/ddk_api.cpp` | 文件 | 共享内存创建、映射、销毁接口实现 |
| `base/ddk_types.h` | 文件 | 共享内存类型定义和错误码 |
| `usb/` | 目录 | USB DDK 实现 |
| `usb/usb_ddk_api.cpp` | 文件 | USB 设备访问接口实现 |
| `usb/usb_config_desc_parser.cpp` | 文件 | USB 配置描述符解析器 |
| `usb/usb_ddk_types.h` | 文件 | USB DDK 类型定义 |
| `usb/usb_ddk_api.h` | 文件 | USB DDK API 声明 |
| `hid/` | 目录 | HID DDK 实现 |
| `hid/input_emit_event.cpp` | 文件 | HID 设备事件发送接口实现 |
| `hid/hid_ddk_api.h` | 文件 | HID DDK API 声明 |
| `hid/hid_ddk_types.h` | 文件 | HID DDK 类型定义 |
| `scsi/` | 目录 | SCSI Peripheral DDK 实现 |
| `scsi/scsi_ddk_api.cpp` | 文件 | SCSI 设备操作接口实现 |
| `scsi/scsi_peripheral_api.h` | 文件 | SCSI DDK API 声明 |
| `scsi/scsi_peripheral_types.h` | 文件 | SCSI DDK 类型定义 |
| `usb_serial/` | 目录 | USB Serial DDK 实现 |
| `usb_serial/usb_serial_ddk_api.cpp` | 文件 | 串口通信接口实现 |
| `usb_serial/usb_serial_api.h` | 文件 | USB Serial DDK API 声明 |
| `usb_serial/usb_serial_types.h` | 文件 | USB Serial DDK 类型定义 |

DDK API 采用 `OH_` 前缀命名，符合 OpenHarmony NDK 规范。错误码以模块为单位划分，如 USB DDK 错误码基址为 27400000，HID DDK 错误码基址为 27300000。

## interfaces 目录详解

### interfaces/innerkits/ 目录

该目录包含系统内部 C++ 接口定义，仅供系统组件使用，不对外暴露给普通应用。

| 文件 | 职责 |
|------|------|
| `driver_ext_mgr_types.h` | 定义设备、驱动相关的 Parcelable 数据类型，包括 DeviceData、DeviceInfoData、DriverInfoData 等 |
| `driver_ext_mgr_client.h` | 定义 `DriverExtMgrClient` 类，提供获取 SA 代理和调用 SA 接口的方法 |
| `hdf_ext_devmgr_interface_code.h` | 定义 IPC 接口调用码，用于 IPC 框架路由 |
| `IDriverExtMgr.idl` | IDL 接口定义文件，描述 SA 的 RPC 接口方法 |
| `IDriverExtMgrCallback.idl` | IDL 回调接口定义，描述设备绑定回调方法 |
| `BUILD.gn` | Innerkits 构建配置 |

IDL 接口是 IPC 通信的核心定义。`IDriverExtMgr.idl` 定义了 8 个方法：QueryDevice（查询设备）、BindDevice（绑定设备）、UnBindDevice（解绑设备）、BindDriverWithDeviceId（按设备ID绑定驱动）、UnBindDriverWithDeviceId（按设备ID解绑驱动）、QueryDeviceInfo（查询设备信息）、QueryDriverInfo（查询驱动信息）、NotifyUsbPeripheralFault（通知设备故障）。`IDriverExtMgrCallback.idl` 定义了 3 个回调方法：OnConnect（连接成功）、OnDisconnect（断开连接）、OnUnBind（解绑完成）。

### interfaces/ddk/ 目录

该目录包含 DDK 头文件定义，与 frameworks/ddk/ 中的实现对应，供驱动开发者在编译时引用。

| 子目录 | 职责 |
|--------|------|
| `base/` | 基础 DDK 头文件 |
| `usb/` | USB DDK 头文件 |
| `hid/` | HID DDK 头文件 |
| `scsi/` | SCSI DDK 头文件 |
| `usb_serial/` | USB Serial DDK 头文件 |

## sa_profile 目录详解

该目录包含 System Ability 的配置文件，用于描述 SA 的元信息。

| 文件 | 职责 |
|------|------|
| `5110.json` | SA 配置文件，定义了 SA ID、进程名、库路径、启动参数等 |
| `BUILD.gn` | SA profile 构建配置 |

SA 配置文件的关键配置项包括：进程名设置为 `hdf_ext_devmgr`；SA ID 为 `5110`；库路径为 `libdriver_extension_manager.z.so`；启动模式为按需启动（`run-on-create: false`）；支持自动重启（`auto-restart: true`）；不启用分布式能力（`distributed: false`）；最小 HDI 代理版本要求 `libusb_proxy_2.0.z.so` 和 `libusb_ddk_proxy_1.1.z.so`。

## services 目录详解

### services/native/driver_extension/ 目录

该目录包含驱动扩展 Ability 的框架实现。

| 目录/文件 | 职责 |
|-----------|------|
| `include/driver_extension.h` | 定义 `DriverExtension` 基类 |
| `include/driver_extension_context.h` | 定义 `DriverExtensionContext` 上下文类 |
| `include/driver_extension_module_loader.h` | 定义模块加载器 |
| `include/js_driver_extension.h` | 定义 JS 驱动扩展类 |
| `include/js_driver_extension_context.h` | 定义 JS 上下文辅助类 |
| `src/` | 实现源码目录 |

`DriverExtension` 类继承自 `ExtensionBase<DriverExtensionContext>`，是所有驱动扩展 Ability 的基类。它定义了 OnStart（启动）、OnConnect（连接）、OnDisconnect（断开）、OnStop（停止）等生命周期回调。`JsDriverExtension` 类继承自 `DriverExtension`，实现了 JavaScript 驱动的运行时支持。

### services/native/driver_extension_manager/ 目录

该目录是核心服务实现的主体，包含多个功能子模块。

| 子目录 | 职责 |
|--------|------|
| `include/` | 头文件目录，包含各子模块的接口定义 |
| `src/` | 实现源码目录 |
| `profile/` | 配置文件目录 |
| `resources/` | 资源文件目录 |

**include/device_manager/** 子目录包含设备管理模块的接口定义：

| 文件 | 职责 |
|------|------|
| `etx_device_mgr.h` | 定义 `ExtDeviceManager` 单例类，核心设备管理类 |
| `device.h` | 定义 `Device` 设备抽象类 |
| `driver_extension_controller.h` | 定义 `DriverExtensionController` 驱动生命周期控制器 |
| `dev_change_callback.h` | 定义设备变化回调接口 |
| `bundle_update_callback.h` | 定义包更新回调接口 |

**include/drivers_pkg_manager/** 子目录包含驱动包管理模块的接口定义：

| 文件 | 职责 |
|------|------|
| `driver_pkg_manager.h` | 定义 `DriverPkgManager` 单例类 |
| `pkg_database.h` | 定义 `PkgDatabase` SQLite 包装类 |
| `pkg_db_helper.h` | 定义 `PkgDbHelper` 数据库操作辅助类 |
| `pkg_tables.h` | 定义数据库表结构 |
| `drv_bundle_state_callback.h` | 定义 BMS 状态回调 |
| `ibundle_update_callback.h` | 定义包更新回调接口 |
| `driver_os_account_subscriber.h` | 定义用户账号切换监听 |

**include/bus_extension/** 子目录包含总线扩展模块的接口定义：

| 文件 | 职责 |
|------|------|
| `core/bus_extension_core.h` | 定义总线扩展注册中心 |
| `usb/usb_bus_extension.h` | 定义 USB 总线扩展实现 |
| `usb/usb_device_info.h` | 定义 USB 设备信息 |
| `usb/usb_driver_info.h` | 定义 USB 驱动信息 |
| `usb/usb_dev_subscriber.h` | 定义 USB 设备热插拔监听器 |
| `usb/usb_driver_change_callback.h` | 定义 USB 驱动变化回调 |

**include/device_notification/** 子目录包含通知模块的接口定义：

| 文件 | 职责 |
|------|------|
| `notification_peripheral.h` | 定义外设通知处理器 |
| `notification_locale.h` | 定义通知本地化支持 |

**include/drivers_hisysevent/** 子目录包含系统事件模块的接口定义：

| 文件 | 职责 |
|------|------|
| `driver_report_sys_event.h` | 定义 `ExtDevReportSysEvent` 系统事件上报类 |

**include/ext_permission_manager.h** 文件定义权限管理模块，提供权限验证、系统应用检查等功能。

**src/** 子目录包含各子模块的实现源码，与 include 目录结构对应。

## utils 目录详解

该目录包含跨模块复用的工具代码。

| 文件 | 职责 |
|------|------|
| `include/edm_errors.h` | 定义错误码枚举，包括 EDM_OK、EDM_ERR_NO_PERM 等 |
| `include/ext_object.h` | 定义扩展对象基类、BusType 枚举 |
| `include/idev_change_callback.h` | 定义设备变化回调接口 |
| `include/idriver_change_callback.h` | 定义驱动变化回调接口 |
| `include/single_instance.h` | 定义单例模式模板 |
| `include/ibus_extension.h` | 定义总线扩展接口 |
| `include/hilog_wrapper.h` | 定义日志宏封装 |

错误码采用分层设计。模块 ID 为 1，错误码基址为 `ErrCodeOffset(SUBSYS_DRIVERS, EDM_MODULE_ID)`。通用错误码包括：0 表示成功；负值或正值按具体枚举值定义，如 `EDM_ERR_NO_PERM` 表示权限不足，`EDM_ERR_NOT_SYSTEM_APP` 表示非系统应用调用。

## 模块职责总结

| 目录 | 主要职责 | 关键类/文件 |
|------|----------|------------|
| frameworks/js/napi | JS API 绑定 | device_manager_middle.cpp |
| frameworks/ddk | 驱动开发套件 | usb_ddk_api.h、hid_ddk_api.h |
| interfaces/innerkits | 内部 C++ 接口 | IDriverExtMgr.idl、driver_ext_mgr_client.h |
| services/native/driver_extension | 驱动扩展框架 | DriverExtension、JsDriverExtension |
| services/native/driver_extension_manager | 核心服务 | ExtDeviceManager、DriverPkgManager |
| sa_profile | SA 配置 | 5110.json |
| utils | 通用工具 | edm_errors.h、single_instance.h |

## 相关文档

| 文档 | 描述 |
|------|------|
| [00_Overview.md](./00_Overview.md) | 项目概览与核心能力 |
| [02_Architecture.md](./02_Architecture.md) | 架构设计与组件关系 |
| [03_NAPI_Reference.md](./03_NAPI_Reference.md) | JS API 接口参考 |
| [04_DDK_Reference.md](./04_DDK_Reference.md) | DDK C API 接口参考 |
| [05_Inner_API.md](./05_Inner_API.md) | 内部模块接口参考 |
