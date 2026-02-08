# FAQ 与排错

> connectivity_cangjie_wrapper 常见问题、定位路径与调试方法

## 常见问题

### Q1: 权限被拒绝 (错误码 201)

**问题描述**: 调用蓝牙或 WiFi API 时返回错误码 201。

**常见原因**:
1. 未在 `module.json5` 中声明权限
2. 权限申请未被用户批准
3. 权限声明的 `usedScene` 配置不正确

**解决方案**:

```json
// module.json5
"requestPermissions": [
  {
    "name": "ohos.permission.ACCESS_BLUETOOTH",
    "usedScene": {
      "abilities": ["EntryAbility"],
      "when": "inUse"
    }
  },
  {
    "name": "ohos.permission.GET_WIFI_INFO",
    "usedScene": {
      "abilities": ["EntryAbility"],
      "when": "inUse"
    }
  }
]
```

**定位路径**:
1. 检查 `module.json5` 是否包含所需权限
2. 检查权限是否在运行时动态申请
3. 检查用户是否授予了权限

---

### Q2: 能力不支持 (错误码 801)

**问题描述**: 调用 API 时返回错误码 801。

**常见原因**:
1. 设备不支持所需系统能力
2. 系统能力未使能

**解决方案**:

```cj
// 检查系统能力
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Communication.Bluetooth.Core"
]
// API 会自动检查，返回 801 错误码
```

**定位路径**:
1. 确认设备类型是否为 "standard"（项目仅支持标准设备）
2. 确认系统镜像是否包含对应能力
3. 查看设备日志确认能力检查结果

---

### Q3: 蓝牙服务不可用 (错误码 2900003)

**问题描述**: 调用蓝牙 API 返回错误码 2900003。

**常见原因**:
1. 蓝牙未开启
2. 蓝牙服务已停止
3. 蓝牙硬件异常

**解决方案**:

```cj
import ohos.bluetooth.connection.*

// 提示用户开启蓝牙
if (!isStateEnabled()) {
    // 引导用户开启蓝牙
}
```

**定位路径**:
1. 检查设备蓝牙开关状态
2. 检查蓝牙服务是否正常运行
3. 查看蓝牙服务日志

---

### Q4: BLE 扫描无结果

**问题描述**: 调用 `startBleScanning()` 后回调中没有扫描结果。

**常见原因**:
1. 周围确实没有 BLE 设备
2. 扫描参数配置不当
3. 扫描持续时间过短

**解决方案**:

```cj
// 使用默认参数开始扫描
startBleScanning([], None)

// 等待足够时间（建议 10 秒以上）
// 在回调中处理结果
on(BleDeviceFind) { results ->
    print("Found ${results.size} devices")
}
```

**定位路径**:
1. 确认周围有可发现的 BLE 设备
2. 检查扫描回调是否正确注册
3. 检查扫描时长是否足够

---

### Q5: P2P 连接失败 (错误码 2801000)

**问题描述**: 调用 `p2pConnect()` 返回错误码 2801000。

**常见原因**:
1. WiFi 未开启
2. P2P 配置错误
3. 目标设备不可达

**解决方案**:

```cj
// 检查 WiFi 状态
if (!isWifiActive()) {
    // 引导用户开启 WiFi
}

// 配置 P2P 连接
let config = WifiP2pConfig()
config.deviceName = "MyDevice"
p2pConnect(config)
```

**定位路径**:
1. 检查 WiFi 是否开启 (`isWifiActive()`)
2. 检查 P2P 配置是否正确
3. 确认目标设备是否在可发现状态
4. 查看 WiFi 服务日志

---

### Q6: 回调不被触发

**问题描述**: 注册回调后，事件发生时回调没有被调用。

**常见原因**:
1. 回调对象被 GC 回收
2. 回调注册失败但未报错
3. 事件类型不匹配

**解决方案**:

```cj
// 保持回调对象引用，避免被 GC 回收
let scanCallback = Callback1Argument<Array<ScanResult>> { results ->
    print("Found ${results.size} devices")
}
on(BleDeviceFind, scanCallback)

// 事件类型必须匹配
if (eventType == BleDeviceFind) {
    // 注册正确的回调
}
```

