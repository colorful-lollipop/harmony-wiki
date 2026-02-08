# N-API 参考

> connectivity_cangjie_wrapper Cangjie API 清单、参数校验与错误码

## 概述

本文档详细列出 `connectivity_cangjie_wrapper` 项目中所有导出的 Cangjie API，包括 API 名称与签名、底层绑定位置、参数说明与校验规则、权限要求以及错误码与异常处理机制。项目采用统一的 `@!APILevel` 注解来标注 API 的版本信息、权限要求和系统能力，确保开发者能够清晰了解每个 API 的使用条件。

## BLE 低功耗蓝牙

### 核心 API 列表

低功耗蓝牙模块提供了完整的 BLE 功能接口，涵盖设备发现、广播和 GATT 通信等核心能力。所有 API 均可通过 `ohos.bluetooth.ble` 包导入使用，底层调用 `bluetooth:cj_bluetooth_ble_ffi` 组件提供的 FFI 接口。

`createGattServer` 函数用于创建 GATT 服务器实例，返回一个 `GattServer` 对象供开发者进行后续的服务注册和操作。该 API 不需要任何参数和权限，直接调用底层的 `FfiBluetoothBleCreateGattServer` 函数获取服务器句柄。创建成功后，开发者可以通过返回的 `GattServer` 对象添加服务、发布广播等。

`createGattClientDevice` 函数根据设备 ID 创建一个 GATT 客户端设备实例。设备 ID 参数需要符合 MAC 地址格式，例如 "11:22:33:AA:BB:FF"。如果格式不正确或底层创建失败，函数将抛出 `BusinessException` 异常，错误码为 2900099。该函数底层调用 `FfiBluetoothBleCreateGattClientDevice` FFI 接口。

`startBleScanning` 函数启动 BLE 设备扫描，支持通过 `ScanFilter` 数组进行设备过滤，可选的 `ScanOptions` 参数用于配置扫描的间隔时间、扫描模式和 PHY 类型。如果不传入过滤条件，函数将发现周围所有可用的 BLE 设备。该函数需要 `ohos.permission.ACCESS_BLUETOOTH` 权限，否则将返回错误码 201。常见的失败原因包括蓝牙服务停止（2900001）、蓝牙禁用（2900003）以及操作本身失败（2900099）。

`stopBleScanning` 函数停止正在进行的 BLE 扫描，与 `startBleScanning` 成对使用。该函数同样需要 `ohos.permission.ACCESS_BLUETOOTH` 权限。

`startAdvertising` 函数启动 BLE 广播，支持两种重载形式。第一种形式接受 `AdvertiseSetting`、`AdvertiseData` 和可选的 `ScanResponse` 三个参数，适用于需要发送完整广播数据和扫描响应的场景。第二种形式接受 `AdvertisingParams` 单一参数，内部会自动提取设置和数据信息，并返回一个 `advertisingId`，该 ID 可用于后续的广播控制。广播数据长度存在上限限制（31 字节基础长度加扩展），超过限制时函数将返回错误码 2902054。当广播资源达到上限时返回 2900010 错误码。

`stopAdvertising` 函数同样提供两种重载形式：无参版本停止当前所有广播，带参版本根据 `advertisingId` 停止指定的广播。如果传入无效的广告 ID，函数将返回 2902055 错误码。

### 回调事件注册

BLE 模块通过 `on` 和 `off` 函数实现回调事件的注册和注销。`on` 函数用于订阅特定的蓝牙事件，支持两种回调类型：`AdvertisingStateChange` 事件监听广播状态变化，`BleDeviceFind` 事件监听扫描到的设备结果。回调参数类型为 `Callback1Argument<T>`，其中 T 分别是 `AdvertisingStateChangeInfo` 和 `Array<ScanResult>`。

`off` 函数用于取消事件订阅，可以指定取消特定的回调对象，也可以不指定参数而清除该事件类型下的所有回调。建议在不需要监听事件时及时调用 `off` 取消订阅，避免资源泄漏。

### BLE 错误码详解

