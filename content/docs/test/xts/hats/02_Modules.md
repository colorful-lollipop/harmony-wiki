# 子系统模块详解

## 子系统概览

| 子系统 | BUILD.gn | C/C++ | 主要职责 |
|--------|----------|-------|----------|
| [HDF](#hdf-硬件驱动框架) | 154 | 393 | 硬件驱动接口测试 |
| [Kernel](#kernel-内核) | 307 | 172 | 系统调用测试 |
| [UserIAM](#useriam-用户认证) | 10 | 23 | 用户身份认证测试 |
| [AI](#ai-神经网络运行时) | 9 | 16 | AI 推理测试 |
| [Powermgr](#powermgr-电源管理) | 13 | 9 | 电源/热管理测试 |
| [Telephony](#telephony-电信) | 4 | 10 | RIL 通信测试 |
| [DistributedHardware](#distributedhardware-分布式硬件) | 3 | 6 | 分布式设备测试 |
| [Startup](#startup-启动) | 3 | 2 | 分区启动测试 |

---

## HDF（硬件驱动框架）

### 模块结构

```
hdf/
├── audio/              # 音频服务（render/capture/manager）
├── bluetooth/          # 蓝牙 HCI/A2DP
├── camera/            # 相机 HAL 测试
├── codec/             # 编解码器（OMX/JPEG）
├── display/           # 显示合成器、Buffer 管理
├── external_device_manager/  # USB DDK、驱动扩展
├── input/             # 输入设备（触摸/键盘）
├── light/             # LED/灯光控制
├── location/          # GNSS/AGPS/地理围栏
├── manager/           # HDF 服务管理器
├── motion/            # 运动传感器
├── nfc/              # NFC/安全元件
├── sensor/            # 各类传感器
├── usb/              # USB 设备/主机
├── vibrator/         # 触觉反馈
└── wlan/             # WiFi 无线
```

### 关键接口

| 接口 | 版本 | 路径 |
|------|------|------|
| `IAudioManager` | V1_0 | `audio/interfaces/inner/` |
| `ICameraHost` | V1_1 | `camera/interfaces/inner/` |
| `ISensorInterface` | V1_0-V2_2 | `sensor/interfaces/` |
| `IDisplayComposer` | V1_0-V1_2 | `display/interfaces/` |
| `IUsbInterface` | V2_0-V2_1 | `usb/interfaces/` |

### 测试用例模式

```cpp
// 音频测试
#include "audio_manager_common_test.cpp"
// 测试项: SUB_Driver_Audio_Manager_xxx

// 显示 Buffer 死亡测试
#include "death_test.cpp"
// 测试项: SUB_Driver_Display_Buffer_Death_xxx

// 传感器回调测试
#include "hdf_sensor_hdiService_test.cpp"
// 测试项: SUB_Driver_Sensor_Hdi_xxx
```

> **证据**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/hdf/audio/audio.gni` - 音频测试聚合

---

## Kernel（内核）

### 模块结构

```
kernel/
├── accesstokenid/     # 访问令牌 ID 测试
├── dmabuffer/         # DMA Buffer 测试
├── freelist/          # 内存释放列表安全
├── libmeminfoPc/      # 内存信息
├── madvise/           # 内存管理建议
├── memtracker/        # 内存追踪
├── mmap/              # 内存映射
├── mmap_v/            # 内存映射变体
├── open_posix_testsuite/  # POSIX 一致性
├── posix_interface/   # POSIX 接口
├── prctl/             # 进程控制
├── purgeableMem/      # 可释放内存
├── rtginterface/      # 实时调度
├── syscall_ipc/       # SysV IPC（sem/shm/msg）
├── syscalls/          # 系统调用（fileio/mem/process...）
```

### 核心测试领域

| 领域 | 测试文件 | 关键 API |
|------|----------|----------|
| 文件 I/O | `fileio/` | `open/read/write/close` |
| 内存管理 | `mem/` | `mmap/mprotect/brk` |
| 进程控制 | `process/` | `fork/exec/wait` |
| 信号处理 | `signal/` | `signal/sigaction/kill` |
| 进程调度 | `schedule/` | `sched_setscheduler/nice` |
| SysV IPC | `syscall_ipc/` | `semget/shmget/msgget` |
| 用户/组 | `user/` | `getuid/setuid/getgid` |

### 测试用例示例

```cpp
// UID/GID 测试
HWTEST_F(UserApiTest, GetgidReturnActualGroupIDSuccess_0004, Function | MediumTest | Level1)
{
    gid_t gid = getgid();
    EXPECT_NE(gid, -1);
    struct passwd *pw = getpwuid(gid);
    EXPECT_TRUE(pw != nullptr);
}

// 权限提升测试
HWTEST_F(UserApiTest, SetuidRootChangeUserIDSuccess_0011, Function | MediumTest | Level1)
{
    uid_t uid = getuid();
    int32_t ret = setuid(0);  // 尝试获取 root 权限
    EXPECT_EQ(ret, 0);
}
```

> **证据**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/kernel/syscalls/user/UserApiTest.cpp:68-82`

---

## UserIAM（用户认证）

### 模块结构

```
useriam/
├── faceauth/              # 人脸识别
├── faceauth_additional/   # 扩展人脸测试
├── fingerprintauth/        # 指纹识别
├── fingerprintauth_additional/
├── pinauth/               # PIN 码认证
├── pinauth_additional/
├── userauth/              # 用户认证编排
├── userauth_additional/
└── common/                # 公共测试工具
```

### 安全敏感接口

| 模块 | 接口 | 关键操作 |
|------|------|----------|
| **userauth** | `IUserAuthInterface` | `Init/AddExecutor/DeleteUser/BeginEnrollment` |
| **fingerprintauth** | `IFingerprintAuth` | `Enroll/Authenticate/Delete/GetProperty` |
| **faceauth** | `IFaceAuth` | `BeginEnrollment/BeginAuthentication` |
| **pinauth** | `IPinAuth` | `GetPinData/Unpin` |

### 认证流程测试

```cpp
// 用户认证测试 (Security_IAM_UserAuth_HDI_FUNC_XXXX)
class UserAuthInterfaceService : public IUserAuthInterface {
    int32_t Init(const std::string& deviceUdid) override;
    int32_t AddExecutor(const std::vector<uint8_t>& info, int32_t& index,
                        std::vector<uint8_t>& publicKey,
                        std::vector<int64_t>& templateIds) override;
    int32_t DeleteUser(int32_t userId, int64_t authToken,
                        std::vector<int64_t>& deletedInfos) override;
};

// 指纹认证测试 (Security_IAM_Fingerprint_HDI_FUNC_XXXX)
class FingerprintAuthInterfaceService : public IFingerprintAuth {
    int32_t Enroll(int32_t scheduleId, const std::vector<uint8_t>& extraInfo,
                   const sptr<IRemoteObject>& callback) override;
    int32_t Authenticate(int32_t scheduleId, const std::vector<int64_t>& templateIdList,
                         const std::vector<uint8_t>& extraInfo,
                         const sptr<IRemoteObject>& callback) override;
};
```

> **证据**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/useriam/userauth/src/user_auth_hdi.cpp`

### 公共测试工具

```cpp
// iam_hat_test.cpp 提供的工具函数
void FillTestBuffer(Parcel &parcel, void *p, uint32_t len);
void FillTestUint8Vector(Parcel &parcel, std::vector<uint8_t> &data);
void FillTestString(Parcel &parcel, std::string &str);
void FillTestInt32Vector(Parcel &parcel, std::vector<int32_t> &data);
```

---

## AI（神经网络运行时）

### 模块结构

```
ai/
└── nnrt/
    └── hdi/
        ├── v1_0/           # NNRT HDI v1.0
        │   ├── nnrtFunctionTest/
        │   └── nnrtStabilityTest/
        └── v2_0/           # NNRT HDI v2.0（当前版本）
            ├── nnrtFunctionTest/
            ├── nnrtFunctionTest_additional/
            └── nnrtStabilityTest/
```

### 关键接口（V2_0）

```cpp
#include <v2_0/innrt_device.h>

class INNRTDevice {
    // 设备信息
    int32_t GetDeviceName(std::string& deviceName);
    int32_t GetVendorName(std::string& vendorName);
    int32_t GetDeviceType(int32_t& deviceType);  // CPU/GPU/ACCELERATOR/OTHER
    int32_t GetDeviceStatus(int32_t& deviceStatus);  // AVAILABLE/BUSY/OFFLINE

    // 能力查询
    int32_t IsFloat16PrecisionSupported(bool& isSupported);
    int32_t IsPerformanceModeSupported(bool& isSupported);
    int32_t IsDynamicInputSupported(bool& isSupported);
    int32_t IsModelCacheSupported(bool& isSupported);

    // 内存管理
    int32_t AllocateBuffer(uint32_t size, std::vector<nnabuffer_buffer>& buffer);
    int32_t ReleaseBuffer(const std::vector<nnabuffer_buffer>& buffer);

    // 模型操作
    int32_t GetSupportedOperation(const Model& model,
                                  std::vector<bool>& supportedOperations);
    int32_t PrepareModel(const Model& model, const ModelConfig& config,
                         sptr<IPreparedModel>& preparedModel);
};

// 执行模型
class IPreparedModel {
    int32_t Execute(const std::vector<IOTensor>& inputs,
                    std::vector<IOTensor>& outputs,
                    const std::vector<IOTensor>& controlTiles);
};
```

### 测试用例分类

| 类别 | 前缀 | 说明 |
|------|------|------|
| 设备信息 | `SUB_AI_NNRt_Func_South_Device_DeviceInfo` | 设备名称/类型/状态 |
| 能力配置 | `SUB_AI_NNRt_Func_South_Device_DeviceConfig` | 精度/性能/动态输入支持 |
| 模型支持 | `SUB_AI_NNRt_Func_South_Device_ModelSupport` | 操作兼容性检查 |
| 内存管理 | `SUB_AI_NNRt_Func_South_Device_Memory` | Buffer 分配/释放 |
| 模型准备 | `SUB_AI_NNRt_Func_South_Device_PrepareModel` | 模型编译 |
| 模型执行 | `SUB_AI_NNRt_Func_South_Device_ExecuteModel` | 推理执行 |

---

## Powermgr（电源管理）

### 模块结构

```
powermgr/
├── power/           # 电源管理
│   ├── hdi_power/
│   ├── hdi_power_additional/
│   └── hdi_power_config/
├── thermal/         # 温控管理
│   ├── hdi_thermal/
│   ├── hdi_thermal_additional/
│   └── hdi_thermal_config/
└── battery/         # 电池管理
    ├── hdi_battery/
    ├── hdi_battery_additional/
    └── hdi_battery_config/
```

### 电源接口（V1_3）

```cpp
class IPowerInterface {
    // 挂起控制
    int32_t RegisterCallback(const sptr<IPowerCallback>& callback);
    int32_t StartSuspend();
    int32_t StopSuspend();

    // 唤醒锁
    int32_t SuspendBlock(const std::string& name);
    int32_t SuspendUnblock(const std::string& name);
    int32_t HoldRunningLock(const RunningLockInfo& info);
    int32_t UnholdRunningLock(const RunningLockInfo& info);

    // 休眠
    int32_t Hibernate();

    // 配置
    int32_t SetPowerConfig(PowerConfigType type, const std::string& value);
};
```

### 温控接口（V1_1）

```cpp
class IThermalInterface {
    // 频率控制
    int32_t SetCpuFreq(int32_t freq);
    int32_t SetGpuFreq(int32_t freq);
    int32_t SetBatteryCurrent(int32_t current);

    // 温度查询
    int32_t GetThermalZoneInfo(ThermalZoneInfo& info);

    // 回调
    int32_t Register(const sptr<IThermalCallback>& callback);
    int32_t Unregister();

    // CPU 隔离
    int32_t IsolateCpu(int32_t num);
};
```

### 电池接口

```cpp
class IBatteryInterface {
    int32_t GetBatteryInfo(BatteryInfo& info);
    int32_t GetSOC(int32_t& soc);
    int32_t GetVoltage(int32_t& voltage);
    int32_t GetTemperature(int32_t& temperature);
    int32_t GetPresent(bool& present);
};
```

---

## Telephony（电信）

### 模块结构

```
telephony/
└── ril/
    ├── hdi_v1.0/          # RIL HDI v1.0
    └── hdi_v1.1_additional/  # 扩展测试
```

### RIL 请求分类（HdiId）

| 范围 | 类别 | 典型操作 |
|------|------|----------|
| 0-99 | Call | Dial/Hangup/Answer/Hold/Conference |
| 100-199 | SMS | SendGsmSms/SendCdmaSms/CB 配置 |
| 200-299 | SIM | GetSimStatus/GetImsi/SimIO |
| 300-399 | Data | ActivatePdpContext/GetPdpContextList |
| 400-499 | Network | GetSignalStrength/GetCsRegStatus |
| 500+ | Common | Modem 公共操作 |

### 回调接口

```cpp
class IRilCallback {
    // 通话回调
    int32_t CallStateUpdated(const RilRadioResponseInfo& info) override;
    int32_t CallRingbackVoice(const RilRadioResponseInfo& info,
                              const RingbackVoice& voice) override;

    // 网络回调
    int32_t SignalStrengthUpdated(const RilRadioResponseInfo& info,
                                   const Rssi& rssi) override;
    int32_t CsRegStatusUpdated(const RilRadioResponseInfo& info,
                               const CsRegStatus& status) override;

    // SIM 回调
    int32_t SimStateUpdated(const RilRadioResponseInfo& info) override;

    // 数据回调
    int32_t PdpContextListUpdated(const RilRadioResponseInfo& info,
                                  const PdpContextList& list) override;
};
```

---

## DistributedHardware（分布式硬件）

### 模块结构

```
distributedhardware/
├── distributedcameratest/           # 分布式相机
└── distributedcameratest_additional/ # 扩展测试
```

### 测试焦点

- 分布式相机提供者注册
- 设备发现与配对
- 流数据传输
- 跨设备相机控制

---

## Startup（启动）

### 模块结构

```
startup/
├── partitionslot/           # 分区槽位测试
└── partitionslot_additional/
```

### 测试用例

```cpp
// 分区加载/卸载测试
auto devmgr = IDeviceManager::Get();
devmgr->LoadDevice(TEST_SERVICE_NAME);
devmgr->UnloadDevice(TEST_SERVICE_NAME);
```

---

## 相关文档

| 文档 | 路径 | 说明 |
|------|------|------|
| 概览 | [00_Overview.md](./00_Overview.md) | 项目定位 |
| 架构 | [01_Architecture.md](./01_Architecture.md) | 系统架构 |
| N-API | [03_N-API.md](./03_N-API.md) | HDI 接口清单 |
| 构建 | [04_Build.md](./04_Build.md) | GN 构建配置 |
| 安全 | [05_Security.md](./05_Security.md) | 安全风险 |
