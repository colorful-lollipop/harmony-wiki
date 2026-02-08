# 扩展外部设备管理模块概览

本文档描述 OpenHarmony 扩展外部设备管理（External Device Manager）模块的项目定位、核心能力、运行环境以及关键概念。该模块为设备厂商提供应用态扩展设备驱动的完整生命周期管理解决方案。

## 项目定位与边界

扩展外部设备管理模块是 OpenHarmony HDF（HDF Driver Foundation）驱动框架的重要组成部分，其核心定位是解决非标准协议可插拔设备驱动的接入问题。传统的 HDF 驱动框架已经支持板载标准外设的驱动能力，但随着外部设备生态的不断拓展，设备厂商经常需要开发基于非标准协议的USB等可插拔设备的驱动程序。扩展外部设备管理架构正是在这一背景下应运而生，它提供了一套完整的应用态扩展设备驱动开发、部署、安装、运行和能力开放的全流程管理机制。

该模块的主要边界包括以下几个方面。首先，在用户层面，模块为三方应用和系统服务提供外部设备的查询、绑定、解绑定等接口能力，使得应用程序能够方便地发现和操作外部设备。其次，在驱动开发层面，模块基于驱动扩展Ability能力，为设备厂商提供标准化的驱动开发框架，厂商可以复用现有的 OpenHarmony 应用开发环境来开发驱动扩展 Ability，并通过系统安全认证后为其他三方应用提供定制化的设备驱动硬件功能接口能力。最后，在系统集成层面，模块与 HDF 驱动框架、USB 总线驱动、系统能力管理（SAMGR）、包管理服务（BMS）等核心子系统保持紧密协作，共同完成设备的枚举、驱动匹配和生命周期管理。

从技术架构来看，扩展外部设备管理模块采用了分层设计思想。最上层是 JS API 层，通过 N-API 为应用提供声明式的设备操作接口；中间是服务层，以 System Ability（SA）形式运行，负责设备管理、驱动匹配和包管理等核心逻辑；底层是 DDK（Driver Development Kit）层，为驱动开发者提供访问 USB、HID、SCSI 等外设的标准 C API。这种分层设计既保证了接口的易用性，又确保了底层操作的灵活性和性能。

## 核心能力

扩展外部设备管理模块提供以下核心能力，这些能力共同构成了设备厂商开发外部设备驱动的完整技术栈。

**设备发现与枚举能力**是模块的基础功能之一。当用户插入 USB 设备时，USB 总线扩展插件负责监听热插拔事件并读取设备信息，随后总线扩展核心模块进行设备枚举，并将发现的设备信息上报给设备管理模块。这一过程完全自动化，对应用层透明。设备枚举不仅包括标准 USB 设备，还支持复合设备和多接口设备，能够准确识别设备的 VID（Vendor ID）、PID（Product ID）、设备类、接口描述符等关键信息。

**驱动匹配与绑定能力**是模块的核心价值所在。模块维护了一个驱动元数据库，当新设备接入时，设备管理模块会查询匹配的驱动信息。匹配算法基于设备的总线类型、厂商 ID、产品 ID 以及设备类信息进行综合判断。一旦找到匹配的驱动，模块会触发驱动的加载和绑定流程，将设备与驱动扩展 Ability 进行关联。绑定成功后，应用可以获得驱动扩展的远程对象引用，从而与驱动进行交互。这种基于匹配的机制使得设备能够即插即用，大大简化了用户的操作流程。

**驱动包生命周期管理能力**涵盖了驱动的安装、更新、卸载全过程。包信息管理模块负责解析扩展驱动包的元数据信息，包括驱动名称、包名、支持的设备列表、版本号等，并将这些信息持久化存储到数据库中。模块会监听包管理服务的广播事件，自动感知驱动的安装和卸载，并相应地更新设备与驱动的匹配关系。当驱动被卸载时，模块会自动解除设备与该驱动的绑定，确保系统状态的正确性。

**设备连接管理能力**处理设备的热插拔和连接断开事件。当用户拔掉已绑定的设备时，模块会收到设备移除通知，并回调应用的断开处理函数。当用户重新插入设备时，如果驱动仍然存在，模块会尝试重新建立连接。连接管理还包括超时处理、错误恢复等机制，确保在各种异常情况下系统都能保持稳定运行。

**硬件访问能力**通过 DDK 层提供给驱动开发者。USB DDK 提供了完整的 USB 设备访问接口，包括设备描述符读取、接口声明、控制传输、批量传输、中断传输等操作。HID DDK 支持 HID 设备的创建和事件发送，适用于游戏手柄、键盘、鼠标等输入设备。USB Serial DDK 提供了串口通信接口，适用于 Modem、Arduino 等串口类设备。SCSI Peripheral DDK 支持 SCSI 存储设备的访问，适用于 U 盘、移动硬盘等大容量存储设备。这些 DDK 接口的设计遵循了各协议规范的标准，同时针对 OpenHarmony 系统进行了适配和优化。

## 运行环境

扩展外部设备管理模块运行在 OpenHarmony Standard 系统上，对硬件平台有以下要求。

**处理器架构支持**方面，模块支持 ARM32 位和 ARM64 位两种架构。在 32 位 ARM 系统上运行需要额外的编译标志来支持 Binder IPC 的 32 位模式。编译时可以通过 `--target-cpu arm64` 参数来选择 64 位 ARM 架构。

