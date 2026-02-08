# N-API 接口参考

本文档描述扩展外部设备管理模块对外提供的 JavaScript API 接口，包括接口清单、参数说明、返回值、错误码以及使用示例。这些 API 供三方应用调用，用于查询、绑定和管理外部设备。

## 接口概览

N-API 模块名为 `driver.deviceManager`，在模块初始化时通过 `napi_module_register()` 注册。模块导出以下 8 个 JS API 方法和 1 个枚举类型：

| 方法名 | 功能 | 同步/异步 | 回调/Promise |
|--------|------|----------|-------------|
| `queryDevices()` | 按总线类型查询设备 | 异步 | Promise |
| `bindDevice()` | 绑定设备到驱动 | 异步 | Promise/Callback |
| `bindDeviceDriver()` | bindDevice 的别名 | 异步 | Promise/Callback |
| `unbindDevice()` | 解绑设备 | 异步 | Promise/Callback |
| `bindDriverWithDeviceId()` | 按设备ID绑定驱动 | 异步 | Promise/Callback |
| `unbindDriverWithDeviceId()` | 按设备ID解绑驱动 | 异步 | Promise/Callback |
| `queryDeviceInfo()` | 查询设备详细信息 | 异步 | Promise |
| `queryDriverInfo()` | 查询驱动信息 | 异步 | Promise |

| 枚举名 | 类型 | 说明 |
|--------|------|------|
| `BusType` | 枚举 | 总线类型枚举 |

## BusType 枚举

`BusType` 枚举定义了模块支持的设备总线类型。目前仅定义 USB 类型，未来可扩展支持其他总线类型。

```javascript
driver.deviceManager.BusType
```

**枚举值**：

| 值 | 常量名 | 说明 |
|----|--------|------|
| 1 | `USB` | USB 总线类型 |

**使用示例**：

```javascript
import driver from '@ohos.driver.deviceManager';

const busType = driver.deviceManager.BusType.USB;
console.log('Bus type:', busType); // 输出: Bus type: 1
```

## queryDevices()

查询指定总线类型的已连接设备列表。

```javascript
driver.deviceManager.queryDevices(busType)
```

**参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| busType | `BusType` | 是 | 要查询的总线类型，传入 `driver.deviceManager.BusType.USB` |

**返回值**：

| 类型 | 说明 |
|------|------|
| `Promise<Device[]>` | 设备数组的 Promise |

**Device 对象结构**：

| 字段 | 类型 | 说明 |
|------|------|------|
| deviceId | `number` | 设备唯一标识符 |
| busType | `BusType` | 总线类型 |
| description | `string` | 设备描述信息 |
| vendorId | `number` | USB 厂商 ID（仅 USB 设备有） |
| productId | `number` | USB 产品 ID（仅 USB 设备有） |

**错误码**：

| 错误码 | 说明 |
|--------|------|
| 201 | 无权限调用 |
| 22900001 | 服务异常 |
| 26300001 | 服务异常 |
| 401 | 参数错误 |

**使用示例**：

```javascript
import driver from '@ohos.driver.deviceManager';

try {
    const devices = await driver.deviceManager.queryDevices(driver.deviceManager.BusType.USB);
    console.log('Found devices:', devices.length);
    devices.forEach(device => {
        console.log(`Device: ${device.deviceId}, VID: 0x${device.vendorId.toString(16)}, PID: 0x${device.productId.toString(16)}`);
    });
} catch (error) {
    console.error('Query devices failed:', error.code, error.message);
}
```

## bindDevice() / bindDeviceDriver()

绑定设备到其匹配的驱动扩展。调用此方法后，系统会启动对应的驱动扩展 Ability，并在启动成功后通过回调返回驱动扩展的远程对象。

```javascript
driver.deviceManager.bindDevice(deviceId, callback)
driver.deviceManager.bindDevice(deviceId)
driver.deviceManager.bindDeviceDriver(deviceId, callback)
```

**参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| deviceId | `number` | 是 | 要绑定的设备 ID |
| callback | `function(err, result)` | 否 | 回调函数，如果不传则返回 Promise |

**callback 参数**：

| 参数名 | 类型 | 说明 |
|--------|------|------|
| err | `BusinessError` | 错误对象，成功时为 null |
| result | `{deviceId, remote}` | 绑定结果 |

**result 对象结构**：

| 字段 | 类型 | 说明 |
|------|------|------|
| deviceId | `number` | 绑定的设备 ID |
| remote | `RemoteObject` | 驱动扩展的远程对象，用于与驱动交互 |

**返回值**：

| 类型 | 说明 |
|------|------|
| `Promise<{deviceId, remote}>` | 绑定结果的 Promise |
| `void` | 如果传入 callback 则无返回值 |

**使用示例（Promise 模式）**：

