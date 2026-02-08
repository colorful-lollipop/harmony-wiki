# 攻击面分析

## 1. 攻击面总览

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              不可信区域                                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │  恶意JS应用   │  │  伪造IPC调用  │  │  恶意蓝牙设备 │  │  篡改配置文件 │         │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘         │
└─────────┼─────────────────┼─────────────────┼─────────────────┼───────────────────┘
          │                 │                 │                 │
          ▼                 ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              信任边界                                             │
│                                                                                  │
│  ┌──────────────────────────────────────────────────────────────────────────┐   │
│  │                      N-API 层 (interface/napi/)                           │   │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │   │
│  │  │  js_mech_manager.cpp                                              │  │   │
│  │  │  - 18个JS API入口                                                  │  │   │
│  │  │  - 参数解析与验证                                                  │  │   │
│  │  └────────────────────────────────────────────────────────────────────┘  │   │
│  └─────────────────────────────────┬────────────────────────────────────────┘   │
│                                    │                                              │
│  ┌─────────────────────────────────▼────────────────────────────────────────┐   │
│  │                      IPC 层 (services/src/)                               │   │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │   │
│  │  │  mechbody_controller_stub.cpp                                     │  │   │
│  │  │  - IPC命令码分发                                                   │  │   │
│  │  │  - Token验证                                                      │  │   │
│  │  └────────────────────────────────────────────────────────────────────┘  │   │
│  └─────────────────────────────────┬────────────────────────────────────────┘   │
│                                    │                                              │
│  ┌─────────────────────────────────▼────────────────────────────────────────┐   │
│  │                      服务层 (MechBodyControllerService)                   │   │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │   │
│  │  │  mechbody_controller_service.cpp                                  │  │   │
│  │  │  - 权限检查 (VerifyAccessToken)                                    │  │   │
│  │  │  - 系统应用检查 (IsSystemApp)                                      │  │   │
│  │  │  - 业务逻辑处理                                                    │  │   │
│  │  └────────────────────────────────────────────────────────────────────┘  │   │
│  └─────────────────────────────────┬────────────────────────────────────────┘   │
│                                    │                                              │
│  ┌─────────────────────────────────▼────────────────────────────────────────┐   │
│  │                      传输层 (transport/)                                  │   │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │   │
│  │  │  ble_send_manager.cpp                                             │  │   │
│  │  │  - BLE GATT通信                                                    │  │   │
│  │  │  - 数据收发                                                        │  │   │
│  │  └────────────────────────────────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. 外部输入清单

### 2.1 N-API 参数输入

#### 2.1.1 高敏感API（需系统应用权限）

| API | 入口文件 | 关键参数 | 风险点 |
|-----|---------|---------|--------|
| **rotate** | `js_mech_manager.cpp:194` | mechId(int), angles(object), duration(int) | mechId越界、angles字段缺失、duration负数 |
| **rotateToEulerAngles** | `js_mech_manager.cpp:385` | mechId, angles, duration | 同上 |
| **rotateBySpeed** | `js_mech_manager.cpp:458` | mechId, speed(object), duration | speed字段缺失、速度值越界 |
| **setCameraTrackingLayout** | `js_mech_manager.cpp:623` | layout(int) | 范围检查(0-3) |
| **setUserOperation** | `js_mech_manager.cpp:541` | operation(int), mac(string), param(string) | **JSON注入、MAC格式** |
| **stopMoving** | `js_mech_manager.cpp:708` | mechId(int) | 设备ID有效性 |
| **searchTarget** | `js_mech_manager.cpp:1295` | targetType, direction | 枚举值范围 |

**参数验证代码片段** (js_mech_manager.cpp:194-282):
```cpp
// rotate API参数验证
bool MechManager::GetRotateParam(napi_env env, napi_callback_info info,
    RotateByDegreeParam &rotateParam, int32_t &mechId)
{
    // ...
    napi_valuetype type;
    napi_typeof(env, args[0], &type);
    if (type != napi_number) {
        // 类型检查
    }
    napi_get_value_int32(env, args[0], &mechId);
    // mechId在服务端再次验证
}
```