**系统服务依赖**方面，模块作为系统能力运行，需要依赖以下核心系统服务。SAMGR（System Ability Manager）负责系统能力的注册、发现和生命周期管理，扩展外部设备管理服务以 SA ID 5100 运行，由 SAMGR 按需启动。IPC 框架提供跨进程通信能力，所有与应用的交互都通过 IPC 完成。包管理服务（BMS）提供应用安装状态监听能力，用于感知驱动扩展包的安装和卸载。账户服务（OS Account Manager）提供用户账号切换监听能力，用于在不同用户间维护设备状态。系统能力 1（HDF）提供 USB 核心服务接口，用于访问 USB 设备。分布式通知服务（ANS）提供通知发布能力，用于向用户展示设备状态变更。

**权限要求**方面，访问扩展外部设备管理 API 需要相应的系统权限。调用设备查询和绑定接口需要 `ohos.permission.ACCESS_EXTENSIONAL_DEVICE_DRIVER` 权限，这是一个系统级权限，仅授予系统应用。使用 DDK 接口进行设备操作也需要对应的权限，如 `ohos.permission.ACCESS_DDK_USB`、`ohos.permission.ACCESS_DDK_HID` 等。权限检查在服务层进行，未授权的调用会收到错误码 `EDM_ERR_NO_PERM`（201）。

**资源占用**方面，根据 bundle.json 中的配置信息，模块的 ROM 占用约为 735KB，RAM 占用约为 8000KB。这些资源包括服务主进程、多个 DDK 共享库、资源文件以及运行时所需的缓存空间。

## 关键概念

理解扩展外部设备管理模块需要掌握以下关键概念。

**驱动扩展 Ability** 是 OpenHarmony Ability 框架的扩展类型，专门用于开发外部设备驱动。驱动扩展 Ability 继承自 `DriverExtension` 基类，实现 `OnConnect`、`OnDisconnect` 等生命周期回调。与普通 Ability 不同，驱动扩展 Ability 运行在独立的进程中，通过 IPC 与设备管理服务通信。每个驱动扩展 Ability 都有一个唯一的 bundle 名称和应用标识符，系统通过这些标识符来定位和加载驱动。

**设备标识符（Device ID）** 是模块内部用于唯一标识一个外部设备的 64 位无符号整数。Device ID 的编码包含总线类型和设备实例信息，不同总线的设备使用不同的编码空间。Device ID 在设备枚举时由总线扩展模块生成，在整个设备生命周期内保持不变。应用在进行设备绑定和操作时需要使用这个标识符。

**驱动标识符（Driver UID）** 是驱动扩展 Ability 的唯一标识符，由系统自动生成并与驱动扩展 Ability 的生命周期绑定。Driver UID 的格式为 `{bundleName}-{abilityName}-{userId}`，包含了驱动所属的包名、Ability 名称和用户 ID。驱动标识符用于在多用户环境下区分不同用户的驱动实例，也用于权限检查时的身份验证。

**总线扩展（Bus Extension）** 是模块的可扩展架构设计，用于支持不同类型的外部设备总线。目前主要支持 USB 总线（`BUS_TYPE_USB = 1`），但架构设计允许添加其他总线类型的扩展，如蓝牙、WiFi 等。总线扩展模块负责总线特定的设备枚举、驱动信息解析和设备匹配逻辑，通过 `IBusExtension` 接口进行抽象，使得核心模块与总线特定逻辑解耦。

**设备数据（DeviceData）** 和 **设备信息（DeviceInfoData）** 是模块中的两类核心数据结构。DeviceData 用于在设备枚举时传递设备的基本信息，包括总线类型、设备 ID 和描述文本。DeviceInfoData 用于向应用展示设备的详细信息，包括设备 ID、是否已匹配驱动、匹配的驱动 UID 等。USB 设备还有专门的 `USBDevice` 和 `USBDeviceInfoData` 子类，包含 VID、PID、接口描述符列表等 USB 特有信息。

**回调接口（Callback）** 是异步操作的重要机制。由于设备绑定是一个异步过程（需要启动驱动扩展 Ability），应用需要提供回调函数来处理绑定结果。`IDriverExtMgrCallback` 接口定义了 `OnConnect`（绑定成功）、`OnDisconnect`（断开连接）、`OnUnBind`（解绑完成）三个回调方法。应用可以同时使用回调函数和 Promise 两种模式，模块会根据参数类型自动选择合适的处理方式。

## 系统能力声明

模块在 bundle.json 中声明了以下 SystemCapability，这些能力标识了模块提供的功能特性。

| 能力名称 | 描述 |
|---------|------|
| SystemCapability.Driver.HID.Extension | HID 驱动扩展能力 |
| SystemCapability.Driver.ExternalDevice | 外部设备管理能力 |
| SystemCapability.Driver.USB.Extension | USB 驱动扩展能力 |
| SystemCapability.Driver.DDK.Extension | DDK 扩展能力 |
| SystemCapability.Driver.UsbSerial.Extension | USB 串口驱动扩展能力 |
| SystemCapability.Driver.SCSI.Extension | SCSI 驱动扩展能力 |

## 相关文档

| 文档 | 描述 |
|------|------|
| [01_Directory_Structure.md](./01_Directory_Structure.md) | 目录结构与模块职责 |
| [02_Architecture.md](./02_Architecture.md) | 架构设计与组件关系 |
| [03_NAPI_Reference.md](./03_NAPI_Reference.md) | JS API 接口参考 |
| [04_DDK_Reference.md](./04_DDK_Reference.md) | DDK C API 接口参考 |
| [05_Inner_API.md](./05_Inner_API.md) | 内部模块接口参考 |
| [06_Build_System.md](./06_Build_System.md) | GN 构建系统说明 |
| [07_Security_Review.md](./07_Security_Review.md) | 安全风险分析 |
| [08_Troubleshooting.md](./08_Troubleshooting.md) | 常见问题与调试 |
