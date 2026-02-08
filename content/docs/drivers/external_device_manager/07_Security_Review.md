# 安全风险分析

本文档对扩展外部设备管理模块进行安全风险评审，识别潜在的攻击面、信任边界和安全风险点，并提供相应的修复建议。评审基于代码静态分析，重点关注输入验证、权限控制、接口安全等方面。

## 信任边界与数据流

### 信任边界定义

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                    信任边界                                           │
│  ┌─────────────────────────────────────────────────────────────────────────────┐  │
│  │                        hdf_ext_devmgr 进程边界                                │  │
│  │                                                                              │  │
│  │  边界内：DriverExtensionManager 服务、ExtDeviceManager、DriverPkgManager       │  │
│  │                                                                              │  │
│  └─────────────────────────────────────────────────────────────────────────────┘  │
│                        ▲                            │                            │
│                        │                            │                            │
│              N-API 调用 │                            │ IPC 调用                  │
│                        │                            │                            │
│                        ▼                            ▼                            │
│  ┌──────────────────────────┐              ┌──────────────────────────┐         │
│  │       应用进程边界         │              │      驱动扩展边界         │         │
│  │                          │              │                          │         │
│  │  • 三方 HAP 应用         │              │  • 驱动扩展 Ability     │         │
│  │  • 系统应用              │              │  • DDK 调用            │         │
│  │                          │              │                          │         │
│  └──────────────────────────┘              └──────────────────────────┘         │
│                                                                                   │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

扩展外部设备管理模块的信任边界分为三个层次。第一层是 `hdf_ext_devmgr` 服务进程边界，这是模块的核心可信区域，运行着设备管理、驱动匹配和包管理等核心逻辑。第二层是应用进程边界，包括三方 HAP 应用和系统应用，它们通过 N-API 调用服务接口。第三层是驱动扩展边界，运行在独立进程中的驱动扩展 Ability，通过 IPC 与服务通信。

### 数据流分析

模块的主要数据流包括：设备枚举数据流（从 USB 内核驱动到应用）、驱动绑定数据流（从应用到驱动扩展 Ability）、DDK 调用数据流（从驱动扩展到硬件设备）。每条数据流都经过不同层次的信任边界检查。

## 攻击面分析

### N-API 接口攻击面

N-API 接口是应用层与系统服务交互的主要入口，存在以下潜在攻击面：

**参数注入风险**：N-API 层接收来自 JavaScript 的参数，这些参数可能包含恶意构造的数据。参数通过 `napi_get_cb_info()` 获取后，需要进行类型和范围检查。当前代码在 `QueryDevice()` 等方法中实现了参数校验，但需确认所有入口点的校验完整性。

**回调函数处理风险**：设备绑定操作使用回调机制，回调函数指针通过 IPC 传递到服务端。服务端需要验证回调的合法性，防止恶意回调的执行。当前代码使用 `IDriverExtMgrCallback` 接口进行回调规范化，但仍需关注异常回调的处理逻辑。

**Promise 状态管理风险**：N-API 使用 Promise 处理异步操作，`AsyncData` 结构管理异步状态。需要在回调完成、取消和异常等各种场景下正确维护 Promise 状态，防止状态不一致。

### IPC 接口攻击面

IPC 接口是跨进程通信的关键通道，存在以下潜在攻击面：

**Parcel 序列化风险**：所有跨进程传输的数据都使用 Parcel 序列化机制。`Unmarshalling()` 方法需要验证数据的完整性和格式正确性，防止恶意构造的序列化数据导致解析错误或内存破坏。当前代码中的 `DeviceData`、`DeviceInfoData` 等类型实现了 `Unmarshalling()`，但需验证边界检查的完整性。

**接口调用码路由风险**：IPC 接口使用 `DriverExtMgrInterfaceCode` 枚举进行方法路由。恶意构造的调用码可能触发未定义行为，需要在框架层验证调用码的有效性。

**回调对象传递风险**：`BindDevice()` 等方法接受回调对象作为参数，回调对象包含远程对象引用。需要验证回调对象的生命周期管理，防止悬空引用导致的崩溃。

### DDK 接口攻击面

DDK 接口允许驱动扩展直接访问硬件，存在以下潜在攻击面：

**设备句柄管理风险**：DDK API 使用设备句柄（handle）进行设备操作。句柄如果被恶意复用或伪造，可能导致未授权设备访问。当前代码使用 `interfaceHandle`、`deviceId` 等标识符，需要验证句柄与调用者绑定的正确性。

**缓冲区溢出风险**：DDK API 中的数据传输函数（如 `OH_Usb_Read()`、`OH_Hid_Write()`）使用缓冲区参数。如果缓冲区大小参数被恶意构造，可能导致缓冲区溢出。当前代码在 DDK 层实现了大小校验，但需确认所有传输路径的完整性。

**共享内存使用风险**：Base DDK 提供 Ashmem 共享内存机制。共享内存的创建、映射和销毁需要在多进程场景下正确同步，防止竞争条件导致的内存安全问题。