BLE 模块的错误码体系遵循统一的规范。错误码 201 表示权限被拒绝，应用需要在 `module.json5` 中声明 `ohos.permission.ACCESS_BLUETOOTH` 权限。错误码 801 表示系统能力不支持，需要检查设备是否具备 `SystemCapability.Communication.Bluetooth.Core` 能力。错误码 2900001 表示蓝牙服务已停止，可能需要等待服务重启。错误码 2900003 表示蓝牙已禁用，需要引导用户开启蓝牙。错误码 2902054 和 2902055 分别表示广播数据过长或无效的广告 ID，需要检查参数的有效性。

### 文件位置与绑定信息

BLE 模块的核心代码位于 `ohos/bluetooth/ble/` 目录下。`ble.cj` 文件定义了主要的 API 接口，`gatt_client_device.cj` 和 `gatt_server.cj` 分别实现了 GATT 客户端和服务器的封装类。`native.cj` 文件包含了所有 FFI 接口的声明和调用逻辑。

## A2DP 音频分发配置

### Profile 创建与实例

A2DP 模块提供了音频分发配置文件的完整接口，主要通过 `A2dpSourceProfile` 类实现。`createA2dpSrcProfile` 函数是该模块的入口函数，用于创建并返回一个 `A2dpSourceProfile` 实例。该实例继承自 `BaseProfile` 接口，提供了 Profile 共有的方法以及 A2DP 特有的音频状态查询功能。

`A2dpSourceProfile` 类的构造函数为 protected 修饰，只能通过 `createA2dpSrcProfile` 工厂函数创建实例，这种设计确保了 Profile 实例创建的统一性。类的内部维护了一个 `CallbackController` 对象用于管理回调注册。

### 设备状态查询

`getPlayingState` 函数用于查询指定设备的播放状态，参数为设备 MAC 地址。函数返回 `PlayingState` 枚举值，包括 `StateNotPlaying`（未播放）和 `StatePlaying`（播放中）两种状态。如果设备 ID 格式错误或底层查询失败，函数将抛出 `BusinessException` 异常，可能的错误码包括 201（权限拒绝）、801（能力不支持）、2900003（蓝牙禁用）、2900004（Profile 不支持）和 2900099（操作失败）。

`getConnectedDevices` 函数返回当前已连接到 A2DP Profile 的设备列表，返回值为 `Array<String>`，包含所有已连接设备的 MAC 地址。该函数同样需要 `ohos.permission.ACCESS_BLUETOOTH` 权限。

`getConnectionState` 函数查询指定设备的 A2DP 连接状态，参数为设备 ID，返回 `ProfileConnectionState` 枚举值。枚举值包括 `StateDisconnected`（已断开）、`StateConnecting`（连接中）、`StateConnected`（已连接）和 `StateDisconnecting`（断开中）四种状态。

### 回调事件处理

A2DP 模块通过 `on` 和 `off` 函数实现连接状态变化的回调监听。`on` 函数订阅 `ConnectionStateChange` 事件，当设备的 A2DP 连接状态发生变化时触发回调，回调参数为 `StateChangeParam` 对象，包含设备 ID、状态值和断开原因等信息。`off` 函数提供两种重载形式：带回调参数版本移除指定的回调，不带参数版本清除所有回调。

### 编码信息相关类型

A2DP 模块定义了丰富的编码信息相关类型，用于描述音频编码的各项参数。`CodecInfo` 类封装了编码类型、采样位深、声道模式、采样率、码率和帧长度等信息。`CodecType` 枚举定义了支持的编码类型，包括 SBC（子带编码）、AAC（高级音频编码）、L2HC、L2HCST 和 LDAC 等。`CodecSampleRate` 枚举定义了支持的采样率，从 44.1kHz 到 192kHz 覆盖了常见的音频采样规格。`CodecChannelMode` 枚举定义了声道模式，包括单声道（Mono）和立体声（Stereo）。`CodecBitsPerSample` 枚举定义了采样位深，支持 16bit、24bit 和 32bit 等规格。

### A2DP 文件位置

A2DP 模块的代码位于 `ohos/bluetooth/a2dp/` 目录下，`a2dp.cj` 文件定义了主要的 API 和类型，`native.cj` 文件包含 FFI 接口声明。

## HFP 免提配置

### Profile 创建与实例

HFP 模块提供了免提配置文件的完整接口，主要通过 `HandsFreeAudioGatewayProfile` 类实现。与 A2DP 模块类似，HFP 模块使用工厂函数 `createHfpAgProfile` 创建 Profile 实例，构造函数同样为 protected 修饰。`HandsFreeAudioGatewayProfile` 类继承自 `BaseProfile` 接口。

