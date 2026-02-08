# 安全风险评审

## 评审范围

| 范围 | 说明 |
|------|------|
| **代码路径** | `services/` 核心服务、`interface/napi/` N-API 层 |
| **IPC 层** | SystemAbility 8550、IPC 通信 |
| **输入验证** | 参数校验、Parcel unmarshalling |
| **权限模型** | AccessToken、TokenID 验证 |
| **攻击面** | N-API、IPC、蓝牙通信、文件系统 |

## 信任边界

```
┌─────────────────────────────────────────────────────────────────────┐
│                         不可信区域                                    │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │
│  │ 第三方应用   │  │ 蓝牙设备    │  │ 网络攻击者  │                 │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘                 │
└─────────┼────────────────┼────────────────┼─────────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         信任边界                                      │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              Mechbody Service (mechbody 进程)                │   │
│  │  - SELinux: u:r:mechbody:s0                                 │   │
│  │  - UID: mechbody                                            │   │
│  │  - 权限: 蓝牙/相机/输入注入                                  │   │
│  └─────────────────────────┬───────────────────────────────────┘   │
│                            │                                          │
│  ┌─────────────────────────▼───────────────────────────────────┐   │
│  │              南向协议适配 (libmech_adapter.z.so)              │   │
│  │  - 厂商实现                                                   │   │
│  │  - 硬件交互                                                   │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

## 攻击面清单

| 攻击面 | 入口 | 说明 |
|--------|------|------|
| **N-API** | `js_mech_manager.cpp` | JS 参数解析、JS 回调处理 |
| **IPC** | `mechbody_controller_stub.cpp` | MessageParcel 数据、命令码分发 |
| **蓝牙** | `ble_send_manager.cpp` | BLE 读写、GATT 特征值 |
| **回调注册** | `Register*Callback` | IRemoteObject 接收 |
| **动态加载** | `load_mechbody_adapter.cpp` | dlopen 加载 vendor 适配层 |

## 安全机制

### 权限检查

**权限定义**：`services/src/mechbody_controller_service.cpp:39`

```cpp
const std::string PERMISSION_NAME = "ohos.permission.CONNECT_MECHANIC_HARDWARE";
```

**权限验证**：`services/src/mechbody_controller_service.cpp:212-215`

```cpp
uint32_t tokenId = IPCSkeleton::GetCallingTokenID();
int32_t ret = AccessTokenKit::VerifyAccessToken(tokenId, PERMISSION_NAME);
if (ret != PERMISSION_GRANTED) {
    return PERMISSION_DENIED;
}
```

**权限配置文件**：`etc/init/mechbody.cfg`

```json
{
  "permission": [
    "ohos.permission.ACCESS_BLUETOOTH",
    "ohos.permission.MANAGE_BLUETOOTH",
    "ohos.permission.MANAGE_LOCAL_ACCOUNTS",
    "ohos.permission.GET_BLUETOOTH_LOCAL_MAC",
    "ohos.permission.INJECT_INPUT_EVENT",
    "ohos.permission.CAMERA"
  ]
}
```

### 系统应用验证

**验证位置**：`js_mech_manager.cpp:1598-1600`

```cpp
uint32_t tokenId = IPCSkeleton::GetCallingTokenID();
if (!TokenIdKit::IsSystemAppByFullTokenID(tokenId)) {
    // 拒绝非系统应用
}
```

### IPC Token 校验

**Token 定义**：`services/include/mechbody_controller_ipc_interface_code.h:24`

```cpp
inline const std::u16string MECH_SERVICE_IPC_TOKEN = u"ohos.mechservice.accessToken";
```

**校验实现**：`services/src/mechbody_controller_stub.cpp:97-101`

```cpp
std::u16string interfaceToken = data.ReadInterfaceToken();
if (interfaceToken != MECH_SERVICE_IPC_TOKEN) {
    HILOGW("OnRemoteRequest interface token check failed!");
    return INVALID_PARAMETERS_ERR;
}
```

### 输入验证

**参数校验模式**：`services/src/mechbody_controller_service.cpp:515-526`

```cpp
// 设备 ID 校验
if (mechId < 0) {
    return MECH_ID_INVALID;
}

// 参数对象校验
auto rotateByDegreeParam = data.ReadParcelable<RotateByDegreeParam>();
if (rotateByDegreeParam == nullptr) {
    return INVALID_PARCELABLE;
}

