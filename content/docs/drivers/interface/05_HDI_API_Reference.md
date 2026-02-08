# N-API/HDI API 参考

## 概述

本文档提供 OpenHarmony HDI 接口的 API 参考，包括核心模块的接口清单、调用方式和参数说明。

**注意**: 本仓库是**接口定义仓库**，实际的 N-API 实现位于 `drivers_peripheral` 仓库。本文档提供 IDL 接口定义的分析和参考。

---

## Audio 模块

### IAudioManager 接口

**文件**: `audio/v1_0/IAudioManager.idl:43-85`

| 方法 | 参数 | 返回值 | 说明 |
|-----|------|--------|------|
| `GetAllAdapters` | `[out] AudioAdapterDescriptor[] descs` | `int32_t` | 获取所有音频适配器 |
| `LoadAdapter` | `[in] AudioAdapterDescriptor desc, [out] IAudioAdapter adapter` | `int32_t` | 加载音频适配器驱动 |
| `UnloadAdapter` | `[in] String adapterName` | `int32_t` | 卸载音频适配器 |
| `ReleaseAudioManagerObject` | 无 | `int32_t` | 释放管理器对象 |

### IAudioRender 接口

**文件**: `audio/v1_0/IAudioRender.idl`

| 方法 | 参数 | 返回值 | 说明 |
|-----|------|--------|------|
| `GetRenderCapability` | `[out] AudioRenderCapability capability` | `int32_t` | 获取渲染能力 |
| `GetRenderPosition` | `[out] uint32_t& frames, [out] AudioTimeStamp time` | `int32_t` | 获取渲染位置 |
| `SetRenderSpeed` | `[in] float speed` | `int32_t` | 设置播放速度 |
| `GetRenderSpeed` | `[out] float speed` | `int32_t` | 获取播放速度 |
| `SetChannelMode` | `[in] AudioChannelMode mode` | `int32_t` | 设置通道模式 |
| `GetChannelMode` | `[out] AudioChannelMode mode` | `int32_t` | 获取通道模式 |

### 调用示例

```cpp
// 获取音频管理器实例
sptr<IAudioManager> manager = IAudioManager::Get();
// 获取适配器列表
std::vector<AudioAdapterDescriptor> adapters;
int32_t ret = manager->GetAllAdapters(adapters);
// 加载适配器
sptr<IAudioAdapter> adapter;
ret = manager->LoadAdapter(adapters[0], adapter);
```

---

## Sensor 模块

### ISensorInterface 接口

**文件**: `sensor/v3_0/ISensorInterface.idl:53-175`

| 方法 | 参数 | 返回值 | 说明 |
|-----|------|--------|------|
| `GetAllSensorInfo` | `[out] HdfSensorInformation[] info` | `int32_t` | 获取所有传感器信息 |
| `Enable` | `[in] DeviceSensorInfo deviceSensorInfo` | `int32_t` | 启用传感器 |
| `Disable` | `[in] DeviceSensorInfo deviceSensorInfo` | `int32_t` | 禁用传感器 |
| `SetBatch` | `[in] DeviceSensorInfo, `[in] long samplingInterval, `[in] long reportInterval` | `int32_t` | 设置采样批次 |
| `SetMode` | `[in] int sensorId, `[in] int mode` | `int32_t` | 设置上报模式 |
| `SetOption` | `[in] int sensorId, `[in] unsigned int option` | `int32_t` | 设置选项 |
| `Register` | `[in] int groupId, `[in] ISensorCallback callbackObj` | `int32_t` | 注册回调 |
| `Unregister` | `[in] int groupId, `[in] ISensorCallback callbackObj` | `int32_t` | 注销回调 |
| `RegSensorPlugCallBack` | `[in] ISensorPlugCallback callbackObj` | `int32_t` | 注册插拔回调 |
| `UnRegSensorPlugCallBack` | `[in] ISensorPlugCallback callbackObj` | `int32_t` | 注销插拔回调 |

### ISensorCallback 回调

**文件**: `sensor/v3_0/ISensorCallback.idl`

| 方法 | 参数 | 说明 |
|-----|------|------|
| `OnDataEvent` | `[in] HdfSensorEvents event` | 数据事件（同步） |
| `OnDataEventAsync` | `[in] HdfSensorEvents[] events` | 数据事件（异步，`[oneway]`） |

---

## Camera 模块

### ICameraHost 接口

**文件**: `camera/v1_5/ICameraHost.idl`

