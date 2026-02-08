# 关键调用链

## 文档信息

- **目的**: 追踪关键功能的完整调用路径
- **适用范围**: 深入理解代码流程、调试复杂问题
- **限制条件**: 基于代码静态分析，实际执行可能因条件分支不同
- **相关文档**: [02_Architecture](02_Architecture.md), [03_Internal_API](03_Internal_API.md)

---

## 启动流程

### 完整调用链

```
main() (系统)
  └─> SystemAbilityProxy::AddSystemAbility(SA ID: 1130)
      └─> BluetoothHostServer::OnStart()
          └─> BluetoothHostServer::Init()
              └─> IAdapterManager::GetInstance()
                  └─> AdapterManager::Start()
                      ├─> AdapterManager::RegisterSystemStateObserver()
                      ├─> AdapterManager::RegisterStateObserver()
                      ├─> IAdapterClassic::Enable()
                      │   └─> ClassicAdapter::Enable()
                      │       └─> Gap::Enable()
                      │           └─> Hci::SendEnableCommand()
                      │               └─> BluetoothHdi::Enable()
                      │                   └─> [HDI 调用 HAL]
                      └─> IAdapterBle::Enable()
                          └─> BleAdapter::Enable()
                              └─> Gap::Enable()
                                  └─> [同 Classic 流程]
          └─> Publish(SA)
              └─> SAMGR::AddSystemAbility()
```

**代码证据**:
- `services/bluetooth/server/include/bluetooth_host_server.h:42` - OnStart
- `services/bluetooth/service/src/common/adapter_manager.cpp` - AdapterManager
- `services/bluetooth/service/src/classic/classic_adapter.cpp` - ClassicAdapter

---

## 启用蓝牙流程

### Classic Bluetooth

```
应用 IPC 请求
  └─> BluetoothHostServer::EnableBt(transport: BREDR)
      └─> PermissionManager::GetCallingName()  [权限检查]
          └─> IAdapterManager::Enable(BT_TRANSPORT_BREDR)
              └─> AdapterManager::Enable(transport: BREDR)
                  └─> IAdapterClassic::Enable()
                      └─> ClassicAdapter::Enable()
                          └─> StateMachine::MoveTo(STATE_TURNING_ON)
                              └─> Gap::Enable()
                                  └─> Hci::SendEnableCommand()
                                      └─> BluetoothHdi::Enable()
                                          └─> HDI Interface::Enable()
                                              └─> [HAL 层]
                                                      └─> [硬件]
```

**代码证据**:
- `services/bluetooth/server/include/bluetooth_host_server.h:47` - EnableBt
- `services/bluetooth/service/src/classic/classic_adapter.cpp` - Enable
- `services/bluetooth/service/src/gap/` - GAP

### BLE

```
应用 IPC 请求
  └─> BluetoothHostServer::EnableBle(noAutoConnect, isAsync, callingName)
      └─> PermissionManager::GetCallingName()
          └─> IAdapterManager::Enable(BT_TRANSPORT_BLE)
              └─> AdapterManager::Enable(transport: BLE)
                  └─> IAdapterBle::Enable()
                      └─> BleAdapter::Enable()
                          └─> StateMachine::MoveTo(STATE_TURNING_ON)
                              └─> Gap::EnableBle()
                                  └─> Hci::SendEnableCommand()
                                      └─> BluetoothHdi::Enable()
                                          └─> [HAL/硬件]
```

**代码证据**:
- `services/bluetooth/server/include/bluetooth_host_server.h:66` - EnableBle
- `services/bluetooth/service/src/ble/ble_adapter.cpp` - Enable

---

## 设备发现流程

### Classic Discovery

```
应用 IPC 请求
  └─> BluetoothHostServer::StartBtDiscovery()
      └─> PermissionManager::GetCallingName()
          └─> IAdapterManager::StartDiscovery()
              └─> IAdapterClassic::StartDiscovery()
                  └─> ClassicAdapter::StartDiscovery()
                      └─> Gap::StartInquiry()
                          └─> Hci::SendInquiryCommand()
                              └─> BluetoothHdi::SendCommand()
                                  └─> [硬件开始 Inquiry]
                                          └─> [HCI Event: Inquiry Result]
                                              └─> Gap::OnInquiryResult()
                                                  └─> Adapter::OnDeviceFound()
                                                      └─> IPC Callback
                                                          └─> 应用收到 OnDeviceFound()
```

**代码证据**:
- `services/bluetooth/server/include/bluetooth_host_server.h:82` - StartBtDiscovery
- `services/bluetooth/service/src/classic/classic_adapter.cpp` - StartDiscovery

### BLE Scan

```
应用 IPC 请求
  └─> IAdapterBle::StartScan()
      └─> BleAdapter::StartScan()
          └─> Gap::StartScan()
              └─> Hci::SendScanCommand()
                  └─> BluetoothHdi::SendCommand()
                      └─> [硬件开始扫描]
                              └─> [HCI Event: Advertising Report]
                                  └─> Gap::OnScanResult()
                                      └─> BleAdapter::OnDeviceFound()
                                          └─> IPC Callback
                                              └─> 应用收到 OnScanResult()
```

