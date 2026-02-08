# N-API 接口说明

## 重要说明

**本仓库不涉及传统 N-API（Node-API）模式。** HATS 是硬件抽象层测试套件，主要测试 HDI（Hardware Driver Interface）接口，而非 JS API 绑定层。

### 原因分析

| 方面 | 说明 |
|------|------|
| **测试目标** | HAL/HDI 层硬件驱动兼容性 |
| **开发语言** | C/C++（不使用 JavaScript/TypeScript） |
| **测试框架** | GoogleTest (gtest)，非 JS 单元测试 |
| **接口类型** | Native C++ 接口，非 JS-to-Native 桥接 |

> **证据**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/README.md:4-8` - "HATS 帮助设备厂商发现 HAL 软件不兼容性"

---

## HDI 接口替代方案

虽然 HATS 不使用 N-API，但其测试的 HDI 接口遵循类似的设计模式。

### 接口注册模式对比

| 层级 | HDI（C++） | N-API（JS） |
|------|------------|-------------|
| **注册** | `IXXXInterface::Get()` | `NAPI_MODULE` |
| **绑定** | 模板特化 | `napi_define_properties` |
| **调用** | 直接方法调用 | `napi_call_function` |
| **回调** | C++ 回调对象 | `napi_callback` |

### HDI 接口示例

```cpp
// HDI 接口获取模式（替代 NAPI_MODULE）
// 文件: /Volumes/lexar/code/d/work/oh/test/xts/hats/useriam/fingerprintauth/src/fingerprint_auth_hdi.cpp

// 获取接口实例
sptr<IFingerprintAuth> g_fingerprintInterface = IFingerprintAuth::Get();

// 回调注册（替代 napi_create_function）
int32_t ret = g_fingerprintInterface->Register(g_fingerprintCallback);

// 异步调用（替代 napi_make_callback）
int32_t ret = g_fingerprintInterface->Authenticate(scheduleId, templateIdList,
                                                     extraInfo, callback);
```

---

## 发现的唯一 N-API 相关引用

### 在 HDF external_device_manager 中发现

| 文件 | 行号 | 引用内容 |
|------|------|----------|
| `/Volumes/lexar/code/d/work/oh/test/xts/hats/hdf/external_device_manager/drivers_pkg_manager_test/BUILD.gn` | 28 | `libnapi_common` 依赖 |

```gn
deps = [
    "access_token:libaccesstoken_sdk",
    "//foundation/ability/ability_runtime/interfaces/inner_api/native:napi_common",  # ← 唯一 N-API 引用
]
```

**分析**：此依赖用于访问权限检查相关功能，非核心测试逻辑。

---

## 进程间通信（IPC）模式

HATS 主要使用以下 IPC 机制：

### 1. Binder IPC

```cpp
// 文件: /Volumes/lexar/code/d/work/oh/test/xts/hats/hdf/manager/managerServiceTest/service_manager_hdi_test.cpp:148-157

OHOS::MessageParcel data;
OHOS::MessageParcel reply;
OHOS::MessageOption option;

// 写入接口令牌
bool ret = data.WriteInterfaceToken(TEST_SERVICE_INTERFACE_DESC);
data.WriteCString("sample_service test call");

// 发送请求
int status = sampleService->SendRequest(SAMPLE_SERVICE_PING, data, reply, option);
```

### 2. System Ability Manager

```cpp
// 文件: /Volumes/lexar/code/d/work/oh/test/xts/hats/hdf/external_device_manager/drivers_pkg_manager_test/driver_pkg_manager_test.cpp:61-124

// 获取 SAMgr
sptr<ISystemAbilityManager> samgr = SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();

// 加载系统能力
int32_t ret = samgr->LoadSystemAbility(HDF_EXTERNAL_DEVICE_MANAGER_SA_ID, loadCallback_);
```

### 3. HDF IoService

```cpp
// 文件: /Volumes/lexar/code/d/work/oh/test/xts/hats/hdf/hdf_lite/manager/common/hdf_ioservice_test.cpp:187-259

// 注册事件监听器
ret = HdfIoServiceGroupRegisterListener(group, &listener0.listener);