| 方法 | 参数 | 返回值 | 说明 |
|-----|------|--------|------|
| `OpenCamera_V1_5` | `[in] String cameraId, `[in] ICameraDeviceCallback callback, `[out] ICameraDevice device` | `int32_t` | 打开相机 V1.5 |
| `OpenSecureCamera_V1_5` | `[in] String cameraId, `[in] ICameraDeviceCallback callback, `[out] ICameraDevice device` | `int32_t` | 打开安全相机 |
| `EntireCloseDevice` | `[in] String cameraId` | `int32_t` | 关闭所有设备 |
| `GetCameraStorageSize` | `[in] int userId, `[out] long storageSize` | `int32_t` | 获取存储大小 |

### IStreamOperator 接口

| 方法 | 参数 | 返回值 | 说明 |
|-----|------|--------|------|
| `Capture` | `[in] int captureId, `[in] CaptureInfo info, `[in] bool isStreaming` | `int32_t` | 捕获图像 |
| `Capture` | `[in] int captureId, `[in] CaptureInfo info` | `int32_t` | 单次捕获 |
| `CancelCapture` | `[in] int captureId` | `int32_t` | 取消捕获 |
| `CheckStreamAllowed` | `[in] int streamId, `[out] bool allowed` | `int32_t` | 检查流是否允许 |

---

## Display 模块

### IDisplayComposer 接口

**文件**: `display/composer/v1_4/IDisplayComposer.idl`

| 方法 | 参数 | 返回值 | 说明 |
|-----|------|--------|------|
| `GetPanelPowerStatus` | `[in] unsigned int devId, `[out] PanelPowerStatus status` | `int32_t` | 获取面板电源状态 |
| `InitSMQInfo` | `[in] unsigned int devId, `[in] SharedMemQueue request, `[out] SharedMemQueue reply` | `int32_t` | 初始化共享内存队列 |
| `SetDisplayColorGamut` | `[in] unsigned int devId, `[in] ColorGamut gamut` | `int32_t` | 设置色域 |
| `GetDisplayConnectionType` | `[in] unsigned int devId, `[out] DisplayConnectionType outType` | `int32_t` | 获取连接类型 |

### IAllocator 接口

**文件**: `display/buffer/v1_0/IAllocator.idl`

| 方法 | 参数 | 返回值 | 说明 |
|-----|------|--------|------|
| `AllocMem` | `[in] AllocInfo info, `[out] BufferHandle& handle` | `int32_t` | 分配内存 |
| `FreeMem` | `[in] BufferHandle handle` | `int32_t` | 释放内存 |

---

## Input 模块

### IInputInterfaces 接口

**文件**: `input/v1_0/IInputInterfaces.idl`

| 方法 | 参数 | 返回值 | 说明 |
|-----|------|--------|------|
| `ScanInputDevice` | `[out] DevDesc[] staArr` | `int32_t` | 扫描输入设备 |
| `OpenInputDevice` | `[in] unsigned int devIndex` | `int32_t` | 打开设备 |
| `CloseInputDevice` | `[in] unsigned int devIndex` | `int32_t` | 关闭设备 |
| `GetInputDevice` | `[in] unsigned int devIndex, `[out] DeviceInfo devInfo` | `int32_t` | 获取设备信息 |
| `SetPowerStatus` | `[in] unsigned int devIndex, `[in] unsigned int status` | `int32_t` | 设置电源状态 |
| `RegisterReportCallback` | `[in] unsigned int devIndex, `[in] IInputCallback callback` | `int32_t` | 注册上报回调 |
| `UnregisterReportCallback` | `[in] unsigned int devIndex` | `int32_t` | 注销上报回调 |
| `RegisterHotPlugCallback` | `[in] IInputCallback callback` | `int32_t` | 注册热插拔回调 |

### IInputCallback 回调

**文件**: `input/v1_0/IInputCallback.idl`

| 方法 | 参数 | 说明 |
|-----|------|------|
| `EventPkgCallback` | `[in] EventPackage[] pkgs, `[in] unsigned int devIndex` | 事件包回调 |
| `HotPlugCallback` | `[in] HotPlugEvent event` | 热插拔事件回调 |

---

## WLAN 模块

### IWlanInterface 接口

**文件**: `wlan/v1_3/IWlanInterface.idl`

| 方法 | 参数 | 返回值 | 说明 |
|-----|------|--------|------|
| `GetFeatureInfo` | `[out] HdfFeatureInfo featureInfo` | `int32_t` | 获取特性信息 |
| `GetApBandwidth` | `[in] String ifName, `[out] unsigned char bandwidth` | `int32_t` | 获取 AP 带宽 |
| `ResetToFactoryMacAddress` | `[in] String ifName` | `int32_t` | 重置 MAC 地址 |
| `SendActionFrame` | `[in] String ifName, `[in] unsigned int freq, `[in] unsigned char[] frameData` | `int32_t` | 发送动作帧 |
| `SetPowerSaveMode` | `[in] String ifName, `[in] int frequency, `[in] int mode` | `int32_t` | 设置省电模式 |