#### 2.1.2 普通API（非系统应用可用）

| API | 入口文件 | 关键参数 | 风险点 |
|-----|---------|---------|--------|
| **on/off** | `js_mech_manager.cpp:56` | eventType(string), callback(function) | eventType长度、回调有效性 |
| **getAttachedDevices** | `js_mech_manager.cpp:472` | 无 | - |
| **getCameraTrackingEnabled** | `js_mech_manager.cpp:482` | 无 | - |
| **getCameraTrackingLayout** | `js_mech_manager.cpp:492` | 无 | - |
| **getRotationAxesStatus** | `js_mech_manager.cpp:1017` | mechId(int) | 设备ID有效性 |

### 2.2 IPC MessageParcel 输入

**入口点**: `mechbody_controller_stub.cpp:90-350`

| IPC命令 | 命令码 | 输入数据 | 风险点 |
|---------|--------|---------|--------|
| SET_USER_OPERATION | 4 | operation(int), mac(string), param(string) | **JSON解析、MAC长度** |
| ROTATE_BY_DEGREE | 11 | mechId(int), cmdId(string), RotateByDegreeParam | 复杂对象反序列化 |
| ROTATE_TO_EULER_ANGLES | 13 | mechId, cmdId, RotateToEulerAnglesParam | 同上 |
| ROTATE_BY_SPEED | 16 | mechId, cmdId, RotateBySpeedParam | 同上 |
| SET_CAMERA_TRACKING_ENABLED | 5 | isEnabled(bool) | 布尔值范围 |
| SET_CAMERA_TRACKING_LAYOUT | 9 | layout(int) | 范围0-3 |
| SEARCH_TARGET | 23 | TargetInfo, SearchParams | 嵌套对象 |

**IPC数据读取代码** (mechbody_controller_stub.cpp:227-240):
```cpp
int32_t MechBodyControllerStub::RotateByDegreeInner(MessageParcel &data, MessageParcel &reply)
{
    int32_t mechId = data.ReadInt32();
    std::string cmdId = data.ReadString();
    std::shared_ptr<RotateByDegreeParam> param(data.ReadParcelable<RotateByDegreeParam>());
    if (param == nullptr) {
        HILOGE("ReadParcelable failed");
        return INVALID_PARAMETERS_ERR;
    }
    // ...
}
```

### 2.3 蓝牙数据输入

**入口点**: `ble_send_manager.cpp:154-180` (OnCharacteristicChanged)

```cpp
void BleGattClientCallback::OnCharacteristicChanged(
    const BluetoothGattCharacteristic &characteristic)
{
    auto data = characteristic.GetValue();
    uint8_t *value = data.get();
    size_t size = data.size();
    
    // 数据流入
    if (bleReceviceListener_ != nullptr) {
        bleReceviceListener_->OnReceive(value, size);  // 外部数据入口
    }
}
```

**协议解析链**:
```
蓝牙数据
    ↓
OnCharacteristicChanged
    ↓
mc_send_adapter.cpp:237 OnReceive
    ↓
mc_protocol_convertor.cpp (协议解析)
    ↓
CommandFactory::CreateFromData (命令创建)
    ↓
具体Command处理
```

**风险点**:
- 数据长度：最大1MB (NOTIFY_DATA_MAX_SIZE = 1024*1024)
- 协议格式：南向协议(0x01/0x02/v1/v2)需要严格解析
- 命令码：111+个命令类，需防止非法命令码

### 2.4 配置文件输入