**代码证据**:
- `services/bluetooth/service/src/ble/ble_adapter.cpp` - StartScan

---

## 配对流程

### Classic Pairing

```
应用 IPC 请求: StartPair(transport, address)
  └─> BluetoothHostServer::StartPair()
      └─> PermissionManager::GetCallingName()
          └─> IAdapterClassic::StartPair()
              └─> ClassicAdapter::StartPair()
                  └─> Sdp::DiscoverServices(address)
                      └─> [获取设备服务信息]
                  └─> [显示配对对话框]
                      └─> DialogManager::ShowPairDialog()
                  └─> 用户确认后
                      └─> Gap::CreateConnection(address)
                          └─> Hci::SendConnectCommand()
                              └─> BluetoothHdi::Connect()
                                  └─> [硬件建立连接]
                                          └─> [HCI Event: Connect Complete]
                                              └─> [开始配对流程]
                                                  └─> Gap::Authenticate()
                                                      └─> Smp::Pairing() [Classic]
                                                      └─> [配对完成]
                                                          └─> Adapter::OnPairStateChanged()
                                                              └─> IPC Callback
                                                                  └─> 应用收到 OnPairResult()
```

**代码证据**:
- `services/bluetooth/server/include/bluetooth_host_server.h:103` - StartPair
- `services/bluetooth/service/src/classic/classic_adapter.cpp` - StartPair
- `services/bluetooth/service/src/sdp/` - SDP
- `services/bluetooth/service/src/dialog/` - Dialog

### BLE Pairing (SMP)

```
应用 IPC 请求: StartPair(transport: BLE, address)
  └─> BluetoothHostServer::StartPair()
      └─> IAdapterBle::StartPair()
          └─> BleAdapter::StartPair()
              └─> Gap::Connect(address)
                  └─> Hci::SendLeConnectCommand()
                      └─> BluetoothHdi::Connect()
                          └─> [硬件建立 LE 连接]
                                  └─> [HCI Event: Connect Complete]
                                      └─> [开始 SMP 配对]
                                          └─> Smp::PairingRequest()
                                              └─> Hci::SendPairingRequest()
                                                  └─> BluetoothHdi::SendCommand()
                                                      └─> [设备响应]
                                                          └─> Smp::VerifyPairing()
                                                              └─> [配对完成]
                                                                  └─> Adapter::OnPairStateChanged()
                                                                      └─> IPC Callback
                                                                          └─> 应用收到 OnPairResult()
```

**代码证据**:
- `services/bluetooth/stack/src/smp/` - SMP 实现
- `services/bluetooth/service/src/ble/ble_security.cpp` - BLE 安全

---

## A2DP 连接流程

```
应用 IPC 请求: ConnectA2dp(address)
  └─> BluetoothA2dpSourceServer::Connect()
      └─> PermissionManager::GetCallingName()
          └─> IProfileA2dpSrc::Connect(address)
              └─> A2dpSourceService::Connect(address)
                  └─> Sdp::DiscoverServices(address, A2DP_UUID)
                      └─> [获取 A2DP SDP 记录]
                  └─> A2dpProfile::Connect(address)
                      └─> Avdtp::Connect(address)
                          └─> L2CAP::Connect(address, PSM: AVDTP)
                              └─> Hci::SendL2capConnect()
                                  └─> BluetoothHdi::Connect()
                                      └─> [硬件建立 L2CAP 连接]
                                              └─> [L2CAP Connect Complete]
                                                  └─> Avdtp::SetConfiguration()
                                                      └─> [协商音频配置: 编码器、采样率]
                                                  └─> Avdtp::OpenStream()
                                                      └─> [开始音频流传输]
                                                          └─> A2dpService::OnStateChanged(CONNECTED)
                                                              └─> IPC Callback
                                                                  └─> 应用收到 OnConnectStateChanged()
```

**代码证据**:
- `services/bluetooth/server/include/bluetooth_a2dp_source_server.h`
- `services/bluetooth/service/src/a2dp_src/a2dp_src_service.cpp`
- `services/bluetooth/service/src/gavdp/a2dp_profile.cpp`
- `services/bluetooth/service/src/gavdp/a2dp_avdtp.cpp`
- `services/bluetooth/stack/src/l2cap/` - L2CAP

---

## GATT 连接流程

### GATT Client 连接