```javascript
import driver from '@ohos.driver.deviceManager';

try {
    const devices = await driver.deviceManager.queryDevices(driver.deviceManager.BusType.USB);
    if (devices.length > 0) {
        const device = devices[0];
        const result = await driver.deviceManager.bindDevice(device.deviceId);
        console.log('Bind success:', result.deviceId);
        // 使用远程对象与驱动交互
        // result.remote.callMethod(...);
    }
} catch (error) {
    console.error('Bind failed:', error.code, error.message);
}
```

**使用示例（Callback 模式）**：

```javascript
import driver from '@ohos.driver.deviceManager';

driver.deviceManager.bindDevice(12345678, (err, result) => {
    if (err) {
        console.error('Bind failed:', err.code, err.message);
        return;
    }
    console.log('Bind success:', result.deviceId);
});
```

## unbindDevice()

解绑设备，释放与驱动扩展的连接。

```javascript
driver.deviceManager.unbindDevice(deviceId, callback)
driver.deviceManager.unbindDevice(deviceId)
```

**参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| deviceId | `number` | 是 | 要解绑的设备 ID |
| callback | `function(err, deviceId)` | 否 | 回调函数 |

**返回值**：

| 类型 | 说明 |
|------|------|
| `Promise<number>` | 解绑的设备 ID 的 Promise |
| `void` | 如果传入 callback 则无返回值 |

**使用示例**：

```javascript
import driver from '@ohos.driver.deviceManager';

try {
    await driver.deviceManager.unbindDevice(deviceId);
    console.log('Unbind success');
} catch (error) {
    console.error('Unbind failed:', error.code, error.message);
}
```

## bindDriverWithDeviceId()

按设备 ID 绑定特定驱动。与 `bindDevice()` 不同，此方法允许显式指定要绑定的驱动（通过设备 ID 隐含驱动信息）。

```javascript
driver.deviceManager.bindDriverWithDeviceId(deviceId, callback)
driver.deviceManager.bindDriverWithDeviceId(deviceId)
```

**参数和返回值**：与 `bindDevice()` 相同。

## unbindDriverWithDeviceId()

按设备 ID 解绑驱动。

```javascript
driver.deviceManager.unbindDriverWithDeviceId(deviceId, callback)
driver.deviceManager.unbindDriverWithDeviceId(deviceId)
```

**参数和返回值**：与 `unbindDevice()` 相同。

## queryDeviceInfo()

查询设备的详细信息，包括设备 ID、匹配状态、驱动 UID、USB 属性等。

```javascript
driver.deviceManager.queryDeviceInfo()
driver.deviceManager.queryDeviceInfo(deviceId)
```

**参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| deviceId | `number` | 否 | 可选的设备 ID，如果不传则返回所有设备信息 |

**返回值**：

| 类型 | 说明 |
|------|------|
| `Promise<DeviceInfo[]>` | 设备信息数组的 Promise |

**DeviceInfo 对象结构**：

| 字段 | 类型 | 说明 |
|------|------|------|
| deviceId | `number` | 设备唯一标识符 |
| isDriverMatched | `boolean` | 是否已匹配驱动 |
| driverUid | `string` | 匹配的驱动 UID（如果已匹配） |
| vendorId | `number` | USB 厂商 ID |
| productId | `number` | USB 产品 ID |
| interfaceDescList | `USBInterfaceDesc[]` | USB 接口描述列表 |

**USBInterfaceDesc 对象结构**：

| 字段 | 类型 | 说明 |
|------|------|------|
| bInterfaceNumber | `number` | 接口号 |
| bClass | `number` | 接口类代码 |
| bSubClass | `number` | 接口子类代码 |
| bProtocol | `number` | 接口协议代码 |

**使用示例**：

```javascript
import driver from '@ohos.driver.deviceManager';

try {
    const deviceInfos = await driver.deviceManager.queryDeviceInfo();
    deviceInfos.forEach(info => {
        console.log(`Device: ${info.deviceId}, Matched: ${info.isDriverMatched}`);
        if (info.driverUid) {
            console.log(`  Driver: ${info.driverUid}`);
        }
        console.log(`  VID: 0x${info.vendorId.toString(16)}, PID: 0x${info.productId.toString(16)}`);
        info.interfaceDescList.forEach(iface => {
            console.log(`  Interface ${iface.bInterfaceNumber}: Class ${iface.bClass}.${iface.bSubClass}`);
        });
    });
} catch (error) {
    console.error('Query device info failed:', error.code, error.message);
}
```

## queryDriverInfo()

查询已安装驱动扩展的信息。

```javascript
driver.deviceManager.queryDriverInfo()
driver.deviceManager.queryDriverInfo(driverUid)
```