### IWlanCallback 回调

**文件**: `wlan/v1_1/IWlanCallback.idl`

| 方法 | 参数 | 说明 |
|-----|------|------|
| `ResetDriverResult` | `[in] unsigned int event, `[in] int code, `[in] String ifName` | 重置驱动结果 |
| `ScanResult` | `[in] unsigned int event, `[in] HdfWifiScanResult scanResult, `[in] String ifName` | 扫描结果 |
| `WifiNetlinkMessage` | `[in] unsigned char[] recvMsg` | Netlink 消息 |

---

## Battery 模块

### IBatteryInterface 接口

**文件**: `battery/v2_0/IBatteryInterface.idl`

| 方法 | 参数 | 返回值 | 说明 |
|-----|------|--------|------|
| `Register` | `[in] IBatteryCallback event` | `int32_t` | 注册电池回调 |
| `UnRegister` | 无 | `int32_t` | 注销回调 |
| `GetCapacity` | `[out] int capacity` | `int32_t` | 获取电量 |
| `GetVoltage` | `[out] int voltage` | `int32_t` | 获取电压 |
| `GetTemperature` | `[out] int temperature` | `int32_t` | 获取温度 |
| `GetHealthState` | `[out] BatteryHealthState healthState` | `int32_t` | 获取健康状态 |
| `GetPluggedType` | `[out] BatteryPluggedType pluggedType` | `int32_t` | 获取充电类型 |
| `GetChargeState` | `[out] BatteryChargeState chargeState` | `int32_t` | 获取充电状态 |
| `GetPresent` | `[out] boolean present` | `int32_t` | 获取电池存在状态 |
| `GetBatteryInfo` | `[out] BatteryInfo info` | `int32_t` | 获取电池信息 |

### IBatteryCallback 回调

**文件**: `battery/v2_0/IBatteryCallback.idl`

| 方法 | 参数 | 说明 |
|-----|------|------|
| `Update` | `[in] BatteryInfo event` | 电池信息更新 |

---

## HUKS (密钥管理) 模块

### IHuks 接口

**文件**: `huks/v1_1/IHuks.idl`

| 方法 | 参数 | 返回值 | 说明 |
|-----|------|--------|------|
| `Init` | `[in] HuksBlob encKey, `[in] HuksParamSet paramSet, `[out] HuksBlob handle, `[out] HuksBlob token` | `int32_t` | 初始化密钥会话 |
| `Update` | `[in] HuksBlob handle, `[in] HuksParamSet paramSet, `[in] HuksBlob inData, `[out] HuksBlob outData` | `int32_t` | 更新密钥操作 |
| `Finish` | `[in] HuksBlob handle, `[in] HuksParamSet paramSet, `[in] HuksBlob inData, `[out] HuksBlob outData` | `int32_t` | 完成密钥操作 |
| `Abort` | `[in] HuksBlob handle, `[in] HuksParamSet paramSet` | `int32_t` | 中止密钥操作 |
| `ImportKey` | `[in] HuksBlob keyAlias, `[in] HuksParamSet paramSet, `[in] HuksBlob keyData, `[out] HuksBlob keyDigest` | `int32_t` | 导入密钥 |
| `ExportKey` | `[in] HuksBlob keyAlias, `[in] HuksParamSet paramSet, `[out] HuksBlob keyData` | `int32_t` | 导出密钥 |
| `Sign` | `[in] HuksBlob encKey, `[in] HuksParamSet paramSet, `[in] HuksBlob srcData, `[out] HuksBlob signature` | `int32_t` | 签名 |
| `Verify` | `[in] HuksBlob encKey, `[in] HuksParamSet paramSet, `[in] HuksBlob srcData, `[in] HuksBlob signature` | `int32_t` | 验签 |

---

## 错误码规范

HDI 接口普遍使用 `int32_t` 作为返回值：

| 返回值 | 含义 |
|--------|------|
| `0` | 成功 |
| 负值 | 错误码（具体定义见各模块） |
| 正值 | 可能表示部分成功或状态值 |

---

## 服务获取方式

### 标准方式

```cpp
// 单例模式获取
sptr<IModule> module = IModule::Get();
// 带实例名称获取
sptr<IModule> module = IModule::GetInstance("serviceName");
```

### 检查服务是否存在

```cpp
sptr<IModule> module = IModule::Get();
if (module == nullptr) {
    // 服务不存在
    return;
}
// 使用服务
```