### 文件系统攻击面

模块使用文件系统存储持久化数据，存在以下潜在攻击面：

**数据库文件安全**：驱动元数据存储在 SQLite 数据库中。数据库文件的权限设置需要防止未授权访问，SQL 查询需要防止注入攻击。

**配置文件解析风险**：模块解析 JSON 配置文件（如 `peripheral_fault_notifier_config.json`）。恶意构造的配置文件可能导致解析器行为异常。

## 安全风险清单

### 风险 1：N-API 参数验证不完整

**风险等级**：中

**风险描述**：部分 N-API 方法可能未对所有输入参数进行完整的范围和类型验证。恶意构造的输入可能导致服务端异常或逻辑错误。

**证据位置**：
- `device_manager_middle.cpp` 中部分参数检查使用 `argc` 判断参数数量，但未对每个参数的类型和范围进行严格校验
- `QueryDevice()` 方法的 `busType` 参数未验证是否为有效的 `BusType` 枚举值

**触发条件**：
- 调用 N-API 时传入非法参数值
- 参数类型不匹配（如传入字符串替代数字）

**影响范围**：
- 服务端可能的异常终止
- 设备管理逻辑错误

**修复建议**：
- 在每个 N-API 方法入口处添加完整的参数验证逻辑
- 使用 `IsMatchType()` 函数验证参数类型
- 对枚举类型参数添加范围检查

### 风险 2：回调引用未验证合法性

**风险等级**：中

**风险描述**：`BindDevice()` 等方法接受回调对象参数，服务端直接使用回调对象进行 IPC 调用。如果回调对象已被销毁或指向恶意进程，可能导致异常。

**证据位置**：
- `device_manager_middle.cpp` 第 105-107 行：`napi_get_reference_value()` 获取回调引用后直接调用
- 未验证回调对象是否属于有效的远程对象

**触发条件**：
- 传入已失效的回调引用
- 回调对象指向恶意进程

**影响范围**：
- IPC 调用失败
- 可能的进程间通信异常

**修复建议**：
- 在使用回调前验证远程对象的有效性
- 添加回调生命周期的超时管理机制

### 风险 3：Parcel 解析边界检查不完整

**风险等级**：中

**风险描述**：自定义数据类型的 `Unmarshalling()` 方法在解析 Parcel 数据时，可能未对所有字段进行边界检查。恶意构造的数据可能导致缓冲区越界读取或整数溢出。

**证据位置**：
- `DeviceData::Unmarshalling()` 方法对 `busType` 字段未验证是否在有效枚举范围内
- `USBDeviceInfoData` 的 `interfaceDescList` 解析未验证列表大小

**触发条件**：
- 发送恶意构造的 Parcel 数据
- 数据字段值超出预期范围

**影响范围**：
- 内存读取越界
- 整数溢出导致的异常行为

**修复建议**：
- 在 `Unmarshalling()` 方法中添加完整的字段验证
- 对列表/数组类型字段添加大小限制

### 风险 4：设备句柄管理存在 TOCTOU 竞争

**风险等级**：低

**风险描述**：设备操作使用句柄标识符，在多线程场景下存在 Time-Of-Check-Time-Of-Use（TOCTOU）竞争条件。句柄验证和使用之间的时间窗口可能被利用。

**证据位置**：
- `usb_ddk_api.cpp` 中设备操作未对句柄进行原子性验证
- 多线程调用 `OH_Usb_*` API 时存在句柄竞争

**触发条件**：
- 多线程并发调用 DDK API
- 一个线程关闭句柄，另一个线程同时使用

**影响范围**：
- 使用已释放的句柄
- 进程崩溃

**修复建议**：
- 使用原子操作管理句柄引用计数
- 在 API 入口处添加句柄有效性验证

### 风险 5：共享内存引用计数管理

**风险等级**：低

**风险描述**：Base DDK 的 Ashmem 操作使用全局映射表管理共享内存。引用计数如果管理不当，可能导致内存泄漏或-use-after-free。

**证据位置**：
- `ddk_api.cpp` 第 28 行：全局映射表 `g_shareMemoryMap` 使用互斥锁保护
- `OH_DDK_DestroyAshmem()` 从映射表中移除条目

**触发条件**：
- 异常路径下未正确清理映射表条目
- 多线程并发访问未完全同步

**影响范围**：
- 内存泄漏
- 悬空指针访问

**修复建议**：
- 使用智能指针管理 Ashmem 生命周期
- 在析构函数中确保清理

### 风险 6：权限验证错误码信息泄露

**风险等级**：低

**风险描述**：权限验证失败时返回的错误信息可能泄露系统内部状态，如驱动是否存在、权限配置等敏感信息。

**证据位置**：
- `edm_errors.h` 中定义了详细的错误码枚举
- 错误码区分了「权限不足」和「驱动不存在」等状态

**触发条件**：
- 枚举探测攻击
- 收集错误响应推断系统状态

**影响范围**：
- 信息泄露（低风险）

**修复建议**：
- 对外部调用统一返回通用错误码
- 详细错误信息仅记录到日志