**参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| driverUid | `string` | 否 | 可选的驱动 UID，如果不传则返回所有驱动信息 |

**返回值**：

| 类型 | 说明 |
|------|------|
| `Promise<DriverInfo[]>` | 驱动信息数组的 Promise |

**DriverInfo 对象结构**：

| 字段 | 类型 | 说明 |
|------|------|------|
| busType | `BusType` | 驱动支持的总线类型 |
| driverUid | `string` | 驱动唯一标识符 |
| driverName | `string` | 驱动名称 |
| bundleSize | `string` | 驱动包大小 |
| version | `string` | 驱动版本 |
| description | `string` | 驱动描述 |
| vids | `number[]` | 支持的 USB 厂商 ID 列表（USB 驱动） |
| pids | `number[]` | 支持的 USB 产品 ID 列表（USB 驱动） |

**使用示例**：

```javascript
import driver from '@ohos.driver.deviceManager';

try {
    const drivers = await driver.deviceManager.queryDriverInfo();
    drivers.forEach(driver => {
        console.log(`Driver: ${driver.driverName} (${driver.driverUid})`);
        console.log(`  Version: ${driver.version}`);
        console.log(`  Bundle: ${driver.bundleSize}`);
        if (driver.vids) {
            console.log(`  VIDs: ${driver.vids.map(v => '0x' + v.toString(16)).join(', ')}`);
        }
        if (driver.pids) {
            console.log(`  PIDs: ${driver.pids.map(p => '0x' + p.toString(16)).join(', ')}`);
        }
    });
} catch (error) {
    console.error('Query driver info failed:', error.code, error.message);
}
```

## 错误码参考

模块定义以下错误码，在所有 N-API 方法中统一使用：

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 201 | `PERMISSION_DENIED` | 权限被拒绝 |
| 202 | `PERMISSION_NOT_SYSTEM_APP` | 非系统应用调用系统 API |
| 401 | `PARAMETER_ERROR` | 参数错误 |
| 22900001 | `SERVICE_EXCEPTION` | 服务异常 |
| 26300001 | `SERVICE_EXCEPTION_NEW` | 服务异常（新） |
| 26300002 | `SERVICE_NOT_ALLOW_ACCESS` | 服务不允许访问 |
| 26300003 | `SERVICE_NOT_BOUND` | 没有绑定关系 |

## 完整使用示例

```javascript
import driver from '@ohos.driver.deviceManager';

class DeviceManagerDemo {
    constructor() {
        this.connectedDevices = new Map();
    }

    // 列出所有 USB 设备
    async listUsbDevices() {
        try {
            const devices = await driver.deviceManager.queryDevices(driver.deviceManager.BusType.USB);
            console.log(`Found ${devices.length} USB device(s):`);
            devices.forEach(device => {
                console.log(`  - Device ID: ${device.deviceId}`);
                console.log(`    VID: 0x${device.vendorId.toString(16)}, PID: 0x${device.productId.toString(16)}`);
                console.log(`    Description: ${device.description}`);
            });
            return devices;
        } catch (error) {
            console.error('Failed to list devices:', error);
            throw error;
        }
    }

    // 绑定设备
    async bindDevice(deviceId) {
        try {
            const result = await driver.deviceManager.bindDevice(deviceId);
            console.log(`Device ${deviceId} bound successfully`);
            this.connectedDevices.set(deviceId, result.remote);
            return result.remote;
        } catch (error) {
            console.error(`Failed to bind device ${deviceId}:`, error);
            throw error;
        }
    }

    // 解绑设备
    async unbindDevice(deviceId) {
        try {
            await driver.deviceManager.unbindDevice(deviceId);
            console.log(`Device ${deviceId} unbound`);
            this.connectedDevices.delete(deviceId);
        } catch (error) {
            console.error(`Failed to unbind device ${deviceId}:`, error);
            throw error;
        }
    }

    // 获取设备详细信息
    async getDeviceInfo(deviceId) {
        try {
            const infos = await driver.deviceManager.queryDeviceInfo(deviceId);
            if (infos.length > 0) {
                return infos[0];
            }
            return null;
        } catch (error) {
            console.error(`Failed to get device info for ${deviceId}:`, error);
            throw error;
        }
    }
}

export default new DeviceManagerDemo();
```

## 相关文档

| 文档 | 描述 |
|------|------|
| [00_Overview.md](./00_Overview.md) | 项目概览与核心能力 |
| [01_Directory_Structure.md](./01_Directory_Structure.md) | 目录结构与模块职责 |
| [02_Architecture.md](./02_Architecture.md) | 架构设计与组件关系 |
| [04_DDK_Reference.md](./04_DDK_Reference.md) | DDK C API 接口参考 |
| [05_Inner_API.md](./05_Inner_API.md) | 内部模块接口参考 |