### 设备状态查询

`getConnectedDevices` 函数返回当前已连接到 HFP Profile 的设备列表，返回 `Array<String>` 类型，包含所有已连接设备的 MAC 地址。函数内部通过 `FfiBluetoothHfpGetConnectedDevices` FFI 接口获取设备列表，并使用 `cArr2cjArr` 辅助函数将 C 字符串数组转换为 Cangjie 字符串数组。

`getConnectionState` 函数查询指定设备的 HFP 连接状态，参数为设备 ID，返回 `ProfileConnectionState` 枚举值。函数内部将底层返回的整型状态值通过 `ProfileConnectionState.parse` 方法转换为枚举值。如果底层调用失败，函数通过 `checkRet` 检查并抛出异常。

### 回调事件处理

HFP 模块的回调处理机制与 A2DP 模块完全一致。通过 `on` 函数订阅 `ConnectionStateChange` 事件，通过 `off` 函数注销回调。回调参数类型为 `StateChangeParam`，包含设备 ID、连接状态和断开原因等信息。`off` 函数在参数校验时会检查事件类型是否为 `ConnectionStateChange`，否则抛出参数错误异常（401）。

### HFP 文件位置

HFP 模块的代码位于 `ohos/bluetooth/hfp/` 目录下，`hands_free_audio_gateway_profile.cj` 文件包含了完整的 Profile 实现。

## BaseProfile 基础框架

### 接口定义

`BaseProfile` 接口定义了所有蓝牙 Profile 的公共接口规范，是 A2DP、HFP 等具体 Profile 类型的基接口。该接口定义了四个核心方法：获取已连接设备列表、获取指定设备的连接状态、订阅连接状态变化事件、取消事件订阅。

`getConnectedDevices` 方法返回当前 Profile 下所有已连接设备的 MAC 地址数组。`getConnectionState` 方法根据设备 ID 查询该设备的 Profile 连接状态。`on` 方法用于注册连接状态变化的回调监听，`off` 方法提供带回调和不带回调两种重载形式用于注销监听。

### 状态变化参数

`StateChangeParam` 类封装了 Profile 状态变化事件的相关参数，包括设备 ID（`deviceId`）、状态值（`state`）和断开原因（`cause`）。断开原因由 `DisconnectCause` 枚举定义，包括用户主动断开（`UserDisconnect`）、需要从键盘端发起连接（`ConnectShouldFromKeyboard`）、需要从鼠标端发起连接（`ConnectShouldFromMouse`）、需要从车载端发起连接（`ConnectShouldFromCar`）、连接设备过多（`TooManyConnectedDevices`）和内部连接失败（`ConnectInternalFail`）等多种情况。

### 回调类型枚举

`ProfileCallbackType` 枚举定义了 Profile 相关的回调事件类型，目前仅包含 `ConnectionStateChange`（连接状态变化）一种类型。枚举提供 `getValue` 方法将枚举值转换为底层 FFI 接口所需的整型标识。

## 连接管理

### 连接管理模块概述

蓝牙连接管理模块位于 `ohos/bluetooth/connection/` 目录下，提供了蓝牙设备配对与连接状态管理的基础功能。该模块的入口文件为 `connection.cj`，FFI 绑定实现位于 `native.cj`。模块依赖 `bluetooth:cj_bluetooth_connection_ffi` 外部组件提供的 FFI 接口。

### 蓝牙常量模块

`ohos.bluetooth.constant` 模块定义了蓝牙相关的枚举常量，其中最重要的是 `ProfileConnectionState` 枚举。该枚举定义了 Profile 连接状态的四种可能值：已断开（`StateDisconnected`）、连接中（`StateConnecting`）、已连接（`StateConnected`）和断开中（`StateDisconnecting`）。枚举提供了 `parse` 静态方法将整型值转换为枚举实例。

## WLAN P2P

### WiFi 状态查询

WLAN P2P 模块通过 `ohos.wifi_manager` 包提供点对点无线连接功能。`isWifiActive` 函数查询当前 WiFi 是否处于激活状态，返回布尔值。该函数不需要特殊权限，但如果 WiFi 操作失败将抛出 `BusinessException` 异常，错误码为 2501000。