```
应用 IPC 请求: ConnectGattClient(address)
  └─> BluetoothGattClientServer::Connect()
      └─> PermissionManager::GetCallingName()
          └─> IGattClientProfile::Connect(address)
              └─> GattClientProfile::Connect(address)
                  └─> GattConnectionManager::Connect(address)
                      └─> Gap::Connect(address)
                          └─> Hci::SendLeConnectCommand()
                              └─> BluetoothHdi::Connect()
                                  └─> [硬件建立 LE 连接]
                                          └─> [Connect Complete]
                                              └─> Att::Connect(address)
                                                  └─> GattClientProfile::OnConnectionStateChanged(CONNECTED)
                                                      └─> [发现服务]
                                                          └─> Att::DiscoverServices()
                                                              └─> [发现 GATT 服务]
                                                                  └─> IPC Callback
                                                                      └─> 应用收到 OnServicesDiscovered()
```

### GATT Characteristic 写入

```
应用 IPC 请求: WriteCharacteristic(handle, value)
  └─> BluetoothGattClientServer::WriteCharacteristic()
      └─> PermissionManager::GetCallingName()
          └─> IGattClientProfile::WriteCharacteristic()
              └─> GattClientService::WriteCharacteristic()
                  └─> Att::WriteRequest(handle, value)
                      └─> L2CAP::SendData()
                          └─> Hci::SendAclData()
                              └─> BluetoothHdi::SendData()
                                  └─> [硬件发送数据]
                                          └─> [设备响应]
                                              └─> Att::OnWriteResponse()
                                                  └─> IPC Callback
                                                      └─> 应用收到 OnWriteResult()
```

**代码证据**:
- `services/bluetooth/server/include/bluetooth_gatt_client_server.h`
- `services/bluetooth/service/src/gatt/gatt_client_service.cpp`
- `services/bluetooth/service/src/gatt/gatt_connection_manager.cpp`
- `services/bluetooth/stack/src/att/` - ATT

---

## 权限检查流程

```
IPC 请求到达
  └─> BluetoothHostServer::XXX()
      └─> PermissionManager::GetCallingName()
          └─> AccessToken::GetNativeTokenId()
              └─> 返回: uid, pid, tokenId
          └─> PermissionManager::IsSystemHap(tokenId)
              └─> AccessToken::GetHapType(tokenId)
                  └─> 判断是否系统应用
          └─> AuthCenter::CheckPermission(uid, requiredPermission)
              └─> AccessToken::VerifyPermission(uid, permission)
                  └─> 返回: 允许/拒绝
          └─> {权限通过}
              └─> 执行业务逻辑
          └─> {权限拒绝}
              └─> 返回错误: BT_ERR_PERMISSION_DENIED
                  └─> HiSysEvent: PERMISSION_DENIED
```

**代码证据**:
- `services/bluetooth/service/src/permission/permission_manager.cpp:27` - GetCallingName
- `services/bluetooth/service/src/permission/auth_center.cpp` - CheckPermission
- `services/bluetooth/service/src/permission/permission_helper.cpp` - 权限辅助

---

## 数据传输流程

### A2DP 音频数据流

```
应用启动音频播放
  └─> AVSession:StartStream()
      └─> [框架通过 IPC]
          └─> BluetoothA2dpSourceServer::StartStream()
              └─> A2dpSourceService::StartStream()
                  └─> A2dpProfile::StartStream()
                      └─> SbcEncoder::Encode(audioData)
                          └─> [SBC 编码]
                      └─> Avdtp::SendMediaPacket(encodedData)
                          └─> L2CAP::SendData()
                              └─> Hci::SendAclData()
                                  └─> BluetoothHdi::SendData()
                                      └─> [硬件发送到蓝牙设备]
```

**代码证据**:
- `services/bluetooth/service/src/gavdp/a2dp_src/a2dp_src_service.cpp`
- `services/bluetooth/service/src/gavdp/a2dp_codec/sbccodecctrl/src/a2dp_encoder_sbc.cpp`
- `services/bluetooth/service/src/gavdp/a2dp_avdtp.cpp`

---

## 错误处理流程

```
操作失败
  └─> 各层返回错误码
      └─> Service 层: return BT_ERR_XXX
          └─> Server 层: return 同一错误码
              └─> IPC: 通过 MessageParcel 返回
                  └─> 应用收到错误
                      └─> 应用处理错误
                          └─> HiLog: 记录错误
                          └─> HiSysEvent: 上报错误事件（如适用）
```

**错误码来源**:
- `services/bluetooth/service/src/common/` - 错误定义
- `services/bluetooth/ipc/` - IPC 错误传递

---

## 总结

调用链特点：

1. **分层清晰**: Server → Service → Stack → Hardware
2. **权限检查**: 每个 IPC 请求都经过权限验证
3. **状态机**: 使用状态机管理生命周期
4. **异步回调**: 大部分操作通过回调返回结果

**关键路径**:
- 启动: OnStart → AdapterManager → Gap → HCI → Hardware
- 操作: IPC → Permission → Service → Stack → Hardware
- 回调: Hardware → Stack → Service → Server → IPC → 应用

**相关文档**:
- 架构: [02_Architecture](02_Architecture.md)
- 内部 API: [03_Internal_API](03_Internal_API.md)
- 常见问题: [07_Common_Issues](07_Common_Issues.md)