**定位路径**:
1. 确认回调对象生命周期足够长
2. 检查回调注册返回值
3. 确认事件类型参数正确

---

### Q7: 构建失败 - 找不到 FFI 依赖

**问题描述**: 构建时提示找不到 `cj_bluetooth_ble_ffi` 等依赖。

**常见原因**:
1. `communication_bluetooth` 组件未构建
2. 依赖版本不匹配

**解决方案**:

```bash
# 确保蓝牙组件已构建
hb build -f --components communication_bluetooth

# 或者完整构建
hb build -f
```

**定位路径**:
1. 检查 `bundle.json` 中的 `deps` 配置
2. 确认 `communication_bluetooth` 和 `communication_wifi` 组件存在
3. 检查 GN 依赖链

---

## 调试方法

### 日志查看

```bash
# 查看蓝牙相关日志
hilog | grep -i bluetooth

# 查看 WiFi 相关日志
hilog | grep -i wifi

# 查看 Cangjie 相关日志
hilog | grep -i cangjie
```

### 参数校验清单

| API | 必填参数 | 参数类型 | 边界值 |
|-----|----------|----------|--------|
| `createGattClientDevice` | deviceId | String | MAC 地址格式 |
| `startBleScanning` | filters | Array<ScanFilter> | 空数组表示不过滤 |
| `startAdvertising` | setting, advData | AdvertiseSetting, AdvertiseData | 数据长度限制 |
| `p2pConnect` | config | WifiP2pConfig | 必填字段检查 |

### 错误码速查表

| 错误码 | 含义 | 常见原因 |
|--------|------|----------|
| 201 | Permission denied | 权限未申请或未授予 |
| 801 | Capability not supported | 设备/系统不支持 |
| 2900001 | Service stopped | 服务已停止 |
| 2900003 | Bluetooth disabled | 蓝牙未开启 |
| 2900010 | Advertising resources exhausted | 广播资源耗尽 |
| 2902054 | Invalid advertising data | 广播数据过长 |
| 2902055 | Invalid advertising ID | 无效广告 ID |
| 2501000 | WiFi operation failed | WiFi 操作失败 |
| 2801000 | P2P operation failed | P2P 操作失败 |
| 2801001 | WiFi STA disabled | WiFi STA 未启用 |

## 调试代码模板

### 蓝牙扫描调试

```cj
import ohos.bluetooth.ble.*

// 1. 检查权限
// 确保已申请 ohos.permission.ACCESS_BLUETOOTH

// 2. 注册扫描结果回调
on(BleDeviceFind) { results ->
    print("=== 扫描到 ${results.size} 个设备 ===")
    for (result in results) {
        print("设备: ${result.deviceId}")
        print("RSSI: ${result.rssi}")
    }
}

// 3. 开始扫描
try {
    startBleScanning([], None)
    print("扫描已开始")
} catch (e: BusinessException) {
    print("扫描失败: ${e.code} - ${e.message}")
}

// 4. 停止扫描
// stopBleScanning()
```

### WiFi P2P 调试

```cj
import ohos.wifi_manager.*

// 1. 检查 WiFi 状态
print("WiFi 状态: ${isWifiActive()}")

// 2. 注册状态回调
on(WifiScanStateChange) { state ->
    print("WiFi 扫描状态变化: ${state}")
}

// 3. 开始发现设备
try {
    startDiscoverDevices()
    print("开始发现设备")
} catch (e: BusinessException) {
    print("发现设备失败: ${e.code} - ${e.message}")
}

// 4. 连接 P2P
let config = WifiP2pConfig()
config.deviceName = "TargetDevice"
try {
    p2pConnect(config)
    print("开始连接")
} catch (e: BusinessException) {
    print("连接失败: ${e.code} - ${e.message}")
}
```

## 相关文档

| 文档 | 说明 |
|------|------|
| [项目概览](./01_Project_Overview.md) | 项目定位与核心能力 |
| [N-API 参考](./04_N-API_Reference.md) | API 详细说明 |
| [安全评审](./07_Security_Review.md) | 安全使用指南 |
| [蓝牙开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/connectivity/bluetooth/cj-bluetooth-overview.md) | 官方开发指南 |
| [WiFi 开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/connectivity/wifi/cj-wifi-development-guide.md) | 官方 WiFi 开发指南 |
