# 安全风险评审

> connectivity_cangjie_wrapper 攻击面分析、信任边界与安全风险

## 概述

本文档对 `connectivity_cangjie_wrapper` 项目进行安全风险评审，分析其攻击面、信任边界和潜在安全风险，并提供修复建议。

## 评审范围

| 范围 | 说明 |
|------|------|
| **代码范围** | `ohos/` 目录下的所有 `.cj` 文件 |
| **测试代码** | `test/` 目录不纳入评审范围 |
| **依赖组件** | cangjie_ark_interop, hiviewdfx_cangjie_wrapper, bluetooth, wifi |

## 攻击面分析

### 输入入口

| 入口类型 | 说明 | 风险等级 |
|----------|------|----------|
| **用户参数** | Cangjie API 的输入参数 | 高 |
| **回调数据** | 底层蓝牙/WiFi 服务返回的数据 | 中 |
| **配置文件** | WifiP2pConfig 等配置对象 | 中 |

### 主要攻击面

```
┌─────────────────────────────────────────────────────────────────┐
│                        攻击面分析图                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │  用户输入   │    │  蓝牙广播   │    │  WiFi 扫描  │         │
│  │  (参数)     │    │  (数据)     │    │  (结果)     │         │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘         │
│         │                  │                  │                 │
│         ▼                  ▼                  ▼                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Cangjie API 层                        │   │
│  │  • 参数校验                                                │   │
│  │  • 类型转换                                                │   │
│  │  • 异常处理                                                │   │
│  └────────────────────────┬────────────────────────────────┘   │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    FFI 接口层                            │   │
│  │  • 数据传递                                                │   │
│  │  • 内存管理                                                │   │
│  └────────────────────────┬────────────────────────────────┘   │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   底层蓝牙/WiFi 服务                      │   │
│  │  (不在本项目评审范围内)                                    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界图                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                      可信区域                             │   │
│  │  ┌─────────────────────────────────────────────────┐    │   │
│  │  │           OpenHarmony 系统层                      │    │   │
│  │  │  • 权限管理                                        │    │   │
│  │  │  • 能力检查 (cap)                             sys │    │   │
│  │  │  • 蓝牙服务 (communication_bluetooth)              │    │   │
│  │  │  • WiFi 服务 (communication_wifi)                  │    │   │
│  │  └─────────────────────────────────────────────────┘    │   │
│  │                           ↑                               │   │
│  │  ┌─────────────────────────────────────────────────┐    │   │
│  │  │      connectivity_cangjie_wrapper (本项目)        │    │   │
│  │  │  • 参数校验                                       │    │   │
│  │  │  • 类型安全                                       │    │   │
│  │  │  • 异常封装                                       │    │   │
│  │  └─────────────────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ↓                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    不可信区域                             │   │
│  │  ┌─────────────────────────────────────────────────┐    │   │
│  │  │              第三方应用层                          │    │   │
│  │  │  • 恶意输入                                       │    │   │
│  │  │  • 权限滥用                                       │    │   │
│  │  │  • 资源耗尽                                       │    │   │
│  │  └─────────────────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 已识别风险点

### 风险 1: 设备 ID 格式验证不足

**证据位置**: `ohos/bluetooth/ble/ble.cj:56-66`

```cj
public func createGattClientDevice(deviceId: String): GattClientDevice {
    var id = 0
    var errorCode: Int32 = 0
    unsafe {
        try (cDeviceId = LibC.mallocCString(deviceId).asResource()) {
            id = FfiBluetoothBleCreateGattClientDevice(cDeviceId.value, inout errorCode)
            checkRet(errorCode)
        }
    }
    return GattClientDevice(id)
}
```

**问题描述**: `deviceId` 参数直接传递给底层 FFI，未进行格式校验。

**触发方式**: 传入非法的 MAC 地址格式字符串。

**潜在影响**: 
- 底层服务可能解析失败
- 内存访问异常
- 静默失败导致用户体验问题

**修复建议**:
```cj
public func createGattClientDevice(deviceId: String): GattClientDevice {
    // 添加 MAC 地址格式校验
    let macPattern = r"^([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}$"
    if (!deviceId.matches(macPattern)) {
        throw BusinessException(2900099, "Invalid device ID format")
    }
    // ... 后续逻辑
}
```

**风险等级**: 中

---

### 风险 2: 回调注册缺乏上限控制

**证据位置**: `ohos/bluetooth/ble/ble.cj:293-307`

```cj
private func commonSubscribe1Arg<CT, T>(callbackType: BluetoothBleCallbackType, callback: CallbackObject,
    ctor: (CT) -> T) where CT <: CType {
    BLUETOOTH_LOG.debug("subscribe ${callbackType}")
    let controller = GLOBAL_CALLBACK_MAP[callbackType]
    if (!controller.isEnabled()) {
        register(callbackType, argWrapper1<CT, T>(controller, ctor))
        controller.enableController()
    } else {
        if (controller.hasCallback(callback)) {
            BLUETOOTH_LOG.info("The ${callbackType} callback is registered, no need to re-registered")
            return
        }
    }
    controller.addCallback(callback)
}
```

**问题描述**: 同一个 `callbackType` 可以注册多个回调，但没有数量限制。

**触发方式**: 应用重复注册大量回调。

**潜在影响**:
- 内存泄漏
- 回调调用性能下降
- 资源耗尽

**修复建议**: 添加回调数量上限检查。

**风险等级**: 低

---

### 风险 3: WiFi 扫描结果信息泄露

**证据位置**: `ohos/wifi_manager/wifi.cj:53-87`

```cj
/**
 * Obtain the scanned station list. If does't have the permission of ohos.permission.GET_WIFI_PEERS_MAC,
 * return random bssid.
 */