### 扫描与发现

`getScanInfoList` 函数获取当前扫描到的 WiFi 热点列表，返回 `Array<WifiScanInfo>` 类型。函数需要 `ohos.permission.GET_WIFI_INFO` 权限，无权限时将返回随机 BSSID 以保护用户隐私。如果扫描失败或权限不足，函数将抛出相应异常。常见的失败原因包括权限拒绝（201）、能力不支持（801）和操作失败（2501000）。

`startDiscoverDevices` 和 `stopDiscoverDevices` 函数分别用于启动和停止 P2P 设备发现。这两个函数都需要 `ohos.permission.GET_WIFI_INFO` 权限。发现过程是异步的，需要通过回调机制获取发现的设备列表。

### P2P 连接控制

`p2pConnect` 函数发起 P2P 连接到指定配置的设备，参数为 `WifiP2pConfig` 对象，包含目标设备的配置信息。函数需要 WiFi STA 模式已启用，否则返回 2801001 错误码。连接失败可能返回 2801000 错误码。

`p2pCancelConnect` 函数取消正在建立中的 P2P 连接。该函数同样是异步操作，需要通过回调或状态检查确认取消结果。

### 回调事件处理

WLAN 模块通过 `on` 和 `off` 函数管理回调事件。目前支持的回调类型为 `WifiScanStateChange`，用于监听 WiFi 扫描状态变化。回调参数类型为 `Callback1Argument<Int32>`，状态码的含义需要参考底层实现文档。

### WLAN 文件位置

WLAN 模块的核心代码位于 `ohos/wifi_manager/` 目录下，`wifi.cj` 文件定义了主要的 API 接口，`wifi_p2p_config.cj` 文件定义了 P2P 连接配置类，`wifi_scan_info.cj` 文件定义了扫描结果信息类。

## Kit 层统一导出

### ConnectivityKit 导出清单

`kit/ConnectivityKit/index.cj` 文件作为 Kit 层统一出口，通过 `public import` 语句导出所有蓝牙和 WLAN 模块的公共接口。导出的模块包括 `ohos.bluetooth.*`（所有蓝牙类型）、`ohos.bluetooth.base_profile.*`（Profile 框架）、`ohos.bluetooth.ble.*`（BLE 功能）、`ohos.bluetooth.constant.*`（常量定义）、`ohos.bluetooth.connection.*`（连接管理）、`ohos.wifi_manager.*`（WiFi 功能）、`ohos.bluetooth.a2dp.*`（A2DP 功能）和 `ohos.bluetooth.hfp.*`（HFP 功能）。

开发者只需要导入 `kit.ConnectivityKit` 即可使用所有 Connectivity 相关的 Cangjie API，无需分别导入各个子模块。

## 通用错误处理模式

### 同步 API 错误处理

项目采用统一的错误处理模式：所有 FFI 调用返回错误码，通过 `checkRet` 函数检查并抛出异常。典型的实现模式如下：首先调用 FFI 函数获取错误码，然后调用 `checkRet(errorCode)` 进行检查。如果错误码为 0，表示操作成功，正常返回结果；如果错误码非零，`checkRet` 函数会抛出 `BusinessException` 异常，包含错误码和错误消息。

### 回调异步错误处理

对于回调相关的 API，错误处理通常在注册阶段进行。`on` 函数内部调用 FFI 接口注册回调，如果注册失败会立即抛出异常。回调触发时的错误由底层服务处理，回调参数中仅传递有效数据。

### 内存安全管理

项目在 FFI 调用中广泛使用 `unsafe` 代码块处理原生内存操作。使用 `LibC.mallocCString` 将 Cangjie 字符串转换为 C 字符串，使用 `safeMalloc` 分配原生结构体内存。资源管理通过 `asResource()` 和 `try` 语句确保资源正确释放，避免内存泄漏。

## 相关文档

本文档应与以下文档配合阅读，以获得完整的技术理解。架构说明文档提供了组件关系与数据流的整体视图，内部 API 文档描述了模块间的依赖方向和接口稳定性，构建系统文档解释了 GN Targets 与编译产物的关系，安全评审文档提供了安全使用指南，FAQ 与排错文档则涵盖了常见问题的解决方案。