**etc/init/mechbody.cfg**:
```json
{
  "services": [{
    "name": "mechbody",
    "path": "/system/bin/sa_main",
    "uid": "mechbody",
    "gid": "mechbody",
    "permission": [
      "ohos.permission.ACCESS_BLUETOOTH",
      "ohos.permission.MANAGE_BLUETOOTH",
      "ohos.permission.MANAGE_LOCAL_ACCOUNTS",
      "ohos.permission.GET_BLUETOOTH_LOCAL_MAC",
      "ohos.permission.INJECT_INPUT_EVENT",
      "ohos.permission.CAMERA"
    ]
  }]
}
```

**攻击场景**: 配置文件被篡改 → 服务以错误权限启动

---

## 3. 敏感操作清单

### 3.1 权限操作

| 操作 | 位置 | 权限要求 | 风险 |
|------|------|---------|------|
| VerifyAccessToken | `service.cpp:212` | ohos.permission.CONNECT_MECHANIC_HARDWARE | 权限绕过 |
| IsSystemApp | `service.cpp:1027` | 系统应用验证 | 非系统应用越权 |
| IPC Token验证 | `stub.cpp:97` | MECH_SERVICE_IPC_TOKEN | Token伪造 |

### 3.2 蓝牙操作

| 操作 | 位置 | 风险 |
|------|------|------|
| GattClient::Connect | `ble_send_manager.cpp:697` | 连接恶意设备 |
| GattClient::WriteCharacteristic | `ble_send_manager.cpp:1027` | **命令注入** |
| GattClient::ReadCharacteristic | `ble_send_manager.cpp:850` | 数据泄露 |
| StartNotify | `ble_send_manager.cpp:950` | DoS攻击 |

### 3.3 系统能力调用

| 操作 | 位置 | 风险 |
|------|------|------|
| 蓝牙服务调用 | 多处 | 蓝牙服务滥用 |
| 相机服务调用 | `mc_camera_tracking_controller.cpp` | 隐私泄露 |
| 输入注入 | `mechbody.cfg` | INJECT_INPUT_EVENT权限 |
| 动态库加载 | `load_mechbody_adapter.cpp` | **代码注入** |

### 3.4 内存/资源操作

| 操作 | 位置 | 风险 |
|------|------|------|
| new uint8_t[] | `mc_data_buffer.cpp:31` | 内存分配失败 |
| memcpy_s | 多处 | 边界绕过 |
| std::thread::detach | `ble_send_manager.cpp:149` | 资源泄漏 |
| cJSON_Parse | `service.cpp:221` | **解析崩溃** |

---

## 4. 信任边界图

### 4.1 分层信任模型

```
┌────────────────────────────────────────────────────────────────┐
│ Layer 4: 应用层 (Untrusted)                                    │
│ ├─ 第三方JS应用 (受限API)                                       │
│ └─ 系统JS应用 (完整API)                                         │
├────────────────────────────────────────────────────────────────┤
│ Layer 3: N-API/ANI层 (Semi-Trusted)                            │
│ ├─ 参数解析与验证                                               │
│ ├─ IsSystemApp检查                                             │
│ └─ IPC调用封装                                                 │
├────────────────────────────────────────────────────────────────┤
│ Layer 2: IPC层 (Trusted Boundary)                              │
│ ├─ IPC Token验证                                               │
│ ├─ VerifyAccessToken                                           │
│ └─ MessageParcel解析                                           │
├────────────────────────────────────────────────────────────────┤
│ Layer 1: 服务层 (Highly Trusted)                               │
│ ├─ MechBodyControllerService                                   │
│ ├─ MotionManager                                               │
│ └─ ControllerManager                                           │
├────────────────────────────────────────────────────────────────┤
│ Layer 0: 硬件层 (Physical)                                     │
│ ├─ BLE GATT通信                                                │
│ └─ 机械体感设备                                                 │
└────────────────────────────────────────────────────────────────┘
```

### 4.2 数据流与信任边界

