# 关键调用链

## API 调用调用链

### rotate() 完整调用链

```
JS/ArkTS
    │
    ▼
js_mech_manager.cpp:784 (MechManager::Rotate)
    │
    ├── 参数校验
    │   ├── js_mech_manager.cpp:896 (napi_get_value_double)
    │   └── js_mech_manager.cpp:895 (napi_get_named_property)
    │
    ├── Promise 创建
    │   └── js_mech_manager.cpp:800 (napi_create_promise)
    │
    ▼
js_mech_manager_client.cpp:265 (MechClient::RotateByDegree)
    │
    ├── IPC Token 写入
    │   └── js_mech_manager_client.cpp:280 (MessageParcel::WriteInterfaceToken)
    │
    ├── 参数序列化
    │   └── js_mech_manager_client.cpp:284-318 (WriteInt32, WriteDouble)
    │
    ├── IPC 发送
    │   └── js_mech_manager_client.cpp:327 (IRemoteObject::SendRequest)
    │
    ▼
samgr (SystemAbilityManager)
    │
    ├── SA 8550 查找
    │   └── LoadSystemAbility(8550)
    │
    ▼
mechbody_controller_stub.cpp:90 (MechBodyControllerStub::OnRemoteRequest)
    │
    ├── Token 验证
    │   └── mechbody_controller_stub.cpp:97 (ReadInterfaceToken)
    │
    ├── 命令分发
    │   └── mechbody_controller_stub.cpp:102 (funcsMap_[code])
    │
    ▼
mechbody_controller_stub.cpp:310 (ROTATE_BY_DEGREE handler)
    │
    ├── Parcel 读取
    │   └── mechbody_controller_stub.cpp:320-350 (ReadInt32, ReadParcelable)
    │
    ├── 权限检查
    │   └── mechbody_controller_service.cpp:212 (VerifyAccessToken)
    │
    ▼
mechbody_controller_service.cpp:515 (RotateByDegree)
    │
    ├── 参数校验
    │   ├── mechbody_controller_service.cpp:515 (mechId < 0)
    │   ├── mechbody_controller_service.cpp:519 (null check)
    │   └── mechbody_controller_service.cpp:523 (duration < 0)
    │
    ├── 权限检查
    │   └── mechbody_controller_service.cpp:349 (PERMISSION_NAME)
    │
    ▼
mc_controller_manager.cpp (McControllerManager::HandleRotate)
    │
    ├── 运动规划
    │   └── mc_motion_manager.cpp (McMotionManager::CalculateTrajectory)
    │
    ├── 协议转换
    │   └── mc_protocol_convertor.cpp (McProtocolConvertor::Encode)
    │
    ▼
mc_send_adapter.cpp (McSendAdapter::SendCommand)
    │
    ├── BLE 发送
    │   └── ble_send_manager.cpp (BleSendManager::Send)
    │
    └── 南向协议
        └── libmech_adapter.z.so
```

---

### getAttachedMechDevices() 调用链

```
JS/ArkTS
    │
    ▼
js_mech_manager.cpp:514 (MechManager::GetAttachedDevices)
    │
    ├── 异步回调创建
    │   └── js_mech_manager.cpp:525 (AsyncCallback)
    │
    ▼
js_mech_manager_client.cpp:88 (MechClient::GetAttachedDevices)
    │
    ├── IPC 调用
    │   └── js_mech_manager_client.cpp:107 (SendRequest)
    │
    ▼
mechbody_controller_stub.cpp:175 (GetAttachedDevicesInner)
    │
    ├── Token 验证
    │
    ▼
mechbody_controller_service.cpp:370 (GetAttachedDevices)
    │
    ├── 权限检查
    │   └── VerifyAccessToken
    │
    ├── 获取设备列表
    │   └── mc_connect_manager.cpp (McConnectManager::GetConnectedDevices)
    │
    ▼
mc_connect_manager.cpp
    │
    ├── 蓝牙扫描结果
    │   └── bluetooth_state_adapter.cpp
    │
    └── 设备过滤
        └── MechInfo 过滤
```

---

### setCameraTrackingEnabled() 调用链

```
JS/ArkTS
    │
    ▼
js_mech_manager.cpp:630 (MechManager::SetCameraTrackingEnabled)
    │
    ├── 参数校验 (布尔)
    │   └── js_mech_manager.cpp:647 (napi_get_value_bool)
    │
    └── IPC 调用
        └── js_mech_manager_client.cpp
            │
            ▼
mechbody_controller_service.cpp:635 (SetCameraTrackingEnabled)
    │
    ├── 权限检查
    │   └── VerifyAccessToken
    │
    ├── 系统应用验证
    │   └── TokenIdKit::IsSystemAppByFullTokenID
    │
    ▼
mc_camera_tracking_controller.cpp
    │
    ├── 相机接口调用
    │   └── camera_framework (Camera HDI)
    │
    └── 状态同步
        └── trackingEventCallback_
```

---

## 事件回调调用链

### attachStateChange 事件流