@!APILevel[
    since: "22",
    permission: "ohos.permission.GET_WIFI_INFO",
    syscap: "SystemCapability.Communication.WiFi.STA",
    throwexception: true
]
public func getScanInfoList(): Array<WifiScanInfo> {
    // ...
}
```

**问题描述**: 文档明确说明无权限时返回随机 BSSID，但实际行为依赖底层实现。

**触发方式**: 应用未申请 `GET_WIFI_PEERS_MAC` 权限。

**潜在影响**:
- 位置信息泄露风险
- 隐私合规问题

**修复建议**: 确认底层实现符合文档描述。

**风险等级**: 低（已通过权限控制）

---

### 风险 4: 跨平台 Mock 代码不一致

**证据位置**: `mock/` 目录下多个 `.cj` 文件

**问题描述**: Windows/macOS 使用 mock 代码，行为可能与实际运行时不一致。

**触发方式**: 在非标准设备平台上测试。

**潜在影响**:
- 测试覆盖不完整
- 隐藏的跨平台问题

**修复建议**: 确保 mock 代码行为与实际实现一致。

**风险等级**: 低（开发阶段风险）

---

### 风险 5: 错误信息泄露

**证据位置**: 多个 `checkRet(errorCode)` 调用

```cj
if (code != SUCCESS_CODE) {
    let err = getCodeAndMsg(code, SYSCAP_WIFI_STA)
    throw BusinessException(err[0], "isWifiActive failed: ${err[1]}")
}
```

**问题描述**: 错误消息可能包含敏感信息（如设备标识符）。

**触发方式**: API 调用失败时抛出异常。

**潜在影响**:
- 敏感信息通过异常日志泄露
- 应用日志中暴露系统信息

**修复建议**: 审查 `getErrorMsg` 返回的错误消息，确保不包含敏感信息。

**风险等级**: 低

---

## 权限与能力检查

### 权限清单

| 权限 | 用途 | 强制程度 |
|------|------|----------|
| `ohos.permission.ACCESS_BLUETOOTH` | 蓝牙扫描、广播 | API 注解声明 |
| `ohos.permission.GET_WIFI_INFO` | WiFi 扫描、P2P 连接 | API 注解声明 |

### 系统能力检查

| 能力 | 用途 | 错误码 |
|------|------|--------|
| `SystemCapability.Communication.Bluetooth.Core` | 蓝牙核心能力 | 801 |
| `SystemCapability.Communication.WiFi.STA` | WiFi STA 模式 | 801 |
| `SystemCapability.Communication.WiFi.P2P` | WiFi P2P 模式 | 801 |

## 安全使用建议

### 应用开发者

1. **权限申请**: 在 `module.json5` 中正确申请所需权限
2. **异常处理**: 捕获 `BusinessException` 并进行适当处理
3. **输入校验**: 对用户输入的设备 ID 等参数进行格式校验
4. **回调管理**: 及时 `off()` 取消不需要的回调，避免资源泄漏

### 集成测试

1. **权限测试**: 验证无权限时的行为符合预期
2. **边界测试**: 验证参数边界值处理
3. **并发测试**: 验证多线程回调场景

## 风险汇总表

| 风险编号 | 风险描述 | 风险等级 | 状态 |
|----------|----------|----------|------|
| R1 | 设备 ID 格式验证不足 | 中 | 待修复 |
| R2 | 回调注册缺乏上限控制 | 低 | 建议优化 |
| R3 | WiFi 扫描结果信息泄露 | 低 | 已控制 |
| R4 | 跨平台 Mock 代码不一致 | 低 | 已知限制 |
| R5 | 错误信息泄露 | 低 | 建议审查 |

## 相关文档

| 文档 | 说明 |
|------|------|
| [项目概览](./01_Project_Overview.md) | 项目定位与核心能力 |
| [N-API 参考](./04_N-API_Reference.md) | API 详细说明 |
| [FAQ 与排错](./08_FAQ_Troubleshooting.md) | 常见问题排查 |
| [OpenHarmony 权限模型](https://gitee.com/openharmony/docs/blob/master/zh-cn/security/权限管理.md) | 权限管理文档 |