### 风险 7：USB 设备 VID/PID 匹配逻辑

**风险等级**：低

**风险描述**：驱动匹配算法使用 VID/PID 列表进行设备匹配。如果列表配置不当，可能导致错误匹配或绕过检测。

**证据位置**：
- `UsbDriverInfo` 的 `Serialize()`/`UnSerialize()` 方法处理 VID/PID 列表
- 匹配逻辑在 `UsbBusExtension::MatchDriver()` 中实现

**触发条件**：
- 恶意设备伪装成合法 VID/PID
- 驱动声明的匹配列表过于宽泛

**影响范围**：
- 错误驱动加载
- 设备功能异常

**修复建议**：
- 考虑添加设备签名验证机制
- 限制 VID/PID 匹配的范围

### 风险 8：系统事件日志敏感信息

**风险等级**：低

**风险描述**：HiSysEvent 事件日志记录了设备连接、驱动匹配等操作信息，可能包含设备标识符、bundle 名称等敏感数据。

**证据位置**：
- `hisysevent.yaml` 定义了 `EXT_DEVICE_EVENT` 事件，包含 `DEVICE_ID`、`DRIVER_UID` 等字段
- `driver_report_sys_event.cpp` 实现事件上报

**触发条件**：
- 日志文件被未授权访问
- 日志聚合系统泄露数据

**影响范围**：
- 敏感信息泄露

**修复建议**：
- 对敏感字段进行脱敏处理
- 配置日志访问控制

## 安全机制说明

### 权限控制机制

模块实现了多层次的权限控制机制：

**权限验证入口**：`ExtPermissionManager` 类提供统一的权限验证接口。

```cpp
class ExtPermissionManager {
public:
    static bool VerifyPermission(std::string permissionName);
    static bool IsSystemApp();
    static bool IsSa();
    static uint32_t GetCallingTokenID();
    static bool GetPermissionValues(const std::string &permission, 
        unordered_set<std::string> &accessibleBundles);
};
```

**权限检查模式**：

| 方法 | 权限要求 | 检查位置 |
|------|---------|---------|
| `QueryDevice()` | `ACCESS_EXTENSIONAL_DEVICE_DRIVER` | `driver_ext_mgr.cpp:127` |
| `BindDevice()` | `ACCESS_EXTENSIONAL_DEVICE_DRIVER` | `driver_ext_mgr.cpp:167` |
| `BindDriverWithDeviceId()` | `ACCESS_DDK_DRIVERS` | `driver_ext_mgr.cpp:203` |
| `QueryDeviceInfo()` | 系统应用或权限 | `driver_ext_mgr.cpp:326` |
| `QueryDriverInfo()` | 系统应用或权限 | `driver_ext_mgr.cpp:358` |
| `NotifyUsbPeripheralFault()` | SA 调用 | `driver_ext_mgr.cpp:391` |

### 令牌管理机制

模块使用 AccessTokenKit 进行令牌管理：

```cpp
// 获取调用者令牌
uint32_t callerToken = IPCSkeleton::GetCallingTokenID();

// 验证访问令牌
bool hasPermission = AccessTokenKit::VerifyAccessToken(callerToken, permissionName);

// 检查系统应用
bool isSystemApp = TokenIdKit::IsSystemAppByFullTokenID(fullTokenId);
```

### 死亡通知机制

模块对关键系统服务注册死亡通知：

| 服务 | 死亡通知接收者 | 处理动作 |
|------|--------------|---------|
| Bundle Manager Service | `BundleMgrDeathRecipient` | 清理 BMS 相关状态 |
| USB HDI Service | `UsbdDeathRecipient` | 触发 SA 卸载 |
| USB Serial DDK | `UsbSerialDeathRecipient` | 清理设备句柄 |
| HID DDK | `HidDeathRecipient` | 清理设备状态 |
| SCSI DDK | `ScsiPeripheralDeathRecipient` | 清理设备状态 |

## 安全审计结论

经过对扩展外部设备管理模块的安全评审，识别出 8 个潜在风险点，风险等级均为低至中。模块实现了基础的权限控制、令牌管理和死亡通知机制，但部分边界检查和竞态条件处理仍有改进空间。建议在后续迭代中重点关注参数验证的完整性和共享资源管理的线程安全性。

## 相关文档

| 文档 | 描述 |
|------|------|
| [00_Overview.md](./00_Overview.md) | 项目概览与核心能力 |
| [01_Directory_Structure.md](./01_Directory_Structure.md) | 目录结构与模块职责 |
| [02_Architecture.md](./02_Architecture.md) | 架构设计与组件关系 |
| [03_NAPI_Reference.md](./03_NAPI_Reference.md) | JS API 接口参考 |
| [04_DDK_Reference.md](./04_DDK_Reference.md) | DDK C API 接口参考 |
| [05_Inner_API.md](./05_Inner_API.md) | 内部模块接口参考 |
| [08_Troubleshooting.md](./08_Troubleshooting.md) | 常见问题与调试 |