// 范围校验
if (duration < 0) {
    return PARAMETER_CHECK_FAILED;
}
```

### Parcel Unmarshalling

**安全 Unmarshalling**：`services/include/mechbody_controller_types.h:62-72`

```cpp
static MechInfo* Unmarshalling(Parcel& data) {
    MechInfo* info = new MechInfo();
    if (!data.ReadInt32(info->mechId)) {
        delete info;
        return nullptr;
    }
    // ... 更多字段
    return info;
}
```

## 风险清单

### 风险 1：N-API 参数类型混淆

| 属性 | 值 |
|------|-----|
| **严重程度** | 中 |
| **可利用性** | 高 |
| **影响范围** | JS 应用崩溃 |

**证据**：`js_mech_manager.cpp:84-122`

```cpp
// 仅检查 napi_typeof，未检查 JS 值语义
napi_valuetype type;
napi_typeof(env, args[0], &type);
if (type != napi_string) {
    napi_throw_type_error(env, ...);
    return nullptr;
}
```

**触发条件**：
- JS 应用传入非预期的 JS 值类型
- 未对 JS 值的语义合法性进行校验

**修复建议**：
```cpp
// 增加语义校验
if (type == napi_string) {
    // 检查字符串内容（长度、字符集）
    if (str.length() > MAX_DEVICE_ID_LEN) {
        napi_throw_error(env, ...);
        return nullptr;
    }
}
```

---

### 风险 2：IPC 回调注册无上限检查

| 属性 | 值 |
|------|-----|
| **严重程度** | 中 |
| **可利用性** | 中 |
| **影响范围** | 内存耗尽 |

**证据**：`services/src/mechbody_controller_service.cpp`

```cpp
std::map<uint32_t, sptr<IRemoteObject>> deviceAttachCallback_;
// 无数量限制，可能被恶意注册大量回调
```

**触发条件**：
- 恶意应用反复调用 `RegisterAttachStateChangeCallback`
- 导致 map 无限增长

**修复建议**：
```cpp
static constexpr size_t MAX_CALLBACK_COUNT = 16;
if (deviceAttachCallback_.size() >= MAX_CALLBACK_COUNT) {
    HILOGE("Too many callbacks registered");
    return MAX_CALLBACK_EXCEEDED;
}
```

---

### 风险 3：Promise 回调未捕获异常

| 属性 | 值 |
|------|-----|
| **严重程度** | 低 |
| **可利用性** | 低 |
| **影响范围** | JS 回调异常 |

**证据**：`js_mech_manager_service.cpp`

```cpp
// napi_handle_scope 管理可能不完善
napi_send_event(env, callback, callbackFn, priority);
```

**触发条件**：
- JS 回调函数抛出未捕获异常
- 可能导致 Node-API 引擎状态异常

**修复建议**：
```cpp
// 增加 try-catch 包装
napi_value jsResult = nullptr;
napi_status status = napi_create_object(env, &jsResult);
// ... 设置结果
// 不需要显式 try-catch，N-API 会处理
```

---

### 风险 4：动态库加载路径固定

| 属性 | 值 |
|------|-----|
| **严重程度** | 高 |
| **可利用性** | 低（需要 root） |
| **影响范围** | 代码执行 |

**证据**：`services/src/utils/load_mechbody_adapter.cpp:41`

```cpp
handle = dlopen("/system/lib64/libmech_adapter.z.so", RTLD_LAZY);
```

**触发条件**：
- 攻击者获得 system 分区写权限
- 替换 libmech_adapter.z.so

**缓解因素**：
- SELinux 限制：`u:r:mechbody:s0`
- system 分区只读

**修复建议**：
```cpp
// 增加完整性校验
if (!VerifyLibrarySignature(handle)) {
    dlclose(handle);
    return ERROR_INVALID_LIBRARY;
}
```

---

### 风险 5：设备 ID 整数溢出

| 属性 | 值 |
|------|-----|
| **严重程度** | 低 |
| **可利用性** | 低 |
| **影响范围** | 错误的设备操作 |

**证据**：`services/src/mechbody_controller_service.cpp:515`

```cpp
if (mechId < 0) {
    return MECH_ID_INVALID;
}
// 仅检查负值，未检查上界
```

**触发条件**：
- 传入超大 mechId 值
- 导致数组越界或错误的设备匹配

**修复建议**：
```cpp
if (mechId < 0 || mechId >= MAX_MECH_ID) {
    return MECH_ID_INVALID;
}
```

---

### 风险 6：死亡监听竞态条件

| 属性 | 值 |
|------|-----|
| **严重程度** | 中 |
| **可利用性** | 中 |
| **影响范围** | 回调泄漏或重复清理 |

**证据**：`services/src/controller/mc_controller_ipc_death_listener.cpp:27-53`

```cpp
void MechControllerIpcDeathListener::OnRemoteDied(const wptr<IRemoteObject> &object) {
    // 竞态：回调可能在其他线程被注销
    MechBodyControllerService::GetInstance().deviceAttachCallback_.erase(tokenId_);
}
```

**触发条件**：
- 客户端在死亡回调触发前主动注销
- 导致 map 操作冲突

**缓解措施**：使用 `std::lock_guard` 保护

**修复建议**：使用更细粒度的锁或原子操作

---

### 风险 7：蓝牙 UUID 硬编码

| 属性 | 值 |
|------|-----|
| **严重程度** | 低 |
| **可利用性** | 低 |
| **影响范围** | 错误的 BLE 设备连接 |

**证据**：`services/src/ble_send_manager.cpp:63-65`

```cpp
static constexpr char BLE_UUID_SERVICE[] = "0000FEED-0000-1000-8000-00805F9B34FB";
static constexpr char BLE_UUID_CHARACTERISTIC[] = "0000FEEE-0000-1000-8000-00805F9B34FB";
```

**触发条件**：
- 错误的 UUID 导致连接错误的 BLE 设备
- 可能泄露数据给错误设备

**修复建议**：
```cpp
// 增加 UUID 白名单校验
if (!IsWhitelistedUUID(uuid)) {
    return ERROR_INVALID_UUID;
}
```

---

### 风险 8：N-API 回调未验证 IRemoteObject 来源

| 属性 | 值 |
|------|-----|
| **严重程度** | 中 |
| **可利用性** | 中 |
| **影响范围** | 伪造回调 |

**证据**：`js_mech_manager_stub.cpp`

```cpp
// IPC 回调处理中未验证发送者身份
int32_t OnAttachStateChangeCallback(MessageParcel& data, MessageParcel& reply) {
    // 直接处理回调数据，未验证来源
}
```

**触发条件**：
- 伪造的 IPC 消息触发回调
- 导致错误的业务逻辑执行

**修复建议**：
```cpp
// 验证发送者 TokenID
uint32_t callingToken = IPCSkeleton::GetCallingTokenID();
if (callingToken != expectedToken_) {
    return ERROR_UNAUTHORIZED;
}
```

---

### 风险 9：HiSysEvent 敏感信息泄露

| 属性 | 值 |
|------|-----|
| **严重程度** | 低 |
| **可利用性** | 低 |
| **影响范围** | 设备 MAC/用户操作泄露 |

**证据**：`services/src/dotReport/hisysevent_utils.cpp`

```cpp
// 设备 MAC 和用户操作可能被记录到日志
HILOGI("Device connected: %{public}s", mac.c_str());
```

**触发条件**：
- 日志级别配置不当
- 敏感信息被持久化

**修复建议**：
```cpp
// 日志脱敏
HILOGI("Device connected: XX:XX:XX:XX:XX:XX");
// 或使用 HILOGD (debug level)
```

---

### 风险 10：角度值浮点精度问题

| 属性 | 值 |
|------|-----|
| **严重程度** | 低 |
| **可利用性** | 低 |
| **影响范围** | 运动控制精度异常 |

**证据**：`services/include/mechbody_controller_types.h`

```cpp
struct EulerAngle {
    double pitch;
    double yaw;
    double roll;
};
```

**触发条件**：
- 浮点数精度问题导致角度偏差
- 累积误差影响运动精度

**修复建议**：
```cpp
// 增加精度校验和舍入
constexpr double ANGLE_EPSILON = 0.001;
if (fabs(angle - round(angle)) > ANGLE_EPSILON) {
    return ERROR_INVALID_ANGLE;
}
```

## 安全检查清单

| 检查项 | 状态 | 文件 |
|--------|------|------|
| 权限检查 | ✅ | service.cpp:212,349,422 |
| 系统应用检查 | ✅ | js_mech_manager.cpp:1598 |
| IPC Token 验证 | ✅ | stub.cpp:97-101 |
| 参数校验 (mechId) | ✅ | service.cpp:515 |
| 参数校验 (duration) | ✅ | service.cpp:523 |
| Parcel Unmarshalling 安全 | ✅ | types.h:62-72 |
| Death Recipient | ✅ | mc_controller_ipc_death_listener.cpp |
| SELinux 上下文 | ✅ | mechbody.cfg |
| 栈保护 | ✅ | BUILD.gn: "-fstack-protector-strong" |
| RELRO 加固 | ✅ | BUILD.gn: "-Wl,-z,relro,-z,now" |
| Sanitizers | ✅ | BUILD.gn: boundary, cfi, ubsan |

## 安全相关文件

| 文件 | 说明 |
|------|------|
| `services/src/mechbody_controller_service.cpp` | 权限检查实现 |
| `services/src/mechbody_controller_stub.cpp` | IPC 安全验证 |
| `etc/init/mechbody.cfg` | SELinux 和权限配置 |
| `services/BUILD.gn` | 编译时安全加固 |
| `services/include/mechbody_controller_ipc_interface_code.h` | IPC Token |

## 相关文档

- [架构说明](02_Architecture.md) → 信任边界
- [N-API 参考](03_NAPI_Reference.md) → API 安全
- [内部 API](04_Inner_API.md) → 模块安全