// 事件回调
static int OnDevEventReceived(
    struct HdfDevEventlistener *listener,
    struct HdfIoService *service,
    uint32_t id,
    struct HdfSBuf *data)
{
    const char *string = HdfSbufReadString(data);
    return 0;
}
```

---

## 接口清单表

### HDF 驱动接口

| 接口名称 | 命名空间 | 版本 | 头文件 |
|----------|----------|------|--------|
| `IAudioManager` | `OHOS::HDI::Audio` | V1_0 | `audio/interfaces/inner/` |
| `ICameraHost` | `OHOS::HDI::Camera` | V1_1 | `camera/interfaces/inner/` |
| `ISensorInterface` | `OHOS::HDI::Sensor` | V1_0-V2_2 | `sensor/interfaces/` |
| `IDisplayComposer` | `OHOS::HDI::Display::Composite` | V1_0-V1_2 | `display/interfaces/` |
| `IUsbInterface` | `OHOS::HDI::Usb` | V2_0-V2_1 | `usb/interfaces/` |
| `IVibratorInterface` | `OHOS::HDI::Vibrator` | V1_0-V2_0 | `vibrator/interfaces/` |
| `ILightInterface` | `OHOS::HDI::Light` | V1_0 | `light/interfaces/` |

### 电源管理接口

| 接口名称 | 命名空间 | 版本 | 关键方法 |
|----------|----------|------|----------|
| `IPowerInterface` | `OHOS::HDI::Power` | V1_2-V1_3 | `StartSuspend/StopSuspend/HoldRunningLock` |
| `IThermalInterface` | `OHOS::HDI::Thermal` | V1_1 | `SetCpuFreq/GetThermalZoneInfo` |
| `IBatteryInterface` | `OHOS::HDI::Battery` | V2_0 | `GetSOC/GetVoltage/GetTemperature` |

### 用户认证接口

| 接口名称 | 命名空间 | 版本 | 关键方法 |
|----------|----------|------|----------|
| `IUserAuthInterface` | `OHOS::HDI::UserAuth` | V4_1 | `Init/AddExecutor/BeginEnrollment` |
| `IFingerprintAuth` | `OHOS::HDI::FingerprintAuth` | V1_0 | `Enroll/Authenticate/Delete` |
| `IFaceAuth` | `OHOS::HDI::FaceAuth` | V1_0 | `BeginEnrollment/BeginAuthentication` |
| `IPinAuth` | `OHOS::HDI::PinAuth` | V1_0 | `GetPinData/Unpin` |

### AI 推理接口

| 接口名称 | 命名空间 | 版本 | 关键方法 |
|----------|----------|------|----------|
| `INNRTDevice` | `OHOS::HDI::Nnrt` | V1_0-V2_0 | `GetDeviceName/PrepareModel/Execute` |
| `IPreparedModel` | `OHOS::HDI::Nnrt` | V2_0 | `Execute/GetInputBuffer/GetOutputBuffer` |

### 电信接口

| 接口名称 | 命名空间 | 版本 | 关键方法 |
|----------|----------|------|----------|
| `IRil` | `OHOS::HDI::Ril` | V1_0-V1_5 | `Dial/SendSms/GetSignalStrength` |

---

## 错误码规范

### 通用错误码

| 错误码 | 宏定义 | 说明 |
|--------|--------|------|
| 0 | `HDF_SUCCESS` | 操作成功 |
| -1 | `HDF_FAILURE` | 操作失败 |
| -2 | `HDF_ERR_INVALID_PARAM` | 参数无效 |
| -3 | `HDF_ERR_NULL_OBJECT` | 空对象 |

### 权限错误码

| 错误码 | 说明 |
|--------|------|
| `PERMISSION_DENIED` | 权限拒绝 |
| `TOKEN_INVALID` | 令牌无效 |
| `BUNDLE_NAME_INVALID` | 包名无效 |

> **证据**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/kernel/accesstokenid/accesstokenid_test.cpp` - AccessToken 权限测试

---

## 相关文档

| 文档 | 路径 | 说明 |
|------|------|------|
| 概览 | [00_Overview.md](./00_Overview.md) | 项目定位 |
| 架构 | [01_Architecture.md](./01_Architecture.md) | 系统架构 |
| 模块 | [02_Modules.md](./02_Modules.md) | 子系统详解 |
| 构建 | [04_Build.md](./04_Build.md) | GN 构建配置 |
| 安全 | [05_Security.md](./05_Security.md) | 安全风险 |
