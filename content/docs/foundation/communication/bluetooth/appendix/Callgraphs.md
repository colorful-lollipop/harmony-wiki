# 关键调用链

> **目的**: 梳理 Bluetooth 模块的关键调用路径，便于问题定位与代码理解  
> **适用范围**: 调试、架构学习、代码审查

## 目录

- [蓝牙开关流程](#蓝牙开关流程)
- [BLE 扫描流程](#ble-扫描流程)
- [BLE 广播流程](#ble-广播流程)
- [GATT 连接流程](#gatt-连接流程)
- [配对流程](#配对流程)

---

## 蓝牙开关流程

### 开启经典蓝牙

```mermaid
sequenceDiagram
    participant App as JS 应用
    participant NAPI as N-API
    participant Host as BluetoothHost
    participant Proxy as HostProxy
    participant SAMGR as SAMGR
    participant Svc as Bluetooth Service

    App->>NAPI: enableBluetooth()
    NAPI->>NAPI: 参数校验
    NAPI->>Host: EnableBt()
    Host->>Host: LoadBluetoothHostService()
    Host->>SAMGR: GetSystemAbilityManager()
    SAMGR-->>Host: 返回 SAMGR Proxy
    Host->>Proxy: EnableBt()
    Proxy->>Proxy: WriteInterfaceToken()
    Proxy->>Proxy: 序列化参数
    Proxy->>SAMGR: SendRequest(BT_ENABLE)
    SAMGR->>Svc: 路由请求
    Svc-->>Proxy: 返回结果
    Proxy-->>Host: 返回结果
    Host-->>NAPI: 返回结果
    NAPI-->>App: Promise resolve
```

**关键代码路径**:
```
App
  ↓
native_module.cpp:75 (BluetoothHostInit)
  ↓
bluetooth_host.h (BluetoothHost::EnableBt)
  ↓
bluetooth_host.cpp:720
  ↓
bluetooth_host_proxy.cpp:73 (BluetoothHostProxy::EnableBt)
  ↓
samgr (IPC)
  ↓
Bluetooth Service (SA 1130)
```

### 开启 BLE

```mermaid
sequenceDiagram
    participant App as JS 应用
    participant NAPI as N-API
    participant Host as BluetoothHost
    participant Proxy as HostProxy
    participant SAMGR as SAMGR
    participant Svc as Bluetooth Service

    App->>NAPI: enableBLE()
    NAPI->>Host: EnableBle()
    Host->>Proxy: EnableBle()
    Proxy->>SAMGR: SendRequest(BT_ENABLE_BLE)
    SAMGR->>Svc: 路由请求
    Svc-->>Proxy: 返回结果
    Proxy-->>Host: 返回结果
    Host-->>NAPI: 返回结果
    NAPI-->>App: Promise resolve
```

**关键代码路径**:
```
App
  ↓
native_module.cpp:75
  ↓
bluetooth_host.h (BluetoothHost::EnableBle)
  ↓
bluetooth_host.cpp:883
  ↓
bluetooth_host_proxy.cpp:276 (BluetoothHostProxy::EnableBle)
  ↓
samgr (IPC)
  ↓
Bluetooth Service
```

---

## BLE 扫描流程

```mermaid
sequenceDiagram
    participant App as JS 应用
    participant NAPI as N-API
    participant Central as BleCentralManager
    participant Proxy as CentralProxy
    participant SAMGR as SAMGR
    participant Svc as Bluetooth Service
    participant Remote as BLE 设备

    App->>NAPI: startBLEScan(params)
    NAPI->>Central: StartScan(params)
    Central->>Proxy: StartScan()
    Proxy->>SAMGR: SendRequest(BLE_START_SCAN)
    SAMGR->>Svc: 路由请求
    Svc->>Remote: HCI 扫描请求
    Remote-->>Svc: 扫描响应
    Svc->>Proxy: OnScanResult()
    Proxy->>Callback: 回调通知
    Callback->>NAPI: on('scanResult')
    NAPI-->>App: 事件通知
```

**关键代码路径**:
```
App
  ↓
native_module_ble.cpp (DefineBLEJSObject)
  ↓
napi_bluetooth_ble.cpp
  ↓
bluetooth_ble_central_manager.h (BleCentralManager::StartScan)
  ↓
bluetooth_ble_central_manager.cpp
  ↓
bluetooth_ble_central_manager_proxy.cpp (BleCentralManagerProxy::StartScan)
  ↓
bluetooth_service_ipc_interface_code.h (BLE_START_SCAN = 65)
  ↓
samgr (IPC)
  ↓
Bluetooth Service
```

---

## BLE 广播流程

```mermaid
sequenceDiagram
    participant App as JS 应用
    participant NAPI as N-API
    participant Advertiser as BleAdvertiser
    participant Proxy as AdvertiserProxy
    participant SAMGR as SAMGR
    participant Svc as Bluetooth Service
    participant Remote as BLE 设备

    App->>NAPI: startAdvertising(params)
    NAPI->>Advertiser: StartAdvertising(params)
    Advertiser->>Proxy: StartAdvertising()
    Proxy->>SAMGR: SendRequest(BLE_START_ADVERTISING)
    SAMGR->>Svc: 路由请求
    Svc->>Remote: HCI 广播
    Svc-->>Proxy: 返回结果
    Proxy-->>Advertiser: 返回结果
    Advertiser-->>NAPI: 返回结果
    NAPI-->>App: Callback
```

**关键代码路径**:
```
App
  ↓
native_module_ble.cpp
  ↓
napi_bluetooth_ble.cpp (DefineBLEJSObject)
  ↓
bluetooth_ble_advertiser.h (BleAdvertiser::StartAdvertising)
  ↓
bluetooth_ble_advertiser.cpp
  ↓
bluetooth_ble_advertiser_proxy.cpp
  ↓
bluetooth_service_ipc_interface_code.h (BLE_START_ADVERTISING = 40)
  ↓
samgr (IPC)
  ↓
Bluetooth Service
```

---

## GATT 连接流程

```mermaid
sequenceDiagram
    participant App as JS 应用
    participant NAPI as N-API
    participant Client as GattClient
    participant Proxy as ClientProxy
    participant SAMGR as SAMGR
    participant Svc as Gatt Service
    participant Remote as 远程设备

    App->>NAPI: gattClient.connect(deviceId)
    NAPI->>Client: Connect(deviceId)
    Client->>Proxy: Connect()
    Proxy->>SAMGR: SendRequest(BT_GATT_CLIENT_CONNECT)
    SAMGR->>Svc: 路由请求
    Svc->>Remote: 建立 GATT 连接
    Remote-->>Svc: 连接完成
    Svc->>Proxy: OnConnectionStateChange()
    Proxy->>Callback: 回调
    Callback->>NAPI: 状态变更事件
    NAPI-->>App: on('connectionStateChange')
```

**关键代码路径**:
```
App
  ↓
native_module_ble.cpp (NapiGattClient::DefineGattClientJSClass)
  ↓
napi_bluetooth_gatt_client.cpp
  ↓
bluetooth_gatt_client.h (GattClient::Connect)
  ↓
bluetooth_gatt_client.cpp
  ↓
bluetooth_gatt_client_proxy.cpp
  ↓
bluetooth_service_ipc_interface_code.h (BT_GATT_CLIENT_CONNECT = 0)
  ↓
samgr (IPC)
  ↓
Bluetooth Service (GATT Profile)
```

---

## 配对流程

```mermaid
sequenceDiagram
    participant App as JS 应用
    participant NAPI as N-API
    participant Host as BluetoothHost
    participant Proxy as HostProxy
    participant SAMGR as SAMGR
    participant Svc as Bluetooth Service
    participant Remote as 远程设备

    App->>NAPI: createBond(deviceId)
    NAPI->>Host: CreateBond(deviceId, transport)
    Host->>Proxy: CreateBond()
    Proxy->>SAMGR: SendRequest(BT_CREATE_BOND)
    SAMGR->>Svc: 路由请求
    Svc->>Remote: 发起配对请求
    Remote-->>Svc: 配对响应
    Svc-->>Proxy: 配对结果
    Proxy->>Host: OnPairRequested/OnPairConfirmed
    Host->>Observer: 回调
    Observer->>NAPI: 配对事件
    NAPI-->>App: on('bondStateChange')
```

**关键代码路径**:
```
App
  ↓
native_module.cpp
  ↓
bluetooth_host.h (BluetoothHost::CreateBond)
  ↓
bluetooth_host.cpp
  ↓
bluetooth_host_proxy.cpp
  ↓
bluetooth_service_ipc_interface_code.h (BT_CREATE_BOND = ...)
  ↓
samgr (IPC)
  ↓
Bluetooth Service
```

---

## Profile 获取流程

```mermaid
sequenceDiagram
    participant App as JS 应用
    participant NAPI as N-API
    participant Manager as ProfileManager
    participant Host as HostProxy
    participant SAMGR as SAMGR
    participant Svc as Bluetooth Service

    App->>NAPI: getProfile(PROFILE_A2DP)
    NAPI->>Manager: GetProfile(PROFILE_A2DP)
    Manager->>Host: GetProfile(A2DP_PROFILE)
    Host->>SAMGR: SendRequest(BT_GETPROFILE)
    SAMGR->>Svc: 路由请求
    Svc-->>Host: 返回 A2DP Remote Object
    Host-->>Manager: 返回 Profile Proxy
    Manager-->>NAPI: 返回 Profile 实例
    NAPI-->>App: 返回 Profile API
```

**关键代码路径**:
```
App
  ↓
native_module.cpp
  ↓
bluetooth_profile_manager.h (BluetoothProfileManager::GetProfileRemote)
  ↓
bluetooth_profile_manager.cpp
  ↓
bluetooth_host_proxy.cpp:136 (BluetoothHostProxy::GetProfile)
  ↓
samgr (IPC)
  ↓
Bluetooth Service
```

---

## 回调注册流程

```mermaid
sequenceDiagram
    participant App as JS 应用
    participant NAPI as N-API
    participant Observer as ObserverStub
    participant SAMGR as SAMGR
    participant Svc as Bluetooth Service

    App->>NAPI: on('BLEDeviceFind', callback)
    NAPI->>NAPI: 创建 JS 回调
    NAPI->>Observer: RegisterCallback()
    Observer->>SAMGR: IPC 注册
    SAMGR->>Svc: 转发注册
    Svc-->>App: 触发事件
    
    loop 事件循环
        Svc->>SAMGR: 发送事件
        SAMGR->>Observer: 转发事件
        Observer->>NAPI: 回调
        NAPI->>App: JS 回调执行
    end
```

---

[返回 SUMMARY](../SUMMARY.md)