```
机械设备 (BLE Notify)
    │
    │ BLE GATT Notification
    ▼
ble_send_manager.cpp (BleSendManager::OnCharacteristicChanged)
    │
    ├── 数据接收
    │   └── ble_send_manager.cpp:117
    │
    └── 协议解析
        └── mc_protocol_convertor.cpp
            │
            ▼
mc_subscription_center.cpp (McSubscriptionCenter::Publish)
    │
    ├── 事件分发
    │   └── mc_subscription_center.cpp
    │
    └── 回调触发
        └── deviceAttachCallback_[tokenId]
            │
            ▼
mechbody_controller_service.cpp:192
    │
    ├── 构造回调数据
    │   └── MechInfo 序列化
    │
    └── IPC 回调发送
        └── SendRequest(ATTACH_STATE_CHANGE_CALLBACK)
            │
            ▼
js_mech_manager_stub.cpp (AttachStateChangeStub)
    │
    ├── 数据 unmarshalling
    │   └── js_mech_manager_stub.cpp:45
    │
    └── N-API 事件发送
        └── js_mech_manager_service.cpp:52 (napi_send_event)
            │
            ▼
JS 回调函数 (on() 注册)
```

---

### 客户端死亡监听流

```
JS 应用崩溃/退出
    │
    │ IRemoteObject 死亡通知
    ▼
IRemoteObjectDeathRecipient::OnRemoteDied
    │
    ▼
mc_controller_ipc_death_listener.cpp (MechControllerIpcDeathListener)
    │
    ├── 识别对象类型
    │   └── mc_controller_ipc_death_listener.cpp:32
    │
    └── 清理回调映射
        ├── DEVICE_ATTACH_CALLBACK: deviceAttachCallback_.erase(tokenId_)
        ├── TRACKING_EVENT_CALLBACK: trackingEventCallback_.erase(tokenId_)
        ├── ROTATION_AXES_STATUS_CHANGE: rotationAxesStatusChangeCallback_.erase(tokenId_)
        └── COMMAND_CHANNEL: cmdChannels_.erase(tokenId_)
```

---

## 初始化调用链

### 服务启动

```
systemd/init
    │
    │ 读取配置文件
    ▼
etc/init/mechbody.cfg
    │
    ├── 设置 SELinux 上下文
    │   └── u:r:mechbody:s0
    │
    └── 设置权限
        └── 蓝牙/相机/输入注入
            │
            ▼
sa_main (SystemAbility Main)
    │
    │ 加载 SA 8550
    ▼
libmechbody_service.z.so
    │
    ├── dlopen
    │   └── load_mechbody_adapter.cpp
    │
    ├── MechBodyControllerService 构造
    │   └── mechbody_controller_service.cpp:46
    │
    ├── SystemAbility 初始化
    │   └── SystemAbility(MECH_SERVICE_SA_ID, true)
    │
    ├── MakeAndRegisterAbility
    │   └── mechbody_controller_service.cpp:49
    │
    └── OnStart 注册
        ├── McControllerManager::Initialize
        │   ├── McMotionManager::Initialize
        │   ├── McConnectManager::Initialize
        │   └── McCameraTrackingController::Initialize
        │
        └── 蓝牙服务订阅
            └── BluetoothServiceStatusChangeListener
```

---

### N-API 模块初始化

```
ArkTS 运行时加载
    │
    │ dlopen
    ▼
libmechanicmanager_napi.so
    │
    ├── NAPI_MODULE 注册
    │   └── js_mech_manager.cpp:1731
    │
    ├── Init 函数调用
    │   └── js_mech_manager.cpp:1676
    │
    ├── 枚举定义
    │   └── js_mech_manager.cpp:1678-1701
    │
    └── API 导出
        └── js_mech_manager.cpp:1704 (napi_define_properties)
```

---

## 错误处理调用链

### 权限拒绝流程

```
API 调用
    │
    ▼
Permission Check
    │
    ├── IPCSkeleton::GetCallingTokenID()
    │   └── 获取调用者 TokenID
    │
    └── AccessTokenKit::VerifyAccessToken(tokenId, PERMISSION_NAME)
        │
        ├── Permission GRANTED
        │   └── 继续执行
        │
        └── Permission DENIED
            │
            ├── 返回 PERMISSION_DENIED (202)
            │
            ├── 记录 HiSysEvent
            │   └── hisysevent_utils.cpp
            │
            └── 记录 HILOG
                └── HILOGW("Permission denied")
```

---

### 设备离线处理

```
BLE Disconnect
    │
    ▼
BluetoothStateListener::OnConnectionStateChanged
    │
    ├── STATE_DISCONNECTED
    │   │
    │   ├── 更新设备状态
    │   │   └── McConnectManager
    │   │
    │   ├── 发布事件
    │   │   └── McSubscriptionCenter::Publish
    │   │
    │   └── 触发回调
    │       └── ATTACH_STATE_CHANGE_CALLBACK
    │           │
    │           └── JS: on('attachStateChange', callback)
    │               └── AttachState.DETACHED
    │
    └── 清理资源
        └── RemoveDevice(mechId)
```