```
┌──────────────┐
│   JS App     │ (Untrusted)
└──────┬───────┘
       │ N-API调用
       ▼
┌──────────────┐
│   N-API层    │ (参数验证、IsSystemApp检查)
└──────┬───────┘
       │ IPC调用
       ▼
┌──────────────┐
│  IPC Stub    │ (Token验证、权限检查) ← 信任边界
└──────┬───────┘
       │ 内部调用
       ▼
┌──────────────┐
│ Service核心  │ (业务逻辑)
└──────┬───────┘
       │ 蓝牙指令
       ▼
┌──────────────┐
│  BLE设备     │ (Physical)
└──────────────┘
```

### 4.3 攻击路径示例

**路径1: N-API参数注入**
```
攻击者 → 构造恶意JS参数 → N-API层(验证绕过) → IPC层 → 服务层
示例: rotate(mechId=-1) 可能导致数组越界
```

**路径2: IPC伪造**
```
攻击者 → 伪造IPC消息(绕过Token验证) → Stub层 → 服务层
需要: 获取MECH_SERVICE_IPC_TOKEN
```

**路径3: 蓝牙协议攻击**
```
恶意设备 → BLE连接 → 发送畸形协议数据 → 协议解析崩溃
示例: 超长数据包导致缓冲区溢出
```

**路径4: JSON注入**
```
攻击者 → setUserOperation(param="{\"device_name\":\"A\"×10000}") 
→ cJSON_Parse → 内存耗尽
```

---

## 5. 攻击面优先级评估

| 攻击面 | 可利用性 | 影响程度 | 风险等级 | 优先级 |
|--------|---------|---------|---------|--------|
| **蓝牙数据接收** | 高(需物理接近) | 高(RCE/崩溃) | **严重** | P0 |
| **N-API参数验证** | 高(任意应用) | 中(越权/崩溃) | **高** | P0 |
| **cJSON解析** | 高(任意应用) | 中(内存耗尽) | **高** | P1 |
| **IPC消息伪造** | 低(需Token) | 高(权限绕过) | **中** | P1 |
| **动态库加载** | 低(需root) | 高(代码注入) | **中** | P2 |

---

## 6. 防护机制清单

| 防护机制 | 位置 | 状态 |
|---------|------|------|
| IPC Token验证 | `stub.cpp:97` | ✅ 启用 |
| VerifyAccessToken | `service.cpp:212` | ✅ 启用 |
| IsSystemApp检查 | `service.cpp:1027` | ✅ 启用 |
| 参数类型检查 | `js_mech_manager.cpp` | ✅ 启用 |
| 参数范围检查 | 多处 | ✅ 启用 |
| MAC地址脱敏 | `utils.cpp` | ✅ 启用 |
| 蓝牙数据长度限制 | `ble_send_manager.cpp:46` | ✅ 启用(1MB) |
| 安全内存拷贝 | `memcpy_s` | ✅ 启用 |
| 栈保护 | `BUILD.gn` | ✅ 启用 |
| RELRO加固 | `BUILD.gn` | ✅ 启用 |
| Sanitizers | `BUILD.gn` | ✅ 启用 |

---

## 7. 建议的安全测试用例

### 7.1 N-API模糊测试
```javascript
// 参数类型混淆
mechManager.rotate("string", {}, null);
mechManager.rotate(-1, {yaw: NaN}, Infinity);

// 超大输入
const hugeString = "A".repeat(100000);
mechManager.setUserOperation(0, hugeString, hugeString);
```

### 7.2 蓝牙协议模糊测试
```cpp
// 畸形协议数据
uint8_t malformed[] = {0xFF, 0xFF, 0xFF, 0xFF}; // 非法命令码
BleSendManager::GetInstance().OnReceive(malformed, sizeof(malformed));
```

### 7.3 IPC模糊测试
```cpp
// 伪造IPC消息
MessageParcel data;
data.WriteInterfaceToken(wrongToken); // 错误Token
data.WriteInt32(-999999); // 越界mechId
stub->OnRemoteRequest(code, data, reply, option);
```

---

*文档创建时间: 2025-02-07*
